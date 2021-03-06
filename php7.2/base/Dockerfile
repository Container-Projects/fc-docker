FROM aliyunfc/php:7.2

MAINTAINER alibaba-serverless-fc


# Server path.
ENV FC_SERVER_PATH=/var/fc/runtime/php7.2

ENV FC_SERVER_PORT=9000 \
    FC_SERVER_LOG_PATH=${FC_SERVER_PATH}/var/log \
    FC_SERVER_LOG_LEVEL=INFO

RUN mkdir -p ${FC_SERVER_LOG_PATH}
RUN chmod 777 ${FC_SERVER_LOG_PATH}
RUN chmod -R 777 /tmp/

# php package path.
ENV FC_PHP_LIB_PATH=${FC_SERVER_PATH}/builtIn

# Function configuration.
ENV FC_FUNC_CODE_PATH=/code/ \
    FC_FUNC_LOG_PATH=/var/log/fc/

ENV LD_LIBRARY_PATH=${FC_FUNC_CODE_PATH}:${FC_FUNC_CODE_PATH}/lib:/usr/local/lib

ENV PHP_INI_SCAN_DIR=/usr/local/etc/php/conf.d:${FC_FUNC_CODE_PATH}extension

# Create directories.
RUN mkdir -p \
    ${FC_SERVER_PATH} \
    ${FC_SERVER_LOG_PATH}

# Change work directory.
WORKDIR ${FC_SERVER_PATH}

# make php and php-cgi default memory is 3G, https://aone.alibaba-inc.com/issue/17996176
RUN echo "memory_limit=3072M" >> /usr/local/etc/php/php.ini

RUN mv /etc/apt/sources.list /etc/apt/sources.list.bak
COPY commons/debian-jessie-sources.list /etc/apt/sources.list

RUN /bin/sh -c 'curl -sS https://getcomposer.org/installer | php && mv composer.phar /usr/local/bin/composer'

RUN apt-get update && apt-get --no-install-recommends  install -y apt-utils ca-certificates \
        git \
        wget \
        unzip\
        fonts-wqy-zenhei=0.9.45-6 \
        fonts-wqy-microhei=0.2.0-beta-2 \
        imagemagick \
        libmagickwand-dev \
        libmagickcore-dev \
        libmemcached-dev=1.0.18-4 \
    && rm -rf /var/lib/apt/lists/*

# phpunit and xdebug
RUN wget -O phpunit https://phar.phpunit.de/phpunit-7.2.7.phar
RUN /bin/sh -c 'chmod +x phpunit && mv phpunit /usr/local/bin/phpunit'

# ensure session.so loads before redis.so, https://github.com/phpredis/phpredis/issues/470
RUN mv /usr/local/etc/php/conf.d/docker-php-ext-session.ini /usr/local/etc/php/conf.d/docker-php-ext-a_session.ini

RUN pecl install redis-4.1.1 \
    && pecl install  xdebug-2.6.0 \
    && pecl install imagick-3.4.3 \
    && pecl install protobuf-3.6.0 \
    && pecl install memcached-3.0.4 \
    && docker-php-ext-enable redis xdebug imagick protobuf memcached

RUN docker-php-ext-install zip

# Change work directory.
WORKDIR ${FC_PHP_LIB_PATH}

# Install third party libraries for user function.
COPY php7.2/base/composer.json ./

RUN composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/
RUN composer install --no-dev

# Change work directory.
WORKDIR ${FC_SERVER_PATH}

# Generate usernames
RUN for i in $(seq 10000 10999); do \
        echo "user$i:x:$i:$i::/tmp:/usr/sbin/nologin" >> /etc/passwd; \
    done

# Start a shell by default
CMD ["bash"]
