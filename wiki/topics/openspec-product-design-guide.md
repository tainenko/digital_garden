---
title: 用 OpenSpec 做產品設計：從想法到可執行規格的完整教學
type: topic
tags: [openspec, spec-driven-development, product-design, ai-coding, tutorial]
created: 2026-04-29
updated: 2026-05-26
sources: [openspec-superpowers-multi-source, kaochenlong-openspec]
---

# 用 OpenSpec 做產品設計：從想法到可執行規格的完整教學

> **適合對象**：想用 AI 開發產品，但不確定「說清楚要做什麼」的人。不需要會寫程式，只需要能描述自己想要什麼。

---

## 為什麼需要 OpenSpec？

Vibe Coding（對 AI 說「幫我做一個電商網站」）的最大問題不是 AI 不聰明，而是**你和 AI 對「電商網站」的定義根本不一樣**。

AI 聽到「電商網站」可能做出一個有商品列表的靜態頁面；你想的是有購物車、金流、庫存管理的完整系統。

OpenSpec 強迫你在 AI 動手之前，把三件事說清楚：
1. **要做什麼**（proposal）
2. **怎麼判斷做對了**（behavioral specs）
3. **為什麼這樣選**（design 決策）

---

## 整體流程一覽

```
你的想法（模糊）
    ↓
[1] Explore   ← 和 AI 一起盤點現況、確認問題
    ↓
[2] Proposal  ← 寫清楚「要做什麼 / 不做什麼」
    ↓
[3] Design    ← 選技術、定架構、記錄原因
    ↓
[4] Specs     ← 寫行為規格（given/when/then）
    ↓
[5] Tasks     ← 拆成每個 AI 可執行的小任務
    ↓
[6] Apply     ← AI 逐任務實作
    ↓
[7] Verify    ← 確認做出來的符合 specs
    ↓
[8] Archive   ← 把這次的決策記錄下來（最常被跳過！）
```

---

## 專案目錄結構（`openspec init` 後）

```
openspec/
├── AGENTS.md       ← 告訴 AI 如何使用 OpenSpec（工作流文件）
├── project.md      ← 技術棧、架構慣例、約束（AI 的持久背景知識）
├── specs/          ← 當前系統規格的唯一真相源
└── changes/        ← 進行中的提案
    └── archive/    ← 已完成的變更（openspec archive 後移入）
```

`AGENTS.md` 和 `project.md` 是 AI 的「長期記憶」——每次請求都會自動注入，讓 AI 不需要每次重新猜測專案背景。

---

## 第一步：Explore（探索）

**目的**：先搞清楚自己真正要解決的問題，不要急著說「我要做什麼功能」。

### 怎麼做

對 AI 說：
> 「我想做一個 [產品類型]，幫助 [目標用戶] 解決 [問題]。幫我探索這個問題空間，問我 5 個關鍵問題來幫你理解需求。」

AI 會問你類似：
- 主要用戶是誰？他們現在怎麼處理這個問題？
- 最核心的一個功能是什麼？去掉它產品就不存在了？
- 第一版（MVP）需要哪些功能才算「可以上線」？
- 有哪些功能是「想要」但不是「必須」的？
- 有什麼不做的（明確排除）？

### Explore 的輸出

```markdown
## 問題確認
- 核心問題：[你想解決的根本問題]
- 目標用戶：[具體的人，不是「所有人」]
- 現有解法的缺點：[用戶現在怎麼做，哪裡不夠好]

## MVP 範圍
- 必做：[3–5 個功能]
- 之後再說：[明確推遲的功能]
- 不做：[明確排除的功能]
```

---

## 第二步：Proposal（提案）

**目的**：把 Explore 的結論整理成一份正式文件，讓 AI 和你都確認「我們在做同一件事」。

### proposal.md 模板

```markdown
# Proposal：[功能或產品名稱]

## 問題陳述
[1–2 句話說清楚要解決什麼問題，以及為什麼現在要解決它]

## 目標
- [ ] [可量化的目標 1]，例如「用戶能在 3 步內完成結帳」
- [ ] [可量化的目標 2]

## 範圍（In Scope）
### MVP 功能
- [功能 1]
- [功能 2]

## 範圍外（Out of Scope）
- [明確不做的功能]（原因：[為什麼不做]）

## 成功標準
完成後如何驗證這個 proposal 達成目標：
- [具體、可測試的標準]

## 假設與限制
- 假設：[這個設計建立在什麼假設上]
- 限制：[技術、資源、時間上的限制]
```

### 關鍵技巧：寫「不做什麼」和「為什麼」

最容易被忽略的是 **Out of Scope**。這是防止 AI 自作主張加功能的護欄。

❌ 不好的寫法：`不做推薦系統`

✅ 好的寫法：`不做個人化推薦（原因：MVP 階段用戶數不足以訓練模型，先用人工精選取代）`

---

## 第三步：Design（設計）

**目的**：記錄技術選型和架構決策，以及**為什麼這樣選**。

### design.md 結構

```markdown
# Design：[功能名稱]

## 技術選型

| 面向 | 選擇 | 備選方案 | 決策理由 |
|------|------|---------|---------|
| 前端框架 | Next.js | Vue / Nuxt | SSR 需求 + 團隊熟悉度 |
| 資料庫 | PostgreSQL | MySQL / MongoDB | 需要 transaction + JSON 欄位 |
| 快取 | Redis | Memcached | 需要 Sorted Set（排行榜） |
| 部署 | Vercel + Railway | AWS | 速度優先，成本次之 |

## 架構圖

[文字描述或 ASCII 圖]
用戶 → Next.js（Vercel）→ API Routes → PostgreSQL（Railway）
                                      → Redis（快取熱資料）

## 資料模型（關鍵）

### User
- id: UUID
- email: string (unique)
- created_at: timestamp

### [其他表格]

## API 設計（關鍵端點）

| 方法 | 路徑 | 說明 |
|------|------|------|
| POST | /api/auth/login | 登入 |
| GET  | /api/products | 商品列表 |

## 決策記錄（ADR）
每個非顯而易見的選擇都要記錄：

### ADR-001：為什麼不用 MongoDB？
- **考慮時間**：2026-04-29
- **決策**：使用 PostgreSQL
- **原因**：訂單與庫存需要嚴格的事務性保證（ACID），MongoDB 的最終一致性在金流場景不可接受
- **代價**：schema migration 較複雜
```

---

## 第四步：Behavioral Specs（行為規格）

**目的**：用「given/when/then」格式描述系統應該怎麼行為，讓 AI 有明確的實作目標，也讓你有明確的測試基準。

### 格式說明

```
Given [初始狀態]
When  [用戶動作 / 系統事件]
Then  [預期結果]
```

### 好的 Spec vs 壞的 Spec

❌ 壞的（描述實作）：
> `validateEmail() 應該回傳 true 或 false`

✅ 好的（描述行為）：
> ```
> Given 用戶在結帳頁面
> When  用戶輸入格式錯誤的 email（缺少 @）並按下「送出」
> Then  表單不送出，在 email 欄位下方顯示「請輸入有效的 email 格式」
> ```

### Spec 涵蓋的三類場景

1. **Happy Path**（主要流程）：用戶做對了，系統做對了
2. **Error Path**（錯誤處理）：用戶做錯了，系統怎麼回應
3. **Edge Case**（邊界條件）：極端情況（空值、超長輸入、並發）

---

## 第五步：Tasks（任務拆分）

**目的**：把 specs 拆成每個 AI 可以在一個 session 完成的小任務。

### 好的任務切分原則

- **每個任務一個關注點**：「建立 Product 資料表和 CRUD API」是一個任務；「建立 Product 資料表、CRUD API 和搜尋功能」是三個任務
- **有明確的完成標準**：任務完成後怎麼驗證？
- **依賴順序清楚**：資料庫 schema → API → 前端，不能亂序

### task-checklist.md 範例

```markdown
# Task Checklist

## Phase 1：基礎建設
- [ ] T01：初始化 Next.js 專案 + 設定 ESLint / Prettier
- [ ] T02：設定 PostgreSQL + Prisma ORM，建立 User / Product / Order schema
- [ ] T03：設定 Redis 連線，建立快取工具函數

## Phase 2：認證系統
- [ ] T04：實作 POST /api/auth/register（含密碼 hash）
- [ ] T05：實作 POST /api/auth/login（含 JWT 簽發）
- [ ] T06：建立 auth middleware（JWT 驗證）

## Phase 3：商品功能
- [ ] T07：實作 GET /api/products（含分頁、篩選）
- [ ] T08：實作 POST /api/products（管理員限定）
- [ ] T09：商品列表頁面（前端）

[以此類推...]

## 驗收標準
所有 behavioral specs 的 happy path 和 error path 通過手動測試。
```

---

## 第六步：Apply（實作）

這階段讓 AI 執行。給 AI 的提示方式：

```
參考 proposal.md 和 design.md 的架構決策，
實作 task-checklist.md 中的 T04（POST /api/auth/register）。

行為規格：
- [貼上相關的 given/when/then specs]

完成標準：
- 新用戶可以成功註冊
- 重複 email 回傳 409 Conflict
- 密碼用 bcrypt hash 儲存（salt rounds = 12）
```

### 搭配 Superpowers

如果同時使用 Superpowers，Apply 階段會強制執行：
1. 先寫失敗測試（RED）
2. 實作讓測試通過（GREEN）
3. 重構（REFACTOR）
4. 自動 code review

---

## 第七步：Verify（驗證）

根據 behavioral specs 逐條測試：

```markdown
## Verify Report：T04 POST /api/auth/register

### Happy Path
- [x] 新用戶送出有效資料 → 201 Created，回傳 user id 和 email
- [x] 密碼在資料庫中為 bcrypt hash，非明文

### Error Path
- [x] 重複 email → 409 Conflict，message: "Email already registered"
- [x] 缺少 email 欄位 → 400 Bad Request，message: "Email is required"
- [x] 密碼少於 8 字元 → 400 Bad Request，message: "Password must be at least 8 characters"

### Edge Case
- [x] email 超過 255 字元 → 400 Bad Request
- [ ] 並發 100 個相同 email 的請求 → 只有一個成功（資料庫 unique constraint 保護）← 待補充測試
```

---

## 第八步：Archive（歸檔，最重要！）

**不做 Archive = 下次換 AI 或換人，完全不知道為什麼這樣設計。**

Archive 的內容：

```markdown
# Archive：[功能名稱] v1.0

## 完成日期：2026-04-29
## 實作時長：8 小時

## 這次做了什麼
[1–2 句話概述]

## 關鍵決策（delta）
- 為什麼選 bcrypt 而非 argon2：bcrypt 在 Node.js 生態有更成熟的套件，且效能滿足需求
- 為什麼 salt rounds = 12：在目標主機上測試，rounds=12 hash 時間 ~250ms，平衡安全與 UX

## 未完成 / 之後要做的
- 並發重複註冊的整合測試（T04-Edge-001）
- email 驗證流程（設計為 Phase 2）

## 什麼沒有做（以及原因）
- 未實作 rate limiting on /register：決定在 API Gateway 層統一處理，不在個別 endpoint
```

---

## 何時適合用 OpenSpec？

| 任務規模 | 建議方式 |
|---------|---------|
| < 2 小時 | 直接 Vibe Coding，不需要 OpenSpec |
| 2–8 小時（個人） | OpenSpec Proposal + Tasks 即可（跳過正式 Design） |
| 4–16 小時（小團隊） | 完整 OpenSpec 8 階段 |
| > 16 小時 | 完整 OpenSpec + Superpowers |

### 何時不需要 Proposal（可直接改程式碼）

即使已採用 OpenSpec，以下情境**可繞過 SDD 工作流**，直接修改：

1. **Bug fix**：讓程式碼行為符合現有規格（規格是對的，程式碼錯了）
2. **錯字與格式調整**：文字修正、空白、縮排
3. **非破壞性依賴更新**：patch/minor 版本升級，不改變外部行為
4. **設定變更**：環境或配置調整，但不改變規格描述的行為
5. **補充測試**：為已記錄功能追加測試案例（不新增功能）

**核心判斷原則**：若這個改動不改變「系統應該做什麼」（specs 所描述的行為），就不需要 Proposal。

詳見 [[OpenSpec文件格式與驗證]]。

---

## 實戰範例

- [[openspec-example-ecommerce|範例一：電商網站（商品 + 購物車 + 結帳）]]
- [[openspec-example-saas|範例二：SaaS 訂閱制後台（訂閱管理 + 帳單 + 用量追蹤）]]

---

## 相關頁面

- [[Spec驅動開發]] — 方法論背景與三道牆
- [[OpenSpec工作流]] — 8 階段流程簡介
- [[Superpowers技能框架]] — Apply 階段的程式碼品質執行
- [[Vibe Coding風險與限制]] — OpenSpec 解決的核心問題
