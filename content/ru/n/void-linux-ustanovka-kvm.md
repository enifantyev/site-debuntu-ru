---
title: "Void Linux Установка QEMU/KVM"
date: 2020-03-17
weight: 10
description: >
  Описание процесса установки QEMU/KVM в Void Linux.
tags:
  - QEMU/KVM
  - QEMU
  - KVM
  - Void Linux
  - Virtualization
---

2020-03-17

QEMU/KVM не требует DKMS-модуля для своей работы, в отличии от VirtualBox. Бла-бла-бла.

## Установка необходимых пакетов
Для эмуляции на основе `qemu/kvm` необходимы следующие компоненты:
- `virt-manager`&nbsp;&mdash; GUI для работы с `libvirt` и `qemu`. Позволяет подключаться к локальному и/или удалённому экземпляру `qemu/kvm`. Поключение к удалённому экземпляру `libvirt` и просмотр консоли удалённой эмулируемой машины обеспечивает ssh-туннель.
- `qemu`&nbsp;&mdash; программа, умеющая эмулировать различные процессоры и другое оборудование.
- `libvirt`&nbsp;&mdash; библиотека для работы с `kvm` из *пространства пользователя*.
- `kvm`&nbsp;&mdash; гипервизор, модуль находящийся в ядре системы, значительно ускоряющий выполнение эмулируемой машины в `qemu`.

```
$ sudo xbps-install -Su virt-manager libvirt qemu
Name               Action    Version           New version            Download size
gnome-ssh-askpass  install   -                 8.1p1_1                9060B
libbsd             install   -                 0.10.0_1               41KB
openbsd-netcat     install   -                 1.206_1                21KB
yajl               install   -                 2.1.0_4                21KB
libvirt-glib       install   -                 2.0.0_3                141KB
gtk-vnc            install   -                 0.9.0_3                97KB
osinfo-db          install   -                 20191125_1             112KB
libosinfo          install   -                 1.7.1_1                569KB
libvirt-python3    install   -                 6.0.0_1                130KB
libxml2-python3    install   -                 2.9.10_2               122KB
virt-manager-tools install   -                 2.2.1_1                164KB
virt-manager       install   -                 2.2.1_1                846KB
dnsmasq            install   -                 2.80_7                 194KB
libnuma            install   -                 2.0.13_1               17KB
libapparmor        install   -                 2.13.3_4               90KB
libvirt            install   -                 6.0.0_1                5175KB
libvde2            install   -                 2.3.2_21               25KB
libbluetooth       install   -                 5.52_1                 45KB
spice              install   -                 0.14.2_2               318KB
virglrenderer      install   -                 0.8.1_1                176KB
libpcsclite        install   -                 1.8.26_1               22KB
libcacard          install   -                 2.7.0_1                30KB
qemu               install   -                 4.2.0_2                100MB
========================================================================
To enable KVM your user must be added to the 'kvm' group:

        $ usermod -aG kvm <username>

Don't forget to load the appropiate KVM module for your CPU (x86 only):

        $ modprobe kvm-amd # for AMD CPUs
        $ modprobe kvm-intel # for Intel CPUs
========================================================================
```

## Дополнительные телодвижения
Добавляем себя к группам пользователей-эмуляторов:
```
$ sudo usermod -aG kvm $USER
$ sudo usermod -aG libvirt $USER
```

Загружаем подходящий модуль (здесь для процессора Intel) и запускаем демоны:
```
modprobe kvm-intel
ln -s /etc/sv/libvirtd /var/service
ln -s /etc/sv/virtlockd /var/service
ln -s /etc/sv/virtlogd /var/service
```

После выхода из своего сеанса и повторного входа, или перезагрузки компьютера, появляется возможность запустить `virt-manager` и создать/добавить первую виртуальную машину.

## Инструменты настройки
Настройка производится с помощью:
- `virsh`&nbsp;&mdash; консольная утилита;
- `virt-manager`&nbsp;&mdash; относительно удобная GUI-утилита.

## Настройка хранилищ (pool)
Хранилища&nbsp;&mdash; это места хранения образов винчестеров виртуальных машин, образов DVD-дисков. Хранилищем может стать каталог файловой системы, или выделенный раздел на LVM2. Перед работой с ними их необходимо добавить в систему.

По умолчанию доступно одно хранилище `/var/lib/libvirt/images`.

## Настройка сети для QEMU/KVM

### Virtual network 'default' : NAT
При запуске демона `libvirtd` в систему автоматически добавляется виртуальный сетевой интерфейс `virbr0` с отдельной подсеткой `192.168.122.0/24`, а в netfilter дописываются правила, с помощью которых, через NAT, виртуальные машины смогут иметь доступ к инету:
```
*mangle
-N LIBVIRT_PRT
-A POSTROUTING -j LIBVIRT_PRT
-A LIBVIRT_PRT -o virbr0 -p udp -m udp --dport 68 -j CHECKSUM --checksum-fill

*nat
-N LIBVIRT_PRT
-A POSTROUTING -j LIBVIRT_PRT
-A LIBVIRT_PRT -s 192.168.122.0/24 -d 224.0.0.0/24 -j RETURN
-A LIBVIRT_PRT -s 192.168.122.0/24 -d 255.255.255.255/32 -j RETURN
-A LIBVIRT_PRT -s 192.168.122.0/24 ! -d 192.168.122.0/24 -p tcp -j MASQUERADE --to-ports 1024-65535
-A LIBVIRT_PRT -s 192.168.122.0/24 ! -d 192.168.122.0/24 -p udp -j MASQUERADE --to-ports 1024-65535
-A LIBVIRT_PRT -s 192.168.122.0/24 ! -d 192.168.122.0/24 -j MASQUERADE

*filter
-N LIBVIRT_FWI
-N LIBVIRT_FWO
-N LIBVIRT_FWX
-N LIBVIRT_INP
-N LIBVIRT_OUT
-A INPUT -j LIBVIRT_INP
-A LIBVIRT_FWI -d 192.168.122.0/24 -o virbr0 -j ACCEPT
-A LIBVIRT_FWI -o virbr0 -j REJECT --reject-with icmp-port-unreachable
-A LIBVIRT_FWO -s 192.168.122.0/24 -i virbr0 -j ACCEPT
-A LIBVIRT_FWO -i virbr0 -j REJECT --reject-with icmp-port-unreachable
-A LIBVIRT_FWX -i virbr0 -o virbr0 -j ACCEPT
-A LIBVIRT_INP -i virbr0 -p udp -m udp --dport 53 -j ACCEPT
-A LIBVIRT_INP -i virbr0 -p tcp -m tcp --dport 53 -j ACCEPT
-A LIBVIRT_INP -i virbr0 -p udp -m udp --dport 67 -j ACCEPT
-A LIBVIRT_INP -i virbr0 -p tcp -m tcp --dport 67 -j ACCEPT
-A LIBVIRT_OUT -o virbr0 -p udp -m udp --dport 53 -j ACCEPT
-A LIBVIRT_OUT -o virbr0 -p tcp -m tcp --dport 53 -j ACCEPT
-A LIBVIRT_OUT -o virbr0 -p udp -m udp --dport 68 -j ACCEPT
-A LIBVIRT_OUT -o virbr0 -p tcp -m tcp --dport 68 -j ACCEPT
```

Также запускается `dnsmasq` для раздачи ip-адресов из назначенной подсети.

Описание сети `default` находится в `/etc/libvirt/qemu/networks/default.xml`. Вручную изменять этот файл не рекомендуется, и хорошей практикой будет использование команды `virsh net-edit default`, которая запустит редактор `vi`, а после завершения редактирования, проверит файл на соответствие правилам.
```
<network>
  <name>default</name>
  <bridge name="virbr0"/>
  <forward mode="nat"/>
  <ip address="192.168.122.1" netmask="255.255.255.0">
    <dhcp>
      <range start="192.168.122.2" end="192.168.122.254"/>
    </dhcp>
  </ip>
</network>
```

Естественно, если демоны `iptables` и `ip6tables` запускаются после `libvirtd`, то iptables-правила для виртуальной сетки будут затёрты. Для решения этой проблемы дописываем в самое начало runit-скрипта `/etc/sv/libvirtd/run` две строки, чтобы `runit` сам расставил зависимости:
```
sv start iptables || exit 1
sv start ip6tables || exit 1
```

Конечно, будет лучше взять правила для файрвола под свой контроль, так как автоматически добавляемые правила не слишком эффективны и не всегда применяются должным образом.

Отменяем автоматическое создание сети командой `virsh net-autostart default --disable`, в результате чего будет удалена ссылка `/etc/libvirt/qemu/networks/autostart/@default.xml`.

### Shared device
Чтобы выставить новую виртуальную машину в текущую локальную сеть создаём мост, в составе которого будут работать текущий физический сетевой интерфейс и сетевые интерфейсы виртуальных машин.

Выключаем демон `dhcpcd`, чтобы не мешал работе `NetworkManager`. В моём случае `dhcpcd` был включен, несмотря на используемый `NetworkManager`, и настойчиво назначал ip-адрес физическому интерфейсу, тогда как `NetworkManager` назначал эти же данные на мост `br0`:
```
sv stop dhcpcd
rm /var/service/dhcpcd
```

Определяем текущее сетевое подключение. В выводе информации о текущих подключениях видим  "Wired connection 1":
```
# nmcli d
DEVICE      TYPE      STATE           CONNECTION         
enp3s0      ethernet  подключено      Wired connection 1
virbr0      bridge    подключено      virbr0
```

После выяснения текущего сетевого соединения выполняем:
1. Выключаем текущее сетевое подключение "Wired connection 1".
2. Отключаем автоподнятие интерфейса "Wired connection 1".
2. Создаём мост `br0`.
3. Добавляем текущий физический сетевой интерфейс в этот мост.
4. Отключаем stp, так как вероятность петли на интерфейсе `br0`, по понятным причинам, стремится к нулю.
5. Поднимаем мост `br0`.

```
nmcli con down "Wired connection 1"
nmcli con mod 'Wired connection 1' connection.autoconnect off
nmcli con add type bridge ifname br0 con-name br0
nmcli con add type bridge-slave ifname enp3s0 master br0 con-name enp3s0s
nmcli con mod br0 bridge.stp no
nmcli con up br0
```

Если в локальной сети присутствует dhcp-сервер, то на `br0` назначится ip-адрес, иначе назначение проводим вручную.

В заключении, указываем новой виртуальной машине `br0` в качестве "Shared physical device".

### Host device eth0: macvtap
Вспомнить. Несколько лет назад. ovh.com. Дополнительный failover ip для которого OVH назначает свой MAC (по моему запросу)? В этом случае я использовал macvtap (с указанием MAC от OVH) для виртуального сервера.
