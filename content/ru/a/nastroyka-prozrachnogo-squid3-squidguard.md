---
title: "Настройка прозрачного squid3 + squidGuard"
date: 2011-03-12
weight: 10
description: >
  Настройка прозрачного squid3 + squidGuard
tags:
  - Squid
  - SquidGuard
aliases:
  - /nastroyka-prozrachnogo-squid3-squidguard
---

2011-03-12

## Установка Squid3 SquidGuard
```
# aptitude install squid3 squidguard
```
Файлы настройки squid3 лежат в `/etc/squid3`. Файл настройки squidGuard лежит в `/etc/squid`. Если бы мы ставили предыдущую версию squid, то настройки обоих приложений лежали в одной папке `/etc/squid`.

## Настройка Squid3

Настройки squid3 лежат в файле `/etc/squid3/squid.conf`. Не надо пугаться обилию информации в этом файле, так как достаточно включить цветовыделение синтаксиса, чтобы увидеть, что работающих строк не больше пары или тройки десятков. Все остальные строки закоментированы и представляют собой описание, настройки по умолчанию и примеры опций squid3.

Файл /etc/squid3/squid.conf состоит из нескольких разделов...

#### *Options for authentication*
Раздел для настройки авторизации. При настройке прозрачного прокси - не нужен. Все строки закомментированы.

#### *Access controls*
Контроль доступа. Важный раздел. Работают 23 строки. Добавляем две работающие строки.
```
acl manager proto cache_object  
acl localhost src 127.0.0.1/32  
acl to_localhost dst 127.0.0.0/8  
# Добавляем описание нашей подсети:
++acl localnet src 192.168.15.0/24
acl SSL_ports port 443  
acl Safe_ports port 80 # http  
acl Safe_ports port 21 # ftp  
acl Safe_ports port 443 # https  
acl Safe_ports port 70 # gopher  
acl Safe_ports port 210 # wais  
acl Safe_ports port 1025-65535 # unregistered ports  
acl Safe_ports port 280 # http-mgmt  
acl Safe_ports port 488 # gss-http  
acl Safe_ports port 591 # filemaker  
acl Safe_ports port 777 # multiling http  
acl CONNECT method CONNECT  
http_access allow manager localhost  
http_access deny manager  
http_access deny !Safe_ports  
http_access deny CONNECT !SSL_ports  
# Этой строчкой разрешаем использование squid из локальной сети:
++http_access allow localnet
http_access allow localhost  
http_access deny all  
icp_access deny all  
htcp_access deny all
```

#### *Network options*
Сетевые настройки. По умолчанию раскомментирована лишь одна строка:
```
http_port 3128
```
Изменяем эту опцию, добавляя интерфейс, на котором принимаем запросы, и включая прозрачность:
```
http_port 192.168.15.1:3128 transparent
```

#### *SSL options*
Настройка прокси для ssl-запросов. Ни одной строчки не раскомментировано. Этот раздел нам не нужен, так как будем выпускать ssl-запросы через iptables.

#### *Options which affect the neighbor selection algorithm*
У нас один прокси сервер squid и это раздел нам не нужен. Раскомментирована одна строка:
```
hierarchy_stoplist cgi-bin ?
```

#### *Memory cache options*
Использование оперативной памяти. Ни одной работающей строчки. Оставляем все настройки по умолчанию.

#### *Disk cache options*
Интересный раздел по кэшированию запросов на винчестере. Ни одной работающей строчки. Оставляем все настройки по умолчанию.

#### *Logfile options*
Одна работающая строка:
```
access_log /var/log/squid3/access.log squid
```

#### *<Options for FTP gatewaying, Options for external support programs>*
Эти разделы полностью закомментированы.

#### *Options for URL rewriting*
Ни одной работающей строки. Здесь включим редирект на squidGuard:
```
url_rewrite_program /usr/bin/squidGuard -c /etc/squid/squidGuard.conf
```
Помним, что из-за этой строки, пока squidGuard не будет настроен, squid3 будет падать сразу после запуска!

#### *<Options for tuning the cache, HTTP options, Timeouts, Administrative parameters, Options for the cache registration service, HTTPD-accelerator options, Delay pool parameters, WCCPv1 and WCCPv2 configuration options, Persistent connection handling, Cache digest options, SNMP options>*
Эти разделы полностью закомментированы.

#### *ICP options*
icp_port 3130

#### *<Multicast ICP options, Internal icon options>*
Эти разделы полностью закомментированы.

#### *Error page options*
Здесь включим русификацию ошибок прокси-сервера, выдаваемых в ответ на запросы пользователей несуществующих страниц в интернете и т.д.:
```
error_directory /usr/share/squid3/errors/Russian-1251
```

#### *< Options influencing request forwarding, Advanced networking options, ICAP options, DNS options>*
Эти разделы полностью закомментированы.

#### *Miscellaneous*
```
coredump_dir /var/spool/squid3
```
Будет лучше переименовать существующий файл `squid3.conf` и создать новый пустой `squid3.conf`, наполнив его приведёнными выше работающими строками.

## Настройка squidGuard
Скачиваем http://www.shallalist.de/Downloads/shallalist.tar.gz (эта база разрешена к использованию только в некоммерческих или личных целях). В этом архиве лежат списки доменов и URL’ов распределённых по соответствующим папкам. Переносим все или избранные папки в `/var/lib/squidguard/db`. В результате получаем, что в `/var/lib/squidguard/db/socialnet` лежат списки адресов социальных сетей. В `/var/lib/squidguard/db/porn` лежат списки сайтов с поревом. И т.д.

Теперь приступим к файлу настроек `/etc/squid/squidGuard.conf`. Вероятно будет лучше переименовать этот файл и создать новый пустой `/etc/squid/squidGuard.conf`, наполнив его приведёнными строками:

<details><p>

```
dbhome /var/lib/squidguard/db  
logdir /var/log/squid  
# SOURCE ADDRESSES:  
src bigboss {  
ip 192.168.0.100 #Компьютер директора  
ip 192.168.0.101 #Компьютер зама  
}  
src admin {  
ip 192.168.0.7 #Мой компьютер  
}  
# DESTINATION CLASSES:  
dest adv {  
domainlist adv/domains  
urllist adv/urls  
}  
dest chat {  
domainlist chat/domains  
urllist chat/urls  
}  
dest costtraps {  
domainlist costtraps/domains  
urllist costtraps/urls  
}  
dest dating {  
domainlist dating/domains  
urllist dating/urls  
}  
dest fortunetelling {  
domainlist fortunetelling/domains  
urllist fortunetelling/urls  
}  
dest gamble {  
domainlist gamble/domains  
urllist gamble/urls  
}  
dest porn {  
domainlist porn/domains  
urllist porn/urls  
}  
dest redirector {  
domainlist redirector/domains  
urllist redirector/urls  
}  
dest sex {  
domainlist sex/domains  
urllist sex/urls  
}  
dest spyware {  
domainlist spyware/domains  
urllist spyware/urls  
}  
dest tracker {  
domainlist tracker/domains  
urllist tracker/urls  
}  
dest socialnet {  
domainlist socialnet/domains  
urllist socialnet/urls  
}  
dest violence {  
domainlist violence/domains  
urllist violence/urls  
}  
dest warez {  
domainlist warez/domains  
urllist warez/urls  
}  
dest webmail {  
domainlist webmail/domains  
urllist webmail/urls  
}  
acl {

bigboss {

pass !costtraps !redirector !porn !spyware !tracker all  
redirect http://192.168.0.1/squidguard.html  
}  
# admin {  
# pass any  
# }  
default {  
pass !adv !chat !costtraps !dating !fortunetelling !gamble !porn !redirector !socialnet !spyware !tracker !violence !warez !webmail all  
redirect http://192.168.0.1/squidguard.html  
}  
}
```
</p></details>

Надо помнить, что без указания `redirect` на какую-либо страницу на произвольном сервере фильтрование работать не будет. В данном случае на машине с ip-адресом 192.168.0.1 поднят Apache и опубликована простая страница с содержимым, состоящим из одного слова - Blocked.

Так же - на каждый объявленный в начале файла параметр `src`, должен быть свой `acl` в конце файла, иначе компьютер с ip-адресом 192.168.0.7 (смотрим вышенаписанный пример) не попадёт ни в какой из фильтров.

Если посмотреть на последние строки файла `squidGuard.conf`, то видно, что для компьютеров директора и его зама доступны все адреса, кроме costtraps, redirector, porn, spyware, tracker. Для всех остальных компьютеров список запретов более обширен. Для компьютера администратора оставлена возможность обхода запретов для его, сугубо администраторских, нужд. Хе, хе.

Теперь запускаем:
```
$ squidGuard -C all
```
Файлы, перечисленные в squidguard.conf, будут преобразованы в формат db для ускорения работы. Например, в папке `adv` появятся два файла: `domains.db` и `urls.db`.

Исходные текстовые файлы удалять не надо. Могут пригодиться при дополнении или пересоздании базы.

Изменяем хозяина и группу на содержимом папки /var/lib/squidguard, так как в эту папку будет лазить squid:
```
$ chown -R proxy:proxy *
```

#### Изменение записей в списках доменов и URL

Пример. Рядом с файлом `domains.db` в папке `/var/lib/squiguard/db/webmail` создаём файл `domains.diff`. В него заносим строку или несколько строк, по одной на каждую запись:
```
-google.com (что означает вычеркнуть этот домен из базы)  
или
+google.com (что означает добавить этот домен в базу)  
```
Даём команды:
```
$ squidGuard -u (обновить базы db из файлов diff. В логах squidguard'а можно посмотреть сколько добавилось/убылось.)  
$ squid3 -k reconfigure (перечитать настройки без перезапуска.)
```
Файл `domains.diff` удалять, или стирать из него записи, не надо. При глобальном обновлении баз этот файл ещё пригодится. И при многократном обновлении не происходит дублирования записей в БД.

## Настройка iptables для работы squid3

192.168.15.0/24 - локальная сеть  
eth0 = 192.168.15.1 - внутренний сетевой интерфейс  
eth1 =  1.1.1.1 - внешний сетевой интерфейс  

```
# Очищаем таблицы и даём правила по умолчанию  
iptables -F -t filter  
iptables -F -t nat  
iptables -F -t mangle  
iptables -P INPUT DROP  
iptables -P OUTPUT DROP  
iptables -P FORWARD DROP  

# Добавляем в iptables строчку перенаправления запросов юзеров с 80-го порта на порт сквида 3128:  
iptables -t nat -A PREROUTING -i eth0 -p tcp -m tcp -s 192.168.15.0/24 ! -d 192.168.15.0/24 --dport 80 -j REDIRECT --to-ports 3128  
# Включаем NAT, чтобы пропускать SSL, DNS, NTP и т.д. запросы:  
iptables -t nat -A POSTROUTING -o eth1 -s 192.168.15.0/24 ! -d 192.168.15.0/24 -j SNAT --to 1.1.1.1  
# Или, для динамически назначаемых внешних ip:  
# iptables -t nat -A POSTROUTING -o eth1 -s 192.168.15.0/24 -j MASQUERADE  
# Разрешаем вход на порт и выход с порта номер 3128  
iptables -A INPUT  -p tcp -m tcp -i eth0 -s 192.168.15.0/24 --dport 3128 -j ACCEPT  
iptables -A OUTPUT -p tcp -m tcp -o eth0 -d 192.168.15.0/24 --sport 3128 -j ACCEPT -m conntrack --ctstate RELATED,ESTABLISHED  
# Разрешаем squid'у выходить в инет  
iptables -A OUTPUT -p tcp -m tcp -o eth1 ! -d 192.168.15.0/24 --dport 80 -j ACCEPT  
iptables -A INPUT  -p tcp -m tcp -i eth1 ! -s 192.168.15.0/24 --sport 80 -j ACCEPT -m conntrack --ctstate RELATED,ESTABLISHED  
# И делаем проход для SSL, DNS, NTP:  
iptables -A FORWARD -p tcp -m tcp -i eth0 -o eth1 ! -d 192.168.15.0/24 -m multiport --dports 53,123,443 -j ACCEPT  
iptables -A FORWARD -p tcp -m tcp -i eth1 -o eth0 ! -s 192.168.15.0/24 -m multiport --sports 53,123,443 -j ACCEPT -m conntrack --ctstate RELATED,ESTABLISHED  
iptables -A FORWARD -p udp -m udp -i eth0 -o eth1 ! -d 192.168.15.0/24 -m multiport --dports 53,123 -j ACCEPT  
iptables -A FORWARD -p udp -m udp -i eth1 -o eth0 ! -s 192.168.15.0/24 -m multiport --sports 53,123 -j ACCEPT -m conntrack --ctstate RELATED,ESTABLISHED  
# Разрешаем вход на локальный 80 порт. Здесь крутится Apache2  
# и сюда идёт перенаправление со squidguard'а.  
iptables -A INPUT  -p tcp -m tcp -i eth0 -s 192.168.15.0/24 --dport 80 -j ACCEPT  
iptables -A OUTPUT -p tcp -m tcp -o eth0 -d 192.168.15.0/24 --sport 80 -j ACCEPT -m conntrack --ctstate RELATED,ESTABLISHED
```
```
# Включаем форвардинг пакетов через ядро для прохода SSL, DNS, NTP и т.д. пакетов,  
# так как squid перекидывает мимо ядра только http пакеты  
echo 1 > /proc/sys/net/ipv4/ip_forward
```

## Настройка локальных машин на использование squid3

На локальных машинах в сетевых настройках достаточно поменять ip-адрес шлюза по умолчанию на ip-адрес сервера с работающим squid’ом, чтобы начать использовать прозрачный прокси-сервер.

## Разделение канала с помощью SQUID

Из книги "Linux-сервер своими руками. Полное руководство. Колисниченко Д.Н."

> Допустим, вам нужно настроить прокси-сервер таким образом, чтобы одна группа компьютеров могла работать с одной скоростью, а другая — с другой. Это может потребоваться, например, для разграничения пользователей, которые используют канал для работы, и пользователей, которые используют ресурсы канала в домашних целях. Естественно, первым пропускная способность канала важнее, чем вторым. С помощью прокси-сервера SQUID можно разделить канал.  
> Для начала в файле конфигурации (/etc/squid/squid.conf) укажите, сколько пулов, то есть групп пользователей, у вас будет:  
> delay_pools 2  
> Затем определите классы пулов. Всего существует три класса:  
> 1\. Используется одно ограничение пропускной способности канала на всех.  
> 2\. Одно общее ограничение и 255 отдельных для каждого узла сети класса С.  
> 3\. Для каждой подсети класса В будет использовано собственное ограничение и отдельное ограничение для каждого узла.  
> В файл squid.conf добавьте следующие директивы:  
> delay class 1 1 # определяет первый пул класса 1 для домашних пользователей  
> delay class 2 2 # определяет второй пул класса 2 для служащих  
> Теперь задайте узлы, которые будут относиться к пулам:  
> асl home src адреса  
> acl workers src адреса  
> delay access 1 allow home  
> delay access 1 deny all  
> delay access 2 allow workers  
> delay access 2 deny all  
> Затем укажите ограничения:  
> delay_parameters 1 14400/14400  
> delay_parameters 2 33600/33600 16800/33600  
> Как я уже отмечал выше, для пула класса 1 используется одно ограничение для всех компьютеров, входящих в пул — 14400 байт. Первое число задает скорость заполнения для всего пула (байт/секунду). Второе — максимальное ограничение.  
> Для пула класса 2, соответственно, используются ограничения на всю подсеть и отдельно на каждого пользователя. Если бы у нас был определен пул класса 3, то для него ограничения выглядели бы примерно так:  
> delay_parameters 3 128000/128000 64000/128000 12800/64000  
> Первые два числа задают соответственно скорость заполнения и максимальное ограничение для всех. Следующая пара чисел определяет скорость заполнения для каждой подсети и максимальное ограничение, а третья - скорость заполнения и максимальное ограничение для индивидуального пользователя.
