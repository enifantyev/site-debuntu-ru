---
title: "Chroot rsync для безопасного сбора бэкапов с удалённых хостов"
date: 2019-05-12
weight: 10
description: >
  Здесь я описал сценарий выкачивания файлов из удалённого хоста на бэкап-хост .
tags:
  - rsync
  - ssh
  - chroot
---

2019-05-12

Один из способов использования [Rsync](https://rsync.samba.org/) для синхронизации файлов между машинами, это подключение **rsync-клиента** к **rsync-серверу** через ssh-сеанс.

Здесь я описал следующий сценарий выкачивания файлов из удалённого хоста `site01.example.com` на *бэкап-хост*:

1. На *клиенте*, которым выступает *бэкап-хост*, запускаем **rsync-клиент**:
  - `rsync -avP rbkp@site01.example.com:/backup/ /backup/`
- *Клиент* стучится на ssh-порт *сервера* `site01.example.com` от имени пользователя *rbkp*.
- На *сервере* служба `sshd` создаёт сеанс для *rbkp* в подготовленной песочнице.
- В контексте сеанса *rbkp* через `/bin/sh` начинает выполняться **rsync-сервер**:
  - `rsync --server --sender -vlogDtpre.iLsf . /backup/`.
- Происходит синхронизация между **rsync-клиентом** и **rsync-сервером**.


### Подготавливаем песочницу на удалённом хосте
Создаём скелет песочницы для **rsync-сервера**, то есть создаём отдельную директорию с небольшим количеством файлов. Подключившийся пользователь, от имени которого запустится `rsync --server ...`, будет иметь доступ только к содержимому этой директории.
```
# mkdir -p /mnt/rbkp/{.ssh,backup,bin}
# chmod g+s /mnt/rbkp/backup
```
- В директории `.ssh` создадим  файл `authorized_keys` со строкой идентификатора публичного ключа для ssh-сеанса.
- В директории `backup` складируем бэкапы, предназначенные для передачи в архив.
- В папке `bin` лежат бинарники, необходимые для работы **rsync-сервера**.
- В процессе подготовки песочницы появятся ещё две или три директории (`lib, lib64, usr/lib` и т.д.) с библиотеками, необходимыми для работы бинарников.

Копируем бинарники:
```
# cp /bin/sh /usr/bin/rsync /mnt/rbkp/bin/
```

Ищем библиотеки необходимые для работы бинарников и копируем их в соответствующие папки. Для `/bin/sh`:
```
# for f in $(ldd /bin/sh | awk '{print $1"\n"$3}' | grep ^/); do echo $f && rsync -aRL $f /mnt/rbkp; done
/lib/x86_64-linux-gnu/libc.so.6
/lib64/ld-linux-x86-64.so.2
```

Для `/usr/bin/rsync`:
```
# for f in $(ldd /usr/bin/rsync | awk '{print $1"\n"$3}' | grep ^/); do echo $f && rsync -aRL $f /mnt/rbkp; done
/lib/x86_64-linux-gnu/libattr.so.1
/lib/x86_64-linux-gnu/libacl.so.1
/lib/x86_64-linux-gnu/libpopt.so.0
/lib/x86_64-linux-gnu/libc.so.6
/lib64/ld-linux-x86-64.so.2
```

В результате выполнения этих команд в песочнице появятся библиотеки, необходимые для работы бинарников.

### Создаём пользователя для работы **rsync-сервера** на удалённом хосте
Создаём пользователя без домашней папки, пароля и с минималистическим командной оболочкой `sh`:
```
# adduser --home /mnt/rbkp --no-create-home --disabled-password --shell /bin/sh rbkp
```
### Назначим правильных владельцев на песочницу
```
# chown -R root:root /mnt/rbkp
# chown -R rbkp:rbkp /mnt/rbkp/.ssh /mnt/rbkp/backup
```

### Настраиваем sshd на удалённом хосте
Подразумевается, что sshd уже настроен на подключение с помощью открытых ключей.

В файл `/etc/ssh/sshd_config` добавляем блок отвечающий за перемещение пользователя rbkp в песочницу. Перемещение выполняется после проверки ключа ssh.
```
Match User rbkp
    ChrootDirectory /mnt/rbkp
```

### Генерируем ssh-ключи и настраиваем свойства подключения
На *бэкап-хосте*, в директории для бэкапов, генерируем новые безпарольные ssh-ключи:
```
ssh-keygen -t ed25519 -a 256 -C rbkp_ed25519_$(date +%Y%m%d) -f rbkp_ed25519_$(date +%Y%m%d)
```

На удалённом хосте в файл `/mnt/rbkp/.ssh/authorized_keys` добавляем публичную часть нового ssh-ключа и указываем опции ужесточающие политику подключения:
```
# cat /mnt/rbkp/.ssh/authorized_keys
command="/bin/rsync --server --sender -vlogDtpre.iLsf . /backup/",no-port-forwarding,no-X11-forwarding,no-user-rc,no-agent-forwarding,no-pty ssh-ed25519 AAAAC3Nza...fDWFH rbkp_ed25519_20190331

```

### Итоги
На бэкап-сервере ставим в крон задачу:
```
rsync -avP -e'ssh -p22 -irbkp_ed25519_20190331 -oUserKnownHostsFile=/dev/null -oStrictHostKeyChecking=no' --bwlimit=1M rbkp@site01.example.com:/backup/ /mnt/backup/example.com/
```

В случае кражи ssh-ключей и информации о параметрах для подключения к удалённому хосту, злоумышленнику будет доступен удалённый просмотр папки `/mnt/rbkp/backup` (посредством `rsync --list-only`) и её выкачивание на свою машину. Если права на папку допускают удаление, то информация в этой папке может быть уничтожена.
