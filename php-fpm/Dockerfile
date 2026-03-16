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
        zip


###########################################################################
# ext-mongodb（PHP 8.1 需指定版本）
###########################################################################
# RUN if [ "${PHP_VERSION}" = "8.1" ]; then \
#         install-php-extensions mongodb-1.21.5; \
#     else \
#         install-php-extensions mongodb; \
#     fi

RUN install-php-extensions mongodb-1.21.5


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

WORKDIR /var/www

CMD ["php-fpm"]

EXPOSE 9000
