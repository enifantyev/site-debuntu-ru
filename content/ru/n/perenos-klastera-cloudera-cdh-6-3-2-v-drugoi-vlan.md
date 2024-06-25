---
title: "Перенос кластера Cloudera CDH 6.3.2 в другой vlan"
date: 2020-10-21
weight: 10
description: >
  Инструкция по работе с кластером в случае изменения ip-адресов.
tags:
  - Cloudera CDH 6.3.2
  - BigData
---

2020-10-21

Работа производилась на Cloudera CDH 6.3.2. Используется встроенный PostgreSQL.

1. Остановить кластер.
1. Остановить Cloudera Service Management.
1. На всех хостах остановить агенты:<br>
  `systemctl stop cloudera-scm-agent`<br>
  `systemctl disable cloudera-scm-agent`
1. На управляющем хосте остановить службы Cloudera Server и встроенный PostgreSQL:<br>
  `systemctl stop cloudera-scm-server`<br>
  `systemctl disable cloudera-scm-server`<br>
  `systemctl stop cloudera-scm-server-db`<br>
  `systemctl disable cloudera-scm-server-db`

<br>
После переезда кластера в новый vlan и назначении на сетевые интерфейсы новых ip-адресов проделать:

1. На всех узлах привести записи в /etc/hosts к актуальному состоянию.
1. Если кластер в домене IPA, а автоматическое обновление DNS-записей не произошло, то добавить в файл /etc/sssd/sssd.conf опцию `ipa_dyndns_update = True` и перезапустить демон `systemctl restart sssd`.
1. На всех хостах в файле /etc/cloudera-scm-agent/config.ini указать новый адрес CM-сервера, пример: `server_host=10.0.0.1`.
1. Включить на всех хостах агенты:<br>
  ` systemctl enable --now cloudera-scm-agent`
1. Включить на mgm-хосте встроенный PostgreSQL и Cloudera Server:<br>
  `systemctl enable cloudera-scm-server-db`<br>
  `systemctl start cloudera-scm-server-db`<br>
  `systemctl enable --now cloudera-scm-server`

<br>
Переходим к WEB-интерфейсу, где:

1. запустить "Cloudera Management Service";
1. применить изменения в конфигурациях сервисов;
1. запустить тесты "Inspect All Hosts" и "Inspect Network Performance";
1. запустить кластер.
