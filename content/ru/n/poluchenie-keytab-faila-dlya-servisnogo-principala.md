---
title: "Получение keytab-файла для сервисного принципала"
date: 2021-04-01
weight: 10
description: >
  Для полного доступа к керберизированному содержимому Hadoop кластера необходимо получить соответствующие билеты. Для этого используются таблицы ключей.
tags:
  - FreeIPA
  - Kerberos
  - keytab
  - Apache Hadoop
  - Cloudera CDH 6.3.2
---

2021-04-01

Ссылки:
https://www.freeipa.org/page/V4/Keytab_Retrieval_Management
https://www.freeipa.org/page/V4/Keytab_Retrieval

Например, находясь на узле со службой, для которой нам необходимо получить keytab, сначала, если ещё не получили, то получаем тикет:
```
$ kinit $USER
```

Назначаем себе право получения таблицу ключей:
```
$ ipa service-allow-retrieve-keytab hbase/$(hostname) --users=$USER
  Имя учётной записи: hbase/prod-hbr01p.example.org@EXAMPLE.ORG
  Псевдоним учётной записи: hbase/prod-hbr01p.example.org@EXAMPLE.ORG
  Managed by: prod-hbr01p.example.org
  Пользователи, которым разрешено получать таблицу ключей: eugene
```

Получаем таблицу ключей и сохраняем её в файле:
```
$ ipa-getkeytab -r -p hbase/$(hostname) -k ~/keytabs/hbase_$(hostname -s).keytab
  Keytab successfully retrieved and stored in: /home/eugene/keytabs/hbase_prod-hbr01p.keytab
```

Всё ещё имея свой персональный kerberos-билет, отнимаем у себя право получения таблицы ключей:
```
ipa service-disallow-retrieve-keytab hbase/$(hostname) --users=$USER
```

Используем полученный keytab для выпуска билета:
```
$ kinit -kt ~/keytabs/hbase_prod-hbr01p.keytab hbase/$(hostname)
```
