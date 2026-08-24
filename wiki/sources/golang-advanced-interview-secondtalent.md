---
title: 23 Advanced Golang Backend Interview Questions
type: source-summary
tags: [golang, interview, concurrency, runtime, system-design, backend]
created: 2026-04-21
updated: 2026-04-21
sources: [2026-04-21_multi-source_golang-principal-interview.md]
---

# 23 Advanced Golang Backend Interview Questions

## Origin
- **標題**: 23 Advanced Golang Backend Interview Questions for Senior Role
- **作者**: Second Talent
- **URL**: https://www.secondtalent.com/interview-guide/golang/

## Key Takeaways
1. Go 排程器 GMP 模型：G (goroutine) / M (OS thread) / P (processor)，是面試必考核心
2. Channels 適合溝通與所有權轉移；Mutex 適合保護共享狀態——「share memory by communicating」
3. Context 是分散式系統中取消/逾時/傳值的統一機制，跨 API 邊界必用
4. 6 大並發模式：Generator、Worker Pool、Fan-in/Fan-out、Rate Limiting、Pipeline、Semaphore
5. GC 採並發三色標記清除，STW pause < 1ms；sync.Pool 可大幅降低 GC 壓力
6. Graceful shutdown：監聽 SIGINT → `http.Server.Shutdown(ctx)` → 等現有連線完成
7. DB 連線池：`SetMaxOpenConns`、`SetMaxIdleConns`、`SetConnMaxLifetime` 三參數調優
8. Rate Limiter：單機用 `golang.org/x/time/rate`（token bucket）；分散式用 Redis atomic ops

## Entities mentioned
（無特定 entity）

## Concepts mentioned
- [[Go執行期內部機制]] — GMP 排程器、GC、escape analysis
- [[Go並發模式]] — 6 大並發模式、channels vs mutexes
- [[Go效能調優]] — sync.Pool、pprof、DB 連線池

## Contradictions/tensions
- Channels 雖是 Go 哲學核心，但在 CPU-intensive 場景效能不如 mutex/atomic

## Questions raised
- 實際工作中何時選擇 channels vs sync primitives 的決策框架？
