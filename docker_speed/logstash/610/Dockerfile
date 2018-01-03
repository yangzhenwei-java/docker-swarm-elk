#  安装openjdk

FROM docker.elastic.co/logstash/logstash:6.1.0


RUN  cd /usr/share/logstash/ && curl -O http://geolite.maxmind.com/download/geoip/database/GeoLite2-City.mmdb.gz && \
     gunzip GeoLite2-City.mmdb.gz

