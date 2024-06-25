---
title: "Настройка LDAP-аутентификации в Nexus Repository Manager"
date: 2020-09-12
weight: 10
description: >
  Описывается переключение Nexus Repository Manager на аутентификацию во FreeIPA с помощью LDAP-подключения.
tags:
  - Nexus Repository Manager
  - LDAP
  - FreeIPA
---

## Факты

- Исполняемое Java-ядро лежит в `/data/Nexus/nexus`.
- Изменяемая часть, включая репозитории, лежат в `/data/Nexus/sonatype-work`. Примерный размер каталога ~265GB. Для бэкапа достаточно сделать копию этого каталога.
- Автозапуск организован ручным созданием простой ссылкой `/etc.init.d/nexus → /data/Nexus/nexus/bin/nexus`.
- Настройка аккаунтов локальных пользователей и подключение по LDAP разнесены в меню.
- Нет необходимости делать бэкап Nexus'а, так как локальная база пользователей и получение пользователей из LDAP мирно сосуществуют рядом, не мешая друг другу.
- Пользователи подключающиеся через LDAP не заносятся в локальную базу, в отличии от GrayLog, например.

## Настройка аутентификации (Security – LDAP)

### Вкладка Connection

Самоподписанный сертификат контроллера домена добавляется в NXRM через кнопку "View certificate", где доступна операция "Add certificate to truststore". После чего, добавленный сертификат можно наблюдать в разделе Security – SSL Certificates.

![](/img/nastroika-ldap-autentifikacii-v-nexus-repository-manager/nxrm-ldap1.png)

### Вкладка User and group
Select template: Generic Ldap Server (делаем это в первый и единственный раз при настройке подключения).

User relative DN: cn=users

User subtree: не активируем, так как все пользователи во FreeIPA находятся в cn=users,cn=accounts,dc=example,dc=org, и нет нужды искать их во вложенных контейнерах.

Object class: inetOrgPerson

User filter: memberOf=cn=_nxrm,cn=groups,cn=accounts,dc=example,dc=org (проверяем наличие доступа к NXRM у  пользователя, через его наследуемое членство в центральной группе _nxrm, в каковую группу вложены подгруппы для авторизации – _nxrm_admins, _nxrm_ubic-users, etc.)

User ID attribute: uid

Real name attribute: cn

Email attribute: mail

Password attribute: оставляем пустым.

![](/img/nastroika-ldap-autentifikacii-v-nexus-repository-manager/nxrm-ldap2.png)

Ниже настраиваем отображение групп пользователей из IPA в соответствующие роли NXRM.

Map LDAP groups as roles: устанавливаем галку.

Group type: Static Groups

Group relative DN: cn=groups

Group subtree: оставляем без активации, так как все необходимые группы в IPA хранятся в cn=groups,cn=accounts,dc=example,dc=org.

Group object class: groupOfNames

Group ID attribute: cn

Group member attribute: member

Group member format: uid=${username},cn=users,cn=accounts,dc=example,dc=org

![](/img/nastroika-ldap-autentifikacii-v-nexus-repository-manager/nxrm-ldap3.png)

После чего появляется возможность добавить внешнюю роль, имя которой берётся из LDAP.

## Настройка авторизации
![](/img/nastroika-ldap-autentifikacii-v-nexus-repository-manager/nxrm-ldap-create-external-role.png)

## Включение LDAP
Добавляем в активные LDAP realms.

![](/img/nastroika-ldap-autentifikacii-v-nexus-repository-manager/nxrm-realms.png)

Предположим, что в локальной базе есть пользователь с совпадающим логином, как в IPA, но с разными паролями. Логика работы аутентификации будет следующей:

1. Сначала nxrm найдёт пользователя в локальной базе, но так как пароль не подойдёт, то будет просмотрены и другие Realms.
2. Наконец в LDAP Realms совпадут и логин и пароль. И пользователь будет аутентифицирован.
