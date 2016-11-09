---
layout: post
title: "Docker Guide"
description: "Virtual container"
catagory: tech
tags: virtualization
---

#Install

```Bash:

* sudo apt-get install docker.io  
  sudo ln -sf /usr/bin/docker.io /usr/local/bin/docker  

* wget -qO- https://get.docker.com/ | sh  

```
  check:

```Bash:
  sudo service docker.io status
```
  Add user to docker group:

```Bash:
sudo usermod -aG docker onos
```

#Command

```Bash:
docker images
docker kill $(docker ps -q) ; 
docker rm $(docker ps -a -q) ;   
docker rmi $(docker images -q -a) 
docker run ubuntu:14.04 /bin/echo 'Hello world'
$ sudo docker run -d ubuntu:14.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"
docker ps 
docker attach
```

#Q&A

+ Unable to download docker image

```Bash:
sudo docker pull dl.dockerpool.com:5000/ubuntu:14.04
sudo docker pull dl.dockerpool.com:5000/ubuntu:14.04
```
[Reference](http://dockerpool.com/article/1413082493) 


+ Use proxy and dns

```Bash:
http_proxy=111.161.126.101:80 docker -d --dns=114.114.114.114 &
```

+ Use nsenter to login container

- Install 

```Bash:
cd /tmp
curl https://www.kernel.org/pub/linux/utils/util-linux/v2.24/util-linux-2.24.tar.gz \
  | tar -zxf-
cd util-linux-2.24
./configure --without-ncurses
make nsenter
cp nsenter /usr/local/bin
```

- docker-enter.sh

```Bash:
#!/bin/sh

  if [ -e $(dirname "$0")/nsenter ]; then
    # with boot2docker, nsenter is not in the PATH but it is in the same folder
    NSENTER=$(dirname "$0")/nsenter
  else
    NSENTER=nsenter
  fi

  if [ -z "$1" ]; then
    echo "Usage: `basename "$0"` CONTAINER [COMMAND [ARG]...]"
    echo ""
    echo "Enters the Docker CONTAINER and executes the specified COMMAND."
    echo "If COMMAND is not specified, runs an interactive shell in CONTAINER."
  else
    PID=$(docker inspect --format "{{.State.Pid}}" "$1")
    if [ -z "$PID" ]; then
      exit 1
    fi
    shift

    OPTS="--target $PID --mount --uts --ipc --net --pid --"

    if [ -z "$1" ]; then
      # No command given.
      # Use su to clear all host environment variables except for TERM,
      # initialize the environment variables HOME, SHELL, USER, LOGNAME, PATH,
      # and start a login shell.
      "$NSENTER" $OPTS su - root
    else
      # Use env to clear all host environment variables.
      "$NSENTER" $OPTS env --ignore-environment -- "$@"
    fi
  fi
```


#Export image

```Bash:
docker export 6c5563 > ./onos_docker.tar.gz
docker import 
docker import onos_docker.tar.gz docker/onos
docker run -d -i -t docker/onos /bin/bash
```
#Link

    http://dockerpool.com/article/1413082493
    http://blog.csdn.net/junjun16818/article/details/30696295


