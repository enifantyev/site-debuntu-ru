---
title: "Администрирование таблиц в Apache HBase"
date: 2020-11-14
weight: 10
description: >
  Описание обычных работ администратора с таблицами в Apache HBase.
tags:
  - Apache HBase
  - Apache Hadoop
  - BigData
  - HDFS
aliases:
  - /a/administrirovanie-tablic-v-apache-hbase/
slug: administrirovanie-tablic-v-apache-hbase
---

2020-11-14

## Работа с таблицами HBase
Работу с таблицами будем производить через CLI-утилиту `hbase`. Есть два способа выполнения команд с `hbase`.

Первый способ заключается в запуске `hbase shell` и интерактивной подаче команд:
```
$ hbase shell
hbase(main):001:0> list
```

Второй способ позволяет использовать `hbase` в скриптах:
```
echo "list" | hbase shell
```

Если кластер керберизированный, то перед работой с сервисами кластера получаем Kerberos-билет с помощью `kinit` или из соответствующего 'keytab':
```
kinit -kt hbase.keytab hbase/$(hostname)
```

Если кластер некерберизированный и не хватает прав для выполнения операций, описанных в этой памятке, то ...

Пытаемся подать команду через `sudo -u hdfs`:
```
sudo -u hdfs hdfs dfs -rm ...
```

Или с помощью `HADOOP_USER_NAME=hdfs`:
```
HADOOP_USER_NAME=hdfs hbase org.apache.hadoop.hbase.mapreduce.Import ...
```

### Экспорт данных из таблицы HBase в HDFS на основе mapreduce
1. Данные из таблицы экспортируем в отдельную папку HDFS.
2. Переносим экспортированные данные в локальную FS.
3. Удаляем экспортированные данные из HDFS.

```
CURDATE=$(date +%y%m%d-%H%M%S)

hbase org.apache.hadoop.hbase.mapreduce.Export 'MyTableName' /tmp/MyTableName_Export_${CURDATE}

hdfs dfs -copyToLocal /tmp/MyTableName_Export_${CURDATE} /data/tmp/

hdfs dfs -rm -r -f -skipTrash /tmp/MyTableName_Export_${CURDATE}
```

### Создание пустой таблицы в HBase
При создании новой таблицы в HBase, второй параметр определяет 'COLUMN FAMILIES'. При необходимости, кроме названия, можно указывать дополнительные опции в 'COLUMN FAMILIES'.
```
echo "create 'MyTableName', 'data'"  | hbase shell

echo "create 'MyTableName', {NAME => 'data', COMPRESSION => 'GZ'}" | hbase shell
```

### Описание COLUMN FAMILIES
Узнать 'COLUMN FAMILIES' из описания к существующей таблице.
```
$ hbase shell
hbase(main):001:0> describe 'SomeTableName'

Table SomeTableName is ENABLED
SomeTableName
COLUMN FAMILIES DESCRIPTION
{NAME => 'data', BLOOMFILTER => 'ROW', VERSIONS => '1', IN_MEMORY => 'false', KEEP_DELETED_CELLS => 'FALSE',
 DATA_BLOCK_ENCODING => 'NONE', TTL => 'FOREVER', COMPRESSION => 'GZ', MIN_VERSIONS => '0', BLOCKCACHE => 't
rue', BLOCKSIZE => '65536', REPLICATION_SCOPE => '0'}
1 row(s) in 0.3900 seconds
```

### Способы импорта данных из HDFS в таблицу HBase
Перед процедурой импорта, размещаем импортируемые данные в папке HDFS.
```
hdfs dfs -copyFromLocal /tmp/MyTableName_Export_201114-123315 /tmp
```

Импортируемые данные не уничтожат существующие данные в таблице. Уничтожение данных производится через `echo "truncate 'myTableName'" | hbase shell`

#### 1. Импорт методом Import
При этом способе импорта, данные заливаются из HDFS напрямую в HBase. Это удобно, но, к сожалению, иногда, процесс импорта прерывается ошибками. В этом случае успешно работает [CompleteBulkLoad](http://hbase.apache.org/book.html#completebulkload).
```
hbase org.apache.hadoop.hbase.mapreduce.Import 'MyTableName' /tmp/MyTableName_Export_201114-123315
```

#### 2. Импорт с помощью CompleteBulkLoad (LoadIncrementalHFiles)
При этом способе импорта, [CompleteBulkLoad](http://hbase.apache.org/book.html#completebulkload), данные предварительно преобразовываются в формат HBase, сохраняясь в отдельной HDFS-папке. Процесс этот довольно продолжительный. Но далее, второй командой, подготовленная папка мгновенно переносится в хранилище HBase.
```
hbase org.apache.hadoop.hbase.mapreduce.Import -Dimport.bulk.output=/tmp/H_TableName_tmp 'TableName' /tmp/TableName_Export_201114-123315

hbase org.apache.hadoop.hbase.tool.LoadIncrementalHFiles /tmp/H_TableName_tmp TableName
```

### Список таблиц
```
echo "list" | hbase shell
```

### Удаление содержимого таблицы
```
echo "truncate 'myTableName'" | hbase shell
```
На время подрезки данных, таблица автоматически переводится в режим 'disabled'.

### Удаление таблицы
```
echo "disable 'MyTableName'" | hbase shell
echo "drop 'MyTableName'" | hbase shell
```

## Работа с правами доступа
[Configuring HBase Authorization | Cloudera Enterprise 6.3.x](https://docs.cloudera.com/documentation/enterprise/6/6.3/topics/cdh_sg_hbase_authorization.html#concept_enm_hhx_yp)

### Получение списка с правами доступа
```
user_permission '.*' # Все таблицы
user_permission 'MyTable' # Для одной таблицы
```

### Предоставление прав доступа
```
grant 'eugene', 'RWXCA', 'MyTable'
```

### Отзыв прав доступа
```
revoke 'eugene', 'MyTable'
```

## Работа со снимками

### Список снимков
```
echo "list_snapshots" | hbase shell
```

### Создание снимка
```
CURDATE=$(date +%y%m%d-%H%M%S)

echo "snapshot 'MyTableName', 'MyTableName_Snapshot_$CURDATA'" | hbase shell
```

### Экспорт снимка в hdfs папку
Создаём в HDFS каталог /tmp/hbase_snapshots и экспортируем туда снимок:
```
hdfs dfs -mkdir -p /tmp/hbase_snapshots
hbase org.apache.hadoop.hbase.snapshot.ExportSnapshot -snapshot MyTableName_Snapshot -copy-to /tmp/hbase_snapshots
```

В результате, в `/tmp/hbase_snapshots` наблюдаем готовую структуру для переноса в другой кластер в hdfs каталог `/hbase`:
```
hdfs dfs -ls /tmp/hbase_snapshots
Found 2 items
drwxrwx---   - hbase supergroup          0 2021-04-01 14:29 /tmp/hbase_snapshots/.hbase-snapshot
drwxrwx---   - hdfs  supergroup          0 2021-04-01 14:29 /tmp/hbase_snapshots/archive
```

После переноса этого каталога в другой кластер, содержимое папок `.hbase-snapshot` и `archive` копируем в такие же папки в каталоге `/hbase`, после чего в списке снимков появится искомый снимок, к которому можно применить команду `restore_snapshot`. Нет нужды создавать новую таблицу для этого снимка. Новая таблица будет создана автоматически, как точная копия таблицы, с которой делался снимок.

### Восстановление снимка
```
echo "disable 'MyTableName'" | hbase shell
echo "restore_snapshot 'MyTableName_Snapshot_201114-141940'" | hbase shell
echo "enable 'MyTableName'" | hbase shell
```

### Клонирование таблицы из снимка
```
echo "clone_snapshot 'MyTableName_Snapshot_201114-141940', 'MyOtherTableName'" | hbase shell
```

### Удаление снимка
```
echo "delete_snapshot 'MyTableName_Snapshot_201114-141940'" | hbase shell
```
