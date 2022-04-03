---
title: "Создание system account для подключения LDAP-клиента к FreeIPA"
date: 2020-11-26
weight: 10
description: >
  Работа со специальным типом FreeIPA-аккаунтов, system accounts, с помощью скрипта freeipa-sam.
tags:
  - FreeIPA
  - LDAP
---

2020-11-26

## Введение
Для подключения некоторых сервисов к FreeIPA по протоколу LDAP, требуется указать учётные данные какого-либо пользователя. Для этих целей мы будем использовать специальный тип аккаунта 'system account'.

Преимущества данного типа аккаунта в том, что у него отсутствуют все атрибуты пользователя, кроме логина и пароля. 'System account' не имеет возможности:
- получить kerberos-билет;
- выполнить вход на хост или сервис;
- возобладать каким-либо объектом;
- записать или изменить в LDAP какую-либо информацию.

Также на 'system account' не распространяются пользовательские политики. Например, политика периодической смены пароля.

Информация о "системных аккаунтах" хранится в отдельной от остальных пользователей ветке 'cn=sysaccounts,cn=etc,dc=example,dc=org'. Для добавления 'system account' можно использовать обычный ldif-файл, но более удобным способом является использование [sh-скрипта freeipa-sam](https://github.com/noahbliss/freeipa-sam).

## Работа со скриптом freeipa-sam
Любым способом, даже через copy/paste, копируем [скрипт](https://raw.githubusercontent.com/noahbliss/freeipa-sam/master/freeipa-sam.sh) на, например, свою рабочую машину, которая имеет доступ к FreeIPA-домену.

Запускаем скрипт '/bin/bash freeipa-sam.sh' и видим:
```
### FreeIPA - System Account Manager ###
1.) ldapserver=
2.) domain= (ldapdomain=)
3.) binduser=
4.) bindpass=UNSET!
5.) ssl=true

Actions (conditions not yet met):
  add | rm | ls | info | passwd

---   Results   ---

--- End Results ---

>
```

Набирая цифру, соответствующую параметру, заполняем:
1. Вводим адрес LDAP-сервера:
  - `ldap.example.org` или `ldap.example.org:636`, если требуется указать нестандартный порт.
2. Самозаполнится, взяв имя домена из адреса сервера:
  - `example.org`
3. Вводим имя своей или, если такая предусмотрена регламентом, другой учётной записи с правами администратора:
  - `eugene`
4. Вводим пароль.
5. Оставляем без изменений.

Результат:
```
### FreeIPA - System Account Manager ###
1.) ldapserver=ldap.example.org
2.) domain=example.org (ldapdomain=dc=example,dc=org)
3.) binduser=uid=eugene,cn=users,cn=accounts,dc=example,dc=org
4.) bindpass=SET!
5.) ssl=true

Actions (ready):
  add | rm | ls | info | passwd

---   Results   ---

--- End Results ---

>
```

Переходим к выполнению операций над "системными аккаунтами":
- `add`&nbsp;&mdash; добавить 'system account'.
- `rm`&nbsp;&mdash; удалить аккаунт.
- `ls`&nbsp;&mdash; вывести краткий список существующих аккаунтов.
- `info`&nbsp;&mdash; вывести расширенную информацию по всем или одному из существующих аккаунтов.
- `passwd`&nbsp;&mdash; изменить пароль определённого аккаунта.

## Добавление аккаунта
Выполняя добавление 'system account', поочерёдно вводим логин и пароль. Запрос 'password expiration date YYYYMMDD (blank for 20380119)' оставляем незаполненным, что автоматически назначит аккаунту дату `20380119031407Z`, означающую бессрочное время действия пароля.
```
> add
uid of new user=binddn_test
password of new user=
password expiration date YYYYMMDD (blank for 20380119)=
```

По команде `ls` можно убедиться в наличии созданного аккаунта:
```
---   Results   ---
dn: uid=sudo,cn=sysaccounts,cn=etc,dc=example,dc=org
dn: uid=binddn_test,cn=sysaccounts,cn=etc,dc=example,dc=org
--- End Results ---
```

Командой `rm binddn_test` удаляем наш тестовый аккаунт.

Через `exit` завершаем работу со скриптом.

## Рекомендации к именованию новых system accounts
Пример: binddn_airflow, binddn_hue, binddn_hive.

Все сервисы в отдельной Информационной Системе, подключаемые к LDAP, имеют один 'system account'. В этом случае, при подозрении на компрометацию пароля с его последующей сменой, понадобится ввести новый пароль лишь в нескольких экземплярах сервиса, тогда как при использовании одного 'system account' для всех сервисов организации, объём работы по вводу нового пароля резко увеличивается.

К тому же, периодическая смена пароля, если будет введён такой регламент, может быть растянут по времени, когда пароль от 'system account' для каждого сервиса меняется в свой отдельный день. В этом случае, миграция производится безболезненно, потому что изменить пароль в трёх местах проще, чем одновременно в сотне мест.

Но замечу, что ничто не мешает создать новый 'system account' с чуть изменённым логином и уникальным паролем и переводить сервисы на его использование постепенно.

## Ссылки на использованные материалы
https://www.freeipa.org/page/HowTo/LDAP#System_Accounts
https://github.com/noahbliss/freeipa-sam
