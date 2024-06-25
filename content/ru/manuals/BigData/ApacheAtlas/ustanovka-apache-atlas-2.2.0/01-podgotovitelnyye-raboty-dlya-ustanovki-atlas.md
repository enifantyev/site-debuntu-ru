---
title: "01. Подготовительные работы для установки Atlas'а"
date: 2021-11-03
weight: 10
description: >
  Описание процесса установки Apache Atlas 2.2.0 в Cloudera CDH 6.3.2.
tags:
  - BigData
  - Apache Atlas
  - Cloudera CDH 6.3.2
slug: podgotovitelnyye-raboty-dlya-ustanovki-atlas
---

Везде, где выполняется работа с керберизированными сервисами, помним о предварительном получении Kerberos-билета, если таковой не был получен ранее.

Подавляющее число операций производим на Atlas-машине, иначе специально указывается иное.

## 1. Введение выделенного для Atlas хоста в домен
Выполняется с помощью  стандартных процедур.

## 2. Добавление хоста к Hadoop-кластеру
Выполняется с помощью  стандартных процедур.

## 3. Добавление учётных записей
### 3.1. Добавление локальной УЗ 'atlas'
На все машины кластера добавляем локальную УЗ 'atlas'.
```
sudo useradd -r -s /sbin/nologin -d /opt/atlas -M atlas
sudo usermod -aG hadoop atlas
```

### 3.2. Добавление сервисной УЗ во FreeIPA
Находясь на Atlas-машине, добавляем в IPA сервисную УЗ для Atlas-машины. Необходимо иметь права на добавление сервиса во FreeIPA.
```
ATLASSERVER="$(hostname)"

# При необходимости получаем kerberos-билет
# kinit

ipa service-add atlas/${ATLASSERVER}
```

### 3.3. Добавление в FreeIPA пользовательских групп для авторизации в Atlas
Создаём по минимуму две группы, 'clustername_atlas_admin' и 'clustername_atlas_user'.

В переменной CLUSTERNAME требуется указать правильное имя кластера!

В переменной JIRA требуется указать правильный номер задачи!
```
CLUSTERNAME="TEST2"
JIRA="SDK-2198"

CLNLOWER=$(echo $CLUSTERNAME | tr [:upper:] [:lower:])
CLNUPPER=$(echo $CLUSTERNAME | tr [:lower:] [:upper:])
ipa group-add --desc="Группа для роли ROLE_ADMIN в Atlas'е в кластере ${CLNUPPER}. ${JIRA}." ${CLNLOWER}_atlas_admins
ipa group-add --desc="Группа для роли DATA_SCIENTIST в Atlas'е в кластере ${CLNUPPER}. ${JIRA}." ${CLNLOWER}_atlas_users
```

## 4. Добавление необходимых ролей на машину с Atlas'ом
На машину с Atlas'ом, через Cloudera Manager, устанавливаем роли, которые требуются для следующего:
- HDFS Gateway — для получения и обновления информации о принадлежности какого-либо аккаунта к группам (UGI — User Group Information), используемым Hadoop'ом. Полезно при отладке.
- HBase Gateway — для первоначального импорта содержимого HBase и здесь Atlas хранит свою базу данных Janus, поэтому требуется постоянный доступ к двум таблицам в сервисе Apache HBase.
- Hive Gateway — только для первоначального импорта содержимого Hive.
- Kafka Gateway — похоже, что исключительно для настройки Sentry через утилиту `kafka-sentry`, работающей только с  Kafka-привилегиями.
- Solr Gateway — похоже, что исключительно для настройки Sentry через утилиту `soltctl`, работающей только с  Solr-привилегиями.
