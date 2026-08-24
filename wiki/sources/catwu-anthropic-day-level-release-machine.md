---
title: 從 6 個月到 1 天：Anthropic 產品負責人 Cat Wu 揭秘日級發布機器
type: source-summary
tags: [anthropic, claude-code, cowork, PM, 日級迭代, cat-wu, 產品管理]
created: 2026-04-30
updated: 2026-04-30
sources: [catwu-anthropic-day-level-release-machine.md]
---

# 從 6 個月到 1 天：Anthropic 產品負責人 Cat Wu 揭秘日級發布機器

**Origin**：微信公眾號（天空之城），mp.weixin.qq.com/s/56kWqjeg6oaIUIZbIR7VRw  
**Author**：天空之城整理，原始為 Lenny's Podcast 訪談  
**Guest**：Cat Wu，Anthropic 雲端編碼與協作產品負責人（Claude Code + CoWork）  
**Type**：長篇訪談逐字稿（約 3 萬字）

---

## Key Takeaways

1. **迭代速度已從 6–12 個月壓縮至 1 天**：Anthropic 透過「研究預覽版」（Research Preview）機制，讓工程師從產生想法到週末就能上線，PM 的核心工作從跨季度協調轉為清障
2. **PM 新核心能力是「產品品味」與定義評測集（Evals）**：隨著寫程式成本降低，決定「寫什麼」遠比「怎麼寫」更有價值
3. **100% 自動化才是真正的杠桿**：95% 準確率的自動化仍需人工審核，只有達到 100% 才能真正釋放時間
4. **Claude 的個性（Character）是核心競爭力**：謙遜、幽默、積極行動的個性，讓使用體驗截然不同於其他模型
5. **Anthropic 的統一使命簡化了所有決策**：「安全 AGI」使命置於任何個別產品之上，使跨組織決策速度極快
6. **工程師 / PM / 設計師的邊界正在消失**：Anthropic 傾向招聘「無自負心、能跨職能行動」的人
7. **多 Agent 協作是下一個構建模組**：從單任務成功 → 多任務並行 → 數百個 Agent 同時執行
8. **CoWork 的核心場景**：非程式碼的知識工作（簡報、郵件、客戶準備）；連接 Slack、Google Drive 等數據源後，品質大幅提升
9. **Applied AI 團隊是 Anthropic 第二大 Token 使用方**（僅次於工程師），用 CoWork 做客戶前置準備
10. **模型升級讓產品功能可以「減少」**：每次新模型讓系統提示詞（System Prompt）可以縮短、功能可以簡化

---

## 核心引言

> 「構建 AI 原生產品的關鍵在於快速迭代，並找到一種能讓你真正做到每週發布功能的方法。」

> 「隨著代碼編寫成本變得越來越低，真正變得更有價值的是決策能力——決定要寫什麼。」

> 「只有達到 100% 準確率的自動化流程才能產生真正的杠桿效應。95% 的準確率往往意味著它還不是真正的自動化。」

> 「人類大腦的價值在於處理隱性知識、情商、常識，以及定義成功的『評測集』（Evals）。」

> 「只有當 Agent 不僅能告訴你怎麼做、還能代你執行任務的那一刻，才是真正的頓悟時刻。」

---

## 結構摘要

### 速度即戰略
- 功能開發週期：6 個月 → 1 個月 → 1 天
- 關鍵機制：Research Preview（降低發布承諾壓力）
- 常青發布室（Evergreen Launch Room）：工程師發布後，文件、PMM、DevRel 各就各位，次日即可發公告
- PM 的新工作：清障（Unblock），不是協調路線圖

### PM 角色演進
- 舊：跨季度協調 + PRD + 跨部門對齊
- 新：定義「黃金路徑」、製作 Evals、建立指標體系、週會讀指標
- PRD 仍存在，但只用於「特別模糊的功能」或「需要大量基礎設施的專案」
- 核心技能：產品品味（Product Taste）——極度稀缺

### Claude Code vs Desktop vs Mobile vs CoWork
| 工具 | 最適場景 |
|------|---------|
| Claude Code CLI | 一次性任務，最新功能最先落地 |
| Claude Desktop | 前端開發（搭配 Preview 視窗）；非技術用戶 |
| Mobile | 移動中啟動任務，無需開筆電 |
| CoWork | 非程式碼產出（簡報、文件、郵件、客戶準備） |

### CoWork 實戰案例（Cat 本人）
- 準備「Code with Claude」大會演講
- 連接 Slack + Google Drive，輸入 PMM 建議要點
- CoWork 工作 1 小時：爬 Twitter、查 Evergreen 發布室、查公告頻道
- 輸出 20 頁設計水準的簡報草稿
- Cat 只需一輪反饋（文字太長）

### 100% 自動化原則
- 95% 準確率 ≠ 自動化（仍需人工審核）
- 100% 才能「真正釋放時間」
- Cat 自己也在 Gmail 收件箱清理上卡在 95%，積極改進中

### Anthropic 使命如何驅動組織
- 使命置於任何個別產品之上：「Claude Code 失敗但 Anthropic 成功，我會很高興」
- 使命簡化優先級權衡：兩個競爭目標 → 哪個對使命更重要？
- 使命讓跨組織決策透明且統一執行

### 模型升級如何改變產品
- 舊模型需要提示強迫：「你把待辦事項清單上的所有項目都完成了嗎？」
- Opus 4 以後：自然完成，無需提醒 → 相關功能可弱化
- 新模型解鎖新功能：Code Review（多個 Agent 並行遍歷代碼庫）

### 未來願景：多 Agent 協作
- 現在：單任務成功（核心構建模組）
- 近期：多任務並行（同時 6 個 Claude）
- 未來：數百個 Claude 同時運行 → 需要遠端執行基礎設施 + 人類監控界面 + 自我優化機制

---

## Entities Mentioned

- [[Cat Wu]]（Anthropic 產品負責人，本文主角）
- [[Anthropic Academy]]
- Boris（Claude Code 技術負責人，即 [[Erik Schluntz]] 同期的技術主導）
- Amanda（負責塑造 Claude 個性的核心人員）
- Alex（PMM 負責人）
- Waymo（Cat 個人最喜愛的產品）

---

## Concepts Mentioned

- [[Anthropic 日級迭代發布機制]]
- [[AI時代PM新角色]]
- [[100%自動化原則與AI杠桿]]
- [[Claude Agent 設計模式]]（多 Agent 願景）
- [[生產環境Vibe Coding四大策略]]（對應）
- [[Superpowers技能框架]]（Skills / Evals）
- [[Claude MCP 伺服器整合指南]]（CoWork MCP 連接）

---

## Contradictions/Tensions

- **PRD 仍然存在**：Cat 說 PRD「偶爾仍然有用」，但強調它不是主要工作模式——與「PM 角色完全改變」的敘述有些微張力
- **定制化的雙面性**：Cat 一方面推薦 Skills/MCP 定制，另一方面警告「過度定制會分散核心目標」
- **95% vs 100% 自動化**：Cat 承認自己在 Gmail 上卡在 95%，意味著即使是 Anthropic 內部人員也面臨同樣困境

---

## Questions Raised

- CoWork 的 Skills 定義（SKILL.md）和 Claude Code 的 Skills 系統是同一套機制嗎？
- Applied AI 團隊在使用 Token 上有沒有量化數據？
- Anthropic 內部有多少「自行建造的辦公工具」正在使用？
- 「研究預覽版」機制有沒有明確的升級標準（什麼時候從 Preview 轉為正式功能）？
