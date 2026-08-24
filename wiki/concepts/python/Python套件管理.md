---
title: Python 套件管理
type: concept
tags: [python, uv, poetry, pyproject, packaging, virtualenv]
created: 2026-04-30
updated: 2026-04-30
sources: []
---

# Python 套件管理

## 工具演進

```
pip + requirements.txt  →  pipenv  →  Poetry  →  uv（2024 現代標準）
```

| 工具 | 速度 | 鎖定檔 | 虛擬環境 | 發布套件 | 推薦度 |
|------|------|-------|---------|---------|-------|
| pip | 慢 | ❌ | 需搭配 venv | ❌ | 舊專案維護 |
| Poetry | 中 | ✅ | ✅ 自動 | ✅ | 穩定成熟 |
| **uv** | **極快（Rust）** | ✅ | ✅ 自動 | ✅ | **2025+ 推薦** |

uv 比 pip 快 **10–100 倍**，由 Astral（Ruff 同一團隊）開發，已是 2025 年業界新標準。

---

## uv 完整工作流

### 安裝

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
# 或
pip install uv
```

### 新建專案

```bash
uv init my-project        # 建立專案結構
cd my-project

# 結構：
# my-project/
# ├── pyproject.toml
# ├── .python-version     ← Python 版本鎖定
# ├── src/
# │   └── my_project/
# │       └── __init__.py
# └── uv.lock             ← 鎖定檔（提交到 git）
```

### 日常指令

```bash
# 新增依賴
uv add fastapi
uv add "sqlalchemy>=2.0"
uv add --dev pytest pytest-cov  # 開發依賴

# 移除依賴
uv remove requests

# 安裝所有依賴（從 uv.lock）
uv sync
uv sync --no-dev               # 只裝 production 依賴

# 執行指令（無需 activate 虛擬環境）
uv run python main.py
uv run pytest
uv run uvicorn app.main:app --reload

# 更新依賴
uv lock --upgrade              # 更新 uv.lock
uv lock --upgrade-package fastapi  # 只更新特定套件
```

### Python 版本管理

```bash
uv python install 3.12        # 安裝指定版本
uv python install 3.11 3.12   # 安裝多版本
uv python list                 # 列出已安裝版本
uv python pin 3.12             # 專案鎖定版本（寫入 .python-version）
```

### 工具（全域指令）

```bash
uv tool install ruff           # 安裝 CLI 工具（隔離環境）
uv tool install black
uvx ruff check .               # 一次性執行（無需安裝）
uvx black --check .
```

---

## pyproject.toml 完整格式

```toml
[project]
name = "my-project"
version = "1.0.0"
description = "API 服務"
readme = "README.md"
license = { text = "MIT" }
authors = [{ name = "Tony", email = "tony@example.com" }]
requires-python = ">=3.12"
dependencies = [
    "fastapi>=0.115",
    "sqlalchemy>=2.0",
    "asyncpg>=0.29",
    "pydantic-settings>=2.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=8.0",
    "pytest-asyncio>=0.23",
    "pytest-cov>=5.0",
    "httpx>=0.27",      # FastAPI TestClient 需要
    "ruff>=0.4",
    "mypy>=1.10",
]

[project.scripts]
serve = "app.main:run"   # uv run serve 執行

[tool.uv]
dev-dependencies = [     # uv 專屬語法（與 optional-dependencies 二選一）
    "pytest>=8.0",
    "ruff>=0.4",
]

# Ruff 設定
[tool.ruff]
line-length = 100
target-version = "py312"

[tool.ruff.lint]
select = ["E", "F", "I", "UP", "B"]
ignore = ["E501"]

[tool.ruff.lint.per-file-ignores]
"tests/**" = ["S101"]   # 允許 assert

# mypy 設定
[tool.mypy]
python_version = "3.12"
strict = true
ignore_missing_imports = true

# pytest 設定
[tool.pytest.ini_options]
asyncio_mode = "auto"
testpaths = ["tests"]
addopts = "--cov=app --cov-report=term-missing"

# coverage 設定
[tool.coverage.run]
omit = ["*/migrations/*", "*/tests/*"]
```

---

## Poetry（仍在使用的專案）

```bash
# 安裝
curl -sSL https://install.python-poetry.org | python3 -

# 常用指令
poetry new my-project          # 新建
poetry init                    # 現有目錄初始化
poetry add fastapi             # 新增依賴
poetry add --group dev pytest  # 新增開發依賴
poetry install                 # 安裝（從 poetry.lock）
poetry install --only main     # 只裝 production
poetry run pytest              # 執行指令
poetry shell                   # 進入虛擬環境
poetry update                  # 更新所有依賴
poetry build                   # 打包
poetry publish                 # 發布到 PyPI
```

---

## 虛擬環境最佳實踐

```bash
# uv 自動管理，無需手動建立
# 虛擬環境位置：.venv/（專案根目錄）

# 若需要手動
uv venv
source .venv/bin/activate     # macOS/Linux
.venv\Scripts\activate        # Windows

# 確認使用正確的 Python
which python
python --version
```

### .gitignore 必加

```
.venv/
__pycache__/
*.pyc
*.pyo
.mypy_cache/
.ruff_cache/
.coverage
htmlcov/
dist/
*.egg-info/
```

---

## 發布套件到 PyPI

```bash
# 1. 確認 pyproject.toml 設定正確
# 2. 建立 dist/
uv build                       # 產生 .whl 和 .tar.gz

# 3. 發布
uv publish                     # 上傳到 PyPI
uv publish --index testpypi    # 先測試

# 設定 API token（一次性）
# ~/.config/uv/credentials.toml 或環境變數
export UV_PUBLISH_TOKEN=pypi-xxxx
```

---

## CI/CD 整合

### GitHub Actions

```yaml
# .github/workflows/ci.yml
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v3      # 官方 action
        with:
          version: "latest"
          enable-cache: true             # 快取 uv 下載
      - run: uv sync --frozen            # 嚴格使用 lock 檔
      - run: uv run ruff check .
      - run: uv run mypy app/
      - run: uv run pytest
```

`--frozen`：不更新 lock 檔，確保 CI 與本機環境一致。

---

## 常見面試問題

**Q: requirements.txt 和 lock 檔的差異？**
A: requirements.txt 通常只指定直接依賴，無法精確重現環境（間接依賴版本不確定）。lock 檔（poetry.lock / uv.lock）鎖定所有依賴的精確版本，確保任何人安裝都完全一致。

**Q: 為何不把 .venv 提交到 git？**
A: 虛擬環境包含平台相關二進位檔，不跨平台；檔案數量龐大（通常數千個）；應透過 lock 檔重現。

**Q: uv 如何做到 10–100x 速度提升？**
A: 用 Rust 實現、並行下載、高效快取機制、只解析必要的 metadata（不下載整個套件才解析依賴）。

**Q: optional-dependencies vs dev-dependencies？**
A: optional-dependencies 是 PEP 621 標準（pip install "pkg[dev]" 可安裝）；uv 的 dev-dependencies 是 uv 專屬的開發依賴語法，不打包進發布版本，兩者功能相近但語法不同。

---

## 相關頁面

- [[Python型別系統與類型提示]] — mypy 型別檢查整合
- [[Python測試策略]] — pytest 與 CI 配置
- [[Python FastAPI深度實戰]] — 完整專案結構
- [[Vibe Coding基礎概念]] — AI 輔助開發環境設定
