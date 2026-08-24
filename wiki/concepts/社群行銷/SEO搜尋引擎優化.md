---
title: SEO 搜尋引擎優化
type: concept
tags: [seo, 行銷, 搜尋引擎, 內容行銷, ai搜尋, geo]
created: 2026-04-30
updated: 2026-04-30
sources: [whoops-seo-2026-guide]
---

# SEO 搜尋引擎優化

## 定義

SEO（Search Engine Optimization）= 透過**內容、技術、架構、內外部連結與信任訊號**的持續優化，讓網站在自然搜尋結果中獲得高排名與穩定流量。

> 核心邏輯：搜尋引擎想回答用戶的問題 → 你的內容是最好的答案 → 排名自然提升

---

## 搜尋引擎運作原理：三階段

```
爬取（Crawl）→ 索引（Index）→ 排名（Rank）

1. 爬取：Google bot 透過超連結探索網站內容
2. 索引：分析內容語意，決定是否收錄到資料庫
3. 排名：依據多項訊號排序，決定顯示位置
```

---

## 四大核心支柱

| 支柱 | 重點技術 | 常見工具 |
|------|---------|---------|
| **關鍵字與搜尋意圖** | 找對關鍵字、符合用戶真實需求 | Ahrefs、SEMrush、Google Search Console |
| **內容與 On-Page SEO** | 完整回答問題、標題/H2/H3 結構、內部連結 | Surfer SEO、Yoast |
| **技術 SEO** | 網站速度、Core Web Vitals、手機友善、索引健康 | PageSpeed Insights、Screaming Frog |
| **權威訊號（Off-Page）** | 反向連結質量、E-E-A-T、品牌提及 | Ahrefs、Majestic |

---

## 關鍵字策略

### 搜尋意圖四類型

| 意圖 | 關鍵字型態 | 用戶需求 | 內容類型 |
|------|----------|---------|---------|
| 資訊型 | 「什麼是 SEO」 | 學習 | 深度文章、指南 |
| 導航型 | 「Google Analytics 登入」 | 找網站 | 品牌頁 |
| 商業調查型 | 「SEO 工具比較」 | 比較選擇 | 比較表、評測 |
| 交易型 | 「SEO 課程報名」 | 購買 | 銷售頁、Landing Page |

### 長尾 vs 短尾

```
短尾關鍵字：SEO（搜尋量大、競爭高、轉換低）
長尾關鍵字：「小型電商 SEO 怎麼做」（搜尋量小、競爭低、轉換高）

新站策略：先攻長尾，累積排名後再打短尾
```

---

## On-Page SEO 技術要點

```
標題（Title Tag）：主關鍵字放前面，50–60 字元
H1：每頁只有一個，含主關鍵字
H2/H3：結構化內容，涵蓋相關關鍵字
Meta Description：150–160 字，含 CTA
內部連結：相關文章互相連結，分散 PageRank
圖片 Alt Text：描述性文字，含關鍵字（不硬塞）
URL 結構：簡短清晰，含關鍵字（/seo-guide/）
```

---

## 技術 SEO 重點

### Core Web Vitals（Google 排名因素）

| 指標 | 全名 | 標準 | 說明 |
|------|------|------|------|
| LCP | Largest Contentful Paint | < 2.5s | 最大內容元素載入時間 |
| CLS | Cumulative Layout Shift | < 0.1 | 版面跳動程度 |
| INP | Interaction to Next Paint | < 200ms | 互動反應時間（2024 取代 FID）|

### 常見技術問題清單

```
□ robots.txt 是否阻擋重要頁面
□ sitemap.xml 是否提交到 Google Search Console
□ 是否有重複內容（canonical tag 設定）
□ 404 頁面是否過多（redirect 處理）
□ HTTP 301 重定向是否正確
□ 手機版本載入速度（行動優先索引）
```

---

## E-E-A-T（Google 評分標準）

```
Experience（經驗）：撰寫者有第一手實際經驗嗎？
Expertise（專業）：具備該領域的深度知識嗎？
Authoritativeness（權威）：業界是否認可？被引用嗎？
Trustworthiness（可信度）：網站是否安全、透明？
```

**提升 E-E-A-T 的具體做法**：
- 作者頁：清楚介紹作者背景、資歷、照片
- 引用數據時標明來源（官方報告、學術論文）
- About 頁、聯絡方式、隱私政策完整
- 真實使用者評論和案例（第一手經驗）

---

## 2026 年最新趨勢

### 1. AISO（AI 搜尋優化）

Google AI Overviews、Perplexity、ChatGPT 等 AI 搜尋崛起，內容需被 **AI 選為答案來源**：

```
容易被 AI 引用的內容特徵：
✅ 定義塊（「X 是什麼？」直接給答案）
✅ 表格、條列（結構清晰，易於截取）
✅ 具體數字和數據（「提升 40% 轉換率」）
✅ 問答格式（FAQ Section）
✅ 短句、主動語態
```

### 2. GEO（生成式引擎優化）

**GEO（Generative Engine Optimization）**：讓品牌內容出現在 AI 生成回答中。

GEO 與 SEO 的差異：
| | 傳統 SEO | GEO |
|--|---------|-----|
| 目標 | Google 藍色連結排名 | AI 回答中被引用 |
| 衡量 | 排名、點擊率 | 被引用率、品牌提及次數 |
| 內容重點 | 關鍵字密度 | 可信度、事實準確性 |

### 3. Query Fan-out

AI 搜尋引擎會將複雜問題**拆解成多個子問題**：
```
用戶問：「電商新手怎麼做行銷？」
AI 分解為：
- 電商行銷是什麼？
- 電商常用的行銷管道有哪些？
- 預算有限的電商如何行銷？
```
→ 涵蓋更廣泛子主題的網站，被引用機率更高

### 4. 第一手經驗（First-hand Experience）

2024 年 Google Helpful Content 更新後，**純 AI 量產內容大幅失利**。有真實使用經驗的人類撰寫內容排名大幅提升。

---

## 新站 SEO 優先順序

```
第 1-3 個月：技術基礎
  → 確保網站可被爬取、速度合格、手機友善

第 3-6 個月：長尾關鍵字內容
  → 每週發布 2-4 篇深度文章，鎖定低競爭長尾詞

第 6-12 個月：建立反向連結
  → 客座文章、資源整合、被媒體引用

第 12 個月+：打競爭關鍵字
  → 以累積的權威攻打高搜尋量關鍵字
```

---

## 常用免費工具

| 工具 | 用途 |
|------|------|
| Google Search Console | 監控排名、點擊、索引狀態 |
| Google Analytics 4 | 流量分析、轉換追蹤 |
| Google PageSpeed Insights | 速度與 Core Web Vitals |
| Bing Webmaster Tools | Bing 搜尋監控 |
| Ubersuggest（免費版）| 關鍵字研究 |
| Answer the Public | 用戶常見問題挖掘 |

---

## 相關頁面

- [[社群內容策略]] — 內容行銷與 SEO 的整合
- [[IG演算法機制]] — IG 平台的搜尋邏輯（類 SEO）
- [[Email行銷與電子報]] — SEO 與 Email 行銷互補
- [[KOL網紅行銷策略]] — 反向連結與品牌聲量建立
