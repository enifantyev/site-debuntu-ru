---
title: "16. HBase. Установка и настройка"
date: 2021-07-02
weight: 16
description: >
  В этой части описывается настройка HBase.
tags:
  - Cloudera CDH 6.3.2
  - Apache Hadoop
  - HBase
  - CentOS
  - CentOS 7
  - BigData
slug: hbase-ustanovka-i-nastroyka
---

## Использованные материалы
1. [Manually Configuring TLS/SSL Encryption for CDH Services](https://docs.cloudera.com/documentation/enterprise/6/6.3/topics/cm_sg_hadoop_ssl_cm.html)

## 1. Добавление сервиса HBase
1. В консоли Cloudera Manager в меню выбираем «Add Service»:
    <center><img src="img-16-1.png" width=300px></center>
2. Выбираем HBase.
3. Распределяем роли. Хосты с именем hbr под HBase Region Servers...
    <center><img src="img-16-2.png" width=500px></center>
4. На следующем шаге оставляем настройки без изменений:
    <center><img src="img-16-3.png" width=500px></center>
5. Ждём добавления сервиса в кластер.
6. После успешного завершения процесса заканчиваем мастер нажатием на 'Finish'.
7. ![](/img/clouderabutton.png)Перезапускаем все зависимые сервисы по приглашению Cloudera Manager Console.

## 2. Перенастройка размещения log'ов
1. В настройках HBase, используя фильтр '/var/', изменяем следующие параметры, добавляя '/data' вместо '/var':
<table>
<tr>
<th>Property</th><th>Value</th><th>Description</th>
</tr>
<tr>
<td><b>HBase REST Server Log Directory</b><br>
<i>hadoop.log.dir</i>
</td>
<td><span style="color: blue"><code>/data/log/hbase</code></span></td>
<td>Directory where HBase REST Server will place its log files.</td>
</tr>
<tr>
<td><b>HBase Thrift Server Log Directory</b><br>
<i>hadoop.log.dir</i>
</td>
<td><span style="color: blue"><code>/data/log/hbase</code></span></td>
<td>Directory where HBase Thrift Server will place its log files.</td>
</tr>
<tr>
<td><b>Master Log Directory</b><br>
<i>hadoop.log.dir</i>
</td>
<td><span style="color: blue"><code>/data/log/hbase</code></span></td>
<td>Directory where Master will place its log files.</td>
</tr>
<tr>
<td><b>RegionServer Log Directory</b><br>
<i>hadoop.log.dir</i>
</td>
<td><span style="color: blue"><code>/data/log/hbase</code></span></td>
<td>Directory where RegionServer will place its log files.</td>
</tr>
</table>

2. Нажимаем **Save Change**.

## 3. Настройка TLS для HBase
### 3.1. Прежде чем вы начнете
- Перед включением TLS убедитесь, что хранилища ключей, содержащие сертификаты, привязанные к соответствующим доменным именам, должны быть доступны на всех хостах, на которых работает хотя бы одна роль демона HBase.
Хранилища ключей для HBase должны принадлежать группе hbase и иметь разрешения 0440 (то есть быть доступными для чтения владельцем и группой).
- Вы должны указать абсолютные пути к файлам хранилища ключей и доверенных сертификатов. Эти настройки применяются ко всем хостам, на которых выполняются роли демонов службы HBase. Следовательно, выбранные вами пути должны быть действительны на всех хостах.
- Cloudera Manager поддерживает конфигурацию TLS для HBase на уровне сервиса. Убедитесь, что вы указали абсолютные пути к файлам хранилища ключей и доверенных сертификатов. Эти настройки применяются ко всем хостам, на которых выполняются роли демонов рассматриваемой службы. Следовательно, выбранные вами пути должны быть действительны на всех хостах.
- Следствием этого является то, что имена файлов хранилища ключей для данной службы должны быть одинаковыми на всех хостах. Если, например, вы получили отдельные сертификаты для демонов HBase на хостах node1.example.com и node2.example.com, вы могли выбрать для хранения этих сертификатов файлы с именами hbase-node1.keystore и hbase-node2.keystore ( соответственно). При развертывании этих хранилищ ключей вы должны дать им одинаковое имя на целевом хосте, например, hbase.keystore.

### 3.2. Настройка TLS для HBase
В настройках службы HBase, используя фильтр «TLS», изменяем следующие параметры:
<table>
<tr>
<th>Property</th><th>Value</th><th>Description</th>
</tr>
<tr>
<td><b>Web UI TLS/SSL Encryption Enabled</b><br>
<i>hadoop.ssl.enabled, hbase.ssl.enabled</i>
</td>
<td><span style="color: blue">☑</span></td>
<td>Enable TLS/SSL encryption for the HBase Master, RegionServer, Thrift Server, and REST Server web UIs.</td>
</tr>
<tr>
<td><b>HBase TLS/SSL Server JKS Keystore File Location</b><br>
<i>ssl.server.keystore.location</i>
</td>
<td><span style="color: blue"><code>/opt/cloudera/security/pki/server.jks</code></span></td>
<td>Path to the keystore file containing the server certificate and private key used for encrypted web UIs.</td>
</tr>
<tr>
<td><b>HBase TLS/SSL Server JKS Keystore File Password</b><br>
<i>ssl.server.keystore.password</i>
</td>
<td>По умолчанию: <span style="color: blue">changeit</span>.</td>
<td>Password for the server keystore file used for encrypted web UIs.</td>
</tr>
<tr>
<td><b>HBase TLS/SSL Server JKS Keystore Key Password</b><br>
<i>ssl.server.keystore.keypassword</i>
</td>
<td>По умолчанию: <span style="color: blue">changeit</span>.</td>
<td>Password that protects the private key contained in the server keystore used for encrypted web UIs.</td>
</tr>
</table>

2. Настройка TLS для HBase REST Server.
<table>
<tr>
<th>Property</th><th>Value</th><th>Description</th>
</tr>
<tr>
<td><b>Enable TLS/SSL for HBase REST Server</b><br>
</td>
<td><span style="color: blue">☑</span></td>
<td>Encrypt communication between clients and HBase REST Server using Transport Layer Security (TLS).</td>
</tr>
<tr>
<td><b>HBase REST Server TLS/SSL Server JKS Keystore File Location</b><br>
</td>
<td><span style="color: blue"><code>/opt/cloudera/security/pki/server.jks</code></span></td>
<td>The path to the TLS/SSL keystore file containing the server certificate and private key used for TLS/SSL. Used when HBase REST Server is acting as a TLS/SSL server. The keystore must be in JKS format.file.</td>
</tr>
<tr>
<td><b>HBase REST Server TLS/SSL Server JKS Keystore File Password</b><br>
</td>
<td>По умолчанию: <span style="color: blue">changeit</span>.</td>
<td>The password for the HBase REST Server JKS keystore file.</td>
</tr>
<tr>
<td><b>HBase REST Server TLS/SSL Server JKS Keystore Key Password</b><br>
</td>
<td>По умолчанию: <span style="color: blue">changeit</span>.</td>
<td>The password that protects the private key contained in the JKS keystore used when HBase REST Server is acting as a TLS/SSL server.</td>
</tr>
</table>

3. Настройка TLS для HBase Thrift Server.
<table>
<tr>
<th>Property</th><th>Value</th><th>Description</th>
</tr>
<tr>
<td><b>Enable TLS/SSL for HBase Thrift Server over HTTP</b><br>
</td>
<td><span style="color: blue">☑</span></td>
<td>Encrypt communication between clients and HBase Thrift Server over HTTP using Transport Layer Security (TLS).</td>
</tr>
<tr>
<td><b>HBase Thrift Server over HTTP TLS/SSL Server JKS Keystore File Location</b><br>
</td>
<td><span style="color: blue"><code>/opt/cloudera/security/pki/server.jks</code></span></td>
<td>Path to the TLS/SSL keystore file (in JKS format) with the TLS/SSL server certificate and private key. Used when HBase Thrift Server over HTTP acts as a TLS/SSL server.</td>
</tr>
<tr>
<td><b>HBase Thrift Server over HTTP TLS/SSL Server JKS Keystore File Password</b><br>
</td>
<td>По умолчанию: <span style="color: blue">changeit</span>.</td>
<td>Password for the HBase Thrift Server JKS keystore file.</td>
</tr>
<tr>
<td><b>HBase Thrift Server over HTTP TLS/SSL Server JKS Keystore Key Password</b><br>
</td>
<td>По умолчанию: <span style="color: blue">changeit</span>.</td>
<td>Password that protects the private key contained in the JKS keystore used when HBase Thrift Server over HTTP acts as a TLS/SSL server.</td>
</tr>
</table>

4. Нажимаем **Save Changes**.

## 4. Configuring Encrypted Transport for HBase
1. В настройках службы HBase, используя категорию 'Security', изменяем следующие параметры:
<table>
<tr>
<th>Property</th><th>Value</th><th>Description</th>
</tr>
<tr>
<td><b>HBase Secure Authentication</b><br>
<i>hbase.security.authentication</i>
</td>
<td>
<span style="color: gray">HBase (Service-Wide)</span><br>
○&nbsp;simple<br>
<span style="color: blue">◉&nbsp;kerberos</span></td>
<td>Choose the authentication mechanism used by HBase.</td>
</tr>
<tr>
<td><b>HBase Secure Authorization</b><br>
<i>hbase.security.authorization</i>
</td>
<td><span style="color: blue">☑</span></td>
<td>Enable HBase authorization.</td>
</tr>
<tr>
<td><b>HBase Thrift Authentication</b><br>
<i>hbase.thrift.security.qop</i>
</td>
<td>
<span style="color: gray">HBase (Service-Wide)</span><br>
○&nbsp;none<br>
○&nbsp;auth<br>
○&nbsp;auth-int<br>
<span style="color: blue">◉&nbsp;auth-conf</span></td>
<td>
<p>If this is set, HBase Thrift Server authenticates its clients. HBase Proxy User Hosts and Groups should be configured to allow specific users to access HBase through Thrift Server.</p>
<p>Установка этого параметра вызывает предупреждение в логах: "Use authentication/integrity/privacy as value for rpc protection configurations instead of auth/auth-int/auth-conf."</p>
<p><span style="color: red">Возможно правильно оставить в 'none'</span>.</p></td>
</tr>
<tr>
<td><b>HBase REST Authentication</b><br>
<i>hbase.rest.authentication.type</i>
</td>
<td>
<span style="color: gray">HBase (Service-Wide)</span><br>
<br>
○&nbsp;simple
<span style="color: blue">◉&nbsp;kerberos</span></td>
<td>If this is set to "kerberos", HBase REST Server will authenticate its clients. HBase Proxy User Hosts and Groups should be configured to allow specific users to access HBase through REST Server.</td>
</tr>
<tr>
<td><b>HBase Transport Security</b><br>
<i>hbase.rpc.protection</i>
</td>
<td>
<span style="color: gray">HBase (Service-Wide)</span><br>
○&nbsp;authentication<br>
○&nbsp;integrity<br>
<span style="color: blue">◉&nbsp;privacy</span></td>
<td>Configure the type of encrypted communication to be used with RPC.</td>
</tr>
</table>

2. Нажимаем **Save Changes**.

## 7. Настройка аутентификации
Дополнительно разобраться с этой темой.

[https://docs.cloudera.com/documentation/enterprise/6/6.3/topics/cdh_sg_hbase_authorization.html](https://docs.cloudera.com/documentation/enterprise/6/6.3/topics/cdh_sg_hbase_authorization.html)

1. В настройках службы HBase изменяем следующие параметры:
<table>
<tr>
<th>Property</th><th>Value</th><th>Description</th>
</tr>
<tr>
<td><b>HBase Service Advanced Configuration Snippet (Safety Valve) for hbase-site.xml</b><br>
</td>
<td>
<b>Name:</b><br>
<span style="color: blue">hbase.security.exec.permission.checks</span><br>
<b>Value:</b><br>
<span style="color: blue">true</span><br>
<b>Description:</b><br>
<span style="color: blue">Without this option, all users will continue to have access to execute endpoint coprocessors.</span>
</td>
<td>For advanced use only, a string to be inserted into <b>hbase-site.xml</b>. Applies to configurations of all roles in this service except client configuration.</td>
</tr>
</table>

2. Нажимаем **Save Changes**.

## 5. Настройка доступа к HBase-таблицам из Hue
По умолчанию, HBase Thrift Server Compact Protocol включён, что мешает доступу к HBase-таблицам из браузера Hue. При попытке увидеть таблицы HBase из Hue, выводится транспарант 'Api Error: TSocket read 0 bytes'. Решение нашёл здесь: [How to create a HBase table on Kerberized Hadoop clusters](https://gethue.com/how-to-create-a-hbase-table-on-kerberized-hadoop-clusters/).

1.В настройках службы HBase изменяем/добавляем следующие параметры:
