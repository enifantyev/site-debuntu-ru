---
title: "Создание keytab-файла для пользовательского принципала"
date: 2021-01-30
weight: 10
description: >
  Получение пользовательского keytab из FreeIPA.
tags:
  - FreeIPA
  - Kerberos
  - keytab
---

2021-01-30

При получении keytab'а из FreeIPA, для пользователя потребуется сменить пароль.

1. В домашнем каталоге пользователя запускаем утилиту 'ipa-getkeytab' для генерации keytab-файла для пользователя;
2. на запрос пароля, дважды вводим НОВЫЙ пароль, который с этого момента станет актуальным для пользовательского УЗ;
3. одновременно, все прочие keytab'ы, сгенерированные ранее, станут недействительными.
```
$ ipa-getkeytab -p username -k username.keytab -P
New Principal Password:
Verify Principal Password:
Keytab successfully retrieved and stored in: username.keytab
```

Проверяем, что в созданном keytab-файле присутствует информация:
```
$ klist -kte username.keytab
Keytab name: FILE:username.keytab
KVNO Timestamp           Principal
---- ------------------- ------------------------------------------------------
  33 01/16/2021 11:52:00 username@EXAMPLE.ORG (aes256-cts-hmac-sha1-96)
  33 01/16/2021 11:52:00 username@EXAMPLE.ORG (aes128-cts-hmac-sha1-96)
```

Получение kerberos-билета:
```
$ kinit -kt username.keytab username
$ klist
Ticket cache: FILE:/tmp/krb5cc_192800004_g8LRFe
Default principal: username@EXAMPLE.ORG
 
Valid starting       Expires              Service principal
01/16/2021 12:00:20  01/17/2021 12:00:20  krbtgt/EXAMPLE.ORG@EXAMPLE.ORG
        renew until 01/23/2021 12:00:20
```

Если полученная информация напоминает вышеприведённую, то можно считать, что keytab-файл успешно создан.
