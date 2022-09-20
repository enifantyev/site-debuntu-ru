---
title: "10. Установка BorgBackup в CentOS и Ubuntu"
date: 2022-03-11
weight: 10
description: >
  Описание процесса установки утилиты для дедуплицированного бэкапа BorgBackup в CentOS и Ubuntu.
tags:
  - BorgBackup
  - Backup SoftWare
  - CentOS
  - Ubuntu
slug: ustanovka-borgbackup-v-centos-i-ubuntu
aliases:
  - /manuals/borgbackup/ustanovka-i-nastroyka-borgbackup/
---

2022-03-11

## 1. Установка BorgBackup в CentOS
### 1.1. Добавление EPEL
На CentOS необходим репо EPEL.
```bash
$ sudo yum install epel-release
```

### 1.2. Установка BorgBackup в CentOS7
```bash
$ sudo yum install borg
...
Dependencies Resolved

=============================================================================
 Package                             Arch    Version       Repository  Size
=============================================================================
Installing:
 borgbackup                          x86_64  1.1.17-2.el7  epel7       1.0 M
Installing for dependencies:
 libb2                               x86_64  0.98.1-2.el7  epel7       20 k
 libzstd                             x86_64  1.5.2-1.el7   epel7       282 k
 python36-llfuse                     x86_64  1.0-2.el7     epel7       351 k
 python36-msgpack                    x86_64  0.5.6-5.el7   epel7       98 k
 python36-packaging                  noarch  16.8-6.el7    epel7       43 k
 python36-pyparsing                  noarch  2.4.0-1.el7   epel7       145 k
 python36-six                        noarch  1.14.0-3.el7  epel7       34 k
 xxhash-libs                         x86_64  0.8.1-1.el7   epel7       32 k

Transaction Summary
=============================================================================
Install  1 Package (+8 Dependent packages)

Total size: 2.0 M
Total download size
```

## 2. Установка BorgBackup в Ubuntu
На Ubuntu добавляем PPA:
```bash
sudo add-apt-repository ppa:costamagnagianfranco/borgbackup
```
