## Context

目前 workspace、php-fpm、php-worker、laravel-echo-server 四個服務已獨立至專案根目錄，各自擁有簡化的 Dockerfile 及設定檔。唯獨 nginx 仍依賴 laradock submodule — CI workflow 需要 checkout submodule、複製 .env、執行 `docker compose config` 與 yq 來動態產生 build args，然後以 `./laradock/nginx` 作為 build context。

laradock 原始 nginx Dockerfile 包含 `CHANGE_SOURCE`（中國鏡像源切換）邏輯，其餘部分相對簡潔。主要 build args 為 `PHP_UPSTREAM_CONTAINER` 和 `PHP_UPSTREAM_PORT`，可直接硬編碼預設值。

## Goals / Non-Goals

**Goals:**
- 將 nginx Dockerfile 及所有設定檔複製至 `nginx/` 目錄，移除 `CHANGE_SOURCE` 邏輯
- 簡化 CI workflow 的 `build-nginx-image` job，移除 submodule checkout、env 複製、docker compose config、yq 安裝等步驟
- 使 nginx build 與其他服務保持一致的建構模式

**Non-Goals:**
- 不修改 nginx.conf、sites、logrotate 等設定檔的內容（僅搬移）
- 不變更 nginx 映像的 tag 命名規則
- 不處理 laradock submodule 的移除（留待後續）

## Decisions

### 1. 直接使用預設 build args 而非動態提取

**選擇**: 在 Dockerfile 中保留 `PHP_UPSTREAM_CONTAINER=php-fpm` 和 `PHP_UPSTREAM_PORT=9000` 作為預設值，CI 不再傳入這些參數。

**理由**: 這些值在所有部署環境中都相同，不需要動態從 laradock .env 提取。使用者如需客製化，可在 `docker build --build-arg` 或 compose.yaml 中覆寫。

### 2. 保留 startup.sh 自動產生 SSL 憑證的功能

**選擇**: 原封不動保留 startup.sh 的 SSL 自簽憑證產生邏輯。

**理由**: 這是開發環境便利功能，無需簡化。

### 3. 搬移所有 sites/*.example 檔案

**選擇**: 將 sites/ 目錄下的所有 .conf 和 .example 檔案一併搬移。

**理由**: 使用者可能需要這些範例設定檔作為參考。

## Risks / Trade-offs

- **[風險] laradock submodule 殘留** → 本次不移除 submodule，因為可能有其他用途。可在確認完全不需要後再移除。
- **[風險] SSL 目錄為空** → sites/ 和 ssl/ 目錄內的 .gitignore 需正確處理，確保目錄結構在 git clone 後存在。
