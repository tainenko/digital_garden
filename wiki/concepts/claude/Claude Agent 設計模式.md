---
title: Claude Agent 設計模式
type: concept
tags: [claude, agent, multi-agent, tool-use, 設計模式]
created: 2026-04-30
updated: 2026-04-30
sources: []
---

# Claude Agent 設計模式

Agent 是讓 Claude 自主使用工具、迭代執行任務的架構模式。理解常見的 Agent 設計模式，能幫你建構更可靠的 AI 系統。

---

## 什麼是 Agent

**Agent = Claude + Tools + Loop**

Claude 本身是無狀態的語言模型，當你賦予它：
1. **工具**（Tool Use）：讓它能執行動作（搜尋、寫檔、呼叫 API）
2. **迴圈**（Agentic Loop）：讓它重複執行直到任務完成

就形成了一個 Agent。

---

## 核心設計模式

### 1. 單一 Agent（Simple Agent）

最基礎的模式：一個 Claude + 多個工具，循環直到完成。

```
用戶輸入 → Claude → [決定用哪個工具] → 執行工具 → 結果回 Claude → 重複，直到完成
```

**適用場景**：
- 資料查詢 + 分析
- 程式碼生成 + 測試
- 文件整理

**Python 範例**：

```python
import anthropic
import subprocess

client = anthropic.Anthropic()

tools = [
    {
        "name": "run_command",
        "description": "在終端機執行指令並回傳輸出",
        "input_schema": {
            "type": "object",
            "properties": {
                "command": {"type": "string"}
            },
            "required": ["command"]
        }
    },
    {
        "name": "write_file",
        "description": "寫入文字到檔案",
        "input_schema": {
            "type": "object",
            "properties": {
                "path": {"type": "string"},
                "content": {"type": "string"}
            },
            "required": ["path", "content"]
        }
    }
]

def run_agent(task: str) -> str:
    messages = [{"role": "user", "content": task}]
    
    while True:
        response = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=4096,
            tools=tools,
            messages=messages
        )
        
        # 完成
        if response.stop_reason == "end_turn":
            return response.content[-1].text
        
        # 需要使用工具
        if response.stop_reason == "tool_use":
            messages.append({"role": "assistant", "content": response.content})
            
            tool_results = []
            for block in response.content:
                if block.type == "tool_use":
                    result = execute_tool(block.name, block.input)
                    tool_results.append({
                        "type": "tool_result",
                        "tool_use_id": block.id,
                        "content": result
                    })
            
            messages.append({"role": "user", "content": tool_results})
```

---

### 2. Orchestrator-Subagent 模式

一個主 Agent 負責規劃和協調，多個子 Agent 執行具體任務。

```
                 Orchestrator（規劃）
                /       |       \
           子 Agent 1  子 Agent 2  子 Agent 3
           （前端）    （後端）    （測試）
```

**適用場景**：任務可以分解為獨立的子任務並行執行

**設計要點**：
- Orchestrator 使用更強的模型（Opus）做規劃
- Subagent 使用較便宜的模型（Sonnet/Haiku）執行
- 子任務之間的依賴關係需要 Orchestrator 管理

```python
def orchestrate_coding_task(feature_description: str):
    # Orchestrator：用 Opus 做高層規劃
    plan = client.messages.create(
        model="claude-opus-4-7",
        messages=[{
            "role": "user",
            "content": f"""
將以下功能拆解成可並行的子任務，每個子任務獨立、互不依賴：

功能：{feature_description}

輸出格式：JSON 陣列，每個元素包含 task_id、description、files_to_modify
"""
        }]
    )
    
    tasks = json.loads(plan.content[0].text)
    
    # 並行執行子任務（使用 ThreadPoolExecutor）
    import concurrent.futures
    with concurrent.futures.ThreadPoolExecutor() as executor:
        futures = {
            executor.submit(run_subagent, task): task
            for task in tasks
        }
        results = {task["task_id"]: f.result() for f, task in futures.items()}
    
    return results
```

---

### 3. Reflection 模式（自我反思）

Agent 在完成任務後，由另一個 Claude 實例（或同一個）審查輸出，並決定是否需要修正。

```
生成 → 審查 → [通過] → 輸出
              [不通過] → 修正 → 審查（重複）
```

**適用場景**：程式碼品質要求高、安全敏感的任務

```python
def generate_with_reflection(task: str, max_iterations: int = 3) -> str:
    result = generate(task)
    
    for i in range(max_iterations):
        review = client.messages.create(
            model="claude-opus-4-7",
            messages=[{
                "role": "user",
                "content": f"""
審查以下程式碼，判斷是否達到以下標準：
1. 無安全漏洞（SQL injection, XSS 等）
2. 有完整的 error handling
3. 有對應的 unit test

程式碼：
{result}

如果全部通過，回覆 PASS。
如果有問題，列出具體問題清單（不要修正，只列問題）。
"""
            }]
        )
        
        review_text = review.content[0].text
        if "PASS" in review_text:
            return result
        
        # 根據審查意見修正
        result = fix(result, review_text)
    
    return result
```

---

### 4. Specialized Agents 模式

針對不同專業領域建立不同的 Agent，每個 Agent 有高度特化的 system prompt 和工具集。

```
路由 Agent
  ├── 程式碼 Agent（精通 Go/Python，有程式碼工具）
  ├── 資料分析 Agent（精通 SQL/Pandas，有資料庫工具）
  └── 文件 Agent（精通 Markdown，有文件工具）
```

```python
AGENTS = {
    "code": {
        "model": "claude-sonnet-4-6",
        "system": "你是資深 Go 後端工程師，專注程式碼品質和效能...",
        "tools": CODE_TOOLS
    },
    "data": {
        "model": "claude-sonnet-4-6",
        "system": "你是資料工程師，精通 SQL 和資料管線設計...",
        "tools": DATA_TOOLS
    }
}

def route_to_agent(user_request: str) -> str:
    # 先讓 Claude 判斷應該路由到哪個 Agent
    routing = client.messages.create(
        model="claude-haiku-4-5-20251001",
        messages=[{
            "role": "user",
            "content": f"判斷這個請求屬於哪個類別（code/data/doc）：{user_request}"
        }]
    )
    agent_type = routing.content[0].text.strip()
    agent_config = AGENTS[agent_type]
    
    return run_agent_with_config(user_request, agent_config)
```

---

## 控制 Agent 的自主程度

自主程度越高，速度越快、成本越低，但風險也越高：

| 模式 | 描述 | 適用場景 |
|------|------|---------|
| 全人工確認 | 每個工具呼叫都要確認 | 生產環境高風險操作 |
| 里程碑確認 | 完成特定階段才確認 | 多步驟開發任務 |
| 規則限制 | 設定允許/禁止的操作清單 | 受限環境自動化 |
| 完全自主 | 只確認最終結果 | 低風險、可逆的任務 |

---

## Agent 設計的核心原則

### 1. 最小權限原則

只給 Agent 完成任務所需的最少工具和權限：

```python
# ❌ 給太多工具
tools = [read_file, write_file, delete_file, run_shell, call_api, send_email]

# ✅ 只給這個任務需要的
tools = [read_file, write_file]  # 只是要做格式轉換
```

### 2. 明確的停止條件

Agent 需要知道什麼時候算「完成」：

```python
system_prompt = """
當以下所有條件達成時，停止並回報結果：
1. 所有 unit test 通過（go test ./... 回傳 0）
2. golangci-lint 沒有新的錯誤
3. 修改的檔案不超過 5 個

如果嘗試 3 次後仍無法達成，停止並說明原因。
"""
```

### 3. Checkpoint 機制

長時間任務要定期儲存狀態，避免中途失敗從頭來：

```python
import json

def agent_with_checkpoint(task: str, checkpoint_file: str):
    # 嘗試從上次的 checkpoint 繼續
    if os.path.exists(checkpoint_file):
        with open(checkpoint_file) as f:
            state = json.load(f)
        messages = state["messages"]
        print(f"從 checkpoint 繼續（{len(messages)} 條訊息）")
    else:
        messages = [{"role": "user", "content": task}]
    
    while True:
        response = run_one_step(messages)
        messages.append(...)
        
        # 每步儲存 checkpoint
        with open(checkpoint_file, "w") as f:
            json.dump({"messages": messages}, f)
        
        if is_done(response):
            os.remove(checkpoint_file)
            return response
```

---

## 常見陷阱

| 陷阱 | 問題 | 對策 |
|------|------|------|
| 無限迴圈 | Agent 沒有停止條件 | 設定最大迭代次數 |
| 工具呼叫爆炸 | 一個任務呼叫幾百次工具 | 限制工具呼叫次數 |
| Context 爆掉 | 長任務超過 context window | 定期壓縮對話歷史 |
| 幻覺工具結果 | Claude 假設工具成功 | 確保所有工具回傳都被驗證 |
| 任務蔓延 | Agent 超出原始任務範圍 | 明確的任務邊界定義 |

---

## 相關頁面

- [[Claude API基礎與最佳實踐]]
- [[Claude Prompt工程核心技巧]]
- [[OpenSpec工作流]]
- [[Superpowers技能框架]]
- [[生產環境Vibe Coding四大策略]]
