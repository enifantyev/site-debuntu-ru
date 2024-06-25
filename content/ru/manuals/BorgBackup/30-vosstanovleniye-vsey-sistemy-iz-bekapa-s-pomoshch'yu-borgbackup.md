---
title: "30. Восстановление всей системы из бэкапа с помощью LVM и BorgBackup"
date: 2022-03-18
weight: 30
description: >
  Описание процесса восстановления всей операционной системы из borg-репо в пустой LV-снимок с его последующим объединением с LV-источником.
tags:
  - BorgBackup
  - Backup SoftWare
  - LVM
slug: vosstanovleniye-vsey-sistemy-iz-bekapa-s-pomoshchyu-borgbackup
---

2022-03-11&nbsp;&ndash;&nbsp;2022-03-18

## Подготовительные мероприятия
В минимальной комплектации необходимо восстановить системные разделы `/`, `/boot`, `/var`. В максимальной комплектации потребуется восстановить ранее забэкапированные `/data` и прочее.

В любом случае, **для распаковки этих данных потребуется достаточное незанятое место на Volume Group'ах**, несущих восстанавливаемые Logical Volume'ы. В простейшем случае для этого необходимо добавить к машине дополнительный диск вместимостью перекрывающим объём восстанавливаемых данных.

### Добавление пространства в Volume Group
Если требуется восстанавливать только один VG, то разбивать дополнительный диск на разделы не требуется. Достаточно будет добавить в VG весь новый диск командой `vgextend vg /dev/sdb`.

В противном случае, когда восстанавливаемые тома находятся на разных VG, потребуется разбивка дополнительного диска на отдельные разделы и добавление этих новых PV в свои VG.

## Процесс восстановления
### Узнаём требуемый размер диска для восстановления архива
Устанавливаем переменные, чтобы не вводить параметры дважды:
```bash
export BORG_RSH="ssh -i ~/.ssh/borg1"
export BORG_REPO="_borg@192.168.50.10:home-ipa"
```

Выбираем нужный архив из списка. Пример списка:
```bash
borg list
...
full_img211202_2022-03-11T10:54:32   Fri, 2022-03-11 10:54:35 [f681b040b6f7579e424cc328207e44aac50205ffaaa6d6923417ec598807b4b4]
```

Узнаём размер распакованных файлов:
```bash
Archive name: full_img211202_2022-03-11T10:54:32
Archive fingerprint: f681b040b6f7579e424cc328207e44aac50205ffaaa6d6923417ec598807b4b4
Comment:
Hostname: img211202
Username: root
Time (start): Mon, 2022-03-11 10:54:35
Time (end): Mon, 2022-04-11 10:55:25
Duration: 50.01 seconds
Number of files: 68781
Command line: /root/.local/bin/borg create -C zstd --stats --progress '::full_{hostname}_{now}' /mnt/sysroot
Utilization of maximum supported archive size: 0%
------------------------------------------------------------------------------
                       Original size      Compressed size    Deduplicated size
This archive:                5.08 GB              1.82 GB            101.91 MB
All archives:              192.91 GB            115.34 GB              9.20 GB

                       Unique chunks         Total chunks
Chunk index:                  165331              2033810
```

Видим, что потребуется создать снэпшот размером не менее 5.08 GB.

### Отключение работающих демонов
В дополнение к остановке главных сервисов, работающих на машине, останавливаем второстепенные, которые могут вызвать переполнение на LV места выделенного под снэпшоты. Пример остановки дополнительных сервисов:
```bash
systemctl stop postfix
systemctl stop rsyslog
service auditd stop
```

### Создание LVM snapshot
Помним, что в Volume Group должно быть достаточно свободного места для размещения данных из бэкапов. В отличие от процесса создания бэкапа, в этом случае нужно много места, чтобы уместить сюда весь бэкап.
```bash
lvcreate -L 6G -s -n snap_root /dev/vg/root
lvcreate -L 6G -s -n snap_var /dev/vg/var
```

### Монтирование снэпшотов + директорию `/boot`
При монтировании xfs используем опцию `nouuid`:
```bash
# Mounting root on /mnt/sysroot
LV='/dev/vg/snap_root'
SYSROOT='/mnt/sysroot'
mkdir -p ${SYSROOT}
FSTYPE=$(df -T ${LV/snap_/} | egrep "ext4|xfs" | awk '{print $2}')
if [[ ${FSTYPE} = "xfs" ]]; then
  mount -o nouuid ${LV} ${SYSROOT}
elif [[ ${FSTYPE} = "ext4" ]]; then
  mount ${LV} ${SYSROOT}
fi

# Mounting /var on /mnt/sysroot/var
LV='/dev/vg/snap_var'
FSTYPE=$(df -T ${LV/snap_/} | egrep "ext4|xfs" | awk '{print $2}')
if [[ ${FSTYPE} = "xfs" ]]; then
  mount -o nouuid ${LV} ${SYSROOT}/var
elif [[ ${FSTYPE} = "ext4" ]]; then
  mount ${LV} ${SYSROOT}/var
fi

# Mounting /boot on /mnt/sysroot/boot
mount -o bind /boot ${SYSROOT}/boot
```

### Проверка монтирования
```bash
mount | grep '/mnt'
```
Пример вывода:
```
/dev/mapper/vg-snap_root on /mnt/sysroot type ext4 (rw,relatime,seclabel)
/dev/mapper/vg-snap_var on /mnt/sysroot/var type ext4 (rw,relatime,seclabel)
/dev/sda1 on /mnt/sysroot/boot type ext4 (rw,relatime,seclabel)
```

### Удаление содержимого снэпшотов и `/boot`
Удаляем всё содержимое снэпшотов и каталога `/boot` с проверкой:
```bash
rm -ri ${SYSROOT}/*
tree -h ${SYSROOT}
```

Результат:
```
/mnt/sysroot
└── [4.0K]  boot

1 directory, 0 files
```

### Восстановление бэкапа в пространство снэпшотов и `/boot`
Переходим в корень и восстанавливаем архив `full_img211202_2022-03-11T10:54:32`, который заполнит файлами готовый для приёма каталог `/mnt/sysroot`:
```bash
cd /
borg extract --list --progress --numeric-owner \
  ::full_img211202_2022-03-11T12:53:10
```

## Исправление загрузки
### chroot для исправления загрузки
```bash
mount -t proc /proc ${SYSROOT}/proc/
mount -t sysfs /sys ${SYSROOT}/sys/
mount --bind /dev ${SYSROOT}/dev/
mount --bind /run ${SYSROOT}/run
chroot ${SYSROOT}
```
## Обновление GRUB
В некоторых случаях, например, если в ОС были обновлены пакеты, отвечающие за загрузку, то потребуется обновить GRUB.

Общая схема обновления GRUB такая:
1. Генерируем новый `/boot/grub/grub.cfg`. Это необходимо сделать, так как восстанавливаются только файлы, но не UUID'ы разделов на диске.
2. Так как песочница, в которой производилась генерация `grub.cfg`, находится на LV с именами `snap_root` и `snap_var`, то исправляем в сгенерированном `/boot/grub/grub.cfg` названия разделов.
3. Переустанавливаем GRUB в MBR (плюс ещё несколько секторов).
    ```bash
    if [ -e /usr/sbin/grub-mkconfig ]; then
      grub-mkconfig -o /boot/grub/grub.cfg
      sed -i 's/vg-snap_root/vg-root/g' /boot/grub/grub.cfg
      grub-install /dev/sda
    elif [ -e /usr/sbin/grub2-mkconfig ]; then
      grub2-mkconfig -o /boot/grub2/grub.cfg
      sed -i 's/vg-snap_root/vg-root/g' /boot/grub2/grub.cfg
      grub2-install /dev/sda
    else
      echo "Houston, we have a problem."
    fi
    ```

Эталонный вывод после `grub-install`:
```
Installing for i386-pc platform.
Installation finished. No error reported.
```

> ⚠️ Если произошли ошибки, то машина с большой вероятностью загрузится в 'grub recovery'-режиме.

### Обновление initramfs
При восстановлении старого снимка этой машины с предыдущей версией Linux-ядра, возможно потребуется указывать версию ядра, для которого генерируется initramfs.

Версию текущего ядра можно узнать через команду `uname -r`.

Версии Linux-ядер в восстанавливаемом снимке видны в каталоге `/boot`.

#### CentOS 7, RedOS 7.2
```bash
dracut -f -v
```

```bash
dracut -f -v /boot/initramfs-4.19.204-2.el7.x86_64.img 4.19.204-2.el7.x86_64
```

#### AltLinux 10
```bash
make-initrd
```
или при необходимости указания версии ядра:
```bash
make-initrd --kernel='5.10.82-std-def-alt1'
```

### Проверяем сетевые настройки
Предполагается, что системный администратор умеет исправить сетевые настройки должным образом.

### SELinux relabel
Если используется SELinux, тогда создаём файл `/.autorelabel` для автоматической перемаркировки всех файлов системы:
```bash
touch /.autorelabel
```

### Выход из песочницы
```bash
exit
```

## Заключительные операции
### Объединение снэпшотов
```bash
umount -R ${SYSROOT}
lvconvert --merge /dev/vg/snap_var
lvconvert --merge /dev/vg/snap_root
```
stdout пример:
```bash
  Delaying merge since origin is open.
  Merging of snapshot vg/snap_var will occur on next activation of vg/var.
```

Stdout последних команд говорит о том, что объединение произойдёт на следующей активации lv-томов, то есть, в нашем случае, после перезагрузки сервера.

### reboot
```bash
sync
sleep 10
reboot -f
```
