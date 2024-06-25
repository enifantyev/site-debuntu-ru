---
title: "40. Установка нового ядра в AltLinux 10 для проверки правильности восстановления машины с помощью BorgBackup"
date: 2022-03-18
weight: 40
description: >
  Описание способа установки нового Linux-ядра в AltLinux-машину после снятия снимка. С последующей попыткой возврата машины к ранее созданному снимку с помощью связки LVM и BorgBackup.
tags:
  - BorgBackup
  - Backup SoftWare
  - LVM
  - AltLinux 10
  - AltLinux
  - Linux Kernel
slug: ustanovka-novogo-yadra-v-altlinux-10-dlya-proverki-vosstanovleniya-mashiny-s-pomoshchyu-borgbackup
---

[https://help.72to.ru/projects/alt-linux/wiki/Откат_на_старую_версию_ядра](https://help.72to.ru/projects/alt-linux/wiki/%D0%9E%D1%82%D0%BA%D0%B0%D1%82_%D0%BD%D0%B0_%D1%81%D1%82%D0%B0%D1%80%D1%83%D1%8E_%D0%B2%D0%B5%D1%80%D1%81%D0%B8%D1%8E_%D1%8F%D0%B4%D1%80%D0%B0)

```bash
$ uname -r
5.10.82-std-def-alt1
```

```bash
apt-repo add  http://ftp.altlinux.ru/pub/distributions/archive/p10/task/archive/_289/295939/ x86_64 classic

apt-repo add  http://ftp.altlinux.ru/pub/distributions/archive/p10/task/archive/_289/295939/ noarch classic
```

```bash
$ sudo apt-cache show kernel-image-std-def

Package kernel-image-std-def is a virtual package provided by:
  kernel-image-std-def#2:5.10.82-alt1:p10+290646.100.3.1@1638544255 2:5.10.82-alt1:p10+290646.100.3.1@1638544255
  kernel-image-std-def#2:5.10.102-alt1:p10+295939.100.1.1@1645810219 2:5.10.102-alt1:p10+295939.100.1.1@1645810219
You should explicitly select one to show.
E: Package kernel-image-std-def is a virtual package with multiple providers.
```

```bash
update-kernel -t std-def -r 2:5.10.102-alt1:p10+295939.100.1.1@1645810219
```

```bash
reboot
```

```bash
$ uname -r
5.10.102-std-def-alt1
```

```bash
remove-old-kernels
```
