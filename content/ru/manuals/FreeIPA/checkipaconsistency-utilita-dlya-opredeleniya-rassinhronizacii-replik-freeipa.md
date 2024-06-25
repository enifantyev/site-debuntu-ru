---
title: "Checkipaconsistency. Утилита для определения рассинхронизации реплик FreeIPA"
date: 2021-08-04
weight: 10
description: >
  Краткое описание утилиты используемой для наглядной демонстрации состояния синхронизации реплик FreeIPA.
tags:
  - FreeIPA
slug: checkipaconsistency-utilita-dlya-opredeleniya-rassinhronizacii-replik-freeipa
aliases:
  - /n/checkipaconsistency-utilita-dlya-opredeleniya-rassinhronizacii-replik-freeipa/
  - /manuals/freeipa/checkipaconsistency-utilita-dlya-opredeleniya-rassinhronizacii-replik-freeipa/
---

2021-08-04<br>
2022-01-27

## 1. Ссылки
https://github.com/peterpakos/checkipaconsistency

## 2. Описание
- Утилита может запускаться на любой из IPA-реплик.
- Так как для работы утилиты требуется пароль для УЗ 'cn=Directory Manager', то использование утилиты потенциально опасно.
- Безопасней запускать утилиту удалённо с применением пароля для УЗ 'cn=Directory Manager' хранящегося в приемлемом Vault'е.
- Утилита выводит таблицу с информацией по всем (если специально не настроено другое) репликам домена. Пример:
    ```
    $ sudo -i
    # cipa
    +--------------------+------------------+------------------+------------------+-------+
    | FreeIPA servers:   | dev-ipa01p       | dev-ipa02p       | dev-ipa03p       | STATE |
    +--------------------+------------------+------------------+------------------+-------+
    | Active Users       | 1                | 1                | 1                | OK    |
    | Stage Users        | 0                | 0                | 0                | OK    |
    | Preserved Users    | 0                | 0                | 0                | OK    |
    | Hosts              | 21               | 21               | 21               | OK    |
    | Services           | 15               | 15               | 15               | OK    |
    | User Groups        | 4                | 4                | 4                | OK    |
    | Host Groups        | 1                | 1                | 1                | OK    |
    | Netgroups          | 0                | 0                | 0                | OK    |
    | HBAC Rules         | 1                | 1                | 1                | OK    |
    | SUDO Rules         | 0                | 0                | 0                | OK    |
    | DNS Zones          | 2                | 2                | 2                | OK    |
    | Certificates       | 26               | 26               | 26               | OK    |
    | LDAP Conflicts     | 0                | 0                | 0                | OK    |
    | Ghost Replicas     | 0                | 0                | 0                | OK    |
    | Anonymous BIND     | ROOTDSE          | ON               | ON               | FAIL  |
    | Microsoft ADTrust  | False            | False            | False            | OK    |
    | Replication Status | dev-ipa02p 0     | dev-ipa01p 0     | dev-ipa02p 0     | OK    |
    |                    | dev-ipa03p 0     | dev-ipa03p 0     | dev-ipa01p 0     |       |
    +--------------------+------------------+------------------+------------------+-------+
    ```
- В вышеуказанном примере видно, что на второй и третьей реплике разрешён анонимный доступ к LDAP-каталогу.

## 3. Установка
1. На любой или на всех IPA-репликах входим в контекст root и устанавливаем утилиту в профиль root'а:
    ```
    $ sudo -i
    # python3 -m pip install --user checkipaconsistency
    ```
2. В каталоге `/root/.local/bin` появился файл запуска утилиты `cipa`. Чтобы запускать его без ввода полного пути, в файле `/root/.bashrc`, добавляем к переменной PATH путь $HOME/.local/bin.
    ```
    HOME=$HOME/.local/bin:$HOME
    ```
3. Предыдущую версию 2.7.10 можно установить через pip `pip install --user checkipaconsistency`, тогда как последнюю версию придётся собирать:
    ```
    $ git clone https://github.com/peterpakos/checkipaconsistency.git
    $ cd checkipaconsistency
    $ pip install --user -r requirements.txt
    $ ./cipa
    Initial config saved to ~/.config/checkipaconsistency - PLEASE EDIT IT!
    IPA domain not set
    ```

## 4. Настройка
1. После первого запуска утилиты cipa появится сообщение с просьбой заполнить содержимое конфигруационного файла `~/.config/checkipaconsistency`.
2. Выполняем эту просьбу. Пример содержимого:
    ```
    [IPA]
    domain = example.org
    #hosts = ipa01, ipa02, ipa03, ipa04, ipa05, ipa06
    binddn = cn=Directory Manager
    bindpw = ~D%K-JEWEJIbn5:Gx1N.TWn<UybGHoOU@cDbgs7iShL
    ```
3. Так как для работы утилиты требуется пароль для УЗ 'cn=Directory Manager', то использование утилиты потенциально опасно.

## 5. Способы использования
### 5.1. Запуск утилиты без параметров
1. Если путь `~/.local/bin` добавлен к пеменной PATH и присутствует файл `~/.config/checkipaconsistency` с надлежащим содержимым, то запускам утилиту без параметров:
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
2. <span style="color: red">Так как для работы утилиты требуется пароль для УЗ 'cn=Directory Manager', то локальное использование утилиты потенциально опасно.</span>

### 5.2. Запуск утилиты с параметрами
1. На любой IPA-реплике из командной строки без использования файла `~/.config/checkipaconsistency` запускаем утилиту, временно отключая запись истории:
    ```
    set +o history
    cipa -d test01.corp -D'cn=Directory Manager' -W '~D%K-JEWEJIbn5:Gx1N.TWn<UybGHoOU@cDbgs7iShL'
    set -o history
    ```
2. Если в Bash переменная HISTCONTROL установлена в значения 'ignorespace' или 'ignoredup', то есть способ запустить команду без попадания в history, если перед командой ввести пробел. В примере ниже лидирующий пробел обозначен символом '▂':
    ```
    ▂cipa -d tes...
    ```

### 5.3. Запуск утилиты удалённо
1. Самый безопасный способ, если исходить из принципа отсутствия на администрируемой машине каких-либо секретов.

## 6. Встреченные проблемы
### 6.1. configparser.InterpolationSyntaxError: '%' must be followed by '%' or '(', found
При наличии символа '%' в пароле 'cn=Directory Studio', запуск `cipa` завершается ошибкой:
```
configparser.InterpolationSyntaxError: '%' must be followed by '%' or '(', found
```

Проблема исчезла после замены в строке пароля символа '%' на два символа '%%', как и написано в сообщении. Теперь запуск `cipa` заканчивается корректным выводом информации по согласованности реплик домена.
