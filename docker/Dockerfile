FROM debian:9-slim
LABEL maintainer="Nils Vogels <n.vogels@aves-it.nl>"
ENV release_version=2.13

WORKDIR /root

RUN apt-get update && \
    apt-get install -y --no-install-recommends ca-certificates  && \
    apt-get install -y --no-install-recommends supervisor  && \
    apt-get install -y --no-install-recommends nginx && \
    apt-get install -y --no-install-recommends php7.0-fpm && \
    apt-get install -y --no-install-recommends wget && \
    apt-get clean && \
    apt-get autoclean
    
RUN wget --quiet https://github.com/filesender/filesender/archive/filesender-${release_version}.tar.gz
RUN mkdir /opt/filesender && \
    tar xzf filesender-${release_version}.tar.gz --directory /opt/filesender

# prepare nginx
RUN sed -i -e"s/keepalive_timeout 65/keepalive_timeout 2/" /etc/nginx/nginx.conf && \
sed -i -e"s/keepalive_timeout 2/keepalive_timeout 2;\n\tclient_max_body_size 100m/" /etc/nginx/nginx.conf && \
echo "daemon off;" >> /etc/nginx/nginx.conf

# prepare php-fpm
RUN  sed -i -e "s/;cgi.fix_pathinfo=1/cgi.fix_pathinfo=0/g" /etc/php/7.0/fpm/php.ini && \
sed -i -e "s/upload_max_filesize\s*=\s*2M/upload_max_filesize = 100M/g" /etc/php/7.0/fpm/php.ini && \
sed -i -e "s/post_max_size\s*=\s*8M/post_max_size = 100M/g" /etc/php/7.0/fpm/php.ini && \
sed -i -e "s/;daemonize\s*=\s*yes/daemonize = no/g" /etc/php/7.0/fpm/php-fpm.conf && \
sed -i -e "s/;catch_workers_output\s*=\s*yes/catch_workers_output = yes/g" /etc/php/7.0/fpm/pool.d/www.conf && \
sed -i -e "s/pm.max_children = 5/pm.max_children = 9/g" /etc/php/7.0/fpm/pool.d/www.conf && \
sed -i -e "s/pm.start_servers = 2/pm.start_servers = 3/g" /etc/php/7.0/fpm/pool.d/www.conf && \
sed -i -e "s/pm.min_spare_servers = 1/pm.min_spare_servers = 2/g" /etc/php/7.0/fpm/pool.d/www.conf && \
sed -i -e "s/pm.max_spare_servers = 3/pm.max_spare_servers = 4/g" /etc/php/7.0/fpm/pool.d/www.conf && \
sed -i -e "s/pm.max_requests = 500/pm.max_requests = 200/g" /etc/php/7.0/fpm/pool.d/www.conf

# prepare ownership of php-fpm socket file
RUN sed -i -e "s/;listen.mode = 0660/listen.mode = 0750/g" /etc/php/7.0/fpm/pool.d/www.conf && \
find /etc/php/7.0/cli/conf.d/ -name "*.ini" -exec sed -i -re 's/^(\s*)#(.*)/\1;\2/g' {} \;

# nginx site conf
#RUN rm -Rf /etc/nginx/conf.d/* && \
#rm -Rf /etc/nginx/sites-enabled/default && \
#rm -Rf /etc/nginx/sites-available/default && \
#mkdir -p /etc/nginx/ssl/
#ADD ./default /etc/nginx/sites-available/default
#RUN ln -s /etc/nginx/sites-available/default /etc/nginx/sites-enabled/default
#ADD ./index.php /usr/share/nginx/html/index.php
RUN chown -Rf www-data.www-data /usr/share/nginx/html/

# Config file
RUN ln -s /opt/filesender/filesender-* /opt/filesender/filesender &&\
    cp /opt/filesender/filesender/config/config_sample.php /opt/filesender/filesender/config/config.php
VOLUME /opt/filesender/filesender/config

# Supervisor Config
ADD ./supervisord.conf /etc/supervisord.conf

# Start Supervisord
ADD ./start.sh /start.sh
RUN chmod 755 /start.sh

# forward request and error logs to docker log collector
RUN ln -sf /dev/stdout /var/log/nginx/access.log \
	&& ln -sf /dev/stderr /var/log/nginx/error.log

EXPOSE 80 443

CMD ["/bin/bash", "/start.sh"]
