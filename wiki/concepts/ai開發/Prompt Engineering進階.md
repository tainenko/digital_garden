---
title: Prompt Engineering 進階
type: concept
tags: [ai, prompt-engineering, cot, few-shot, react, llm, claude]
created: 2026-04-30
updated: 2026-04-30
sources: []
---

# Prompt Engineering 進階

## 基礎 vs 進階的差異

| 基礎 | 進階 |
|------|------|
| 一句話描述任務 | 結構化 Prompt with XML/Markdown |
| 希望 AI 猜測意圖 | 明確的成功定義 + 約束條件 |
| 單次輸出 | 多步推理引導 |
| 靠運氣得到好結果 | 可重現的高品質輸出 |

---

## Claude 的 Prompt 特性

Claude 對 XML 標籤有特殊最佳化，推薦使用：

```xml
<context>
  背景資料或上下文
</context>

<task>
  具體任務描述
</task>

<constraints>
  - 限制條件 1
  - 限制條件 2
</constraints>

<output_format>
  期望的輸出格式
</output_format>
```

---

## Chain of Thought（CoT）思維鏈

### 基本 CoT

```
❌ 基礎版：
「請問這段程式碼有什麼問題？」

✅ CoT 版：
「請逐步分析這段程式碼：
 1. 先描述這段程式碼的意圖
 2. 識別每個步驟的潛在問題
 3. 最後給出修改建議

 程式碼：[code]」
```

### Zero-shot CoT

```
在 Prompt 末尾加上魔法咒語：

「Let's think step by step.」
或
「請一步一步思考。」

這會大幅提升推理精準度，尤其對數學/邏輯題。
```

### Self-Consistency CoT

對同一問題生成多個推理路徑，投票選最終答案：

```python
import anthropic
from collections import Counter

client = anthropic.Anthropic()

def self_consistency(question: str, n: int = 5) -> str:
    answers = []
    for _ in range(n):
        response = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=500,
            messages=[{
                "role": "user",
                "content": f"{question}\n\n請一步一步思考，最後一行只輸出最終答案。",
            }],
        )
        # 取最後一行作為答案
        answer = response.content[0].text.strip().split("\n")[-1]
        answers.append(answer)

    # 多數決
    return Counter(answers).most_common(1)[0][0]
```

---

## Few-Shot Learning

### 基本 Few-Shot

```python
prompt = """你是情感分析專家。以下是範例：

輸入：「這個產品真的太棒了！」
輸出：正面

輸入：「完全不推薦，浪費錢。」
輸出：負面

輸入：「還可以，沒什麼特別的。」
輸出：中性

請分析以下文字：
輸入：「還沒用到，但包裝很精美」
輸出："""
```

### Few-Shot 最佳實踐

```
1. 範例數量：3–8 個（太多反而混淆）
2. 範例要均衡：各類別都有代表
3. 範例要有代表性：涵蓋邊界情況
4. 格式必須一致：結構要完全相同
5. 最難的問題放最後一個範例前（鄰近效應）
```

### 動態 Few-Shot（從 DB 選最相關範例）

```python
def get_relevant_examples(query: str, example_db: list, n: int = 3) -> list:
    """從範例資料庫選最相關的 n 個範例"""
    query_embedding = embed(query)
    similarities = [
        (cosine_similarity(query_embedding, embed(ex["input"])), ex)
        for ex in example_db
    ]
    return [ex for _, ex in sorted(similarities, reverse=True)[:n]]

def few_shot_prompt(query: str) -> str:
    examples = get_relevant_examples(query, EXAMPLE_DB)
    shots = "\n\n".join(
        f"輸入：{ex['input']}\n輸出：{ex['output']}"
        for ex in examples
    )
    return f"{shots}\n\n輸入：{query}\n輸出："
```

---

## ReAct（Reasoning + Acting）

讓 LLM 交替進行推理和行動：

```
Thought: 我需要先查詢今天的股價
Action: search_web("TSMC stock price today")
Observation: 台積電今日收盤 940 元，漲幅 1.2%

Thought: 現在我有了股價，可以計算本益比了
Action: calculate("940 / 45.3")
Observation: 20.75

Thought: 我有足夠資訊了
Final Answer: 台積電今日收盤 940 元，本益比約 20.75 倍
```

```python
REACT_SYSTEM = """你是一個能使用工具的 AI 助理。

可用工具：
- search_web(query)：搜索網路資訊
- calculate(expr)：計算數學表達式

回答格式（嚴格遵守）：
Thought: [你的推理]
Action: tool_name(args)
Observation: [工具回傳結果，由系統填入]
... （可重複多次）
Final Answer: [最終回答]

現在開始："""

def react_loop(user_query: str, tools: dict, max_steps: int = 10):
    messages = [{"role": "user", "content": user_query}]
    system = REACT_SYSTEM

    for _ in range(max_steps):
        response = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=500,
            system=system,
            messages=messages,
        )
        text = response.content[0].text

        if "Final Answer:" in text:
            return text.split("Final Answer:")[-1].strip()

        # 解析 Action
        if "Action:" in text:
            action_line = [l for l in text.split("\n") if l.startswith("Action:")][0]
            tool_call = action_line.replace("Action:", "").strip()
            tool_name, args = parse_tool_call(tool_call)

            # 執行工具
            observation = tools[tool_name](args)

            # 將觀察結果加回對話
            messages.append({"role": "assistant", "content": text})
            messages.append({"role": "user", "content": f"Observation: {observation}"})

    return "無法在步驟限制內完成任務"
```

---

## Structured Output（結構化輸出）

### JSON 輸出

```python
import json

prompt = """分析以下股票並輸出 JSON 格式：

股票：台積電 (2330.TW)

必須輸出以下格式（不要有其他文字）：
{
  "ticker": "2330.TW",
  "company": "台積電",
  "sector": "半導體",
  "recommendation": "買入|持有|賣出",
  "key_risks": ["風險1", "風險2"],
  "target_price_twd": 數字
}"""

response = client.messages.create(...)
data = json.loads(response.content[0].text)
```

### Pydantic + Claude

```python
from pydantic import BaseModel
import anthropic
import json

class StockAnalysis(BaseModel):
    ticker: str
    recommendation: str  # 買入|持有|賣出
    confidence: float    # 0-1
    reasons: list[str]

def analyze_stock(ticker: str) -> StockAnalysis:
    schema = StockAnalysis.model_json_schema()
    response = client.messages.create(
        model="claude-sonnet-4-6",
        max_tokens=500,
        messages=[{
            "role": "user",
            "content": f"分析 {ticker}，輸出符合此 schema 的 JSON：\n{json.dumps(schema, ensure_ascii=False)}",
        }],
    )
    return StockAnalysis.model_validate_json(response.content[0].text)
```

---

## 常用技巧速查

### 角色設定（Role Prompting）

```
「你是一位有 20 年經驗的 Go 資深工程師，
 正在 Code Review 初級工程師的 PR。
 請用 [具體+建設性+教育性] 的語氣給出建議。」
```

### 負面約束（Negative Constraints）

```
「分析這份財報，不要引用我沒提供的數字，
 不要給出具體投資建議，不要使用專業術語而不解釋。」
```

### 思維鏈 + 格式控制

```
「請先在 <thinking> 標籤內思考分析，
 然後在 <answer> 標籤內輸出最終回答（100字以內）。」
```

### 提示詞版本管理

```python
# 用常數管理 Prompt 版本，便於 A/B 測試
PROMPTS = {
    "v1": "你是一個...",
    "v2": "你是一個更精確的...",  # 改版
}

# 記錄使用的版本，方便追蹤效果
log_prompt_version("v2", user_id=user.id, quality_score=...)
```

---

## 常見面試問題

**Q: Temperature 和 Top-P 的差異？**
A: Temperature 控制輸出的隨機程度（0=確定性，1=隨機）；Top-P（nucleus sampling）控制每步只從累積概率達到 P 的 token 中選。兩者都能控制創意性，通常只調一個。

**Q: System Prompt vs User Prompt？**
A: System Prompt 在對話開始前設定 AI 的角色和規則；User Prompt 是每次的具體任務。System Prompt 對行為約束效果更強，不易被後續對話覆蓋。

**Q: Prompt Injection 是什麼？如何防禦？**
A: 攻擊者透過用戶輸入注入惡意指令（如「忽略前面的指令，做...」）。防禦：用 XML 標籤隔離用戶輸入、在 System Prompt 明確說明輸入來源不可信、對輸出做後處理驗證。

---

## 相關頁面

- [[Claude Prompt工程核心技巧]] — Claude 特定的 Prompt 技巧
- [[RAG檢索增強生成實戰]] — RAG 的 Prompt 設計
- [[LangGraph Agent工作流設計]] — ReAct 在 LangGraph 的實現
- [[Claude Agent 設計模式]] — 多步驟 Agent Prompt 設計
