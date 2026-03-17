## ADDED Requirements

### Requirement: System packages SHALL be upgraded to latest security patches
The php-fpm Dockerfile SHALL execute `apt-get update && apt-get upgrade -y` after all PHP extensions are installed, ensuring all system packages receive the latest available security patches from Debian repositories.

#### Scenario: Security patches applied during build
- **WHEN** the Docker image is built
- **THEN** all system packages SHALL be upgraded to their latest available versions from Debian stable security repositories

#### Scenario: PHP extensions remain functional after upgrade
- **WHEN** the system packages are upgraded
- **THEN** all installed PHP extensions (bcmath, curl, gd, igbinary, imagick, imap, mbstring, memcached, opcache, redis, soap, sqlite3, xmlrpc, xml, zip, mongodb, xdebug) SHALL continue to load and function correctly

### Requirement: Build tools SHALL be removed after extension compilation
The Dockerfile SHALL remove binutils, cpp, gcc, and related build toolchain packages after all PHP extensions have been compiled and installed. This removal MUST use `apt-get purge` followed by `apt-get autoremove` to ensure complete cleanup of build dependencies.

#### Scenario: Binutils packages removed from final image
- **WHEN** the Docker image build completes
- **THEN** the packages binutils, binutils-common, binutils-x86-64-linux-gnu, libbinutils, libctf0, libctf-nobfd0, libsframe1, libgprofng0, cpp, and gcc SHALL NOT be present in the final image

#### Scenario: PHP-FPM operates normally without build tools
- **WHEN** php-fpm starts in a container built from the image
- **THEN** php-fpm SHALL start successfully and serve requests with all extensions loaded

### Requirement: ImageMagick policy SHALL restrict high-risk operations
The Dockerfile SHALL configure ImageMagick's policy.xml to disable high-risk delegates and restrict resource consumption, reducing the attack surface for ImageMagick-related CVEs.

#### Scenario: URL-based image fetching disabled
- **WHEN** ImageMagick processes an image referencing an external URL
- **THEN** the operation SHALL be denied by the policy

#### Scenario: Standard image formats remain functional
- **WHEN** ImageMagick processes JPEG, PNG, GIF, or WebP images
- **THEN** the processing SHALL complete successfully

### Requirement: Build artifacts and caches SHALL be cleaned up
The Dockerfile SHALL remove apt package lists, temporary files, and build caches in the final layer to minimize image size and reduce information leakage.

#### Scenario: No apt lists in final image
- **WHEN** the Docker image build completes
- **THEN** the directory /var/lib/apt/lists/ SHALL be empty
- **THEN** the directories /tmp/ and /var/tmp/ SHALL not contain build artifacts
