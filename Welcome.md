# 知識庫

> 由 Claude 維護的 LLM Wiki。你負責提供來源與提問，Claude 負責萃取、整理、交叉連結。

---

## 快速導航

| 分類 | 入口 |
|------|------|
| 所有頁面目錄 | [[wiki/index\|Wiki Index]] |
| 操作紀錄 | [[wiki/log\|Activity Log]] |
| 原始資料 | `sources/` 與 `Clippings/` 資料夾 |
| Wiki 工作規則 | [[CLAUDE]] |

---

## 知識分類

### 📈 股票 / 期貨投資
- [[wiki/topics/investment-primer|投資入門路徑圖]] — 被動投資 / 主動選股 / 期貨三條路線
- [[wiki/concepts/期貨基礎概念]] — 保證金、槓桿、期貨種類
- [[wiki/concepts/台指期交易機制]] — 大台/小台/微台規格、盈虧計算
- [[wiki/concepts/股票基礎概念]] — 定義、獲利方式、台灣市場結構
- [[wiki/concepts/投資策略框架]] — ETF vs 個股、定期定額
- [[wiki/concepts/基本面分析入門]] — EPS、P/E、ROE、財報三表
- [[wiki/concepts/股票術語大全]] — 術語速查

### 📈 半導體 / 產業分析
- [[wiki/entities/UMC 聯電\|UMC 聯電]] — 2026 H2 晶圓漲價
- [[wiki/concepts/成熟製程AI需求]] — AI Server 的隱形成熟製程需求
- [[wiki/concepts/PMIC供需動態]] — 交期拉長 +14 週，漲價結構性支撐
- [[wiki/concepts/AI伺服器供應鏈]] — GPU 以外的完整供應鏈

### 🎥 內容創作
- [[wiki/entities/Ali Abdaal]] — YouTuber / 作家 / 創業家
- [[wiki/concepts/三階段成長框架]] — Get Going / Get Good / Get Smart
- [[wiki/concepts/HIVE影片結構框架]] — Hook / Intro / Value / End Screen
- [[wiki/concepts/內容飛輪]] — YouTube 主引擎 + 多平台延伸
- [[wiki/concepts/利基優先策略]] — 從窄眾切入建立專家地位

### 🎨 AI 圖像生成 / LoRA
- [[wiki/topics/lora-portrait-training|LoRA 人像訓練最佳實踐]] — Flux.1-dev + Z-Image-Turbo 完整參數指南
- [[wiki/concepts/LoRA訓練流程]] — 三階段工作流（初訓→迭代改善→重現）
- [[wiki/concepts/Feature Disentanglement]] — 三層標籤解耦，防止 concept bleeding
- [[wiki/concepts/訓練資料策略]] — Danbooru vs 合成資料策略
- [[wiki/entities/Z-Image-Turbo]] — 二次元風格基底模型

### 💻 系統設計面試
- [[wiki/topics/sd-overview\|題解總覽（19題）]] — 含學習路徑
- [[wiki/concepts/系統設計面試框架比較]] — 各框架對照
- [[wiki/concepts/RESHADED面試框架]] — R-E-S-H-A-D-E-D
- [[wiki/concepts/面試評分標準]] — 四維度評分

---

## 操作指令

| 操作 | 說明 |
|------|------|
| `ingest [URL 或檔名]` | 讀取來源、萃取知識、更新 wiki |
| `lint the wiki` | 健檢：找斷連、孤立頁、矛盾 |
| 直接提問 | Claude 查 wiki 後回答，附引用 |

---

## 統計

- Wiki 頁面：**73 頁**
- 來源數：**12 篇**（11 篇原始 + 1 篇合成）
- 最後更新：**2026-04-20**
