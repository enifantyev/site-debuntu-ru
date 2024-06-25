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

useradd -r -s /bin/sh \
  -m -p $(tr -dc 'A-Za-z0-9' </dev/urandom | head -c 21; echo) ${BORGUSERNAME}
```

1.2. Создаём папку `.ssh`, где позже разместим файлы необходимые для подключения по ssh.
```bash
sudo -u ${BORGUSERNAME} /bin/bash -c 'mkdir -p ~/.ssh && chmod 700 ~/.ssh'
```

1.3. Установка borg-утилиты в `/home/_borg/.local/bin`:
```bash
sudo -u ${BORGUSERNAME} python3 -m pip install --upgrade --user pip
sudo -u ${BORGUSERNAME} python3 -m pip install --user pkgconfig setuptools setuptools-scm wheel msgpack
sudo -u ${BORGUSERNAME} python3 -m pip install --user borgbackup
```
