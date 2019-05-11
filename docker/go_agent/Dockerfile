FROM atmaram/go_agent_selenium_ready:latest
MAINTAINER anaik@thoughtworks.com
COPY ssh.config /home/go/.ssh/config
COPY setup_permissions.sh /docker-entrypoint.d/
RUN chmod +x /docker-entrypoint.d/setup_permissions.sh