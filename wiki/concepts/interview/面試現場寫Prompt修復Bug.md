---
title: 面試現場寫 Prompt 修復 Bug：AI 協作能力面試
type: concept
tags: [面試, AI, Prompt, 技術面試, Claude, 極海Channel]
created: 2026-04-30
updated: 2026-04-30
sources: [jihaichannel-interview-videos]
---

# 面試現場寫 Prompt 修復 Bug：AI 協作能力面試

極海Channel 揭示了一種新型技術面試題型：**面試官給你一段有 bug 的代碼，要求你在現場用 AI 工具（Claude/GPT）寫 Prompt 來找出並修復問題**。這不只考察你的 AI 工具使用能力，更考察你對代碼的理解深度和問題分解能力。

> 「這個面試題型測試的不是你能不能用 AI，而是你能不能在壓力下，用清晰的語言描述一個模糊的問題。」

---

## 為什麼這類面試題正在增加

傳統技術面試的邏輯是：「你能不能自己寫出正確的代碼？」

2025-2026 年，很多公司的邏輯變成：「你能不能和 AI 協作，快速解決真實問題？」

```
傳統面試考察的：
✓ 算法實現
✓ 數據結構選擇
✓ 邊界條件處理

AI 協作面試考察的：
✓ 問題描述能力（你能說清楚問題嗎？）
✓ AI 輸出評估（你能判斷 AI 的回答對不對嗎？）
✓ 迭代優化（AI 第一次給的答案不對，你能怎麼引導？）
✓ 代碼理解（你真的讀懂了 AI 生成的修復方案嗎？）
```

---

## 現場 Prompt 修復 Bug 的標準流程

假設面試官給你這段代碼，說「找出 bug 並修復」：

```python
def find_duplicates(nums):
    seen = set()
    duplicates = []
    for num in nums:
        if num in seen:
            duplicates.append(num)
        seen.add(num)
    return duplicates

# 測試：
print(find_duplicates([1, 2, 3, 2, 4, 3, 3]))
# 期望輸出：[2, 3]（每個重複的只出現一次）
# 實際輸出：[2, 3, 3]（3 出現了兩次）
```

### Step 1：自己先分析問題（1-2 分鐘）

**不要馬上開啟 AI。先自己讀代碼，找到問題所在。**

```
問自己：
1. 輸出 [2, 3, 3] 是為什麼？
   → 因為 3 出現了 3 次，第 2 次和第 3 次被看到 3 時都加入了 duplicates

2. 期望的行為是什麼？
   → 每個重複的數字只在 duplicates 裡出現一次

3. 解法方向是什麼？
   → 需要追蹤「已加入 duplicates 的數字」，避免重複加入
```

### Step 2：寫出精確的 Prompt

```
❌ 模糊 Prompt（AI 很難給出好回答）：
「這段代碼有 bug，幫我修復」

✅ 精確 Prompt：
「以下 Python 函數有一個 bug：

```python
def find_duplicates(nums):
    seen = set()
    duplicates = []
    for num in nums:
        if num in seen:
            duplicates.append(num)
        seen.add(num)
    return duplicates
```

問題：當一個數字出現 3 次以上時，它會在 duplicates 中出現多次。
期望行為：每個重複的數字在 duplicates 中只出現一次。
例如：find_duplicates([1,2,3,2,4,3,3]) 應返回 [2, 3]，而不是 [2, 3, 3]。

請修復這個 bug，並解釋修復的原理。」
```

### Step 3：評估 AI 的回答

AI 可能給出以下修復方案：

```python
# 方案一：用第二個 set 追蹤已加入的數字
def find_duplicates(nums):
    seen = set()
    duplicates = set()  # 改為 set，自動去重
    for num in nums:
        if num in seen:
            duplicates.add(num)
        seen.add(num)
    return list(duplicates)
```

你需要**當場評估**這個方案：

```
評估 checklist：
✓ 邏輯正確嗎？
  → 是，duplicates 用 set 自動去重

✓ 有沒有引入新問題？
  → 注意：list(set()) 的順序是不確定的！
    如果題目要求保持出現順序，這個方案不完整

✓ 時間/空間複雜度？
  → O(n) 時間，O(n) 空間，合理

✓ 邊界條件？
  → 空 list？→ 返回 []，正確
  → 全部不重複？→ 返回 []，正確
```

### Step 4：若 AI 方案有缺陷，迭代追問

```
「你的方案是對的，但它改變了元素的順序。
如果要求返回的列表按第一次出現重複的順序排列，
請修改方案。」
```

AI 修改後的方案：

```python
def find_duplicates(nums):
    seen = set()
    duplicates_set = set()
    duplicates = []
    for num in nums:
        if num in seen and num not in duplicates_set:
            duplicates.append(num)
            duplicates_set.add(num)
        seen.add(num)
    return duplicates
```

---

## 面試中展示 AI 協作能力的關鍵

### 大聲說出你的思考過程

面試官評估的不是最終答案，而是**你如何思考**：

```
「我先自己看一下這段代碼...
好，我發現問題在於當一個數字出現 3 次時，
它會被加入 duplicates 兩次。
我打算這樣描述給 AI：[說出你的 Prompt]

AI 給了這個方案...我來評估一下：
邏輯上是對的，但我注意到它使用了 set，
這可能改變返回順序。
如果題目有順序要求，我需要追問 AI。」
```

### 不要完全依賴 AI 的第一個回答

面試官最不希望看到的是：「AI 說這樣做，所以我就這樣做。」

展示你的**獨立判斷**：

```
即使 AI 的答案是對的，也要說：
「AI 建議用 [方案]。我認為這個方案是正確的，因為 [原因]。
它的時間複雜度是 O(n)，空間複雜度也是 O(n)，
對這道題的規模是合適的。
唯一需要注意的是 [潛在問題]，在我們的場景中 [是否需要處理]。」
```

---

## 常見的 Bug 類型和 Prompt 策略

### 並發 Bug

```
Prompt 模板：
「以下代碼在多線程環境下會有競態條件問題：
[代碼]
問題：[描述問題]
場景：[幾個線程，做什麼操作]
請識別競態條件所在，並提供線程安全的修復方案。
不使用 synchronized 的方案更佳（因為 [業務原因]）。」
```

### 性能 Bug（N+1 查詢）

```
Prompt 模板：
「以下代碼有性能問題，當 users 列表很大時會很慢：
[代碼]
問題：這裡有 N+1 查詢問題（每個用戶都單獨查詢一次 orders 表）
請提供一個批量查詢的修復方案，並說明查詢次數從 O(n) 降至 O(1)。」
```

### 內存洩漏

```
Prompt 模板：
「以下代碼在長時間運行後，JVM heap 會持續增長：
[代碼]
懷疑：[描述懷疑的原因]
請分析可能的內存洩漏原因，並提供修復方案。
請同時說明如何用工具驗證修復有效（例如 JVisualVM 或 Heap Dump 分析）。」
```

---

## 面試後的反思：AI 協作能力的本質

這類面試題真正考察的是：

```
1. 問題描述能力
   → 你能把「代碼跑出來不對」翻譯成精確的技術問題嗎？

2. 代碼理解能力
   → 你看懂了 AI 給的方案嗎？還是只是複製貼上？

3. 批判性思考
   → 你能評估 AI 方案的優缺點嗎？

4. 迭代引導能力
   → AI 第一次答案不滿意，你能繼續追問嗎？

這些能力，恰恰是 AI 時代最需要的工程師能力。
```

---

## 相關頁面

- [[大廠技術面試的底層邏輯]]
- [[AI時代新人如何自救]]
- [[Claude Prompt工程核心技巧]]
- [[Claude Prompt工程核心技巧]]（含「給 AI 交底而不是許願」章節）
