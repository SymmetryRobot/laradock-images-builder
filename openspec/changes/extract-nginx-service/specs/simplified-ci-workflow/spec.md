## ADDED Requirements

### Requirement: Nginx CI 使用根目錄 Dockerfile 建構
GitHub Actions workflow 的 `build-nginx-image` job MUST 將 build context 設為 `./nginx`，直接使用專案根目錄下的獨立 Dockerfile。

#### Scenario: nginx 建構路徑
- **WHEN** CI 觸發 nginx 建構
- **THEN** docker build context MUST 為 `./nginx`

### Requirement: Nginx CI 移除動態 build args 提取
`build-nginx-image` job MUST NOT 包含 checkout submodule、複製 .env、執行 `docker compose config`、安裝 yq 等動態提取 build args 的步驟。

#### Scenario: 無 submodule 與 yq 依賴
- **WHEN** 檢查 `build-nginx-image` job 的 steps
- **THEN** MUST 不存在 `submodules: true`、`docker compose config`、`yq` 相關步驟

#### Scenario: Build args 簡化
- **WHEN** 查看 nginx build-push-action 的 build-args
- **THEN** MUST 僅包含 `http_proxy=` 和 `https_proxy=`（PHP upstream 使用 Dockerfile 預設值）
