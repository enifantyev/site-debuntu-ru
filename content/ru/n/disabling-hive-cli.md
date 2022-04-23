---
title: "Disabling Hive CLI"
date: 2020-10-05
weight: 10
description: >
  После активации Cloudera Sentry, необходимо предотвратить возможность использования консольной утилиты hive пользователями.
tags:
  - Apache Hive
  - Apache Hadoop
  - Cloudera CDH 6.3.2
  - BigData
---

Перед выполнением этой инструкции, необходимо ознакомиться с [Блокирование доступа внешних программ к Hive metastore](/n/blokirovanie-dostupa-vneshnih-programm-k-hive-metastore), где описано более централизованный способ блокирования доступа к Metastore Hive.

После активации Cloudera Sentry, необходимо предотвратить возможность использования консольной утилиты `hive` пользователями. Вместо `hive`, пользователи должны использовать утилиту `beeline`.

Каталог HIVE_HOME можно найти из-под root командой:
```
$ sudo hive -e '!env'|grep HIVE_HOME
HIVE_HOME=/usr/lib/hive
```
Таким образом, отключение Hive CLI производим следующими командами:
```
HIVE_HOME=$(hive -e '!env'|grep HIVE_HOME|awk -F'=' '{print $2}')
setfacl -m u:hive:rx $HIVE_HOME/bin/hive
chmod 754 $HIVE_HOME/bin/hive
```
Разъяснение: Hive не будет стартовать, если не оставить ему доступ к этому файлу.

При необходимости, снимаем запрет командами:
```
HIVE_HOME=$(hive -e '!env'|grep HIVE_HOME|awk -F'=' '{print $2}')
setfacl -b $HIVE_HOME/bin/hive
chmod 755 $HIVE_HOME/bin/hive
```
