---
title: "03. Подготовка служб ADH-кластера к развёртыванию Atlas'а"
date: 2023-01-17
weight: 1
description: >
  Подготовительные работы в домене и на хосте.
tags:
  - BigData
  - Apache Atlas
  - ArenaData Hadoop
---

## Заметки
- Apache HBase используется Atlas'ом для хранения своей Janus базы данных.
- Apache Solr используется для хранения журналов аудита и поиск в них.
- Apache Kafka используется как передатчик сообщений из Atlas-библиотек, то есть внедрённых в Hadoop-сервисы hook'ов, в сам Atlas.

## 1. Apache HBase
### 1.1. Создание необходимых таблиц в HBase
1. На Atlas-машине, или на любой другой машине с установленной ролью 'HBase Gateway', создаём необходимые таблицы:
    ```bash
    # Названия таблиц по умолчанию:
    TABLE1="apache_atlas_entity_audit"
    TABLE2="apache_atlas_janus"

    echo "create '${TABLE1}', 'dt'"  | hbase shell
    echo "create '${TABLE2}', 's'" | hbase shell
    ```

### 1.2. Проверяем наличие созданных таблиц
1. На Atlas-машине, или на любой другой машине с установленной ролью 'HBase Gateway', выполняем:

    ```bash
    echo "list" | hbase shell
    ```

    stdout:
    ```
    Took 0.0028 seconds
    list
    TABLE                        
    apache_atlas_entity_audit                             
    apache_atlas_janus
    2 row(s)
    Took 0.6872 seconds                 
    ["apache_atlas_entity_audit", "apache_atlas_janus"]
    ```

### 1.3. Настройка Ranger-политики для доступа УЗ  'atlas' в Apache HBase
1. Пример:
<table>
<tr><td>Policy Name</td><td>atlas</td></tr>
<tr><td>HBase Table</td><td>apache_atlas_entity_audit</br>apache_atlas_janus</td></tr>
<tr><td>HBase Column-family</td><td>*</td></tr>
<tr><td>HBase Column</td><td>*</td></tr>
<tr><td>Allow Conditions</td><td>atlas/udrvsts-adatls1t.adht1.corp/ADHT1.CORP: RWCA</td></tr>
</table>

## 2. Apache Hive
### 2.1. Настройка Ranger-политики для доступа УЗ  'atlas' в Apache HIve
1. В существующую политику 'all - database, table, column' добавляем условие с правами 'Select' и 'Read' для УЗ 'atlas'.

## 3. Apache Kafka
1. Apache Kafka используется Atlas'ом для получения сообщений о произошедших событиях в сервисах Hadoop'а. Сообщения отправляются с помощью специальных библиотек из состава Atlas'а, внедрённых в определённые сервисы. На данный момент Atlas читает сообщения о таких событиях в Hbase и Hive, как создание и удаление таблиц, добавления столбцов, бла-бла-бла...

### 3.1. Установка Apache Kafka
1. Установка и настройка Kafka выполняется по инструкции 01. ArenaData Hadoop. Добавление Apache Kafka в керберизованный ADH. Инструкция.

### 3.2. Добавление в Kafka необходимых топиков
1. Для работы Apache Atlas требуется наличие трёх топиков в Apache Kafka. Создаём их на машине с установленной Kafka:

    ```bash
    BOOTSTRAP="localhost:9091"
    # Список имён топиков по умолчанию:
    TOPIC1="_HOATLASOK"
    TOPIC2="ATLAS_ENTITIES"
    TOPIC3="ATLAS_HOOK"
    PATH=$PATH:/opt/kafka/bin/

    kafka-topics.sh --bootstrap-server ${BOOTSTRAP} --create --topic ${TOPIC1}
    kafka-topics.sh --bootstrap-server ${BOOTSTRAP} --create --topic ${TOPIC2}
    kafka-topics.sh --bootstrap-server ${BOOTSTRAP} --create --topic ${TOPIC3}
    ```

### 3.3. Проверка наличия топиков
1. Здесь же проверим наличие информации о топиках через `kafka-topics`:

    ```bash
    kafka-topics.sh --bootstrap-server ${BOOTSTRAP} --list
    ..
    ATLAS_ENTITIES
    ATLAS_HOOK
    _HOATLASOK
    ```

### 3.4. Настройка Ranger-политики для доступа УЗ  'atlas' в Apache Kafka
Вероятно, в Kafka используемой только для Atlas, нет нужды добавлять ranger-плагин.

## 4. Apache Solr
На тестовом кластере использовалась четыре узла Apache Solr. В общем случае, рекомендуется использовать не менее двух узлов.

### 4.1. Установка и настройка Apache Solr
Обычным способом через ADCM WebUI.

### 4.2. Добавление нового шаблона в Solr
Configuring Apache Solr as the indexing backend for the Graph Repository

Сейчас я могу зайти на zk-машиину и из-под рядового аккаунта, не получая Kerberos-тикет, запустить `zookeeper-client` и выполнить команду, например, `ls /`.

Из-под 'root'-а не могу. Kerberos-тикет не помогает.

На zookeeper-машине определяем zookeeper-ноду выделенную под работу Solr:
```bash
kinit nifantevea
ZKSERVER=$(hostname)
  
echo "ls /" | zookeeper-client -server ${ZKSERVER}
```

stdout:
```
...
[Arenadata.Hadoop-10.solr.server, arenadata, hadoop-ha, hbase, kafka, zookeeper]
Проверяем в Zookeeper'е текущий список конфигов:

echo "ls /Arenadata.Hadoop-10.solr.server/configs" | zookeeper-client -server ${ZKSERVER}
...
[zk: udrvsts-adzk1t.adht1.corp(CONNECTED) 0] ls /Arenadata.Hadoop-10.solr.server/configs
[_default]
```

На машине с установленным Atlas подготавливаем файл `atlasSolrConfig.zip`. Напомню, что в этом файле содержимое каталога `atlas/conf/solr/`:
```bash
kinit nifantevea
SOLRCONFIGDIR='/opt/atlas/conf/solr'
 
cd $SOLRCONFIGDIR
zip -r ~/tmp/atlasSolrConfig.zip *
```

Выясняем используемый Solr'ом порт для API:


Пока в ADH-кластере не включен TLS, используем http, вместо https.
Добавляем atlas-конфиг в Apache Solr:
```bash
FILENAME="/root/tmp/atlasSolrConfig.zip"
 
SOLRSERVER="http://udrvsts-adslr1t.adht1.corp:8983/solr"
 
# Отправляем конфиг в Solr через любой Solr-хост
curl --negotiate -u : -X POST --header "Content-Type:application/octet-stream" --data-binary @${FILENAME} "${SOLRSERVER}/admin/configs?action=UPLOAD&name=atlasTemplate"
```

На zookeeper-хосте проверяем факт, что конфиг появился в Zookeeper'е

Сначала выводим список кофигов:
```bash
ZKSERVER="$(hostname)
 
echo "ls /Arenadata.Hadoop-10.solr.server/configs" | zookeeper-client -server ${ZKSERVER}
```

stdout:
```
...
[_default, atlasTemplate]
```

Потом, убедившись в наличии конфига 'atlasTemplate', выводим его содержимое:
```bash
echo "ls /Arenadata.Hadoop-10.solr.server/configs/atlasTemplate" | \
  zookeeper-client -server ${ZKSERVER}
...
[currency.xml, lang, protwords.txt, schema.xml, solrconfig.xml, stopwords.txt, synonyms.txt]
```

После появления нового шаблона, создаём в Apache Solr три коллекции. Так как у нас четыре Solr-сервера, то выбираем 'numShards=2' и 'replicationFactor=2':
```bash
# Названия коллекций по умолчанию:
COLLECTION1="fulltext_index"
COLLECTION2="edge_index"
COLLECTION3="vertex_index"
 
curl --negotiate -u : "${SOLRSERVER}/admin/collections?action=CREATE&name=${COLLECTION1}&collection.configName=atlasTemplate&numShards=2&replicationFactor=2"
curl --negotiate -u : "${SOLRSERVER}/admin/collections?action=CREATE&name=${COLLECTION2}&collection.configName=atlasTemplate&numShards=2&replicationFactor=2"
curl --negotiate -u : "${SOLRSERVER}/admin/collections?action=CREATE&name=${COLLECTION3}&collection.configName=atlasTemplate&numShards=2&replicationFactor=2"
example stdout
{
  "responseHeader":{
    "status":0,
    "QTime":1875},
  "success":{
    "udrvsts-adslr1t.adht1.corp:8983_solr":{
      "responseHeader":{
        "status":0,
        "QTime":1344},
      "core":"vertex_index_shard1_replica_n1"}}}
```

Через Web UI Solr'а наблюдаем три ранее добавленные коллекции:






### 4.4. Настройка Ranger-политики для доступа УЗ  'atlas' в Apache Solr
Пример:

Policy Name	atlas
Solr Collection	fulltext_index
edge_index
vertex_index
Allow Conditions	atlas/udrvsts-adatls1t.adht1.corp@ADHT1.CORP: Query, Update, Others
