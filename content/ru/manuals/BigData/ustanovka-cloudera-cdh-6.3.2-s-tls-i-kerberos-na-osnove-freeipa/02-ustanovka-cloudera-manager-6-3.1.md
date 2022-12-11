---
title: "02. Установка Cloudera Manager 6.3.1"
date: 2021-06-16
weight: 2
description: >
  В этой части описывается установка Cloudera Manager 6.3.1.
tags:
  - Cloudera CDH 6.3.2
  - Apache Hadoop
  - CentOS
  - CentOS 7
  - BigData
aliases:
  - /a/ustanovka-cloudera-manager-6-3.1/
  - /a/01-ustanovka-cloudera-manager-6-3.1
slug: ustanovka-cloudera-manager-6-3.1
---

2021-06-16

## 1. Введение
Номер последней версии Cloudera Manager, которая может работать без лицензии, это 6.3.1.

После преднастроек и проверок приступаем к установке Cloudera Manager на один из хостов будущего кластера. То есть сначала установим Cloudera Manager на машину управления, с помощью которого, позже, установим сервисы Zookeeper, HDFS, YARN, и далее по списку.

Все операции выполняем на командной машине с помощью ансибла из ранее подготовленного каталога `hadoop`, где созданы все необходимые файлы.

## 2. Выделение машины управления кластером
Машина управления кластером имеет название, которое включает в себя буквы 'mgm'. Определив ip-адрес этой машины, добавим новые строки в инвентори файл ансибла:
```bash
printf '\n[mgm]\n10.1.4.6\n' >> cluster.inv
```

## 3. Добавление файлов repo на машину управления
При установке компонента Hue, потребуется пакет `libtidy`, который находится в EPEL-репозитории. Поэтому подготавливаем не только файлы репозиториев cloudera, но и epel. Файлы Cloudera-репо будут заброшены на машину управления кластером, тогда как epel-репо на все машины кластера:
```bash
cd hadoop

cat << EOF > cloudera-manager.repo
[cloudera-manager]
USERNAME=nx-pubrepo-reader
PASSWORD=xxxxxxxxxxxxxxxxx
name = Cloudera Manager, Version cloudera-cm-6.3.1-hosted
baseurl = http://10.1.1.1:8081/repository/cloudera-cm-6.3.1-hosted/
gpgcheck = 0
EOF

cat << EOF > cloudera-cdh.repo
[cloudera-cdh]
USERNAME=nx-pubrepo-reader
PASSWORD=xxxxxxxxxxxxxxxxx
name = Cloudera Manager, Version cloudera-cdh-6.3.2-hosted
baseurl = http://10.1.1.1:8081/repository/cloudera-cdh-6.3.2-hosted/
gpgcheck = 0
EOF

cat << EOF > epel.repo
[epel7]
USERNAME=nx-pubrepo-reader
PASSWORD=xxxxxxxxxxxxxxxxx
name=Extra Packages for Enterprise Linux \$releasever - \$basearch
baseurl=http://10.1.1.1:8081/repository/all_epel/\$releasever/\$basearch
failovermethod=priority
enabled=1
gpgcheck=0
gpgkey=http://10.1.1.1:8081/repository/all_epel/RPM-GPG-KEY-EPEL-\$releasever
EOF
```

Копируем CM репо-файл на машину cloudera server и epel repo-файл на все машины:
```bash
ansible mgm -i cluster.inv -m copy -a "src=cloudera-manager.repo dest='/etc/yum.repos.d/'" --become
ansible all -i cluster.inv -m copy -a "src=epel.repo dest='/etc/yum.repos.d/'" --become
```

## 4. Установка JDK и Cloudera Manager
```bash
ansible mgm -i cluster.inv -m yum -a "name=oracle-j2sdk1.8 state=latest" --become
ansible mgm -i cluster.inv -m yum -a "name=cloudera-manager-daemons,cloudera-manager-agent,cloudera-manager-server,cloudera-manager-server-db-2 state=latest" --become
```

## 5. Установка дополнительных пакетов
Дополнительно устанавливаем свежие версии python-пакета `psycopg2-binary` и `python-snappy`, которые используется Hue-компонентом. Так как неизвестно точно на каких хостах будут добавлены экземпляры Hue, то устанавливаем пакеты на все хосты.
```bash
ansible all -i cluster.inv -m pip -a "name=psycopg2-binary,python-snappy state=latest" --become
```

## 6. Запуск PostgreSQL и Cloudera Manager
```bash
ansible mgm -i cluster.inv -m systemd -a "name=cloudera-scm-server-db enabled=yes state=started" --become
ansible mgm -i cluster.inv -m systemd -a "name=cloudera-scm-server enabled=yes state=started" --become
```

Web-интерфейс доступен по адресу:

http://dev-mgm01p.example.org:7180/
