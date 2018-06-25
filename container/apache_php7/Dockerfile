FROM php:7.2-apache-stretch

# timezone / date   
RUN echo "Asia/Shanghai" > /etc/timezone && dpkg-reconfigure -f noninteractive tzdata

RUN echo "deb http://mirrors.163.com/debian/ buster main contrib  \\n \
          deb-src http://mirrors.163.com/debian/ buster main contrib \\n \
          deb http://ftp.cn.debian.org/debian/ buster main contrib  \\n \
          deb-src http://ftp.cn.debian.org/debian/ buster main contrib \\n \
          deb http://repo.mysql.com/apt/debian/ jessie mysql-5.7" > /etc/apt/sources.list

# install packages
RUN apt-get update -y && \
  apt-get install -y --no-install-recommends --allow-unauthenticated \
  less vim wget unzip rsync git mysql-client \
  libcurl3 libfreetype6 libjpeg62-turbo libpng-dev libjpeg-dev libxml2-dev libwebp6 libxpm4 && \
  apt-get clean && \
  apt-get autoremove -y && \
  rm -rf /var/lib/apt/lists/* && \
  echo "export TERM=xterm" >> /root/.bashrc

# install php extensions
RUN docker-php-ext-configure gd --with-jpeg-dir=/usr/local/ && \
  docker-php-ext-install -j$(nproc) zip bcmath soap pdo_mysql gd

# composer stuff
RUN php -r 'readfile("https://getcomposer.org/installer");' > composer-setup.php \
  && php composer-setup.php --install-dir=/usr/local/bin --filename=composer \
  && rm -f composer-setup.php \
  && chown www-data:www-data /var/www

# prepare entrypoint.d pattern
RUN mkdir /entrypoint.d
COPY ./files/entrypoint.sh /entrypoint.sh
COPY ./entrypoint.d/* /entrypoint.d/
RUN chmod 777 /entrypoint.sh && chmod 777 /entrypoint.d/*

# apache stuff
RUN /usr/sbin/a2enmod rewrite && /usr/sbin/a2enmod headers && /usr/sbin/a2enmod expires
COPY ./files/000-default.conf /etc/apache2/sites-available/000-default.conf

# oxid stuff
COPY ./files/config.inc.php /var/www/config.inc.php

ENTRYPOINT ["/entrypoint.sh"]
CMD ["apache2-foreground"]

