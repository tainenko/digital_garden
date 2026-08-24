---
title: Python 資料庫整合
type: concept
tags: [python, sqlalchemy, asyncpg, postgresql, orm, database]
created: 2026-04-30
updated: 2026-04-30
sources: []
---

# Python 資料庫整合

## 工具選型

| 工具 | 類型 | 適用場景 | 特點 |
|------|------|---------|------|
| SQLAlchemy 2.x | ORM + Core | 複雜查詢 + 多資料庫 | 功能最全，2.x 原生 async |
| asyncpg | 低層驅動 | 極高效能 PostgreSQL | 最快的 pg 驅動，無 ORM |
| psycopg3 | 驅動 | 同步 + async PostgreSQL | psycopg2 繼任者 |
| SQLModel | ORM | FastAPI + SQLAlchemy 簡化 | Pydantic + SQLAlchemy 合一 |
| Tortoise ORM | ORM | async-first 專案 | Django-like，pure async |

**推薦組合**：FastAPI 微服務用 **SQLAlchemy 2.x + asyncpg**；中小型專案可用 **SQLModel**。

---

## SQLAlchemy 2.x 核心概念

### 模型定義

```python
from sqlalchemy import String, Numeric, ForeignKey, func
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column, relationship
from datetime import datetime
from decimal import Decimal

class Base(DeclarativeBase):
    pass

class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)
    email: Mapped[str] = mapped_column(String(255), unique=True, nullable=False)
    name: Mapped[str] = mapped_column(String(100))
    created_at: Mapped[datetime] = mapped_column(server_default=func.now())

    orders: Mapped[list["Order"]] = relationship(back_populates="user")

class Order(Base):
    __tablename__ = "orders"

    id: Mapped[int] = mapped_column(primary_key=True)
    user_id: Mapped[int] = mapped_column(ForeignKey("users.id"), index=True)
    amount: Mapped[Decimal] = mapped_column(Numeric(12, 2))
    status: Mapped[str] = mapped_column(String(20), default="pending")

    user: Mapped["User"] = relationship(back_populates="orders")
```

`Mapped[T]` 是 SQLAlchemy 2.x 的型別化語法，讓 mypy 可以正確推斷型別。

### 引擎與 Session（同步版）

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import Session, sessionmaker

engine = create_engine(
    "postgresql+psycopg2://user:pass@localhost/db",
    pool_size=10,
    max_overflow=20,
    pool_pre_ping=True,  # 自動偵測死連線
    echo=False,          # True 時印出 SQL（debug 用）
)
SessionLocal = sessionmaker(bind=engine, autocommit=False, autoflush=False)

# 使用
with Session(engine) as session:
    user = session.get(User, 1)
```

### 引擎與 Session（非同步版）

```python
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession, async_sessionmaker

async_engine = create_async_engine(
    "postgresql+asyncpg://user:pass@localhost/db",
    pool_size=10,
    max_overflow=20,
)
AsyncSessionLocal = async_sessionmaker(async_engine, expire_on_commit=False)

# FastAPI 依賴注入
async def get_db() -> AsyncGenerator[AsyncSession, None]:
    async with AsyncSessionLocal() as session:
        yield session
```

---

## CRUD 操作

### 查詢

```python
from sqlalchemy import select, and_, or_, desc

async def get_user(db: AsyncSession, user_id: int) -> User | None:
    result = await db.execute(select(User).where(User.id == user_id))
    return result.scalar_one_or_none()

# 複雜條件
async def search_orders(
    db: AsyncSession,
    user_id: int,
    status: str | None = None,
    skip: int = 0,
    limit: int = 20,
) -> list[Order]:
    stmt = (
        select(Order)
        .where(Order.user_id == user_id)
        .order_by(desc(Order.created_at))
        .offset(skip)
        .limit(limit)
    )
    if status:
        stmt = stmt.where(Order.status == status)
    result = await db.execute(stmt)
    return list(result.scalars().all())

# JOIN
async def get_orders_with_user(db: AsyncSession) -> list[Order]:
    stmt = select(Order).join(Order.user).options(
        selectinload(Order.user)  # 避免 N+1
    )
    result = await db.execute(stmt)
    return list(result.scalars().all())
```

### 寫入

```python
async def create_user(db: AsyncSession, email: str, name: str) -> User:
    user = User(email=email, name=name)
    db.add(user)
    await db.commit()
    await db.refresh(user)  # 取得資料庫生成的欄位（id, created_at）
    return user

async def update_order_status(db: AsyncSession, order_id: int, status: str) -> Order | None:
    order = await db.get(Order, order_id)
    if not order:
        return None
    order.status = status
    await db.commit()
    await db.refresh(order)
    return order

async def delete_user(db: AsyncSession, user_id: int) -> bool:
    user = await db.get(User, user_id)
    if not user:
        return False
    await db.delete(user)
    await db.commit()
    return True
```

---

## N+1 問題與解決

```python
# ❌ N+1：查 N 個 orders，每個再查一次 user
orders = (await db.execute(select(Order))).scalars().all()
for order in orders:
    print(order.user.name)  # 每次都發一個 SELECT

# ✅ selectinload：一次 IN 查詢
from sqlalchemy.orm import selectinload
stmt = select(Order).options(selectinload(Order.user))

# ✅ joinedload：JOIN 一次搞定（適合 many-to-one）
from sqlalchemy.orm import joinedload
stmt = select(Order).options(joinedload(Order.user))
```

---

## 原生 SQL（Core / Text）

```python
from sqlalchemy import text

# 複雜報表查詢用原生 SQL
async def get_monthly_revenue(db: AsyncSession, year: int, month: int):
    result = await db.execute(
        text("""
            SELECT DATE_TRUNC('day', created_at) AS day, SUM(amount) AS revenue
            FROM orders
            WHERE status = 'completed'
              AND EXTRACT(YEAR FROM created_at) = :year
              AND EXTRACT(MONTH FROM created_at) = :month
            GROUP BY 1
            ORDER BY 1
        """),
        {"year": year, "month": month},
    )
    return result.mappings().all()
```

---

## asyncpg 直接使用

無 ORM，效能最高：

```python
import asyncpg

# 連線池
async def create_pool():
    return await asyncpg.create_pool(
        dsn="postgresql://user:pass@localhost/db",
        min_size=5,
        max_size=20,
    )

# 使用
async def get_user(pool: asyncpg.Pool, user_id: int):
    async with pool.acquire() as conn:
        row = await conn.fetchrow("SELECT * FROM users WHERE id = $1", user_id)
        return dict(row) if row else None

# 批次插入（極高效能）
async def bulk_insert_orders(pool: asyncpg.Pool, orders: list[dict]):
    async with pool.acquire() as conn:
        await conn.executemany(
            "INSERT INTO orders (user_id, amount) VALUES ($1, $2)",
            [(o["user_id"], o["amount"]) for o in orders],
        )
```

---

## 資料庫 Migration（Alembic）

```bash
pip install alembic
alembic init alembic
```

```python
# alembic/env.py
from app.models import Base
target_metadata = Base.metadata
```

```bash
alembic revision --autogenerate -m "add_orders_table"
alembic upgrade head
alembic downgrade -1
alembic history
```

**生產環境 migration 原則**：
1. 先加新欄位（nullable 或有 default）
2. 部署新版應用
3. 回填資料
4. 加 NOT NULL constraint
5. 不在一次 migration 同時改結構 + 改資料

---

## 事務管理

```python
# 自動事務（session context manager 自動 commit/rollback）
async with AsyncSessionLocal() as session:
    async with session.begin():
        session.add(Order(user_id=1, amount=100))
        session.add(OrderLog(order_id=1, action="created"))
    # 離開 begin() 自動 commit；例外自動 rollback

# 巢狀事務（Savepoint）
async with session.begin_nested():
    # 這段失敗只 rollback 到 savepoint，不影響外層
    ...
```

---

## 常見面試問題

**Q: SQLAlchemy Session 的 expire_on_commit 是什麼？**
A: 預設 True，commit 後所有物件屬性標記為過期，下次訪問會再發 SELECT。async 場景應設 False，因為 commit 後 session 可能已關閉。

**Q: 如何防止 SQL Injection？**
A: 永遠使用參數化查詢（`where(User.id == user_id)` 或 `text("... WHERE id = :id")` + bind params），絕不拼接字串。

**Q: ORM vs 原生 SQL 如何選？**
A: CRUD 和業務查詢用 ORM（可讀性好、防注入）；複雜報表、批次操作、效能瓶頸點用原生 SQL。

**Q: pool_pre_ping 的作用？**
A: 從連接池取出連線前先 ping 一下，若連線已死（資料庫重啟等情況）自動捨棄並重建，避免「使用死連線」導致的 500 錯誤。

---

## 相關頁面

- [[Python FastAPI深度實戰]] — async session 與 Depends 整合
- [[Python測試策略]] — testcontainers + session rollback 隔離
- [[Go PostgreSQL測試策略]] — 對照 Go 的測試策略
- [[DDD領域驅動設計]] — Repository pattern 在資料層的運用
