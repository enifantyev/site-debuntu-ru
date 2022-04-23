---
title: "Mikrotik: Wireless WDS Mesh With Mesh Interface"
date: 2018-10-05
weight: 10
description: >
  Здесь кратко описан способ создания mesh-сети с помощью mesh-интерфейсов (есть ещё способ с использованием bridge-интерфейсов).
tags:
  - meshnet
  - Mikrotik
---

2018-10-05

#### Предварительная настройка mikrotik routers
Рабочая частота, имя сети, безопасность wireless-интерфейсов должны быть одинаковыми на всех точках.

#### На каждом mesh-узле произвести следующие одинаковые действия.
1. Добавление mesh-интерфейса:
  - ```interface mesh add name=mesh4```
2. Добавление wireless-интерфейса с указанием автоматического добавления порта для wds-моста:
  - ```interface wireless add wlan4 master-interface=wlan1 ssid=MESHNET mode=ap-bridge wds-mode=dynamic-mesh wds-default-bridge=mesh4```
3. Добавление порта mesh для возможности подключения клиентов:
  - ```interface mesh port add interface=wlan4 mesh=mesh4```

#### На каждом mesh-узле указываем разные локальные ip-адреса.
4. Указание адреса на mesh-интерфейсе первого узла:
  - ```ip address add address=192.168.65.1/24 interface=mesh4```
5. Указание адреса на mesh-интерфейсе второго узла:
  - ```ip address add address=192.168.65.2/24 interface=mesh4```
6. Указание адреса на mesh-интерфейсе третьего узла:
  - ```ip address add address=192.168.65.3/24 interface=mesh4```

#### Ускоренное отключение клиентов со слабым сигналом
```
/interface wireless access-list
add authentication=yes comment=«Accept strong clients» signal-range=-71..120
add authentication=no comment=«Block weak clients» signal-range=-120..-70
```

#### Только хорошее качество сигнала для построения WDS
```
/interface wireless connect-list
add comment=«Block weak wds» interface=mesh4 connect=no \
security-profile=default signal-range=-120..-60
```

#### На первом шлюзе
1. Добавление пула адресов:
  - ```/ip pooladd name=dhcp_meshnet ranges=192.168.65.65-192.168.65.249```
2. Добавление dhcp-сервера:
  - ```/ip dhcp-server add add-arp=yes address-pool=dhcp_meshnet interface=mesh4 lease-time=8h name=dhcp```

#### Дополнительные заметки
- Нет нужды на рядовых mesh-узлах добавлять relay dhcp-запросов, или настраивать файрвол, или ещё каким-либо образом администрировать пакеты от клиентов, так как mesh-технология прозрачно передаёт пакеты внутри mesh-моста на выходной узел.
- На выходном узле необходимо добавить правило в файрвол для пропуска пакетов из/в новую mesh-сеть.

https://mum.mikrotik.com//presentations/RU15/presentation_2573_1443518501.pdf
