---
title: "14. HDFS. Настройка"
date: 2021-07-09
weight: 14
description: >
  В этой части описывается настройка HDFS.
tags:
  - Cloudera CDH 6.3.2
  - Apache Hadoop
  - РВАЫ
  - CentOS
  - CentOS 7
  - BigData
slug: hdfs-nastroyka
---

2021-07-09&nbsp;&ndash;&nbsp;2021-10-15

## Enable High Availability
[Enabling HDFS HA](https://docs.cloudera.com/documentation/enterprise/6/6.3/topics/cdh_hag_hdfs_ha_enabling.html)

1. В настройках сервиса HDFS, используя фильтр по слову 'dfs.journalnode.edits.dir', изменяем следующий параметр:
<table>
<tr>
<th>Property</th><th>Value</th><th>Description</th>
</tr>
<tr>
<td><b>JournalNode Edits Directory</b><br>
<i>dfs.journalnode.edits.dir</i></td>
<td>JournalNode Default Group:<br>
<span style="color: blue"><code>/data/dfs/jn</code></span></td>
<td>Directory on the local file system where NameNode edits are written.</td>
</tr>
</table>

2. Нажимаем Save Changes.
3. На странице службы HDFS, через кнопку Actions, запускаем wizard включения HA:
    <center><img src="/img/hdfs-nastroyka/img14-1.png" style="height:500px"></center>
4. **Getting Started.**
    Задаём уникальное имя для Nameservice:
    <center><img src="/img/hdfs-nastroyka/img14-2.png" style="width:600px"></center>
5. **Assign Roles**.<br>
    JournalNodes должны работать на хостах с такими же hardware specification, как NameNodes. Cloudera рекомендует поместить две JournalNode на те же хосты с NameNodes, а третий JournalNode на хост с похожими ресурсами, таким как JobTracker.<br>
    Распределяем роли:
    <center><img src="/img/hdfs-nastroyka/img14-3.png" style="width:600px"></center>
6. **Review Changes**.<br>
    Знакомимся с предварительными результатами и указываем каталог для JournalNode `/data/dfs/jn`:
    <center><img src="/img/hdfs-nastroyka/img14-4.png" style="width:600px"></center>
7. **Command Details**.
    <center><img src="/img/hdfs-nastroyka/img14-5.png" style="width:600px"></center>
8. **Final Steps**.
    <center><img src="/img/hdfs-nastroyka/img14-6.png" style="width:300px"></center>

## Настройка LDAP-аутентификации в HDFS
{{% alert title="⚠ Описание настройки уже в процессе переработки." color="warning" %}}
Важно пересмотреть эту настройку прямого подключения Hadoop к LDAP-каталогу. Нужно оставить использование `org.apache.hadoop.security.ShellBasedUnixGroupsMapping`, так как прозрачное подключение к LDAP на машинах введённых в домен уже реализовано через SSSD.

Переписать инструкцию!

Держать в уме, что после отключения HDFS LDAP Mapping, перестала работать аутентификация в Solr! К сожалению, не помню, Solr был добавлен до включения LDAP Mapping или после, что могло бы повлиять на такое его поведение.
{{% /alert %}}

{{% pageinfo %}}
При прямом подключении к LDAP не работают вложенные пользовательские группы?
{{% /pageinfo %}}

{{% pageinfo %}}
Полезно в /etc/sssd/sssd.conf добавить фильтрацию Hadoop'овских УЗ и групп из LDAP в секции [nss], иначе одноимённая доменная УЗ возобладает преимуществом и нарушит работу hadoop-кластера.

```
[nss]
filter_groups = root,mysql,hadoop,yarn,hdfs,mapred,kms,httpfs,hbase,hive,sentry,spark,solr,sqoop,oozie,hue,flume,impala,llama,postgres,sqoop2,kudu,kafka,accumulo,zookeeper,cloudera-scm,keytrustee
filter_users = root,mysql,cloudera-scm,zookeeper,yarn,hdfs,mapred,kms,httpfs,hbase,hive,sentry,spark,solr,sqoop,oozie,hue,flume,impala,llama,sqoop2,postgres,kudu,kafka,accumulo,keytrustee
reconnection_retries = 3
```

Возможно, что полезно указывать, если используется прямое подключение к LDAP:
- [⚠ YARN History Server не отображает отработанные задачи после включения Kerberos](yarn-nastroyka)
{{% /pageinfo %}}

1. В настройках службы HDFS, используя фильтр по слову &laquo;hadoop.security.group.mapping.ldap.ssl&raquo;, изменяем следующие параметры:
<table>
<tr>
<th>Property</th><th>Value</th><th>Description</th>
</tr>
<tr>
<td><b>Hadoop User Group Mapping LDAP TLS/SSL Truststore</b><br>
<i>hadoop.security.group.mapping.ldap.ssl.keystore</i>
</td>
<td><span style="color: blue"><code>/usr/java/jdk1.8.0_181-cloudera/jre/lib/security/jssecacerts</code></span></td>
<td>File path to a jks-format truststore containing the TLS/SSL certificate used sign the LDAP server's certificate. Note that in previous releases this was erroneously referred to as a "keystore".</td>
</tr>
<tr>
<td><b>Hadoop User Group Mapping LDAP TLS/SSL Truststore Password</b><br>
<i>hadoop.security.group.mapping.ldap.ssl.keystore.password</i>
</td>
<td>По умолчанию: <span style="color: blue">changeit</span>.</td>
<td>The password for the TLS/SSL truststore.</td>
</tr>
</table>

2. Нажимаем **Save Changes**.
3. <span style="color: red">Используем следующие настройки только в случае использования прямого подключения Hadoop'а к LDAP, вместо рекомендованного подключения через SSSD.</span><br>
В настройках службы HDFS, используя категорию &laquo;Security&raquo;, изменяем следующие параметры:

<details>

<table border="1">
<tr>
<th>Property</th><th>Value</th><th>Description</th>
</tr>
<tr>
<td><b>Hadoop User Group Mapping Implementation</b><br>
<i>hadoop.security.group.mapping</i>
</td>
<td>
○&nbsp;org.apache.hadoop.security.JniBasedUnixGroupsMapping<br>
○&nbsp;org.apache.hadoop.security.ShellBasedUnixGroupsMapping<br>
<span style="color: blue">◉&nbsp;org.apache.hadoop.security.LdapGroupsMapping</span></td>
<td>Class for user to group mapping (get groups for a given user).</td>
</tr>
<tr>
<td><b>Hadoop User Group Mapping LDAP URL</b><br>
<i>hadoop.security.group.mapping.ldap.url</i></td>
<td><span style="color: blue">ldaps://dev-ipa01p.test.lan ldaps://dev-ipa02p.test.lan ldaps://dev-ipa03p.test.lan</span>
<br>Список из URL, разделённых пробелами.</td>
<td>The URL of the LDAP server. The URL must be prefixed with ldap:// or ldaps://. The URL can optionally specify a custom port, for example: ldaps://ldap_server.example.com:1636. Note that usernames and passwords will be transmitted in the clear unless either an ldaps:// URL is used, or "Enable LDAP TLS" is turned on (where available). Also note that encryption must be in use between the client and this service for the same reason.

For more detail on the LDAP URL format, see RFC 2255 . A space-separated list of URLs can be entered; in this case the URLs will each be tried in turn until one replies.</td>
</tr>
<tr>
<td><b>Hadoop User Group Mapping LDAP TLS/SSL Enabled</b><br>
<i>hadoop.security.group.mapping.ldap.use.ssl</i></td>
<td><span style="color: blue">☑</span></td>
<td>Whether or not to use TLS/SSL when connecting to the LDAP server.</td>
</tr>
<tr>
<td><b>Hadoop User Group Mapping LDAP TLS/SSL Truststore</b><br>
<i>hadoop.security.group.mapping.ldap.ssl.keystore</i></td>
<td><span style="color: blue"><code>/usr/java/jdk1.8.0_181-cloudera/jre/lib/security/jssecacerts</code></span></td>
<td>File path to a jks-format truststore containing the TLS/SSL certificate used sign the LDAP server's certificate. Note that in previous releases this was erroneously referred to as a "keystore".</td>
</tr>
<tr>
<td><b>Hadoop User Group Mapping LDAP TLS/SSL Truststore Password</b><br>
<i>hadoop.security.group.mapping.ldap.ssl.keystore.password</i></td>
<td>По умолчанию: <span style="color: blue">changeit</span>.</td>
<td>The password for the TLS/SSL truststore.</td>
</tr>
<tr>
<td><b>Hadoop User Group Mapping LDAP Bind User Distinguished Name</b><br>
<i>hadoop.security.group.mapping.ldap.bind.user</i></td>
<td><span style="color: blue">uid=binddn_test2,cn=sysaccounts,cn=etc,dc=test2,dc=lan</span></td>
<td>Distinguished name of the user to bind as. This is used to connect to LDAP/AD for searching user and group information. This may be left blank if the LDAP server supports anonymous binds.</td>
</tr>
<tr>
<td><b>Hadoop User Group Mapping LDAP Bind User Password</b><br>
<i>hadoop.security.group.mapping.ldap.bind.password</i></td>
<td>Пароль для вышеуказанного системного аккаунта binddn_test2.</td>
<td>The password of the bind user.</td>
</tr>
<tr>
<td><b>Hadoop User Group Mapping Search Base</b><br>
<i>hadoop.security.group.mapping.ldap.base</i></td>
<td><span style="color: blue">cn=accounts,dc=test2,dc=lan</span></td>
<td>The search base for the LDAP connection. This is a distinguished name, and will typically be the root of the LDAP directory.</td>
</tr>
<tr>
<td><b>Hadoop User Group Mapping LDAP User Search Filter</b><br>
<i>hadoop.security.group.mapping.ldap.search.filter.user</i></td>
<td><span style="color: blue">(&(objectClass=person)(uid={0}))</span></td>
<td>An additional filter to use when searching for LDAP users. The default will usually be appropriate for Active Directory installations. If connecting to a generic LDAP server, ''sAMAccountName'' will likely be replaced with ''uid''. {0} is a special string used to denote where the username fits into the filter.</td>
</tr>
<tr>
<td><b>Hadoop User Group Mapping LDAP Group Search Filter</b><br>
<i>hadoop.security.group.mapping.ldap.search.filter.group</i></td>
<td><span style="color: blue">(objectClass=ipausergroup)</span></td>
<td>An additional filter to use when searching for groups.</td>
</tr>
<tr>
<td><b>Hadoop User Group Mapping LDAP Group Membership Attribute</b><br>
<i>hadoop.security.group.mapping.ldap.search.attr.member</i></td>
<td><span style="color: blue">member</span></td>
<td>The attribute of the group object that identifies the users that are members of the group. The default will usually be appropriate for any LDAP installation.</td>
</tr>
<tr>
<td><b>Hadoop User Group Mapping LDAP Group Name Attribute</b><br>
<i>hadoop.security.group.mapping.ldap.search.attr.group.name</i></td>
<td><span style="color: blue">cn</span></td>
<td>The attribute of the group object that identifies the group name. The default will usually be appropriate for all LDAP systems.</td>
</tr>
</table>

- Нажимаем Save Changes.

</details>

## Создание группы для доступа в HDFS с правами superuser
