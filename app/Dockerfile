FROM ubuntu:18.04

LABEL maintainer="Chris Fidao"

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update \
    && apt-get install -y gnupg tzdata \
    && echo "UTC" > /etc/timezone \
    && dpkg-reconfigure -f noninteractive tzdata

RUN apt-get update \
    && apt-get install -y curl zip unzip git supervisor sqlite3 \
       nginx php7.2-fpm php7.2-cli \
       php7.2-pgsql php7.2-sqlite3 php7.2-gd \
       php7.2-curl php7.2-memcached \
       php7.2-imap php7.2-mysql php7.2-mbstring \
       php7.2-xml php7.2-zip php7.2-bcmath php7.2-soap \
       php7.2-intl php7.2-readline php7.2-xdebug \
       php-msgpack php-igbinary \
    && php -r "readfile('http://getcomposer.org/installer');" | php -- --install-dir=/usr/bin/ --filename=composer \
    && mkdir /run/php \
    && apt-get -y autoremove \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* \
    && echo "daemon off;" >> /etc/nginx/nginx.conf

RUN ln -sf /dev/stdout /var/log/nginx/access.log \
    && ln -sf /dev/stderr /var/log/nginx/error.log

ADD default /etc/nginx/sites-available/default
ADD supervisord.conf /etc/supervisor/conf.d/supervisord.conf
ADD php-fpm.conf /etc/php/7.2/fpm/php-fpm.conf
ADD start-container /usr/bin/start-container
RUN chmod +x /usr/bin/start-container

#
# fixuid finally resolved my Docker permissions problems. 
#
# https://boxboat.com/2017/07/25/fixuid-change-docker-container-uid-gid/
#
RUN addgroup --gid 1000 docker && \
    adduser --uid 1000 --ingroup docker --home /home/docker --shell /bin/sh --disabled-password --gecos "" docker

# What parts of these usrmods are really doing anything? 

# Add root to docker 
RUN usermod -aG sudo docker 

# Add docker to root
RUN usermod -aG docker root 

# Add www-data user to docker group.
RUN usermod -aG www-data docker 

# Add docker user to www-data group.
RUN usermod -aG docker www-data 

RUN USER=docker \
    && GROUP=docker \
    && curl -SsL https://github.com/boxboat/fixuid/releases/download/v0.4/fixuid-0.4-linux-amd64.tar.gz | tar -C /usr/local/bin -xzf - \
    && chown root:root /usr/local/bin/fixuid \
    && chmod 4755 /usr/local/bin/fixuid \
    && mkdir -p /etc/fixuid \
    && printf "user: $USER\ngroup: $GROUP\n" > /etc/fixuid/config.yml    

#RUN chown --recursive 1000:1000 /var/www/html    
RUN chown --recursive www-data:www-data /var/www



WORKDIR /var/www/html

ENTRYPOINT ["start-container"]



