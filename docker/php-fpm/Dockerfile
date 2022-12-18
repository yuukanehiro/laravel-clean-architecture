ARG PHP_VERSION=7.4.19
FROM php:${PHP_VERSION}-fpm-alpine

ENV TZ Asia/Tokyo

# Git と PHP 拡張のインストール
RUN set -eux \
 && apk add --update --no-cache git autoconf g++ libtool make libzip-dev libpng-dev libjpeg-turbo-dev freetype-dev mysql-client redis \
 && pecl install redis \
 && docker-php-ext-configure gd --with-jpeg=/usr \
 && docker-php-ext-configure opcache --enable-opcache \
 && docker-php-ext-install opcache bcmath pdo_mysql gd exif zip \
 && docker-php-ext-enable redis \
 && apk del autoconf g++ libtool make \
 && rm -rf /tmp/*


COPY ./docker/php-fpm/conf/zzz-.conf /usr/local/etc/php-fpm.d/zzz-.conf
COPY ./docker/php-fpm/conf/php.ini /usr/local/etc/php/php.ini

RUN mkdir -p /var/www/app
COPY ./src/ /var/www/app/
WORKDIR /var/www/app

# Composer のインストール
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/bin --filename=composer
RUN /usr/bin/composer config platform.php ${PHP_VERSION}
# 対策 Warning: The lock file is not up to date with the latest changes in composer.json.
# ハッシュ値のみ更新。パッケージはアップデートしない
RUN /usr/bin/composer update --lock
RUN COMPOSER_MEMORY_LIMIT=-1 /usr/bin/composer install --no-dev

RUN php artisan cache:clear && \
    php artisan config:clear

# nginxコンテナに共有する
RUN mkdir -p /var/run/php-fpm
VOLUME ["/var/run/php-fpm"]

# VOLUMEでマウントするとroot権限になるので権限設定
RUN chown -R www-data:www-data storage && \
    chown -R www-data:www-data bootstrap/cache && \
    chmod 2755 storage

RUN chown -R www-data:www-data /var/www/app