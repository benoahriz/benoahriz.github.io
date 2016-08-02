---
title: Docker Hacks
date: 2016-07-18 14:09:25
tags:
    - docker
    - hacks
---

I've found this snippet handy when you need to restart the docker daemon and want to resume currently running containers.


``` bash
docker ps |tail -n +2 > /docker.running
cat /docker.running
service docker restart
docker start $(awk '{print $1}' /docker.running)
```

