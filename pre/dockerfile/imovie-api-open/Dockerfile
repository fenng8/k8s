FROM 192.168.100.215:5000/tomcat:8v3
MAINTAINER yinf@imovie.cn

ADD ./imovie-api-open.war /tmp/ROOT.war


WORKDIR /usr/local/tomcat

RUN rm -rf ./webapps/* &&  mv /tmp/ROOT.war ./webapps/

CMD ["./bin/catalina.sh","run"]

EXPOSE 8080
