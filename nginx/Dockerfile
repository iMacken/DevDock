FROM nginx:latest

MAINTAINER Macken <rystlee100@gmail.com>

ADD nginx.conf /etc/nginx/
COPY sites/*.conf /etc/nginx/sites-available/

RUN echo "upstream php-upstream { server php-fpm:9000; }" > /etc/nginx/conf.d/upstream.conf

RUN usermod -u 1000 www-data

CMD ["nginx"]

EXPOSE 80 443
