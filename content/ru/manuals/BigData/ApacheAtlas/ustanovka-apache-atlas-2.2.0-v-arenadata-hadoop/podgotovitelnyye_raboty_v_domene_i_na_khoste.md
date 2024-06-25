---
title: "02. Подготовительные работы в домене и на хосте
"
date: 2023-01-17
weight: 1
description: >
  Подготовительные работы в домене и на хосте.
tags:
  - BigData
  - Apache Atlas
  - ArenaData Hadoop
---

## 1. Введение выделенного для Atlas хоста в домен
Выполняется с помощью  стандартных процедур. Например, 🚀Введение новой машины в домен.

## 2. Добавление хоста к Hadoop-кластеру
Выполняется с помощью  стандартных процедур. Например, 60. Добавление новых хостов в керберизированный кластер с TLS-шифрованием.

## 3. Добавление учётных записей
3.1. Аккаунт 'atlas' может существовать в IPA, поэтому использование этого аккаунта должно быть  запрещено через фильтры в файле `/etc/sssd/sssd.conf`.
Находясь на Atlas-машине, добавляем фильтры в раздел '[nss]'. Пример показан здесь:
```
[nss]
filter_groups = root,accumulo,apache,atlas,flume,hadoop,hbase,hdfs,hive,httpfs,hue,impala,kafka,keytrustee,kudu,llama,kms,mapred,mysql,oozie,postgres,sentry,solr,spark,sqoop,sqoop2,yarn,zookeeper
filter_users = root,accumulo,apache,atlas,flume,hbase,hdfs,hive,httpfs,hue,impala,kafka,keytrustee,kudu,llama,kms,mapred,mysql,oozie,postgres,sentry,solr,spark,sqoop,sqoop2,yarn,zookeeper
reconnection_retries = 3
homedir_substring = /home
```

3.2. Добавление локальной УЗ 'atlas' на Atlas-хост
Находясь на Atlas-машине, добавляем локальную УЗ 'atlas':
```
useradd -r -s /sbin/nologin -d /opt/atlas -M atlas
usermod -aG hadoop atlas
```

3.3. Добавление сервисных УЗ во FreeIPA
Находясь на Atlas-машине, добавляем в IPA сервисные УЗ для Atlas-машины. (Необходимо иметь права на добавление сервиса во FreeIPA):
```
kinit nifantevea
ATLASSERVER="$(hostname)"
 
ipa service-add atlas/${ATLASSERVER}
ipa service-add HTTP/${ATLASSERVER}
```

3.4. Добавление в FreeIPA пользовательских групп для авторизации в Atlas
Создаём по минимуму две группы, '${IS_NAME}_${CLUSTER_NAME}_atlas_admins' и '${IS_NAME}_${CLUSTER_NAME}_atlas_users'.

В переменной CLUSTERNAME требуется указать правильное имя кластера!

В переменной JIRA требуется указать правильный номер задачи!
```
IS_NAME="tst"
CLUSTER_NAME="ADHT1"
JIRA="SDK-????"
 
CL_N_LOWER=$(echo $CLUSTER_NAME | tr [:upper:] [:lower:])
CL_N_UPPER=$(echo $CLUSTER_NAME | tr [:lower:] [:upper:])
 
ipa group-add --desc="Группа для роли ROLE_ADMIN в Atlas'е в кластере ${CL_N_UPPER}. ${JIRA}." ${IS_NAME}_${CL_N_LOWER}_atlas_admins
ipa group-add --desc="Группа для роли DATA_STEWARD в Atlas'е в кластере ${CL_N_UPPER}. ${JIRA}." ${IS_NAME}_${CL_N_LOWER}_atlas_steward
```

## 4. Добавление необходимых Hadoop-ролей на машину с Atlas'ом
На машину с Atlas'ом, через ADCM WebUI, устанавливаем роли, которые требуются для следующего:
- HDFS Client — для получения и обновления информации о принадлежности какого-либо аккаунта к группам (UGI — User Group Information), используемым Hadoop'ом. Полезно при отладке.
- HBase Client — для первоначального импорта содержимого HBase и здесь Atlas хранит свою базу данных Janus, поэтому требуется постоянный доступ к двум таблицам в сервисе Apache HBase. 
- Hive Client — для первоначального импорта содержимого Hive.
