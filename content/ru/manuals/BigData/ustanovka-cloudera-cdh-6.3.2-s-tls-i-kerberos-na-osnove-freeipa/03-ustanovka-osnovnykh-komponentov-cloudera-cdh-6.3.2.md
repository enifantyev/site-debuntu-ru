---
title: "03. Установка основных компонентов Cloudera CDH 6.3.2"
date: 2021-06-16
weight: 3
description: >
  В этой части описывается установка основных компонентов Cloudera CDH 6.3.2.
tags:
  - Cloudera CDH 6.3.2
  - Apache Hadoop
  - CentOS
  - CentOS 7
  - BigData
slug: ustanovka-osnovnykh-komponentov-cloudera-cdh-6.3.2
---

2021-06-16

Может быть будет удобнее, если установить основные пакеты для Hadoop'а вручную, как описано на странице Ручная установка набора Cloudera-пакетов.

## Welcome
После входа на веб-страницу Cloudera Manager http://prod-mgm01p.example.org:7180/ соглашаемся с пользовательским соглашением и выбираем бесплатную лицензию Cloudera Express.

## Cluster Basics
Указываем имя кластера. Это имя не имеет никаких зависимостей в работе кластера и в любой момент может быть изменено. Cloudera Manager позволяет управлять несколькими кластерами, каждый из которых имеет своё имя.
![](/img/ustanovka-cloudera-cdh-6.3.2-s-tls-i-kerberos-na-osnove-freeipa/image_3_1.png)

## Specify host
Вводим список FQDN-имён всех узлов будущего кластера, включая узел на котором установлен CLoudera Manager.

> ![](/img/dialog-warning.png) <span style="color: red">Запрещено указывать ip-адреса, так как работа Kerberos и TLS основана на FQDN-именах.</span>

![](/img/ustanovka-cloudera-cdh-6.3.2-s-tls-i-kerberos-na-osnove-freeipa/image_3_2.png)

## Select Repository
Указываем корпоративный nexus-репозиторий cloudera-manager для установки Cloudera Manager Agent на все узлы кластера и репозиторий cloudera-cdh для установки на определённые узлы.

Вместо автоматически вставленного в поле репозитория 'http://nxrm.example.org:8081'  указываем полный URL:<br>
`http://nx-pubrepo-reader:rAhbjZh0z3LCCjUhgqNdA@nxrm.example.org:8081/repository/cloudera-cm-6.3.1-hosted/`
![](/img/ustanovka-cloudera-cdh-6.3.2-s-tls-i-kerberos-na-osnove-freeipa/image_3_3.png)

Ниже, для CDH, указываем репозиторий для пакетов CDH:<br>
`http://nx-pubrepo-reader:iWnZ0Ho7JNV03df1zgEj6@nxrm.example.org:8081/repository/cloudera-cdh-6.3.2-hosted/`
![](/img/ustanovka-cloudera-cdh-6.3.2-s-tls-i-kerberos-na-osnove-freeipa/image_3_4.png)

## Enter Login Credentials
Указываем пользователя имеющего root-права на всех хостах будущего кластера. Вместо пароля  пользователя вводим его приватный ssh-ключ:
![](/img/ustanovka-cloudera-cdh-6.3.2-s-tls-i-kerberos-na-osnove-freeipa/image_3_5.png)

## Установка Агентов
Здесь наблюдаем параллельную установку Cloudera Agents и всех пакетов Cloudera CDH на все хосты будущего кластера. Так как сразу же устанавливается всё необходимое ПО, то в дальнейшем, при добавлении выбранной роли на какой-то узел, сразу же будет запущен соответствующий демон, без дополнительной установки каких-либо rpm-пакетов.

---
На этом этапе могут происходить ошибки "Installation failed. Failed to authenticate.", где в Details обнаруживаем объяснение "Exhausted available authentication methods".
![](/img/ustanovka-cloudera-cdh-6.3.2-s-tls-i-kerberos-na-osnove-freeipa/image_3_6.png)
<br>

<details><summary>/var/log/cloudera-scm-server/cloudera-scm-server.log...</summary>

```
2021-06-19 17:15:22,427 INFO NodeConfiguratorThread-0-0:com.cloudera.server.cmf.node.NodeConfiguratorProgress: dev-aux90p.test.lan: Transitioning from AUTHENTICATE (PT0.292S) to MAKE_TEMP_DIR
2021-06-19 17:15:22,471 ERROR NodeConfiguratorThread-0-1:net.schmizz.concurrent.Promise: <<authenticated>> woke to: net.schmizz.sshj.userauth.UserAuthException: Unknown packet received during publickey auth: GLOBAL_REQUEST
2021-06-19 17:15:22,471 WARN NodeConfiguratorThread-0-1:com.cloudera.server.cmf.node.NodeConfigurator: Could not authenticate to dev-aux91p.test.lan
net.schmizz.sshj.userauth.UserAuthException: Exhausted available authentication methods
        at net.schmizz.sshj.SSHClient.auth(SSHClient.java:232)
        at net.schmizz.sshj.SSHClient.auth(SSHClient.java:208)
        at com.cloudera.server.cmf.node.NodeConfigurator.connect(NodeConfigurator.java:416)
        at com.cloudera.server.cmf.node.NodeConfigurator.configure(NodeConfigurator.java:1028)
        at com.cloudera.server.cmf.node.NodeConfigurator.run(NodeConfigurator.java:1106)
        at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
        at java.util.concurrent.FutureTask.run(FutureTask.java:266)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
        at java.lang.Thread.run(Thread.java:748)
Caused by: net.schmizz.sshj.userauth.UserAuthException: Unknown packet received during publickey auth: GLOBAL_REQUEST
        at net.schmizz.sshj.userauth.method.AbstractAuthMethod.handle(AbstractAuthMethod.java:52)
        at net.schmizz.sshj.userauth.method.AuthPublickey.handle(AuthPublickey.java:47)
        at net.schmizz.sshj.userauth.UserAuthImpl.handle(UserAuthImpl.java:136)
        at net.schmizz.sshj.transport.TransportImpl.handle(TransportImpl.java:490)
        at net.schmizz.sshj.transport.Decoder.decode(Decoder.java:107)
        at net.schmizz.sshj.transport.Decoder.received(Decoder.java:175)
        at net.schmizz.sshj.transport.Reader.run(Reader.java:60)
2021-06-19 17:15:22,482 INFO NodeConfiguratorThread-0-1:com.cloudera.server.cmf.node.NodeConfiguratorProgress: dev-aux91p.test.lan: Setting AUTHENTICATE as failed and done state
2021-06-19 17:15:22,482 INFO NodeConfiguratorThread-0-1:net.schmizz.sshj.transport.TransportImpl: Disconnected - BY_APPLICATION
```
</details>
<br>
Не пытаемся нажимать кнопку "Retry", так как в этом случае, как я убедился на собственном опыте, после многократного нажимания "Retry" процесс установки всё-таки может запуститься, но, с большой вероятностью, визард не сможет завершиться успешно. На одном из следующих этапов визарда, нажатие кнопки "Next" будет приводить к выводу безымянного баннера с содержимым из двух символов "{ }" и далее ничего происходить не будет.  В этом случае прекращаем попытки продолжить выполнение визарда, принудительно заходим в Cloudera Manager Admin Console и вновь запускаем процедуру добавления кластера.

Сейчас же дожидаемся окончания процесса с другими хостами и только тогда нажимаем "Retry Failed Hosts". В этом случае, с большой вероятностью, визард сможет успешно завершиться.

## Inspect Cluster
Перед проверкой Inspect Network Performance и Inspect Hosts, исправляем на всех машинах шебанг в файле `/opt/cloudera/cm-agent/service/perf/host_perf_diag.py`, иначе проверка Inspect Network Performance завершится ошибкой, так как по умолчанию на машинах используется python3.
```bash
ansible all -i cluster.inv -m replace \
-a 'path=/opt/cloudera/cm-agent/service/perf/host_perf_diag.py regexp=^#!\/usr\/bin\/python replace=#!/usr/bin/python2' --become
```

Далее запускаем обе проверки одновременно и ожидаем успеха.
![](/img/ustanovka-cloudera-cdh-6.3.2-s-tls-i-kerberos-na-osnove-freeipa/image_3_7.png)

## Select Services
Здесь указываем 'Custom Services', после чего выбираем основные компоненты:

- HDFS
- YARN (MR2 Included)
- ZooKeeper

## Assign Roles
Раскидываем основные роли по соответствующим хостам. Распределяем по узлам в зависимости от имени: dn – datanodes, nn – namenodes, zk – zookepers.

<span style="color: red">Не забываем про роль HttpFS, которая будет необходима для работы Hue</span>. При отсутствии этой роли, Hue будет использовать один из NameNode, что может повлечь его неработоспособность при переключении Active NameNode (при работе в режиме HDFS High Availability), который мы включим чуть позже. Похоже, что отсутсвие роли HttpFS вызывает и другие неприятные моменты в работе Hue.

## Setup Database
Нажимаем кнопку 'Test Connection' и 'Continue'.
![](/img/ustanovka-cloudera-cdh-6.3.2-s-tls-i-kerberos-na-osnove-freeipa/image_3_8.png)

## Review Changes
Дополнительный ёмкий диск на машинах зацеплен за `/data`, поэтому исправляем пути к каталогам добавляя префикс '/data'.  
![](/img/ustanovka-cloudera-cdh-6.3.2-s-tls-i-kerberos-na-osnove-freeipa/image_3_9.png)

## Завершение установки
После завершения установки попадаем в Cloudera Managment Console.
![](/img/ustanovka-cloudera-cdh-6.3.2-s-tls-i-kerberos-na-osnove-freeipa/image_3_10.png)
