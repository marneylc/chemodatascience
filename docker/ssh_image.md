### Create DockerFile
```bash
FROM ubuntu:16.04

RUN apt-get update && apt-get install -y openssh-server
RUN mkdir /var/run/sshd
RUN echo 'root:screencast' | chpasswd
RUN sed -i 's/PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
```

### SSH login fix. Otherwise user is kicked off after login - from https://docs.docker.com/engine/examples/running_ssh_service/
```bash
RUN sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd

ENV NOTVISIBLE "in users profile"
RUN echo "export VISIBLE=now" >> /etc/profile

EXPOSE 22
CMD ["/usr/sbin/sshd", "-D"]
```

Build the image and run:
```bash
$ docker build -t eg_sshd .
docker run -d -P --name test_sshd eg_sshd
docker port test_sshd 22
```

ssh as root 
```
ssh root@192.168.1.2 -p 49154
# The password is ``screencast``
```
