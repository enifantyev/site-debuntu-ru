---
title: "20. Создание бэкапа всей системы с помощью LVM и BorgBackup"
date: 2022-03-11
weight: 20
description: >
  Описание процесса создания LVM-снимка всей операционной системы и его архивирование в borg-репо.
tags:
  - BorgBackup
  - Backup SoftWare
  - LVM
slug: sozdaniye-bekapa-vsey-sistemy-s-pomoshchyu-borgbackup
---

2022-03-11

## Подготовительные мероприятия
Для создания LVM-снэпшотов требуется некоторое незанятое место на тех VG, для LV которых будут создаваться снэпшоты. В том случае, если на машине остановлены все сервисы, могущие создавать большой объём записываемых данных, то для создания снэпшотов много места не потребуется и достаточно, например, временно отключить swap и удалить соответствующий LV для освобождения пары Гигабайт под создание снэпшотов.

Возможно, будет правильней добавить к машине дополнительный диск достаточного объёма на постоянной основе, который будет использоваться не только для создания бэкапов, но и восстановления всей системы.

### Описание бэкапируемой системы
Дисковая подсистема стандартной бэкапируемой системы состоит из следующего:
- Два диска
  - Первый диск системный, где расположены отдельные разделы для:
    - `/dev/sda1` - `/boot`
    - `/dev/vg/root` - `/`
    - `/dev/vg/var` - `/var`
  - Второй диск дополнительный. Возможно, что он:
    - диск не разбит или разбит на один раздел. Пространство подключёно к LVM2 и именовано и подключено так `/dev/vg/data` - `/data`;
    - диск не разбит или разбит на один раздел. Пространство подключёно к LVM2 и именовано и подключено так `/dev/vgdata/data` - `/data`;
    - может быть организован по другому.

```bash
# parted /dev/sda unit mib print free
Model: VMware Virtual disk (scsi)
Disk /dev/sda: 51200MiB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags:

Number  Start    End       Size      Type     File system  Flags
 1      1,00MiB  1025MiB   1024MiB   primary  xfs          boot
 2      1025MiB  51200MiB  50175MiB  primary               lvm
```

### Добавление пространства в Volume Group
Если требуется бэкап LV-томов на одном VG, то разбивать дополнительный диск на разделы не требуется. Достаточно будет добавить в VG весь новый диск командой `vgextend /dev/vg /dev/sdb`.

В противном случае, когда восстанавливаемые тома находятся на разных VG, потребуется разбивка дополнительного диска на отдельные разделы и добавление этих новых PV в свои VG.

## Процесс резервного копирования
### Отключение работающих демонов
Останавливаем и выключаем главные сервисы, работающие на хосте, чтобы предотвратить автоматический запуск демонов после восстановления системы. Например:
```bash
ipactl stop
systemctl disable --now dirsrv@TEST4.LAN.service
systemctl disable --now pki-tomcatd
systemctl disable --now named
systemctl disable --now httpd
systemctl disable --now ipa-otpd
systemctl disable --now ipa-dnskeysyncd
```

В дополнение к остановке главных сервисов, работающих на машине, останавливаем второстепенные, которые могут вызвать переполнение на LV места выделенного под снэпшоты. Пример остановки дополнительных сервисов:
```bash
systemctl stop postfix
systemctl stop rsyslog
service auditd stop
```

### Создание временных LVM snapshot'ов
Помним, что в Volume Group должно быть достаточно свободного места для размещения снэпшотов.
```bash
lvcreate -L 500M -s -n snap_root /dev/vg/root
lvcreate -L 500M -s -n snap_var /dev/vg/var
```

Если требуется освободить немного места в VG, то можно попробовать освободить пару гигабайтов за счёт уменьшения, например `/dev/vg/data`, конечно, если на томе используется файловая система 'ext4':
```bash
/usr/sbin/lvreduce -L -2G -r /dev/vg/data
```

После завершения бэкапа вновь увеличиваем LV:
```bash
/usr/sbin/lvextend -l +100%FREE -r /dev/vg/data
```

### Монтирование снэпшотов + директорию `/boot`
При монтировании xfs используем опцию `nouuid`, поэтому проверяем тип файловой системы:
```bash
# Mounting root on /mnt
LV='/dev/vg/snap_root'
FSTYPE=$(df -T ${LV/snap_/} | egrep "ext4|xfs" | awk '{print $2}')
if [[ ${FSTYPE} = "xfs" ]]; then
  mount -o nouuid,ro ${LV} /mnt
elif [[ ${FSTYPE} = "ext4" ]]; then
  mount -o ro ${LV} /mnt
fi

# Mounting /var on /mnt/var
LV='/dev/vg/snap_var'
FSTYPE=$(df -T ${LV/snap_/} | egrep "ext4|xfs" | awk '{print $2}')
if [[ ${FSTYPE} = "xfs" ]]; then
  mount -o nouuid,ro ${LV} /mnt/var
elif [[ ${FSTYPE} = "ext4" ]]; then
  mount -o ro ${LV} /mnt/var
fi

# Mounting /boot on /mnt/boot
mount -o bind,ro /boot /mnt/boot
```

### Проверка монтирования
```bash
mount | grep '/mnt'
```
Пример вывода:
```
/dev/mapper/vg-snap_root on /mnt type ext4 (rw,relatime,seclabel)
/dev/mapper/vg-snap_var on /mnt/var type ext4 (rw,relatime,seclabel)
/dev/sda1 on /mnt/boot type ext4 (rw,relatime,seclabel)
```

### Резервное копирование
Устанавливаем переменные, чтобы не вводить параметры дважды:
```bash
export BORG_RSH="ssh -i ~/.ssh/borg1"
export BORG_REPO="_borg@192.168.50.10:home-ipa"
```

Выполняем бэкап со сжатием передовым компрессором [Zstandard](https://ru.wikipedia.org/wiki/Zstandard):
```bash
borg create -C zstd --stats --progress ::full_{hostname}_{now} /mnt
```

### Удаление временных снэпшотов
```bash
umount -R /mnt
lvremove -y /dev/vg/snap_var
lvremove -y /dev/vg/snap_root
```

### Запуск остановленных демонов
Запускаем ранее остановленные локальные демоны:
```bash
systemctl start postfix
systemctl start rsyslog
service auditd start
```
