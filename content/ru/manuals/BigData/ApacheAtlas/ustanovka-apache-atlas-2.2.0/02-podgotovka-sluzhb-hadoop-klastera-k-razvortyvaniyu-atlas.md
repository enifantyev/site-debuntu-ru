---
title: "02. Подготовка служб Hadoop-кластера к развёртыванию Atlas'а"
date: 2021-11-03
weight: 10
description: >
  Описание процесса установки Apache Atlas 2.2.0 в Cloudera CDH 6.3.2.
tags:
  - BigData
  - Apache Atlas
  - Cloudera CDH 6.3.2
slug: podgotovka-sluzhb-hadoop-klastera-k-razvortyvaniyu-atlas
---

2021-08-10 – 2021-11-03

## 1. Apache HBase
Apache HBase используется Atlas'ом для хранения своей Janus базы данных. Бла-бла-бла...

### 1.1. Настройка Apache HBase
Настройка HBase в тестовом кластере была выполнена в соответствии с инструкцией 16. HBase. Установка и настройка.

### 1.2. Создание необходимых таблиц в HBase
1.2.1. На Atlas-машине, или на любой машине с установленной ролью 'HBase Gateway', создаём необходимые таблицы и даём на них все права для УЗ 'atlas':
```
# Названия таблиц по умолчанию:
TABLE1="apache_atlas_entity_audit"
TABLE2="apache_atlas_janus"

echo "create '${TABLE1}', 'dt'; grant 'atlas', 'RWXCA', '${TABLE1}'"  | hbase shell
echo "create '${TABLE2}', 's'; grant 'atlas', 'RWXCA', '${TABLE2}'" | hbase shell
```

> В случае внесения УЗ 'atlas' в параметр 'hbase.superuser', или в IPA-группу 'test2_hbase_su' (в случае прямого подключения Hadoop'а к LDAP), или назначении всех прав на HBase для УЗ 'atlas' — `echo "grant 'atlas', 'RWXCA'" | hbase shell`, Atlas самостоятельно создаст необходимые базы и назначит для УЗ 'atlas' права на них. Это иногда удобно, например, для исследовательских целей.<br>
Замечу, что с Atlas 2.2.0 такое не получилось в случае добавления 'atlas' в 'hbase.superuser'. В след раз попробую `echo "grant 'atlas', 'RWXCA'" | hbase shell`.

### 1.3. Проверяем назначенные на таблицы права
1.3.1. На Atlas-машине, или на любой машине с установленной ролью 'HBase Gateway', выполняем:
```
echo "user_permission 'apache_atlas_.*'" | hbase shell

user_permission 'apache_atlas_.*'
User                               Namespace,Table,Family,Qualifier:Permission                                                      
 atlas                             default,apache_atlas_entity_audit,,: [Permission: actions=READ,WRITE,EXEC,CREATE,ADMIN]          
 atlas                             default,apache_atlas_janus,,: [Permission: actions=READ,WRITE,EXEC,CREATE,ADMIN]                 
2 row(s)
```

## 2. Apache Hive
Установка и настройка Apache Hive выполняется по инструкции '18. Hive. Установка и настройка'.

## 3. Apache Kafka
Apache Kafka используется Atlas'ом для получения сообщений о произошедших событиях в сервисах Hadoop'а. Сообщения отправляются с помощью специальных библиотек из состава Atlas'а, внедрённых в определённые сервисы. На данный момент Atlas читает сообщения о таких событиях в Hbase и Hive, как создание и удаление таблиц, добавления столбцов, бла-бла-бла...

### 3.1. Установка Apache Kafka
3.1.1. Установка и настройка Kafka выполняется по инструкции '22. Kafka. Установка и настройка'.

### 3.2. Добавление в Kafka необходимых топиков
3.2.1 Для работы Apache Atlas требуется наличие трёх топиков в Apache Kafka. Создаём их на любой машине с ролью 'Kafka Gateway' командами:
```
ZKSERVERS="dev-zk110p.test2.lan,dev-zk111p.test2.lan,dev-zk112p.test2.lan/kafka"
# Список имён топиков по умолчанию:
TOPIC1="_HOATLASOK"
TOPIC2="ATLAS_ENTITIES"
TOPIC3="ATLAS_HOOK"

kafka-topics --zookeeper ${ZKSERVERS} --create --replication-factor 2 --partitions 2 --topic ${TOPIC1}
kafka-topics --zookeeper ${ZKSERVERS} --create --replication-factor 2 --partitions 2 --topic ${TOPIC2}
kafka-topics --zookeeper ${ZKSERVERS} --create --replication-factor 2 --partitions 2 --topic ${TOPIC3}
```

### 3.3. Проверка наличия топиков
3.3.1. Здесь же проверим наличие информации о топиках через `kafka-topics`:
```
kafka-topics --zookeeper ${ZKSERVERS} --list
...
ATLAS_ENTITIES
ATLAS_HOOK
_HOATLASOK
```

3.3.2. И/или на любом хосте кластера даём команду:
```
ZKSERVER="dev-zk110p.test2.lan"

echo "ls /kafka/brokers/topics" | zookeeper-client -server ${ZKSERVER}
...
ment complete on server localhost/127.0.0.1:2181, sessionid = 0x37c9e0ee4460070, negotiated timeout = 30000

WATCHER::

WatchedEvent state:SyncConnected type:None path:null
[zk: localhost:2181(CONNECTED) 0] ls /kafka/brokers/topics
[ATLAS_HOOK, ATLAS_ENTITIES, _HOATLASOK]
```

### 3.4. Настройка роли в Sentry для доступа Atlas'а к топикам в Kafka
3.4.1. На машине с ролью 'Kafka Gateway' и 'Sentry Gateway' создаём в Sentry роль 'kafka4atlas_role:
```
KROLE="kafka4atlas_role"

kafka-sentry -cr -r ${KROLE}
```

3.4.2. Назначаем созданную роль на группу atlas:
```
kafka-sentry -arg -r ${KROLE} -g atlas
```

3.4.3. Назначаем привилегии для потребителя:
https://docs.cloudera.com/documentation/enterprise/6/6.3/topics/kafka_security.html#concept_s4z_nlh_znb__section_elp_rph_znb
```
# Список имён топиков по умолчанию:
TOPIC1="_HOATLASOK"
TOPIC2="ATLAS_ENTITIES"
TOPIC3="ATLAS_HOOK"

kafka-sentry -gpr -r ${KROLE} -p "Host=*->CONSUMERGROUP=*->action=read"
kafka-sentry -gpr -r ${KROLE} -p "Host=*->CONSUMERGROUP=*->action=describe"

kafka-sentry -gpr -r ${KROLE} -p "HOST=*->TOPIC=${TOPIC1}->action=read"
kafka-sentry -gpr -r ${KROLE} -p "HOST=*->TOPIC=${TOPIC2}->action=read"
kafka-sentry -gpr -r ${KROLE} -p "HOST=*->TOPIC=${TOPIC3}->action=read"
kafka-sentry -gpr -r ${KROLE} -p "HOST=*->TOPIC=${TOPIC1}->action=describe"
kafka-sentry -gpr -r ${KROLE} -p "HOST=*->TOPIC=${TOPIC2}->action=describe"
kafka-sentry -gpr -r ${KROLE} -p "HOST=*->TOPIC=${TOPIC3}->action=describe"
```

3.4.4. Назначаем привилегии для продюсера:
https://docs.cloudera.com/documentation/enterprise/6/6.3/topics/kafka_security.html#concept_s4z_nlh_znb__section_jn2_sph_znb
```
kafka-sentry -gpr -r ${KROLE} -p "HOST=*->TOPIC=${TOPIC1}->action=write"
kafka-sentry -gpr -r ${KROLE} -p "HOST=*->TOPIC=${TOPIC2}->action=write"
kafka-sentry -gpr -r ${KROLE} -p "HOST=*->TOPIC=${TOPIC3}->action=write"
```

### 3.5. Проверка настроек Sentry для Kafka
3.5.1. Выводим список существующих ролей:
```
$ kafka-sentry -lr
....
solradm_role
kafka4atlas_role
```

3.5.2. Выводим список групп с назначенными им ролями:
```
$ kafka-sentry -lg
...
atlas = kafka4atlas_role
test2_solr_admins = solradm_role
```

3.5.3. Выводим список привилегий:
```
$ kafka-sentry -lp -r kafka4atlas_role
...
HOST=*->TOPIC=_HOATLASOK->action=read
HOST=*->TOPIC=_HOATLASOK->action=describe
HOST=*->TOPIC=ATLAS_HOOK->action=read
HOST=*->TOPIC=ATLAS_ENTITIES->action=describe
HOST=*->TOPIC=ATLAS_HOOK->action=describe
HOST=*->CONSUMERGROUP=*->action=describe
HOST=*->TOPIC=_HOATLASOK->action=write
HOST=*->TOPIC=ATLAS_ENTITIES->action=write
HOST=*->TOPIC=ATLAS_HOOK->action=write
HOST=*->TOPIC=ATLAS_ENTITIES->action=read
HOST=*->CONSUMERGROUP=*->action=read
```

> Если наблюдаем лишние привилегии, то удаляем их командой, например:<br>
  `  $ kafka-sentry -r kafka4atlas_role -rpr -p 'HOST=*->TOPIC=*->action=all'`
