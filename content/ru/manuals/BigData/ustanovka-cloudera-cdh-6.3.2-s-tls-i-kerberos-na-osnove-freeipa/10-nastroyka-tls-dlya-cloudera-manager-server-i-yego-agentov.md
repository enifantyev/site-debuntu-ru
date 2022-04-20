---
title: "10. Настройка TLS для Cloudera Manager Server и его агентов"
date: 2021-06-17
weight: 10
description: >
  В этой части описывается настройка TLS для Cloudera Manager Server и его агентов.
tags:
  - Cloudera CDH 6.3.2
  - Apache Hadoop
  - CentOS
  - CentOS 7
  - BigData
slug: nastroyka-tls-dlya-cloudera-manager-server-i-yego-agentov
---

## Использованные материалы
- [Manually Configuring TLS Encryption for Cloudera Manager](https://docs.cloudera.com/documentation/enterprise/6/6.3/topics/how_to_configure_cm_tls.html)
- [Manually Configuring TLS Encryption on the Agent Listening Port](https://docs.cloudera.com/documentation/enterprise/6/6.3/topics/configure_tls_cm_agent_port.html)

## Вступление
Когда вы настраиваете аутентификацию и авторизацию в кластере, Cloudera Manager Server отправляет различную конфиденциальную информацию по сети на узлы кластера, например, таблицы ключей Kerberos и файлы конфигурации, содержащие пароли. Чтобы защитить эту передачу, необходимо настроить TLS-шифрование между Cloudera Manager Server и всеми узлами кластера.

Шифрование TLS также используется для защиты клиентских подключений к административному интерфейсу Cloudera Manager с помощью HTTPS.

Cloudera Manager также поддерживает аутентификацию TLS. Если кластер не настроен на TLS-аутентификацию, то злоумышленник может добавить свой хост в Cloudera Manager, установив программное обеспечение Cloudera Manager Agent и настроив его для связи с Cloudera Manager Server. Чтобы предотвратить это, вы должны установить сертификаты на каждом хосте агента и настроить Cloudera Manager Server на доверие этим сертификатам.

## 1. Генерация TLS-сертификатов
Установка TLS-сертификатов выполняется на каждом узле кластера. Так как все узлы кластера добавлены во FreeIPA-домен, то запрос сертификата в Центре Сертификации и автоматическое обновление имеющихся сертификатов грамотней будет поручить Certmonger'у, а не следовать инструкции от Cloudera. Это, а также добавление ключей и сертификатов в java-хранилища, удобней сделать через Ansible-плэйбук cloudera_setup_tls.

Плэйбук выполняет операцию `ipa-getcert request -k HTTP/$(hostname -f) ...` с размещением ключа и TLS-сертификата к нему в каталоге `/opt/cloudera/security/pki/`. К выпуску сертификата привязано выполнение bash-скрипта, который разместит сертификат в надлежащих хранилищах, используемых при выполнении java-приложений. После выпуска сертификата и выполнения скрипта, плэйбук перенастраивает Cloudera Manager Agents на использование TLS-сертификатов и перезапускает агентов.

<span style="color: red">Перед разбросом ключей и сертификатов, остановливаем кластер и менеджмент, так как будет потеряна связь с агентами!!!</span>

```bash
# Скачиваем весь ansible из gitlab'а
git clone git@git.example.org:adm/ansible.git
# Подправляем плэйбук
sed -i s/all-root/cl/ ansible/playbooks/cloudera_setup_tls.yml

cd ansible

# Выполняем тестовый --check прогон
ansible-playbook -i ../cluster.inv playbooks/cloudera_setup_tls.yml --extra-vars '{"IPA_CERT_MANAGER":"admin"}' --become --check

# Выполняем плэйбук
ansible-playbook -i ../cluster.inv playbooks/cloudera_setup_tls.yml --extra-vars '{"IPA_CERT_MANAGER":"admin"}' --become
```

Замечу, что выполняя сейчас все действия из ансибл-плэйбука, мы слегка опережаем клаудеровскую инструкцию. Поэтому очерёдность операция в этой инструкции не совпадает с клаудеровскими инструкциями.

Не запуская остановленные Cloudera Management Service и кластер, продолжаем выполнять шаги этой инструкции.

## 2. Настройка TLS для CM, клиентов, и связи между CM и агентами
1. Входим Cloudera Manager Admin Console.
2. Выбираем Administration > Settings.
3. Выбираем категорию Security.
4. В строке фильтрации результатов указываем TLS.
5. Настраиваем следующие TLS параметры по порядку:
<table>
<tr><th>Property</th><th>Value</th><th>Description</th></tr>
<tr>
<td><b>Use TLS Encryption for Admin Console</b></td>
<td><span style="color: blue">☑</span></td>
<td>Enable TLS encryption (HTTPS) between the user and the Cloudera Manager Admin Console. When checked, the HTTPS port will be used.</td>
</tr>
<tr>
<td><b>Use TLS Encryption for Agents</b></td>
<td><span style="color: blue">☑</span></td>
<td>Select this option to enable TLS encryption between the Server and Agents.</td>
</tr>
<tr>
<td><b>Use TLS Authentication of Agents to Server</b></td>
<td><span style="color: blue">☑</span></td>
<td>Select this option to enable TLS Authentication of Agents to the Server.</td>
</tr>
<tr>
<td><b>Cloudera Manager TLS/SSL Server JKS Keystore File Location</b></td>
<td><span style="color: blue">/opt/cloudera/security/pki/server.jks</code></span></td>
<td>The path to the TLS/SSL keystore file containing the server certificate and private key used for TLS/SSL. Used when Cloudera Manager is acting as a TLS/SSL server. The keystore must be in JKS format.</td>
</tr>
<tr>
<td><b>Cloudera Manager TLS/SSL Server JKS Keystore File Password</b></td>
<td>По умолчанию: <span style="color: blue">changeit</code></span>.</td>
<td>The password for the Cloudera Manager JKS keystore file.</td>
</tr>
<tr>
<td><b>Cloudera Manager TLS/SSL Client Trust Store File</b></td>
<td><span style="color: blue">/usr/java/jdk1.8.0_181-cloudera/jre/lib/security/jssecacerts</td>
<td>The location on disk of the trust store, in .jks format, used to confirm the authenticity of TLS/SSL servers that Cloudera Manager might connect to. This is used when Cloudera Manager is the client in a TLS/SSL connection. This trust store must contain the certificate(s) used to sign the service(s) connected to. If this parameter is not provided, the default list of well-known certificate authorities is used instead.</td>
</tr>
<tr>
<td><b>Cloudera Manager Server TLS/SSL Certificate Trust Store Password</b></td>
<td>По умолчанию: <span style="color: blue">changeit</code></span>.</td>
<td>The password for the Cloudera Manager TLS/SSL Certificate Trust Store File. This password is not required to access the trust store; this field can be left blank. This password provides optional integrity checking of the file. The contents of trust stores are certificates, and certificates are public information.</td>
</tr>
</table>
6. Нажимаем **Save Changes**.<br>
7. В настройках Cloudera Management Service (http://cloudera:7180/cmf/services/6/config), используя категорию 'Security', изменяем следующие параметры:
<table>
<tr><th>Property</th><th>Value</th><th>Description</th></tr>
<tr>
<td><b>TLS/SSL Client Truststore File Location</b><br><i>ssl.client.truststore.location</i></td>
<td><span style="color: blue">/usr/java/jdk1.8.0_181-cloudera/jre/lib/security/jssecacerts</code></span></td>
<td>Path to the client truststore file used in HTTPS communication. This truststore contains certificates of trusted servers, or of Certificate Authorities trusted to identify servers. If set, this is used to verify certificates in HTTPS communication with CDH services and the Cloudera Manager Server. If not set, the default Java truststore is used to verify certificates. The contents of this truststore can be modified without restarting the Cloudera Management Service roles. By default, changes to its contents are picked up within ten seconds.</td>
</tr>
<tr>
<td><b>Cloudera Manager Server TLS/SSL Client Trust Store Password</b>
<i>ssl.client.truststore.password</i></td>
<td>По умолчанию: <span style="color: blue">changeit</code></span>.</td>
<td>The password for the Cloudera Manager Server TLS/SSL Certificate Trust Store File. This password is not required to access the trust store; this field can be left blank. This password provides optional integrity checking of the file. The contents of trust stores are certificates, and certificates are public information.</td>
</tr>
<tr>
<td><b>Enable TLS/SSL for Firehose Debug Server</b>
<i>debug.servlet.https.enabled</i></td>
<td><span style="color: blue">☑</span></td>
<td>Encrypt communication between clients and Firehose Debug Server using Transport Layer Security (TLS) (formerly known as Secure Socket Layer (SSL)).</td>
</tr>
<tr>
<td><b>Firehose Debug Server TLS/SSL Server JKS Keystore File Location</b>
<i>debug.servlet.https.keystorePath</i></td>
<td><span style="color: blue">/opt/cloudera/security/pki/server.jks</code></span></td>
<td>The path to the TLS/SSL keystore file containing the server certificate and private key used for TLS/SSL. Used when Firehose Debug Server is acting as a TLS/SSL server. The keystore must be in JKS format.</td>
</tr>
<tr>
<td><b>Firehose Debug Server TLS/SSL Server JKS Keystore File Password</b>
<i>debug.servlet.https.keystorePassword</i></td>
<td>По умолчанию: <span style="color: blue">changeit</code></span>.</td>
<td>The password for the Firehose Debug Server JKS keystore file.</td>
</tr>
</table>
8. Нажимаем **Save Changes**.<br><br>

>Без настроенных параметров Firehose Debug Server, после включения в настройках ZooKeeper параметра "Enable TLS client authentication for JMX port", Cloudera Manager перестаёт видеть состояние серверов ZooKeeper.

9. Выходим из каталога `ansible`, где чуть ранее запускалась задача разброса сертификатов, в наш рабочий каталог, и  перезапускаем Cloudera Manager server:
    ```bash
    cd ..
    ansible mgm -i cluster.inv -m systemd \
    -a "name=cloudera-scm-server state=restarted" --become
    ```
10. Через несколько минут поключаемся к Cloudera Manager Admin Console используя HTTPS на 7183 порту: https://dev-mgm01p.test.lan:7183.
11. Сначала запускаем Cloudera Management Service:
![](/img/ustanovka-cloudera-cdh-6.3.2-s-tls-i-kerberos-na-osnove-freeipa/image_10_1.png)<br>
Здесь видим, что все 13 хостов находятся на связи, а значит TLS-шифрование между узлами работает.
12. Потом запускаем кластер и наблюдаем, что всё в норме:
![](/img/ustanovka-cloudera-cdh-6.3.2-s-tls-i-kerberos-na-osnove-freeipa/image_10_2.png)

## 3. Отладка при необходимости
### Verify that Cloudera Manager Server and Agents are Communicating
In the Cloudera Manager Admin Console, go to Hosts > All Hosts. If you see successful heartbeats reported in the Last Heartbeat column after restarting the agents and server, TLS certificate authentication is working properly. If not, check the agent log (/var/log/cloudera-scm-agent/cloudera-scm-agent.log) for errors.

For example, you might see the following error:

WrongHost: Peer certificate commonName does not match host, expected 192.0.2.155, got cdh-1.example.com
[02/May/2018 15:04:15 +0000] 4655 MainThread agent        ERROR    Heartbeating to 192.0.2.155:7182 failed
For this scenario, make sure that your DNS and /etc/hosts file are configured correctly, and that your server_host parameter in /etc/cloudera-scm-agent/config.ini uses the Cloudera Manager Server hostname, and not IP address.

### Прочее
1. Проверка TLS-работоспособности Cloudera Agent'ов на хостах:
    ```bash
    openssl s_client -connect <hostname>:9000
    ```
2. Можно обнаружить в логах Cloudera Manager Server запись "com.cloudera.cmf.command.CmdNoopPropagateException: All roles are unlicensed and cannot be started". Внимания можно не обращать, так как последствий не обнаружено. Проблема была в использовании ip-адреса CM, вместо его FQDN-имени в файле настройки агентов.<br>
Проверить в '/etc/cloudera-scm-agent/config.ini' параметр 'server_host' должен содержать полное FQDN-имя, как в TLS-сертификате, например:
    ```
    server_host=dev-mgm01p.test.lan
    ```
