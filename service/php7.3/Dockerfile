FROM php:7.1-fpm

#Install build packages and libaries
ARG APTPACKAGES="gettext libcurl4-openssl-dev libpq-dev libmysqlclient-dev libldap2-dev libxslt-dev \
    libxml2-dev libicu-dev libfreetype6-dev libjpeg62-turbo-dev libmemcached-dev \
    zlib1g-dev libpng12-dev"

ARG APTLIBS="locales libaio1 libcurl3 libgss3 libicu52 libmysqlclient18 libpq5 libmemcached11 libmemcachedutil2 \
    libldap-2.4-2 libxml2 libxslt1.1 libmcrypt-dev"

RUN apt-get update \
    && apt-get install -y --no-install-recommends $APTPACKAGES $APTLIBS curl git unzip ghostscript locales apt-transport-https


# Add locale
RUN echo 'en_US.UTF-8 UTF-8' >> /etc/locale.gen \
    && echo 'en_AU.UTF-8 UTF-8' >> /etc/locale.gen && locale-gen

# Add PHP-extensions
RUN docker-php-ext-install \
    intl \
    mysqli \
    opcache \
    pgsql \
    soap \
    xsl \
    xmlrpc \
    zip

# Install gd
RUN docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd

# Instal ldap
RUN docker-php-ext-configure ldap --with-libdir=lib/x86_64-linux-gnu/ \
    && docker-php-ext-install -j$(nproc) ldap

# Install external Tools
RUN pecl install solr memcached redis apcu igbinary \
    && docker-php-ext-enable solr memcached redis apcu igbinary \
    && echo 'apc.enable_cli = On' >> /usr/local/etc/php/conf.d/docker-php-ext-apcu.ini

# Keep our image size down..
RUN pecl clear-cache \
    && apt-get remove --purge -y $APTPACKAGES \
    && apt-get autoremove -y \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

EXPOSE 9000

RUN sed -ri 's/^www-data:x:33:33:/www-data:x:1000:100:/' /etc/passwd

ARG MOODLE_DOCKER_DATAROOT=/var/www/moodledata
ARG MOODLE_DOCKER_BEHAT_DATAROOT=/var/www/behatdata
ARG MOODLE_DOCKER_BEHAT_FAILDUMP=/var/www/behatfaildump
ARG MOODLE_DOCKER_PHPUNIT_DATAROOT=/var/www/phpunitdata

RUN mkdir -p $MOODLE_DOCKER_DATAROOT && chown www-data: $MOODLE_DOCKER_DATAROOT -R && chmod 775 $MOODLE_DOCKER_DATAROOT -R \
    && mkdir -p $MOODLE_DOCKER_BEHAT_DATAROOT && chown www-data: $MOODLE_DOCKER_BEHAT_DATAROOT -R && chmod 775 $MOODLE_DOCKER_BEHAT_DATAROOT -R \
    && mkdir -p $MOODLE_DOCKER_BEHAT_FAILDUMP && chown www-data: $MOODLE_DOCKER_BEHAT_FAILDUMP -R && chmod 775 $MOODLE_DOCKER_BEHAT_FAILDUMP -R \
    && mkdir -p $MOODLE_DOCKER_PHPUNIT_DATAROOT && chown www-data: $MOODLE_DOCKER_PHPUNIT_DATAROOT -R && chmod 775 $MOODLE_DOCKER_PHPUNIT_DATAROOT -R

RUN  curl https://raw.githubusercontent.com/AminMkh/docker-php7-oci8-apache/b7c740638776552f00178a5d12905cefb50c7848/oracle/instantclient-basic-linux.x64-12.1.0.2.0.zip\
    -o /tmp/instantclient-basic-linux.x64-12.1.0.2.0.zip \
    && curl https://raw.githubusercontent.com/AminMkh/docker-php7-oci8-apache/b7c740638776552f00178a5d12905cefb50c7848/oracle/instantclient-sdk-linux.x64-12.1.0.2.0.zip\
    -o /tmp/instantclient-sdk-linux.x64-12.1.0.2.0.zip \
    && curl https://raw.githubusercontent.com/AminMkh/docker-php7-oci8-apache/b7c740638776552f00178a5d12905cefb50c7848/oracle/instantclient-sqlplus-linux.x64-12.1.0.2.0.zip\
    -o /tmp/instantclient-sqlplus-linux.x64-12.1.0.2.0.zip

RUN unzip /tmp/instantclient-basic-linux.x64-12.1.0.2.0.zip -d /usr/local/ \
    && rm /tmp/instantclient-basic-linux.x64-12.1.0.2.0.zip
RUN unzip /tmp/instantclient-sdk-linux.x64-12.1.0.2.0.zip -d /usr/local/ \
    && rm /tmp/instantclient-sdk-linux.x64-12.1.0.2.0.zip
RUN unzip /tmp/instantclient-sqlplus-linux.x64-12.1.0.2.0.zip -d /usr/local/ \
    && rm /tmp/instantclient-sqlplus-linux.x64-12.1.0.2.0.zip

RUN ln -s /usr/local/instantclient_12_1 /usr/local/instantclient \
    && ln -s /usr/local/instantclient/libclntsh.so.12.1 /usr/local/instantclient/libclntsh.so \
    && ln -s /usr/local/instantclient/sqlplus /usr/bin/sqlplus

RUN echo 'instantclient,/usr/local/instantclient' | pecl install oci8 && docker-php-ext-enable oci8 \
    && echo 'oci8.statement_cache_size = 0' >> /usr/local/etc/php/conf.d/docker-php-ext-oci8.ini