---
title: "Void Linux Послеустановочные шаги"
date: 2020-04-08
weight: 10
description: >
  Описание телодвижений после установки Void Linux, для приведения его в то состояние, которое удовлетворяет моим желаниям.
tags:
  - Void Linux
---

2020-04-08

Для своего нетбука Asus 1215n я выбрал [дистрибутив Void Linux с `libc`-библиотекой и Mate для графической оболочки](https://a-hel-fi.m.voidlinux.org/live/current/void-live-x86_64-20191109-mate.iso), так как для дистрибутива с `musl`-библиотекой отсутствует возможность установки проприетарных драйверов для видеокарты "Nvidia ION".

Для стационарного компьютера, с "NVidia GeForce GT 630" на борту, мной был выбран [Void Linux, также с `libc`-библиотекой + Cinnamon](https://a-hel-fi.m.voidlinux.org/live/current/void-live-x86_64-20191109-cinnamon.iso)

Установка с флэшки быстра и незатейлива и здесь приведена не будет. Отмечу только, что для корневой ФС выбираю BtrFS.

### Обновление и подключение дополнительных репо
После перезагрузки дважды выполняем обновление системы. Второе обновление мне понадобилось, чтобы разрешить зависимости пакетов, установленных при первом обновлении:
```
xbps-install -Suy
xbps-install -Suy
reboot
```

Удаляем старые ядра Linux, кроме последнего:
```
vkpurge rm all
```

Делаем запрос на получение списка доступных репозиториев и подключаем три из них:
```
# xbps-query -Rs void-repo
[-] void-repo-debug-9_4            Void Linux drop-in file for the debug repository
[-] void-repo-multilib-6_3         Void Linux drop-in file for the multilib repository
[-] void-repo-multilib-nonfree-6_3 Void Linux drop-in file for the multilib/nonfree repository
[-] void-repo-nonfree-9_4          Void Linux drop-in file for the nonfree repository

# xbps-install -Su void-repo-multilib void-repo-multilib-nonfree void-repo-nonfree
```

### Безопасность SSH
В Void Linux по умолчанию устанавливается демон `sshd` с настройками, позволяющими авторизоваться root по паролю. В современных условиях это потенциальная дыра в безопасности системы. Без промедления исправляем:
- останавливаем демон sshd:  
`# sv stop sshd`
- при необходимости запрещаем запуск демона sshd:  
`# rm /var/service/sshd`

Замечу, что я использую ssh ключи ed25519, поэтому в файле `/etc/ssh/sshd_config`:
- комментирую все host-ключи, кроме `/etc/ssh/ssh_host_ed25519_key`
- далее запрещаем вход для root'а:  
`PermitRootLogin no`
- разрешаем вход по открытому ключу:  
`PubkeyAuthentication yes`
- запрещаем вход по паролю:  
`PasswordAuthentication no`

### Отключение аккаунта root
Так как моя учётная запись пользователя учавствует в группе `wheel`, то отключаю `root` за ненадобностью:
```
$ sudo passwd -dl root
```

Так как root теперь отключён, то загрузка в recovery mode завершится сообщением о невозможности ввода пароля для root по причиние его отключённого состояния и предложением прочитать sulogin(8). Из чтения данного мана выясняется, что единственный способ выйти из этого положения, это разрешить беспарольный вход, то есть без запроса пароля root'а. Реализуется этот способ добавлением ключа `-e` или `--force` к `sulogin`.

В файле `/etc/sv/sulogin/run` в строке `exec setsid sulogin ${OPTS:=-p} < $tty >$tty 2>&1`, к опциям запуска sulogin добавляем ключ `-e`:
```
exec setsid sulogin ${OPTS:=-pe} < $tty >$tty 2>&1
```

Внимание!!! Данный беспарольный способ входа, по понятным причинами, является дырой в безопасности. Поэтому я его использовал только для проведения манипуляций с BTRFS из параграфа "Timeshift". После проведения работ, беспарольный sulogin мной был вновь отключен. Что, конечно же, не помешает подготовленному злоумышленнику загрузиться с флэшки.

Если использовать FDE (полнодисковое шифрование), то, вероятно, `sulogin -pe` можно применять без страха компрометации компьютера.

### Отключение "смягчения" уязвимостей для старого ноута
Надо проверить, имеет ли смысл отключать "смягчения", так как в старом ноуте "Asus 1215n" установлен процессор "Intel Atom", который не подвержен некоторым узвимостям, а значит замедление работы, вследствии применения "смягчений", может быть незначительным.

В `/etc/default/grub` добавляем в параметры загрузки ядра `mitigations=off`:
```
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=4 slub_debug=P page_poison=1 mitigations=off"
```

Обновляем GRUB и перегружаемся:
```
$ sudo update-grub
$ sudo reboot
```

### Включение Firewall
[Firewall Configuration](https://wiki.voidlinux.org/Firewall_Configuration)
По умолчанию в Void Linux установлен `iptables`. Как обычно, правила храняться в `/etc/iptables/iptables.rules` и `/etc/iptables/ip6tables.rules`.

Пример `iptables.rules` :
```
*filter
:INPUT DROP
:FORWARD DROP
:OUTPUT ACCEPT
-A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A INPUT -i lo -j ACCEPT
#-A INPUT -m conntrack --ctstate INVALID -j LOG --log-prefix "IPV4 INVALID packets: "
-A INPUT -m conntrack --ctstate INVALID -j DROP
# incoming ping & traceroute
#-A INPUT -p icmp -m icmp --icmp-type echo-request -m conntrack --ctstate NEW -j ACCEPT
#-A INPUT -p udp -m udp --dport 33434:33534 -m conntrack --ctstate NEW -j ACCEPT
# ssh
#-A INPUT -p tcp -m tcp --dport 22 -m conntrack --ctstate NEW -j ACCEPT
# logging drop packets
#-A INPUT -j LOG --log-prefix "IPV4 DROP packets: "
-A INPUT -j DROP
COMMIT

*nat
:PREROUTING ACCEPT
:INPUT ACCEPT
:OUTPUT ACCEPT
:POSTROUTING ACCEPT
COMMIT

*mangle
:PREROUTING ACCEPT
:INPUT ACCEPT
:FORWARD ACCEPT
:OUTPUT ACCEPT
:POSTROUTING ACCEPT
COMMIT

*raw
:PREROUTING ACCEPT
:OUTPUT ACCEPT
COMMIT

```

Пример `ip6tables.rules` :
```
*filter
:INPUT DROP
:FORWARD DROP
:OUTPUT ACCEPT
-A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A INPUT -i lo -j ACCEPT
#-A INPUT -m conntrack --ctstate INVALID -j LOG --log-prefix "IPV6 INVALID packets: "
-A INPUT -m conntrack --ctstate INVALID -j DROP
# icmpv6
-A INPUT -p icmpv6 --icmpv6-type router-advertisement -m conntrack --ctstate UNTRACKED -m hl --hl-eq 255 -j ACCEPT
-A INPUT -p icmpv6 --icmpv6-type neighbour-advertisement -m conntrack --ctstate UNTRACKED -m hl --hl-eq 255 -j ACCEPT
-A INPUT -p icmpv6 --icmpv6-type neighbour-solicitation -m conntrack --ctstate UNTRACKED -m hl --hl-eq 255 -j ACCEPT
# incoming ping traceroute
#-A INPUT -p icmpv6 -m icmpv6 --icmpv6-type echo-request -m conntrack --ctstate NEW -j ACCEPT -m limit --limit 5/second
#-A INPUT -p udp -m udp --dport 33434:33534 -m conntrack --ctstate NEW -j ACCEPT
# ssh
#-A INPUT -p tcp -m tcp --dport 22 -m conntrack --ctstate NEW -j ACCEPT
# logging drop packets
#-A INPUT -j LOG --log-prefix "IPV6 DROP packets: "
-A INPUT -j DROP
COMMIT

*nat
:PREROUTING ACCEPT
:INPUT ACCEPT
:OUTPUT ACCEPT
:POSTROUTING ACCEPT
COMMIT

*mangle
:PREROUTING ACCEPT
:INPUT ACCEPT
:FORWARD ACCEPT
:OUTPUT ACCEPT
:POSTROUTING ACCEPT
COMMIT

*raw
:PREROUTING ACCEPT
:OUTPUT ACCEPT
COMMIT

```
</p></detailes><br>

Для восстановления правил после перезагрузки включаем демоны:
```
ln -s /etc/sv/iptables /var/service/
ln -s /etc/sv/ip6tables /var/service/
```

### Служба времени
```
xbps-install -S chrony
ln -s /etc/sv/chronyd/ /var/service/
```

`chronyc makestep` &ndash; для немедленной синхронизации.

### Локализация и русификация консоли
Подправляем локаль для правильной работы программ, в частности `mosh`, `gnome-terminal`, клиента NextCloud. Также русифицируем консоль с переключением клавиатурных раскладок по "Alt+Shift".

В `/etc/default/libc-locales` раскомментируем необходимые нам локали и запускаем их генерацию (для `ru_RU.UTF-8`):
```
sed -i -e "s|#ru_RU.UTF-8|ru_RU.UTF-8|" /etc/default/libc-locales
xbps-reconfigure -f glibc-locales
```

Задаём локаль по умолчанию:
```
# echo "LANG=ru_RU.UTF-8" > /etc/locale.conf
```

Выбираем консольный шрифт [Terminus](http://terminus-font.sourceforge.net/) размером 24 и полужирным начертанием, и добавляем в консоль переключение между языками по "Alt+Shift". Для этого изменяем соответствующие строки в `/etc/rc.conf`:
```
KEYMAP="ruwin_alt_sh-UTF-8"
FONT="ter-v24b"
```

Изменения вступят в силу после перезагрузки.

Заметка: консольные шрифты хранятся в `/usr/share/kbd/consolefonts`. Сверяясь с этим каталогом и переключившись в консоль, подбираем для себя шрифт командой `setfont`. Изменения происходят мгновенно. Название файла с шрифтом можно задавать любым способом, показанным ниже. Например:
```
$ setfont /usr/share/kbd/consolefonts/UniCyrExt_8x16.psf.gz
$ setfont /usr/share/kbd/consolefonts/ruscii_8x16
$ setfont ter-v24n
```

### Logging
По умолчанию в Void Linux отсутствует логгирование. При необходимости в нём, на странице [Logging in Void Linux](https://docs.voidlinux.org/config/services/logging.html), предлагается установить `socklog`:
```
xbps-install -S socklog-void
ln -s /etc/sv/socklog-unix /var/service/
ln -s /etc/sv/nanoklogd /var/service/
```

Запись логов ведётся в `/var/log/socklog/`. Для их чтения добавляем себя в группу `socklog`:
```
$ sudo usermod -aG socklog $USER
```

### Добавление часто используемого ПО
Устанавливаем часто используемые пакеты:
```
$ sudo xbps-install -Su unzip atom vscode mosh mc chromium opera \
torbrowser-launcher keepassxc encfs libreoffice tint2 stellarium \
net-tools mtr NetworkManager-openvpn wireshark-qt zenmap tcpdump \
calibre atril file-roller mplayer vlc gnome-calculator gnome-system-monitor \
gedit clamav rkhunter chkrootkit smartmontools parted remmina \
telegram-desktop cool-retro-term font-3270 ntfs-3g attr-progs acl-progs \
libcap-ng-progs vsv bootiso bridge-utils io.elementary.photos

$ sudo usermod -a -G wireshark $USER
```

### VGA
#### nvidia340
Так как мой старенький нетбук имеет дополнительный видеочип на "Nvidia ION", а его поддержка закончилась на 340-ой версии драйверов от Nvidia, то устанавливаем соответствующий драйвер; добавляем свой логин в группу пользователей этого видеочипа, и добавляем демон `bumblebeed` в автозагрузку:
```
$ sudo xbps-install -Su nvidia340 bumblebee bbswitch
$ sudo usermod -a -G bumblebee $USER
$ sudo ln -s /etc/sv/bumblebeed /var/service
```

После ближайшей перезагрузки запускаем различные тесты для проверки работы bumblebee и изменяем пункты меню тех приложений, которым требуется для запуска GLX:
```
$ optirun glxgears -info
$ optirun glxspheres64
$ optirun stellarium
$ optirun cool-retro-term
```

#### nvidia390
На моём стационарном компе установлен GeForce GT 630, для которого, судя по информации с сайта nvidia, поддержка закончилась на 390-ой версии.
```
$ sudo xbps-install -Su nvidia390
```

#### Intel GPU
Второй монитор в "Linux Mint" был обнаружен и подключён автоматически. Здесь счастья не случилось. Пока не знаю способа решения.
<!--
На моём стационарном компе доступна встроенная intel gpu.
[Intel GPU - Void Linux Handbook](https://docs.voidlinux.org/config/graphics/intel.html)
```
$ sudo xbps-install -S mesa-dri intel-video-accel
```
-->

### Wifi adapter Broadcom BCM4313
Можно оставаться на драйвере, разрабатываемом сообществом. Или, при желании, ищем подходящие проприетарные драйвера в репозиториях и устанавливаем:
```
# xbps-query -Rs broadcom
[-] b43-fwcutter-019_3                 Firmware extraction tool for Broadcom wireless driver
[-] broadcom-bt-firmware-12.0.1.1011_1 Broadcom Bluetooth firmware for Linux kernel
[-] broadcom-wl-dkms-6.30.223.271_8    Broadcom proprietary wireless drivers for Linux - DKMS kernel module

# xbps-install -Su broadcom-wl-dkms
```

Интересно, что после установки проприетарных драйверов, соединение с базовой станцией (Mikrotik) происходит не с первой попытки. Иногда только после закрытия/открытия крышки нетбука, то есть через спящий режим. Позже попробуем разобраться в причинах такого поведения, а пока нащупал костыль: `# rmmod wl cfg80211 && sleep 1 && modprobe wl`, после применения которого происходит мгновенное подключение.

### NextCloud
```
# xbps-install -Su nextcloud-client libgnome-keyring
...
========================================================================
To actually use qtkeychain-qt5 you need to either have kwallet or
libgnome-keyring installed.
========================================================================
qtkeychain-qt5-0.9.1_1: installed successfully.
nextcloud-client-2.6.0_1: configuring ...
Updating GTK+ icon cache for /usr/share/icons/hicolor...
Updating MIME database...
nextcloud-client-2.6.0_1: post-install message:
========================================================================
NextCloud client end-to-end encryption (e2e) is currently unavailable
(LibreSSL 2.9.2 does not provide EVP_PKEY_CTX_set_rsa_oaep_md primitive)
```
`libgnome-keyring` &ndash; для подавления запроса авторизации в браузере.

### Timeshift
По умолчанию в Void Linux не предусмотрен какой-нибудь *планировщик задач*, поэтому устанавливаем его для обеспечения запуска задач из `timeshift`:
```
xbps-install -Su timeshift cronie libgee
ln -s /etc/sv/crond /var/service/
```

Void Linux устанавливается в btrfs-раздел без использования *подтомов*, тогда как Timeshift нацелен на их использование в стиле Ubuntu, когда система размещена в *подтоме* **@**, а `/home` в подтоме **@home**. Для соблюдения этих условий необходимо:

1. Загрузиться в Recovery Mode, или с *live*-флэшки, например voidlinux. При прохождении этого квеста из режима восстановления, по понятным причинам, выполнение пункта 4 необходимо отложить до загрузки в нормальном режиме.
2. Примонтировать системный раздел (у меня sda3) к, например, `/mnt`:
`mount /dev/sda3 /mnt`
3. Создать снимок корня системного раздела в *подтом* с названием `@` (древний символ, использовавшийся торговцами для учёта амфор с вином или оливковым маслом):
`btrfs subvolume snapshot /mnt /mnt/@`
4. Удалить содержимое корневого тома, кроме подтома `@` (только не в Recovery Mode):
`cd /mnt`
`find -maxdepth 1 ! -name @ -exec rm -rf {} \;`
5. в файле `/mnt/@/etc/fstab`, в строке монтирования системного раздела, после `defaults` через запятую, добавить опцию `subvol=@` (здесь приведён результат работы через `vi`):
`vi /mnt/@/etc/fstab`
`UUID=b8d4b8a5-0392-41e4-ad14-1255f12321b6 / btrfs defaults,subvol=@ 0 1`
6. Отмонтировать системный раздел:
`umount /mnt`
7. Примонтировать к `/mnt` только что созданный подтом `@` :
`mount -o subvol=@ /dev/sda3 /mnt`
8. Монтируем важные каталоги и делаем `chroot` в `/mnt`:
`mount -t proc /proc /mnt/proc`
`mount --bind /dev /mnt/dev`
`mount --bind /sys /mnt/sys`
`mount --bind /run /mnt/run`
`chroot /mnt /bin/bash`
9. Обновляем загрузчик системы и выходим из песочницы:
`update-grub`
`grub-install /dev/sda`
`exit`
10. Перегружаем компьютер:
`sync && reboot`
11. После загрузки в нормальном режиме, при необходимости удаления каталогов, как в пунке 4, монтируем корневой подтом файловой системы BTRFS к каталогу `mnt`. Так как корень BTRFS всегда имеет ID=5, то его монтирование, удаление ненужных каталогов, с последующим отмонтированием, делаем так:
`mount -o subvolid=5 /dev/sda3 /mnt`
`cd /mnt`
`find -maxdepth 1 ! -name @ -exec rm -rf {} \;`
`umount /mnt`

При наличии UEFI, перед выполнением перемещения в песочницу `chroot` в пункте 8, необходимо дополнительно примонтировать efi-раздел (у меня sda2), например: `mount /dev/sda2 /mnt/boot/efi`. А после, находясь в песочнице, при установке загрузчика (пункт 9), выполнить: `grub-install` без указания устройства.

### powertop
```
xbps-install -Su powertop xset
echo "powertop --auto-tune" >> /etc/rc.local
```

### Добавление МФУ HP LaserJet Pro 200 MFP M276nw
[SANE - Void Linux Wiki](https://wiki.voidlinux.org/SANE)
Установил необходимые пакеты:
```
$ sudo xbps-install -Su sane xsane simple-scan hplip gnupg system-config-printer
```

Добавил себя в группы:
```
$ sudo usermod -aG lp,lpadmin $USER
```

Запустил CUPS:
```
$ sudo ln -s /etc/sv/cupsd /var/service/
```

Запустил процедуру установки МФУ:
```
$ sudo hp-setup -i 192.168.120.10
```

Добавил в `/etc/sane.d/dll.conf` строку `hpaio`:
```
$ sudo sh -c 'echo "hpaio" >> /etc/sane.d/dll.conf'
```

Проверка сканера:
```
$ scanimage -L
device `hpaio:/net/HP_LaserJet_200_colorMFP_M276nw?ip=192.168.120.10' is a Hewlett-Packard HP_LaserJet_200_colorMFP_M276nw all-in-one
```

Указание предпочтительного размера печатного листа:
```
# sudo sh -c 'echo a4 >> /etc/papersize'
```

### TeamViewer
Для подключения к другим компьютерам через TeamViewer, необходимо установить в систему два пакета:
```
# xbps-install -Su qt5-webkit qt5-quickcontrols
```
Скачиваем [teamviewer for linux](https://download.teamviewer.com/download/linux/teamviewer_amd64.tar.xz) и распаковываем в, например, `/opt`. Запуск teamviewer производим под обычным пользователем `$ /opt/teamviewer/teamviewer`. В случае неудачи можем проверить наличие в системе требуемых библиотек `$ /opt/teamviewer/tv-setup checklibs`.

### Помощь проекту статистикой
[Usage Statistics](https://docs.voidlinux.org/contributing/usage-statistics.html)
Посмотреть собираемые данные можно [здесь](http://popcorn.voidlinux.org/). Каждые 24 часа PopCorn посылает информацию об установленных пакетах и их версиях, версии ядра, архитектуре процессора, и ещё пара-тройка параметров. Каждый клиент генерирует уникальный UUID, на основе которого можно отслеживать источники статистики. Поэтому мой стационарный комп посылает статистику, а ноут пока не посылает. Я параноик, но не совсем.

Скачиваем и запускаем PopCorn:
```
xbps-install PopCorn
ln -s /etc/sv/popcorn /var/service/
```

### Прочее
Настройку NetworkManager, Tint2 и прочее здесь описывать не буду.
