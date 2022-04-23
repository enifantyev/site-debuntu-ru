---
title: "LVM2 — создание, перемещение и удаление логических томов"
date: 2018-09-03
weight: 10
description: >
  LVM2 — создание, перемещение и удаление логических томов
tags:
  - LVM
  - LVM2
---

2018-09-03

### Создание Группы Томов (Volume Group)
Для имени Группы Томов (VG) на одиночном сервере я обычно выбираю значение "vg". Потом, при желании, его легко можно переименовать в любой момент. В Группу Томов можно добавить любое блочное устройство.

После добавления устройства в VG:
```
# vgcreate /dev/vg /dev/sde /dev/sdd1 /dev/sdd2 /dev/md10 /dev/mapper/pv5
```
можно получить информацию о VG:
```
# vgdisplay /dev/vg
```
переименовать VG:
```
# vgrename /dev/vg /dev/vg0
```
получить информацию о конкретном PV:
```
# pvdisplay /dev/md10
```

### Создание Логического Тома (Logical Volume)
Создать LV с именем /dev/vg/dock в любом свободном месте:
```
lvcreate -ndock -L20G /dev/vg
```
Создать LV, заняв всё свободное место (100% свободных экстентов) на группе томов /dev/vg:
```
lvcreate -ndock -l100%FREE /dev/vg
```

### Сбор информации о физическом размещении LV на физических томах (PV)
Размещение логических томов (LV) на физических томах (PV) выдаёт команда:
```
# lvs -a -o +devices
```

<details><p>

```
LV             VG        Attr       LSize    Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert Devices
  allegrograph0  vg        -wi-ao----    5,00g                                                     /dev/mapper/pv10(117096)
  arangodb0      vg        -wi-ao----    5,00g                                                     /dev/mapper/pv10(114536)
  orientdb0      vg        -wi-ao----    5,00g                                                     /dev/mapper/pv10(115816)
  tmp            vg        -wi-ao----  100,00g                                                     /dev/mapper/pv10(68728)
  vol            vg        -wi-ao----  500,00g                                                     /dev/mapper/pv10(59999)
  dock           vg        -wi-ao----   10,00g                                                     /dev/mapper/pv2(28160)
  mariadb0       vg        -wi-ao----    5,00g                                                     /dev/mapper/pv2(25600)
  mongodb0       vg        -wi-ao----    5,00g                                                     /dev/mapper/pv12(85601)
  nextcloud-data vg        -wi-ao----  100,00g                                                     /dev/mapper/pv2(0)
  nextcloud-www  vg        -wi-ao----    5,00g                                                     /dev/mapper/pv2(26880)
  vol            vg        -wi-ao----  500,00g                                                     /dev/mapper/pv5(0)
  vol            vg        -wi-ao----  500,00g                                                     /dev/mapper/pv12(93281)
```
</p></details>

### Перетаскивание части или всего LV
Перенос экстентов указанного LV на другой PV:
```
# pvmove -n docker /dev/mapper/pv13 /dev/mapper/pv11
```
Если экстенты LV находятся в разных местах, то посмотреть карту размещения можно с помощью команды:
```
# lvdisplay -m /dev/vg/vol
```

<details><p>

```
File descriptor 7 (pipe:[274631]) leaked on lvdisplay invocation. Parent PID 22239: bash
  --- Logical volume ---
  LV Path                /dev/vg/vol
  LV Name                vol
  VG Name                vg
  LV UUID                99e01850-fd35-11e8-b1b6-13207852fa16
  LV Write Access        read/write
  LV Creation host, time srv0, 2016-06-10 13:13:07 +0000
  LV Status              available
  # open                 1
  LV Size                500,00 GiB
  Current LE             128000
  Segments               3
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:11

  --- Segments ---
  Logical extents 0 to 113175:              # Номера экстентов внутри логического тома
    Type                linear
    Physical volume     /dev/mapper/pv11    # Физический том
    Physical extents    0 to 113175         # Номера физических экстентов занимаемых внутри физического тома

  Logical extents 113176 to 119270:
    Type                linear
    Physical volume     /dev/mapper/pv4
    Physical extents    0 to 6094

  Logical extents 119271 to 127999:
    Type                linear
    Physical volume     /dev/mapper/pv10
    Physical extents    59999 to 68727
```
</p></details>

Карту физического тома можно посмотреть через команду:
```
# pvdisplay -m /dev/mapper/pv2
```

<details><p>

```
File descriptor 7 (pipe:[274631]) leaked on pvdisplay invocation. Parent PID 22239: bash
  --- Physical volume ---
  PV Name               /dev/mapper/pv2
  VG Name               vg
  PV Size               <442,10 GiB / not usable 2,00 MiB
  Allocatable           yes
  PE Size               4,00 MiB
  Total PE              113176          # Общее количество экстентов на физическом томе
  Free PE               77336           # Количество свободных экстентов
  Allocated PE          35840           # Количество занятых экстентов
  PV UUID               L27diY-MiUX-AXcZ-3gKu-PIO5-Xqbq-fsjxku

  --- Physical Segments ---
  Physical extent 0 to 25599:                   # Физические экстенты с 0 по 25599
    Logical volume      /dev/vg/nextcloud-data  # заняты этим томом
    Logical extents     0 to 25599              # его логическими экстентами с 0 по 25599
  Physical extent 25600 to 26879:
    Logical volume      /dev/vg/mariadb0
    Logical extents     0 to 1279
  Physical extent 26880 to 28159:
    Logical volume      /dev/vg/nextcloud-www
    Logical extents     0 to 1279
  Physical extent 28160 to 30719:
    Logical volume      /dev/vg/dock
    Logical extents     0 to 2559               # 2560 экстентов*(4*1024)MiB = 10485760/1048576=10GiB
  Physical extent 30720 to 31999:
    Logical volume      /dev/vg/mongodb0
    Logical extents     0 to 1279
  Physical extent 32000 to 33279:
    Logical volume      /dev/vg/allegrograph0
    Logical extents     0 to 1279               # 1280 экстентов*(4*1024)MiB = 5242880/1048576=5GiB
  Physical extent 33280 to 34559:
    Logical volume      /dev/vg/arangodb0
    Logical extents     0 to 1279
  Physical extent 34560 to 35839:
    Logical volume      /dev/vg/orientdb0
    Logical extents     0 to 1279
  Physical extent 35840 to 113175:
    FREE
```
</p></details>

### Перемещение extent'ов с помощью pvmove
Перенос определённых PE между разными томами:
```
# pvmove -nstorj /dev/mapper/pv3:0-66312  /dev/mapper/pv13:35243-119200
```

Однажды я ошибся и указал диапазон источника переноса через двоеточие, вместо дефис: `/dev/mapper/pv3:0:66312`. В результате были перемещены два PE с адресами 0 и 66312.

Перенос определённых PE в пределах одного тома:
```
# pvmove --alloc anywhere /dev/mapper/pv13:35242-52862 /dev/mapper/pv13:17621-35241
```

В процессе моего изучения LVM2 пару дней посвятил команде pvmove. Мне было необходимо подвинуть большой фрагмент LV внутри PV на два экстента (PE), чтобы в начало PV влезли два экстента. Попробовал так:
```
# for (( i=101575; i>=0; i-- )); do pvmove --alloc anywhere /dev/mapper/pv12:$i /dev/mapper/pv12:$(($i+2)); done
```
Сработало (оптимальней было двигать сразу по два PE), но на одну подвижку тратилось секунд по 15, так как судя по ману, для каждого сеанса переноса, создаётся временный LV с именем pvmove, занимающий целевые PExtents. А после успешного подтверждения переноса, временный LV исчезает и занятые PE меняют принадлежность исходному LV. Видимо поэтому время выполнения этого скрипта на моём домашнем сервере вылилось бы в неделю или даже больше дней. Нажал Ctrl+c и перемещение прервалось. Один из PE остался в подвешенном состоянии во временном LV с именем "pvmove0", поэтому дал команду `pvmove --abort`, чтобы процесс корректно завершился. Вероятно, надёжней использовать опцию `--atomic`? (или что-то подобное, уточнить в мане), чтобы каждый сеанс переноса полностью завершился. Завершил задачу перемещением всего пакета на другой PV и обратным переносом со сдвигом на два PE. Всё заняло несколько часов.

### Удаление PV из VG
Шаг первый. Переместить все PE на другие PV:
``# pvmove /dev/mapper/pv11``
Шаг второй. Уменьшить VG, выкинув этот том из группы томов VG:
``# vgreduce /dev/vg /dev/mapper/pv11``
Шаг третий. Зачистить заголовок диска с информацией о PV командой:
``# pvremove /dev/mapper/pv11``


[pvmove (8) - Linux Man Pages](https://www.systutorials.com/docs/linux/man/8-pvmove/)
