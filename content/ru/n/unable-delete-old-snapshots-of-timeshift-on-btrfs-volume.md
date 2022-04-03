---
title: "Unable delete old snapshots of timeshift on btrfs volume"
date: 2019-03-10
weight: 10
description: >
  Проблемы с удалением старых снимков на томе с файловой системой btrfs.
tags:
  - Btrfs
---

2019-03-10

#### Synopsis
На корневом btrfs-томе включён timeshift для периодического снятия снимков файловой системы. Замечено, что на томе осталось мало свободного места. При попытке удаления старых timeshift-снимков некоторые снимки не были удалены с возникновением ошибки `cannot delete '*SnapshotName*'': Directory not empty`.

#### Solution

- Так-как интересующие нас timeshift-снимки является подтомом корневого btrfs-тома, то для удобства работы с ними создаём отдельную директорию:
```
# mkdir /mnt/btrfs-root
```
- К этой директории подключаем корень интересующего нас раздела:
```
# mount /dev/sdb3 /mnt/btrfs-root
```
- Просматриваем список подтомов корневого тома:
```
# cd /mnt/btrfs-root
# btrfs subvolume list /mnt/btrfs-root
ID 257 gen 97699 top level 5 path @
ID 411 gen 97572 top level 5 path timeshift-btrfs/snapshots/2019-03-01_16-00-01/@
ID 412 gen 90577 top level 411 path timeshift-btrfs/snapshots/2019-03-01_16-00-01/@/@
```
- Также можно посмотреть обычным способом:
```
# ls -l /mnt/btrfs-root/snapshots
2019-03-01_16-00-01/
2019-03-01_16-00-01/

```
- Удаляем сначала вложенный том интересующего нас подтома:
```
# btrfs subvolume delete timeshift-btrfs/snapshots/2019-03-01_16-00-01/@/@
```
- Удаляем основной подтом:
```
# btrfs subvolume delete timeshift-btrfs/snapshots/2019-03-01_16-00-01/@
```
