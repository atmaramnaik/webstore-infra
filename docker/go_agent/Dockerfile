# When upping gocd-agent, check if it comes with a more recent ubuntu and adjust docker further down
# Maybe you don't need custom docker images anymore as we just want docker 1.8 or higher
FROM gocd/gocd-agent-ubuntu-16.04:v17.11.0
MAINTAINER anaik@thoughtworks.com
WORKDIR /usr/local

ARG agent
ENV WORK_DIR $agent

COPY go_agent_config /etc/default/go-agent

COPY ssh.config /home/go/.ssh/config
#This is done to allow go user to write to the /tmp folder when running gradle
RUN chmod 777 /tmp

USER root
# /go is the working directory of go agent
RUN mkdir -p /go/config
COPY autoregister.properties /go/config/autoregister.properties
RUN echo "agent.auto.register.hostname=$agent" >> /go/config/autoregister.properties
RUN chown -R go:go /go
# /home/go is the home directory of go agent
RUN chown -R go:go /home/go

# This is in accordance to : https://www.digitalocean.com/community/tutorials/how-to-install-java-with-apt-get-on-ubuntu-16-04
RUN apt-get update && \
	apt-get install -y openjdk-8-jdk && \
	apt-get install -y ant && \
	apt-get clean && \
	rm -rf /var/lib/apt/lists/* && \
	rm -rf /var/cache/oracle-jdk8-installer;

# Fix certificate issues, found as of
# https://bugs.launchpad.net/ubuntu/+source/ca-certificates-java/+bug/983302
RUN apt-get update && \
	apt-get install -y ca-certificates-java && \
	apt-get clean && \
	update-ca-certificates -f && \
	rm -rf /var/lib/apt/lists/* && \
	rm -rf /var/cache/oracle-jdk8-installer;

# Setup JAVA_HOME, this is useful for docker commandline
ENV JAVA_HOME /usr/lib/jvm/java-8-openjdk-amd64/
RUN apt-get -y update
RUN apt-get install -y software-properties-common
RUN groupadd -g 997 docker && useradd -g 997 -u 997 -b /home -G sudo -m docker && usermod -G docker go

# Setup to fetch latest docker. Relies on knowing the ubuntu version
# Remove this if docker 1.8 is available by default !
RUN apt-get install -y  apt-transport-https  ca-certificates  curl
RUN curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
RUN add-apt-repository  "deb [arch=amd64] https://download.docker.com/linux/ubuntu  $(lsb_release -cs)  stable"
RUN apt-get update
RUN apt-get install -y docker-ce=17.09.0~ce-0~ubuntu

RUN curl -L "https://github.com/docker/compose/releases/download/1.21.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
RUN chmod +x /usr/local/bin/docker-compose

RUN apt-get -y install nodejs npm \
    && npm install npm -g \
    && npm install n -g \
    && n 7.7.2 && apt-get clean

RUN npm install -g gulp bower yarn
