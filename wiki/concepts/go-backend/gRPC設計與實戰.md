---
title: gRPC 設計與實戰
type: concept
tags: [grpc, protobuf, golang, microservices, rpc]
created: 2026-04-29
updated: 2026-04-29
---

# gRPC 設計與實戰

## 為什麼用 gRPC

| 面向 | REST/JSON | gRPC/Protobuf |
|------|-----------|---------------|
| 序列化 | JSON（文字，大） | Protobuf（二進位，小 3–10x） |
| 效能 | 較慢 | 快（HTTP/2 多路複用） |
| 型別安全 | 無（靠文件） | 有（由 proto 生成代碼） |
| Streaming | 靠 SSE / WebSocket | 原生支援四種 streaming |
| 適合場景 | 外部 API、瀏覽器 | 內部服務間通訊 |

## Protobuf 定義

```protobuf
syntax = "proto3";
package order.v1;
option go_package = "github.com/yourco/order/gen/order/v1;orderv1";

// 服務定義
service OrderService {
  // Unary RPC（最常用）
  rpc CreateOrder(CreateOrderRequest) returns (CreateOrderResponse);
  rpc GetOrder(GetOrderRequest) returns (Order);

  // Server Streaming：一個請求，多個回應（如訂單狀態推送）
  rpc WatchOrderStatus(WatchOrderStatusRequest) returns (stream OrderStatusEvent);

  // Client Streaming：多個請求，一個回應（如批次上傳）
  rpc BatchCreateOrders(stream CreateOrderRequest) returns (BatchCreateOrderResponse);

  // Bidirectional Streaming（如即時聊天）
  rpc Chat(stream ChatMessage) returns (stream ChatMessage);
}

message CreateOrderRequest {
  string user_id = 1;
  repeated OrderItem items = 2;
  Address shipping_address = 3;
}

message OrderItem {
  string product_id = 1;
  int32 quantity = 2;
  int64 unit_price_cents = 3; // 用最小單位避免浮點
}

message CreateOrderResponse {
  string order_id = 1;
  OrderStatus status = 2;
  int64 total_cents = 3;
}

enum OrderStatus {
  ORDER_STATUS_UNSPECIFIED = 0; // proto3 必須有 0 值
  ORDER_STATUS_PENDING = 1;
  ORDER_STATUS_PAID = 2;
  ORDER_STATUS_SHIPPED = 3;
  ORDER_STATUS_COMPLETED = 4;
  ORDER_STATUS_CANCELLED = 5;
}
```

**生成 Go 代碼**：
```bash
protoc --go_out=. --go_opt=paths=source_relative \
       --go-grpc_out=. --go-grpc_opt=paths=source_relative \
       proto/order/v1/order.proto
```

## Server 實作

```go
package main

import (
    "context"
    "log"
    "net"

    "google.golang.org/grpc"
    "google.golang.org/grpc/codes"
    "google.golang.org/grpc/status"

    orderv1 "github.com/yourco/order/gen/order/v1"
)

type OrderServer struct {
    orderv1.UnimplementedOrderServiceServer // 必須嵌入，向前相容
    repo OrderRepository
}

func (s *OrderServer) CreateOrder(ctx context.Context, req *orderv1.CreateOrderRequest) (*orderv1.CreateOrderResponse, error) {
    // 1. 驗證輸入
    if req.UserId == "" {
        return nil, status.Error(codes.InvalidArgument, "user_id is required")
    }
    if len(req.Items) == 0 {
        return nil, status.Error(codes.InvalidArgument, "items cannot be empty")
    }

    // 2. 業務邏輯
    order, err := s.repo.Create(ctx, req)
    if err != nil {
        // 根據錯誤類型回傳對應的 gRPC status code
        switch {
        case isNotFound(err):
            return nil, status.Errorf(codes.NotFound, "product not found: %v", err)
        case isConflict(err):
            return nil, status.Errorf(codes.AlreadyExists, "duplicate order: %v", err)
        default:
            return nil, status.Errorf(codes.Internal, "internal error: %v", err)
        }
    }

    return &orderv1.CreateOrderResponse{
        OrderId:    order.ID,
        Status:     orderv1.OrderStatus_ORDER_STATUS_PENDING,
        TotalCents: order.TotalCents,
    }, nil
}

// Server Streaming 範例
func (s *OrderServer) WatchOrderStatus(req *orderv1.WatchOrderStatusRequest, stream orderv1.OrderService_WatchOrderStatusServer) error {
    for {
        select {
        case <-stream.Context().Done():
            return nil // 客戶端斷線，正常結束
        case event := <-s.repo.OrderEvents(req.OrderId):
            if err := stream.Send(&orderv1.OrderStatusEvent{
                OrderId: req.OrderId,
                Status:  event.Status,
            }); err != nil {
                return err
            }
        }
    }
}

func main() {
    lis, err := net.Listen("tcp", ":50051")
    if err != nil {
        log.Fatal(err)
    }

    s := grpc.NewServer(
        grpc.ChainUnaryInterceptor(
            loggingInterceptor,
            recoveryInterceptor,
            authInterceptor,
        ),
    )
    orderv1.RegisterOrderServiceServer(s, &OrderServer{})

    log.Println("gRPC server listening on :50051")
    if err := s.Serve(lis); err != nil {
        log.Fatal(err)
    }
}
```

## Client 實作（含重試與 Context）

```go
package client

import (
    "context"
    "time"

    "google.golang.org/grpc"
    "google.golang.org/grpc/credentials/insecure"
    "google.golang.org/grpc/keepalive"

    orderv1 "github.com/yourco/order/gen/order/v1"
)

func NewOrderClient(target string) (orderv1.OrderServiceClient, *grpc.ClientConn, error) {
    conn, err := grpc.NewClient(target,
        grpc.WithTransportCredentials(insecure.NewCredentials()), // 生產用 TLS
        grpc.WithKeepaliveParams(keepalive.ClientParameters{
            Time:    30 * time.Second,
            Timeout: 10 * time.Second,
        }),
        grpc.WithDefaultServiceConfig(`{
            "methodConfig": [{
                "name": [{"service": "order.v1.OrderService"}],
                "retryPolicy": {
                    "maxAttempts": 3,
                    "initialBackoff": "0.1s",
                    "maxBackoff": "1s",
                    "backoffMultiplier": 2,
                    "retryableStatusCodes": ["UNAVAILABLE", "DEADLINE_EXCEEDED"]
                }
            }]
        }`),
    )
    if err != nil {
        return nil, nil, err
    }
    return orderv1.NewOrderServiceClient(conn), conn, nil
}

// 使用範例
func CreateOrderExample(client orderv1.OrderServiceClient) {
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
    defer cancel()

    resp, err := client.CreateOrder(ctx, &orderv1.CreateOrderRequest{
        UserId: "user-123",
        Items: []*orderv1.OrderItem{
            {ProductId: "prod-456", Quantity: 2, UnitPriceCents: 2999},
        },
    })
    if err != nil {
        // 解析 gRPC 錯誤
        st, _ := status.FromError(err)
        switch st.Code() {
        case codes.NotFound:
            // 商品不存在
        case codes.DeadlineExceeded:
            // 超時，可以重試
        }
        return
    }
    _ = resp
}
```

## Interceptor（中間件）

```go
// Logging Interceptor
func loggingInterceptor(ctx context.Context, req interface{}, info *grpc.UnaryServerInfo, handler grpc.UnaryHandler) (interface{}, error) {
    start := time.Now()
    resp, err := handler(ctx, req)
    log.Printf("method=%s duration=%s err=%v", info.FullMethod, time.Since(start), err)
    return resp, err
}

// Panic Recovery Interceptor
func recoveryInterceptor(ctx context.Context, req interface{}, info *grpc.UnaryServerInfo, handler grpc.UnaryHandler) (resp interface{}, err error) {
    defer func() {
        if r := recover(); r != nil {
            err = status.Errorf(codes.Internal, "panic: %v", r)
        }
    }()
    return handler(ctx, req)
}

// Auth Interceptor（JWT 驗證）
func authInterceptor(ctx context.Context, req interface{}, info *grpc.UnaryServerInfo, handler grpc.UnaryHandler) (interface{}, error) {
    // 跳過不需要認證的方法
    if info.FullMethod == "/order.v1.OrderService/HealthCheck" {
        return handler(ctx, req)
    }

    md, ok := metadata.FromIncomingContext(ctx)
    if !ok {
        return nil, status.Error(codes.Unauthenticated, "missing metadata")
    }

    tokens := md.Get("authorization")
    if len(tokens) == 0 {
        return nil, status.Error(codes.Unauthenticated, "missing token")
    }

    userID, err := validateJWT(tokens[0])
    if err != nil {
        return nil, status.Error(codes.Unauthenticated, "invalid token")
    }

    ctx = context.WithValue(ctx, userIDKey, userID)
    return handler(ctx, req)
}
```

## gRPC 錯誤碼對應

| gRPC Code | HTTP 對應 | 使用場景 |
|-----------|-----------|---------|
| OK | 200 | 成功 |
| InvalidArgument | 400 | 輸入驗證失敗 |
| NotFound | 404 | 資源不存在 |
| AlreadyExists | 409 | 重複資源 |
| PermissionDenied | 403 | 無權限 |
| Unauthenticated | 401 | 未認證 |
| ResourceExhausted | 429 | 超過限制 |
| Internal | 500 | 內部錯誤 |
| Unavailable | 503 | 服務暫時不可用（可重試）|
| DeadlineExceeded | 504 | 超時（可重試）|

## 健康檢查（K8s 整合）

```go
import "google.golang.org/grpc/health/grpc_health_v1"

type HealthServer struct{}

func (h *HealthServer) Check(ctx context.Context, req *grpc_health_v1.HealthCheckRequest) (*grpc_health_v1.HealthCheckResponse, error) {
    return &grpc_health_v1.HealthCheckResponse{
        Status: grpc_health_v1.HealthCheckResponse_SERVING,
    }, nil
}

// 在 K8s 用 grpc_health_probe 做 liveness/readiness check
```

## 相關頁面

- [[微服務架構設計原則]] — 何時用 gRPC vs REST
- [[OpenTelemetry分散式追蹤]] — gRPC 呼叫鏈追蹤
- [[Go優雅關機與健康檢查]] — gRPC server 的優雅關機
- [[Circuit Breaker熔斷器]] — gRPC client 端的熔斷保護
