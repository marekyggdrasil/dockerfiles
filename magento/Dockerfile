#
# Dockerfile for magento
#

FROM php:5.6-apache
MAINTAINER kev<noreply@easypi.info>

RUN a2enmod rewrite

RUN set -xe \
    && apt-get update \
    && apt-get install -y libcurl3-dev \
                          libfreetype6 \
                          libfreetype6-dev \
                          libjpeg62-turbo \
                          libjpeg62-turbo-dev \
                          libmcrypt-dev \
                          libmcrypt4 \
                          libpng12-dev \
                          libpng12-dev \
                          libxslt-dev \
                          zziplib-bin \
                          zlib1g-dev \
                          libicu-dev \
                          g++ \
                          cron \
    && docker-php-ext-configure gd --with-png-dir=/usr --with-jpeg-dir=/usr \
    && docker-php-ext-configure intl \
    && docker-php-ext-install gd mcrypt mbstring pdo_mysql \
    && docker-php-ext-install intl \
    && docker-php-ext-install xsl \
    && docker-php-ext-install zip \
    && apt-get purge -y --auto-remove libcurl3-dev \
                                      libfreetype6-dev \
                                      libjpeg62-turbo-dev \
                                      libmcrypt-dev \
                                      libpng12-dev \
                                      libpng12-dev \
    && rm -rf /var/lib/apt/lists/*
RUN echo 'always_populate_raw_post_data = -1\nmax_execution_time = 240\nmax_input_vars = 1500\nupload_max_filesize = 32M\npost_max_size = 32M\nmemory_limit = 2048M' > /usr/local/etc/php/conf.d/typo3.ini
WORKDIR /usr/src
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
WORKDIR /var/www/html
ENV MAGENTO_MD5 a1bf0c943e831dbd580c7c9fc20ff090
ENV MAGENTO_URL https://s3.amazonaws.com/aftership.testing.s3/ask-nick-or-marek-for-docker-business-team-to-test/Magento-CE-2.1.0-2016-06-23-02-28-19.tar.gz
ENV MAGENTO_FILE magento.tar.gz
RUN curl -sSL ${MAGENTO_URL} -o ${MAGENTO_FILE} \
    && echo "${MAGENTO_MD5}  ${MAGENTO_FILE}" | md5sum -c \
    && tar xzf ${MAGENTO_FILE} \
    && rm ${MAGENTO_FILE} \
    && chown -R www-data:www-data . \
    && /etc/init.d/apache2 restart \
    && composer install
WORKDIR /var/www
RUN echo "* * * * * /usr/local/bin/php /var/www/html/bin/magento cron:run | grep -v "Ran jobs by schedule" >> /var/www/html/var/log/magento.cron.log" >> mycron\
    && echo "* * * * * /usr/local/bin/php /var/www/html/update/cron.php >> /var/www/html/var/log/update.cron.log" >> mycron \
    && echo "* * * * * /usr/local/bin/php /var/www/html/bin/magento setup:cron:run >> /var/www/html/var/log/setup.cron.log" >> mycron \
    && crontab -u www-data mycron \
    && rm mycron

COPY script.sh .
RUN chown www-data:www-data script.sh
ENTRYPOINT ./script.sh
