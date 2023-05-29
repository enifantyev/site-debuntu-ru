---
title: "PostgreSQL14. CentOS7. Установка"
date: 2023-05-02
weight: 10
description: >
  Описание установки PostgreSQL на CentOS 7 с переносом базы в новое место.
tags:
  - PostgreSQL
  - CentOS7
slug: postgresql14-centos7-ustanovka
---
2023-05-02

## 1. Добавление yum-репозитория

1. Добавляем файл-репо:
    ```bash
    USERNAME=nx-pubrepo-reader
    PASSWORD=xxxxxxxxxxxxxxxxx

    cat << EOF > /etc/yum.repos.d/pgdg-14.repo
    [pgdg-common]
    username=$USERNAME
    password=$PASSWORD
    name=PostgreSQL common RPMs for RHEL/CentOS \$releasever - \$basearch
    baseurl=http://nexus.example.org:8081/repository/pub_postgresql_yum/common/redhat/rhel-\$releasever-\$basearch
    enabled=1
    gpgcheck=0
    gpgkey=http://nexus.example.org:8081/repository/pub_postgresql_yum/RPM-GPG-KEY-PGDG-AARCH64

    [pgdg14]
    username=$USERNAME
    password=$PASSWORD
    name=PostgreSQL 14 for RHEL/CentOS \$releasever - \$basearch
    baseurl=http://nexus.example.org:8081/repository/pub_postgresql_yum/14/redhat/rhel-\$releasever-\$basearch
    enabled=1
    gpgcheck=0
    gpgkey=http://nexus.example.org:8081/repository/pub_postgresql_yum/RPM-GPG-KEY-PGDG-AARCH64
    EOF
    ```

2. Проверяем подключение к репо:
    ```bash
    yum check-update
    ```

## 2. Установка PostgreSQL

1. Устанавливаем PostgreSQL командой:
    ```bash
    yum install -y postgresql14-server
    ```

## 3. Инициализация базы данных

1. Выполняем:
    ```bash
    /usr/pgsql-14/bin/postgresql-14-setup initdb
    ```

    База данных будет создана в `/var/lib/pgsql/14/data`.

## 4. Перенос базы данных в `/srv`

1. При необходимости, копируем ранее созданную базу данных в другое место `/srv`, к которому подключен большой диск:
    ```bash
    rsync -av /var/lib/pgsql /srv/
    ```

## 5. Запуск postgresql-14

1. Переопределяем путь к базе данных в systemd-юните 'postgresql-14.service':
    ```bash
    UNITNAME='postgresql-14.service'
    DBPATH='/srv/pgsql/14/data/'

    mkdir -p /etc/systemd/system/${UNITNAME}.d
    echo -e "[Service]\nEnvironment=PGDATA=${DBPATH}" > \
    /etc/systemd/system/${UNITNAME}.d/override.conf
    ```

2. Перечитываем systemd и запускаем PostgreSQL:
    ```bash
    systemctl daemon-reload
    systemctl enable --now $UNITNAME
    ```

## 6. Заключительная зачистка

1. Удаляем оставшуюся копию базы данных:
    ```bash
    rm -rf /var/lib/pgsql
    ```