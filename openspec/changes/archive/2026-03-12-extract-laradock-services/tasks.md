## 1. Workspace 服務

- [x] 1.1 建立 `workspace/` 目錄，基於 `laradock/workspace/Dockerfile` 精簡改寫，僅保留 PHP >= 8.2 流程；同時從 laradock/workspace/ 複製必要設定檔（aliases.sh、crontab/、xdebug.ini）
- [x] 1.2 改用 `phusion/baseimage:noble-1.0.2` 為基底，透過 ondrej/php PPA 安裝 PHP CLI 及擴展，移除所有 CHANGE_SOURCE 和 PHP < 8.2 版本判斷邏輯
- [x] 1.3 將 Chromium 安裝邏輯直接寫入 Dockerfile（從 Debian bookworm 倉庫安裝），取代 CI 階段的 awk 注入
- [x] 1.4 使用 multi-stage build 安裝 NVM/Node，NODE_VERSION 可透過 build arg 指定
- [x] 1.5 內建 Composer、SSH、Xdebug、MongoDB 擴展安裝

## 2. PHP-FPM 服務

- [x] 2.1 建立 `php-fpm/` 目錄，基於 `laradock/php-fpm/Dockerfile` 精簡改寫；從 laradock/php-fpm/ 複製必要設定檔（laravel.ini、xdebug.ini、xlaravel.pool.conf、php8.x.ini）
- [x] 2.2 改用 `php:${PHP_VERSION}-fpm-alpine` 為基底，透過 `mlocati/php-extension-installer` 安裝所有擴展，移除數十個 ARG 開關和所有 PHP < 8.2 版本判斷
- [x] 2.3 移除 CHANGE_SOURCE 鏡像源切換邏輯，固定安裝擴展清單（bcmath, curl, gd, igbinary, imagick, imap, mbstring, memcached, mongodb, opcache, redis, soap, sqlite3, xmlrpc, xml, xdebug, zip）

## 3. PHP-Worker 服務

- [x] 3.1 建立 `php-worker/` 目錄，基於 `laradock/php-worker/Dockerfile` 精簡改寫；從 laradock/php-worker/ 複製必要設定檔（supervisord.conf、supervisord.d/）
- [x] 3.2 改用 `php:${PHP_VERSION}-alpine` 為基底，透過 `install-php-extensions` 安裝擴展，加裝 supervisor，移除所有 PHP < 8.2 版本判斷和 CHANGE_SOURCE 邏輯
- [x] 3.3 確認 entrypoint 為 supervisord，且 `/etc/supervisord.d/` 可供外部 volume 掛載

## 4. Laravel Echo Server 服務

- [x] 4.1 建立 `laravel-echo-server/` 目錄，基於 `laradock/laravel-echo-server/Dockerfile` 精簡改寫；從 laradock/laravel-echo-server/ 複製 package.json、laravel-echo-server.json
- [x] 4.2 移除 CHANGE_SOURCE 鏡像源切換邏輯，保持 `node:alpine` 基底

## 5. CI Workflow 更新

- [x] 5.1 修改 `.github/workflows/builder.yml` 中 workspace、php-fpm、php-worker 的 build context 從 `./laradock/<service>` 改為 `./<service>`
- [x] 5.2 移除「Copy env file」和 `docker compose config` 步驟（三個 PHP 服務的 build job）
- [x] 5.3 移除「產生建構參數」步驟中的 yq 下載和 build_args 動態提取，改為直接在 build-push-action 中明確指定 build args
- [x] 5.4 移除「增加 apt update 命令在安裝 xdebug 之前」步驟
- [x] 5.5 移除「Install Chromium-browser into workspace」步驟
- [x] 5.6 修改 laravel-echo-server 的 build context 從 `./laradock/laravel-echo-server` 改為 `./laravel-echo-server`，同樣簡化其 build args 產生流程
- [x] 5.7 保留 nginx 相關的 build job 不變（仍使用 laradock 子模組）

## 6. Compose 和設定更新

- [x] 6.1 更新 `compose.yaml` 中 php-fpm 的 volume mount 路徑（ini 檔從 `./php-fpm/php${PHP_VERSION}.ini` 掛載）
- [x] 6.2 更新 `compose.yaml` 中 php-worker 的 volume mount 路徑
- [x] 6.3 更新 `compose.yaml` 中 laravel-echo-server 的 volume mount 路徑

## 7. 驗證

- [x] 7.1 確認四個 Dockerfile 中無 `CHANGE_SOURCE`、`PHP_MAJOR_VERSION = "5"`、`PHP_MAJOR_VERSION = "7"` 等殘留
- [ ] 7.2 確認 `docker build` 能成功建構每個服務（本地測試至少一個 PHP 版本）
