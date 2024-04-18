FROM phpdockerio/php:8.2-fpm

RUN apt-get update \
    && apt-get -y --no-install-recommends install \
        git \ 
        php8.2-intl php8.2-mbstring php8.2-sqlite3\
        #For PostgreSQL
        php8.2-pgsql \
        #For MySQL
        #php8.2-mysql \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /usr/share/doc/*

WORKDIR "/var/www/myapp"
