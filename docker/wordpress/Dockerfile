FROM debian:stable-slim AS builder

ARG WP_CLI_VERSION=2.12.0
WORKDIR /tmp

RUN apt-get update && apt-get install -y --no-install-recommends curl ca-certificates \
    && rm -rf /var/lib/apt/lists/*

RUN curl -fsSLo wp-cli.phar "https://github.com/wp-cli/wp-cli/releases/download/v${WP_CLI_VERSION}/wp-cli-${WP_CLI_VERSION}.phar" \
    && curl -fsSLo wp-cli.phar.sha512 "https://github.com/wp-cli/wp-cli/releases/download/v${WP_CLI_VERSION}/wp-cli-${WP_CLI_VERSION}.phar.sha512" \
    && echo "$(cat wp-cli.phar.sha512)  wp-cli.phar" | sha512sum -c - \
    && chmod +x wp-cli.phar

FROM wordpress:6.8-php8.3-apache AS workspace

ARG XDEBUG_VERSION=3.4.2

COPY --from=builder /tmp/wp-cli.phar /usr/local/bin/wp
RUN wp --info

RUN pecl install "xdebug-${XDEBUG_VERSION}" && docker-php-ext-enable xdebug

COPY ./php.ini "${PHP_INI_DIR}/conf.d/99-dev.ini"