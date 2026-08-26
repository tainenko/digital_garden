---
title: Trail of Bits modern-python skill — 現代 Python 工具鏈指南
type: source-summary
tags: [python, AI輔助開發, uv, ruff, 工具鏈, Trail-of-Bits]
created: 2026-08-25
updated: 2026-08-25
sources: [trailofbits-modern-python-skill]
---

# Trail of Bits modern-python skill

## Origin

- **名稱**：modern-python（Trail of Bits Skills）
- **出品方**：Trail of Bits（美國資安公司）
- **版本**：v1.3.0（最後更新 2026-08）
- **GitHub**：trailofbits/skills（6,900+ ⭐）
- **安裝**：`/skill add trailofbits/modern-python`（Claude Code）
- **來源**：基於 trailofbits/cookiecutter-python 模板
- **授權**：AGPL-3.0

## Key Takeaways

1. **定位是工具鏈現代化**，不是語言功能現代化（對比 [[go-modern-guidelines 現代化規則]] 的語言層規則）
2. **uv 全面取代 pip/virtualenv/Poetry**：統一套件管理、環境管理、腳本執行
3. **ruff 合併 lint + format**：一個 Rust 工具取代 flake8 + black + isort，速度快 10-100x
4. **ty 取代 mypy/pyright**：新一代型別檢查器（Ruff 團隊開發）
5. **不手動 activate 虛擬環境**：改用 `uv run`，環境管理完全交給 uv
6. **dependency-groups 分層**：dev/test/docs 用 `[dependency-groups]`，不用 optional extras
7. **適用場景**：新專案建立、pyproject.toml 設定、從舊工具鏈遷移
8. **不適用**：專案需要 Python < 3.11、或明確要保留舊工具鏈

## Entities Mentioned

- [[Trail of Bits]] — 出品方，美國知名資安研究公司
- [[uv]] — Astral 開發的 Python 套件管理工具
- [[ruff]] — Astral 開發的 Python Linter/Formatter

## Concepts Mentioned

- [[modern-python 現代工具鏈]] — 核心主題

## Contradictions / Tensions

- ty 是新工具，成熟度不如 mypy；部分舊型別語法支援度尚不完整
- uv 的普及速度快，但企業環境可能仍依賴 pip + requirements.txt 的舊流程
- AGPL-3.0 授權：商業閉源專案使用需留意

## Questions Raised

- ty 什麼時候能完全取代 mypy？目前缺少哪些功能？
- uv 的 workspace 功能是否足以取代 Poetry 的 monorepo 管理？
- 其他語言有沒有類似的社群 skill？（Rust、TypeScript？）
