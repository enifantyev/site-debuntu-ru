---
title: "00. Удаление Apache Atlas"
date: 2023-02-26
weight: 1
description: >
  Описание процесса удаления Apache Atlas 2.2.0 из ADH.
tags:
  - BigData
  - Apache Atlas
  - ArenaData Hadoop
---

## 1. Удаление данных Atlas'а из кластера
### Systemd
1. Выключаем и останавливаем демон Atlas'а. Удаляем systemd unit.
    ```
    systemctl disable --now atlas
    rm -f /etc/systemd/systemd/atlas.service
    systemctl daemon-reload
    ```

### Hbase
1. На любой машине с ролью 'HBase Client', выполняем удаление двух таблиц в HBase.
Список названий таблиц по умолчанию:
– apache_atlas_entity_audit
– apache_atlas_janus
    ```
    # kinit nifantev
    TABLE1="apache_atlas_entity_audit"
    TABLE2="apache_atlas_janus"
    
    echo "disable '${TABLE1}'; disable '${TABLE2}'" | hbase shell
    echo "drop '${TABLE1}'; drop '${TABLE2}'" | hbase shell
    ```

### Solr
1. Удаляем из Solr три "Коллекции", указывая в `curl` любой Solr-хост из кластера.
Список названий "Коллекций" по умолчанию:
- fulltext_index
- edge_index
- vertex_index
    ```
    # kinit nifantev
    SOLRSERVER="dev-app111p.test2.lan"
    COLLECTION1="fulltext_index"
    COLLECTION2="edge_index"
    COLLECTION3="vertex_index"
    
    curl --negotiate -u : "https://${SOLRSERVER}:8985/solr/admin/collections?action=DELETE&name=${COLLECTION1}"
    curl --negotiate -u : "https://${SOLRSERVER}:8985/solr/admin/collections?action=DELETE&name=${COLLECTION2}"
    curl --negotiate -u : "https://${SOLRSERVER}:8985/solr/admin/collections?action=DELETE&name=${COLLECTION3}"
    ```

2. Удаляем из Solr'а Atlas-шаблон 'atlasTemplate':
```
solrctl config --delete atlasTemplate
```

### Kafka
1. Из Apache Kafka удаляем три топика:
    ```
    ZKSERVERS="dev-zk110p.test2.lan,dev-zk111p.test2.lan,dev-zk112p.test2.lan/kafka"
    # Список имён топиков по умолчанию:
    TOPIC1="_HOATLASOK"
    TOPIC2="ATLAS_ENTITIES"
    TOPIC3="ATLAS_HOOK"
      
    kafka-topics --zookeeper ${ZKSERVERS} --delete --topic ${TOPIC1}
    kafka-topics --zookeeper ${ZKSERVERS} --delete --topic ${TOPIC2}
    kafka-topics --zookeeper ${ZKSERVERS} --delete --topic ${TOPIC3}
    ```

### Zookeeper
1. Вероятно, что из znode `/Arenadata.Hadoop-10.solr.server` потребуется удалить некоторую информацию?

## 2. Удаление из Hadoop-кластера параметров для работы Atlas hooks
1. Из настроек Hive удаляем Atlas Hive Hook.
2. Из настроек HBase удаляем Atlas HBase Hook.

## 3. Зачистка информации из FreeIPA
### 3.1. Удаление пользовательских групп
Удаляем пользовательские группы:

### 3.2. Удаление сервисных УЗ atlas
Удаляем сервисные УЗ:

 