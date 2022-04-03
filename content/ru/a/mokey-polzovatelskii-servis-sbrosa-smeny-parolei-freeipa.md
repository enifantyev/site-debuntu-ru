---
title: "Mokey - пользовательский сервис сброса/смены паролей FreeIPA"
date: 2021-05-27
weight: 10
description: >
  Сервис Mokey предназачен для самообслуживания пользователей FreeIPA домена, с помощью которого предоставляется возможность смены пароля или сброса забытого пароля, с отправкой одноразовой ссылки на зарегистрированную в IPA почту пользователя.
tags:
  - Mokey
  - FreeIPA
---

2021-05-27

## Введение
Сервис Mokey предназачен для самообслуживания пользователей FreeIPA домена, с помощью которого предоставляется возможность смены пароля или сброса забытого пароля, с отправкой одноразовой ссылки на зарегистрированную в IPA почту пользователя. Замечу, что **mokey, не работает с учётными записями, состоящими во встроенной IPA-группе 'admins'**, так как роль helpdesk не сможет изменить пароль таких УЗ. Вообще, рекомендуется использовать отдельную УЗ, участницу группы 'admins', для администрирования FreeIPA.

## Установка mokey
Установка mokey банальна и не вызывает интереса. Скачиваем последний релиз rpm-пакет из [github.com/ubccr/mokey](https://github.com/ubccr/mokey) и устанавливаем обычным способом.

## Установка MariaDB и создание базы mokey
Устанавливаем mariadb-server:
```
yum install mariadb-server
systemctl enable --now mariadb-server
systemctl enable --now mariadb
```

Далее стандартный вызов `mysql_secure_installation` с придумыванием хорошего пароля root'а для mariadb:
```
mysql_secure_installation
  Root Password:
  Remove anonymous users? [Y/n] y
  Disallow root login remotely? [Y/n] y
  Remove test database and access to it? [Y/n] y
  Reload privilege tables now? [Y/n] y
```

Далее создание базы данных mokey с логином/паролем и импорт схемы таблиц:
```
mysql -u root -p
  mysql> create database mokey;
  mysql> grant all on mokey.* to [user]@localhost identified by '[pass]'
  mysql> exit
mysql -u root -p mokey < /usr/share/mokey/ddl/schema.sql
```

## Создание сервиса mokey и получение keytab'ов
Выполняем создание сервиса и **создание keytab'а** на первом контроллере домена с правами обычного пользователя, но с принципалом, имеющим администраторские права во FreeIPA (здесь с принципалом admin):
```
kinit admin

# Добавляем сервис mokey
ipa service-add mokey/ipa-selfservice.test.lan --skip-host-check --force

# Даём юзеру право на создание keytab'а
ipa service-allow-create-keytab mokey/ipa-selfservice.test.lan --users=admin

# Создаём и сохраняем keytab в каталоге
ipa-getkeytab -p mokey/ipa-selfservice.test.lan -k /etc/mokey/keytab/mokeyapp.keytab

# Отнимаем у пользователя право на создание keytab'а для этого сервиса
ipa service-disallow-create-keytab mokey/ipa-selfservice.test.lan --users=admin

chmod 640 /etc/mokey/keytab/mokeyapp.keytab
chgrp mokey /etc/mokey/keytab/mokeyapp.keytab
```

На других контроллерах домена выполняем **получение keytab'а** (здесь с принципалом admin):
```
kinit admin

# Даём юзеру право на получение keytab'а
ipa service-allow-retrieve-keytab mokey/ipa-selfservice.test.lan --users=admin

# Создаём и сохраняем keytab в каталоге
ipa-getkeytab -p mokey/ipa-selfservice.test.lan -k /etc/mokey/keytab/mokeyapp.keytab -r

# Отнимаем у пользователя право на получение keytab'а для этого сервиса
ipa service-disallow-retrieve-keytab mokey/ipa-selfservice.test.lan --users=admin

chmod 640 /etc/mokey/keytab/mokeyapp.keytab
chgrp mokey /etc/mokey/keytab/mokeyapp.keytab
```

## Создание HTTP-сервиса для mokey и получение сертификатов
На первом контроллере домена выполняем создание сервиса HTTP и запрос на получение сертификата (здесь с принципалом admin):
```
kinit admin

# Создаём сервис без привязки к хосту
ipa service-add HTTP/ipa-selfservice.test.lan --skip-host-check --force

# Даём управление сервисом для всех контроллеров домена
ipa service-add-host HTTP/ipa-selfservice.test.lan \
    --host prod-ipa01p.test.lan \
    --host prod-ipa02p.test.lan \
    --host prod-ipa03p.test.lan \

# Создаём ключи и запрашиваем сертификат с одновременной постановкой на отслеживание
ipa-getcert request -w -K "HTTP/$(hostname)" \
                          -k "/etc/mokey/mokey_ssl.key" \
                          -f "/etc/mokey/mokey_ssl.crt" \
                          -D $(hostname) \
                          -D "ipa-selfservice.test.lan" \
                          -C "/bin/bash -c '/bin/chgrp mokey /etc/mokey/mokey_ssl.* && /bin/chmod 640 /etc/mokey/mokey_ssl.*'"
```

На других контроллерах домена выполняем только запрос на получение сертификата (здесь с принципалом admin):
```
kinit admin

ipa-getcert request -w -K "HTTP/$(hostname)" \
                          -k "/etc/mokey/mokey_ssl.key" \
                          -f "/etc/mokey/mokey_ssl.crt" \
                          -D $(hostname) \
                          -D "ipa-selfservice.test.lan" \
                          -C "/bin/bash -c '/bin/chgrp mokey /etc/mokey/mokey_ssl.* && /bin/chmod 640 /etc/mokey/mokey_ssl.*'"
```

## Редактируем /etc/mokey/mokey.yml
`vi /etc/mokey/mokey.yaml`:
```
dsn: "mokey:[Соответствующий Пароль]@/mokey?parseTime=true"
driver: "mysql"
port: 8081
bind: "127.0.0.1"
min_passwd_len: 12
min_passwd_classes: 2
auth_key: <!-- здесь строка из 64 случайных символа из словаря 0123456789abcdef -->
enc_key: <!-- здесь строка из 32 случайных символа из словаря 0123456789abcdef -->
templates: /usr/share/mokey/templates
ipahost: "prod-ipa01p.test.lan"
keytab: "/etc/mokey/keytab/mokeyapp.keytab"
ktuser: "mokey/ipa-selfservice.test.lan"
rate_limit: false

smtp_host: "10.0.0.1"
smtp_port: 25
smtp_tls: "off"

email_from: "bigdata-servers@test.lan"
email_sig: "System Administrators && Co"

#------------------------------------------------------------------------------
# Base URL of mokey server. Used for links in emails
#------------------------------------------------------------------------------
email_link_base: "https://ipa-selfservice.test.lan:8095"
```

Стартуем mokey:
```
systemctl start mokey
```

В случае проблем, для отладки, запускаем `/usr/bin/mokey -c /etc/mokey/mokey.yaml -d server`.

## Донастройка httpd-сервера из состава FreeIPA

```
Listen 8085 https
SSLSessionCache         shmcb:/run/httpd/sslcache2(512000)
SSLSessionCacheTimeout  300
SSLRandomSeed startup file:/dev/urandom  256
SSLRandomSeed connect builtin
SSLCryptoDevice builtin
<VirtualHost _default_:8085>
  ErrorLog logs/error_ss_log
  TransferLog logs/access_ss_log
  LogLevel info
  <Location "/">
    ProxyPass "http://127.0.0.1:8081/"
  </Location>
  SSLEngine on
  SSLProtocol +TLSv1 +TLSv1.1 +TLSv1.2
  SSLHonorCipherOrder on
  SSLCipherSuite kEECDH:kRSA:kEDH:!aDSS:!EXP:!3DES:!DES:!RC4:!RC2:!IDEA:!SEED:!eNULL:!aNULL:!MD5:!SSLv2:!ADH
  SSLProxyCipherSuite kEECDH:kRSA:kEDH:!aDSS:!EXP:!3DES:!DES:!RC4:!RC2:!IDEA:!SEED:!eNULL:!aNULL:!MD5:!SSLv2:!ADH
  SSLCertificateFile /etc/mokey/mokey_ssl.crt
  SSLCertificateKeyFile /etc/mokey/mokey_ssl.key
  SSLCACertificateFile /etc/ipa/ca.crt
  SSLVerifyDepth 5
  BrowserMatch "MSIE [2-5]" \
         nokeepalive ssl-unclean-shutdown \
         downgrade-1.0 force-response-1.0
  CustomLog logs/ssl_request_log \
          "%t %h %{SSL_PROTOCOL}x %{SSL_CIPHER}x \"%r\" %b"
</VirtualHost>
```
Перезапускаем Apache:
```
systemctl restart httpd
```

## Проверка
Так как ранее мы создали TLS-сертификаты с SAN для имени ipa-selfservice.test.lan, то возможно добавить соответствующую запись в DNS и указать пользователям использовать URL https://ipa-selfservice.test.lan:8085/.

Через браузер проверяем наличие морд на URL'ах:
- https://prod-ipa01p.test.lan:8085
- https://prod-ipa02p.test.lan:8085
- https://prod-ipa03p.test.lan:8085
- https://ipa-selfservice.test.lan:8085/
