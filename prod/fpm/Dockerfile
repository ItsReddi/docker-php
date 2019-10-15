FROM php:7.3.10-fpm


###Environments defaults
ENV PHP_SESSION_SAVE_HANDLER files
ENV PHP_SESSION_SAVE_PATH /tmp
ENV PHP_SESSION_NAME PHPSESSID
ENV PHP_OP_CACHE_ENABLE 1
ENV PHP_OP_CACHE_REVALIDATE_FREQ 600
ENV PHP_OP_CACHE_SAVE_COMMENTS 1
ENV PHP_CLEAR_ENV yes
ENV PHP_TIMEZONE UTC
ENV MAX_INPUT_VARS 1000
ENV PHPREDIS_VERSION 4.2.0
ENV PHPXDEBUG_VERSION 2.6.1

RUN mkdir -p /usr/src/php/ext/redis \
    && curl -L https://github.com/phpredis/phpredis/archive/$PHPREDIS_VERSION.tar.gz | tar xvz -C /usr/src/php/ext/redis --strip 1 \
    && echo 'redis' >> /usr/src/php-available-exts

RUN apt-get update && apt-get upgrade -y && apt-get install --no-install-recommends -y \
        cron \
        libc-client-dev \ 
        libkrb5-dev \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libmcrypt-dev \
        libpng-dev \
        libicu-dev \
        zlib1g-dev \
        g++ \
    && pecl install redis-$PHPREDIS_VERSION \
    && docker-php-ext-enable redis \
    && docker-php-ext-install -j$(nproc) iconv mysqli pdo pdo_mysql intl opcache \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd \
    && docker-php-ext-configure imap --with-imap-ssl --with-kerberos \
    && docker-php-ext-install -j$(nproc) imap \
    && apt-get remove --purge -y g++\
    && apt-get autoremove -y \
    && apt-get clean \
    && rm -r /var/lib/apt/lists/*

COPY conf/php.ini /usr/local/etc/php/
COPY conf/php-cli.ini /usr/local/etc/php/
COPY conf/php-fpm.conf /usr/local/etc/
COPY conf/www.conf /usr/local/etc/php-fpm.d/

#Clear env directive correction
RUN sed -i "s|clear_env\s*=\s*no|clear_env = $PHP_CLEAR_ENV|g" /usr/local/etc/php-fpm.d/docker.conf \
&& sed -i "s|clear_env\s*=\s*yes|clear_env = $PHP_CLEAR_ENV|g" /usr/local/etc/php-fpm.d/www.conf

#ports
EXPOSE 9000
