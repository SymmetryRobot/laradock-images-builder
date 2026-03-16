## Context

本專案用 GitHub Actions 每月自動建構 laradock 衍生的 Docker 映像（workspace、php-fpm、php-worker、laravel-echo-server、nginx），推送至 Docker Hub。目前直接使用 laradock Git 子模組中的 Dockerfile，並在 CI 階段透過 `docker compose config` 產生 build args、用 awk/sed 注入 Chromium 安裝和 xdebug apt-update 等修改。

laradock 原始 Dockerfile 為了相容 PHP 5.6~8.4 包含大量版本分支邏輯（`if PHP_MAJOR_VERSION = "5"` 等），加上數十個 `ARG INSTALL_XXX=false` 的可選套件區塊，造成：
- 建構時間長（即使大部分 ARG 為 false，Docker 仍需評估每個 RUN 層）
- CI workflow 複雜（需要動態修改 Dockerfile）
- 除錯困難（laradock 更新可能影響建構）

精簡工作將直接基於 laradock 子模組中的原始 Dockerfile 進行改寫。

## Goals / Non-Goals

**Goals:**
- 在根目錄建立四個獨立服務目錄，各含自維護的 Dockerfile 及設定檔
- 移除所有 PHP < 8.2 的版本判斷程式碼
- 簡化 CI workflow，消除 awk/sed 動態修改 Dockerfile 的步驟
- 保持映像對外介面不變（相同的 port、entrypoint、volume 掛載點）

**Non-Goals:**
- 不移除 laradock 子模組（nginx 和其他服務仍需要）
- 不重新設計 CI 的 multi-platform build + digest merge 架構
- 不更改 Docker Hub 的 image namespace/tag 命名規則
- 不處理 nginx 服務的抽取

## Decisions

### 1. 基底映像選擇

| 服務 | 基底映像 | 理由 |
|------|---------|------|
| workspace | `phusion/baseimage:noble-1.0.2` | 需要 init 系統管理多個服務（SSH、cron），Ubuntu 基底可用 ondrej/php PPA |
| php-fpm | `php:${PHP_VERSION}-fpm-alpine` | 官方 PHP 映像，Alpine 精簡，搭配 `install-php-extensions` 簡化擴展安裝 |
| php-worker | `php:${PHP_VERSION}-alpine` | 同上，加裝 supervisor 管理 worker 程序 |
| laravel-echo-server | `node:alpine` | 純 Node.js 服務，無 PHP 依賴 |

**替代方案考量**：workspace 曾考慮也用 Alpine，但 phusion/baseimage 內建 runit 服務管理和 SSH 支援更適合開發用途。

### 2. PHP 擴展安裝策略

- php-fpm / php-worker：使用 `mlocati/php-extension-installer`（COPY --from 取得），一行指令安裝所有擴展，自動處理依賴
- workspace：使用 ondrej/php PPA 的 `php${PHP_VERSION}-xxx` APT 套件，因為 Ubuntu 基底不適用 `docker-php-ext-install`

**替代方案**：直接用 pecl install 逐一安裝 — 太冗長且需要手動管理依賴。

### 3. 擴展清單

根據 `.env.self` 和 backup/ 中的配置，固定安裝以下擴展（移除 ARG 開關）：

**共同擴展**：bcmath, curl, gd, igbinary, imagick, imap, mbstring, memcached, opcache, redis, soap, sqlite3, xmlrpc, xml, zip

**額外擴展**：
- mongodb：所有服務都需要，PHP 8.1 需指定版本 `mongodb-1.21.5`
- xdebug：workspace 和 php-fpm 需要，PHP 8.1 需指定版本
- supervisor 相關系統套件：僅 php-worker

### 4. Chromium 安裝（workspace）

保留目前從 Debian bookworm 倉庫安裝 Chromium 的策略，直接寫入 Dockerfile 而非 CI 階段注入。這解決了 Ubuntu Noble 上 Chromium 為 snap 包無法在容器中使用的問題。

### 5. NVM/Node 安裝（workspace）

使用 multi-stage build：第一階段在 Ubuntu 中安裝 NVM 和 Node，第二階段 COPY 過來。避免在最終映像中留下安裝工具鏈的殘留。

### 6. CI Workflow 變更策略

- build context 從 `./laradock/<service>` 改為 `./<service>`
- 移除「增加 apt update 命令在安裝 xdebug 之前」步驟（已直接在 Dockerfile 處理）
- 移除「Install Chromium-browser into workspace」步驟（已直接在 Dockerfile 處理）
- 移除「Copy env file / docker compose config」步驟（不再依賴 laradock 的 docker-compose 產生 build args）
- build args 改為直接在 workflow 中明確列出，而非從 laradock compose config 動態提取

## Risks / Trade-offs

- **[風險] 擴展版本差異** → 使用 `install-php-extensions` 自動選取最新相容版本，僅對已知有問題的組合（如 PHP 8.1 + mongodb）硬編碼版本
- **[風險] laradock 更新不同步** → 由於四個服務不再依賴 laradock Dockerfile，laradock 的安全修補需要手動套用到獨立 Dockerfile。可接受，因為原本也不會頻繁更新子模組
- **[取捨] 固定擴展清單** → 移除 ARG 開關意味著所有擴展都會安裝，映像體積可能略增。但簡化了配置管理，且團隊實際上總是啟用這些擴展
- **[取捨] 不再支援 PHP 8.0 及以下** → `.env.self` 和 workflow 矩陣已只建構 8.1, 8.3, 8.4，實際無影響
