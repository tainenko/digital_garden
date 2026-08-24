---
title: Python FastAPI 深度實戰
type: concept
tags: [python, fastapi, web, api, async, pydantic]
created: 2026-04-30
updated: 2026-04-30
sources: []
---

# Python FastAPI 深度實戰

## 為何選 FastAPI

| 框架 | 效能 | 型別支援 | 異步 | 學習曲線 | 適用場景 |
|------|------|---------|------|---------|---------|
| FastAPI | ★★★★★ | 原生 Pydantic | 原生 async | 低 | API / 微服務 |
| Django REST | ★★★ | 弱 | 需外掛 | 高 | 全棧 + 管理後台 |
| Flask | ★★★ | 弱 | 需外掛 | 低 | 小型 API |
| Litestar | ★★★★★ | 原生 | 原生 async | 中 | FastAPI 替代 |

FastAPI 底層是 **Starlette**（ASGI）+ **Pydantic**（驗證），效能接近 Node.js。

---

## 專案結構

```
app/
├── main.py              ← FastAPI 入口
├── api/
│   ├── __init__.py
│   ├── deps.py          ← 共用依賴（DB session、JWT 解析）
│   └── v1/
│       ├── orders.py    ← router
│       └── users.py
├── models/              ← SQLAlchemy ORM models
├── schemas/             ← Pydantic request/response schemas
├── services/            ← 業務邏輯層
├── repositories/        ← 資料存取層
└── core/
    ├── config.py        ← 環境變數設定
    └── security.py      ← JWT、密碼雜湊
```

---

## 核心概念

### 路由與 HTTP 方法

```python
from fastapi import APIRouter, status

router = APIRouter(prefix="/orders", tags=["orders"])

@router.get("/", response_model=list[OrderResponse])
async def list_orders(skip: int = 0, limit: int = 20):
    return await order_service.list(skip=skip, limit=limit)

@router.post("/", response_model=OrderResponse, status_code=status.HTTP_201_CREATED)
async def create_order(body: CreateOrderRequest, db: Session = Depends(get_db)):
    return await order_service.create(db, body)

@router.get("/{order_id}", response_model=OrderResponse)
async def get_order(order_id: int, db: Session = Depends(get_db)):
    order = await order_service.get(db, order_id)
    if not order:
        raise HTTPException(status_code=404, detail="Order not found")
    return order

@router.delete("/{order_id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete_order(order_id: int):
    await order_service.delete(order_id)
```

### Pydantic Schema（Request / Response 分離）

```python
from pydantic import BaseModel, Field, model_validator
from decimal import Decimal
from datetime import datetime

class CreateOrderRequest(BaseModel):
    user_id: int
    amount: Decimal = Field(gt=0, description="訂單金額，必須大於 0")
    currency: str = Field(default="TWD", pattern="^[A-Z]{3}$")

    @model_validator(mode="after")
    def check_amount_precision(self):
        if self.amount.as_tuple().exponent < -2:
            raise ValueError("最多兩位小數")
        return self

class OrderResponse(BaseModel):
    id: int
    user_id: int
    amount: Decimal
    status: str
    created_at: datetime

    model_config = {"from_attributes": True}  # 允許從 ORM 物件轉換
```

### 依賴注入（Depends）

```python
# api/deps.py
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from app.core.security import decode_token
from app.db import AsyncSession, get_async_session

bearer_scheme = HTTPBearer()

async def get_current_user(
    credentials: HTTPAuthorizationCredentials = Depends(bearer_scheme),
    db: AsyncSession = Depends(get_async_session),
):
    payload = decode_token(credentials.credentials)
    if not payload:
        raise HTTPException(status_code=401, detail="Invalid token")
    user = await user_repo.get(db, user_id=payload["sub"])
    if not user:
        raise HTTPException(status_code=401, detail="User not found")
    return user

# 使用
@router.get("/me", response_model=UserResponse)
async def get_me(current_user: User = Depends(get_current_user)):
    return current_user
```

---

## 異步資料庫（SQLAlchemy 2.x async）

```python
# core/database.py
from sqlalchemy.ext.asyncio import create_async_engine, async_sessionmaker, AsyncSession
from sqlalchemy.orm import DeclarativeBase

engine = create_async_engine(
    "postgresql+asyncpg://user:pass@localhost/db",
    pool_size=10,
    max_overflow=20,
)
AsyncSessionLocal = async_sessionmaker(engine, expire_on_commit=False)

class Base(DeclarativeBase):
    pass

async def get_async_session():
    async with AsyncSessionLocal() as session:
        yield session

# 在 Repository 使用
class OrderRepository:
    async def get(self, db: AsyncSession, order_id: int) -> Order | None:
        result = await db.execute(
            select(Order).where(Order.id == order_id)
        )
        return result.scalar_one_or_none()

    async def create(self, db: AsyncSession, data: CreateOrderRequest) -> Order:
        order = Order(**data.model_dump())
        db.add(order)
        await db.commit()
        await db.refresh(order)
        return order
```

---

## 錯誤處理

### 統一錯誤格式

```python
from fastapi import Request
from fastapi.responses import JSONResponse

class AppError(Exception):
    def __init__(self, code: str, message: str, status_code: int = 400):
        self.code = code
        self.message = message
        self.status_code = status_code

@app.exception_handler(AppError)
async def app_error_handler(request: Request, exc: AppError):
    return JSONResponse(
        status_code=exc.status_code,
        content={"error": exc.code, "message": exc.message},
    )

@app.exception_handler(RequestValidationError)
async def validation_error_handler(request: Request, exc: RequestValidationError):
    return JSONResponse(
        status_code=422,
        content={"error": "VALIDATION_ERROR", "details": exc.errors()},
    )
```

---

## Middleware

```python
import time
from fastapi import Request

@app.middleware("http")
async def add_process_time_header(request: Request, call_next):
    start = time.perf_counter()
    response = await call_next(request)
    elapsed = time.perf_counter() - start
    response.headers["X-Process-Time"] = f"{elapsed:.4f}"
    return response

# CORS
from fastapi.middleware.cors import CORSMiddleware
app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://yourdomain.com"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

---

## Background Tasks

```python
from fastapi import BackgroundTasks

def send_welcome_email(email: str):
    # 同步函數也可以，FastAPI 會在 thread pool 執行
    email_client.send(to=email, subject="Welcome")

@router.post("/users", status_code=201)
async def create_user(body: CreateUserRequest, background_tasks: BackgroundTasks):
    user = await user_service.create(body)
    background_tasks.add_task(send_welcome_email, email=user.email)
    return user  # 立即回傳，email 在背景寄送
```

---

## Lifespan（啟動/關閉事件）

```python
from contextlib import asynccontextmanager

@asynccontextmanager
async def lifespan(app: FastAPI):
    # 啟動：初始化連接池、載入模型等
    await redis_pool.init()
    yield
    # 關閉：清理資源
    await redis_pool.close()
    await engine.dispose()

app = FastAPI(lifespan=lifespan)
```

---

## 自動文件

FastAPI 自動產生：
- **Swagger UI**：`http://localhost:8000/docs`
- **ReDoc**：`http://localhost:8000/redoc`
- **OpenAPI JSON**：`http://localhost:8000/openapi.json`

自訂文件：
```python
app = FastAPI(
    title="Order API",
    description="訂單管理系統 API",
    version="1.0.0",
    docs_url="/api/docs",
    redoc_url=None,  # 關閉 ReDoc
)
```

---

## 效能最佳化

```python
# 1. 使用 async 所有 I/O 路徑
@router.get("/orders")
async def list_orders():  # ✅ async
    ...

# 2. 同步函數自動跑 thread pool（避免阻塞 event loop）
@router.get("/report")
def generate_report():  # FastAPI 偵測 sync，自動用 threadpool
    return heavy_sync_computation()

# 3. 連接池調優
engine = create_async_engine(url, pool_size=20, max_overflow=40, pool_timeout=30)

# 4. Response Model 排除 None
class OrderResponse(BaseModel):
    model_config = {"from_attributes": True}

@router.get("/", response_model=list[OrderResponse], response_model_exclude_none=True)
async def list_orders(): ...
```

---

## 常見面試問題

**Q: FastAPI 為何比 Flask 快？**
A: FastAPI 基於 ASGI（異步）而非 WSGI（同步），可同時處理大量 I/O 請求而不阻塞。Flask 每個請求佔用一個 thread。

**Q: Depends() 的執行順序？**
A: 依賴形成 DAG，FastAPI 會自動分析依賴鏈並按順序執行。相同依賴在一個請求中只執行一次（預設 use_cache=True）。

**Q: 如何做請求限流？**
A: 搭配 `slowapi`（基於 limits 庫）或在 Nginx/API Gateway 層做；生產環境建議在反向代理層處理。

**Q: 如何處理大型 File Upload？**
A: 使用 `UploadFile`（streaming），避免一次性讀入記憶體：
```python
@router.post("/upload")
async def upload(file: UploadFile):
    contents = await file.read()  # 小檔案
    # 大檔案用 file.read(chunk_size) 分塊處理
```

---

## 相關頁面

- [[Python測試策略]] — FastAPI TestClient 測試
- [[Python資料庫整合]] — SQLAlchemy async 模式
- [[Python非同步程式設計]] — asyncio 底層機制
- [[Go優雅關機與健康檢查]] — 對照 Go 的 lifespan 管理
