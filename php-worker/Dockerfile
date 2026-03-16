ARG PHP_VERSION=8.4

FROM php:${PHP_VERSION}-alpine

ARG PHP_VERSION

COPY --from=mlocati/php-extension-installer /usr/bin/install-php-extensions /usr/local/bin/

###########################################################################
# 使用者設定
###########################################################################
ARG PUID=1000
ENV PUID=${PUID}
ARG PGID=1000
ENV PGID=${PGID}

RUN addgroup -g ${PGID} laradock && \
    adduser -D -G laradock -u ${PUID} laradock

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
        pcntl \
        pdo_mysql \
        redis \
        soap \
        sqlite3 \
        xmlrpc \
        xml \
        zip

###########################################################################
# ext-mongodb（PHP 8.1 需指定版本）
###########################################################################
RUN if [ "${PHP_VERSION}" = "8.1" ]; then \
        install-php-extensions mongodb-1.21.5; \
    else \
        install-php-extensions mongodb; \
    fi

###########################################################################
# 系統依賴（supervisor 等）
###########################################################################
RUN apk --update add \
        supervisor \
    && rm -rf /var/cache/apk/*

###########################################################################
# Supervisor 設定
###########################################################################
COPY supervisord.conf /etc/supervisord.conf

###########################################################################
# 時區
###########################################################################
ARG TZ=UTC
ENV TZ=${TZ}

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

###########################################################################
# Final
###########################################################################
RUN mkdir -p /var/www

WORKDIR /etc/supervisor/conf.d/

ENTRYPOINT ["/usr/bin/supervisord", "-n", "-c", "/etc/supervisord.conf"]
