---
title: "11. Установка BorgBackup в AltLinux 9"
date: 2022-03-31
weight: 11
description: >
  Описание процесса установки утилиты для дедуплицированного бэкапа BorgBackup в AltLinux 9.
tags:
  - BorgBackup
  - Backup SoftWare
  - AltLinux 9
  - AltLinux
---

2022-03-31

### Установка из APT
```
apt-get install borg
```

```
$ borg -V
borg 1.1.10
```

### Установка из pip
```
apt-get install python3-module-pkgconfig python3-module-setuptools \
python3-module-wheel python3-module-msgpack libssl-devel python3-dev \
libacl-devel libacl libssl-devel liblz4-devel libzstd-devel \
libxxhash-devel gcc-c++ python3-module-setuptools_scm
```

```
python3 -m pip install --upgrade pip
```

```
python3 -m pip install borgbackup
```

```
$ borg -V
borg 1.1.17
```
