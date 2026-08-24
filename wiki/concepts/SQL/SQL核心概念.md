---
title: SQL 核心概念
type: concept
tags: [sql, 資料庫, 後端, 入門, 查詢優化]
created: 2026-05-05
updated: 2026-05-05
sources: [theprimeagen-sql-course-bootdev]
---

# SQL 核心概念

> 以 [[Boot.dev]] × [[ThePrimeagen]] 4 小時課程為骨架的 SQL 知識地圖。  
> 課程 URL：https://www.youtube.com/watch?v=rf57kE3HJD0

---

## 1. Introduction — SQL 是什麼

- **SQL**（Structured Query Language）：用來與關聯式資料庫溝通的語言
- 關聯式資料庫將資料儲存於「資料表（table）」，資料表之間透過關係連結
- 常見 RDBMS：PostgreSQL、MySQL、SQLite、SQL Server

---

## 2. Tables — 資料表與型別

```sql
CREATE TABLE users (
    id   INTEGER PRIMARY KEY,
    name TEXT    NOT NULL,
    age  INTEGER
);
```

- **資料型別**：INTEGER、TEXT、REAL、BLOB、BOOLEAN、TIMESTAMP
- `CREATE TABLE` 建表；`DROP TABLE` 刪表；`ALTER TABLE` 改表結構

---

## 3. Constraints — 約束條件

| 約束 | 說明 |
|------|------|
| `NOT NULL` | 欄位不可為空 |
| `UNIQUE` | 欄位值唯一 |
| `PRIMARY KEY` | 主鍵（NOT NULL + UNIQUE）|
| `FOREIGN KEY` | 外鍵，參照另一張表的主鍵 |
| `DEFAULT` | 未插入時使用預設值 |
| `CHECK` | 自訂條件式限制 |

```sql
CREATE TABLE posts (
    id      INTEGER PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users(id),
    title   TEXT    NOT NULL,
    body    TEXT    DEFAULT ''
);
```

---

## 4. CRUD — 基本操作

```sql
-- Create
INSERT INTO users (name, age) VALUES ('Alice', 30);

-- Read
SELECT * FROM users;
SELECT name, age FROM users WHERE age > 25;

-- Update
UPDATE users SET age = 31 WHERE name = 'Alice';

-- Delete
DELETE FROM users WHERE name = 'Alice';
```

---

## 5. Basic Queries — 基礎查詢

```sql
SELECT name, age
FROM   users
WHERE  age > 18
ORDER  BY age DESC
LIMIT  10
OFFSET 20;   -- 分頁
```

- `WHERE`：過濾條件（AND、OR、NOT、IN、BETWEEN、LIKE）
- `ORDER BY`：排序（ASC 升冪、DESC 降冪）
- `LIMIT` + `OFFSET`：分頁查詢

---

## 6. Structuring — 資料庫設計原則

- **單一職責**：每張表只存一種「事物」
- **避免重複資料**：共用資料抽成獨立表，用外鍵關聯
- **命名慣例**：表名複數（`users`、`posts`），欄位名蛇形命名（`created_at`）

---

## 7. Aggregations — 聚合函數

```sql
SELECT department,
       COUNT(*)    AS headcount,
       AVG(salary) AS avg_salary,
       MAX(salary) AS max_salary
FROM   employees
GROUP  BY department
HAVING COUNT(*) > 5;
```

| 函數 | 說明 |
|------|------|
| `COUNT(*)` | 行數 |
| `SUM(col)` | 加總 |
| `AVG(col)` | 平均值 |
| `MIN/MAX(col)` | 最小/最大值 |

- `GROUP BY`：分組聚合
- `HAVING`：對聚合結果過濾（`WHERE` 無法用在聚合函數上）

---

## 8. Subqueries — 子查詢

```sql
-- 子查詢作為 WHERE 條件
SELECT name FROM users
WHERE id IN (
    SELECT user_id FROM orders WHERE amount > 1000
);

-- 子查詢作為衍生表（Derived Table）
SELECT dept, avg_sal
FROM (
    SELECT department AS dept, AVG(salary) AS avg_sal
    FROM employees
    GROUP BY department
) AS dept_stats
WHERE avg_sal > 60000;
```

- 子查詢可出現在 `WHERE`、`FROM`、`SELECT` 中
- 相關子查詢（Correlated Subquery）：內層查詢引用外層表的欄位，效能通常較差

---

## 9. Normalization — 資料正規化

正規化目的：消除重複資料、減少更新異常。

| 正規形式 | 要求 |
|----------|------|
| **1NF** | 每格只存一個值（不可有陣列/清單） |
| **2NF** | 符合 1NF，且非主鍵欄位完全依賴主鍵（無部分依賴） |
| **3NF** | 符合 2NF，且無遞移依賴（non-key → non-key） |

**反正規化**：為效能犧牲部分正規化（如儲存冗餘計數欄位），在高讀取場景常見。

---

## 10. Joins — 表關聯

```sql
-- INNER JOIN：只取兩表都有的資料
SELECT u.name, o.amount
FROM   users u
INNER JOIN orders o ON u.id = o.user_id;

-- LEFT JOIN：保留左表所有資料，右表無對應則 NULL
SELECT u.name, o.amount
FROM   users u
LEFT JOIN orders o ON u.id = o.user_id;
```

| JOIN 類型 | 說明 |
|-----------|------|
| `INNER JOIN` | 只有兩表都匹配的行 |
| `LEFT JOIN` | 左表全部 + 右表匹配（無則 NULL） |
| `RIGHT JOIN` | 右表全部 + 左表匹配 |
| `FULL OUTER JOIN` | 兩表全部，無匹配則 NULL |
| `CROSS JOIN` | 笛卡兒積（每行配每行） |
| `SELF JOIN` | 同一張表 JOIN 自己（如員工-主管關係） |

---

## 11. Performance — 效能調優

### 索引（Index）

```sql
-- 建立索引
CREATE INDEX idx_users_email ON users(email);

-- 複合索引（最左前綴原則）
CREATE INDEX idx_orders_user_date ON orders(user_id, created_at);
```

- 索引加速 `WHERE`、`JOIN`、`ORDER BY` 查詢
- 代價：寫入變慢（INSERT/UPDATE/DELETE 需同步更新索引）
- 不適合：低基數欄位（如 boolean）、頻繁更新欄位

### EXPLAIN / EXPLAIN ANALYZE

```sql
EXPLAIN ANALYZE
SELECT * FROM orders WHERE user_id = 42;
```

- `Seq Scan`：全表掃描（通常需要優化）
- `Index Scan`：使用索引（理想）
- `Nested Loop / Hash Join / Merge Join`：JOIN 演算法選擇

### 常見效能問題

| 問題 | 解法 |
|------|------|
| N+1 查詢 | 改用 JOIN 或批次查詢 |
| 缺少索引 | EXPLAIN 定位後加索引 |
| SELECT * | 只取需要的欄位 |
| 大量 OFFSET 分頁 | 改用 Cursor-based pagination（WHERE id > last_id）|

---

## 相關頁面

- [[ThePrimeagen]] — 課程主講者
- [[Boot.dev]] — 課程平台
- [[theprimeagen-sql-course-bootdev|課程來源摘要]]
- [[SQL vs NoSQL 選型框架]] — 系統設計中的資料庫選型決策
- [[Go資料庫選型]] — GORM vs sqlc vs pgx，Go 專案的 SQL 實踐
- [[MySQL索引與B+樹]] — MySQL/InnoDB 索引內部原理（B+ 樹、聚簇/覆蓋索引）
- [[MySQL事務與隔離級別]] — ACID、隔離級別與 MVCC
- [[資料庫正規化]] — 三大正規形式
