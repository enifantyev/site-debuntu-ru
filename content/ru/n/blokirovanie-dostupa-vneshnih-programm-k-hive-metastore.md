---
title: "Блокирование доступа внешних программ к Hive metastore"
date: 2021-07-19
weight: 10
description: >
  После активации Cloudera Sentry, необходимо предотвратить возможность использования пользователями сторонних приложений для доступа к Hive metastore.
tags:
  - Apache Hive
  - Apache Hadoop
  - Cloudera CDH 6.3.2
  - BigData
---

2021-07-19

[Before Enabling the Sentry Service](https://docs.cloudera.com/documentation/enterprise/6/6.3/topics/sg_sentry_service_config.html#concept_z5b_42s_p4__section_lvc_4g4_rp)

<ol>
<li>В настройках службы Hive, используя фильтр 'Hive Metastore Access Control and Proxy User Groups Override', изменяем следующие параметры:</li>

<table>
<thead style="background: lightgray"><tr>
<td>Property</td>
<td>Value</td>
<td>Description</td>
</tr></thead>
<tr><td><b>Hive Metastore Access Control and Proxy User Groups Override</b>
<i>hadoop.proxyuser.hive.groups</i></td>
<td>
<span style="color:blue"><ul><li>hive</li>
<li>hue</li>
<li>sentry</li>
</ul></span>

Могут быть указаны дополнительные пользовательские группы.
</td>
<td>
This configuration <b>overrides</b> the value set for Hive Proxy User Groups configuration in HDFS service for use by Hive Metastore Server. Specify a comma-delimited list of groups that you want to <b>allow access to Hive Metastore metadata</b> and allow the Hive user to impersonate. A value of '*' allows all groups. The default value of empty inherits the value set for Hive Proxy User Groups configuration in the HDFS service.
</td></tr>
</table>

<li>Нажимаем <b>Save Changes</b>.</li>
<li>Перезапускаем все зависимые сервисы по приглашению Cloudera Manager Console.</li>
</ol>
