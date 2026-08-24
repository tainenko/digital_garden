---
title: Golang 微服務完整指南
type: topic
tags: [golang, microservices, architecture, grpc, kafka, kubernetes]
created: 2026-04-29
updated: 2026-04-29
---

# Golang 微服務完整指南

> 從架構設計到生產部署，涵蓋 Go 微服務開發所需的所有核心知識。

---

## 學習路徑

```
基礎（必學）
  ├─ 微服務架構設計原則 → 何時拆、沿什麼邊界拆
  ├─ gRPC 設計與實戰 → 服務間通訊協議
  └─ Go 錯誤處理最佳實踐 → 分層錯誤、wrapping

可靠性（中級）
  ├─ Circuit Breaker 熔斷器 → 防止故障級聯
  ├─ 冪等性設計 → 安全重試
  ├─ 分散式鎖 → 跨進程互斥
  └─ 服務發現與負載均衡 → 動態路由

進階模式（高級）
  ├─ Saga Pattern 分散式事務 → 跨服務事務
  ├─ CQRS 與 Event Sourcing → 讀寫分離
  └─ 事件驅動架構與 Kafka → 非同步解耦

工程實踐（所有層級）
  ├─ OpenTelemetry 分散式追蹤 → 可觀測性
  ├─ Go 優雅關機與健康檢查 → K8s 生命週期
  ├─ Go 微服務配置管理 → 環境/Secret 管理
  ├─ Go 依賴注入與 Wire → 依賴組裝
  ├─ Go 介面設計模式 → 抽象與測試
  └─ Go 微服務框架比較 → Kratos / go-kit / go-zero
```

---

## 架構決策速查

### 服務邊界怎麼切？

沿 **Domain-Driven Design 的 Bounded Context**：
- ✅ 商品服務 / 訂單服務 / 用戶服務（按業務能力）
- ❌ Controller 服務 / Service 服務 / DAO 服務（按技術分層）

詳見：[[微服務架構設計原則]]

### 服務間通訊用什麼？

| 場景 | 選擇 |
|------|------|
| 需要即時回應的內部呼叫 | gRPC（[[gRPC設計與實戰]]）|
| 外部 API / 瀏覽器 | REST（gin / echo）|
| 非同步、解耦、可重試 | Kafka（[[事件驅動架構與Kafka]]）|

### 跨服務事務怎麼做？

沒有全域事務，用 **Saga Pattern**（[[Saga Pattern分散式事務]]）：
- 簡單流程（2–3 步）→ Choreography（事件驅動）
- 複雜流程（4+ 步，有條件）→ Orchestration（中央協調）
- 配合 Outbox Pattern 保證訊息投遞

### 讀寫壓力不對稱怎麼處理？

**CQRS**（[[CQRS與Event Sourcing]]）：
- Write Model → 正規化 DB，保證一致性
- Read Model → 非正規化（Redis / Elasticsearch），快速查詢

### 如何防止服務 A 拖垮服務 B？

**Circuit Breaker 熔斷器**（[[Circuit Breaker熔斷器]]）：
- 失敗率超過閾值 → Open（快速失敗）
- 等待 timeout → Half-Open（探測）
- 探測成功 → Closed（恢復）

---

## 推薦技術棧（2026）

### 核心依賴

```go
// go.mod 核心依賴
require (
    // HTTP
    github.com/gin-gonic/gin v1.10.0
    // gRPC
    google.golang.org/grpc v1.64.0
    google.golang.org/protobuf v1.34.0
    // DB
    github.com/jackc/pgx/v5 v5.6.0
    github.com/sqlc-dev/sqlc v1.26.0   // SQL → Go 代碼生成
    // Redis
    github.com/redis/go-redis/v9 v9.5.0
    // Kafka
    github.com/confluentinc/confluent-kafka-go/v2 v2.4.0
    // Observability
    go.opentelemetry.io/otel v1.27.0
    go.uber.org/zap v1.27.0
    // Config
    github.com/spf13/viper v1.19.0
    // DI
    github.com/google/wire v0.6.0
    // 熔斷器
    github.com/sony/gobreaker v1.0.0
    // 分散式鎖
    github.com/go-redsync/redsync/v4 v4.12.0
    // 測試
    github.com/stretchr/testify v1.9.0
    github.com/stretchr/mock v0.4.0     // testify/mock
)
```

### 目錄結構（Clean Architecture）

```
order-service/
├── cmd/
│   └── order/
│       ├── main.go           ← 入口
│       ├── wire.go           ← Wire 注入定義
│       └── wire_gen.go       ← Wire 自動生成
├── api/
│   └── order/v1/
│       ├── order.proto       ← gRPC 定義
│       └── order.pb.go       ← 自動生成
├── internal/
│   ├── config/               ← 設定結構
│   ├── domain/               ← 業務實體、介面定義
│   │   ├── order.go          ← Order struct
│   │   └── repository.go     ← Repository 介面
│   ├── service/              ← 業務邏輯（Use Case）
│   │   └── order_service.go
│   ├── repository/           ← Repository 實作（DB）
│   │   └── postgres_order.go
│   ├── handler/              ← HTTP / gRPC Handler
│   │   ├── http/
│   │   └── grpc/
│   └── infrastructure/       ← 基礎設施（DB、Redis 連線）
│       ├── database.go
│       └── redis.go
├── pkg/                      ← 可跨服務共用的套件
│   ├── middleware/
│   └── telemetry/
└── deploy/
    ├── k8s/                  ← K8s manifest
    └── docker-compose.yaml   ← 本地開發
```

---

## 生產部署清單

### Kubernetes 配置要點

```yaml
# 每個 Deployment 必備
spec:
  replicas: 3  # 至少 3 個（HA）
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0  # 零停機部署

  template:
    spec:
      containers:
        - name: order-service
          # 資源限制（防止一個服務佔用所有資源）
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "500m"
              memory: "512Mi"

          # 三種 Probe（見 [[Go優雅關機與健康檢查]]）
          livenessProbe:
            httpGet: { path: /health/live, port: 8080 }
          readinessProbe:
            httpGet: { path: /health/ready, port: 8080 }

          # 環境變數和 Secret
          envFrom:
            - configMapRef: { name: order-service-config }
            - secretRef: { name: order-service-secret }

      # 優雅關機時間
      terminationGracePeriodSeconds: 30
```

### 可觀測性清單

```
Tracing（[[OpenTelemetry分散式追蹤]]）：
□ 所有 HTTP Handler 有 span
□ 所有 DB 操作有 span（含 SQL）
□ 所有 gRPC 呼叫有 span 傳播
□ 生產環境採樣率 10%（Parent-based）

Metrics：
□ HTTP request count / duration / error_rate
□ DB connection pool 使用率
□ Redis 命中率
□ Kafka consumer lag

Logging（結構化）：
□ JSON 格式輸出到 stdout
□ 每條 log 包含 trace_id（關聯 trace）
□ 不 log 敏感資訊（密碼、token、信用卡號）
□ 生產環境只輸出 INFO 以上
```

### 健康檢查清單

```
□ /health/live：只要 process 活著就 200
□ /health/ready：DB + Redis 都 ping 成功才 200
□ 關機流程：SIGTERM → setNotReady → 等 5s → Shutdown → 關資源
□ terminationGracePeriodSeconds > shutdownTimeout + 5s（留緩衝）
```

### 安全清單

```
□ 服務間通訊用 mTLS（Istio 或手動設定）
□ 不在 log 中輸出 Secret
□ JWT Secret 至少 256-bit 隨機
□ DB 連線用最低必要權限
□ 每個環境用不同的 Secret
□ 啟動時驗證所有必要配置
```

---

## 常見陷阱

### 過早微服務化

```
❌ 3 個工程師拆了 20 個微服務
   → 每個功能都需要改 5 個 repo
   → 本地開發需要跑 20 個 service
   → 成為「分散式單體」

✅ 從單體開始，等業務邊界清晰再拆
```

### 共用資料庫

```
❌ 所有服務共用同一個 PostgreSQL
   → 服務 A 的 schema 改動影響服務 B
   → 無法獨立擴展
   → 資料庫成為單點耦合

✅ Database per service（見 [[微服務架構設計原則]]）
```

### 沒有設計冪等

```
❌ 訂單服務重試 → 重複扣款
   → K8s 重啟 → 重複發通知

✅ 所有 POST / PATCH 操作都實作冪等性（見 [[冪等性設計]]）
```

### 同步呼叫鏈太長

```
❌ API → 訂單服務 → 庫存服務 → 金流服務 → 通知服務（鏈式同步）
   → 任一環節慢 100ms，整體慢 400ms
   → 任一環節掛掉，整體失敗

✅ 只保留最必要的同步呼叫（查詢庫存），其他用 Kafka 非同步
```

---

## 相關頁面（核心概念）

- [[微服務架構設計原則]] — 邊界設計、12-Factor、通訊策略
- [[gRPC設計與實戰]] — Protobuf、Streaming、Interceptor、錯誤碼
- [[Circuit Breaker熔斷器]] — 故障隔離、sony/gobreaker、Fallback
- [[Saga Pattern分散式事務]] — Choreography vs Orchestration、Outbox
- [[CQRS與Event Sourcing]] — Command/Query 分離、Event Store
- [[事件驅動架構與Kafka]] — Producer/Consumer、DLQ、Fan-out
- [[服務發現與負載均衡]] — K8s DNS、Consul、Least Connections
- [[OpenTelemetry分散式追蹤]] — Traces/Metrics/Logs、OTel SDK
- [[Go優雅關機與健康檢查]] — SIGTERM、Liveness/Readiness、Watchdog
- [[冪等性設計]] — Idempotency Key、ON CONFLICT、Kafka 去重
- [[分散式鎖]] — Redis SETNX、Redlock、Watchdog 續期
- [[Go微服務框架比較]] — Kratos / go-kit / go-zero / Gin
- [[Go依賴注入與Wire]] — Wire / uber-go/fx / 手動 DI
- [[Go介面設計模式]] — Repository / Decorator / Strategy / Options
- [[Go微服務配置管理]] — Viper / K8s Secret / Vault
- [[Go錯誤處理最佳實踐]] — 分層錯誤、Sentinel / Custom / Wrapping

### 已有頁面（Go 核心）

- [[Go執行期內部機制]] — GMP、GC、Escape Analysis
- [[Go並發模式]] — Channel、Worker Pool、Fan-in/Fan-out
- [[Go效能調優]] — pprof、GOGC、sync.Pool、PGO
