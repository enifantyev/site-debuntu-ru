---
title: "03. Выбор и подготовка поискового движка для использования Atlas'ом"
date: 2021-11-03
weight: 10
description: >
  Описание процесса установки Apache Atlas 2.2.0 в Cloudera CDH 6.3.2.
tags:
  - BigData
  - Apache Atlas
  - Cloudera CDH 6.3.2
slug: vybor-i-podgotovka-poiskovogo-dvizhka-dlya-ispolzovaniya-atlas
---

2021-08-10 – 2021-11-03

## 1. Выбор поискового движка: Apache Solr or Elasticsearch
1.1. Стандартно используется Apache Solr.

1.2. В расчёте на дальнейшую установку Amundsen'а, необходимо выбрать ElsaticSarch. На данный момент находится в разработке.

> Я попробую установить оба Index-сервиса с двумя экземплярами Atlas'а. Первый экземпляр Atlas'а будет использовать Apache Solr, а второй экземпляр Atlas'а, соответственно, Elasticsearch.<br>
<br>
Мысли под запись:<br>
УЗ 'atlas', как локальная, так и сервисная УЗ 'atlas/_HOST', будут использоваться обоими экземплярами Atlas'а.
Apache Kafka — один сервис, который будет обслуживать оба Atlas'а.
Apache HBase — создадим два набора таблиц для хранения сущностей.
У Atlas'а есть параметр 'atlas.cluster.name=primary'. Для второго экземпляра выбрать 'secondary'?

## 2. Apache Solr
На тестовом кластере использовалась четыре узла Apache Solr. В общем случае, рекомендуется использовать не менее двух узлов.

### 2.1. Установка и настройка Apache Solr
Установка и настройка Apache Solr выполняется по инструкции '21. Solr. Установка и настройка'.

### 2.2. Добавление нового шаблона в Solr
[Configuring Apache Solr as the indexing backend for the Graph Repository](https://atlas.apache.org/2.1.0/index.html#/Installation)

2.2.1. Проверяем в Zookeeper'е текущий список конфигов:
```
ZKSERVER="dev-zk110p.test2.lan"

echo "ls /solr/configs" | zookeeper-client -server ${ZKSERVER}
```

<details>
<summary>Пример stdout...</summary>

```
...
[zk: localhost:2181(CONNECTED) 17] ls /solr/configs
[managedTemplate, schemalessTemplate, _default, managedTemplateSecure, schemalessTemplateSecure]
```

</details>
<br>

2.2.2. Добавляем atlas-конфиг в Apache Solr.

>В переменной NXUSERPASS требуется указать правильный логин/пароль!

```
set +o history
NXUSERPASS="eugene:xxxxxxxxxxxxx"
set -o history
# Файл конфиг для Solr
FILENAME="atlasSolrConfig.zip"

SOLRSERVER="https://dev-app111p.test2.lan"

# Скачиваем сборку с Nexus'а
mkdir -p ~/tmp
cd ~/tmp
curl -LO -u ${NXUSERPASS} http://nexus.example.org:8081/repository/dud_evolut_raw/atlas/${FILENAME}

# Отправляем конфиг в Solr через любой Solr-хост
curl --negotiate -u : -X POST --header "Content-Type:application/octet-stream" --data-binary @${FILENAME} "${SOLRSERVER}:8985/solr/admin/configs?action=UPLOAD&name=atlasTemplate"
```

2.2.3. Проверяем факт, что конфиг появился в Zookeeper'е.

Сначала выводим список кофигов:
```
ZKSERVER="dev-zk110p.test2.lan"

echo "ls /solr/configs" | zookeeper-client -server ${ZKSERVER}
```

<details>
<summary>Пример stdout...</summary>

```
...
[zk: dev-zk111p(CONNECTED) 0] ls /solr/configs
[managedTemplate, schemalessTemplate, _default, managedTemplateSecure, atlasTemplate, schemalessTemplateSecure]
````

</details>
<br>

Потом, убедившись в наличии конфига 'atlasTemplate', выводим его содержимое:
```
ZKSERVER="dev-zk110p.test2.lan"

echo "ls /solr/configs/atlasTemplate" | zookeeper-client -server ${ZKSERVER}
```

<details>
<summary>Пример stdout...</summary>

```
...
[zk: dev-zk110p.test2.lan(CONNECTED) 0] ls /solr/configs/atlasTemplate
[currency.xml, protwords.txt, solrconfig.xml, synonyms.txt, stopwords.txt, schema.xml, lang]
```

</details>
<br>

После появления нового шаблона, создаём в Apache Solr три коллекции:
```
SOLRSERVER="dev-app111p.test2.lan"
# Названия коллекций по умолчанию:
COLLECTION1="fulltext_index"
COLLECTION2="edge_index"
COLLECTION3="vertex_index"

curl --negotiate -u : "https://${SOLRSERVER}:8985/solr/admin/collections?action=CREATE&name=${COLLECTION1}&collection.configName=atlasTemplate&numShards=2&replicationFactor=2"
curl --negotiate -u : "https://${SOLRSERVER}:8985/solr/admin/collections?action=CREATE&name=${COLLECTION2}&collection.configName=atlasTemplate&numShards=2&replicationFactor=2"
curl --negotiate -u : "https://${SOLRSERVER}:8985/solr/admin/collections?action=CREATE&name=${COLLECTION3}&collection.configName=atlasTemplate&numShards=2&replicationFactor=2"
```

<details>
<summary>Пример stdout...</summary>

{
  "responseHeader":{
    "status":0,
    "QTime":3663},
  "success":{
    "dev-zk112p.test2.lan:8985_solr":{
      "responseHeader":{
        "status":0,
        "QTime":1477},
      "core":"edge_index_shard1_replica_n1"},
    "dev-zk110p.test2.lan:8985_solr":{
      "responseHeader":{
        "status":0,
        "QTime":1962},
      "core":"edge_index_shard2_replica_n4"},
    "dev-app111p.test2.lan:8985_solr":{
      "responseHeader":{
        "status":0,
        "QTime":2289},
      "core":"edge_index_shard2_replica_n6"},
    "dev-zk111p.test2.lan:8985_solr":{
      "responseHeader":{
        "status":0,
        "QTime":2291},
</details>
<br>

Через Web UI Solr'а наблюдаем три ранее добавленные коллекции:
![Solr collections](/img/ustanovka-apache-atlas-2.2.0/solr-collections.png)

### 2.3. Добавление роли в Sentry для доступа Atlas'а в Solr
https://docs.cloudera.com/documentation/enterprise/6/6.3/topics/search_sentry.html#concept_iw1_5dp_wk

> Аккаунт, под которым производится добавление ролей в Sentry, должен быть упомянут (напрямую или через членство в группу)  в следующих параметрах Sentry-сервиса:<br>
sentry.service.admin.group<br>
sentry.service.allow.connect

2.3.1. Создаём роль 'solr4atlas_role':
```
SROLE="solr4atlas_role"
# Названия коллекций по умолчанию:
COLLECTION1="fulltext_index"
COLLECTION2="edge_index"
COLLECTION3="vertex_index"

solrctl sentry --create-role ${SROLE}

solrctl sentry --grant-privilege ${SROLE} "collection=${COLLECTION1}->action=*"
solrctl sentry --grant-privilege ${SROLE} "collection=${COLLECTION2}->action=*"
solrctl sentry --grant-privilege ${SROLE} "collection=${COLLECTION3}->action=*"

solrctl sentry --grant-privilege ${SROLE} "Config=${COLLECTION1}->action=*"
solrctl sentry --grant-privilege ${SROLE} "Config=${COLLECTION2}->action=*"
solrctl sentry --grant-privilege ${SROLE} "Config=${COLLECTION3}->action=*"
```

2.3.2. Назначим роль на группу 'atlas':
```
solrctl sentry --add-role-group ${SROLE} atlas
```

### 2.4. Проверка настроек Sentry для Solr
2.4.1. Список существующих ролей:
```
$ solrctl sentry --list-roles
...
solr_adm_role
kafka4atlas_role
hive_admins
solr4atlas_role
```

2.4.2. Вывод назначенных ролей на группу 'atlas':
```
$ solrctl sentry --list-roles -g atlas
...
solr4atlas_role
kafka4atlas_role
```

2.4.3. Вывод списка привилегий:
```
$ solrctl sentry --list-privileges solr4atlas_role
..
Config=vertex_index->action=*
Collection=edge_index->action=*
Collection=fulltext_index->action=*
Config=edge_index->action=*
Config=fulltext_index->action=*
Collection=vertex_index->action=*
```

3.4.4. При необходимости удаляем лишние привилегии командой, пример:
```
$ solrctl sentry --revoke-privilege solr4atlas_role 'Config=${collection1}->action=*'
```

> Продолжение инструкции находится в стадии разработки и не требует выполнения!

## 3. Установка Elasticsearch 6.x (на стадии разработки)
> Здесь рассмотрена бесплатная версия Elasticsearch, которая не имеет какого-либо способа аутентификации?

> В расчёте на последующие попытки подключения Amundsen, устанавливаем Elasticsearch 6-ой версии, а не 7-ой. Elasticsearch 6.x указана, как поддерживаемая в требованиях Amundsen.

### 3.1. Добавление репо-файла
3.1.1. Выполняем из-под 'root':
```
USERNAME=nx-pubrepo-r
PASSWORD=eu6yWX6jfzUebShTwF1Qa434hovhm4

cat << EOF > /etc/yum.repos.d/elasticsearch.repo
[elasticsearch7]
username=$USERNAME
password=$PASSWORD
name=Elasticsearch repository for 7.x packages
baseurl=http://nexus.example.org:8081/repository/all_elasticsearch_yum/packages/7.x/yum
gpgcheck=1
gpgkey=http://nexus.example.org:8081/repository/all_elasticsearch_yum/GPG-KEY-elasticsearch
enabled=0
autorefresh=1
type=rpm-md

[elasticsearch6]
username=$USERNAME
password=$PASSWORD
name=Elastic repository for 6.x packages
baseurl=http://nexus.example.org:8081/repository/all_elasticsearch_yum/packages/6.x/yum
gpgcheck=1
gpgkey=http://nexus.example.org:8081/repository/all_elasticsearch_yum/GPG-KEY-elasticsearch
enabled=0
autorefresh=1
type=rpm-md
EOF
```

### 3.2. Устанавливаем Elasticsearch
3.2.1. Включаем репо для Elasticsearch 6.x и выполняем установку пакета определённой версии, с которым производилась сборка Atlas'а (не знаю, насколько критично точное совпадение используемых версий):
```
yum install -y --enablerepo=elasticsearch6 elasticsearch-6.8.20-1
```

### 3.3. Настройка конфигурации
3.3.1. Изменяем следующие параметры:
```
path.data: /data/elasticsearch

path.logs: /data/log/elasticsearch

# По умолчанию, Elasticsearch привязывается только в 127.0.0.1
network.host: 10.1.137.11
```

### 3.4. Запуск
3.4.1. Запускаем Elasticsearch:
```
systemctl start elasticsearch
```
