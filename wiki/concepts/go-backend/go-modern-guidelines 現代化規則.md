---
title: go-modern-guidelines 現代化規則
type: concept
tags: [golang, 現代化, AI輔助開發, JetBrains, best-practices]
created: 2026-08-25
updated: 2026-08-25
sources: [jetbrains-go-modern-guidelines]
---

# go-modern-guidelines 現代化規則

JetBrains 開源工具，解決 AI 寫出過時 Go 程式碼的問題。版本感知，只推薦 `go.mod` 中 Go 版本適用的規則。

**GitHub**：`github.com/JetBrains/go-modern-guidelines`

---

## 為什麼 AI 寫出舊式 Go？

兩個根本原因：

1. **訓練資料截止日**：模型訓練截止後的新版功能（如 Go 1.26 的 `new(val)`）不在訓練資料內
2. **頻率偏差**：網路上舊寫法的程式碼比例遠高於新寫法，AI 傾向重複舊模式

> "Language releases move faster than model training cycles."

---

## 安裝與使用

```bash
# Claude Code：啟用 skill
/use-modern-go

# 其他 agent（Cursor、Codex 等）
# 透過 skills.sh 安裝，或直接參考 GitHub repo

# CLI 查詢（需 Go 1.25+ toolchain）
go run github.com/JetBrains/go-modern-guidelines list
go run github.com/JetBrains/go-modern-guidelines explain <rule-id>
```

Junie（GoLand 內建 AI）在 2xx.620.xx+ 版自動啟用，可於 Settings → Tools → Junie → Project Settings → Go 關閉。

---

## 規則範例（Before / After）

### 1. 切片包含判斷（Go 1.21+）

```go
// ❌ 舊式手動迴圈
func HasAccess(role string) bool {
    for _, allowed := range allowedRoles {
        if role == allowed {
            return true
        }
    }
    return false
}

// ✅ 現代寫法
func HasAccess(role string) bool {
    return slices.Contains(allowedRoles, role)
}
```

### 2. 指標初始化帶值（Go 1.26+）

```go
// ❌ 舊式兩步驟
x := int64(300)
ptr := &x

// ✅ 現代寫法（Go 1.26 new 帶初始值）
ptr := new(int64(300))
```

### 3. 型別安全的 error 轉型（Go 1.26+）

```go
// ❌ 舊式 errors.As + 目標變數
var target *MyError
if errors.As(err, &target) {
    // use target
}

// ✅ 現代寫法
if target, ok := errors.AsType[*MyError](err); ok {
    // use target
}
```

### 4. Goroutine 啟動（Go 1.25+）

```go
// ❌ 舊式 WaitGroup 模板
var wg sync.WaitGroup
wg.Add(1)
go func() {
    defer wg.Done()
    doWork()
}()
wg.Wait()

// ✅ 現代寫法
var wg sync.WaitGroup
wg.Go(doWork)
wg.Wait()
```

### 5. 多值回退（cmp.Or）（Go 1.22+）

```go
// ❌ 舊式 nil check 鏈
func getConfig(a, b, c string) string {
    if a != "" {
        return a
    }
    if b != "" {
        return b
    }
    return c
}

// ✅ 現代寫法
func getConfig(a, b, c string) string {
    return cmp.Or(a, b, c)
}
```

### 6. 最大/最小值（Go 1.21+）

```go
// ❌ 舊式 if-else
func maxInt(a, b int) int {
    if a > b {
        return a
    }
    return b
}

// ✅ 現代寫法（內建 min/max）
result := max(a, b)
```

---

## 版本功能對應速查

| 功能 | 最低版本 | 規則 |
|------|---------|------|
| `slices.Contains` / `slices.Index` | Go 1.21 | 取代手動迴圈 |
| `maps.Keys` / `maps.Values` | Go 1.21 | 取代手動 map 迭代 |
| `min` / `max` 內建函式 | Go 1.21 | 取代 if-else 比較 |
| `cmp.Or` | Go 1.22 | 取代 nil check 鏈 |
| range over integer | Go 1.22 | `for i := range 10` |
| `sync.WaitGroup.Go` | Go 1.25 | 簡化 goroutine 啟動 |
| `new(val)` 帶初始值 | Go 1.26 | 取代兩步指標初始化 |
| `errors.AsType[T]` | Go 1.26 | 型別安全 error 轉型 |
| Generic Methods | Go 1.27 | method 可有自己的 type param |
| `encoding/json/v2` | Go 1.27 | 新 JSON 引擎 |

---

## 與 `go fix` 的關係

Go 1.26 大幅擴充的 `go fix` modernizer 與本工具互補：
- `go fix`：**批次自動修改**現有程式碼（適合一次性遷移）
- `go-modern-guidelines`：**引導 AI agent** 在生成新程式碼時直接用現代寫法（預防勝於治療）

---

## 相關頁面

- [[Go1.21 新功能]] / [[Go1.22新功能實戰]] / [[Go1.26新功能實戰]] / [[Go1.27新功能實戰]]
- [[Go泛型設計]]
- [[ai開發]] — AI 輔助開發工具系列
