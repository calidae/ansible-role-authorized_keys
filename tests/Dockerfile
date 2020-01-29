FROM ubuntu
RUN apt-get update && apt-get install --assume-yes openssh-server
RUN mkdir /run/sshd
CMD /usr/sbin/sshd -De
