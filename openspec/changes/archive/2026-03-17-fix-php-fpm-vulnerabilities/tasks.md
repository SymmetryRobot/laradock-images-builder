## 1. 系統套件安全升級

- [x] 1.1 在 php-fpm/Dockerfile 中，於所有 `install-php-extensions` 和 PECL 擴充安裝完成後，新增 `RUN apt-get update && apt-get upgrade -y` 步驟

## 2. 移除建置工具

- [x] 2.1 在安全升級步驟之後，新增 `apt-get purge` 步驟移除 binutils、binutils-common、binutils-x86-64-linux-gnu、cpp、gcc 及相關建置套件
- [x] 2.2 執行 `apt-get autoremove -y` 清除被移除套件的殘餘依賴

## 3. ImageMagick 安全限制

- [x] 3.1 在 Dockerfile 中新增 RUN 步驟，修改 ImageMagick 的 policy.xml，禁用 URL、HTTPS、MVG、MSL、TEXT、LABEL 等高風險 coder/delegate
- [x] 3.2 在 policy.xml 中設定資源限制（memory、disk、map 等）

## 4. 最終清理

- [x] 4.1 在 Dockerfile 最後階段新增清理步驟：`apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*`
- [x] 4.2 確保清理步驟與升級/移除步驟合併在同一個 RUN 層，減少映像層數

## 5. 驗證

- [x] 5.1 確認 Dockerfile 語法正確，所有 PHP 擴充安裝指令未被影響
- [x] 5.2 確認建置工具移除步驟在所有 PECL/擴充編譯完成之後
