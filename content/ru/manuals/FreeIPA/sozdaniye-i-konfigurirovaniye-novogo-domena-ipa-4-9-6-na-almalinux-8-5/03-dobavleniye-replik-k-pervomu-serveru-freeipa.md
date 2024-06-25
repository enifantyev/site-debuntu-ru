---
title: "03. Добавление реплик к первому серверу FreeIPA"
date: 2021-12-24
weight: 13
description: >
  Описание процесса добавления реплик к первому IPA-серверу.
tags:
  - FreeIPA 4.9.6
  - FreeIPA
  - AlmaLinux 8.5
slug: dobavleniye-replik-k-pervomu-serveru-freeipa
draft: false
---

## 1. Заметки по установке реплик
В отличие от установки первого FreeIPA-сервере, где скрипт `ipa-server-install` сначала поднял контроллер домена с новым доменом, а потом ввёл машину в этот домен, то в случае добавления реплик, необходимо сначала ввести машины в домен, а только потом установить на машины ПО контроллеров домена.

## 2. Введение машин в домен
На второй и третьей реплике выполняем одинаковые действия. Отличия могут возникнуть в том случае, если на машинах используется несколько ip-адресов. В этом случае, указываем 'master ip address' в опции '--ip-address'.

### Указание DNS-резолвера
На следующем хосте, предназначенном для дополнительной реплики, добавляем запись в `/etc/resolv.conf`, чтобы система могла найти контроллер домена 'test.lan':
```bash
echo "nameserver 10.1.160.23" | sudo tee /etc/resolv.conf
```

### Ввод хоста в домен
{{% pageinfo color="primary" %}}
При вводе машины в домен, если на машине один ip-адрес, то опцию '--ip-address' можно опустить.

Также, в большинстве случаев, можно опустить опцию '--realm', так как это значение, при правильно работающем DNS, автоматически разрешится из SRV-записи '_kerberos.example.org'.

Опцией '--ntp-server' можно пренебречь, если в DNS присутствуют необходимые записи. (Необходимо разобраться в этом вопросе).
{{% /pageinfo %}}

1. Запускаем процедуру ввода машины в домен:
    ```bash
    set +o history           # Отключаем запись history
    PRINCIPAL='admin'        # Принципал с ролью 'Enrollment Administrator'
    PASSWORD='eik3Co2rohnah'
    set -o history           # Возвращаем запись history
    IP_ADDRESS='10.1.160.24'
    REALM_NAME='TEST3.LAN'
    NTP_SERVER1='10.0.0.5'
    NTP_SERVER2='10.0.0.6'
    NTP_SERVER3='10.0.0.7'
      
    sudo ipa-client-install --unattended \
      --principal=${PRINCIPAL} --password=${PASSWORD} \
      --ip-address=${IP_ADDRESS} \
      --realm=${REALM_NAME} \
      --ntp-server=${NTP_SERVER1} --ntp-server=${NTP_SERVER2} --ntp-server=${NTP_SERVER3} \
      --mkhomedir
      ```

    <details>
    <summary>stdout...</summary>

    ```
    This program will set up IPA client.
    Version 4.9.6
    
    Discovery was successful!
    Client hostname: dev-ipa05p.test3.lan
    Realm: TEST3.LAN
    DNS Domain: test3.lan
    IPA Server: dev-ipa04p.test3.lan
    BaseDN: dc=test3,dc=lan
    NTP server: 10.1.85.5
    NTP server: 10.1.85.6
    NTP server: 10.1.85.7
    
    Synchronizing time
    Configuration of chrony was changed by installer.
    Attempting to sync time with chronyc.
    Time synchronization was successful.
    Successfully retrieved CA cert
        Subject:     CN=test3.lan DO Evo Group Root CA,O=test3.lan DO Evo Group
        Issuer:      CN=test3.lan DO Evo Group Root CA,O=test3.lan DO Evo Group
        Valid From:  2021-12-20 17:35:47
        Valid Until: 2041-12-20 17:35:47
    
    Enrolled in IPA realm TEST3.LAN
    Created /etc/ipa/default.conf
    Configured sudoers in /etc/authselect/user-nsswitch.conf
    Configured /etc/sssd/sssd.conf
    Configured /etc/krb5.conf for IPA realm TEST3.LAN
    Systemwide CA database updated.
    Hostname (dev-ipa05p.test3.lan) does not have A/AAAA record.
    Missing reverse record(s) for address(es): 10.1.160.24.
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
    ```
    </details>

## 3. Процесс установки дополнительных реплик FreeIPA
### 3.1. Установка IPA-реплики
1. На обоих хостах, поочерёдно, выполняем установку FreeIPA-реплик:
    ```bash
    # Временно выключаем запись history для предотвращения компрометации пароля
    set +o history
    
    # Принципал любой, но с правом добавления реплик в домен
    PRINCIPAL='admin'
    PASSWORD='eik3Co2rohnah'
    
    # Возвращаем запись history
    set -o history
    
    IP_ADDRESS='10.1.160.24'
    
    sudo ipa-replica-install --unattended \
      --principal=${PRINCIPAL} --admin-password=${PASSWORD} \
      --ip-address=${IP_ADDRESS} \
      --setup-ca \
      --setup-dns --no-forwarders
      ```

    <details>
    <summary>stdout...</summary>

    ```
    Lookup failed: Preferred host dev-ipa05p.test3.lan does not provide DNS.
    Run connection check to master
    Connection check OK
    Disabled p11-kit-proxy
    Configuring directory server (dirsrv). Estimated time: 30 seconds
      [1/38]: creating directory server instance
      [2/38]: tune ldbm plugin 
      [3/38]: adding default schema
      [4/38]: enabling memberof plugin
      [5/38]: enabling winsync plugin
      [6/38]: configure password logging
      [7/38]: configuring replication version plugin
      [8/38]: enabling IPA enrollment plugin
      [9/38]: configuring uniqueness plugin
      [10/38]: configuring uuid plugin
      [11/38]: configuring modrdn plugin
      [12/38]: configuring DNS plugin
      [13/38]: enabling entryUSN plugin
      [14/38]: configuring lockout plugin
      [15/38]: configuring topology plugin
      [16/38]: creating indices
      [17/38]: enabling referential integrity plugin
      [18/38]: configuring certmap.conf
      [19/38]: configure new location for managed entries
      [20/38]: configure dirsrv ccache and keytab
      [21/38]: enabling SASL mapping fallback
      [22/38]: restarting directory server
      [23/38]: creating DS keytab
      [24/38]: ignore time skew for initial replication
      [25/38]: setting up initial replication
    Starting replication, please wait until this has completed.
    Update in progress, 5 seconds elapsed
    Update succeeded
    
      [26/38]: prevent time skew after initial replication
      [27/38]: adding sasl mappings to the directory
      [28/38]: updating schema 
      [29/38]: setting Auto Member configuration
      [30/38]: enabling S4U2Proxy delegation
      [31/38]: initializing group membership
      [32/38]: adding master entry
      [33/38]: initializing domain level
      [34/38]: configuring Posix uid/gid generation
      [35/38]: adding replication acis
      [36/38]: activating sidgen plugin
      [37/38]: activating extdom plugin
      [38/38]: configuring directory to start on boot
    Done configuring directory server (dirsrv).
    Replica DNS records could not be added on master: Insufficient access: Insufficient 'add' privilege to add the entry 'idnsname=dev-ipa05p,idnsname=test3.lan.,cn=dns,dc=test3,dc=lan'.
    Configuring Kerberos KDC (krb5kdc)
      [1/5]: configuring KDC
      [2/5]: adding the password extension to the directory
      [3/5]: creating anonymous principal
      [4/5]: starting the KDC  
      [5/5]: configuring KDC to start on boot
    Done configuring Kerberos KDC (krb5kdc).
    Configuring kadmin
      [1/2]: starting kadmin
      [2/2]: configuring kadmin to start on boot
    Done configuring kadmin.
    Configuring directory server (dirsrv)
      [1/3]: configuring TLS for DS instance
      [2/3]: importing CA certificates from LDAP
      [3/3]: restarting directory server
    Done configuring directory server (dirsrv).
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
    Configuring ipa-otpd
      [1/2]: starting ipa-otpd 
      [2/2]: configuring ipa-otpd to start on boot
    Done configuring ipa-otpd. 
    Custodia uses 'dev-ipa04p.test3.lan' as master peer.
    Configuring ipa-custodia
      [1/4]: Generating ipa-custodia config file
      [2/4]: Generating ipa-custodia keys
      [3/4]: starting ipa-custodia
      [4/4]: configuring ipa-custodia to start on boot
    Done configuring ipa-custodia.
    Configuring certificate server (pki-tomcatd). Estimated time: 3 minutes
      [1/29]: creating certificate server db
      [2/29]: setting up initial replication
    Starting replication, please wait until this has completed.
    Update in progress, 7 seconds elapsed
    Update succeeded
    
      [3/29]: creating ACIs for admin
      [4/29]: creating installation admin user
      [5/29]: configuring certificate server instance
      [6/29]: stopping certificate server instance to update CS.cfg
      [7/29]: backing up CS.cfg
      [8/29]: Add ipa-pki-wait-running
      [9/29]: secure AJP connector
      [10/29]: reindex attributes
      [11/29]: exporting Dogtag certificate store pin
      [12/29]: disabling nonces
      [13/29]: set up CRL publishing
      [14/29]: enable PKIX certificate path discovery and validation
      [15/29]: authorizing RA to modify profiles
      [16/29]: authorizing RA to manage lightweight CAs
      [17/29]: Ensure lightweight CAs container exists
      [18/29]: destroying installation admin user
      [19/29]: starting certificate server instance
      [20/29]: Finalize replication settings
      [21/29]: configure certmonger for renewals
      [22/29]: Importing RA key
      [23/29]: configure certificate renewals
      [24/29]: Configure HTTP to proxy connections
      [25/29]: updating IPA configuration
      [26/29]: enabling CA instance
      [27/29]: importing IPA certificate profiles
      [28/29]: configuring certmonger renewal for lightweight CAs
      [29/29]: deploying ACME service
    Done configuring certificate server (pki-tomcatd).
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
      [8/10]: stopping directory server
      [9/10]: restoring configuration
      [10/10]: starting directory server
    Done.
    Finalize replication settings
    Restarting the KDC
    dnssec-validation yes
    Configuring DNS (named)
      [1/8]: generating rndc key file
      [2/8]: setting up our own record
      [3/8]: adding NS record to the zones
      [4/8]: setting up kerberos principal
      [5/8]: setting up named.conf
    created new /etc/named.conf
    created named user config '/etc/named/ipa-ext.conf'
    created named user config '/etc/named/ipa-options-ext.conf'
    named user config '/etc/named/ipa-logging-ext.conf' already exists
      [6/8]: setting up server configuration
      [7/8]: configuring named to start on boot
      [8/8]: changing resolv.conf to point to ourselves
    Done configuring DNS (named).
    Restarting the web server to pick up resolv.conf changes
    Configuring DNS key synchronization service (ipa-dnskeysyncd)
      [1/7]: checking status
      [2/7]: setting up bind-dyndb-ldap working directory
      [3/7]: setting up kerberos principal
      [4/7]: setting up SoftHSM
      [5/7]: adding DNSSEC containers
    DNSSEC container exists (step skipped)
      [6/7]: creating replica keys
      [7/7]: configuring ipa-dnskeysyncd to start on boot
    Done configuring DNS key synchronization service (ipa-dnskeysyncd).
    Restarting ipa-dnskeysyncd 
    Restarting named
    Updating DNS system records
    
    Global DNS configuration in LDAP server is not empty
    The following configuration options override local settings in named.conf:
    
    API Version number was not sent, forward compatibility not guaranteed. Assuming server's API version, 2.245
      Global forwarders: 8.8.8.8, 1.1.1.1
      Forward policy: only
      Allow PTR sync: TRUE
      IPA DNS servers: dev-ipa04p.test3.lan
    
    Configuring SID generation 
      [1/7]: creating samba domain object
    Samba domain object already exists
      [2/7]: adding admin(group) SIDs
    Admin SID already set, nothing to do
    Admin group SID already set, nothing to do
      [3/7]: adding RID bases  
    RID bases already set, nothing to do
      [4/7]: updating Kerberos config
    'dns_lookup_kdc' already set to 'true', nothing to do.
      [5/7]: activating sidgen task
      [6/7]: restarting Directory Server to take MS PAC and LDAP plugins changes into account
      [7/7]: adding fallback group
    Fallback group already set, nothing to do
    Done.
    The ipa-replica-install command was successful
    ```
    </details><br>

### 3.2. Отключение анонимного доступа к LDAP-каталогу.
1. Процедура выполняется для каждого сервера, так как изменения производятся в нереплицируемой части LDAP-каталога 'cn=config':
    ```bash
    IPAREPLICA=$(hostname)
    DMPASS="DirectoryManagerPassword"
      
    ldapmodify -x -D "cn=Directory Manager" -w ${DMPASS} -h ${IPAREPLICA} -Z << EOF
    dn: cn=config
    changetype: modify
    replace: nsslapd-allow-anonymous-access
    nsslapd-allow-anonymous-access: rootdse
    EOF

    ipactl restart
    ```

## 4. Запуск ipa-healthcheck
1. Запуск ipa-healtcheck завершается с пятью замечаниями:
    ```bash
    sudo ipa-healthcheck
    ```

    <details>
    <summary>stdout...</summary>

    ```
    [
      {
        "source": "ipahealthcheck.ipa.certs",
        "check": "IPADogtagCertsMatchCheck",
        "result": "ERROR",
        "uuid": "9cbe0184-f345-4124-8c2a-984e7054a937",
        "when": "20220110183632Z",
        "duration": "0.493357",
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
        "uuid": "02ba1554-6554-4351-bae2-28a136895d97",
        "when": "20220110183632Z",
        "duration": "0.543730",
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
        "uuid": "5310274f-427c-4d4e-aa8e-bf10a8635102",
        "when": "20220110183632Z",
        "duration": "0.591251",
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
        "uuid": "a07b3794-ff38-4c8e-afa2-e206bff57579",
        "when": "20220110183632Z",
        "duration": "0.639254",
        "kw": {
          "key": "Server-Cert cert-pki-ca",
          "nickname": "Server-Cert cert-pki-ca",
          "dbdir": "/etc/pki/pki-tomcat/alias",
          "msg": "{nickname} certificate in NSS DB {dbdir} does not match entry in LDAP"
        }
      },
      {
        "source": "ipahealthcheck.ipa.dna",
        "check": "IPADNARangeCheck",
        "result": "WARNING",
        "uuid": "4cc6b198-2e6e-45c5-9477-6e6d3661c7e3",
        "when": "20220110183635Z",
        "duration": "0.306345",
        "kw": {
          "range_start": 0,
          "range_max": 0,
          "next_start": 0,
          "next_max": 0,
          "msg": "No DNA range defined. If no masters define a range then users and groups cannot be created."
        }
      }
    ]
    ```
    </details>

## 5. Исправление замечаний
### 5.1. 'No DNA range defined. If no masters define a range then users and groups cannot be created.'
Сразу после установки новых реплик на данное сообщение можно не реагировать. Диапазоны будут автоматически добавлены после первого же создания новой учётной записи или новой Posix-группы, прозведённого через новую же реплику. Об этом написано в `man ipa-replica-manage`.
> New IPA masters do not automatically get a DNA range assignment.  A  range  assignment  is done only when a user or POSIX group is added on that master.

Для проверки этого утверждения проведём опыт:

1. На одном из контроллеров домена проверяем текущие диапазоны:
    ```
    # ipa-replica-manage dnarange-show
    Directory Manager password:
    
    dev-ipa04p.test3.lan: 1443200004-1443399999
    dev-ipa05p.test3.lan: No range set
    dev-ipa06p.test3.lan: No range set
    ```

2. С помощью Web UI соответствующих контроллеров доменов добавляем новые пользовательские группы или УЗ.

3. Вновь проверяем текущие диапазоны и убеждаемся в том, что диапазоны были автоматически добавлены:
    ```
    # ipa-replica-manage dnarange-show
    Directory Manager password:
    
    dev-ipa04p.test3.lan: 1443200005-1443250499
    dev-ipa05p.test3.lan: 1443300501-1443399999
    dev-ipa06p.test3.lan: 1443250501-1443300499
    ```
4. Скриншот из Web UI с номерами групп:
    <div style="text-align:center;"><img src="/img/dobavleniye-replik-k-pervomu-serveru-freeipa/image2022-1-11_12-15-28.png"></div>
