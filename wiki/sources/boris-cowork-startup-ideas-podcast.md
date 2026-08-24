---
title: "Boris 示範 CoWork：Startup Ideas Podcast"
type: source-summary
tags: [claude-code, cowork, anthropic, boris, agent]
created: 2026-04-30
updated: 2026-04-30
sources: []
---

# Boris 示範 CoWork：Startup Ideas Podcast

## Origin

- **節目**：Startup Ideas Podcast
- **主持人**：Greg
- **來賓**：Boris（Claude Code 創造者，Anthropic）
- **主題**：CoWork 實機 Demo + Boris 的 Claude Code 使用心法（99K 書籤推文展開）

---

## Key Takeaways

1. **CoWork = Claude Code 的圖形化介面**：底層是同一套 Claude Agent SDK，把終端機操作包進桌面 App，非技術人員打開就能用（macOS 先行，Windows 近期跟進）

2. **逆向引導（Reverse Elicitation）**：遇到不確定的情境（如收據無日期），Claude 不自己猜，而是主動問用戶——這是負責任的 AI 行為設計

3. **跨 App 自動化**：CoWork 可跨越 Finder → Excel → Google Sheets → Gmail 完成端到端任務，用戶只需要打幾句話

4. **沙盒安全架構**：底層是完整虛擬機，含刪除保護 + Prompt Injection 防護，Anthropic 的安全研究從模型訓練延伸到產品層

5. **並行工作法**：Boris 同時跑 5–10 個 Claude，像管理者照顧任務而非親自執行——單一任務人類更快，但並行整體效率倒轉

6. **Boris 的 Claude Code 四大心法**（爆紅推文 99K 書籤）：
   - **計畫模式（Plan Mode）**：最被低估的功能；計畫好了，程式碼就好了
   - **Opus 4.5 + 思考模式**：每 token 貴但更聰明、用更少 token，整體成本反而低
   - **CLAUDE.md check in 到 Git**：全團隊共同維護，每次 Claude 犯錯就加進去，同一件事不用講第二次
   - **給 Claude 驗證自己的方式**：用 Chrome 擴充套件讓 Claude 看到自己的輸出——能自我驗證的 Claude，品質大幅提升

7. **指數成長心態**：Boris 過去兩個月 Claude Code 寫了他 100% 的程式碼，驗證了他與 Dario 一年前的預測。不能用線性思維推模型進步

---

## Entities Mentioned

- [[Boris]] — Claude Code 創造者，Anthropic
- [[Anthropic]] — CoWork 母公司
- [[Cat Wu]] — 同為 Anthropic Claude 產品負責人（間接相關）

## Concepts Mentioned

- [[CoWork桌面工具指南]] — CoWork 使用方式 + Boris 四大心法
- [[Claude Code 入門完整指南]] — Plan Mode 的核心地位
- [[CLAUDE.md撰寫最佳實踐]] — Git check in 全團隊維護
- [[Claude Code多人團隊協作指南]] — 並行多任務管理模式

## Contradictions / Tensions

- Boris 說「Opus 4.5 雖貴但整體 token 用量少，成本反而低」——違反直覺，與「用便宜模型省錢」的常識相反，值得在自己的工作流中實測

## Questions Raised

- CoWork 的 Skills 自訂細節（AutoCAD、Salesforce 等特殊工具）如何實作？
- Chrome 擴充套件如何讓 Claude 「看到」自己的輸出？具體技術機制？
- 多個 Claude 並行跑時，如何管理 context 不互相干擾？
