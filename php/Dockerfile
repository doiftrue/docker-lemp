##
## @var $PHP_INI_DIR eg: /usr/local/etc/php
## @var $PHP_VERSION eg: 7.4.33
##
ARG PHP_VER
FROM php:${PHP_VER}-fpm

ARG UID=1000
ARG GUID=1000
ARG USER=www-data
ARG USER_HOME_DIR=/var/www
ARG WORK_DIR=/var/app

RUN set -x; \
# WORK_DIR capabilities
	usermod -u $UID $USER; \
	groupmod -g $GUID $USER; \
	chown -R $USER:$USER $WORK_DIR; \
# Give access for the "www-data" to its HOME "/var/www" dir (needed for example for composer).
	chown -R $USER:$USER $USER_HOME_DIR; \
# Use the default PHP configuration.
# IMPORTANT: Use `php.ini-production` file for Prod env.
    cp "$PHP_INI_DIR/php.ini-development" "$PHP_INI_DIR/php.ini";


## OS Utilities
RUN set -xe; \
	apt-get update; \
	apt-get -y install \
		libpng-dev           \
		libonig-dev          \
		libxml2-dev          \
		libzip-dev           \
		default-mysql-client \
		vim                  \
		nano                 \
		unzip                \
		zip                  \
		wget                 \
		curl                 \
		git                  \
		tree                 \
		mc                   \
		time                 \
		htop                 \
		msmtp                \
		msmtp-mta            \
;

## PHP Extensions
RUN curl -sSLf -o /usr/local/bin/install-php-extensions \
	https://github.com/mlocati/docker-php-extension-installer/releases/latest/download/install-php-extensions && \
	chmod +x /usr/local/bin/install-php-extensions; \
	install-php-extensions \
		opcache            \
		xdebug             \
		mysqli             \
		pdo                \
		pdo_mysql          \
		redis              \
		gd                 \
		imagick            \
		tokenizer          \
		calendar           \
		bcmath             \
		mbstring           \
		exif               \
		intl               \
		zip                \
;

RUN set -ex; \
# Install Composer
# See: https://getcomposer.org/download
	wget https://getcomposer.org/download/latest-stable/composer.phar \
	&& chmod +x composer.phar \
	&& mv composer.phar /usr/local/bin/composer; \
# Install WP-CLI
# See: https://make.wordpress.org/cli/handbook/guides/installing/
# See: https://github.com/wp-cli/wp-cli/tags
	wget https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar \
	&& chmod +x wp-cli.phar \
	&& mv wp-cli.phar /usr/local/bin/wp \
	&& wget https://raw.githubusercontent.com/wp-cli/wp-cli/main/utils/wp-completion.bash \
	&& mv wp-completion.bash /usr/local/include/wp-cli-completion.bash \
	&& echo '\n\nsource /usr/local/include/wp-cli-completion.bash' >>  /etc/bash.bashrc;

## Install RootCA certificate for the CURL work correctly
COPY --chmod=644 ./certs/ROOT_CA_CERT/myRootCA.pem.crt /usr/local/share/ca-certificates/
RUN	apt-get install -y ca-certificates && update-ca-certificates

## Additional config
RUN set -ex; \
# .bashrc
	{ \
		echo '\n## Aliases'; \
		echo 'alias ll="ls -alh --color"'; \
		echo '\n## Git auto-completion'; \
		echo 'source /usr/share/bash-completion/completions/git'; \
    	echo '\n## Save the command history after each command (not only at the end of the session)'; \
		echo 'PROMPT_COMMAND="history -a; ${PROMPT_COMMAND:-}"'; \
		echo '\n## Do not save duplicate commands that follow one another'; \
		echo 'HISTCONTROL=ignoredups'; \
	} >> /etc/bash.bashrc; \
# xDebug default config
# See: https://xdebug.org/docs/all_settings
	{ \
		echo 'xdebug.mode=off'; \
		echo 'xdebug.client_host=host.docker.internal'; \
		echo 'xdebug.client_port=9003'; \
	} >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini;


WORKDIR $WORK_DIR

USER $USER

# it's just for documentation (clearity)
EXPOSE 9000
