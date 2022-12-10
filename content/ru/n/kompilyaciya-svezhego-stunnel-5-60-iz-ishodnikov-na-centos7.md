---
title: "Компиляция свежего stunnel 5.60 из исходников на CentOS7"
date: 2021-11-26
weight: 10
description: >
  Описание компиляции stunnel 5.60, так как в репозиториях CentOS7 имеется только древний centos 4.56.
tags:
  - stunnel
  - CentOS
  - CentOS 7
aliases:
  - /manuals/n/kompilyaciya-svezhego-stunnel-5-60-iz-ishodnikov-na-centos7
---

2021-11-26

## Подготовка с сборке
Устанавливаем набор инструментов:
```
$ sudo yum groupinstall "Development Tools"
```

## Сборка
```
PCKG="stunnel-5.60"

mkdir ~/src && cd ~/src
curl -LO https://www.stunnel.org/downloads/${PCKG}.tar.gz
curl -LO https://www.stunnel.org/downloads/${PCKG}.tar.gz.sha256
sha256sum -c ${PCKG}.tar.gz.sha256
tar xvf ${PCKG}.tar.gz
cd ${PCKG}
./configure
make
```

## Результат
Скомпилированный stunnel можно найти здесь `~/src/stunnel-5.60/src/stunnel`.

Переименуем файл для удобства дальнейшего использования по инструкции [Обфускация ssh-трафика с помощью stunnel](/a/obfuskaciya-ssh-trafika-s-pomoschyu-stunnel):
```
mv ~/src/stunnel-5.60/src/stunnel ~/src/stunnel-5.60/src/stunnel-5.60
```
