---
title: BDD 行為驅動開發
type: concept
tags: [BDD, Gherkin, Given-When-Then, 測試, SDD, AI開發]
created: 2026-05-18
updated: 2026-05-18
sources: [devtldrlss-ears-bdd-sdd, cashwu-sdd-from-tdd]
---

# BDD 行為驅動開發（Behavior-Driven Development）

BDD 是 TDD 的延伸，強調以「行為」而非「實作」描述系統應該如何運作。核心語法是 **Gherkin**，使用 Given/When/Then 三部曲，將需求轉化為可執行的驗收測試。

在 AI 輔助開發時代，BDD 格式因其結構固定、語意清晰，成為「讓 AI 精確理解需求」的最佳語法之一。

---

## Gherkin 三部曲

| 關鍵字 | 角色 | 說明 |
|--------|------|------|
| 🎬 **Given（背景）** | 前提條件 | 設定場景，系統目前處於什麼狀態 |
| ⚡ **When（動作）** | 觸發事件 | 使用者或系統執行了什麼操作 |
| ✅ **Then（結果）** | 預期結果 | 系統應該呈現什麼可觀察的結果 |

---

## 電子錢包扣款範例

```gherkin
# 正常扣款
Scenario: 餘額充足時成功扣款
  Given 使用者帳戶餘額為 $500
  When 使用者發起 $200 的扣款請求
  Then 系統應回傳 HTTP 200
  And 帳戶餘額應更新為 $300
  And 審計日誌應記錄本次扣款

# 餘額不足
Scenario: 餘額不足時拒絕扣款
  Given 使用者帳戶餘額為 $50
  When 使用者發起 $200 的扣款請求
  Then 系統應回傳 HTTP 402
  And 回應內容應包含當前餘額 $50
  And 帳戶餘額應保持 $50 不變
```

---

## BDD 在 AI 開發中的三個優勢

**1. 結構化**：固定的 Given/When/Then 框架讓 AI 不需猜測格式，直接解析意圖。

**2. 具體化**：實例化的數字和狀態（$500、HTTP 402）比抽象描述（「餘額夠」、「拒絕請求」）更精確。

**3. 可驗證**：每個 Then 子句都可以直接轉化為測試斷言（assertion），與 Jest、pytest 等測試框架天然整合。

---

## BDD + SDD 的整合公式（Cash Wu）

```
規格（WHAT）→ BDD 場景（驗收標準）→ AI 實作（HOW）→ 測試驗證
```

- 規格定義「要做什麼」
- BDD 場景定義「做對了嗎」的驗收標準
- AI 負責「怎麼做」的具體實作
- 測試自動驗證 BDD 場景是否通過

這個閉環讓 SDD 從「規格到程式碼」變成「規格到可驗證的程式碼」。

---

## BDD vs EARS vs User Story

| 格式 | 核心精神 | 寫作形式 | 主要受眾 | 解決痛點 |
|------|---------|---------|---------|---------|
| **BDD** | 共識與驗證 | 電影劇本 | 工程師、QA、AI | 邏輯驗證 |
| **EARS** | 精準定義 | 法律條文 | PM、架構師 | 需求歧義 |
| **User Story** | 用戶視角 | 對話體 | PM、PO | 需求溝通 |

---

## 相關頁面

- [[EARS需求語法]] — BDD 的互補工具：EARS 立規矩，BDD 確認演出
- [[好規格寫作原則]] — BDD 是好規格「Testing」區域的語法工具
- [[Spec驅動開發]] — BDD 是 SDD 驗收標準的標準格式
- [[OpenSpec文件格式與驗證]] — OpenSpec 的 Behavioral Specs 使用 Gherkin Scenario 格式
