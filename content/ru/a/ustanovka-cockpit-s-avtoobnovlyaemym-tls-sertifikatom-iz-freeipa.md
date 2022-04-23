---
title: "Установка Cockpit с автообновляемым TLS-сертификатом из FreeIPA"
date: 2021-10-11
weight: 10
description: >
  Краткое описание процесса установки Cockpit с выпуском автообновляемого TLS-сертификатом из FreeIPA для обеспечения https.
tags:
  - Cockpit
  - FreeIPA
  - CentOS
  - CentOS 7
---

2021-10-11

## Условия
- Выполняем все команды на целевой машине.
- Предполагается, что машина введена во FreeIPA-домен.

## Установка и включение Cockpit
```
sudo yum install cockpit
sudo systemctl enable --now cockpit.service
sudo systemctl enable --now cockpit.socket
```

## Изменение номера порта

https://cockpit-project.org/guide/latest/listen.html

По умолчанию, cockpit слушает порт номер 9090. Перевесим его на 443-ий порт:
```
sudo mkdir -p /etc/systemd/system/cockpit.socket.d/

cat << EOF | sudo tee /etc/systemd/system/cockpit.socket.d/listen.conf
[Socket]
ListenStream=
ListenStream=443
EOF

sudo systemctl daemon-reload
```

## Скрипт для запуска после автоматической смены сертификата
```
cat << EOF | sudo tee /etc/cockpit/RunAfterUpdateCert.sh 
#!/bin/bash
rm -f /etc/cockpit/ws-certs.d/0-self-signed.cert.1
mv /etc/cockpit/ws-certs.d/0-self-signed.cert /etc/cockpit/ws-certs.d/0-self-signed.cert.1
cat /etc/pki/tls/certs/HTTP_$(hostname).crt \
    /etc/pki/tls/private/HTTP_$(hostname).key > \
    /etc/cockpit/ws-certs.d/0-self-signed.cert

/usr/bin/systemctl restart cockpit.service
/usr/bin/systemctl restart cockpit.socket
EOF
```
## Запрос TLS-сертификата с автообновлением
Предположим, что альтернативное имя для хоста будет 'cp01.example.org'.
```
ALIAS="cp01"

kinit
ipa dnsrecord-add $(hostname -d) ${ALIAS} --cname-rec="$(hostname)."
ipa service-add HTTP/$(hostname)@$(hostname -d)|tr [:lower:] [:upper:])
ipa service-add-principal HTTP/$(hostname) HTTP/${ALIAS}.$(hostname -d)

sudo ipa-getcert request -K HTTP/$(hostname) \
-f /etc/pki/tls/certs/HTTP_$(hostname).crt \
-k /etc/pki/tls/private/HTTP_$(hostname).key \
-D "${ALIAS}.$(hostname -d)" -C "/bin/bash -c '/etc/cockpit/RunAfterUpdateCert.sh'"
```
