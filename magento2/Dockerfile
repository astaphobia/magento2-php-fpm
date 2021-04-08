#FROM quay.io/alexcheng1982/apache2-php7:7.3.12
FROM php:7.4-apache

LABEL php_version="7.4"
LABEL magento_version="2.4.2"

ENV MAGENTO_VERSION 2.4.2
ENV INSTALL_DIR /var/www/html
ENV COMPOSER_HOME /var/www/.composer/

COPY --from=composer:latest /usr/bin/composer /usr/local/bin/composer
#RUN curl -sS https://getcomposer.org/installer | php \
    #&& mv composer.phar /usr/local/bin/composer
COPY ./auth.json $COMPOSER_HOME

#RUN requirements="libpng-dev libzip-dev libmcrypt-dev libmcrypt4 libcurl3-dev libfreetype6 libjpeg62-turbo-dev libfreetype6-dev libicu-dev libxslt1-dev unzip libonig-dev git cron ibmagickwand-dev" \
RUN requirements="libfreetype6-dev libjpeg62-turbo-dev libmcrypt-dev libpng-dev libwebp-dev curl libcurl4 libcurl4-openssl-dev zlib1g-dev libicu-dev unzip libzip-dev zip vim git cron libmagickwand-dev libxslt1-dev libonig-dev libmagickwand-dev" \
#RUN requirements="libpng++-dev libzip-dev libmcrypt-dev libmcrypt5 libcurl3-dev libfreetype6 libjpeg-turbo8 libjpeg-turbo8-dev libfreetype6-dev libicu-dev libxslt1-dev unzip" \
    && apt-get update \
    && apt-get install -y $requirements \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* \
    #&& docker-php-ext-configure gd --with-freetype --with-jpeg --with-webp --enable-gd \
    && docker-php-ext-configure gd --with-freetype --with-jpeg=/usr/include/ --enable-gd

RUN docker-php-ext-install gd \
    && docker-php-ext-install sockets \
    && docker-php-ext-install pdo_mysql \
    && docker-php-ext-install mbstring \
    && docker-php-ext-install zip \
    && docker-php-ext-install intl \
    && docker-php-ext-install xsl \
    && docker-php-ext-install soap \
    && docker-php-ext-install bcmath

RUN yes '' | pecl install mcrypt-1.0.3 \
    && echo 'extension=mcrypt.so' > /usr/local/etc/php/conf.d/mcrypt.ini

# install imagick
RUN pecl install imagick
RUN docker-php-ext-enable imagick

RUN chsh -s /bin/bash www-data

#RUN cd /tmp && \
  #curl https://codeload.github.com/magento/magento2/tar.gz/$MAGENTO_VERSION -o $MAGENTO_VERSION.tar.gz && \
  #tar xvf $MAGENTO_VERSION.tar.gz && \
  #mv magento2-$MAGENTO_VERSION/* magento2-$MAGENTO_VERSION/.htaccess $INSTALL_DIR
COPY ./magento2/ /var/www/html

RUN chown -R www-data:www-data /var/www
#RUN su www-data -c "cd $INSTALL_DIR && composer update php-amqplib/php-amqplib"
#RUN su www-data -c "cd $INSTALL_DIR && composer install"
#RUN su www-data -c "cd $INSTALL_DIR && composer config repositories.magento composer https://repo.magento.com/"

RUN cd $INSTALL_DIR \
    && find . -type d -exec chmod 770 {} \; \
    && find . -type f -exec chmod 660 {} \; \
    && chmod u+x bin/magento

COPY ./install-magento /usr/local/bin/install-magento
RUN chmod +x /usr/local/bin/install-magento

COPY ./install-sampledata /usr/local/bin/install-sampledata
RUN chmod +x /usr/local/bin/install-sampledata

RUN a2enmod rewrite
RUN echo "memory_limit=2048M" > /usr/local/etc/php/conf.d/memory-limit.ini

RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

WORKDIR $INSTALL_DIR

# Add cron job
ADD crontab /etc/cron.d/magento2-cron
RUN chmod 0644 /etc/cron.d/magento2-cron \
    && crontab -u www-data /etc/cron.d/magento2-cron

VOLUME $INSTALL_DIR
