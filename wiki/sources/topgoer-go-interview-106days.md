---
title: "topgoer 面試題：Go 語言中文文檔・106 天面試題"
type: source-summary
tags: [golang, interview, code-puzzle, 八股文, gitbook]
created: 2026-06-29
updated: 2026-06-29
sources: [topgoer-go-interview-106days]
---

# topgoer 面試題（Go 語言中文文檔）

## Origin

- **Title**：面試題 · Go 語言中文文檔（mian.topgoer.com）
- **Platform**：[[topgoer]] — 中文 Go 學習文檔站（GitBook 架構）
- **內容來源**：轉自公眾號「Golang 來啦」；部分題目引自知識星球《Go 項目》
- **Language**：簡體中文
- **結構**：第一天 → 第一百零六天（106 個單元），標註「持續更新中」
- **URL**：http://mian.topgoer.com/ （⚠️ HTTPS 憑證已過期，需用 http 抓取）

## Key Takeaways

1. **體裁是「代碼輸出陷阱題」**：絕大多數題型為「下面這段代碼輸出什麼，說明原因」或單選（A/B/compilation error），重點在**踩坑點 + 原因解析**，而非演算法。屬典型 Go 面試「八股文」/ gotcha 題庫。
2. **規模**：106 天 × 每天數題，依日序鬆散排列（無主題分類），第一天為 HR/行為題（自我介紹、離職原因、薪資期望等 40 題），技術題從第二天起。
3. **高頻考點橫切**（從抽樣各天歸納）：
   - **for-range 變數捕獲**：`m[key] = &val` 取的都是同一個迴圈變數 `val` 的位址 → map 全部指向最後一次的值（第二天經典題）。
   - **型別系統 / 可賦值性**：`int + float` 不能相加（編譯錯誤）；具名型別（named type）與底層型別相同仍不可互賦，除非至少一方非具名型別（第十、第九十天，附 Go spec 原文）。
   - **三索引切片** `a[3:4:4]`：low:high:max 控制容量（第十天）。
   - **defer / recover / 命名返回值互動**：`defer` 修改具名返回值 `r`、未初始化的 `defer f()` 觸發 panic 後被 recover——「三步拆解法」算最終返回值（第三十天，答案 7）。
   - **嵌入結構 + 指標方法 nil 解引用**：`S{ *T }` 零值時 `s.bar()` 展開為 `(*s.T).bar()`，`s.T` 為 nil → panic（第六十天）。
4. **解析品質**：每題附「參考答案及解析」，常引用 Go 語言規範手冊原文與外部文章（如「5 年 Gopher 都不知道的 defer 細節」），適合用來自測與補洞，而非系統化學習。

## Entities mentioned

- [[topgoer]] — 承載本題庫的 Go 中文文檔平台

## Concepts mentioned

- [[Go面試陷阱題彙整]] — 本源萃取出的高頻陷阱題型彙整（本次新建）
- [[Go defer與panic]] — defer/recover/命名返回值互動的系統化版本
- [[Go執行期內部機制]]、[[Go同步原語與記憶體模型]] — 部分題目的底層原理依據

## Contradictions / tensions

- **與既有 Go 面試頁的角色差異**：[[Go defer與panic]]、[[Go執行期內部機制]] 等是「系統化知識」取向；本源是「碎片化 gotcha 自測題」取向。兩者互補：前者建立心智模型，後者驗證盲點。
- **時效性風險**：「持續更新中」但無日期標註，部分題目年代久遠（pre-generics 時代風格較重）；Go 1.22+ 已修正 for-range 迴圈變數語意（每次迭代新建變數），故第二天「經典陷阱」在新版 Go 行為**已改變**——使用此題庫需注意 Go 版本前提。

## Questions raised

- 是否值得逐天抽取全部 106 天題目入庫？（建議：否；以「陷阱題型彙整」概念頁收斂高頻 pattern 即可，逐題冗長且重複）
- 哪些題目因 Go 1.22 迴圈變數語意變更 / 泛型引入而**過時或答案改變**？值得單獨標注。
- 與 [[Coinbase面試題與Go解答]]、[[HashiCorp_Banking_System_OA]] 等「實作題」如何分工？本源補的是「語言細節 / 編譯器行為」維度。
