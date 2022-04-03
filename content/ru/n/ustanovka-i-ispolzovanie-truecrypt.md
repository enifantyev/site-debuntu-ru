---
title: "Установка и использование Truecrypt"
date: 2011-03-30
weight: 10
description: >
  Установка и использование Truecrypt
tags:
  - TrueCrypt
---

<h2><a id="Установка Truecrypt" name="Установка Truecrypt"></a>Установка Truecrypt</h2>
<p>Установка truecrypt проста.</p>
<p>Устанавливаем необходимые пакеты для установки трукрипта:<br /># aptitude install libfuse2 fuse-utils dmsetup libsm6 libgtk2.0-0 sudo</p>
<p>Скачиваем с www.truecrypt.org соответствующий архив:<br /># lynx www.truecrypt.org</p>
<p>На сайте переходим на страницу downloads. Потом выбираем соответствующий архив и скачиваем его к себе. На текущий момент truecrypt-6.2a-ubuntu-x86.tar.gz.</p>
<p>Распаковываем его:<br /># tar -xvf truecrypt*.</p>
<p>Запускаем распакованный скрипт truecrypt-6.2a-setup-ubuntu-x86 и отвечаем на вопросы:</p>
<ul>
<li>Инсталлировать трукрипт или только распаковать? Выбираем 1 - инсталлировать.</li>
<li>Читаем лицензию, нажимая пробел</li>
<li>Согласен с написанным? Согласен.</li>
</ul>
<p>Установка завершена.</p>
<p>Можно добавить репозиторий <a href="http://notesalexp.blogspot.com/p/blog-page.html">http://notesalexp.blogspot.com/p/blog-page.html</a>:<br />deb http://notesalexp.org/debian/lucid/ lucid main</p>
<p>Но, возможно, надёжней юзать версию с сайта truecrypt.org.</p>
<h2><a id="Создание контейнера из командной строки" name="Создание контейнера из командной строки"></a>Создание контейнера из командной строки:</h2>
<p># truecrypt -t -c my_cont.vmdk<br />Отвечаем на вопросы. Файловую систему выбираем none.</p>
<p>Монтируем созданный контейнер на какую-нибудь созданную папку:<br /># truecrypt -t --filesystem=none /path/my_cont.vmdk /home/folder</p>
<p>Смотрим, как обзывается наш контейнер в системе:<br /># truecrypt -t -l<br />Вылезло: 1: /path/my_cont.vmdk /dev/mapper/truecrypt1</p>
<p>Форматируем наш контейнер:<br /># mkfs.ext3 /dev/mapper/truecrypt1</p>
<p>Размонтируем наш контейнер:<br /># truecrypt -t -d /path/my_cont.vmdk или truecrypt -t -d (размонтирование всех контейнеров)</p>
<p>Теперь опять монтируем и пользуемся:<br /># truecrypt -t /path/my_cont.vmdk /home/folder</p>
<p>Таким же макаром можно зашифровать раздел или винчестер:<br /># truecrypt -t -c /dev/sda5</p>
<p>Зашифровать системный раздел или весь системный винчестер не получится под Linux'ом, но реально под Windows.</p>
<p>У трукрипта есть и GUI.</p>
<p>Полезные ссылки:<br /><a href="http://www.cgsecurity.org/wiki/Recover_a_TrueCrypt_Volume" rel="noopener" target="_blank">Recover a TrueCrypt Volume</a></p>
