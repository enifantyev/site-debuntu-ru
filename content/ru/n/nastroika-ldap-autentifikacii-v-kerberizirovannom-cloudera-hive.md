---
title: "Настройка LDAP-аутентификации Hive в керберизированном Cloudera CDH 6.3.2"
date: 2020-11-08
weight: 10
description: >
  Настройка LDAP-аутентификации Hive в керберизированном Cloudera CDH 6.3.2 проста и незатейлива.
tags:
  - Apache Hive
  - Apache Hadoop
  - Cloudera CDH 6.3.2
  - BigData
---

## Ссылки
[HiveServer2 Security Configuration](https://docs.cloudera.com/documentation/enterprise/latest/topics/cdh_sg_hiveserver2_security.html)

## Настройка LDAP-аутентификации
На странице Configurations для Hive, используем фильтр ldap, и:

1. Включаем LDAP
    **Enable LDAP Authentication  Hive (Service-Wide)**: &#9745;
2. Указываем адрес LDAP-сервера
    **LDAP URL**
    hive.server2.authentication.ldap.url: `ldaps://ldap1.example.org ldaps://ldap2.example.org ldaps://ldap3.example.org`
3. Указываем контейнер поиска пользователей
    **LDAP BaseDN**
    hive.server2.authentication.ldap.baseDN: `cn=users,cn=accounts,dc=example,dc=org`
![Cloudera Hive LDAP](/img/nastroika-ldap-autentifikacii-v-kerberizirovannom-cloudera-hive/cloudera_hive_ldap.png)
