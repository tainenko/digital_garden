---
title: CoWork 桌面工具指南
type: concept
tags: [cowork, claude-code, anthropic, desktop, non-technical, agent]
created: 2026-04-30
updated: 2026-04-30
sources: [boris-cowork-startup-ideas-podcast]
---

# CoWork 桌面工具指南

## 什麼是 CoWork

CoWork 是 [[Anthropic]] 推出的桌面應用程式，底層是 **Claude Agent SDK**——和 Claude Code 用同一套引擎，但把終端機操作包進了圖形介面。

**目標用戶**：不會寫程式、不想碰命令列的人
**平台**：macOS（Windows 近期跟進）
**安裝**：裝好 CoWork + Chrome 擴充套件即可開始

---

## 核心能力示範（Boris Demo）

### 1. 資料夾操作
```
掛載資料夾 → 打一句話 → Claude 讀取檔案內容執行
「幫我把檔名改成收據上的日期」
→ Claude 讀取每張收據、辨識日期、重新命名
```

### 2. 逆向引導（Reverse Elicitation）
遇到不確定的情況，Claude **主動問你**而非自己猜：
```
遇到沒有日期的收據
→ Claude：「這張收據沒有日期，您希望我怎麼處理？」
```
這是設計行為，讓 AI 在不確定時向用戶求助，而非自行腦補。

### 3. 跨 App 自動化
Claude 可以直接操作瀏覽器和應用程式：
```
「把這個資料整理成試算表」→ 產出 Excel
「我想要 Google Sheet」→ 打開瀏覽器、開 Google Sheets、逐筆貼入
格式錯誤 → Claude 自己發現、自己修
「幫我打開 Gmail，把這個表寄給 Amy」→ 找到聯絡人、寫好郵件草稿
```

### 4. 並行多任務
Boris 的工作方式：**同時跑 5–10 個 Claude**
- 一個在整理收據
- 一個在做研究
- 一個在寄信
- Boris 在任務之間切換，像管理者而非執行者

> 單一任務人類可能更快，但並行之後整體效率完全倒轉。

---

## 安全架構

| 機制 | 說明 |
|------|------|
| 虛擬機沙盒 | 所有操作在獨立 VM 中，不影響主系統 |
| 刪除保護 | 刪除操作前先確認 |
| Prompt Injection 防護 | 內建惡意指令偵測 |
| Anthropic 安全研究 | 模型對齊訓練 → 機制可解釋性 → 產品層防護，三層完整體系 |

Anthropic 的理念：早期發布到真實環境中觀察，而非只在實驗室測試。

---

## 開始使用

1. 下載並安裝 CoWork
2. 安裝 Chrome 擴充套件（讓 Claude 能看到瀏覽器內容，自我驗證）
3. **不要急著客製化**：先掛載一個資料夾，從整理檔案、重新命名開始
4. 發現某類任務 CoWork 處理不好時，再考慮寫 Skills

### Skills 是什麼？

Skills 是**可重複使用的做事方式**。CoWork 內建一些 Skills（如 Excel 處理）。對於特殊工具（AutoCAD、Salesforce 等），自己寫一個 Skill，Claude 就能學會處理。Skills 越多，在相關任務上的表現越好。

---

## Boris 的 Claude Code 四大心法

（來自 99,000 書籤推文——Claude Code 的技法，在 CoWork 同樣適用）

### ① 計畫模式（Plan Mode）——最被低估的功能

幾乎每個 session 都從計畫模式開始：

```
用戶：「我想要 [目標]」
Claude：[思考、提問、釐清]
用戶：確認計畫
Claude：[自動執行]
```

> 「計畫好了，程式碼就好了。」

用 Opus 4.5 的話，一旦計畫確定，模型幾乎能一次到位。

### ② 永遠用 Opus 4.5 + 思考模式

違反直覺的成本邏輯：

```
Opus 每 token 更貴
↓
但 Opus 更聰明，不需要太多引導和重試
↓
最終用的 token 更少
↓
整體成本往往比用 Sonnet 更低
```

在複雜、需要多步推理的任務上，這個效應尤為明顯。

### ③ CLAUDE.md check in 到 Git，全團隊共同維護

```
Claude 做錯某件事 → 立刻加入 CLAUDE.md
Code Review 發現問題 → tag Claude，讓它更新 CLAUDE.md
```

這樣同一件事只需說一次，新成員、新 session 都自動繼承。

詳見 [[CLAUDE.md撰寫最佳實踐]]。

### ④ 給 Claude 驗證自己成果的方式

> 「就像畫家不能戴著眼罩畫畫，工程師不能寫了程式卻永遠不跑。」

讓 Claude 用 **Chrome 擴充套件**看到自己做出來的東西：
- Claude 看到頁面 → 發現錯誤 → 自己修 → 再確認
- 能自我驗證的 Claude，輸出品質大幅提升

---

## CoWork vs Claude Code

| | Claude Code | CoWork |
|-|------------|--------|
| 介面 | 終端機 CLI | 桌面 GUI |
| 目標用戶 | 工程師 | 所有人 |
| 底層引擎 | Claude Agent SDK | Claude Agent SDK（相同）|
| Skills | 需自己設定 | 內建 + 可擴充 |
| 沙盒 | 視設定而定 | 內建虛擬機 |

---

## 相關頁面

- [[Boris]] — Claude Code 創造者，四大心法來源
- [[Claude Code 入門完整指南]] — Claude Code 詳細教程
- [[CLAUDE.md撰寫最佳實踐]] — CLAUDE.md 最佳實踐
- [[Claude Code多人團隊協作指南]] — 團隊並行工作流
- [[Anthropic 日級迭代發布機制]] — Anthropic 的產品文化
