---
title: "Установка Monitorix в Alpine Linux"
date: 2020-04-14
weight: 10
description: >
  Краткая инструкция по ручной установке Monitorix'а.
tags:
  - Monitorix
  - Alpine Linux
---

2020-04-14

## Подготовка
Скачиваем, и распаковываем куда-нибудь, актуальную версию monitorix:
```
$ wget https://www.monitorix.org/monitorix-3.12.0.tar.gz
$ tar xvf monitorix-3.12.0.tar.gz
```

Содержимое распакованного каталога раскидываем по папкам. Схема размещения приведена ниже:
<details><p>
```
$ tree monitorix-3.12.0
monitorix-3.12.0
├── Changes
├── COPYING
├── docs
│   ├── debian.conf
│   ├── htpasswd.pl
│   ├── monitorix-alert.sh
│   ├── monitorix-apache.conf
│   ├── monitorix-deb.init
│   ├── monitorix.init
│   ├── monitorix-lighttpd.conf
│   ├── monitorix.logrotate     --> /etc/logrotate.d/monitorix
│   ├── monitorix.nanorc
│   ├── monitorix.service
│   ├── monitorix.spec
│   ├── monitorix.sysconfig     --> /etc/conf.d/monitorix
│   └── monitorix.upstart
├── lib                         --> /usr/lib/monitorix/
│   ├── ambsens.pm              --> /usr/lib/monitorix/ambsens.pm
│   ├── apache.pm               --> /usr/lib/monitorix/apache.pm
│   ├── apcupsd.pm              --> ...
│   ├── bind.pm                 ...
│   ├── chrony.pm
│   ├── disk.pm
│   ├── du.pm
│   ├── emailreports.pm
│   ├── fail2ban.pm
│   ├── fs.pm
│   ├── ftp.pm
│   ├── gensens.pm
│   ├── hptemp.pm
│   ├── HTTPServer.pm
│   ├── icecast.pm
│   ├── int.pm
│   ├── ipmi.pm
│   ├── kern.pm
│   ├── libvirt.pm
│   ├── lighttpd.pm
│   ├── lmsens.pm
│   ├── mail.pm
│   ├── memcached.pm
│   ├── mongodb.pm
│   ├── Monitorix.pm
│   ├── mysql.pm
│   ├── net.pm
│   ├── netstat.pm
│   ├── nfsc.pm
│   ├── nfss.pm
│   ├── nginx.pm
│   ├── ntp.pm
│   ├── nut.pm
│   ├── nvidia.pm
│   ├── pagespeed.pm
│   ├── phpapc.pm
│   ├── phpfpm.pm
│   ├── port.pm
│   ├── process.pm
│   ├── proc.pm
│   ├── raspberrypi.pm
│   ├── serv.pm
│   ├── squid.pm
│   ├── system.pm
│   ├── tc.pm
│   ├── traffacct.pm
│   ├── unbound.pm
│   ├── user.pm
│   ├── varnish.pm
│   ├── verlihub.pm
│   ├── wowza.pm
│   └── zfs.pm
├── logo_bot.png        --> /var/lib/monitorix/www/
├── logo_top.png        --> /var/lib/monitorix/www/
├── Makefile
├── man
│   ├── man5
│   │   └── monitorix.conf.5
│   └── man8
│       └── monitorix.8
├── monitorix           --> /usr/bin/
├── monitorix.cgi       --> /var/lib/monitorix/www/cgi/
├── monitorix.conf      --> /etc/monitorix/
├── monitorixico.png    --> /var/lib/monitorix/www/
├── README
├── README.FreeBSD
├── README.md
├── README.NetBSD
├── README.nginx
├── README.OpenBSD
└── reports             --> /var/lib/monitorix/www/
    ├── ca.html         --> /var/lib/monitorix/www/reports/
    ├── de.html         --> /var/lib/monitorix/www/reports/
    ├── en.html         --> ...
    ├── fr.html         ...
    ├── it.html
    ├── nl_NL.html
    ├── pl.html
    ├── sk.html
    └── zh_CN.html
```
</p></details><br>


Перед раскидыванием файлов мониторикса по соответствующим каталогам, зададим им правильного владельца:
```
# cd monitorix-3.12.0 && chown root:root * -R
```

<!-- Перепроверить при случае.
Создание каталогов и копирование файлов. Не забыть выставить правильные права:
```
mkdir -p /etc/monitorix/conf.d /var/lib/monitorix/www/cgi /var/lib/monitorix/www/imgs

cp monitorix /usr/bin/
cp monitorix.conf /etc/monitorix/
cp monitorix.cgi /var/lib/monitorix/www/cgi/
cp logo_bot.png /var/lib/monitorix/www/
cp logo_top.png /var/lib/monitorix/www/
cp monitorixico.png /var/lib/monitorix/www/
cp lib/* /usr/lib/monitorix/
cp -r reports /var/lib/monitorix/www/
```
-->

## Добавление в систему необходимых пакетов
Установка perl-пакетов, необходимых для работы monitorix:
```
# apk add perl perl-cgi perl-libwww perl-mailtools perl-mime-lite perl-dbi perl-xml-simple perl-xml-libxml perl-http-server-simple perl-io-socket-ssl rrdtool perl-rrd
```

Пакет perl-config-general отсутствует в репозиториях Alpine Linux, поэтому скачал General.pm из metacpan.org в каталог `/usr/share/perl5/vendor_perl/Config`:
```
# mkdir -p /usr/share/perl5/vendor_perl/Config/
# wget -O /usr/share/perl5/vendor_perl/Config/General.pm https://metacpan.org/release/Bundle-PBib/source/lib/Config/General.pm?raw=1
```

Необходимо установит шрифт ttf-dejavu, ttf-freefont или font-noto, иначе вместо символов, на графиках будут видны квадраты.
```
# apk add ttf-dejavu
```


## Включение автозапуска
Готовый скрипт для openrc взял из генту `https://gitweb.gentoo.org/repo/gentoo.git/tree/www-misc/monitorix/files/monitorix` слегка подредактировав строку `command=/usr/sbin/monitorix`:
```
#!/sbin/openrc-run
# Copyright 1999-2019 Gentoo Authors
# Distributed under the terms of the GNU General Public License v2

name="Monitorix"
description="Monitorix is a lightweight system monitoring tool"
command=/usr/bin/monitorix
command_args="-c /etc/monitorix/monitorix.conf -p /var/run/$name.pid"
pidfile=/var/run/monitorix.pid

checkconfig() {
    if [[ ! -e /etc/monitorix/monitorix.conf ]]; then
        eerror "Please check that the configuration file exists."
        return 1
    fi
}

start() {
    checkconfig || return 1
    ebegin "Starting $name"
    start-stop-daemon --start --name $name --pidfile /var/run/$name.pid --exec $command -- $command_args
    eend $?
}

stop() {
    ebegin "Stopping $name"
    start-stop-daemon --stop --pidfile /var/run/$name.pid
    eend $?
}
```

Добавляем в автозагрузку:
```
# rc-update add monitorix default
```

## Замечено
### Users using the system (user.pm)
В musl нет реализации wtmp, utmp, lastlog. По этой причине модуль `user.pm` не работает.

### Process statistics (process.pm)
Для правильно отображения *Process statistics*, добавляем полноценную версию утилиты `ps`, вместо встроенной в busybox:
```
# apk add procps
```

### Netstat statistics (netstat.pm)
Требует установки утилиты `ss` или некастрированной `netstat` (вместо кастрированной из busybox, которая присутствует в Alpine Linux по умолчанию):
```
apk add net-tools
```
