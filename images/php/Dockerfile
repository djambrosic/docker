FROM php:8.2-fpm as base

ARG PHP_EXT_DIR=/usr/local/lib/php/extensions/no-debug-non-zts-20220829/
ARG PHP_CONF_DIR=/usr/local/etc/php/conf.d/
ARG PHP_INCLUDE_EXT_DIR=/usr/local/include/php/ext/

RUN apt-get update && apt-get install -y \
    zlib1g-dev \
    libpng-dev \
    libzip-dev \
    curl \
    libfreetype6-dev \
    libjpeg62-turbo-dev \
    libmcrypt-dev \
    libpng-dev \
    libxml2-dev \
    unzip \
    && docker-php-ext-install -j$(nproc) pdo_mysql zip mysqli soap \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j$(nproc) gd

FROM base as composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

FROM base as xdebug
RUN pecl install xdebug && docker-php-ext-enable xdebug

FROM base as redis
RUN pecl install redis && docker-php-ext-enable redis

FROM base as intl
RUN docker-php-ext-configure intl && docker-php-ext-install intl

FROM base as final
COPY --from=composer /usr/local/bin/composer /usr/local/bin/composer
COPY --from=xdebug ${PHP_EXT_DIR} ${PHP_EXT_DIR}
COPY --from=xdebug ${PHP_CONF_DIR} ${PHP_CONF_DIR}
COPY --from=redis ${PHP_EXT_DIR} ${PHP_EXT_DIR}
COPY --from=redis ${PHP_CONF_DIR} ${PHP_CONF_DIR}
COPY --from=intl ${PHP_EXT_DIR} ${PHP_EXT_DIR}
COPY --from=intl ${PHP_CONF_DIR} ${PHP_CONF_DIR}

RUN apt-get -y autoremove && apt-get clean && rm -rf /var/lib/apt.list/* /tmp/* /var/tmp/*

RUN usermod -u 1000 www-data

RUN chown -R www-data:1000 /var/www/

USER www-data

WORKDIR /var/www/html

CMD ["php-fpm"]

EXPOSE 9000
EXPOSE 9001
