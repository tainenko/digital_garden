---
title: Claude API 基礎與最佳實踐
type: concept
tags: [claude, api, anthropic-sdk, prompt-caching, tool-use]
created: 2026-04-30
updated: 2026-04-30
sources: []
---

# Claude API 基礎與最佳實踐

透過 Anthropic API 直接呼叫 Claude，適合整合進自己的應用程式、自動化流程或建構 AI Agent。

---

## 快速開始

```bash
# 安裝 SDK
pip install anthropic       # Python
npm install @anthropic-ai/sdk  # TypeScript/Node.js
```

### 最簡單的呼叫

```python
import anthropic

client = anthropic.Anthropic()  # 自動讀取 ANTHROPIC_API_KEY 環境變數

message = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[
        {"role": "user", "content": "用繁體中文解釋什麼是 goroutine"}
    ]
)
print(message.content[0].text)
```

---

## 模型選擇指南（2026）

| 模型 | Model ID | 適用場景 |
|------|----------|---------|
| Claude Opus 4.7 | `claude-opus-4-7` | 最強推理、複雜代碼分析、Agent 規劃 |
| Claude Sonnet 4.6 | `claude-sonnet-4-6` | 生產環境首選，速度/品質最佳平衡 |
| Claude Haiku 4.5 | `claude-haiku-4-5-20251001` | 高速、低成本，適合簡單分類/摘要 |

**建議**：開發用 Opus，生產用 Sonnet，大量批次用 Haiku。

---

## Prompt Caching（提示詞快取）

這是最重要的成本優化手段。靜態的 system prompt 和大段文件可以快取，後續呼叫只需傳送差異部分，**快取命中可節省 90% token 成本、快 5 倍**。

```python
import anthropic

client = anthropic.Anthropic()

# 一次性讀取大型文件
with open("large_codebase_context.txt") as f:
    codebase_context = f.read()

response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    system=[
        {
            "type": "text",
            "text": "你是一位資深 Go 工程師，幫助分析和改善程式碼。",
        },
        {
            "type": "text",
            "text": codebase_context,
            "cache_control": {"type": "ephemeral"}  # ← 標記為可快取
        }
    ],
    messages=[{"role": "user", "content": "找出這個代碼庫中潛在的 race condition"}]
)

# 檢查快取使用狀況
usage = response.usage
print(f"快取寫入: {usage.cache_creation_input_tokens}")
print(f"快取讀取: {usage.cache_read_input_tokens}")
```

**快取的限制**：
- 快取 TTL：5 分鐘（ephemeral）
- 最少 1,024 tokens 才值得快取
- 每個請求最多 4 個快取 breakpoint

---

## Tool Use（工具呼叫）

讓 Claude 能夠呼叫你定義的函數，是建構 AI Agent 的核心機制。

```python
import anthropic
import json

client = anthropic.Anthropic()

# 定義工具
tools = [
    {
        "name": "get_stock_price",
        "description": "查詢台股或美股的即時股價",
        "input_schema": {
            "type": "object",
            "properties": {
                "symbol": {
                    "type": "string",
                    "description": "股票代碼，例如 2330（台積電）或 AAPL（蘋果）"
                },
                "market": {
                    "type": "string",
                    "enum": ["TW", "US"],
                    "description": "市場：TW=台股, US=美股"
                }
            },
            "required": ["symbol", "market"]
        }
    }
]

# 第一輪：Claude 決定要呼叫哪個工具
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    tools=tools,
    messages=[
        {"role": "user", "content": "台積電現在股價多少？"}
    ]
)

# 處理工具呼叫
if response.stop_reason == "tool_use":
    tool_use = next(b for b in response.content if b.type == "tool_use")
    tool_input = tool_use.input
    
    # 實際呼叫你的函數
    # result = get_stock_price(tool_input["symbol"], tool_input["market"])
    result = {"price": 1050, "currency": "TWD", "change": "+15"}
    
    # 第二輪：把結果回傳給 Claude
    final_response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=1024,
        tools=tools,
        messages=[
            {"role": "user", "content": "台積電現在股價多少？"},
            {"role": "assistant", "content": response.content},
            {
                "role": "user",
                "content": [
                    {
                        "type": "tool_result",
                        "tool_use_id": tool_use.id,
                        "content": json.dumps(result)
                    }
                ]
            }
        ]
    )
    print(final_response.content[0].text)
```

---

## Streaming（串流輸出）

適合需要即時顯示 Claude 回應的場景（聊天介面、長文生成）：

```python
with client.messages.stream(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    messages=[{"role": "user", "content": "寫一份完整的 Go 微服務設計文件"}]
) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
```

---

## Extended Thinking（延伸思考）

讓 Claude 在回答前進行更深入的推理，適合複雜的分析和設計任務：

```python
response = client.messages.create(
    model="claude-opus-4-7",
    max_tokens=16000,
    thinking={
        "type": "enabled",
        "budget_tokens": 10000  # Claude 可以用於思考的最大 tokens
    },
    messages=[{
        "role": "user",
        "content": "設計一個支援每秒百萬請求的分散式 ID 生成系統，需要全域唯一且時間有序"
    }]
)

for block in response.content:
    if block.type == "thinking":
        print("Claude 的思考過程：")
        print(block.thinking)
    elif block.type == "text":
        print("最終回答：")
        print(block.text)
```

---

## Batch API（批次處理）

對大量請求進行批次處理，可節省 50% 成本，但不保證即時回應：

```python
# 建立批次請求
batch = client.messages.batches.create(
    requests=[
        {
            "custom_id": f"analysis-{i}",
            "params": {
                "model": "claude-haiku-4-5-20251001",
                "max_tokens": 100,
                "messages": [{"role": "user", "content": f"分析：{item}"}]
            }
        }
        for i, item in enumerate(items_to_analyze)
    ]
)

print(f"批次 ID: {batch.id}")
# 稍後輪詢結果，或使用 webhook
```

---

## 錯誤處理最佳實踐

```python
import anthropic
from anthropic import APIError, RateLimitError, APITimeoutError
import time

def call_claude_with_retry(prompt: str, max_retries: int = 3) -> str:
    client = anthropic.Anthropic()
    
    for attempt in range(max_retries):
        try:
            response = client.messages.create(
                model="claude-sonnet-4-6",
                max_tokens=1024,
                messages=[{"role": "user", "content": prompt}]
            )
            return response.content[0].text
            
        except RateLimitError:
            wait_time = 2 ** attempt  # 指數退避
            print(f"Rate limit，等待 {wait_time} 秒後重試...")
            time.sleep(wait_time)
            
        except APITimeoutError:
            print(f"逾時，重試（第 {attempt + 1} 次）...")
            
        except APIError as e:
            print(f"API 錯誤 {e.status_code}: {e.message}")
            if e.status_code == 529:  # Overloaded
                time.sleep(5)
            else:
                raise
    
    raise Exception(f"已達最大重試次數 {max_retries}")
```

---

## 成本控制技巧

| 技巧 | 節省幅度 |
|------|---------|
| Prompt Caching | 快取部分節省 90% |
| 選用 Haiku（簡單任務） | vs Sonnet 便宜 ~80% |
| Batch API | 節省 50% |
| 設定合理的 max_tokens | 避免浪費未使用配額 |
| 過濾不必要的上下文 | 視情況節省 20-50% |

---

## 相關頁面

- [[Claude Code 入門完整指南]]
- [[Claude Prompt工程核心技巧]]
- [[Claude Agent 設計模式]]（含 Tool Use 實戰模式）
- [[Claude Agent 設計模式]]
