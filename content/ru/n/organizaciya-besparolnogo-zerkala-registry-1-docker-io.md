---
title: "Организация беспарольного зеркала registry-1.docker.io"
date: 2021-03-05
weight: 10
description: >
  Использовать Nexus Repository Manager для организации зеркала docker.io не слишком удобно из-за обязательного применения 'docker login' перед использованием такого репозитория, поэтому запускаем зеркало с помощью официального docker-образа.
tags:
  - Docker
  - Nexus Repository Manager
---

2021-03-05

https://docs.docker.com/registry/deploying/

Машине, где установлен Nexus Repository Manager, для его работы предоставлен выход в инет. Использовать сам NXRM для организации зеркала docker.io не слишком удобно из-за обязательного применения 'docker login' перед использованием такого репозитория, поэтому запускаем зеркало:
```
docker run -d -p 6000:5000 \
-e REGISTRY_PROXY_REMOTEURL=https://registry-1.docker.io \
 --restart always \
 --name registry registry:2
```

На целевой машине добавляем в /etc/docker/daemon.json запись:
```
{
  "registry-mirrors": ["http://nexus.example.org:6000"]
}
```

После перезапуска docker, стало возможным, без дополнительных 'docker login', сразу выполнять:
```
docker pull alpine
```
