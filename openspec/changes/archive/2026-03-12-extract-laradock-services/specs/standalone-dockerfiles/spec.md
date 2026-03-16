## ADDED Requirements

### Requirement: Workspace Dockerfile 獨立於根目錄
系統 SHALL 在 `workspace/` 目錄下提供一個獨立的 Dockerfile，以 `phusion/baseimage:noble-1.0.2` 為基底，透過 ondrej/php PPA 安裝 PHP CLI 及擴展，且 MUST 不包含任何 PHP < 8.2 的版本判斷邏輯。

#### Scenario: 建構 workspace 映像
- **WHEN** 使用 `docker build --build-arg PHP_VERSION=8.4 workspace/` 建構
- **THEN** 映像 MUST 包含 PHP 8.4 CLI、Composer、NVM/Node、Chromium、SSH 服務、Xdebug、MongoDB 擴展

#### Scenario: PHP 版本矩陣
- **WHEN** 使用 PHP_VERSION 為 8.1、8.3、8.4 任一值建構
- **THEN** 建構 MUST 成功完成且所有擴展正確安裝

### Requirement: PHP-FPM Dockerfile 獨立於根目錄
系統 SHALL 在 `php-fpm/` 目錄下提供一個獨立的 Dockerfile，以 `php:${PHP_VERSION}-fpm-alpine` 為基底，使用 `mlocati/php-extension-installer` 安裝擴展。

#### Scenario: 建構 php-fpm 映像
- **WHEN** 使用 `docker build --build-arg PHP_VERSION=8.4 php-fpm/` 建構
- **THEN** 映像 MUST 包含以下擴展：bcmath, curl, gd, igbinary, imagick, imap, mbstring, memcached, mongodb, opcache, redis, soap, sqlite3, xmlrpc, xml, xdebug, zip

#### Scenario: FPM 服務正常啟動
- **WHEN** 容器啟動
- **THEN** php-fpm MUST 監聽 port 9000

### Requirement: PHP-Worker Dockerfile 獨立於根目錄
系統 SHALL 在 `php-worker/` 目錄下提供一個獨立的 Dockerfile，以 `php:${PHP_VERSION}-alpine` 為基底，包含 supervisor 及必要 PHP 擴展。

#### Scenario: 建構 php-worker 映像
- **WHEN** 使用 `docker build --build-arg PHP_VERSION=8.4 php-worker/` 建構
- **THEN** 映像 MUST 包含 supervisor 及共同擴展清單，且 entrypoint 為 supervisord

#### Scenario: Supervisor 設定掛載
- **WHEN** 使用者掛載 supervisord.d 目錄
- **THEN** supervisor MUST 載入 `/etc/supervisord.d/` 下的設定檔

### Requirement: Laravel Echo Server Dockerfile 獨立於根目錄
系統 SHALL 在 `laravel-echo-server/` 目錄下提供一個獨立的 Dockerfile，以 `node:alpine` 為基底，移除中國鏡像源切換邏輯。

#### Scenario: 建構 laravel-echo-server 映像
- **WHEN** 使用 `docker build laravel-echo-server/` 建構
- **THEN** 映像 MUST 包含 laravel-echo-server 且監聽 port 3000

### Requirement: 設定檔隨 Dockerfile 一同管理
每個服務目錄 SHALL 包含建構所需的所有設定檔（如 xdebug.ini、laravel.ini、supervisord.conf 等），MUST 不再依賴 laradock 子模組中的設定檔。

#### Scenario: workspace 設定檔完整性
- **WHEN** 檢查 `workspace/` 目錄
- **THEN** MUST 包含：Dockerfile、aliases.sh、crontab/、xdebug.ini

#### Scenario: php-fpm 設定檔完整性
- **WHEN** 檢查 `php-fpm/` 目錄
- **THEN** MUST 包含：Dockerfile、laravel.ini、xdebug.ini、xlaravel.pool.conf、以及各 PHP 版本的 ini 檔

#### Scenario: php-worker 設定檔完整性
- **WHEN** 檢查 `php-worker/` 目錄
- **THEN** MUST 包含：Dockerfile、supervisord.conf、supervisord.d/

#### Scenario: laravel-echo-server 設定檔完整性
- **WHEN** 檢查 `laravel-echo-server/` 目錄
- **THEN** MUST 包含：Dockerfile、package.json、laravel-echo-server.json

### Requirement: 移除所有 PHP < 8.2 相容程式碼
所有 Dockerfile MUST NOT 包含 `PHP_MAJOR_VERSION = "5"`、`PHP_MAJOR_VERSION = "7"`、`PHP_VERSION_ID = "50600"` 等針對 PHP 8.2 以下版本的條件判斷。

#### Scenario: 掃描版本判斷程式碼
- **WHEN** 以 grep 搜尋四個 Dockerfile 中的 PHP 版本判斷
- **THEN** MUST 不存在任何 PHP 5.x 或 7.x 的條件分支

### Requirement: 移除中國鏡像源切換邏輯
所有 Dockerfile MUST NOT 包含 `CHANGE_SOURCE` 相關的鏡像源切換邏輯。

#### Scenario: 掃描 CHANGE_SOURCE
- **WHEN** 以 grep 搜尋四個 Dockerfile 中的 CHANGE_SOURCE
- **THEN** MUST 無任何匹配結果
