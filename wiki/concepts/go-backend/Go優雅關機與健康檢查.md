---
title: Go 優雅關機與健康檢查
type: concept
tags: [graceful-shutdown, health-check, kubernetes, golang, lifecycle]
created: 2026-04-29
updated: 2026-04-29
---

# Go 優雅關機與健康檢查

## 為什麼優雅關機很重要

K8s 滾動更新（rolling update）流程：
1. K8s 啟動新版本 Pod
2. 向舊版本 Pod 發送 `SIGTERM`
3. 等待 `terminationGracePeriodSeconds`（預設 30 秒）
4. 若 Pod 還沒退出，發 `SIGKILL` 強制殺死

如果你的 Go 服務不處理 `SIGTERM`：
- 正在處理中的請求被強行中斷 → 客戶端收到 connection reset
- 資料庫連線沒有正常關閉 → 連線洩漏
- 正在寫入的資料可能不完整 → 資料損壞

## 標準優雅關機模式

```go
package main

import (
    "context"
    "errors"
    "log"
    "net/http"
    "os"
    "os/signal"
    "sync"
    "syscall"
    "time"
)

func main() {
    // 1. 建立 HTTP Server
    mux := http.NewServeMux()
    mux.HandleFunc("/orders", handleOrders)
    mux.HandleFunc("/health", handleHealth)
    mux.HandleFunc("/ready", handleReady)

    srv := &http.Server{
        Addr:         ":8080",
        Handler:      mux,
        ReadTimeout:  30 * time.Second,
        WriteTimeout: 30 * time.Second,
        IdleTimeout:  120 * time.Second,
    }

    // 2. 啟動 server（非阻塞）
    go func() {
        if err := srv.ListenAndServe(); !errors.Is(err, http.ErrServerClosed) {
            log.Fatalf("server error: %v", err)
        }
    }()
    log.Println("server started on :8080")

    // 3. 等待關機信號
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
    sig := <-quit
    log.Printf("received signal: %v, shutting down...", sig)

    // 4. 開始優雅關機
    // 設定關機超時（必須 < K8s terminationGracePeriodSeconds）
    ctx, cancel := context.WithTimeout(context.Background(), 25*time.Second)
    defer cancel()

    // 5. 停止接受新請求，等待進行中的請求完成
    if err := srv.Shutdown(ctx); err != nil {
        log.Printf("server shutdown error: %v", err)
    }

    // 6. 關閉其他資源（資料庫、Redis、Kafka...）
    cleanup()

    log.Println("server exited gracefully")
}

var (
    db    *sql.DB
    redis *redis.Client
    kafka *kafka.Producer
)

func cleanup() {
    var wg sync.WaitGroup

    wg.Add(1)
    go func() {
        defer wg.Done()
        if err := db.Close(); err != nil {
            log.Printf("db close error: %v", err)
        }
        log.Println("database connection closed")
    }()

    wg.Add(1)
    go func() {
        defer wg.Done()
        if err := redis.Close(); err != nil {
            log.Printf("redis close error: %v", err)
        }
        log.Println("redis connection closed")
    }()

    wg.Add(1)
    go func() {
        defer wg.Done()
        kafka.Flush(5000) // 等最多 5 秒，確保訊息投遞
        kafka.Close()
        log.Println("kafka producer flushed and closed")
    }()

    wg.Wait()
}
```

## gRPC Server 的優雅關機

```go
grpcServer := grpc.NewServer(...)

go func() {
    if err := grpcServer.Serve(lis); err != nil {
        log.Fatalf("grpc server error: %v", err)
    }
}()

// 接收信號
<-quit

// GracefulStop：等待正在進行的 RPC 完成，不接受新的
// 注意：若 streaming RPC 不結束，GracefulStop 會等到 context 超時
done := make(chan struct{})
go func() {
    grpcServer.GracefulStop()
    close(done)
}()

select {
case <-done:
    log.Println("grpc server stopped gracefully")
case <-time.After(20 * time.Second):
    grpcServer.Stop() // 強制停止
    log.Println("grpc server force stopped")
}
```

## Kubernetes 健康檢查三種 Probe

```yaml
# deployment.yaml
spec:
  containers:
    - name: order-service
      livenessProbe:    # 存活探針：失敗則重啟 Pod
        httpGet:
          path: /health/live
          port: 8080
        initialDelaySeconds: 10
        periodSeconds: 10
        failureThreshold: 3

      readinessProbe:   # 就緒探針：失敗則從 LB 移除（不重啟）
        httpGet:
          path: /health/ready
          port: 8080
        initialDelaySeconds: 5
        periodSeconds: 5
        failureThreshold: 2

      startupProbe:     # 啟動探針：給慢啟動的服務更多時間
        httpGet:
          path: /health/live
          port: 8080
        failureThreshold: 30   # 最多等 30 * 10s = 300s
        periodSeconds: 10
```

## 健康檢查 Handler 實作

```go
package health

import (
    "context"
    "database/sql"
    "encoding/json"
    "net/http"
    "sync/atomic"
    "time"
)

type Checker struct {
    db      *sql.DB
    redis   RedisClient
    ready   atomic.Bool  // 是否已就緒（初始化完成）
    healthy atomic.Bool  // 是否健康
}

func NewChecker(db *sql.DB, redis RedisClient) *Checker {
    c := &Checker{db: db, redis: redis}
    c.healthy.Store(true)
    return c
}

// LivenessHandler：只要 process 活著就 200
// 不要在這裡查 DB（DB 掛了不代表 Pod 要重啟）
func (c *Checker) LivenessHandler(w http.ResponseWriter, r *http.Request) {
    if !c.healthy.Load() {
        http.Error(w, "unhealthy", http.StatusServiceUnavailable)
        return
    }
    w.WriteHeader(http.StatusOK)
    w.Write([]byte("ok"))
}

// ReadinessHandler：依賴的資源都準備好才 200
// 失敗時 K8s 把這個 Pod 從 LB 移除，但不重啟
func (c *Checker) ReadinessHandler(w http.ResponseWriter, r *http.Request) {
    if !c.ready.Load() {
        http.Error(w, "not ready", http.StatusServiceUnavailable)
        return
    }

    report := map[string]string{}
    allOK := true

    // 檢查 DB
    ctx, cancel := context.WithTimeout(r.Context(), 2*time.Second)
    defer cancel()
    if err := c.db.PingContext(ctx); err != nil {
        report["database"] = "unhealthy: " + err.Error()
        allOK = false
    } else {
        report["database"] = "healthy"
    }

    // 檢查 Redis
    if err := c.redis.Ping(r.Context()).Err(); err != nil {
        report["redis"] = "unhealthy: " + err.Error()
        allOK = false
    } else {
        report["redis"] = "healthy"
    }

    w.Header().Set("Content-Type", "application/json")
    if !allOK {
        w.WriteHeader(http.StatusServiceUnavailable)
    }
    json.NewEncoder(w).Encode(report)
}

// 在啟動完成後設定 ready
func (c *Checker) SetReady() {
    c.ready.Store(true)
}

// 開始關機時設定 not ready，讓 K8s 停止把流量發進來
func (c *Checker) SetNotReady() {
    c.ready.Store(false)
}
```

## 完整的啟動流程（含健康檢查整合）

```go
func main() {
    // 1. 初始化依賴（DB、Redis...）
    db := initDB()
    redis := initRedis()

    checker := health.NewChecker(db, redis)

    // 2. 啟動 server，此時 readiness = false
    mux := http.NewServeMux()
    mux.HandleFunc("/health/live", checker.LivenessHandler)
    mux.HandleFunc("/health/ready", checker.ReadinessHandler)
    mux.Handle("/", otelhttp.NewHandler(appRouter(), "order-service"))

    srv := &http.Server{Addr: ":8080", Handler: mux}
    go srv.ListenAndServe()

    // 3. 完成初始化後才設定 ready（Startup Probe 開始通過）
    checker.SetReady()
    log.Println("service is ready")

    // 4. 等待關機信號
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGTERM, syscall.SIGINT)
    <-quit

    // 5. 立即設定 not ready，讓 K8s 停止發新流量（同時等 LB 更新）
    checker.SetNotReady()
    time.Sleep(5 * time.Second) // 等 K8s readiness check 週期

    // 6. 等待進行中的請求完成
    ctx, cancel := context.WithTimeout(context.Background(), 20*time.Second)
    defer cancel()
    srv.Shutdown(ctx)

    // 7. 關閉所有依賴
    db.Close()
    redis.Close()
    log.Println("shutdown complete")
}
```

## 背景 Worker 的優雅關機

```go
type Worker struct {
    wg sync.WaitGroup
}

func (w *Worker) Start(ctx context.Context) {
    w.wg.Add(1)
    go func() {
        defer w.wg.Done()
        for {
            select {
            case <-ctx.Done():
                // Context 取消，結束 worker
                return
            default:
                w.process()
            }
        }
    }()
}

// 等所有 worker 完成
func (w *Worker) Stop() {
    w.wg.Wait()
}

// 在 main 中：
ctx, cancel := context.WithCancel(context.Background())
worker := &Worker{}
worker.Start(ctx)

<-quit
cancel() // 通知 worker 停止
worker.Stop() // 等 worker 完成
```

## 相關頁面

- [[微服務架構設計原則]] — 12-Factor App 第 9 點（Disposability）
- [[服務發現與負載均衡]] — 健康檢查與服務注銷的整合
- [[OpenTelemetry分散式追蹤]] — 關機前 flush traces
- [[Go並發模式]] — context 取消樹、WaitGroup 用法
