FROM wordpress:6.1.1-php8.2-fpm

LABEL maintainer="Vimlesh Kumar <vimlesh1gmail.com>"

RUN apt-get update
RUN apt-get install --no-install-recommends -y tidy csstidy nano netcat 

RUN mkdir -p /usr/src/php/ext

# Install needed php extensions: memcached
#
RUN apt-get install -y libpq-dev libmemcached-dev && \
    curl -o memcached.tgz -SL https://pecl.php.net/get/memcached-3.2.0.tgz && \
        tar -xf memcached.tgz -C /usr/src/php/ext/ && \
        echo extension=memcached.so >> /usr/local/etc/php/conf.d/memcached.ini && \
        rm memcached.tgz && \
        mv /usr/src/php/ext/memcached-3.2.0 /usr/src/php/ext/memcached

# set recommended PHP.ini settings
# see https://secure.php.net/manual/en/opcache.installation.php
RUN { \
        echo 'opcache.memory_consumption=128'; \
        echo 'opcache.interned_strings_buffer=8'; \
        echo 'opcache.max_accelerated_files=4000'; \
        echo 'opcache.revalidate_freq=2'; \
        echo 'opcache.fast_shutdown=1'; \
        echo 'opcache.enable_cli=1'; \
    } > /usr/local/etc/php/conf.d/opcache-recommended.ini


# Install needed php extensions: memcache
#
RUN apt-get install --no-install-recommends -y zlib1g-dev unzip libssl-dev libpcre3 libpcre3-dev && \
    cd /usr/src/php/ext/ && \
    curl -sSL -o memcache-8.tgz https://pecl.php.net/get/memcache-8.0.tgz && \
    tar -xf memcache-8.tgz && \
    mv memcache-8.0 memcache && \
    docker-php-ext-configure memcache --with-php-config=/usr/local/bin/php-config && \
    docker-php-ext-install memcache && \
    echo "extension=memcache.so" > /usr/local/etc/php/conf.d/ext-memcache.ini && \
    rm -rf /tmp/memcache-8.0 memcache-8.tgz


# Install needed php extensions: zip
#
RUN apt-get install -y libz-dev libzip && \
    curl -o zip.tgz -SL https://pecl.php.net/get/zip-1.21.1.tgz && \
        tar -xf zip.tgz -C /usr/src/php/ext/ && \
        rm zip.tgz && \
        mv /usr/src/php/ext/zip-1.21.1 /usr/src/php/ext/zip

RUN docker-php-ext-install memcached
RUN docker-php-ext-install memcache
RUN docker-php-ext-install zip

# lumen packages
# RUN apt-get install openldap-back-mdb
# RUN apt-get install --update --virtual .build-deps autoconf g++ make zlib-dev curl-dev libidn2-dev libevent-dev icu-dev libidn-dev openldap libxml2-dev


# Install needed php extensions: ldap
#
# RUN apt-get install --no-cache ldb-dev
# RUN ln -s /usr/lib/x86_64-linux-gnu/libldap.so /usr/lib/libldap.so \
#     && ln -s /usr/lib/x86_64-linux-gnu/liblber.so /usr/lib/liblber.so
# RUN apt-get install libldap2-dev -y && \
#     docker-php-ext-configure ldap && \
#     docker-php-ext-install ldap

# Install needed wordpress extensions: WP-FFPC
#
RUN cd /usr/src/wordpress/wp-content/plugins && \
    curl -o wp-ffpc.zip -L https://downloads.wordpress.org/plugin/wp-ffpc.zip && \
    unzip -o wp-ffpc.zip && \
    chown -R www-data:www-data wp-ffpc && \
    rm -f wp-ffpc.zip

# Enable Wordpress Caching
RUN cd /usr/src/wordpress && ls -l && \
    sed -ri -e "s#(<\?php)#\\1\ndefine('WP_CACHE', 'true');#g" wp-config-sample.php

# Install needed wordpress extensions: Simple-Ldap-Login
#
RUN cd /usr/src/wordpress/wp-content/plugins && \
    curl -o simple-ldap-login.zip -L https://downloads.wordpress.org/plugin/simple-ldap-login.1.6.0.zip && \
    unzip -o simple-ldap-login.zip && \
    chown -R www-data:www-data simple-ldap-login && \
    rm -f simple-ldap-login.zip

# Cleanup
RUN rm -rf /var/lib/apt/lists/*


# ENTRYPOINT resets CMD
ENTRYPOINT ["docker-entrypoint.sh"]
CMD ["php-fpm"]