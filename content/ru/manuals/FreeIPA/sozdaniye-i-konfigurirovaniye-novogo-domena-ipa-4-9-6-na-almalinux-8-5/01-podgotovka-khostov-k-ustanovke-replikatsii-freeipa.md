---
title: "01. Подготовка хостов к установке реплик FreeIPA"
date: 2021-12-24
weight: 11
description: >
  Подготовительные работы перед установкой FreeIPA.
tags:
  - FreeIPA
  - FreeIPA 4.9.6
  - AlmaLinux 8.5
slug: podgotovka-khostov-k-ustanovke-replikatsii-freeipa
---

2021-12-24

## 1. Изменение сетевых настроек и имени хоста:

1.1. Исправляем `/etc/hosts`:
```bash
cat << EOF | sudo tee /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
EOF
```

1.2. Обнуляем `/etc/sysconfig/network`:
```bash
echo "" | sudo tee /etc/sysconfig/network
```

1.3. Учитывая, что обращение за rpm-пакетами к Nexus Repository Manager производится по ip-адресу, то обнуляем `/etc/resolv.conf`:
```bash
echo "" | sudo tee /etc/resolv.conf
```

1.4. Задаём имя машины в нижнем регистре с указанием имени будущего домена:
```bash
DOMAINNAME="test3.lan"
sudo hostnamectl set-hostname $(echo $(hostname -s) | tr '[:upper:]' '[:lower:]').${DOMAINNAME}
```



## 2. Добавление необходимых пакетов

2.1. Смотрим список доступных потоков:
```bash
[user@host ~]$ sudo dnf module list idm
Last metadata expiration check: 0:06:45 ago on Чт 27 янв 2022 12:04:24.
AlmaLinux 8 - AppStream
Name  Stream      Profiles                                  Summary                                                       
idm   DL1         adtrust, client, common [d], dns, server  The Red Hat Enterprise Linux Identity Management system module
idm   client [d]  common [d]                                RHEL IdM long term support client module                      

Hint: [d]efault, [e]nabled, [x]disabled, [i]nstalled
```

2.2. Включаем модуль, в котором находится всё необходимое для установки FreeIPA-сервера:
```bash
sudo dnf -y module enable idm:DL1
```
<details>

```
Dependencies resolved.
==========================================================================================================================
 Package                      Architecture                Version                      Repository                    Size
==========================================================================================================================
Enabling module streams:
 389-ds                                                   1.4
 httpd                                                    2.4
 idm                                                      DL1                                                            
 pki-core                                                 10.6                                                           
 pki-deps                                                 10.6                                                           

Transaction Summary
==========================================================================================================================

Complete!
```

</details><br/>

> В случае возникновения ошибки:<br>
    *Error: It is not possible to switch enabled streams of a module unless explicitly enabled via configuration option module_
    stream_switch.
    It is recommended to rather remove all installed content from the module, and reset the module using 'dnf module reset <mo
    dule_name>' command. After you reset the module, you can install the other stream.*
    <br>
    необходимо выполнить команду:<br>
    `sudo dnf -y module reset idm:DL1 && sudo dnf -y module enable idm:DL1`

2.3. Переключаем получение необходимых для установки FreeIPA пакетов на доставку из модуля 'idm:DL1', одновременно синхронизируя уже полученные пакеты:
```bash
sudo dnf -y distro-sync
```
<details>

```
Dependencies resolved.
==========================================================================================================================
 Package                       Architecture     Version                                         Repository           Size
==========================================================================================================================
Upgrading:
 ipa-client                    x86_64           4.9.6-10.module_el8.5.0+2603+92118e57           appstream           280 k
 ipa-client-common             noarch           4.9.6-10.module_el8.5.0+2603+92118e57           appstream           183 k
 ipa-common                    noarch           4.9.6-10.module_el8.5.0+2603+92118e57           appstream           795 k
 ipa-selinux                   noarch           4.9.6-10.module_el8.5.0+2603+92118e57           appstream           175 k
 python3-ipaclient             noarch           4.9.6-10.module_el8.5.0+2603+92118e57           appstream           687 k
 python3-ipalib                noarch           4.9.6-10.module_el8.5.0+2603+92118e57           appstream           754 k
 python3-jwcrypto              noarch           0.5.0-1.module_el8.5.0+2603+92118e57            appstream            64 k
 python3-pyusb                 noarch           1.0.0-9.module_el8.5.0+2603+92118e57            appstream            87 k
 python3-qrcode-core           noarch           5.1-12.module_el8.5.0+2603+92118e57             appstream            45 k
 python3-yubico                noarch           1.3.2-9.module_el8.5.0+2603+92118e57            appstream            62 k

Transaction Summary
==========================================================================================================================
Upgrade  10 Packages

Total download size: 3.1 M
```

</details><br>

2.4. Добавляем в систему пакеты для последующей установки FreeIPA-сервера со встроенным DNS-резольвером:
```bash
sudo dnf -y module install idm:DL1/dns
```

<details>

```
Dependencies resolved.
==========================================================================================================================
 Package                              Arch        Version                                            Repository      Size
==========================================================================================================================
Installing group/module packages:
 ipa-healthcheck                      noarch      0.7-6.module_el8.5.0+2603+92118e57                 appstream      101 k
 ipa-healthcheck-core                 noarch      0.7-6.module_el8.5.0+2603+92118e57                 appstream       51 k
 ipa-server                           x86_64      4.9.6-10.module_el8.5.0+2603+92118e57              appstream      528 k
 ipa-server-dns                       noarch      4.9.6-10.module_el8.5.0+2603+92118e57              appstream      191 k
Installing dependencies:
 389-ds-base                          x86_64      1.4.3.23-10.module_el8.5.0+2538+47000435           appstream      2.5 M
 389-ds-base-libs                     x86_64      1.4.3.23-10.module_el8.5.0+2538+47000435           appstream      1.4 M
 almalinux-logos-httpd                noarch      84.5-1.el8                                         appstream       29 k
 almalinux-logos-ipa                  noarch      84.5-1.el8                                         appstream       35 k
 alsa-lib                             x86_64      1.2.5-4.el8                                        appstream      488 k
 ant                                  noarch      1.10.5-1.module_el8.0.0+6031+d80e135e              appstream      193 k
 ant-lib                              noarch      1.10.5-1.module_el8.0.0+6031+d80e135e              appstream      2.0 M
 apache-commons-cli                   noarch      1.4-4.module_el8.0.0+6035+97aff910                 appstream       74 k
 apache-commons-codec                 noarch      1.11-3.module_el8.0.0+6035+97aff910                appstream      288 k
 apache-commons-io                    noarch      1:2.6-3.module_el8.0.0+6035+97aff910               appstream      223 k
 apache-commons-lang3                 noarch      3.7-3.module_el8.0.0+6035+97aff910                 appstream      482 k
 apache-commons-logging               noarch      1.2-13.module_el8.0.0+6035+97aff910                appstream       85 k
 apache-commons-net                   noarch      3.6-3.module_el8.5.0+2577+9e95fe00                 appstream      296 k
 apr                                  x86_64      1.6.3-12.el8                                       appstream      128 k
 apr-util                             x86_64      1.6.1-6.el8                                        appstream      105 k
 atk                                  x86_64      2.28.1-1.el8                                       appstream      271 k
 bea-stax-api                         noarch      1.2.0-16.module_el8.5.0+2577+9e95fe00              appstream       36 k
 bind                                 x86_64      32:9.11.26-6.el8                                   appstream      2.1 M
 bind-dyndb-ldap                      x86_64      11.6-2.module_el8.5.0+2603+92118e57                appstream      127 k
 bind-pkcs11                          x86_64      32:9.11.26-6.el8                                   appstream      397 k
 bind-pkcs11-libs                     x86_64      32:9.11.26-6.el8                                   appstream      1.1 M
 bind-pkcs11-utils                    x86_64      32:9.11.26-6.el8                                   appstream      259 k
 cairo                                x86_64      1.15.12-3.el8                                      appstream      721 k
 copy-jdk-configs                     noarch      4.0-2.el8                                          appstream       29 k
 custodia                             noarch      0.6.0-3.module_el8.5.0+2603+92118e57               appstream       32 k
 cyrus-sasl-md5                       x86_64      2.1.27-5.el8                                       baseos          65 k
 cyrus-sasl-plain                     x86_64      2.1.27-5.el8                                       baseos          47 k
 fontawesome-fonts                    noarch      4.7.0-4.el8                                        appstream      203 k
 fontconfig                           x86_64      2.13.1-4.el8                                       baseos         273 k
 fontpackages-filesystem              noarch      1.44-22.el8                                        baseos          16 k
 fribidi                              x86_64      1.0.4-8.el8                                        appstream       89 k
 gdk-pixbuf2                          x86_64      2.36.12-5.el8                                      baseos         467 k
 gdk-pixbuf2-modules                  x86_64      2.36.12-5.el8                                      appstream      109 k
 giflib                               x86_64      5.1.4-3.el8                                        appstream       51 k
 glassfish-fastinfoset                noarch      1.2.13-9.module_el8.5.0+2577+9e95fe00              appstream      353 k
 glassfish-jaxb-api                   noarch      2.2.12-8.module_el8.5.0+2577+9e95fe00              appstream      100 k
 glassfish-jaxb-core                  noarch      2.2.11-11.module_el8.5.0+2577+9e95fe00             appstream      157 k
 glassfish-jaxb-runtime               noarch      2.2.11-11.module_el8.5.0+2577+9e95fe00             appstream      935 k
 glassfish-jaxb-txw2                  noarch      2.2.11-11.module_el8.5.0+2577+9e95fe00             appstream       89 k
 graphite2                            x86_64      1.3.10-10.el8                                      appstream      121 k
 gtk-update-icon-cache                x86_64      3.22.30-8.el8                                      appstream       31 k
 harfbuzz                             x86_64      1.7.5-3.el8                                        appstream      295 k
 hicolor-icon-theme                   noarch      0.17-2.el8                                         appstream       48 k
 httpcomponents-client                noarch      4.5.5-4.module_el8.0.0+6035+97aff910               appstream      718 k
 httpcomponents-core                  noarch      4.4.10-3.module_el8.0.0+6035+97aff910              appstream      637 k
 httpd                                x86_64      2.4.37-43.module_el8.5.0+2597+c4b14997.alma        appstream      1.4 M
 httpd-filesystem                     noarch      2.4.37-43.module_el8.5.0+2597+c4b14997.alma        appstream       39 k
 httpd-tools                          x86_64      2.4.37-43.module_el8.5.0+2597+c4b14997.alma        appstream      106 k
 ipa-server-common                    noarch      4.9.6-10.module_el8.5.0+2603+92118e57              appstream      611 k
 istack-commons-runtime               noarch      2.21-9.el8+7                                       appstream       43 k
 jackson-annotations                  noarch      2.10.0-1.module_el8.5.0+2577+9e95fe00              appstream       70 k
 jackson-core                         noarch      2.10.0-1.module_el8.5.0+2577+9e95fe00              appstream      344 k
 jackson-databind                     noarch      2.10.0-1.module_el8.5.0+2577+9e95fe00              appstream      1.3 M
 jackson-jaxrs-json-provider          noarch      2.9.9-1.module_el8.5.0+2577+9e95fe00               appstream       23 k
 jackson-jaxrs-providers              noarch      2.9.9-1.module_el8.5.0+2577+9e95fe00               appstream       44 k
 jackson-module-jaxb-annotations      noarch      2.7.6-4.module_el8.5.0+2577+9e95fe00               appstream       45 k
 jasper-libs                          x86_64      2.0.14-5.el8                                       appstream      166 k
 java-1.8.0-openjdk                   x86_64      1:1.8.0.312.b07-2.el8_5                            appstream      340 k
 java-1.8.0-openjdk-devel             x86_64      1:1.8.0.312.b07-2.el8_5                            appstream      9.9 M
 java-1.8.0-openjdk-headless          x86_64      1:1.8.0.312.b07-2.el8_5                            appstream       34 M
 javapackages-filesystem              noarch      5.3.0-2.module_el8.0.0+6004+2fc32706               appstream       30 k
 javapackages-tools                   noarch      5.3.0-2.module_el8.0.0+6004+2fc32706               appstream       44 k
 jbigkit-libs                         x86_64      2.1-14.el8                                         appstream       54 k
 jboss-annotations-1.2-api            noarch      1.0.0-4.el8                                        appstream       40 k
 jboss-jaxrs-2.0-api                  noarch      1.0.0-6.el8                                        appstream      113 k
 jboss-logging                        noarch      3.3.0-5.el8                                        appstream       71 k
 jboss-logging-tools                  noarch      2.0.1-6.el8                                        appstream      174 k
 jdeparser                            noarch      2.0.0-5.el8                                        appstream      217 k
 jss                                  x86_64      4.9.1-1.module_el8.5.0+2592+8eb38ccc               appstream      1.2 M
 krb5-pkinit                          x86_64      1.18.2-14.el8                                      baseos         174 k
 krb5-server                          x86_64      1.18.2-14.el8                                      baseos         1.1 M
 ldapjdk                              noarch      4.23.0-1.module_el8.5.0+2592+8eb38ccc              appstream      322 k
 ldns                                 x86_64      1.7.0-21.el8                                       appstream      165 k
 libX11                               x86_64      1.6.8-5.el8                                        appstream      610 k
 libX11-common                        noarch      1.6.8-5.el8                                        appstream      157 k
 libXau                               x86_64      1.0.9-3.el8                                        appstream       37 k
 libXcomposite                        x86_64      0.4.4-14.el8                                       appstream       28 k
 libXcursor                           x86_64      1.1.15-3.el8                                       appstream       36 k
 libXdamage                           x86_64      1.1.4-14.el8                                       appstream       26 k
 libXext                              x86_64      1.3.4-1.el8                                        appstream       45 k
 libXfixes                            x86_64      5.0.3-7.el8                                        appstream       25 k
 libXft                               x86_64      2.3.3-1.el8                                        appstream       66 k
 libXi                                x86_64      1.7.10-1.el8                                       appstream       48 k
 libXinerama                          x86_64      1.1.4-1.el8                                        appstream       15 k
 libXrandr                            x86_64      1.5.2-1.el8                                        appstream       33 k
 libXrender                           x86_64      0.9.10-7.el8                                       appstream       33 k
 libXtst                              x86_64      1.2.3-7.el8                                        appstream       21 k
 libdatrie                            x86_64      0.2.9-7.el8                                        appstream       33 k
 libfontenc                           x86_64      1.1.3-8.el8                                        appstream       37 k
 libjpeg-turbo                        x86_64      1.5.3-12.el8                                       appstream      156 k
 libthai                              x86_64      0.1.27-2.el8                                       appstream      203 k
 libtiff                              x86_64      4.0.9-20.el8                                       appstream      187 k
 libxcb                               x86_64      1.13.1-1.el8                                       appstream      231 k
 lua                                  x86_64      5.3.4-12.el8                                       appstream      191 k
 mailcap                              noarch      2.1.48-3.el8                                       baseos          39 k
 mod_auth_gssapi                      x86_64      1.6.1-7.1.el8                                      appstream       84 k
 mod_http2                            x86_64      1.15.7-3.module_el8.5.0+2576+c8e0c271              appstream      153 k
 mod_lookup_identity                  x86_64      1.0.0-4.el8                                        appstream       30 k
 mod_session                          x86_64      2.4.37-43.module_el8.5.0+2597+c4b14997.alma        appstream       73 k
 mod_ssl                              x86_64      1:2.4.37-43.module_el8.5.0+2597+c4b14997.alma      appstream      135 k
 open-sans-fonts                      noarch      1.10-6.el8                                         appstream      482 k
 opencryptoki                         x86_64      3.16.0-5.el8                                       baseos         136 k
 opencryptoki-icsftok                 x86_64      3.16.0-5.el8                                       baseos         288 k
 opencryptoki-libs                    x86_64      3.16.0-5.el8                                       baseos          59 k
 opendnssec                           x86_64      2.1.7-1.module_el8.5.0+2603+92118e57               appstream      472 k
 openldap-clients                     x86_64      2.4.46-18.el8                                      baseos         201 k
 openssl-perl                         x86_64      1:1.1.1k-4.el8                                     baseos          80 k
 pango                                x86_64      1.42.4-8.el8                                       appstream      296 k
 perl-Algorithm-Diff                  noarch      1.1903-9.el8                                       baseos          51 k
 perl-Archive-Tar                     noarch      2.30-1.el8                                         baseos          79 k
 perl-Compress-Raw-Bzip2              x86_64      2.081-1.el8                                        baseos          40 k
 perl-Compress-Raw-Zlib               x86_64      2.081-1.el8                                        baseos          68 k
 perl-DB_File                         x86_64      1.842-1.el8                                        appstream       83 k
 perl-IO-Compress                     noarch      2.081-1.el8                                        baseos         258 k
 perl-IO-Zlib                         noarch      1:1.10-420.el8                                     baseos          79 k
 perl-Text-Diff                       noarch      1.45-2.el8                                         baseos          45 k
 pixman                               x86_64      0.38.4-1.el8                                       appstream      257 k
 pki-acme                             noarch      10.11.2-2.module_el8.5.0+2592+8eb38ccc             appstream      1.0 M
 pki-base                             noarch      10.11.2-2.module_el8.5.0+2592+8eb38ccc             appstream      294 k
 pki-base-java                        noarch      10.11.2-2.module_el8.5.0+2592+8eb38ccc             appstream      666 k
 pki-ca                               noarch      10.11.2-2.module_el8.5.0+2592+8eb38ccc             appstream      1.3 M
 pki-kra                              noarch      10.11.2-2.module_el8.5.0+2592+8eb38ccc             appstream      288 k
 pki-server                           noarch      10.11.2-2.module_el8.5.0+2592+8eb38ccc             appstream      2.6 M
 pki-servlet-4.0-api                  noarch      1:9.0.30-3.module_el8.5.0+2577+9e95fe00            appstream      281 k
 pki-servlet-engine                   noarch      1:9.0.30-3.module_el8.5.0+2577+9e95fe00            appstream      5.4 M
 pki-symkey                           x86_64      10.11.2-2.module_el8.5.0+2592+8eb38ccc             appstream       56 k
 pki-tools                            x86_64      10.11.2-2.module_el8.5.0+2592+8eb38ccc             appstream      794 k
 publicsuffix-list                    noarch      20180723-1.el8                                     baseos          79 k
 python3-argcomplete                  noarch      1.9.3-6.el8                                        appstream       60 k
 python3-asn1crypto                   noarch      0.24.0-3.el8                                       baseos         181 k
 python3-custodia                     noarch      0.6.0-3.module_el8.5.0+2603+92118e57               appstream      120 k
 python3-distro                       noarch      1.4.0-2.module_el8.5.0+2569+5c5719bc               appstream       36 k
 python3-ipaserver                    noarch      4.9.6-10.module_el8.5.0+2603+92118e57              appstream      1.6 M
 python3-kdcproxy                     noarch      0.4-5.module_el8.5.0+2603+92118e57                 appstream       38 k
 python3-lib389                       noarch      1.4.3.23-10.module_el8.5.0+2538+47000435           appstream      887 k
 python3-lxml                         x86_64      4.2.3-3.el8                                        appstream      1.5 M
 python3-mod_wsgi                     x86_64      4.6.4-4.el8                                        appstream      2.5 M
 python3-pip                          noarch      9.0.3-20.el8                                       appstream       19 k
 python3-pki                          noarch      10.11.2-2.module_el8.5.0+2592+8eb38ccc             appstream      164 k
 python3-psutil                       x86_64      5.4.3-11.el8                                       appstream      372 k
 python3-setuptools                   noarch      39.2.0-6.el8                                       baseos         162 k
 python3-systemd                      x86_64      234-8.el8                                          appstream       81 k
 python3-webencodings                 noarch      0.5.1-6.el8                                        appstream       27 k
 python36                             x86_64      3.6.8-38.module_el8.5.0+2569+5c5719bc              appstream       18 k
 relaxngDatatype                      noarch      2011.1-7.module_el8.5.0+2577+9e95fe00              appstream       26 k
 resteasy                             noarch      3.0.26-6.module_el8.5.0+2577+9e95fe00              appstream      1.1 M
 slapi-nis                            x86_64      0.56.6-4.module_el8.5.0+2603+92118e57              appstream      157 k
 slf4j                                noarch      1.7.25-4.module_el8.5.0+2577+9e95fe00              appstream       76 k
 slf4j-jdk14                          noarch      1.7.25-4.module_el8.5.0+2577+9e95fe00              appstream       24 k
 softhsm                              x86_64      2.6.0-5.module_el8.5.0+2603+92118e57               appstream      430 k
 sqlite                               x86_64      3.26.0-15.el8                                      baseos         667 k
 sscg                                 x86_64      2.3.3-14.el8                                       appstream       49 k
 stax-ex                              noarch      1.7.7-8.module_el8.5.0+2577+9e95fe00               appstream       54 k
 tomcatjss                            noarch      7.7.0-1.module_el8.5.0+2592+8eb38ccc               appstream       38 k
 ttmkfdir                             x86_64      3.0.9-54.el8                                       appstream       62 k
 tzdata-java                          noarch      2021e-1.el8                                        appstream      190 k
 words                                noarch      3.0-28.el8                                         baseos         1.4 M
 xalan-j2                             noarch      2.7.1-38.module_el8.5.0+2577+9e95fe00              appstream      1.9 M
 xerces-j2                            noarch      2.11.0-34.module_el8.5.0+2577+9e95fe00             appstream      1.2 M
 xml-commons-apis                     noarch      1.4.01-25.module_el8.5.0+2577+9e95fe00             appstream      233 k
 xml-commons-resolver                 noarch      1.2-26.module_el8.5.0+2577+9e95fe00                appstream      114 k
 xmlstreambuffer                      noarch      1.5.4-8.module_el8.5.0+2577+9e95fe00               appstream       86 k
 xorg-x11-font-utils                  x86_64      1:7.5-41.el8                                       appstream      102 k
 xorg-x11-fonts-Type1                 noarch      7.5-19.el8                                         appstream      522 k
 xsom                                 noarch      0-19.20110809svn.module_el8.5.0+2577+9e95fe00      appstream      398 k
Installing weak dependencies:
 apr-util-bdb                         x86_64      1.6.1-6.el8                                        appstream       24 k
 apr-util-openssl                     x86_64      1.6.1-6.el8                                        appstream       27 k
 bash-completion                      noarch      1:2.7-5.el8                                        baseos         273 k
 gtk2                                 x86_64      2.24.32-5.el8                                      appstream      3.4 M
 python3-beautifulsoup4               noarch      4.6.3-2.el8.1                                      epel           185 k
 python3-cssselect                    noarch      0.9.2-10.el8                                       epel            40 k
 python3-html5lib                     noarch      1:0.999999999-6.el8                                appstream      214 k
 python3-nss                          x86_64      1.0.1-10.module_el8.5.0+2577+9e95fe00.alma         appstream      285 k
Installing module profiles:
 idm/dns

Transaction Summary
==========================================================================================================================
Install  177 Packages

Total download size: 110 M
Installed size: 319 M
...
  Installing       : sqlite-3.26.0-15.el8.x86_64                                                                  137/177
  Running scriptlet: opendnssec-2.1.7-1.module_el8.5.0+2603+92118e57.x86_64                                       138/177
  Installing       : opendnssec-2.1.7-1.module_el8.5.0+2603+92118e57.x86_64                                       138/177
  Running scriptlet: opendnssec-2.1.7-1.module_el8.5.0+2603+92118e57.x86_64                                       138/177
Slot 0 has a free/uninitialized token.
The token has been initialized and is reassigned to slot 1520725004
*WARNING* This will erase all data in the database; are you sure? [y/N] Database setup successfully.
...
```

</details>
