---
title: Vibe Coding 完整指南
type: topic
tags: [vibe-coding, ai-coding, tools, risks, open-source, openspec, superpowers, sdd]
created: 2026-04-27
updated: 2026-04-28
sources: [abmedia-vibe-coding-complete-guide-2026, wikipedia-vibe-coding, simular-cursor-vibe-coding, infoq-ai-vibe-coding-open-source-crisis, airabbi-vibe-coding-intro, openspec-superpowers-multi-source]
---

# Vibe Coding 完整指南

> Vibe Coding = 用自然語言引導 AI 寫程式，開發者從「寫手」變「導演」。
> 2025 年 2 月由 [[Andrej Karpathy]] 提出，2026 年已有 92% 美國開發者採用某種形式的 AI 輔助編程。

---

## 一、什麼是 Vibe Coding？

詳見 [[Vibe Coding基礎概念]]。

核心工作流：
```
描述需求（自然語言）→ AI 生成程式碼 → 測試驗證 → 迭代修正
```

Karpathy 的原始定義強調「完全給 vibes」——不看 code diff，直接接受 AI 輸出，靠結果而非程式碼本身做判斷。

---

## 二、工具選擇

詳見 [[Vibe Coding工具比較]]。

**快速選擇指南**：

| 你的狀況 | 推薦工具 |
|---------|---------|
| 完全沒有程式背景，想快速驗證想法 | [[Bolt.new]] |
| 初學者，邊學邊做 | [[Replit]] |
| 有基礎，要最完整體驗 | [[Cursor]] |
| 進階工程師，偏好 terminal | Claude Code |
| 已用 VS Code，不想換環境 | GitHub Copilot |

---

## 三、風險與限制

詳見 [[Vibe Coding風險與限制]]。

三大核心風險：
1. **安全性**：AI 生成程式碼的安全漏洞機率是人類的 2.74 倍
2. **技術債**：AI 不考慮架構，快速累積難以維護的程式碼
3. **理解空洞**：依賴 AI 的開發者可能不理解自己的系統

**反直覺數據**：有經驗的開發者使用 AI 工具後反而慢了 19%（METR 2025 研究）。AI 在簡單任務快，複雜任務反而增加認知負擔。

---

## 四、對開源生態的衝擊

詳見 [[Vibe Coding開源生態衝擊]]。

Vibe Coding 正在破壞開源的商業可持續性：
- 開發者委託 AI 選套件 → 停止閱讀文件 → 維護者失去收入
- 大量低品質 AI 貢獻湧入 → 維護者不堪負荷（「AI Slopageddon」）
- cURL、Ghostty、tldraw 已採取緊急措施

---

## 五、進階：Spec 驅動開發（SDD）

純 Vibe Coding 面臨三道牆：需求錯位（Wall 1）、缺工程紀律（Wall 2）、設計理由消失（Wall 3）。[[Spec驅動開發]] 框架是對 Vibe Coding 的「成熟化」：

| 框架 | 解決 | 核心指令 |
|------|------|---------|
| [[OpenSpec]] | Wall 1 + Wall 3 | `propose` → `apply` → `archive` |
| [[Superpowers]] | Wall 2 | 自動 TDD + code review |

**疊加建議**：

| 任務規模 | 推薦組合 |
|---------|---------|
| < 2h 原型 | Claude Code 單獨 |
| 2–8h 個人功能 | Claude Code + Superpowers |
| 4–16h 團隊 | Claude Code + OpenSpec + Superpowers |

實際案例：Next.js + PostgreSQL 部落格，8 小時完成，87% 測試覆蓋率，第一週零 bug。

---

## 六、誰應該用 Vibe Coding？

### 適合立即開始
- **想快速驗證 app 概念的非技術創辦人**
- **想自動化日常工作的一般職場工作者**（報表、排程、資料處理）
- **想加速原型開發的設計師/產品經理**

### 需要謹慎的情境
- 處理個人資料、金融交易、認證系統
- 需要長期維護的核心產品功能
- 多人協作的大型專案架構

### 核心心態
不論技術背景，Vibe Coding 有效的前提是：
> 把 AI 輸出當作**起點**，不是答案。批判性驗證 > 盲目信任。

---

## 七、延伸閱讀

### 概念頁面
- [[Vibe Coding基礎概念]]
- [[Vibe Coding工具比較]]
- [[Vibe Coding風險與限制]]
- [[Vibe Coding開源生態衝擊]]
- [[Spec驅動開發]] — 成熟化的 Vibe Coding 方法論
- [[OpenSpec工作流]] — 8 階段 SDD 流程
- [[Superpowers技能框架]] — TDD + code review 自動化

### 實體頁面
- [[Andrej Karpathy]] — 概念提出者
- [[Cursor]] — 最主流的 Vibe Coding IDE
- [[Bolt.new]] — 非技術用戶首選
- [[Replit]] — 初學者雲端 IDE
- [[OpenSpec]] — Spec 驅動框架（Fission-AI）
- [[Superpowers]] — TDD 執行框架（obra）

### 來源頁面
- [[abmedia-vibe-coding-complete-guide-2026|Vibe Coding 2026 完整指南（ABMedia）]]
- [[wikipedia-vibe-coding|Vibe Coding（Wikipedia）]]
- [[simular-cursor-vibe-coding|Cursor AI Vibe Coding 教學（夏木樂）]]
- [[infoq-ai-vibe-coding-open-source-crisis|AI Vibe Coding 威脅開源生態（InfoQ）]]
- [[airabbi-vibe-coding-intro|Vibe Coding 跟一般人的關係（瑞比智慧）]]
- [[openspec-superpowers-multi-source|OpenSpec + Superpowers Spec 驅動開發指南]]
