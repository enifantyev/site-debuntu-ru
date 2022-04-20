---
title: "04. После установки основных компонентов"
date: 2021-06-16
weight: 4
description: >
  В этой части описывается завершение установки основных компонентов.
tags:
  - Cloudera CDH 6.3.2
  - Apache Hadoop
  - CentOS
  - CentOS 7
  - BigData
slug: posle-ustanovki-osnovnykh-komponentov
---

2021-06-16

## Настройка каталогов для логгирования
В центральном окне Cloudera Web UI выбираем раздел Log Directories, где изменяем все '/var/log' на '/data/log':
![](/img/ustanovka-cloudera-cdh-6.3.2-s-tls-i-kerberos-na-osnove-freeipa/image_4_1.png)

Также и для каждого Cloudera-сервиса заходим в конфигурацию и ищём параметры по признаку '/var', которые и меняем на '/data/log'.

Применяем изменения для Cloudera Management и для кластера.

## Выполняем предложения Cloudera
Рассматриваем предложения Cloudera Management Console и исправляем ситуацию. Стараемся, по возможности, добиться полностью "зелёного" состояния кластера, но часто это невозможно из-за, например, недостатка ресурсов выделенных для инициализации кластера.
![](/img/ustanovka-cloudera-cdh-6.3.2-s-tls-i-kerberos-na-osnove-freeipa/image_4_2.png)
