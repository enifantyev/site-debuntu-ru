---
title: "Обфускация ssh-трафика с помощью stunnel"
date: 2021-11-19
weight: 10
description: >
  Иногда требуется скрыть от посторонних глаз наличие ssh-трафика или дополнительно защитить sshd-порт. Описывается способ с аутентификацией по TLS-сертификату.
tags:
  - stunnel
  - Obfuscation
  - CentOS
  - CentOS 7
  - ssh
aliases:
  - /a/obfuskaciya-ssh-trafika-s-pomoschyu-stunnel/
slug: obfuskaciya-ssh-trafika-s-pomoschyu-stunnel
---

2021-11-19

## Настройка сервера
Устанавливаем на сервер пакет stunnel:
```
sudo yum install stunnel
```

Компилируем свежий stunnel по инструкции [Компиляция свежего stunnel 5.60 из исходников на CentOS7](/n/kompilyaciya-svezhego-stunnel-5-60-iz-ishodnikov-na-centos7).

Подменяем установленный stunnel 4.56, новым stunnel 5.60:
```
sudo mv /usr/bin/stunnel /usr/bin/stunnel-4.56
sudo cp ~/src/stunnel-5.60/src/stunnel-5.60 /usr/bin
cd /usr/bin
ln -s stunnel-5.60 stunnel
```

Добавляем системного пользователя, под которым будет запускаться stunnel:
```
sudo useradd -r -s /sbin/nologin -M stunnel
```

<Здесь используем FreeIPA-сертификаты, сгенерированные на сервере. Потом допишу процесс получения и автообновления с помощью Certmonger.>

Создаём конфигурационный файл для stunnel:
```
cat << EOF | sudo tee /etc/stunnel/stunnel.conf 
#output = /var/log/stunnel/stunnel.log
pid = /run/stunnel/stunnel.pid

[ssh]
client = no
accept = 2222
connect = 127.0.0.1:22

CAfile = /etc/ipa/ca.crt
cert = /etc/pki/tls/certs/HTTP_srv.example.org.crt
key = /etc/pki/tls/private/HTTP_srv.example.org.key
CApath = /etc/stunnel/certs

# CentOS7 - obsolete options.
#sslVersion = TLSv1.2
#verify = 2

# New options.
requireCert = yes
verifyChain = yes

sslVersion = all
options    = NO_SSLv2
options    = NO_SSLv3
options    = NO_TLSv1
options    = NO_TLSv1.1
EOF
```

Создадим каталог для log-файла и назначим права:
```
sudo mkdir /var/log/stunnel
sudo chown stunnel.stunnel /var/log/stunnel
sudo chown -R stunnel.stunnel /etc/stunnel 
```

Создаём systemd unit для автоматического запуска stunnel:
```
cat << EOF | sudo tee /etc/systemd/system/stunnel.service
[Unit]
Description=Stunnel service
After=network.target syslog.target
 
[Service]
RuntimeDirectory=stunnel
RuntimeDirectoryMode=0775
User=stunnel
Group=stunnel
Type=forking
#ExecStart=/usr/bin/stunnel -v /etc/stunnel/stunnel.conf
ExecStart=/usr/bin/stunnel  /etc/stunnel/stunnel.conf
Restart=on-failure
RestartSec=5s
PrivateTmp=true

[Install]
WantedBy=multi-user.target
EOF
```

Перечитываем systemd и запускаем stunnel.
```
sudo systemctl daemon-reload
sudo systemctl enable --now stunnel
```

## Получение сертификата пользователя
На своей рабочей машине создаём свой приватный ключ `eugene.key`и запрос на сертификацию `cert.csr` (опция '-nodes' опускает задание пароля для ключа):
```
$ openssl req -new -newkey rsa:4096 -nodes -keyout eugene.key -out cert.csr -subj '/CN=eugene'
Generating a RSA private key
...........++++
.........++++
writing new private key to 'eugene.key'
```

<Здесь дописать процедуру получения TLS-сертификата из FreeIPA с помощью созданного файла с запросом на сертификацию.>

После получения сертификата, вычисляем его hash:
```
$ openssl x509 -hash -noout -in eugene.crt
0518aad7
```

Создаём на сервере в каталоге `/etc/stunnel/certs` файл `0518aad7.0` с содержимым, копией содержимого полученного сертификата. Цифра в расширении имени файла означает порядковый номер. Следующему добавленному сертификату дадим имя `xxxxxxxx.1`.

Возможно, что на сервере, после добавления очередного сертификата, потребуется перезапустить демон tunnel:
```
sudo systemctl restart stunnel
```

## Настройка клиента
Устанавливаем пакет stunnel:
```
sudo apt install stunnel4
```

Создаём конфигурационный файл для stunnel:
```
mkdir ~/.config/stunnel/

cat << EOF > ~/.config/stunnel/stunnel1.conf
output = /tmp/stunnel.log
pid = /tmp/stunnel.pid

[ssh]
client = yes
accept = 127.0.0.1:2222
connect = 10.1.61.28:2222

CAfile = /home/user/.config/stunnel/ca.crt
cert = /home/user/.config/stunnel/eugene_example.org.pub
key = /home/user/.config/stunnel/eugene.key

verifyChain = yes
EOF
```

С этого момента можно сразу вручную прокинуть мост до удалённого сервера командой `stunnel ~/.config/stunnel/stunnel1.conf`.

Для облегчения запуска туннеля, создаём запускаемый файл в каталоге `~/.local/bin`:
```
cat << EOF > ~/.local/bin/stun1
#!/bin/bash
stunnel ~/.config/stunnel/stunnel1.conf
EOF

chmod u+x ~/.local/bin/stun1
```

С этого момента можно запускать туннель командой `stun1` при условии, что в переменной PATH указан путь `~/.local/bin`.

Добавляем Alias для входа на удалённый sshd через TLS-туннель:
```
alias ssh1="ssh -p 2222 eugene@127.0.0.1"
```

При желании, можно добавить этот Alias в `~/.bashrc`.

## Процедура подключения к ssh-серверу
Поднимаем туннель и подключаемся через него к удалённому серверу:
```
stun1
ssh1
```

При необходимости роняем TLS-туннель командой, например:
```
pkill stunnel
```
