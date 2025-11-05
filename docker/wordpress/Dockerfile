FROM debian:stable-slim AS builder

RUN apt update && apt install -y curl && rm -rf /var/lib/apt/lists/*

WORKDIR /tmp

RUN curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar

RUN chmod +x ./wp-cli.phar

FROM wordpress:latest AS workspace

COPY --from=builder /tmp/wp-cli.phar /usr/local/bin/wp

RUN wp --info

RUN pecl install xdebug && docker-php-ext-enable xdebug

COPY ./php.ini "${PHP_INI_DIR}/conf.d"