# syntax=docker/dockerfile:experimental
FROM --platform=$TARGETPLATFORM alpine:3.11 as php

LABEL maintainer="khs1994-docker/lnmp <khs1994@khs1994.com>"

ARG VCS_REF

LABEL org.label-schema.vcs-ref=$VCS_REF \
      org.label-schema.vcs-url="https://github.com/khs1994-docker/php"

ENV TZ=Asia/Shanghai \
    APP_ENV=development

# ARG PHP_VERSION=

ARG ALPINE_URL=dl-cdn.alpinelinux.org

ARG PHP_EXTENSION_EXTRA

ARG PECL_EXTENSION_EXTRA

ARG APK_EXTRA

ARG APK_DEV_EXTRA

# dependencies required for running "phpize"
# these get automatically installed and removed by "docker-php-ext-*" (unless they're already installed)
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

# persistent / runtime deps
RUN set -ex \
      && sed -i "s/dl-cdn.alpinelinux.org/${ALPINE_URL}/g" /etc/apk/repositories \
      && apk add --no-cache \
                 curl \
                 tar \
                 xz \
# https://github.com/docker-library/php/issues/494
                 openssl \
                 tzdata \
# ensure www-data user exists
# RUN set -x \
      && addgroup -g 82 -S www-data \
      && adduser -u 82 -D -S -G www-data www-data
# 82 is the standard uid/gid for "www-data" in Alpine
# http://git.alpinelinux.org/cgit/aports/tree/main/apache2/apache2.pre-install?h=v3.3.2
# http://git.alpinelinux.org/cgit/aports/tree/main/lighttpd/lighttpd.pre-install?h=v3.3.2
# http://git.alpinelinux.org/cgit/aports/tree/main/nginx-initscripts/nginx-initscripts.pre-install?h=v3.3.2

ENV PHP_INI_DIR /usr/local/etc/php
# RUN mkdir -p $PHP_INI_DIR/conf.d

##<autogenerated>##
ENV PHP_EXTRA_CONFIGURE_ARGS \
        --disable-phpdbg \
        # https://github.com/docker-library/php/issues/510
        # --enable-embed=shared \
        --enable-fpm \
        --with-fpm-user=www-data \
        --with-fpm-group=www-data \
        --disable-cgi \
        --with-gettext=shared \
        --enable-gd=shared \
            --with-freetype \
            --disable-gd-jis-conv \
            --with-jpeg \
            --with-webp \
            --with-xpm \
        --with-pdo-mysql=shared \
        --with-pdo-pgsql=shared \
        # --with-xsl=shared \
        --enable-bcmath=shared \
        --enable-inline-optimization \
        --enable-mbregex \
        --enable-pcntl=shared \
        --enable-shmop=shared \
        # --enable-soap=shared \
        --enable-sockets=shared \
        --enable-sysvmsg=shared \
        --enable-sysvsem=shared \
        --enable-sysvshm=shared \
        --enable-xml \
        --with-zip \
        --enable-calendar=shared \
        --enable-intl=shared \
        --enable-exif=shared \
        --with-bz2=shared \
        # --with-tidy=shared \
        --with-gmp=shared \
        --with-kerberos \
        --with-imap=shared \
            --with-imap-ssl \
        --with-xmlrpc=shared \
        --with-pic \
        --with-enchant=shared \
        --enable-fileinfo=shared \
        --with-ldap=shared \
            --with-ldap-sasl \
        --enable-phar \
        --enable-posix=shared \
        --with-pspell=shared \
        --enable-shmop=shared \
        # --with-snmp=shared \
        --with-mysqli=shared \
        --with-pgsql=shared \
        --with-ffi=shared \
        --with-iconv=/usr
##</autogenerated>##

# Apply stack smash protection to functions using local buffers and alloca()
# Make PHP's main executable position-independent (improves ASLR security mechanism, and has no performance impact on x86_64)
# Enable optimization (-O2)
# Enable linker optimization (this sorts the hash buckets to improve cache locality, and is non-default)
# Adds GNU HASH segments to generated executables (this is used if present, and is much faster than sysv hash; in this configuration, sysv hash is also generated)
# https://github.com/docker-library/php/issues/272
ENV PHP_CFLAGS="-fstack-protector-strong -fpic -fpie -O2"
ENV PHP_CPPFLAGS="$PHP_CFLAGS"
ENV PHP_LDFLAGS="-Wl,-O1 -Wl,--hash-style=both -pie"

# ENV GPG_KEYS CBAF69F173A0FEA4B537F470D66C9593118BCCB6 F38252826ACD957EF380D39F2F7956BC5DA04B5D

# ENV PHP_VERSION 7.3.0beta2
# ENV PHP_URL="https://downloads.php.net/~cmb/php-7.3.0beta2.tar.xz" PHP_ASC_URL="https://downloads.php.net/~cmb/php-7.3.0beta2.tar.xz.asc"
# ENV PHP_SHA256="32670f40aecce130727d841e3191d30237caff643a239d3c16cd579e762bc4c6" PHP_MD5=""

# COPY --from=php:alpine /usr/local/bin/docker-php-* /usr/local/bin/

# COPY --from=php:alpine /usr/local/bin/docker-php-source /usr/local/bin/
# COPY --from=php:alpine /usr/local/bin/docker-php-entrypoint /usr/local/bin/
# COPY --from=php:alpine /usr/local/bin/docker-php-ext-configure /usr/local/bin/
# COPY --from=php:alpine /usr/local/bin/docker-php-ext-enable /usr/local/bin/
# COPY --from=php:alpine /usr/local/bin/docker-php-ext-install /usr/local/bin/

ARG PHP_GIT=https://github.com/php/php-src.git
# ARG PHP_GIT=https://gitee.com/mirrors/php-src

RUN --mount=type=bind,from=php:fpm-alpine,source=/usr/local/bin,target=/opt/bin,rw \
  set -xe \
  \
  && export PATH=$PATH:/opt/bin \
  ; cp /opt/bin/docker-php-entrypoint /usr/local/bin \
  \
  ; mkdir -p $PHP_INI_DIR/conf.d \
  # && apk add --no-cache --virtual .fetch-deps gnupg \
	&& mkdir -p /usr/src \
	&& cd /usr/src \
  && apk add --no-cache git openssh-client \
  \
  && if [ -d php ];then \
       cd php; \
       git reset --hard origin/master; \
       git pull --depth=1; \
     else \
       git clone --depth=1 ${PHP_GIT} php; \
     fi \
  \
  && apk del --no-network git openssh-client \
	# wget -O php.tar.xz "$PHP_URL"; \
	# if [ -n "$PHP_SHA256" ]; then \
	# 	echo "$PHP_SHA256 *php.tar.xz" | sha256sum -c -; \
	# fi; \
	# if [ -n "$PHP_MD5" ]; then \
	# 	echo "$PHP_MD5 *php.tar.xz" | md5sum -c -; \
	# fi; \
	# \
	# if [ -n "$PHP_ASC_URL" ]; then \
	# 	wget -O php.tar.xz.asc "$PHP_ASC_URL"; \
	# 	export GNUPGHOME="$(mktemp -d)"; \
	# 	for key in $GPG_KEYS; do \
	# 		gpg --keyserver ha.pool.sks-keyservers.net --recv-keys "$key"; \
	# 	done; \
	# 	gpg --batch --verify php.tar.xz.asc php.tar.xz; \
	# 	command -v gpgconf > /dev/null && gpgconf --kill all; \
	# 	rm -rf "$GNUPGHOME"; \
	# fi; \
	# && apk del --no-network  .fetch-deps \
	&& apk add --no-cache --virtual .build-deps \
          $PHPIZE_DEPS \
          argon2-dev \
          coreutils \
          curl-dev \
          libedit-dev \
          libsodium-dev \
          libxml2-dev \
          oniguruma-dev \
          openssl-dev \
          sqlite-dev \
          cyrus-sasl-dev \
          postgresql-dev \
          libzip-dev \
          zlib-dev \
          libpng-dev \
          freetype-dev \
          libjpeg-turbo-dev \
          libwebp-dev \
          libxpm-dev \
          libexif-dev \
          # libxslt-dev \
          gmp-dev \
          xmlrpc-c-dev \
          bzip2-dev \
          enchant-dev \
          imap-dev \
          gettext-dev \
          icu-dev \
          krb5-dev \
          aspell-dev \
          openldap-dev \
          pcre2-dev \
          # tidyhtml-dev \
          # net-snmp-dev \
          openldap-dev \
          \
          bison \
          libgcrypt-dev \
          libgpg-error-dev \
          gnu-libiconv-dev \
          ${APK_DEV_EXTRA:-} \
	\
	&& export CFLAGS="$PHP_CFLAGS" \
		        CPPFLAGS="$PHP_CPPFLAGS" \
		        LDFLAGS="$PHP_LDFLAGS" \
            ARGON2_LIBS="-largon2" \
            ARGON2_CFLAGS="-I/usr/include" \
	# && docker-php-source extract \
  && mv /usr/include/gnu-libiconv/*.h /usr/include \
  && rm -rf /usr/include/gnu-libiconv \
	&& cd /usr/src/php \
  && make clean || echo \
	&& gnuArch="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)" \
  && ./buildconf --force \
	&& ./configure \
		--build="$gnuArch" \
		--with-config-file-path="$PHP_INI_DIR" \
		--with-config-file-scan-dir="$PHP_INI_DIR/conf.d" \
		\
# make sure invalid --configure-flags are fatal errors intead of just warnings
		--enable-option-checking=fatal \
		\
# https://github.com/docker-library/php/issues/439
		--with-mhash \
		\
# --enable-ftp is included here because ftp_ssl_connect() needs ftp to be compiled statically (see https://github.com/docker-library/php/issues/236)
		--enable-ftp \
# --enable-mbstring is included here because otherwise there's no way to get pecl to use it properly (see https://github.com/docker-library/php/issues/195)
		--enable-mbstring \
# --enable-mysqlnd is included here because it's harder to compile after the fact than extensions are (since it's a plugin for several extensions, not an extension in itself)
		--enable-mysqlnd \
# https://wiki.php.net/rfc/libsodium
		--with-sodium=shared \
		\
		--with-curl \
		--with-libedit \
		--with-openssl \
		--with-zlib \
# https://wiki.php.net/rfc/argon2_password_hash (7.2+)
		--with-password-argon2 \
		\
# bundled pcre does not support JIT on s390x
# https://manpages.debian.org/stretch/libpcre3-dev/pcrejit.3.en.html#AVAILABILITY_OF_JIT_SUPPORT
		$(test "$gnuArch" = 's390x-linux-musl' && echo '--without-pcre-jit') \
		\
		$PHP_EXTRA_CONFIGURE_ARGS \
	&& make -j "$(nproc)" \
  && find -type f -name '*.a' -delete \
	&& make install \
	&& { find /usr/local/bin /usr/local/sbin -type f -perm +0111 -exec strip --strip-all '{}' + || true; } \
	&& make clean \
  && cp -v php.ini-* "$PHP_INI_DIR/" \
	&& cd / \
  # && docker-php-source delete \
  # && rm -rf /usr/src/php \
  && docker-php-ext-enable bcmath \
                           bz2 \
                           calendar \
                           enchant \
                           exif \
                           fileinfo \
                           gd \
                           gettext \
                           gmp \
                           imap \
                           intl \
                           mysqli \
                           pcntl \
                           pdo_mysql \
                           pdo_pgsql \
                           pgsql \
                           posix \
                           pspell \
                           shmop \
                           # snmp \
                           # soap \
                           sodium \
                           sockets \
                           sysvmsg \
                           sysvsem \
                           sysvshm \
                           # tidy \
                           xmlrpc \
                           # xsl \
                           # opcache 已默认安装，需要自行载入
                           opcache \
                           # 7.4 +
                           ffi \
                           ${PHP_EXTENSION_EXTRA:-} \
	&& runDeps="$( \
		scanelf --needed --nobanner --format '%n#p' --recursive /usr/local \
			| tr ',' '\n' \
			| sort -u \
			| awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
	)" \
	&& apk add --no-cache $runDeps \
  \
#  && { find $(php-config --extension-dir) -type f -perm +0111 -exec strip --strip-all '{}' + || true; } \
  && apk del --no-network  .build-deps \
  \
# sodium was built as a shared module (so that it can be replaced later if so desired), so let's enable it too (https://github.com/docker-library/php/issues/598)
# RUN docker-php-ext-enable sodium
  \
# ENTRYPOINT ["docker-php-entrypoint"]
  \
# RUN set -ex \
	&& cd /usr/local/etc \
	&& if [ -d php-fpm.d ]; then \
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
	fi \
	&& { \
		echo '[global]'; \
		echo 'error_log = /proc/self/fd/2'; \
    echo; echo '; https://github.com/docker-library/php/pull/725#issuecomment-443540114'; echo 'log_limit = 8192'; \
		echo; \
		echo '[www]'; \
		echo '; if we send this to /proc/self/fd/1, it never appears'; \
		echo 'access.log = /proc/self/fd/2'; \
		echo; \
		echo 'clear_env = no'; \
		echo; \
		echo '; Ensure worker stdout and stderr are sent to the main error log.'; \
		echo 'catch_workers_output = yes'; \
    echo 'decorate_workers_output = no'; \
	} | tee php-fpm.d/docker.conf \
	&& { \
		echo '[global]'; \
		echo 'daemonize = no'; \
		echo; \
		echo '[www]'; \
		echo 'listen = 9000'; \
	} | tee php-fpm.d/zz-docker.conf \
# 创建日志文件夹
  && mkdir -p /var/log/php-fpm \
  && ln -sf /dev/stdout /var/log/php-fpm/access.log \
  && ln -sf /dev/stderr /var/log/php-fpm/error.log \
  && ln -sf /dev/stderr /var/log/php-fpm/xdebug-remote.log \
  && chmod -R 777 /var/log/php-fpm \
  && rm -rf /usr/src/php \
  && curl -fsSL -o /usr/bin/pickle \
       https://github.com/khs1994-php/pickle/releases/download/nightly/pickle-debug.phar \
  && chmod +x /usr/bin/pickle

ENTRYPOINT ["docker-php-entrypoint"]

ENV PECL_BUILD_DEPS \
      libmemcached-dev \
      yaml-dev \
      zlib-dev

ENV PECL_RUN_DEPS \
      libmemcached-libs \
      yaml \
      zlib

ENV PECL_EXTENSION \
      # mongodb \
      https://github.com/mongodb/mongo-php-library/archive/master.tar.gz \
      igbinary \
      # redis \
      https://github.com/phpredis/phpredis/archive/develop.tar.gz \
      memcached \
      # xdebug-beta \
      yaml \
      ${PECL_EXTENSION_EXTRA:-}

RUN --mount=type=bind,from=php:alpine,source=/usr/local/bin,target=/opt/bin,rw \
    --mount=type=cache,target=/usr/local/lib/php/test \
    --mount=type=cache,target=/usr/local/lib/php/doc \
      set -x \
      && export PATH=$PATH:/opt/bin \
      && apk add --no-cache --virtual .pecl_build_deps $PECL_BUILD_DEPS $PHPIZE_DEPS \
      && apk add --no-cache --virtual .pecl_run_deps $PECL_RUN_DEPS \
      && echo "--enable-redis-igbinary" > /tmp/redis.configure.options \
      && echo "--enable-memcached-igbinary" > /tmp/memcached.configure.options \
      && pickle install $PECL_EXTENSION -n --defaults --strip --cleanup --continue-on-error \
      && pickle install \
           https://github.com/xdebug/xdebug/archive/master.tar.gz \
# https://github.com/tideways/php-xhprof-extension.git
           https://github.com/tideways/php-xhprof-extension/archive/master.tar.gz \
           -n --defaults --no-write --strip --cleanup --continue-on-error \
      # && phpize \
      # && ./configure --enable-xdebug-dev \
      # && make all -j "$(nproc)"\
      # && make install \
      && rm -rf /tmp/* \
      && apk del --no-network  .pecl_build_deps \
      && rm -rf /usr/local/lib/php/.registry/.channel.pecl.php.net/* \
      && php -m \
      && ls -la $(php-config --extension-dir)

STOPSIGNAL SIGQUIT
EXPOSE 9000
CMD ["php-fpm"]

WORKDIR /app
