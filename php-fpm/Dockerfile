ARG PHP_VERSION=8.4

FROM php:${PHP_VERSION}-fpm

COPY --from=mlocati/php-extension-installer /usr/bin/install-php-extensions /usr/local/bin/

ARG PHP_VERSION

###########################################################################
# PHP 擴展
###########################################################################
RUN install-php-extensions \
        bcmath \
        curl \
        gd \
        igbinary \
        imagick \
        imap \
        mbstring \
        memcached \
        opcache \
        redis \
        soap \
        sqlite3 \
        xmlrpc \
        xml \
        zip \
        opentelemetry \
        rdkafka


###########################################################################
# ext-mongodb
# PHP8.5 安裝最新版
# PHP8.4 安裝 2.1.1
# PHP8.1~8.3 安裝 1.21.5
###########################################################################
RUN if [ "${PHP_VERSION}" = "8.5" ]; then \
        install-php-extensions mongodb; \
    elif [ "${PHP_VERSION}" = "8.4" ]; then \
        install-php-extensions mongodb-2.1.1; \
    else \
        install-php-extensions mongodb-1.21.5; \
    fi


###########################################################################
# Xdebug（PHP 8.1 需指定版本）
###########################################################################
ARG XDEBUG_PORT=9003

RUN if [ "${PHP_VERSION}" = "8.1" ]; then \
        install-php-extensions xdebug-3.4.1; \
    else \
        install-php-extensions xdebug; \
    fi

COPY ./xdebug.ini /usr/local/etc/php/conf.d/xdebug.ini
RUN sed -i "s/xdebug.client_port=9000/xdebug.client_port=${XDEBUG_PORT}/" /usr/local/etc/php/conf.d/xdebug.ini

###########################################################################
# 設定檔
###########################################################################
COPY ./laravel.ini /usr/local/etc/php/conf.d
COPY ./xlaravel.pool.conf /usr/local/etc/php-fpm.d/

###########################################################################
# 使用者設定
###########################################################################
ARG PUID=1000
ENV PUID=${PUID}
ARG PGID=1000
ENV PGID=${PGID}


# 若基礎映像改用 Alpine 版 php:${PHP_VERSION}-fpm-alpine,則改用：
# RUN apk add --no-cache shadow autoconf && \
#     groupmod -o -g ${PGID} www-data && \
#     usermod -o -u ${PUID} -g www-data www-data && \
#     apk del shadow

# 若基礎映像改用 Debian 版 php:${PHP_VERSION}-fpm（非 alpine），則改用：
RUN apt-get update && apt-get install -y passwd && \
    groupmod -o -g ${PGID} www-data && \
    usermod -o -u ${PUID} -g www-data www-data && \
    apt-get purge -y passwd && apt-get autoremove -y && rm -rf /var/lib/apt/lists/*

###########################################################################
# 安全強化：升級套件、移除建置工具、清理
# 注意：移除 binutils 後若需在容器內臨時編譯 PHP 擴充，請先執行：
#   apt-get update && apt-get install -y binutils gcc make autoconf
###########################################################################
RUN apt-get update \
    && apt-get upgrade -y \
    && apt-get purge -y \
        binutils \
        binutils-common \
        binutils-x86-64-linux-gnu \
        libbinutils \
        libctf-nobfd0 \
        libctf0 \
        libgprofng0 \
        libsframe1 \
        cpp \
        gcc \
        2>/dev/null || true \
    && apt-get autoremove -y \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

###########################################################################
# ImageMagick policy：限制高風險 coder/delegate 和資源用量
###########################################################################
COPY ./imagemagick-policy.xml /tmp/imagemagick-policy.xml
RUN POLICY_DIR=$(ls -d /etc/ImageMagick* 2>/dev/null | head -1) \
    && if [ -n "$POLICY_DIR" ]; then cp /tmp/imagemagick-policy.xml "$POLICY_DIR/policy.xml"; fi \
    && rm -f /tmp/imagemagick-policy.xml

WORKDIR /var/www

CMD ["php-fpm"]

EXPOSE 9000
