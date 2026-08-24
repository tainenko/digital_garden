---
title: GitHub spec-kit
type: entity
tags: [GitHub, spec-kit, SDD, 規範驅動開發, AI開發工具]
created: 2026-05-18
updated: 2026-05-18
sources: [xiediandongxiba-github-spec-kit-sdd]
---

# GitHub spec-kit

GitHub 官方開源的規範驅動開發（SDD）工具。透過 `/specify → /plan → /tasks` 三個斜線指令，將自然語言需求轉化為結構化規範文件、架構計畫與排序任務清單，最終生成可執行的專案骨架。

## 核心設計哲學
- **規範是一等公民**：所有開發決策從規範出發，改動先改規範
- **AI 無關**：支援 Claude、GitHub Copilot、Gemini、Cursor
- **項目憲法**：`constitution.md` 預先設定架構原則，約束 AI 的所有決策

## 安裝
```bash
# 需要 Python ≥ 3.11 + uv
uvx --from git+https://github.com/github/spec-kit.git specify init my-project --ai claude
```

## 相關頁面
- [[Spec驅動開發]] — SDD 方法論總覽，含 spec-kit vs OpenSpec 比較
- [[OpenSpec]] — 另一套 SDD 框架（社群，Fission-AI）
- [[OpenSpec工作流]] — OpenSpec 的 8 步工作流，可與 spec-kit 三步對照
