---
title: "04. Исправление топологии после установки двух дополнительных реплик"
date: 2021-12-24
weight: 14
description: >
  Описание процесса исправления топологиии IPA-кластера после добавления реплик.
tags:
  - FreeIPA 4.9.6
  - FreeIPA
  - AlmaLinux 8.5
slug: ispravleniye-topologii-posle-ustanovki-dopolnitelnoy-dvukhrebernoy
draft: true
---

## 1. Топология перед исправлением 
После установки третьей реплики, топология кластера из трёх FreeIPA-реплик выглядит так:
https://dev-ipa01p.test3.lan/ipa/ui/#/p/topology-graph