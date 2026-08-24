---
title: Go + PostgreSQL 測試策略
type: concept
tags: [golang, postgresql, testing, testcontainers, integration-test, repository, sqlc, senior]
created: 2026-04-30
updated: 2026-04-30
---

# Go + PostgreSQL 測試策略

## 測試分層策略

```
┌─────────────────────────────────────────────────────────────┐
│  Layer             測試對象           使用 DB？  執行速度    │
├─────────────────────────────────────────────────────────────┤
│  Unit Test         Domain Logic        ❌ Mock    極快 <1ms  │
│  Unit Test         Application Service ❌ Mock    極快 <1ms  │
│  Integration Test  Repository          ✅ 真實    中 1-5s    │
│  Integration Test  DB Migration        ✅ 真實    中 5-30s   │
│  E2E Test          HTTP API            ✅ 真實    慢 >30s    │
└─────────────────────────────────────────────────────────────┘

核心原則：
- Domain / Application Layer → Mock Repository，測試純業務邏輯
- Repository Layer → 真實 PostgreSQL，測試 SQL 查詢的正確性
- 兩者都需要，互補不可取代
```

---

## 一、Mock Repository（Application Layer 測試）

Repository 抽象為介面，Application Service 的測試不需要真實 DB：

```go
// domain/order/repository.go
type OrderRepository interface {
    FindByID(ctx context.Context, id OrderID) (*Order, error)
    Save(ctx context.Context, order *Order) error
    FindByCustomerID(ctx context.Context, customerID CustomerID) ([]*Order, error)
}

// 手動 Mock（簡單場景）
type MockOrderRepository struct {
    orders map[OrderID]*Order
    SaveFn func(ctx context.Context, order *Order) error
}

func NewMockOrderRepository() *MockOrderRepository {
    return &MockOrderRepository{orders: make(map[OrderID]*Order)}
}

func (m *MockOrderRepository) FindByID(ctx context.Context, id OrderID) (*Order, error) {
    if o, ok := m.orders[id]; ok {
        return o, nil
    }
    return nil, ErrOrderNotFound
}

func (m *MockOrderRepository) Save(ctx context.Context, order *Order) error {
    if m.SaveFn != nil {
        return m.SaveFn(ctx, order)
    }
    m.orders[order.ID()] = order
    return nil
}

// Application Service 測試
func TestOrderService_PlaceOrder_Success(t *testing.T) {
    repo := NewMockOrderRepository()
    svc := application.NewOrderService(repo, mockPricing)

    orderID, err := svc.PlaceOrder(context.Background(), PlaceOrderCommand{
        CustomerID: "cust-123",
        Items:      testItems,
    })

    require.NoError(t, err)
    assert.NotEmpty(t, orderID)
    assert.Equal(t, 1, len(repo.orders)) // 確認呼叫了 Save
}

func TestOrderService_PlaceOrder_EmptyItems(t *testing.T) {
    repo := NewMockOrderRepository()
    svc := application.NewOrderService(repo, mockPricing)

    _, err := svc.PlaceOrder(context.Background(), PlaceOrderCommand{
        CustomerID: "cust-123",
        Items:      nil, // 空品項
    })

    assert.ErrorIs(t, err, ErrOrderMustHaveItems)
    assert.Empty(t, repo.orders) // 確認沒有呼叫 Save
}
```

---

## 二、testcontainers-go（Repository Integration Test）

用 Docker 啟動真實 PostgreSQL，讓 Repository 測試針對真實資料庫執行。

### 安裝

```bash
go get github.com/testcontainers/testcontainers-go
go get github.com/testcontainers/testcontainers-go/modules/postgres
go get github.com/golang-migrate/migrate/v4
```

### 共用 Container（TestMain）

```go
// repository/testmain_test.go

package repository_test

import (
    "context"
    "database/sql"
    "fmt"
    "os"
    "testing"

    "github.com/golang-migrate/migrate/v4"
    _ "github.com/golang-migrate/migrate/v4/database/postgres"
    _ "github.com/golang-migrate/migrate/v4/source/file"
    "github.com/testcontainers/testcontainers-go/modules/postgres"
    "github.com/testcontainers/testcontainers-go"
    _ "github.com/jackc/pgx/v5/stdlib"
)

var testDB *sql.DB

func TestMain(m *testing.M) {
    ctx := context.Background()

    // 啟動 PostgreSQL Container
    pgContainer, err := postgres.RunContainer(ctx,
        testcontainers.WithImage("postgres:16-alpine"),
        postgres.WithDatabase("testdb"),
        postgres.WithUsername("test"),
        postgres.WithPassword("test"),
        testcontainers.WithWaitStrategy(
            wait.ForLog("database system is ready to accept connections").
                WithOccurrence(2).
                WithStartupTimeout(30*time.Second),
        ),
    )
    if err != nil {
        fmt.Fprintf(os.Stderr, "failed to start postgres: %v\n", err)
        os.Exit(1)
    }
    defer pgContainer.Terminate(ctx)

    // 取得連線字串
    connStr, err := pgContainer.ConnectionString(ctx, "sslmode=disable")
    if err != nil {
        fmt.Fprintf(os.Stderr, "failed to get connection string: %v\n", err)
        os.Exit(1)
    }

    // 開啟 DB
    testDB, err = sql.Open("pgx", connStr)
    if err != nil {
        fmt.Fprintf(os.Stderr, "failed to open db: %v\n", err)
        os.Exit(1)
    }
    defer testDB.Close()

    // 執行 Migration（確保 Schema 正確）
    m := migrate.New(
        "file://../../migrations",  // migration 檔案路徑
        connStr,
    )
    if err := m.Up(); err != nil && err != migrate.ErrNoChange {
        fmt.Fprintf(os.Stderr, "failed to run migrations: %v\n", err)
        os.Exit(1)
    }

    // 執行所有測試
    os.Exit(m.Run())
}
```

### 事務隔離（每個測試獨立）

每個測試用 Transaction 包裹，測試結束 Rollback，確保測試間互不干擾：

```go
// repository/testing_helpers_test.go

// newTestTx 建立一個事務，測試結束後自動 Rollback
func newTestTx(t *testing.T) *sql.Tx {
    t.Helper()
    tx, err := testDB.BeginTx(context.Background(), nil)
    require.NoError(t, err)
    t.Cleanup(func() {
        // 無論測試成功或失敗，都 Rollback
        tx.Rollback() // 錯誤可以忽略（已 commit 或 rollback 都返回 error）
    })
    return tx
}

// newTestDB 返回一個 db.DB（可以是 *sql.Tx 或 *sql.DB）
type DBTX interface {
    ExecContext(context.Context, string, ...interface{}) (sql.Result, error)
    QueryContext(context.Context, string, ...interface{}) (*sql.Rows, error)
    QueryRowContext(context.Context, string, ...interface{}) *sql.Row
}
```

### Repository 測試

```go
// repository/order_repository_test.go

package repository_test

import (
    "context"
    "testing"
    "time"

    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/require"
)

func TestOrderRepository_Save_And_FindByID(t *testing.T) {
    tx := newTestTx(t) // 每個測試獨立事務，結束後自動 Rollback
    repo := NewPostgresOrderRepository(tx)
    ctx := context.Background()

    // 準備測試資料
    order := &Order{
        ID:         "order-001",
        CustomerID: "cust-123",
        Status:     OrderStatusPending,
        Items: []OrderItem{
            {ProductID: "prod-a", Quantity: 2, UnitPrice: 1000},
        },
        CreatedAt: time.Now().UTC().Truncate(time.Millisecond), // DB 精度
    }

    // 執行
    err := repo.Save(ctx, order)
    require.NoError(t, err)

    // 驗證
    found, err := repo.FindByID(ctx, "order-001")
    require.NoError(t, err)
    assert.Equal(t, order.ID, found.ID)
    assert.Equal(t, order.CustomerID, found.CustomerID)
    assert.Equal(t, order.Status, found.Status)
    assert.Len(t, found.Items, 1)
    assert.Equal(t, order.Items[0].ProductID, found.Items[0].ProductID)
}

func TestOrderRepository_FindByID_NotFound(t *testing.T) {
    tx := newTestTx(t)
    repo := NewPostgresOrderRepository(tx)

    _, err := repo.FindByID(context.Background(), "non-existent")
    assert.ErrorIs(t, err, ErrOrderNotFound)
}

func TestOrderRepository_FindByCustomerID(t *testing.T) {
    tx := newTestTx(t)
    repo := NewPostgresOrderRepository(tx)
    ctx := context.Background()

    // 插入多筆資料
    for i := 1; i <= 3; i++ {
        order := buildTestOrder(fmt.Sprintf("order-%03d", i), "cust-123")
        require.NoError(t, repo.Save(ctx, order))
    }
    // 其他客戶的訂單
    require.NoError(t, repo.Save(ctx, buildTestOrder("order-other", "cust-999")))

    // 查詢
    orders, err := repo.FindByCustomerID(ctx, "cust-123")
    require.NoError(t, err)
    assert.Len(t, orders, 3)
    for _, o := range orders {
        assert.Equal(t, "cust-123", o.CustomerID)
    }
}

// 測試並發寫入（樂觀鎖）
func TestOrderRepository_OptimisticLock(t *testing.T) {
    // 這個測試需要兩個獨立連線（事務隔離會影響並發測試）
    // 所以直接用 testDB 而非 tx
    ctx := context.Background()
    repo := NewPostgresOrderRepository(testDB)

    order := buildTestOrder("order-lock-test", "cust-123")
    require.NoError(t, repo.Save(ctx, order))
    t.Cleanup(func() {
        testDB.ExecContext(ctx, "DELETE FROM orders WHERE id = $1", order.ID)
    })

    // 兩個事務同時讀取同一筆記錄
    order1, err := repo.FindByID(ctx, order.ID)
    require.NoError(t, err)

    order2, err := repo.FindByID(ctx, order.ID)
    require.NoError(t, err)

    // 第一個更新成功
    require.NoError(t, repo.Update(ctx, order1))

    // 第二個應該因版本衝突失敗
    err = repo.Update(ctx, order2)
    assert.ErrorIs(t, err, ErrVersionConflict)
}

// 輔助函數
func buildTestOrder(id, customerID string) *Order {
    return &Order{
        ID:         id,
        CustomerID: customerID,
        Status:     OrderStatusPending,
        Items:      []OrderItem{{ProductID: "prod-1", Quantity: 1, UnitPrice: 100}},
        CreatedAt:  time.Now().UTC().Truncate(time.Millisecond),
    }
}
```

---

## 三、pgx + sqlc 的測試方式

如果使用 pgx 和 sqlc 代碼生成，測試方式略有不同：

```go
// pgx 版本的 TestMain
import (
    "github.com/jackc/pgx/v5/pgxpool"
)

var testPool *pgxpool.Pool

func TestMain(m *testing.M) {
    ctx := context.Background()

    pgContainer, _ := postgres.RunContainer(ctx, /* ... */)
    defer pgContainer.Terminate(ctx)

    connStr, _ := pgContainer.ConnectionString(ctx, "sslmode=disable")

    // pgxpool 更適合生產環境
    testPool, err = pgxpool.New(ctx, connStr)
    if err != nil { os.Exit(1) }
    defer testPool.Close()

    runMigrations(connStr)
    os.Exit(m.Run())
}

// 針對 pgx 的事務隔離輔助
func newTestPgxTx(t *testing.T) pgx.Tx {
    t.Helper()
    tx, err := testPool.Begin(context.Background())
    require.NoError(t, err)
    t.Cleanup(func() { tx.Rollback(context.Background()) })
    return tx
}

// sqlc 生成的 Queries 可以接受 pgx.Tx
func TestQueries_CreateOrder(t *testing.T) {
    tx := newTestPgxTx(t)
    q := db.New(tx) // sqlc 生成的 Queries，接受 DBTX interface

    order, err := q.CreateOrder(context.Background(), db.CreateOrderParams{
        ID:         "order-001",
        CustomerID: "cust-123",
        Status:     "pending",
    })

    require.NoError(t, err)
    assert.Equal(t, "order-001", order.ID)
    assert.Equal(t, "pending", order.Status)
}
```

---

## 四、Fixtures（測試資料管理）

### 方法 1：Go 代碼建立 Fixture

```go
// repository/fixtures_test.go

type Fixtures struct {
    db  DBTX
    ctx context.Context
}

func NewFixtures(t *testing.T, db DBTX) *Fixtures {
    t.Helper()
    return &Fixtures{db: db, ctx: context.Background()}
}

func (f *Fixtures) CreateCustomer(t *testing.T, opts ...func(*Customer)) *Customer {
    t.Helper()
    c := &Customer{
        ID:    uuid.New().String(),
        Name:  "Test Customer",
        Email: fmt.Sprintf("test-%s@example.com", uuid.New().String()[:8]),
    }
    for _, opt := range opts { opt(c) }

    _, err := f.db.ExecContext(f.ctx,
        "INSERT INTO customers (id, name, email) VALUES ($1, $2, $3)",
        c.ID, c.Name, c.Email,
    )
    require.NoError(t, err)
    return c
}

func (f *Fixtures) CreateOrder(t *testing.T, customerID string, opts ...func(*Order)) *Order {
    t.Helper()
    o := &Order{
        ID:         uuid.New().String(),
        CustomerID: customerID,
        Status:     "pending",
    }
    for _, opt := range opts { opt(o) }

    _, err := f.db.ExecContext(f.ctx,
        "INSERT INTO orders (id, customer_id, status) VALUES ($1, $2, $3)",
        o.ID, o.CustomerID, o.Status,
    )
    require.NoError(t, err)
    return o
}

// 使用 Fixtures
func TestOrderRepository_CancelOrder(t *testing.T) {
    tx := newTestTx(t)
    fix := NewFixtures(t, tx)
    repo := NewPostgresOrderRepository(tx)

    customer := fix.CreateCustomer(t)
    order := fix.CreateOrder(t, customer.ID)

    err := repo.Cancel(context.Background(), order.ID, "customer request")
    require.NoError(t, err)

    found, err := repo.FindByID(context.Background(), order.ID)
    require.NoError(t, err)
    assert.Equal(t, "cancelled", found.Status)
}
```

### 方法 2：SQL Fixtures 檔案

```sql
-- testdata/fixtures/orders.sql
INSERT INTO customers (id, name, email) VALUES
    ('cust-001', 'Alice', 'alice@example.com'),
    ('cust-002', 'Bob', 'bob@example.com');

INSERT INTO orders (id, customer_id, status, created_at) VALUES
    ('order-001', 'cust-001', 'pending', NOW()),
    ('order-002', 'cust-001', 'completed', NOW()),
    ('order-003', 'cust-002', 'pending', NOW());
```

```go
func loadFixtures(t *testing.T, tx *sql.Tx, path string) {
    t.Helper()
    data, err := os.ReadFile(path)
    require.NoError(t, err)
    _, err = tx.ExecContext(context.Background(), string(data))
    require.NoError(t, err)
}

func TestOrderRepository_WithFixtures(t *testing.T) {
    tx := newTestTx(t)
    loadFixtures(t, tx, "testdata/fixtures/orders.sql")
    repo := NewPostgresOrderRepository(tx)

    orders, err := repo.FindByCustomerID(context.Background(), "cust-001")
    require.NoError(t, err)
    assert.Len(t, orders, 2)
}
```

---

## 五、Migration 測試

確保每次 migration 都可以正確 up 和 down：

```go
// migrations/migration_test.go

func TestMigrations_UpDown(t *testing.T) {
    // 每個 migration 測試用獨立的 DB（避免干擾）
    ctx := context.Background()
    pgContainer, _ := postgres.RunContainer(ctx, /* ... */)
    defer pgContainer.Terminate(ctx)
    connStr, _ := pgContainer.ConnectionString(ctx, "sslmode=disable")

    m, err := migrate.New("file://.", connStr)
    require.NoError(t, err)

    // Up
    err = m.Up()
    require.NoError(t, err, "migration up should succeed")

    // 驗證 table 存在
    db, _ := sql.Open("pgx", connStr)
    var tableName string
    err = db.QueryRowContext(ctx,
        "SELECT table_name FROM information_schema.tables WHERE table_name = 'orders'",
    ).Scan(&tableName)
    require.NoError(t, err)
    assert.Equal(t, "orders", tableName)

    // Down（回滾所有）
    err = m.Down()
    require.NoError(t, err, "migration down should succeed")

    // 驗證 table 不存在
    err = db.QueryRowContext(ctx,
        "SELECT table_name FROM information_schema.tables WHERE table_name = 'orders'",
    ).Scan(&tableName)
    assert.ErrorIs(t, err, sql.ErrNoRows)
}

// 測試特定版本的 migration
func TestMigration_V3_AddIndexOnCustomerID(t *testing.T) {
    // 先跑到 V2，再跑 V3，驗證 index 建立正確
    // ...
}
```

---

## 六、完整 CI 設定

### 執行測試分類

```bash
# 只跑 Unit Test（不需要 DB，快）
go test -short ./...

# 跑 Integration Test（需要 Docker）
go test -run Integration ./...

# 或用 build tag 分類
go test -tags integration ./...
```

### 用 Build Tag 區分

```go
// repository/order_repository_integration_test.go
//go:build integration

package repository_test

// 只有 go test -tags integration 時才編譯這個檔案
func TestOrderRepository_Integration(t *testing.T) {
    // ... 真實 DB 測試
}
```

```go
// 在 test 函數頂部用 testing.Short() 跳過
func TestOrderRepository_Save(t *testing.T) {
    if testing.Short() {
        t.Skip("skipping integration test in short mode")
    }
    // ...
}
```

### GitHub Actions CI

```yaml
# .github/workflows/test.yml
jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16-alpine
        env:
          POSTGRES_DB: testdb
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with: { go-version: '1.24' }

      - name: Run migrations
        run: |
          go install -tags 'postgres' github.com/golang-migrate/migrate/v4/cmd/migrate@latest
          migrate -path ./migrations -database "postgres://test:test@localhost:5432/testdb?sslmode=disable" up

      - name: Run tests
        env:
          TEST_DATABASE_URL: postgres://test:test@localhost:5432/testdb?sslmode=disable
        run: go test -race -timeout 120s ./...
```

**或用 testcontainers（不需要 services）**：

```yaml
      - name: Run tests (testcontainers)
        run: go test -race -timeout 120s ./...
        # testcontainers 自動啟動 Docker，CI 上 Docker 通常已安裝
```

---

## 七、測試速度優化

```go
// 技巧 1：TestMain 只啟動一次 Container，所有測試共用
// （不要每個測試函數都啟動新 Container）

// 技巧 2：t.Parallel() 讓 Repository 測試並行執行
func TestOrderRepository_FindByID(t *testing.T) {
    t.Parallel() // ✅ 配合事務隔離可以安全並行
    tx := newTestTx(t)
    // ...
}

// 技巧 3：在本機用 Docker Compose 保持 DB 常駐，省去每次啟動時間
// docker-compose.yml 起一個 test DB
// TEST_DATABASE_URL=... go test ./...（指定外部 DB，跳過 testcontainers）

func getTestDB(t *testing.T) *sql.DB {
    if url := os.Getenv("TEST_DATABASE_URL"); url != "" {
        db, err := sql.Open("pgx", url)
        require.NoError(t, err)
        return db
    }
    // 沒有環境變數時用 testcontainers
    return startTestContainer(t)
}

// 技巧 4：goleak 確保測試沒有 goroutine 洩漏
func TestMain(m *testing.M) {
    // ...
    goleak.VerifyTestMain(m)
}
```

---

## 常見陷阱

```go
// ❌ 陷阱 1：time.Time 精度問題
// Go 的 time.Time 有奈秒精度，PostgreSQL 的 timestamptz 只有微秒
order.CreatedAt = time.Now()
// ... save and retrieve ...
assert.Equal(t, order.CreatedAt, found.CreatedAt) // 可能失敗！

// ✅ 修法：存入前截斷到微秒
order.CreatedAt = time.Now().UTC().Truncate(time.Microsecond)

// ❌ 陷阱 2：測試間共用狀態（忘記用事務隔離）
// 一個測試插入的資料影響其他測試的 COUNT 或 FindAll 結果

// ❌ 陷阱 3：並發測試用 Rollback 事務時，其他連線看不到未 commit 的資料
// 如果要測試並發場景，需要用獨立的連線（commit 後手動清理）

// ❌ 陷阱 4：測試 migration 時忘記 down
// 每次 up 前確認 DB 是乾淨的，或用全新的 Container

// ❌ 陷阱 5：在 CI 上 Docker in Docker 權限問題
// testcontainers 需要存取 /var/run/docker.sock
// GitHub Actions 的 runner 通常有 Docker，但某些環境需要特殊設定
```

---

## 相關頁面

- [[依賴注入與控制反轉]] — Repository 介面設計讓 Mock 成為可能
- [[DDD領域驅動設計]] — Repository Pattern 的設計原則
- [[Go測試基準與模糊測試]] — 測試工具（testify、goleak）的深度使用
- [[Go記憶體洩漏排查]] — goleak 在測試中偵測 goroutine 洩漏
