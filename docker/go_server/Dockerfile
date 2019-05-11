FROM gocd/gocd-server:v19.3.0
MAINTAINER anaik@thoughtworks.com

USER go
#/home/go is the home directory of go server
COPY ssh.config /home/go/.ssh/config

USER root
ADD .secrets/htpasswd /etc/
RUN chmod 0644 /etc/htpasswd
RUN addgroup --gid 997 docker && adduser --uid 997 --home /home -S docker docker && addgroup go docker
RUN apk add docker
RUN apk add openrc
RUN apk update
RUN rc-update add docker boot
RUN mkdir -p /gradle/jars
RUN mkdir -p /gradle/wrapper
RUN chown -R 1000:1000 /gradle
RUN chmod -R 777 /gradle