---
title: "Создание ролей в керберизированным Hive с включённым TLS в Cloudera CDH с помощью утилиты beeline"
date: 2021-06-09
weight: 10
description: >
  При включённом TLS, в строку подключения к Hive требуется добавить опцию 'ssl=true' и путь к хранилищу ca-сертификатов.
tags:
  - Apache Hive
  - beeline
---

2021-06-09

Использованные ссылки:
- [Using the Beeline CLI](https://docs.cloudera.com/documentation/enterprise/latest/topics/cdh_ig_hiveserver2_start_stop.html#topic_18_8_1)
- [Hive SQL Syntax for Use with Sentry](https://docs.cloudera.com/documentation/enterprise/6/6.3/topics/sg_hive_sql.html)

Запрашиваем kerberos-билет на машине с установленной ролью Hive, запускаем `beeline` и подключаемся к Hive:
```
$ kinit /run/cloudera-scm-agent/process/.../hive.keytab hive/prod-aux02p.example.org
$ beeline
beeline> !connect jdbc:hive2://prod-aux02p.example.org:10000/;principal=hive/prod-aux02p.example.org@EXAMPLE.ORG;ssl=true;sslTrustStore=/usr/java/jdk1.8.0_181-cloudera/jre/lib/security/jssecacerts
0: jdbc:hive2://prod-aux02p.example.org:1000> _
```

Без указания ssl-настроек в строке подключения, в логах HiveServer2 будут видны ошибки:
```
TThreadPoolServer	
[HiveServer2-Handler-Pool: Thread-47]: Error occurred during processing of message.
java.lang.RuntimeException: org.apache.thrift.transport.TTransportException: javax.net.ssl.SSLException: Unrecognized SSL message, plaintext connection?
```

Добавление роли для администрирования Hive (server1) и назначение этой роли на группу администраторов:
```
CREATE ROLE admin_role;
GRANT ALL ON SERVER server1 TO ROLE admin_role WITH GRANT OPTION;
GRANT ROLE admin_role TO GROUP prod_hue_admins;
GRANT ROLE admin_role TO GROUP prod_sentry_admins;
```

На всякий случай, привожу различные операции. Кстати, команды можно набирать буквами в нижнем регистре.

Список существующих ролей:
```
show roles;
```

Удаление ошибочно созданного назначения роли на группу:
```
GRANT ROLE admin_role TO GROUP admin_group;
REVOKE ROLE admin_role FROM GROUP admin_group;
```
