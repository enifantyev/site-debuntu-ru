---
title: "13. Установка BorgBackup в RedOS 7.2"
date: 2022-03-17
weight: 13
description: >
  Описание процесса установки утилиты для дедуплицированного бэкапа BorgBackup в RedOS 7.2.
tags:
  - BorgBackup
  - Backup SoftWare
  - RedOS 7.2
  - RedOS
slug: ustanovka-borgbackup-v-redos-7.2
---

2022-03-17

## Установка пакетов, необходимых для сборки borgbackup
```bash
yum -y install openssl-devel python3-devel libacl-devel libacl lz4-devel \
libzstd-devel libxxhash-devel gcc-c++ fuse-devel
```

## Обновляем pip
```bash
python3 -m pip install -U pip
```

## Добавляем borgbackup в /roo/.local/bin
### Пакеты pkgconfig setuptools wheel msgpack
```bash
python3 -m pip install --user pkgconfig setuptools setuptools-scm wheel msgpack
```
stdout:
```
WARNING: Running pip install with root privileges is generally not a good idea. Try `__main__.py install --user` instead.
Collecting pip
  Downloading https://files.pythonhosted.org/packages/a4/6d/6463d49a933f547439d6b5b98b46af8742cc03ae83543e4d7688c2420f8b/pip-21.3.1-py3-none-any.whl (1.7MB)
    100% |████████████████████████████████| 1.7MB 814kB/s
Installing collected packages: pip
  Found existing installation: pip 9.0.3
    Uninstalling pip-9.0.3:
      Successfully uninstalled pip-9.0.3
Successfully installed pip-21.3.1
[root@home-ipa01 ~]# python3 -m pip install --user pkgconfig setuptools wheel msgpack llfuse
Collecting pkgconfig
  Downloading pkgconfig-1.5.5-py3-none-any.whl (6.7 kB)
Requirement already satisfied: setuptools in /usr/lib/python3.6/site-packages (39.2.0)
Collecting wheel
  Downloading wheel-0.37.1-py2.py3-none-any.whl (35 kB)
Collecting msgpack
  Downloading msgpack-1.0.3-cp36-cp36m-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (299 kB)
     |████████████████████████████████| 299 kB 1.6 MB/s            
Installing collected packages: wheel, pkgconfig, msgpack
  WARNING: The script wheel is installed in '/root/.local/bin' which is not on PATH.
  Consider adding this directory to PATH or, if you prefer to suppress this warning, use --no-warn-script-location.
Successfully installed msgpack-1.0.3 pkgconfig-1.5.5 wheel-0.37.1
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv
```

### Установка утилиты borgbackup
```bash
python3 -m pip install --user borgbackup
```
stdout:
```
Collecting borgbackup
  Downloading borgbackup-1.1.17.tar.gz (3.8 MB)
     |████████████████████████████████| 3.8 MB 59 kB/s             
  Preparing metadata (setup.py) ... done
Collecting packaging
  Downloading packaging-21.3-py3-none-any.whl (40 kB)
     |████████████████████████████████| 40 kB 4.0 MB/s            
Collecting pyparsing!=3.0.5,>=2.0.2
  Downloading pyparsing-3.0.7-py3-none-any.whl (98 kB)
     |████████████████████████████████| 98 kB 4.4 MB/s            
Building wheels for collected packages: borgbackup
  Building wheel for borgbackup (setup.py) ... done
  Created wheel for borgbackup: filename=borgbackup-1.1.17-cp36-cp36m-linux_x86_64.whl size=2173525 sha256=74d5e8b324dbae08ff8db8e30174eae00d66b6eab3ea3490d9e30fe496e41f26
  Stored in directory: /root/.cache/pip/wheels/e5/f0/fb/7226f7782c8d1ffefb17b8e351006868ea3feb28ac7175223f
Successfully built borgbackup
Installing collected packages: pyparsing, packaging, borgbackup
Successfully installed borgbackup-1.1.17 packaging-21.3 pyparsing-3.0.7
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv
```
