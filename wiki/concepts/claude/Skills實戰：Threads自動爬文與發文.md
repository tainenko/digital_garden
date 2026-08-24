---
title: Skills 實戰：Threads 自動爬文與發文
type: concept
tags: [claude, skills, threads, 社群媒體, 自動化, MCP, playwright]
created: 2026-04-30
updated: 2026-04-30
sources: []
---

# Skills 實戰：Threads 自動爬文與發文

本教程用 Claude Code Skills + MCP + Threads API，建立一套**自動化社群操作工作流**：
- `threads-scrape`：爬取特定話題或帳號的貼文
- `threads-draft`：根據爬回的內容讓 Claude 生成草稿
- `threads-post`：審核後發布到 Threads

> 核心原則：**爬與讀由 Claude 執行，發文一律人工確認後才送出。**

---

## 整體架構

```
Claude Code
    │
    ├── threads-scrape (Skill)
    │       └── Playwright MCP → threads.net（無帳號讀取公開貼文）
    │           或 Threads API → 讀取自己帳號的資料
    │
    ├── threads-draft (Skill)
    │       └── Claude 分析爬回的貼文 → 生成草稿 → 存入 drafts/
    │
    └── threads-post (Skill)
            └── 人工確認 → Threads API → 發文
```

---

## 前置準備

### 方案 A：Threads 官方 API（推薦，限制多但穩定）

Threads API 是 Meta 的官方 Graph API 延伸，支援發文和讀取自己帳號的內容。

```bash
# 需要的東西：
# 1. Meta Developer 帳號 → 建立 App
# 2. 申請 Threads API 存取權限
# 3. 取得 Long-Lived Access Token（有效期 60 天）

# 安裝 Python 客戶端
pip install requests python-dotenv
```

```bash
# .env
THREADS_ACCESS_TOKEN=EAAxxxxxxxx...
THREADS_USER_ID=12345678901234
```

**Threads API 的限制**（重要）：
- 發文：每個帳號每 24 小時上限 250 篇
- 讀取：只能讀自己帳號和公開內容，不能大量爬其他帳號
- 沒有「搜尋關鍵字」的 API，公開搜尋只能靠瀏覽器自動化

### 方案 B：Playwright MCP（讀取公開內容）

用瀏覽器自動化爬取 Threads 公開頁面，不需要 API，但需要登入才能看完整內容。

```bash
# 安裝 Playwright MCP Server
npm install -g @playwright/mcp

# 在 settings.json 加入
```

```json
// .claude/settings.json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp", "--headless"]
    }
  }
}
```

---

## 專案目錄結構

```
threads-automation/
├── CLAUDE.md              ← Claude 行為規範（必讀）
├── .claude/
│   └── settings.json      ← MCP + 權限設定
├── .claude/skills/
│   ├── threads-scrape.md  ← 爬文 Skill
│   ├── threads-draft.md   ← 生成草稿 Skill
│   └── threads-post.md    ← 發文 Skill
├── scripts/
│   ├── threads_api.py     ← Threads API 封裝
│   ├── scraper.py         ← 瀏覽器爬蟲邏輯
│   └── post.py            ← 發文邏輯
├── data/
│   └── scraped/           ← 爬回的原始貼文（JSONL）
├── drafts/                ← Claude 生成的草稿（等待審核）
└── posted/                ← 已發出的貼文記錄
```

---

## CLAUDE.md 規範

```markdown
# Threads 自動化 — Claude 規範

## ⛔ 絕對禁止
- 未經我確認，不得呼叫任何發文 API（POST 類請求）
- 不得儲存或分享任何帳號的登入憑證
- 不得爬取私人帳號或需要特別授權的內容

## ✅ 可以自主執行
- 讀取 Threads 公開頁面
- 呼叫 Threads API 的 GET 請求（讀取資料）
- 在 data/scraped/ 存入爬回的貼文
- 在 drafts/ 建立草稿檔
- 分析貼文內容、生成草稿

## 📋 發文流程（不可跳過）
1. 草稿存入 drafts/<timestamp>.md
2. 顯示草稿內容給我看
3. 等我輸入「確認發文」才能呼叫 post.py
4. 發文後記錄到 posted/<timestamp>.json

## 🛑 遇到以下情況必須停下
- API 回傳任何錯誤
- 草稿內容涉及爭議性話題
- 一次要發超過 1 篇文
```

---

## Skill 1：threads-scrape

**檔案位置**：`.claude/skills/threads-scrape.md`

```markdown
# threads-scrape

## 觸發條件
當用戶說「爬 Threads」、「抓貼文」、「scrape threads」時使用

## 用途
爬取 Threads 上的公開貼文，儲存到 data/scraped/ 供後續分析

## 執行步驟

### 模式 A：爬取指定帳號（用 Threads API）
1. 讀取 .env 取得 THREADS_ACCESS_TOKEN 和 THREADS_USER_ID
2. 執行：`python scripts/threads_api.py get-posts --limit 20`
3. 結果存入 `data/scraped/<YYYYMMDD_HHMMSS>_<帳號>.jsonl`
4. 顯示摘要：共幾篇、時間範圍、互動數最高的 3 篇

### 模式 B：爬取關鍵字（用 Playwright 瀏覽器）
1. 詢問用戶：「請輸入要搜尋的關鍵字或 hashtag」
2. 用 Playwright MCP 開啟 https://www.threads.net/search?q=<關鍵字>
3. 滾動頁面抓取可見貼文（最多 30 篇）
4. 結果存入 `data/scraped/<YYYYMMDD_HHMMSS>_<關鍵字>.jsonl`
5. 顯示：共幾篇、熱門帳號、高互動貼文前 3 名

## 資料格式（JSONL，每行一個 JSON）
```json
{
  "id": "...",
  "username": "...",
  "text": "...",
  "timestamp": "2026-04-30T10:00:00Z",
  "likes": 123,
  "replies": 45,
  "reposts": 12,
  "url": "https://www.threads.net/@.../post/..."
}
```

## 允許的工具
allowed-tools: Bash(python scripts/threads_api.py:*), Bash(python scripts/scraper.py:*), Write(data/scraped/*), Read

## 不允許
- 不讀取需要登入才能看的私人內容
- 不儲存個人身份資料（頭像、私訊等）
```

### 對應的 Python 腳本

```python
# scripts/threads_api.py
import os, sys, json, requests
from datetime import datetime
from pathlib import Path
from dotenv import load_dotenv

load_dotenv()

TOKEN = os.environ["THREADS_ACCESS_TOKEN"]
USER_ID = os.environ["THREADS_USER_ID"]
BASE_URL = "https://graph.threads.net/v1.0"


def get_posts(limit: int = 20) -> list[dict]:
    """讀取自己帳號的貼文"""
    url = f"{BASE_URL}/{USER_ID}/threads"
    params = {
        "fields": "id,text,timestamp,like_count,replies_count,repost_count,permalink",
        "limit": limit,
        "access_token": TOKEN,
    }
    resp = requests.get(url, params=params)
    resp.raise_for_status()
    return resp.json().get("data", [])


def save_posts(posts: list[dict], label: str) -> Path:
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    out_path = Path(f"data/scraped/{timestamp}_{label}.jsonl")
    out_path.parent.mkdir(parents=True, exist_ok=True)
    with open(out_path, "w", encoding="utf-8") as f:
        for post in posts:
            f.write(json.dumps(post, ensure_ascii=False) + "\n")
    return out_path


if __name__ == "__main__":
    import argparse
    parser = argparse.ArgumentParser()
    subparsers = parser.add_subparsers(dest="cmd")

    p = subparsers.add_parser("get-posts")
    p.add_argument("--limit", type=int, default=20)
    p.add_argument("--label", default="my_account")

    args = parser.parse_args()
    if args.cmd == "get-posts":
        posts = get_posts(args.limit)
        path = save_posts(posts, args.label)
        print(f"✅ 儲存 {len(posts)} 篇貼文到 {path}")
        # 顯示摘要
        sorted_posts = sorted(posts, key=lambda p: p.get("like_count", 0), reverse=True)
        print("\n📊 互動最高前 3 篇：")
        for i, post in enumerate(sorted_posts[:3], 1):
            preview = (post.get("text") or "")[:60]
            print(f"  {i}. ❤️ {post.get('like_count', 0)} | {preview}...")
```

---

## Skill 2：threads-draft

**檔案位置**：`.claude/skills/threads-draft.md`

```markdown
# threads-draft

## 觸發條件
當用戶說「生成草稿」、「幫我寫 Threads 文」、「draft」時使用

## 用途
根據 data/scraped/ 的內容、或用戶提供的主題，生成 Threads 貼文草稿

## 執行步驟

1. 確認草稿需求：
   - 詢問用戶：「要根據最近爬到的內容延伸？還是給我一個主題？」
   - 若根據爬到的內容：讀取 data/scraped/ 最新一個檔案
   - 若是新主題：直接以主題生成

2. 生成草稿時遵守：
   - Threads 單篇上限 500 字（繁體中文約 200–250 字）
   - 語氣自然、像在跟朋友說話，不要像廣告文案
   - 結尾可放 1–2 個相關 hashtag，不要超過 3 個
   - 不要第一行就 hashtag，要有實質內容開頭

3. 生成 3 個版本（語氣各不同）：
   - 版本 A：觀點分享型（我認為...）
   - 版本 B：提問互動型（你有沒有...？）
   - 版本 C：短乾貨型（3 個重點...）

4. 存入 drafts/<YYYYMMDD_HHMMSS>.md：
   ```markdown
   # 草稿 - 2026-04-30 10:00
   主題：...
   來源：...（爬取資料或用戶指定主題）
   
   ## 版本 A（觀點分享）
   ...
   
   ## 版本 B（提問互動）
   ...
   
   ## 版本 C（短乾貨）
   ...
   ```

5. 顯示給用戶選擇，等待用戶選「A/B/C」或要求修改

## 允許的工具
allowed-tools: Read(data/scraped/*), Write(drafts/*), Read(drafts/*)

## 禁止
- 不要生成虛假資訊或誤導性內容
- 不要抄貼其他帳號的原文（改寫可以）
- 不要在草稿中放 URL（Threads 不利於外部連結傳播）
```

### 草稿生成的 Prompt 技巧

當 Claude 執行 threads-draft，它會使用以下框架思考（可以寫進 SKILL.md 的補充說明）：

```markdown
## Claude 的思考框架（附在 threads-draft.md 內）

生成草稿時，考慮以下框架：

Threads 高互動貼文的特徵（來自觀察）：
1. **開頭有張力**：第一句話讓人想繼續讀（問題、反直覺觀點、故事開頭）
2. **具體勝於抽象**：「我花了 3 小時學了 1 件事」比「學習很重要」更好
3. **留白讓人回應**：結尾的問題要真的有意思，不是「你同意嗎？」
4. **長度節制**：Threads 不是部落格，150–200 字最佳
5. **hashtag 克制**：1–2 個精準 tag > 5 個泛標籤

禁忌：
- 連結農場（整篇都在推銷或導流）
- 垃圾 hashtag 堆疊
- 複製貼上的格式感（清單太多）
- AI 感很重的開頭（「當然！以下是...」）
```

---

## Skill 3：threads-post

**檔案位置**：`.claude/skills/threads-post.md`

```markdown
# threads-post

## 觸發條件
當用戶說「發文」、「post」、「確認發文」時使用
⚠️ 必須有明確的確認指令，不能因為草稿已完成就自動觸發

## 前置條件（必須全部滿足才能執行）
- [ ] drafts/ 目錄下有已選定的草稿檔案
- [ ] 用戶在對話中明確說了「確認發文」或「post version X」
- [ ] 草稿內容已顯示給用戶看過

## 執行步驟

1. 再次顯示要發送的完整貼文內容
2. 詢問最後確認：「確認要發送這篇到 Threads 嗎？（輸入 yes 繼續）」
3. 等用戶輸入 yes
4. 執行：`python scripts/post.py --draft drafts/<選定的草稿>.md --version <A/B/C>`
5. 顯示發文結果（成功 + 貼文 URL）
6. 將記錄寫入 posted/<timestamp>.json

## 允許的工具
allowed-tools: Read(drafts/*), Bash(python scripts/post.py:*), Write(posted/*)

## 絕對禁止
- 沒有用戶明確的 yes 回應就執行 post.py
- 一次發超過 1 篇
- 修改草稿內容後直接發（修改後要重新確認）
```

### 對應的發文腳本

```python
# scripts/post.py
import os, sys, json, re, argparse
from datetime import datetime
from pathlib import Path
import requests
from dotenv import load_dotenv

load_dotenv()

TOKEN = os.environ["THREADS_ACCESS_TOKEN"]
USER_ID = os.environ["THREADS_USER_ID"]
BASE_URL = "https://graph.threads.net/v1.0"


def extract_version(draft_path: Path, version: str) -> str:
    """從草稿 md 文件提取指定版本的內容"""
    content = draft_path.read_text(encoding="utf-8")
    # 找到對應版本區塊
    version_map = {"A": "版本 A", "B": "版本 B", "C": "版本 C"}
    section = version_map.get(version.upper())
    if not section:
        raise ValueError(f"無效版本：{version}，請輸入 A/B/C")
    
    # 解析 markdown 區塊
    pattern = rf"## {re.escape(section)}.*?\n(.*?)(?=\n##|\Z)"
    match = re.search(pattern, content, re.DOTALL)
    if not match:
        raise ValueError(f"找不到 {section} 的內容")
    return match.group(1).strip()


def create_post(text: str) -> dict:
    """呼叫 Threads API 發文（兩步驟：先建立 container，再發布）"""
    # 步驟 1：建立 media container
    container_url = f"{BASE_URL}/{USER_ID}/threads"
    container_resp = requests.post(container_url, data={
        "media_type": "TEXT",
        "text": text,
        "access_token": TOKEN,
    })
    container_resp.raise_for_status()
    container_id = container_resp.json()["id"]

    # 步驟 2：發布
    publish_url = f"{BASE_URL}/{USER_ID}/threads_publish"
    publish_resp = requests.post(publish_url, data={
        "creation_id": container_id,
        "access_token": TOKEN,
    })
    publish_resp.raise_for_status()
    return publish_resp.json()


def save_record(text: str, result: dict, draft_path: Path, version: str):
    timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
    record = {
        "timestamp": timestamp,
        "draft_source": str(draft_path),
        "version": version,
        "text": text,
        "post_id": result.get("id"),
    }
    out = Path(f"posted/{timestamp}.json")
    out.parent.mkdir(exist_ok=True)
    out.write_text(json.dumps(record, ensure_ascii=False, indent=2))
    return out


if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--draft", required=True)
    parser.add_argument("--version", required=True, choices=["A", "B", "C"])
    args = parser.parse_args()

    draft_path = Path(args.draft)
    text = extract_version(draft_path, args.version)

    print(f"\n📝 即將發送的內容：\n{'─'*40}\n{text}\n{'─'*40}")
    print(f"\n字數：{len(text)} 字")
    
    # 最後防線：確認
    confirm = input("\n⚠️  確認發送？（輸入 yes 繼續，其他取消）：").strip().lower()
    if confirm != "yes":
        print("❌ 已取消")
        sys.exit(0)

    result = create_post(text)
    record_path = save_record(text, result, draft_path, args.version)
    
    post_id = result.get("id", "unknown")
    print(f"\n✅ 發文成功！")
    print(f"   Post ID: {post_id}")
    print(f"   記錄存於: {record_path}")
```

---

## 完整工作流程示範

### 情境 1：爬熱門話題 → 生成相關貼文

```
你：幫我爬 Threads 上的 #ai 話題，然後根據爬到的內容生成草稿

Claude：（執行 threads-scrape，模式 B，關鍵字 ai）
✅ 爬取完成，共 28 篇貼文存入 data/scraped/20260430_103045_ai.jsonl

📊 互動最高前 3 篇：
1. ❤️ 2,341 | Claude 4 讓我重新思考 AI 助手的邊界...
2. ❤️ 1,872 | 用 vibe coding 三天做出了一個...
3. ❤️  934 | 不是 AI 不夠強，是我們問的問題太弱...

（執行 threads-draft）
📝 已生成草稿 drafts/20260430_103122.md，共 3 個版本：

版本 A（觀點分享）：
「AI 最大的限制不在模型，在使用者的想像力上限。
我發現很多人用 Claude 只是在做「高級 Google」——問問題、
要答案。但當你開始把它當成一個可以對話的思考夥伴，
整個使用體驗完全不同。你最近讓 AI 做了什麼讓你最驚訝的事？
#AI #工作效率」

版本 B（提問互動）：...
版本 C（短乾貨）：...

你想選哪個版本？或是要我再調整？

你：版本 A 不錯，幫我把結尾的問題改得更具體一點

Claude：（修改後顯示新版本）
...

你：好，確認發文版本 A

Claude：⚠️ 確認要發送以下內容到 Threads 嗎？
[顯示完整內容]
輸入 yes 繼續 → 

（等待你在終端輸入 yes）
✅ 發文成功！
```

---

## Hooks 設定：定時爬文提醒

```json
// .claude/settings.json
{
  "mcpServers": { ... },
  "hooks": {
    "Stop": [
      {
        "matcher": ".*",
        "hooks": [{
          "type": "command",
          "command": "bash -c 'ls data/scraped/ | wc -l | xargs -I{} echo \"📁 目前已爬 {} 批貼文，最新：$(ls -t data/scraped/ | head -1)\"'"
        }]
      }
    ]
  }
}
```

---

## 常見問題

| 問題 | 原因 | 解法 |
|------|------|------|
| API 403 錯誤 | Token 過期（60天有效） | 重新申請 Long-Lived Token |
| 爬到的貼文很少 | Threads 有 bot 偵測 | 加入 `--slow` 模式，模擬人類滾動速度 |
| 草稿生成太像 AI | SKILL.md 沒有限制語氣 | 在 threads-draft.md 加入更多語氣禁忌 |
| 不小心重複發文 | 沒有去重機制 | 檢查 posted/ 目錄，加入 text hash 比對 |

---

## 相關頁面

- [[Claude Code Hooks 深度設定]]
- [[Claude MCP 伺服器整合指南]]
- [[Superpowers技能框架]]
- [[Anthropic Academy 13堂課程完整指南]]（Introduction to Agent Skills）
- [[Skills實戰：自動交易機器人]]
