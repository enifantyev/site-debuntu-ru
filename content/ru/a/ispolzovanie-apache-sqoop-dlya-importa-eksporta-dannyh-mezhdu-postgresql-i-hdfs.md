---
title: "Использование Apache Sqoop для импорта/экспорта данных между PostgreSQL и HDFS"
date: 2020-11-01
weight: 10
description: >
  Apache Sqoop предназначен для импорта/экспорта данных между RDBSM и Apache Hadoop. В данной статье используется керберизированный Hadoop керберезирован и находящийся во FreeIPA домене.
tags:
  - Apache Sqoop
  - Apache Hadoop
  - HDFS
  - PostgreSQL
  - BigData
---

2020-11-01

Использованные материалы:
- [Sqoop User Guide (v1.4.6)](https://sqoop.apache.org/docs/1.4.6/SqoopUserGuide.html)
- [An HDFS Tutorial for Data Analysts Stuck with Relational Databases](https://www.toptal.com/database/hdfs-tutorial-data-migration-from-postgresql)

## Импорт из PostgreSQL в HDFS
### Подготовка к импорту
PostgreSQL должен слушать на внешнем порту, чтобы к нему была возможность подключиться по сети. Если в PostgreSQL используется сетевой порт, отличный от стандартного (5432), то его необходимо указывать в подаваемых командах.

Выполнять 'sqoop import'  не обязательно с того хоста, у которого есть роль 'Sqoop'. Он работает с любого хоста кластера, у которого есть соответствующий gateway в тот сервис, куда импортируются/экспортируются данные. Подтверждено.

Добавляем в /var/lib/sqoop java-библиотеку [postgresql-42.2.18.jar](https://jdbc.postgresql.org/download/postgresql-42.2.18.jar).

Создаём базу данных, пользователя для работы с ней, и сохраняем информацию для подключения к базе в специальный файл '.pgpass':
```
cd ~
sudo -u postgres psql -c 'CREATE DATABASE sqoop_test_import_db;'
password=$(cat /dev/urandom | tr -dc A-Za-z0-9 | head -c15)
sudo -u postgres psql -c "CREATE USER sqoop_test_u WITH password '$password';"
sudo -u postgres psql -c "GRANT ALL PRIVILEGES ON DATABASE sqoop_test_import_db TO sqoop_test_u;"
echo "10.1.112.99:5432:sqoop_test_import_db:sqoop_test_u:${password}" > ~/.pgpass;
chmod 600 ~/.pgpass
```

Создаём таблицу 'sales' и заполняем информацией:
```
psql -d sqoop_test_import_db -U sqoop_test_u -h 10.1.112.99 \
-c 'CREATE TABLE sales (pkSales integer primary key, saleDate date, saleAmount money, orderID int, itemID int);'

psql -d sqoop_test_import_db -U sqoop_test_u -h 10.1.112.99 -c "
insert into sales values (1, '2016-09-27', 1.23, 1, 1);
insert into sales values (2, '2016-09-27', 2.34, 1, 2);
insert into sales values (3, '2016-09-27', 1.23, 2, 1);
insert into sales values (4, '2016-09-27', 2.34, 2, 2);
insert into sales values (5, '2016-09-27', 3.45, 2, 3);
insert into sales values (6, '2016-09-28', 3.45, 3, 3);
insert into sales values (7, '2016-09-28', 4.56, 3, 4);
insert into sales values (8, '2016-09-28', 5.67, 3, 5);
insert into sales values (9, '2016-09-28', 1.23, 4, 1);
insert into sales values (10, '2016-09-28', 1.23, 5, 1);
insert into sales values (11, '2016-09-28', 1.23, 6, 1);
insert into sales values (12, '2016-09-29', 1.23, 7, 1);
insert into sales values (13, '2016-09-29', 2.34, 7, 2);
insert into sales values (14, '2016-09-29', 3.45, 7, 3);
insert into sales values (15, '2016-09-29', 4.56, 7, 4);
insert into sales values (16, '2016-09-29', 5.67, 7, 5);
insert into sales values (17, '2016-09-29', 6.78, 7, 6);
insert into sales values (18, '2016-09-29', 7.89, 7, 7);
insert into sales values (19, '2016-09-29', 7.89, 8, 7);
insert into sales values (20, '2016-09-30', 1.23, 9, 1);
"
```

Проверяем факт наличия новой таблицы:
```
psql -d sqoop_test_import_db -U sqoop_test_u -h 10.1.112.99 -c 'table sales;'
 pksales |  saledate  | saleamount | orderid | itemid
---------+------------+------------+---------+--------
       1 | 2016-09-27 |      $1.23 |       1 |      1
       2 | 2016-09-27 |      $2.34 |       1 |      2
       3 | 2016-09-27 |      $1.23 |       2 |      1
       4 | 2016-09-27 |      $2.34 |       2 |      2
       5 | 2016-09-27 |      $3.45 |       2 |      3
       6 | 2016-09-28 |      $3.45 |       3 |      3
       7 | 2016-09-28 |      $4.56 |       3 |      4
       8 | 2016-09-28 |      $5.67 |       3 |      5
       9 | 2016-09-28 |      $1.23 |       4 |      1
      10 | 2016-09-28 |      $1.23 |       5 |      1
      11 | 2016-09-28 |      $1.23 |       6 |      1
      12 | 2016-09-29 |      $1.23 |       7 |      1
      13 | 2016-09-29 |      $2.34 |       7 |      2
      14 | 2016-09-29 |      $3.45 |       7 |      3
      15 | 2016-09-29 |      $4.56 |       7 |      4
      16 | 2016-09-29 |      $5.67 |       7 |      5
      17 | 2016-09-29 |      $6.78 |       7 |      6
      18 | 2016-09-29 |      $7.89 |       7 |      7
      19 | 2016-09-29 |      $7.89 |       8 |      7
      20 | 2016-09-30 |      $1.23 |       9 |      1
(20 rows)
```

### Импорт
- Так как в YARN для default-очереди выделено ноль ресурсов, то буду использовать одну из трёх доступных очередей с помощь опции '-Dmapreduce.job.queuename='.
- Дополнительно, в Глобальном Каталоге, я добавил себя в группу 'yarn-queue-dev', участники которой имею право на использование этой очереди. Не совсем понял ситуации и, к сожалению, не отследил, но очередь для меня стала доступна не сразу после включения в группу, а, примерно, после полчаса-часа ожидания.

Пытаемся импортировать таблицу sales в свой домашний каталог HDFS:
```
kinit

sqoop import -Dmapreduce.job.queuename="root.yarn-queue-dev" \
--connect 'jdbc:postgresql://10.1.112.99/sqoop_test_import_db' \
--username 'sqoop_test_u' -P \
--table 'sales' \
--target-dir 'SqoopTestSales' \
--split-by 'pksales'
```

### Результат импорта в HDFS
```
$ hdfs dfs -ls SqoopTestSales
Found 5 items
-rw-rw----   2 eugene eugene          0 2020-11-06 17:03 SqoopTestSales/_SUCCESS
-rw-rw----   2 eugene eugene        110 2020-11-06 17:03 SqoopTestSales/part-m-00000
-rw-rw----   2 eugene eugene        111 2020-11-06 17:03 SqoopTestSales/part-m-00001
-rw-rw----   2 eugene eugene        115 2020-11-06 17:03 SqoopTestSales/part-m-00002
-rw-rw----   2 eugene eugene        115 2020-11-06 17:03 SqoopTestSales/part-m-00003
$ hdfs dfs -cat SqoopTestSales/part-m-00000
1,2016-09-27,1.23,1,1
2,2016-09-27,2.34,1,2
3,2016-09-27,1.23,2,1
4,2016-09-27,2.34,2,2
5,2016-09-27,3.45,2,3
$ hdfs dfs -cat SqoopTestSales/part-m-00001
6,2016-09-28,3.45,3,3
7,2016-09-28,4.56,3,4
8,2016-09-28,5.67,3,5
9,2016-09-28,1.23,4,1
10,2016-09-28,1.23,5,1
$ hdfs dfs -cat SqoopTestSales/part-m-00002
11,2016-09-28,1.23,6,1
12,2016-09-29,1.23,7,1
13,2016-09-29,2.34,7,2
14,2016-09-29,3.45,7,3
15,2016-09-29,4.56,7,4
$ hdfs dfs -cat SqoopTestSales/part-m-00003
16,2016-09-29,5.67,7,5
17,2016-09-29,6.78,7,6
18,2016-09-29,7.89,7,7
19,2016-09-29,7.89,8,7
20,2016-09-30,1.23,9,1
```

## Экспорт данных из HDFS в PostgreSQL
### Подготовка к экспорту
Создаём новую DB и даём на неё права ранее созданному пользователю sqoop_test_u:
```
cd ~
password=$(awk -F: '{print $5}' ~/.pgpass)
sudo -u postgres psql -c 'CREATE DATABASE sqoop_test_export_db;'
sudo -u postgres psql -c "GRANT ALL PRIVILEGES ON DATABASE sqoop_test_export_db TO sqoop_test_u;"
echo "10.1.112.99:5432:sqoop_test_export_db:sqoop_test_u:${password}" >> ~/.pgpass;
```

Создаём таблицу для экспорта данных из HDFS. Столбцу 'saleAmount' задал тип numeric, так как с типом 'money' экспорт прерывался ошибкой.
```
psql -d sqoop_test_export_db -U sqoop_test_u -h 10.1.112.99 \
-c 'CREATE TABLE sales (pkSales int primary key, saleDate date, saleAmount numeric, orderID int, itemID int);'
```

### Проверка
Видит ли Sqoop базу данных:
```
sqoop list-databases \
--connect 'jdbc:postgresql://10.1.112.99/sqoop_test_export_db' \
--username 'sqoop_test_u' -P

postgres
template1
template0
sqoop_test_import_db
sqoop_test_export_db
```

Видит ли Sqoop таблицы:
```
sqoop list-tables \
--connect 'jdbc:postgresql://10.1.112.99/sqoop_test_export_db' \
--username 'sqoop_test_u' -P

sales
```

### Экспорт
```
sqoop export -Dmapreduce.job.queuename="root.yarn-queue-dev" \
--connect 'jdbc:postgresql://10.1.112.99/sqoop_test_export_db' \
--username 'sqoop_test_u' -P \
--table 'sales' \
--export-dir 'SqoopTestSales'
```

### Результат экспорта из HDFS в PostgreSQL
```
psql -d sqoop_test_export_db -U sqoop_test_u -h 10.1.112.99 -c 'table sales order by pksales;'
 pksales |  saledate  | saleamount | orderid | itemid
---------+------------+------------+---------+--------
       1 | 2016-09-27 |       1.23 |       1 |      1
       2 | 2016-09-27 |       2.34 |       1 |      2
       3 | 2016-09-27 |       1.23 |       2 |      1
       4 | 2016-09-27 |       2.34 |       2 |      2
       5 | 2016-09-27 |       3.45 |       2 |      3
       6 | 2016-09-28 |       3.45 |       3 |      3
       7 | 2016-09-28 |       4.56 |       3 |      4
       8 | 2016-09-28 |       5.67 |       3 |      5
       9 | 2016-09-28 |       1.23 |       4 |      1
      10 | 2016-09-28 |       1.23 |       5 |      1
      11 | 2016-09-28 |       1.23 |       6 |      1
      12 | 2016-09-29 |       1.23 |       7 |      1
      13 | 2016-09-29 |       2.34 |       7 |      2
      14 | 2016-09-29 |       3.45 |       7 |      3
      15 | 2016-09-29 |       4.56 |       7 |      4
      16 | 2016-09-29 |       5.67 |       7 |      5
      17 | 2016-09-29 |       6.78 |       7 |      6
      18 | 2016-09-29 |       7.89 |       7 |      7
      19 | 2016-09-29 |       7.89 |       8 |      7
      20 | 2016-09-30 |       1.23 |       9 |      1
(20 rows)
```

## Зачистка после испытаний
### Удаление базы данных  и пользователя из PostgreSQL
```
sudo -u postgres psql -c 'DROP DATABASE sqoop_test_import_db;'
sudo -u postgres psql -c 'DROP DATABASE sqoop_test_export_db;'
sudo -u postgres psql -c 'DROP USER sqoop_test_u;'
rm -f ~/.pgpass
```

### Удаление каталога SqoopTestSales из HDFS
```
kinit

hdfs dfs -rm -r -f -skipTrash SqoopTestSales .staging

kdestroy
```
