---
title: "AppArmor препятствует электронному подписыванию документов LibreOffice"
date: 2018-11-27
weight: 10
description: >
  AppArmor препятствует электронному подписыванию документов LibreOffice
tags:
  - AppArmor
  - LibreOffice
---

2018-11-27

### Description
При попытке подписать документ, в списке доступных подписей не наблюдаю доступных ключей. В менеджере криптографических ключей seahorse, вызываемого из libreoffice, также пустые списки.

В syslog заносятся записи:
```
apparmor="DENIED" operation="open" profile="libreoffice-soffice//gpg" name="/home/user/.gnupg/trustdb.gpg" comm="gpg" requested_mask="w" denied_mask="w"
apparmor="DENIED" operation="connect" profile="libreoffice-soffice//gpg" name="/run/user/1000/gnupg/S.gpg-agent"
comm="gpg" requested_mask="wr" denied_mask="wr"
```

### Solution
В файле `/etc/apparmor.d/abstractions/gnupg` добавил строку:
```
owner /run/user/*/gnupg/S.gpg-agent rw,
```

В конце файла `/etc/apparmor.d/usr.lib.libreoffice.soffice.bin`, в секции "profile gpg", добавил строку:
```
#include <abstractions/gnupg>
```
