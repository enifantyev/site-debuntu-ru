---
title: "10. Установка и настройка BorgBackup"
date: 2022-03-11
weight: 10
description: >
  Описание процесса установки утилиты для дедуплицированного бэкапа BorgBackup в CentOS и Ubuntu.
tags:
  - BorgBackup
  - Backup SoftWare
  - CentOS
  - Ubuntu
---

2022-03-11

## 1. Установка BorgBackup
### 1.1. Добавление EPEL
На CentOS необходим репо EPEL.
```
$ sudo yum install epel-release
```

### 1.2. Установка BorgBackup в CentOS7
```
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

### 1.3. Ubuntu
На Ubuntu добавляем PPA:
```
sudo add-apt-repository ppa:costamagnagianfranco/borgbackup
```

## 2. Настройка Borg-сервера
2.1. Добавляем системного пользователя '_borg', под которым клиенты будут подключаться по ssh к серверу и запускать приложение `borg`. Пароль для этой УЗ не запоминаем, так как он не потребуется в дальнейшем, но без наличия пароля будет невозможно подключиться к borg-аккаунту через ssh.
```
export BORGUSERNAME="_borg"

set +o history
useradd -r -s /bin/sh \
  -m -p $(pwgen 21 1) ${BORGUSERNAME}
set -o history
```

2.2. Создаём папку `.ssh`, где позже разместим файлы необходимые для подключения по ssh.
```
sudo -u ${BORGUSERNAME} /bin/bash -c 'mkdir -p ~/.ssh && chmod 700 ~/.ssh'
```

2.3. Создаём ssh-ключи для будущих клиентов этого репо. Кстати, здесь их хранить не обязательно. Наверное, будет правильней для каждого репо создавать отдельные ключи.
```
export SSHKEYNAME="borg_repo1"

sudo -u ${BORGUSERNAME} /bin/bash -c "ssh-keygen -t ed25519 -q -N '' -f ~/.ssh/${SSHKEYNAME}"
```

## 3. Настройка первого репо для бэкапов

3.1. Создаём первый каталог для хранения первого репо и пробрасываем линк к нему в домашний каталог. По этой короткой ссылке будет удобно указывать название репо для архивов, вместо длинного полного пути к бэкап-каталогу.
```
export REPODIR="/data/borg/repo1"

sudo mkdir -p ${REPODIR}
sudo chown ${BORGUSERNAME}.${BORGUSERNAME} ${REPODIR}
sudo -u ${BORGUSERNAME} /bin/bash -c "ln -s ${REPODIR} ~/"
```

3.2. Если файл `authorized_keys` отсутствует, то создаём его с единственной записью-комментом "*Требуется указывать полный путь к каждому репо*".
```
sudo -u ${BORGUSERNAME} /bin/bash -c "cd ~/.ssh; echo '# Требуется указывать полный путь к каждому репо' > authorized_keys"
```

3.3. Добавляем в `authorized_keys` запись для доступа к репо:
```
sudo -u ${BORGUSERNAME} /bin/bash -c "cd ~/.ssh; echo -e 'command=\"/usr/bin/borg serve --restrict-to-path ${REPODIR}\",restrict $(cat ${SSHKEYNAME}.pub)' >> authorized_keys"
```

3.4. Инициализируем первый репо без шифрования:
```
REPOLINKNAME="repo1" # Имя ссылки в home на репо
export BORG_RSH="ssh -i ~/.ssh/${SSHKEYNAME}"
export BORG_REPO="${BORGUSERNAME}@localhost:${REPOLINKNAME}"

# Вызываем sudo -E для передачи переменных окружения в borg
sudo -E -u ${BORGUSERNAME} /bin/bash -c 'borg init -e none'
```

На ошибку `Error: Permission denied` внимания не обращаем, так как сама ошибка не локализована, но структура репо создаётся и в дальнейшем репо работает нормально. При создании же нового репо из клиента, данная ошибка не появляется.

3.5. Проверяем, что в каталоге первого репо создана структура:
```
$ ls -al ${REPODIR}
total 72
drwxr-xr-x 3 _borg _borg  4096 мар 11 17:30 .
drwxr-xr-x 3 root        root         4096 мар 10 15:42 ..
-rw------- 1 _borg _borg   209 мар 11 17:30 config
drwx------ 3 _borg _borg  4096 мар 11 17:30 data
-rw------- 1 _borg _borg    52 мар 11 17:30 hints.1
-rw------- 1 _borg _borg 41258 мар 11 17:30 index.1
-rw------- 1 _borg _borg   190 мар 11 17:30 integrity.1
-rw------- 1 _borg _borg    73 мар 11 17:30 README
```
