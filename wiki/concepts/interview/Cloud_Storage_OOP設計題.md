---
title: Cloud Storage OOP 設計題
type: concept
tags: [OOP, OOD, CodeSignal, 面試, Python, LLD, 雲端儲存, backup/restore]
created: 2026-05-12
updated: 2026-05-12
sources: [github-jessezhuang-incode-python3]
---

# Cloud Storage OOP 設計題

## 題目背景

平台：CodeSignal（推測），OOP 設計題型
語言：Python（`sortedcontainers` 輔助）
特色：四層遞進，Level 3 引入多用戶容量管理，Level 4 加入 backup/restore 快照

---

## 資料結構

```python
class File:
    name: str
    size: int
    user_id: str = "admin"

class User:
    user_id: str
    capacity: int          # 剩餘可用空間（admin 為無限大）
    files: set             # 目前持有的檔名集合
    backup: set | None     # 最近一次備份的檔名集合

class CloudStorage:
    storage: dict[str, File]   # filename → File
    users:   dict[str, User]   # user_id → User
    # 初始化時自動建立 admin 用戶（無限容量）
```

---

## Level 1 — 基礎檔案操作（admin 帳戶）

```
add_file(name, size) -> "true" | "false"
  新增檔案至 admin 帳戶；已存在回傳 "false"

get_file_size(name) -> str(size) | ""
  取得檔案大小；不存在回傳 ""

delete_file(name) -> str(size) | ""
  刪除檔案並回傳大小；不存在回傳 ""
```

## Level 2 — 檔案排名

```
n_largest(prefix, n) -> list[str]
  回傳名稱符合 prefix 的最大 n 個檔案
  排序規則：size 降序，同 size 則 name 字典序升序
  格式："name(size)"
```

## Level 3 — 多用戶容量管理

```
add_user(user_id, capacity) -> "true" | "false"
  建立用戶，設定容量上限；已存在回傳 "false"

add_file_by(user_id, name, size) -> str(remaining_capacity) | ""
  以指定用戶新增檔案；容量不足或用戶不存在回傳 ""
  成功後回傳用戶剩餘容量

merge_user(user_id_1, user_id_2) -> str(remaining_capacity) | ""
  將 user_id_2 合併入 user_id_1
  轉移所有檔案所有權 + 剩餘容量加總
  user_id_2 消失；兩者相同或任一不存在回傳 ""
```

## Level 4 — 備份與還原

```
backup(user_id) -> str(file_count) | ""
  快照用戶目前的檔案集合（深複製）
  回傳目前持有的檔案數量；用戶不存在回傳 ""

restore(user_id, timestamp) -> str(file_count) | ""
  還原至最近一次 backup 前的狀態
  跳過與現存檔案命名衝突的檔案
  用戶不存在或無 backup 回傳 ""
```

---

## 解題思路

- `admin` 用戶容量設為 `float('inf')` 或極大整數，統一走 `add_file_by` 邏輯
- `n_largest` 需先 filter prefix，再用 `heapq.nlargest` 或 sorted + slice
- `merge_user`：遍歷 user_id_2 的 files，更新 `storage[f].user_id = user_id_1`，再轉移容量
- `backup` / `restore`：用 `copy.copy(user.files)` 存快照；restore 時排除命名衝突

### 複雜度

| 操作 | 時間複雜度 |
|------|-----------|
| add_file / delete_file | O(1) |
| get_file_size | O(1) |
| n_largest | O(F log F) — F 為符合 prefix 的檔案數 |
| merge_user | O(F) — F 為 user_id_2 的檔案數 |
| backup / restore | O(F) |

---

## 與其他 OOP 設計題的共同模式

| 特性 | Cloud Storage | In-Memory DB | Banking System |
|------|--------------|--------------|----------------|
| backup/restore | ✓（Level 4）| ✓（Level 4）| — |
| 用戶/帳戶合併 | ✓（Level 3）| — | ✓（Level 4）|
| 容量/餘額限制 | ✓ | — | ✓ |
| 排名/排序查詢 | ✓（n_largest）| ✓（scan）| ✓（top_spenders）|

> backup/restore 是 CodeSignal OOP 設計題的常見 Level 4 模式，見 [[In_Memory_Database]]

---

## 相關頁面

- [[In_Memory_Database]] — Coinbase HA Version B，同樣有 backup/restore Level 4
- [[Banking_System]] — Coinbase HA Version A，merge_accounts 與 merge_user 邏輯相似
- [[Low Level Design OOD設計題型]] — OOP/LLD 設計題通用五步驟框架
- [[JesseZhuang]] — 此題 Python 實作的來源作者
