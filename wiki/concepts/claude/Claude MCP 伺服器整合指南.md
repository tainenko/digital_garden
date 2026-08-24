---
title: Claude MCP 伺服器整合指南
type: concept
tags: [claude, claude-code, MCP, 工具, 整合, scope]
created: 2026-04-30
updated: 2026-05-07
sources: [youtube-codememayb-claude-code-deep-dive]
---

# Claude MCP 伺服器整合指南

MCP（Model Context Protocol）是 Anthropic 推出的開放協定，讓 Claude 能透過標準化介面連接外部工具和資料來源。在 Claude Code 中啟用 MCP Server，能讓 Claude 直接操作資料庫、呼叫 API、讀寫外部服務。

---

## MCP 的設計理念

**傳統 Tool Use**：你在程式碼中定義工具，Claude 呼叫你的程式。
**MCP**：工具定義在獨立的 MCP Server 中，Claude Code 連接它，無需修改程式碼。

好處：
- 一次定義，多個 Claude 實例共用
- 社群分享 MCP Server（GitHub 上已有數百個）
- 工具邏輯獨立，可單獨測試和部署

---

## 在 Claude Code 中設定 MCP

### MCP 安裝 Scope（三個層級）

`claude mcp add` 支援三個安裝層級，用 `--scope` 指定：

| Scope | 指令 | 設定檔位置 | 是否可 check in Git |
|-------|------|-----------|-------------------|
| `local`（預設）| `--scope local` | `~/.claude.json`（使用者目錄）| 否 |
| `project` | `--scope project` | `<專案根>/.mcp.json` | **可以** |
| `user` | `--scope user` | 使用者層級設定 | 否 |

```bash
# 預設：local scope（只在本機這個專案有效）
claude mcp add playwright npx playwright-mcp

# project scope：存成 .mcp.json，可 check in Git 讓團隊共用
claude mcp add --scope project playwright npx playwright-mcp

# user scope：跨所有專案都有效
claude mcp add --scope user sqlite npx @anthropic/mcp-server-sqlite --db-path ./mydb.sqlite
```

**重要**：同一個 MCP server 安裝了兩個 scope（如 local 和 project 都有 playwright），啟動 Claude Code 時會出現衝突，需刪除其中一個。

### MCP 比喻

- **MCP Host**（Claude Code）= 你的電腦
- **MCP Server** = 外接 USB 裝置（工具提供者）
- **Tools** = 裝置上的每個功能（函式）

Claude Code 連上 MCP Server 後，LLM 就能把這些 Tools 當作 Tool Use 的一部分呼叫。

### 方法一：Claude Code 設定介面

```bash
# 啟動 Claude Code 後
claude mcp add <server-name> <command>

# 範例：加入 SQLite MCP Server
claude mcp add sqlite npx @anthropic/mcp-server-sqlite --db-path ./mydb.sqlite
```

### 方法二：直接編輯設定檔

```json
// ~/.claude/settings.json（全域）
// 或 .claude/settings.json（專案）
{
  "mcpServers": {
    "sqlite": {
      "command": "npx",
      "args": ["@anthropic/mcp-server-sqlite", "--db-path", "./mydb.sqlite"]
    },
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/Users/tonyk/projects"]
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_..."
      }
    }
  }
}
```

### 驗證 MCP 連接

```bash
# 啟動 Claude Code 後，輸入
/mcp

# 應該看到已連接的 MCP Server 清單
```

---

## 常用 MCP Server 清單

### 資料庫

| Server | 安裝 | 功能 |
|--------|------|------|
| SQLite | `@anthropic/mcp-server-sqlite` | 查詢、修改 SQLite 資料庫 |
| PostgreSQL | `@benborla29/mcp-server-postgres` | 連接 PostgreSQL |
| MySQL | `@benborla29/mcp-server-mysql` | 連接 MySQL |

### 開發工具

| Server | 安裝 | 功能 |
|--------|------|------|
| Filesystem | `@modelcontextprotocol/server-filesystem` | 讀寫本機檔案系統 |
| GitHub | `@modelcontextprotocol/server-github` | 操作 GitHub Issues/PR |
| Git | `@modelcontextprotocol/server-git` | Git 操作 |
| Docker | `mcp-server-docker` | 管理 Docker 容器 |

### 外部服務

| Server | 功能 |
|--------|------|
| Slack | 讀取 Slack 訊息、發送通知 |
| Notion | 讀寫 Notion 頁面 |
| Linear | 管理 Linear 工單 |
| Brave Search | 網頁搜尋 |

---

## 實際使用範例

### 場景一：讓 Claude 直接查資料庫

設定 PostgreSQL MCP Server 後，可以直接在對話中：

```
你：查一下 users 表裡最近 7 天內註冊，但還沒有完成 email 驗證的用戶有多少人

Claude：（使用 MCP 執行 SQL）
SELECT COUNT(*) FROM users 
WHERE created_at > NOW() - INTERVAL '7 days' 
AND email_verified_at IS NULL;

結果：共有 342 位用戶符合條件。
需要我進一步分析這些用戶的來源或行為嗎？
```

### 場景二：Claude 自動建立 GitHub Issue

設定 GitHub MCP Server 後：

```
你：幫我把剛才發現的 memory leak 問題建成 GitHub Issue，
   標記 bug 和 high-priority 標籤，指派給 @tonyk

Claude：（使用 MCP 呼叫 GitHub API）
Issue 建立成功：
標題：Memory leak in connection pool handler
URL：https://github.com/your-org/repo/issues/123
已指派給 @tonyk，標記 bug、high-priority
```

### 場景三：分析本機日誌

設定 Filesystem MCP Server 後：

```
你：讀取 /var/log/app/error.log，找出最常出現的 5 種錯誤類型

Claude：（使用 Filesystem MCP 讀取檔案）
分析 error.log（234 MB）完成，Top 5 錯誤：
1. "connection timeout to postgres" - 1,247 次
2. "rate limit exceeded" - 893 次
...
```

---

## 自己寫 MCP Server

如果找不到現成的 MCP Server，可以自己寫：

### TypeScript 版本（推薦）

```typescript
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const server = new Server(
  { name: "my-custom-server", version: "1.0.0" },
  { capabilities: { tools: {} } }
);

// 定義工具
server.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [
    {
      name: "get_taiwan_stock_price",
      description: "查詢台股即時股價",
      inputSchema: {
        type: "object",
        properties: {
          symbol: { type: "string", description: "台股代碼，例如 2330" }
        },
        required: ["symbol"]
      }
    }
  ]
}));

// 實作工具邏輯
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  if (request.params.name === "get_taiwan_stock_price") {
    const { symbol } = request.params.arguments as { symbol: string };
    // 呼叫台股 API...
    const price = await fetchTWStockPrice(symbol);
    return {
      content: [{ type: "text", text: JSON.stringify(price) }]
    };
  }
  throw new Error(`Unknown tool: ${request.params.name}`);
});

// 啟動
const transport = new StdioServerTransport();
await server.connect(transport);
```

### 在 settings.json 中加入自訂 Server

```json
{
  "mcpServers": {
    "tw-stock": {
      "command": "node",
      "args": ["/Users/tonyk/mcp-servers/tw-stock/build/index.js"]
    }
  }
}
```

---

## MCP vs 傳統 Tool Use 的選擇

| | MCP Server | 傳統 Tool Use |
|--|-----------|--------------|
| 適用場景 | Claude Code、長期使用的工具 | 一次性應用程式、API 整合 |
| 設定複雜度 | 高（需要獨立部署） | 低（直接在程式碼中定義） |
| 複用性 | 高（多個 Claude 實例共用） | 低（綁定特定程式碼） |
| 社群生態 | 豐富（GitHub 有大量現成的） | 無（自己寫） |
| 偵錯難度 | 稍高 | 低 |

---

## 安全注意事項

1. **MCP Server 的權限**：某些 Server（如 Filesystem）有極高的系統存取權，限制 `allowed_paths`
2. **環境變數保護**：API Key 和 Token 放在環境變數，不要硬編碼在設定檔
3. **不信任第三方 Server**：社群 MCP Server 要審查程式碼再使用
4. **注意 Prompt Injection**：惡意資料來源可能透過 MCP 注入指令

---

## 相關頁面

- [[Claude Code 入門完整指南]]
- [[Claude Code Hooks 深度設定]]
- [[Claude Agent 設計模式]]
- [[CLAUDE.md撰寫最佳實踐]]
