---
title: Vibe Coding 生產部署四階段
type: concept
tags: [vibe-coding, 生產部署, 安全, 架構, DevOps, claude-code]
created: 2026-04-30
updated: 2026-04-30
sources: []
---

# Vibe Coding 生產部署四階段

Vibe Coding 能快速原型，但「能跑的 demo」和「可以給真實用戶的生產系統」之間有一條鴻溝。這四個階段是跨越這條鴻溝的系統性方法。

> 「Vibe Coding 生產化是一條四段流水線：稽核 AI 真正做了什麼 → 強化無聊但關鍵的基礎設施 → 觀察真實環境中的運作 → 在沙箱後部署。」

---

## 為什麼 Demo 不等於 Production

| 維度 | Demo / 原型 | 生產系統 |
|------|------------|---------|
| **使用者** | 你一個人 | 真實用戶，可能帶惡意 |
| **數據** | 假數據或你的數據 | 用戶的真實資料，需要保護 |
| **錯誤** | 壞掉重來就好 | 每個錯誤都有成本 |
| **規模** | 一次一個請求 | 並發、峰值、邊緣案例 |
| **可觀測性** | 不需要 | 必要（不然出事不知道） |

AI 生成的代碼通常能解決你描述的問題，但不會主動考慮你沒提到的邊緣情況。

---

## 第一階段：稽核（Audit）

**核心問題**：AI 真正做了什麼？

在任何「讓外人看到」之前，徹底理解 AI 生成的代碼。

### 1.1 架構稽核

用 Claude 幫你做架構審查：

```
請閱讀整個 src/ 目錄，然後：
1. 描述現有的架構（資料流、主要模組、依賴關係）
2. 找出所有的外部依賴（API、資料庫、第三方服務）
3. 找出所有涉及用戶資料的程式碼路徑
4. 標出任何你認為在生產環境可能有問題的地方
```

### 1.2 安全稽核

不要假設 AI 知道安全最佳實踐。讓 Claude 做安全審查：

```
對以下安全問題進行完整稽核：

1. 輸入驗證（Input Validation）
   - 所有用戶輸入有沒有做驗證？
   - SQL 查詢有沒有用 parameterized query？
   - 文件上傳有沒有做類型和大小限制？

2. 認證與授權（Auth）
   - Session token 的存儲和傳輸方式是否安全？
   - 每個 endpoint 是否正確驗證用戶身份？
   - 是否有水平越權漏洞（A 用戶可以看到 B 用戶的資料）？

3. 密鑰管理
   - 有沒有硬編碼的 API key、密碼、secret？
   - 是否依賴 .env 文件但沒有 .gitignore 保護？

輸出：問題清單，包含嚴重程度（Critical/High/Medium/Low）和具體位置（file:line）
```

### 1.3 數據模型稽核

AI 最容易在數據模型上埋下長期技術債：

```
審查資料庫 schema（migrations/ 或 schema.sql），檢查：
1. 是否有沒有索引的外鍵？
2. 是否有可能在業務增長後成為瓶頸的設計？
3. 用戶敏感資料（email、電話、地址）是否有加密或其他保護？
4. 是否有適當的 NOT NULL 和 UNIQUE 約束？
```

---

## 第二階段：強化（Harden）

**核心問題**：把「無聊但關鍵」的基礎設施補齊。

### 2.1 環境與密鑰管理

```bash
# .env.example（提交到 git，只有 key 沒有 value）
DATABASE_URL=postgres://user:password@host:5432/dbname
OPENAI_API_KEY=sk-...
STRIPE_SECRET_KEY=sk_...
JWT_SECRET=your-jwt-secret-here

# .env（不提交，真實值）
# 用 secret manager（AWS Secrets Manager、GCP Secret Manager）管理
```

用 Claude 建立配置驗證：

```python
# config.py
import os
from typing import Optional

REQUIRED_ENV_VARS = [
    "DATABASE_URL",
    "JWT_SECRET",
    "STRIPE_SECRET_KEY"
]

def validate_config():
    missing = [var for var in REQUIRED_ENV_VARS if not os.getenv(var)]
    if missing:
        raise RuntimeError(f"Missing required env vars: {missing}")
    
validate_config()  # 在應用啟動時執行
```

### 2.2 認證強化

AI 常見問題：自己實作 JWT 但忽略 expiry、自己實作 session 但用明文存儲。

```
審查並強化認證系統：
1. JWT token 是否設置了合理的過期時間（access: 15 分鐘，refresh: 7 天）？
2. Session 是否存儲在安全的 httpOnly、SameSite Cookie？
3. 密碼是否用 bcrypt 或 argon2 哈希（不是 MD5 或 SHA1）？
4. 是否有防暴力破解機制（rate limiting on /login）？
5. 是否有帳號鎖定機制（多次失敗後鎖定）？

如果有問題，直接修復，不要只是報告。
```

### 2.3 Rate Limiting

```python
# 使用 slowapi (FastAPI) 或 flask-limiter (Flask)
from slowapi import Limiter
from slowapi.util import get_remote_address

limiter = Limiter(key_func=get_remote_address)

@app.post("/login")
@limiter.limit("5/minute")  # 登入：每分鐘 5 次
async def login(request: Request, ...):
    ...

@app.post("/api/generate")  
@limiter.limit("10/minute;100/hour")  # AI 功能：更嚴格
async def generate(request: Request, ...):
    ...
```

### 2.4 錯誤處理標準化

AI 生成的代碼常見問題：把 internal error details 直接暴露給用戶。

```python
# 標準化錯誤回應
class ErrorResponse(BaseModel):
    error: str          # 給用戶看的訊息（不含 internal details）
    code: str           # 錯誤代碼（用於前端處理）
    request_id: str     # 追蹤 ID（用於 debug）

@app.exception_handler(Exception)
async def global_exception_handler(request: Request, exc: Exception):
    request_id = generate_request_id()
    logger.error(f"[{request_id}] Unhandled exception: {exc}", exc_info=True)
    
    return JSONResponse(
        status_code=500,
        content=ErrorResponse(
            error="An unexpected error occurred. Please try again.",
            code="INTERNAL_ERROR",
            request_id=request_id
        ).dict()
    )
```

### 2.5 Input Validation

```python
# 使用 Pydantic 做輸入驗證（FastAPI 內建）
from pydantic import BaseModel, validator, constr
import re

class CreateUserRequest(BaseModel):
    email: str
    password: constr(min_length=8, max_length=128)
    username: constr(min_length=3, max_length=50, regex="^[a-zA-Z0-9_-]+$")
    
    @validator('email')
    def validate_email(cls, v):
        if not re.match(r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$', v):
            raise ValueError('Invalid email format')
        return v.lower()
```

---

## 第三階段：觀察（Observe）

**核心問題**：在真實環境中，發生了什麼？

你無法改善你看不到的東西。

### 3.1 結構化日誌

```python
import structlog
import uuid

log = structlog.get_logger()

# 每個 request 綁定一個追蹤 ID
@app.middleware("http")
async def add_request_id(request: Request, call_next):
    request_id = str(uuid.uuid4())
    with structlog.contextvars.bound_contextvars(request_id=request_id):
        response = await call_next(request)
        response.headers["X-Request-ID"] = request_id
        return response

# 在業務邏輯中使用
async def create_order(user_id: int, items: list):
    log.info("creating_order", user_id=user_id, item_count=len(items))
    try:
        order = await db.create_order(user_id, items)
        log.info("order_created", order_id=order.id, total=order.total)
        return order
    except Exception as e:
        log.error("order_creation_failed", error=str(e), user_id=user_id)
        raise
```

### 3.2 關鍵指標監控

告訴 Claude 你的業務 KPI，讓它幫你設定監控：

```
為這個電商應用設定 Prometheus 指標監控，重點追蹤：

業務指標：
- 每分鐘訂單數量
- 訂單成功率 vs 失敗率
- 平均訂單金額

技術指標：
- API P50/P95/P99 延遲（按 endpoint 分組）
- 錯誤率（4xx 和 5xx 分開）
- 資料庫查詢時間

告警規則：
- 錯誤率 > 1% 持續 5 分鐘 → PagerDuty
- P95 延遲 > 2 秒 持續 5 分鐘 → Slack
- 訂單成功率 < 98% → 緊急告警
```

### 3.3 健康檢查

```python
@app.get("/health")
async def health_check():
    checks = {}
    
    # 資料庫連通性
    try:
        await db.execute("SELECT 1")
        checks["database"] = "ok"
    except Exception as e:
        checks["database"] = f"error: {e}"
    
    # Redis 連通性（如果有）
    try:
        await redis.ping()
        checks["redis"] = "ok"
    except Exception as e:
        checks["redis"] = f"error: {e}"
    
    status = "healthy" if all(v == "ok" for v in checks.values()) else "degraded"
    return {"status": status, "checks": checks}
```

---

## 第四階段：部署（Deploy）

**核心問題**：安全地把系統交給真實用戶。

### 4.1 Staging 環境

永遠不要直接把 Vibe Coding 的輸出部署到 production：

```
環境流水線：
Local Dev → Staging → Production

Staging 環境要求：
- 使用 production 相同的基礎設施（不用 SQLite 替代 PostgreSQL）
- 使用匿名化的生產數據快照（不用假數據）
- 獨立的第三方 API key（Stripe sandbox、Twilio test）
```

### 4.2 Feature Flags

避免大爆炸式發布，用功能旗標控制誰可以看到新功能：

```python
# 使用 GrowthBook 或 LaunchDarkly
from growthbook import GrowthBook

def get_gb(user_id: int) -> GrowthBook:
    gb = GrowthBook(
        attributes={"id": user_id},
        features=fetch_features_from_api()
    )
    return gb

@app.post("/api/generate")
async def generate(request: Request, user: User):
    gb = get_gb(user.id)
    
    if gb.is_on("new-generate-algorithm"):
        return await new_generate_algorithm(request)
    else:
        return await old_generate_algorithm(request)
```

### 4.3 Database Migration 安全

AI 很容易生成「刪除欄位」或「修改欄位類型」的 migration，這在有真實用戶的情況下非常危險：

```
審查以下 database migration，確認它在有 10,000 行以上的 production 表上是否安全執行：
[貼上 migration SQL]

特別注意：
- 這個 migration 是否需要 table lock？
- 是否可以做 zero-downtime migration？
- 回滾計畫是什麼？
- 執行時間估算（基於表大小）？
```

### 4.4 漸進式上線（Gradual Rollout）

```python
# 用 feature flag 做 10% → 50% → 100% 的漸進上線
def should_use_new_feature(user_id: int, rollout_percentage: int = 10) -> bool:
    # 確定性隨機：同一個 user_id 永遠得到相同結果
    import hashlib
    hash_val = int(hashlib.md5(str(user_id).encode()).hexdigest(), 16)
    return (hash_val % 100) < rollout_percentage
```

---

## 不可委派給 AI 的工作

即使 AI 能做大部分生產化工作，有些決策必須由人類做：

| 決策 | 原因 |
|------|------|
| 定義「可接受的錯誤率」 | 這是業務決策，不是技術決策 |
| 用戶數據的保留政策 | 法律責任、隱私法規 |
| 服務等級協議（SLA）| 對用戶的承諾 |
| 安全事件的回應決策 | 需要判斷業務影響 |
| 功能旗標的最終開關 | 需要人類確認業務就緒 |

---

## 部署前 Checklist

```
安全
✅ 所有用戶輸入都有驗證
✅ SQL 查詢使用 parameterized queries
✅ 沒有硬編碼的 secrets
✅ 認證 token 有適當的過期設定
✅ Rate limiting 在關鍵 endpoint 上生效

基礎設施
✅ 環境變數驗證（應用啟動時）
✅ 健康檢查 endpoint 存在且正確
✅ 結構化日誌配置完成
✅ 錯誤不會暴露 internal details 給用戶

部署
✅ Staging 環境測試通過
✅ Database migration 安全性確認
✅ 回滾計畫就緒
✅ 告警規則配置完成
✅ 監控 dashboard 建立
```

---

## 相關頁面

- [[生產環境Vibe Coding四大策略]]（Erik Schluntz 的四大原則）
- [[CLAUDE.md撰寫最佳實踐]]（讓 Claude 在整個部署過程中遵循安全規範）
- [[Claude Code Hooks 深度設定]]（自動化稽核和強化流程）
- [[Skills實戰：自動交易機器人]]（高風險自動化的護欄設計）
- [[Claude Prompt工程核心技巧]]（如何描述生產需求給 Claude）
