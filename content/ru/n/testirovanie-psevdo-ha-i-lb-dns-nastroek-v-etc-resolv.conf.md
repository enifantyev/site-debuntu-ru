---
title: "Тестирование псевдо HA и LB dns-настроек в /etc/resolv.conf"
date: 2021-07-14
weight: 10
description: >
  Псевдо High Availability и Load Balancing достигается поочерёдным опросом всех указанных в файле `/etc/resolv.conf` dns-серверов, даже в том случае, если какой-нибудь из dns-серверов находится в офлайне.
tags:
  - resolv.conf
  - DNS
---

2021-07-14

## Введение
Псевдо High Availability и Load Balancing достигается поочерёдным опросом всех указанных в файле `/etc/resolv.conf` dns-серверов, даже в том случае, если какой-нибудь из dns-серверов находится в офлайне.

Сразу приведу пример такого HA и LB 'resolv.conf':
```
options rotate timeout:1 attempts:1
search test.lan
nameserver 10.1.120.146
nameserver 10.1.120.147
nameserver 10.1.120.148
```

Замечу, что использование локальных кэширующих dns-прослоек, типа dnsmasq, умеющих обращаться сразу ко всем dns-серверами, может несколько увеличить HA и LB.

## Подготовка к тестированию
Создадим следующий python-файл и сделаем его исполняемым:
```
$ cat << EOF > test.py
#!/usr/bin/python3
import socket
for x in range(6):
   print(socket.getaddrinfo('dev-ipa01p.test.lan', 80));
EOF

$ chmod u+x test.py
```

## Тестирование со стандартным resolv.conf
Создадим стандартный `/etc/resolv.conf`:
```
$ cat << EOF | sudo tee /etc/resolv.conf
search test.lan
nameserver 10.1.120.146
nameserver 10.1.120.147
nameserver 10.1.120.148
EOF
```

### Первый dns-сервер в онлайне
Проверим работу настроек:
```
$ strace -e connect ./test.py 2>&1 | grep 53

connect(3, {sa_family=AF_INET, sin_port=htons(53), sin_addr=inet_addr("10.1.120.146")}, 16) = 0
connect(3, {sa_family=AF_INET, sin_port=htons(53), sin_addr=inet_addr("10.1.120.146")}, 16) = 0
connect(3, {sa_family=AF_INET, sin_port=htons(53), sin_addr=inet_addr("10.1.120.146")}, 16) = 0
...
```

Здесь мы видим, что запросы идут только к первому dns-серверу.

### Все dns-серверы в офлайне
Чтобы не выключать первый dns-сервер, просто поменяем ip-адрес для соответствующей записи в `/etc/resolv.conf`:
```
$ sudo sed -i 's/nameserver 10.1.120.146/nameserver 10.1.120.140/' /etc/resolv.conf
```

Проверим работу настроек:
```
$ strace -e connect ./test.py 2>&1 | grep 53

connect(3, {sa_family=AF_INET, sin_port=htons(53), sin_addr=inet_addr("10.1.120.140")}, 16) = 0
connect(3, {sa_family=AF_INET, sin_port=htons(53), sin_addr=inet_addr("10.1.120.147")}, 16) = 0
connect(3, {sa_family=AF_INET, sin_port=htons(53), sin_addr=inet_addr("10.1.120.140")}, 16) = 0
connect(3, {sa_family=AF_INET, sin_port=htons(53), sin_addr=inet_addr("10.1.120.147")}, 16) = 0
```

Здесь мы видим, что запросы идут только к первому и, после некоторой задержки, ко второму dns-серверу.

## Тестирование с отказоустойчивым resolv.conf
Создадим отказоустойчивый `/etc/resolv.conf`:
```
$ cat << EOF | sudo tee /etc/resolv.conf
options rotate timeout:1 attempts:1
search test.lan
nameserver 10.1.120.146
nameserver 10.1.120.147
nameserver 10.1.120.148
EOF
```

Здесь добавлены опции:
- опция 'rotate' заставляет отправлять dns-запросы по очереди к каждому dns-серверу по кругу;
- опция 'timeout:1' уменьшает ожидание ответа dns-сервера с дефолтных трёх секунд до одной секунды;
- опция 'attempts:1' разрешает только один запрос к каждому DNS-серверу;

### Все dns-серверы в онлайне
Проверим работу настроек:
```
$ strace -e connect ./test.py 2>&1 | grep 53

connect(3, {sa_family=AF_INET, sin_port=htons(53), sin_addr=inet_addr("10.1.120.147")}, 16) = 0
connect(3, {sa_family=AF_INET, sin_port=htons(53), sin_addr=inet_addr("10.1.120.148")}, 16) = 0
connect(3, {sa_family=AF_INET, sin_port=htons(53), sin_addr=inet_addr("10.1.120.146")}, 16) = 0
connect(3, {sa_family=AF_INET, sin_port=htons(53), sin_addr=inet_addr("10.1.120.147")}, 16) = 0
connect(3, {sa_family=AF_INET, sin_port=htons(53), sin_addr=inet_addr("10.1.120.148")}, 16) = 0
connect(3, {sa_family=AF_INET, sin_port=htons(53), sin_addr=inet_addr("10.1.120.146")}, 16) = 0
```

Здесь мы видим, что запросы идут к каждому dns-серверу по кругу.

### Первый dns-сервер в офлайне
Чтобы не выключать первый dns-сервер, просто поменяем ip-адрес для соответствующей записи в `/etc/resolv.conf`:
```
$ sudo sed -i 's/nameserver 10.1.120.146/nameserver 10.1.120.140/' /etc/resolv.conf
```

Проверим работу настроек:
```
$ strace -e connect ./test.py 2>&1 | grep 53

connect(3, {sa_family=AF_INET, sin_port=htons(53), sin_addr=inet_addr("10.1.120.140")}, 16) = 0
connect(3, {sa_family=AF_INET, sin_port=htons(53), sin_addr=inet_addr("10.1.120.147")}, 16) = 0
connect(3, {sa_family=AF_INET, sin_port=htons(53), sin_addr=inet_addr("10.1.120.147")}, 16) = 0
connect(3, {sa_family=AF_INET, sin_port=htons(53), sin_addr=inet_addr("10.1.120.148")}, 16) = 0
connect(3, {sa_family=AF_INET, sin_port=htons(53), sin_addr=inet_addr("10.1.120.140")}, 16) = 0
connect(3, {sa_family=AF_INET, sin_port=htons(53), sin_addr=inet_addr("10.1.120.147")}, 16) = 0
connect(3, {sa_family=AF_INET, sin_port=htons(53), sin_addr=inet_addr("10.1.120.147")}, 16) = 0
connect(3, {sa_family=AF_INET, sin_port=htons(53), sin_addr=inet_addr("10.1.120.148")}, 16) = 0
```

Здесь мы видим, что запросы так и продолжают выполняться к каждому dns-серверу по кругу, но с одним малозначимым отличием. После попытки резольвинга от неотвечающего dns-сервера, к следующему работающему dns-серверу производится два запроса. На данный момент я не знаю причин такого поведения, тем более, что не вижу здесь проблемы. Чтение исходников могло бы пролить свет.
