FROM php:8.2-fpm AS base

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

FROM base AS composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

FROM base AS xdebug
RUN pecl install xdebug && docker-php-ext-enable xdebug

FROM base AS redis
RUN pecl install redis && docker-php-ext-enable redis

FROM base AS intl
RUN docker-php-ext-configure intl && docker-php-ext-install intl

FROM base AS final
COPY --from=composer /usr/local/bin/composer /usr/local/bin/composer
COPY --from=xdebug ${PHP_EXT_DIR} ${PHP_EXT_DIR}
COPY --from=xdebug ${PHP_CONF_DIR} ${PHP_CONF_DIR}
COPY --from=redis ${PHP_EXT_DIR} ${PHP_EXT_DIR}
COPY --from=redis ${PHP_CONF_DIR} ${PHP_CONF_DIR}
COPY --from=intl ${PHP_EXT_DIR} ${PHP_EXT_DIR}
COPY --from=intl ${PHP_CONF_DIR} ${PHP_CONF_DIR}

RUN apt-get -y autoremove && apt-get clean && rm -rf /var/lib/apt.list/* /tmp/* /var/tmp/*

ARG USERNAME=app
ARG USER_ID
ARG GROUP_ID

RUN groupadd -g $GROUP_ID -o $USERNAME
RUN useradd -m -u $USER_ID -g $GROUP_ID -o -s /bin/bash $USERNAME

RUN chown -R www-data:$USERNAME /var/www/

USER $USERNAME

WORKDIR /var/www/html

CMD ["php-fpm"]

EXPOSE 9000
EXPOSE 9001
