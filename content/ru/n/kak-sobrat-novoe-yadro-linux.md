---
title: "Как собрать новое ядро linux"
date: 2011-04-01
weight: 10
description: >
  Как собрать новое ядро linux
tags:
  - Linux Kernel
  - make
---

2011-04-01

Взято с: http://ebash.in/howto/Kak-sobrat-svoe-yadro-v-Debian-Etch

В каждом дистрибутиве имеется своя специфика сборки ядра и это HowTo ориентировано именно на то, как это сделать в Debian Etch. Так же раскрывается вопрос, как наложить тот или иной патч на ядро, когда необходима поддержка определенной функциональности или нового оборудования в Вашей системе. HowTo предназначено в первую очередь на более подготовленных пользователей и нет никаких гарантий, что этот способ будет работать так, как надо и все описанные действия и ответственность ложатся на Вас.

Примечание.

Будет описано два способа сборки ядра. Первым будет описан вариант сборки .deb пакетов, которые могут быть установлены в Вашей или другой системе. Второй метод, так называемый "traditional" way :-)

## Способ первый. Сборка ядра в .deb пакеты.

### Установка необходимых пакетов для компиляции ядра.
Для начала обновим списки пакетов:
```
# apt-get update
```
Установим нужные нам пакеты:
```
# apt-get install kernel-package libncurses5-dev fakeroot wget bzip2 build-essential
```
### Скачиваем исходники ядра.
Переходим в каталог `/usr/src`, идем на www.kernel.org и выбираем нужную версию ядра. В данном случае будет рассмотрена версия `linux-2.6.23.1.tar.bz2`. Скачиваем:
```
# cd /usr/src
# wget http://www.kernel.org/pub/linux/kernel/v2.6/linux-2.6.23.1.tar.bz2
```
Распакуем исходники и создадим символьную ссылку:
```
 # tar xjf linux-2.6.23.1.tar.bz2
 # ln -sf linux-2.6.23.1 linux 
 # cd /usr/src/linux
```
### Накладываем патчи.
**Опционально и без необходимости не делайте этого!**

Иногда требуются драйвера или средства, которые не поддерживаются в имеющемся ядре, например технология виртуализации или иная другая специфика, которой нет в текущем релизе. В любом случае это исправляется наложением так называемых патчей (если таковые имеются).

Итак, предположим вы скачали необходимый патч (для примера назовем `patch.bz2`) в `/usr/src`. Применим скачанный патч на наши исходники (Вы должны быть все еще в каталоге `/usr/src/linux`):
```
# bzip2 -dc /usr/src/patch.bz2 | patch -p1 --dry-run
# bzip2 -dc /usr/src/patch.bz2 | patch -p1
```
Первая команда -- только тест и никакие изменения не будут применены к исходникам. Если после первой команды не было выдано никаких ошибок, можно выполнить вторую команду для применения патча. Ни в коем случае не выполняйте вторую команду, если после первой были выданы ошибки!

Таким образом Вы можете накладывать патчи на исходники ядра. Например, имеются некоторые особенности, которые доступны только в 2.6.23.8 ядре, а исходники не содержали необходимой функциональности, но выпущен патч `patch-2.6.23.8.bz2`. Вы можете применить этот патч к исходникам ядра 2.6.23, но не 2.6.23.1 или 2.6.23.3 и т.д.

Предъисправления (препатчи) - эквивалентны альфа релизам; патчи должны быть применены к исходникам полного предыдущего релиза с 3-х значной версией (например, патч 2.6.12-rc4 может быть применен к исходникам версии 2.6.11, но не к версии 2.6.11.10.)

Это значит, если мы хотим собрать ядро 2.6.23.8, необходимо скачать исходники версии 2.6.23 (http://www.kernel.org/pub/linux/kernel/v2.6/linux-2.6.23.tar.gz) применительно во втором способе "traditonal" way!

Применяем патч patch-2.6.23.8.bz2 к ядру 2.6.23:
```
# cd /usr/src
# wget http://www.kernel.org/pub/linux/kernel/v2.6/patch-2.6.22.8.bz2
# cd /usr/src/linux
# bzip2 -dc /usr/src/patch-2.6.23.8.bz2 | patch -p1 --dry-run
# bzip2 -dc /usr/src/patch-2.6.23.8.bz2 | patch -p1
```
**Дополнение**:

Можно скачать файлик с расширением patch в `/usr/src/linux` и применить:
```
$ cat file.patch | patch -p1
```
### Конфигурирование ядра.
Неплохой идеей будет использование существующего конфигурационного файла работающего ядра и для нового. Поэтому копируем существующую конфигурацию в `/usr/src/linux`:
```
# make clean && make mrproper
# cp /boot/config-`uname -r` ./.config
```
Далее даем команду:
```
# make menuconfig
```
после которой загрузится графическое меню конфигурации ядра. Выбираем в меню конфигуратора пункт "Load an Alternate Configuration File" и нажимаем "Оk". Затем (если требуется) сделайте необходимые изменения в конфигурации ядра перемещаясь по меню (подробности конфигурации ядра можно найти в www.google.com :-) ). Когда закончите и нажмете "Exit", будет задан вопрос "Do you wish to save your new kernel configuration?", отвечаем утвердительно "Yes".

### Компиляция ядра.
Сборка ядра выполняется всего в две команды:
```
# make-kpkg clean
# fakeroot make-kpkg --initrd --append-to-version=-cybermind kernel_image kernel_headers
```
После `--append-to-version=`, можно написать любое название, какое Вам угодно, но оно должно начинаться со знака минус (-) и не иметь пробелов.

Процесс компиляции и сборки .deb пакетов может занят довольно продолжительное время. Все будет зависить от конфигурации ядра и возможностей Вашего процессора.

### Установка нового ядра.
Когда удачно завершится сборка ядра, в каталоге /usr/src будут созданы два .deb пакета:
```
# cd /usr/src
# ls -l
```
`linux-image-2.6.23.1-cybermind_2.6.23.1-cybermind-10.00.Custom_i386.deb` - собственно само актуальное ядро и

`linux-headers-2.6.23.1-cybermind_2.6.23.1-cybermind-10.00.Custom_i386.deb` - заголовки ядра, необходимые для сборки других модулей (например при сборке модулей драйвера nVidia).

Инсталлируем их:
```
# dpkg -i linux-image-2.6.23.1-cybermind_2.6.23.1-cybermind-10.00.Custom_i386.deb
# dpkg -i linux-headers-2.6.23.1-cybermind_2.6.23.1-cybermind-10.00.Custom_i386.deb
```
(Эти пакеты теперь могут быть установлены на другой системе и собирать их заново уже не будет необходимости.)

Всё, инсталяция завершена, меню загрузчика, установка нового рамдиска и ядра будут сделаны автоматически. Остается только перезагрузиться:
```
# reboot
```

## Способ второй. "traditional" way :-)

Выполняем все пункты, описанные выше ДО пункта "Компиляция ядра".

Далее, традиционный способ:
```
# make all
# make modules_install
# make install
```
Как обычно, сборка может занять продолжительное время, зависящее от конфигурации ядра и возможностей процессора.

Следующие шаги:

Ядро собрано и установлено, но еще теперь необходимо создать рамдиск (без которого ядро просто не загрузится) и необходимо обновить загрузчик GRUB. Для этого выполним следующее:
```
# depmod 2.6.23.1
# apt-get install yaird
```
Установим рамдиск:
```
# mkinitrd.yaird -o /boot/initrd.img-2.6.23.1 2.6.23.1
```
Обновим легко и безболезненно загрузчик:
```
# update-grub
```
Всё, загрузчик и новое ядро готовы, остается только перезагрузиться:
```
# reboot
```

## Проблемы.

Если после перезагрузки, выбранное вами новое ядро не загружается, перезагрузитесь и выберите Ваше предыдущее ядро и можно попробовать проделать весь процесс заново, чтоб собрать рабочееядро. Не забывайте в таком случае удалить строчки нерабочего ядра в `/boot/grub/menu.lst`.

Ссылки:
http://ebash.in/howto/Kak-sobrat-svoe-yadro-v-Debian-Etch
http://www.linuxcenter.ru/lib/articles/system/kernel26_install.phtml
http://www.opennet.ru/base/sys/install_linux_kernel.txt.html
http://www.mr-h1z.com/linux/ubuntu/%D0%A1%D0%BE%D0%B1%D0%B8%D1%80%D0%B0%D0%B5%D0%BC-%D1%81%D0%B2%D0%BE%D0%B5-%D1%8F%D0%B4%D1%80%D0%BE-%D0%B4%D0%BB%D1%8F-%D1%83%D0%B1%D1%83%D0%BD%D1%82%D1%8B
