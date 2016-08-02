---
title: apt-cacher-ng with docker builds
date: 2016-07-20 14:05:46
tags:
    - DOCKER 
    - DOCKERFILE 
    - APT-CACHER-NG 
    - APT-GET 
    - UBUNTU 
    - CONFIGURATION 
    - DOCKER FOR MAC
---

# apt-cacher-ng with docker builds

Working with docker on a laptop testing configuration of containers has some of the same annoyances as working with other configuration managment tools and testing frameworks like test-kitchen, chef, ansible etc…

It takes forever to download dependencies. I might test some infrastructure code using vagrant or test-kitchen many many times a day and it sucks having to download apt packages from mirrors that time out or get bogged down randomly. Previously I had used a test-kitchen plugin to deal with that which I’ll cover in another post. In this I’m going to go over some simple hacks I’ve managed to get working for speeding up building of docker images using docker for mac beta.

See this forum thread in regards to the ongoing headaches with networking in docker for mac beta. Link

I was trying a bunch of ways to do this and just settled on the alias on lo0.
I just added this to my bash alias.

``` bash
ifconfig lo0 |grep 192.168.168.167 >/dev/null
if [ $? -ne 0 ]; then
  sudo ifconfig lo0 alias 192.168.168.167
else
  echo "docker host alias is in good shape move along."
fi
```

My use case was that I am running apt-cacher-ng on my mac natively. Using just a brew install apt-cacher-ng and running it as a service all the time. This speeds up test-kitchen and other configuration management tool testing.

Adding this.

``` bash
ARG APT_PROXY_PORT=
ARG HOST_IP=
COPY detect-apt-proxy.sh /root
RUN /root/detect-apt-proxy.sh ${APT_PROXY_PORT}
```


The contents of detect-apt-proxy.sh are as follows.

``` bash
#!/bin/bash
set -x
APT_PROXY_PORT=$1
if [ -z "${HOST_IP}" ]; then
  HOST_IP=$(route -n | awk '/^0.0.0.0/ {print $2}')
fi
nc -z "$HOST_IP" ${APT_PROXY_PORT}
if [ $? -eq 0 ]; then
    cat >> /etc/apt/apt.conf.d/30proxy <<EOL
    Acquire::http::Proxy "http://$HOST_IP:$APT_PROXY_PORT";
    Acquire::http::Proxy::ppa.launchpad.net DIRECT;
EOL
    cat /etc/apt/apt.conf.d/30proxy
    echo "Using host's apt proxy"
else
    echo "No apt proxy detected on Docker host"
fi
```

Then when building my docker images…

``` bash 
docker build -t newcontainer --build-arg APT_PROXY_PORT=3142 --build-arg HOST_IP=192.168.168.167 .
```

This allows me to run the cacher locally and programmatically check for it inside the container and set the apt proxy options. This also works on Linux or docker-machine just leave off the build-arg for HOST_IP and it will use the older method of grabbing the gateway interface instead of the aliased interface…. This should in theory work with xhyve but the gateway interface is somehow hidden or locked down from the vm… Some insight on why you can’t access the gateway interface in xhyve vs virtualbox would be nice…