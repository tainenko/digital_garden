---
title: 系統設計題面試八股文背誦版（牛客網）
type: source-summary
tags: [系統設計, 面試, 八股文, QPS, 分散式ID, TinyURL, Top-K, 中文]
created: 2026-05-13
updated: 2026-05-13
sources: [nowcoder-system-design-bagu]
---

# 系統設計題面試八股文背誦版（牛客網）

## Origin

- **URL**: https://www.nowcoder.com/discuss/377179563154055168
- **Author**: 烟雨（廣州大學，Java 方向）
- **Date**: 2022-07-14
- **Platform**: 牛客網（Nowcoder）

## Key Takeaways

- 四步答題框架：①澄清需求與用例 → ②高層架構設計 → ③核心組件詳設 → ④識別與優化瓶頸
- **QPS/TPS 效能基準**（面試估算常用）：Nginx 30萬、Redis 1萬–10萬、MySQL 讀數十萬/寫約10萬、Tomcat 2萬
- 指標體系：PV、UV、IP、DAU、MAU；響應時間、並發數、吞吐量、QPS、TPS
- **TinyURL 設計要點**：建議 302 暫時重定向（可追蹤點擊數據）而非 301 永久重定向；支援一個長網址對應多個短網址以便分析
- **分散式 ID 生成**：UUID、多主 MySQL + 自增步長、Twitter Snowflake（時間戳+機器碼+序列號）
- **任務排程**：DelayQueue 實作延時任務；分散式排程需考慮冪等性與分散式鎖
- **串流算法題型**（Top K / Streaming）：Top 10 高頻 IP（1小時滑動窗口）、Top K 頻繁元素、基數估計（HyperLogLog）、頻率估計（Count-Min Sketch）、範圍查詢、成員查詢（Bloom Filter）
- **KV Storage Engine** 設計：LSM Tree + SSTable 為常見答案

## Entities Mentioned

- 牛客網（Nowcoder）— 中文技術面試準備平台

## Concepts Mentioned

- [[系統設計效能基準與QPS速查]] — QPS/TPS 基準數字整理
- [[Back-of-the-Envelope 估算]] — 配合 QPS 基準使用
- [[分散式任務排程系統]] — DelayQueue + 分散式鎖
- [[sd-url-shortener|URL Shortener]] — 302 vs 301 重定向設計決策

## Contradictions/Tensions

- MySQL 寫入 QPS「約10萬」偏樂觀，實際生產環境視硬體與查詢複雜度差異極大
- 此文 2022 年發布，部分技術選型（如 Snowflake ID 替代方案）已有更多新選項（如 ULID、NanoID）

## Questions Raised

- Count-Min Sketch 與 HyperLogLog 的具體面試考法？是否有對應 LeetCode 題？
- Snowflake ID 在時鐘回撥（clock skew）情況下的處理方式值得深入
