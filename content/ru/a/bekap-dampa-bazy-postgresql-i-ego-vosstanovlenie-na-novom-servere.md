---
title: "Бэкап дампа базы Postgresql и его восстановление на новом сервере"
date: 2020-04-21
weight: 10
description: >
  Описание создания, перемещения и восстановления экземпляра PostgreSQL.
tags:
  - PostgreSQL
  - DB
---

2020-04-21

Создание дампа может занять продолжительное время, поэтому работы производим через `screen`, который позволит нам сохранить работающий сеанс на сервере, даже если мы отключимся случайно или по какой-то причине.

## Работы на исходном сервере
### Сбор сведений перед созданием дампа
На исходном сервере запоминаем версию postgresql:
```bash
# rpm -qa|grep sql
postgresql11-server-11.4-1PGDG.rhel7.x86_64
postgresql11-11.4-1PGDG.rhel7.x86_64
postgresql11-libs-11.4-1PGDG.rhel7.x86_64
```

Смотрим `who` и активные сетевые подключения через `ss -tul`, чтобы удостовериться в своём одиноком присутствии на сервере.

Ознакомимся со свободным местом на сервере, где увидим, что есть свободные 340G:
```bash
# df -h
Filesystem             Size  Used Avail Use% Mounted on
/dev/mapper/rhel-root  1.3T  906G  340G  73% /
devtmpfs                16G     0   16G   0% /dev
tmpfs                   16G  8.0K   16G   1% /dev/shm
tmpfs                   16G  353M   16G   3% /run
tmpfs                   16G     0   16G   0% /sys/fs/cgroup
/dev/sda1              950M  251M  700M  27% /boot
tmpfs                  3.2G     0  3.2G   0% /run/user/1000
tmpfs                  3.2G     0  3.2G   0% /run/user/1007
```

Ищем postgresql в обычном месте `/var/lib` и находим экземпляр в `/var/lib/pgsql/11/data`. Узнаём размер каталога:
```bash
# du -hs du -hs /var/lib/pgsql/11/data
895G    du -hs /var/lib/pgsql/11/data
```

Огромная база в 900 Гигабайт и свободное место на 340 Гигабайт. Интересуемся у ответственного лица о содержимом базы:
```
> Ещё вопрос, пожалуйста. Внутри базы postgres хорошо сжимаемые данные, типа текста, или что-нибудь несжимаемое, типа jpg?

< картинок нет точно. в основном это csvшки с данными

> отлично. Ещё раз спасибо.
```

Из вышеприведённого диалога понятно, что база заполнена хорошо сжимаемым содержимым. Опережая события сообщу, что база сжалась до 52G (за 1,5 часа?).

В противном случае нам пришлось бы сжимать базу с автоматическим разделением дампа на части, чтобы была возможность переносить готовые куски на новый сервер, освобождая место на старом сервере для следующих параллельно создаваемых частей. Или, в крайнем случае, останавливаем postgres и, за 10-20 часов, в зависимости от ширины канала, копируем содержимое `/var/lib/pgsql/11/data` на новый сервер.

### Выбор места создания дампа
После знакомства со структурой каталогов на сервере, решаем попробовать задампиться в `/var/lib/pgsql/11/backups`. Мне кажется, что выбор этого места логичен в данном случае.

### Создание полного дампа исходного экземпляра posgresql
Определяем количество ядер у процессора (через `top`, `htop`, `cat /proc/cpuinfo`) и запускаем создание дампа со сжатием через lbzip2, с указанием найденного количества ядер для ускорения работы (в данном случае восемь потоков):
```bash
sudo su
su - postgres
time pg_dumpall -v | lbzip2 -n 8 -9 > /var/lib/pgsql/11/backups/pgsql_dumpall_$(date -u +%Y%m%d-%H%M%S).bz2
```

Забегая вперёд покажу затраченное время:
```
real    88m17.100s
user    463m50.451s
sys     16m31.101s
```

Посмотреть на текущий, и всё увеличивающийся, размер базы можно через дополнительный "экран screen" (масло масляное :-).

Пока дамп создаётся, не теряем времени и заходим на новый сервер.

## Подготовка нового сервера к поднятию копии postgesql
### Стандартные действия после установки системы
На новом сервере произвёл:
- обновление системы `yum update`
- перезагрузка `reboot`
- подключение репо postgresql

Памятуя о версии на исходном сервере, производим установку конкретной версии posgresql:
```bash
# yum install postgresql11-server-11.4-1PGDG.rhel7.x86_64
...
=======================================================================================
 Package                        Arch       Version            Repository     Size
=======================================================================================
Installing:
 postgresql11-server            x86_64     11.4-1PGDG.rhel7   postgres11     4.7 M
Installing for dependencies:
 postgresql11                   x86_64     11.4-1PGDG.rhel7   postgres11     1.6 M
 postgresql11-libs              x86_64     11.4-1PGDG.rhel7   postgres11     361 k
...
```

Устанавливаем пакет `yum-plugin-versionlock` для возможности блокировки пакетов от апгрейда и выполняем блокировку пакетов `postgresql11*`:
```bash
# yum install yum-plugin-versionlock
# yum versionlock postgresql11*
Loaded plugins: product-id, rhnplugin, search-disabled-repos, versionlock
This system is receiving updates from RHN Classic or Red Hat Satellite.
Adding versionlock on: 0:postgresql11-libs-11.4-1PGDG.rhel7
Adding versionlock on: 0:postgresql11-server-11.4-1PGDG.rhel7
Adding versionlock on: 0:postgresql11-11.4-1PGDG.rhel7
versionlock added: 3
```

При необходимости, актуальный список залоченных пакетов смотрим здесь `cat /etc/yum/pluginconf.d/versionlock.list`.

### Монтирование большого раздела к /var/lib/pgsql
На новом сервер наблюдаем отдельный большой раздел 1.5TB подмонтрованный к `/data`. Создадим на нём каталог pgsql и прочее:
```bash
mkdir -p /data/pgsql/11/{data,backup}
chown postgres:postgres /data/pgsql -R
chmod 700 /data/pgsql -R
cp /var/lib/pgsql/.bash_profile /data/pgsql
```

Добавим в /etc/fstab следующую строку для автоматического биндинга `/data/pgsql` к `/var/lib/pgsql`:
```bash
/data/pgsql /var/lib/pgsql none defaults,bind 0 0
```

Важно!!! Если после последней строки в файле `/etc/fstab` не будет добавлена пустая строка, то сервер зависнет на этапе загрузки системы. Не помню причины этого явления, поэтому примите на веру, если не знали.

### Инициализация и запуск
```bash
/usr/pgsql-11/bin/postgresql-11-setup initdb
systemctl start postgresql-11
```

## Завершающие действия

### Копирование дампа на новый сервер
В нашем случае прямое ssh-соединение между серверами отсутствует, поэтому скопируем полученный дамп через форпост, который доступен из обоих сегментов сети.

Сейчас мы находимся на промежуточном сервере. Сначала подключаемся по ssh к одному из серверов, на каковом сервере открывается локальный порт 50000, который связывается с портом 22 другого сервера (через промежуточный сервер!):
```bash
# ssh -R 127.0.0.1:50000:10.0.0.2:22 root@10.0.0.1
[root@10.0.0.1]# _
```

Теперь, находясь на сервере 10.0.0.1 (что видно по приглашению командной строки) помним, что постучавшись на локальный порт `127.0.0.1:50000`, мы, на самом деле, стучимся на `10.0.0.2:22`, но не на прямую, а через промежуточный сервер.

Поэтому здесь же запускаем процесс копирования с докачкой и проверкой. Копируем дамп и файлы настроек:
```bash
rsync -avP -e "ssh -p50000" /var/lib/pgsql/11/backup/pgsql_dumpall_20200421-103341.bz2 127.0.0.1:/data/pgsql/11/backup/
rsync -vP -e "ssh -p50000" /var/lib/pgsql/11/data/* 127.0.0.1:/data/pgsql/11/backup/data
```

В результате этой команды, наш дамп, через промежуточный сервер, будет залит на новый сервер.

### Восстановление дампа на новом сервере
```bash
sudo su
su - postgres
lbzip2 -c -d /data/pgsql/11/backup/psql_dumpall_20200421-103341.bz2 | psql postgres
```

### Восстановление настроек экземпляра postresgql
```bash
cp /data/pgsql/11/backup/data/* /data/pgsql/11/data/
```

### Настройка атозапуска postgesql
```bash
systemctl enable postgresql-11
```

### Перезагрузка для проверки
```bash
reboot
```

Использованный материал:
[How to restrict yum to install or upgrade a package to a fixed specific package version?](https://access.redhat.com/solutions/98873)
[25.1. SQL Dump Chapter 25. Backup and Restore](https://www.postgresql.org/docs/11/backup-dump.html)
