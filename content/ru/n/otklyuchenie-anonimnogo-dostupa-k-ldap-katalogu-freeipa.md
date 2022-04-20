---
title: "Отключение анонимного доступа к LDAP-каталогу FreeIPA"
date: 2021-05-07
weight: 10
description: >
  По умолчанию, FreeIPA позволяет просматривать LDAP-каталог анонимным пользователям, но, к счастью, это нежелательное явление можно пресечь.
tags:
  - FreeIPA
---

2021-05-07

Использованные ссылки:
- [Disabling Anonymous Binds](https://abbra.fedorapeople.org/.todo/freeipa-docs/#disabling-anon-binds)

По умолчанию, FreeIPA позволяет просматривать LDAP-каталог анонимным пользователям:
```
[root@prod-ipa02 /]# ldapsearch -x -h localhost -p 389 -b uid=ipara,ou=People,o=ipaca description
# extended LDIF
#
# LDAPv3
# base <uid=ipara,ou=People,o=ipaca> with scope subtree
# filter: (objectclass=*)
# requesting: description 
#

# ipara, people, ipaca
dn: uid=ipara,ou=people,o=ipaca
description: 2;7;CN=Certificate Authority,O=EXAMPLE.ORG;CN=IPA RA,O=EXAMPLE.ORG

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1
```

Мне неизвестна причина такого решения разработчиков. К счастью, это нежелательное явление можно пресечь добавлением, на каждой реплике FreeIPA, в нереплицируемую часть каталога, аттрибута 'nsslapd-allow-anonymous-access: rootdse' (после ввода блока с командами нажимаем стандартное 'Ctrl+d'):
```
# ldapmodify -x -D "cn=Directory Manager" -W -h localhost -p 389

dn: cn=config
changetype: modify
replace: nsslapd-allow-anonymous-access
nsslapd-allow-anonymous-access: rootdse
```

В инструкции указано, что необходимо перезапустить службу 'dirsrv', но в моём случае изменения применились немедленно:
```
[root@prod-ipa02 /]# ldapsearch -x -h localhost -p 389 -b uid=ipara,ou=People,o=ipaca description 
# extended LDIF
#
# LDAPv3
# base <uid=ipara,ou=People,o=ipaca> with scope subtree
# filter: (objectclass=*)
# requesting: description 
#

# search result
search: 2
result: 48 Inappropriate authentication
text: Anonymous access is not allowed.

# numResponses: 1
```

Применяемый аттрибут можно найти (или не найти) на каждом сервере в `/etc/dirsrv/slapd-EXAMPLE-ORG/dse.ldif`.
