---
title: Python 測試策略
type: concept
tags: [python, testing, pytest, mock, testcontainers, tdd]
created: 2026-04-30
updated: 2026-04-30
sources: []
---

# Python 測試策略

## 測試金字塔

```
        /E2E\          ← 少量，慢，高信心
       /------\
      /整合測試\        ← 中量，測邊界
     /----------\
    /  單元測試  \      ← 大量，快，低成本
   /--------------\
```

原則：**越底層越多，越頂層越少**。Python 專案常見比例 70% 單元 / 20% 整合 / 10% E2E。

---

## pytest 核心用法

### 基本結構

```python
# tests/test_order.py
import pytest
from app.order import OrderService

class TestOrderService:
    def test_create_order_success(self):
        svc = OrderService()
        order = svc.create(user_id=1, amount=100)
        assert order.id is not None
        assert order.amount == 100

    def test_create_order_negative_amount_raises(self):
        svc = OrderService()
        with pytest.raises(ValueError, match="amount must be positive"):
            svc.create(user_id=1, amount=-1)
```

### Fixture

```python
# conftest.py（自動被 pytest 發現，無需 import）
import pytest
from app.db import get_session
from app.models import Base

@pytest.fixture(scope="function")
def db_session():
    """每個測試獨立 session，自動 rollback"""
    session = get_session()
    yield session
    session.rollback()
    session.close()

@pytest.fixture(scope="module")
def client(app):
    """整個模組共用一個 test client"""
    return app.test_client()
```

Fixture scope 選擇：
| Scope | 重建時機 | 適用場景 |
|-------|---------|---------|
| `function`（預設）| 每個測試 | 有副作用的資源 |
| `class` | 每個 class | class 內共享狀態 |
| `module` | 每個檔案 | 初始化成本高 |
| `session` | 整次測試 | 唯讀共享資源 |

### 參數化測試

```python
@pytest.mark.parametrize("amount,expected_fee", [
    (100, 1),
    (1000, 8),
    (10000, 50),
])
def test_calculate_fee(amount, expected_fee):
    assert calculate_fee(amount) == expected_fee
```

### 標記與過濾

```python
@pytest.mark.slow
def test_heavy_computation():
    ...

@pytest.mark.skip(reason="API not ready")
def test_external_api():
    ...

@pytest.mark.skipif(sys.platform == "win32", reason="Unix only")
def test_unix_feature():
    ...
```

```bash
pytest -m "not slow"          # 跳過慢測試
pytest -m "slow"              # 只跑慢測試
pytest -k "test_order"        # 名稱過濾
pytest -x                     # 第一個失敗即停止
pytest --tb=short             # 縮短 traceback
pytest -v                     # 詳細輸出
```

---

## Mock 策略

### unittest.mock 核心用法

```python
from unittest.mock import MagicMock, patch, AsyncMock

# 直接建立 mock
def test_send_email_called():
    email_service = MagicMock()
    user_service = UserService(email_service=email_service)
    user_service.register(email="a@b.com")
    email_service.send.assert_called_once_with(to="a@b.com", subject="Welcome")

# patch 裝飾器（patch 目標：被測試模組的引用路徑）
@patch("app.order.payment_gateway")
def test_process_payment(mock_gateway):
    mock_gateway.charge.return_value = {"status": "ok"}
    result = process_payment(amount=100)
    assert result.success is True

# patch 作為 context manager
def test_get_price():
    with patch("app.pricing.redis_client") as mock_redis:
        mock_redis.get.return_value = b"99.9"
        price = get_price("AAPL")
        assert price == 99.9
```

### AsyncMock（async 函數）

```python
from unittest.mock import AsyncMock, patch

@pytest.mark.asyncio
async def test_async_fetch():
    with patch("app.client.httpx.AsyncClient.get", new_callable=AsyncMock) as mock_get:
        mock_get.return_value.json.return_value = {"price": 150}
        result = await fetch_stock_price("AAPL")
        assert result == 150
```

### Mock 的常見陷阱

```python
# ❌ 錯誤：patch 的路徑是模組定義處，不是使用處
@patch("requests.get")           # 不一定有效

# ✅ 正確：patch 被測試模組引用的名稱
@patch("app.order.requests.get") # 有效
```

### pytest-mock（更簡潔的語法）

```python
# pip install pytest-mock
def test_with_mocker(mocker):
    mock_send = mocker.patch("app.email.send_email")
    trigger_welcome_email(user_id=1)
    mock_send.assert_called_once()
```

---

## 依賴注入讓測試更容易

```python
# ❌ 難測試：直接依賴全域
class OrderService:
    def create(self, user_id: int, amount: float):
        db.session.add(...)  # 無法注入假 db

# ✅ 易測試：依賴注入
class OrderService:
    def __init__(self, repo: OrderRepository):
        self.repo = repo

    def create(self, user_id: int, amount: float):
        return self.repo.save(Order(user_id, amount))

# 測試時注入 mock
def test_create_order():
    mock_repo = MagicMock(spec=OrderRepository)
    mock_repo.save.return_value = Order(id=1, user_id=1, amount=100)
    svc = OrderService(repo=mock_repo)
    order = svc.create(user_id=1, amount=100)
    assert order.id == 1
```

---

## 整合測試：testcontainers-python

真實資料庫測試，無需手動啟動容器：

```python
# pip install testcontainers[postgres]
import pytest
from testcontainers.postgres import PostgresContainer
import psycopg2

@pytest.fixture(scope="session")
def postgres():
    with PostgresContainer("postgres:16") as pg:
        yield pg

@pytest.fixture(scope="function")
def db_conn(postgres):
    conn = psycopg2.connect(postgres.get_connection_url())
    conn.autocommit = False
    yield conn
    conn.rollback()
    conn.close()

def test_insert_and_query(db_conn):
    cursor = db_conn.cursor()
    cursor.execute("CREATE TABLE IF NOT EXISTS users (id SERIAL, name TEXT)")
    cursor.execute("INSERT INTO users (name) VALUES (%s)", ("Tony",))
    cursor.execute("SELECT name FROM users WHERE name = %s", ("Tony",))
    row = cursor.fetchone()
    assert row[0] == "Tony"
```

### 搭配 SQLAlchemy

```python
from testcontainers.postgres import PostgresContainer
from sqlalchemy import create_engine
from app.models import Base

@pytest.fixture(scope="session")
def engine():
    with PostgresContainer("postgres:16") as pg:
        engine = create_engine(pg.get_connection_url())
        Base.metadata.create_all(engine)
        yield engine
        Base.metadata.drop_all(engine)

@pytest.fixture
def session(engine):
    connection = engine.connect()
    transaction = connection.begin()
    Session = sessionmaker(bind=connection)
    session = Session()
    yield session
    session.close()
    transaction.rollback()
    connection.close()
```

---

## FastAPI 測試

```python
from fastapi.testclient import TestClient
from app.main import app

client = TestClient(app)

def test_create_order():
    response = client.post("/orders", json={"user_id": 1, "amount": 100})
    assert response.status_code == 201
    assert response.json()["id"] is not None

def test_auth_required():
    response = client.get("/orders/1")
    assert response.status_code == 401

# 覆蓋依賴（dependency override）
def override_get_db():
    yield mock_session

app.dependency_overrides[get_db] = override_get_db

def test_with_fake_db():
    response = client.get("/orders")
    assert response.status_code == 200
```

---

## 覆蓋率

```bash
pip install pytest-cov

pytest --cov=app --cov-report=term-missing --cov-report=html
# term-missing：顯示哪幾行未覆蓋
# html：生成 htmlcov/ 可視化報告
```

```ini
# pyproject.toml
[tool.pytest.ini_options]
addopts = "--cov=app --cov-fail-under=80"

[tool.coverage.run]
omit = ["*/migrations/*", "*/tests/*"]
```

覆蓋率目標：**業務邏輯 90%+ / 整體 80%**。覆蓋率不等於品質，測試斷言才是重點。

---

## CI 配置範例（GitHub Actions）

```yaml
# .github/workflows/test.yml
jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_PASSWORD: test
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - run: pip install uv && uv sync
      - run: uv run pytest --cov=app --cov-fail-under=80
        env:
          DATABASE_URL: postgresql://postgres:test@localhost/test
```

---

## 常見面試問題

**Q: Mock 和 Stub 的差異？**
A: Stub 只回傳固定值；Mock 額外驗證互動（呼叫次數、參數）。`MagicMock` 兼具兩者。

**Q: 為何不該只靠高覆蓋率？**
A: 測試可以跑過每一行但完全沒斷言，覆蓋率 100% 卻抓不到任何 bug。覆蓋率是必要條件非充分條件。

**Q: 什麼時候用 `scope="session"` fixture？**
A: 只用於初始化成本很高且完全唯讀的資源（如：啟動 testcontainer）。有副作用的資源應用 `function` scope 並清理。

**Q: 如何測試 private 方法？**
A: 通常不直接測試（實作細節），透過公開介面覆蓋。若難以透過公開介面測試，表示需要重構。

---

## 相關頁面

- [[Python非同步程式設計]] — asyncio 與 AsyncMock 搭配
- [[Python資料庫整合]] — SQLAlchemy session 與測試隔離
- [[Go測試基準與模糊測試]] — Go 測試策略對照
- [[Go PostgreSQL測試策略]] — testcontainers Go 版
