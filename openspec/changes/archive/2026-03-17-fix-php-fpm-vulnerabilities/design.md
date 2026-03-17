## Context

php-fpm image 基於 `php:${PHP_VERSION}-fpm`（Debian bookworm），透過 `install-php-extensions` 安裝 PHP 擴充時會引入 binutils、gcc 等建置工具鏈。這些工具在編譯完成後仍殘留在最終映像中，帶來大量 binutils 相關 CVE。此外，基礎映像中的系統套件（libc6、libexpat1、curl、libxml2 等）未經升級，累積了多個 CRITICAL/HIGH 漏洞。ImageMagick 作為 imagick 擴充的依賴，本身也有多個 CRITICAL CVE。

目前 Dockerfile 結構：基礎映像 → install-php-extensions → 使用者設定 → 完成。沒有任何安全升級或建置工具清理步驟。

## Goals / Non-Goals

**Goals:**
- 透過 `apt-get upgrade` 修補系統套件的已知漏洞
- 移除編譯完成後不再需要的建置工具（binutils 等），消除相關 CVE
- 透過 ImageMagick policy 限制高風險格式，降低 ImageMagick CVE 的攻擊面
- 保持所有 PHP 擴充功能正常運作
- 保持 Dockerfile 結構簡潔，不影響現有 CI/CD 流程

**Non-Goals:**
- 不更換基礎映像（如切換至 Alpine）
- 不移除 ImageMagick/imagick 擴充（業務需求）
- 不處理 linux-libc-dev 的 kernel header CVE（容器環境中不可利用，且無上游修補）
- 不處理上游標記為 disputed/won't-fix 的舊 CVE

## Decisions

### 決定 1：在擴充安裝後立即執行 apt-get upgrade

**選擇**: 在所有 `install-php-extensions` 呼叫之後、最終清理之前執行 `apt-get update && apt-get upgrade -y`

**替代方案**:
- A) 在基礎映像之後立即升級 → 但 `install-php-extensions` 會再次引入舊版依賴
- B) 使用 multi-stage build 只複製 PHP 執行時檔案 → 過於複雜，且 `install-php-extensions` 不易拆分

**理由**: 在所有套件安裝完成後統一升級，確保所有依賴都獲得最新修補，且只需一次操作。

### 決定 2：移除 binutils 等建置工具

**選擇**: 使用 `apt-get purge` 移除 binutils、cpp、gcc 等建置工具及其依賴，搭配 `apt-get autoremove`

**替代方案**:
- A) 使用 `install-php-extensions` 的 cleanup 功能 → 該工具的自動清理不夠徹底
- B) 保留建置工具 → 不可接受，binutils 本身就有 50+ 個 CVE

**理由**: PHP-FPM 在執行時不需要編譯工具。移除後可消除所有 binutils 相關 CVE，同時減少映像體積。

> **注意**：移除 binutils 後，若需在執行中的容器內臨時安裝 PECL 擴充（如 debug 用途），須先重新安裝建置工具：
> ```bash
> apt-get update && apt-get install -y binutils gcc make autoconf
> ```
> 安裝完成後即可正常使用 `pecl install` 或 `install-php-extensions`。
> 對於生產環境，建議透過重新建置映像來新增擴充，而非在容器內操作。

### 決定 3：透過 policy.xml 限制 ImageMagick

**選擇**: 設定 ImageMagick policy.xml 禁用高風險的 delegate（如 URL、ephemeral 等）和限制資源用量

**替代方案**:
- A) 完全移除 ImageMagick → 業務需要圖片處理功能
- B) 不處理 → CRITICAL CVE 不處理不合理

**理由**: policy.xml 是 ImageMagick 官方推薦的安全強化方式，可在不移除功能的前提下降低風險。

## Risks / Trade-offs

- **[Risk] 移除建置工具後無法在容器內即時編譯 PHP 擴充** → 這是預期行為，生產環境不應在執行時編譯擴充。如需新擴充應重新建置映像。
- **[Risk] apt-get upgrade 可能引入不相容的套件版本** → Debian stable 的升級通常只包含安全修補，破壞性極低。CI 中的 Trivy 掃描可驗證結果。
- **[Risk] ImageMagick policy 可能過度限制某些圖片處理功能** → 僅限制明確高風險的格式（SVG URL fetch、PS/PDF delegate），常見的 JPEG/PNG/WebP 不受影響。
- **[Risk] 部分 CVE 為上游 Debian 尚未修補** → 這些 CVE 需等待上游發布修補，`apt-get upgrade` 會在修補可用時自動套用。
