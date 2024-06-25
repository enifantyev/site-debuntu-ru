---
title: "Void Linux Установка Docker"
date: 2020-02-22
weight: 10
description: >
  Void Linux Установка Docker
tags:
  - Docker
  - Void Linux
  - Virtualization
---

2020-02-22

### Установка требуемых пакетов
```
# xbps-install -Su docker
Name       Action    Version           New version            Download size
runc       install   -                 1.0.0_12               4084KB
containerd install   -                 1.3.2_1                30MB
docker     install   -                 19.03.6_2              37MB
```

```
ln -s /etc/sv/containerd /var/service
ln -s /etc/sv/docker /var/service
```

После запуска служб:
- создаётся известный каталог и в нём файл `/etc/docker/key.json`;
- в систему добавляется виртуальный сетевой интерфейс `docker0`;
- в `netfilter` добавляются правила, обеспечивающие будущие docker-контейнеры доступом к физической сети через NAT.
- в систему добавляется группа `docker`, обеспечивающая своим участникам запуск контейнеров, что может быть ими использовано для получения root-привилегий на хосте. Смотри [Docker Daemon Attack Surface](https://docs.docker.com/engine/security/security/#docker-daemon-attack-surface) для подробностей.

Написано:
> Adding a user to the “docker” group grants them the ability to run containers which can be used to obtain root privileges on the Docker host. Refer to [Docker Daemon Attack Surface](https://docs.docker.com/engine/security/security/#docker-daemon-attack-surface) for more information.

Ещё заметка:
> To install Docker without root privileges, see [Run the Docker daemon as a non-root user (Rootless mode)](https://docs.docker.com/engine/security/rootless/).

> Rootless mode is currently available as an experimental feature.
