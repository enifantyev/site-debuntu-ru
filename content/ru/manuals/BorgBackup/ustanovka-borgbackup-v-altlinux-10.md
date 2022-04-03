---
title: "12. Установка BorgBackup в AltLinux 10"
date: 2022-03-14
weight: 11
description: >
  Описание процесса установки утилиты для дедуплицированного бэкапа BorgBackup в AltLinux 10.
tags:
  - BorgBackup
  - Backup SoftWare
  - AltLinux 10
  - Altlinux
---

2022-03-14

### Установка borg 1.1.17
```
apt-get install borg
```

### Установка borg 1.2
```
$ sudo apt-get install python3-module-pkgconfig python3-module-setuptools \
python3-module-wheel python3-module-msgpack libssl-devel python3-dev \
libacl-devel libacl libssl-devel liblz4-devel libzstd-devel libxxhash-devel

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

```
pip install borgbackup
```

```
borg -V
borg 1.2.0
```
