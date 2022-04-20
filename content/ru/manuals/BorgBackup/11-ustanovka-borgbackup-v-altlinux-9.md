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
slug: ustanovka-i-nastroyka-borgbackup
---

2022-03-31

### Установка из APT
```bash
apt-get install borg
```

```bash
$ borg -V
borg 1.1.10
```

### Установка из pip
```bash
apt-get install python3-module-pkgconfig python3-module-setuptools \
python3-module-wheel python3-module-msgpack libssl-devel python3-dev \
libacl-devel libacl libssl-devel liblz4-devel libzstd-devel \
libxxhash-devel gcc-c++ python3-module-setuptools_scm
```

```bash
python3 -m pip install --upgrade pip
```

```bash
python3 -m pip install borgbackup
```

```bash
$ borg -V
borg 1.1.17
```
