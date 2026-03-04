# --- STAGE 1: BUILDER ---
FROM php:8.4-fpm-alpine AS builder

# Install build dependencies
RUN apk add --no-cache \
    $PHPIZE_DEPS \
    freetype-dev \
    libjpeg-turbo-dev \
    libpng-dev \
    libzip-dev \
    libxml2-dev \
    icu-dev \
    oniguruma-dev \
    linux-headers

# Install & Configure PHP Extensions
RUN docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j$(nproc) \
    pdo_mysql \
    mysqli \
    zip \
    soap \
    gd \
    intl \
    opcache

# Install PECL extensions
RUN pecl install redis xdebug \
    && docker-php-ext-enable redis xdebug

# --- STAGE 2: FINAL ---
FROM php:8.4-fpm-alpine AS final

# Install Runtime libraries (Alpine native names)
RUN apk add --no-cache \
    bash \
    curl \
    git \
    unzip \
    libpng \
    libjpeg-turbo \
    freetype \
    libzip \
    libxml2 \
    icu-libs

# Copy compiled extensions and configs
COPY --from=builder /usr/local/lib/php/extensions /usr/local/lib/php/extensions
COPY --from=builder /usr/local/etc/php/conf.d /usr/local/etc/php/conf.d
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# --- The Alpine Way to Handle Users ---
ARG USERNAME=app
ARG USER_ID=1000
ARG GROUP_ID=1000

# -G: group name, -g: group id
# -D: don't assign a password, -u: user id, -G: add to group, -h: home directory
RUN addgroup -g $GROUP_ID $USERNAME \
    && adduser -u $USER_ID -D -G $USERNAME -h /home/$USERNAME -s /bin/bash $USERNAME

WORKDIR /var/www/html

# Set permissions for Symfony directories
RUN chown -R $USERNAME:$USERNAME /var/www/html

USER $USERNAME

EXPOSE 9000
CMD ["php-fpm"]