## Why

Nginx 是目前唯一仍依賴 laradock git submodule 進行建構的服務。其 CI workflow 需要 checkout submodule、複製 .env、執行 `docker compose config` 與 yq 來動態產生 build args，流程複雜且脆弱。將 nginx 獨立出來可完全移除 laradock submodule 依賴，與其他已獨立的服務（workspace、php-fpm、php-worker、laravel-echo-server）保持一致。

## What Changes

- 建立 `nginx/` 目錄，包含簡化後的 Dockerfile 及所有必要設定檔（nginx.conf、startup.sh、logrotate、sites、ssl）
- 移除 Dockerfile 中的 `CHANGE_SOURCE`（中國鏡像源）邏輯
- 更新 `.github/workflows/builder.yml` 的 `build-nginx-image` job：移除 submodule checkout、env 複製、`docker compose config`、yq 安裝等步驟，改為直接使用 `./nginx` 作為 build context
- 更新 `compose.yaml` 的 nginx service build context 指向 `./nginx`

## Capabilities

### New Capabilities

（無新增 — nginx 已存在於 standalone-dockerfiles 與 simplified-ci-workflow 的適用範圍）

### Modified Capabilities

- `standalone-dockerfiles`: 新增 nginx 服務至獨立 Dockerfile 清單
- `simplified-ci-workflow`: nginx build job 改為與其他服務一致的簡化流程

## Impact

- **CI workflow**：`build-nginx-image` job 大幅簡化，移除 5 個動態步驟
- **Git submodule**：laradock submodule 可完全移除（不再有任何 job 依賴它）
- **Docker build context**：從 `./laradock/nginx` 改為 `./nginx`
- **compose.yaml**：nginx build context 需更新
