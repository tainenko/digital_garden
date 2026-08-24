---
title: "SD題解：SSO（單點登入）"
type: topic
tags: [system-design, sso, oauth, jwt, authentication, golang]
created: 2026-04-20
updated: 2026-04-20
---

# SD題解：SSO（單點登入）

> **難度**: 概念題 ｜ **頻率**: 高（Auth 架構必考）

---

## 什麼是 SSO

用戶只需登入一次，即可訪問多個相關系統，不需要每個系統都重新輸入帳密。

**例子**：登入 Google 帳號 → 自動登入 Gmail、YouTube、Google Drive

---

## 兩種主要實作方式

| | OAuth 2.0 + OIDC | SAML 2.0 |
|--|--|--|
| 資料格式 | JSON / JWT | XML |
| 適用場景 | Web、Mobile App | 企業內部系統 |
| 複雜度 | 中 | 高 |
| 常見用途 | 「用 Google 登入」 | 企業 SSO（Okta、Salesforce）|

**面試幾乎都考 OAuth 2.0 + JWT**

---

## OAuth 2.0 授權碼流程

```
用戶        客戶端 App      授權伺服器        資源伺服器
  |              |                |                |
  |-- 登入請求 -->|                |                |
  |              |-- 重定向到 -->  |                |
  |<------ 登入頁面（Google 登入）--|                |
  |-- 輸入帳密 ->|                |                |
  |              |<--- 授權碼 ----|                |
  |              |-- 授權碼 + Secret --> |          |
  |              |<-- Access Token + Refresh Token--|
  |              |-- Access Token -------->         |
  |              |<-- 用戶資料 --------------------|
```

---

## Go 實現

### JWT 生成與驗證

```go
package auth

import (
    "errors"
    "time"

    "github.com/golang-jwt/jwt/v5"
)

var secretKey = []byte("your-secret-key") // 實際應從環境變數讀取

type Claims struct {
    UserID   string   `json:"user_id"`
    Email    string   `json:"email"`
    Roles    []string `json:"roles"`
    jwt.RegisteredClaims
}

// GenerateToken 生成 JWT
func GenerateToken(userID, email string, roles []string) (string, error) {
    claims := Claims{
        UserID: userID,
        Email:  email,
        Roles:  roles,
        RegisteredClaims: jwt.RegisteredClaims{
            ExpiresAt: jwt.NewNumericDate(time.Now().Add(1 * time.Hour)),
            IssuedAt:  jwt.NewNumericDate(time.Now()),
            Issuer:    "my-sso-service",
        },
    }

    token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
    return token.SignedString(secretKey)
}

// ValidateToken 驗證並解析 JWT
func ValidateToken(tokenString string) (*Claims, error) {
    token, err := jwt.ParseWithClaims(tokenString, &Claims{},
        func(token *jwt.Token) (interface{}, error) {
            if _, ok := token.Method.(*jwt.SigningMethodHMAC); !ok {
                return nil, errors.New("unexpected signing method")
            }
            return secretKey, nil
        })

    if err != nil {
        return nil, err
    }

    if claims, ok := token.Claims.(*Claims); ok && token.Valid {
        return claims, nil
    }
    return nil, errors.New("invalid token")
}
```

### SSO 服務核心邏輯

```go
package sso

import (
    "crypto/rand"
    "encoding/base64"
    "sync"
    "time"
)

// Session 存儲（生產環境用 Redis）
type SessionStore struct {
    sessions map[string]*Session
    mu       sync.RWMutex
}

type Session struct {
    UserID    string
    CreatedAt time.Time
    ExpiresAt time.Time
}

func (s *SessionStore) Create(userID string) string {
    sessionID := generateRandomString(32)
    s.mu.Lock()
    s.sessions[sessionID] = &Session{
        UserID:    userID,
        CreatedAt: time.Now(),
        ExpiresAt: time.Now().Add(24 * time.Hour),
    }
    s.mu.Unlock()
    return sessionID
}

func (s *SessionStore) Get(sessionID string) (*Session, bool) {
    s.mu.RLock()
    defer s.mu.RUnlock()
    session, ok := s.sessions[sessionID]
    if !ok || time.Now().After(session.ExpiresAt) {
        return nil, false
    }
    return session, true
}

// SSOService 核心服務
type SSOService struct {
    sessionStore *SessionStore
    appRegistry  map[string]string // appID → callback URL
}

// Login 用戶登入，回傳 SSO Token
func (s *SSOService) Login(username, password string) (string, error) {
    userID, err := validateCredentials(username, password)
    if err != nil {
        return "", err
    }

    // 建立 Session
    sessionID := s.sessionStore.Create(userID)

    // 生成 JWT（含 Session ID）
    token, err := GenerateToken(userID, username+"@example.com", []string{"user"})
    if err != nil {
        return "", err
    }
    _ = sessionID // 在 cookie 中設置
    return token, nil
}

// ValidateServiceToken 子系統驗證 SSO Token
func (s *SSOService) ValidateServiceToken(token string) (*Claims, error) {
    return ValidateToken(token)
}

func generateRandomString(n int) string {
    b := make([]byte, n)
    rand.Read(b)
    return base64.URLEncoding.EncodeToString(b)
}
```

### Refresh Token 機制

```go
type TokenPair struct {
    AccessToken  string `json:"access_token"`
    RefreshToken string `json:"refresh_token"`
    ExpiresIn    int    `json:"expires_in"` // seconds
}

// GenerateTokenPair 生成 Access Token（短效）+ Refresh Token（長效）
func GenerateTokenPair(userID string) (*TokenPair, error) {
    accessToken, err := GenerateToken(userID, "", nil)
    // Access Token: 15 分鐘
    // Refresh Token: 30 天，存在 DB 中（可撤銷）
    refreshToken := generateRandomString(64)

    // 將 refreshToken hash 後存入 DB
    storeRefreshToken(userID, hashToken(refreshToken))

    return &TokenPair{
        AccessToken:  accessToken,
        RefreshToken: refreshToken,
        ExpiresIn:    900, // 15 minutes
    }, err
}
```

---

## 架構圖

```
           ┌─────────────────────────────────────┐
           │           SSO Service               │
           │  /login  /validate  /refresh        │
           │         Redis Session Store         │
           └──────┬──────────────┬───────────────┘
                  │              │
         ┌────────┴───┐    ┌─────┴──────┐
         │  App A     │    │   App B    │
         │ (驗證 JWT) │    │ (驗證 JWT) │
         └────────────┘    └────────────┘
```

---

## Trade-offs

| 決策 | 選擇 | 理由 |
|------|------|------|
| Token 類型 | JWT | 無狀態，不需查 DB 驗證；Stateless |
| Token 存放 | HttpOnly Cookie | 防 XSS；不用 localStorage |
| Refresh Token | 存 DB（可撤銷） | 用戶登出/帳號異常可立即失效 |
| Session 儲存 | Redis | 分散式，快速 TTL 管理 |

## 相關概念

- [[sd-api-gateway-vs-lb|SD題解：API Gateway vs LB]] — SSO Token 在 Gateway 層驗證
- [[系統設計核心技術棧]] — Redis Session 存儲
