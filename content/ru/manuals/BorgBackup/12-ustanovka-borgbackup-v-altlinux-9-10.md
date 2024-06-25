---
title: "12. Установка BorgBackup в AltLinux 9/10"
date: 2022-03-14
weight: 12
description: >
  Описание процесса установки утилиты для дедуплицированного бэкапа BorgBackup в AltLinux 9/10.
tags:
  - BorgBackup
  - Backup SoftWare
  - AltLinux 10
  - AltLinux 9
  - Altlinux
slug: ustanovka-borgbackup-v-altlinux-9-10
alias:
  - /manuals/borgbackup/ustanovka-borgbackup-v-altlinux-10/
  - /manuals/borgbackup/ustanovka-borgbackup-v-altlinux-9/
---

2022-03-14

## Установка BorgBackup в AltLinux 10
### Установка borg 1.1.17
```bash
apt-get install borg
```

### Установка borg 1.2
```bash
$ sudo apt-get install python3-module-pkgconfig python3-module-setuptools \
python3-module-wheel python3-module-msgpack libssl-devel python3-dev \
libacl-devel libacl libssl-devel liblz4-devel libzstd-devel libxxhash-devel
```

srdout:
```
Reading Package Lists... Done
Building Dependency Tree... Done
libacl is already the newest version.
The following extra packages will be installed:
  libncurses-devel libtinfo-devel libxxhash rpm-build-python3 rpm-macros-python3 tests-for-installed-python3-pkgs
The following NEW packages will be installed:
  libacl-devel liblz4-devel libncurses-devel libssl-devel libtinfo-devel libxxhash libxxhash-devel libzstd-devel
  python3-dev python3-module-msgpack python3-module-pkgconfig python3-module-setuptools python3-module-wheel
  rpm-build-python3 rpm-macros-python3 tests-for-installed-python3-pkgs
0 upgraded, 16 newly installed, 0 removed and 0 not upgraded.
Need to get 2010kB of archives.
After unpacking 8373kB of additional disk space will be used.
Do you want to continue? [Y/n]
```

```bash
pip install borgbackup
```

```bash
borg -V
borg 1.2.0
```

## Установка BorgBackup в AltLinux 9
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
python3 -m pip install --user --upgrade pip
```

```bash
python3 -m pip install --user borgbackup
```

```bash
$ borg -V
borg 1.1.17
```
