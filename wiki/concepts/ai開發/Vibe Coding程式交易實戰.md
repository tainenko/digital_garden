---
title: Vibe Coding 程式交易實戰指南
type: concept
tags: [vibe-coding, algorithmic-trading, CLAUDE.md, AI-coding, trading-system, risk-management]
created: 2026-04-30
updated: 2026-04-30
sources: [gucci-ai-vibe-coding-trading]
---

# Vibe Coding 程式交易實戰指南

> 核心問題：AI 最大的危險不是「不會寫」，而是它會自作主張，為了「優化、簡化、重構」，把你已經寫好的交易邏輯改掉。交易程式裡，這件事特別致命。

## 為什麼程式交易特別危險

```
一般軟體 bug：UI 跑版、功能失效 → 使用者體驗差
交易程式 bug：停損方向反了、部位大小算錯 → 直接虧錢

AI 的「善意優化」在交易程式裡的典型破壞：

- 「簡化」停損條件：把 close < entry * 0.97 改成 close < entry * stop_pct
  → 看起來更通用，但你原本的 3% 停損有其交易邏輯理由
- 「重構」進場信號函數：把五個條件拆成可配置的 filter
  → 破壞了條件的順序相依性
- 「優化」部位大小計算：把固定張數改成「更合理的」動態 Kelly 公式
  → 直接改變了你的資金管理策略
- 「修正」看起來像 bug 的邏輯：把你刻意為之的特殊條件刪掉
  → 那個條件其實是應對特定市場狀況的保護機制
```

---

## 一、寫一份 AI 規範文件（CLAUDE.md / AGENTS.md）

在專案根目錄放 `CLAUDE.md`（Claude Code 讀）或 `AGENTS.md`（通用），告訴 AI 明確的邊界。

### 交易專案的 CLAUDE.md 範本

```markdown
# 交易系統 AI 規範文件

## ⛔ 絕對不能動的部分

以下邏輯沒有我明確的文字確認，**任何情況下禁止修改**：

### 策略核心（strategy/core/）
- **進場條件**（entry_signal.py / entry.go）
- **出場條件**（exit_signal.py / exit.go）
- **停損邏輯**（stop_loss.py / stop.go）
- **停利邏輯**（take_profit.py / take.go）
- **部位大小計算**（position_sizing.py / sizing.go）
- **風控規則**（risk_manager.py / risk.go）

### 受保護的常數與參數
- 任何標記 `# PROTECTED` 或 `// PROTECTED` 的行
- 所有策略參數（止損百分比、進場條件閾值、最大部位等）

## ✅ 可以自由修改的部分

- 資料讀取（data/）
- 視覺化和圖表（charts/、plot/）
- 回測報表格式（reports/）
- 券商 API 介面層（broker/）
- 工具函式（utils/、helpers/）
- 日誌和監控（logging/、monitoring/）
- 測試檔案（tests/）

## 📋 修改前的必要流程

1. **先說明**：告訴我你打算改哪些檔案、改哪些地方、為什麼
2. **等確認**：等我說「OK」或「繼續」後再動手
3. **禁止自主重構**：沒有明確指示，不能自行「順便優化」或「重構」
4. **一次一件事**：每次只做一個具體的任務

## 🚨 遇到以下情況必須停下來問我

- 發現策略核心有「看起來像 bug」的地方
- 覺得某個邏輯「可以更好」
- 需要修改進出場條件才能完成任務
- 任何你不確定是否屬於「核心」的修改
```

---

## 二、程式碼分區設計

把程式明確拆成兩個區域：

```
trading-system/
│
├── strategy/                  ← 🔒 核心策略層（AI 不輕易碰）
│   ├── core/
│   │   ├── entry.py          # 進場條件
│   │   ├── exit.py           # 出場條件
│   │   ├── stop_loss.py      # 停損邏輯
│   │   ├── take_profit.py    # 停利邏輯
│   │   ├── sizing.py         # 部位大小
│   │   └── risk.py           # 風控規則
│   └── params.py             # 策略參數（PROTECTED）
│
└── infrastructure/            ← ✅ 外圍基礎設施（AI 可以改）
    ├── data/                  # 資料讀取、清洗
    ├── broker/                # 券商 API 串接
    ├── backtest/              # 回測引擎、報表
    ├── charts/                # 圖表、視覺化
    ├── monitoring/            # 監控、警報
    └── utils/                 # 工具函式
```

**為什麼這樣分區有效**：

```python
# strategy/core/stop_loss.py
# PROTECTED: 此文件的所有邏輯未經明確批准不得修改

def calculate_stop_loss(entry_price: float, atr: float, multiplier: float = 2.0) -> float:
    """
    停損 = 進場價 - (ATR * 倍數)
    
    注意：這個公式是刻意設計的，不要「優化」成其他形式。
    ATR 倍數 2.0 是經過 3 年回測確認的參數。
    """
    return entry_price - (atr * multiplier)  # PROTECTED

def should_stop_out(current_price: float, stop_price: float, is_long: bool) -> bool:
    """做多停損判斷，注意 is_long=False 時邏輯相反"""
    if is_long:
        return current_price <= stop_price   # PROTECTED
    else:
        return current_price >= stop_price   # PROTECTED（做空時停損在上方）
```

---

## 三、給 AI 的任務設計

### ❌ 危險的指令模式

```
「幫我優化整個交易系統的效能」
→ AI 可能改掉計算邏輯、合併函數、改變資料結構

「幫我重構 entry.py，讓它更好讀」
→ AI 的「好讀」可能改掉你刻意設計的條件順序

「這段代碼有點亂，幫我整理一下」
→ AI 不知道什麼是「刻意的亂」（保護特定邊界條件）

「幫我把這個系統改成支援多策略」
→ 這種架構大改幾乎必然觸碰核心邏輯
```

### ✅ 安全的指令模式

```
「只修正 data/fetcher.py 第 47 行的錯誤，
 不要改 strategy/ 下的任何檔案。
 先告訴我你要改什麼，等我確認後再改。」

「在 charts/equity_curve.py 裡新增一個畫 drawdown 的函數，
 不修改任何現有函數。」

「幫我在 broker/api.py 裡加入下單失敗的重試邏輯，
 不要修改 strategy/ 下的任何東西，
 先說明你的修改計畫。」

「tests/test_backtest.py 裡的 TestBacktestReport 測試失敗，
 只修正這個測試，不要動 strategy/core/ 的任何檔案。」
```

### 任務拆解原則

```
大任務（危險）：「幫我把回測系統改成事件驅動架構」
          ↓ 拆解
小任務 1：「在 backtest/ 下新增一個 EventQueue 類別」
小任務 2：「把 data_loader.py 改成發送 DataEvent 到 EventQueue」
小任務 3：「在 reports.py 裡讀取 EventQueue 產生報告」
→ 每個任務都不觸碰 strategy/core/
```

---

## 四、Git 保護策略

```bash
# 用 git 追蹤核心策略的每次修改
git log --follow strategy/core/entry.py

# 設定 .gitattributes 標記重要檔案（供 PR review 注意）
# .gitattributes
strategy/core/* linguist-generated=false merge=ours

# 建議工作流程：
# main 分支：穩定版策略
# feature/* 分支：只改外圍功能
# strategy/* 分支：策略修改（需要特別審查）

# PR 規則：
# - strategy/core/ 下的任何變更需要人工 review，不允許自動 merge
# - 用 CODEOWNERS 強制指定 reviewer
```

```
# .github/CODEOWNERS
strategy/core/   @your-username   # 核心策略只有你能 approve
```

---

## 五、交易邏輯的測試保護

**固化現有行為**（Characterization Tests）：在修改前，先把現有邏輯的輸入輸出固化成測試，讓 AI 任何改動都會被測試抓到：

```python
# tests/test_strategy_core.py
# 這些測試的目的不是「驗證邏輯正確」
# 而是「固化現有行為，任何改變都要被發現」

import pytest
from strategy.core.stop_loss import calculate_stop_loss, should_stop_out

class TestStopLossIsUnchanged:
    """
    Characterization Tests：固化現有停損邏輯。
    這些測試的值是從現有代碼取得的，不是從「應該怎樣」推導的。
    如果這些測試失敗，代表有人動了停損邏輯！
    """

    def test_long_stop_calculation(self):
        # entry=100, atr=2.0, multiplier=2.0 → stop=96.0
        assert calculate_stop_loss(100.0, 2.0, 2.0) == 96.0

    def test_long_stop_calculation_default_multiplier(self):
        # 預設倍數 2.0
        assert calculate_stop_loss(150.0, 3.0) == 144.0

    def test_long_position_should_stop(self):
        assert should_stop_out(95.9, 96.0, is_long=True) is True

    def test_long_position_should_not_stop_at_exact_price(self):
        # 恰好等於停損價：不觸發（刻意設計的邊界）
        assert should_stop_out(96.0, 96.0, is_long=True) is False

    def test_short_position_stop_is_above(self):
        # 做空停損在上方
        assert should_stop_out(104.1, 104.0, is_long=False) is True
        assert should_stop_out(103.9, 104.0, is_long=False) is False


class TestEntrySignalIsUnchanged:
    """固化進場條件，確保 AI 沒有偷改條件順序或閾值"""

    @pytest.mark.parametrize("price,ma20,rsi,volume_ratio,expected", [
        (105.0, 100.0, 45.0, 1.5, True),   # 標準進場
        (99.0,  100.0, 45.0, 1.5, False),  # 價格在均線下，不進場
        (105.0, 100.0, 75.0, 1.5, False),  # RSI 超買，不進場
        (105.0, 100.0, 45.0, 0.8, False),  # 成交量不足，不進場
    ])
    def test_entry_conditions(self, price, ma20, rsi, volume_ratio, expected):
        from strategy.core.entry import should_enter_long
        result = should_enter_long(price, ma20, rsi, volume_ratio)
        assert result == expected, f"Entry condition changed! price={price}"
```

```bash
# 把測試加入 pre-commit hook
# .git/hooks/pre-commit
#!/bin/sh
python -m pytest tests/test_strategy_core.py -q
if [ $? -ne 0 ]; then
    echo "❌ 策略核心測試失敗！請確認你的修改是否誤動了核心邏輯。"
    exit 1
fi
```

---

## 六、常見 AI 破壞模式與防範

```python
# 破壞模式 1：把「固定值」改成「可配置參數」
# 原始（刻意的 3% 硬停損）：
if current_price < entry_price * 0.97:  # PROTECTED
    exit_position()

# AI「優化」後（看起來更好，實際改變了邏輯）：
if current_price < entry_price * (1 - self.stop_pct):
    exit_position()
# → 問題：self.stop_pct 預設值可能不是 0.03

# 防範：在停損行加 # PROTECTED 和測試固化

# ────────────────────────────────────────────

# 破壞模式 2：「簡化」多條件判斷
# 原始（條件順序有意義）：
def should_enter(self) -> bool:
    if not self.is_market_hours():    # 1. 先確認交易時段
        return False
    if self.has_open_position():      # 2. 確認沒有持倉
        return False
    if not self.trend_is_up():        # 3. 確認趨勢
        return False
    if self.rsi > 70:                 # 4. 過濾超買
        return False
    return self.volume_breakout()     # 5. 成交量確認

# AI「重構」後（用 all() 簡化，失去了 short-circuit 的語意）：
def should_enter(self) -> bool:
    return all([
        self.is_market_hours(),
        not self.has_open_position(),
        self.trend_is_up(),
        self.rsi <= 70,
        self.volume_breakout()
    ])
# → 問題：all() 仍然 short-circuit，但可讀性上失去了「優先級」的語意表達
# 更大的問題：AI 可能悄悄改掉條件的值（rsi > 70 → rsi <= 70）

# ────────────────────────────────────────────

# 破壞模式 3：「修正」刻意為之的特殊條件
# 原始（看起來像 bug 但不是）：
if position_size == 0 and self.signal == 'BUY':
    position_size = 1  # 特殊情況：信號有但部位計算為 0 時的保底邏輯

# AI 看到 position_size = 1 是硬編碼「修正」成：
position_size = max(1, self.calculate_position())
# → 破壞了這個保底邏輯的原意
```

---

## 七、回測驗證工作流程

每次 AI 修改後（即使只改外圍程式碼），都要確認回測結果沒有變化：

```python
# scripts/verify_strategy_unchanged.py
"""
用這個腳本確認策略邏輯沒有被意外改動。
在任何 AI 修改後執行，比較回測指標是否和基準一致。
"""
import json
from backtest.engine import run_backtest
from backtest.metrics import calculate_metrics

BASELINE_FILE = "tests/baseline_metrics.json"

def verify():
    # 執行回測
    trades = run_backtest(
        symbol="TXFR1",
        start="2023-01-01",
        end="2023-12-31",
    )
    metrics = calculate_metrics(trades)

    # 和基準比較
    with open(BASELINE_FILE) as f:
        baseline = json.load(f)

    mismatches = []
    for key in ["total_trades", "win_rate", "profit_factor", "max_drawdown"]:
        if abs(metrics[key] - baseline[key]) > 1e-6:
            mismatches.append(f"{key}: baseline={baseline[key]}, current={metrics[key]}")

    if mismatches:
        print("❌ 策略結果改變！可能有核心邏輯被修動：")
        for m in mismatches:
            print(f"  - {m}")
        exit(1)
    else:
        print("✅ 策略邏輯未改變，回測結果一致。")

if __name__ == "__main__":
    verify()
```

```bash
# 建立基準（策略確認正確時執行一次）
python scripts/create_baseline.py

# 每次 AI 修改後驗證
python scripts/verify_strategy_unchanged.py
```

---

## 核心心態

```
❌ 錯誤心態：讓 AI 當「交易策略設計師」
✅ 正確心態：讓 AI 當「受控工程師」

策略怎麼交易 → 由你決定
AI 負責什麼 → 在你畫好的範圍內寫程式

這不是不相信 AI，而是認清 AI 的角色邊界：
AI 很擅長寫 utility function、串接 API、生成報表、修復語法錯誤
AI 不適合做策略決策，因為它不知道你的策略為什麼這樣設計
```

---

## 相關頁面

- [[Spec驅動開發]] — 用規格文件約束 AI 的實作範圍
- [[OpenSpec工作流]] — 完整的 AI 開發流程框架
- [[生產環境Vibe Coding四大策略]] — Erik Schluntz 的葉節點策略（先改外圍再改核心）
- [[Vibe Coding風險與限制]] — AI Coding 的普遍風險
