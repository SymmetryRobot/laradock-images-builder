## ADDED Requirements

### Requirement: Nginx Dockerfile 獨立於根目錄
系統 SHALL 在 `nginx/` 目錄下提供一個獨立的 Dockerfile，以 `nginx:alpine` 為基底，包含 logrotate、自簽 SSL 憑證產生、PHP upstream 設定，且 MUST NOT 包含 `CHANGE_SOURCE` 鏡像源切換邏輯。

#### Scenario: 建構 nginx 映像
- **WHEN** 使用 `docker build nginx/` 建構
- **THEN** 映像 MUST 包含 nginx、logrotate、openssl、bash，並設定 php-upstream 為 `php-fpm:9000`

#### Scenario: 自訂 PHP upstream
- **WHEN** 使用 `docker build --build-arg PHP_UPSTREAM_CONTAINER=custom-fpm --build-arg PHP_UPSTREAM_PORT=9001 nginx/` 建構
- **THEN** upstream.conf MUST 設定為 `custom-fpm:9001`

### Requirement: Nginx 設定檔隨 Dockerfile 一同管理
`nginx/` 目錄 SHALL 包含建構所需的所有設定檔，MUST 不再依賴 laradock 子模組中的設定檔。

#### Scenario: nginx 設定檔完整性
- **WHEN** 檢查 `nginx/` 目錄
- **THEN** MUST 包含：Dockerfile、nginx.conf、startup.sh、logrotate/nginx、sites/default.conf、sites/*.example

### Requirement: Nginx Dockerfile 移除中國鏡像源切換邏輯
`nginx/Dockerfile` MUST NOT 包含 `CHANGE_SOURCE` 相關的鏡像源切換邏輯。

#### Scenario: 掃描 CHANGE_SOURCE
- **WHEN** 以 grep 搜尋 `nginx/Dockerfile` 中的 CHANGE_SOURCE
- **THEN** MUST 無任何匹配結果
