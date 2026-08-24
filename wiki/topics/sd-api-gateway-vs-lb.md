---
title: "SD題解：API Gateway vs Load Balancer"
type: topic
tags: [system-design, api-gateway, load-balancer, golang]
created: 2026-04-20
updated: 2026-04-20
---

# SD題解：API Gateway vs Load Balancer

> **難度**: 概念題 ｜ **頻率**: 高（架構設計必備概念）

---

## 核心差異

| 面向 | Load Balancer | API Gateway |
|------|--------------|------------|
| **Layer** | L4（TCP/UDP）或 L7（HTTP）| L7（HTTP/HTTPS）|
| **主要職責** | 流量分發，確保高可用 | 請求管理、安全、可觀測性 |
| **感知內容** | IP / Port / HTTP Header | 完整 HTTP 請求內容 |
| **常見功能** | Round Robin、Health Check、SSL Termination | Auth、Rate Limit、路由、協定轉換 |
| **部署位置** | 最前端 或 服務間 | 通常在 LB 後面、微服務前面 |
| **典型產品** | AWS ALB/NLB、Nginx、HAProxy | Kong、AWS API Gateway、Nginx（兼用）|

---

## 架構位置

```
Internet
    ↓
Load Balancer        ← 分散流量到多台 API Gateway
    ↓
API Gateway          ← 認證、限流、路由
    ↓         ↓         ↓
Service A  Service B  Service C   ← 各微服務
    ↓         ↓         ↓
     各自的 Load Balancer（可選）
```

---

## API Gateway 的核心功能

### 1. 認證與授權

```go
// JWT 驗證 Middleware
func AuthMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        token := r.Header.Get("Authorization")
        if token == "" {
            http.Error(w, "Unauthorized", http.StatusUnauthorized)
            return
        }
        // 驗證 JWT，取出 user context
        claims, err := validateJWT(token)
        if err != nil {
            http.Error(w, "Invalid token", http.StatusUnauthorized)
            return
        }
        ctx := context.WithValue(r.Context(), "userID", claims.UserID)
        next.ServeHTTP(w, r.WithContext(ctx))
    })
}
```

### 2. 路由（請求轉發）

```go
type Router struct {
    routes map[string]string // path prefix → upstream service URL
}

func (r *Router) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    for prefix, upstream := range r.routes {
        if strings.HasPrefix(req.URL.Path, prefix) {
            // 反向代理到對應的微服務
            proxy := httputil.NewSingleHostReverseProxy(mustParseURL(upstream))
            // 移除路由前綴，轉發給上游
            req.URL.Path = strings.TrimPrefix(req.URL.Path, prefix)
            proxy.ServeHTTP(w, req)
            return
        }
    }
    http.Error(w, "Not Found", http.StatusNotFound)
}
```

### 3. Rate Limiting + 請求聚合（BFF Pattern）

API Gateway 常實作 **Backend for Frontend（BFF）**：把多個微服務的資料聚合成一個回應，減少客戶端的 round-trip：

```go
// 單次請求聚合多個上游服務的回應
func aggregateHandler(w http.ResponseWriter, r *http.Request) {
    userID := r.Context().Value("userID").(string)

    // 並行呼叫多個服務
    var wg sync.WaitGroup
    var userInfo, orderHistory, recommendations interface{}

    wg.Add(3)
    go func() { defer wg.Done(); userInfo = fetchUserService(userID) }()
    go func() { defer wg.Done(); orderHistory = fetchOrderService(userID) }()
    go func() { defer wg.Done(); recommendations = fetchRecoService(userID) }()
    wg.Wait()

    json.NewEncoder(w).Encode(map[string]interface{}{
        "user":            userInfo,
        "orders":          orderHistory,
        "recommendations": recommendations,
    })
}
```

---

## Load Balancer 的演算法

```go
type LoadBalancer struct {
    backends []*Backend
    mu       sync.Mutex
    current  int // Round Robin 計數器
}

type Backend struct {
    URL     string
    Alive   bool
    Weight  int
    mu      sync.RWMutex
}

// Round Robin
func (lb *LoadBalancer) NextRoundRobin() *Backend {
    lb.mu.Lock()
    defer lb.mu.Unlock()

    for i := 0; i < len(lb.backends); i++ {
        idx := lb.current % len(lb.backends)
        lb.current++
        if lb.backends[idx].isAlive() {
            return lb.backends[idx]
        }
    }
    return nil // 所有節點都掛了
}

// Least Connections（最少連線數）
func (lb *LoadBalancer) NextLeastConnections() *Backend {
    lb.mu.Lock()
    defer lb.mu.Unlock()

    var best *Backend
    for _, b := range lb.backends {
        if !b.isAlive() {
            continue
        }
        if best == nil || b.activeConnections() < best.activeConnections() {
            best = b
        }
    }
    return best
}

// IP Hash（讓同一個 IP 固定打到同一台，Session Affinity）
func (lb *LoadBalancer) NextIPHash(clientIP string) *Backend {
    h := fnv32(clientIP)
    aliveBackends := lb.aliveBackends()
    if len(aliveBackends) == 0 {
        return nil
    }
    return aliveBackends[h%uint32(len(aliveBackends))]
}

// Health Check
func (lb *LoadBalancer) HealthCheck() {
    for _, b := range lb.backends {
        go func(backend *Backend) {
            for {
                resp, err := http.Get(backend.URL + "/health")
                backend.mu.Lock()
                backend.Alive = err == nil && resp.StatusCode == 200
                backend.mu.Unlock()
                time.Sleep(10 * time.Second)
            }
        }(b)
    }
}
```

---

## 面試要點

- LB 解決**可用性**問題（一台掛了，流量自動轉移）
- API Gateway 解決**橫切關注點**問題（Auth/RateLimit 不用每個服務自己實作）
- 兩者不互斥，通常同時存在
- 單體應用不需要 API Gateway；微服務架構幾乎必須有

## 相關概念

- [[sd-rate-limiter|SD題解：Rate Limiter]] — API Gateway 的重要功能之一
- [[sd-sso|SD題解：SSO]] — API Gateway 的認證整合
- [[分散式系統基礎概念]] — Load Balancing 理論
