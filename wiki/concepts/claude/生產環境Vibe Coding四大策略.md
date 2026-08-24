---
title: 生產環境 Vibe Coding 四大策略
type: concept
tags: [claude, vibe-coding, 生產環境, TDD, ai開發]
created: 2026-04-29
updated: 2026-04-29
sources: [schluntz-vibe-coding-production-4rules.md]
---

# 生產環境 Vibe Coding 四大策略

由 [[Erik Schluntz]]（Anthropic MTS）從內部 22,000 行生產環境合併案例中總結，針對工程師在正式系統中安全使用 AI 編程的核心方法。

---

## 策略一：可驗證的抽象層（Verifiable Abstraction Layer）

建立測試體系，使驗證 AI 輸出**無需閱讀底層實作程式碼**。

- 優先寫測試（TDD），測試即規格，AI 的任務是讓測試通過
- 閱讀順序：**先看測試碼，再看實作碼**——測試碼是意圖，實作碼是細節
- 測試覆蓋越高，可信任 AI 的範圍越大；可信任範圍越大，速度越快
- 對應 [[Superpowers技能框架]] 的 TDD RED → GREEN → REFACTOR 循環

## 策略二：葉節點策略（Leaf Node Strategy）

聚焦代碼庫中的**末端功能單元**（leaf nodes），讓 AI 處理邊緣，人類保護主幹。

- Leaf node：只被其他模組呼叫、自身不依賴核心系統狀態的功能單元
- 允許在 leaf node 區域累積技術債，集中在後期清理，不影響系統主幹穩定性
- 高風險的核心路徑（auth、資料庫 schema、API contract）仍由人工主導

## 策略三：前置規劃工作流（Pre-Planning Workflow）

在 AI 正式執行任務前，先投入 **15–20 分鐘**進行結構化溝通：

1. 讓 Claude 探索代碼庫，找出相關檔案與依賴
2. 共同制定清晰的執行計畫與限制條件
3. 將規格、需求、邊界整合成**一個完整提示**後交付執行
4. 效果：任務成功率呈指數級提升，後期反覆修改的成本大幅降低

與 [[OpenSpec工作流]] 的 explore → propose → design → specs 階段高度對應。

## 策略四：充分上下文準備（Rich Context Preparation）

上下文品質直接決定 AI 輸出品質：

- 提供**代碼庫導航路徑**：告訴 Claude 從哪裡開始閱讀、哪些模組相關
- 提供**需求規格**：功能要求 + 非功能要求（效能、安全、相容性）
- 提供**明確限制條件**：哪些不能動、哪些有既有約定
- 避免讓 AI 在資訊不足的情況下「自由發揮」，那是最貴的錯誤

---

## 案例：22,000 行生產環境合併

- **背景**：Anthropic 內部強化學習代碼庫，重構任務
- **成果**：原估兩週 → 實際 1 天完成，程式碼主要由 Claude 生成
- **關鍵**：前置充分規劃 + 可驗證測試體系 + 人工負責核心邏輯決策

---

## 工程師角色轉型

Erik Schluntz 主張，實踐以上策略後，工程師的核心職責轉變為：

| 舊職責 | 新職責 |
|--------|--------|
| 撰寫實作程式碼 | 設計測試規格 |
| 逐行 Debug | 驗證 AI 輸出的邊界行為 |
| 學習語言語法 | 利用 AI 加速架構試錯 |
| 技術執行者 | Claude 的「產品經理」 |

---

## 相關頁面

- [[Erik Schluntz]]
- [[Vibe Coding基礎概念]]
- [[Vibe Coding風險與限制]]
- [[OpenSpec工作流]]
- [[Superpowers技能框架]]
- [[Spec驅動開發]]
