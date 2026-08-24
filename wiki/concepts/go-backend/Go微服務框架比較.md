---
title: Go 微服務框架比較
type: concept
tags: [golang, microservices, frameworks, go-kit, kratos, go-zero, grpc]
created: 2026-04-29
updated: 2026-04-29
---

# Go 微服務框架比較

## 框架全景

| 框架 | 作者 | Stars | 定位 | 適合規模 |
|------|------|-------|------|---------|
| **go-kit** | Peter Bourgon | ~27k | 工具包（非框架），自由組合 | 大型團隊，自定義需求強 |
| **Kratos** | Bilibili | ~23k | 微服務框架，含 protobuf 工具鏈 | 中大型，bilibili 生產驗證 |
| **go-zero** | tal-tech | ~28k | 一站式框架，含代碼生成 | 快速開發，中小型 |
| **go-micro** | Asim Aslam | ~22k | 插件化微服務框架 | 需要插件化的場景 |
| **標準庫 + 手動組合** | — | — | 純 net/http + 自選組件 | 小型服務，控制欲強 |

## go-kit

**哲學**：不是框架，是一組工具包。你自己決定怎麼組合。

```go
// go-kit 的 Endpoint 模式
import (
    "github.com/go-kit/kit/endpoint"
    "github.com/go-kit/kit/transport/http"
)

// Service 介面（純業務邏輯）
type OrderService interface {
    CreateOrder(ctx context.Context, req CreateOrderRequest) (*Order, error)
}

// Endpoint：將 Service 方法包裝成 go-kit endpoint
func makeCreateOrderEndpoint(svc OrderService) endpoint.Endpoint {
    return func(ctx context.Context, request interface{}) (interface{}, error) {
        req := request.(CreateOrderRequest)
        order, err := svc.CreateOrder(ctx, req)
        if err != nil {
            return nil, err
        }
        return order, nil
    }
}

// Middleware：logging、metrics、rate limiting...
func loggingMiddleware(logger log.Logger) endpoint.Middleware {
    return func(next endpoint.Endpoint) endpoint.Endpoint {
        return func(ctx context.Context, request interface{}) (response interface{}, err error) {
            defer func(begin time.Time) {
                logger.Log("method", "CreateOrder", "took", time.Since(begin), "err", err)
            }(time.Now())
            return next(ctx, request)
        }
    }
}

// Transport：決定協議（HTTP / gRPC）
func NewHTTPHandler(endpoints Endpoints) http.Handler {
    return http.NewServer(
        endpoints.CreateOrder,
        decodeCreateOrderRequest,
        encodeResponse,
    )
}
```

**優點**：極度靈活，各層清晰分離
**缺點**：boilerplate 多，學習曲線陡峭，小型專案 overkill

---

## Kratos（Bilibili 出品）

**哲學**：意見式框架，整合 wire、protobuf、kratos CLI。

```bash
# 安裝
go install github.com/go-kratos/kratos/cmd/kratos/v2@latest

# 建立新服務
kratos new order-service
```

```go
// 用 protobuf 定義 API
// api/order/v1/order.proto

// Kratos 自動生成 HTTP + gRPC server 代碼
// 你只需要實作業務邏輯

// internal/service/order.go
type OrderService struct {
    pb.UnimplementedOrderServer
    uc *biz.OrderUseCase
}

func (s *OrderService) CreateOrder(ctx context.Context, req *pb.CreateOrderRequest) (*pb.CreateOrderResponse, error) {
    order, err := s.uc.CreateOrder(ctx, &biz.CreateOrderReq{
        UserID: req.UserId,
        Items:  convertItems(req.Items),
    })
    if err != nil {
        return nil, err
    }
    return &pb.CreateOrderResponse{OrderId: order.ID}, nil
}

// Kratos 的 Wire 依賴注入
// 在 cmd/order/wire.go 用 google/wire 自動組裝依賴
```

**Kratos 目錄結構**（約定式）：
```
order-service/
├── api/          ← Proto 定義
├── cmd/          ← 入口（wire_gen.go 自動生成）
├── internal/
│   ├── biz/      ← 業務邏輯（Use Case）
│   ├── data/     ← 資料存取（Repository 實作）
│   ├── service/  ← gRPC/HTTP Handler
│   └── conf/     ← 設定結構
└── third_party/  ← Proto 依賴
```

**優點**：生態完整（wire + proto + 設定），bilibili 生產驗證，社群活躍
**缺點**：意見強烈，需要完整採用一套工具鏈

---

## go-zero

**哲學**：goctl 代碼生成，從 API 定義直接生成 CRUD 代碼。

```bash
# 定義 API
cat > order.api << 'EOF'
type CreateOrderReq {
    UserID string `json:"user_id"`
    Items  []Item `json:"items"`
}
type CreateOrderResp {
    OrderID string `json:"order_id"`
}
service order-api {
    @handler CreateOrder
    post /orders (CreateOrderReq) returns (CreateOrderResp)
}
EOF

# 生成代碼
goctl api go -api order.api -dir . -style go_zero
```

生成後的結構：
```
order-api/
├── etc/order-api.yaml   ← 設定
├── internal/
│   ├── config/
│   ├── handler/         ← 生成的 HTTP Handler
│   ├── logic/           ← 你填業務邏輯的地方
│   ├── middleware/
│   ├── svc/             ← ServiceContext（依賴注入）
│   └── types/
└── order.go             ← main
```

```go
// logic/createorderlogic.go（只需填業務邏輯）
func (l *CreateOrderLogic) CreateOrder(req *types.CreateOrderReq) (resp *types.CreateOrderResp, err error) {
    // 只需要寫這裡！handler、路由、middleware 都生成好了
    order, err := l.svcCtx.OrderModel.Insert(l.ctx, &model.Orders{
        UserId: req.UserID,
    })
    if err != nil {
        return nil, err
    }
    return &types.CreateOrderResp{OrderID: fmt.Sprintf("%d", order.LastInsertId())}, nil
}
```

**優點**：開發速度極快，學習曲線低，整合 Redis/MySQL/JWT 開箱即用
**缺點**：生成代碼難以 customize，強依賴 goctl 工具，靈活性差

---

## 標準庫 + 手動組合（推薦給 < 5 人團隊）

```go
// 選擇：net/http + chi + zerolog + pgx + go-redis + sonic
import (
    "github.com/go-chi/chi/v5"
    "github.com/go-chi/chi/v5/middleware"
    "github.com/rs/zerolog/log"
    "github.com/jackc/pgx/v5/pgxpool"
)

func main() {
    r := chi.NewRouter()
    r.Use(middleware.RequestID)
    r.Use(middleware.RealIP)
    r.Use(middleware.Logger)
    r.Use(middleware.Recoverer)
    r.Use(middleware.Timeout(60 * time.Second))

    r.Post("/orders", createOrderHandler)
    r.Get("/orders/{id}", getOrderHandler)

    http.ListenAndServe(":8080", r)
}
```

---

## Web 框架比較

| 框架 | 路由 | 效能 | 生態 | 推薦場景 |
|------|------|------|------|---------|
| **Gin** | httprouter | 極快 | 最豐富 | REST API，最常見 |
| **Echo** | 自研 | 快 | 豐富 | 類似 Gin，API 更優雅 |
| **chi** | 標準庫相容 | 快 | 中等 | 喜歡原生 Handler 的場景 |
| **Fiber** | fasthttp | 最快 | 中等 | 極度效能敏感 |
| **net/http** | — | 快 | — | 微型服務，低依賴 |

**Gin 基礎用法**：

```go
import "github.com/gin-gonic/gin"

func main() {
    r := gin.New()
    r.Use(gin.Recovery())
    r.Use(gin.Logger())

    // 路由群組
    v1 := r.Group("/api/v1")
    v1.Use(authMiddleware())
    {
        v1.POST("/orders", createOrder)
        v1.GET("/orders/:id", getOrder)
        v1.PATCH("/orders/:id/status", updateOrderStatus)
    }

    r.Run(":8080")
}

func createOrder(c *gin.Context) {
    var req CreateOrderRequest
    if err := c.ShouldBindJSON(&req); err != nil {
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        return
    }

    // 從 middleware 注入的 userID
    userID := c.GetString("user_id")

    order, err := orderService.Create(c.Request.Context(), userID, req)
    if err != nil {
        c.JSON(http.StatusInternalServerError, gin.H{"error": err.Error()})
        return
    }

    c.JSON(http.StatusCreated, order)
}
```

---

## 選型決策樹

```
需要極快開發速度（個人/小型創業）？
  → go-zero（goctl 生成）或 Gin + 標準庫

已有 protobuf 定義的大型微服務架構？
  → Kratos（完整工具鏈）

需要高度自定義，工程師素質強？
  → go-kit（靈活）或 標準庫手動組合

只是個簡單的 REST API 服務？
  → Gin / Echo + pgx + go-redis，不需要微服務框架
```

## 推薦組合（2026）

**中小型新專案**：
- HTTP: `gin` 或 `chi`
- DB: `pgx/v5` + `sqlc`（SQL → Go 代碼生成）
- Config: `viper`
- Logger: `zerolog`
- DI: `uber-go/fx` 或手動
- gRPC: `google.golang.org/grpc`
- Observability: `go.opentelemetry.io/otel`

**大型微服務**：
- 以上 + `Kratos`（框架） 或 `go-kit`（工具包）
- Service Mesh: `Istio`（K8s 環境）

## 相關頁面

- [[微服務架構設計原則]] — 何時需要微服務框架
- [[gRPC設計與實戰]] — 所有框架都可以整合 gRPC
- [[Go效能調優]] — 框架選型也影響效能
- [[Go依賴注入與Wire]] — Kratos 和 go-kit 都重度使用 DI
