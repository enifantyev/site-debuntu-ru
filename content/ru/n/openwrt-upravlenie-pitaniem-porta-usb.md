---
title: "OpenWRT Управление питанием порта USB"
date: 2015-10-29
weight: 10
description: >
  OpenWRT Управление питанием порта USB
tags:
  - OpenWRT
aliases:
  - /note/openwrt-upravlenie-pitaniem-porta-usb
  - /n/openwrt-upravlenie-pitaniem-porta-usb/
---

2015-10-29

[OpenWRT. The USB Port: An Overview](http://wiki.openwrt.org/doc/howto/usb.overview)

На TP-Link TL-WR842ND ver.2 проверил:
```
# ls /sys/class/gpio/
export gpio4 gpiochip0 unexport
```
Видимо используется пин 4, поэтому применил для выключения:
```
# echo 0 > /sys/class/gpio/gpio4/value
```
Для включения:
```
# echo 1 > /sys/class/gpio/gpio4/value
```
Чтение состояния:
```
# echo /sys/class/gpio/gpio4/value
```
Работает.
