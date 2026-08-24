---
title: "OpenSpec 實戰：SaaS 訂閱制後台（訂閱管理 + 帳單 + 用量追蹤）"
type: topic
tags: [openspec, spec-driven-development, saas, subscription, example, tutorial]
created: 2026-04-29
updated: 2026-04-29
---

# OpenSpec 實戰：SaaS 訂閱制後台（訂閱管理 + 帳單 + 用量追蹤）

> 這個範例展示 OpenSpec 在**功能比電商更複雜**的場景下的應用：訂閱制有多種方案、計費週期、用量限制，需要更嚴謹的規格設計才不會出錯。

---

## 背景假設

- **產品**：一個 AI 文案產生器 SaaS，用 API 呼叫次數計費
- **規模**：早期，< 500 個付費用戶
- **這次要做的功能**：用戶能看到訂閱方案、訂閱/升級/降級、看帳單記錄、查用量

---

## Explore 結論摘要

和 AI 討論後確認的核心問題：

- **方案結構**：Free（100 calls/月）、Pro（5,000 calls/月，$29/月）、Team（無限，$99/月）
- **計費方式**：Stripe Billing，月付，不支援年付（MVP）
- **用量追蹤**：每次 API 呼叫記一筆，月底重置
- **升降級**：升級立即生效；降級等到當期結束才生效
- **核心限制**：Free 方案超過用量直接返回 429，不能超用

---

## 📄 proposal.md

```markdown
# Proposal：SaaS 訂閱與用量管理系統

## 問題陳述
用戶目前看不到自己的用量和剩餘次數，也無法自助升級方案，
需要寫信才能升級，流程慢且容易流失潛在付費用戶。

## 目標
- [ ] 用戶能看到當月用量和剩餘次數（即時更新，延遲 < 1 分鐘）
- [ ] 用戶能自助完成訂閱/升級（不需要聯絡支援）
- [ ] 超出用量時 API 自動返回 429，不允許超用
- [ ] 降級不影響當期已付費的服務

## 範圍（In Scope）
- 訂閱方案頁（列出三種方案，標示當前方案）
- 訂閱/升級流程（Stripe Billing 整合）
- 降級申請（等下期生效）
- 用量 Dashboard（當月使用量、剩餘量、歷史趨勢）
- 帳單記錄頁（歷史發票列表，可下載 PDF）
- API 用量限制中間件（超過則 429）

## 範圍外（Out of Scope）
- 年付方案（原因：MVP 先驗證月付轉換率）
- 用量超額購買（原因：超額設計複雜，先強制升級）
- 團隊多成員管理（原因：Team 方案 MVP 先給單一帳號，多座位之後再加）
- 客製企業方案（原因：需要手動報價流程，MVP 先跑標準方案）

## 成功標準
1. Free 用戶超過 100 calls/月，第 101 次呼叫收到 429
2. Pro 用戶可以在後台自助升級為 Team，Stripe 立即收費
3. 用戶看到的用量數字在 API 呼叫 60 秒內更新

## 假設與限制
- 假設：用量統計可以容忍 < 1 分鐘的最終一致性（不需要 real-time 強一致）
- 假設：每個用戶一個訂閱，不支援多 workspace（MVP）
- 限制：Stripe 是唯一支援的金流
```

---

## 📄 design.md

```markdown
# Design：SaaS 訂閱與用量管理系統

## 技術選型

| 面向 | 選擇 | 備選 | 理由 |
|------|------|------|------|
| 訂閱計費 | Stripe Billing | Paddle / LemonSqueezy | 文件最完整，webhook 生態成熟 |
| 用量計數 | Redis INCR + PostgreSQL 月結 | 純 PostgreSQL | Redis 原子操作防 race condition；PG 保存歷史 |
| 後台框架 | Next.js（與主產品同框架） | — | 共用框架減少維護負擔 |
| 前端 UI | shadcn/ui + Recharts（圖表） | — | 輕量，圖表功能足夠 |

## 資料模型

### subscriptions（訂閱記錄）
| 欄位 | 型別 | 說明 |
|------|------|------|
| id | UUID | |
| user_id | UUID | FK → users（unique，一人一訂閱）|
| plan | text | free / pro / team |
| status | text | active / past_due / cancelled |
| stripe_subscription_id | text | Stripe 訂閱 ID |
| stripe_customer_id | text | Stripe 客戶 ID |
| current_period_start | timestamptz | 當期開始 |
| current_period_end | timestamptz | 當期結束 |
| cancel_at_period_end | boolean | 是否已申請降級 |
| pending_plan | text | 下期生效的方案（降級用）|

### usage_logs（用量記錄）
| 欄位 | 型別 | 說明 |
|------|------|------|
| id | UUID | |
| user_id | UUID | |
| endpoint | text | 呼叫的 API 端點 |
| called_at | timestamptz | |
| period_month | text | 'YYYY-MM'，方便查詢 |

### invoices（帳單，從 Stripe Webhook 同步）
| 欄位 | 型別 | 說明 |
|------|------|------|
| id | UUID | |
| user_id | UUID | |
| stripe_invoice_id | text | unique |
| amount | integer | 金額（美分）|
| status | text | paid / open / void |
| invoice_pdf_url | text | Stripe 產生的 PDF 連結 |
| period_start | timestamptz | |
| period_end | timestamptz | |
| created_at | timestamptz | |

## 關鍵架構：用量計數

```
API 呼叫
    │
    ▼
用量限制 Middleware
    ├─ 1. GET Redis key: usage:{user_id}:{YYYY-MM}
    ├─ 2. 若 key 不存在 → 從 PostgreSQL 查本月用量回填 Redis（設 TTL 1小時）
    ├─ 3. 與方案上限比較
    │       ├─ 超過 → 回傳 429 Too Many Requests
    │       └─ 未超過 → 放行
    └─ 4. 放行後：非同步 INCR Redis key + INSERT usage_logs（異步，不阻塞請求）
```

為什麼 Redis 在前，PostgreSQL 在後？
- Redis INCR 是原子操作，防止並發超用
- PostgreSQL 寫入非同步，P99 不受影響
- Redis key TTL = 當月剩餘天數，月底自動過期

## 用量重置機制

每月 1 號 00:00 UTC，Stripe 的 `invoice.paid` webhook 觸發：
1. 確認下期帳單已付（plan 繼續）
2. 刪除 Redis key `usage:{user_id}:{last_month}`（或讓 TTL 自然過期）

## 升降級流程

**升級（立即生效）**：
```
用戶點「升級為 Pro」
    → POST /api/subscription/upgrade
    → Stripe API: subscription.items.update（proration 立即計費）
    → Webhook: customer.subscription.updated
    → 更新 subscriptions.plan
    → 更新 Redis 中的 plan 設定（讓新的上限立即生效）
```

**降級（下期生效）**：
```
用戶點「降級為 Free」
    → POST /api/subscription/downgrade
    → Stripe API: subscription.update(cancel_at_period_end=true)
    → 更新 subscriptions.cancel_at_period_end = true, pending_plan = 'free'
    → 頁面顯示「已申請降級，將於 [日期] 生效」
    → 當期結束 Webhook: customer.subscription.deleted 或 updated
    → 更新 subscriptions.plan = pending_plan
```

## ADR 記錄

### ADR-001：升級用 Stripe proration，不自己計算
- **決策**：升級時讓 Stripe 計算費用差額（proration）
- **原因**：自己計算容易出錯，尤其在月中升級、有折扣的情況；Stripe 已處理所有邊界情況
- **代價**：依賴 Stripe 的 proration 邏輯，需要測試 Stripe test mode

### ADR-002：用量計數用 Redis INCR 而非 SELECT + UPDATE
- **決策**：用 `INCR usage:{user_id}:{month}` 計數
- **原因**：PostgreSQL 的 UPDATE 在高並發時有鎖競爭；INCR 是 Redis 的原子操作，天然防 race condition
- **代價**：Redis 資料是 ephemeral，需要 PostgreSQL 作為持久備份（月底歸檔）

### ADR-003：降級不立即生效
- **決策**：降級設定 `cancel_at_period_end = true`，等當期結束才切換方案
- **原因**：用戶已付費當期，立即降級不公平且容易引發退款爭議
- **代價**：需要在 UI 清楚顯示「生效日期」，避免用戶困惑
```

---

## 📄 behavioral-specs.md（節錄）

```markdown
# Behavioral Specs：SaaS 訂閱與用量管理

---

## 功能：用量限制

### SPEC-U01：Free 方案超用
Given 用戶為 Free 方案，本月已呼叫 API 100 次
When  用戶發出第 101 次 API 呼叫
Then
  - HTTP 回傳 429 Too Many Requests
  - Body: {"error": "usage_limit_exceeded", "message": "Monthly limit reached. Upgrade to Pro for more.", "upgrade_url": "/settings/billing"}
  - 本次呼叫不記入 usage_logs

### SPEC-U02：Pro 方案未超用
Given 用戶為 Pro 方案（上限 5,000），本月已呼叫 4,999 次
When  用戶發出第 5,000 次呼叫
Then  API 正常回應，usage 計為 5,000

### SPEC-U03：用量即時顯示
Given 用戶查看 Dashboard 的用量儀表板
When  用戶進行一次 API 呼叫後，在 60 秒內重新整理頁面
Then  Dashboard 顯示的本月用量 +1

### SPEC-U04：月底用量重置
Given Pro 用戶本月已使用 4,800 次，今天是訂閱週期最後一天
When  訂閱週期翻新（Stripe `invoice.paid` webhook 到達）
Then  用量計數歸零，新的週期用量從 0 開始

---

## 功能：訂閱升級

### SPEC-S01：查看方案頁
Given 任何已登入用戶進入 /settings/billing
When  頁面載入
Then
  - 顯示三個方案（Free / Pro / Team）的功能和價格
  - 當前方案有「目前方案」標籤
  - 更高方案顯示「升級」按鈕，更低方案顯示「降級」

### SPEC-S02：Free 升級 Pro
Given Free 用戶在方案頁點擊「升級為 Pro（$29/月）」
When  進入 Stripe Checkout，填入信用卡資料並完成付款
Then
  - Stripe 立即向信用卡收取 $29（或按比例的當月費用）
  - 用戶方案立即切換為 Pro（新的 API 呼叫上限 5,000）
  - 頁面顯示「已成功升級為 Pro」
  - 帳單頁新增一筆記錄

### SPEC-S03：Pro 升級 Team（立即生效）
Given Pro 用戶點擊「升級為 Team」，已有 Stripe 付款方式
When  確認升級
Then
  - Stripe 立即收取差額（按比例）
  - 方案立即切換為 Team（API 呼叫無限制）
  - 無需重新填信用卡（使用已儲存的付款方式）

### SPEC-S04：降級申請（下期生效）
Given Pro 用戶點擊「降級為 Free」
When  在確認對話框中點擊「確認降級」
Then
  - 畫面顯示「已申請降級，將於 [current_period_end 日期] 生效」
  - 在此日期之前，方案仍為 Pro，API 上限不變
  - 帳單頁顯示「降級申請中」狀態

### SPEC-S05：降級生效
Given 用戶已申請降級，今天是 current_period_end
When  Stripe `customer.subscription.updated` webhook 到達，plan = free
Then
  - 資料庫 subscriptions.plan 更新為 free
  - 用戶的 API 上限切換為 100 calls/月
  - 當月用量重置（新週期開始）

---

## 功能：帳單記錄

### SPEC-B01：查看帳單列表
Given 付費用戶進入帳單頁
When  頁面載入
Then  顯示所有歷史發票，含日期、金額、狀態（paid/open）、下載連結

### SPEC-B02：下載發票 PDF
Given 帳單列表有一筆 status = paid 的發票
When  用戶點擊「下載」
Then  在新分頁開啟 Stripe 提供的 invoice PDF URL
```

---

## 📄 task-checklist.md

```markdown
# Task Checklist：SaaS 訂閱與用量管理

## Phase 1：資料庫與基礎設定（1h）
- [ ] T01：建立 subscriptions / usage_logs / invoices 資料表（依 design.md）
- [ ] T02：設定 Stripe SDK（server-side），設定 webhook secret
- [ ] T03：設定 Redis 連線（用於用量計數）

## Phase 2：用量限制中間件（2h）
- [ ] T04：實作 checkUsageLimit() 函數（Redis INCR + 方案上限比較，SPEC-U01/U02）
- [ ] T05：實作 recordUsage() 函數（非同步 INCR Redis + INSERT usage_logs）
- [ ] T06：將中間件掛上 API 路由（所有需要計費的 endpoint）
- [ ] T07：寫 unit test：Free 超用返回 429 / Pro 未超用放行

## Phase 3：Stripe 整合（3h）
- [ ] T08：POST /api/subscription/create-checkout（產生 Stripe Checkout Session）
- [ ] T09：POST /api/webhooks/stripe——處理以下事件：
  - customer.subscription.updated（升降級）
  - customer.subscription.deleted（降級完成）
  - invoice.paid（帳單付款成功 + 用量重置，SPEC-U04）
  - invoice.payment_failed（付款失敗，status → past_due）
- [ ] T10：POST /api/subscription/downgrade（設定 cancel_at_period_end，SPEC-S04）
- [ ] T11：Webhook 冪等性處理（用 stripe_invoice_id 去重）

## Phase 4：用戶 UI（2h）
- [ ] T12：/settings/billing 方案頁（顯示三方案 + 當前方案標籤，SPEC-S01）
- [ ] T13：升級流程 UI（點擊 → Stripe Checkout → 成功跳回，SPEC-S02/S03）
- [ ] T14：降級確認 Dialog（含生效日期說明，SPEC-S04）
- [ ] T15：帳單列表頁（SPEC-B01/B02）

## Phase 5：用量 Dashboard（1.5h）
- [ ] T16：GET /api/usage/current——返回本月用量和剩餘次數
- [ ] T17：用量儀表板元件（進度條 + 歷史趨勢 Recharts 折線圖）
- [ ] T18：用量接近上限（80%）的 warning 提示

## 驗收清單
- [ ] Stripe test mode E2E：Free → 升級 Pro → 降級 Free
- [ ] 用量超額 429 測試（模擬 100+ 次呼叫）
- [ ] Webhook 冪等性：同一 invoice 的 webhook 送兩次，只記一筆帳單
- [ ] 月底重置測試（用 Stripe test clock 模擬時間推進）
```

---

## 這個範例與電商範例的關鍵差異

| 面向 | 電商 | SaaS 訂閱 |
|------|------|----------|
| **最難的問題** | 庫存並發超賣 | 用量計數的 race condition + Stripe webhook 冪等性 |
| **ADR 重點** | 金額用最小單位、地址快照 | 升降級時序、Redis vs PostgreSQL 職責分工 |
| **失敗 spec 最重要的** | 結帳時庫存不足 | 超用 429、付款失敗降級 |
| **測試最難的** | Stripe 付款 E2E | Stripe test clock（模擬月底）|
| **可以跳過的** | 推薦系統、評論 | 年付、企業方案、多成員 |

---

## 相關頁面

- [[openspec-product-design-guide|OpenSpec 產品設計教學]] — 完整教學，包含每個步驟說明
- [[openspec-example-ecommerce|範例一：電商網站]] — 另一個實戰範例
- [[Spec驅動開發]] — 方法論背景
- [[OpenSpec工作流]] — 8 階段流程
