---
title: "Использование прокси-сервер Approx для локального кэширования APT-репозиториев"
date: 2011-03-30
weight: 10
description: >
  Использование прокси-сервер Approx для локального кэширования APT-репозиториев
tags:
  - Approx
  - APT
---

2011-03-30
`Approx` -- это кэширующий прокси-сервер, установленный на одном компьютере и слушающий 9999 порт. На всех компьютерах настроены `/etc/apt/sources.list` на использование этого approx'а.

Вот фрагмент клиентского sources.list:
```
deb http://approx.local.net:9999/debian1 stable main
```

А вот фрагмент настройки аппрокса `/etc/approx/approx.conf`:
```
debian1 http://ftp.debian.org/debian
```

В результате, когда клиент обращается на `http://approx.local.net:9999/debian1 stable main`, то Approx сервер видит ключевое слово ***debian1*** и подставляет вместо него `http://ftp.debian.org/debian` (получается `http://ftp.debian.org/debian stable main`) и скачивает необходимый пакет в свой кэш. После чего выдаёт его всем запросившим.

На одной машине в сетке устанавливаем Approx:
```
# aptitude install approx
```
В отдельной папке `/var/cache/approx` будут храниться скаченные
пакеты.

Перенастраиваем `/etc/approx/approx.conf` на обслуживание локальных дебианов и убунт и перенастраиваем клиентов. Радуемся.

> Была проблема. Создал раздел на винте для кэша аппрокса. Перенёс туда существующие директории кэша (для дебиана) и прописал в `/etc/fstab` автоматическое подцепление этого раздела в каталог `/var/cache/approx`. Создал машину на Ubuntu и не смог её обновить через Approx. Включил в Approx дебаггинг и увидел, что Approx не мог создать новые каталоги в кэше для клиента убунты. Не хватало прав. Назначил права и всё заработало.

###### Содержимое `/etc/approx/approx.conf` на сервере:

<details><p>

```
# Here are some examples of remote repository mappings.
# See http://www.debian.org/mirror/list for mirror sites.

debian http://ftp.debian.org/debian
debian-security http://security.debian.org/debian-security
debian-volatile http://volatile.debian.org/debian-volatile

ubuntu http://ru.archive.ubuntu.com/ubuntu
ubuntu-canonical http://archive.canonical.com/ubuntu
ubuntu-security http://security.ubuntu.com/ubuntu
ubuntu-extras http://extras.ubuntu.com/ubuntu

ppa http://ppa.launchpad.net

# The following are the default parameter values, so there is
# no need to uncomment them unless you want a different value.
# See approx.conf(5) for details.

#$cache /var/cache/approx
#$max_rate unlimited
#$max_redirects 5
#$user approx
#$group approx
#$syslog daemon
#$pdiffs true
#$offline false
#$max_wait 10
#$verbose false
#$debug false
```

</p></details>

###### Содержимое `/etc/apt/sources.list` на клиенте ubuntu 10.10:

<details><p>

```
# deb cdrom:[Ubuntu 10.10 _Maverick Meerkat_ - Release amd64 (20101007)]/ maverick main restricted
# See http://help.ubuntu.com/community/UpgradeNotes for how to upgrade to
# newer versions of the distribution.

deb http://approx.domain.com:9999/ubuntu/ maverick main restricted
deb-src http://approx.domain.com:9999/ubuntu/ maverick main restricted

## Major bug fix updates produced after the final release of the
## distribution.
deb http://approx.domain.com:9999/ubuntu/ maverick-updates main restricted
deb-src http://approx.domain.com:9999/ubuntu/ maverick-updates main restricted

## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu
## team. Also, please note that software in universe WILL NOT receive any
## review or updates from the Ubuntu security team.
deb http://approx.domain.com:9999/ubuntu/ maverick universe
deb-src http://approx.domain.com:9999/ubuntu/ maverick universe
deb http://approx.domain.com:9999/ubuntu/ maverick-updates universe
deb-src http://approx.domain.com:9999/ubuntu/ maverick-updates universe

## N.B. software from this repository is ENTIRELY UNSUPPORTED by the Ubuntu
## team, and may not be under a free licence. Please satisfy yourself as to
## your rights to use the software. Also, please note that software in
## multiverse WILL NOT receive any review or updates from the Ubuntu
## security team.
deb http://approx.domain.com:9999/ubuntu/ maverick multiverse
deb-src http://approx.domain.com:9999/ubuntu/ maverick multiverse
deb http://approx.domain.com:9999/ubuntu/ maverick-updates multiverse
deb-src http://approx.domain.com:9999/ubuntu/ maverick-updates multiverse

## Uncomment the following two lines to add software from the 'backports'
## repository.
## N.B. software from this repository may not have been tested as
## extensively as that contained in the main release, although it includes
## newer versions of some applications which may provide useful features.
## Also, please note that software in backports WILL NOT receive any review
## or updates from the Ubuntu security team.
deb http://approx.domain.com:9999/ubuntu/ maverick-backports main restricted universe multiverse
deb-src http://approx.domain.com:9999/ubuntu/ maverick-backports main restricted universe multiverse

## Uncomment the following two lines to add software from Canonical's
## 'partner' repository.
## This software is not part of Ubuntu, but is offered by Canonical and the
## respective vendors as a service to Ubuntu users.
deb http://approx.domain.com:9999/ubuntu-canonical maverick partner
deb-src http://approx.domain.com:9999/ubuntu-canonical maverick partner

## This software is not part of Ubuntu, but is offered by third-party
## developers who want to ship their latest software.
deb http://approx.domain.com:9999/ubuntu-extras maverick main
deb-src http://approx.domain.com:9999/ubuntu-extras maverick main

deb http://approx.domain.com:9999/ubuntu-security maverick-security main restricted
deb-src http://approx.domain.com:9999/ubuntu-security maverick-security main restricted
deb http://approx.domain.com:9999/ubuntu-security maverick-security universe
deb-src http://approx.domain.com:9999/ubuntu-security maverick-security universe
deb http://approx.domain.com:9999/ubuntu-security maverick-security multiverse
deb-src http://approx.domain.com:9999/ubuntu-security maverick-security multiverse
deb http://archive.ubuntu.com/ubuntu maverick restricted multiverse universe main
```

</p></details>
