---
title: go-gin-clean-arch（GitHub thnkrn）
type: source-summary
tags: [Golang, Gin, CleanArchitecture, Wire, GORM, JWT, Swagger]
created: 2026-05-04
updated: 2026-05-04
sources: []
---

# go-gin-clean-arch（GitHub thnkrn）

## Origin

- **Repo**: https://github.com/thnkrn/go-gin-clean-arch
- **Branch**: master
- **Stack**: Go + Gin + GORM/PostgreSQL + Wire + Viper + JWT + Swagger

---

## Key Takeaways

1. **4 層 Clean Architecture**：Domain → Repository（interface + impl）→ UseCase（interface + impl）→ Handler；依賴方向只能由外向內
2. **每層都有 interface**：`pkg/repository/interface/` 和 `pkg/usecase/interface/` 讓上層只依賴抽象，不依賴具體實作
3. **Wire 自動產生 DI**：`pkg/di/wire.go` 宣告 provider chain；`make wire` 生成 `wire_gen.go`，消除手寫 constructor chain
4. **DTO 轉換用 copier**：Handler 層以 `Response` struct + `copier.Copy()` 將 domain 物件轉換為 API 回應，避免直接暴露 domain
5. **JWT middleware**：Bearer token 驗證在 `pkg/api/middleware/auth.go`，掛在 `/api` group
6. **Swagger 自動生成**：Handler 上方加 `// @summary` 等 godoc 註解，`make swag` 產生 `cmd/api/docs/`
7. **config 統一由 Viper 管理**：`.env` 檔案 + `mapstructure` tag，啟動時 validate

---

## Entities Mentioned

- 無特定實體

---

## Concepts Mentioned

- [[Gin_Clean_Architecture]] — 架構詳解、層級職責、新增 Domain 流程、OpenSpec AI 工作流
- [[Go Wire深度實戰]] — Wire provider/injector 原理
- [[DDD領域驅動設計]] — domain layer 概念背景
- [[Go介面設計模式]] — Repository/UseCase interface 模式

---

## Contradictions / Tensions

- Domain struct 直接用 GORM tag（如 `gorm:"unique;not null"`）：嚴格 Clean Architecture 主張 domain 不應知道 ORM，此 repo 為方便而妥協

---

## Questions Raised

- 如何加入 Unit Test 覆蓋 UseCase 層（mock Repository interface）？
- 多 Domain 時如何組織 server.go 路由（feature module vs single router）？
