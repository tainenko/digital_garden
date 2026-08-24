---
title: GitHub Spec Kit 中文教程：用規範驅動開發（SDD）從想法直達代碼
type: source-summary
tags: [spec-kit, SDD, GitHub, 規範驅動開發, AI開發]
created: 2026-05-18
updated: 2026-05-18
sources: [xiediandongxiba-github-spec-kit-sdd]
---

# GitHub Spec Kit 中文教程：用規範驅動開發（SDD）從想法直達代碼

## Origin
- **平台**：寫點東西吧（Google Sites）
- **URL**：https://sites.google.com/view/xiediandongxiba/github-spec-kit-中文教程用規範驅動開發sdd從想法直達代碼
- **主題**：GitHub 官方 spec-kit 工具的中文操作教程

## Key Takeaways

- **GitHub spec-kit 定位**：GitHub 官方開源工具，將自然語言需求轉化為結構化功能規範→詳細實現計畫→排序任務清單→可執行專案骨架，規範為「一等公民」。
- **三步核心工作流**：`/specify`（定義 WHAT & WHY）→ `/plan`（生成架構文件）→ `/tasks`（拆解可執行任務，標記可並行項 `[P]`）
- **項目憲法（constitution.md）**：九大架構原則預先約束 AI，包含庫優先、CLI 接口要求、測試優先、簡化原則（初始專案數 ≤3）、反抽象原則。
- **效率數據**：傳統文件流程 12 小時（PRD 2–3h、設計 2–3h、規範 3–4h、測試計畫 2h）→ SDD 15 分鐘（三個指令各 5 分鐘）。
- **規範寫作三條鐵律**：① 專注 WHAT & WHY，避免 HOW；② 用 `[需要澄清]` 代替猜測；③ 驗收標準必須可測試、可量化。
- **支援多 AI 助手**：`--ai claude / copilot / gemini / cursor` 旗標，工具本身 AI 無關。
- **適用邊界清晰**：特別適合 0→1 新產品、企業級項目、多團隊協作；**不適合**線上緊急 Hotfix、一次性腳本、個人學習 Demo。
- **改動先改規範**：規範是唯一事實來源，任何功能變更先改 spec.md，再推導 plan、tasks。

## 標準專案結構
```
your-project/
├── .specify/
│   ├── memory/（constitution.md — 項目憲法）
│   ├── scripts/
│   └── templates/
└── specs/
    └── 001-feature-name/
        ├── spec.md
        ├── plan.md
        ├── research.md
        ├── data-model.md
        ├── contracts/
        ├── quickstart.md
        └── tasks.md
```

## Entities Mentioned
- [[GitHub-spec-kit]] — 本文主角工具
- GitHub — 工具開發者

## Concepts Mentioned
- [[Spec驅動開發]] — spec-kit 是 SDD 理念的 GitHub 官方實作
- [[OpenSpec工作流]] — 另一套 SDD 框架，可與 spec-kit 對照

## Contradictions / Tensions
- spec-kit 的「15 分鐘」效率數據來自展示案例，實際複雜度高的專案效果未必如此線性。
- spec-kit 與 OpenSpec 在定位上有重疊（都是 SDD），但 spec-kit 是 GitHub 官方、聚焦工具鏈；OpenSpec 是社群框架、聚焦決策追溯與人機協作協議。

## Questions Raised
- constitution.md 九大原則的「庫優先」在 monorepo 架構下如何實踐？
- `/tasks` 的 `[P]` 並行標記由誰判斷——AI 自動推斷還是需要人工確認？
- spec-kit 與 OpenSpec 可否混用？（spec-kit 生成結構，OpenSpec Delta Spec 追蹤變更）
