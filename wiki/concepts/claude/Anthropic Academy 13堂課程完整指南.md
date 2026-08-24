---
title: Anthropic Academy 13 堂課程完整指南
type: concept
tags: [claude, anthropic-academy, 課程, 免費, 學習路線]
created: 2026-04-30
updated: 2026-04-30
sources: [bnext-anthropic-academy-13-courses.md]
---

# Anthropic Academy 13 堂課程完整指南

[[Anthropic Academy]] 是 Anthropic 官方免費學習平台，提供 13 堂系統性課程，從 AI 素養到進階 Agent 開發一應俱全。全部免費，部分課程完成後頒發結業證書。

**報名方式**：前往 academy.anthropic.com 註冊，選擇課程後點「Enroll in Course」即可開始。

---

## 一、開發技術實戰類（8 堂）

### 1. Claude 101（AI 基礎應用）

| 項目 | 內容 |
|------|------|
| 證書 | 無 |
| 先修 | 無技術要求 |
| 適合 | 一般大眾、職場人士 |

**學習內容**：
- 將 Claude 應用於日常工作任務
- 使用 Projects 和 Artifacts 組織知識
- 探索企業搜尋與研究工作模式

**什麼人應該上**：完全沒用過 Claude 的新手，或是想把 Claude 用在工作的非技術人員。

---

### 2. Claude Code in Action（實戰 Vibe Coding）

| 項目 | 內容 |
|------|------|
| 堂數 | 15 堂 |
| 時長 | 約 1 小時 |
| 證書 | ✓ |
| 先修 | 熟悉 CLI，需有 Claude Code 存取權 |
| 適合 | 軟體工程師、開發者 |

**學習內容**：
- 檔案操作與 shell 指令執行
- 使用 `/init`、`CLAUDE.md` 管理專案上下文
- 啟用計畫模式（Plan Mode）與延伸思考（Extended Thinking）
- 建立自動化指令腳本與 GitHub 整合
- 編寫 Hooks 實現自動化流程

**什麼人應該上**：想用 Claude Code 做 Vibe Coding 的工程師，這是最直接對應的官方課程。

**延伸閱讀**：[[Claude Code 入門完整指南]]、[[Claude Code Hooks 深度設定]]、[[CLAUDE.md撰寫最佳實踐]]

---

### 3. Introduction to Agent Skills（Agent 技能入門）

| 項目 | 內容 |
|------|------|
| 證書 | 無 |
| 先修 | 具備 Claude Code 基礎經驗 |
| 適合 | 開發團隊、軟體工程師 |

**學習內容**：
- 撰寫 `SKILL.md` 建立可重複使用的指令技能
- 使用**漸進式揭露**（Progressive Disclosure）保持 Context 效率
- 設定 `allowed-tools` 限制技能的工具存取範圍
- 透過外掛或企業管理在團隊間分享技能
- 排解技能觸發失敗或優先級衝突問題

**什麼人應該上**：想為團隊建立標準化 AI 工作流、讓 Claude 技能可跨成員複用的開發者。

**延伸閱讀**：[[Superpowers技能框架]]

---

### 4. Building with the Claude API（API 開發完整課）

| 項目 | 內容 |
|------|------|
| 堂數 | 84 堂 |
| 時長 | 約 8.1 小時 |
| 證書 | ✓ |
| 先修 | 精通 Python、熟悉 JSON、有 API Key |
| 適合 | 軟體工程師 |

**學習內容**：
- API 請求處理、多輪對話設計、結構化輸出
- 系統化測試與評估提示詞（Prompt Eval）
- 實作 RAG 系統（文本分塊、嵌入、混合搜尋）
- 建構代理架構（Agentic Architecture）
- 整合 Model Context Protocol（MCP）

**什麼人應該上**：想把 Claude 整合進自己產品、建構 AI 應用的工程師。這是最完整的 API 開發課。

**延伸閱讀**：[[Claude API基礎與最佳實踐]]、[[Claude Agent 設計模式]]

---

### 5. Introduction to Model Context Protocol（MCP 入門）

| 項目 | 內容 |
|------|------|
| 堂數 | 16 堂 |
| 時長 | 約 1 小時 |
| 證書 | ✓ |
| 先修 | 基礎 Python、了解 async 模式、熟悉 API 概念 |
| 適合 | 工程師 |

**學習內容**：
- MCP 架構與主客從（Host / Client / Server）通訊模型
- 使用 Python SDK 建構 MCP Server
- 實作 MCP Client 連接外部應用
- 使用 MCP Inspector 進行測試與除錯

**什麼人應該上**：想讓 Claude 連接外部工具（資料庫、API、本機檔案）的工程師。

**延伸閱讀**：[[Claude MCP 伺服器整合指南]]

---

### 6. Model Context Protocol: Advanced Topics（MCP 進階）

| 項目 | 內容 |
|------|------|
| 堂數 | 15 堂 |
| 時長 | 約 1.1 小時 |
| 證書 | ✓ |
| 先修 | 完成 MCP 入門、熟悉 async 程式設計 |
| 適合 | 進階工程師 |

**學習內容**：
- 實作具備日誌記錄與進度通知的 MCP Server
- 處理雙向通訊（Bidirectional Communication）
- 使用 Roots 權限模型設定安全的檔案系統存取
- 實作採樣回呼（Sampling Callback）啟用伺服器發起的 LLM 請求
- 比較 stdio 與 HTTP 傳輸模式
- 使用 JSON-RPC 訊息進行低層除錯

**什麼人應該上**：已建好基礎 MCP Server、想讓它更健壯、更安全、支援複雜互動的工程師。

**延伸閱讀**：[[Claude MCP 伺服器整合指南]]

---

### 7. Claude with Amazon Bedrock

| 項目 | 內容 |
|------|------|
| 堂數 | 85 堂 |
| 時長 | 約 8 小時 |
| 證書 | ✓ |
| 先修 | 精通 Python、有 AWS 帳號且已開通 Bedrock |
| 適合 | 使用 AWS 技術棧的開發者 |

**學習內容**：
- 透過 AWS Bedrock 發送 Claude 請求
- 建構 RAG 管道、多步驟工具執行工作流
- 最佳化 Prompt Caching 與 Extended Thinking
- 實作 Computer Use 進行 UI 自動化

---

### 8. Claude with Google Cloud's Vertex AI

| 項目 | 內容 |
|------|------|
| 堂數 | 85 堂 |
| 時長 | 約 8 小時 |
| 證書 | ✓ |
| 先修 | 精通 Python、有 GCP 帳號且已開通 Vertex AI |
| 適合 | 使用 GCP 技術棧的開發者 |

**學習內容**：
- 使用 Anthropic SDK 在 Vertex AI 進行身分驗證
- 根據智能、速度、成本選擇合適的 Claude 模型
- 建構 RAG 管道、實作網路搜尋與檔案操作
- 設計條件式路由與平行執行的代理工作流

---

## 二、AI 素養類（2 堂）

### 9. AI Fluency: Framework & Foundations（4D 框架入門）

| 項目 | 內容 |
|------|------|
| 堂數 | 14 堂 |
| 時長 | 約 1.1 小時 |
| 證書 | ✓ |
| 先修 | 無 |
| 適合 | 一般大眾、跨領域專業人士 |

**學習內容**：
- 理解生成式 AI 系統的基礎、能力與限制
- 深入學習 **4D 框架**：Delegate（委託）、Describe（描述）、Discern（洞察）、Diligent（勤勉）
- 針對創意、商業、教育情境撰寫有效提示詞並迭代

**4D 框架速覽**：

| D | 意義 | 實踐 |
|---|------|------|
| Delegate（委託） | 判斷哪些任務適合交給 AI | 分析任務風險與可逆性 |
| Describe（描述） | 清楚描述任務背景和要求 | 角色、目標、格式、限制 |
| Discern（洞察） | 批判性評估 AI 的輸出 | 驗證、查核、判斷 |
| Diligent（勤勉） | 負責任地使用 AI | 透明度、道德考量 |

---

### 10. Teaching AI Fluency（AI 素養教學設計）

| 項目 | 內容 |
|------|------|
| 堂數 | 7 堂 |
| 時長 | 約 0.6 小時 |
| 證書 | ✓ |
| 先修 | 建議先完成課程 9 |
| 適合 | 大學教師、教學設計師 |

---

## 三、教育應用類（3 堂）

| 課程 | 堂數 | 適合對象 | 證書 |
|------|------|---------|------|
| AI Fluency for Educators | 4堂 | 教師、教學設計師 | ✓ |
| AI Fluency for Students | 5堂 | 學生 | ✓ |
| AI Fluency for Nonprofits | 9堂 | 非營利組織 | ✓ |

三堂課都以 4D 框架為基礎，應用到各自的情境（教學、學業、組織營運）。

---

## 快速對照：我應該上哪堂課？

| 我的目標 | 推薦課程 |
|---------|---------|
| 第一次用 Claude，想提升工作效率 | Claude 101 → AI Fluency |
| 想學 Vibe Coding | Claude Code in Action |
| 想建立 Skills 讓團隊共用 | Introduction to Agent Skills |
| 想用 API 建構 AI 應用 | Building with the Claude API |
| 想學 MCP 連接外部工具 | MCP 入門 → MCP 進階 |
| 在 AWS 上部署 AI | Claude with Amazon Bedrock |
| 在 GCP 上部署 AI | Claude with Google Cloud Vertex AI |

完整的學習路線圖請見：[[Vibe Coding + Skills + MCP 學習路線圖]]

---

## 相關頁面

- [[Anthropic Academy]]
- [[Vibe Coding + Skills + MCP 學習路線圖]]
- [[Claude Code 入門完整指南]]
- [[Claude MCP 伺服器整合指南]]
- [[Superpowers技能框架]]
- [[Claude Agent 設計模式]]
