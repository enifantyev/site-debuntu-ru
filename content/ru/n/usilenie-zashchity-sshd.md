---
title: "Усиление защиты sshd"
date: 2016-01-20
weight: 10
description: >
  Усиление защиты sshd
tags:
  - ssh
  - OpenSSH
---

2016-01-20

https://stribika.github.io/2015/01/04/secure-secure-shell.html
https://www.opennet.ru/tips/2877_ssh_crypt_setup_security_nsa.shtml

Добавить ssh-audit

При создании новых ключей использовать ED25519 или RSA с длиной ключа более 4096:
```
ssh-keygen -o -a 129 -t ed25519 -C e0_20160116_ed25519 -f ~/.ssh/id_ed25519
```
```
ssh-keygen -t rsa -b 4096 -o -a 132 -C e0_20160116_rsa -f ~/.ssh/id_rsa
```
--------------------------------------------------------------------------------
На сервере в `sshd_config` добавить:
```
KexAlgorithms curve25519-sha256@libssh.org,diffie-hellman-group-exchange-sha256

Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com,aes256-ctr,aes192-ctr,aes128-ctr

MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com,hmac-ripemd160-etm@openssh.com,umac-128-etm@openssh.com,hmac-sha2-512,hmac-sha2-256,hmac-ripemd160,umac-128@openssh.com
```
Там же оставить указание только на сильные серверные ключи:
```
Protocol 2
HostKey /etc/ssh/ssh_host_ed25519_key
HostKey /etc/ssh/ssh_host_rsa_key
```
И пересоздать два серверных ключа rsa и ed25519 и сделать заглушки для отказа в создании новых dsa и ecdsa:
```
# cd /etc/ssh
# rm ssh_host_*key*
# ssh-keygen -t ed25519 -N '' -f ssh_host_ed25519_key < /dev/null
# ssh-keygen -t rsa -b 4096 -N '' -f ssh_host_rsa_key < /dev/null
# ln -s /etc/ssh/ssh_host_ecdsa_key /etc/ssh/ssh_host_ecdsa_key
# ln -s /etc/ssh/ssh_host_key /etc/ssh/ssh_host_key
# ln -s /etc/ssh/ssh_host_dsa_key /etc/ssh/ssh_host_dsa_key
```
Заглушки помешают последующим обновлениям, поэтому будет достаточно закомментить соответствующие строки в файле настройки sshd.

Отредактировать `/etc/ssh/moduli` для использования RSA с длиной ключа больше 2000.
```
# awk '$5 > 2000' moduli > "${HOME}/moduli"
# wc -l "${HOME}/moduli"
# mv "${HOME}/moduli" moduli
```

На клиенте в `/etc/ssh/ssh_config` добавить:
```
Host *
KexAlgorithms curve25519-sha256@libssh.org,diffie-hellman-group-exchange-sha256

Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com,aes256-ctr,aes192-ctr,aes128-ctr

MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com,hmac-ripemd160-etm@openssh.com,umac-128-etm@openssh.com,hmac-sha2-512,hmac-sha2-256,hmac-ripemd160,umac-128@openssh.com
```
