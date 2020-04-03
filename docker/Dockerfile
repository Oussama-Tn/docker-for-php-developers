FROM php:7.2-apache-stretch

COPY --chown=www-data:www-data . /var/www/html
COPY /docker/vhost.conf /etc/apache2/sites-available/000-default.conf

#RUN pecl install -o -f redis \
#    &&  rm -rf /tmp/pear \
#    &&  echo "extension=redis.so" > /usr/local/etc/php/conf.d/redis.ini \
#    && a2enmod rewrite \
#    && docker-php-ext-install pdo_mysql

RUN a2enmod rewrite \
    && docker-php-ext-install pdo_mysql
