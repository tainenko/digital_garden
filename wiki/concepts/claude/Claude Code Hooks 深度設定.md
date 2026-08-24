---
title: Claude Code Hooks 深度設定
type: concept
tags: [claude, claude-code, hooks, 自動化, 設定]
created: 2026-04-30
updated: 2026-04-30
sources: []
---

# Claude Code Hooks 深度設定

Claude Code Hooks 讓你在特定事件發生時自動執行 shell 指令，不需要每次手動要求 Claude 執行。這是實現「自動化品質門禁」的關鍵機制。

---

## Hooks 的本質

Hooks 是**你定義的、在特定時機自動執行的 shell 指令**。它們在 `settings.json` 中設定，由 Claude Code 框架呼叫——不是由 Claude 本身呼叫。Claude 看得到 hook 的輸出結果，但無法修改或繞過 hook 設定。

---

## 支援的事件類型

| 事件 | 觸發時機 |
|------|---------|
| `PreToolUse` | Claude 即將呼叫某個工具之前 |
| `PostToolUse` | Claude 完成工具呼叫之後 |
| `Notification` | Claude Code 發出通知時 |
| `Stop` | Claude 完成回應，停止輸出時 |
| `SubagentStop` | 子 agent 完成任務時 |

---

## 設定位置

```
# 專案級（只影響當前專案）
.claude/settings.json

# 全域（影響所有專案）
~/.claude/settings.json
```

---

## 基礎設定格式

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "gofmt -w $CLAUDE_FILE_PATHS"
          }
        ]
      }
    ]
  }
}
```

### 關鍵欄位說明

- **`matcher`**：正則表達式，匹配工具名稱（`Write`、`Edit`、`Bash`、`Read` 等）
- **`command`**：要執行的 shell 指令
- **`type`**：目前只有 `"command"`

---

## 實用環境變數

Hook 執行時可使用以下環境變數：

| 變數 | 內容 |
|------|------|
| `$CLAUDE_FILE_PATHS` | 被修改的檔案路徑（空格分隔） |
| `$CLAUDE_TOOL_NAME` | 當前工具名稱 |
| `$CLAUDE_TOOL_INPUT` | 工具的輸入（JSON） |
| `$CLAUDE_TOOL_OUTPUT` | 工具的輸出（JSON） |
| `$CLAUDE_SESSION_ID` | 當前對話 ID |

---

## 實際範例

### 範例一：自動格式化（Go）

每次 Claude 寫入 `.go` 檔案後，自動執行 `gofmt` 和 `goimports`：

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "bash -c 'for f in $CLAUDE_FILE_PATHS; do [[ $f == *.go ]] && goimports -w $f; done'"
          }
        ]
      }
    ]
  }
}
```

### 範例二：自動執行測試

每次 Claude 完成 Go 檔案修改後，自動跑受影響套件的測試：

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "bash -c 'for f in $CLAUDE_FILE_PATHS; do [[ $f == *.go ]] && go test $(dirname $f)/... && break; done'"
          }
        ]
      }
    ]
  }
}
```

### 範例三：禁止修改特定目錄

在 `PreToolUse` 攔截 Claude 對高風險目錄的修改，回傳非零 exit code 可阻止操作：

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "bash -c 'for f in $CLAUDE_FILE_PATHS; do [[ $f == *migrations/* ]] && echo \"錯誤：禁止修改 migrations 目錄\" && exit 1; done'"
          }
        ]
      }
    ]
  }
}
```

### 範例四：完成後桌面通知

Claude 完成長時間任務後自動彈出 macOS 通知：

```json
{
  "hooks": {
    "Stop": [
      {
        "matcher": ".*",
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Claude 完成了！\" with title \"Claude Code\"'"
          }
        ]
      }
    ]
  }
}
```

### 範例五：Lint 門禁

修改完成後執行 lint，若有問題則輸出錯誤供 Claude 自動修正：

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "bash -c '[[ $CLAUDE_FILE_PATHS == *.go ]] && golangci-lint run $CLAUDE_FILE_PATHS 2>&1 || true'"
          }
        ]
      }
    ]
  }
}
```

注意：`|| true` 讓 hook 不會因為 lint 錯誤而中斷流程，但 lint 輸出會出現在 Claude 的 context 中，它會看到並嘗試修正。

---

## Hook 的回傳機制

| Exit Code | 行為 |
|-----------|------|
| `0` | 成功，繼續正常流程 |
| 非 `0`（PreToolUse） | **阻止**工具呼叫，回傳錯誤給 Claude |
| 非 `0`（PostToolUse） | 輸出錯誤訊息到 Claude context，讓它知道有問題 |

stdout 的輸出會出現在 Claude 的 context 中，Claude 能讀到並據此調整行為。

---

## 設計原則

1. **冪等性**：Hook 指令要能重複執行不出錯
2. **速度**：慢的 hook 會拖慢每次互動，格式化可以接受，大量測試要謹慎
3. **範圍最小化**：只對真正需要的檔案類型觸發，用正則表達式做 matcher
4. **失敗策略**：預設讓 hook 失敗不影響主流程（`|| true`），除非是安全門禁

---

## 相關頁面

- [[Claude Code 入門完整指南]]
- [[CLAUDE.md撰寫最佳實踐]]
- [[生產環境Vibe Coding四大策略]]
- [[Superpowers技能框架]]
