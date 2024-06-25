---
title: "Обновление Nexus Repository Manager"
date: 2020-05-07
weight: 10
description: >
  Краткое описание процесса обновления Nexus Repository Manager
tags:
  - Nexus Repository Manager
---

2020-05-07

[Upgrading Nexus Repository Manager 3](https://support.sonatype.com/hc/en-us/articles/115000350007)

### Краткое описание
Nexus работает на java и почти не зависит от хоста.

Nexus состоит из двух каталогов:
1. The application directory `nexus-3.16.1-02`.
2. The data directory `sonatype-work`.
```
├── nexus-3.16.1-02
├── sonatype-work
```

И не забываем о запуске вместе с ОС. На данный момент видно, что загрузка организована через ссылку `/etc/init.d/nexus`, которая указывает на `/data/nexus-3.16.1-02/bin/nexus`.

### Подготовка к обновлению
Скачиваем, проверяем хэш-сумму, и распаковываем новый nexus:
```
$ cd tmp
$ wget https://download.sonatype.com/nexus/3/latest-unix.tar.gz
$ wget https://download.sonatype.com/nexus/3/latest-unix.tar.gz.sha1
$ sha1sum latest-unix.tar.gz
8298bd6db3b0f4d083942343a89f47cae7d791de latest-unix.tar.gz
$ cat latest-unix.tar.gz.sha1
8298bd6db3b0f4d083942343a89f47cae7d791de
$ tar -xvf latest-unix.tar.gz
$ sudo chown nexus:nexus nexus-3.23.0-03 -R
```

Переносим новый экземпляр Nexus в тот же каталог, где находится работающий экземпляр старого Nexus, чтобы было так:
```
.
├── nexus-3.16.1-02
├── nexus-3.23.0-03
├── sonatype-work
```

Останавливаем работающий Nexus.
```
$ sudo /etc/init.d/nexus stop
```

Для быстрого восстановления в случае неудачи, делаем копию "The data directory `sonatype-work`":
```
$ rsync -av sonatype-work tmp/
```

Изменяем ссылку `/etc/init.d/nexus`, чтобы она указывала на `/data/nexus-3.23.0-03/bin/nexus`.
```
# rm /etc/init.d/nexus
# ln -s /data/nexus-3.23.0-03/bin/nexus /etc/init.d/
```

В файле `/data/nexus-3.23.0-03/bin/nexus.rc` указываем пользователя под которым будет работать Nexus:
```
run_as_user="nexus"
```

Сравнил `nexus-3.16.1-02/bin/nexus.vmoptions` и `nexus-3.23.0-03/bin/nexus.vmoptions`. Убедившись, что на машине не так много оперативной памяти, изменил параметры в новом `nexus.vmoptions` в соответствии с параметрами из старого `nexus.vmoptions`.

### Обновление
Запускаем новую версию Nexus.
```
# /etc/init.d/nexus start
```

На этом процесс обновления закончен. Необходимо зайти в GUI и проверить статус и чего-нибудь ещё.

### Отладка
Для отладки запуска приложения, я запускал Nexus из командной строки:
```
$ ./nexus run
```
