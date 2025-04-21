FROM docker:dind

RUN apk add curl wget shadow openjdk17  openssh

RUN ssh-keygen -A

COPY jenkins.home.arpa.pub /root/.ssh/authorized_keys

COPY dind/daemon.json /etc/docker/daemon.json

CMD ["/bin/sh" , "-c" , "/usr/sbin/sshd && /usr/local/bin/dockerd-entrypoint.sh"]