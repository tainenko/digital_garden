---
title: Go 微服務配置管理
type: concept
tags: [golang, configuration, viper, kubernetes, secrets, microservices]
created: 2026-04-29
updated: 2026-04-29
---

# Go 微服務配置管理

## 配置的來源層次（優先級從高到低）

```
環境變數（最高）
    ↓ 覆蓋
命令列參數（--port=8080）
    ↓ 覆蓋
本地設定檔（config.yaml）
    ↓ 覆蓋
預設值（最低）
```

## 12-Factor App 的配置原則

- **不要把配置放進代碼**（不 hardcode，不 commit secret）
- **環境變數是標準介面**（不同環境用不同值）
- **開發/測試/生產的區別只在配置**，代碼相同

## 結構化配置（推薦做法）

定義 Config struct，代碼中只引用 struct，不直接呼叫 os.Getenv：

```go
// config/config.go
type Config struct {
    Server   ServerConfig
    Database DatabaseConfig
    Redis    RedisConfig
    Auth     AuthConfig
}

type ServerConfig struct {
    Port            string        `mapstructure:"port" default:"8080"`
    ReadTimeout     time.Duration `mapstructure:"read_timeout" default:"30s"`
    WriteTimeout    time.Duration `mapstructure:"write_timeout" default:"30s"`
    ShutdownTimeout time.Duration `mapstructure:"shutdown_timeout" default:"25s"`
}

type DatabaseConfig struct {
    DSN             string        `mapstructure:"dsn"`
    MaxOpenConns    int           `mapstructure:"max_open_conns" default:"25"`
    MaxIdleConns    int           `mapstructure:"max_idle_conns" default:"10"`
    ConnMaxLifetime time.Duration `mapstructure:"conn_max_lifetime" default:"5m"`
}

type RedisConfig struct {
    Addr     string `mapstructure:"addr"`
    Password string `mapstructure:"password"` // 從 Secret 注入
    DB       int    `mapstructure:"db" default:"0"`
}

type AuthConfig struct {
    JWTSecret     string        `mapstructure:"jwt_secret"` // 從 Secret 注入
    TokenDuration time.Duration `mapstructure:"token_duration" default:"24h"`
}
```

## Viper 整合

```go
// config/loader.go
import "github.com/spf13/viper"

func Load() (*Config, error) {
    v := viper.New()

    // 1. 設定檔（開發環境）
    v.SetConfigName("config")        // config.yaml / config.json
    v.SetConfigType("yaml")
    v.AddConfigPath("./config")      // 相對路徑
    v.AddConfigPath("$HOME/.order-service")

    if err := v.ReadInConfig(); err != nil {
        if _, ok := err.(viper.ConfigFileNotFoundError); !ok {
            return nil, fmt.Errorf("read config: %w", err)
        }
        // 找不到設定檔是允許的（生產環境純靠環境變數）
    }

    // 2. 環境變數（自動覆蓋，生產環境）
    v.SetEnvPrefix("ORDER")         // ORDER_SERVER_PORT, ORDER_DATABASE_DSN
    v.AutomaticEnv()
    v.SetEnvKeyReplacer(strings.NewReplacer(".", "_", "-", "_"))

    // 3. 預設值
    v.SetDefault("server.port", "8080")
    v.SetDefault("database.max_open_conns", 25)
    v.SetDefault("auth.token_duration", "24h")

    var cfg Config
    if err := v.Unmarshal(&cfg); err != nil {
        return nil, fmt.Errorf("unmarshal config: %w", err)
    }

    // 4. 驗證必要配置
    if err := validate(cfg); err != nil {
        return nil, fmt.Errorf("invalid config: %w", err)
    }

    return &cfg, nil
}

func validate(cfg Config) error {
    if cfg.Database.DSN == "" {
        return errors.New("DATABASE_DSN is required")
    }
    if cfg.Auth.JWTSecret == "" {
        return errors.New("AUTH_JWT_SECRET is required")
    }
    if len(cfg.Auth.JWTSecret) < 32 {
        return errors.New("AUTH_JWT_SECRET must be at least 32 characters")
    }
    return nil
}
```

## config.yaml（開發環境）

```yaml
# config/config.yaml（進 .gitignore！）
server:
  port: "8080"
  read_timeout: "30s"
  write_timeout: "30s"

database:
  dsn: "postgres://user:pass@localhost:5432/orders_dev?sslmode=disable"
  max_open_conns: 10
  max_idle_conns: 5

redis:
  addr: "localhost:6379"
  password: ""

auth:
  jwt_secret: "dev-secret-change-in-production-minimum-32-chars"
  token_duration: "24h"
```

```yaml
# config/config.yaml.example（進版控，敏感值用佔位符）
server:
  port: "8080"
database:
  dsn: "postgres://USER:PASSWORD@HOST:5432/DB?sslmode=require"
redis:
  addr: "REDIS_HOST:6379"
auth:
  jwt_secret: "REPLACE_WITH_STRONG_SECRET"
```

## Kubernetes Secret 管理

生產環境的 Secret 不能放在代碼或 ConfigMap，要用 K8s Secret：

```yaml
# k8s/secret.yaml（不要進版控！用 sealed-secrets 或 Vault）
apiVersion: v1
kind: Secret
metadata:
  name: order-service-secret
  namespace: production
type: Opaque
stringData:
  DATABASE_DSN: "postgres://user:strongpass@rds.amazonaws.com:5432/orders_prod"
  AUTH_JWT_SECRET: "production-secret-must-be-32-chars-minimum"
  REDIS_PASSWORD: "production-redis-password"
---
# k8s/deployment.yaml
spec:
  containers:
    - name: order-service
      image: order-service:v1.2.0
      # 方法一：整個 Secret 作為環境變數
      envFrom:
        - secretRef:
            name: order-service-secret
      # 方法二：選擇特定的 key
      env:
        - name: ORDER_DATABASE_DSN
          valueFrom:
            secretKeyRef:
              name: order-service-secret
              key: DATABASE_DSN
```

## Sealed Secrets（Secret 的版控方案）

```bash
# 安裝 kubeseal
brew install kubeseal

# 加密 Secret（可以進版控）
kubeseal --format yaml < k8s/secret.yaml > k8s/sealed-secret.yaml
# sealed-secret.yaml 可以安全地 commit 到 git

# K8s 集群中的 Sealed Secrets Controller 會自動解密
```

## Vault（企業級 Secret 管理）

```go
import "github.com/hashicorp/vault/api"

func loadSecretsFromVault(vaultAddr, token string) (map[string]string, error) {
    client, err := api.NewClient(&api.Config{Address: vaultAddr})
    if err != nil {
        return nil, err
    }
    client.SetToken(token)

    // 從 Vault KV v2 讀取
    secret, err := client.KVv2("secret").Get(context.Background(), "order-service/production")
    if err != nil {
        return nil, fmt.Errorf("vault read: %w", err)
    }

    result := make(map[string]string)
    for k, v := range secret.Data {
        result[k] = fmt.Sprint(v)
    }
    return result, nil
}
```

## 配置熱更新（Hot Reload）

```go
// 使用 fsnotify 監聽設定檔變化
v.WatchConfig()
v.OnConfigChange(func(e fsnotify.Event) {
    log.Infof("config file changed: %s", e.Name)

    var newCfg Config
    if err := v.Unmarshal(&newCfg); err != nil {
        log.Errorf("failed to reload config: %v", err)
        return
    }

    // 更新可動態調整的配置（注意並發安全）
    configMu.Lock()
    currentCfg = newCfg
    configMu.Unlock()

    log.Info("config reloaded successfully")
})
```

**注意**：只有無狀態的配置（log level、timeout）適合熱更新。DB 連線字串更新需要重啟。

## 環境分層

```
environments/
├── development/
│   └── config.yaml    ← 本地開發（不進版控）
├── staging/
│   └── config.yaml    ← 給 CI/CD 用的 staging 配置
└── production/
    └── config.yaml    ← 僅非敏感配置，敏感值用 Secret
```

## 安全清單

```
□ 不 commit .env 或 config.yaml（含密碼）到 git
□ .gitignore 包含 *.env, config.local.yaml, secrets/
□ 生產環境 Secret 用 K8s Secret / Vault
□ JWT secret 至少 32 字元，隨機生成
□ 資料庫密碼不共用（不同環境不同密碼）
□ 啟動時驗證必要配置（validate 函數）
□ 敏感值不在日誌中出現
□ CI/CD 的 Secret 用 GitHub Secrets / GitLab CI Variables
```

## 相關頁面

- [[Go依賴注入與Wire]] — 配置透過 DI 注入各元件
- [[Go優雅關機與健康檢查]] — 配置載入失敗時 fail-fast
- [[微服務架構設計原則]] — 12-Factor App 第 3 點（Config）
