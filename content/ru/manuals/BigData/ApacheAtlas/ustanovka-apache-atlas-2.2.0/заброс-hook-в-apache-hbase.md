---
title: "06. 🎣 Заброс Hook'а в Apache HBase"
date: 2021-10-22
weight: 11
description: >
  Описание процесса сборки Apache Atlas 2.2.0 для Cloudera CDH 6.3.2.
tags:
  - BigData
  - Apache Atlas
  - Apache HBase
  - Cloudera CDH 6.3.2
---

2021-08-30 / 2021-10-22

## 1.Введение
Механизм работы передачи информации об изменениях в Apache Hive в Apache Atlas очень прост. В Apache Hive добавляется Hook, то есть java-библиотека, которая будет отправлять сообщения в Apache Kafka при любых? изменениях в Apache Hive. Apache Atlas, после получения этих сообщений, приводит свой багаж знаний в соответствии с информацией из сообщений.

## 2. Создание Atlas-папки на хостах с ролью 'HBase Master'
2.1. На хостах с ролью 'HBase Master' создаём atlas-каталоги и скачиваем с Nexus'а необходимый файл:
```
FILENAME="?????????????????????????"
set +o history
NXUSERPASS="eugene:xxxxxxxxxxxxx"
set -o history
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

2.2. Залинковываем jar-библиотеки Atlas'а в каталог `/usr/lib/hbase/lib`:
```
sudo ln -s /opt/atlas/hook/hbase/* /usr/lib/hbase/lib/
```

2.3. Создаём файл 'atlas-application.properties':
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

2.4. Создаём, если таковой ещё не был сгененирован, и получаем новый keytab. Напомню, что <span style="color:red">последующие получения keytab'а необходимо выполнять с опцией '-r', иначе вместо получения существующего keytab'а будет создан новый keytab</span>, вследствие чего уже работающие сервисы Atlas'а перестанут аутентифицировать и потребуется повторить для них получение keytab'ов и перезапустить сервисы:
```
REALMNAME="$(hostname -d | tr [:lower:] [:upper:])"
IPAADMIN="eugene"
# Закомментировать след строку, если генерируем новый ключ для keytab'а.
OPTION="-r"

# kinit ${IPAADMIN}
ipa service-add atlas/$(hostname)

ipa service-allow-create-keytab atlas/$(hostname)@${REALMNAME} --users=${IPAADMIN}
ipa service-allow-retrieve-keytab atlas/$(hostname)@${REALMNAME} --users=${IPAADMIN}

ipa-getkeytab -p atlas/$(hostname) -k ~/atlas.keytab ${OPTION}
ipa service-disallow-retrieve-keytab atlas/$(hostname)@${REALMNAME} --users=${IPAADMIN}
ipa service-disallow-create-keytab atlas/$(hostname)@${REALMNAME} --users=${IPAADMIN}

sudo mv ~/atlas.keytab /opt/atlas/conf/
sudo chown root.root /opt/atlas/conf/atlas.keytab
sudo setfacl -m u:hbase:r /opt/atlas/conf/atlas.keytab

unset OPTION
```

3. Подключение Hook'а к Apache HBase

3.1. В настройках службы Hue изменяем следующие параметры:
<table>
<tr>
<th>Property</th><th>Value</th><th>Description</th>
</tr>
<tr>
<td>
<b>HBase Coprocessor Master Classes</b><br>
hbase.coprocessor.master.classes
</td>
<td>
- <span style="color:blue">org.apache.hadoop.hbase.security.access.AccessController</span><br>
- <span style="color:blue">org.apache.atlas.hbase.hook.HBaseAtlasCoprocessor</span>
<br><br>
При следующих установках Atlas'а попытаться определить момент, когда происходит ошибка <span style="color:salmon">HBase. Ошибка при выполнении 'user_permission'. ERROR: DISABLED: Security features are not available.</span> Именно для её решения понадобилось добавить 'org.apache.hadoop.hbase.security.access.AccessController'.
</td>
<td>
List of org.apache.hadoop.hbase.coprocessor.MasterObserver coprocessors that are loaded by default on the active HMaster process. For any implemented coprocessor methods, the listed classes will be called in order. After implementing your own MasterObserver, just put it in HBase's classpath and add the fully qualified class name here.
</td>
</tr>
<tr>
<td>Java Configuration Options for HBase Master</td>
<td>
{{JAVA_GC_ARGS}} -XX:ReservedCodeCacheSize=256m <span style="color:blue">-Datlas.conf=/opt/atlas/conf/</span>
</td>
<td>
These arguments will be passed as part of the Java command line. Commonly, garbage collection flags, PermGen, or extra debugging flags would be passed here. Note: When CM version is 6.3.0 or greater, {{JAVA_GC_ARGS}} will be replaced by JVM Garbage Collection arguments based on the runtime Java JVM version.
</td>
</tr>
</table>

3.2. Нажимаем Save Changes.

3.3. ![](/img/clouderabutton.png)Перезапускаем все зависимые сервисы по приглашению Cloudera Manager Console.

## 4. Использованные материалы
[Apache Atlas Hook & Bridge for Apache HBase](https://atlas.apache.org/#/HookHBase)
