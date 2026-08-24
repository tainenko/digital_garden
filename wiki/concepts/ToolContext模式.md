---
title: ToolContext 模式
type: concept
tags: [agent-harness, tool-design, dependency-injection, typescript]
created: 2026-06-26
updated: 2026-06-26
sources: [alienzhou-zero2agent]
---

# ToolContext 模式

## 定義

Agent Harness 中用來統一傳遞工具運行環境的**顯式注入物件**。把原本散落在各工具內部的隱式假設（如 `process.cwd()`）集中到一個結構化物件中，再由 Loop 統一注入給每個工具呼叫。

## 問題背景

[[Zero2Agent]] 課程在 E01-S001、S002 時，`read_file` 和 `grep_search` 都隱式依賴 `process.cwd()` 來解析相對路徑。第三個工具 `find_files`（E01-S003）進來時，這個隱式假設被暴露——三個工具各自解析工作目錄，行為不一致，難以在不同執行環境下移植。

## 解法

定義 `ToolContext` 介面（`packages/core/src/tools/types.ts`），至少包含：

```typescript
interface ToolContext {
  cwd: string;         // 工作目錄，由 Agent 在啟動時注入
  // ... 後續可擴充：logger、permissions、session id 等
}
```

Loop（`packages/core/src/loop.ts`）在每次呼叫工具前，將 context 作為第二個參數傳入。工具函數簽名從 `tool(input)` 改為 `tool(input, ctx: ToolContext)`。

## 設計亮點

- **擴展點**：`ToolContext` 加新字段不需改工具簽名，向後兼容
- **相對路徑輸出**：工具輸出結果時使用相對於 `cwd` 的路徑，省 token + 工具鏈銜接一致性
- **顯式 > 隱式**：參考 [[好規格寫作原則]] 的「明確邊界」精神，避免隱式全域狀態導致的不可預測行為

## 類比

類似後端框架中的 `RequestContext` / `ExecutionContext`——把請求範圍內的共享狀態集中管理，而非各層自行從全域讀取。

## 相關頁面

- [[ReAct Pattern]] — Loop 是注入 ToolContext 的執行者
- [[AI Agent核心架構 Model+Context+Tools]] — Tools 需要 Context 才能正確定位環境
- [[alienzhou]] — ToolContext 模式的提出者（Zero2Agent E01-S003）
