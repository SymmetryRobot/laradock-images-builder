## Why

目前專案依賴 laradock Git 子模組中的原始 Dockerfile 來建構 workspace、php-fpm、php-worker、laravel-echo-server 映像。這些 Dockerfile 包含大量 PHP 5.x / 7.x 的歷史相容程式碼、中國鏡像源切換邏輯、以及數十個以 ARG 開關控制的可選套件安裝區塊，導致建構時間長、維護困難，且 CI workflow 需要在 build 階段用 awk/sed 動態注入修改。將四個服務的 Dockerfile 及相關設定檔抽取至專案根目錄獨立管理，僅保留 PHP >= 8.2 的流程，可大幅簡化建構邏輯並脫離對 laradock 子模組 Dockerfile 的依賴。

## What Changes

- 在專案根目錄建立 `workspace/`、`php-fpm/`、`php-worker/`、`laravel-echo-server/` 四個資料夾，各含精簡後的 Dockerfile 及必要設定檔
- workspace Dockerfile：以 `phusion/baseimage:noble-1.0.2` 為基底，使用 ondrej/php PPA 安裝 PHP，multi-stage build 安裝 NVM/Node，內建 Chromium、Composer、SSH、Xdebug、MongoDB 擴展
- php-fpm Dockerfile：以 `php:${PHP_VERSION}-fpm-alpine` 為基底，透過 `mlocati/php-extension-installer` 安裝所有擴展，去除所有 PHP < 8.2 的版本判斷
- php-worker Dockerfile：以 `php:${PHP_VERSION}-alpine` 為基底，同樣使用 `install-php-extensions` 並安裝 supervisor
- laravel-echo-server Dockerfile：以 `node:alpine` 為基底，移除中國鏡像源切換邏輯
- 更新 `.github/workflows/builder.yml` 將 build context 從 `./laradock/<service>` 改為 `./<service>`，移除 CI 中的 awk/sed Dockerfile 動態修改步驟
- 更新 `compose.yaml` 的 volume mount 路徑（如 php-fpm ini 檔、supervisord 設定檔）
- **BREAKING**：不再支援 PHP 8.0 及以下版本的建構

## Capabilities

### New Capabilities
- `standalone-dockerfiles`: 從 laradock 子模組中獨立出 workspace、php-fpm、php-worker、laravel-echo-server 四個服務的精簡 Dockerfile 及設定檔，僅保留 PHP >= 8.2 的建構流程
- `simplified-ci-workflow`: 簡化 CI workflow，直接使用根目錄的 Dockerfile，移除 build 階段的動態 Dockerfile 修改邏輯

### Modified Capabilities
<!-- 無既有 specs 需修改 -->

## Impact

- **程式碼**：新增 4 個服務目錄及其 Dockerfile/設定檔，修改 `.github/workflows/builder.yml` 和 `compose.yaml`
- **依賴**：仍依賴 laradock 子模組（用於 nginx 等其他服務），但四個抽出服務不再依賴其 Dockerfile
- **建構**：PHP 版本矩陣改為 8.2+ (目前已經是 8.1, 8.3, 8.4)，建構速度預期提升（更少的條件分支和無用安裝步驟）
- **系統**：Docker Hub 映像 tag 規則不變，但映像內容可能因精簡而有差異
