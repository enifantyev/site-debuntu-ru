---
title: "Избавляемся от root'а в Alpine Linux"
date: 2019-12-23
weight: 10
description: >
  Избавляемся от root'а в Alpine Linux
tags:
  - root
  - Alpine Linux
---

2019-12-23

### Добавляем нового пользователя
Устанавливаем `sudo` в систему и добавляем нового пользователя `newuser`. Назначаем его участником группы `wheel`. Пользователям из этой группы позволяется *рулить* системой через `sudo`:
```
apk update
apk add sudo bash
adduser -s /bin/bash newuser
addgroup newuser wheel
```

Раскомментируем строку в файле `/etc/sudoers`, которая позволит группе `wheel` использовать `sudo`. Одной командой с помощью замечательного `sed`:
```
sed -i -e "s/^#.%wheel ALL=(ALL) ALL/%wheel ALL=(ALL) ALL/" /etc/sudoers
```
Или вручную с помощью команды `visudo` раскомментируем строку:
```
%wheel ALL=(ALL) ALL
```

### После входа под новым пользователем
Блокируем root'а:
```
passwd -l root
```

Запрещаем вход для root по ssh с помощью `sed`:
```
sed -i -e "s/^PermitRootLogin.*$/PermitRootLogin no/" /etc/ssh/sshd_config
```
Или вручную меняем соответствующую строку в файле `/etc/ssh/sshd_config`:
```
PermitRootLogin no
```

Рестарт `sshd`:
```
/etc/init.d/sshd restart
```
