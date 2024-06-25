---
title: "02. Установка первого FreeIPA-сервера"
date: 2021-12-24
weight: 12
description: >
  Описание процесса установки первого IPA-сервера.
tags:
  - FreeIPA 4.9.6
  - FreeIPA
  - AlmaLinux 8.5
slug: ustanovka-pervogo-freeipa-servera
draft: false
---

## 1. Установка первого FreeIPA-сервера на первом хосте
### 1.1. Описание опций установочного скрипта
Установку будем производить в бездиалоговом режиме с заданием всех необходимых опций из командной строки.

Описание некоторых опций:
- `--ds-password` — задаёт особо секретный пароль аккаунта 'cn=Directory Manager', на основе которого синхронизируются реплики IPA и зашифрован ключ Центра Сертификации. Так-как этой УЗ доступен весь LDAP-каталог, что позволяет наблюдать содержимое особо секретных веток, то необходимо ограничить известность этого пароля узким кругом лиц.
- `--admin-password` — задаёт пароль первого пользовательского аккаунта во FreeIPA, состоящего в группе 'admins', с помощью которого будет произведена первичная настройка FreeIPA. Членам группы 'admins' позволительны все доступные операции, что позволяет нанести необратимый вред установленному экземпляру FreeIPA, поэтому после первичной настройки заблокируем эту УЗ, а доступ к паролю ограничим узким кругом лиц.
- `--ip-address` — указывает соновной ip-адрес. Необязательный параметр в случае наличия только одного ip-адреса.
- `--unattended` — подавляет все запросы инсталлятора.
- `--setup-dns --no-forwarders` — добавляет роль DNS-сервера без настройки форвардеров, которые мы настроим позже, после установки первого контроллера.
- `--auto-reverse --allow-zone-overlap` — две опции, которые необходимы для автоматического добавления обратной зоны DNS, в которой находится основной ip-адрес сервера FreeIPA. (https://access.redhat.com/solutions/4237121)
- `--no-hbac-allow` — запрещает автоматическое добавление HBAC-правила, которое позволяет всем пользователям домена заходить на все машины домена через ssh.
- `--ntp-server` — задаёт ip-адрес NTP-сервера. Адрес будет добавлен в `/etc/chrony.conf`.
- `--mkhomedir` — опция для скрипта ipa-client-install. Скрипт автоматически вызывается из ipa-server-install после поднятия домена и необходим для введения локальной машины в домен FreeIPA. Опция же добавляет в локальную операционную систему PAM-правило, отвечающее за автоматическое создание домашней директории для входящего на эту машину по ssh пользователю.
- `--subject-base` — the certificate subject base (default O=\<realm-name\>).
- `--ca-subject` — the CA certificate subject DN (default CN=Certificate Authority,O=\<realm-name\>).
Ниже показаны различия в именах сертификатах без использования или с использованием этих опций.

<table>
  <tr>
    <th>Сертификат №1<br>CN=Certificate Authority,O=TEST2.LAN</th>
    <th>Сертификат №1<br>CN=test3.lan Root CA,O=test3.lan</th>
  </tr>
  <tr>
    <td>
    <img src="/img/sozdaniye-i-konfigurirovaniye-novogo-domena-ipa-4-9-6-na-almalinux-8-5/cn-certificate-authority.png">
    </td>
    <td>
      <img src="/img/sozdaniye-i-konfigurirovaniye-novogo-domena-ipa-4-9-6-na-almalinux-8-5/cn-certificate-authority.png">
    </td>
  </tr>
</table>

В Chromium стандартный и специально именованный сертификаты отображаются так:


### 1.2. Правила задания паролей для DM-аккаунта
Судя по исходникам, пароли могут содержать все ASCII-символы, кроме обратного слэша (видим следующее присваивание — bad_characters = '\\' — в регулярных выражениях символ обратного слэша обычно обозначается двумя обратными слэшами). Наверное, наравне с обратным слэшем, лучше будет воздержаться от использования одинарных, обратных и двойных кавычек, а также прямого слэша и вертикальной черты.

Для генерации пароля, который не будет включать в себя символы слэшей и кавычек (одинарных, двойных и обратных), проще использовать bash-строку:
```bash
tr -dc 'A-Za-z0-9!#$%&()*+,-.:;<=>?@[]^_{}~' </dev/urandom | head -c 43; echo
```

или утилиту `pwgen`:
```bash
$ pwgen --secure --symbols --remove-chars="\\|/\"\'\`" 42 1
tD*$6?6F~sBX.Q_8<_7RmR^$~1]sWJs70rY5_TqFyO
```

Код, отвечающий за проверку паролей, можно найти в файле `freeipa-4.9.6/ipaserver/install/server/install.py`:
<details>
<summary>stdout...</summary>

```
def validate_dm_password(password):
   if len(password) < 8:
       raise ValueError("Password must be at least 8 characters long")
   if any(ord(c) < 0x20 for c in password):
       raise ValueError("Password must not contain control characters")
   if any(ord(c) >= 0x7F for c in password):
       raise ValueError("Password must only contain ASCII characters")

   # Disallow characters that pkisilent doesn't process properly:
   bad_characters = '\\'
   if any(c in bad_characters for c in password):
       raise ValueError('Password must not contain these characters: %s' %
                        ', '.join('"%s"' % c for c in bad_characters))

   # TODO: Check https://fedorahosted.org/389/ticket/47849
   # Actual behavior of setup-ds.pl is that it does not accept white
   # space characters in password when called interactively but does when
   # provided such password in INF file. But it ignores leading and trailing
   # whitespaces in INF file.

   # Disallow leading/trailing whaitespaces
   if password.strip() != password:
       raise ValueError('Password must not start or end with whitespace.')

def validate_admin_password(password):
   if len(password) < 8:
       raise ValueError("Password must be at least 8 characters long")
   if any(ord(c) < 0x20 for c in password):
       raise ValueError("Password must not contain control characters")
   if any(ord(c) >= 0x7F for c in password):
       raise ValueError("Password must only contain ASCII characters")

   # Disallow characters that pkisilent doesn't process properly:
   bad_characters = '\\'
   if any(c in bad_characters for c in password):
       raise ValueError('Password must not contain these characters: %s' %
                        ', '.join('"%s"' % c for c in bad_characters))
```
</details>

### 1.3. Процесс установки первого сервера FreeIPA
Выполняем следующий скрипт и ждём окончания процедуры. В случае наличия на хосте только одного ip-адреса, переменную IP_ADDRESS и опцию `--ip-address` можно не применять.
```bash
# Для облегчения дальнейшей генерации паролей устанавливаем пакет pwgen
sudo dnf -y install pwgen

# Generating new passwords
DM_PASSWORD=$(pwgen --secure --symbols --remove-chars="\\|/\"\'\`" 42 1)
ADMIN_PASSWORD=$(pwgen 14 1)

# Saving new passwords to /root/freeipa_passwords.txt
[ -f /root/freeipa_passwords.txt ] && \
sudo mv /root/freeipa_passwords.txt /root/freeipa_passwords.txt.$(date +%y%m%d-%H%M%S)
echo -e "${DM_PASSWORD}\n${ADMIN_PASSWORD}" | sudo tee /root/freeipa_passwords.txt

IP_ADDRESS="10.1.160.23"
REALM_NAME="TEST3.LAN"
NTP_SERVER1="10.0.0.5"
NTP_SERVER2="10.0.0.6"
NTP_SERVER3="10.0.0.7"
SUBJECT_BASE="O=test3.lan DO Evo Group"
CA_SUBJECT="CN=test3.lan DO Evo Group Root CA,O=test3.lan DO Evo Group"

sudo ipa-server-install --unattended \
  --ds-password=${DM_PASSWORD} --admin-password=${ADMIN_PASSWORD} \
  --ip-address=${IP_ADDRESS} --realm=${REALM_NAME} \
  --setup-dns --no-forwarders --auto-reverse --allow-zone-overlap \
  --no-hbac-allow --mkhomedir \
  --ntp-server=${NTP_SERVER1} --ntp-server=${NTP_SERVER2} --ntp-server=${NTP_SERVER3} \
  --subject-base="${SUBJECT_BASE}" --ca-subject="${CA_SUBJECT}"
```

<details>
<summary>stdout...</summary>

```
The log file for this installation can be found in /var/log/ipaserver-install.log
==============================================================================
This program will set up the IPA Server.
Version 4.9.6

This includes:
  * Configure a stand-alone CA (dogtag) for certificate management
  * Configure the NTP client (chronyd)
  * Create and configure an instance of Directory Server
  * Create and configure a Kerberos Key Distribution Center (KDC)
  * Configure Apache (httpd)
  * Configure DNS (bind)
  * Configure SID generation
  * Configure the KDC to enable PKINIT

Warning: skipping DNS resolution of host dev-ipa04.test3.lan
The domain name has been determined based on the host name.

Checking DNS domain test3.lan., please wait ...
DNS check for domain test3.lan. failed: The DNS operation timed out after 24.01217746734619 seconds.
Invalid IP address fe80::250:56ff:fea2:b7ed for dev-ipa04.test3.lan: cannot use link-local IP address fe80::250:56ff:
fea2:b7ed
Checking DNS forwarders, please wait ...
Reverse zone 160.15.10.in-addr.arpa. will be created
Using reverse zone(s) 160.15.10.in-addr.arpa.
Trust is configured but no NetBIOS domain name found, setting it now.

The IPA Master Server will be configured with:
Hostname:       dev-ipa04.test3.lan
IP address(es): 10.1.160.23
Domain name:    test3.lan  
Realm name:     TEST3.LAN  


The CA will be configured with:
Subject DN:   CN=test3.lan DO Evo Group Root CA
Subject base: O=test3.lan DO Evo Group
Chaining:     self-signed  

BIND DNS server will be configured to serve IPA domain with:
Forwarders:       8.8.8.8, 8.8.8.9
Forward policy:   only
Reverse zone(s):  160.15.10.in-addr.arpa.

NTP server:     10.0.0.5
NTP server:     10.0.0.6
NTP server:     10.0.0.7
Adding [10.1.160.23 dev-ipa04.test3.lan] to your /etc/hosts file
Disabled p11-kit-proxy
Synchronizing time
Configuration of chrony was changed by installer.
Attempting to sync time with chronyc.
Time synchronization was successful.
Configuring directory server (dirsrv). Estimated time: 30 seconds
  [1/40]: creating directory server instance
  [2/40]: tune ldbm plugin
  [3/40]: adding default schema
  [4/40]: enabling memberof plugin
  [5/40]: enabling winsync plugin
  [6/40]: configure password logging
  [7/40]: configuring replication version plugin
  [8/40]: enabling IPA enrollment plugin
  [9/40]: configuring uniqueness plugin
  [10/40]: configuring uuid plugin
  [11/40]: configuring modrdn plugin
  [12/40]: configuring DNS plugin
  [13/40]: enabling entryUSN plugin
  [14/40]: configuring lockout plugin
  [15/40]: configuring topology plugin
  [16/40]: creating indices
  [17/40]: enabling referential integrity plugin
  [18/40]: configuring certmap.conf
  [19/40]: configure new location for managed entries
  [20/40]: configure dirsrv ccache and keytab
  [21/40]: enabling SASL mapping fallback
  [22/40]: restarting directory server
  [23/40]: adding sasl mappings to the directory
  [24/40]: adding default layout
  [25/40]: adding delegation layout
  [26/40]: creating container for managed entries
  [27/40]: configuring user private groups
  [28/40]: configuring netgroups from hostgroups
  [29/40]: creating default Sudo bind user
  [30/40]: creating default Auto Member layout
  [31/40]: adding range check plugin
  [32/40]: adding entries for topology management
  [33/40]: initializing group membership
  [34/40]: adding master entry
  [35/40]: initializing domain level
  [36/40]: configuring Posix uid/gid generation
  [37/40]: adding replication acis
  [38/40]: activating sidgen plugin
  [39/40]: activating extdom plugin
  [40/40]: configuring directory to start on boot
Done configuring directory server (dirsrv).
Configuring Kerberos KDC (krb5kdc)
  [1/10]: adding kerberos container to the directory
  [2/10]: configuring KDC  
  [3/10]: initialize kerberos container
  [4/10]: adding default ACIs
  [5/10]: creating a keytab for the directory
  [6/10]: creating a keytab for the machine
  [7/10]: adding the password extension to the directory
  [8/10]: creating anonymous principal
  [9/10]: starting the KDC
  [10/10]: configuring KDC to start on boot
Done configuring Kerberos KDC (krb5kdc).
Configuring kadmin
  [1/2]: starting kadmin
  [2/2]: configuring kadmin to start on boot
Done configuring kadmin.
Configuring ipa-custodia
  [1/5]: Making sure custodia container exists
  [2/5]: Generating ipa-custodia config file
  [3/5]: Generating ipa-custodia keys
  [4/5]: starting ipa-custodia
  [5/5]: configuring ipa-custodia to start on boot
Done configuring ipa-custodia.
Configuring certificate server (pki-tomcatd). Estimated time: 3 minutes
  [1/28]: configuring certificate server instance
  [2/28]: stopping certificate server instance to update CS.cfg
  [3/28]: backing up CS.cfg
  [4/28]: Add ipa-pki-wait-running
  [5/28]: secure AJP connector
  [6/28]: reindex attributes
  [7/28]: exporting Dogtag certificate store pin
  [8/28]: disabling nonces
  [9/28]: set up CRL publishing
  [10/28]: enable PKIX certificate path discovery and validation
  [11/28]: authorizing RA to modify profiles
  [12/28]: authorizing RA to manage lightweight CAs
  [13/28]: Ensure lightweight CAs container exists
  [14/28]: starting certificate server instance
  [15/28]: configure certmonger for renewals
  [16/28]: requesting RA certificate from CA
  [17/28]: publishing the CA certificate
  [18/28]: adding RA agent as a trusted user
  [19/28]: configure certificate renewals
  [20/28]: Configure HTTP to proxy connections
  [21/28]: updating IPA configuration
  [22/28]: enabling CA instance
  [23/28]: importing IPA certificate profiles
  [24/28]: migrating certificate profiles to LDAP
  [25/28]: adding default CA ACL
  [26/28]: adding 'ipa' CA entry
  [27/28]: configuring certmonger renewal for lightweight CAs
  [28/28]: deploying ACME service
Done configuring certificate server (pki-tomcatd).
Configuring directory server (dirsrv)
  [1/3]: configuring TLS for DS instance
  [2/3]: adding CA certificate entry
  [3/3]: restarting directory server
Done configuring directory server (dirsrv).
Configuring ipa-otpd
  [1/2]: starting ipa-otpd
  [2/2]: configuring ipa-otpd to start on boot
Done configuring ipa-otpd.
Configuring the web interface (httpd)
  [1/21]: stopping httpd
  [2/21]: backing up ssl.conf
  [3/21]: disabling nss.conf
  [4/21]: configuring mod_ssl certificate paths
  [5/21]: setting mod_ssl protocol list
  [6/21]: configuring mod_ssl log directory
  [7/21]: disabling mod_ssl OCSP
  [8/21]: adding URL rewriting rules
  [9/21]: configuring httpd
Nothing to do for configure_httpd_wsgi_conf
  [10/21]: setting up httpd keytab
  [11/21]: configuring Gssproxy
  [12/21]: setting up ssl  
  [13/21]: configure certmonger for renewals
  [14/21]: publish CA cert
  [15/21]: clean up any existing httpd ccaches
  [16/21]: configuring SELinux for httpd
  [17/21]: create KDC proxy config
  [18/21]: enable KDC proxy
  [19/21]: starting httpd  
  [20/21]: configuring httpd to start on boot
  [21/21]: enabling oddjobd
Done configuring the web interface (httpd).
Configuring Kerberos KDC (krb5kdc)
  [1/1]: installing X509 Certificate for PKINIT
Done configuring Kerberos KDC (krb5kdc).
Applying LDAP updates
Upgrading IPA:. Estimated time: 1 minute 30 seconds
  [1/10]: stopping directory server
  [2/10]: saving configuration
  [3/10]: disabling listeners
  [4/10]: enabling DS global lock
  [5/10]: disabling Schema Compat
  [6/10]: starting directory server
  [7/10]: upgrading server
Could not get dnaHostname entries in 60 seconds
  [8/10]: stopping directory server
  [9/10]: restoring configuration
  [10/10]: starting directory server
Done.
Restarting the KDC
dnssec-validation yes
Configuring DNS (named)
  [1/12]: generating rndc key file
  [2/12]: adding DNS container
  [3/12]: setting up our zone
  [4/12]: setting up reverse zone
  [5/12]: setting up our own record
  [6/12]: setting up records for other masters
  [7/12]: adding NS record to the zones
  [8/12]: setting up kerberos principal
  [9/12]: setting up named.conf
created new /etc/named.conf
created named user config '/etc/named/ipa-ext.conf'
created named user config '/etc/named/ipa-options-ext.conf'
created named user config '/etc/named/ipa-logging-ext.conf'
  [10/12]: setting up server configuration
  [11/12]: configuring named to start on boot
  [12/12]: changing resolv.conf to point to ourselves
Done configuring DNS (named).
Restarting the web server to pick up resolv.conf changes
Configuring DNS key synchronization service (ipa-dnskeysyncd)
  [1/7]: checking status
  [2/7]: setting up bind-dyndb-ldap working directory
  [3/7]: setting up kerberos principal
  [4/7]: setting up SoftHSM
  [5/7]: adding DNSSEC containers
  [6/7]: creating replica keys
  [7/7]: configuring ipa-dnskeysyncd to start on boot
Done configuring DNS key synchronization service (ipa-dnskeysyncd).
Restarting ipa-dnskeysyncd
Restarting named
Updating DNS system records
Configuring SID generation
  [1/8]: creating samba domain object
  [2/8]: adding admin(group) SIDs
  [3/8]: adding RID bases  
  [4/8]: updating Kerberos config
'dns_lookup_kdc' already set to 'true', nothing to do.
  [5/8]: activating sidgen task
  [6/8]: restarting Directory Server to take MS PAC and LDAP plugins changes into account
  [7/8]: adding fallback group
  [8/8]: adding SIDs to existing users and groups
This step may take considerable amount of time, please wait..
Done.
Configuring client side components
This program will set up IPA client.
Version 4.9.6

Using existing certificate '/etc/ipa/ca.crt'.
Client hostname: dev-ipa04.test3.lan
Realm: TEST3.LAN
DNS Domain: test3.lan
IPA Server: dev-ipa04.test3.lan
BaseDN: dc=test3,dc=lan

Configured sudoers in /etc/authselect/user-nsswitch.conf
Configured /etc/sssd/sssd.conf
Systemwide CA database updated.
Adding SSH public key from /etc/ssh/ssh_host_rsa_key.pub
Adding SSH public key from /etc/ssh/ssh_host_ed25519_key.pub
Adding SSH public key from /etc/ssh/ssh_host_ecdsa_key.pub
SSSD enabled
Configured /etc/openldap/ldap.conf
Configured /etc/ssh/ssh_config
Configured /etc/ssh/sshd_config
Configuring test3.lan as NIS domain.
Client configuration complete.
The ipa-client-install command was successful

==============================================================================
Setup complete

Next steps:
        1. You must make sure these network ports are open:
                TCP Ports:
                  * 80, 443: HTTP/HTTPS
                  * 389, 636: LDAP/LDAPS
                  * 88, 464: kerberos
                  * 53: bind
                UDP Ports:
                  * 88, 464: kerberos
                  * 53: bind
                  * 123: ntp

        2. You can now obtain a kerberos ticket using the command: 'kinit admin'
           This ticket will allow you to use the IPA tools (e.g., ipa user-add)
           and the web user interface.

Be sure to back up the CA certificates stored in /root/cacert.p12
These files are required to create replicas. The password for these
files is the Directory Manager password
The ipa-server-install command was successful
```
</details>

### 1.4. Вход в Web UI FreeIPA
Пока отсутствует DNS resolving для нового домена test3.lan, добавляем на свою рабочую машину в файл `/etc/hosts` запись для FreeIPA-сервера:
```bash
echo "10.1.160.23 dev-ipa04.test3.lan" | sudo tee -a /etc/hosts
```

Теперь можно открыть WEB UI страницу FreeIPA-сервера по адресу 'https://dev-ipa04.test3.lan' и зайти в консоль управления под учётными данными аккаунта 'admin':

CA-сертификат, для добавления нового домена test3.lan в "надёжные", можно скачать здесь:

### 1.5. Настройка DNS перед добавление дополнительных реплик
По умолчанию, при введении новой машины в домен, соответствующая PTR-запись в обратной зоне DNS не создаётся. Если я правильно помню, то это делается в целях сохранения производительности FreeIPA. Видимо, при достаточном количестве машин одновременно меняющих ip-адреса.... бла-бла-бла. Опция 'Allow PTR sync' доступна для отдельной DNS-зоны и глобально для DNS-сервера. В нашем случае включим опцию для DNS-сервера.

Также добавим пару форвардеров и переведём политику пересылки DNS-запросов в режим 'forward only', так как наши FreeIPA-серверы лишены доступа к интернету и не имеют возможности  рекурсивно опрашивать публичные DNS-серверы. Напомню, что в режиме 'forward first', DNS-сервер сначала отправит запрос форвардер-серверами. В случае не успеха, DNS-сервер начнёт сам опрашивать те самые корневые DNS-сервера и в зависимости от их ответа, дальше рекурсивно постарается дойти до авторитативного DNS-сервера, который держит зону имеющую ответ на первоначальный запрос.



Включаем опции для DNS-сервера одним из двух способов:

<table>
<tr>
  <th>Через командную строку</th>
  <th>Через Web UI по адресу</th>
</tr>
<tr>
  <td>
<pre><code>
kinit admin<br>
ipa dnsconfig-mod \
  --allow-sync-ptr=1 \
  --forwarder="8.8.8.8" \
  --forwarder="1.1.1.1" \
  --forward-policy='only'
</code></pre>
  </td>
  <td>
https://url-ipa-server.example.org/ipa/ui/#/e/dnsconfig/details
  </td>
</tr>
</table>

### 1.6. Отключение анонимного доступа к LDAP-каталогу
Процедура выполняется для каждого сервера, так как изменения производятся в нереплицируемой части LDAP-каталога 'cn=config'.
```bash
IPAREPLICA="dev-ipa04.test3.lan"
set +o history
DMPASS="qMSxV(EvB#o>Brd,0<6n;"
set -o history

ldapmodify -x -D "cn=Directory Manager" -w ${DMPASS} -h ${IPAREPLICA} -ZZ << EOF
dn: cn=config
changetype: modify
replace: nsslapd-allow-anonymous-access
nsslapd-allow-anonymous-access: rootdse
EOF
```

```bash
ipactl restart
```

## 2. Бэкап важной особо секретной информации
После первого поднятия домена необходимо сохранить следующую важную информацию в безопасном месте с ограниченным доступом:
1. Пароль аккаунта 'cn=Directory Manager'. Находится в первой строке файла `/root/freeipa_passwords.txt`.
2. Пароль аккаунта 'uid=admin,cn=users,cn=accounts,dc=test3,dc=lan'. Находится в второй строке файла `/root/freeipa_passwords.txt`.
3. Файл `/root/cacert.p12` с приватным ключом и сертификатом от Центра Сертификации. Здесь необходимо продублировать пароль от 'cn=Directory Manager', так как на его основе, на данный момент, зашифровано содержимое файла `cacert.p12`.

Надёжно сохранив пароли для УЗ 'cn=Directory Manager' и 'uid=admin' в надёжном месте, удаляем файл `/root/freeipa_passwords.txt`:
```bash
sudo shred -u /root/freeipa_passwords.txt
```

Напомню, что безопасное удаление с помощью утилиты 'shred' может быть обратимо, например, на журналируемых файловых системах. Поэтому, в некоторых случаях, необходимо переработать данный сценарий установки FreeIPA-сервера, где пароли бы не сохранялись в файл на жёсткий диск.

## 3. Утилита ipa-healthcheck
Запуск ipa-healtcheck завершается с четырьмя замечаниями:
```bash
sudo ipa-healthcheck
[                                                                                                                      
  {
    "source": "ipahealthcheck.ipa.certs",
    "check": "IPADogtagCertsMatchCheck",
    "result": "ERROR",
    "uuid": "9128d242-ce51-460d-8d86-55327e988f52",
    "when": "20220110145522Z",
    "duration": "0.475297",
    "kw": {
      "key": "ocspSigningCert cert-pki-ca",
      "nickname": "ocspSigningCert cert-pki-ca",
      "dbdir": "/etc/pki/pki-tomcat/alias",
      "msg": "{nickname} certificate in NSS DB {dbdir} does not match entry in LDAP"
    }
  },
  {
    "source": "ipahealthcheck.ipa.certs",
    "check": "IPADogtagCertsMatchCheck",
    "result": "ERROR",
    "uuid": "19846c1c-a7f5-4dc0-9821-716d644fc063",
    "when": "20220110145522Z",
    "duration": "0.523545",
    "kw": {
      "key": "subsystemCert cert-pki-ca",
      "nickname": "subsystemCert cert-pki-ca",
      "dbdir": "/etc/pki/pki-tomcat/alias",
      "msg": "{nickname} certificate in NSS DB {dbdir} does not match entry in LDAP"
    }
  },
  {
    "source": "ipahealthcheck.ipa.certs",
    "check": "IPADogtagCertsMatchCheck",
    "result": "ERROR",
    "uuid": "87f5ae43-3460-4aca-8d58-b31a2370b460",
    "when": "20220110145522Z",
    "duration": "0.571182",
    "kw": {
      "key": "auditSigningCert cert-pki-ca",
      "nickname": "auditSigningCert cert-pki-ca",
      "dbdir": "/etc/pki/pki-tomcat/alias",
      "msg": "{nickname} certificate in NSS DB {dbdir} does not match entry in LDAP"
    }
  },
  {
    "source": "ipahealthcheck.ipa.certs",
    "check": "IPADogtagCertsMatchCheck",
    "result": "ERROR",
    "uuid": "6833a0ac-0e37-4185-b9de-9a01e91bc8e2",
    "when": "20220110145522Z",
    "duration": "0.618206",
    "kw": {
      "key": "Server-Cert cert-pki-ca",
      "nickname": "Server-Cert cert-pki-ca",
      "dbdir": "/etc/pki/pki-tomcat/alias",
      "msg": "{nickname} certificate in NSS DB {dbdir} does not match entry in LDAP"
    }
  }
]
```
Надо будет разрешить эти проблемки.

```bash
certutil -d sql:/etc/pki/pki-tomcat/alias -L

Certificate Nickname                                         Trust Attributes
                                                             SSL,S/MIME,JAR/XPI

caSigningCert cert-pki-ca                                    CTu,Cu,Cu
ocspSigningCert cert-pki-ca                                  u,u,u
auditSigningCert cert-pki-ca                                 u,u,Pu
subsystemCert cert-pki-ca                                    u,u,u
Server-Cert cert-pki-ca                                      u,u,u
```
