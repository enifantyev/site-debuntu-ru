---
title: "12. Включение Kerberos в Cloudera CDH 6.3.2"
date: 2021-10-14
weight: 12
description: >
  В этой части описывается включение Kerberos в кластере Hadoop.
tags:
  - Cloudera CDH 6.3.2
  - Apache Hadoop
  - CentOS
  - CentOS 7
  - BigData
slug: vklyucheniye-kerberos-v-cloudera-cdh-6.3.2
---

2021-06-17&nbsp;&ndash;&nbsp;2021-10-14

## 1. Использованные материалы
1. [How to Configure Clusters to Use Kerberos for Authentication](https://docs.cloudera.com/documentation/enterprise/latest/topics/cm_sg_using_cm_sec_config.html)
2. [Configuring Authentication in Cloudera Manager](https://docs.cloudera.com/documentation/enterprise/latest/topics/cm_sg_authentication.html)
3. [Enabling Kerberos Authentication for CDH](https://docs.cloudera.com/documentation/enterprise/latest/topics/cm_sg_intro_kerb.html)
    - [Step 1: Install Cloudera Manager and CDH](https://docs.cloudera.com/documentation/enterprise/latest/topics/cm_sg_s1_install_cm_cdh.html)
    - [Step 2: Install JCE Policy Files for AES-256 Encryption](https://docs.cloudera.com/documentation/enterprise/latest/topics/cm_sg_s2_jce_policy.html)
    - [Step 3: Create the Kerberos Principal for Cloudera Manager Server](https://docs.cloudera.com/documentation/enterprise/latest/topics/cm_sg_s3_cm_principal.html)
    - [Step 4: Enabling Kerberos Using the Wizard](https://docs.cloudera.com/documentation/enterprise/latest/topics/cm_sg_s4_kerb_wizard.html)
    - [Step 5: Create the HDFS Superuser](https://docs.cloudera.com/documentation/enterprise/latest/topics/cm_sg_s5_hdfs_principal.html)
    - [Step 6: Get or Create a Kerberos Principal for Each User Account](https://docs.cloudera.com/documentation/enterprise/latest/topics/cm_sg_s6_user_principals.html)
    - [Step 7: Prepare the Cluster for Each User](https://docs.cloudera.com/documentation/enterprise/latest/topics/cm_sg_s7_prepare_cluster.html)
    - [Step 8: Verify that Kerberos Security is Working](https://docs.cloudera.com/documentation/enterprise/latest/topics/cm_sg_s8_verify_kerb.html)
    - [Step 9: (Optional) Enable Authentication for HTTP Web Consoles for Hadoop Roles](https://docs.cloudera.com/documentation/enterprise/latest/topics/cm_sg_web_auth.html)
4. [Sample Kerberos Configuration Files](https://docs.cloudera.com/documentation/enterprise/6/6.3/topics/sg_kerberos_troubleshoot.html#sample-kerberos-files)

## 2. Заметки
### 2.1. Выдержка из материала по ссылке #1.
"Cloudera кластеры могут использовать Kerberos для аутентификации сервисов, запускаемых в кластере, и пользователей, нуждающихся в доступе к этим сервисам. Для кластеров развёрнутых с помощь 'Cloudera Manager Server', Cloudera рекомендует использовать 'Kerberos configuration wizard', доступный через 'Cloudera Manager Admin Console'. Смотри ссылку #2."

### 2.2. Выдержка из материала по ссылке #2.
"Cloudera кластеры могут быть сконфигурированы для использования Kerberos-аутентификации вручную или используя "мастер конфигурации", доступный из 'Cloudera Manager Admin Console'. CLoudera рекомендует использовать "мастер" для автоматизации множества изменений в конфигурации и задачах их распределения. Вдобавок, включение Kerberos из "мастера" также включит Kerberos-аутентификацию для всех CDH-компонентов установленных (и устанавливаемых в будущем?) в кластере.

Перед включением Kerberos-аутентификации, сконфигурируйте TLS/SSL в CLouder Manager, иначе секретные Kerberos keytab будут перебрасываться по сети без шифрования."

### 2.3. О krb5.conf и создании keytab'ов
Cloudera Server для генерации сервисов и их keytab'ов использует утилиту `kadmin`, для работы которой необходимо добавить в `krb5.conf` параметр 'admin_server', при отсутствии которых работа kerberos-визарда будет остановлена с ошибкой, подобной:

Generate Missing Credentials
<details><summary>посмотреть...</summary>

```
/opt/cloudera/cm/bin/gen_credentials_ipa.sh failed with exit code 1 and output of <<
+ CMF_REALM=TEST.LAN
+ export PATH=/usr/kerberos/bin:/usr/kerberos/sbin:/usr/lib/mit/sbin:/usr/sbin:/usr/lib/mit/bin:/usr/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin
+ PATH=/usr/kerberos/bin:/usr/kerberos/sbin:/usr/lib/mit/sbin:/usr/sbin:/usr/lib/mit/bin:/usr/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin
+ kinit -k -t /var/run/cloudera-scm-server/cmf2399691723203886836.keytab cmadmin-84a0694d@TEST.LAN
+ KEYTAB_OUT=/var/run/cloudera-scm-server/cmf4842231189498651716.keytab
+ PRINCIPAL=hdfs/dev-dn90p.test.lan@TEST.LAN
+ MAX_RENEW_LIFE=432000
+ '[' -z /etc/krb5.conf ']'
+ echo 'Using custom config path '\''/etc/krb5.conf'\'', contents below:'
+ cat /etc/krb5.conf
+ PRINC=hdfs
++ echo hdfs/dev-dn90p.test.lan@TEST.LAN
++ cut -d / -f 2
++ cut -d @ -f 1
+ HOST=dev-dn90p.test.lan
+ set +e
+ ipa host-find dev-dn90p.test.lan
+ ERR=0
+ set -e
+ [[ 0 -eq 0 ]]
+ echo 'Host dev-dn90p.test.lan exists'
+ set +e
+ ipa service-find hdfs/dev-dn90p.test.lan@TEST.LAN
+ ERR=0
+ set -e
+ [[ 0 -eq 0 ]]
+ echo 'Principal hdfs/dev-dn90p.test.lan@TEST.LAN exists'
+ PRINC_EXISTS=yes
+ KADMIN='kadmin -k -t /var/run/cloudera-scm-server/cmf2399691723203886836.keytab -p cmadmin-84a0694d@TEST.LAN -r TEST.LAN'
+ '[' 432000 -gt 0 ']'
+ kadmin -k -t /var/run/cloudera-scm-server/cmf2399691723203886836.keytab -p cmadmin-84a0694d@TEST.LAN -r TEST.LAN -q 'modprinc -maxrenewlife "432000 sec" hdfs/dev-dn90p.test.lan@TEST.LAN'
kadmin: Missing parameters in krb5.conf required for kadmin client while initializing kadmin interface
```

</details><br>

>Утилита kadmin от MIT на данный момент не умеет определять admin_server из DNS._kerberos-adm._tcp<br>
<br>
This should list port 749 on your master KDC. Support for it is not complete at this time, but it will eventually be used by the kadmin program and related utilities. For now, you will also need the admin_server entry in krb5.conf.

### 2.4. Установка JCE Policy файлы для AES-256 шифрования
Этот шаг мы можем пропустить, так как "этот шаг не требуется в случае использования JDK => 1.8.0_161. В JDK 1.8.0_161 по умолчанию включено сильное шифрование без всякого ограничения."

Проверяем наличие поддержки AES256:
```bash
$ sudo rpm -qa *1.8
oracle-j2sdk1.8-1.8.0+update181-1.x86_64

$ kinit nifanteve
Password for nifanteve@TEST.LAN:

$ klist -e
Ticket cache: KEYRING:persistent:1000:1000
Default principal: nifanteve@TEST.LAN

Valid starting       Expires              Service principal
02/03/2021 11:16:42  02/04/2021 11:16:34  krbtgt/TEST.LAN@TEST.LAN
        Etype (skey, tkt): aes256-cts-hmac-sha1-96, aes256-cts-hmac-sha1-96
```

Видим 'aes256-cts-hmac-sha1-96', поэтому пропускаем этот шаг.

### 2.5. Kerberos-принципал для Cloudera Manager Server во FreeIPA
Забегая вперёд отметим, что в конце выполнения "мастера включения Kerberos" Cloudera Manager Server создаст host-принципалы и загрузит keytab'ы для всех сервисов в кластере. Для этого Cloudera Manager Server нуждается в отдельном принципале с соответствующими правами, позволяющими проделать эту работу.

Если вы используете FreeIPA, визард создаст свой собственный аккаунт с необходимыми привилегиями. Для этого потребуется ввод учётных данных администратора FreeIPA. Эти учетные данные не сохраняются и используются только для создания нового пользователя и роли (с именами cmadin- <random_id> и cmadminrole соответственно) и получения его keytab. Cloudera Manager сохраняет этот keytab для будущих операций Kerberos, таких как восстановление учетных данных аккаунтов служб CDH.

![](/img/ustanovka-cloudera-cdh-6.3.2-s-tls-i-kerberos-na-osnove-freeipa/image_12_1.png)

## 3. Включение Kerberos используя визард
### 3.1. Запуск
Выбираем в меню 'Enable Kerberos':

![](/img/ustanovka-cloudera-cdh-6.3.2-s-tls-i-kerberos-na-osnove-freeipa/image_12_2.png)

### 3.2. Getting Started
Первый экран Getting Started:

![](/img/ustanovka-cloudera-cdh-6.3.2-s-tls-i-kerberos-na-osnove-freeipa/image_12_3.png)

Ставим все галочки соглашаясь, что для включения в кластере Kerberos, мы уверены в том, что:
- Все узлы кластера введены в домен.
- Kerbros Ticker политика установлена в не ноль. В нашем случае, во FreeIPA указаны сроки в 7 дней и 24 часа: ![](/img/ustanovka-cloudera-cdh-6.3.2-s-tls-i-kerberos-na-osnove-freeipa/image_12_4.png)
- Сетевые имена хостов набраны в нижнем регистре (dev-mgm01p.test.lan).
- На всех узлах установлены пакеты, а они скорей всего установлены в нашем FreeIPA случае:
    - krb5-workstation
    - krb5-libs
    - freeipa-client (ipa-client)
- На Cloudera Manager Server установлен пакет 'openldap-clients', если будет использоваться Active Directory. В нашем случае с FreeIPA неприменимо.
- Мы имеем во FreeIPA УЗ с правами администратора, с помощью которой Cloudera Manager создаст свою спец УЗ 'cmadmin-<random>'.
- В кластере не включен HA для YARN Resource Manager, иначе понадобится выполнить процедуру: ![](/img/ustanovka-cloudera-cdh-6.3.2-s-tls-i-kerberos-na-osnove-freeipa/image_12_5.png)

### 3.3. Setup KDC
<table>
<tr>
<th>Property</th><th>Value</th><th>Description</th>
</tr>
<tr>
<td><b>KDC Type</b></td>
<td><span style="color: blue">◉ Red Hat IPA</span></td>
<td></td>
</tr>
<tr>
<td><b>Kerberos Encryption Types</b></td>
<td><span style="color: blue"><code>aes256-cts-hmac-sha1-96</code></span></td>
<td>Encryption types supported by KDC. Note: To use AES encryption, make sure you have deployed JCE Unlimited Strength Policy File by following the instructions here.</td>
</tr>
<tr>
<td><b>Kerberos Security Realm</b></td>
<td><span style="color: blue"><code>TEST.LAN</code></span></td>
<td>The realm to use for Kerberos security. Note: Changing this setting would clear up all existing credentials and keytabs from Cloudera Manager.</td>
</tr>
<tr>
<td><b>KDC Server Host</b><br>
<i>kdc</i></td>
<td><span style="color: blue"><code>dev-ipa01p.test.lan</code></span></td>
<td>Host where the KDC server is located. Port number is optional and can be provided as hostname[:port]</td>
</tr>
<tr>
<td><b>KDC Admin Server Host</b><i>admin_server</i></td>
<td><span style="color: blue"><code>dev-ipa01p.test.lan</code></span></td>
<td>Host where the KDC Admin server is located. Port number is optional and can be provided as hostname[:port]</td>
</tr>
</table>

### 3.4. Manage krb5.conf
Включаем эту настройку, чтобы Cloudera Manager управлял содержимым файла `/etc/krb5.conf`. После чего устанавливаем следующие параметры:
<table>
<tr>
<th>Property</th><th>Value</th><th>Description</th>
</tr>
<tr>
<td><b>DNS Lookup KDC</b><br>
<i>dns_lookup_kdc</i></td>
<td><span style="color: blue">☑</span></td>
<td>Indicate whether DNS SRV records should be used to locate the KDCs and other servers for a realm, if they are not listed in the krb5.conf information for the realm.</td>
</tr>
<tr>
<td><b>Forwardable Tickets</b><br>
<i>forwardable</i></td>
<td><span style="color: blue">☑</span></td>
<td>If this flag is true, initial tickets will be forwardable by default, if allowed by the KDC.</td>
</tr>
<tr>
<td><b>KDC Timeout</b><br>
<i>kdc_timeout</i></td>
<td><span style="color: blue"><code>1 seconds</code></span></td>
<td>The maximum time to wait for a reply from the KDC. A time of 0 seconds means "use the client's default".</td>
</tr>
<tr>
<td><b>Advanced Configuration Snippet (Safety Valve) for [libdefaults] section of krb5.conf</b></td>
<td><span style="color: blue">
<pre><code>
max_retries = 0
dns_lookup_realm = true
rdns = false
dns_canonicalize_hostname = false
udp_preference_limit = 0
</code></pre>
</span></td>
<td>For advanced use only. Any text here will be emitted verbatim in the [libdefaults] section of krb5.conf.</td>
</tr>
<tr>
<td><b>Advanced Configuration Snippet (Safety Valve) for the Default Realm in krb5.conf</b></td>
<td><span style="color: blue"><pre><code>
kdc = dev-ipa02p.test.lan
admin_server = dev-ipa02p.test.lan
kdc = dev-ipa03p.test.lan
admin_server = dev-ipa03p.test.lan
pkinit_anchors = FILE:/var/lib/ipa-client/pki/kdc-ca-bundle.pem
pkinit_pool = FILE:/var/lib/ipa-client/pki/ca-bundle.pem
</code></pre></span></td>
<td>For advanced use only. Any text here will be emitted verbatim in the [realms] section of krb5.conf for the specified security realm. If you want to add realms besides the default one, configure them using Advanced Configuration Snippet (Safety Valve) for remaining krb5.conf.</td>
</tr>
<tr>
<td><b>Advanced Configuration Snippet (Safety Valve) for remaining krb5.conf</b></td>
<td><span style="color: blue"><pre><code>
.test.lan = TEST.LAN
test.lan = TEST.LAN
</code></pre></span></td>
<td>For advanced use only. Cloudera Manager configures the [libdefaults], [realms] and [domain_realm] section of krb5.conf. Any text here will be emitted verbatim after them in krb5.conf.</td>
</tr>
</table>

### 3.5. Setup KDC Account
Здесь надо использовать УЗ с участием в группе 'admins', так как Cloudera создаст свою уникальную специальную УЗ с бессрочным паролем и привилегиями, перечисленными чуть ниже.

![](/img/ustanovka-cloudera-cdh-6.3.2-s-tls-i-kerberos-na-osnove-freeipa/image_12_6.png)

Для справки. Используя предоставленные учётные данные, Cloudera визард создаст во FreeIPA свою УЗ вида cmadmin-<random> с ролью cmadminrole, которая имеет привилегию 'Service Administrators', которая имеет разрешения:
- System: Add Services
- System: Manage Service Keytab
- System: Manage Service Keytab Permissions
- System: Manage Service Principals
- System: Modify Services
- System: Remove Services
- System: Add Service Delegations
- System: Modify Service Delegation Membership
- System: Read Service Delegations
- System: Remove Service Delegations

### 3.6. Import KDC Account Manager Credentials Command

![](/img/ustanovka-cloudera-cdh-6.3.2-s-tls-i-kerberos-na-osnove-freeipa/image_12_7.png)

>Во FreeIPA появилась УЗ 'cmadmin-<random>', в поле 'Job Title' которой я написал:
"Cloudera Manager TEST1 создал эту УЗ из мастера включения Kerberos. EP-973"

Если в дальнейшем потребуется изменить пароль для аккаунта 'cmadmin', то после изменения выполняем:

1. In the Cloudera Manager Admin Console, select Administration > Security.
2. Go to the Kerberos Credentials tab and click Import Kerberos Account Manager Credentials.
3. In the Import Kerberos Account Manager Credentials dialog box, enter the username and password for the user that can create principals for CDH cluster in the KDC.

На этом этапе может происходить ошибка "kinit: Client 'cmadmin-<random>@TEST.LAN' not found in Kerberos database while getting initial credentials". Ошибка происходила из-за недоступности первого контроллера домена, который был выключен для проверки отказоустойчивости клиентов.

### 3.7. Configure Kerberos

![](/img/ustanovka-cloudera-cdh-6.3.2-s-tls-i-kerberos-na-osnove-freeipa/image_12_8.png)

Так как требуемые пакеты у нас были установлены на этапе введения хостов в домен, то оставляем без изменений.

Просто ради интереса временно снял галку 'Use Default Kerberos Principals' и увидел:

![](/img/ustanovka-cloudera-cdh-6.3.2-s-tls-i-kerberos-na-osnove-freeipa/image_12_9.png)

### 3.8. Enable Kerberos Command
Здесь наблюдаем прогресс включения Kerberos, после чего видим результат:

![](/img/ustanovka-cloudera-cdh-6.3.2-s-tls-i-kerberos-na-osnove-freeipa/image_12_10.png)

### 3.9. Summary

![](/img/ustanovka-cloudera-cdh-6.3.2-s-tls-i-kerberos-na-osnove-freeipa/image_12_11.png)

## 5. Включение Kerberos-аутентификации для HTTP Web Консоли для Hadoop ролей
### 5.1. Включение и перезапуск сервисов
Аутентификацию для доступа к веб-консолям ролей HDFS и YARN можно включить с помощью параметра конфигурации для соответствующей службы. Чтобы включить эту аутентификацию, активизируем параметр 'Enable Kerberos Authentication for HTTP Web-Consoles' у каждой роли, сохраняем изменения и перезапускаем сервисы.

### 5.2. Ошибки после включения Kerberos-аутентификации
Пару раз, после включения опции 'Enable Kerberos' для роли HDFS, высветилась ошибка:

<span style="color: red">
General Error(s)
<br>
Role is missing Kerberos keytab. Go to the Kerberos Credentials page and click the Generate Missing Credentials button.
</span>
<br><br>

По совету из предупреждения выполнил 'Generate Missing Credentials', но в ответ получил баннер:

![](/img/ustanovka-cloudera-cdh-6.3.2-s-tls-i-kerberos-na-osnove-freeipa/image_12_12.png)

Игнорирую сообщение и перезапускаю кластер. После перезапуска кластера 'Yarn Healt test' выдал ошибку. Перезапустил YARN. Если судить по приборам, то всё в норме.
