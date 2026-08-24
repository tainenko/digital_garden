---
title: 新手接觸 Vibe Coding 肯定遇到的問題：AI 亂改交易邏輯怎麼辦？
type: source-summary
tags: [vibe-coding, algorithmic-trading, ai-safety, claude-md, characterization-tests]
created: 2026-04-30
updated: 2026-04-30
sources: []
---

# 新手接觸 Vibe Coding 肯定遇到的問題：AI 亂改交易邏輯怎麼辦？

## Origin

- **類型**：社群貼文（YouTube Community Post）
- **來源**：追日Gucci-AI效率革命聯盟（YouTube 頻道社群）
- **URL**：https://www.youtube.com/post/UgkxxPsMZUp5KLh1teI09iK5InKGv50UC-5C
- **日期**：2026-04-30 ingested

## Key Takeaways

- **核心問題**：AI 在 Vibe Coding 過程中會「善意地」改動交易邏輯——停損方向、倉位計算公式、進場條件——導致實際財務損失，這是程式交易 Vibe Coding 最危險的特有風險
- **CLAUDE.md / AGENTS.md 是護欄**：在 spec 文件中明確列出「禁止修改區域」（停損、停利、倉位、風控）和「允許修改區域」（UI、日誌、API），讓 AI 在動工前就知道邊界
- **代碼目錄分離**：`strategy/core/`（AI 禁區）vs `infrastructure/`（AI 可操作）；讓目錄結構本身傳達語意
- **安全指令模板**：「只修改 X 檔案第 Y 行，不要碰 strategy/，告訴我你打算改什麼再動手」——先描述後執行的工作流防止意外修改
- **Characterization Tests（行為凍結測試）**：對現有交易邏輯寫斷言測試，凍結已知正確的輸出值；AI 若改壞邏輯，測試立即失敗
- **小任務範圍**：一次只給 AI 一個明確任務，避免 AI 在「順手整理」時動到策略代碼
- **Backtest 驗證工作流**：每次 AI 修改後自動跑回測，與基準 JSON 比對 PnL、勝率、最大回撤；指標偏差超過閾值即告警
- **Git CODEOWNERS**：設定 `strategy/core/ @human-lead` 強制人工 review，防止 AI 代碼繞過審查直接 merge
- **核心心態轉變**：AI 是「受控工程師」，不是「策略設計師」；策略的每個決策都應能追溯到人類的主動選擇，而非 AI 的「優化建議」

## Entities Mentioned

- [[追日Gucci]] — 貼文作者，YouTube 頻道（AI效率革命聯盟），專注 AI 效率工具與量化交易應用

## Concepts Mentioned

- [[Vibe Coding程式交易實戰]] — 本文核心知識整理頁
- [[Vibe Coding基礎概念]] — Vibe Coding 定義與基本工作流
- [[Vibe Coding風險與限制]] — AI 在程式開發中的一般性風險
- [[Spec驅動開發]] — CLAUDE.md 本質上是一種 spec 文件

## Contradictions / Tensions

- **Vibe Coding 的「信任 AI」哲學 vs 交易的「零容忍」要求**：一般 Vibe Coding 鼓勵放手讓 AI 完成，但交易系統的一個 off-by-one 就可能造成方向性錯誤；需要比一般軟體開發更嚴格的人工審查層
- **小任務 vs 重構需求**：交易系統往往需要跨檔案重構（參數化、模組化），但小任務原則要求縮小 AI 的操作範圍；兩者需要精心設計任務邊界才能兼顧

## Questions Raised

- Characterization Tests 的「正確值」本身如何驗證？若歷史邏輯就有 bug，凍結錯誤值反而會鎖定缺陷
- CODEOWNERS 機制在個人單兵開發（solo developer）的場景下如何替代？（可能需要自律紀律或 pre-commit hook）
- 多個策略共用同一 AI 代碼庫時，protected zone 的粒度如何設計？
