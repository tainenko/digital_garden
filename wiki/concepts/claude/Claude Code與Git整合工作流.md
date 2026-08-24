---
title: Claude Code 與 Git/GitHub 整合工作流
type: concept
tags: [claude-code, git, github, workflow, pr, ci]
created: 2026-04-30
updated: 2026-04-30
sources: []
---

# Claude Code 與 Git/GitHub 整合工作流

## Claude Code 的 Git 能力

Claude Code 可直接操作 Git，不需要你手動執行：

```bash
# Claude 能做到的事
git status / diff / log
git add / commit / push
git checkout / branch / merge
gh pr create / review / merge  # 透過 GitHub CLI
```

但有些操作 Claude Code **預設需要確認**（安全機制）：
- `git push`（影響遠端）
- `git reset --hard`（破壞性操作）
- `git rebase -i`（互動模式）

---

## 日常開發工作流

### 開始新功能

```bash
# 告訴 Claude Code：
「幫我從 main 建立一個 feature/user-auth 分支，然後開始實作 JWT 認證」

# Claude 會：
# 1. git checkout -b feature/user-auth
# 2. 開始寫程式碼
# 3. 分批 commit（有意義的小 commit）
```

### 讓 Claude 自動分 commit

```bash
# 在 CLAUDE.md 設定：
「每完成一個獨立功能，就 commit 一次。
 Commit message 格式：
 <type>(<scope>): <description>
 例如：feat(auth): add JWT token generation」

# Claude 會自動在合適時機 commit，不需要你說
```

### PR 建立

```bash
# 告訴 Claude Code：
「這個功能做完了，幫我建立 PR，描述要包含：
 - 做了什麼
 - 如何測試
 - 有什麼 breaking change」

# Claude 會執行：
gh pr create \
  --title "feat(auth): add JWT authentication" \
  --body "## Summary\n..."  \
  --base main \
  --draft  # 先建 draft，讓你確認
```

---

## CLAUDE.md 的 Git 設定

```markdown
## Git 規範

### Commit 規範
格式：<type>(<scope>): <description>

Types:
- feat: 新功能
- fix: Bug 修復
- refactor: 重構（不改功能）
- test: 新增/修改測試
- docs: 文件更新
- chore: 工具設定更新

### 分支命名
- 功能：feature/<ticket-id>-<short-description>
- Bug修復：fix/<ticket-id>-<short-description>
- 緊急修復：hotfix/<description>

### PR 規範
- 每個 PR 不超過 400 行變更
- 必須有測試覆蓋
- Draft PR 先建，review 後才 ready

### 禁止操作
- 不要 force push main/develop 分支
- 不要 commit .env 檔案
- Commit 前必須確認 `git diff` 沒有敏感資訊
```

---

## 與 GitHub Actions 整合

### 讓 Claude 撰寫 CI 配置

```bash
「幫我寫 GitHub Actions，要做到：
 - PR 時跑 lint + test + build
 - main 合併後自動部署到 staging
 - 測試覆蓋率低於 80% 就 fail」
```

### 讓 Claude Code 看 CI 結果

```bash
# CI 失敗後，告訴 Claude：
「gh run list 看最新的 CI，找出失敗原因並修復」

# Claude 會：
# 1. gh run list --limit 1
# 2. gh run view <id> --log-failed
# 3. 分析錯誤，修改程式碼
# 4. 再次 push
```

---

## Code Review 輔助

### 讓 Claude Review 你的 PR

```bash
# 在 Claude Code 中：
「review 這個 PR 的變更，重點關注：
 1. 安全性問題（SQL Injection、XSS、敏感資料洩露）
 2. 效能問題（N+1 查詢、不必要的 DB 呼叫）
 3. 測試是否完整
 4. 是否符合我們的 CLAUDE.md 規範」

# Claude 會讀取 git diff 並給出具體意見
```

### 自動回應 Review 意見

```bash
「GitHub PR #42 有幾條 review 意見，幫我看一下並修改：
 1. 'This function is too long, split it up'
 2. 'Missing error handling for nil case'」

# Claude 會：
# 1. 讀取相關程式碼
# 2. 修改
# 3. Commit 並 push
# 4. 在 PR 留言「已根據 review 意見修改」（需你確認）
```

---

## Commit Message 品質

### 讓 Claude 改善 Commit Message

```bash
「看一下最近 10 個 commit，幫我改寫描述不清楚的 commit message」

# Claude 會列出：
# abc1234 - "fix bug" → 建議改為 "fix(auth): handle nil user in JWT validation"
# def5678 - "update" → 建議改為 "refactor(db): extract query builder to repository layer"
```

### Commit Signing（GPG）

```bash
# 設定 CLAUDE.md：
「所有 commit 使用 GPG 簽署：git commit -S」

# 或在 git config：
git config --global commit.gpgsign true
```

---

## Gitignore 管理

```bash
「幫我更新 .gitignore，這個 Python + Go 混合專案需要忽略哪些檔案？」

# Claude 會加入：
# Python: __pycache__, *.pyc, .venv, .pytest_cache, .coverage
# Go: 編譯輸出, vendor/（若不使用 vendor）
# IDE: .idea, .vscode（或只忽略私人設定不忽略共享設定）
# 環境: .env, .env.local, *.pem, *.key
```

---

## 常見情境

### 緊急還原（Rollback）

```bash
「生產環境出問題，要還原到昨天的版本」

# Claude 會問你確認後：
# 1. git log --oneline 找昨天的 commit
# 2. git revert <commit>（不破壞歷史）
# 或
# 3. 告訴你用 git reset --hard 前後的差異，讓你決定
```

### 解決 Merge Conflict

```bash
「合併 main 到 feature/xxx 有衝突，幫我解決」

# Claude 會：
# 1. 讀取衝突檔案
# 2. 理解兩邊的意圖
# 3. 解決衝突（說明解決邏輯）
# 4. git add . && git commit
```

### Stash 管理

```bash
「我正在做 A 功能，突然要切去修一個緊急 Bug，怎麼辦？」

# Claude 會建議並執行：
# git stash push -m "WIP: feature-A"
# git checkout fix/urgent-bug
# ... 修 bug，commit，push ...
# git checkout feature/A
# git stash pop
```

---

## 安全注意事項

1. **不要讓 Claude 自動 push main**：用分支保護規則（Branch Protection）
2. **敏感資訊保護**：在 CLAUDE.md 明確說明「不 commit .env」
3. **確認 Push 操作**：設定 Claude Code 的 permission 讓 push 需要確認
4. **PR 合併前人工確認**：不讓 Claude 自動合併 PR

```json
// .claude/settings.json
{
  "permissions": {
    "allow": [
      "git:commit",
      "git:branch",
      "git:status",
      "git:diff"
    ],
    "deny": [
      "git:push:main",
      "git:push:production"
    ]
  }
}
```

---

## 相關頁面

- [[CLAUDE.md撰寫最佳實踐]] — 規範 Claude 的 Git 行為
- [[Claude Code Hooks 深度設定]] — commit 前自動執行 lint/test
- [[Claude Code 入門完整指南]] — Claude Code 基礎操作
- [[生產環境Vibe Coding四大策略]] — 生產環境的 Git 安全策略
