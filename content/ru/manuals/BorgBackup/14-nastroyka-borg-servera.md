---
title: "14. Настройка BorgBackup-сервера"
date: 2022-03-11
weight: 14
description: >
  Описание процесса настройки Borg-сервера.
tags:
  - BorgBackup
  - Backup SoftWare
slug: nastroyka-borg-servera
---

2022-03-11

## 1. Настройка Borg-сервера
1.1. Добавляем системного пользователя '_borg', под которым клиенты будут подключаться по ssh к серверу и запускать приложение `borg`. Пароль для этой УЗ не запоминаем, так как он не потребуется в дальнейшем, но без наличия пароля будет невозможно подключиться к borg-аккаунту через ssh.
```bash
export BORGUSERNAME="_borg"

set +o history
useradd -r -s /bin/sh \
  -m -p $(pwgen 21 1) ${BORGUSERNAME}
set -o history
```

1.2. Создаём папку `.ssh`, где позже разместим файлы необходимые для подключения по ssh.
```bash
sudo -u ${BORGUSERNAME} /bin/bash -c 'mkdir -p ~/.ssh && chmod 700 ~/.ssh'
```

1.3. Создаём ssh-ключи для будущих клиентов этого репо. Кстати, здесь их хранить не обязательно. Наверное, будет правильней для каждого репо создавать отдельные ключи.
```bash
export SSHKEYNAME="borg_repo1"

sudo -u ${BORGUSERNAME} /bin/bash -c "ssh-keygen -t ed25519 -q -N '' -f ~/.ssh/${SSHKEYNAME}"
```

## 2. Настройка первого репо для бэкапов

2.1. Создаём первый каталог для хранения первого репо и пробрасываем линк к нему в домашний каталог. По этой короткой ссылке будет удобно указывать название репо для архивов, вместо длинного полного пути к бэкап-каталогу.
```bash
export REPODIR="/data/borg/repo1"

sudo mkdir -p ${REPODIR}
sudo chown ${BORGUSERNAME}.${BORGUSERNAME} ${REPODIR}
sudo -u ${BORGUSERNAME} /bin/bash -c "ln -s ${REPODIR} ~/"
```

2.2. Если файл `authorized_keys` отсутствует, то создаём его с единственной записью-комментом "*Требуется указывать полный путь к каждому репо*".
```bash
sudo -u ${BORGUSERNAME} /bin/bash -c "cd ~/.ssh; echo '# Требуется указывать полный путь к каждому репо' > authorized_keys"
```

2.3. Добавляем в `authorized_keys` запись для доступа к репо:
```bash
sudo -u ${BORGUSERNAME} /bin/bash -c "cd ~/.ssh; echo -e 'command=\"/usr/bin/borg serve --restrict-to-path ${REPODIR}\",restrict $(cat /home/${BORGUSERNAME}/.ssh/${SSHKEYNAME}.pub)' >> authorized_keys"
```

2.4. Инициализируем первый репо без шифрования:
```bash
REPOLINKNAME="repo1" # Имя ссылки в home на репо
export BORG_RSH="ssh -i ~/.ssh/${SSHKEYNAME}"
export BORG_REPO="${BORGUSERNAME}@localhost:${REPOLINKNAME}"

# Вызываем sudo -EH для передачи переменных
# окружения в borg и смены HOME.
sudo -EH -u ${BORGUSERNAME} /bin/bash -c 'borg init -e none'
```

2.5. Проверяем, что в каталоге первого репо создана структура:
```bash
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
