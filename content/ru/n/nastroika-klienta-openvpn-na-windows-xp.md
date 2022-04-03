---
title: "Настройка клиента OpenVPN на Windows XP"
date: 2011-05-28
weight: 10
description: >
  Настройка клиента OpenVPN на Windows XP
tags:
  - OpenVPN
  - Windows XP
---

2011-05-28

## Подготовка к настройке OpenVPN клиента
Скачал с openvpn.net/download.html windows клиент и установил его, соглашаясь со всеми запросами, используя учётную запись администратора.
Копирую файл `C:\Program Files\OpenVPN\sample-config\client.ovpn` в папку `C:\Program Files\OpenVPN\config\`

В эту же папку копирую с сервера сгенерированные файлы:

- ca.crt — сертификат центра сертификации
- client.key — приватный ключ клиента
- client.crt — сертификат клиента
- ta.key — TLS-ключ, если используется в настройках OpenVPN сервера

## Настройка OpenVPN клиента
Запускаю Пуск — Все программы — OpenVPN — OpenVPN GUI

Рядом с часами обнаруживаю иконку — два малиновых компьютерных экранчика с земным шариком. Нажимаю на эту иконку правую кнопку мышки и выбираю Edit Config. Запускается Блокнот с содержимым файла `client.ovpn`. Содержимое файла в линуксоидном формате, поэтому, чтобы в ручную не форматировать файл, всё содержимое стёр и вставил строки из `sample-config-files/client.conf`.

Отредактировал новое содержимое под свои нужды:

<details><p>

```
# Specify that we are a client and that we
# will be pulling certain config file directives
# from the server.
client

# Use the same setting as you are using on
# the server.
# On most systems, the VPN will not function
# unless you partially or fully disable
# the firewall for the TUN/TAP interface.
;dev tap
dev tun

# Windows needs the TAP-Win32 adapter name
# from the Network Connections panel
# if you have more than one.  On XP SP2,
# you may need to disable the firewall
# for the TAP adapter.
;dev-node MyTap

# Are we connecting to a TCP or
# UDP server?  Use the same setting as
# on the server.
;proto tcp
proto udp

# The hostname/IP and port of the server.
# You can have multiple remote entries
# to load balance between the servers.
remote 192.168.0.1 1194
;remote my-server-2 1194

# Choose a random host from the remote
# list for load-balancing.  Otherwise
# try hosts in the order specified.
;remote-random

# Keep trying indefinitely to resolve the
# host name of the OpenVPN server.  Very useful
# on machines which are not permanently connected
# to the internet such as laptops.
resolv-retry infinite

# Most clients don't need to bind to
# a specific local port number.
nobind

# Downgrade privileges after initialization (non-Windows only)
;user nobody
;group nobody

# Try to preserve some state across restarts.
persist-key
persist-tun

# If you are connecting through an
# HTTP proxy to reach the actual OpenVPN
# server, put the proxy server/IP and
# port number here.  See the man page
# if your proxy server requires
# authentication.
;http-proxy-retry # retry on connection failures
;http-proxy [proxy server] [proxy port #]

# Wireless networks often produce a lot
# of duplicate packets.  Set this flag
# to silence duplicate packet warnings.
;mute-replay-warnings

# SSL/TLS parms.
# See the server config file for more
# description.  It's best to use
# a separate .crt/.key file pair
# for each client.  A single ca
# file can be used for all clients.
ca ca.crt
cert client.crt
key client.key

# Verify server certificate by checking
# that the certicate has the nsCertType
# field set to "server".  This is an
# important precaution to protect against
# a potential attack discussed here:
#  http://openvpn.net/howto.html#mitm
#
# To use this feature, you will need to generate
# your server certificates with the nsCertType
# field set to "server".  The build-key-server
# script in the easy-rsa folder will do this.
;ns-cert-type server

# If a tls-auth key is used on the server
# then every client must also have the key.
tls-auth ta.key 1

# Select a cryptographic cipher.
# If the cipher option is used on the server
# then you must also specify it here.
cipher BF-CBC

# Enable compression on the VPN link.
# Don't enable this unless it is also
# enabled in the server config file.
comp-lzo

# Set log file verbosity.
verb 3

# Silence repeating messages
;mute 20
```
</p></details>

## Использование OpenVPN клиента
Для работы OpenVPN клиента пользователю необходимы администраторские привилегии. Если у пользователя нет администраторских прав, то необходима дополнительная настройка компьютера.

Если OpenVPN клиент установлен на Windows Home Edition, то никаких дополнительных настроек не нужно. В этой операционной системе у обычных пользователей есть администраторские права, которые необходимы для работы OpenVPN клиента.

Пользователь запускает иконку "OpenVPN GUI" на рабочем столе или в меню Пуск и нажимает правую мышь на появившуюся иконку OpenVPN рядом с часами. Далее выбирает пункт "Connect". Для разрыва связи пользователь выбирает пункт "Disconnect".
