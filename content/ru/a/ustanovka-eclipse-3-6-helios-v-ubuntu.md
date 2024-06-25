---
title: "Установка Eclipse 3.6 (Helios) в Ubuntu"
date: 2010-02-19
weight: 10
description: >
  Установка Eclipse 3.6 (Helios) в Ubuntu
tags:
  - IDE
  - Eclipse
---

<p>2010-02-19</p>
<p>Ссылки: <a href="http://www.eclipse.org/">www.eclipse.org</a></p>
<h2><a id="Описание" name="Описание"></a>Описание</h2>
<h3><a id="Установка приложения без использования репозитория" name="Установка приложения без использования репозитория"></a>Установка приложения без использования репозитория</h3>
<p>Скачиваем с <a href="http://www.eclipse.org/downloads/">http://www.eclipse.org/downloads/</a> дистрибутив. Я выбрал Eclipse Classic 3.6.1, 169 MB для Linux 64 bit, так как у меня 64-битная Ubuntu. Полученный файл распаковал в /opt. Добавил ссылку на рабочий стол /opt/eclipse/eclipse.</p>
<h3><a id="Включение функциональности классического обновления" name="Включение функциональности классического обновления"></a>Включение функциональности классического обновления</h3>
<p>Ставим галку в меню Window - Preferences - General - Capabilities - Classic Update.</p>
<h3><a id="Подключение некоторых репозиториев" name="Подключение некоторых репозиториев"></a>Подключение некоторых репозиториев</h3>
<p>Установка дополнений, как и русификация, производится через меню Help - Install New Software. В появившемся окне “Install” нажимаем кнопку Add..., вызывая окно “Add repository”. В этом окне заполняем поле Name произвольным названием репозитория. А в поле Location: вставляем путь к репозиторию. После заполнения нажимаем Ok.<br />Окно “Add repository” исчезнет, а в нижележащем окне “Install” появится слово Pending. Ждём. Если с сетью всё в порядке, то через какое-то время появится список дополнений, которые можно установить из добавленного репозитория.<br />В дальнейшем в окне “Install” можно оперировать добавлением plugin’ов простым выбором необходимого репозитория из выпадающего меню.</p>
<h5><a id="Русификация" name="Русификация"></a>Русификация</h5>
<p>Ставим пакет Babel <a href="http://www.eclipse.org/babel/downloads.php">http://www.eclipse.org/babel/downloads.php</a>.<br />Name: Babel<br />Location: <a href="http://download.eclipse.org/technology/babel/update-site/R0.8.1/helios">http://download.eclipse.org/technology/babel/update-site/R0.8.1/helios</a></p>
<h5><a id="Пакет для разработки динамических web-приложений " name="Пакет для разработки динамических web-приложений "></a>Пакет для разработки динамических web-приложений</h5>
<p>Aptana Studio 2.0.5 <a href="http://aptana.com/">http://aptana.com</a><br />Name: Aptana<br />Location: <a href="http://download.aptana.com/tools/studio/plugin/install/studio">http://download.aptana.com/tools/studio/plugin/install/studio</a><br />После установки и перезагрузки Eclipse идём в Help - Install Aptana Features и устанавливаем дополнения к пакету.</p>
<h5><a id="Google Apps Engine plugin for Eclipse" name="Google Apps Engine plugin for Eclipse"></a>Google Apps Engine plugin for Eclipse</h5>
<p><a href="http://code.google.com/intl/ru/appengine/docs/java/tools/eclipse.html">http://code.google.com/intl/ru/appengine/docs/java/tools/eclipse.html</a><br />Name: Google Apps Engine<br />Location: <a href="http://dl.google.com/eclipse/plugin/3.6">http://dl.google.com/eclipse/plugin/3.6</a><br />Если при попытке добавления происходит ошибка<br />'org.eclipse.wst.xml.core 0.0.0' but it could not be found,<br />то надо добавить репозиторий Eclipse plugins - <a href="http://download.eclipse.org/releases/helios">http://download.eclipse.org/releases/helios</a>.</p>
<h5><a id="Пакет для программирования на C/C++" name="Пакет для программирования на C/C++"></a>Пакет для программирования на C/C++</h5>
<p>Name: CDT<br />Location: : <a href="http://download.eclipse.org/tools/cdt/releases/helios">http://download.eclipse.org/tools/cdt/releases/helios</a></p>
<h3><a id="Добавление репозитория ppa и установка Eclipse" name="Добавление репозитория ppa и установка Eclipse"></a>Добавление репозитория ppa и установка Eclipse</h3>
<p>Даю команды:<br />$ sudo add-apt-repository ppa:eclipse-team/debian-package<br />$ sudo apt-get update<br />Далее для обновления установленного eclipse’а:<br />$ sudo apt-get upgrade<br />Или для установки в систему без установленного eclipse’а:<br />$ sudo apt-get install eclipse</p>
<h3><a id="Результат" name="Результат"></a>Результат</h3>
<p>Из коробки не завёлся. Появляется окно с надписью продукта и исчезает.<br />В ~/.eclipse/ и далее в подпапках можно найти логи с ошибками.</p>
