FROM php:7.0.32-fpm-stretch

# Install SSL root cert
ADD ssl/printed.com-RootCA.crt /usr/local/share/ca-certificates

RUN echo "deb http://http.us.debian.org/debian/ testing non-free contrib main" >> /etc/apt/sources.list \
    && apt-get update

RUN apt-get install -y \
	bzip2 \
	g++ \
	git \
	imagemagick \
	libbz2-dev \
	libfreetype6-dev \
	libghc-postgresql-libpq-dev \
	libicu-dev \
	libjpeg62-turbo-dev \
	libmcrypt-dev \
	libmemcached-dev \
	libnode64 \
	libpng-dev \
	libpq-dev \
	libwebp-dev \
	libxml2-dev \
	libxpm-dev \
	libxslt1-dev \
	make \
	nano \
	nodejs \
	pkg-config \
	telnet \
	vim \
	zlib1g-dev \
	zsh

# @todo this is currently broken: "freetype-config not found"
# Explanation here: https://bugs.php.net/bug.php?id=76324
# Possible solution here: https://github.com/elivz/docker-php-dev/blob/master/Dockerfile
RUN docker-php-ext-configure gd  -with-freetype-dir=/usr/include/ -with-jpeg-dir=/usr/include/

RUN docker-php-ext-install mcrypt mbstring bz2 bcmath zip \
	&& docker-php-ext-configure pgsql -with-pgsql=/usr/include/postgresql/ \
	&& docker-php-ext-install gd pgsql pdo_pgsql pdo_mysql soap intl xsl \
	&& docker-php-ext-enable opcache

RUN pecl install -o -f xdebug \
    && rm -rf /tmp/pear \
    && echo "zend_extension=/usr/local/lib/php/extensions/no-debug-non-zts-20151012/xdebug.so" > /usr/local/etc/php/conf.d/xdebug.ini \
    && echo "xdebug.remote_enable=on"  >> /usr/local/etc/php/conf.d/xdebug.ini \
    && echo "xdebug.remote_host=10.0.2.2" >> /usr/local/etc/php/conf.d/xdebug.ini \
    && echo "xdebug.remote_autostart=off" >> /usr/local/etc/php/conf.d/xdebug.ini \
    && echo "xdebug.remote_connect_back=On" >> /usr/local/etc/php/conf.d/xdebug.ini \
    && echo "xdebug.idekey=PHPSTORM" >> /usr/local/etc/php/conf.d/xdebug.ini \
    && echo "memory_limit = 128M" >> /usr/local/etc/php/conf.d/php.ini \
    && echo "upload_max_filesize = 8M" >> /usr/local/etc/php/conf.d/php.ini

# Install imagick
RUN runtimeRequirements="libmagickwand-6.q16-dev --no-install-recommends" \
    && apt-get update && apt-get install -y ${runtimeRequirements} \
    && ln -s /usr/lib/x86_64-linux-gnu/ImageMagick-6.8.9/bin-Q16/MagickWand-config /usr/bin/ \
    && pecl install imagick-3.4.1 \
    && echo "extension=imagick.so" > /usr/local/etc/php/conf.d/ext-imagick.ini \
    && rm -rf /var/lib/apt/lists/*

# Install Memcached for php 7
RUN curl -L -o /tmp/memcached.tar.gz "https://github.com/php-memcached-dev/php-memcached/archive/php7.tar.gz" \
    && mkdir -p memcached \
    && tar -C memcached -zxvf /tmp/memcached.tar.gz --strip 1 \
    && ( \
        cd memcached \
        && phpize \
        && ./configure \
        && make -j$(nproc) \
        && make install \
    ) \
    && rm -r memcached \
    && rm /tmp/memcached.tar.gz \
    && docker-php-ext-enable memcached

# Install npm bower and gulp
RUN curl -L https://npmjs.org/install.sh | sh \
    && npm install bower -g \
    && npm install gulp -g \
    && curl -sS https://getcomposer.org/installer | php \
    && mv composer.phar /usr/bin/composer \
    && update-ca-certificates \
    && apt-get autoremove -y \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

CMD ["php-fpm", "-F"]

EXPOSE 9000

