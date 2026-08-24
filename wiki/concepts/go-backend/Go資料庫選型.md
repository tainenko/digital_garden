---
title: Go 資料庫選型
type: concept
tags: [go, database, gorm, sqlc, pgx, postgresql, orm]
created: 2026-04-30
updated: 2026-04-30
sources: []
---

# Go 資料庫選型

## 三大方案比較

| | GORM | sqlc | pgx（直接） |
|---|------|------|------------|
| 類型 | Full ORM | SQL→Go 代碼生成 | 原生驅動 |
| 學習曲線 | 低 | 中 | 高 |
| 效能 | 中（反射）| 高（生成代碼）| 最高 |
| 型別安全 | 弱（interface{}）| 強（編譯期）| 強 |
| 複雜查詢 | 麻煩 | 原生 SQL | 原生 SQL |
| Migration | 自動（Auto-migrate）| 需配合工具 | 需配合工具 |
| 適用場景 | CRUD 多、快速開發 | 生產 API、效能敏感 | 批次處理、最大控制 |

**推薦**：生產 API 用 **sqlc + pgx**；原型或 CRUD 密集用 **GORM**。

---

## GORM

### 基本設定

```go
// go get gorm.io/gorm gorm.io/driver/postgres
import (
    "gorm.io/gorm"
    "gorm.io/driver/postgres"
)

type User struct {
    gorm.Model          // 內嵌 ID, CreatedAt, UpdatedAt, DeletedAt
    Email    string     `gorm:"uniqueIndex;not null"`
    Name     string
    Orders   []Order    `gorm:"foreignKey:UserID"`
}

type Order struct {
    gorm.Model
    UserID uint
    Amount float64
    Status string `gorm:"default:pending"`
}

func NewDB(dsn string) (*gorm.DB, error) {
    db, err := gorm.Open(postgres.Open(dsn), &gorm.Config{
        Logger: logger.Default.LogMode(logger.Info), // 開發時印 SQL
    })
    if err != nil {
        return nil, err
    }
    sqlDB, _ := db.DB()
    sqlDB.SetMaxOpenConns(25)
    sqlDB.SetMaxIdleConns(10)
    sqlDB.SetConnMaxLifetime(5 * time.Minute)
    return db, nil
}
```

### CRUD

```go
// Create
user := User{Email: "tony@example.com", Name: "Tony"}
result := db.Create(&user)  // user.ID 自動填入

// Read
var user User
db.First(&user, 1)                          // 找 id=1
db.First(&user, "email = ?", "tony@example.com")
db.Where("name LIKE ?", "%Tony%").Find(&users)

// Preload（避免 N+1）
db.Preload("Orders").Find(&users)
db.Preload("Orders", "status = ?", "completed").First(&user, id)

// Update
db.Model(&user).Update("Name", "New Name")
db.Model(&user).Updates(User{Name: "New", Email: "new@example.com"})  // 只更新非零值
db.Model(&user).Updates(map[string]any{"name": "New", "email": "new@example.com"})

// Delete（軟刪除，gorm.Model 自動）
db.Delete(&user, 1)
// 硬刪除
db.Unscoped().Delete(&user, 1)
```

### GORM 常見陷阱

```go
// ❌ 陷阱 1：Updates 只更新非零值
user.Name = ""
db.Save(&user)  // Name 不會更新（零值被忽略）
// ✅ 用 map
db.Model(&user).Updates(map[string]any{"name": ""})

// ❌ 陷阱 2：忘記處理 Error
db.Find(&users)  // 沒有 .Error 檢查
// ✅
result := db.Find(&users)
if result.Error != nil {
    return result.Error
}

// ❌ 陷阱 3：N+1
for _, order := range orders {
    db.First(&user, order.UserID)  // N+1！
}
// ✅
db.Preload("User").Find(&orders)
```

---

## sqlc

### 工作流程

```
1. 寫 .sql 查詢檔案
2. 跑 sqlc generate
3. 使用生成的型別安全 Go 代碼
```

### 設定

```yaml
# sqlc.yaml
version: "2"
sql:
  - engine: "postgresql"
    queries: "queries/"    # .sql 查詢檔案目錄
    schema: "schema.sql"   # DB schema
    gen:
      go:
        package: "db"
        out: "internal/db"
        sql_package: "pgx/v5"   # 使用 pgx v5
        emit_json_tags: true
        emit_db_tags: true
```

### 查詢定義

```sql
-- queries/users.sql

-- name: GetUser :one
SELECT id, email, name, created_at
FROM users
WHERE id = $1;

-- name: ListUsers :many
SELECT id, email, name, created_at
FROM users
ORDER BY created_at DESC
LIMIT $1 OFFSET $2;

-- name: CreateUser :one
INSERT INTO users (email, name)
VALUES ($1, $2)
RETURNING *;

-- name: UpdateUserName :one
UPDATE users
SET name = $2, updated_at = NOW()
WHERE id = $1
RETURNING *;

-- name: DeleteUser :exec
DELETE FROM users WHERE id = $1;
```

### 生成後使用

```go
// sqlc generate 後生成以下（無需手動寫）：
// internal/db/users.sql.go

// 使用
import "myapp/internal/db"

func GetUserHandler(w http.ResponseWriter, r *http.Request) {
    queries := db.New(dbPool)  // dbPool 是 *pgxpool.Pool

    userID, _ := strconv.ParseInt(chi.URLParam(r, "id"), 10, 64)
    user, err := queries.GetUser(r.Context(), userID)
    if err == pgx.ErrNoRows {
        http.Error(w, "not found", 404)
        return
    }
    if err != nil {
        http.Error(w, "internal error", 500)
        return
    }
    json.NewEncoder(w).Encode(user)
}
```

### sqlc + 事務

```go
// 事務封裝（推薦模式）
func (s *UserService) CreateUserWithOrder(ctx context.Context, email string, amount float64) error {
    return withTx(ctx, s.pool, func(tx pgx.Tx) error {
        q := db.New(tx)  // sqlc 接受 pgx.Tx

        user, err := q.CreateUser(ctx, db.CreateUserParams{Email: email, Name: "new"})
        if err != nil {
            return err
        }

        _, err = q.CreateOrder(ctx, db.CreateOrderParams{
            UserID: user.ID,
            Amount: pgtype.Numeric{...},
        })
        return err
    })
}

func withTx(ctx context.Context, pool *pgxpool.Pool, fn func(pgx.Tx) error) error {
    tx, err := pool.Begin(ctx)
    if err != nil {
        return err
    }
    defer tx.Rollback(ctx)
    if err := fn(tx); err != nil {
        return err
    }
    return tx.Commit(ctx)
}
```

---

## pgx 直接使用

適合批次處理、COPY 操作、自訂 codec：

```go
// go get github.com/jackc/pgx/v5/pgxpool
import "github.com/jackc/pgx/v5/pgxpool"

func NewPool(ctx context.Context, connString string) (*pgxpool.Pool, error) {
    config, err := pgxpool.ParseConfig(connString)
    if err != nil {
        return nil, err
    }
    config.MaxConns = 25
    config.MinConns = 5
    config.MaxConnLifetime = 5 * time.Minute
    config.MaxConnIdleTime = 1 * time.Minute
    return pgxpool.NewWithConfig(ctx, config)
}

// COPY 批次插入（極高效能）
func BulkInsertOrders(ctx context.Context, pool *pgxpool.Pool, orders []Order) error {
    conn, err := pool.Acquire(ctx)
    defer conn.Release()

    _, err = conn.Conn().CopyFrom(
        ctx,
        pgx.Identifier{"orders"},
        []string{"user_id", "amount", "status"},
        pgx.CopyFromSlice(len(orders), func(i int) ([]any, error) {
            return []any{orders[i].UserID, orders[i].Amount, "pending"}, nil
        }),
    )
    return err
}
```

---

## 選型決策樹

```
需要快速原型 / CRUD 多 / 有 Admin 後台？
└─ Yes → GORM

需要高效能 / 複雜 SQL / 型別安全？
└─ Yes → sqlc + pgx
          ├─ 查詢由 SQL 定義，生成 Go 代碼
          └─ 批次/COPY 操作用 pgx 直接寫

已有舊系統用 GORM？
└─ 混用：新功能用 sqlc，舊功能留 GORM
```

---

## 常見面試問題

**Q: GORM 的 AutoMigrate 可以用在生產環境嗎？**
A: 不推薦。AutoMigrate 只會加欄位，不會刪欄位、改型別，無法精確控制 migration 行為。生產環境應用 golang-migrate 或 goose，有版本管理和 rollback 能力。

**Q: sqlc 的限制是什麼？**
A: 不支援動態查詢（如可選的 WHERE 條件）。動態查詢需要用 squirrel 等 query builder，或直接用 pgx 拼 SQL（搭配參數化防注入）。

**Q: pgxpool 和 database/sql 的差異？**
A: pgxpool 是 pgx 原生連接池，針對 PostgreSQL 最佳化，支援 prepared statement 快取、Listen/Notify、COPY 等 pg 特有功能；`database/sql` 是通用介面，可換底層驅動但功能受限。

**Q: N+1 如何在 sqlc 中解決？**
A: 用 JOIN 查詢或兩次查詢（取 IDs → IN 查詢），手動在 Service 層組合。sqlc 不像 GORM 有 Preload，需要自己設計 Repository 的批次查詢方法。

---

## 相關頁面

- [[Go PostgreSQL測試策略]] — testcontainers 整合測試
- [[DDD領域驅動設計]] — Repository pattern 設計
- [[Go依賴注入與Wire]] — DB Pool 的依賴注入
- [[Python資料庫整合]] — SQLAlchemy 對照
