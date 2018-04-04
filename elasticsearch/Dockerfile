FROM docker.elastic.co/elasticsearch/elasticsearch:6.2.3

MAINTAINER Macken <rystlee100@gmail.com>

ADD config /usr/share/elasticsearch/config

RUN ./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v6.2.3/elasticsearch-analysis-ik-6.2.3.zip

EXPOSE 9200 9300

