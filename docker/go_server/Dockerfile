FROM gocd/gocd-server:v17.11.0
MAINTAINER anaik@thoughtworks.com
WORKDIR /usr/local

# /go-working-dir is the working directory of go server
RUN mkdir -p /go-working-dir/config
RUN chown -R go:go /go-working-dir

USER go
#/home/go is the home directory of go server
COPY ssh.config /home/go/.ssh/config
COPY cruise-config.xml /go-working-dir/config/cruise-config.xml

USER root

RUN chown -R go:go /go-working-dir
ADD .secrets/htpasswd /etc/
RUN chmod 0644 /etc/htpasswd
