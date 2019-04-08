FROM mysql:5.7

ADD startup /etc/mysql/startup

RUN chown -R mysql:root /var/lib/mysql/

CMD ["mysqld"]

EXPOSE 3306
