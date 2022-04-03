---
title: "cryptsetup короткая памятка"
date: 2018-12-10
weight: 10
description: >
  cryptsetup короткая памятка
tags:
  - cryptsetup
  - LUKS
  - dm-setup
---

2018-12-10

Создание крипто-тома с двумя ключами. В первом слоте используется ключевой файл, а в пятом парольная фраза:
```
# cryptsetup luksFormat /dev/md11 pv11.key
# cryptsetup -d pv11.key luksOpen /dev/md11 pv11
# cryptsetup luksAddKey --key-slot 5 /dev/md11
Enter password: ххххххххххххххх
```

После применения `cryptsetup`, для указанного тома (/dev/md11) UUID изменится на новый, а TYPE станет "crypto_LUKS". Например, был том:
```
# blkid /dev/md11
/dev/md11: UUID="394d0e20-fbbf-11e8-b8b2-272ad0095723" TYPE="ext4"
```

После применения `cryptsetup`:
```
# blkid /dev/md11
/dev/md11: UUID="2c326492-fbbf-11e8-83e1-db657904e51b" TYPE="crypto_LUKS"
```
поэтому, при необходимости, необходимо поправить те скрипты, где используется том по ссылке `/dev/disk/by-uuid/`.

Зачистить определённый слот хранения пароля в заголовке:
```
# cryptsetup -d /etc/cryptsetup-keys/pv11.key luksKillSlot /dev/md11 1
```

Бэкап заголовка зашифрованного тома:
```
# cryptsetup luksHeaderBackup --header-backup-file \
md11_$(date +%Y%m%d)_$(blkid -o value | head -n1).header.backup /dev/md11
```

или бэкап заголовка в ramfs, шифрованием и подписыванием бэкапа ключом gpg, сохранением зашифрованного бэкапа в текущую директорию:
```
# mkdir ramfs && mount -t ramfs ramfs ramfs && \
cryptsetup luksHeaderBackup --header-backup-file ramfs/tmp.header.backup \
/dev/md11 && gpg -e -s -r srv0_backup_2018 \
-o md11_$(date +%Y%m%d)_$(blkid -o value | head -n1).header.backup.gpg \
ramfs/tmp.header.backup && shred -u ramfs/tmp.header.backup && \
umount ramfs && rmdir ramfs
```

В заголовке тома хранится информация о параметрах шифрования тома, мастер-ключ (128, 256 или 512 бит), которым шифруются сектора тома, и восемь слотов по 256 байт, для хранения хэшей паролей к мастер-ключу.

Размер заголовка `luks` тома примерно один мегабайт при длине ключа шифрования 256 бит и около двух мегабайтов при длине ключа 512 бит. Для надёжного уничтожения заголовка, а значит и мастер-ключа, достаточно зачистить первые три мегабайта тома:
```
# dd if=/dev/urandom of=/dev/md11 bs=1M count=3; sync
```
или
```
# head -c 3M /dev/urandom > /dev/md11; sync
```

Если ранее был сделан бэкап заголовка, то с его помощью можно восстановить всю или часть информации из тома. В этом случае, для надёжной зачистки, нужно применить более продолжительное по времени перезапись всего пространства тома.

Обычно, перед окончанием аренды сервера, я запускаю сервер через rescue-образ, и выполняю для каждого диска:
```
# dd if=/dev/urandom of=/dev/sda bs=1M status=progress; sync
```

При недостатке времени на перезапись всех дисков, зачищаю только заголовки зашифрованных томов и всё пространство незашифрованных разделов.
