---
title: Spectra
type: entity
tags: [OpenSpec, GUI, 桌面應用, SDD, 台灣]
created: 2026-05-26
updated: 2026-05-26
sources: [kaochenlong-spectra-openspec]
---

# Spectra

[[OpenSpec]] 的桌面圖形介面（GUI）管理工具，由高見龍（[[高見龍]]）以 SDD 方法論開發。目標是讓規格管理脫離終端機，提供視覺化的 specs/changes/tasks 操作體驗。

- **下載**：https://spectra.5xcamp.us/
- **CLI 啟動**：`spectra .`

## 核心功能

### 規格瀏覽
- 視覺化瀏覽 `specs/`、`changes/`、`archive/` 目錄
- 自動解析 delta spec，區分 ADDED / MODIFIED / REMOVED 區塊
- 顯示每份文件的最後修改時間

### 全文搜尋
- SQLite 索引支援高效搜尋
- 模糊搜尋，顯示行號與上下文
- 取代在終端機手動 grep 規格文件

### 任務追蹤介面
- 將 `tasks.md` 的 checkbox 解析為互動清單
- 勾選/取消直接寫回 `tasks.md`
- 拖曳重排優先順序
- 進度指示（如「3/5」）
- 批次操作：全部完成 / 重置進度

### 即時檔案監控
- 監聽 `openspec/` 目錄變動，AI 修改檔案後自動重載，不需手動刷新

### AI 工具配置
- 圖形化管理 Claude Code、Cursor、Windsurf 等 AI 工具的 OpenSpec 整合
- 自動生成對應設定檔與目錄結構

### 其他
- 六種主題（深色/淺色）、繁體中文/英文
- 備份/還原：`openspec/` 目錄匯出為 ZIP
- 自動更新、最近專案側欄（可星號標記）
- 備忘錄：獨立記事區，用於想法草稿，與規格文件分離

## 開發背景

Spectra 本身以 SDD 方法論開發，是 [[OpenSpec]] 的實際應用示範。

## 相關頁面

- [[OpenSpec]] — 底層框架
- [[OpenSpec工作流]] — Spectra 視覺化呈現的工作流程
- [[高見龍]] — 作者
