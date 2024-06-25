---
title: "Краткий список утилит и команд оболочки для работы с процессами"
date: 2
weight: 10
description: >
  Краткий справочник по утилитам и командам оболочки для работы с процессами Linux.
tags:
  - Linux process
---

2022-03-29

## Обозреть работающие процессы

### `ps`
`ps`&nbsp;&mdash; process status.
```
user@wks01:~$ ps aux
```

Linux Minit:
```
user@wks01:~$ ps 1
    PID TTY      STAT   TIME COMMAND
      1 ?        Ss     0:08 /sbin/init
```

CentOS:
```
[root@srv01 ~]$ ps 1
  PID TTY      STAT   TIME COMMAND
    1 ?        Ss     0:55 /usr/lib/systemd/systemd --switched-root --system --deserialize 22
```

### `ptree`
`ptree`&nbsp;&mdash; process tree. Выводит дерево процессов.

### `top`, `htop`
Информация обо всех процессах в режиме реального времени.

## Получить информацию о процессе

### `pidoff`
`pidoff`&nbsp;&mdash; find the process ID of a running program.
```
user@wks01:~$ pidof mc
146747 45720 12313 7516
```

### `pgrep`
`pgrep`&nbsp;&mdash; process grep.
```
user@wks01:~$ pgrep -l fire
10900 firefox-bin
```

### `pwdx`
`pwdx`&nbsp;&mdash; print working dir process.
```
user@wks01:~$ pwdx 12313
12313: /home/user
```

### `lsof`
`lsof`&nbsp;&mdash; list open files.
```
[root@home-ipa05 ~]# lsof -i -n -P
COMMAND  PID    USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
rpcbind 2610     rpc    6u  IPv4  15059      0t0  UDP *:111
rpcbind 2610     rpc    7u  IPv4  15064      0t0  TCP *:111 (LISTEN)
rpcbind 2610     rpc    8u  IPv6  15065      0t0  UDP *:111
rpcbind 2610     rpc    9u  IPv6  15066      0t0  TCP *:111 (LISTEN)
chronyd 2647 _chrony    5u  IPv4  15253      0t0  UDP 127.0.0.1:323
chronyd 2647 _chrony    6u  IPv6  15254      0t0  UDP [::1]:323
sshd    2888    root    3u  IPv4  16102      0t0  TCP *:22 (LISTEN)
sshd    2888    root    4u  IPv6  16104      0t0  TCP *:22 (LISTEN)
sshd    2914    root    3u  IPv4  16110      0t0  TCP 172.16.14.10:22->172.16.14.1:45098 (ESTABLISHED)
```

### `fuser`
`fuser`&nbsp;&mdash; identify processes using files or sockets.

Какой процесс используется порт 22000 в пространстве имён &laquo;tcp&raquo;:
```
 user@wks01:~$ sudo fuser -v -n tcp 22000
                     USER        PID ACCESS COMMAND
22000/tcp:           eugene    221135 F.... syncthing
```

## Убить выбранный процесс
### `kill`, `pkill`, `killall`
`kill`, `pkill`, `killall`&nbsp;&mdash; process kill.
```
user@wks01:~$ kill 221135
user@wks01:~$ pkill -9 firefox
user@wks01:~$ sudo killall -u eugene
```

## Изменить приоритет процесса
### `nice`, `renice`
`nice`, `renice`&nbsp;&mdash; запуск процесса с заданным приоритетом &laquo;nice&raquo; или изменение приоритета &laquo;nice&raquo; работающего процесса.

Приоритет &laquo;nice&raquo; нельзя путать с действительный приоритетом процесса, выданным &laquo;планировщиком&raquo;.

Текущее значение &laquo;nice&raquo; при использовании утилиты `top`&nbsp;&mdash; столбец &laquo;NI&raquo;, тогда как действительный приоритет&nbsp;&mdash; столбец &laquo;PR&raquo;.

* &minus;20&nbsp;&mdash; самый высокий приоритет &laquo;nice&raquo; выполнения процесса.
* +20&nbsp;&mdash; самый низкий приоритет &laquo;nice&raquo;.

### `bg`, `fg`
- `bg`&nbsp;&mdash; запуск приложения из командной строки в фоновом режиме.
- `fg`&nbsp;&mdash; вывод приложенияиз фонового режима.

## Изучить работающий процесс
### `strace`
`strace`&nbsp;&mdash; system call tracer.
