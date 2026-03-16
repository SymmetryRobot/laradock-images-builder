FROM nginx:alpine

COPY nginx.conf /etc/nginx/
COPY logrotate/nginx /etc/logrotate.d/
COPY startup.sh /opt/startup.sh

ARG PHP_UPSTREAM_CONTAINER=php-fpm
ARG PHP_UPSTREAM_PORT=9000

RUN apk update \
    && apk upgrade \
    && apk add --no-cache logrotate openssl bash curl \
    && rm -rf /var/cache/apk/* \
    && set -x \
    && addgroup -g 82 -S www-data || true \
    && (adduser -u 82 -D -S -G www-data www-data || true) \
    && touch /var/log/messages \
    && echo "upstream php-upstream { server ${PHP_UPSTREAM_CONTAINER}:${PHP_UPSTREAM_PORT}; }" > /etc/nginx/conf.d/upstream.conf \
    && rm /etc/nginx/conf.d/default.conf \
    && sed -i 's/\r//g' /opt/startup.sh

CMD ["/bin/bash", "/opt/startup.sh"]

EXPOSE 80 81 443
