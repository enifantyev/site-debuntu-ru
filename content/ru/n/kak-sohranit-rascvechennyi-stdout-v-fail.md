---
title: "Как сохранить расцвеченный выхлоп stdout в файл"
date: 2021-10-19
weight: 10
description: >
  Расцвеченный stdout становится монохромным при передаче через pipe, но есть способ сохранить цвет.
tags:
  - less
  - colorized stdout
---

2021-10-19

Использованные ссылки:
- https://superuser.com/questions/352697/preserve-colors-while-piping-to-tee

Установка необходимого пакета, который содержим `unbuffer`:
```
apt install expect-dev
# or
yum install expect-devel
```

Запускам, например, сборку maven-приложения через `unbuffer` и сохраняем расцвеченный вывод `mvn` в файл `install_1.log`:
```
unbuffer mvn clean -DskipTests package -Pdist -X -T 4 2>&1 | tee install_1.log
```

Смотрим ранее сохраннёный лог-файл с ESC-кодами:
```
less -r install_1.log
```
