---
title: Back-of-the-Envelope 估算
type: concept
tags: [system-design, interview, estimation, scalability]
created: 2026-04-20
updated: 2026-04-20
sources: [2026-04-20_aritra-sen_beginners-guide-system-design.md]
---

# Back-of-the-Envelope 估算

系統設計面試的第二步。用粗略計算量化系統規模，為後續所有架構選型提供依據。

> 不需要精確，需要合理。估算的目的是驅動設計決策，不是考數學。

---

## 四個核心指標

### 1. DAU（Daily Active Users，每日活躍用戶）

最基礎的規模指標。

範例：
- Instagram 規模：~5億 DAU
- 中型 App：~100萬 DAU
- 新創早期：~10萬 DAU

### 2. 讀/寫頻率

判斷系統是**讀重**還是**寫重**，直接影響：
- 資料庫類型選擇
- 是否需要讀取副本（Read Replica）
- Cache 的重要程度

計算方式：
```
每秒寫入請求（WPS）= DAU × 每人每天寫入次數 / 86400
每秒讀取請求（RPS）= DAU × 每人每天讀取次數 / 86400
```

### 3. 儲存需求

```
每日儲存 = 每日寫入次數 × 每筆資料平均大小
年儲存量 = 每日儲存 × 365
```

**資料大小參考**：
| 類型 | 大小 |
|------|------|
| 文字（推文/訊息） | ~100 Bytes–1 KB |
| 用戶資料 | ~1–10 KB |
| 圖片（壓縮） | ~100 KB–1 MB |
| 影片（1分鐘） | ~10–100 MB |

> 注意：非長期活動系統（如售票App）只需估算活動期間，不需估1年。

### 4. 網路頻寬

```
每秒頻寬 = 峰值 RPS × 每個請求平均傳輸大小
```

判斷是否需要：
- **CDN**：靜態資源分發
- **壓縮**：減少傳輸大小
- **更多頻寬配置**

---

## 估算→決策的對應關係

| 估算結果 | 架構決策 |
|---------|---------|
| 寫重（WPS >> RPS） | NoSQL、Write-optimized DB、Queue 削峰 |
| 讀重（RPS >> WPS） | Read Replica、Cache 層、CDN |
| 儲存量 > TB 級 | Sharding、Object Storage（S3） |
| 峰值 RPS 極高 | Load Balancer、Queue、Cache |

---

## 常用換算

- 1天 = 86,400 秒（估算時常簡化為 10萬秒）
- 1年 ≈ 3,000萬秒（3 × 10⁷）
- 1 KB = 1,000 Bytes / 1 MB = 10⁶ Bytes / 1 GB = 10⁹ Bytes

---

## 相關概念

- [[系統設計面試模板]] — 估算是 Step 2
- [[SQL vs NoSQL 選型框架]] — 讀寫比決定資料庫選型
- [[分散式系統基礎概念]] — Cache、CDN、Sharding 的使用時機

## 來源

- [[aritra-sen-beginners-guide-system-design|A Beginner's Guide to System Design]]
