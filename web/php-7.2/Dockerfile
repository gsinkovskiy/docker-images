FROM phusion/baseimage

LABEL maintainer="Hlib Synkovskyi <gleb@itworks-soft.com>"

#RUN rm -f /etc/service/sshd/down
ARG DEBIAN_FRONTEND=noninteractive

# Install VCSs
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
  git \
  subversion \
  && rm -rf /var/lib/apt/lists/* && rm -rf /var/cache/apk/* && rm -rf /tmp/*

# Install common software
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
    mysql-client \
    postgresql-client \
    curl \
    wget \
    mc \
    htop \
    screen \
    iputils-ping \
    sudo \
    vim \
 && rm -rf /var/lib/apt/lists/* && rm -rf /var/cache/apk/* && rm -rf /tmp/*

# Install NodeJS
RUN curl -sL https://deb.nodesource.com/setup_6.x | bash - && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends nodejs build-essential \
 && rm -rf /var/lib/apt/lists/* && rm -rf /var/cache/apk/* && rm -rf /tmp/*

# Install Yarn
RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - \
 && echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list \
 && apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends yarn \
 && rm -rf /var/lib/apt/lists/* && rm -rf /var/cache/apk/* && rm -rf /tmp/*

ENV PHP_VERSION 7.2
ENV PHP_INI_DIR /etc/php/$PHP_VERSION

RUN add-apt-repository ppa:certbot/certbot \
 && apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
    python-certbot-apache \
 && rm -rf /var/lib/apt/lists/* && rm -rf /var/cache/apk/* && rm -rf /tmp/*

# Install PHP and extensions
RUN add-apt-repository ppa:ondrej/php \
 && apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
    apache2 \
    apache2-utils \
    ssl-cert \
    libapache2-mod-php$PHP_VERSION \
    php$PHP_VERSION \
    php$PHP_VERSION-cli \
    php$PHP_VERSION-common \
    php$PHP_VERSION-dev \
    php$PHP_VERSION-gd \
    php$PHP_VERSION-imagick \
    php$PHP_VERSION-mysql \
    php$PHP_VERSION-pgsql \
    php$PHP_VERSION-curl \
    php$PHP_VERSION-intl \
    php$PHP_VERSION-xsl \
    php$PHP_VERSION-mbstring \
    php$PHP_VERSION-zip \
    php$PHP_VERSION-bcmath \
    php$PHP_VERSION-iconv \
    php$PHP_VERSION-imap \
    php$PHP_VERSION-apcu \
    php$PHP_VERSION-soap \
    php$PHP_VERSION-redis \
    php$PHP_VERSION-ldap \
    php$PHP_VERSION-mailparse \
    php-xdebug \
 && a2enmod ssl rewrite headers include proxy alias \
 && a2ensite default-ssl \
 && rm -rf /var/lib/apt/lists/* && rm -rf /var/cache/apk/* && rm -rf /tmp/*

# Oracle-client
RUN apt-get update \
 && apt-get -y install unzip php7.2-dev php-pear build-essential libaio1 \
 && rm -rf /var/lib/apt/lists/* && rm -rf /var/cache/apk/* && rm -rf /tmp/*
RUN mkdir /opt/oracle
RUN cd /opt/oracle && wget -c https://bitbucket.org/msyniukitworks/download/downloads/instantclient-sdk-linux.x64-12.1.0.2.0.zip && \
 unzip instantclient-sdk-linux.x64-12.1.0.2.0.zip && \
 wget -c https://bitbucket.org/msyniukitworks/download/downloads/instantclient-basic-linux.x64-12.1.0.2.0.zip && \
 unzip instantclient-basic-linux.x64-12.1.0.2.0.zip && \
 wget -c https://bitbucket.org/msyniukitworks/download/downloads/instantclient-jdbc-linux.x64-12.1.0.2.0.zip && \
 unzip instantclient-jdbc-linux.x64-12.1.0.2.0.zip &&\
 ln -s /opt/oracle/instantclient_12_1/libclntsh.so.12.1 /opt/oracle/instantclient_12_1/libclntsh.so && \
 ln -s /opt/oracle/instantclient_12_1/libocci.so.12.1 /opt/oracle/instantclient_12_1/libocci.so && \
 echo /opt/oracle/instantclient_12_1 > /etc/ld.so.conf.d/oracle-instantclient
RUN ldconfig
RUN sudo sh -c "echo 'instantclient,/opt/oracle/instantclient_12_1' | pecl install oci8"
RUN echo "extension = oci8.so" >> /etc/php/7.2/mods-available/oci.ini
RUN phpenmod oci
RUN php -m | grep 'oci8'

# Install Blackfire
RUN version=$(php -r "echo PHP_MAJOR_VERSION.PHP_MINOR_VERSION;") \
 && curl -A "Docker" -o /tmp/blackfire-probe.tar.gz -D - -L -s https://blackfire.io/api/v1/releases/probe/php/linux/amd64/$version \
 && tar zxpf /tmp/blackfire-probe.tar.gz -C /tmp \
 && mv /tmp/blackfire-*.so $(php -r "echo ini_get('extension_dir');")/blackfire.so \
# && printf "extension=blackfire.so\nblackfire.agent_socket=tcp://blackfire:8707\n" > $PHP_INI_DIR/apache2/conf.d/blackfire.ini \
 && mkdir -p /tmp/blackfire \
 && curl -A "Docker" -L https://blackfire.io/api/v1/releases/client/linux_static/amd64 | tar zxp -C /tmp/blackfire \
 && mv /tmp/blackfire/blackfire /usr/bin/blackfire \
 && rm -Rf /tmp/*

# Install PHPCS
RUN curl -sL https://squizlabs.github.io/PHP_CodeSniffer/phpcs.phar -o /usr/local/bin/phpcs && chmod +x /usr/local/bin/phpcs
RUN curl -sL https://squizlabs.github.io/PHP_CodeSniffer/phpcbf.phar -o /usr/local/bin/phpcbf && chmod +x /usr/local/bin/phpcbf

# Install Composer
ENV COMPOSER_VERSION 1.7.3
RUN curl -sL https://getcomposer.org/download/$COMPOSER_VERSION/composer.phar -o /usr/local/bin/composer && chmod +x /usr/local/bin/composer

# Install PHPUnit
ENV PHPUNIT_VERSION 6.4
RUN curl -sL https://phar.phpunit.de/phpunit-$PHPUNIT_VERSION.phar -o /usr/local/bin/phpunit && chmod +x /usr/local/bin/phpunit

#COPY etc/my_init.d/01_start_sshd.sh /etc/my_init.d/01_start_sshd.sh
#RUN chmod +x /etc/my_init.d/01_start_sshd.sh

ENV CONTAINER_USER dev
ENV CONTAINER_GROUP $CONTAINER_USER
ENV HOME /home/$CONTAINER_USER

RUN groupadd $CONTAINER_GROUP -g 1000 \
 && useradd -u 1000 --home $HOME --create-home --shell /bin/bash -g $CONTAINER_GROUP $CONTAINER_USER \
#    echo "abc:x:${uid}:${gid}:Abc,,,:/home/abc:/bin/bash" >> /etc/passwd && \
#    echo "abc:x:${uid}:" >> /etc/group && \
 && echo "$CONTAINER_USER ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/$CONTAINER_USER \
 && chmod 0440 /etc/sudoers.d/$CONTAINER_USER

ENV APACHE_RUN_USER $CONTAINER_USER
ENV APACHE_RUN_GROUP $CONTAINER_GROUP

RUN mkdir -p /var/www/html && chown $CONTAINER_USER:$CONTAINER_GROUP -R /var/www/html

#USER $CONTAINER_USER

ENV PASSWORD ""
ENV COMPOSER_AUTH ""

ENV PHP_CONFIG ""
ENV PHP_CONFIG_CLI ""
ENV PHP_CONFIG_WEB ""

COPY etc/php/* /etc/php/$PHP_VERSION/apache2/conf.d/
COPY etc/php/* /etc/php/$PHP_VERSION/cli/conf.d/

COPY usr/local/bin/* /usr/local/bin/
RUN chmod +x /usr/local/bin/docker-entrypoint.sh /usr/local/bin/run-apache.sh

VOLUME ["/var/www", "/var/log/apache2"]
WORKDIR /var/www/html

EXPOSE 80 443

ENTRYPOINT ["/sbin/my_init", "--", "setuser", "dev", "/usr/local/bin/docker-entrypoint.sh"]
CMD ["run-apache.sh"]
