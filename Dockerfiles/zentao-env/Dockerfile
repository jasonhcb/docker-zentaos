FROM ubuntu:16.04

MAINTAINER Jason<chubo.he@hand-china.com>

# Surpress Upstart errors/warning
RUN dpkg-divert --local --rename --add /sbin/initctl
RUN ln -sf /bin/true /sbin/initctl

# Install packages
ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update
RUN apt-get upgrade -y

RUN apt-get install -y supervisor \
	git \
	pwgen \
	bash-completion \
	hostname \
	wget \
	curl 

RUN cp /usr/share/zoneinfo/Europe/London /etc/localtime



#nginx install
#for compatibiltiy
ENV DOCKER_USER_ID 501
ENV DOCKER_USER_GID 20

ENV BOOT2DOCKER_ID 1000
ENV BOOT2DOCKER_GID 50

# # Tweaks to give the web service and php write permissions to the app
RUN usermod -u ${BOOT2DOCKER_ID} www-data
RUN usermod -G staff www-data

RUN groupmod -g $(($BOOT2DOCKER_GID + 10000)) $(getent group $BOOT2DOCKER_GID | cut -d: -f1)
RUN groupmod -g ${BOOT2DOCKER_GID} staff

# Install packages
ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update

RUN apt-get install -y software-properties-common \
    python-software-properties \
    language-pack-en-base

RUN LC_ALL=en_US.UTF-8 add-apt-repository ppa:ondrej/php
RUN add-apt-repository ppa:nginx/stable

RUN apt-get update
RUN apt-get upgrade -y

RUN apt-get install -y nginx

# Xdebug Remote Host IP
ENV XDEBUG_REMOTE_HOST_IP 192.168.65.1

RUN apt-get install -y php5.6-fpm \
    php5.6-mysql \
    php5.6-curl \
    php5.6-gd \
    php5.6-intl \
    php5.6-mcrypt \
    php-memcache \
    php5.6-sqlite \
    php5.6-tidy \
    php5.6-xmlrpc \
    php5.6-pgsql \
    php5.6-ldap \
    freetds-common \
    php5.6-pgsql \
    php5.6-sqlite3 \
    php5.6-json \
    php5.6-xml \
    php5.6-mbstring \
    php5.6-soap \
    php5.6-zip \
    php5.6-cli \
    php5.6-sybase \
    php5.6-odbc \
    php5.6-dev

# Install xdebug
RUN wget http://xdebug.org/files/xdebug-2.4.0.tgz
RUN tar -xvzf xdebug-2.4.0.tgz
RUN cd xdebug-2.4.0 && phpize
RUN cd xdebug-2.4.0 && ./configure
RUN cd xdebug-2.4.0 && make
RUN cd xdebug-2.4.0 && cp modules/xdebug.so /usr/lib/php/20131226
ADD php-conf/xdebug.conf /xdebug.conf
RUN cat /xdebug.conf >> /etc/php/5.6/fpm/php.ini
RUN rm -rf /xdebug-2.4.0
RUN rm -rf /xdebug-2.4.0.tgz

#tweek nginx config
RUN sed -i -e '/worker_processes/c\worker_processes  5;' /etc/nginx/nginx.conf
RUN sed -i -e '/keepalive_timeout/c\keepalive_timeout  2;' /etc/nginx/nginx.conf
RUN sed -i -e '/client_max_body_size/c\client_max_body_size 100m;' /etc/nginx/nginx.conf
RUN echo "daemon off;" >> /etc/nginx/nginx.conf

# nginx site conf
RUN mkdir -p /etc/nginx/ssl/
ADD nginx-ssl/nginx.crt /etc/nginx/ssl/nginx.crt
ADD nginx-ssl/nginx.key /etc/nginx/ssl/nginx.key

RUN rm -Rf /etc/nginx/conf.d/*
RUN rm -Rf /etc/nginx/sites-available/default
RUN rm -Rf /etc/nginx/sites-enabled/default

# Supervisor Config
ADD nginx-conf/supervisord.conf /etc/supervisord.conf

# change nginx directory
RUN mkdir -p /app/src/public
RUN mkdir -p /var/www/html/logs

# add test PHP file
ADD nginx-src/index.html /app/src/public/index.html

#tweek php-fpm config
RUN sed -i -e "s/;cgi.fix_pathinfo=1/cgi.fix_pathinfo=0/g" /etc/php/5.6/fpm/php.ini
RUN sed -i -e "s/upload_max_filesize\s*=\s*2M/upload_max_filesize = 100M/g" /etc/php/5.6/fpm/php.ini
RUN sed -i -e "s/post_max_size\s*=\s*8M/post_max_size = 100M/g" /etc/php/5.6/fpm/php.ini
RUN sed -i -e "s/;daemonize\s*=\s*yes/daemonize = no/g" /etc/php/5.6/fpm/php-fpm.conf
RUN sed -i -e "s/;catch_workers_output\s*=\s*yes/catch_workers_output = yes/g" /etc/php/5.6/fpm/pool.d/www.conf
RUN sed -i -e "s/pm.max_children = 5/pm.max_children = 9/g" /etc/php/5.6/fpm/pool.d/www.conf
RUN sed -i -e "s/pm.start_servers = 2/pm.start_servers = 3/g" /etc/php/5.6/fpm/pool.d/www.conf
RUN sed -i -e "s/pm.min_spare_servers = 1/pm.min_spare_servers = 2/g" /etc/php/5.6/fpm/pool.d/www.conf
RUN sed -i -e "s/pm.max_spare_servers = 3/pm.max_spare_servers = 4/g" /etc/php/5.6/fpm/pool.d/www.conf
RUN sed -i -e "s/pm.max_requests = 500/pm.max_requests = 200/g" /etc/php/5.6/fpm/pool.d/www.conf
RUN sed -i -e "/listen\s*=\s*\/run\/php\/php5.6-fpm.sock/c\listen = 127.0.0.1:9100" /etc/php/5.6/fpm/pool.d/www.conf
RUN sed -i -e "/pid\s*=\s*\/run/c\pid = /run/php5.6-fpm.pid" /etc/php/5.6/fpm/php-fpm.conf


#fix ownership of sock file
RUN sed -i -e "s/;listen.mode = 0660/listen.mode = 0750/g" /etc/php/5.6/fpm/pool.d/www.conf
RUN find /etc/php/5.6/cli/conf.d/ -name "*.ini" -exec sed -i -re 's/^(\s*)#(.*)/\1;\2/g' {} \;

# Supervisor Config
ADD php-conf/supervisord.conf /supervisord.conf
RUN cat /supervisord.conf >> /etc/supervisord.conf

# Start Supervisord
ADD php-scripts/start.sh /start.sh
RUN chmod 755 /start.sh

# Setup Volume
# VOLUME ["/app"]

# add test PHP file
ADD php-src/index.php /app/src/public/index.php
ADD php-src/index.php /app/src/public/info.php
RUN chown -Rf www-data.www-data /app
RUN chown -Rf www-data.www-data /var/www/html

RUN apt-get install -y unzip libaio-dev
#设置ORACLE环境变量
ENV OCI_INSTANT_PATH /usr/lib/instantclient_12_1
#安装ORACLE
RUN cd /usr/lib \
    && curl -L -o instantclient-basic-linux.x64.zip http://files.saas.hand-china.com/oracle/instantclient/instantclient-basic-linux.x64-12.1.0.2.0.zip \
    && curl -L -o instantclient-sdk-linux.x64.zip http://files.saas.hand-china.com/oracle/instantclient/instantclient-sdk-linux.x64-12.1.0.2.0.zip \
    && unzip instantclient-basic-linux.x64.zip && unzip instantclient-sdk-linux.x64.zip \
    && ln -s $OCI_INSTANT_PATH/libclntsh.so.*.1  $OCI_INSTANT_PATH/libclntsh.so \
    && ln -s $OCI_INSTANT_PATH/libclntshcore.so.*.1 $OCI_INSTANT_PATH/libclntshcore.so \
    && ln -s $OCI_INSTANT_PATH/libocci.so.*.1 $OCI_INSTANT_PATH/libocci.so \
    && rm instantclient*.zip

RUN export ORACLE_HOME=/usr/lib/instantclient_12_1
ADD oci8-2.0.8.tgz /
RUN cd /oci8-2.0.8 && phpize \
	&& ./configure --with-php-config=/usr/bin/php-config5.6 --with-oci8=instantclient,/usr/lib/instantclient_12_1 \
	&& make \
	&& make install
ADD php-conf/oci8.ini /etc/php/5.6/fpm/conf.d/oci8.ini
RUN rm -R oci8-2.0.8

# Cleanup
RUN apt-get remove --purge -y software-properties-common \
    python-software-properties

RUN apt-get autoremove -y
RUN apt-get clean
RUN apt-get autoclean

COPY nginx-conf/nginx-site.conf /etc/nginx/sites-enabled/default.conf

# Expose Ports
EXPOSE 443
EXPOSE 80
EXPOSE 9000

CMD ["/bin/bash", "/start.sh"]
