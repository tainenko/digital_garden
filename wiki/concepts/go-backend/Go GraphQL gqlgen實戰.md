---
title: Go GraphQL gqlgen 實戰
type: concept
tags: [Go, GraphQL, gqlgen, 後端, API設計, 代碼生成]
created: 2026-05-12
updated: 2026-05-12
sources: [github-alejandroaldana99-go-codes]
---

# Go GraphQL gqlgen 實戰

## 什麼是 gqlgen

`gqlgen` 是 Go 最主流的 GraphQL 框架，採用**代碼生成（codegen）**方式：先寫 GraphQL Schema（.graphql 檔），再由工具自動生成 Go 的 resolver 骨架、型別定義與接線代碼。

相比運行時反射方式（如 `graphql-go`），gqlgen 優勢：
- 完整型別安全（Type-safe resolvers）
- IDE 自動補全
- 生成代碼可審查，不依賴運行時 magic

---

## 目錄結構

```
Server-GraphQL/graphql/
├── graph/
│   ├── schema.graphqls   ← 定義 Query/Mutation/Type
│   ├── model/            ← gqlgen 生成的 Go 型別
│   └── resolver.go       ← 手寫業務邏輯
├── controller/           ← HTTP handler
├── database/             ← DB 連線（MongoDB）
├── gqlgen.yml            ← gqlgen 代碼生成配置
├── go.mod
└── server.go             ← 主程式
```

---

## server.go 核心模式

```go
package main

import (
    "log"
    "net/http"
    "os"

    "github.com/99designs/gqlgen/graphql/handler"
    "github.com/99designs/gqlgen/graphql/playground"
    "your/module/graph"
    "your/module/database"
)

func main() {
    database.GetConnection() // 建立 DB 連線

    port := os.Getenv("PORT")
    if port == "" {
        port = "8080"
    }

    srv := handler.NewDefaultServer(graph.NewExecutableSchema(graph.Config{
        Resolvers: &graph.Resolver{},
    }))

    http.Handle("/", playground.Handler("GraphQL playground", "/query"))
    http.Handle("/query", srv)

    log.Printf("connect to http://localhost:%s/ for GraphQL playground", port)
    log.Fatal(http.ListenAndServe(":"+port, nil))
}
```

---

## Schema 範例（書籍管理）

```graphql
type Book {
  id: ID!
  title: String!
  userId: String!
  author: Author
}

type Author {
  id: ID!
  name: String!
}

type Query {
  books: [Book!]!
}

type Mutation {
  createBook(title: String!, name: String!, userId: String!): Book!
}
```

---

## gqlgen.yml 基本配置

```yaml
schema:
  - graph/*.graphqls

exec:
  filename: graph/generated.go
  package: graph

model:
  filename: graph/model/models_gen.go
  package: model

resolver:
  layout: follow-schema
  dir: graph
  package: graph
```

---

## 常用指令

```bash
# 安裝 gqlgen
go get github.com/99designs/gqlgen

# 初始化專案
go run github.com/99designs/gqlgen init

# 重新生成代碼（Schema 修改後）
go generate ./...
```

---

## REST vs GraphQL 選型

| 面向 | REST | GraphQL |
|------|------|---------|
| 資料獲取 | 多個端點，固定回傳欄位 | 單一端點，客戶端指定欄位 |
| 版本管理 | /v1 /v2 | Schema 漸進演進 |
| 適用場景 | 簡單 CRUD、行動端帶寬敏感 | 複雜關聯資料、多客戶端需求 |
| Go 主流框架 | Gin、Echo、Chi | gqlgen |
| 型別安全 | 需手動定義 | gqlgen 自動生成型別 |

---

## 相關頁面

- [[gRPC設計與實戰]] — 另一種結構化 API 協定，適合服務間通訊
- [[Gin_Clean_Architecture]] — Go REST API 的 Clean Architecture 參考
- [[微服務架構設計原則]] — 選擇 API 協定的架構考量
- [[AlejandroAldana99]] — 此 gqlgen 範例的來源專案作者
