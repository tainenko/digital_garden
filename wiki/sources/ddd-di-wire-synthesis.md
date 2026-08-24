---
title: DDD、依賴注入與 Go Wire 完整知識合集
type: source-summary
tags: [ddd, di, IoC, wire, golang, domain-driven-design, dependency-injection]
created: 2026-04-30
updated: 2026-04-30
sources: []
---

# DDD、依賴注入與 Go Wire 完整知識合集

## Origin

- **類型**：合成知識（Synthesized Knowledge）
- **來源**：Eric Evans《Domain-Driven Design》、Vaughn Vernon《Implementing Domain-Driven Design》、Google Wire 官方文件、工程實戰整理
- **日期**：2026-04-30

## Key Takeaways

- **DDD 的核心價值**是讓程式碼結構反映業務模型——Ubiquitous Language 讓代碼即文件；Bounded Context 給出服務邊界的理論依據（一個微服務通常對應一個 Bounded Context）
- **戰術設計四要素**：Entity（有 ID、追蹤同一性）、Value Object（無 ID、不可變、用屬性比較）、Aggregate（一致性邊界，Aggregate Root 統一存取）、Domain Event（記錄發生了什麼，解耦副作用）
- **Repository 介面定義在 Domain 層、實作在 Infrastructure 層**——這是依賴反轉原則的直接體現，讓 Domain Layer 完全不依賴任何框架或資料庫技術
- **DI 是實踐 SOLID 中「D」的機制**：高層模組（Application Service）依賴抽象（Repository 介面），低層模組（PostgresRepo）實作抽象；建構子注入是最推薦的方式，讓依賴關係明確、測試友善
- **Wire 的核心設計哲學**：編譯時靜態分析（非執行時反射）、生成的是普通 Go 代碼（可讀可 debug）、Provider 就是普通函數（無特殊標記）、ProviderSet 讓大型專案模組化組裝
- **Wire 的 cleanup 機制**自動按依賴的反向順序清理資源（類似 defer LIFO），不需要手動管理清理順序
- **wire.Bind 解決介面綁定**：當 Provider 返回具體型別但需要注入介面時；wire.FieldsOf 解決大型 Config struct 的欄位提取；wire.Struct 取代冗長的建構子函數
- **Aggregate 應該盡量小**：同一個事務內需要保持一致的最小集合；跨 Aggregate 的一致性用 Domain Events + 最終一致性（不用分散式事務）
- **Application Service 只協調，不含業務邏輯**：事務管理、授權檢查、DTO 轉換在這裡；所有業務規則放在 Entity、Value Object 或 Domain Service 中
- **DDD + DI + Wire 的組合效果**：Domain Layer 純淨（無框架依賴）→ 單元測試無需 DB；Repository 介面 + Wire.Bind → 換資料庫只改一行；ProviderSet 按層分組 → 依賴圖清晰可維護

## Concepts Mentioned

- [[DDD領域驅動設計]] — 戰略設計（Bounded Context、Ubiquitous Language）+ 戰術設計（Entity、Value Object、Aggregate、Domain Event、Repository、Application Service）
- [[依賴注入與控制反轉]] — IoC 原理、三種注入方式、可測試性、SOLID、生命週期管理、反模式
- [[Go Wire深度實戰]] — Provider、Injector、ProviderSet、介面綁定、cleanup、FieldsOf、Struct、常見錯誤

## Related Existing Concepts

- [[Go依賴注入與Wire]] — 原有的基礎 Wire 頁面（含 uber-go/fx 比較）
- [[微服務架構設計原則]] — Bounded Context 與微服務邊界
- [[Go介面設計模式]] — Repository、Strategy 等介面模式
- [[Go測試基準與模糊測試]] — DI 帶來的可測試性

## Contradictions / Tensions

- **DDD Aggregate 小 vs 一致性**：越小的 Aggregate 越容易並發，但跨 Aggregate 用最終一致性需要接受暫時不一致；對於強一致性需求（如金融轉帳）需要謹慎設計
- **Wire vs fx 的選擇**：Wire 生成靜態代碼，適合依賴圖固定的服務；fx 支援動態 Module、Lifecycle hook，但用反射有執行時開銷且錯誤訊息較難讀；兩者都是 Google 出品但設計哲學不同
- **DDD 的複雜度成本**：DDD 引入了大量抽象（Entity、Value Object、Repository、Application Service 等），對於簡單 CRUD 服務是過度設計；適用於業務邏輯複雜、預期長期演化的核心域

## Questions Raised

- 在 Go 的 No-GC（stack allocation heavy）場景，Value Object 的「不可變建立新物件」是否有顯著的 GC 壓力？（逃逸分析能否優化到 stack？）
- Wire 是否支援條件性 Provider（如：dev 環境用 InMemoryRepo，prod 用 PostgresRepo）？（目前需要用 build tag 或不同的 Injector）
- Aggregate 邊界的決策在實際專案中如何做到不過大也不過小？
