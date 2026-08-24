---
title: OpenTelemetry 分散式追蹤
type: concept
tags: [opentelemetry, observability, tracing, metrics, logging, golang, microservices]
created: 2026-04-29
updated: 2026-04-29
---

# OpenTelemetry 分散式追蹤

## 可觀測性三大支柱

| 支柱 | 描述 | 工具 |
|------|------|------|
| **Traces** | 一個請求穿越多個服務的完整路徑 | Jaeger, Tempo, Zipkin |
| **Metrics** | 時間序列數值（QPS、延遲、錯誤率）| Prometheus + Grafana |
| **Logs** | 結構化日誌事件 | Loki, Elasticsearch |

**OpenTelemetry（OTel）** 是統一的 SDK，產生 traces + metrics + logs，發送到任何後端。

## 核心概念

```
Trace（追蹤）：一個完整請求的生命週期
  └─ Span（跨度）：一個操作單元（如 HTTP call, DB query）
      ├─ Attributes：鍵值對標籤
      ├─ Events：時間點事件
      └─ Links：關聯到其他 Trace
```

```
Trace 視覺化：
[HTTP Request: POST /orders]                    400ms
  ├─ [Auth middleware]                           10ms
  ├─ [DB: SELECT user]                           20ms
  ├─ [gRPC: inventory.Reserve]                  150ms
  │    └─ [DB: UPDATE inventory]               140ms
  └─ [DB: INSERT order]                          30ms
```

## Go 初始化 OTel

```go
package telemetry

import (
    "context"

    "go.opentelemetry.io/otel"
    "go.opentelemetry.io/otel/exporters/otlp/otlptrace/otlptracegrpc"
    "go.opentelemetry.io/otel/sdk/resource"
    sdktrace "go.opentelemetry.io/otel/sdk/trace"
    semconv "go.opentelemetry.io/otel/semconv/v1.24.0"
)

func InitTracer(ctx context.Context, serviceName, endpoint string) (func(), error) {
    // 1. 設定 Exporter（送到 Jaeger / Tempo）
    exporter, err := otlptracegrpc.New(ctx,
        otlptracegrpc.WithEndpoint(endpoint), // "jaeger:4317"
        otlptracegrpc.WithInsecure(),
    )
    if err != nil {
        return nil, err
    }

    // 2. 設定 Resource（服務元資訊）
    res, _ := resource.New(ctx,
        resource.WithAttributes(
            semconv.ServiceName(serviceName),
            semconv.ServiceVersion("1.0.0"),
            semconv.DeploymentEnvironment("production"),
        ),
    )

    // 3. 建立 TracerProvider
    tp := sdktrace.NewTracerProvider(
        sdktrace.WithBatcher(exporter),
        sdktrace.WithResource(res),
        sdktrace.WithSampler(sdktrace.TraceIDRatioBased(0.1)), // 採樣 10%（生產環境）
    )
    otel.SetTracerProvider(tp)

    // 返回 cleanup 函數
    return func() {
        tp.Shutdown(context.Background())
    }, nil
}
```

## 在 HTTP Handler 加 Tracing

```go
import (
    "go.opentelemetry.io/otel"
    "go.opentelemetry.io/otel/attribute"
    "go.opentelemetry.io/otel/codes"
    "go.opentelemetry.io/contrib/instrumentation/net/http/otelhttp"
)

var tracer = otel.Tracer("order-service")

// Middleware：自動為所有 HTTP 請求建立 Root Span
func main() {
    mux := http.NewServeMux()
    mux.HandleFunc("/orders", createOrder)
    // otelhttp.NewHandler 自動建立 span，傳播 trace context
    http.ListenAndServe(":8080", otelhttp.NewHandler(mux, "order-service"))
}

func createOrder(w http.ResponseWriter, r *http.Request) {
    // 從 HTTP Header 提取 trace context（跨服務傳播）
    ctx := r.Context()

    // 建立 child span
    ctx, span := tracer.Start(ctx, "createOrder")
    defer span.End()

    // 加入業務標籤
    span.SetAttributes(
        attribute.String("user.id", r.Header.Get("X-User-ID")),
        attribute.String("order.source", "web"),
    )

    // 記錄事件
    span.AddEvent("validation_started")

    orderID, err := orderService.Create(ctx, parseRequest(r))
    if err != nil {
        // 標記 span 為錯誤
        span.RecordError(err)
        span.SetStatus(codes.Error, err.Error())
        http.Error(w, err.Error(), http.StatusBadRequest)
        return
    }

    span.SetAttributes(attribute.String("order.id", orderID))
    span.AddEvent("order_created")
    span.SetStatus(codes.Ok, "")

    json.NewEncoder(w).Encode(map[string]string{"order_id": orderID})
}
```

## 在資料庫查詢加 Tracing

```go
import "go.opentelemetry.io/otel/semconv/v1.24.0/trace" // DB semconv

func (r *OrderRepository) FindByID(ctx context.Context, orderID string) (*Order, error) {
    ctx, span := tracer.Start(ctx, "db.FindOrderByID",
        trace.WithSpanKind(trace.SpanKindClient),
    )
    defer span.End()

    // 遵循 DB span 命名規範
    span.SetAttributes(
        semconv.DBSystem("postgresql"),
        semconv.DBName("orders_db"),
        semconv.DBStatement("SELECT * FROM orders WHERE id = $1"),
        semconv.DBOperation("SELECT"),
    )

    var order Order
    err := r.db.GetContext(ctx, &order, "SELECT * FROM orders WHERE id = $1", orderID)
    if err != nil {
        span.RecordError(err)
        span.SetStatus(codes.Error, err.Error())
        return nil, err
    }
    return &order, nil
}
```

## gRPC 中的 Trace 傳播

```go
import "go.opentelemetry.io/contrib/instrumentation/google.golang.org/grpc/otelgrpc"

// Server 端：接收 trace context
grpc.NewServer(
    grpc.StatsHandler(otelgrpc.NewServerHandler()),
)

// Client 端：注入 trace context 到 outgoing header
conn, _ := grpc.NewClient("inventory-service:50051",
    grpc.WithStatsHandler(otelgrpc.NewClientHandler()),
)
```

## Metrics（指標）

```go
import (
    "go.opentelemetry.io/otel/metric"
    "go.opentelemetry.io/otel/exporters/prometheus"
    sdkmetric "go.opentelemetry.io/otel/sdk/metric"
)

var (
    requestCount    metric.Int64Counter
    requestDuration metric.Float64Histogram
    activeOrders    metric.Int64UpDownCounter
)

func initMetrics() {
    // Prometheus exporter（讓 Prometheus 來 scrape）
    exporter, _ := prometheus.New()
    provider := sdkmetric.NewMeterProvider(sdkmetric.WithReader(exporter))
    meter := provider.Meter("order-service")

    requestCount, _ = meter.Int64Counter("http.requests.total",
        metric.WithDescription("Total HTTP requests"),
    )
    requestDuration, _ = meter.Float64Histogram("http.request.duration.seconds",
        metric.WithDescription("HTTP request duration"),
    )
    activeOrders, _ = meter.Int64UpDownCounter("orders.active",
        metric.WithDescription("Number of active orders"),
    )
}

// 在 handler 中使用
func (h *Handler) CreateOrder(ctx context.Context, req *CreateOrderRequest) (*CreateOrderResponse, error) {
    start := time.Now()

    // 計數
    requestCount.Add(ctx, 1, metric.WithAttributes(
        attribute.String("method", "CreateOrder"),
        attribute.String("service", "order"),
    ))

    resp, err := h.service.Create(ctx, req)

    // 延遲直方圖
    requestDuration.Record(ctx, time.Since(start).Seconds(), metric.WithAttributes(
        attribute.String("method", "CreateOrder"),
        attribute.Bool("success", err == nil),
    ))

    if err == nil {
        activeOrders.Add(ctx, 1)
    }

    return resp, err
}
```

## 結構化日誌（zap + OTel 整合）

```go
import (
    "go.uber.org/zap"
    "go.opentelemetry.io/otel/trace"
)

func newLogger() *zap.Logger {
    logger, _ := zap.NewProduction()
    return logger
}

// 從 context 提取 trace ID，加入日誌（關聯 logs 和 traces）
func logWithTrace(ctx context.Context, logger *zap.Logger, msg string, fields ...zap.Field) {
    span := trace.SpanFromContext(ctx)
    sc := span.SpanContext()

    if sc.IsValid() {
        fields = append(fields,
            zap.String("trace_id", sc.TraceID().String()),
            zap.String("span_id", sc.SpanID().String()),
        )
    }
    logger.Info(msg, fields...)
}
```

## 採樣策略

| 策略 | 適用場景 |
|------|---------|
| `AlwaysSample` | 開發 / 測試環境 |
| `TraceIDRatioBased(0.1)` | 生產環境正常流量（10% 採樣）|
| `ParentBased` | 跟隨父 span 的採樣決定（最常用）|
| Tail-based sampling | 只保留慢請求和錯誤（需要 OTel Collector）|

```go
// 生產環境推薦：Parent-based + 10% 採樣
sdktrace.WithSampler(
    sdktrace.ParentBased(sdktrace.TraceIDRatioBased(0.1)),
)
```

## 相關頁面

- [[微服務架構設計原則]] — 可觀測性是微服務的必要條件
- [[gRPC設計與實戰]] — gRPC trace 傳播
- [[Go優雅關機與健康檢查]] — 關機前 flush traces
- [[Go效能調優]] — trace 輔助 pprof 找效能瓶頸
