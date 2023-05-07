---
title: "01. Важные замечания относительно Atlas + ADH
"
date: 2023-01-17
weight: 1
description: >
  Важные замечания относительно Atlas + ADH.
tags:
  - BigData
  - Apache Atlas
  - ArenaData Hadoop
---

2023-01-17

## Hardware
### Для Atlas'а надо много памяти
4GB не хватает. В логах записи:
```
There is insufficient memory for the Java Runtime Environment to continue.
```

При 8GB не пробовал.

При 16GB запуск успешен.

## Apache Atlas
### О возможности удаления сущностей из Atlas'а
Наблюдаю в настройках Atlas'а следующее:
```
# Delete handler
#
# This allows the default behavior of doing "soft" deletes to be changed.
#
# Allowed Values:
# org.apache.atlas.repository.store.graph.v1.SoftDeleteHandlerV1 - all deletes are "soft" deletes
# org.apache.atlas.repository.store.graph.v1.HardDeleteHandlerV1 - all deletes are "hard" deletes
#
#atlas.DeleteHandlerV1.impl=org.apache.atlas.repository.store.graph.v1.SoftDeleteHandlerV1
```