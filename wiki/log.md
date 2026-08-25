---
title: Wiki Activity Log
type: log
---

# Wiki Activity Log

Append-only chronological record of all wiki operations.
Each entry format: `## [YYYY-MM-DD] <operation> | <title>`
Operations: `ingest` | `query` | `lint` | `init`

## [2026-06-26] ingest | TradingView MCP Bridge

- Created: [[tradingview-mcp-bridge]]（source summary）、[[tradesdontlie]]（entity）、[[TradingView MCP Bridge]]（concept）
- Updated: wiki/index.md（+4 entries，total 474 pages）
- Key additions: 用 Chrome DevTools Protocol 橋接 Claude Code 與 TradingView Desktop；78 個 MCP 工具覆蓋讀圖/控圖/Pine Script 開發/回放練習；與富途 Skills 互補——TradingView MCP 做圖表分析，富途 Skills 執行下單；重要法律風險：TradingView ToS 限制自動化數據採集，且使用未文件化 Electron 內部 API 隨時可能失效

## [2026-06-26] ingest | 富途 Skill Hub：futuapi 官方技能頁

- Created: [[futunn-skillhub-openapi]]（source summary）
- Updated: [[富途OpenD Skills整合]]（futuapi 數字修正為 56 個 API 介面，補充回測範例）、[[富途 Futu]]（同步數字）；wiki/index.md（+1 entry，total 470 pages）
- Key additions: 精確數字——56 個 API 介面（35 行情 + 14 交易 + 7 推送）；官方 MACD 回測範例說明 Skills 可支援策略程式碼生成 + 歷史回測，不只是即時查詢；釐清腳本數（高層）vs API 介面數（底層）的計量差異

## [2026-06-26] ingest | 富途 Skills 入門指南：讓 AI Agent 幫你炒股

- Created: [[futuhk-futu-skills-intro]]（source summary）
- Updated: [[富途 Futu]]（補充五大市場、8 個具名 Skills 全表、13 種委託類型）、[[富途OpenD Skills整合]]（補充安裝指令、四層安全框架、即將推出的三個訊號 Skills、Level 2 訊號覆蓋更新）；wiki/index.md（+1 entry，total 469 pages）
- Key additions: 補充三個資訊分析 Skills（futu-news-search/futu-stock-digest/futu-comment-sentiment）已上線細節；三個訊號 Skills（技術面/資金面/衍生工具）即將推出；確認市場覆蓋為五大市場（含新加坡、日本）；安裝觸發語句：「根據指引安裝 Futu Skills：https://www.futunn.com/skills/futu-install.md」

## [2026-06-26] ingest | 富途 OpenD AI 整合指南

- Created: [[futunn-opend-ai-integration]]（source summary）、[[富途 Futu]]（entity）、[[富途OpenD Skills整合]]（concept）
- Updated: wiki/index.md（+3 entries，total 468 pages）
- Key additions: 富途官方提供符合 Claude Code Skills 標準的技能包（futuapi + install-futu-opend），含 14 行情 + 8 交易 + 5 訂閱腳本；與 Alpaca 的關鍵差異在於本地 OpenD 代理架構 vs 雲端 REST；預設模擬盤是重要安全設計

## [2026-06-26] ingest | Zero2Agent：從零實現產品級 Agent Harness

- Created: [[alienzhou-zero2agent]]（source summary）、[[alienzhou]]（entity）、[[ToolContext模式]]（concept）、[[Agent Harness實作]]（concept）
- Updated: wiki/index.md（+4 entries，total 465 pages）
- Key additions: Zero2Agent 是目前最完整的「從零實作 Agent Harness」公開教學課程，以 TypeScript 實作 ReAct Loop + ripgrep 工具鏈，全程附 VibeCoding 對話記錄與 SDD 設計文件；ToolContext 模式是 E01-S003 引入的關鍵架構決策（顯式注入取代隱式 process.cwd()）

---

## [2026-06-11] ingest | AI 算八字到底准不准？+ BaziQA Dataset（AuraMate / GitHub: chenjiangxi/baziqa）
- Created: [[auramate-ai-bazi-reasoning-benchmark]], [[github-chenjiangxi-baziqa]], [[AuraMate]], [[BaziQA-Benchmark]], [[八字推命]]
- Updated: wiki/index.md (420 pages, +4 entries)
- Key additions: BaziQA-Benchmark 作為 LLM 符號×時序推理基準的完整框架（450 題、排行榜、SRP 方法論）；AuraMate 實體頁；八字推命領域入門概念頁

## [2026-06-09] ingest | 祛魅，才是變強的開始，一個人高能量的根本（YouTube Shorts）
- Created: [[祛魅]], [[匱乏感投射]], wiki/sources/quwei-become-stronger.md
- Updated: [[個體化歷程]]（新增祛魅反向連結）
- Key additions: 祛魅概念完整框架（定義/三層次/vs 幻滅對比）；匱乏感投射機制（與白色陰影投射的連結）；均整合入心理學分類

## [2026-05-25] ingest | ForceInjection/domain-driven-design-skills（GitHub）

- Created (4):
  - `wiki/sources/forceinjection-ddd-skills.md`（來源摘要：10條 Key Takeaway；9技能×5階段架構；回溯觸發機制；Cargo Shipping 盲跑 85.8%；矛盾分析：中小團隊 DDD 基礎門檻；待解問題：端到端自動化程度）
  - `wiki/entities/ForceInjection.md`（GitHub 組織；OpenSpec-practise 317⭐ + domain-driven-design-skills 兩大 repo；修復之前健檢發現的缺失 entity）
  - `wiki/concepts/ai開發/DDD AI Agent技能流水線.md`（9技能清單×輸出產物；非線性5種入口；SKILL.md 七節契約格式；會議室範例完整走過 5 階段；Cargo Shipping 驗證結果）
  - `wiki/concepts/ai開發/DDD回溯觸發機制.md`（8條品質門禁完整觸發條件表；3個最常觸發規則詳解；防死迴圈設計；與 OpenSpec validate / TDD / BDD 的對比）
- Updated (5):
  - `wiki/concepts/go-backend/DDD領域驅動設計.md`（新增 DDD AI Agent技能流水線 + DDD回溯觸發機制 兩個反向連結）
  - `wiki/concepts/ai開發/OpenSpec工作流.md`（新增 ddd-openspec-bridge 連結說明）
  - `wiki/entities/OpenSpec.md`（新增 DDD AI Agent技能流水線 相關頁面連結）
  - `wiki/index.md`（401→405 pages；Entities 新增 ForceInjection；AI開發 Concepts 新增 2 個；Source Summaries 新增 1 個）
  - `wiki/log.md`
- Key additions: 建立「DDD × AI Agent」知識體系——核心創新是**回溯觸發機制**（8條量化品質門禁，允許 AI Agent 在建模流程中自我修正而非盲目推進）；`ddd-openspec-bridge` 打通 DDD 戰術建模→OpenSpec 工程實作的最後一公里；ForceInjection entity 補齊已引用但缺頁的 broken link。

---

## [2026-05-25] lint | Wiki 健檢修復（第三次）

- 修復 (6 處斷鏈):
  - `[[AMM流動性提供與無常損失]]` → `[[無常損失與AMM流動性提供]]`（`概念/加密量化交易/資金費率套利.md`）
  - `[[Banking System]]` → `[[Banking_System]]`（`概念/CodeSignal/Coinbase_HA_總覽.md`）
  - `[[In-Memory Database]]` → `[[In_Memory_Database]]`（`概念/CodeSignal/Coinbase_HA_總覽.md`）
  - `[[explainthis.io]]` → `[[explainthis-io|explainthis.io]]`（index.md + ThePrimeagen + Boot.dev + source 共 4 檔）
  - `[[Claude Skills]]` → `[[Claude Code Plugins插件系統|Claude Skills]]`（`entities/Skillsmap.md`）
  - `[[Claude Skills完整教學]]` → `[[Claude Code Plugins插件系統]]`（`entities/Skillsmap.md`）
- 建立 (1 個缺失 entity):
  - `wiki/entities/榮格 Carl Jung.md`（分析心理學創始人；被心理學概念區 8+ 頁引用但無實體頁）
- 補連結 (2 頁):
  - `榮格陰影理論.md` + `個體化歷程.md`：新增 `[[榮格 Carl Jung]]` 反向連結
- Updated (1): `wiki/index.md`（400→401 pages；新增榮格 Carl Jung 實體條目）

---

## [2026-05-22] ingest | 加密貨幣量化交易策略全攻略（quant67.com）

- Created (12):
  - `wiki/sources/quant67-14-crypto-strategies.md`
  - `wiki/concepts/加密量化交易/資金費率套利.md`
  - `wiki/concepts/加密量化交易/無常損失與AMM流動性提供.md`
  - `wiki/concepts/加密量化交易/Curve-Convex收益策略.md`
  - `wiki/concepts/加密量化交易/加密借貸利率套利.md`
  - `wiki/concepts/加密量化交易/跨鏈橋風險.md`
  - `wiki/concepts/加密量化交易/加密量化投資組合配置.md`
  - `wiki/concepts/加密量化交易/加密量化基礎設施.md`
  - `wiki/entities/Curve Finance.md`
  - `wiki/entities/Convex Finance.md`
  - `wiki/entities/ccxt.md`
  - `wiki/entities/Fireblocks.md`
- Updated (2): `wiki/index.md`（388→400 pages；新增加密量化交易 concepts 區塊 + 4 entities）、`wiki/log.md`
- Key additions: 建立加密量化交易完整知識體系——六大策略類別（資金費率套利/AMM LP/Curve-Convex/借貸利率/跨鏈套利/搬磚）；重點突出無常損失公式與跨鏈橋 $2.5B+ 安全損失；收錄 $1B AUM 機構級工程棧與投資組合配置框架；核心觀點：「生存本身即 alpha」

## [2026-05-19] ingest | 刑事程序裡的證據法則（一）（三）（四）（五）——朱石炎系列批次（Legis-pedia）

- Created (10):
  - `wiki/sources/legis-pedia-evidence-rules-criminal-procedure-1.md`
  - `wiki/sources/legis-pedia-evidence-rules-criminal-procedure-3.md`
  - `wiki/sources/legis-pedia-evidence-rules-criminal-procedure-4.md`
  - `wiki/sources/legis-pedia-evidence-rules-criminal-procedure-5.md`
  - `wiki/concepts/積極證據.md`（核心強調：76台上4986 + 113台上774兩大判例）
  - `wiki/concepts/待證事實.md`
  - `wiki/concepts/證據裁判原則與無罪推定.md`
  - `wiki/concepts/直接證據與間接證據.md`
  - `wiki/concepts/證據能力.md`
  - `wiki/concepts/法院調查證據程序.md`
- Updated (3): `wiki/entities/朱石炎.md`（新增四篇作品）、`wiki/index.md`（378→388 pages）、`wiki/log.md`
- Key additions: 完整建立刑事證據法則系列骨幹體系（第一至六篇全覆蓋）；核心突出積極證據概念——定罪須有積極證據達超越合理懷疑，間接證據彼此補強亦可構成；收錄最高法院76年台上4986號及113年台上774號兩大指標判例

## [2026-05-19] ingest | 刑事程序裡的證據法則（六）——證明力（朱石炎／Legis-pedia）

- Created (7):
  - `wiki/sources/legis-pedia-evidence-rules-criminal-procedure-6.md`
  - `wiki/entities/朱石炎.md`
  - `wiki/entities/Legis-pedia.md`
  - `wiki/concepts/證明力.md`
  - `wiki/concepts/自由心證原則.md`
  - `wiki/concepts/嚴格證明與自由證明.md`
  - `wiki/concepts/台灣刑事訴訟法.md`
- Updated (2): `wiki/index.md`（371→378 pages）、`wiki/log.md`
- Key additions: 建立台灣刑事證據法核心概念體系——證明力評價層級（直接>間接>物證>人證）、自由心證原則的論理/經驗法則約束、嚴格vs自由證明雙軌分流，以及「超越合理懷疑」標準的中文定義

## [2026-05-13] organize | 時序資料庫設計.md, 分散式任務排程系統.md, 系統設計效能基準與QPS速查.md

- Created: `時序資料庫設計.md` → `系統設計/` — 系統設計技術主題，與同資料夾的分散式系統基礎概念、SQL vs NoSQL 選型框架屬同一領域
- Created: `分散式任務排程系統.md` → `系統設計/` — 分散式系統設計主題，使用 Cassandra / ZooKeeper / Message Broker 等系統設計核心技術
- Created: `系統設計效能基準與QPS速查.md` → `系統設計/` — 系統設計估算輔助工具，與同資料夾的 Back-of-the-Envelope 估算直接配套
- No new category created — `系統設計/` 已有 11 頁系統設計相關內容，完全適合

## [2026-05-13] ingest | JesseZhuang/SystemDesign + nowcoder 八股文（批次）

### JesseZhuang/SystemDesign（https://github.com/JesseZhuang/SystemDesign）
- Created (1): `wiki/sources/github-jessezhuang-systemdesign.md`
- Updated (1): `wiki/entities/JesseZhuang.md`（新增 SystemDesign repo；擴充相關頁面）
- Key additions: 確認 JesseZhuang 同時有 Python OOP 與系統設計兩個學習 repo；DDIA 12 章筆記；22 個設計案例含時序DB、任務排程兩個 wiki 未有主題

### nowcoder 系統設計八股文（https://www.nowcoder.com/discuss/377179563154055168）
- Created (1): `wiki/sources/nowcoder-system-design-bagu.md`
- Key additions: QPS 效能基準速查（Nginx/Redis/MySQL/Tomcat）；TinyURL 302 vs 301（已有 wiki 頁面驗證）；串流算法題型清單（Top K / HyperLogLog / Bloom Filter / Count-Min Sketch）

### nowcoder mianshi/top（https://m.nowcoder.com/mianshi/top）
- ⚠️ 頁面返回 404，無法取得題目清單；已記錄平台為重要中文面試準備資源（牛客網），待後續訪問有效 URL

### 概念頁面（共 3 個）
- Created: `wiki/concepts/系統設計/時序資料庫設計.md` — TimescaleDB + Pinterest Goku 雙方案詳解
- Created: `wiki/concepts/系統設計/分散式任務排程系統.md` — Cassandra + Broker + ZooKeeper，含 DelayQueue Python 實作
- Created: `wiki/concepts/系統設計/系統設計效能基準與QPS速查.md` — 組件 QPS 基準、延遲表、估算公式速查

## [2026-05-12] organize | Cloud_Storage_OOP設計題.md

- Created: `Cloud_Storage_OOP設計題.md` → `interview/` — OOP 面試設計題（CodeSignal 四層題型），與同資料夾的 Banking_System、In_Memory_Database、Low Level Design OOD設計題型屬同一領域
- No new category created — `interview/` 已有多個 OOP 設計題頁面，完全適合

## [2026-05-12] ingest | JesseZhuang/InCodeLearning-Python3（GitHub）

- Created (3):
  - `wiki/sources/github-jessezhuang-incode-python3.md`（來源摘要：六個 OOP 設計題分析；banking_system 合併版特性；cloud_storage 獨有題型；backup/restore 設計慣例觀察）
  - `wiki/entities/JesseZhuang.md`（Python 演算法學習者，409 commits，最完整 OOP 設計題 Python 實作）
  - `wiki/concepts/interview/Cloud_Storage_OOP設計題.md`（四層雲端儲存 OOP 設計：Level 1 基礎 CRUD / Level 2 n_largest 排名 / Level 3 多用戶容量+merge_user / Level 4 backup/restore；與 In-Memory DB 和 Banking System 的共同設計模式對照表）
- Updated (1):
  - `wiki/concepts/CodeSignal/Banking_System.md`（新增 JesseZhuang 合併版 Python 實作參照）
- Key additions: 首次收錄 Cloud Storage OOP 設計題（疑為 CodeSignal 題庫）；確認 backup/restore 是 CodeSignal Level 4 的慣用模式；banking_system.py 同時含 cancel_payment + pay_v2 cashback，暗示不同批次面試規格可能混合使用。

## [2026-05-12] organize | Go GraphQL gqlgen實戰.md

- Created: `Go GraphQL gqlgen實戰.md` → `go-backend/` — Go 微服務/後端工程概念，與同資料夾的 gRPC、REST API 架構等後端技術屬同一領域；`go-backend/` 已有 30+ 頁 Go 後端相關內容
- No new category created — `go-backend/` 完全適合

## [2026-05-12] ingest | AlejandroAldana99/Go-codes（GitHub）

- Created (3):
  - `wiki/sources/github-alejandroaldana99-go-codes.md`（來源摘要：六個子專案分析；gqlgen GraphQL、MongoDB REST API 七層架構、JWT 訂單狀態機、Activity Tracker OOP 設計）
  - `wiki/entities/AlejandroAldana99.md`（Go backend developer，五個練習專案）
  - `wiki/concepts/go-backend/Go GraphQL gqlgen實戰.md`（gqlgen codegen 模式、server.go 完整範例、Schema 定義、gqlgen.yml 配置、REST vs GraphQL 選型表）
- Key additions: Wiki 首次加入 Go GraphQL（gqlgen）完整頁面；確認 Credit-Assignment API Postman collection 含「YoFio」字樣，疑為 YoFio 公司面試題。

## [2026-05-12] organize | HashiCorp_Banking_System_OA.md, Acresium_Kubernetes_SRE_面試題.md

- Created: `HashiCorp_Banking_System_OA.md` → `interview/` — OOP 面試設計題，與同資料夾的 Low Level Design OOD 設計題型屬同一領域
- Created: `Acresium_Kubernetes_SRE_面試題.md` → `interview/` — 真實 SRE Lead 面試記錄，符合 interview/ 資料夾的「面試準備與真實題目」定位
- No new category created — `interview/` 已有 14 頁面試相關內容，完全適合

## [2026-05-12] ingest | karuppiah7890/interview-questions（GitHub）

- Created (4):
  - `wiki/sources/github-karuppiah7890-interview-questions.md`（來源摘要：三個面試來源分析；Banking System 跨公司重複題型警示；Cashback vs cancel_payment 差異）
  - `wiki/entities/karuppiah7890.md`（軟體工程師，面試題庫作者；HashiCorp / Acresium / LinkedIn 三個面試記錄）
  - `wiki/concepts/interview/HashiCorp_Banking_System_OA.md`（HashiCorp OOP 預評測：4層題目、Level 2 TOP_ACTIVITY/TOP_SPENDERS、Level 3 Cashback 2%；與 Coinbase 版本差異對照表；Go+Python 雙語）
  - `wiki/concepts/interview/Acresium_Kubernetes_SRE_面試題.md`（2025 SRE Lead 面試，Kubernetes+分散式系統，截圖形式）
- Updated (1):
  - `wiki/concepts/CodeSignal/Banking_System.md`（新增 HashiCorp 版本交叉參照）
- Key additions: Banking System OOP 設計題確認為跨公司重複題型（HashiCorp + Coinbase），且兩版本在 Level 3 API 上有明顯差異（Cashback vs cancel_payment）。新增 Acresium SRE Lead K8s 面試記錄，補充 Kubernetes/SRE 面試場景。

## [2026-05-06] ingest | 我用 Claude Code 一個下午做出 4 個 AI 交易員（guccidgi.com）

- Created (4):
  - `wiki/sources/gucci-claude-code-4-ai-traders.md`（來源摘要：10 條 Key Takeaway；矛盾分析：「不用寫程式」的門檻澄清、Iron Condor 風報比警示；待解問題：Trailing Stop 觸發機制、Form 4 延遲）
  - `wiki/concepts/ai開發/Claude Code量化交易四層系統.md`（Level 1–4 完整框架：Alpaca 連線→底撈機器人＋Trailing Stop→SEC Form 4 內線追蹤＋Capitol Trades 國會追蹤→方向性選擇權掃描＋Iron Condor；Routines 排程 + Telegram Bot 推播）
  - `wiki/concepts/投資/選擇權交易策略.md`（Trailing Stop、IV Crush、Credit/Debit Spread、Iron Condor 四大策略速查；Delta/PoP/Premium 指標說明；風報比警示）
  - `wiki/entities/Alpaca.md`（API-first 美國券商；Paper Trading 功能；Claude Code 量化系統的執行平台）
- Updated (2):
  - `wiki/entities/追日Gucci.md`（新增第二個 source；補充 Alpaca/四層系統/選擇權相關頁面連結）
  - `wiki/index.md`（Entities 新增 Alpaca、更新 追日Gucci；AI 開發概念新增 Claude Code量化交易四層系統；投資概念新增 選擇權交易策略；Source Summaries 新增 1 個；頁數 278→282）
- Key additions: 「三大鴻溝」框架（資訊差/紀律差/方便差）是散戶 vs 法人的結構性分析，Claude Code 分別在 Level 3/2/4 解決；「選擇權交易策略」是本 wiki 投資分類的第一批衍生品內容，特別標注 Iron Condor 的風報比問題（Max Loss $3,100 vs Max Profit $369，勝率需夠高才能正期望值）；Alpaca Paper Trading 是學習成本最低的自動化交易入門路徑。

---

## [2026-05-06] ingest | Claude Code創始人Boris揭秘完整工作流設定（YouTube Xm-n4m7IaZk）

- Created (3):
  - `wiki/sources/boris-claude-code-setup-youtube.md`（來源摘要：Boris 個人設定公開揭秘，原始爆紅推文 8M 瀏覽；10 條 Key Takeaway；矛盾分析：「簡單」的真正含義）
  - `wiki/concepts/claude/複利工程思維.md`（Dan Shipper 概念 × Boris 實踐：CLAUDE.md 飛輪 / @.claude PR 評論 / slash command 積累；複利 vs 一般自動化對比；Anthropic 工程師產出提升 200% 佐證）
  - `wiki/concepts/claude/Boris的Claude Code完整工作流.md`（完整設定 10 節：環境/模型/工作流/Slash Commands/Hooks/MCP/CLAUDE.md/Context管理/提示詞技巧/簡約哲學）
- Updated (2):
  - `wiki/entities/Boris.md`（全面擴充：加入 Boris Cherny 全名、Staff Engineer 職稱、8M 瀏覽數據、複利工程思維說明、100% AI 程式碼/6個月無手寫 SQL 等工作方式細節）
  - `wiki/index.md`（Entities 更新 Boris 條目；Claude 概念新增複利工程思維 + Boris完整工作流；Source Summaries 新增 1 個；頁數 275→278）
- Key additions: 「複利工程思維」是本次最重要的新概念——Boris 明確命名的核心哲學：每次對工作流的小改善（CLAUDE.md 更新、新 slash command、@.claude PR 評論）都自動在未來所有 session 和所有團隊成員中生效，形成複利效應；「設定極簡開箱即用」是主張，但「簡單」指不需客製化 Claude 行為，工作流框架（worktree/iTerm2通知/hooks/MCPs）仍需建立；驗證是 #1 優先級（2–3x 品質提升）。

---

## [2026-05-06] create | 用 Claude 從零開發 Golang Web Service 完整教程
- Created: [[claude-golang-webservice-from-scratch]]
- Updated: wiki/index.md（Golang 架構分類新增條目，page count 274→275）
- Key additions: 7 步從零到一教程，涵蓋架構設計對話、CLAUDE.md 護欄建立、逐層 scaffold、以 POST /api/users 為完整 Spec 驅動開發實例（含 handler/usecase/repository 三層 Behavioral Spec + 提示詞模板 + 驗證指令）

---

## [2026-05-05] ingest | 李惠心理：永远不要指出你身边人的问题，哪怕你是对的（YouTube MoR2rG78eP4）

- Created (3):
  - `wiki/sources/lihuipsychology-dont-point-out-problems.md`（完整逐字稿分析；10條金句；核心論點：被看見 vs 被糾正；防禦機制；課題分離；空間即尊重）
  - `wiki/entities/李惠心理.md`（中國心理咨詢師/督導師；150萬+訂閱；精神分析×自體心理學×阿德勒；六階段付費課程體系）
  - `wiki/concepts/被看見vs被糾正.md`（核心心理需求差異；防禦機制啟動機制；關係損耗模式；課題分離；例外邊界討論）
- Updated (1):
  - `wiki/index.md`（Entities 新增 1 個、Concepts 心理學新增 1 個、Source Summaries 新增 1 個；頁數 270→274）
- Key additions: 「被看見 vs 被糾正」是本次最核心的新概念——指出別人問題會觸發防禦機制，使溝通失效，且會磨損關係（「道理贏了，感情沒了」）；長久相處的核心是給予空間，而非成為修正器；阿德勒課題分離的實用應用。字幕透過 Node.js youtube-transcript 套件抓取。

---

## [2026-05-05] ingest | ThePrimeagen × Boot.dev：4 小時 SQL 完整課程（YouTube rf57kE3HJD0）

- Created (2):
  - `wiki/sources/theprimeagen-sql-course-bootdev.md`（11 章課程結構 + 完整 Timestamps；實況格式說明；Boot.dev 課程連結）
  - `wiki/concepts/SQL核心概念.md`（11 章完整語法速查：Tables/Constraints/CRUD/Queries/Aggregations/Subqueries/Normalization/Joins/Performance；EXPLAIN 調優；N+1/Cursor Pagination 等常見問題）
- Updated (3):
  - `wiki/entities/ThePrimeagen.md`（新增 11 章時間戳表格、TJ DeVries 共講資訊、課程連結）
  - `wiki/entities/Boot.dev.md`（新增課程結構、訂閱方案、內容引流策略說明）
  - `wiki/index.md`（Concepts 新增「SQL / 資料庫」分類；Source Summaries 新增 1 個；頁數 268→270）
- Key additions: SQL 從入門到效能調優的完整 11 章知識地圖，以 ThePrimeagen 課程結構為骨架；正規化（1NF/2NF/3NF）、四種 JOIN、索引原理與 EXPLAIN 是核心重點；影片原始標題「Ex-Netflix Engineer fails SQL for 4 hours」已被 Boot.dev 更名為「Our Blazingly Fast SQL Course Speerdun」。

---

## [2026-05-05] ingest | ThePrimeagen：前 Netflix 工程師教的免費 SQL 入門影片（explainthis.io Threads）

- Created (4):
  - `wiki/sources/explainthis-theprimeagen-sql-4hours.md`（source summary：ThePrimeagen 實況解題 SQL 4 小時影片，Boot.dev 平台，YouTube rf57kE3HJD0）
  - `wiki/entities/ThePrimeagen.md`（前 Netflix 工程師、YouTuber；實況解題教學風格；37K views in 2 days）
  - `wiki/entities/explainthis-io.md`（台灣軟體工程學習平台；Threads 內容創作；E+ 成長計畫）
  - `wiki/entities/Boot.dev.md`（後端學習平台；遊戲化課程；與 ThePrimeagen 合作免費 YouTube 引流）
- Updated (1):
  - `wiki/index.md`（Entities 新增 3 個、Source Summaries 新增 1 個；頁數 264→268）
- Key additions: ThePrimeagen 的「實況解題教學法」——展示資深工程師卡住時如何思考解決，而非預先剪輯的順暢示範；explainthis.io 作為台灣工程師學習資源推薦的新來源頻道。

---

## [2026-05-04] init | 用 OpenSpec 漸進式遷移 Legacy Gin+GORM 架構

- Created (1):
  - `wiki/topics/gin-legacy-to-clean-arch-openspec.md`（7階段遷移指南：Explore→Proposal→Design→Behavioral Specs→Tasks→Apply→Verify→Archive；/api/v2 共存遷移法；各層具體 Prompt 模板；5大常見陷阱；一個 Module 完整 Prompt 序列）
- Updated (1):
  - `wiki/index.md`（Topics 新增「Golang 架構」分類，頁數 259→260）
- Key additions: 漸進式遷移的核心設計（新功能只用新架構、舊功能等自然觸碰再遷移）；v1/v2 路由共存法；每個遷移 phase 的具體 Claude Code prompt；Behavioral Specs 的 Given/When/Then 格式範例；避免讓 AI 一次重構整個 repo 的邊界設定技巧。

---

## [2026-05-04] ingest | Coinbase OA 題目總表（interviewdb.io + prachub + hack2hire + GitHub）

- Created (4):
  - `wiki/concepts/CodeSignal/Coinbase_OA題目總表.md`（14道題目索引：難度、資料完整度、各題摘要、核心演算法速查；來源：interviewdb.io / prachub.com / hack2hire.com / GitHub）
  - `wiki/concepts/CodeSignal/Max_Fee.md`（Maximize Block Fee：Greedy Level 1 + Tree DP parentId依賴 Level 2；完整Go解法）
  - `wiki/concepts/CodeSignal/NFT_Generator.md`（NFT Metadata Generator：Mixed-Radix Decoding + coprime step唯一性生成；big.Int；完整Go解法）
  - `wiki/concepts/CodeSignal/Order_Management.md`（Crypto Order Management：3-Level狀態機→user索引→fnv sharding；完整Go解法）
- Updated (1):
  - `wiki/index.md`（CodeSignal分類新增4個頁面，頁數 260→264）
- Key additions: 首次系統性整理所有Coinbase OA題目；完整描述的題目：Max Fee、NFT Generator、Order Management（加上已有的Banking System / In-Memory Database共5題完整Go解法）；Garbled Logs / Exchange / Price Stream / Transaction Filter 有部分描述；Tabemono / Drone / Blockchain Indexer 僅有推測。

---

## [2026-05-04] ingest | go-gin-clean-arch（GitHub thnkrn）

- Created (2):
  - `wiki/sources/github-go-gin-clean-arch.md`
  - `wiki/concepts/Golang/Gin_Clean_Architecture.md`（4層架構說明、完整目錄結構、各層程式碼範例、新增Domain 7步流程、OpenSpec AI設計工作流、Wire DI chain、架構優缺點）
- Updated (1):
  - `wiki/index.md`（Golang 新增「架構與框架」子分類，新增 Gin_Clean_Architecture 條目 + source summary，頁數 257→259）
- Key additions: Clean Architecture 在 Go/Gin 的完整實作範例；關鍵模式：constructor 回傳 interface（依賴反轉）、copier DTO 轉換、Wire provider chain；新增 Domain 需要 7 個檔案的步驟清單；提供 OpenSpec/Claude Code 的實際 Prompt 範例。

---

## [2026-05-04] ingest | Coinbase Online Assessment 2025/2026（Lodely）

- Created (1):
  - `wiki/sources/lodely-coinbase-oa-2025.md`
- Updated (3):
  - `wiki/concepts/CodeSignal/Coinbase_HA_總覽.md`（新增：監考機制表、14道DSA題型清單含LeetCode對應、3大Fintech特定題型）
  - `wiki/concepts/CodeSignal/Coinbase_面試全流程.md`（新增：IC3~IC8薪資對照表，Levels.fyi 2026/03資料）
  - `wiki/index.md`（新增 source summary 條目，頁數 256→257）
- Key additions: DSA 題型清單（14題+LeetCode對應）首次加入 wiki；監考機制細節（alt-tab偵測、分數快取）；薪資 IC5 $410K / IC8 $1.19M+ median TC；⚠️ 分數制度矛盾待確認（Lodely: 500+/800 vs HA格式: 500+/1000）。

---

## [2026-05-04] ingest | Coinbase's Interview Process (2026)（TechPrep）

- Created (2):
  - `wiki/sources/techprep-coinbase-interview-process-2026.md`
  - `wiki/concepts/CodeSignal/Coinbase_面試全流程.md`（完整5階段：Recruiter→OA→Phone→Onsite→HM；各輪準備重點；LLD code modularity優先；SQL cursor-based pagination必考；Behavioral apolitical文化適配）
- Updated (2):
  - `wiki/concepts/CodeSignal/Coinbase_HA_總覽.md`（新增連結至 Coinbase_面試全流程）
  - `wiki/index.md`（新增 Coinbase_面試全流程 概念頁、source summary 條目，頁數 254→256）
- Key additions: 補齊 OA 以外的完整面試流程；關鍵新資訊：LLD Tech Execution 是最具決定性的關卡（code modularity > 演算法效率）；2025 候選人明確報告 SQL cursor-based pagination 必考；2026 開始明確詢問 apolitical 文化適配。

---

## [2026-05-04] ingest | Coinbase CodeSignal HA 題目（Banking System + In-Memory Database）

- Created (3):
  - `wiki/concepts/CodeSignal/Coinbase_HA_總覽.md`（考試格式：1大題4關，90分鐘，250分/關；Version A/B 兩種題目池；Level難度表；應試策略；新建 `CodeSignal` 概念分類）
  - `wiki/concepts/CodeSignal/Banking_System.md`（Version A 完整題目（Level 1–4）+ Go 解法：HashMap帳戶管理、top_spenders排序、排程付款processPayments、get_balance binary search、MergeAccounts sorted merge）
  - `wiki/concepts/CodeSignal/In_Memory_Database.md`（Version B 完整題目（Level 1–4）+ Go 解法：nested HashMap CRUD、scan/scan_by_prefix排序、TTL惰性過期、backup deep copy、restore TTL偏移重算）
- Updated (1):
  - `wiki/index.md`（新增 `CodeSignal 面試題` 分類區塊，頁數 251→254）
- Key additions: 新建 CodeSignal 面試題分類；涵蓋 Coinbase HA 兩種題目池的完整 Go 解法；重點演算法：processPayments（排程付款觸發）、balanceAt（binary search 歷史餘額）、sorted merge（帳戶合併）、deep copy + TTL偏移（backup/restore）。

---

## [2026-05-01] init | 台灣法規違規檢舉分類建立

- Created (3):
  - `wiki/concepts/台灣法規/台灣法規違規檢舉指南.md`（分類總覽：主要法規、檢舉獎金制度、違規辨識重點、分析連結標準流程與報告模板；新建 `台灣法規` 概念分類）
  - `wiki/concepts/台灣法規/食品安全衛生管理法.md`（食安法：標示必要項目、廣告違規宣稱類型、禁用添加物、罰則、第49條之1獎金制度、蒐證清單）
  - `wiki/concepts/台灣法規/化粧品衛生安全管理法.md`（化粧品法：化妝品vs藥品宣稱界線、全成分標示規範、禁用限用成分、廣告高風險關鍵字、分析步驟）
- Updated (1):
  - `wiki/index.md`（新增 `台灣法規` 分類區塊，頁數 248→251）
- Key additions: 新建台灣消費者法規檢舉研究分類，涵蓋食品、化妝品、健康食品三大領域；內建分析連結的標準流程與報告模板，後續用戶丟入連結可直接套用。

---

## [2026-04-30] ingest | 操控人心的說話術（Hami 書城，2022）

- Created (3):
  - `wiki/concepts/心理學/煤氣燈操縱法.md`（定義、四種核心模式、破解方式；新建 `心理學` 概念分類）
  - `wiki/concepts/心理學/操控說話術六大戰術.md`（六種操控話語戰術速覽與說明）
  - `wiki/sources/books-tw-control-speech-manipulation-2022.md`
- Updated (1):
  - `wiki/index.md`（新增 `心理學` 分類區塊、兩個概念條目、source summary 條目，頁數 245→248）
- Key additions: 新建「心理學」概念分類；核心概念為煤氣燈操縱法（Gaslighting）的四種模式與識別方法；書中強調操縱者多為潛意識運作，並釐清「說出感受」與「情緒勒索」的差異。

---

## [2026-04-30] ingest | Chase AI：Claude Code + Higgsfield MCP = Content MACHINE（YouTube 教學）

- Created (3):
  - `wiki/entities/Higgsfield.md`（AI 影像／影片生成平台，提供 MCP Server，文字生成圖像／影片，風格一致性控制）
  - `wiki/concepts/內容創作/輪播內容自動化工作流.md`（Claude Code + Higgsfield MCP 四步流水線、工具角色分工、CLAUDE.md 品牌規範、效率對比表、在 LIFE 系統的定位）
  - `wiki/sources/chase-ai-higgsfield-mcp-carousel.md`
- Updated (1):
  - `wiki/index.md`（新增 Higgsfield 實體、輪播內容自動化工作流 概念、source summary 條目，頁數 242→245）
- Key additions: 以 Claude Code + Higgsfield MCP 為核心的 IG 輪播內容自動化流水線；強調 CLAUDE.md 作為品牌記憶體的角色，以及此工作流在 LIFE 根系系統 I 層的定位。

---

## [2026-04-30] ingest | growithfyn 輪播是 IG 個人品牌建立信任的關鍵資產（影片貼文）

- Created (2):
  - `wiki/concepts/內容創作/IG輪播轉換設計.md`（輪播=精簡銷售流程；被看見→被信任→願意付費；心理路徑設計；Reels vs 輪播分工；在 LIFE 系統中的定位）
  - `wiki/sources/growithfyn-carousel-conversion-ig.md`
- Updated (2):
  - `wiki/entities/growithfyn.md`（新增 IG輪播轉換設計 框架與反向連結）
  - `wiki/index.md`（240→242 頁）
- Key additions:
  - 輪播核心定位：短影音觸及陌生人，輪播深化信任並轉換，角色分工清晰
  - 心理路徑：現況共鳴 → 問題放大 → 新觀點 → CTA，每張都是一個決策節點
  - 1.1 萬留言印證痛點：多數創作者只輸出零散資訊，缺乏引導決策的結構設計

## [2026-04-30] ingest | growithfyn LIFE 根系內容系統（IG 輪播）

- Created (3):
  - `wiki/entities/growithfyn.md`（台灣 IG 創作者，2026/4 付費社群 6,008 人）
  - `wiki/concepts/內容創作/LIFE根系內容系統.md`（L/I/F/E 四觸點框架、三步驟啟動、週 SOP、留言漏斗設計）
  - `wiki/sources/growithfyn-life-content-system-ig.md`
- Updated: `wiki/index.md`（237→240 頁）
- Key additions:
  - LIFE = Long-form / Influence / Funnel / Ecosystem，一篇根系長文分發四個觸點，全部匯入同一付費社群入口
  - 三步驟：確立核心 POV → 建立分發系統（一文拆成輪播/Skool/Email）→ 接上付費社群漏斗
  - 留言關鍵字漏斗：留言「LIFE」→ DM 自動回覆免費資源 → Email 暖身 → 付費升級
  - 核心主張：創作者缺的不是努力，而是讓內容複利生長的「系統」

## [2026-04-30] ingest | 電商行銷四大主題（SEO / Email / KOL / 電商SOP）

- Created (8):
  - `wiki/concepts/社群行銷/SEO搜尋引擎優化.md`（四大支柱、E-E-A-T、Core Web Vitals、AISO/GEO/Query Fan-out 2026 趨勢）
  - `wiki/sources/whoops-seo-2026-guide.md`
  - `wiki/concepts/社群行銷/Email行銷與電子報.md`（EDM ROI 1:36、Google 2024 新規 DMARC、行銷自動化序列、ESP 選型）
  - `wiki/sources/newsleopard-email-marketing-2026.md`
  - `wiki/concepts/社群行銷/KOL網紅行銷策略.md`（KOL vs KOC、台灣報價分層、六步合作流程、個人化邀約、奈米 KOC 崛起）
  - `wiki/sources/adnex-kol-koc-2026.md`
  - `wiki/concepts/社群行銷/電商行銷年度SOP.md`（三層活動結構、促銷機制、LINE 推播策略、會員四階段）
  - `wiki/sources/2026-ecommerce-marketing-sop.md`
- Updated: `wiki/index.md`（229→237 頁，新增「社群行銷」概念分類）
- Key additions:
  - GEO（生成式引擎優化）：衡量指標從排名/點擊率 → 被 AI 引用率，2026 新興趨勢
  - Email ROI 1:36 仍是電商留存最高性價比渠道；Google 2024 新規：DMARC 驗證 + 垃圾信率 < 0.1% + 一鍵退訂
  - KOL 市場：奈米 KOC（< 1 萬粉）轉換率往往超越 Mega KOL，品牌預算持續向下移動
  - 電商年度 SOP 核心：三層活動結構 + 滿額門檻技巧 + 會員四階段（招募→活化→分層→回購）

## [2026-04-30] ingest | Boris 示範 CoWork（Startup Ideas Podcast）

- Created (3):
  - `wiki/sources/boris-cowork-startup-ideas-podcast.md`
  - `wiki/entities/Boris.md`（Claude Code 創造者，四大心法：計畫模式/Opus優先/CLAUDE.md Git/自我驗證）
  - `wiki/concepts/claude/CoWork桌面工具指南.md`（CoWork 介紹 + 逆向引導 + 跨App自動化 + Boris四大心法）
- Updated (2):
  - `wiki/concepts/claude/CLAUDE.md撰寫最佳實踐.md`（新增「CLAUDE.md check in 到 Git 全團隊共同維護」章節）
  - `wiki/index.md`（225→229 頁，新增 Boris / 追日Gucci / Anthropic entity 頁 + CoWork 概念頁）
- Key additions:
  - CoWork = Claude Agent SDK 的圖形化桌面介面，非技術用戶可直接使用
  - 逆向引導（Reverse Elicitation）：Claude 不確定時主動問用戶而非自行腦補
  - Boris 爆紅推文四大心法：計畫模式最被低估、Opus 4.5 整體成本反而低、CLAUDE.md check in Git 全團隊維護、給 Claude 自我驗證方式（Chrome 擴充套件）
  - Boris 工作方式：同時跑 5–10 個 Claude，過去兩個月 100% 程式碼由 Claude Code 產出

## [2026-04-30] lint | Wiki 健檢修復（第二次）

- 修復 (6 處 broken link):
  - 建立 `wiki/entities/Anthropic.md`（被 Erik Schluntz entity 引用但無頁面）
  - 建立 `wiki/entities/追日Gucci.md`（gucci-ai-vibe-coding-trading 來源引用的創作者）
  - `[[追日Gucci-AI效率革命聯盟]]` → `[[追日Gucci]]`（修正命名，與 entity 頁對齊）
  - `[[Claude Code in Action]]` → 移除 wikilink（Anthropic 課程名稱，非 wiki 頁面）
  - `[[給AI交底而不是許願]]` → `[[Claude Prompt工程核心技巧]]`（章節標題改引用所在頁）
  - `[[Claude Tool Use 實戰模式]]` → `[[Claude Agent 設計模式]]`（頁面不存在，改指向含相同內容的頁面）

## [2026-04-30] write | 教程系列 14 篇（Python / 面試 / AI開發 / Claude / Go）

- Created (14):
  - **Python** (4):
    - `wiki/concepts/python/Python測試策略.md`（pytest/fixture/mock/AsyncMock/testcontainers/coverage/CI）
    - `wiki/concepts/python/Python FastAPI深度實戰.md`（Pydantic/Depends/async ORM/Middleware/Lifespan/CORS/背景任務）
    - `wiki/concepts/python/Python資料庫整合.md`（SQLAlchemy 2.x Mapped[T]/asyncpg/N+1防禦/Alembic）
    - `wiki/concepts/python/Python套件管理.md`（uv 10-100x加速/pyproject.toml/Poetry/CI/PyPI發布）
  - **面試** (3):
    - `wiki/concepts/interview/Low Level Design OOD設計題型.md`（停車場/電梯/圖書館/聊天室 Python 完整實作）
    - `wiki/concepts/interview/薪資談判與Offer比較框架.md`（TC計算/談判腳本/量化評分表/台灣薪資行情）
    - `wiki/concepts/interview/系統設計現場白板執行技巧.md`（45分鐘時間分配/各階段技巧/Trade-off公式）
  - **AI 開發** (3):
    - `wiki/concepts/ai開發/RAG檢索增強生成實戰.md`（Chunking策略/pgvector/Chroma/Hybrid Search/Reranking/RAGAS）
    - `wiki/concepts/ai開發/LangGraph Agent工作流設計.md`（State/Node/Edge/ReAct/Human-in-the-Loop/Send API）
    - `wiki/concepts/ai開發/Prompt Engineering進階.md`（CoT/Zero-shot/Self-Consistency/動態Few-Shot/ReAct/結構化輸出）
  - **Claude** (2):
    - `wiki/concepts/claude/Claude Code與Git整合工作流.md`（Git整合/CLAUDE.md規範/PR建立/CI/Code Review/安全邊界）
    - `wiki/concepts/claude/Claude Code多人團隊協作指南.md`（分層設定/共享settings.json/Subagent共享/Onboarding清單）
  - **Go** (2):
    - `wiki/concepts/go-backend/Go安全性實踐.md`（JWT/Refresh Token/CORS/令牌桶+Redis限流/SQL Injection/bcrypt）
    - `wiki/concepts/go-backend/Go資料庫選型.md`（GORM vs sqlc vs pgx/sqlc工作流/pgx COPY/選型決策樹）
- Updated: `wiki/index.md`（211→225 頁）
- Key additions: 補齊 Python 完整後端工程知識面（測試/Web/DB/套件管理）；面試新增 LLD 設計題型 + 薪資談判 + 白板執行三篇；AI 開發新增 RAG + LangGraph + Prompt Engineering 三篇進階實戰；Claude 新增 Git 工作流 + 多人協作兩篇；Go 補齊安全性與資料庫選型兩篇

## [2026-04-30] ingest | Cat Wu 訪談：從 6 個月到 1 天的日級發布機器（Lenny's Podcast）

- Created:
  - `wiki/sources/catwu-anthropic-day-level-release-machine.md`
  - `wiki/entities/Cat Wu.md`
  - `wiki/concepts/claude/Anthropic 日級迭代發布機制.md`
  - `wiki/concepts/claude/AI時代PM新角色.md`
  - `wiki/concepts/claude/100%自動化原則與AI杠桿.md`
- Updated: `wiki/index.md`（192→197 頁）
- Key additions:
  - 日級迭代三機制：Research Preview（降低承諾壓力）+ Evergreen 發布室（常駐跨職能待命）+ 清障型 PM（消除阻礙）
  - PM 角色演進：從跨部門協調 → 產品品味定義者 + Evals 建造者 + 模型反思技巧
  - 100% 自動化原則：95% 準確率仍需人工審核 = 未真正自動化；最後 5% 才能釋放杠桿效應
  - CoWork 實戰（1 小時生成 20 頁簡報）、Applied AI 客戶準備工作流
  - 「只管去做」文化、Anthropic 使命如何簡化組織決策

## [2026-04-30] write | Skills 實戰教程系列（2 篇）

- Created:
  - `wiki/concepts/claude/Skills實戰：Threads自動爬文與發文.md`
  - `wiki/concepts/claude/Skills實戰：自動交易機器人.md`
- Updated: `wiki/index.md`（190→192 頁）
- Key additions:
  - Threads 教程：threads-scrape/draft/post 三個 SKILL.md 完整定義；Threads API 兩步發文流程；Playwright MCP 爬取公開頁面；發文雙重確認機制（Skill 層 + Python 腳本 input(yes)）
  - 交易機器人教程：market-scan/risk-check/trade-plan/post-trade 四個 SKILL.md；風控計算邏輯（PASS/WARN/BLOCK）；帳戶 2%/6% 雙層風控；模擬/實盤切換架構；「Claude 是分析師不是交易員」的核心安全原則

## [2026-04-30] ingest | Anthropic Academy 13 堂免費課程（數位時代）

- Created:
  - `wiki/sources/bnext-anthropic-academy-13-courses.md`
  - `wiki/entities/Anthropic Academy.md`
  - `wiki/concepts/claude/Anthropic Academy 13堂課程完整指南.md`
  - `wiki/topics/vibe-coding-skills-mcp-roadmap.md`（Vibe Coding + Skills + MCP 學習路線圖）
- Updated: `wiki/index.md`（186→190 頁）
- Key additions: 13 堂官方課程逐一說明（堂數/時長/先修/適合對象）；4D 框架（Delegate/Describe/Discern/Diligent）；14 週 Vibe Coding + Skills + MCP 完整學習路線圖，含 5 個 Phase 和每週實作目標

## [2026-04-30] write | Claude 教程系列（7 篇）

- Created:
  - `wiki/concepts/claude/Claude Code 入門完整指南.md`
  - `wiki/concepts/claude/CLAUDE.md撰寫最佳實踐.md`
  - `wiki/concepts/claude/Claude Prompt工程核心技巧.md`
  - `wiki/concepts/claude/Claude Code Hooks 深度設定.md`
  - `wiki/concepts/claude/Claude API基礎與最佳實踐.md`
  - `wiki/concepts/claude/Claude Agent 設計模式.md`
  - `wiki/concepts/claude/Claude MCP 伺服器整合指南.md`
- Updated: `wiki/index.md`（179→186 頁）
- Key additions: 完整的 Claude 使用教程體系，涵蓋 Claude Code CLI 操作、CLAUDE.md 規範設定、Prompt 工程技巧、Hooks 自動化、API 整合（含 Prompt Caching）、Agent 設計模式、MCP 伺服器整合

## [2026-04-30] write | Claude 股市與期貨分析實戰

- Created: `wiki/concepts/ai開發/Claude股市期貨分析實戰.md`
- Updated: `wiki/index.md`（178→179 頁）
- Key additions: 兩大使用模式（直接分析 vs Vibe Coding 建工具）的明確邊界；台股/期貨資料來源整理（FinMind、yfinance、CCXT、CFTC）；技術分析/財報/籌碼/COT/未平倉量的 Prompt 模板；Screener、回測框架、基差分析的 Vibe Coding 建工具範例；Claude 分析邊界與限制；Prompt 品質原則

## [2026-04-30] ingest | Vibe Coding 程式交易實戰（追日Gucci）

- Created: `wiki/concepts/ai開發/Vibe Coding程式交易實戰.md`, `wiki/sources/gucci-ai-vibe-coding-trading.md`
- Updated: `wiki/index.md`（177→178 頁）
- Key additions: 程式交易 Vibe Coding 的核心護欄策略——CLAUDE.md 明確禁止/允許區域、strategy/core 目錄分離、Characterization Tests 凍結交易邏輯行為、Backtest 自動驗證工作流、Git CODEOWNERS 強制人工審查；核心心態：AI 是「受控工程師」而非「策略設計師」

## [2026-04-30] write | Go + PostgreSQL 測試策略

- Created: `wiki/concepts/go-backend/Go PostgreSQL測試策略.md`
- Updated: `wiki/index.md`（176→177 頁）
- Key additions: 完整 PostgreSQL 測試分層策略：Mock Repository（Application Layer 純業務邏輯）+ testcontainers-go（真實 DB）+ 事務隔離（newTestTx + t.Cleanup Rollback）+ pgx/sqlc 的 DBTX 介面 + Fixtures 管理（Go 代碼和 SQL 檔案兩種方式）+ Migration up/down 測試 + CI（GitHub Actions service vs testcontainers）+ 常見陷阱（time.Time 精度、並發測試隔離）

## [2026-04-30] ingest | DDD、依賴注入與 Go Wire（3 頁）

- Created Concepts (3):
  - `wiki/concepts/go-backend/DDD領域驅動設計.md`（戰略設計：Ubiquitous Language/Bounded Context/Context Map；戰術設計：Entity/Value Object/Aggregate/Domain Event/Repository/Domain Service/Application Service；DDD 分層架構圖；Go 專案目錄結構；常見面試問題）
  - `wiki/concepts/go-backend/依賴注入與控制反轉.md`（IoC 原理、三種注入方式及選擇、可測試性（mock 注入）、手動 DI 的 Composition Root、SOLID-D 連結、介面設計原則、生命週期管理、常見反模式）
  - `wiki/concepts/go-backend/Go Wire深度實戰.md`（Provider/Injector/ProviderSet 完整範例、wire.Bind 介面綁定、wire.Struct/Value/FieldsOf、cleanup 反向清理機制、完整 DDD+Wire 專案結構、常見錯誤與除錯）
- Created Source Summary (1): `wiki/sources/ddd-di-wire-synthesis.md`
- Updated: `wiki/index.md`（Golang 微服務與後端工程區段新增 3 頁，Source Summaries 新增 1 筆，頁數 172→176）
- Key additions: DDD 從戰略到戰術的完整實作指南（含 Go 代碼）；DI 強調可測試性和 SOLID 連結；Wire 深度涵蓋 ProviderSet 模組化、cleanup 清理、FieldsOf 參數提取等生產環境常用模式

## [2026-04-30] ingest | Python 資深工程師核心知識點合集（6 頁）

- Created Concepts (6):
  - `wiki/concepts/python/Python GIL與並發模型.md`（GIL 機制與引用計數的關係、三種並發模型比較表、threading/multiprocessing/asyncio 實戰、PEP 703 No-GIL 展望）
  - `wiki/concepts/python/Python記憶體管理.md`（引用計數即時回收、分代 GC 循環引用、pymalloc 記憶體池、弱引用 WeakValueDictionary、__slots__ 省記憶體 75%、常見洩漏場景）
  - `wiki/concepts/python/Python非同步程式設計.md`（asyncio event loop 協作式多工、gather/TaskGroup/wait/as_completed 差異、取消與 timeout、asyncio.Queue、run_in_executor 整合同步代碼、常見陷阱）
  - `wiki/concepts/python/Python資料模型與描述器.md`（__new__/__init__/__del__ 生命週期、運算子重載、descriptor protocol 資料/非資料描述器、property、__getattr__ vs __getattribute__、metaclass 與 __init_subclass__）
  - `wiki/concepts/python/Python型別系統與類型提示.md`（TypeVar/Generic、Protocol 結構型別、dataclass + slots、TypedDict、Literal/Annotated、overload、ParamSpec、pydantic 執行時驗證）
  - `wiki/concepts/python/Python裝飾器與Context Manager.md`（decorator factory 三層嵌套、functools.wraps、class-based decorator、contextmanager generator、ExitStack 動態 context、functools 工具集）
- Created Source Summary (1): `wiki/sources/python-senior-knowledge-synthesis.md`
- Updated: `wiki/index.md`（程式語言/Python 子類別填充 6 頁，Source Summaries 新增 1 筆，頁數 165→172）
- Key additions: 涵蓋 Python 資深工程師面試核心——GIL 與並發選擇（最高頻考點）、記憶體管理機制、asyncio 深度、Python 資料模型（描述器/metaclass）、型別系統（Protocol/TypeVar）、裝飾器實戰。每頁含面試常問問題與反直覺陷阱說明。

## [2026-04-30] admin | 重組 index：新增「程式語言」頂層類別

- Updated: `wiki/index.md`
  - 新增 `## 程式語言` 頂層類別（與 Entities / Concepts / Topics 並列）
  - 原 `### Go / Backend 工程` 和 `### Go / Backend 主題綜述` 移入 `## 程式語言 / ### Golang`
  - Golang 概念頁依功能拆成四個子類別：**核心語言機制** / **可觀測性與診斷** / **版本特性** / **微服務與後端工程** / **面試與職涯**
  - Topics 中的 `### Go / Backend 工程` 改名為 `### Golang`
  - 新增 `### Python`（空白佔位，待後續 ingest）
- 頁數不變（165 頁）

## [2026-04-30] ingest | Go 1.22–1.25 新功能實戰（4 頁）

- Created Concepts (4):
  - `wiki/concepts/go-backend/Go1.22新功能實戰.md`（迴圈變數修復、range-over-int、net/http 方法路由與路徑參數、math/rand/v2、sql.Null[T]）
  - `wiki/concepts/go-backend/Go1.23新功能實戰.md`（range-over-func 迭代器完整指南、iter.Seq/Seq2/Pull、鏈式惰性求值、unique 套件 interning、Timer/Ticker Reset 修正）
  - `wiki/concepts/go-backend/Go1.24新功能實戰.md`（泛型型別別名、weak.Pointer GC-friendly cache、os.Root 路徑沙盒、Swiss Table Map 效能、tool 指令取代 tools.go、testing/synctest 實驗性）
  - `wiki/concepts/go-backend/Go1.25新功能實戰.md`（synctest 完善、t.Context()、iter.Concat/slices.Chunk、後量子 ML-KEM TLS、版本特性速查表）
- Created Source Summary (1): `wiki/sources/go-release-notes-1.22-1.25.md`
- Updated: `wiki/index.md`（Go/Backend 概念區段新增 4 頁，Source Summaries 新增 1 筆，頁數 160→165）
- Key additions: 從語言層面（迴圈語意、迭代器協議、泛型別名）到標準庫（iter/unique/weak/os.Root）到工具鏈（tool 指令）到測試（synctest）的完整版本演進實戰指南；每頁含可直接使用的 Go 代碼範例，並標注升級優先度和常見陷阱

## [2026-04-30] ingest | Golang 資深工程師核心知識點合集（7 頁）

- Created Concepts (7):
  - `wiki/concepts/go-backend/Go Context深度解析.md`（Context 樹、四個建構子、WithValue key-type 反模式、context.Cause Go 1.20+、WithoutCancel Go 1.21+）
  - `wiki/concepts/go-backend/Go泛型設計.md`（型別參數語法、Constraint/~底層型別、GCShape 效能模型、泛型工具函數庫、限制與陷阱）
  - `wiki/concepts/go-backend/Go pprof實戰完整指南.md`（6 種 profile 類型、CPU 火焰圖解讀 flat vs cum、Heap -base 差值比對、三種診斷情境流程、Pyroscope 持續 profiling）
  - `wiki/concepts/go-backend/Go記憶體洩漏排查.md`（Goroutine 洩漏 3 場景、無界快取 LRU 修法、unclosed 資源、sub-slice 大 array retain、goleak 測試整合）
  - `wiki/concepts/go-backend/Go同步原語與記憶體模型.md`（Happens-Before 規則表、race detector、Mutex/RWMutex/atomic 選型指南、singleflight 防快取擊穿、errgroup SetLimit）
  - `wiki/concepts/go-backend/Go defer與panic.md`（LIFO 三規則、命名返回值面試題、迴圈 defer 陷阱、recovery middleware with stack trace、跨 goroutine recover 不可行）
  - `wiki/concepts/go-backend/Go測試基準與模糊測試.md`（表格驅動測試、t.Parallel + t.Cleanup、testify assert/require/mock、Benchmark b.ReportAllocs/避免編譯器優化、Fuzz Testing corpus 管理）
- Created Source Summary (1): `wiki/sources/golang-senior-knowledge-synthesis.md`
- Updated: `wiki/index.md`（Go/Backend 概念區段新增 7 頁，Source Summaries 新增 1 筆，頁數 152→160）
- Key additions: 深度涵蓋 Go 資深工程師核心技術——context 生命週期管理、泛型 GCShape 模型、pprof 火焰圖解讀、goroutine 洩漏五大根源、happens-before 記憶體模型、defer 命名返回值互動、Fuzz Testing 自動探索邊界案例。每頁含生產環境實戰代碼與常見陷阱說明。

## [2026-04-29] write | Golang 微服務完整知識庫（16 頁）
- Created Concepts (15):
  - `wiki/concepts/go-backend/微服務架構設計原則.md`（DDD、12-Factor、CAP、BFF）
  - `wiki/concepts/go-backend/gRPC設計與實戰.md`（Protobuf、Streaming、Interceptor、錯誤碼）
  - `wiki/concepts/go-backend/Circuit Breaker熔斷器.md`（三狀態、sony/gobreaker、Fallback）
  - `wiki/concepts/go-backend/Saga Pattern分散式事務.md`（Choreography/Orchestration、Outbox）
  - `wiki/concepts/go-backend/CQRS與Event Sourcing.md`（Command/Query 分離、Event Store、Snapshot）
  - `wiki/concepts/go-backend/事件驅動架構與Kafka.md`（Producer/Consumer、DLQ、冪等去重）
  - `wiki/concepts/go-backend/服務發現與負載均衡.md`（Consul、K8s DNS、Least Connections）
  - `wiki/concepts/go-backend/OpenTelemetry分散式追蹤.md`（Traces/Metrics/Logs、採樣策略）
  - `wiki/concepts/go-backend/Go優雅關機與健康檢查.md`（SIGTERM、三種 Probe、K8s 整合）
  - `wiki/concepts/go-backend/冪等性設計.md`（Idempotency Key、ON CONFLICT、Saga 重試）
  - `wiki/concepts/go-backend/分散式鎖.md`（SETNX+Lua、Redlock、Watchdog 續期）
  - `wiki/concepts/go-backend/Go微服務框架比較.md`（Kratos/go-kit/go-zero/Gin 選型）
  - `wiki/concepts/go-backend/Go依賴注入與Wire.md`（Wire 代碼生成、uber-go/fx lifecycle）
  - `wiki/concepts/go-backend/Go介面設計模式.md`（Repository/Decorator/Strategy/Options）
  - `wiki/concepts/go-backend/Go微服務配置管理.md`（Viper、K8s Secret、Vault）
  - `wiki/concepts/go-backend/Go錯誤處理最佳實踐.md`（Sentinel/Custom/Wrapping、分層 mapping）
- Created Topics (1): `wiki/topics/golang-microservices-guide.md`（含架構決策速查、推薦技術棧、生產部署清單）
- Updated: `wiki/index.md`（Go/Backend 概念區段大幅擴充、Topics 新增微服務總覽）
- Key additions: 涵蓋 Go 微服務開發完整知識面——從 gRPC 通訊協議、Saga 分散式事務到 Circuit Breaker 故障保護、CQRS 讀寫分離、OTel 可觀測性、K8s 優雅關機、分散式鎖、依賴注入。每頁含 Go 代碼實作範例。

## [2026-04-29] write | OpenSpec 產品設計教學 + 實戰範例（電商 + SaaS）
- Created:
  - `wiki/topics/openspec-product-design-guide.md`（主教學）
  - `wiki/topics/openspec-example-ecommerce.md`（電商實戰）
  - `wiki/topics/openspec-example-saas.md`（SaaS 訂閱實戰）
- Updated: `wiki/index.md`（AI 開發區段新增 3 頁）
- Key additions: 完整 OpenSpec 8 階段操作教學（含模板和技巧）；電商範例涵蓋完整的 proposal/design/behavioral-specs/task-checklist；SaaS 範例聚焦訂閱升降級時序、Redis 用量計數防 race condition、Stripe Webhook 冪等性等複雜場景；兩範例末尾附 Archive 模板

## [2026-04-29] write | SD題解：CDN 設計
- Created: `wiki/topics/sd-cdn.md`
- Updated: `wiki/index.md`（入門題新增 CDN）、`wiki/topics/sd-overview.md`（新增 CDN 列、題數 19→23）
- Key additions: Pull vs Push CDN 決策、Anycast BGP vs GeoDNS 路由、三層快取（L1/L2/Origin）、Singleflight 防快取雪崩、主動 Purge API，含完整 Go 實現

## [2026-04-29] ingest | 在生產環境中 Vibe Coding：Anthropic 編程智能體負責人的 4 條黃金法則
- Created: `wiki/sources/schluntz-vibe-coding-production-4rules.md`、`wiki/entities/Erik Schluntz.md`、`wiki/concepts/claude/生產環境Vibe Coding四大策略.md`
- Updated: `wiki/concepts/ai開發/Vibe Coding風險與限制.md`（新增技術/非技術用戶風險差異段落）、`wiki/index.md`
- Key additions: Erik Schluntz（Anthropic MTS）從 22,000 行生產合併案例歸納四大策略——可驗證抽象層、葉節點策略、前置規劃工作流、充分上下文準備；首批頁面正式填充 claude 分類

## [2026-04-29] admin | 新增 claude 分類
- Created: `wiki/concepts/claude/`（新目錄）
- Updated: `wiki/index.md`（新增 Claude 區段）
- Key additions: 在 concepts 下建立 claude 分類，供後續 ingest Claude 相關來源使用

## [2026-04-20] init | Knowledge base initialized

- Created: CLAUDE.md (schema), wiki/index.md, wiki/log.md, sources/README.md
- Directory structure: sources/, wiki/entities/, wiki/concepts/, wiki/topics/, wiki/sources/
- Wiki is empty and ready for first source ingestion

---

## [2026-04-20] ingest | 系統設計題解（19題 Go 代碼實現）

- **新建頁面** (19):
  - 概念題：sd-rate-limiter, sd-consistent-hashing, sd-api-gateway-vs-lb, sd-sso
  - 入門題：sd-url-shortener, sd-distributed-cache, sd-parking-garage
  - 中級題：sd-social-media-feed, sd-chat-system, sd-video-streaming, sd-music-streaming, sd-typeahead, sd-ticketing-system
  - 進階題：sd-uber, sd-web-crawler, sd-dropbox, sd-google-docs
  - 總覽：sd-overview
- **Key additions**: 每題含 RESHADED 分析 + Go 核心代碼 + Trade-off 說明

---

## [2026-04-20] ingest | 系統設計面試資源綜合整理（11篇）

- **來源**: ByteByteGo、Exponent、DesignGurus、SystemDesignHandbook、Hello Interview、interviewing.io 等
- **新建頁面** (6):
  - `wiki/sources/system-design-interview-synthesis.md`
  - `wiki/concepts/系統設計面試框架比較.md`
  - `wiki/concepts/RESHADED面試框架.md`
  - `wiki/concepts/面試時間分配.md`
  - `wiki/concepts/面試評分標準.md`
  - `wiki/concepts/系統設計常見題型.md`
  - `wiki/concepts/系統設計核心技術棧.md`
- **更新頁面**: 系統設計面試模板（加入跨連結）
- **Key additions**: 框架比較（4步/5步/6步/7步/RESHADED）、具體時間分配、50題分類、Redis/Kafka/ES 使用指南、四維評分標準。

---

## [2026-04-20] lint | Wiki 健檢修復

- **修復項目**:
  - 重新命名 3 個 entity 檔案以符合 Obsidian 連結解析（ali-abdaal.md→Ali Abdaal.md 等）
  - 修正 2 處 `[[HIVE 影片結構框架]]`（多餘空格）→ `[[HIVE影片結構框架]]`
  - 新建缺失頁面 5 個：AI伺服器供應鏈、LifeNotes、LifeOS、Part-Time YouTuber Academy
  - 更新 index（28頁）
- **剩餘低優先問題**: NVIDIA、Gaurav Sen (GKCS)、Alex Xu 尚無專頁（各僅1次引用）

---

## [2026-04-20] ingest | UMC H2 2026 晶圓調漲分析

- **來源**: https://www.threads.com/@unclestocknotes/post/DXPnzMdD0V9
- **新建頁面** (5):
  - `wiki/sources/unclestocknotes-umc-price-hike-h2-2026.md`
  - `wiki/entities/umc.md`
  - `wiki/concepts/成熟製程AI需求.md`
  - `wiki/concepts/PMIC供需動態.md`
- **Key additions**: UMC H2 2026 漲價邏輯——AI 伺服器帶動成熟製程元件（PMIC/BMC）需求，疊加 8 吋廠產能萎縮，形成結構性漲價支撐。

---

## [2026-04-20] ingest | A Beginner's Guide to System Design

- **來源**: https://medium.com/@sentalkssane/a-beginners-guide-to-system-design-76d64689788b
- **作者**: Aritra Sen（前 Google L4 → Meta Reality Labs）
- **新建頁面** (7):
  - `wiki/sources/aritra-sen-beginners-guide-system-design.md`
  - `wiki/entities/aritra-sen.md`
  - `wiki/concepts/系統設計面試模板.md`
  - `wiki/concepts/Functional vs Non-functional Requirements.md`
  - `wiki/concepts/Back-of-the-Envelope 估算.md`
  - `wiki/concepts/SQL vs NoSQL 選型框架.md`
  - `wiki/concepts/分散式系統基礎概念.md`
- **Key additions**: 系統設計面試五步準備法，以及六步面試模板（需求→估算→API→高層→DB→詳細選型）。

---

## [2026-04-20] ingest | Ali Abdaal：2026數位內容創作全攻略

- **來源**: https://www.youtube.com/watch?v=116Rio28Enk
- **新建頁面** (8):
  - `wiki/sources/ali-abdaal-2026-content-guide.md`
  - `wiki/entities/ali-abdaal.md`
  - `wiki/concepts/三階段成長框架.md`
  - `wiki/concepts/HIVE影片結構框架.md`
  - `wiki/concepts/利基優先策略.md`
  - `wiki/concepts/內容飛輪.md`
  - `wiki/concepts/批次內容生產.md`
  - `wiki/concepts/常青內容策略.md`
- **Key additions**: Ali Abdaal 的 2026 內容創作策略，核心框架包含三階段成長、HIVE 結構、利基優先、內容飛輪與批次生產。

---

## [2026-04-20] ingest | Z-Image-Turbo 二次元人物 LoRA 訓練經驗完整分享

- **來源**: https://vocus.cc/article/695e5799fd89780001575129
- **作者**: 蘇嘉冠 JiaKuan Su
- **新建頁面** (8):
  - `wiki/sources/lora-training-z-image-turbo.md`
  - `wiki/entities/Z-Image-Turbo.md`
  - `wiki/entities/Ostris AI Toolkit.md`
  - `wiki/entities/Nano Banana Pro.md`
  - `wiki/concepts/LoRA訓練流程.md`
  - `wiki/concepts/Feature Disentanglement.md`
  - `wiki/concepts/訓練資料策略.md`
- **Key additions**: 二次元角色 LoRA 訓練三階段工作流

---

## [2026-04-20] ingest | 股票與期貨入門（7篇文章合成）

- **來源**（7 篇）:
  - https://rich01.com/what-is-futures/（Mr.Market 期貨）
  - https://www.sinotrade.com.tw/richclub/futures/（豐雲學堂台指期）
  - https://www.spf.com.tw/mktinfo/Futures/OA/teach-007.html（永豐期貨5步驟）
  - https://rich01.com/learn-stock-all/（Mr.Market 股票懶人包）
  - https://growingbar.co/ultimate-investment-guide-for-beginner/（Growing Bar 2025）
  - https://startingedu.com/stock-market-101/（啟程教育8大重點）
  - https://school.george-dewi.com/posts/started-with-stocks（慢活學院）
- **新建頁面** (15):
  - Sources (7): futures-intro-mrmarket, taiwan-futures-zoesin, futures-5steps-spf, stock-intro-mrmarket, stock-intro-2025-growingbar, stock-intro-startingedu, stock-intro-georgedewi
  - Concepts (8): 期貨基礎概念, 台指期交易機制, 期貨下單操作, 股票基礎概念, 台股交易機制, 投資策略框架, 基本面分析入門, 股票術語大全
  - Topics (1): investment-primer（投資入門路徑圖）
- **Key additions**: 完整投資入門知識庫，涵蓋期貨（保證金機制、大台小台微台規格、ROD/IOC/FOK 下單）與股票（ETF vs 個股、基本面指標 EPS/P/E/ROE/殖利率、台美股差異）。提供被動/主動/期貨三條學習路線。（初訓→迭代改善→重現）。核心洞察：用 Nano Banana Pro 生成統一畫風訓練資料取代 Danbooru 多畫師資料；Feature Disentanglement 三層標籤（外觀/服裝/情境）防止 concept bleeding；依樣本圖目視選 checkpoint（非 Loss 曲線）；Linear Rank 16 + Timestep Bias Low Noise 為關鍵參數。成本：$1–2 USD / 1–2 小時。

---

## [2026-04-21] lint | Wiki 全面健檢修復

- **修復**:
  1. `index.md` 所有別名 wikilink `\|` → `|`（共 32 處）
  2. 5 個缺失 topic 加入 index.md（sd-ad-click-aggregator, sd-key-value-store, sd-live-streaming, sd-notification-system, sd-yelp）
  3. 建立 7 個缺失 entity 頁面（Alex Xu, ByteByteGo, Gaurav Sen GKCS, Hello Interview, Interviewing.io, NVIDIA, 蘇嘉冠 JiaKuan Su）
  4. index.md 頁面總數更新：72 → 84
  5. 系統設計題解標題更新：19 → 24 題
- **新建頁面** (7): entities/Alex Xu, ByteByteGo, Gaurav Sen (GKCS), Hello Interview, Interviewing.io, NVIDIA, 蘇嘉冠 JiaKuan Su
- **更新頁面** (2): wiki/index.md, wiki/log.md

---

## [2026-04-21] ingest | IG/FB 社群媒體經營策略（4篇文章合成）

- **來源**（4 篇）:
  - https://startingedu.com/instagram-marketing/（啟程教育學院 — 2026 IG行銷8大密技）
  - https://your-planner-jin.com/...（你的文案企劃師 — FB/IG/LINE策略大全）
  - https://shacho.com.tw/facebook-marketing/（頭家製造所 — FB行銷2026）
  - https://www.brainmax-marketing.com/...（布雷米數位媒體 — 2026社群行銷趨勢）
- **新建頁面** (11):
  - Sources (4): ig-marketing-2026-startingedu, social-media-strategy-all-platforms, fb-marketing-2026-shacho, ig-trends-2026-brainmax
  - Concepts (5): IG演算法機制, 社群內容策略, 社群電商變現, 社群廣告投放, 社群KPI指標
  - Topics (1): ig-fb-growth-strategy（IG/FB 經營策略綜合指南）
  - Entities (1): Meta
- **Key additions**: 2026 年 IG/FB 經營策略全面知識庫。

---

## [2026-04-27] ingest | Vibe Coding 教學文章（5篇合成）

- **來源**（5 篇）:
  - https://abmedia.io/vibe-coding-complete-guide-2026（鏈新聞 ABMedia — 2026 完整指南，繁中）
  - https://en.wikipedia.org/wiki/Vibe_coding（Wikipedia — 權威定義與批評，英文）
  - https://simular.co/blog/post/cursor-ai-vibe-coding（夏木樂 — Cursor 操作教學，繁中）
  - https://www.infoq.com/news/2026/02/ai-floods-close-projects/（InfoQ — 開源生態危機，英文）
  - https://airabbi.com/blog/what-is-vibe-coding/（瑞比智慧 — 非工程師向入門，繁中）
- **新建頁面** (14):
  - Sources (5): abmedia-vibe-coding-complete-guide-2026, wikipedia-vibe-coding, simular-cursor-vibe-coding, infoq-ai-vibe-coding-open-source-crisis, airabbi-vibe-coding-intro
  - Concepts (4, 新分類 `ai開發/`): Vibe Coding基礎概念, Vibe Coding工具比較, Vibe Coding風險與限制, Vibe Coding開源生態衝擊
  - Entities (4): Andrej Karpathy, Cursor, Bolt.new, Replit
  - Topics (1): vibe-coding-guide
- **新建分類**: `wiki/concepts/ai開發/`（wiki-organize 判斷：現有 7 個分類無合適歸屬，且此領域預期持續擴充）
- **Key additions**: Vibe Coding 由 Andrej Karpathy 2025-02 提出，2026 年 92% 美國開發者採用。核心洞察：AI 生成程式碼安全漏洞機率是人類的 2.74 倍；有經驗開發者使用 AI 工具反而慢 19%（METR 研究）；「AI Slopageddon」正在破壞開源可持續性（cURL/Ghostty/tldraw 案例）。工具選擇原則：非技術人員用 Bolt.new；有基礎用 Cursor；進階工程師用 Claude Code。

---

## [2026-04-27] refactor | wiki/concepts 按類別分 folder

- **移動檔案** (41 個，無內容修改):
  - `系統設計/` (11)：面試模板、框架比較、常見題型、核心技術棧、時間分配、評分標準、RESHADED、分散式基礎、BoE估算、FnNFR、SQL vs NoSQL
  - `go-backend/` (5)：Go執行期、Go並發、Go效能、Principal框架、行為面試CARL
  - `內容創作/` (6)：三階段成長、HIVE、利基優先、內容飛輪、批次生產、常青內容
  - `社群行銷/` (5)：IG演算法、社群內容策略、社群電商、社群廣告、社群KPI
  - `投資/` (8)：股票基礎、台股交易、投資策略、基本面分析、術語大全、期貨基礎、台指期、期貨下單
  - `半導體/` (3)：成熟製程AI需求、PMIC供需、AI伺服器供應鏈
  - `ai圖像/` (3)：LoRA訓練流程、Feature Disentanglement、訓練資料策略
- **Obsidian 連結不受影響**（`[[Page Name]]` 靠檔名解析，與路徑無關）

---

## [2026-04-27] write | 自媒體行銷漏斗完整指南

- **新建頁面** (1):
  - `wiki/topics/selfmedia-marketing-funnel.md`
- **Key additions**: 整合自媒體經營邏輯與 TOFU/MOFU/BOFU 行銷漏斗框架，涵蓋四層漏斗結構、各平台角色分工、內容飛輪執行方式、六個月實戰路線圖，以及常見錯誤避坑指南。

---

## [2026-04-21] ingest | Golang/Backend Principal Engineer 面試準備（5篇文章合成）

- **來源**（5 篇）:
  - https://www.secondtalent.com/interview-guide/golang/（23 Advanced Golang Backend Interview Questions）
  - https://www.onsites.fyi/blog/article/google-L6-software-engineer-interview-questions（Google L6 Staff Engineer Interview Guide）
  - https://codeforgeek.com/senior-golang-interview-questions/（Senior Golang: Advanced Concurrency & Performance）
  - https://newsletter.eng-leadership.com/p/how-to-nail-big-tech-behavioral-interviews（Behavioral Interviews for Senior Engineers）
  - https://leapcell.io/blog/go-production-performance-tips（Go in Production: 20 Performance Tips）
- **新建頁面** (12):
  - Sources (5): golang-advanced-interview-secondtalent, google-l6-interview-guide, golang-perf-advanced-codeforgeek, behavioral-interview-senior-eng-leadership, go-production-perf-20tips
  - Concepts (5): Go執行期內部機制, Go並發模式, Go效能調優, Principal工程師面試框架, 行為面試CARL法
  - Topics (1): golang-principal-interview（Golang Principal Engineer 面試完整指南）
  - Entities (1): （無新增）
- **Key additions**: Principal/Staff Engineer（L6+）Go 面試完整知識庫。核心洞察：系統設計佔 hiring decision 60%，behavioral 決定 leveling；GMP 排程器（G/M/P 模型）和 GC（GOGC 調優）是 runtime 必考核心；CARL 法（含 Learnings）優於 STAR 法；5 個深度故事 > 20 個淺薄故事；pprof 優先，直覺後；容器環境必用 uber-go/automaxprocs；PGO（Go 1.21+）可帶來 2-7% 免費效能提升。核心洞察：演算法已從 Hashtag 轉向 SEO 關鍵字語意分析，儲存數是最高權重信號；內容黃金配比 40% Carousel + 40% Reels + 20% 單圖；私訊成交率比公開貼文高 5 倍；台灣 FB 用戶覆蓋率達 74%；Meta Advantage+ 為 2026 年推薦廣告工具；IG 互動率基準 0.4%，FB 為 0.05%。

---

## [2026-04-28] ingest | OpenSpec + Superpowers Spec 驅動開發框架

- **Created**:
  - Sources (1): openspec-superpowers-multi-source
  - Entities (2): OpenSpec, Superpowers
  - Concepts (3): Spec驅動開發, OpenSpec工作流, Superpowers技能框架
- **Updated**:
  - Concepts (1): Vibe Coding工具比較（新增框架層段落）
  - Topics (1): vibe-coding-guide（新增第五節：Spec 驅動開發）
- **Key additions**: OpenSpec（Fission-AI）與 Superpowers（obra）是 Vibe Coding 的「成熟化」框架層——OpenSpec 解決需求錯位與技術決策消失（Delta Spec System），Superpowers 透過多 agent 強制 TDD + code review + git worktree 隔離。兩者不自動串聯，需在 CLAUDE.md 明確整合。疊加建議：< 2h 用 Claude Code 單獨；2–8h 加 Superpowers；4–16h 跑 full triple stack。

---

## [2026-04-30] ingest | 快刀青衣：Claude Code 產品負責人說了 9 句大實話

- **Created**:
  - Sources (1): kuaidaoqingyi-catwu-9-truths
- **Updated**:
  - Concepts (2): AI時代PM新角色（新增「新核心技能四：預判模型能力缺口」）, Claude Prompt工程核心技巧（新增「給 AI 交底而不是許願」章節）
- **Key additions**: 快刀青衣對 Cat Wu 訪談的精煉整理，補充兩個原始摘錄未明確命名的核心觀點：(1)「為一個月後的模型做產品」——預判模型能力缺口，提前佈局，新模型一出即領先；(2)「給 AI 交底而不是許願」——提供背景材料、成功定義、限制條件，而非模糊指令。核心洞察：「快，不是因為技術快，是因為決策快；決策快，是因為每個人都清楚方向，不需要等誰來告訴自己該幹什麼。」

---

## [2026-04-30] ingest | Cat Wu Lenny's Podcast 影片版（B 站，中英字幕）

- **Created**:
  - Sources (1): bilibili-catwu-lenny-podcast-video
- **Updated**: 無（內容與已入庫的文字版相同）
- **Key additions**: 登記 B 站影片格式入口（https://www.bilibili.com/video/BV1XiogBoEcX/）。內容與 catwu-anthropic-day-level-release-machine（文字版）及 kuaidaoqingyi-catwu-9-truths（摘要版）為同一訪談，額外價值在於中英雙語字幕與 B 站可直接觀看。

---

## [2026-04-30] write | 三篇新 Claude 教程文章

- **Created**:
  - Concepts (3):
    - Claude Code Subagents 完整指南（內建/自定義 subagent、YAML frontmatter 格式、Explore-Plan-Execute 模式、持久記憶）
    - Claude Code 記憶體系統深度指南（CLAUDE.md 五層架構、.claude/rules/ 路徑範圍規則、Auto Memory 機制、/memory 命令）
    - Vibe Coding 生產部署四階段（稽核→強化→觀察→部署四階段流水線、安全稽核 prompt 模板、結構化日誌、Feature Flags）
- **Sources consulted**: Anthropic 官方文件（code.claude.com/docs/en/sub-agents、code.claude.com/docs/en/memory）、MindStudio、DEV Community、blink.new Vibe Coding 生產指南等
- **Key additions**: (1) Subagents 是 Claude Code 特有的任務委派機制（不同於 API 層的 agent pattern），支援自定義模型、工具限制、持久記憶，YAML frontmatter 格式與存放路徑（.claude/agents/）；(2) 記憶體系統四層架構：Managed Policy → 用戶層 → 專案層 → 本地層 → Auto Memory，`.claude/rules/` 支援 path-scoped rules；(3) Vibe Coding 生產化四階段流水線，特別強調 AI 生成代碼的常見安全盲點（硬編碼 secret、缺失 rate limiting、暴露 internal error details）。

---

## [2026-04-30] ingest | 极海Channel 面試系列影片

- **Sources**: 极海Channel YouTube（@Channel-wq3sw）+ Bilibili（space.bilibili.com/1525355）
  - 前阿里巴巴工程師，専注大廠面試準備、系統設計、AI 時代求職策略
- **Created** (9 頁):
  - Entities (1): 极海Channel
  - Concepts/Interview (7):
    - 大廠技術面試的底層邏輯（三層評估框架、社招 vs 校招差異、STAR 加工程維度）
    - 投簡歷90%的人犯的錯誤（三大投遞錯誤、漏斗意識、投遞順序策略）
    - AI時代新人如何自救（需求結構改變原因、三條生存路徑、6 個月計畫）
    - 面試現場寫Prompt修復Bug（AI 協作新題型、五步流程、三類 Bug 的 Prompt 模板）
    - 校招生如何吊打大廠面試官（3-4 個月刷題路徑、CS 基礎高頻考點、12 週備戰計畫）
    - 12306系統設計面試解題框架（Bitmap 區間表示、Redis Lua Script、反轉索引搶票池）
    - 九分鐘看懂系統設計面試（五步框架、常見題型對照表、展示老手感的語感）
- **Key additions**: 影片「面試現場寫 Prompt 修復 Bug」代表一種新題型——考察候選人能否在壓力下用清晰語言描述問題並評估 AI 輸出，而非只考算法實現。12306 系統設計深度解析：Bitmap 表示區間佔用、Redis Lua Script 原子扣減庫存、以「區間」為 key 的反轉索引——這套方案同時解決超賣/區間座位/查詢性能三大難點。

---

## [2026-05-07] ingest | Claude 2026 完整學習指南（AI 郵報）

- **Source**: https://www.aiposthub.com/claude-tutorial-2026/（AI 郵報，需登入）
- **Created** (8 頁):
  - Sources (1): aiposthub-claude-tutorial-2026
  - Entities (2): AI郵報、Skillsmap
  - Concepts/Claude (5):
    - Claude生態系四種應用（Claude.ai / Claude Code / Cowork / Code Channels 選擇指南）
    - Claude Design完整教學（2026-04 推出，非設計師原型工具，含 vs Figma MCP 比較）
    - Claude for Chrome完整教學（瀏覽器代理，點按鈕/填表，含 vs Gemini in Chrome 比較）
    - Anthropic Managed Agents（雲端目標驅動自動化，取代 n8n 節點拉線）
    - Claude Dispatch遠端控制（2026-03-18，手機→桌電背景執行）
- **Updated** (2 頁): Anthropic（新增 2026 產品線）、index.md
- **Key additions**: 本指南收錄 25+ 篇 AI 郵報 Claude 教學，最關鍵新知：(1) 2026-02 後免費版開放 Skills/Connectors/檔案建立；(2) Claude Design（2026-04）讓非技術人員用自然語言生成 App 原型，定位明確區隔 Figma MCP（給設計師的雙向代碼工作流）；(3) Dispatch + Managed Agents 完成了「任何地方發指令、AI 自動完成」的閉環，前者操作本機，後者跑在雲端。

---

## [2026-05-07] ingest | Claude Code 工程化實戰（黃佳，極客時間）

- **Source**: https://github.com/huangjia2019/claude-code-engineering（577 stars, 211 forks）
- **背景**: 極客時間專欄《Claude Code 工程化实战》配套 repo，2026-01-28 上線，萬人訂閱，總榜第一
- **Created** (7 頁):
  - Sources (1): github-huangjia2019-claude-code-engineering
  - Entities (2): 黃佳 huangjia2019、極客時間 GeekBang
  - Concepts/Claude (4):
    - Claude Code工程化架構與全景（七層可擴展框架全景圖，五種工具原子操作）
    - Claude Code Headless模式與CICD（無人值守、GitHub Actions PR Review、安全考量）
    - Claude Code Rules規則系統（指令規則路徑匹配 + 三級權限 deny/ask/allow）
    - Claude Code Plugins插件系統（Skills/Commands/Hooks 打包分發，manifest 格式）
- **Updated** (1 頁): index.md
- **Key additions**: (1) Rules 系統是本 repo 最完整的新知——指令規則（.claude/rules/*.md 按目錄路徑條件載入）與權限規則（三級評估 deny→ask→allow）是雙軌獨立系統，與 CLAUDE.md 分工明確；(2) Headless 模式讓 Claude Code 嵌入 CI/CD，是「工程化」的關鍵一跳，但需搭配 Rules deny 清單做最後防線；(3) Plugins 是整個課程的終點——從個人 Skill 到團隊可分發的工程能力包，manifest + 分發策略補全了 Skillsmap 生態的供給側邏輯。

---

## [2026-05-07] ingest | IG演算法2026年最新版：6大突破性變化與實戰優化指南

- **Source**: Clippings/IG演算法2026年最新版：6大突破性變化與實戰優化指南.md（ig-hero.com，Chief Strategiest Yen）
- **資料基礎**: Meta 官方公告 + Adam Mosseri Q1 宣告 + 3,500+ 台灣帳號數據 + Hootsuite/Social Media Today
- **Created** (1 頁):
  - Sources (1): ig-hero-ig-algorithm-2026
- **Updated** (1 頁): IG演算法機制（大幅擴寫，從 56 行擴充至 200+ 行）
- **Key additions**: (1) 互動權重大洗牌——分享是按讚的 4.2 倍，1個分享帶來12–15個新觸及，與舊版 wiki「儲存數優先」有衝突，本次以最新數據為準更新；(2) 四大板塊量化權重首次完整記錄（Reels：完播35%/重播25%/分享20%；Feed：關係強度40%/預測互動30%）；(3) 粉絲 B 理論——殭屍粉佔比超過40–50%不只無效，而是主動拉低互動率讓演算法降評整個帳號，這是「買粉真正的危害」的底層邏輯；(4) 原創懲罰機制：30天轉發超10次=限流1個月，且網紅與品牌合作不能發同一影片。

---

## [2026-05-07] ingest | 【深入淺出 Claude Code】運作原理、MCP、Agent Skills、Hooks、Subagents、Plugins 一次搞懂

- **Source**: YouTube `https://www.youtube.com/watch?v=PmlEWW8WMf0`（Code Me Maybe 碼上學）；本地 .webm + TurboScribe .txt 逐字稿（30 分鐘後截斷）
- **Created** (3 頁):
  - Sources (1): youtube-codememayb-claude-code-deep-dive
  - Entities (1): Code Me Maybe 碼上學（台灣 YouTube 頻道）
  - Concepts/Claude (1): Claude Code內部運作機制（Tool Use 循環機制原理）
- **Updated** (3 頁):
  - Claude Code 入門完整指南（新增特殊前綴表、斜線指令完整表、鍵盤快捷鍵表、啟動選項 `claude -c`）
  - CLAUDE.md撰寫最佳實踐（新增四個層級+載入時機對照、子目錄 CLAUDE.md 按需載入的節省 context 設計、`/init` 自動生成說明）
  - Claude MCP 伺服器整合指南（新增三種 scope 說明、設定檔位置、MCP 比喻）
- **Key additions**: (1) Claude Code = Coding Agent 的核心解釋：LLM 本身無法讀檔，Agent 在 prompt 後附加工具說明，讓 LLM 回傳工具呼叫格式，再執行並回傳結果；(2) CLAUDE.md 子目錄版只有 Claude 讀取該目錄時才載入（≠ 根目錄版每次都夾帶），是 monorepo context 節省的關鍵設計；(3) MCP scope 三層（local → `~/.claude.json` / project → `.mcp.json` 可 check in / user 全域），`--scope project` 讓團隊共用 MCP 設定；(4) 斜線指令補完：`/rewind`（雙Esc快捷）、`/context`（查 200k token 細分）、`/task`（背景任務管理）首次完整記錄。

---

## [2026-05-07] ingest | AI Agent 實戰營 + 28天速通LeetCode（s09g / Bojie Li）

- **Source**: GitHub @s09g（Bojie Li，李博杰）
  - https://github.com/s09g/leetcode-fast-pass（28天速通LeetCode）
  - https://github.com/s09g/ai-agent-book-projects（AI Agent 實戰營）
  - 課程頁: https://01.me/2025/08/ai-agent-bootcamp/
- **作者背景**: Chief Scientist at Pine AI；PhD USTC+MSRA；PyTorch contributor；ACM 中國優秀博士學位論文獎
- **Created** (9 頁):
  - Sources (2): s09g-leetcode-fast-pass, s09g-ai-agent-bootcamp
  - Entities (1): Bojie Li (s09g)
  - Concepts/interview (1): LeetCode28日速通課程（28天課表、與其他資源對比）
  - Concepts/ai開發 (3): AI Agent核心架構 Model+Context+Tools、Context Engineering最佳實踐、AI Agent評測基準
- **Updated** (0 頁):（無更新，全為新增）
- **Key additions**: (1) Agent = Model + Context + Tools 統一框架首次完整入庫——三個組成部分的語義定義（Context 是「作業系統」）以及 LLM vs RL 樣本效率 250–400× 的量化；(2) Context Engineering 獨立成頁：KV Cache 友好設計原則（靜態→穩定→動態順序）、Contextual Retrieval 技術（檢索失敗率 -49–67%）、雙層記憶架構；(3) AI Agent 評測基準完整整理：SWE-bench 四版本、GAIA 三難度、Terminal-Bench（Docker 沙盒）、OSWorld/Android World/Tau2-bench 對比表；(4) LeetCode 28 日課程表完整記錄，補充與 NeetCode 150、Blind 75 的對比視角。

## [2026-05-15] ingest | ForceInjection/OpenSpec-practise

- **Source:** https://github.com/ForceInjection/OpenSpec-practise（317⭐，OpenSpec 1.3.0，Apache-2.0）
- **Created (3 頁):**
  - `wiki/sources/github-forceinjection-openspec-practise.md` — 完整來源摘要頁
  - `wiki/concepts/ai開發/OpenSpec文件格式與驗證.md` — 四份文件格式規範 + validate 三類錯誤 + Gherkin 撰寫原則
  - `wiki/concepts/ai開發/OpenSpec三大角色.md` — Context Anchor / Contract Guardian / Collaboration Middleware 完整說明 + 四階段開發週期
- **Updated (1 頁):**
  - `wiki/concepts/ai開發/OpenSpec工作流.md` — 新增 DDD × OpenSpec 映射表 + `/opsx:propose` 一鍵生成說明
- **Key additions:** (1) OpenSpec 文件格式強制規範首次完整入庫——proposal.md 三段結構、spec.md Delta Headers + Gherkin 場景格式、validate 指令的三類錯誤，這是現有 OpenSpec 頁面最大的空白；(2) 三大角色框架提供了「OpenSpec 為何必要」的結構性解釋，Context Anchor 角色與 CLAUDE.md 的關係首次明確連結；(3) DDD-to-OpenSpec 映射表填補了現有 DDD 與 OpenSpec 兩個概念群之間的橋接缺口。

## [2026-05-15] ingest | The Complete Guide to System Design in 2026: AI-Native and Serverless

- **Source:** https://dev.to/devin-rosario/the-complete-guide-to-system-design-in-2026-ai-native-and-serverless-1kpb
- **Created (6 頁):**
  - `wiki/sources/dev-to-system-design-2026.md` — 完整來源摘要頁（含 contradictions、open questions）
  - `wiki/concepts/系統設計/AI-Native架構.md` — AI 嵌入請求路徑、DSLM、Feature Store、持續訓練回饋迴圈
  - `wiki/concepts/系統設計/Serverless-First架構.md` — Stateful Serverless 四種模式、邊緣運算、冷啟動處理、適用邊界
  - `wiki/concepts/系統設計/Data Mesh與Lakehouse.md` — 四大原則、Data Contracts、Delta Lake/Iceberg 技術棧
  - `wiki/concepts/系統設計/FinOps與GreenOps.md` — Right-sizing、Carbon-aware Scheduling、GreenOps 三模式
  - `wiki/concepts/系統設計/Observability 3.0 Causal Tracing.md` — Agentic 因果追蹤架構、OTel AI 擴展、實作建議
- **Updated (0 頁):** 無（現有系統設計頁面為面試導向，新頁面為 2026 生產架構趨勢，以交叉連結整合）
- **Key additions:** (1) 四大架構支柱完整入庫，首次在 wiki 中建立 2026 生產架構的系統性框架；(2) Observability 3.0 填補了現有 OTel 頁面對 AI 系統可觀測性的空白，定義了 Causal Tracing 的 Span 邊界（Thought/Action/Observation）；(3) Data Mesh 與 Lakehouse 作為獨立概念入庫，補充現有系統設計技術棧中缺失的資料治理層；(4) FinOps/GreenOps 首次出現，將成本與碳排放納入設計約束而非事後考量。

## [2026-05-15] ingest | 一部影片看完 Stanford AI 系統課程，從 LLM 到 Agentic Workflow

- **Source:** https://www.youtube.com/watch?v=eKW9ITaltWw（繁中摘要 Stanford Agentic AI Webinar）
- **Created (3 頁):**
  - `wiki/sources/stanford-agentic-ai-youtube-summary.md` — 完整來源摘要頁
  - `wiki/concepts/ai開發/LLM限制與解決方案.md` — 五大限制 × 解決方案對照表 + Base LLM → Agentic 進化路徑
  - `wiki/concepts/ai開發/ReAct Pattern.md` — Thought→Action→Observation 循環；Reasoning+Acting 橋樑概念
- **Updated (0 頁):** 無（現有 RAG / Prompt Engineering / AI Agent 頁面已覆蓋重疊部分，新頁面以交叉連結整合）
- **Key additions:** (1) LLM 限制的系統性分類框架首次入庫，五大限制逐一對應解決方案，提供「何時用 RAG vs Fine-Tuning vs Tool Use」的決策視角；(2) ReAct Pattern 獨立成頁，明確定義 Thought/Action/Observation 三元素的循環結構，補充現有 Prompt Engineering 頁面對 ReAct 的片段描述；(3) Base LLM → Prompting → RAG → Tool Use → ReAct → Single Agent → Multi-agent 的完整進化路徑圖，提供 Agentic AI 學習的心智模型。

## [2026-05-18] lint | Wiki 健檢

- **已修復：**
  - `wiki/concepts/被看見vs被糾正.md` → `wiki/concepts/心理學/` （孤立在 concepts 根目錄）
  - `wiki/concepts/SQL核心概念.md` → `wiki/concepts/SQL/` （新建 SQL/ 資料夾；index 已有對應章節）
  - `wiki/concepts/Golang/Gin_Clean_Architecture.md` → `wiki/concepts/go-backend/` （Golang/ 資料夾僅剩 1 檔，與 go-backend/ 重疊；空資料夾已移除）
  - `wiki/index.md` 頁數更新：340 → 336；日期更新：2026-05-15 → 2026-05-18
- **待補 source 頁（無對應 wiki/sources/*.md，已在 concept/index 中引用）：**
  - `jihaichannel-interview-videos` — 被 极海Channel entity + 7 個 interview/ 概念頁引用，但 source 摘要頁缺失
  - `coinbase-codesignal-ha-guide` — 被 Coinbase_HA_總覽 / Banking_System / In_Memory_Database 引用，source 摘要頁缺失
- **無問題：** 所有頁面均有入站連結（無孤兒頁）；未發現明顯內容矛盾

## [2026-05-18] ingest | 《活出你的本來面目》讀後心得（閱讀前哨站·瓦基）

- **Created（10頁）:**
  - `wiki/sources/readingoutpost-be-true-be-you.md` — 來源摘要，含矛盾與待追問題
  - `wiki/entities/閱讀前哨站.md` — 瓦基的台灣書評平台
  - `wiki/entities/瓦基.md` — 前台積電、現全職創作者
  - `wiki/entities/鐘穎.md` — 書籍作者、榮格心理學諮詢師
  - `wiki/concepts/心理學/榮格陰影理論.md` — 陰影形成、投射識別、整合實踐（深挖）
  - `wiki/concepts/心理學/心理補償機制.md` — 愛↔權力補償動態、三種關係場域（深挖）
  - `wiki/concepts/心理學/人格面具.md` — Persona 概念、面具與陰影的互補結構
  - `wiki/concepts/心理學/個人神話與代價.md` — 遺憾作為意義基石
  - `wiki/concepts/心理學/個體化歷程.md` — 榮格個體化四階段
- **Updated（1頁）:**
  - `wiki/concepts/心理學/被看見vs被糾正.md` — 新增與榮格三頁的交叉連結
- **Key additions:** (1) 榮格陰影理論首次入庫，重點補充「投射」識別機制與整合實踐路徑，並以瓦基案例錨定；(2) 心理補償機制建立愛↔權力對照表，含三種關係場域（親密/親子/職場）的具體呈現，是現有煤氣燈操縱法的理論底層補充；(3) 五頁榮格概念形成完整體系（陰影→面具→補償→神話→個體化），與既有心理學頁面交叉串聯。

## [2026-05-18] ingest | 榮格陰影理論 × 3 來源深化

- **Created（3頁）:**
  - `wiki/sources/leslieinnerjourney-shadow-projection.md` — 投射四大心理機制、正面陰影、轉化案例
  - `wiki/sources/soler-shadow-work-4steps.md` — 陰影三種起源、Shadow Work 四步驟、能量轉化表
  - `wiki/sources/mbalib-jung-shadow-theory.md` — 1912起源、1917原著定義、心靈四層結構、同性投射
- **Updated（1頁）:**
  - `wiki/concepts/心理學/榮格陰影理論.md` — 大幅擴充：加入歷史起源段、心靈四層結構表、陰影三種成因分類、投射四機制、同性投射說明、Shadow Work 完整四步驟、陰影能量轉化對照表；來源由 1 篇擴至 4 篇
- **Key additions:** (1) 補上榮格原著時間線（1912/1917），奠定學術基底；(2) 「投射四機制」精確化投射的觸發條件，含正面陰影（嫉妒=壓抑的渴望）這一常被忽略的面向；(3) Shadow Work 實踐框架從「覺察」到「能量轉化」全面落地，配合對照表可直接操作。

## [2026-05-18] ingest | GitHub Spec Kit 中文教程（寫點東西吧）

- **Created（2頁）:**
  - `wiki/sources/xiediandongxiba-github-spec-kit-sdd.md` — 來源摘要，含三步工作流、項目憲法、效率數據、適用邊界
  - `wiki/entities/GitHub-spec-kit.md` — GitHub 官方 SDD 工具實體頁
- **Updated（1頁）:**
  - `wiki/concepts/ai開發/Spec驅動開發.md` — 加入 GitHub spec-kit vs OpenSpec 對照表、spec-kit 文件格式、規範寫作三條鐵律、SDD 適用邊界分類表；框架比較表新增 spec-kit 列
- **Key additions:** (1) GitHub spec-kit 首次入庫，補上 OpenSpec 之外的 SDD 官方工具視角；(2) spec-kit vs OpenSpec 對照表直接對齊兩者在工作流步驟、核心文件、決策追溯、適用邊界等八個維度的差異，提供選型依據；(3) 規範寫作三條鐵律（WHAT&WHY / [需要澄清] / 可量化驗收）是本次新增的實操知識點。

## [2026-05-18] ingest | SDD 適用邊界深挖 × 3 來源

- **Created（4頁）:**
  - `wiki/sources/isoform-limits-sdd.md` — 四大失敗模式，Context Engineering 替代方案
  - `wiki/sources/marmelab-sdd-waterfall.md` — 七大問題，SDD=AI版瀑布，NLD 替代方案
  - `wiki/sources/williamyeh-genai-se-level-up.md` — 瓶頸轉移論，規格有效/失效條件，測試治理
  - `wiki/concepts/ai開發/SDD適用邊界.md` — 跨四來源合成的深挖概念頁
- **Updated（2頁）:**
  - `wiki/concepts/ai開發/Spec驅動開發.md` — 加入 SDD適用邊界 連結
  - `wiki/index.md` — 新增 SDD適用邊界 概念條目與三篇 source 摘要
- **Key additions:** (1) 四大系統性失敗模式（維護稅/缺失context/虛假完整感/錯誤抽象）首次完整入庫；(2) SDD=瀑布類比（Marmelab）提供最強的批評框架——「規劃無法消除本質複雜性」；(3) William Yeh 的「瓶頸轉移」論點是本次最具原創性的視角：即使 SDD 不適用，工程紀律仍以測試/可理解性/ADR 等形式存續；(4) 決策矩陣和規格失效警訊使概念頁可直接操作。

## [2026-05-18] ingest | SDD 優質資源 × 7 來源

- **Created（11頁）:**
  - `wiki/sources/addyosmani-good-spec-ai-agents.md` — Addy Osmani：六大結構區域、三層邊界
  - `wiki/sources/devtldrlss-ears-bdd-sdd.md` — EARS 五句型、BDD 三部曲、對照表
  - `wiki/sources/thoughtworks-sdd-unpacking-2025.md` — Thoughtworks：SDD≠瀑布的機構立場
  - `wiki/sources/cashwu-sdd-from-tdd.md` — Cash Wu：SDD×TDD 整合閉環
  - `wiki/sources/kaochenlong-sdd.md` — 高見龍：成熟度三層次、工具比較
  - `wiki/sources/jimmysong-sdd-overview.md` — Jimmy Song：協議棧、準確率標準、8工具全景
  - `wiki/sources/milkmidi-github-spec-kit.md` — Milk Midi：spec-kit 六步實戰、/clarify 指令
  - `wiki/concepts/ai開發/EARS需求語法.md` — 五種句型、與 BDD/User Story 的對照、AI 開發的精準化價值
  - `wiki/concepts/ai開發/BDD行為驅動開發.md` — Gherkin 三部曲、SDD×TDD 整合公式、vs EARS 對照
  - `wiki/concepts/ai開發/好規格寫作原則.md` — 六大結構區域、三層邊界、三條鐵律、反模式
  - `wiki/concepts/ai開發/SDD成熟度三層次.md` — Spec-first/anchored/as-source 三層定義與升級時機
- **Updated（2頁）:**
  - `wiki/concepts/ai開發/Spec驅動開發.md` — 加入 4 個新概念頁連結
  - `wiki/concepts/ai開發/SDD適用邊界.md` — 加入 Thoughtworks vs Marmelab 的正面衝突對照
- **Key additions:** (1) EARS 和 BDD 首次入庫，補上「如何寫出精確規格」的語法工具層；(2) SDD 成熟度三層次（Spec-first/anchored/as-source）提供組織評估自身 SDD 實踐的框架；(3) 好規格寫作原則整合 Addy Osmani 六大結構區域 + Thoughtworks 四標準 + spec-kit 三鐵律，是 SDD 知識體系中最具操作性的頁面；(4) Thoughtworks vs Marmelab「SDD是否等於瀑布」的正面衝突首次並陳，提供更完整的視角。

## [2026-05-18] ingest | 榮格陰影理論——4 個高質量網路來源
- **Created（4頁 sources）:**
  - `wiki/sources/thesap-jungian-shadow.md` — SAP（英國榮格分析學會）學術觀點：Trickster 原型、組織陰影、榮格自身陰影
  - `wiki/sources/rafaelkruger-definitive-shadow-work.md` — 意識態度機制、情結作為傀儡操縱者、整合的倫理義務
  - `wiki/sources/jungianvault-shadow-work-guide.md` — 投射「20%→100%」量化框架、陰影黃金、五步整合法
  - `wiki/sources/asisiam-shadow-multilevel.md` — 黑色/白色陰影區分、多層級陰影（個人→家庭→群體→國家）、古比奧村野狼寓言
- **Updated（1頁 concepts）:**
  - `wiki/concepts/心理學/榮格陰影理論.md` — 大幅擴充：新增意識態度機制、黑色/白色陰影、家庭禁止訊息、多層級陰影、投射鉤子（20%→100%）、情結機制、陰影黃金、古比奧野狼寓言、Trickster 原型、整合的倫理義務（行動義務）
- **Key additions:** (1) 黑色陰影 vs 白色陰影——高尚者的陰影是未整合的本能，道德缺失者的陰影是未整合的善，這個對稱讓陰影理論更立體；(2) 多層級陰影（個人/家庭/群體/國家）首次入庫，補上集體心理的維度；(3) 整合的「倫理義務」強調光靠 Shadow Work 反思不夠，必須轉化為行動；(4) 投射「20%→100%」提供可操作的量化隱喻；(5) 古比奧村野狼寓言是最清晰的陰影整合隱喻。sources 來源數量從 4 擴充到 8。

## [2026-05-26] ingest | SDD 規格驅動開發（高見龍）— 補充實體頁

- Source previously ingested: 2026-05-18（kaochenlong-sdd）
- Created: [[高見龍]]、[[Kiro]]、[[Tessl]]
- Updated: wiki/index.md（+3 entities，頁數 405→408）
- Key additions: 補建三個在原始 ingest 中遺漏的實體頁——高見龍（作者）、Kiro（Amazon Spec-anchored IDE）、Tessl（Spec-as-source 框架）

## [2026-05-26] ingest | OpenSpec 讓 SDD 變簡單的三個指令（高見龍）

- Created: [[kaochenlong-openspec]] 來源摘要
- Updated: [[OpenSpec文件格式與驗證]]（+AGENTS.md 目錄結構、+CLI 指令完整表、+何時不需要 Proposal 五條清單）、[[高見龍]]（+第二篇文章記錄）
- Updated: wiki/index.md（+1 source summary，頁數 408→409）
- Key additions: 補充「何時不需要 Proposal」決策清單（繞過 SDD 的合法情境）；釐清 AGENTS.md/project.md 與 config.yaml 的關係；高見龍 Brownfield-first 實踐視角

## [2026-05-26] ingest | OpenSpec 讓 SDD 變簡單的三個指令（高見龍）— 補充 topic 頁

- Updated: [[openspec-product-design-guide]]（+專案目錄結構 AGENTS.md/project.md；+何時不需要 Proposal 五條清單；sources 加入 kaochenlong-openspec）
- No new pages created（topic 頁更新，頁數不變）

## [2026-05-26] ingest | Spectra：給 OpenSpec 的圖形介面（高見龍）

- Created: [[kaochenlong-spectra-openspec]] 來源摘要、[[Spectra]] 實體頁
- Updated: [[OpenSpec]]（+v1.0 主要變化節、+Spectra 相關連結）、[[OpenSpec工作流]]（+v1.0 新指令系統完整節：非線性工作流/新指令速查/DAG 依賴/即時進度追蹤）、[[OpenSpec文件格式與驗證]]（解除 config.yaml vs project.md 矛盾注記，確認 config.yaml 為 v1.0 正式命名）
- Updated: wiki/index.md（+1 entity +1 source，頁數 409→411）
- Key additions: OpenSpec 1.0 架構轉向非線性工作流（DAG 依賴）；/opsx: 指令前綴與 /opsx:ff 快進指令；Spectra 桌面 GUI 完整功能集；config.yaml 命名爭議已確認解決

## [2026-05-26] ingest | OpenSpec：AI 時代的輕量文件管理（Scott Hsiao／方格子）

- Created: [[scott-hsiao-openspec-vocus]] 來源摘要、[[BMAD]] 實體頁
- Updated: [[Spec驅動開發]]（+三工具規模定位速查表 BMAD/Spec-kit/OpenSpec；+製造業 BOM/ECR/ECO/ECN 類比節；完整工具清單加入 BMAD；相關頁面加入 BMAD）
- Updated: wiki/index.md（+1 entity +1 source，頁數 411→413）
- Key additions: SDD 工具三層規模定位（大型組織/0→1/1→100）；製造業工程變更管理類比為 SDD 提供直覺解釋；BMAD 框架完整建檔

## [2026-06-12] ingest | 薛南：黃山料社群經營全分析（Facebook）

- Created: [[xuenan-huangshanliao-brand-analysis]] 來源摘要、[[黃山料]]、[[薛南]] 實體頁、[[無雜質社群經營]]、[[個人品牌宗教性]]（內容創作/）、[[平台分工策略]]（社群行銷/）
- Updated: [[社群內容策略]]、[[匱乏感投射]]、[[祛魅]]、[[ig-fb-growth-strategy]]、[[selfmedia-marketing-funnel]]（相關頁面加入回鏈）
- Updated: wiki/index.md（+2 entities +3 concepts +1 source，頁數 420→426）
- Key additions: 「黃山料不是作家，是品牌」案例完整入庫——無雜質經營、三平台刻意分工（Threads 只巡不發）、邪教教主五條件框架；並記錄與既有「人味內容」主張及 28–40 秒短影片通則的兩處張力。

## [2026-06-12] organize | 黃山料分析三個新概念頁分類

- Created: 無雜質社群經營.md → 內容創作/ — 創作者內容策略框架，與 LIFE根系/常青內容同類
- Created: 個人品牌宗教性.md → 內容創作/ — 主用途為品牌經營方法論；心理機制以回鏈連至 心理學/匱乏感投射、祛魅（close call，未拆分）
- Created: 平台分工策略.md → 社群行銷/ — 平台層操作策略，與 IG演算法機制、社群內容策略同棚
- No new category created

## [2026-06-12] ingest | 黃山料品牌分析九篇（多米多羅事件分析潮 + 一件襯衫早期專訪）

- Created（9 來源摘要）: [[awei-emotional-commodity-market-logic]]（阿唯/情感商品論）、[[udn-publishing-insider-huangshanliao-value]]（前出版人/供應鏈論）、[[rapunzel-80wan-backlash-publishing-economics]]（救命稻草論）、[[chiukaun-huangshanliao-data-analysis]]（周加恩/服務業論，2024）、[[tnl-huangshanliao-publishing-dilemma]]（關鍵評論網/銷量品質混淆，原站403以報導片段整理）、[[newtalk-dk-huangshanliao-traffic-formula]]（DK四點流量密碼+三爭議）、[[domidoro-worst-books-video]]（引爆點影片）、[[meet-bnext-the-shirts-8seconds]]、[[marketersgo-the-shirts-renwei]]（2019一件襯衫時期）
- Created（2 實體 + 1 概念）: [[多米多羅]]、[[一件襯衫]]、[[情感商品]]（內容創作/）
- Updated: [[黃山料]]（大幅擴充：品牌史時間線2017–2026、DK流量密碼、爭議履歷、量尺之爭）、[[薛南]]（補事件脈絡）、[[個人品牌宗教性]]、[[無雜質社群經營]]（+情感商品回鏈）
- Updated: wiki/index.md（+2 entities +1 concept +9 sources，頁數 426→438）
- Key additions: (1) 情感商品框架——量尺錯置（文學vs商品）是整場爭論失焦的根源，阿唯與周加恩相隔20個月獨立提出同構概念；(2) 黃山料品牌史完整時間線入庫，一件襯衫時期的8秒公式與白襯衫識別是「創作者即作品」的起點；(3) 薛南「Threads易燃物」判斷獲事件級驗證（80萬人炎上正發生在Threads）；(4) 三方對立保留為活張力：產業病灶論vs救命稻草論vs供應鏈論。

## [2026-06-12] ingest | 黃山料沒料？（下）：AI戳破品質幻覺（關鍵評論網）

- Created: [[tnl-huangshanliao-ai-quality-illusion]] 來源摘要（原站403，以搜尋片段整理）
- Updated: [[tnl-huangshanliao-publishing-dilemma]]（（下）篇待抓問題已結案）、[[多米多羅]]（觀看數更新：6天188萬；三采文化法律動作）、[[黃山料]]（時間線補：雙軌回應——溫柔人設+法務蒐證）
- Updated: wiki/index.md（+1 source，頁數 438→439）
- Key additions: (1) 「品質幻覺」概念——演算法推送+排行榜從眾+銷量堆疊製造品質假象，AI是照妖鏡不是兇手（開燈比喻）；(2) 與阿唯情感商品論的張力：換量尺 vs 找回量尺；(3) 新追問：AI讓情感商品文字工序可複製後，護城河只剩人格層/宗教性。

## [2026-06-12] ingest | 關鍵評論網黃山料相關文章全面盤點（3篇收錄、1篇排除）

- 盤點範圍: thenewslens.com 站內全部黃山料相關文章（6篇相關，2篇已先入庫）
- Created: [[tnl-domidoro-pure-vs-popular-literature]]（純文學vs大眾文學百年歧視鏈）、[[tnl-huangshanliao-dv-remarks-backlash]]（2024家暴論炎上+共犯結構論）、[[tnl-huangshanliao-dv-universe-bl]]（家暴宇宙文本分析，本collection唯一進入文本內部的批評）
- 排除: feature/chiayiyouth/136767（一件襯衫Ｘ種種影像，2020嘉義青年專題）——業配性質訪談，資訊已被一件襯衫實體頁覆蓋
- Updated: [[黃山料]]（2024-06時間線補吉隆坡原句與家暴宇宙框架；產品設計補28字/行與三秒法則；sources 14篇）、[[情感商品]]（+產品安全段：需求真實≠供給無害）
- Updated: wiki/index.md（+3 sources，頁數 439→442）
- Key additions: (1) 受眾擴張論——黃山料開發的是「原本不買書」的新客群，非搶純文學讀者；(2) 共犯結構論——讀者/演算法/市場共同把自承「文字造詣不足」者推上排行榜第一；(3) 家暴宇宙——爭議發言與作品世界觀互為表裡，對「理性行動者說」構成最強文本挑戰；(4) 同站立場不統一：（上）篇病灶論 vs 歧視鏈篇文人相輕論。後續候選：盧郁佳 Voicettank 家暴宇宙系列。

## [2026-06-12] ingest | 盧郁佳：黃山料家暴宇宙——和平越無底線，戰爭風險越高（思想坦克）

- Created: [[luyujia-dv-universe-peace-war-risk]] 來源摘要（Voicettank 原站可正常抓取，全文整理）、[[盧郁佳]] 實體頁
- Updated: [[tnl-huangshanliao-dv-universe-bl]]（待追問題結案：經查為單篇非系列）、[[黃山料]]（sources 15篇）
- Updated: wiki/index.md（+1 entity +1 source，頁數 442→444）
- Key additions: (1) 綏靖邏輯——「和平越無底線，戰爭風險越高」，無底線容忍提高暴力升級風險，直接反駁「平衡點」說；(2) 病理代際複製——把扛得住虐待內化為堅強是受虐兒因應模式的成人複製；(3) 外部壓迫（恐同）不能正當化對內殘忍（情感利用）；(4) 與關鍵評論網家暴宇宙文互相獨立印證「發言＝作品世界觀」。

## [2026-06-16] ingest | LLMs Reproduce Human Purchase Intent via Semantic Similarity (SSR)

- Source: arXiv:2510.08338（PyMC Labs × Colgate-Palmolive；Maier, Aslak, Wiecki et al.；2025-10）；經 BAAI Hub 連結發現
- Created（1 來源摘要）: [[maier-ssr-llm-purchase-intent]]
- Created（2 概念，ai開發/）: [[語意相似度評分SSR]]（SSR 方法：錨句+embedding→Likert 機率分布）、[[合成消費者調查模擬]]（LLM 作為合成受訪者的框架）
- Created（2 實體）: [[PyMC Labs]]、[[Thomas Wiecki]]
- Updated: [[LLM限制與解決方案]]（+SSR 回鏈：數值分布失真案例）、[[AI Agent評測基準]]（+SSR/合成消費者回鏈：另一種對齊人類的評測）
- Updated: wiki/index.md（+2 entities +2 concepts +1 source，頁數 444→449）
- Key additions: (1) 核心洞見——直接叫 LLM 打 Likert 分數會塌向中間值、分布失真（KS≈0.26），SSR 改用自由文字+embedding 映射成機率分布，達 ~90% 人類 test-retest 信度且 KS>0.85；(2) 方法階梯 DLR(80%)→FLR(85-90%)→SSR(90-92%)，躍升來自映射到分布而非單一整數；(3) 「排序對 ≠ 分布像」——合成面板須同時滿足相關性與分布真實度兩個指標；(4) 已在 57 份個人護理調查/9,300 名真人上驗證，但性別/地區細分群複製不佳、無法反映預算與行銷曝光等真實約束；(5) 官方開源 github.com/pymc-labs/semantic-similarity-rating。

## [2026-06-22] ingest | 數據庫面試簡答、30道高頻面試題（赐我白日夢 / cnblogs）

- Source: https://www.cnblogs.com/ZhuChangwu/p/14238621.html （中文後端面試八股文：MySQL 約21題 + Redis 約11題）
- Created（1 來源摘要）: [[zhuchangwu-database-interview-30-questions]]
- Created（5 概念）: [[MySQL索引與B+樹]]、[[MySQL事務與隔離級別]]、[[資料庫正規化]]（→ concepts/SQL/）；[[Redis快取三大問題]]、[[Redis核心機制]]（→ 新建 concepts/redis/）
- Created（1 實體）: [[赐我白日夢（古法博客）]]
- Updated: [[SQL核心概念]]（+3 回鏈至 MySQL/正規化頁）；wiki/index.md（+1 entity、+3 SQL 概念、新增「Redis / 快取」分類含 2 頁、+1 source，頁數 449→456）
- Key additions: (1) B+ 樹勝出關鍵——非葉節點不存資料故矮胖、3 層約 17 億筆、葉節點鏈結支援範圍查詢，對照 Hash（無範圍）/B 樹/紅黑樹（I/O 多）；(2) InnoDB 預設 Repeatable Read，並以 MVCC + Gap Lock 在 RR 級別就解決幻讀（非標準 SQL 需 Serializable）；(3) 快取三大問題速記——雪崩（大量同時過期，隨機過期+高可用）/穿透（查不存在，布隆過濾器+快取空值）/擊穿（單熱點過期，互斥鎖+永不過期）；(4) Redis 快與單執行緒理由：瓶頸在記憶體頻寬非 CPU；(5) 快取一致性採 Cache Aside「先更新 DB 再刪快取」，最終一致性。後續可補：RDB/AOF 細節、RedLock 爭議。

## [2026-06-29] ingest | codejunkie99/agentic-harness（Rust 原生 Agent Harness）

- Source: https://github.com/codejunkie99/agentic-harness （Apache-2.0；Rust 99.8%；v0.1.1 2026-05-12；~83⭐）
- Created（1 來源摘要）: [[github-codejunkie99-agentic-harness]]
- Created（1 實體）: [[codejunkie99]]
- Updated: [[Agent Harness實作]]（+對比列、新增「語言路線對比：TypeScript vs Rust」表，sources +1）
- Updated: wiki/index.md（+1 entity、+1 source summary，頁數 474→476）
- Key additions: (1) Rust 原生 Agent 執行時/SDK/CLI——「寫一次、編一個 binary，跑遍 laptop/CI/遠端 Linux 沙盒/Cloudflare 邊緣」；四賣點 one toolchain（Cargo）/one artifact/typed end-to-end/easy to test；(2) 八個核心抽象 AgentApp/Session/Task/Role/Skill/SessionEnv/ModelClient/Connector，agent 身分用 URL 路徑 POST /agents/<name>/<id> 維持 session 連續；(3) 內建 MCP 掛載（McpServerOptions）、schema-guided 輸出（---RESULT_START/END--- 抽 typed JSON）、自動 session compaction 與持久化、平行 Task；(4) 與 [[alienzhou]] 的 Zero2Agent（TypeScript）形成 Agent Harness 落地語言對照組——TS 重生態/迭代/教學透明 vs Rust 重可攜 binary/編譯期型別安全/成品框架。後續可追：等價設計決策文件、Cloudflare 邊緣對 agent 工作負載的實際限制。

## [2026-06-29] ingest | topgoer 面試題：Go 中文文檔・106 天 Go 陷阱題

- Source: http://mian.topgoer.com/ （Go 語言中文文檔之面試題子站；GitBook；轉自公眾號「Golang 來啦」；HTTPS 憑證過期需用 http 抓取）
- Created（1 來源摘要）: [[topgoer-go-interview-106days]]
- Created（1 實體）: [[topgoer]]
- Created（1 概念，go-backend/）: [[Go面試陷阱題彙整]]
- Updated: [[Go defer與panic]]（+回鏈至陷阱題彙整）；wiki/index.md（+1 entity、+1 Golang 核心語言機制概念、+1 source summary，頁數 476→479）
- Key additions: (1) 體裁＝「代碼輸出陷阱題」（給 code 問輸出/能否編譯），106 天鬆散排列，第一天為 HR 行為題、技術題從第二天起；(2) 萃取五大高頻 pattern——for-range 變數捕獲（&val 全指最後值）、具名型別可賦值性（底層型別同仍不可互賦，須至少一方非 named type）、三索引切片 a[low:high:max] 控容量、defer×recover×命名返回值「三步拆解」(f(3)=7)、嵌入 *T 零值 nil 解引用 panic；(3) ⚠️ 版本陷阱——Go 1.22 已改 for-range 迴圈變數為每輪新建，第二天經典題在新版答案改變，做舊題庫須先確認 Go 版本；(4) 與既有 Go 後端頁分工：系統化知識（[[Go defer與panic]]/[[Go執行期內部機制]]）vs 碎片化自測；與實作題（[[Coinbase面試題與Go解答]]）補的是語言細節/編譯器行為維度。決定不逐天入庫（重複冗長），以陷阱題彙整概念頁收斂。

## [2026-08-25] ingest | 台灣數位醫療產業
- Created（4 實體）: [[AlleyPin 翔評互動]]、[[Health2Sync 智抗糖]]、[[雲象科技]]、[[WaCare 遠距健康]]
- Created（1 topic）: [[台灣數位醫療產業]]
- Created（1 新概念分類）: wiki/concepts/數位醫療/
- Updated: wiki/index.md（+4 entities、+1 topic，頁數 479→484）
- Key additions: (1) 四大賽道地圖——診所 SaaS（AlleyPin 牙醫數位化、FT 亞太第 7、日本拓展）、慢性病管理（Health2Sync 糖尿病→C 輪 $2,000 萬美元、IPO 籌備）、AI 醫療影像（雲象科技數位病理、FDA+IVDR 雙認證、2026 創新板掛牌、2025 營收 +91%）、遠距醫療社群（WaCare）；(2) 政策環境：電子病歷 FHIR 跨院互通（2026/01 技術驗證）、衛福部遠距醫療鬆綁、SaMD 監管框架到位；(3) 台灣結構性優勢：全民健保 30 年資料 + ICT 供應鏈 + 臨床合作開放度；(4) 核心觀察：台灣新創一致以日本為第一個國際站點，電子病歷 FHIR 互通後資料平台新創機會值得追蹤。

## [2026-08-25] ingest | IC Care
- Created（1 實體）: [[IC Care]]
- Updated: wiki/index.md（+1 entity，頁數 484→485）
- Key additions: 慷驊股份有限公司旗下遠距醫療平台，台灣唯一 24 小時線上緊急醫療諮詢；定位為旅外/出差族群，與產險公司及旅遊業者 B2B 合作為主要商業模式；有日本線上看診服務；與 WaCare（社群）、Health2Sync（慢病）定位明顯不同。

## [2026-08-25] ingest | 糖尿病數位管理競爭地圖
- Created（3 實體）: [[Kakao Healthcare]]、[[mySugr]]、[[Dario Health]]
- Created（1 概念，數位醫療/）: [[糖尿病數位管理競爭地圖]]
- Updated: wiki/index.md（+3 entities、+1 concept，頁數 485→489）
- Key additions: (1) 亞洲雙雄對決——Health2Sync（台灣）vs Kakao PASTA（韓國），兩家都整合 CGM + 智慧胰島素筆、都鎖定日本為下一站；(2) 全球格局：mySugr（Roche、600 萬用戶、歐美）、Dario Health（那斯達克、B2B 雇主端）、Livongo/Teladoc（$185 億美元收購）；(3) 三大護城河：CGM 整合深度、製藥廠智慧筆蓋合作、醫院/藥局 B2B 通路；(4) 共同趨勢：從糖尿病往多慢性病擴展、日本必爭、AI 功能標配化。

## [2026-08-25] ingest | LINE Healthcare
- Created（1 實體）: [[LINE Healthcare]]
- Updated: wiki/index.md（+1 entity，頁數 489→490）
- Key additions: LINE Doctor 已於 2025/06/10 關閉，LY Corporation 整合重疊業務所致；接棒為 SoftBank 旗下 HELPO Doctor；對台韓業者進軍日本的警示——即使背靠 LINE 月活 9,500 萬用戶，日本遠距問診商業化仍困難。

## [2026-08-25] ingest | 遠距問診法規比較（台灣 vs 日本）
- Created（1 概念，數位醫療/）: [[遠距問診法規比較（台灣vs日本）]]
- Updated: wiki/index.md（頁數 490→491）
- Key additions: (1) 台日兩地均為「例外許可制」，初診原則不可遠距、慢性病複診合法；(2) 台灣 2024 修正通訊診察治療辦法，新增電子處方箋與擴大慢性病範圍；(3) 日本 2025/12 將オンライン診療正式寫入醫療法，法制化晚但趨嚴；(4) LINE Doctor 2025/06 關閉印證日本商業化困難；(5) 對新創啟示：慢性病複診是合規路徑、「諮詢」vs「問診」的法規套利邏輯、日本市場比台灣更難進入。

## [2026-08-25] ingest | Go 1.27 新功能實戰
- Created（1 概念，go-backend/）: [[Go1.27新功能實戰]]
- Updated: wiki/index.md（+1 concept，頁數 491→492）
- Key additions: (1) Generic Methods——method 可宣告自己的 type parameter，限制：interface method 不能有 type param、generic method 不能實作 interface；(2) encoding/json/v2 正式進標準庫（1.25 GOEXPERIMENT 畢業），有破壞性變更（大小寫匹配、omitempty vs omitzero）；(3) uuid 進標準庫；(4) Goroutine Leak Profile 新 pprof 類型；(5) Post-Quantum ML-KEM TLS 層自動受益。

## [2026-08-25] ingest | Go 1.26 新功能實戰
- Created（1 概念，go-backend/）: [[Go1.26新功能實戰]]
- Updated: wiki/index.md（+1 concept，頁數 492→493）
- Key additions: (1) Green Tea GC 正式預設——以連續記憶體頁掃描取代逐指針，GC overhead 降 10–40%，高分配服務有感；(2) new 帶初始值（ptr := new(int64(300))）；(3) 泛型自參照約束（type Adder[A Adder[A]] interface）解鎖遞迴泛型；(4) crypto/hpke RFC 9180 後量子混合加密；(5) go fix 擴展為可插拔 modernizer 框架；(6) cgo overhead -30%。
