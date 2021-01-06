FROM php:7.4.13-fpm

# Copy composer.lock and composer.json
COPY composer.lock composer.json /var/www/
COPY docker-entry.sh /
RUN chmod +x /docker-entry.sh

# Set working directory
WORKDIR /var/www

#Root privileges for download
USER root

# Install dependencies
RUN apt-get update && apt-get install -y \
    build-essential \
    mariadb-client \
    libpng-dev \
    libjpeg62-turbo-dev \
    libfreetype6-dev \
    libonig-dev \
    zlib1g-dev\
    locales \
    zip \
    jpegoptim optipng pngquant gifsicle \
    vim \
    unzip \
    git \
    curl \
    nginx

# Clear cache
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

# Install extensions
RUN docker-php-ext-install pdo_mysql
RUN docker-php-ext-install exif
RUN docker-php-ext-install pcntl
RUN docker-php-ext-configure gd --with-freetype --with-jpeg
RUN docker-php-ext-install gd

ADD https://github.com/mlocati/docker-php-extension-installer/releases/latest/download/install-php-extensions /usr/local/bin/
RUN chmod +x /usr/local/bin/install-php-extensions && sync && \
    install-php-extensions bcmath \
    calendar \
    gettext \
    imagick \
    ioncube_loader \
    mcrypt \
    mysqli \
    soap \
    sockets \
    xsl \
    zip

# Adding pdflib.so file to extension_dir path
# COPY ./docker/php/php_pdflib.so /usr/local/lib/php/extensions/no-debug-non-zts-20190902/

# Install composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Add user for laravel application
RUN groupadd -g 1000 www
RUN useradd -u 1000 -ms /bin/bash -g www www

# Copy existing application directory contents
COPY . /var/www

COPY docker/nginx/conf.d/ /etc/nginx/conf.d/

COPY docker/php/local.ini /usr/local/etc/php/conf.d/local.ini
RUN cat /usr/local/etc/php/conf.d/local.ini

RUN rm -rf /etc/nginx/sites-enabled
RUN mkdir -p /etc/nginx/sites-enabled

# Copy existing application directory permissions
COPY --chown=www:www . /var/www

# Change current user to www
USER www

#Composer update for /vendor/autoload.php
RUN composer update --no-scripts

# Expose port 9000 and start php-fpm server
EXPOSE 80
CMD ["/docker-entry.sh"]
