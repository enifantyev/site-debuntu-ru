---
title: "Ручное удаление реплики FreeIPA из RedOS7.2"
date: 2021-07-13
weight: 10
description: >
  Ручное удаление реплики FreeIPA из RedOS7.2 после неудачных опытов.
tags:
  - FreeIPA
  - RedOS
  - RedOS 7.2
---

2021-07-13

После неудачных опытов приходится выковыривать останки FreeIPA.
```
yum autoremove bind bind-dyndb-ldap ipa-server*
pip freeze | xargs pip uninstall -y
```
Удаление большинства python-пакетов производим через поочерёдное `yum autoremove`, `pip freeze > list.txt` и `pip uninstall -y -r list.txt`. В файле `list.txt`, при необходимости, удаляем названия пакетов, неудаляемых через pip.

При необходимости можно выполнить полное удаление python3:
```
yum -y autoremove python3
rm -rf /usr/lib/python3.6 /usr/local/lib/python3.6
```

```
rm -rf /var/log/{dirsrv,httpd,ipa}
rm -f /var/log/{ipa*.log}
rm -rf /var/lib/{dirsrv,certmonger,gssproxy,ipa,ipa-client}
rm -rf /var/lib/pki/pki-tomcat
rm -rf /var/kerberos
rm -rf /etc/{dirsrv,httpd,gssproxy,ipa}
rm -rf /etc/pki/pki-tomcat
rm -f /etc/sysconfig/{dirsrv*,ipa-*,pki-tomcat,tomcat}
rm -rf /etc/sysconfig/pki/tomcat
```
