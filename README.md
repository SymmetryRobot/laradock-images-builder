# ThanatosDilaradock Images Builder

自動化建置並發布客製化 Laradock Docker 映像檔的 CI/CD 系統，支援多架構（AMD64 / ARM64）與多 PHP 版本。

## 建置的映像檔

| 映像檔 | 基底 | 說明 | PHP 版本 |
|--------|------|------|----------|
| **laradock-workspace** | Ubuntu 24.04 | 完整開發環境，包含 PHP CLI、Node.js、Composer、Chromium、SSH 等 | 8.1, 8.3, 8.4 |
| **laradock-php-fpm** | php-fpm (Debian) | PHP-FPM 伺服器，包含完整擴充套件 | 8.1, 8.3, 8.4 |
| **laradock-php-worker** | php-cli (Alpine) | 透過 Supervisor 管理的背景佇列工作者 | 8.1, 8.3, 8.4 |
| **laradock-nginx** | nginx (Alpine) | Nginx 網頁伺服器，含 SSL/TLS 與 logrotate | - |
| **laradock-laravel-echo-server** | Node.js | WebSocket 廣播伺服器 | - |

## 專案結構

```
├── .github/workflows/
│   └── builder.yml              # GitHub Actions CI/CD 工作流程
├── workspace/                   # Workspace 映像檔
│   ├── Dockerfile
│   ├── aliases.sh               # Shell 別名
│   ├── crontab                  # 排程任務
│   └── xdebug.ini
├── php-fpm/                     # PHP-FPM 映像檔
│   ├── Dockerfile
│   ├── laravel.ini              # Laravel PHP 設定
│   ├── xlaravel.pool.conf       # FPM pool 設定
│   └── php8.{1,3,4}.ini
├── php-worker/                  # PHP Worker 映像檔
│   ├── Dockerfile
│   ├── supervisord.conf         # Supervisor 設定
│   └── supervisord.d/
├── nginx/                       # Nginx 映像檔
│   ├── Dockerfile
│   ├── nginx.conf
│   ├── sites/                   # 站台設定
│   ├── ssl/                     # SSL 憑證
│   └── startup.sh
├── laravel-echo-server/         # Echo Server 映像檔
│   ├── Dockerfile
│   └── laravel-echo-server.json
└── .env.self                    # 建置環境變數
```

## CI/CD 工作流程

透過 GitHub Actions 自動建置，觸發條件：

- **排程**：每月 1 日 23:00 (UTC) 自動執行
- **手動**：可透過 `workflow_dispatch` 手動觸發

### 建置流程

1. **Build 階段**：針對每組 (PHP 版本 × 服務 × 架構) 組合平行建置
   - 支援架構：`linux/amd64`、`linux/arm64`
   - 使用 GitHub Actions cache 加速建置
2. **Merge 階段**：合併各架構的 digest，建立多架構 manifest 並推送至 Docker Hub

### 映像檔標籤

- PHP 相關服務：`{php-version}`、`{php-version}-latest`、`{php-version}-{date}`
- Nginx：`latest`、`{date}`

## 主要建置參數

| 參數 | 預設值 | 說明 |
|------|--------|------|
| `PHP_VERSION` | 8.4 | PHP 版本 |
| `NODE_VERSION` | 24.14.0 | Node.js 版本 |
| `PUID` / `PGID` | 1000 | 容器內使用者/群組 ID |
| `TZ` | Asia/Taipei | 時區 |
| `XDEBUG_PORT` | 9003 | Xdebug 除錯埠 |

## 特色功能

- **多架構支援**：同時建置 AMD64 與 ARM64（Apple Silicon 相容）
- **多 PHP 版本**：同時支援 PHP 8.1、8.3、8.4
- **MongoDB 版本鎖定**：PHP 8.1 相容性固定使用 mongodb 擴充 v1.21.5
- **Xdebug 支援**：PHP 8.1 使用 3.4.1、其餘使用最新版
- **Chromium 整合**：Workspace 映像檔內建 Chromium 瀏覽器
- **SSL/TLS**：Nginx 支援 TLS 1.2/1.3
- **Logrotate**：Nginx 自動日誌輪替

## 授權條款

[MIT License](LICENSE) - Copyright (c) 2026 古丁丁
