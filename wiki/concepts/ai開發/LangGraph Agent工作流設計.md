---
title: LangGraph Agent 工作流設計
type: concept
tags: [ai, langgraph, agent, workflow, llm, graph]
created: 2026-04-30
updated: 2026-04-30
sources: []
---

# LangGraph Agent 工作流設計

## 為何需要 LangGraph

基本 LLM 呼叫是單步（問 → 答），現實任務需要：
- **多步推理**：一個複雜任務拆成多個子步驟
- **工具使用**：LLM 決定呼叫搜索/計算/資料庫
- **條件分支**：依據中間結果走不同路徑
- **循環**：不滿意就重試，直到滿足條件
- **人工介入**（Human-in-the-Loop）：關鍵決策前暫停等待

LangGraph 用**有向圖（DAG + 循環）**建模這些需求。

---

## 核心概念

```
State（狀態）：圖中傳遞的資料結構
Node（節點）：執行函數（LLM 呼叫、工具、判斷）
Edge（邊）：節點間的連接
Conditional Edge：根據狀態決定下一個節點
```

---

## 最小範例

```python
from typing import TypedDict, Annotated
from langgraph.graph import StateGraph, START, END
from langgraph.graph.message import add_messages
from langchain_anthropic import ChatAnthropic

# 1. 定義 State
class State(TypedDict):
    messages: Annotated[list, add_messages]  # add_messages 會自動追加而非覆蓋

# 2. 定義 Node
llm = ChatAnthropic(model="claude-sonnet-4-6")

def chatbot(state: State) -> State:
    response = llm.invoke(state["messages"])
    return {"messages": [response]}

# 3. 建立 Graph
builder = StateGraph(State)
builder.add_node("chatbot", chatbot)
builder.add_edge(START, "chatbot")
builder.add_edge("chatbot", END)
graph = builder.compile()

# 4. 執行
result = graph.invoke({"messages": [("user", "你好！")]})
print(result["messages"][-1].content)
```

---

## ReAct Agent（最常用模式）

Reasoning + Acting：LLM 決定要用哪個工具，看結果後繼續思考。

```python
from langchain_core.tools import tool
from langgraph.prebuilt import create_react_agent

@tool
def search_web(query: str) -> str:
    """搜索網路獲取最新資訊"""
    # 實際接 Tavily / SerpAPI 等
    return f"搜索結果：{query} 的相關資訊..."

@tool
def calculate(expression: str) -> str:
    """計算數學表達式"""
    try:
        return str(eval(expression))  # 生產環境用 numexpr 或沙盒
    except Exception as e:
        return f"計算錯誤：{e}"

llm = ChatAnthropic(model="claude-sonnet-4-6")
tools = [search_web, calculate]

agent = create_react_agent(llm, tools)

result = agent.invoke({
    "messages": [("user", "台積電今天股價是多少？再幫我算 NVDA 的 P/E")]
})
```

ReAct 內部循環：
```
用戶問題 → LLM 思考 → 決定用工具 → 執行工具 → LLM 看結果 → 繼續思考... → 最終回答
```

---

## 自定義多節點 Graph

```python
from langgraph.graph import StateGraph, START, END
import operator
from typing import TypedDict, Annotated

class ResearchState(TypedDict):
    question: str
    search_results: list[str]
    draft: str
    final_answer: str
    revision_count: int

def search_node(state: ResearchState) -> ResearchState:
    """搜索相關資訊"""
    results = web_search(state["question"])
    return {"search_results": results}

def draft_node(state: ResearchState) -> ResearchState:
    """根據搜索結果起草回答"""
    draft = llm.invoke([
        ("system", "根據以下資料起草回答"),
        ("user", f"資料：{state['search_results']}\n問題：{state['question']}"),
    ]).content
    return {"draft": draft, "revision_count": state.get("revision_count", 0) + 1}

def review_node(state: ResearchState) -> ResearchState:
    """審查草稿品質"""
    review = llm.invoke([
        ("user", f"評估這個回答品質（輸出 GOOD 或 REVISE）：\n{state['draft']}"),
    ]).content
    return {"review_result": review}

def should_revise(state: ResearchState) -> str:
    """條件邊：決定繼續修改還是結束"""
    if "REVISE" in state.get("review_result", "") and state["revision_count"] < 3:
        return "draft"  # 回到起草節點
    return "finalize"

def finalize_node(state: ResearchState) -> ResearchState:
    return {"final_answer": state["draft"]}

# 建構 Graph
builder = StateGraph(ResearchState)
builder.add_node("search", search_node)
builder.add_node("draft", draft_node)
builder.add_node("review", review_node)
builder.add_node("finalize", finalize_node)

builder.add_edge(START, "search")
builder.add_edge("search", "draft")
builder.add_edge("draft", "review")
builder.add_conditional_edges("review", should_revise, {
    "draft": "draft",      # 修改 → 回到 draft
    "finalize": "finalize" # 通過 → 結束
})
builder.add_edge("finalize", END)

graph = builder.compile()
```

---

## Human-in-the-Loop（人工審核）

```python
from langgraph.checkpoint.memory import MemorySaver

# 加入 checkpointer 才能暫停和恢復
memory = MemorySaver()

builder = StateGraph(State)
# ... 節點定義 ...

# interrupt_before：到達這個節點前暫停，等人工確認
graph = builder.compile(
    checkpointer=memory,
    interrupt_before=["execute_trade"],  # 執行交易前暫停
)

# 第一次執行（會在 execute_trade 前暫停）
config = {"configurable": {"thread_id": "trade-001"}}
result = graph.invoke({"plan": "買 100 股 TSMC"}, config=config)
print("等待人工確認：", result["plan"])

# 人工確認後，繼續執行
graph.invoke(None, config=config)  # None 表示繼續，不修改 state

# 或人工修改後繼續
graph.update_state(config, {"plan": "買 50 股 TSMC（修改數量）"})
graph.invoke(None, config=config)
```

---

## 持久化（Checkpointing）

```python
# 記憶體（測試用）
from langgraph.checkpoint.memory import MemorySaver
checkpointer = MemorySaver()

# PostgreSQL（生產用）
from langgraph.checkpoint.postgres import PostgresSaver
checkpointer = PostgresSaver.from_conn_string("postgresql://user:pass@localhost/db")

graph = builder.compile(checkpointer=checkpointer)

# thread_id 用於隔離不同用戶/對話
config = {"configurable": {"thread_id": f"user-{user_id}-session-{session_id}"}}
graph.invoke({"messages": [...]}, config=config)

# 查看歷史
for checkpoint in graph.get_state_history(config):
    print(checkpoint.created_at, checkpoint.values)
```

---

## 平行執行（Send API）

```python
from langgraph.types import Send

def fan_out(state: State) -> list[Send]:
    """平行搜索多個主題"""
    topics = state["topics"]
    return [Send("research_topic", {"topic": t}) for t in topics]

def research_topic(state: dict) -> dict:
    result = llm.invoke(f"研究：{state['topic']}")
    return {"results": [result.content]}

builder.add_conditional_edges("planning", fan_out, ["research_topic"])
builder.add_node("research_topic", research_topic)
```

---

## 常見 Agent 設計模式

| 模式 | 描述 | 使用場景 |
|------|------|---------|
| ReAct | 思考 → 行動 → 觀察循環 | 通用工具使用 |
| Plan & Execute | 先規劃再執行 | 複雜多步任務 |
| Reflection | 輸出後自我評估 | 提高輸出品質 |
| Multi-Agent | 多個專業 agent 協作 | 大型複雜任務 |
| Supervisor | 一個 LLM 分配任務給子 agent | 任務路由 |

---

## 最佳實踐

1. **State 設計先行**：State 是 Graph 的合約，設計要考慮所有節點的輸入輸出
2. **節點小而純**：每個節點只做一件事，便於測試和除錯
3. **加入 max_iterations 防無限循環**：條件邊必須有退出條件
4. **用 thread_id 隔離對話**：不同用戶/對話用不同 thread_id
5. **生產環境必須用持久化 Checkpointer**：MemorySaver 重啟後遺失

---

## 相關頁面

- [[RAG檢索增強生成實戰]] — RAG 作為 Agent 的工具
- [[Prompt Engineering進階]] — 如何設計 Agent 的系統 Prompt
- [[Claude Agent 設計模式]] — Claude SDK Agent 模式
- [[Claude Code Subagents 完整指南]] — Claude Code 的 Subagent 機制
