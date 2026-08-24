---
title: Spectra：給 OpenSpec 的圖形介面（高見龍）
type: source-summary
tags: [OpenSpec, Spectra, GUI, v1.0, 工具, 台灣]
created: 2026-05-26
updated: 2026-05-26
sources: [kaochenlong-spectra-openspec]
---

# Spectra：給 OpenSpec 的圖形介面（高見龍）

## Origin
- **作者**：高見龍（[[高見龍]]）
- **日期**：2026-02-03
- **URL**：https://kaochenlong.com/spectra-with-openspec

## Key Takeaways

### OpenSpec 1.0 架構變化

- **非線性工作流**：舊版強制線性執行 `proposal → apply → archive`；1.0 允許任意順序執行，系統自行維護狀態
- **指令前綴改為 `/opsx:`**，取代舊版 `/openspec:` 前綴
- **新指令**：
  - `/opsx:explore` — 腦力激盪，不生成文件
  - `/opsx:new` — 建立提案（取代 `/openspec:proposal`）
  - `/opsx:continue` — 依依賴關係逐一生成文件
  - `/opsx:ff`（fast-forward）— 一次性生成所有所需文件
  - `/opsx:apply`、`/opsx:archive` — 同舊版語意
- **即時任務進度追蹤**：每次執行前讀取 `tasks.md` 計算完成率（如「3/5 tasks complete」），不再仰賴 AI 自行判斷
- **DAG 文件依賴管理**：`spec.md` 依賴 `proposal.md`；`tasks.md` 依賴 `spec.md` + `design.md`；拓撲排序確保生成順序正確
- **Skills 系統整合**：從 slash commands 改為存放在 `.claude/skills/` 的 Skills，相容 Claude Code、Cursor、Windsurf 等多工具
- **配置統一為 `config.yaml`**：由 `openspec/project.md` 遷移至 `openspec/config.yaml`，確保各 AI 工具穩定存取技術規格與架構慣例

### Spectra — 桌面 GUI 應用

- **定位**：OpenSpec 的桌面視覺管理器，用 SDD 方法論開發
- **下載**：https://spectra.5xcamp.us/
- **核心功能**：
  1. Specs/Changes/Archives 視覺瀏覽，自動解析 delta spec 的 ADDED/MODIFIED/REMOVED
  2. SQLite 全文搜尋 + 模糊搜尋，顯示行號與上下文（取代 grep）
  3. 任務追蹤介面：勾選框→直接更新 `tasks.md`、拖曳排序、進度顯示、批次操作
  4. 即時檔案監控：`openspec/` 目錄變動自動重載
  5. AI 工具配置 GUI：圖形化管理支援的 AI 工具，自動生成設定檔
  6. 六種主題（深色/淺色）、繁體中文/英文
  7. 備份/還原：匯出/匯入 `openspec/` 為 ZIP
  8. CLI 整合：`spectra .` 快速啟動
  9. 自動更新、最近專案、備忘錄功能

## Entities Mentioned
- [[高見龍]] — 作者
- [[OpenSpec]] — 本文核心框架（v1.0 更新）
- [[Spectra]] — 本文介紹的新 GUI 工具

## Concepts Mentioned
- [[OpenSpec工作流]] — v1.0 非線性化與新指令
- [[OpenSpec文件格式與驗證]] — config.yaml 確認為當前正確命名

## Contradictions Resolved
- **config.yaml vs project.md**：前一篇（kaochenlong-openspec）提到 `project.md`；本文確認 v1.0 已統一為 `config.yaml`。`project.md` 為中間過渡版本命名。[[OpenSpec文件格式與驗證]] 中的矛盾注記已可解除。
