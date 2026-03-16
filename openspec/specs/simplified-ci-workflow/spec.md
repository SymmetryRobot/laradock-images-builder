## ADDED Requirements

### Requirement: CI 使用根目錄 Dockerfile 建構
GitHub Actions workflow MUST 將 build context 設為 `./<service>`（而非 `./laradock/<service>`），直接使用專案根目錄下的獨立 Dockerfile。

#### Scenario: workspace 建構路徑
- **WHEN** CI 觸發 workspace 建構
- **THEN** docker build context MUST 為 `./workspace`

#### Scenario: php-fpm 建構路徑
- **WHEN** CI 觸發 php-fpm 建構
- **THEN** docker build context MUST 為 `./php-fpm`

#### Scenario: php-worker 建構路徑
- **WHEN** CI 觸發 php-worker 建構
- **THEN** docker build context MUST 為 `./php-worker`

#### Scenario: laravel-echo-server 建構路徑
- **WHEN** CI 觸發 laravel-echo-server 建構
- **THEN** docker build context MUST 為 `./laravel-echo-server`

### Requirement: 移除 CI 中的動態 Dockerfile 修改
CI workflow MUST NOT 包含使用 awk、sed 或其他工具動態修改 Dockerfile 內容的步驟。

#### Scenario: 無 awk/sed Dockerfile 修改步驟
- **WHEN** 檢查 workflow YAML
- **THEN** MUST 不存在「增加 apt update 命令」和「Install Chromium-browser into workspace」等動態修改步驟

### Requirement: 簡化 build args 傳遞
CI workflow MUST 直接在 workflow 中指定必要的 build args（如 PHP_VERSION、XDEBUG_PORT），而非透過 `docker compose config` + yq 從 laradock 配置中動態提取。

#### Scenario: 不依賴 docker compose config
- **WHEN** 建構 workspace、php-fpm、php-worker 映像
- **THEN** workflow MUST NOT 執行 `docker compose config` 來產生 build args

#### Scenario: Build args 明確定義
- **WHEN** 查看 workflow 的 build-push-action build-args
- **THEN** MUST 直接包含 `PHP_VERSION=${{ matrix.php-version }}` 等明確值

### Requirement: 保持 multi-platform + digest merge 架構
CI workflow MUST 保持現有的 multi-platform build（amd64 + arm64）和 digest merge 流程不變。

#### Scenario: 多平台建構矩陣
- **WHEN** workflow 被觸發
- **THEN** MUST 為每個服務 + PHP 版本 + 平台組合建構獨立映像

#### Scenario: 映像合併
- **WHEN** 所有平台建構完成
- **THEN** MUST 將 digests 合併為多平台 manifest 並推送最終 tag
