---
title: AI 方法論 Skill 地圖
type: concept
tags: [AI輔助開發, DDD, Clean Architecture, SOLID, Claude Code, skill, 方法論]
created: 2026-08-25
updated: 2026-08-25
sources: [ai-methodology-skills-landscape]
---

# AI 方法論 Skill 地圖

讓 AI Coding Agent 按特定設計哲學撰寫程式碼的 Skill 全景。與工具鏈型 Skill（[[go-modern-guidelines 現代化規則]]、[[modern-python 現代工具鏈]]）不同，方法論 Skill 約束的是**設計決策**而非語法選擇。

---

## 兩大 Skill 類型對比

| 類型 | 目的 | 例子 |
|------|------|------|
| **工具鏈型** | 讓 AI 用現代語法/工具 | go-modern-guidelines、modern-python |
| **方法論型** | 讓 AI 遵守設計哲學 | DDD、SOLID、Clean Architecture |

---

## 方法論 Skill 全景

### 🏛️ 架構模式

#### nathankim0/clean-architecture-skills
- **安裝**：`/plugin marketplace add nathankim0/clean-architecture-skills`
- **兩個 Skill**：
  - `clean-architecture`：Robert Martin 依賴規則驗證（Entity → UseCase → Adapter → Infra 單向依賴）、SOLID 合規檢查
  - `kent-beck-style`：Code Smell 偵測（Bloater、OO Abuser、Change Preventer 等）+ Refactoring 技法

#### ruvnet/v3-ddd-architecture
- **安裝**：Clone from `github.com/ruvnet/agentic-flow`
- **特色**：God Object 分解為 Bounded Context、Clean Architecture 整合、模組化可測試程式碼結構

#### zudochkin/go-clean-ddd-skill（Go 專用）
- **安裝**：Clone + `/ddd-model [domain-name]`
- **6 階段互動式 DDD 建模流程**：
  1. Domain Discovery
  2. Bounded Context 定義
  3. Tactical Modeling（Aggregate、Entity、Value Object）
  4. Clean Architecture 目錄結構生成
  5. Uber Style Guide 合規
  6. 生產就緒 Go 程式碼輸出
- 附帶完整 Banking Domain 範例

---

### ⚙️ 原則驅動

#### ramziddin/solid-skills（最嚴格）
- **安裝**：`npx skills add ramziddin/solid-skills`
- **SOLID 五原則**：

| 原則 | 核心約束 |
|------|---------|
| Single Responsibility | 一個 class 只有一個改變原因 |
| Open/Closed | 開放擴展、封閉修改 |
| Liskov Substitution | 子類可完全替換父類 |
| Interface Segregation | 介面要小而專注 |
| Dependency Inversion | 依賴抽象，不依賴具體實作 |

- **硬性尺寸約束**：method < 10 行、class < 50 行
- **附帶**：TDD Red-Green-Refactor 強制流程、Design Patterns、Value Object、Clean Architecture

---

### 📚 ZLStas/booklib-ai — 22 本書蒸餾

**最完整的方法論 Skill 集合**，每個 Skill 都是一本經典著作的精華提取。

```bash
# 安裝全部
npx @booklib/skills add --all --global

# 按語言 profile 安裝
npx @booklib/skills add --profile=python --global    # Python 相關
npx @booklib/skills add --profile=ts --global        # TypeScript
npx @booklib/skills add --profile=jvm --global       # Java / Kotlin
npx @booklib/skills add --profile=rust --global      # Rust
```

**四層架構**：
| 層次 | 說明 | 數量 |
|------|------|------|
| **Skills** | 依檔案類型自動觸發 | 22 |
| **Commands** | 明確呼叫（slash command） | 22 |
| **Agents** | 自主審查 Agent | 8 |
| **Rules** | 每次 Session 載入的常駐規則 | 6 |

**完整書單**：

| Skill 名稱 | 書籍 | 適用場景 |
|-----------|------|---------|
| `domain-driven-design` | Eric Evans《DDD》 | Aggregate、Bounded Context、Ubiquitous Language |
| `design-patterns` | Head First Design Patterns | Creational、Structural、Behavioral 模式 |
| `clean-code-reviewer` | Robert C. Martin《Clean Code》| 命名、函式、注解品質 |
| `microservices-patterns` | Chris Richardson | Saga、CQRS、API Gateway、Event Sourcing |
| `data-intensive-patterns` | Kleppmann《DDIA》 | 分散式系統、一致性、複製 |
| `system-design-interview` | Alex Xu | 可擴展架構設計 |
| `effective-python` | Brett Slatkin（第三版） | Pythonic 慣例 |
| `effective-typescript` | Dan Vanderkam（第二版） | 型別系統精通 |
| `effective-java` | Joshua Bloch | Java 最佳實踐 |
| `effective-kotlin` | Marcin Moskała | Kotlin 慣例、Coroutine |
| `refactoring-ui` | Wathan & Schoger | UI 設計原則 |
| `storytelling-with-data` | Cole Nussbaumer Knaflic | 資料視覺化敘事 |
| `lean-startup` | Eric Ries | 快速迭代方法論 |
| `data-pipelines` | James Densmore | Pipeline 架構 |
| `using-asyncio-python` | Caleb Hattingh | Python async/await |
| `rust-in-action` | Tim McNamara | Rust 所有權系統 |
| `spring-boot-in-action` | Craig Walls | Spring Boot 模式 |

---

## 推薦組合

### Go 後端嚴格版
```bash
/use-modern-go                          # 語言現代化
/ddd-model [domain-name]               # DDD 建模（zudochkin）
npx @booklib/skills add --skill microservices-patterns
```

### Python 後端架構版
```bash
/skill add trailofbits/modern-python   # 工具鏈
npx skills add ramziddin/solid-skills  # SOLID + TDD
npx @booklib/skills add --skill domain-driven-design
```

### 全方位程式碼品質
```bash
/plugin marketplace add nathankim0/clean-architecture-skills  # CA 驗證
npx @booklib/skills add --skill clean-code-reviewer           # Clean Code
npx @booklib/skills add --skill design-patterns               # Design Patterns
```

---

## 相關頁面

- [[go-modern-guidelines 現代化規則]] — 工具鏈型，Go 語言
- [[modern-python 現代工具鏈]] — 工具鏈型，Python
- [[DDD領域驅動設計]] — DDD 概念知識（wiki 現有頁）
- [[ai開發]] — AI 輔助開發工具總覽
