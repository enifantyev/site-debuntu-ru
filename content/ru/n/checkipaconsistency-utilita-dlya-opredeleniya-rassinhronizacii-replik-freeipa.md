---
title: "Checkipaconsistency. Утилита для определения рассинхронизации реплик FreeIPA"
date: 2021-08-04
weight: 10
description: >
  Краткое описание утилиты используемой для наглядной демонстрации состояния синхронизации реплик FreeIPA.
tags:
  - FreeIPA
---

2021-08-04

https://github.com/peterpakos/checkipaconsistency

## Установка checkipaconsistency
Предыдущую версию 2.7.10 можно установить через pip `pip install --user checkipaconsistency`, тогда как последнюю версию придётся собирать:
```
$ git clone https://github.com/peterpakos/checkipaconsistency.git
$ cd checkipaconsistency
$ pip install --user -r requirements.txt
$ ./cipa
Initial config saved to ~/.config/checkipaconsistency - PLEASE EDIT IT!
IPA domain not set
```

## Настройка
Настройки находятся в файле `~/.config/checkipaconsistency`, где укажем пароль от 'cn=Directory Manager', что не безопасно. Напомню, что этот специальный аккаунт критически важен для безопасности домена FreeIPA.
```
cat << EOF > ~/.config/checkipaconsistency
[IPA]
domain = test.lan
hosts = dev-ipa01p, dev-ipa02p, dev-ipa03p
binddn = cn=Directory Manager
bindpw = DirectoryManagerPassWord
EOF
```

## Использование
```
$ ./cipa
+--------------------+--------------+--------------+--------------+-------+
| FreeIPA servers:   | dev-ipa01p   | dev-ipa02p   | dev-ipa03p   | STATE |
+--------------------+--------------+--------------+--------------+-------+
| Active Users       | 19           | 19           | 19           | OK    |
| Stage Users        | 0            | 0            | 0            | OK    |
| Preserved Users    | 1            | 1            | 1            | OK    |
| Hosts              | 16           | 16           | 16           | OK    |
| Services           | 65           | 65           | 65           | OK    |
| User Groups        | 33           | 33           | 33           | OK    |
| Host Groups        | 1            | 1            | 1            | OK    |
| Netgroups          | 0            | 0            | 0            | OK    |
| HBAC Rules         | 1            | 1            | 1            | OK    |
| SUDO Rules         | 0            | 0            | 0            | OK    |
| DNS Zones          | 2            | 2            | 2            | OK    |
| Certificates       | 108          | 108          | 108          | OK    |
| LDAP Conflicts     | 0            | 0            | 0            | OK    |
| Ghost Replicas     | 0            | 0            | 0            | OK    |
| Anonymous BIND     | ON           | ON           | ON           | OK    |
| Microsoft ADTrust  | False        | False        | False        | OK    |
| Replication Status | dev-ipa03p 0 | dev-ipa03p 0 | dev-ipa02p 0 | OK    |
|                    | dev-ipa02p 0 | dev-ipa01p 0 | dev-ipa01p 0 |       |
+--------------------+--------------+--------------+--------------+-------+
```
