---
title: "11. Первоначальный импорт содержимого HBase"
date: 2021-08-29
weight: 12
description: >
  Описание процесса установки Apache Atlas 2.2.0 в Cloudera CDH 6.3.2.
tags:
  - BigData
  - Apache Atlas
  - Apache HBase
  - Cloudera CDH 6.3.2
slug: pervonachalnyy-import-soderzhimogo-hbase
---

2021-08-29

Первоначальный импорт содержимого HBase в Atlas производится только один раз, сразу после настройки Atlas Hook'а для HBase. Apache Atlas должен быть запущен и слушать 'atlas.rest.address=https://dev-app111p.test2.lan:21443'. После одноразового импорта, дальнейшее соответствие информации между Atlas и HBase производится только на основании сообщений из Kafka.

Повторное использование ручного импорта не приведёт Атласовскую информацию о содержимом HBase'е в их полное соответствие. Те объекты, которые есть в Atlas'е, но отсутствуют в HBase'е, не будут удалены из Atals'а. Такие объекты могут быть удалены повторным созданием объекта в HBase'е с их последующим удалением. В этом случае, такие объекты в Atlas'е будут помечены, как удалённые.

1. Получаем Kerberos-билет для УЗ с администраторскими правами в Atlas'е, переходим в рабочий каталог Atlas'а, устанавливаем необходимые переменные, и запускаем импорт:
```
#kinit eugene

cd /opt/atlas

export JAVA_HOME=/usr/java/jdk1.8.0_181-cloudera
export HBASE_HOME=/usr/lib/hbase
export HBASE_CONF_DIR=/etc/hbase/conf
export ATLAS_CONF_DIR=/opt/atlas/conf
export ATLAS_LOG_DIR=/opt/atlas/logs

hook-bin/import-hbase.sh -Dsun.security.jgss.debug=true
```

stdout:
```
>>>>> hook-bin/import-hbase.sh
>>>>> /opt/atlas
Using HBase configuration directory [/etc/hbase/conf]
/etc/hadoop/conf:/usr/lib/hadoop/lib/*:/usr/lib/hadoop/.//*:/usr/lib/hadoop-hdfs/./:/usr/lib/hadoop-hdfs/lib/*:/usr/lib/hadoop-hdfs/.//*:/usr/lib/hadoop-mapreduce/.//*:/usr/lib/hadoop-yarn/lib/*:/usr/lib/hadoop-yarn/.//*
Log file for import is /opt/atlas/logs/import-hbase.log
Search Subject for Kerberos V5 INIT cred (<<DEF>>, sun.security.jgss.krb5.Krb5InitCredential)
Search Subject for Kerberos V5 INIT cred (<<DEF>>, sun.security.jgss.krb5.Krb5InitCredential)
Search Subject for SPNEGO INIT cred (<<DEF>>, sun.security.jgss.spnego.SpNegoCredElement)
Search Subject for Kerberos V5 INIT cred (<<DEF>>, sun.security.jgss.krb5.Krb5InitCredential)
HBase Data Model imported successfully!!!
```

2. Для безопасности удаляем свой kerberos-билет:
```
$ kdestroy
```

Все дальнейшие изменения в HBase'е будут автоматически транслироваться в Apache Atlas через Apache Kafka.
