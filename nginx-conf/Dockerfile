################################################################################
# Base image
################################################################################

FROM nginx
#FROM ubuntu:vivid

################################################################################
# Build instructions
################################################################################

# Remove default nginx configs.
RUN rm -f /etc/nginx/conf.d/*

# Install packages
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -my --no-install-recommends \
  supervisor \
  apt-utils \
  curl \
  wget \
  git \
  nano \
#  vim-tiny \
#  optipng \
#  phpmyadmin \
  zip \
  htop \
  php5-curl \
  php5-cli \
  php5-fpm \
  php5-gd \
  php5-memcached \
  php5-mysql \
  php5-sqlite \
  php5-xdebug \
  mysql-client

RUN apt-get clean

# Setup PHPMyAdmin
#RUN echo -e "\n# Include PHPMyAdmin configuration\nInclude /etc/phpmyadmin/apache.conf\n" >> /etc/apache2/apache2.conf
#RUN sed -i -e "s/\/\/ \$cfg\['Servers'\]\[\$i\]\['AllowNoPassword'\]/\$cfg\['Servers'\]\[\$i\]\['AllowNoPassword'\]/g" /etc/phpmyadmin/config.inc.php


# Ensure that PHP5 FPM is run as root.
RUN sed -i "s/user = www-data/user = root/" /etc/php5/fpm/pool.d/www.conf
RUN sed -i "s/group = www-data/group = root/" /etc/php5/fpm/pool.d/www.conf

# Install HHVM
#RUN apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0x5a16e7281be7a449
#RUN echo deb http://dl.hhvm.com/debian jessie main | tee /etc/apt/sources.list.d/hhvm.list
#RUN apt-get update && apt-get install -y hhvm

# Add configuration files
COPY conf/nginx.conf /etc/nginx/
COPY conf/supervisord.conf /etc/supervisor/conf.d/
COPY conf/php.ini /etc/php5/fpm/conf.d/40-custom.ini
COPY conf/php.ini /etc/php5/cli/conf.d/40-custom.ini
COPY drush/drushrc.php /root/.drush/drushrc.php

# Install Composer
RUN curl -sS https://getcomposer.org/installer | php \
    && mv composer.phar /usr/local/bin/composer

# Install latest Drush
RUN composer global require drush/drush:dev-master \
    && composer global update
# Unfortunately, adding the composer vendor dir to the PATH doesn't seem to work. So:
RUN ln -s /root/.composer/vendor/bin/drush /usr/local/bin/drush

# Goto temporary directory.
#WORKDIR /tmp

# Run composer and phpunit installation.
#RUN composer selfupdate && \
#    composer require "phpunit/phpunit:~5.0.8" --prefer-source --no-interaction && \
#    ln -s /tmp/vendor/bin/phpunit /usr/local/bin/phpunit

# Install PHP Code sniffer
#RUN curl -OL https://squizlabs.github.io/PHP_CodeSniffer/phpcs.phar \
#    && chmod 755 phpcs.phar \
#    && mv phpcs.phar /usr/local/bin/phpcs \
#    && curl -OL https://squizlabs.github.io/PHP_CodeSniffer/phpcbf.phar \
#    && chmod 755 phpcbf.phar \
#    && mv phpcbf.phar /usr/local/bin/phpcbf

# Setup Blackfire.
# Get the sources and install the Debian packages.
# We create our own start script. If the environment variables are set, we
# simply start Blackfire in the foreground. If not, we create a dummy daemon
# script that simply loops indefinitely. This is to trick Supervisor into
# thinking the program is running and avoid unnecessary error messages.
#RUN wget -O - https://packagecloud.io/gpg.key | apt-key add -
#RUN echo "deb http://packages.blackfire.io/debian any main" > /etc/apt/sources.list.d/blackfire.list
#RUN apt-get update \
#    && apt-get install -y blackfire-agent blackfire-php
#RUN echo -e '#!/bin/bash\n\
#if [[ -z "$BLACKFIREIO_SERVER_ID" || -z "$BLACKFIREIO_SERVER_TOKEN" ]]; then\n\
#    while true; do\n\
#        sleep 1000\n\
#    done\n\
#else\n\
#    /usr/bin/blackfire-agent -server-id="$BLACKFIREIO_SERVER_ID" -server-token="$BLACKFIREIO_SERVER_TOKEN"\n\
#fi\n\
#' > /usr/local/bin/launch-blackfire
#RUN chmod +x /usr/local/bin/launch-blackfire
#RUN mkdir -p /var/run/blackfire

################################################################################
# Volumes
################################################################################

VOLUME ["/var/www", "/etc/nginx/conf.d", "/root/db.dumps", "/etc/mysql/conf.d"]

#WORKDIR /var/www

################################################################################
# Ports
################################################################################

EXPOSE 80 443 9000

################################################################################
# Entrypoint
################################################################################

ENTRYPOINT ["/usr/bin/supervisord"]
