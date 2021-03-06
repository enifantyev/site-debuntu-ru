---
title: "Создание копии сайта Plone"
date: 2019-05-10
weight: 10
description: >
  Частичный перевод статьи https://docs.plone.org/manage/deploying/copy.html с моими дополнениями.
tags:
  - Plone
  - CMS
  - translation
---

2019-05-10

> Краткая инструкция о том, как создать копию Plone-инсталляции.

### Введение
Эта инструкция расскажет вам основы создания дубля Plone-сайта для тестирования или бэкапа.

### Необходимые условия
- Возможность копирования файлов из/в удалённый сервер
- Возможность использования командной строки

### Содержимое Plone-сайта
Что должно быть скопировано:

- `buildout.cfg` - определяет конфигурацию вашего сайта
- `src folder` - содержит аддоны разработанные вами
- `var/filestorage/Data.fs` - ZODB база данных вашего сайта
- `var/blobstorage folder` - содержит объекты типа "файл" базы данных ZODB (BLOBы)

Другие папки (eggs, downloads, parts и т.д.) генерируются утилитой buildout и могут оставаться пустыми.

### Копирование и подготовка к первому старту Plone-сайта
- Создайте новый сайт в новом месте с помощью *Унифицированного Plone Установщика* и убедитесь, что можете войти в admin-аккаунт с его временным паролем.
- Удалите содержимое папки `var/filestorage/` на новой системе.
- Скопируйте файл `var/filestorage/Data.fs` из старой системы в новую систему и заметьте, что пароль админа сохраняется в этом копируемом файле, поэтому временный пароль, полученный во время установки нового сайта более недействителен.
- Удалите содержимое папки `var/blobstorage` на новой системе.
- Скопируйте папку `var/blobstorage`.
- Скопируйте папку `src`, если у вас есть свои разработки для сайта.
- Скопируйте `buildout.cfg` и другие `.cfg` файлы. 
- **Перезапустите `sudo -u plone_buildout bin/buildout` для автоматической загрузки и настройки всех Python-пакетов необходимых для запуска нового сайта**

Теперь можно запустить сайт `sudo -u plone_daemon /srv/plone/zinstance/bin/plonectl fg`.

По какой-то причине я не смог увидеть содержимое моего скопированного сайта по адресу `http://127.0.0.1:8080/id1`. Отображалась пустая страница, хотя в HTML-код содержал полную страницу скопированного сайта.  Рабочим адресом является `http://localhost:8080/id1`.
