---
title: "Cloudera Hue Log: Couldn't import snappy. Support for snappy compression disabled"
date: 2021-06-05
weight: 10
description: >
  Предупреждение исчезает после установки python2-пакета python-snappy.
tags:
  - Hue
  - Apache Hadoop
  - BigData
---

2021-06-05

## Анамнез
После перезапуска Hue в логе видны предупреждения "Couldn't import snappy. Support for snappy compression disabled.":
![Couldn't import snappy. Support for snappy compression disabled.](/img/cloudera-hue-log-couldnt-import-snappy-support-for-snappy-compression-disabled/hue_couldnt_import_snappy.png)

## Решение
На всех машинах кластера выполнил из-под root:
```
pip2 install python-snappy
```
Предупреждение исчезло.
