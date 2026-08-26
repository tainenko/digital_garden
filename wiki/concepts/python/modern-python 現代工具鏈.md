---
title: modern-python 現代工具鏈
type: concept
tags: [python, uv, ruff, ty, 工具鏈, AI輔助開發, Trail-of-Bits]
created: 2026-08-25
updated: 2026-08-25
sources: [trailofbits-modern-python-skill]
---

# modern-python 現代工具鏈

Trail of Bits 出品的 AI agent skill，讓 AI（Claude Code、Cursor、Codex）生成符合 2026 年標準的 Python 程式碼。定位是**工具鏈現代化**，對比 [[go-modern-guidelines 現代化規則]] 的語言層語法現代化。

**安裝（Claude Code）**：`/skill add trailofbits/modern-python`

---

## 工具鏈對照：舊 → 新

| 面向 | 舊工具 | 現代替代 | 優勢 |
|------|--------|---------|------|
| 套件管理 | pip + requirements.txt | **uv** | 速度快 10-100x、統一環境管理 |
| 依賴鎖定 | pip-tools、Poetry | **uv**（uv.lock） | 跨平台確定性 |
| 虛擬環境 | virtualenv、venv | **uv**（自動管理） | 不需手動 activate |
| Lint | flake8 + isort | **ruff** | 單一工具、Rust 實作極快 |
| 格式化 | black | **ruff format** | 與 ruff lint 整合 |
| 型別檢查 | mypy、pyright | **ty** | Ruff 團隊開發、速度更快 |
| 測試 | unittest | **pytest** + pytest-asyncio | 更簡潔的測試語法 |
| pre-commit | pre-commit | **prek** | 更輕量 |

---

## 核心規則

### 1. 用 `uv` 管理一切，不手動 activate

```bash
# ❌ 舊式
python -m venv .venv
source .venv/bin/activate
pip install requests

# ✅ 現代
uv add requests          # 自動管理 .venv
uv run python script.py  # 在 venv 中執行，不需 activate
uv run pytest            # 測試同理
```

### 2. 依賴分層：dependency-groups

```toml
# pyproject.toml

[project]
dependencies = [
    "fastapi>=0.110",
    "pydantic>=2.0",
]

# ✅ 現代：用 dependency-groups（不是 optional extras）
[dependency-groups]
dev = ["ruff>=0.4", "ty>=0.1", "pytest>=8.0", "pytest-cov"]
test = ["pytest>=8.0", "pytest-asyncio", "httpx"]
docs = ["mkdocs", "mkdocs-material"]
```

```bash
uv sync --group dev      # 安裝 dev 依賴
uv sync --group test     # 安裝 test 依賴
```

### 3. pyproject.toml 統一設定

```toml
[tool.ruff]
line-length = 88
target-version = "py311"

[tool.ruff.lint]
select = ["E", "F", "I", "UP", "B", "SIM"]  # UP = pyupgrade 規則
ignore = ["E501"]

[tool.ruff.format]
quote-style = "double"

[tool.ty]
python-version = "3.11"

[tool.pytest.ini_options]
asyncio_mode = "auto"
testpaths = ["tests"]
```

### 4. 型別提示：現代語法（Python 3.10+）

```python
# ❌ 舊式（Python 3.9 以前需要 from __future__ import annotations）
from typing import Optional, List, Dict, Union

def process(items: Optional[List[str]] = None) -> Dict[str, int]:
    ...

# ✅ 現代
def process(items: list[str] | None = None) -> dict[str, int]:
    ...
```

### 5. 新專案腳手架

```bash
# 建立新專案
uv init my-project
cd my-project

# 加入依賴
uv add fastapi pydantic
uv add --group dev ruff ty pytest pytest-cov

# 執行
uv run fastapi dev src/main.py

# Lint + Format
uv run ruff check .
uv run ruff format .

# 型別檢查
uv run ty check .

# 測試
uv run pytest --cov=src
```

---

## 與 go-modern-guidelines 的對比

| | go-modern-guidelines | modern-python |
|--|---------------------|--------------|
| **出品方** | JetBrains（官方） | Trail of Bits（資安公司） |
| **重點** | 語言新功能（slices、cmp.Or…） | 工具鏈替換（uv、ruff、ty） |
| **Claude Code** | `/use-modern-go` | `/skill add trailofbits/modern-python` |
| **原因** | AI 訓練截止日造成的語法落後 | AI 不知道 uv/ruff 已取代舊工具 |

---

## 使用限制

- 需要 Python **3.11+**（ty 的部分功能需要更新版本）
- ty 尚在快速迭代，成熟度不及 mypy（某些邊緣型別語法支援不完整）
- 企業環境若有 Nexus/Artifactory 私有 registry，需另外設定 uv 的 index 來源

---

## 相關頁面

- [[Python套件管理]] — uv 詳細用法
- [[Python型別系統與類型提示]] — ty / mypy 型別系統
- [[Python測試策略]] — pytest 詳細設定
- [[go-modern-guidelines 現代化規則]] — Go 語言的對應工具
