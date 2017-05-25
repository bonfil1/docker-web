# from https://www.drupal.org/requirements/php#drupalversions
FROM php:5.6-apache

# Install some extensions as a test
RUN a2enmod rewrite

# Install the PHP extensions we need, and another packages needed like mysql client.
RUN apt-get update \
    && apt-get install -y wget \
    && apt-get install -y sendmail-bin libpng12-dev libjpeg-dev libpq-dev libmcrypt-dev vim git-core mysql-client ruby-full build-essential php5-dev netcat \
    && rm -rf /var/lib/apt/lists/* \
    && docker-php-ext-configure gd --with-png-dir=/usr --with-jpeg-dir=/usr && docker-php-ext-install gd mbstring opcache pdo pdo_mysql zip mcrypt

# Install node.
RUN curl -sL https://deb.nodesource.com/setup_4.x | bash - \
    && apt-get install -y nodejs \
    && node --version && npm version


# Apache - Copy default config
COPY hardening/apache2.conf /etc/apache2/apache2.conf

# Apache - Copy virtual hosts
COPY hardening/sites-available/001-drupal.local.conf /etc/apache2/sites-available/001-drupal.local.conf

# PHP Custom config
COPY ./php.ini /usr/local/etc/php/

# Install Composer, Drush & Drupal Console
RUN curl -sS https://getcomposer.org/installer | php \
    && mv composer.phar /usr/local/bin/composer \
    && composer global require drush/drush:8.1.9 \
    && cp -Rf /root/.composer /usr/share/composer \
    && ln -nsf /usr/share/composer/vendor/bin/drush /usr/local/bin/drush \
    && curl https://github.com/hechoendrupal/drupal-console-launcher/releases/download/1.0.0-rc16/drupal.phar -L -o drupal.phar \
    && mv drupal.phar /usr/local/bin/drupal \
    && chmod +x /usr/local/bin/drupal

# Finally start the virtual host on the server
WORKDIR /etc/apache2/sites-available

RUN a2dissite 000-default.conf \
    && a2ensite 001-drupal.local.conf

# Move hosts config to the box hosts
RUN echo '127.0.0.1  drupal.local' >> /etc/hosts

#RUN service apache2 reload
ONBUILD RUN bash -c "service apache2 reload"