---
title: "04. Распаковка и настройка Apache Atlas"
date: 2021-11-02
weight: 10
description: >
  Описание процесса установки Apache Atlas 2.2.0 в Cloudera CDH 6.3.2.
tags:
  - BigData
  - Apache Atlas
  - Cloudera CDH 6.3.2
slug: raspakovka-i-nastroyka-apache-atlas
---

2021-10-20 – 2021-11-02

>Везде, где выполняется работа с керберизированными сервисами, помним о предварительном получении Kerberos-билета, если таковой не был получен ранее.
<br><br>
Подавляющее число операций производим на Atlas-машине, иначе специально указывается иное.

## 1. Скачивание и распаковка собранного приложения Apache Atlas
Стандартное размещение Atlas'а находится в каталоге `/opt/atlas`. Для удобства работы с множеством версий, мы разместим эту и последующие сборки Atlas'а, каждую в своём каталоге, например `/opt/apache-atlas-2.2.0`. После чего создадим линк `/opt/atlas` для текущей актуальной сборки. Все пути в конфиг-файлах унифицированы и указывают на `/opt/atlas`.

1.1. На машине, предназначенной для работы Apache Atlas'а, скачиваем и распаковываем файл с уже собранным Atlas'ом (описываемая версия 2.2.0) в каталог `/opt/apache-atlas-2.2.0`, после чего делаем ссылку `/opt/atlas` на этот каталога.

>В переменной NXUSERPASS требуется указать правильный логин/пароль!

```
ATLASDIR="apache-atlas-2.2.0_rs-api-2.1.1"
FILENAME="apache-atlas-2.2.0_rs-api-2.1.1.tar.bz2"

set +o history
NXUSERPASS="eugene:xxxxxxxxxxxxx"
set -o history

## Скачиваем сборку с Nexus'а
mkdir -p ~/tmp
cd ~/tmp
curl -LO -u ${NXUSERPASS} http://nexus.example.org:8081/repository/dud_evolut_raw/atlas/${FILENAME}

# Распаковываем в /opt и делаем символическую ссылку
sudo tar -xvf ${FILENAME} -C /opt
sudo chown -R atlas.atlas /opt/${ATLASDIR}

sudo ln -s /opt/${ATLASDIR} /opt/atlas
```

## 2. Создание keytab'а для atlas
Так как Hadoop-кластер, с которым мы работает, керберизирован, то каждый участник кластера обязан иметь свой kerberos-принципал с ключами.

### 2.1.  Получение Kerberos-ключа для принципала atlas/$(hostname)
Этот принципал — 'atlas/$(hostname)' — используется для Kerberos-аутентификации Atlas'а в различных Hadoop-сервисах.

2.1.1. Напомню, что последующие получения ключей необходимо выполнять с опцией '-r', иначе вместо получения keytab'а с существующим kerberos-ключём, будет создан новый ключ с новым номером KVNO (key version number), вследствие чего уже работающие сервисы Atlas'а перестанут аутентифицироваться и потребуется повторить для них получение ключей с последующим перезапуском сервиса:
```
REALMNAME="$(hostname -d | tr [:lower:] [:upper:])"
IPAADMIN="eugene"

## При необходимости получаем kerberos-билет
# kinit ${IPAADMIN}

## Если требуются права на создание ключей.
ipa service-allow-create-keytab atlas/$(hostname)@${REALMNAME} --users=${IPAADMIN}
## Если требуются права на получение ключей, ранее уже созданных.
ipa service-allow-retrieve-keytab atlas/$(hostname)@${REALMNAME} --users=${IPAADMIN}
ipa-getkeytab -p atlas/$(hostname) -k ~/atlas.keytab
ipa service-disallow-retrieve-keytab atlas/$(hostname)@${REALMNAME} --users=${IPAADMIN}
ipa service-disallow-create-keytab atlas/$(hostname)@${REALMNAME} --users=${IPAADMIN}

sudo mv ~/atlas.keytab /opt/atlas/conf/
sudo chown atlas.atlas /opt/atlas/conf/atlas.keytab
```

### 2.2. Получение Kerberos-ключа для принципала HTTP/$(hostname)
>Пересмотреть данный пункт.<br>
<br>
Необходимо локализовать параметры, отвечающие за kerberos-аутентификацию пользователей в Atlas Web UI. Существуют сомнения в части использования этого принципала для упомянутых целей.
Выяснить, для каких целей используются принципалы HTTP в Hadoop-кластере.<br>
<br>
Выполнение этого пункта не наносит вреда, но этот принципал, на данный момент, нигде не используется.

Этот принципал используется для Kerberos-аутентификации пользователей в Atlas Web UI.

2.2.1. Есть большая вероятность, что при добавлении машины к Hadoop-кластеру и добавления на машину какой-либо роли, Cloudera Manager уже сгенерировал для машины этот keytab, поэтому запрашиваем его с опцией '-r', иначе создаём новый keytab по аналогии с предыдущим пунктом "Keytab для atlas/_HOST":
```
ipa service-allow-retrieve-keytab HTTP/$(hostname)@${REALMNAME} --users=${IPAADMIN}
ipa-getkeytab -p HTTP/$(hostname) -k /opt/atlas/conf/atlas.keytab -r
ipa service-disallow-retrieve-keytab HTTP/$(hostname)@${REALMNAME} --users=${IPAADMIN}
```

### 2.3. Проверяем наличие ключей в созданном keytab-файле
2.3.1. Выполняем:
```
$ klist -kte /opt/atlas/conf/atlas.keytab

Keytab name: FILE:/opt/atlas/conf/atlas.keytab
KVNO Timestamp           Principal
---- ------------------- ------------------------------------------------------
   1 03.11.2021 14:28:59 atlas/dev-app111p.test2.lan@TEST2.LAN (aes256-cts-hmac-sha1-96)
   1 03.11.2021 14:35:11 HTTP/dev-app111p.test2.lan@TEST2.LAN (aes256-cts-hmac-sha1-96)
```

## 3. Создание хранилища для безопасного хранения паролей
Atlas поддерживает новый способ хранения паролей для jks-хранилищ. Если раньше пароли указывались в открытом виде в conf-файла, то теперь пароли от хранилищ скрываются от глаз в отдельном хранилище.

>Напомню, что дефолтным паролем для java-хранилищ является 'changeit'.

3.1. Добавляем jceks-файла `/opt/atlas/conf/atlas.jceks` с паролями от хранилищ. Запускаем Atlas-утилиту `cputil.py` и на её запросы вводим:<br>
путь к хранилищу: jceks:///opt/atlas/conf/atlas.jceks;<br>
пароли для хранилищ: changeit.<br>
```
export JAVA_HOME="/usr/java/jdk1.8.0_181-cloudera/"
sudo -E /opt/atlas/bin/cputil.py
```

<details>

```
Please enter the full path to the credential provider: jceks:///opt/atlas/conf/atlas.jceks
Please enter the password value for keystore.password: changeit
Please enter the password value for keystore.password again: changeit
Please enter the password value for truststore.password: changeit
Please enter the password value for truststore.password again: changeit
Please enter the password value for password: changeit
Please enter the password value for password again: changeit
```

</details>
<br>

3.2. Исправляем права на созданный файл:
```
sudo chown atlas.atlas /opt/atlas/conf/atlas.jceks
```

## 4. Настройка Apache Atlas
### 4.1. Подготовка ролей Atlas'а для Admin'ов и User'ов
На одном из предыдущих этапов, во FreeIPA были подготовлены две группы — 'test2_atlas_admins' и 'test2_atlas_users' — для авторизации в Atlas. Используем их для назначения ролей авторизующимся в Atlas пользователям.

В конце файла `/opt/atlas/conf/atlas-simple-authz-policy.json` добавляем (или исправляем сущестующие)  сопоставления, ранее созданных LDAP-групп с ролями Atlas'а, как показано здесь:

<table>
<tr>
<th>При получении названий групп через UGI (предпочтительней):</th>
<th>При получении названий групп напрямую из LDAP-каталога:</th>
</tr>
<tr>
<td>
<pre><code>
  "groupRoles": {
    "test2_atlas_admins": [ "ROLE_ADMIN" ],
    "test2_atlas_users": [ "ROLE_SCIENTIST" ],
    "RANGER_TAG_SYNC": [ "DATA_SCIENTIST" ]
  }
</code></pre>
</td>
<td>
<pre><code>
  "groupRoles": {
    "ROLE_TEST2_ATLAS_ADMINS": [ "ROLE_ADMIN" ],
    "ROLE_TEST2_ATLAS_USERS": [ "ROLE_SCIENTIST" ],
    "RANGER_TAG_SYNC": [ "DATA_SCIENTIST" ]
  }
</code></pre>
</td>
</tr>
</table>

## 4.2. Изменения файла '/opt/atlas/conf/atlas-application.properties'
В данном примере показан способ получения пользовательских групп, необходимых для авторизации пользователя в Atlas Web UI, через Hadoop UGI (UserGroupInformation), а не через  LDAP. Преимущество этого способа в том, что с ним работают вложенные группы. То есть достаточно добавлять отдельные пользовательские группы в группу 'test2_atlas_admin', вместо добавления отдельных пользовательских аккаунтов.

Если этот способ не сработает, выполняем полную настройку LDAP-подключения с настройкой фильтров для получения информации о членстве УЗ, соответствующей введённому логину. Для этого необходимо расскомментировать соответствующие строки.

Основной файл настроек Atlas'а сгенерируем следующим кодом. Перед запуском этого сценария приводим переменные к актуальным значениям:

>В тех местах настроек, где наблюдаются словосочетания 'atlas/_HOST@TEST2.LAN', Atlas автоматически вставляет вместо '_HOST' полное имя локальной машины.

<details>

```
CLUSTERNAME="TEST2"
REALMNAME="TEST2.LAN"
LDAPDOMAINNAME="dc=test2,dc=lan"
LDAPSERVERS="ldaps://dev-ipa110p.test2.lan ldaps://dev-ipa111p.test2.lan ldaps://dev-ipa112p.test2.lan"
ZOOKEEPERSERVERS="dev-zk110p.test2.lan:2181,dev-zk111p.test2.lan:2181,dev-zk112p.test2.lan:2181"
KAFKASERVERS="dev-dn110p.test2.lan:9092,dev-dn111p.test2.lan:9092,dev-dn112p.test2.lan:9092"
ATLASURL="https://$(hostname)"

CLNLOWER=$(echo $CLUSTERNAME | tr [:upper:] [:lower:])

cat << EOF | sudo tee /opt/atlas/conf/atlas-application.properties
 #
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

#########  Graph Database Configs  #########

# Graph Database

#Configures the graph database to use.  Defaults to JanusGraph
#atlas.graphdb.backend=org.apache.atlas.repository.graphdb.janus.AtlasJanusGraphDatabase

# Graph Storage
# Set atlas.graph.storage.backend to the correct value for your desired storage
# backend. Possible values:
#
# hbase
# cassandra
# embeddedcassandra - Should only be set by building Atlas with  -Pdist,embedded-cassandra-solr
# berkeleyje
#
# See the configuration documentation for more information about configuring the various  storage backends.
#
atlas.graph.storage.backend=hbase2
atlas.graph.storage.hbase.table=apache_atlas_janus

#Hbase
#For standalone mode , specify localhost
#for distributed mode, specify zookeeper quorum here
atlas.graph.storage.hostname=${ZOOKEEPERSERVERS}
atlas.graph.storage.hbase.regions-per-server=1
atlas.graph.storage.lock.wait-time=10000

#In order to use Cassandra as a backend, comment out the hbase specific properties above, and uncomment the
#the following properties
#atlas.graph.storage.clustername=
#atlas.graph.storage.port=

# Gremlin Query Optimizer
#
# Enables rewriting gremlin queries to maximize performance. This flag is provided as
# a possible way to work around any defects that are found in the optimizer until they
# are resolved.
#atlas.query.gremlinOptimizerEnabled=true

# Delete handler
#
# This allows the default behavior of doing "soft" deletes to be changed.
#
# Allowed Values:
# org.apache.atlas.repository.store.graph.v1.SoftDeleteHandlerV1 - all deletes are "soft" deletes
# org.apache.atlas.repository.store.graph.v1.HardDeleteHandlerV1 - all deletes are "hard" deletes
#
#atlas.DeleteHandlerV1.impl=org.apache.atlas.repository.store.graph.v1.SoftDeleteHandlerV1

# Entity audit repository
#
# This allows the default behavior of logging entity changes to hbase to be changed.
#
# Allowed Values:
# org.apache.atlas.repository.audit.HBaseBasedAuditRepository - log entity changes to hbase
# org.apache.atlas.repository.audit.CassandraBasedAuditRepository - log entity changes to cassandra
# org.apache.atlas.repository.audit.NoopEntityAuditRepository - disable the audit repository
#
atlas.EntityAuditRepository.impl=org.apache.atlas.repository.audit.HBaseBasedAuditRepository

# if Cassandra is used as a backend for audit from the above property, uncomment and set the following
# properties appropriately. If using the embedded cassandra profile, these properties can remain
# commented out.
# atlas.EntityAuditRepository.keyspace=atlas_audit
# atlas.EntityAuditRepository.replicationFactor=1

# Graph Search Index
atlas.graph.index.search.backend=solr

#Solr
#Solr cloud mode properties
atlas.graph.index.search.solr.mode=cloud
atlas.graph.index.search.solr.zookeeper-url=${ZOOKEEPERSERVERS}/solr
atlas.graph.index.search.solr.zookeeper-connect-timeout=60000
atlas.graph.index.search.solr.zookeeper-session-timeout=60000
atlas.graph.index.search.solr.wait-searcher=true

#Solr http mode properties
#atlas.graph.index.search.solr.mode=http
#atlas.graph.index.search.solr.http-urls=http://localhost:8983/solr

# ElasticSearch support (Tech Preview)
# Comment out above solr configuration, and uncomment the following two lines. Additionally, make sure the
# hostname field is set to a comma delimited set of elasticsearch master nodes, or an ELB that fronts the masters.
#
# Elasticsearch does not provide authentication out of the box, but does provide an option with the X-Pack product
# https://www.elastic.co/products/x-pack/security
#
# Alternatively, the JanusGraph documentation provides some tips on how to secure Elasticsearch without additional
# plugins: https://docs.janusgraph.org/latest/elasticsearch.html
#atlas.graph.index.search.hostname=dev-app111p.test2.lan:9200
atlas.graph.index.search.elasticsearch.client-only=true

# Solr-specific configuration property
atlas.graph.index.search.max-result-set-size=150

#########  Import Configs  #########
#atlas.import.temp.directory=/temp/import

#########  Notification Configs  #########
atlas.notification.embedded=false
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

## Server port configuration
#atlas.server.http.port=21000
#atlas.server.https.port=21443

#########  Security Properties  #########

# SSL config
atlas.enableTLS=true

#truststore.file=/path/to/truststore.jks
#cert.stores.credential.provider.path=jceks://file/path/to/credentialstore.jceks
truststore.file=/usr/java/jdk1.8.0_181-cloudera/jre/lib/security/jssecacerts
cert.stores.credential.provider.path=jceks://file///opt/atlas/conf/atlas.jceks

#following only required for 2-way SSL
#keystore.file=/path/to/keystore.jks
keystore.file=/opt/cloudera/security/pki/server.jks

atlas.authentication.method.pam=false
atlas.authentication.method.pam.service=login

# Authentication config
atlas.authentication.method=kerberos
atlas.authentication.keytab=/opt/atlas/conf/atlas.keytab
atlas.authentication.principal=atlas/_HOST@${REALMNAME}

atlas.authentication.method.kerberos=true

atlas.authentication.method.kerberos.principal=atlas/_HOST@${REALMNAME}
atlas.authentication.method.kerberos.keytab=/opt/atlas/conf/atlas.keytab
atlas.authentication.method.kerberos.name.rules=RULE:[2:\$1@\$0](atlas@${REALMNAME})s/.*/atlas/
atlas.authentication.method.kerberos.token.validity=3600

#### user credentials file
atlas.authentication.method.file=false
#atlas.authentication.method.file.filename=${sys:atlas.home}/conf/users-credentials.properties

#### ldap.type= LDAP or AD
atlas.authentication.method.ldap=true
atlas.authentication.method.ldap.type=ldap

### groups from UGI
atlas.authentication.method.ldap.ugi-groups=true

######## LDAP properties #########
atlas.authentication.method.ldap.url=${LDAPSERVERS}
atlas.authentication.method.ldap.userDNpattern=uid={0},cn=users,cn=accounts,${LDAPDOMAINNAME}
#atlas.authentication.method.ldap.groupSearchBase=cn=groups,cn=accounts,${LDAPDOMAINNAME}
#atlas.authentication.method.ldap.groupSearchFilter=(&(member={0})(cn=${CLNLOWER}_atlas_*))
#atlas.authentication.method.ldap.groupRoleAttribute=cn
#atlas.authentication.method.ldap.base.dn=cn=accounts,${LDAPDOMAINNAME} #atlas.authentication.method.ldap.bind.dn=uid=${BINDUSER},cn=sysaccounts,cn=etc,${LDAPDOMAINNAME} #atlas.authentication.method.ldap.bind.password=$(BINDPASS)
#atlas.authentication.method.ldap.referral=follow
#atlas.authentication.method.ldap.user.searchfilter=(uid={0})
atlas.authentication.method.ldap.default.role=ROLE_USER

#########  JAAS Configuration ########
atlas.jaas.KafkaClient.loginModuleName=com.sun.security.auth.module.Krb5LoginModule
atlas.jaas.KafkaClient.loginModuleControlFlag=required
atlas.jaas.KafkaClient.option.useKeyTab=true
atlas.jaas.KafkaClient.option.storeKey=true
atlas.jaas.KafkaClient.option.serviceName=kafka
atlas.jaas.KafkaClient.option.keyTab=/opt/atlas/conf/atlas.keytab
atlas.jaas.KafkaClient.option.principal=atlas/_HOST@${REALMNAME}

atlas.jaas.Client.loginModuleName=com.sun.security.auth.module.Krb5LoginModule
atlas.jaas.Client.loginModuleControlFlag=required
atlas.jaas.Client.option.useKeyTab=true
atlas.jaas.Client.option.storeKey=true
atlas.jaas.Client.option.keyTab=/opt/atlas/conf/atlas.keytab
atlas.jaas.Client.option.principal=atlas/_HOST@${REALMNAME}

#########  Server Properties  #########
atlas.rest.address=${ATLASURL}:21443
# If enabled and set to true, this will run setup steps when the server starts
#atlas.server.run.setup.on.start=false

#########  Entity Audit Configs  #########
atlas.audit.hbase.tablename=apache_atlas_entity_audit
atlas.audit.zookeeper.session.timeout.ms=1000
atlas.audit.hbase.zookeeper.quorum=${ZOOKEEPERSERVERS}

#########  High Availability Configuration ########
atlas.server.ha.enabled=false
#### Enabled the configs below as per need if HA is enabled #####
#atlas.server.ids=id1
#atlas.server.address.id1=localhost:21000
#atlas.server.ha.zookeeper.connect=localhost:2181
#atlas.server.ha.zookeeper.retry.sleeptime.ms=1000
#atlas.server.ha.zookeeper.num.retries=3
#atlas.server.ha.zookeeper.session.timeout.ms=20000
## if ACLs need to be set on the created nodes, uncomment these lines and set the values ##
#atlas.server.ha.zookeeper.acl=<scheme>:<id>
#atlas.server.ha.zookeeper.auth=<scheme>:<authinfo>

######### Atlas Authorization #########
atlas.authorizer.impl=simple
atlas.authorizer.simple.authz.policy.file=atlas-simple-authz-policy.json

#########  Type Cache Implementation ########
# A type cache class which implements
# org.apache.atlas.typesystem.types.cache.TypeCache.
# The default implementation is org.apache.atlas.typesystem.types.cache.DefaultTypeCache
# which is a local in-memory type cache.
#atlas.TypeCache.impl=

#########  Performance Configs  #########
#atlas.graph.storage.lock.retries=10
#atlas.graph.storage.cache.db-cache-time=120000

#########  CSRF Configs  #########
atlas.rest-csrf.enabled=true
atlas.rest-csrf.browser-useragents-regex=^Mozilla.*,^Opera.*,^Chrome.*
atlas.rest-csrf.methods-to-ignore=GET,OPTIONS,HEAD,TRACE
atlas.rest-csrf.custom-header=X-XSRF-HEADER

############ KNOX Configs ################
#atlas.sso.knox.browser.useragent=Mozilla,Chrome,Opera
#atlas.sso.knox.enabled=true
#atlas.sso.knox.providerurl=https://<knox gateway ip>:8443/gateway/knoxsso/api/v1/websso
#atlas.sso.knox.publicKey=

############ Atlas Metric/Stats configs ################
# Format: atlas.metric.query.<key>.<name>
atlas.metric.query.cache.ttlInSecs=900
#atlas.metric.query.general.typeCount=
#atlas.metric.query.general.typeUnusedCount=
#atlas.metric.query.general.entityCount=
#atlas.metric.query.general.tagCount=
#atlas.metric.query.general.entityDeleted=
#
#atlas.metric.query.entity.typeEntities=
#atlas.metric.query.entity.entityTagged=
#
#atlas.metric.query.tags.entityTags=

#########  Compiled Query Cache Configuration  #########

# The size of the compiled query cache.  Older queries will be evicted from the cache
# when we reach the capacity.

#atlas.CompiledQueryCache.capacity=1000

# Allows notifications when items are evicted from the compiled query
# cache because it has become full.  A warning will be issued when
# the specified number of evictions have occurred.  If the eviction
# warning threshold <= 0, no eviction warnings will be issued.

#atlas.CompiledQueryCache.evictionWarningThrottle=0


#########  Full Text Search Configuration  #########

#Set to false to disable full text search.
#atlas.search.fulltext.enable=true

#########  Gremlin Search Configuration  #########

#Set to false to disable gremlin search.
atlas.search.gremlin.enable=false


########## Add http headers ###########

#atlas.headers.Access-Control-Allow-Origin=*
#atlas.headers.Access-Control-Allow-Methods=GET,OPTIONS,HEAD,PUT,POST
#atlas.headers.<headerName>=<headerValue>


#########  UI Configuration ########

atlas.ui.default.version=v1
EOF

sudo chown atlas.atlas /opt/atlas/conf/atlas-application.properties
```

</details>
<br>

### 4.3. Изменения в файле '/opt/atlas/conf/atlas-env.sh'
Этот файл используется для передачи различных Java-параметров запускаемому экземпляру Atlas'а. Сгенерируем его следующим сценарием:
<details>

```
cat << EOF | sudo tee /opt/atlas/conf/atlas-env.sh
#!/usr/bin/env bash
#
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
# The java implementation to use. If JAVA_HOME is not found we expect java and jar to be in path
#export JAVA_HOME=
export JAVA_HOME="/usr/java/jdk1.8.0_181-cloudera"

# any additional java opts you want to set. This will apply to both client and server operations
#export ATLAS_OPTS=

# any additional java opts that you want to set for client only
#export ATLAS_CLIENT_OPTS=

# java heap size we want to set for the client. Default is 1024MB
#export ATLAS_CLIENT_HEAP=

# any additional opts you want to set for atlas service.
#export ATLAS_SERVER_OPTS=

# indicative values for large number of metadata entities (equal or more than 10,000s)
#export ATLAS_SERVER_OPTS="-server -XX:SoftRefLRUPolicyMSPerMB=0 -XX:+CMSClassUnloadingEnabled -XX:+UseConcMarkSweepGC -XX:+CMSParallelRemarkEnabled -XX:+PrintTenuringDistribution -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=dumps/atlas_server.hprof -Xloggc:logs/gc-worker.log -verbose:gc -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=10 -XX:GCLogFileSize=1m -XX:+PrintGCDetails -XX:+PrintHeapAtGC -XX:+PrintGCTimeStamps"
export ATLAS_SERVER_OPTS="-server -XX:SoftRefLRUPolicyMSPerMB=0 -XX:+CMSClassUnloadingEnabled -XX:+UseConcMarkSweepGC \
  -XX:+CMSParallelRemarkEnabled -XX:+PrintTenuringDistribution -XX:+HeapDumpOnOutOfMemoryError \
  -XX:HeapDumpPath=dumps/atlas_server.hprof -Xloggc:logs/gc-worker.log -verbose:gc -XX:+UseGCLogFileRotation \
  -XX:NumberOfGCLogFiles=10 -XX:GCLogFileSize=1m -XX:+PrintGCDetails -XX:+PrintHeapAtGC -XX:+PrintGCTimeStamps \
  -Djava.security.krb5.conf=/etc/krb5.conf \
  -Djava.security.auth.login.config=/opt/atlas/conf/jaas.conf"

# java heap size we want to set for the atlas server. Default is 1024MB
#export ATLAS_SERVER_HEAP=

# indicative values for large number of metadata entities (equal or more than 10,000s) for JDK 8
export ATLAS_SERVER_HEAP="-Xms15360m -Xmx15360m -XX:MaxNewSize=5120m -XX:MetaspaceSize=100M -XX:MaxMetaspaceSize=512m"

# What is is considered as atlas home dir. Default is the base locaion of the installed software
#export ATLAS_HOME_DIR=
export ATLAS_HOME_DIR="/opt/atlas"

# Where log files are stored. Defatult is logs directory under the base install location
#export ATLAS_LOG_DIR=
export ATLAS_LOG_DIR="/opt/atlas/logs"

# Where pid files are stored. Defatult is logs directory under the base install location
export ATLAS_PID_DIR="/opt/atlas"

# where the atlas titan db data is stored. Defatult is logs/data directory under the base install location
#export ATLAS_DATA_DIR=
export ATLAS_DATA_DIR="/opt/atlas/data"

# Where do you want to expand the war file. By Default it is in /server/webapp dir under the base install dir.
#export ATLAS_EXPANDED_WEBAPP_DIR=
export ATLAS_EXPANDED_WEBAPP_DIR="/opt/atlas/server/webapp"

# indicates whether or not a local instance of HBase should be started for Atlas
export MANAGE_LOCAL_HBASE=false

# indicates whether or not a local instance of Solr should be started for Atlas
export MANAGE_LOCAL_SOLR=false

# indicates whether or not cassandra is the embedded backend for Atlas
export MANAGE_EMBEDDED_CASSANDRA=false

# indicates whether or not a local instance of Elasticsearch should be started for Atlas
export MANAGE_LOCAL_ELASTICSEARCH=false

export HBASE_CONF_DIR="/etc/hbase/conf"
EOF

sudo chown atlas.atlas /opt/atlas/conf/atlas-env.sh
```
</details>
<br>

### 4.4. Добавление jaas-файла для работы с керберизированным Apache Solr
В файле `/opt/apache-atlas/conf/atlas-application.properties` есть необходимые строки для подключения приложений через jaas-конфигурацию. Но в случае работы с Apache Solr это не сработало, поэтому используем отдельный jaas-файл.

Создаём файл `/opt/atlas/conf/jaas.conf` со следующим содержимым:
```
REALMNAME="$(hostname -d | tr [:lower:] [:upper:])"
PRINCIPAL="atlas/$(hostname)@${REALMNAME}"

cat << EOF | sudo tee /opt/atlas/conf/jaas.conf
Client {
  com.sun.security.auth.module.Krb5LoginModule required
  useKeyTab=true
  keyTab="/opt/atlas/conf/atlas.keytab"
  storeKey=true
  principal="${PRINCIPAL}"
  debug=false;
};
EOF

sudo chown atlas.atlas /opt/atlas/conf/jaas.conf
```

## 5. Подготовка systemd unit'а для запуска Atlas
Создаём файл `/etc/systemd/system/atlas.service`:
```
cat << EOF | sudo tee /etc/systemd/system/atlas.service
[Unit]
Description=Apache Atlas Service
After=network.target

[Service]
Type=simple
PIDFile=/opt/atlas/atlas.pid
WorkingDirectory=/opt/atlas

User=atlas
Group=atlas

ExecStart=/opt/atlas/bin/atlas_start.py
ExecStop=/opt/atlas/bin/atlas_stop.py
TimeoutSec=300

[Install]
WantedBy=multi-user.target
EOF
```

Обязательно обновляем конфигурацию Systemd:
```
sudo systemctl daemon-reload
```

На данный момент включать и запускать демон atlas.service не требуется.

## 7. Список дополнительных способов аутентификации в Atlas
Дополнительные способы аутентификации в Atlas могут пригодиться на стадии отладки.

### 7.1. PAM-аутентификация
Метод рабочий и очень простой, при условии, что машина в домене FreeIPA. Рекомендуется к использованию только при тестировании Atlas'а.

Изменения файла '/opt/apache-atlas/conf/atlas-application.properties', которые позволяют использовать для аутентификации, PAM-модуль `/etc/pam.d/login`:
```
atlas.authentication.method.pam=true
atlas.authentication.method.pam.service=login
```

### 7.2. Аутентификация по УЗ из файла
Изменения файла '/opt/apache-atlas/conf/atlas-application.properties':
```
atlas.authentication.method.file=true
atlas.authentication.method.file.filename=${sys:atlas.home}/conf/users-credentials.properties
```

Файл `users-credentials.properties` имеет вид (логин/пароль: admin/admin):
```
#username=group::sha256-password
admin=ADMIN::8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918
rangertagsync=RANGER_TAG_SYNC::0afe7a1968b07d4c3ff4ed8c2d809a32ffea706c66cd795ead9048e81cfaf034
```

Генерация хэша слова 'admin', без 'перевода строки' и 'возврата каретки' в конце строки,  выполняется, например так:
```
printf "admin" | sha256sum
8c6976e5b5410415bde908bd4dee15dfb167a9c873fc4bb8a81f6f2ab448a918  -
```

### 7.3. Kerberos-аутентификация
Есть сомнения в данном пункте, в части правильного определения параметров, отвечающих за Kerberos-аутентификацию пользователей при входе в Atlas Web UI.

Не работает для автоматического входа, хотя браузер настроен и Kerberos-билет в наличии. Автоматический вход не происходит. При попытке ввода учётных данных высвечивается ошибка Error while authentication.

Получение keytab'а для принципала HTTP/_HOST@TEST.LAN, обязательно без пересоздания его (-r опция), так как перестанут работать уже настроенные роли Cloudera-кластера, вследствие чего будет необходимо выполнить перегенерацию kerberos-принципалов средствами Cloudera Manager'а:
```
REALMNAME="EXAMPLE.ORG"
IPAADMIN="eugene"
#kinit

ipa service-allow-retrieve-keytab HTTP/$(hostname)@${REALMNAME} --users=${IPAADMIN}
ipa-getkeytab -r -p HTTP/$(hostname) -k /opt/atlas/conf/atlas.keytab
ipa service-disallow-retrieve-keytab HTTP/$(hostname)@${REALMNAME} --users=${IPAADMIN}
```

Изменения файла '/opt/apache-atlas/conf/atlas-application.properties':
```
atlas.authentication.method.kerberos = true

atlas.authentication.method.kerberos.principal=HTTP/_HOST@${REALMNAME}
atlas.authentication.method.kerberos.keytab = /opt/atlas/conf/atlas.keytab
atlas.authentication.method.kerberos.name.rules = RULE:[2:$1@$0](atlas@${REALMNAME})s/.*/atlas/
atlas.authentication.method.kerberos.token.validity = 3600
```

## 8. Обнаруженные проблемы
Solr пишет:
```
FieldTypePluginLoader	TokenizerFactory is using deprecated 5.0.0 emulation. You should at some point declare and reindex to at least 6.0,&#8203; because 5.x emulation is deprecated and will be removed in 7.0
```

В файле `/usr/lib/solr/atlasTemplate/solrconfig.xml` исправил версию подходящего Solr'а с пятой на 7.4 и переинициализировал Atlas-обвязку.
