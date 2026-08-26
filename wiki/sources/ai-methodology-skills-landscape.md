---
title: AI 方法論 Skill 全景 — DDD、Clean Architecture、SOLID
type: source-summary
tags: [AI輔助開發, DDD, Clean Architecture, SOLID, Claude Code, skill]
created: 2026-08-25
updated: 2026-08-25
sources: [ai-methodology-skills-landscape]
---

# AI 方法論 Skill 全景

## Origin

- **整理日期**：2026-08-25
- **來源**：agenticskills.io、GitHub（nathankim0、ramziddin、ZLStas/booklib-ai、zudochkin、ruvnet）
- **背景**：研究 JetBrains go-modern-guidelines 後的延伸調查，探索是否有讓 AI 按架構方法論寫程式的 skill

## Key Takeaways

1. **兩大 Skill 類型**：工具鏈型（modern-python、go-modern-guidelines）vs 方法論型（DDD、SOLID、Clean Architecture）——前者讓 AI 用現代語法，後者讓 AI 遵守設計哲學
2. **ZLStas/booklib-ai 是最完整的書蒸餾專案**：22 本經典著作 → AI Skill，含 DDD、Design Patterns、Clean Code、Microservices Patterns、DDIA 等；支援 Profile 安裝（Python/TypeScript/Rust/JVM）
3. **solid-skills 最嚴格**：method < 10 行、class < 50 行的硬性約束，附 TDD Red-Green-Refactor 強制流程
4. **go-clean-ddd-skill 最完整的語言特化**：6 階段互動式 DDD 建模 + Uber Style Guide + Clean Architecture 目錄結構
5. **nathankim0/clean-architecture-skills** 提供雙 Skill：Clean Architecture（依賴規則驗證）+ Kent Beck 重構風格
6. **四層啟動架構**（ZLStas）：Skills（自動觸發）/ Commands（明確呼叫）/ Agents（自主審查）/ Rules（每次載入）

## Entities Mentioned

- [[ZLStas / booklib-ai]] — 22 本書蒸餾 Skill，最完整的方法論 Skill 集合
- [[Trail of Bits]] — modern-python 出品方（工具鏈型對照）
- [[JetBrains]] — go-modern-guidelines 出品方（工具鏈型對照）

## Concepts Mentioned

- [[AI 方法論 Skill 地圖]] — 核心整理頁

## Questions Raised

- booklib-ai 的 Skill 品質評測（80%+ pass rate）是用什麼 benchmark 衡量？
- 方法論 Skill + 工具鏈 Skill 同時使用時會不會互相衝突？
- 有沒有針對 React / Next.js 的架構方法論 Skill（Component Patterns、State Management）？
