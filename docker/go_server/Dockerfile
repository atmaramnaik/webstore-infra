FROM gocd/gocd-server:v19.3.0
MAINTAINER anaik@thoughtworks.com

USER go
#/home/go is the home directory of go server
COPY ssh.config /home/go/.ssh/config

USER root
ADD .secrets/htpasswd /etc/
RUN chmod 0644 /etc/htpasswd