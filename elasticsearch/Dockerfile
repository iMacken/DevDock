FROM elasticsearch:2.3.5

MAINTAINER Macken <rystlee100@gmail.com>

ADD plugins/ik/elasticsearch-analysis-ik-1.9.5.zip /tmp/

RUN unzip /tmp/elasticsearch-analysis-ik-1.9.5.zip -d /usr/share/elasticsearch/plugins/ik

COPY config /usr/share/elasticsearch/config

EXPOSE 9200 9300