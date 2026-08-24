---
title: Coinbase 面試題與 Go 解答
type: concept
tags: [面試, Coinbase, Golang, 機器碼, 系統設計, OOP, 算法, CodeSignal]
created: 2026-04-30
updated: 2026-04-30
sources: []
---

# Coinbase 面試題與 Go 解答

Coinbase 是加密貨幣領域頭部公司，面試側重**生產級代碼品質**而非純 LeetCode 最優解。面試官更在意：類的劃分、錯誤處理、命名規範、狀態管理，而不只是算法的正確性。

---

## 面試流程

```
1. Online Assessment（CodeSignal，90 分鐘）
   └── 4 道題：第 1 題熱身，後 3 題 LeetCode Medium 難度

2. Technical Interview 1（Technical Execution Round，60 分鐘）
   ├── Machine Coding：從零實現一個功能模組（帶多輪 Extension）
   └── DSA：算法題 1-2 道

3. Technical Interview 2（Domain Execution Round，60 分鐘）
   ├── Machine Coding：OOP 設計，強調類劃分和代碼整潔
   └── System Design：輕量版系統設計討論

4. Behavioral Interview（45 分鐘）
```

**重點**：Machine Coding 不是「能跑就好」，面試官要看的是：

```
✓ 類的邊界劃分是否合理
✓ 方法命名是否清晰表達意圖
✓ 錯誤處理是否完整
✓ 狀態管理是否安全（並發考量）
✓ 能否在接到 Extension 時快速擴展（開放-封閉原則）
```

---

## 題目一：In-Memory Banking System（CodeSignal OA 最高頻）

### 題目描述

實現一個內存銀行系統，支持以下操作（所有操作帶 timestamp，按時間順序輸入）：

```
createAccount(timestamp, accountId) → bool
deposit(timestamp, accountId, amount) → string（新餘額）
transfer(timestamp, fromId, toId, amount) → string（發送方新餘額）
topSpenders(timestamp, n) → string（前 n 名累計支出用戶）
schedulePayment(timestamp, accountId, toId, delay, amount) → paymentId
cancelPayment(timestamp, paymentId) → bool
getBalance(timestamp, accountId, atTimestamp) → string（歷史餘額）
```

### Go 解答

```go
package banking

import (
	"fmt"
	"sort"
	"strconv"
)

type Transaction struct {
	timestamp int
	amount    int // positive = deposit, negative = withdrawal
	balance   int // balance after this transaction
}

type Account struct {
	id           string
	balance      int
	totalSpent   int
	transactions []Transaction
}

type ScheduledPayment struct {
	id        string
	fromID    string
	toID      string
	executeAt int
	amount    int
	cancelled bool
}

type Bank struct {
	accounts        map[string]*Account
	payments        map[string]*ScheduledPayment
	paymentCounter  int
}

func NewBank() *Bank {
	return &Bank{
		accounts: make(map[string]*Account),
		payments: make(map[string]*ScheduledPayment),
	}
}

func (b *Bank) processScheduledPayments(timestamp int) {
	for _, p := range b.payments {
		if !p.cancelled && p.executeAt <= timestamp {
			from, ok1 := b.accounts[p.fromID]
			to, ok2 := b.accounts[p.toID]
			if ok1 && ok2 && from.balance >= p.amount {
				from.balance -= p.amount
				from.totalSpent += p.amount
				from.transactions = append(from.transactions, Transaction{p.executeAt, -p.amount, from.balance})
				to.balance += p.amount
				to.transactions = append(to.transactions, Transaction{p.executeAt, p.amount, to.balance})
			}
			p.cancelled = true // mark executed
		}
	}
}

func (b *Bank) CreateAccount(timestamp int, accountID string) bool {
	b.processScheduledPayments(timestamp)
	if _, exists := b.accounts[accountID]; exists {
		return false
	}
	b.accounts[accountID] = &Account{
		id:           accountID,
		transactions: []Transaction{{timestamp, 0, 0}},
	}
	return true
}

func (b *Bank) Deposit(timestamp int, accountID string, amount int) string {
	b.processScheduledPayments(timestamp)
	acc, ok := b.accounts[accountID]
	if !ok {
		return ""
	}
	acc.balance += amount
	acc.transactions = append(acc.transactions, Transaction{timestamp, amount, acc.balance})
	return strconv.Itoa(acc.balance)
}

func (b *Bank) Transfer(timestamp int, fromID, toID string, amount int) string {
	b.processScheduledPayments(timestamp)
	from, ok1 := b.accounts[fromID]
	to, ok2 := b.accounts[toID]
	if !ok1 || !ok2 || fromID == toID {
		return ""
	}
	if from.balance < amount {
		return ""
	}
	from.balance -= amount
	from.totalSpent += amount
	from.transactions = append(from.transactions, Transaction{timestamp, -amount, from.balance})
	to.balance += amount
	to.transactions = append(to.transactions, Transaction{timestamp, amount, to.balance})
	return strconv.Itoa(from.balance)
}

func (b *Bank) TopSpenders(timestamp int, n int) string {
	b.processScheduledPayments(timestamp)
	type entry struct {
		id    string
		spent int
	}
	entries := make([]entry, 0, len(b.accounts))
	for id, acc := range b.accounts {
		entries = append(entries, entry{id, acc.totalSpent})
	}
	sort.Slice(entries, func(i, j int) bool {
		if entries[i].spent != entries[j].spent {
			return entries[i].spent > entries[j].spent
		}
		return entries[i].id < entries[j].id
	})
	if n > len(entries) {
		n = len(entries)
	}
	result := ""
	for i := 0; i < n; i++ {
		if i > 0 {
			result += ", "
		}
		result += fmt.Sprintf("%s(%d)", entries[i].id, entries[i].spent)
	}
	return result
}

func (b *Bank) SchedulePayment(timestamp int, fromID, toID string, delay, amount int) string {
	b.processScheduledPayments(timestamp)
	if _, ok := b.accounts[fromID]; !ok {
		return ""
	}
	b.paymentCounter++
	id := fmt.Sprintf("payment%d", b.paymentCounter)
	b.payments[id] = &ScheduledPayment{
		id:        id,
		fromID:    fromID,
		toID:      toID,
		executeAt: timestamp + delay,
		amount:    amount,
	}
	return id
}

func (b *Bank) CancelPayment(timestamp int, paymentID string) bool {
	b.processScheduledPayments(timestamp)
	p, ok := b.payments[paymentID]
	if !ok || p.cancelled {
		return false
	}
	p.cancelled = true
	return true
}

// GetBalance returns the balance at or just before atTimestamp
func (b *Bank) GetBalance(timestamp int, accountID string, atTimestamp int) string {
	b.processScheduledPayments(timestamp)
	acc, ok := b.accounts[accountID]
	if !ok {
		return ""
	}
	// Binary search for the last transaction at or before atTimestamp
	balance := -1
	for _, tx := range acc.transactions {
		if tx.timestamp <= atTimestamp {
			balance = tx.balance
		} else {
			break
		}
	}
	if balance < 0 {
		return ""
	}
	return strconv.Itoa(balance)
}
```

**設計要點**：
- `processScheduledPayments` 在每次操作前觸發，確保排程付款按時執行
- 每個 Account 保存完整的 `transactions` 歷史，支持時間點查詢
- Transfer 前必須檢查餘額，避免透支
- TopSpenders 按累計支出降序，同支出按 ID 字母升序

---

## 題目二：Connect 4（Machine Coding OOP 高頻題）

### 題目描述

實現 Connect 4 遊戲：
- 6 行 7 列的棋盤
- 兩名玩家輪流投入棋子，棋子因重力落到最低空位
- 率先在水平、垂直或對角線方向連成 4 子者獲勝

### Go 解答（強調 OOP）

```go
package connect4

import "errors"

type Player int

const (
	Empty  Player = 0
	Yellow Player = 1
	Red    Player = 2
)

func (p Player) String() string {
	switch p {
	case Yellow:
		return "Yellow"
	case Red:
		return "Red"
	default:
		return "."
	}
}

const (
	rows    = 6
	cols    = 7
	winLen  = 4
)

type Board struct {
	grid [rows][cols]Player
}

func (b *Board) Drop(col int, player Player) (int, error) {
	if col < 0 || col >= cols {
		return -1, errors.New("column out of range")
	}
	for row := rows - 1; row >= 0; row-- {
		if b.grid[row][col] == Empty {
			b.grid[row][col] = player
			return row, nil
		}
	}
	return -1, errors.New("column is full")
}

func (b *Board) CheckWin(row, col int, player Player) bool {
	directions := [][2]int{{0, 1}, {1, 0}, {1, 1}, {1, -1}}
	for _, d := range directions {
		if b.countLine(row, col, d[0], d[1], player) >= winLen {
			return true
		}
	}
	return false
}

func (b *Board) countLine(row, col, dr, dc int, player Player) int {
	count := 1
	for i := 1; i < winLen; i++ {
		r, c := row+dr*i, col+dc*i
		if r < 0 || r >= rows || c < 0 || c >= cols || b.grid[r][c] != player {
			break
		}
		count++
	}
	for i := 1; i < winLen; i++ {
		r, c := row-dr*i, col-dc*i
		if r < 0 || r >= rows || c < 0 || c >= cols || b.grid[r][c] != player {
			break
		}
		count++
	}
	return count
}

func (b *Board) IsFull() bool {
	for col := 0; col < cols; col++ {
		if b.grid[0][col] == Empty {
			return false
		}
	}
	return true
}

type GameState int

const (
	InProgress GameState = iota
	Win
	Draw
)

type Game struct {
	board         Board
	currentPlayer Player
	state         GameState
	winner        Player
	moveCount     int
}

func NewGame() *Game {
	return &Game{currentPlayer: Yellow}
}

func (g *Game) Drop(col int) (GameState, Player, error) {
	if g.state != InProgress {
		return g.state, g.winner, errors.New("game is already over")
	}
	row, err := g.board.Drop(col, g.currentPlayer)
	if err != nil {
		return g.state, Empty, err
	}
	g.moveCount++
	if g.board.CheckWin(row, col, g.currentPlayer) {
		g.state = Win
		g.winner = g.currentPlayer
		return Win, g.winner, nil
	}
	if g.board.IsFull() {
		g.state = Draw
		return Draw, Empty, nil
	}
	g.switchPlayer()
	return InProgress, Empty, nil
}

func (g *Game) switchPlayer() {
	if g.currentPlayer == Yellow {
		g.currentPlayer = Red
	} else {
		g.currentPlayer = Yellow
	}
}

func (g *Game) CurrentPlayer() Player { return g.currentPlayer }
func (g *Game) State() GameState      { return g.state }
func (g *Game) Winner() Player        { return g.winner }
```

**設計要點**：
- `Board` 和 `Game` 分離：Board 管棋盤狀態，Game 管遊戲流程
- `CheckWin` 只需從最後落子位置向四個方向延伸，無需掃描整個棋盤，O(1)
- 方法名清晰表達意圖（`Drop`、`CheckWin`、`IsFull`）
- 錯誤通過 `error` 返回而非 panic

**Extension 方向**（面試中常見的追問）：

```go
// Extension 1: 支持任意棋盤大小和勝利條件
type Config struct {
    Rows, Cols, WinLen int
}

// Extension 2: 支持多人（3人以上）
// 只需把 Player 從 enum 改成 interface

// Extension 3: AI（Minimax）
func (b *Board) BestMove(player Player, depth int) int { ... }
```

---

## 題目三：Currency Exchange（外匯兌換，圖的最優路徑）

### 題目描述

給定貨幣兌換關係列表，每條記錄為 `(from, to, rate)`。查詢從貨幣 A 到貨幣 B 的最優兌換率（乘積最大）。

```
Input: [("USD", "BTC", 0.000021), ("BTC", "ETH", 15.2), ("ETH", "USD", 2800)]
Query: "USD" → "ETH"
Output: 0.000021 × 15.2 = 0.0003192
```

### Go 解答（BFS + 圖建模）

```go
package exchange

import "math"

type ExchangeSystem struct {
	rates map[string]map[string]float64
}

func NewExchangeSystem() *ExchangeSystem {
	return &ExchangeSystem{
		rates: make(map[string]map[string]float64),
	}
}

func (e *ExchangeSystem) AddRate(from, to string, rate float64) {
	if e.rates[from] == nil {
		e.rates[from] = make(map[string]float64)
	}
	if e.rates[to] == nil {
		e.rates[to] = make(map[string]float64)
	}
	e.rates[from][to] = rate
	e.rates[to][from] = 1.0 / rate // 反向匯率自動計算
}

type state struct {
	currency string
	rate     float64
}

// BestRate 返回從 start 到 end 的最優（最大）兌換率
// 使用 BFS + 貪心（不適用負權重，所以不需要 Bellman-Ford）
func (e *ExchangeSystem) BestRate(start, end string) float64 {
	if start == end {
		return 1.0
	}
	if _, ok := e.rates[start]; !ok {
		return -1
	}

	// BFS：用最大堆模擬（這裡簡化為普通 BFS，適合稀疏圖）
	// 對於需要保證最優解的場景，使用 Dijkstra（取 log 後轉最短路）
	visited := make(map[string]float64)
	visited[start] = 1.0
	queue := []state{{start, 1.0}}

	best := -1.0
	for len(queue) > 0 {
		cur := queue[0]
		queue = queue[1:]

		if cur.currency == end {
			if cur.rate > best {
				best = cur.rate
			}
			continue
		}

		for next, rate := range e.rates[cur.currency] {
			newRate := cur.rate * rate
			if prev, seen := visited[next]; !seen || newRate > prev {
				visited[next] = newRate
				queue = append(queue, state{next, newRate})
			}
		}
	}
	return best
}

// BestRateDijkstra 使用 Dijkstra 求最優兌換率（取對數轉換為最短路問題）
func (e *ExchangeSystem) BestRateDijkstra(start, end string) float64 {
	if start == end {
		return 1.0
	}
	// 取 -log(rate)，最大乘積 → 最小和
	dist := make(map[string]float64)
	for k := range e.rates {
		dist[k] = math.Inf(1)
	}
	dist[start] = 0

	// 簡化版 Dijkstra（面試中可以用 visited set 替代優先隊列）
	visited := make(map[string]bool)

	for i := 0; i < len(e.rates); i++ {
		// 找未訪問的最小距離節點
		u := ""
		for node := range e.rates {
			if !visited[node] && (u == "" || dist[node] < dist[u]) {
				u = node
			}
		}
		if u == "" || math.IsInf(dist[u], 1) {
			break
		}
		visited[u] = true

		for v, rate := range e.rates[u] {
			newDist := dist[u] + (-math.Log(rate))
			if newDist < dist[v] {
				dist[v] = newDist
			}
		}
	}

	if math.IsInf(dist[end], 1) {
		return -1
	}
	return math.Exp(-dist[end])
}
```

**設計要點**：
- 自動計算反向匯率（`AddRate` 中同時加入 from→to 和 to→from）
- BFS 版本適合簡單圖，Dijkstra 版本保證最優解
- 取 `-log(rate)` 將「最大乘積問題」轉為「最短路問題」

---

## 題目四：Skip Iterator（設計模式，CodeSignal 高頻）

### 題目描述

實現一個 `SkipIterator`，包裝任意 Iterator，支持 `skip(val)` 操作：下次遇到 `val` 時跳過一次。

```
iter = SkipIterator([1, 2, 3, 3, 4])
iter.skip(3)   // 下次遇到 3 跳過
iter.next()    // → 1
iter.next()    // → 2
iter.next()    // → 3（第二個 3，第一個被跳過了）
iter.next()    // → 4
```

### Go 解答

```go
package iterator

type Iterator interface {
	HasNext() bool
	Next() int
}

type SliceIterator struct {
	data  []int
	index int
}

func NewSliceIterator(data []int) *SliceIterator {
	return &SliceIterator{data: data}
}

func (it *SliceIterator) HasNext() bool {
	return it.index < len(it.data)
}

func (it *SliceIterator) Next() int {
	val := it.data[it.index]
	it.index++
	return val
}

type SkipIterator struct {
	inner     Iterator
	skipCount map[int]int
	nextVal   *int // peeked value
}

func NewSkipIterator(inner Iterator) *SkipIterator {
	return &SkipIterator{
		inner:     inner,
		skipCount: make(map[int]int),
	}
}

func (s *SkipIterator) Skip(val int) {
	s.skipCount[val]++
}

func (s *SkipIterator) HasNext() bool {
	if s.nextVal != nil {
		return true
	}
	for s.inner.HasNext() {
		v := s.inner.Next()
		if count := s.skipCount[v]; count > 0 {
			s.skipCount[v]--
			if s.skipCount[v] == 0 {
				delete(s.skipCount, v)
			}
			continue
		}
		s.nextVal = &v
		return true
	}
	return false
}

func (s *SkipIterator) Next() int {
	if !s.HasNext() {
		panic("no more elements")
	}
	val := *s.nextVal
	s.nextVal = nil
	return val
}
```

**設計要點**：
- `skipCount` 用 map 存「每個值還需要跳過幾次」，支持同一值被 skip 多次
- `nextVal` 是 peek buffer，讓 `HasNext()` 具備「預取」能力
- 將 skip 邏輯封裝在 `HasNext` 中，保持 `Next` 的乾淨

---

## 題目五：Buy/Sell Stock with K Transactions（CodeSignal DSA 題）

### 題目描述

給定股票每日價格，最多允許 k 次交易（每次交易：買入 + 賣出），求最大利潤。每次必須賣出後才能買入。

### Go 解答

```go
package stocks

func MaxProfitKTransactions(prices []int, k int) int {
	n := len(prices)
	if n == 0 || k == 0 {
		return 0
	}

	// 如果 k >= n/2，等同於無限次交易
	if k >= n/2 {
		return maxProfitUnlimited(prices)
	}

	// dp[i][j] = 最多 i 次交易，到第 j 天的最大利潤
	// 但優化空間：只需要當前交易次數的狀態
	// hold[i] = 持有股票時，第 i 次交易的最大利潤
	// sold[i] = 已賣出時，第 i 次交易的最大利潤
	hold := make([]int, k+1)
	sold := make([]int, k+1)
	for i := range hold {
		hold[i] = -1 << 30
	}

	for _, price := range prices {
		for i := k; i >= 1; i-- {
			// 賣出：從持有狀態賣出
			if sold[i] < hold[i]+price {
				sold[i] = hold[i] + price
			}
			// 買入：從上一次賣出狀態買入（消耗一次交易機會）
			if hold[i] < sold[i-1]-price {
				hold[i] = sold[i-1] - price
			}
		}
	}

	return sold[k]
}

func maxProfitUnlimited(prices []int) int {
	profit := 0
	for i := 1; i < len(prices); i++ {
		if prices[i] > prices[i-1] {
			profit += prices[i] - prices[i-1]
		}
	}
	return profit
}
```

---

## 題目六：Balanced Parentheses with Wildcards（CodeSignal 高頻）

### 題目描述

給定一個包含 `(`、`)` 和 `*` 的字符串，`*` 可以是 `(`、`)` 或空字符串，判斷括號是否合法。

### Go 解答

```go
package parens

func CheckValidParentheses(s string) bool {
	// lo：可能的最小開放括號數
	// hi：可能的最大開放括號數
	lo, hi := 0, 0
	for _, c := range s {
		switch c {
		case '(':
			lo++
			hi++
		case ')':
			lo--
			hi--
		case '*':
			lo-- // * 當作 )
			hi++ // * 當作 (
		}
		if hi < 0 {
			return false // 無論 * 怎麼解釋，右括號都多了
		}
		if lo < 0 {
			lo = 0 // 最小不能是負數
		}
	}
	return lo == 0 // 最少情況下，開放括號數為 0
}
```

**原理**：用區間 `[lo, hi]` 追蹤可能的未閉合括號數量。`*` 讓 lo 減一（最樂觀：當右括號用）、hi 加一（最悲觀：當左括號用）。

---

## 題目七：Graph Shortest Path with Edge Failures（進階題）

### 題目描述

給定圖和邊的列表，找從 source 到 destination 的最短路徑。但有些邊可能失效（故障），需要找出在邊故障情況下仍能到達目的地的最短路。

### Go 解答（Dijkstra）

```go
package graph

import (
	"container/heap"
	"math"
)

type Edge struct {
	To     int
	Weight int
	ID     int // 邊 ID，用於標記失效
}

type PriorityItem struct {
	node int
	dist int
	idx  int
}

type PQ []PriorityItem

func (pq PQ) Len() int            { return len(pq) }
func (pq PQ) Less(i, j int) bool  { return pq[i].dist < pq[j].dist }
func (pq PQ) Swap(i, j int)       { pq[i], pq[j] = pq[j], pq[i] }
func (pq *PQ) Push(x interface{}) { *pq = append(*pq, x.(PriorityItem)) }
func (pq *PQ) Pop() interface{} {
	old := *pq
	n := len(old)
	x := old[n-1]
	*pq = old[:n-1]
	return x
}

func ShortestPath(graph [][]Edge, src, dst int, failedEdges map[int]bool) int {
	n := len(graph)
	dist := make([]int, n)
	for i := range dist {
		dist[i] = math.MaxInt64
	}
	dist[src] = 0

	pq := &PQ{{src, 0, 0}}
	heap.Init(pq)

	for pq.Len() > 0 {
		cur := heap.Pop(pq).(PriorityItem)
		if cur.dist > dist[cur.node] {
			continue
		}
		for _, e := range graph[cur.node] {
			if failedEdges[e.ID] {
				continue
			}
			newDist := dist[cur.node] + e.Weight
			if newDist < dist[e.To] {
				dist[e.To] = newDist
				heap.Push(pq, PriorityItem{e.To, newDist, 0})
			}
		}
	}

	if dist[dst] == math.MaxInt64 {
		return -1
	}
	return dist[dst]
}
```

---

## 面試中的 Go 代碼風格要點

Coinbase 面試官特別注意以下生產代碼規範：

```go
// ❌ 面試中常見的不佳風格
func process(d []int, k int) int {
    r := 0
    for i := 0; i < len(d); i++ {
        if d[i] > k { r += d[i] }
    }
    return r
}

// ✓ 面試中展示的生產風格
func sumValuesAboveThreshold(data []int, threshold int) int {
    total := 0
    for _, val := range data {
        if val > threshold {
            total += val
        }
    }
    return total
}
```

```
生產代碼五要素：
1. 命名清晰：變量名表達含義，不用單字母（除非是慣用的 i/j/k）
2. 錯誤處理：不 panic，返回 error
3. 類型劃分：相關數據和操作封裝為 struct + method
4. 邊界防守：nil check、空 slice、越界保護
5. 並發安全：共享狀態加 sync.Mutex（若面試官問到高並發場景）
```

---

## 備考建議

```
OA（CodeSignal）：
- 重點：Banking System（最常考）、Stock Trading、括號匹配
- 練習：LeetCode 346/362（Moving Average）、303（Range Sum）、155（Min Stack）

Machine Coding 輪：
- Connect 4 → 熟悉到可以 30 分鐘內寫出乾淨的 OOP 版本
- 設計原則：SOLID 中的 SRP（單一職責）和 OCP（開放封閉）
- 每個 Extension 出來先說設計再寫代碼

算法輪（DSA）：
- Graph（BFS/DFS/Dijkstra）：貨幣兌換、最短路
- DP：股票交易、揹包問題
- Sliding Window：滑動視窗最大值、最長子串
```

---

## 相關頁面

- [[九分鐘看懂系統設計面試]]
- [[12306系統設計面試解題框架]]
- [[校招生如何吊打大廠面試官]]
- [[大廠技術面試的底層邏輯]]
