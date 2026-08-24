---
title: 系統設計題解總覽
type: topic
tags: [system-design, interview, index]
created: 2026-04-20
updated: 2026-04-20
---

# 系統設計題解總覽

共 23 道題解，含完整 Go 代碼實現。

---

## 概念題（Concept-Based）

| 題目 | 核心算法/模式 | 代碼重點 |
|------|------------|---------|
| [[sd-rate-limiter\|Rate Limiter]] | Token Bucket / Sliding Window Counter | Redis Lua 原子操作、Fail-open |
| [[sd-consistent-hashing\|Consistent Hashing]] | 虛擬節點、環狀 Hash | sort.SearchInts 二分搜尋 |
| [[sd-api-gateway-vs-lb\|API Gateway vs LB]] | Round Robin / Least Conn / IP Hash | Health Check、反向代理 |
| [[sd-sso\|SSO 單點登入]] | JWT / OAuth 2.0 | Token Pair、HttpOnly Cookie |

---

## 入門題（Easy）

| 題目 | 核心算法/模式 | 代碼重點 |
|------|------------|---------|
| [[sd-url-shortener\|URL Shortener]] | Base62 編碼、Cache-Aside | Encode/Decode、302 vs 301 |
| [[sd-distributed-cache\|分散式快取 / LRU]] | HashMap + 雙向鏈表 | Cache Stampede / Avalanche / Penetration 三大問題 |
| [[sd-parking-garage\|停車場系統]] | OOD、Strategy Pattern | 費率策略、Redis SETNX 原子佔位 |
| [[sd-cdn\|CDN 設計]] | Pull/Push、Anycast BGP、分層快取 | L1/L2/Origin 三層、Singleflight 防雪崩、主動 Purge |

---

## 中級題（Medium）

| 題目 | 核心算法/模式 | 代碼重點 |
|------|------------|---------|
| [[sd-social-media-feed\|社群媒體 Feed]] | Fanout on Write/Read/混合 | Redis Sorted Set、Kafka Fanout Worker |
| [[sd-chat-system\|即時聊天系統]] | WebSocket、Pub/Sub | Hub 連線管理、Snowflake ID、Cassandra Schema |
| [[sd-video-streaming\|影片串流]] | HLS 自適應碼率 | 分塊上傳、FFmpeg 轉碼、Master Playlist |
| [[sd-music-streaming\|音樂串流]] | DRM 加密、協同過濾推薦 | HMAC 簽名 URL、離線下載授權 |
| [[sd-typeahead\|搜尋自動補全]] | Trie + 預存 TopK | DFS → 預計算優化 O(L)、Debounce |
| [[sd-ticketing-system\|訂票系統]] | Optimistic Lock、Redis SETNX | 排隊 Waiting Room、15分鐘暫保留 |

---

## 進階題（Hard）

| 題目 | 核心算法/模式 | 代碼重點 |
|------|------------|---------|
| [[sd-uber\|叫車服務（Uber）]] | Geohash、相鄰格子搜尋 | Haversine 距離、司機配對逾時處理 |
| [[sd-web-crawler\|分散式網路爬蟲]] | Bloom Filter、優先佇列 | 去重記憶體估算、禮貌性爬取 |
| [[sd-dropbox\|雲端儲存（Dropbox）]] | Chunking、Content-addressable | SHA-256 去重、Delta Sync、衝突處理 |
| [[sd-google-docs\|協作文件（Google Docs）]] | OT（Operational Transformation）| Transform 函數、樂觀更新、游標同步 |

---

## 跨題共用的核心技術

```
Redis ──→ Rate Limiter, URL Shortener, Feed Cache, Ticketing, Presence
Kafka ──→ Feed Fanout, Chat Routing, Video Transcoding
WebSocket → Chat, Uber Location, Google Docs
Bloom Filter → Web Crawler URL 去重
Geohash ──→ Uber 位置索引
Trie ───→ Typeahead 搜尋
OT ────→ Google Docs 協作
```

---

## 學習路徑建議

```
1. LRU Cache（資料結構基礎）
2. Consistent Hashing（分散式基礎）
3. Rate Limiter（Redis 實戰）
4. URL Shortener（完整小系統）
     ↓
5. Social Media Feed（Fanout 核心概念）
6. Chat System（WebSocket + Kafka）
7. Ticketing System（高並發核心）
     ↓
8. Uber（地理索引）
9. Web Crawler（Bloom Filter）
10. Dropbox（分塊 + 去重）
11. Google Docs（OT，最難）
```

---

## 相關頁面

- [[系統設計面試模板]] — 每題的回答框架
- [[RESHADED面試框架]] — 結構化思考工具
- [[面試時間分配]] — 45分鐘怎麼分配
- [[面試評分標準]] — 面試官在看什麼
- [[系統設計核心技術棧]] — Redis/Kafka/ES 快速參考
