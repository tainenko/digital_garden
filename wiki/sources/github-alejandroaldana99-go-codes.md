---
title: AlejandroAldana99/Go-codes
type: source-summary
tags: [Go, REST API, GraphQL, MongoDB, gqlgen, 後端, JWT, 學習專案]
created: 2026-05-12
updated: 2026-05-12
sources: [github-alejandroaldana99-go-codes]
---

# AlejandroAldana99/Go-codes

## Origin

- **URL**: https://github.com/AlejandroAldana99/Go-codes
- **Author**: AlejandroAldana99（Go backend developer）
- **Description**: "Backend Golang projects which use REST and GraphQL as software architecture style"
- **Language**: Go 100%
- **Commits**: 18
- **Date Ingested**: 2026-05-12

## Key Takeaways

- 六個子專案，涵蓋 REST、GraphQL、In-Memory 資料結構、Best Practices 四個主題
- REST API 兩個專案（Credit-Assignment、Package-Delivery）均採用相同的七層架構：`config → constants → models → repositories → services → controllers → server`，搭配 MongoDB + `.env` 設定
- **Package-Delivery-REST-API** 實作完整 JWT 認證流程：Create User（role:Admin）→ Login（回傳 JWT）→ 後續請求攜帶 JWT header；訂單狀態機：`creado → recolectado → en_estacion → en_ruta → entregada/cancelada`
- **Server-GraphQL** 使用 **gqlgen** 框架（代碼生成方式）建立 GraphQL server；GraphQL Playground 在根路徑，查詢端點為 `/query`；支援 Mutation（createBook）與 Query（getBooks with author）
- **Best-Practices/activities** 實作活動追蹤 OOP 設計：AddActivity / UpdateActivity / GetActivity / ActivitySummary / ScheduleEvent / GetAgenda（按時間範圍過濾排序）——面試常見設計題型
- **In-Memory-DB** 展示 `NewInMemoryDB()` + Set/Get/Delete 基本 KV 操作，搭配資料結構與排序演算法
- 所有 REST API 項目均需 Go 1.18+、MongoDB 4.0+，預設監聽 `localhost:5050`

## Entities Mentioned

- [[AlejandroAldana99]] — 專案作者，Go backend developer

## Concepts Mentioned

- [[Go GraphQL gqlgen實戰]] — gqlgen 框架建立 GraphQL server 完整模式
- [[Gin_Clean_Architecture]] — 類似的 Go 分層架構（此 repo 以 MongoDB 取代 GORM/PostgreSQL）
- [[Go資料庫選型]] — MongoDB 與 Go 的整合模式補充

## Contradictions/Tensions

- 兩個 REST API 專案架構高度相似，未使用 Gin 等 framework，直接用標準庫或輕量 router
- In-Memory-DB 的 main.go 主要邏輯以註解形式存在，非完整可執行示範

## Questions Raised

- gqlgen 相較 graphql-go、graph-gophers 有何優勢？代碼生成 vs 運行時反射的取捨
- Credit-Assignment API 的 YoFio Postman collection 是否暗示這是 YoFio 公司面試題？
- activities.go 中的 ScheduleEvent + GetAgenda 是否為特定公司面試題？
