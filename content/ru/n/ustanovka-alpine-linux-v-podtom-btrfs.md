---
title: "Установка Alpine Linux в подтом btrfs"
date: 2019-12-22
weight: 10
description: >
  Описание способа установки Alpine Linux в подтом BTRFS для возможности  откатов состояния системы с помощью предварительно сделанных снимков файловой системы.
tags:
  - Btrfs
  - Alpine Linux
  - syslinux
---

2019-12-22

### Установка Alpine Linux
Устанавливаем Alpine Linux 3.11 обычным порядком, но с указанием `lvmsys` для размещения системного раздела в lvm-томе.

После завершения процедуры `setup-alpine`, но до перезагрузки, будет удобно дать себе возможность войти в этот работающий экземпляр установки через ssh:
```
sed -i -e "s/^#PermitRootLogin.*$/PermitRootLogin yes/" /etc/ssh/sshd_config
/etc/init.d/sshd restart
```

Заходим в работающий экземпляр установки alpine linux через ssh под root'ом и продолжаем работать.

### Работаем с экземпляром через ssh
Устанавливаем необходимые пакеты, уменьшаем системный раздел и конвертируем его из `ext4` в `btrfs`:
```
apk update && apk add btrfs-progs btrfs-progs-extra lvm2 lvm2-extra device-mapper e2fsprogs e2fsprogs-extra util-linux
lvchange -ay /dev/vg0/lv_root
lvresize -r -L50G /dev/vg0/lv_root
btrfs-convert /dev/vg0/lv_root
```

Создаём снимок системного раздела в подтом `@` с целью использовать его в дальнейшем как системный. Удаляем корневую систему и монтируем подтом `@`, как актуальный системный раздел, в каталог `/mnt/`:
```
mount -t btrfs /dev/vg0/lv_root /mnt
btrfs subvolume delete /mnt/ext2_saved
btrfs subvolume snapshot /mnt /mnt/@
cd /mnt
find -maxdepth 1 ! -name @ -exec rm -rf {} \;
cd /
umount /mnt
mount -t btrfs -o subvol=@ /dev/vg0/lv_root /mnt
```

Подготавливаем песочницу и входим в неё через `chroot`:
```
mount /dev/sda1 /mnt/boot
mount --bind /proc /mnt/proc
mount --bind /dev /mnt/dev
mount --bind /sys /mnt/sys
mount --bind /run /mnt/run
chroot /mnt
```

### Работа в песочнице
Добавляем в будущую систему пакеты btrfs:
```
apk update
apk add btrfs-progs btrfs-progs-extra
```

Оказалось, что новая система загрузится и без корректировки этого файла, но для порядка изменяем строку в файле `/etc/fstab`:
```
-/dev/vg0/lv_root        /       ext4    rw,relatime 0 1
+/dev/vg0/lv_root        /       btrfs    subvol=@,rw,relatime 0 1
```

В файл `/etc/mkinitfs/mkinitfs.conf` добавляем запись о модуле `btrfs`:
```
-features="ata base ide scsi usb virtio ext4 lvm"
+features="ata base ide scsi usb virtio ext4 lvm btrfs"
```
и пересоздаём initrd командой:
```
mkinitfs
```

Изменяем строки в файле `/etc/update-extlinux.conf`, для подгрузки модуля btrfs и монтирования `/dev/vg0/lv_root` в `/sysroot` во время запуска initramfs:
```
-modules=sd-mod,usb-storage,ext4
+modules=sd-mod,usb-storage,ext4,btrfs
-default_kernel_opts="nomodeset quiet rootfstype=ext4"
+default_kernel_opts="nomodeset rootflags=subvol=@"
```
Пересоздаём `/boot/extlinux.cfg`:
```
update-extlinux
```

С настройкой загрузки системы в подтом `@` btrfs-раздела закончено. Сейчас можно настроить sshd, чтобы после перезагрузки была возможность сразу войти в новую систему через сеть, не дотрагиваясь до консоли. Минимально мы вновь можем изменить `PermitRootLogin`, но уже для будущей системы, пока работающей в песочнице:
```
sed -i -e "s/^#PermitRootLogin.*$/PermitRootLogin yes/" /etc/ssh/sshd_config
```

### Выход из песочницы и перезагрузка
```
exit
sync
reboot
```
