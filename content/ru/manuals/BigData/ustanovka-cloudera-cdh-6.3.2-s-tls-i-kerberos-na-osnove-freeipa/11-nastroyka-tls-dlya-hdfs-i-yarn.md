---
title: "11. Настройка TLS для HDFS и YARN"
date: 2021-06-01
weight: 11
description: >
  В этой части описывается настройка TLS для HDFS и YARN.
tags:
  - Cloudera CDH 6.3.2
  - Apache Hadoop
  - CentOS
  - CentOS 7
  - BigData
slug: nastroyka-tls-dlya-hdfs-i-yarn
---

## Использованные материалы
[Manually Configuring TLS/SSL Encryption for CDH Services](https://docs.cloudera.com/documentation/enterprise/6/6.3/topics/cm_sg_hadoop_ssl_cm.html)

## Вступление
Помимо настройки кластера Cloudera Manager для использования TLS, различные службы CDH, работающие в кластере, также должны быть настроены для использования TLS. Процесс настройки TLS зависит от компонента, поэтому выполните следующие действия, если это необходимо для вашей системы. Однако, прежде чем пытаться настроить TLS, убедитесь, что ваш кластер соответствует предварительным требованиям.

В общем, все роли на любом узле в кластере могут использовать одни и те же сертификаты, при условии, что сертификаты имеют соответствующий формат (JKS, PEM) и что конфигурация правильно указывает на расположение.

><span style="color: red">Примечание. TLS для основных сервисов Hadoop — HDFS и YARN — необходимо включить одновременно. TLS для других компонентов, таких как HBase, Hue и Oozie, можно включить независимо.</span>

Не все компоненты поддерживают TLS, и не все внешние механизмы поддерживают TLS. Если явно не указано в этом руководстве, компонент, который вы хотите настроить, может в настоящее время не поддерживать TLS. Например, Sqoop в настоящее время не поддерживает TLS для Oracle, MySQL или других баз данных.

### Предпосылки
Cloudera рекомендует, чтобы кластер и все службы использовали Kerberos для проверки подлинности. Если вы включите TLS для кластера, который не был настроен для использования Kerberos, отобразится предупреждение. Прежде чем продолжить, вам следует интегрировать кластер с развертыванием Kerberos.

Для выполнения описанных ниже шагов требуется, чтобы в кластере был включен TLS, и что сертификат сервера Cloudera Manager и сертификаты агента Cloudera Manager правильно настроены и уже установлены. Кроме того, у вас должны быть готовы сертификаты и ключи, необходимые для конкретного сервера CDH.

Если кластер соответствует этим требованиям, вы можете настроить конкретную службу CDH для использования TLS, как подробно описано в этом разделе.

## 1. Настройка TLS для HDFS и YARN
TLS для основных сервисов Hadoop — HDFS, MapReduce и YARN — необходимо включить одновременно. Поскольку большинство кластеров запускают либо MapReduce, либо YARN, а не оба сразу, вы обычно включаете HDFS и YARN или HDFS и MapReduce. Включение TLS в HDFS необходимо, прежде чем его можно будет включить в MapReduce или YARN.
Примечание. Если вы включаете TLS для HDFS, вы также должны включить его для MapReduce или YARN.

Следующие шаги включают включение аутентификации Kerberos для веб-консолей HTTP. При включении TLS для основных служб Hadoop в кластере без включения аутентификации отображается предупреждение.

### 1.1. Прежде чем вы начнёте
Перед включением TLS хранилища ключей, содержащие сертификаты, привязанные к соответствующим доменным именам, должны быть доступны на всех хостах, на которых работает хотя бы одна роль демона HDFS или YARN.

Поскольку демоны HDFS и YARN действуют как клиенты TLS, а также как серверы TLS, они должны иметь доступ к хранилищам доверенных сертификатов. Во многих случаях наиболее практичным подходом является развертывание доверенных хранилищ на всех хостах в кластере, поскольку может быть нежелательно заранее определять набор хостов, на которых будут работать клиенты.

Хранилища ключей для HDFS и YARN должны принадлежать группе hadoop и иметь разрешения 0440 (то есть их могут читать владелец и группа). У хранилищ доверенных сертификатов должны быть разрешения 0444 (то есть быть доступными для чтения всем).

Cloudera Manager поддерживает конфигурацию TLS для HDFS и YARN на уровне обслуживания. Для каждой из этих служб необходимо указать абсолютные пути к файлам хранилища ключей и доверенных сертификатов. Эти настройки применяются ко всем хостам, на которых выполняются роли демонов рассматриваемой службы. Следовательно, выбранные вами пути должны быть действительны на всех хостах.

Следствием этого является то, что имена файлов хранилища ключей для данной службы должны быть одинаковыми на всех хостах. Если, например, вы получили отдельные сертификаты для демонов HDFS на хостах node1.example.com и node2.example.com, вы могли бы выбрать для хранения этих сертификатов файлы с именами hdfs-node1.keystore и hdfs-node2.keystore ( соответственно). При развертывании этих хранилищ ключей вы должны дать им одинаковое имя на целевом хосте, например, hdfs.keystore.

Несколько демонов, запущенных на хосте, могут совместно использовать сертификат. Например, если на одном хосте запущены DataNode и сервер Oozie, они могут использовать один и тот же сертификат.

### 1.2. Настройка TLS для HDFS
1. В настройках службы HDFS, используя фильтр 'ssl.server.keystore', изменяем следующие параметры:
<table>
<tr>
<th>Property</th><th>Value</th><th>Description</th>
</tr>
<tr>
<td><b>Hadoop TLS/SSL Server Keystore File Location</b></td>
<td><span style="color: blue"><code>/opt/cloudera/security/pki/server.jks</code></span></td>
<td>Path to the keystore file containing the server certificate and private key.</td>
</tr>
<tr>
<td><b>Hadoop TLS/SSL Server Keystore File Password</b></td>
<td>По умолчанию: <span style="color: blue"><code>changeit</code></span>.</td>
<td>Password for the server keystore file.</td>
</tr>
<tr>
<td><b>Hadoop TLS/SSL Server Keystore Key Password</b></td>
<td>По умолчанию: <span style="color: blue"><code>changeit</code></span>.</td>
<td>Password that protects the private key contained in the server keystore.</td>
</tr>
</table>

![](/img/ustanovka-cloudera-cdh-6.3.2-s-tls-i-kerberos-na-osnove-freeipa/image_11_1.png)

2. Настройка TLS/SSL client truststore. Эти параметры могут быть переопределены в настройках YARN, но там мы их настраивать не будем.
Для роли HDFS, используя фильтр 'ssl.client.truststore', изменяем следующие параметры:
<table>
<tr>
<th>Property</th><th>Value</th><th>Description</th>
</tr>
<tr>
<td><b>Cluster-Wide Default TLS/SSL Client Truststore Location</b></td>
<td><span style="color: blue"><code>/usr/java/jdk1.8.0_181-cloudera/jre/lib/security/jssecacerts</code></span></td>
<td>Path to the client truststore file. This truststore contains certificates of trusted servers, or of Certificate Authorities trusted to identify servers.</td>
</tr>
<tr>
<td><b>Cluster-Wide Default TLS/SSL Client Truststore Password</b></td>
<td>По умолчанию: <span style="color: blue"><code>changeit</code></span>.</td>
<td>Password for the client truststore file.</td>
</tr>
</table>

![](/img/ustanovka-cloudera-cdh-6.3.2-s-tls-i-kerberos-na-osnove-freeipa/image_11_2.png)

3. <span style="color: lightgray"><b>Этот пункт инструкции необходимо перести в соответствующий раздел, так как здесь Kerberos ещё не активирован</b>.<br>
(Optional) Эта настройка была предложена к выполнению в предыдущей инструкции "Включение FreeIPA Kerberos-аутентификации в Cloudera CDH 6.3.2". Если параметр не был настроен, то есть шанс сделать это сейчас.<br>
Cloudera рекомендует включить аутентификацию веб-интерфейса для службы HDFS. Аутентификация веб-интерфейса использует SPNEGO. После включения вы не сможете получить доступ к веб-консолям Hadoop без действительного билета Kerberos и правильной конфигурации на стороне клиента.</span>
<table>
<tr>
<th>Property</th><th>Value</th><th>Description</th>
</tr>
<tr>
<td><b>Enable Authentication for HTTP Web-Consoles</b></td>
<td><span style="color: blue">☑</span></td>
<td>Enables authentication for Hadoop HTTP web-consoles for all roles of this service. Note: This is effective only if security is enabled for the HDFS service.</td>
</tr>
</table>

![](/img/ustanovka-cloudera-cdh-6.3.2-s-tls-i-kerberos-na-osnove-freeipa/image_11_3.png)

4. Нажимаем **Save Changes**.

>При применении этого изменения может высветиться ошибка:<br><br>
General Error(s)<br><br>
<span style="color: red">Role is missing Kerberos keytab. Go to the Kerberos Credentials page and click the Generate Missing Credentials button</span>.<br><br>
Но после перезапуска сервисов ошибка пропадает.

### 1.3. Настройка TLS для YARN
1. В настройках роли YARN, используя фильтр 'ssl.server.keystore', изменяем следующие параметры:
<table>
<tr>
<th>Property</th><th>Value</th><th>Description</th>
</tr>
<tr>
<td><b>Hadoop TLS/SSL Server Keystore File Location</b></td>
<td><span style="color: blue"><code>/opt/cloudera/security/pki/server.jks</code></span></td>
<td>Path to the keystore file containing the server certificate and private key.</td>
</tr>
<tr>
<td><b>Hadoop TLS/SSL Server Keystore File Password</b></td>
<td>По умолчанию: <span style="color: blue"><code>changeit</code></span>.</td>
<td>Password for the server keystore file.</td>
</tr>
<tr>
<td><b>Hadoop TLS/SSL Server Keystore Key Password</b></td>
<td>По умолчанию: <span style="color: blue"><code>changeit</code></span>.</td>
<td>Password that protects the private key contained in the server keystore.</td>
</tr>
</table>

![](/img/ustanovka-cloudera-cdh-6.3.2-s-tls-i-kerberos-na-osnove-freeipa/image_11_4.png)

2. <span style="color: lightgray">Настройки YARN для TLS/SSL Client Truststore File пропускаем, так как они переопределяют такие же настройки в роли HDFS, что не имеет смысла.</span>

3. <span style="color: lightgray"><b>Этот пункт инструкции необходимо перести в соответствующий раздел, так как здесь Kerberos ещё не активирован.</b><br>
Cloudera рекомендует включить аутентификацию веб-интерфейса для рассматриваемой службы. Эта настройка была предложена к выполнению в предыдущей инструкции "Включение FreeIPA Kerberos-аутентификации в Cloudera CDH 6.3.2". Если параметр не был настроен, то есть шанс сделать это сейчас.</span>
<table>
<tr>
<th>Property</th><th>Value</th><th>Description</th>
</tr>
<tr>
<td><b>Enable Authentication for HTTP Web-Consoles</b></td>
<td><span style="color: blue">☑</span></td>
<td>Enables authentication for Hadoop HTTP web-consoles for all roles of this service. Note: This is effective only if security is enabled for the HDFS service.</td>
</tr>
</table>

![](/img/ustanovka-cloudera-cdh-6.3.2-s-tls-i-kerberos-na-osnove-freeipa/image_11_5.png)

>При применении этого изменения может высветиться ошибка:<br><br>
General Error(s)<br><br>
<span style="color: red">Role is missing Kerberos keytab. Go to the Kerberos Credentials page and click the Generate Missing Credentials button</span>.<br><br>
Но после перезапуска сервисов ошибка пропадает.

4. Нажимаем **Save Changes**.

### 1.4. Включение TLS для HDFS и YARN
1. В настройках службы HDFS, используя фильтр 'hadoop.ssl.enabled', изменяем следующие параметры:
<table>
<tr>
<th>Property</th><th>Value</th><th>Description</th>
</tr>
<tr>
<td><b>Hadoop TLS/SSL Enabled</b></td>
<td><span style="color: blue">☑</span></td>
<td>Enable TLS/SSL encryption for HDFS, MapReduce, and YARN web UIs, as well as encrypted shuffle for MapReduce and YARN.</td>
</tr>
</table>

![](/img/ustanovka-cloudera-cdh-6.3.2-s-tls-i-kerberos-na-osnove-freeipa/image_11_6.png)

2. <span style="color: lightgray"><b>Этот пункт инструкции необходимо перести в соответствующий раздел, так как здесь Kerberos ещё не активирован, впрочем это предупреждение может и не появиться.</b><br>
Для выполнения появившейся рекомендации, изменяем значение HDFS-параметра 'DataNode Transceiver Port' c '1004' на предлагаемый номер по умолчанию '9866':</span>
<table>
<tr>
<th>Property</th><th>Value</th><th>Description</th>
</tr>
<tr>
<td><b>DataNode Transceiver Port</b><br>
<i>dfs.datanode.address</i></td>
<td><span style="color: blue"><code>9866</code></span></td>
<td>Port for DataNode's XCeiver Protocol. Combined with the DataNode's hostname to build its address.</td>
</tr>
</table>

3. Нажимаем **Save Changes**.

4. ![](/img/clouderabutton.png)Перезапускаем все зависимые сервисы по приглашению Cloudera Manager Console.
