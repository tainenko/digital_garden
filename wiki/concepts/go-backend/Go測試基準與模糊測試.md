---
title: Go 測試、基準測試與模糊測試
type: concept
tags: [golang, testing, benchmark, fuzz, testify, coverage, senior]
created: 2026-04-30
updated: 2026-04-30
---

# Go 測試、基準測試與模糊測試

## 表格驅動測試（Table-Driven Tests）

Go 社群最推薦的測試模式：

```go
func TestParseAmount(t *testing.T) {
    tests := []struct {
        name    string
        input   string
        want    int64
        wantErr bool
    }{
        {name: "正整數", input: "100", want: 100, wantErr: false},
        {name: "帶小數", input: "99.99", want: 9999, wantErr: false},
        {name: "空字串", input: "", want: 0, wantErr: true},
        {name: "負數", input: "-50", want: 0, wantErr: true},
        {name: "超大數", input: "99999999999999", want: 0, wantErr: true},
    }

    for _, tc := range tests {
        t.Run(tc.name, func(t *testing.T) {
            got, err := ParseAmount(tc.input)
            if (err != nil) != tc.wantErr {
                t.Errorf("ParseAmount(%q) error = %v, wantErr = %v", tc.input, err, tc.wantErr)
                return
            }
            if !tc.wantErr && got != tc.want {
                t.Errorf("ParseAmount(%q) = %d, want %d", tc.input, got, tc.want)
            }
        })
    }
}
```

**優點**：
- 加新案例只需在 table 裡加一行
- `t.Run` 讓每個 case 有獨立名稱，失敗時容易定位
- 可用 `-run TestParseAmount/正整數` 只跑單一 case

---

## 子測試（Subtests）與 t.Parallel

```go
func TestOrderWorkflow(t *testing.T) {
    // 共用的 setup
    db := setupTestDB(t)
    defer db.Close()

    t.Run("建立訂單", func(t *testing.T) {
        t.Parallel() // 這個 subtest 可以和其他 Parallel 的 subtest 並行
        order, err := CreateOrder(db, testOrder)
        // ...
    })

    t.Run("付款成功", func(t *testing.T) {
        t.Parallel()
        // ...
    })

    t.Run("庫存不足時拒絕", func(t *testing.T) {
        t.Parallel()
        // ...
    })
}
```

### t.Cleanup：自動清理

```go
func setupTestDB(t *testing.T) *sql.DB {
    t.Helper() // 讓錯誤報告指向呼叫者，而非這個函數

    db, err := sql.Open("pgx", testDSN)
    if err != nil {
        t.Fatalf("failed to open DB: %v", err)
    }

    t.Cleanup(func() {
        db.Close() // 測試結束後自動呼叫，無論成功還是失敗
    })

    return db
}
```

---

## testify：斷言與 Mock

```go
import (
    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/require"
    "github.com/stretchr/testify/mock"
)

// assert vs require：
// assert.Equal → 失敗但繼續執行
// require.Equal → 失敗立即停止（t.FailNow）
func TestUser(t *testing.T) {
    user, err := GetUser("123")
    require.NoError(t, err)           // err != nil → 立即停止
    assert.Equal(t, "Alice", user.Name)
    assert.NotEmpty(t, user.ID)
    assert.WithinDuration(t, time.Now(), user.CreatedAt, time.Second)
}
```

### Mock 依賴

```go
// 定義介面（讓 mock 可以替換真實實作）
type UserRepository interface {
    FindByID(ctx context.Context, id string) (*User, error)
    Save(ctx context.Context, user *User) error
}

// testify/mock 自動生成（或手寫）
type MockUserRepo struct {
    mock.Mock
}

func (m *MockUserRepo) FindByID(ctx context.Context, id string) (*User, error) {
    args := m.Called(ctx, id)
    if args.Get(0) == nil {
        return nil, args.Error(1)
    }
    return args.Get(0).(*User), args.Error(1)
}

// 測試中使用
func TestOrderService_CreateOrder(t *testing.T) {
    mockRepo := new(MockUserRepo)
    mockRepo.On("FindByID", mock.Anything, "user-123").
        Return(&User{ID: "user-123", Name: "Alice"}, nil)

    svc := NewOrderService(mockRepo)
    _, err := svc.CreateOrder(context.Background(), "user-123", testItems)
    require.NoError(t, err)

    mockRepo.AssertExpectations(t) // 驗證所有 On() 都被呼叫過
}
```

**建議工具**：`mockery` 自動從介面生成 mock 程式碼：
```bash
mockery --name=UserRepository --output=./mocks
```

---

## 基準測試（Benchmark）

```go
func BenchmarkJSONMarshal(b *testing.B) {
    order := generateTestOrder() // setup（不計時）

    b.ResetTimer() // 重設計時器，排除 setup 時間
    b.ReportAllocs() // 報告記憶體分配次數和大小

    for i := 0; i < b.N; i++ { // b.N 由框架決定，直到結果穩定
        _, err := json.Marshal(order)
        if err != nil {
            b.Fatal(err)
        }
    }
}

// 執行
// go test -bench=BenchmarkJSONMarshal -benchmem -count=5 ./...
// -benchmem：顯示記憶體分配（等同 b.ReportAllocs）
// -count=5：重複 5 次（減少雜訊）
// -benchtime=10s：跑滿 10 秒（或 -benchtime=1000x 跑 1000 次）
```

```
輸出解讀：
BenchmarkJSONMarshal-8   1000000   1023 ns/op   256 B/op   3 allocs/op
                    │         │         │           │           │
              CPU 核數    執行次數  每次耗時     每次分配B    每次分配次數
```

### 比較多個實作

```go
func BenchmarkMarshal(b *testing.B) {
    order := generateTestOrder()

    b.Run("encoding/json", func(b *testing.B) {
        b.ReportAllocs()
        for i := 0; i < b.N; i++ {
            json.Marshal(order)
        }
    })

    b.Run("sonic", func(b *testing.B) {
        b.ReportAllocs()
        for i := 0; i < b.N; i++ {
            sonic.Marshal(order)
        }
    })

    b.Run("jsoniter", func(b *testing.B) {
        b.ReportAllocs()
        for i := 0; i < b.N; i++ {
            jsoniter.Marshal(order)
        }
    })
}

// 用 benchstat 比較結果（可安裝 golang.org/x/perf/cmd/benchstat）
// go test -bench=BenchmarkMarshal -count=10 > result.txt
// benchstat result.txt
```

### 基準測試 + pprof

```go
// 同時生成 CPU 和記憶體 profile
go test -bench=BenchmarkJSONMarshal -cpuprofile=cpu.prof -memprofile=mem.prof ./...

// 分析
go tool pprof cpu.prof
go tool pprof -alloc_space mem.prof
```

### 避免編譯器優化消除基準

```go
var result []byte // package-level 變數，防止結果被優化掉

func BenchmarkMarshal(b *testing.B) {
    var r []byte
    for i := 0; i < b.N; i++ {
        r, _ = json.Marshal(testOrder)
    }
    result = r // 賦值給 package 變數，防止編譯器認為結果未使用而優化掉整個呼叫
}
```

---

## 模糊測試（Fuzz Testing，Go 1.18+）

Fuzz testing 讓 Go 自動生成隨機輸入，尋找會讓程式崩潰的邊界案例。

```go
// 基本結構
func FuzzParseAmount(f *testing.F) {
    // Seed corpus：已知的有效輸入（讓 fuzzer 從這些開始變異）
    f.Add("100")
    f.Add("0")
    f.Add("99.99")
    f.Add("-1")
    f.Add("")

    f.Fuzz(func(t *testing.T, input string) {
        // Fuzzer 會自動生成大量類似 input 的字串來測試
        result, err := ParseAmount(input)
        if err != nil {
            return // 錯誤是允許的，只要不 panic
        }
        // 不變式（Invariant）：成功解析的結果必須滿足的條件
        if result < 0 {
            t.Errorf("ParseAmount(%q) = %d, should not be negative", input, result)
        }
    })
}
```

```bash
# 一般跑法（只用 seed corpus，類似普通測試）
go test -run=FuzzParseAmount ./...

# 開始真正的 fuzzing（持續跑直到找到問題或 Ctrl+C）
go test -fuzz=FuzzParseAmount ./...

# 限制時間
go test -fuzz=FuzzParseAmount -fuzztime=60s ./...
```

**Corpus 目錄**：
```
testdata/
└── fuzz/
    └── FuzzParseAmount/
        ├── seed/         ← f.Add() 的內容
        └── <hash>        ← fuzzer 發現的問題案例（自動保存）
```

發現問題後，fuzzer 會把觸發錯誤的輸入存到 `testdata/fuzz/FuzzParseAmount/` 目錄。這些輸入之後的 `go test -run` 也會跑到（永久性的 regression test）。

### 多參數 Fuzz

```go
func FuzzHTTPHeader(f *testing.F) {
    f.Add("Content-Type", "application/json")
    f.Add("Authorization", "Bearer token123")

    f.Fuzz(func(t *testing.T, key, value string) {
        // 測試 header 解析不會 panic
        result := ParseHeader(key, value)
        _ = result
    })
}
```

---

## 測試覆蓋率（Coverage）

```bash
# 生成覆蓋率報告
go test -coverprofile=coverage.out ./...

# 在終端顯示覆蓋率摘要
go tool cover -func=coverage.out

# 在瀏覽器以 HTML 顯示（每行是否被覆蓋）
go tool cover -html=coverage.out

# 顯示整體覆蓋率
go test -cover ./...

# 只看特定套件的覆蓋率
go test -coverprofile=coverage.out -coverpkg=./internal/... ./...
```

```
覆蓋率輸出範例：
github.com/myapp/order.go:   CreateOrder   85.7%
github.com/myapp/order.go:   CancelOrder   100.0%
github.com/myapp/order.go:   RefundOrder   0.0%    ← 未測試！
total:                                      72.3%
```

**注意**：覆蓋率高不等於測試品質好。重要的是覆蓋**邊界條件**，而非追求 100%。

---

## 測試輔助模式

### Golden File Testing

```go
// 測試複雜輸出（例如 HTML 渲染、JSON API response）
func TestRenderInvoice(t *testing.T) {
    result := RenderInvoice(testOrder)

    golden := "testdata/invoice.golden"
    if *update { // go test -update 旗標
        os.WriteFile(golden, []byte(result), 0644)
        return
    }

    expected, _ := os.ReadFile(golden)
    assert.Equal(t, string(expected), result)
}

var update = flag.Bool("update", false, "update golden files")
```

### 測試輔助建構器

```go
// 避免在每個測試中重複建立複雜物件
type OrderBuilder struct {
    order Order
}

func NewOrderBuilder() *OrderBuilder {
    return &OrderBuilder{
        order: Order{
            ID:     "test-order-001",
            Status: StatusPending,
            Items:  []Item{{ProductID: "p1", Quantity: 1, Price: 100}},
        },
    }
}

func (b *OrderBuilder) WithStatus(s OrderStatus) *OrderBuilder {
    b.order.Status = s
    return b
}

func (b *OrderBuilder) WithItems(items []Item) *OrderBuilder {
    b.order.Items = items
    return b
}

func (b *OrderBuilder) Build() Order { return b.order }

// 使用
order := NewOrderBuilder().WithStatus(StatusPaid).Build()
```

---

## go test 常用旗標速查

```bash
go test ./...                        # 跑所有測試
go test -v ./...                     # 詳細輸出
go test -run TestParseAmount ./...   # 只跑名稱匹配的測試
go test -run TestOrder/建立          # 只跑特定 subtest
go test -race ./...                  # 開啟 race detector（必備！）
go test -count=1 ./...              # 禁用 test cache（強制重跑）
go test -timeout=30s ./...          # 設定超時（預設 10m）
go test -short ./...                # 跳過標記為 short 的測試
go test -parallel=4 ./...           # 最多 4 個並行測試套件

# 快速確認測試是否通過（不顯示輸出）
go test ./... 2>&1 | grep -E "^(ok|FAIL)"
```

---

## 相關頁面

- [[Go pprof實戰完整指南]] — Benchmark + pprof 的結合使用
- [[Go記憶體洩漏排查]] — goleak 在測試中偵測 goroutine 洩漏
- [[Go效能調優]] — 從 Benchmark 結果找優化方向
- [[Go錯誤處理最佳實踐]] — 測試錯誤路徑的策略
