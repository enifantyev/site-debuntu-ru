---
title: "30. Восстановление всей системы из бэкапа с помощью BorgBackup"
date: 2022-03-18
weight: 30
description: >
  Описание процесса восстановления всей операционной системы из borg-репо в пустой LV-снимок с его последующим объединением с LV-источником.
tags:
  - BorgBackup
  - Backup SoftWare
  - LVM
---

2022-03-11
2022-03-18

## Подготовительные мероприятия
В минимальной комплектации необходимо восстановить системные разделы `/`, `/boot`, `/var`. В максимальной комплектации потребуется восстановить ранее забэкапированные `/data` и прочее.

В любом случае, **для распаковки этих данных потребуется достаточное незанятое место на Volume Group'ах**, несущих восстанавливаемые Logical Volume'ы. В простейшем случае для этого необходимо добавить к машине дополнительный диск вместимостью перекрывающим объём восстанавливаемых данных.

### Добавление пространства в Volume Group
Если требуется восстанавливать только один VG, то разбивать дополнительный диск на разделы не требуется. Достаточно будет добавить в VG весь новый диск командой `vgextend /dev/vg /dev/sdb`.

В противном случае, когда восстанавливаемые тома находятся на разных VG, потребуется разбивка дополнительного диска на отдельные разделы и добавление этих новых PV в свои VG.

## Процесс восстановления
### Отключение работающих демонов
В дополнение к остановке главных сервисов, работающих на машине, останавливаем второстепенные, которые могут вызвать переполнение на LV места выделенного под снэпшоты. Пример остановки дополнительных сервисов:
```
systemctl stop postfix
systemctl stop rsyslog
service auditd stop
```

### Создание LVM snapshot
Помним, что в Volume Group должно быть достаточно свободного места для размещения данных из бэкапов. В отличие от процесса создания бэкапа, в этом случае нужно много места, чтобы уместить сюда весь бэкап.
```
lvcreate -L 4G -s -n snap_root /dev/vg/root
lvcreate -L 4G -s -n snap_var /dev/vg/var
```

### Монтирование снэпшотов + директорию `/boot`
При монтировании xfs используем опцию `nouuid`:
```
# Mounting root on /mnt
LV='/dev/vg/snap_root'
FSTYPE=$(df -T ${LV/snap_/} | egrep "ext4|xfs" | awk '{print $2}')
if [[ ${FSTYPE} = "xfs" ]]; then
  mount -o nouuid ${LV} /mnt
elif [[ ${FSTYPE} = "ext4" ]]; then
  mount ${LV} /mnt
fi

# Mounting /var on /mnt/var
LV='/dev/vg/snap_var'
FSTYPE=$(df -T ${LV/snap_/} | egrep "ext4|xfs" | awk '{print $2}')
if [[ ${FSTYPE} = "xfs" ]]; then
  mount -o nouuid ${LV} /mnt/var
elif [[ ${FSTYPE} = "ext4" ]]; then
  mount ${LV} /mnt/var
fi

# Mounting /boot on /mnt/boot
mount -o bind /boot /mnt/boot
```

### Проверка монтирования
```
mount | grep '/mnt'
```
Пример вывода:
```
/dev/mapper/vg-snap_root on /mnt type ext4 (rw,relatime,seclabel)
/dev/mapper/vg-snap_var on /mnt/var type ext4 (rw,relatime,seclabel)
/dev/sda1 on /mnt/boot type ext4 (rw,relatime,seclabel)
```

### Удаление содержимого снэпшотов и `/boot`
Удаляем всё содержимое снэпшотов и каталога `/boot` с проверкой:
```
rm -rf /mnt/*
du /mnt
```

Результат:
```
0	/mnt/boot
0	/mnt/var
0	/mnt
```

### Восстановление бэкапа в пространство снэпшотов и `/boot`
Устанавливаем переменные, чтобы не вводить параметры дважды:
```
export BORG_RSH="ssh -i ~/.ssh/borg1"
export BORG_REPO="_borg@192.168.50.10:home-ipa"
```

Выбираем нужный архив из списка. Пример списка:
```
borg list
...
full_img211202_2022-03-11T10:54:32   Fri, 2022-03-11 10:54:35 [f681b040b6f7579e424cc328207e44aac50205ffaaa6d6923417ec598807b4b4]
```

Переходим в корень и восстанавливаем архив `full_img211202_2022-03-11T10:54:32`, который заполнит файлами готовый для приёма каталог `/mnt`:
```
cd /
borg extract --list --progress --numeric-owner \
  ::full_img211202_2022-03-11T12:53:10
```

## Исправление загрузки
### chroot для исправления загрузки
```
mount -t proc /proc /mnt/proc/
mount -t sysfs /sys /mnt/sys/
mount --bind /dev /mnt/dev/
mount --bind /run /mnt/run
chroot /mnt
```

### Обновление initramfs
При восстановлении старого снимка этой машины с предыдущей версией Linux-ядра, возможно потребуется указывать версию ядра, для которого генерируется initramfs.

Версию текущего ядра можно узнать через команду `uname -r`.

Версии Linux-ядер в восстанавливаемом снимке видны в каталоге `/boot`.

#### CentOS 7, RedOS 7.2
```
dracut -f -v
```

```
dracut -f -v /boot/initramfs-4.19.204-2.el7.x86_64.img 4.19.204-2.el7.x86_64
```

#### AltLinux 10
```
make-initrd
```
или при необходимости указания версии ядра:
```
make-initrd --kernel='5.10.82-std-def-alt1'
```

### Проверяем сетевые настройки
Предполагается, что системный администратор умеет исправить сетевые настройки должным образом.

### Выход из песочницы
```
exit
```

## Заключительные операции
### Объединение снэпшотов
```
umount -R /mnt
lvconvert --merge /dev/vg/snap_var
lvconvert --merge /dev/vg/snap_root
```
stdout пример:
```
  Delaying merge since origin is open.
  Merging of snapshot vg/snap_var will occur on next activation of vg/var.
```

Stdout последних команд говорит о том, что объединение произойдёт на следующей активации lv-томов, то есть, в нашем случае, после перезагрузки сервера.

### reboot
```
sync
sleep 10
reboot -f
```
