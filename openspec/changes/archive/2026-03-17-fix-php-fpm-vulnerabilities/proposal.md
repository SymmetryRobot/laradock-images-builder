## Why

php-fpm image 的 Trivy 漏洞掃描報告顯示大量 HIGH 和 CRITICAL 級別的 CVE，涵蓋 ImageMagick（6 個 CRITICAL）、libc6（4 個 CRITICAL）、libexpat1、libopenexr、libheif 等核心套件，以及 binutils、curl、libxml2 等開發/執行時期依賴。這些漏洞對生產環境的安全構成風險，需要盡快修補。

## What Changes

- 在 Dockerfile 中加入 `apt-get upgrade` 步驟，確保基礎映像的系統套件取得最新安全修補
- 在 PECL 擴充編譯完成後，移除不再需要的建置工具（binutils 及相關套件），大幅削減攻擊面
- 在映像建置最終階段執行清理，移除暫存檔案和套件快取
- 利用 ImageMagick 的 policy.xml 限制高風險格式的處理，降低 ImageMagick CVE 的可利用性

## Capabilities

### New Capabilities
- `php-fpm-security-hardening`: 涵蓋 php-fpm Dockerfile 的安全強化措施，包含系統套件升級、建置工具清理、ImageMagick policy 限制

### Modified Capabilities

## Impact

- **php-fpm/Dockerfile**: 主要修改對象，新增安全更新和清理步驟
- **映像大小**: 移除 binutils 等建置工具後，映像體積可能略微減小
- **建置時間**: 新增 `apt-get upgrade` 步驟會略微增加建置時間
- **依賴**: 不影響 PHP 擴充功能的正常運作（擴充已在移除建置工具前完成編譯）
- **CI/CD**: 現有 GitHub Actions workflow 無需修改
