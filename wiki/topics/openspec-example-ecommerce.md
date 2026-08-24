---
title: "OpenSpec 實戰：電商網站（商品 + 購物車 + 結帳）"
type: topic
tags: [openspec, spec-driven-development, ecommerce, example, tutorial]
created: 2026-04-29
updated: 2026-04-29
---

# OpenSpec 實戰：電商網站（商品 + 購物車 + 結帳）

> 這是一個完整的 OpenSpec 實戰範例，展示如何從「我想做一個電商網站」到可以交給 AI 執行的規格文件。
> 適合直接照抄修改。

---

## 背景假設

- **產品**：手作皮革商品的品牌電商
- **規模**：MVP，預計月訂單 < 500 筆
- **技術能力**：能理解技術選型，但請 AI 實作
- **開發時長預估**：8–12 小時（含測試）

---

## Explore 階段對話紀錄

**你的需求描述（給 AI 的初始提示）**：
> 我要做一個賣手作皮革商品的電商網站。商品大概 20–50 種，顧客可以瀏覽、加購物車、結帳。我自己管理庫存和訂單。

**AI 問了 5 個關鍵問題，你的回答**：

1. **金流怎麼處理？** → 先用 Stripe，台灣之後再加綠界
2. **有沒有會員系統？** → 要，要記住購買記錄和收件地址
3. **庫存管理多複雜？** → 每個商品有庫存數量，賣完要顯示「已售完」
4. **需要促銷功能嗎？** → MVP 不需要折扣碼，之後再加
5. **商品有沒有規格（顏色/尺寸）？** → 有，比如皮帶有長度選擇（S/M/L）

---

## 📄 proposal.md

```markdown
# Proposal：手作皮革電商 MVP

## 問題陳述
手作皮革品牌目前透過 Instagram DM 接單，流程不透明、容易漏單，
客人無法自助查詢訂單狀態。需要一個自有電商平台取代 DM 接單流程。

## 目標
- [ ] 顧客能在 5 分鐘內完成從選商品到付款的完整流程
- [ ] 店主能在後台管理訂單狀態（待出貨 / 已出貨 / 完成）
- [ ] 庫存不足時自動阻擋下單

## 範圍（In Scope）

### 顧客端
- 商品列表（含分類篩選、規格選擇）
- 商品詳細頁（圖片、描述、庫存狀態）
- 購物車（新增/修改/刪除商品）
- 結帳流程（填收件資料 → 付款 → 確認頁）
- 會員系統（註冊/登入/訂單記錄/收件地址管理）

### 店主端（後台）
- 訂單列表（含搜尋/篩選）
- 訂單狀態更新（待出貨 → 已出貨 → 完成）
- 商品管理（新增/編輯/下架）
- 庫存調整

## 範圍外（Out of Scope）
- 折扣碼 / 促銷系統（原因：MVP 先驗證購買流程，促銷影響數據難以分析）
- 綠界金流（原因：先用 Stripe 快速上線，台灣金流之後獨立迭代）
- 商品評論（原因：評論需要信任管理機制，複雜度高於價值）
- 推薦系統（原因：商品數 < 50，人工策展比演算法更有效）

## 成功標準
1. 完整的購買流程（選商品 → 加購物車 → 結帳 → 付款成功）可以跑通
2. 庫存歸零時下單被阻擋（不能超賣）
3. 店主可以更新訂單狀態

## 假設與限制
- 假設：同時在線用戶 < 100 人（MVP 規模）
- 假設：每筆訂單只用一種付款方式（不分期）
- 限制：預算有限，部署選雲端免費方案（Vercel + Supabase）
```

---

## 📄 design.md

```markdown
# Design：手作皮革電商 MVP

## 技術選型

| 面向 | 選擇 | 備選方案 | 決策理由 |
|------|------|---------|---------|
| 前端框架 | Next.js 14（App Router） | Remix / Nuxt | SSR 對 SEO 重要（商品頁需要被 Google 索引） |
| 資料庫 | Supabase（PostgreSQL） | PlanetScale / Railway | 有免費方案 + 內建 Auth + RLS |
| Auth | Supabase Auth | NextAuth / Clerk | 與資料庫同平台，減少整合複雜度 |
| 金流 | Stripe | 綠界 | Stripe 文件最完整，本地開發 webhook 容易 |
| 圖片儲存 | Supabase Storage | Cloudinary | 與資料庫同平台，免費方案足夠 |
| 部署 | Vercel | Netlify | Next.js 官方推薦 |
| UI 元件 | shadcn/ui + Tailwind | MUI / Chakra | 可客製化，不引入不必要的 bundle |

## 架構圖

```
顧客瀏覽器
    │
    ▼
Next.js（Vercel）
    ├─ App Router（SSR 商品頁 / CSR 購物車）
    ├─ API Routes（/api/*）
    │       ├─ Supabase Client（資料存取）
    │       └─ Stripe SDK（金流）
    │
Supabase（PostgreSQL + Auth + Storage）
    ├─ products 表
    ├─ variants 表（商品規格）
    ├─ orders 表
    ├─ order_items 表
    ├─ users / addresses 表
    └─ Storage（商品圖片）
```

## 資料模型

### products
| 欄位 | 型別 | 說明 |
|------|------|------|
| id | UUID | Primary Key |
| name | text | 商品名稱 |
| description | text | 商品描述 |
| price | integer | 單價（新台幣分，避免浮點）|
| category | text | 分類（belt / wallet / bag）|
| images | text[] | Supabase Storage 路徑陣列 |
| is_active | boolean | 是否上架 |
| created_at | timestamptz | |

### variants（商品規格）
| 欄位 | 型別 | 說明 |
|------|------|------|
| id | UUID | |
| product_id | UUID | FK → products |
| name | text | 規格名稱，如「S / M / L」|
| stock | integer | 庫存數量 |
| sku | text | 庫存管理碼（可選）|

### orders
| 欄位 | 型別 | 說明 |
|------|------|------|
| id | UUID | |
| user_id | UUID | FK → auth.users |
| status | text | pending / paid / shipped / completed / cancelled |
| total_amount | integer | 總金額（分）|
| stripe_payment_intent_id | text | 用於對帳 |
| shipping_address | jsonb | 快照收件地址（不 FK，保留歷史）|
| created_at | timestamptz | |

### order_items
| 欄位 | 型別 | 說明 |
|------|------|------|
| id | UUID | |
| order_id | UUID | FK → orders |
| variant_id | UUID | FK → variants |
| quantity | integer | |
| unit_price | integer | 下單當下的快照價格（不 FK price） |

## 關鍵決策記錄（ADR）

### ADR-001：金額用「分」而非「元」
- **決策**：所有金額欄位儲存最小單位（新台幣分），前端顯示時 /100
- **原因**：避免浮點數精度問題（0.1 + 0.2 ≠ 0.3）；Stripe 本身也用最小單位
- **代價**：前端需要統一轉換顯示

### ADR-002：shipping_address 用 jsonb 快照而非 FK
- **決策**：order 建立時把收件地址複製為 JSON 存入 orders
- **原因**：用戶之後修改地址不應影響歷史訂單
- **代價**：地址修改不會回溯到舊訂單（這是預期行為）

### ADR-003：庫存扣減在資料庫層用 check constraint
- **決策**：variants.stock 加上 CHECK (stock >= 0)，在 DB 層阻止超賣
- **原因**：應用層的檢查在並發時有 race condition 風險
- **代價**：下單 API 需要 catch UNIQUE/CHECK violation 並回傳友善錯誤訊息

### ADR-004：購物車存在 localStorage 而非資料庫
- **決策**：未登入用戶的購物車存在瀏覽器 localStorage
- **原因**：減少資料庫操作；登入後 merge 到後端（或讓 Zustand 管理）
- **代價**：換裝置購物車消失（MVP 可接受）
```

---

## 📄 behavioral-specs.md（節錄重要場景）

```markdown
# Behavioral Specs：手作皮革電商 MVP

---

## 功能：商品列表

### SPEC-P01：瀏覽商品列表
Given 任何訪客（未登入或已登入）進入首頁
When  頁面載入完成
Then  顯示所有 is_active = true 的商品，每個商品顯示名稱、主圖、價格

### SPEC-P02：依分類篩選
Given 商品列表頁有顯示分類篩選標籤
When  用戶點擊「皮帶」分類
Then  只顯示 category = 'belt' 的商品，URL 更新為 ?category=belt

### SPEC-P03：已售完商品顯示
Given 某商品的所有 variants 的 stock 均為 0
When  商品列表頁載入
Then  該商品卡片顯示「已售完」標籤，商品仍可點入查看（但無法加入購物車）

---

## 功能：購物車

### SPEC-C01：加入購物車（未登入）
Given 訪客瀏覽商品頁，選擇規格「M」，庫存 > 0
When  點擊「加入購物車」
Then  購物車圖示數字 +1，localStorage 中新增此商品記錄

### SPEC-C02：加入超過庫存數量
Given 購物車已有某 variant 3 件，該 variant 實際庫存只有 3 件
When  用戶在商品頁再點一次「加入購物車」
Then  顯示提示「已達庫存上限」，購物車數量不變

### SPEC-C03：修改購物車數量
Given 購物車頁面，某項商品數量顯示為 2
When  用戶點擊「+」按鈕
Then  數量變為 3，小計重新計算，若庫存不足 3 則無法點擊「+」

### SPEC-C04：刪除商品
Given 購物車有 2 項商品
When  用戶點擊某商品的「刪除」按鈕
Then  該商品從購物車移除，購物車只剩 1 項，總金額重新計算

---

## 功能：結帳

### SPEC-O01：結帳前登入驗證
Given 訪客購物車有商品，點擊「前往結帳」
When  用戶未登入
Then  導向登入頁面，登入完成後返回結帳頁（購物車保留）

### SPEC-O02：填寫收件資料
Given 已登入用戶在結帳頁
When  用戶選擇已儲存的地址或填寫新地址
Then  表單顯示正確，「下一步」按鈕可點擊

### SPEC-O03：結帳時庫存不足
Given 用戶購物車有商品 A（數量 2），但在用戶結帳過程中商品 A 被其他人買走剩 1 件
When  用戶點擊「確認下單」
Then  顯示錯誤「商品 [A] 庫存不足，請調整數量」，訂單不建立，回到購物車

### SPEC-O04：Stripe 付款成功
Given 用戶填完收件資料，點擊「前往付款」
When  Stripe 付款成功，webhook 回傳 payment_intent.succeeded
Then
  - orders.status 更新為 'paid'
  - 對應 variants.stock 各減去購買數量
  - 用戶收到確認信（含訂單編號）
  - 購物車清空
  - 導向訂單確認頁

### SPEC-O05：Stripe 付款失敗
Given 用戶在 Stripe Checkout 頁面
When  付款失敗（如卡片餘額不足）
Then  Stripe 顯示失敗訊息，用戶可以重試，訂單保持 pending 狀態，庫存不扣減

---

## 功能：後台訂單管理

### SPEC-A01：查看訂單列表
Given 店主登入後台
When  進入訂單管理頁
Then  顯示所有訂單，預設按建立時間倒序，顯示訂單編號/顧客/金額/狀態

### SPEC-A02：更新訂單狀態
Given 後台訂單列表，某訂單狀態為 paid
When  店主點擊「標記已出貨」
Then  訂單狀態更新為 shipped，記錄更新時間

### SPEC-A03：狀態不可逆限制
Given 某訂單狀態為 shipped
When  店主試圖將狀態改回 paid
Then  後台不顯示此選項（狀態轉換遵循單向流：paid → shipped → completed）
```

---

## 📄 task-checklist.md

```markdown
# Task Checklist：手作皮革電商 MVP

## Phase 1：基礎建設（預計 1.5h）
- [ ] T01：初始化 Next.js 14 + shadcn/ui + Tailwind，設定 ESLint/Prettier
- [ ] T02：設定 Supabase 專案，建立資料表（依 design.md schema）
- [ ] T03：設定 Supabase Auth（email/password），建立 middleware 保護 /admin/* 路由
- [ ] T04：設定 Supabase Storage bucket（products-images，public 讀取）

## Phase 2：商品功能（預計 2h）
- [ ] T05：商品列表頁（/products）——SSR，含分類篩選，符合 SPEC-P01/P02
- [ ] T06：商品詳細頁（/products/[id]）——SSR，含規格選擇下拉選單
- [ ] T07：商品「已售完」狀態顯示（SPEC-P03）
- [ ] T08：後台商品管理 CRUD（/admin/products）

## Phase 3：購物車（預計 1.5h）
- [ ] T09：購物車 Zustand store（含 localStorage 持久化）
- [ ] T10：加入購物車功能（含庫存上限檢查，SPEC-C01/C02）
- [ ] T11：購物車頁（/cart）——修改數量、刪除、小計計算（SPEC-C03/C04）

## Phase 4：結帳與金流（預計 3h）
- [ ] T12：結帳頁（/checkout）——收件地址表單，未登入導向登入（SPEC-O01/O02）
- [ ] T13：POST /api/orders——建立 pending 訂單，含庫存並發檢查（SPEC-O03）
- [ ] T14：整合 Stripe Checkout——建立 payment intent，導向 Stripe 付款頁
- [ ] T15：Stripe Webhook（POST /api/webhooks/stripe）——處理 payment_intent.succeeded/failed（SPEC-O04/O05）

## Phase 5：後台訂單管理（預計 1h）
- [ ] T16：後台訂單列表（/admin/orders）——分頁、篩選（SPEC-A01）
- [ ] T17：訂單狀態更新 API + UI（SPEC-A02/A03）

## Phase 6：會員功能（預計 1h）
- [ ] T18：會員訂單記錄頁（/account/orders）
- [ ] T19：收件地址管理（/account/addresses）

---

## 驗收清單
- [ ] 完整購買流程 E2E 測試（Stripe test card 4242...）
- [ ] 庫存超賣保護測試（並發下單同一 variant）
- [ ] 手機版 RWD 確認（重點：商品頁、購物車、結帳）
```

---

## Archive 範本（完成後填寫）

```markdown
# Archive：手作皮革電商 MVP v1.0

## 完成日期：[填入]
## 實際開發時長：[填入]（預估 8–12h）

## 完成了什麼
手作皮革品牌電商 MVP 上線，涵蓋商品瀏覽、購物車、Stripe 結帳、
後台訂單管理。顧客可以自助完成從選商品到付款的完整流程。

## 與原設計的差異
- [記錄任何實作中發現需要調整的地方]

## 關鍵技術決策（實際確認）
- 庫存鎖定：使用 PostgreSQL 的 `UPDATE variants SET stock = stock - $1 WHERE id = $2 AND stock >= $1` 配合 RETURNING，在一個 atomic 操作中扣減並確認成功
- Stripe Webhook 冪等性：用 stripe_payment_intent_id 做去重，避免 webhook 重送時重複扣庫存

## 未完成 / Phase 2 計畫
- 折扣碼系統
- 綠界金流整合
- 訂單出貨追蹤號輸入
- Email 確認信（Resend API）
```

---

## 相關頁面

- [[openspec-product-design-guide|OpenSpec 產品設計教學]] — 完整教學，包含每個步驟的說明
- [[openspec-example-saas|範例二：SaaS 訂閱制後台]] — 另一個實戰範例
- [[OpenSpec工作流]] — 8 階段流程概念介紹
- [[Spec驅動開發]] — 方法論背景
