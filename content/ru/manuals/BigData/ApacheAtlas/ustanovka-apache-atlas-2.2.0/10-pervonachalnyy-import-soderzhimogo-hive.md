---
title: "10. Первоначальный импорт содержимого Hive"
date: 2021-08-29
weight: 12
description: >
  Описание процесса установки Apache Atlas 2.2.0 в Cloudera CDH 6.3.2.
tags:
  - BigData
  - Apache Atlas
  - Apache Hive
  - Cloudera CDH 6.3.2
slug: pervonachalnyy-import-soderzhimogo-hive
---

2021-08-29

Первоначальный импорт содержимого Hive в Atlas производится только один раз, сразу после настройки Atlas Hook'а для Hive. Apache Atlas должен быть запущен и слушать 'atlas.rest.address=https://dev-app111p.test2.lan:21443'. После одноразового импорта, дальнейшее соответствие информации между Atlas и Hive производится только на основании сообщений из Kafka.

Повторное использование ручного импорта не приведёт Атласовскую информацию о содержимом Hive'е в их полное соответствие. Те таблицы, которые есть в Atlas'е, но отсутствуют в Hive'е, не будут удалены из Atals'а. Такие таблицы могут быть удалены повторным созданием таблицы в Hive'е с их последующим удалением. В этом случае, такие таблицы в Atlas'е будут помечены, как удалённые.

На машине с установленным Atlas'ом и ролью 'Hive Gateway' получаем Kerberos-билет для УЗ с администраторскими правами в Atlas'е, переходим в рабочий каталог Atlas'а, устанавливаем необходимые переменные и запускаем импорт:
```
kinit eugene
export JAVA_HOME=/usr/java/jdk1.8.0_181-cloudera
export HIVE_HOME=/usr/lib/hive
export HIVE_CONF_DIR=/etc/hive/conf
export ATLAS_CONF_DIR=/opt/atlas/conf
export ATLAS_LOG_DIR=/opt/atlas/logs
bin/import-hive.sh -Dsun.security.jgss.debug=true
```

Stdout:
```
Using Hive configuration directory [/etc/hive/conf]
Log file for import is /opt/atlas/logs/import-hive.log
log4j:WARN No such property [maxFileSize] in org.apache.log4j.PatternLayout.
log4j:WARN No such property [maxBackupIndex] in org.apache.log4j.PatternLayout.
Search Subject for Kerberos V5 INIT cred (<<DEF>>, sun.security.jgss.krb5.Krb5InitCredential)
Search Subject for SPNEGO INIT cred (<<DEF>>, sun.security.jgss.spnego.SpNegoCredElement)
Search Subject for Kerberos V5 INIT cred (<<DEF>>, sun.security.jgss.krb5.Krb5InitCredential)
Hive Meta Data imported successfully!!!
```

Все дальнейшие изменения в Hive'е автоматически транслируются в Apache Atlas через Apache Kafka.
