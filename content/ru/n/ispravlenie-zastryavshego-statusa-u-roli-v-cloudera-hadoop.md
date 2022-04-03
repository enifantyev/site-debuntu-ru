---
title: "Исправление застрявшего статуса у роли в Cloudera CDH"
date: 2020-09-26
weight: 10
description: >
  Из-за поспешной перезагрузки агентов или management service, статус роли, которую перегружали, может застрять в RUNNING или STOPPING. В результате, с ролью ничего нельзя сделать.
tags:
  - Cloudera CDH 6.3.2
  - BigData
---

2020-09-26

Из-за поспешной перезагрузки агентов или management service, статус роли, которую перегружали, может застрять в RUNNING или STOPPING. В результате, с ролью ничего нельзя сделать.

## Исправление
Заходим в клаудеровскую базу данных PostgreSQL.

В случае использования встроенной БД, находим автоматический созданный пароль так:
```
cat /var/lib/cloudera-scm-server-db/data/generated_password.txt

MnPwGeWaip
```

Заходим в CLI:
```
psql -U cloudera-scm -p 7432 -h localhost -d postgres
Password for user cloudera-scm: MnPwGeWaip
```

Переходим в базу scm и выполняем поиск застрявшего статуса, например STOPPING, и его замену:
```
# \c scm

# select role_id, name, configured_status from ROLES where configured_status = 'STOPPING';
 role_id |                               name                               | configured_status
---------+------------------------------------------------------------------+-------------------
      25 | mgmt-SERVICEMONITOR-edbf339a77d8be878f36462f131bc862             | RUNNING
      58 | hbase-HBASETHRIFTSERVER-cd4fc95f13f38837f08699095a260ef7         | STOPPING
      33 | hdfs-DATANODE-b7b6861c2160c74fd2362f9919c9331b                   | RUNNING

# update roles set configured_status = 'STOPPED' where role_id = 58;
UPDATE 1

# select role_id, name, configured_status from ROLES where configured_status = 'STOPPED';
 role_id |                           name                           | configured_status
---------+----------------------------------------------------------+-------------------
      58 | hbase-HBASETHRIFTSERVER-cd4fc95f13f38837f08699095a260ef7 | STOPPED
(1 строка)
```
Для обновления статуса, необходимо перезапустить cloudera-scm-server.

[Custom add on service stuck in starting state, and now cannot start,stop,delete this service.](https://community.cloudera.com/t5/Support-Questions/Custom-add-on-service-stuck-in-starting-state-and-now-cannot/m-p/83958)
[Managing the Embedded PostgreSQL Database](https://docs.cloudera.com/documentation/enterprise/latest/topics/cm_ig_embed_pstgrs.html)
