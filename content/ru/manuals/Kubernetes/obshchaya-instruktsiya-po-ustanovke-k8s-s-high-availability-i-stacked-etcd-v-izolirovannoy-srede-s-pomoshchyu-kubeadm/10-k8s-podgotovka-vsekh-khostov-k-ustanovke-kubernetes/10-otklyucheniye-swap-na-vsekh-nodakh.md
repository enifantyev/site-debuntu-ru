---
title: "10. Отключение swap на всех нодах"
date: 2023-08-08
weight: 10
description: >
  Отключение swap перед установкой Kubernetes
tags:
  - swap
slug: otklyucheniye-swap-na-vsekh-nodakh
---

2023-08-08

### Использованные материалы
1. https://kubernetes.io/docs/setup/production-environment/container-runtimes/

### Обязательное отключение swap
1. Проверяем включен ли 'swap' и отключаем его:
    ```console
    swapon -a
    if [[ $(swapon -s) != "" ]]; then
      cp /etc/fstab /etc/fstab.$(date +%y%m%d-%H%M%S).bak
      swapoff -a
      sed -i '/swap/ s/^/#/' /etc/fstab
    fi
    ```

2. Повторно пытаемся включить swap и проверяем состояние swap:
    ```console
    swapon -a
    swapon -s
    ```

3. Если выхлоп в stdout пуст, то продолжаем выполнять инструкцию. Иначе разбираемся в причинах неудачи.
