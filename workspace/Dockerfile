ARG PHP_VERSION=8.4
ARG NODE_VERSION=24.14.0

###########################################################################
# NVM build stage
###########################################################################
FROM ubuntu:24.04 AS nvm-builder
ARG NODE_VERSION
ENV NVM_DIR=/home/laradock/.nvm

RUN apt-get update && apt-get install -y curl ca-certificates \
    && mkdir -p ${NVM_DIR} \
    && curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash \
    && . ${NVM_DIR}/nvm.sh \
    && nvm install ${NODE_VERSION} \
    && nvm alias default ${NODE_VERSION} \
    && npm update -g minimatch tar \
    && npm audit fix --global 2>/dev/null || true \
    && nvm cache clear \
    && npm cache clean --force

###########################################################################
# Main image
###########################################################################
FROM phusion/baseimage:noble-1.0.2

ARG PHP_VERSION
ENV DEBIAN_FRONTEND=noninteractive

USER root

###########################################################################
# 系統安全更新
###########################################################################
RUN apt-get update && apt-get upgrade -y -o Dpkg::Options::="--force-confold" && rm -rf /var/lib/apt/lists/*



###########################################################################
# PHP（透過 ondrej/php PPA）
###########################################################################
RUN apt-get update && apt-get install -y software-properties-common \
    && add-apt-repository -y ppa:ondrej/php \
    && apt-get update \
    && apt-get install -y \
        php${PHP_VERSION}-cli \
        php${PHP_VERSION}-common \
        php${PHP_VERSION}-dev \
        php${PHP_VERSION}-xml \
        php${PHP_VERSION}-curl \
        php${PHP_VERSION}-gd \
        php${PHP_VERSION}-mbstring \
        php${PHP_VERSION}-opcache \
        php${PHP_VERSION}-zip \
        php${PHP_VERSION}-bcmath \
        php${PHP_VERSION}-redis \
        php${PHP_VERSION}-sqlite3 \
        php${PHP_VERSION}-igbinary \
        php${PHP_VERSION}-memcached \
        php${PHP_VERSION}-xmlrpc \
        php${PHP_VERSION}-imagick \
        php${PHP_VERSION}-imap \
        php${PHP_VERSION}-soap \
        iputils-ping \
        wget \
        curl \
        zip \
        unzip \
        git \
    && rm -rf /var/lib/apt/lists/*

###########################################################################
# Chromium（從 Debian bookworm 倉庫安裝，支援 amd64 和 arm64）
###########################################################################
RUN set -eux; \
    apt-get update; \
    apt-get install -y --no-install-recommends wget gnupg2 ca-certificates; \
    gpg --keyserver keyserver.ubuntu.com --recv-keys 0E98404D386FA1D9; \
    gpg --export 0E98404D386FA1D9 | gpg --dearmour -o /etc/apt/trusted.gpg.d/debian-0E98404D.gpg; \
    gpg --keyserver keyserver.ubuntu.com --recv-keys 6ED0E7B82643E131; \
    gpg --export 6ED0E7B82643E131 | gpg --dearmour -o /etc/apt/trusted.gpg.d/debian-6ED0E7B8.gpg; \
    gpg --keyserver keyserver.ubuntu.com --recv-keys F8D2585B8783D481; \
    gpg --export F8D2585B8783D481 | gpg --dearmour -o /etc/apt/trusted.gpg.d/debian-F8D2585B.gpg; \
    echo "deb http://deb.debian.org/debian bookworm main" >> /etc/apt/sources.list; \
    apt-get update; \
    apt-get install -y --no-install-recommends chromium; \
    ln -sf /usr/bin/chromium /usr/bin/chromium-browser; \
    rm -rf /var/lib/apt/lists/*

###########################################################################
# Composer
###########################################################################
COPY --from=composer:latest /usr/bin/composer /usr/local/bin/composer

###########################################################################
# SSH（啟用 phusion/baseimage 內建的 sshd 服務）
###########################################################################
RUN rm -f /etc/service/sshd/down

###########################################################################
# 使用者設定
###########################################################################
ARG PUID=1000
ENV PUID=${PUID}
ARG PGID=1000
ENV PGID=${PGID}

RUN set -xe; \
    groupmod --new-name laradock ubuntu && \
    usermod --login laradock ubuntu --move-home --home /home/laradock && \
    usermod -p "*" laradock -s /bin/bash

###########################################################################
# NVM / Node
###########################################################################
ENV NVM_DIR=/home/laradock/.nvm

COPY --from=nvm-builder ${NVM_DIR} ${NVM_DIR}

RUN chown -R laradock:laradock ${NVM_DIR} \
    && echo '' >> /home/laradock/.bashrc \
    && echo 'export NVM_DIR="$HOME/.nvm"' >> /home/laradock/.bashrc \
    && echo '[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"' >> /home/laradock/.bashrc \
    && echo '' >> /root/.bashrc \
    && echo 'export NVM_DIR="/home/laradock/.nvm"' >> /root/.bashrc \
    && echo '[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"' >> /root/.bashrc \
    && find ${NVM_DIR} -type f -name node -exec ln -s {} /usr/local/bin/node \; \
    && NODE_DIR="${NVM_DIR}/versions/node/$(. ${NVM_DIR}/nvm.sh && node -v)/lib/node_modules" \
    && ln -s ${NODE_DIR}/npm/bin/npm-cli.js /usr/local/bin/npm \
    && ln -s ${NODE_DIR}/npm/bin/npx-cli.js /usr/local/bin/npx

###########################################################################
# 時區
###########################################################################
ARG TZ=UTC
ENV TZ=${TZ}

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

###########################################################################
# Crontab
###########################################################################
COPY ./crontab /etc/cron.d
RUN chmod -R 644 /etc/cron.d

###########################################################################
# User Aliases
###########################################################################
COPY ./aliases.sh /root/aliases.sh
COPY ./aliases.sh /home/laradock/aliases.sh

RUN sed -i 's/\r//' /root/aliases.sh && \
    sed -i 's/\r//' /home/laradock/aliases.sh && \
    chown laradock:laradock /home/laradock/aliases.sh && \
    echo "" >> ~/.bashrc && \
    echo "# Load Custom Aliases" >> ~/.bashrc && \
    echo "source ~/aliases.sh" >> ~/.bashrc && \
    echo "" >> ~/.bashrc && \
    echo "" >> /home/laradock/.bashrc && \
    echo "# Load Custom Aliases" >> /home/laradock/.bashrc && \
    echo "source ~/aliases.sh" >> /home/laradock/.bashrc && \
    echo "" >> /home/laradock/.bashrc

###########################################################################
# ext-mongodb
###########################################################################
# RUN pecl channel-update pecl.php.net && \
#     if [ "${PHP_VERSION}" = "8.1" ]; then \
#         MAKE="make -j$(nproc)" pecl install mongodb-1.21.5; \
#     else \
#         MAKE="make -j$(nproc)" pecl install mongodb; \
#     fi && \
#     echo "extension=mongodb.so" >> /etc/php/${PHP_VERSION}/cli/conf.d/30-mongodb.ini && \
#     rm -rf /tmp/pear

RUN pecl channel-update pecl.php.net && \
    MAKE="make -j$(nproc)" pecl install mongodb-1.21.5 && \
    echo "extension=mongodb.so" >> /etc/php/${PHP_VERSION}/cli/conf.d/30-mongodb.ini && \
    rm -rf /tmp/pear


###########################################################################
# Xdebug
###########################################################################
ARG XDEBUG_PORT=9003

RUN if [ "${PHP_VERSION}" = "8.1" ]; then \
        MAKE="make -j$(nproc)" pecl install xdebug-3.4.1; \
    else \
        MAKE="make -j$(nproc)" pecl install xdebug; \
    fi && \
    echo "zend_extension=xdebug.so" >> /etc/php/${PHP_VERSION}/cli/conf.d/20-xdebug.ini && \
    rm -rf /tmp/pear

COPY ./xdebug.ini /etc/php/${PHP_VERSION}/cli/conf.d/xdebug.ini

RUN sed -i "s/xdebug.client_port=9000/xdebug.client_port=${XDEBUG_PORT}/" /etc/php/${PHP_VERSION}/cli/conf.d/xdebug.ini

###########################################################################
# 最終安全更新 & 清理
###########################################################################
RUN apt-get update \
    && apt-get upgrade -y \
    && apt-get autoremove -y \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

###########################################################################
# Final
###########################################################################
WORKDIR /var/www

CMD ["/sbin/my_init"]
