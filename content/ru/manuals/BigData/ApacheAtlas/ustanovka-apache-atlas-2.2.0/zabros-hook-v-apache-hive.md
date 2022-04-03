---
title: "05. 🎣 Заброс Hook'а в Apache Hive"
date: 2021-08-30
weight: 10
description: >
  Описание процесса установки Apache Atlas 2.2.0 в Cloudera CDH 6.3.2.
tags:
  - BigData
  - Apache Atlas
  - Apache Hive
  - Cloudera CDH 6.3.2
---

2021-08-30

## 1.Введение
Механизм работы передачи информации об изменениях в Apache Hive в Apache Atlas очень прост. В Apache Hive добавляется Hook, то есть java-библиотека, которая будет отправлять сообщения в Apache Kafka при любых? изменениях в Apache Hive. Apache Atlas, после получения этих сообщений, приводит свой багаж знаний в соответствии с информацией из сообщений.

## 2. Создание Atlas-папки на хостах с ролью 'HiveServer2'
2.1. На хостах с ролью 'HiveServer2' создаём atlas-каталоги и скачиваем с Nexus'а необходимый файл:
```
FILENAME="?????????????????"
## Пробел нужен, чтобы пароль не попал в history
 NXUSERPASS="eugene:xxxxxxxxxxxxx"
DIR="apache-atlas-2.2.0_cdh6.3.2_j8.181_mvn3.8.1"

## Скачиваем сборку с Nexus'а
mkdir -p ~/tmp
cd ~/tmp
curl -LO -u ${NXUSERPASS} http://nexus.example.org:8081/repository/dud_evolut_raw/atlas/${FILENAME}

sudo tar xvf ${FILENAME} -C /opt
cd /opt
sudo ln -s ${DIR} atlas

sudo chown -R root.root ${DIR}
sudo chmod -R u=rwX,go=rX ${DIR}
```

2.2. Создаём файл 'atlas-application.properties':
```
ZOOKEEPERSERVERS="dev-zk110p.test2.lan:2181,dev-zk111p.test2.lan:2181,dev-zk112p.test2.lan:2181"
KAFKASERVERS="dev-dn110p.test2.lan:9092,dev-dn111p.test2.lan:9092,dev-dn112p.test2.lan:9092"
REALMNAME="TEST2.LAN"

sudo mkdir -p /opt/atlas/conf
cat << EOF | sudo tee /opt/atlas/conf/atlas-application.properties
#########  Notification Configs  #########

atlas.kafka.zookeeper.connect=${ZOOKEEPERSERVERS}
atlas.kafka.bootstrap.servers=${KAFKASERVERS}
atlas.kafka.zookeeper.session.timeout.ms=60000
atlas.kafka.zookeeper.connection.timeout.ms=60000
atlas.kafka.zookeeper.sync.time.ms=20
atlas.kafka.auto.commit.interval.ms=1000
atlas.kafka.hook.group.id=atlas

atlas.kafka.enable.auto.commit=true
atlas.kafka.auto.offset.reset=earliest
atlas.kafka.session.timeout.ms=30000
atlas.kafka.offsets.topic.replication.factor=1
atlas.kafka.poll.timeout.ms=1000

atlas.kafka.security.protocol=SASL_PLAINTEXT
atlas.kafka.sasl.mechanism=GSSAPI
atlas.kafka.sasl.kerberos.service.name=kafka

#########  JAAS Configuration ########

atlas.jaas.KafkaClient.loginModuleName=com.sun.security.auth.module.Krb5LoginModule
atlas.jaas.KafkaClient.loginModuleControlFlag=required
atlas.jaas.KafkaClient.option.useKeyTab=true
atlas.jaas.KafkaClient.option.storeKey=true
atlas.jaas.KafkaClient.option.serviceName=kafka
atlas.jaas.KafkaClient.option.keyTab=/opt/atlas/conf/atlas.keytab
atlas.jaas.KafkaClient.option.principal=atlas/_HOST@${REALMNAME}
EOF
```

2.3. Создаём, если таковой ещё не был сгененирован, и получаем новый keytab. Напомню, что <span style="color:red">последующие получения keytab'а необходимо выполнять с опцией '-r', иначе вместо получения существующего keytab'а будет создан новый keytab</span>, вследствие чего уже работающие сервисы Atlas'а перестанут аутентифицировать и потребуется повторить для них получение keytab'ов и перезапустить сервисы:
```
REALMNAME="$(hostname -d | tr [:lower:] [:upper:])"
IPAADMIN="eugene"
# Закомментировать след строку, если генерируем новый ключ для keytab'а.
OPTION="-r"

# kinit ${IPAADMIN}
ipa service-allow-create-keytab atlas/$(hostname)@${REALMNAME} --users=${IPAADMIN}
ipa service-allow-retrieve-keytab atlas/$(hostname)@${REALMNAME} --users=${IPAADMIN}
ipa-getkeytab -p atlas/$(hostname) -k ~/atlas.keytab ${OPTION}
ipa service-disallow-retrieve-keytab atlas/$(hostname)@${REALMNAME} --users=${IPAADMIN}
ipa service-disallow-create-keytab atlas/$(hostname)@${REALMNAME} --users=${IPAADMIN}

sudo mv ~/atlas.keytab /opt/atlas/conf/
sudo chown root.root /opt/atlas/conf/atlas.keytab
sudo setfacl -m u:hive:r /opt/atlas/conf/atlas.keytab

unset OPTION
```

## 3. Заброс Hook'а в Apache Hive через Cloudera Manager

3.1. В настройках службы Hive изменяем следующие параметры:
<table>
<tr>
<th>Property</th><th>Value</th><th>Description</th>
</tr>
<tr>
<td><b>HiveServer2 Advanced Configuration Snippet (Safety Valve) for hive-site.xml</b></td>
<td>
<b>Name</b>: <span style="color:blue">hive.exec.post.hooks</span><br>
<b>Value</b>: <span style="color:blue">org.apache.atlas.hive.hook.HiveHook</span><br>
<b>Description</b>:<br>
<hr>
<span style="color:lightgrey"> Следующие параметры не используем. После проверки, будут удалены или активированы.

<b>Name</b>: hive.reloadable.aux.jars.path<br>
<b>Value</b>: /opt/atlas/hook/hive<br>
<b>Description:</b><br>
<br>
<b>Name</b>: atlas.cluster.name<br>
<b>Value</b>: primary<br>
<b>Description</b>:<br>
</span>
</td>
<td>
	For advanced use only. A string to be inserted into hive-site.xml for this role only.
</td>
</tr>
<tr>
<td><b>Java Configuration Options for HiveServer2</b></td>
<td>{{JAVA_GC_ARGS}} <span style="color:blue">-Datlas.conf=/opt/atlas/conf/</span></td>
<td>
	These arguments will be passed as part of the Java command line. Commonly, garbage collection flags, PermGen, or extra debugging flags would be passed here. Note: When CM version is 6.3.0 or greater, {{JAVA_GC_ARGS}} will be replaced by JVM Garbage Collection arguments based on the runtime Java JVM version.
</td>
</tr>
<tr>
<td><b>HiveServer2 Environment Advanced Configuration Snippet (Safety Valve)</b></td>
<td><span style="color:blue">HIVE_AUX_JARS_PATH=/opt/atlas/hook/hive/</span></td>
<td>For advanced use only, key-value pairs (one on each line) to be inserted into a role's environment. Applies to configurations of this role except client configuration.</td>
</tr>
</table>

3.2. Нажимаем Save Changes.

3.3. ![](/img/clouderabutton.png)Перезапускаем все зависимые сервисы по приглашению Cloudera Manager Console.

## 4. Использованные материалы
[Apache Atlas Hook & Bridge for Apache Hive](https://atlas.apache.org/#/HookHive)
