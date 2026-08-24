---
title: Skills 實戰：自動交易機器人
type: concept
tags: [claude, skills, trading, 自動交易, 期貨, 台股, 風控, MCP]
created: 2026-04-30
updated: 2026-04-30
sources: []
---

# Skills 實戰：自動交易機器人

本教程用 Claude Code Skills 建立一套**半自動交易輔助系統**：Claude 負責訊號分析、風控檢查、策略建議，**下單動作永遠由人工確認**，或設定在嚴格的規則框架內自動執行。

> 核心原則：**Claude 是分析師，不是交易員。真正的執行決策權在你手上。**
>
> 延伸閱讀：[[Vibe Coding程式交易實戰]] — AI 輔助交易的 CLAUDE.md 護欄設計

---

## 整體架構

```
Claude Code
    │
    ├── market-scan (Skill)      ← 掃描訊號，每天/每小時
    │       └── FinMind API / yfinance → 抓報價、計算指標
    │
    ├── risk-check (Skill)       ← 下單前的風控確認
    │       └── 讀取當前帳戶狀態、計算部位大小、確認風控
    │
    ├── trade-plan (Skill)       ← 生成完整交易計畫（含進出場、停損）
    │       └── Claude 分析 → 輸出結構化計畫 → 人工確認
    │
    └── post-trade (Skill)       ← 交易後記錄與覆盤
            └── 記錄交易、計算損益、更新績效統計
```

---

## CLAUDE.md 交易系統規範

**這是最重要的一步，先設定邊界才能動其他東西。**

```markdown
# 交易機器人 — Claude 行為規範

## ⛔ 絕對禁止（無論任何情況）
- 直接呼叫任何券商 API 的下單功能（broker.place_order / 任何 POST /orders）
- 修改 strategy/core/ 下的任何檔案
- 修改策略參數（strategy/params.py 或 config/strategy.yaml）
- 在沒有我明確確認的情況下執行任何資金操作

## ✅ 可以自主執行（不需確認）
- 讀取市場資料（GET 類請求）
- 計算技術指標（MA、RSI、ATR、布林通道等）
- 讀取帳戶資訊（查詢，不是操作）
- 將分析結果寫入 reports/ 或 signals/
- 執行回測（讀取歷史資料，不碰實盤）

## 📋 需要我確認才能執行
- 生成交易計畫（我要說「確認計畫」才算通過）
- 發出下單建議（我要說「確認下單」才算通過）
- 修改停損/停利設定

## 🚨 遇到以下情況立刻停止並通知
- 訊號強烈但和上週回測方向矛盾
- 帳戶可用資金低於 50%
- 任何 API 錯誤或連線問題
- 計算出的部位大小超過帳戶 10%

## 高風險區域（只有我能改）
- strategy/core/      ← 進出場邏輯
- strategy/params.py  ← 策略參數
- broker/             ← 券商 API 介面
```

---

## 前置準備

### 安裝依賴

```bash
pip install finmind yfinance pandas numpy ta-lib python-dotenv requests

# 若使用台指期（元大 API）：
pip install sinopac-python-api  # 永豐金 API
# 或
pip install fubon-neo            # 富邦 API

# 若使用加密貨幣：
pip install ccxt
```

### 設定環境變數

```bash
# .env
# 台股資料來源
FINMIND_API_KEY=your_finmind_key

# 券商（選一個）
SHIOAJI_PERSON_ID=xxx          # 永豐金 shioaji
SHIOAJI_PASSWORD=xxx
FUBON_ACCOUNT=xxx              # 富邦
FUBON_PASSWORD=xxx

# 加密貨幣（選一個）
BINANCE_API_KEY=xxx
BINANCE_SECRET=xxx

# 通知（可選）
LINE_NOTIFY_TOKEN=xxx          # LINE 通知
```

---

## 專案結構

```
trading-bot/
├── CLAUDE.md
├── .claude/
│   ├── settings.json
│   └── skills/
│       ├── market-scan.md
│       ├── risk-check.md
│       ├── trade-plan.md
│       └── post-trade.md
│
├── strategy/                  ← 🔒 禁區（Claude 不修改）
│   ├── core/
│   │   ├── entry.py          # 進場條件
│   │   ├── exit.py           # 出場條件
│   │   ├── stop_loss.py      # 停損邏輯
│   │   └── sizing.py         # 部位大小計算
│   └── params.py             # 策略參數
│
├── infrastructure/            ← ✅ Claude 可以碰
│   ├── data/
│   │   ├── fetcher.py        # 市場資料抓取
│   │   └── indicators.py     # 技術指標計算
│   ├── broker/
│   │   ├── base.py           # 介面定義
│   │   ├── paper.py          # 模擬交易（預設）
│   │   └── shioaji.py        # 永豐金實盤（需手動啟用）
│   ├── risk/
│   │   └── checker.py        # 風控計算
│   └── reports/
│       └── generator.py      # 報表生成
│
├── signals/                   ← Claude 寫入訊號分析結果
├── plans/                     ← Claude 寫入交易計畫（等待確認）
├── trades/                    ← 已確認並執行的交易記錄
└── reports/                   ← 績效報表
```

---

## Skill 1：market-scan（市場掃描）

**檔案位置**：`.claude/skills/market-scan.md`

```markdown
# market-scan

## 觸發條件
當用戶說「掃市場」、「看今天訊號」、「scan」時使用
也可以排程：每天早上 8:45（台股開盤前 15 分鐘）自動觸發

## 掃描範圍（根據 params.py 設定）
- 台股：讀取 strategy/params.py 的 WATCHLIST
- 台指期：讀取 TXFR1 近月合約報價
- 加密貨幣：讀取 BTC/ETH 等設定的幣對

## 執行步驟

1. 抓取報價資料
   `python infrastructure/data/fetcher.py --symbols all --period 90d`

2. 計算技術指標
   `python infrastructure/data/indicators.py --input data/latest.csv`

3. 套用進場條件篩選
   `python strategy/core/entry.py --scan`（只讀，不會修改任何檔案）

4. 把符合條件的訊號寫入 signals/<YYYYMMDD>.json

5. 顯示摘要報告：
   - 符合進場條件的標的（股票代碼 + 訊號強度）
   - 目前持倉狀態
   - 整體市場氛圍（大盤位置、VIX/VVIX 水位）
   - 風險提示（若有）

## 輸出格式
```json
{
  "scan_time": "2026-04-30T08:45:00+08:00",
  "market_overview": {
    "taiex": 21543.0,
    "taiex_ma20": 21200.0,
    "trend": "上升趨勢",
    "vix": 18.5
  },
  "signals": [
    {
      "symbol": "2330",
      "name": "台積電",
      "signal": "LONG",
      "strength": 0.82,
      "entry_price": 1055.0,
      "suggested_stop": 1020.0,
      "reason": "突破 20MA，RSI=58，成交量放大 1.8x"
    }
  ],
  "open_positions": [...],
  "warnings": []
}
```

## 允許的工具
allowed-tools: 
  Bash(python infrastructure/data/fetcher.py:*),
  Bash(python infrastructure/data/indicators.py:*),
  Bash(python strategy/core/entry.py --scan),
  Write(signals/*),
  Read(strategy/params.py),
  Read(trades/*)

## 禁止
- 不呼叫任何下單相關函數
- 不修改 strategy/ 下的任何檔案
```

### 資料抓取腳本

```python
# infrastructure/data/fetcher.py
import os, json, argparse
from datetime import datetime, timedelta
from pathlib import Path
import pandas as pd

try:
    from FinMind.data import DataLoader  # 台股優先用 FinMind
    FINMIND_AVAILABLE = True
except ImportError:
    FINMIND_AVAILABLE = False

try:
    import yfinance as yf  # 備援或美股/加密貨幣
    YFINANCE_AVAILABLE = True
except ImportError:
    YFINANCE_AVAILABLE = False


def fetch_tw_stock(symbol: str, days: int = 90) -> pd.DataFrame:
    """抓取台股日 K"""
    if not FINMIND_AVAILABLE:
        raise RuntimeError("請安裝 FinMind：pip install FinMind")
    
    api = DataLoader()
    api_key = os.environ.get("FINMIND_API_KEY", "")
    if api_key:
        api.login_by_token(api_token=api_key)
    
    end = datetime.today()
    start = end - timedelta(days=days)
    
    df = api.taiwan_stock_daily(
        stock_id=symbol,
        start_date=start.strftime("%Y-%m-%d"),
        end_date=end.strftime("%Y-%m-%d"),
    )
    df = df[["date", "open", "max", "min", "close", "Trading_Volume"]].copy()
    df.columns = ["date", "open", "high", "low", "close", "volume"]
    return df.sort_values("date").reset_index(drop=True)


def fetch_crypto(symbol: str, days: int = 90) -> pd.DataFrame:
    """抓取加密貨幣日 K（用 yfinance）"""
    ticker = yf.Ticker(f"{symbol}-USD")
    df = ticker.history(period=f"{days}d")
    df = df[["Open", "High", "Low", "Close", "Volume"]].copy()
    df.columns = ["open", "high", "low", "close", "volume"]
    df.index.name = "date"
    return df.reset_index()


if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--symbols", default="all")
    parser.add_argument("--period", default="90d")
    args = parser.parse_args()

    days = int(args.period.replace("d", ""))
    
    # 讀取監控清單
    import importlib.util
    spec = importlib.util.spec_from_file_location("params", "strategy/params.py")
    params = importlib.util.module_from_spec(spec)
    spec.loader.exec_module(params)
    
    watchlist = params.WATCHLIST if args.symbols == "all" else args.symbols.split(",")
    
    Path("data").mkdir(exist_ok=True)
    results = {}
    for sym in watchlist:
        try:
            df = fetch_tw_stock(sym, days)
            out = f"data/{sym}.csv"
            df.to_csv(out, index=False)
            results[sym] = {"status": "ok", "rows": len(df), "file": out}
            print(f"✅ {sym}: {len(df)} 筆資料")
        except Exception as e:
            results[sym] = {"status": "error", "error": str(e)}
            print(f"❌ {sym}: {e}")
    
    Path("data/latest_fetch.json").write_text(
        json.dumps(results, ensure_ascii=False, indent=2)
    )
```

---

## Skill 2：risk-check（下單前風控）

**檔案位置**：`.claude/skills/risk-check.md`

```markdown
# risk-check

## 觸發條件
自動在 trade-plan 執行後觸發
或當用戶說「風控檢查」、「risk check」時

## 執行步驟

1. 讀取帳戶狀態（只讀）：
   `python infrastructure/broker/paper.py --status`（模擬）
   或 `python infrastructure/broker/shioaji.py --status`（實盤，只讀）

2. 讀取擬定的交易計畫（plans/ 最新檔案）

3. 執行風控計算：
   `python infrastructure/risk/checker.py --plan plans/latest.json`

4. 輸出風控報告，包含：
   - 計畫部位佔帳戶比例（超過 10% 發警告）
   - 單日最大虧損估計（超過帳戶 3% 發警告）
   - 本月已實現損益
   - 連續虧損次數（超過 3 次建議暫停）
   - 最終建議：PASS / WARN / BLOCK

5. 若風控結果為 BLOCK：
   - 停止後續流程
   - 解釋原因
   - 不生成下單指令

## 風控規則（來自 strategy/params.py，不得修改）
- 單筆風險：帳戶 2%
- 單日最大虧損：帳戶 6%
- 最大持倉數：5 個
- 最大槓桿：3x

## 允許的工具
allowed-tools:
  Bash(python infrastructure/broker/paper.py --status),
  Bash(python infrastructure/broker/shioaji.py --status),
  Bash(python infrastructure/risk/checker.py:*),
  Read(plans/*),
  Read(strategy/params.py),
  Write(reports/risk_*)
```

### 風控計算核心

```python
# infrastructure/risk/checker.py
import json, sys, argparse
from pathlib import Path
from dataclasses import dataclass
from typing import Optional

@dataclass
class RiskReport:
    position_pct: float      # 新部位佔帳戶比例
    daily_loss_pct: float    # 今日已虧損比例
    consecutive_losses: int  # 連續虧損次數
    max_position_count: int  # 目前持倉數
    
    result: str              # "PASS" / "WARN" / "BLOCK"
    warnings: list[str]
    block_reasons: list[str]


def check(plan_path: Path, account_value: float, 
          daily_pnl: float, open_positions: int,
          consecutive_losses: int,
          # 風控參數（來自 params.py）
          max_position_pct: float = 0.10,
          max_daily_loss_pct: float = 0.06,
          max_positions: int = 5,
          max_consecutive_losses: int = 3) -> RiskReport:
    
    plan = json.loads(plan_path.read_text())
    planned_amount = plan["position_size_twd"]  # 計畫投入金額（台幣）
    
    position_pct = planned_amount / account_value
    daily_loss_pct = abs(min(0, daily_pnl)) / account_value
    
    warnings = []
    block_reasons = []
    
    # 檢查各風控條件
    if position_pct > max_position_pct:
        block_reasons.append(
            f"部位比例 {position_pct:.1%} 超過上限 {max_position_pct:.1%}"
        )
    
    if daily_loss_pct > max_daily_loss_pct:
        block_reasons.append(
            f"今日虧損 {daily_loss_pct:.1%} 已超過日上限 {max_daily_loss_pct:.1%}"
        )
    
    if open_positions >= max_positions:
        block_reasons.append(
            f"已有 {open_positions} 個持倉，達到上限 {max_positions}"
        )
    
    if consecutive_losses >= max_consecutive_losses:
        warnings.append(
            f"⚠️ 連續虧損 {consecutive_losses} 次，建議人工審視策略是否失效"
        )
    
    if position_pct > max_position_pct * 0.8:  # 超過 80% 就警告
        warnings.append(f"部位比例 {position_pct:.1%} 接近上限")
    
    result = "BLOCK" if block_reasons else ("WARN" if warnings else "PASS")
    
    return RiskReport(
        position_pct=position_pct,
        daily_loss_pct=daily_loss_pct,
        consecutive_losses=consecutive_losses,
        max_position_count=open_positions,
        result=result,
        warnings=warnings,
        block_reasons=block_reasons,
    )


if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--plan", required=True)
    args = parser.parse_args()
    
    # 讀取帳戶狀態（實際使用時從 broker API 取得）
    account = json.loads(Path("data/account_status.json").read_text())
    
    report = check(
        plan_path=Path(args.plan),
        account_value=account["total_value"],
        daily_pnl=account["today_pnl"],
        open_positions=account["open_positions_count"],
        consecutive_losses=account["consecutive_losses"],
    )
    
    # 輸出報告
    print(f"\n{'='*50}")
    print(f"🛡️  風控報告")
    print(f"{'='*50}")
    print(f"計畫部位：{report.position_pct:.1%} of 帳戶")
    print(f"今日損益：{report.daily_loss_pct:.1%}")
    print(f"連續虧損：{report.consecutive_losses} 次")
    print(f"目前持倉：{report.max_position_count} 個")
    
    if report.warnings:
        print(f"\n⚠️  警告：")
        for w in report.warnings:
            print(f"   {w}")
    
    if report.block_reasons:
        print(f"\n🚫 阻止原因：")
        for r in report.block_reasons:
            print(f"   {r}")
    
    print(f"\n結果：{'✅ PASS' if report.result == 'PASS' else '⚠️ WARN' if report.result == 'WARN' else '🚫 BLOCK'}")
    
    if report.result == "BLOCK":
        sys.exit(1)  # 非零 exit code，讓 Claude 知道被阻止
```

---

## Skill 3：trade-plan（生成交易計畫）

**檔案位置**：`.claude/skills/trade-plan.md`

```markdown
# trade-plan

## 觸發條件
當用戶說「幫我做計畫」、「生成交易計畫」、「plan <標的>」時

## 執行步驟

1. 讀取最新訊號：`signals/<最新日期>.json`
2. 讀取用戶指定的標的（或讓用戶從訊號清單中選）
3. 計算完整交易計畫：
   - 進場價格範圍（市價 or 限價）
   - 停損位（根據 strategy/core/stop_loss.py 計算）
   - 停利位（根據 strategy/core/take_profit.py 計算）
   - 部位大小（根據 strategy/core/sizing.py + 帳戶資金計算）
   - 預估最大虧損金額
   - 風險報酬比
4. 寫入 plans/<YYYYMMDD_HHMMSS>_<標的>.json
5. **自動觸發 risk-check**
6. 顯示完整計畫 + 風控結果給用戶
7. 等待用戶輸入「確認計畫」才標記為 approved

## 計畫格式
```json
{
  "created_at": "2026-04-30T09:15:00+08:00",
  "symbol": "2330",
  "name": "台積電",
  "direction": "LONG",
  "entry_type": "LIMIT",
  "entry_price": 1055.0,
  "entry_range": [1050.0, 1060.0],
  "stop_loss": 1020.0,
  "take_profit_1": 1090.0,
  "take_profit_2": 1130.0,
  "shares": 1000,
  "position_size_twd": 1055000,
  "max_loss_twd": 35000,
  "risk_reward_ratio": 1.0,
  "signal_basis": "...",
  "approved": false,
  "approved_at": null
}
```

## 關鍵限制
- 部位大小**只由** strategy/core/sizing.py 計算，不自行計算
- 停損位**只由** strategy/core/stop_loss.py 計算，不自行設定
- 風險報酬比 < 1.5 時主動提醒用戶，但不阻止

## 允許的工具
allowed-tools:
  Read(signals/*),
  Bash(python strategy/core/stop_loss.py --symbol:* --entry:*),
  Bash(python strategy/core/take_profit.py:*),
  Bash(python strategy/core/sizing.py:*),
  Write(plans/*),
  Read(strategy/params.py)
```

---

## Skill 4：post-trade（交易後覆盤）

**檔案位置**：`.claude/skills/post-trade.md`

```markdown
# post-trade

## 觸發條件
當用戶說「記錄交易」、「覆盤」、「post trade」時

## 用途
記錄已執行的交易，分析表現，更新績效統計

## 執行步驟

1. 讓用戶輸入交易結果：
   - 買入/賣出價格
   - 實際成交數量
   - 手續費
   - 結果（獲利/虧損/持倉中）

2. 計算指標：
   - 損益金額（點數）
   - 是否符合計畫（進出場是否在計畫範圍內）
   - 是否有提前停損或過早獲利了結

3. 觸發 Claude 做簡短覆盤分析：
   - 這筆交易哪裡做對了？
   - 哪裡可以改進？
   - 和歷史相似案例比較

4. 記錄到 trades/<YYYYMMDD>_<標的>.json
5. 更新 reports/performance.json（累計統計）

## Claude 覆盤框架
分析時考慮：
- 訊號品質：進場訊號在事後看是否清晰？
- 執行紀律：有沒有照計畫執行？
- 情緒因素：是否有因恐懼/貪婪提前動作？
- 改進方向：下次類似情境可以怎麼做？

## 允許的工具
allowed-tools: Read(plans/*), Write(trades/*), Write(reports/*), Read(trades/*)
```

---

## 完整工作流程示範

### 典型的一天

```
# 8:45 AM — 開盤前掃描
你：掃市場

Claude：（執行 market-scan）
📊 市場掃描完成 — 2026-04-30 08:45

大盤概況：
加權指數 21,543（高於 20MA 21,200）
整體趨勢：上升 ✅

符合進場條件（2 個）：
┌─────────────────────────────────────────────┐
│ 1. 台積電 2330   LONG  強度 ★★★★☆          │
│    突破 20MA，RSI=58，成交量放大 1.8x        │
│    建議進場：1,050–1,060                    │
│    停損參考：1,020（ATR 停損）              │
└─────────────────────────────────────────────┘
│ 2. 聯發科 2454   LONG  強度 ★★★☆☆          │
│    RSI 從超賣反彈，成交量平均               │
└─────────────────────────────────────────────┘

目前持倉：0 個
風險提示：無

---

你：幫台積電做計畫

Claude：（執行 trade-plan + risk-check）
📋 交易計畫 — 台積電 2330

方向：做多（LONG）
進場：限價 1,050–1,060（建議等回測到 1,052）
停損：1,018（ATR × 2.0，最大虧損 $34,000）
停利一：1,092（+3.8%，R:R = 1.7）
停利二：1,135（+8.1%，剩餘半倉）
部位：1,000 股（$1,052,000，佔帳戶 7.8%）
預估最大虧損：$34,000（帳戶 2.5%）

🛡️ 風控結果：✅ PASS
   部位比例：7.8%（上限 10%）
   今日虧損：0%（上限 6%）
   連續虧損：0 次

風險報酬比：1.7（良好 ✅）

輸入「確認計畫」以批准此計畫。

---

你：確認計畫

Claude：✅ 計畫已批准，記錄於 plans/20260430_091523_2330.json

現在可以手動在券商 App 下單，或輸入「自動下單」使用模擬帳戶執行。
⚠️ 實盤下單請在券商 App 手動操作，本系統不執行實際下單。

---

# 下午收盤後
你：記錄今天台積電的交易結果，最後賣在 1,091，賺了 39,000

Claude：（執行 post-trade）
📝 交易記錄

買入：1,052（計畫：1,050–1,060 ✅）
賣出：1,091（計畫停利一：1,092 ✅ 接近目標）
損益：+$39,000（+3.7%）
持倉時間：0.5 天（當沖）

覆盤分析：
✅ 進場點精準，在計畫範圍內
✅ 停利執行到位，沒有貪心繼續撐
📌 改進：成交量放大訊號出現後進場稍慢，
   下次可以在確認第一根爆量 K 收盤後即下單

月累計損益：+$39,000 / 1 筆交易
勝率：100%（樣本太小，不具統計意義）
```

---

## 模擬 vs 實盤切換

```python
# infrastructure/broker/base.py
from abc import ABC, abstractmethod

class BrokerBase(ABC):
    @abstractmethod
    def get_account_status(self) -> dict: ...
    
    @abstractmethod
    def get_positions(self) -> list[dict]: ...
    
    # 下單函數刻意設計成需要多個確認步驟
    @abstractmethod
    def place_order(self, symbol: str, direction: str, 
                    quantity: int, price: float,
                    require_human_confirm: bool = True) -> dict: ...
```

```python
# infrastructure/broker/paper.py（模擬帳戶，預設使用）
class PaperBroker(BrokerBase):
    def __init__(self, initial_capital: float = 2_000_000.0):
        self.capital = initial_capital
        self.positions = {}
        self.trades = []
    
    def place_order(self, symbol, direction, quantity, price,
                    require_human_confirm=True):
        if require_human_confirm:
            raise RuntimeError(
                "模擬帳戶：下單需要先在 Claude 中確認交易計畫。\n"
                "這個函數不應該被 Claude 直接呼叫。"
            )
        # 模擬成交
        cost = quantity * price
        if direction == "BUY":
            self.capital -= cost
            self.positions[symbol] = {"quantity": quantity, "avg_price": price}
        print(f"📋 [模擬] {direction} {symbol} × {quantity} @ {price}")
        return {"status": "filled", "price": price}
```

---

## Hooks 整合：開盤自動提醒

```json
// .claude/settings.json（搭配 cron 或排程器使用）
{
  "hooks": {
    "Stop": [
      {
        "matcher": ".*",
        "hooks": [{
          "type": "command",
          "command": "python scripts/notify_summary.py"
        }]
      }
    ]
  }
}
```

```python
# scripts/notify_summary.py
# 透過 LINE Notify 或終端輸出摘要
import os, json
from pathlib import Path

latest_signal = sorted(Path("signals").glob("*.json"))
if latest_signal:
    data = json.loads(latest_signal[-1].read_text())
    count = len(data.get("signals", []))
    msg = f"📊 今日掃描完成，{count} 個符合條件的訊號"
    
    line_token = os.environ.get("LINE_NOTIFY_TOKEN")
    if line_token:
        import requests
        requests.post("https://notify-api.line.me/api/notify",
            headers={"Authorization": f"Bearer {line_token}"},
            data={"message": f"\n{msg}"})
    else:
        print(msg)
```

---

## 關鍵風險提醒

| 風險 | 說明 | 對策 |
|------|------|------|
| Claude 幻覺 | Claude 可能計算出錯誤的技術指標值 | 關鍵數字都用 Python 腳本計算，Claude 只負責解讀 |
| API 斷線 | 券商 API 可能在關鍵時刻斷線 | 永遠設好停損，不依賴軟體停損 |
| 過度自動化 | 太相信系統導致不看盤 | 只自動化「輔助分析」，不自動化「決策執行」 |
| 策略過擬合 | 回測好看，實盤失效 | 樣本外測試（Walk-Forward）才算有效驗證 |
| 連線中斷 | 網路或 Claude API 問題 | 系統異常時預設暫停所有操作，不要預設繼續 |

---

## 相關頁面

- [[Vibe Coding程式交易實戰]] — AI 輔助交易的 CLAUDE.md 護欄設計
- [[Claude股市期貨分析實戰]] — 用 Claude 分析市場的 Prompt 模板
- [[Claude Code Hooks 深度設定]] — 自動觸發的 Hook 設定
- [[Claude MCP 伺服器整合指南]] — MCP 連接券商 API
- [[Superpowers技能框架]] — Skills 的底層框架
- [[Skills實戰：Threads自動爬文與發文]] — 另一個 Skills 實戰範例
