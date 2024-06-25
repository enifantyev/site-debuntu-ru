---
title: "Получение прав Superuser'а в ZooKeeper'е Cloudera CDH 6.3.2"
date: 2021-08-11
weight: 10
description: >
  Иногда в ZooKeeper'е требуется удалить неудаляемое.
tags:
  - Apache Zookeeper
  - Apache Hadoop
  - Cloudera Hadoop
alias:
  - /n/poluchenie-prav-superusera-v-zookeepere-cloudera-cdh-6-3.2/
slug: poluchenie-prav-superusera-v-zookeepere-cloudera-cdh-6-3.2
---

2021-08-11

## Введение

При удалении ноды `/solr` через `zookeeper-client` выводилась ошибка:
```
[zk: localhost:2181(CONNECTED) 0] rmr /solr
The command 'rmr' has been deprecated. Please use 'deleteall' instead.
Authentication is not valid : /solr/security
```

Решение этой проблемы нашёл здесь: https://community.cloudera.com/t5/Support-Questions/How-to-delete-an-acl-in-zookeeper/td-p/292421

## Алгоритм получения прав superuser'а

Переходим в домашний каталог Zookeeper'а и запускаем генерацию дайджеста пароля для только что придуманного логина, под которым возобладаем superuser-правами в Zookeeper'е, например 'zoosu:TsKlydBdycdq':
```
# cd /usr/lib/zookeeper && \
  /usr/java/jdk1.8.0_181-cloudera/bin/java -cp "./zookeeper.jar:lib/slf4j-api-1.7.25.jar" org.apache.zookeeper.server.auth.DigestAuthenticationProvider zoosu:TsKlydBdycdq
```

В выводе команды видим дайджест 'zoosu:LC4PzmQLqFSlkfUMsN4oYrUGaM8=':
```
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
zoosu:TsKlydBdycdq->zoosu:LC4PzmQLqFSlkfUMsN4oYrUGaM8=
```

Вставляем эти учётные данные ('zoosu:LC4PzmQLqFSlkfUMsN4oYrUGaM8=') в строку параметров для java-машины ZooKeeper'а ('<span style="color:blue">-Dzookeeper.DigestAuthenticationProvider.superDigest=zoosu:LC4PzmQLqFSlkfUMsN4oYrUGaM8=</span>'), которую размещаем в настройках ZooKeeper'а в Cloudera Manager.

В настройках ZooKeeper, используя фильтр 'Java Configuration Options for Zookeeper Server', изменяем следующие параметры:

<table>
<tr><th>Property</th><th>Value</th><th>Description</th></tr>
<tr valign=top>
<td><b>Java Configuration Options for Zookeeper Server</b></td>
<td><span style="color:blue">-Dzookeeper.DigestAuthenticationProvider.superDigest=<br>zoosu:LC4PzmQLqFSlkfUMsN4oYrUGaM8=</span></td>
<td>These arguments will be passed as part of the Java command line. Commonly, garbage collection flags, PermGen, or extra debugging flags would be passed here. Note: When CM version is 6.3.0 or greater, {{JAVA_GC_ARGS}} will be replaced by JVM Garbage Collection arguments based on the runtime Java JVM version.</td></tr>
</table>

Нажимаем **Save Changes**.

Перезапускаем все зависимые сервисы по приглашению Cloudera Manager Console.

Или останавливаем кластер и запускаем только сервис ZooKeeper'а, если, конечно, хотим удалить вышеуказанную строку из настроек, после выполнения удаления ноды. После чего останавливаем ZooKeeper и запускаем весь кластер.

## Удаление ноды
На ZooKeeper-узле запускаем `zookeeper-client`, авторизуемся с помощью команды `addauth digest zoosu:TsKlydBdycdq`, и выполняем удаление ноды `/solr`:
```
[zk: localhost:2181(CONNECTED) 0] ls /solr
[configs, overseer, security.json, solr.xml, autoscaling.json, security, aliases.json, live_nodes, collections, overseer_elect, clusterstate.json, autoscaling, clusterprops.json]
[zk: localhost:2181(CONNECTED) 1] addauth digest zoosu:TsKlydBdycdq
[zk: localhost:2181(CONNECTED) 2] deleteall /solr
[zk: localhost:2181(CONNECTED) 3] ls /
[zookeeper, hadoop-ha, kafka, hive_zookeeper_namespace_hive, hbase]
[zk: localhost:2181(CONNECTED) 4]
```
