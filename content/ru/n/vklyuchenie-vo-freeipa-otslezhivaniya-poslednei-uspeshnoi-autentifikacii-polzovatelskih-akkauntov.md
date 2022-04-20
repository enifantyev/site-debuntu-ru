---
title: "Включение во FreeIPA отслеживания последней успешной аутентификации пользовательских аккаунтов"
date: 2021-05-07
weight: 10
description: >
  По умолчанию, во FreeIPA, для уменьшения нагрузки на серверы, отключено отслеживание последней успешной аутентификации пользовательских аккаунтов. Здесь описано, как выключить и вновь включить плагин, отвечающие за эту опцию.
tags:
  - FreeIPA
---

2021-05-07

Использованные источники:
- [Enabling Tracking of Last Successful Kerberos Authentication](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/linux_domain_identity_authentication_and_policy_guide/enabling-tracking-of-last-successful-kerberos-authentication)

## Отключение плагина через GUI
IPA Server&nbsp;&mdash;&nbsp;Configuration&nbsp;&mdash;&nbsp;Password plugin features:<br>
☑ KDC:Disable Last Success

## Отключение плагина через CLI
На каждом сервере FreeIPA смотрим текущие применяемые password-плагины:
```
# ipa config-show | grep "Password plugin features"
  Password plugin features: AllowNThash, KDC:Disable Last Success
```

Отключаем плагин блокирующий трэкинг последней успешной аутентификации, оставляя прочие плагины включёнными:
```
# ipa config-mod --ipaconfigstring='AllowNThash'
```

Если нагрузка на сервер превысила его ресурсы, то вновь отключаем трэкинг последней успешной аутентификации:
```
# ipa config-mod --ipaconfigstring='AllowNThash' --ipaconfigstring='KDC:Disable Last Success'
```

Проверю при первой возможности, обязательно ли надо перезапускать одну или все реплики, так как после выполнения команд `ipa config-mod` изменения немедленно реплицируются на все реплики. Но, сейчас, следуя инструкциям, выполняем:
```
# ipactl restart
```

Посмотреть дата/время последней успешной аутентификации:
```
$ ipa user-status admin
----------------------
Account disabled: False
----------------------
  Server: prod-ipa01.example.org
  Failed logins: 1
  Last successful authentication: 20210505154121Z
  Last failed authentication: 20210506113029Z
  Time now: 2021-05-07T07:03:56Z

  Server: prod-ipa03.example.org
  Failed logins: 4
  Last successful authentication: N/A
  Last failed authentication: 20210210175128Z
  Time now: 2021-05-07T07:03:56Z

  Server: prod-ipa02.example.org
  Failed logins: 1
  Last successful authentication: N/A
  Last failed authentication: 20210329132149Z
  Time now: 2021-05-07T07:03:56Z
----------------------------
Number of entries returned 3
----------------------------
```
