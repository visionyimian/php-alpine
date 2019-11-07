from alpine:3.10

### ENV with NGINX and PHP
#### NGINX
ENV NGINX_VERSION 1.16.1
ENV NJS_VERSION   0.3.5
ENV PKG_RELEASE   1
#### PHP
ENV PHPIZE_DEPS \
        autoconf \
        dpkg-dev dpkg \
        file \
        g++ \
        gcc \
        libc-dev \
        make \
        pkgconf \
        re2c
ENV PHP_INI_DIR /usr/local/etc/php
ENV PHP_EXTRA_CONFIGURE_ARGS --enable-fpm --with-fpm-user=www-data --with-fpm-group=www-data --disable-cgi
ENV PHP_CFLAGS="-fstack-protector-strong -fpic -fpie -O2 -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64"
ENV PHP_CPPFLAGS="$PHP_CFLAGS"
ENV PHP_LDFLAGS="-Wl,-O1 -Wl,--hash-style=both -pie"

ENV GPG_KEYS 1729F83938DA44E27BA0F4D3DBDB397470D12172 B1B44D8F021E4E2D6021E995DC9FF8D3EE5AF27F

ENV PHP_VERSION 7.2.24
ENV PHP_URL="https://www.php.net/get/php-7.2.24.tar.xz/from/this/mirror" PHP_ASC_URL="https://www.php.net/get/php-7.2.24.tar.xz.asc/from/this/mirror"
ENV PHP_SHA256="a6a6cc03388060aa5f8f9e45799b72bce1c7ed7b9d7b3f1187787202aad91d25" PHP_MD5=""

### change the resource
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories; apk update

### NGINX install
RUN set -x \
    && addgroup -g 101 -S nginx \
    && adduser -S -D -H -u 101 -h /var/cache/nginx -s /sbin/nologin -G nginx -g nginx nginx \
    && apkArch="$(cat /etc/apk/arch)" \
    && nginxPackages=" \
        nginx=${NGINX_VERSION}-r${PKG_RELEASE} \
        nginx-module-xslt=${NGINX_VERSION}-r${PKG_RELEASE} \
        nginx-module-geoip=${NGINX_VERSION}-r${PKG_RELEASE} \
        nginx-module-image-filter=${NGINX_VERSION}-r${PKG_RELEASE} \
        nginx-module-njs=${NGINX_VERSION}.${NJS_VERSION}-r${PKG_RELEASE} \
    " \
    && case "$apkArch" in \
        x86_64) \
            set -x \
            && KEY_SHA512="e7fa8303923d9b95db37a77ad46c68fd4755ff935d0a534d26eba83de193c76166c68bfe7f65471bf8881004ef4aa6df3e34689c305662750c0172fca5d8552a *stdin" \
            && apk add --no-cache --virtual .cert-deps \
                openssl \
            && wget -O /tmp/nginx_signing.rsa.pub https://nginx.org/keys/nginx_signing.rsa.pub \
            && if [ "$(openssl rsa -pubin -in /tmp/nginx_signing.rsa.pub -text -noout | openssl sha512 -r)" = "$KEY_SHA512" ]; then \
                echo "key verification succeeded!"; \
                mv /tmp/nginx_signing.rsa.pub /etc/apk/keys/; \
            else \
                echo "key verification failed!"; \
                exit 1; \
            fi \
            && printf "%s%s%s\n" \
                "https://nginx.org/packages/alpine/v" \
                `egrep -o '^[0-9]+\.[0-9]+' /etc/alpine-release` \
                "/main" \
            | tee -a /etc/apk/repositories \
            && apk del .cert-deps \
            ;; \
        *) \
            set -x \
            && tempDir="$(mktemp -d)" \
            && chown nobody:nobody $tempDir \
            && apk add --no-cache --virtual .build-deps \
                gcc \
                libc-dev \
                make \
                openssl-dev \
                pcre-dev \
                zlib-dev \
                linux-headers \
                libxslt-dev \
                gd-dev \
                geoip-dev \
                perl-dev \
                libedit-dev \
                mercurial \
                bash \
                alpine-sdk \
                findutils \
            && su nobody -s /bin/sh -c " \
                export HOME=${tempDir} \
                && cd ${tempDir} \
                && hg clone https://hg.nginx.org/pkg-oss \
                && cd pkg-oss \
                && hg up -r 429 \
                && cd alpine \
                && make all \
                && apk index -o ${tempDir}/packages/alpine/${apkArch}/APKINDEX.tar.gz ${tempDir}/packages/alpine/${apkArch}/*.apk \
                && abuild-sign -k ${tempDir}/.abuild/abuild-key.rsa ${tempDir}/packages/alpine/${apkArch}/APKINDEX.tar.gz \
                " \
            && echo "${tempDir}/packages/alpine/" >> /etc/apk/repositories \
            && cp ${tempDir}/.abuild/abuild-key.rsa.pub /etc/apk/keys/ \
            && apk del .build-deps \
            ;; \
    esac \
    && apk add --no-cache $nginxPackages \
    && if [ -n "$tempDir" ]; then rm -rf "$tempDir"; fi \
    && if [ -n "/etc/apk/keys/abuild-key.rsa.pub" ]; then rm -f /etc/apk/keys/abuild-key.rsa.pub; fi \
    && if [ -n "/etc/apk/keys/nginx_signing.rsa.pub" ]; then rm -f /etc/apk/keys/nginx_signing.rsa.pub; fi \
    && sed -i '$ d' /etc/apk/repositories \
    && apk add --no-cache --virtual .gettext gettext \
    && mv /usr/bin/envsubst /tmp/ \
    \
    && runDeps="$( \
        scanelf --needed --nobanner /tmp/envsubst \
            | awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
            | sort -u \
            | xargs -r apk info --installed \
            | sort -u \
    )" \
    && apk add --no-cache $runDeps \
    && apk del .gettext \
    && mv /tmp/envsubst /usr/local/bin/ \
    && apk add --no-cache tzdata \
    && ln -sf /dev/stdout /var/log/nginx/access.log \
    && ln -sf /dev/stderr /var/log/nginx/error.log

### NGINX mkdir
RUN mkdir -p /run/nginx/

### PHP install

RUN apk add --no-cache \
        ca-certificates \
        curl \
        tar \
        xz \
        openssl

RUN set -eux; \
    #addgroup -g 82 -S www-data; \
    adduser -u 82 -D -S -G www-data www-data

RUN set -eux; \
    mkdir -p "$PHP_INI_DIR/conf.d"; \
    [ ! -d /var/www/html ]; \
    mkdir -p /var/www/html; \
    chown www-data:www-data /var/www/html; \
    chmod 777 /var/www/html

RUN set -eux; \
    \
    apk add --no-cache --virtual .fetch-deps gnupg; \
    \
    mkdir -p /usr/src; \
    cd /usr/src; \
    \
    curl -fsSL -o php.tar.xz "$PHP_URL"; \
    \
    if [ -n "$PHP_SHA256" ]; then \
        echo "$PHP_SHA256 *php.tar.xz" | sha256sum -c -; \
    fi; \
    if [ -n "$PHP_MD5" ]; then \
        echo "$PHP_MD5 *php.tar.xz" | md5sum -c -; \
    fi; \
    \
    if [ -n "$PHP_ASC_URL" ]; then \
        curl -fsSL -o php.tar.xz.asc "$PHP_ASC_URL"; \
        export GNUPGHOME="$(mktemp -d)"; \
        for key in $GPG_KEYS; do \
            gpg --batch --keyserver ha.pool.sks-keyservers.net --recv-keys "$key"; \
        done; \
        gpg --batch --verify php.tar.xz.asc php.tar.xz; \
        gpgconf --kill all; \
        rm -rf "$GNUPGHOME"; \
    fi; \
    \
    apk del --no-network .fetch-deps

COPY docker-php-source /usr/local/bin/

RUN chmod u+x /usr/local/bin/docker-php-source

RUN set -eux; \
    apk add --no-cache --virtual .build-deps \
        $PHPIZE_DEPS \
        argon2-dev \
        coreutils \
        curl-dev \
        libedit-dev \
        libsodium-dev \
        libxml2-dev \
        openssl-dev \
        sqlite-dev \
    ; \
    \
    export CFLAGS="$PHP_CFLAGS" \
        CPPFLAGS="$PHP_CPPFLAGS" \
        LDFLAGS="$PHP_LDFLAGS" \
    ; \
    docker-php-source extract; \
    cd /usr/src/php; \
    gnuArch="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)"; \
    ./configure \
        --build="$gnuArch" \
        --with-config-file-path="$PHP_INI_DIR" \
        --with-config-file-scan-dir="$PHP_INI_DIR/conf.d" \
        \
        --enable-option-checking=fatal \
        \
        --with-mhash \
        \
        --enable-ftp \
        --enable-mbstring \
        --enable-mysqlnd \
        --with-password-argon2 \
        --with-sodium=shared \
        --with-pdo-sqlite=/usr \
        --with-sqlite3=/usr \
        \
        --with-curl \
        --with-libedit \
        --with-openssl \
        --with-zlib \
        \
        $(test "$gnuArch" = 's390x-linux-musl' && echo '--without-pcre-jit') \
        \
        ${PHP_EXTRA_CONFIGURE_ARGS:-} \
    ; \
    make -j "$(nproc)"; \
    find -type f -name '*.a' -delete; \
    make install; \
    find /usr/local/bin /usr/local/sbin -type f -perm +0111 -exec strip --strip-all '{}' + || true; \
    make clean; \
    \
    cp -v php.ini-* "$PHP_INI_DIR/"; \
    \
    cd /; \
    docker-php-source delete; \
    \
    runDeps="$( \
        scanelf --needed --nobanner --format '%n#p' --recursive /usr/local \
            | tr ',' '\n' \
            | sort -u \
            | awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
    )"; \
    apk add --no-cache $runDeps; \
    \
    apk del --no-network .build-deps; \
    \
    pecl update-channels; \
    rm -rf /tmp/pear ~/.pearrc; \
    php --version

COPY docker-php-ext-* docker-php-entrypoint /usr/local/bin/

RUN chmod u+x /usr/local/bin/docker-php-ext-*

RUN docker-php-ext-enable sodium

WORKDIR /var/www/html

RUN set -eux; \
    cd /usr/local/etc; \
    if [ -d php-fpm.d ]; then \
        # for some reason, upstream's php-fpm.conf.default has "include=NONE/etc/php-fpm.d/*.conf"
        sed 's!=NONE/!=!g' php-fpm.conf.default | tee php-fpm.conf > /dev/null; \
        cp php-fpm.d/www.conf.default php-fpm.d/www.conf; \
    else \
        # PHP 5.x doesn't use "include=" by default, so we'll create our own simple config that mimics PHP 7+ for consistency
        mkdir php-fpm.d; \
        cp php-fpm.conf.default php-fpm.d/www.conf; \
        { \
            echo '[global]'; \
            echo 'include=etc/php-fpm.d/*.conf'; \
        } | tee php-fpm.conf; \
    fi; \
    { \
        echo '[global]'; \
        echo 'error_log = /proc/self/fd/2'; \
        echo; \
        echo '[www]'; \
        echo '; if we send this to /proc/self/fd/1, it never appears'; \
        echo 'access.log = /proc/self/fd/2'; \
        echo; \
        echo 'clear_env = no'; \
        echo; \
        echo '; Ensure worker stdout and stderr are sent to the main error log.'; \
        echo 'catch_workers_output = yes'; \
    } | tee php-fpm.d/docker.conf; \
    { \
        echo '[global]'; \
        echo 'daemonize = no'; \
        echo; \
        echo '[www]'; \
        echo 'listen = 9000'; \
    } | tee php-fpm.d/zz-docker.conf

### 自己安装扩展

RUN /usr/local/bin/docker-php-ext-install mysqli

RUN /usr/local/bin/docker-php-ext-install pdo_mysql

RUN /usr/local/bin/docker-php-ext-install opcache

RUN /usr/local/bin/docker-php-ext-install json

RUN curl 'http://pecl.php.net/get/redis-5.1.0.tgz' -o redis-5.1.0.tgz

RUN curl 'http://pecl.php.net/get/igbinary-3.0.1.tgz' -o igbinary-3.0.1.tgz

RUN curl 'http://pecl.php.net/get/memcached-3.1.4.tgz' -o memcached-3.1.4.tgz

RUN apk add --no-cache --virtual .build-deps autoconf gcc g++ make libffi-dev openssl-dev libmemcached zlib zlib-dev libmemcached-dev; \
    pecl install igbinary-3.0.1.tgz; \
    pecl install redis-5.0.1.tgz; \
    pecl install memcached-3.1.4.tgz; \
    apk del .build-deps

RUN chmod 755 /usr/local/lib/php/extensions/no-debug-non-zts-20170718/igbinary.so;chmod 755 /usr/local/lib/php/extensions/no-debug-non-zts-20170718/redis.so;chmod 755 /usr/local/lib/php/extensions/no-debug-non-zts-20170718/memcached.so;

COPY config/docker-php-ext-user.ini /usr/local/etc/php/conf.d/docker-php-ext-user.ini

# 进程管理器,超越supervisor的存在,cnm的垃圾,不支持shell强制写死bash
RUN apk add --no-cache bash

COPY forego-stable-linux-amd64.tgz forego-stable-linux-amd64.tgz

COPY Procfile Procfile

COPY start.sh start.sh

RUN chmod +x start.sh

RUN tar -zxvf forego-stable-linux-amd64.tgz;

RUN mkdir -p /run/nginx/ && mkdir -p /var/log/php/

COPY config/www.conf /usr/local/etc/php-fpm.d/www.conf

COPY config/nginx.conf /etc/nginx/nginx.conf

COPY config/php-server.conf /etc/nginx/conf.d/php-server.conf

RUN rm forego-stable-linux-amd64.tgz igbinary-3.0.1.tgz memcached-3.1.4.tgz redis-5.1.0.tgz

COPY php-server /var/www/html/php-server

RUN chown -R www-data:www-data /var/www/html/php-server

RUN apk add --no-cache libmemcached libmemcached-dev;

#RUN apk add --no-cache ffmpeg;

EXPOSE 80

STOPSIGNAL SIGTERM

CMD ["sh","-c","/var/www/html/start.sh"]