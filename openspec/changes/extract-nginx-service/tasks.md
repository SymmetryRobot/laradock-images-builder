## 1. 建立 nginx 獨立目錄與 Dockerfile

- [x] 1.1 建立 `nginx/Dockerfile`：從 laradock/nginx/Dockerfile 簡化，移除 `CHANGE_SOURCE` 邏輯，保留 logrotate、www-data 使用者、PHP upstream 設定、startup.sh
- [x] 1.2 複製 `nginx/nginx.conf` 從 laradock/nginx/nginx.conf
- [x] 1.3 複製 `nginx/startup.sh` 從 laradock/nginx/startup.sh
- [x] 1.4 複製 `nginx/logrotate/nginx` 從 laradock/nginx/logrotate/nginx
- [x] 1.5 複製 `nginx/sites/` 目錄（default.conf 及所有 *.example）從 laradock/nginx/sites/
- [x] 1.6 建立 `nginx/ssl/.gitignore` 確保 ssl 目錄存在

## 2. 更新 CI Workflow

- [x] 2.1 修改 `build-nginx-image` job：移除 `submodules: true`、「Copy env file for Nginx」、「產生 Nginx 建構參數」步驟
- [x] 2.2 將 build context 從 `./laradock/nginx` 改為 `./nginx`
- [x] 2.3 簡化 build-args：移除 `${{ steps.build_args.outputs.args }}`，僅保留 `http_proxy=` 和 `https_proxy=`

## 3. 驗證

- [x] 3.1 確認 `nginx/Dockerfile` 不包含 CHANGE_SOURCE
- [x] 3.2 本地建構 nginx 映像驗證成功
