FROM php:7.1-fpm-alpine
LABEL maintainer "Raphaël Jadot"

# Install gd
RUN apk --update --no-cache add \
    freetype-dev \
    libjpeg-turbo-dev \
    libpng-dev \
  && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
  && docker-php-ext-install -j$(nproc) gd

# Install Imagick
RUN apk --update --no-cache add \
    imagemagick \
    imagemagick-dev \
    ${PHPIZE_DEPS} \
  && yes ""|pecl install imagick \
  && docker-php-ext-enable imagick \
  && apk del ${PHPIZE_DEPS}
  
# Install additional image tools
RUN apk --update --no-cache add \
    optipng \
    gifsicle

# set recommended PHP.ini settings
# see https://secure.php.net/manual/en/opcache.installation.php
RUN { \
    echo 'opcache.memory_consumption=128'; \
    echo 'opcache.interned_strings_buffer=8'; \
    echo 'opcache.max_accelerated_files=4000'; \
    echo 'opcache.revalidate_freq=2'; \
    echo 'opcache.fast_shutdown=1'; \
    echo 'opcache.enable_cli=0'; \
    } > /usr/local/etc/php/conf.d/opcache-recommended.ini

# Active apache rewrite mode
# (as it's not ubuntu a2enmode doesn't work. Instead we activate the module in spip.apache.conf)
    
VOLUME ["/var/www/html/core/"]
VOLUME ["/var/www/html/data/"]

ENV PIWIGO_VERSION 2.9
ENV PIWIGO_PACKAGE 2.9.4


# Install SPIP
RUN set -ex; \
  curl -o piwigo.tar.gz -fSL "github.com/Piwigo/Piwigo/archive/${PIWIGO_PACKAGE}.tar.gz"; \
  tar -xf piwigo.tar.gz -C /usr/src/; \
  rm piwigo.tar.gz; \
  mv /usr/src/Piwigo-${PIWIGO_PACKAGE} /usr/src/piwigo; \
  chown -R www-data:www-data /usr/src/piwigo

# PHP
ENV PHP_MAX_EXECUTION_TIME 60
ENV PHP_MEMORY_LIMIT 256M
ENV PHP_POST_MAX_SIZE 40M
ENV PHP_UPLOAD_MAX_FILESIZE 32M
ENV PHP_TIMEZONE America/Guadeloupe

EXPOSE 9000

COPY ./docker-entrypoint.sh /

ENTRYPOINT ["sh","/docker-entrypoint.sh"]

CMD ["php-fpm"]
