---
title: "Установка и использование в FreeBSD 9.1 нового пакетного менеджера pkgng"
date: 2013-04-24
weight: 10
description: >
  Установка и использование в FreeBSD 9.1 нового пакетного менеджера pkgng
tags:
  - FreeBSD
  - pkgng
---

<div> </div>
<div>ОС: FreeBSD 9.1</div>
<div>Последнее существенное изменение: 2013-04-24</div>
<ul>
<li><a href="#Введение в pkgng">Введение</a></li>
<li><a href="#Установка pkgng">Установка</a></li>
<li><a href="#Настройка pkgng">Настройка</a></li>
<li><a href="#Использование pkgng">Использование</a></li>
<li><a href="#Ссылки">Ссылки</a></li>
</ul>
<h3><a id="Введение в pkgng" name="Введение в pkgng"></a>Введение</h3>
<p>Мой опыт использования FreeBSD не превышает нескольких дней, поэтому подразумевается, что в тексте обильно рассыпаны фразеологизмы "насколько я понял", "если я правильно понял" и т.п. Если их не видно, это не значит, что их там нет.</p>
<p>Пакетный менеджер pkgng пришёл на смену pkg_install и более старым способам сборки портов прямо на компьютере. Изначально процесс установки пакета состоял из скачивания исходников пакета на компьютер и сборку пакета через make и т.д. Потом автоматизировали процесс, появился пакетный менеджер pkg_install, который позволил скачивать готовые собранные пакеты и устанавливать их через команду pkg_add, удалять через pkg_delete и т.д. Но и этот менеджер пакетов не был пределом мечтаний и в году 2010 начали создавать новый пакетный менеджер pkgng, который мне напоминает apt в debian.</p>
<h3><a id="Установка pkgng" name="Установка pkgng"></a>Установка</h3>
<p>На чистой FreeBSD предусмотрительно надо установить несколько пакетов, которые облегчат в дальнейшем переход на pkgng:</p>
<pre># pkg_add -r wget screen mc</pre>
<p>Установку нового pkgng я произвёл посредством старого pkg_install:</p>
<p># pkg_add -r pkg</p>
<p>Потом, по инструкции, конвертировал базу установленных пакетов в новый формат:</p>
<p># pkg2ng</p>
<p>В файл /etc/make.conf добавил строку:</p>
<pre>WITH_PKGNG=yes</pre>
<p>Формат баз установленных портов для pkgng и pkg_install разные, поэтому после установки pkgng и конвертации базы в новый формат нельзя использовать старый pkg_install(?). Я пока не знаю как полностью избавиться от pkg_install, да и подозреваю, что на данный момент этого делать не стоит.</p>
<h3><a id="Настройка pkgng" name="Настройка pkgng"></a>Настройка</h3>
<p>Файл настроек находится в /usr/local/etc/pkg.conf, который я сделал из тут же лежащего pkg.conf.sample. Вначале я ничего не менял в этом файле и помучился с сообщениями об отсутствии в репозитории файла repo.txz, когда попытался установить mc:</p>
<pre># pkg install mc</pre>
<p>Потом прочитал, что <em>открылся специальный репозиторий для pkgng<a href="#link3"><sup>3</sup></a></em> и внёс изменения в файл pkg.conf, в соответствии с мануалом<a href="#link4"><sup>4</sup></a>. Вот что там у меня сейчас:</p>
<pre><strong># cat pkg.conf</strong>
# System-wide configuration file for pkg(1)
# For more information on the file format and
# options please refer to the pkg.conf(5) man page</pre>
<pre># Configuration options
PACKAGESITE         : http://ftp.pcbsd.org/pub/mirror/packages/9.1-RELEASE/amd64/
#PACKAGESITE         : http://mirror.corbina.net/pub/pcbsd/packages/9.1-RELEASE/amd64/
#PKG_DBDIR          : /var/db/pkg
PKG_CACHEDIR        : /usr/local/tmp
#PORTSDIR           : /usr/ports
PUBKEY              : /usr/local/etc/pkg-pubkey.cert
#HANDLE_RC_SCRIPTS   : NO
#PKG_MULTIREPOS     : NO
#ASSUME_ALWAYS_YES   : NO
#SYSLOG             : YES
#SHLIBS             : NO
#AUTODEPS           : NO
#PORTAUDIT_SITE     : http://portaudit.FreeBSD.org/auditfile.tbz

# Repository definitions
#repos:
#  default : http://example.org/pkgng/
#  repo1 : http://somewhere.org/pkgng/repo1/
#  repo2 : http://somewhere.org/pkgng/repo2/</pre>
<div>Может такое статься, что этот репозиторий закрыт по географическому признаку, поэтому я подготовил заремаренную ссылку на корбиновский репозиторий. Прочие российские зеркала pkgng репозитория можно найти здесь: <a href="http://www.pcbsd.org/getmirrors.php?url=packages">http://www.pcbsd.org/getmirrors.php?url=packages</a>.</div>
<p>Потом, следуя инструкции<a href="#link4"><sup>4</sup></a>, я скачал открытый ключ репозитория и поместил его здесь:</p>
<pre># cd /usr/local/etc
# wget http://trac.pcbsd.org/export/21629/pcbsd/current/src-sh/pc-extractoverlay/desktop-overlay/usr/local/etc/pkg-pubkey.cert</pre>
<p>И обновил установленные пакеты до версий из нового репозитория:</p>
<pre># pkg upgrade -fy</pre>
<h3><a id="Использование pkgng" name="Использование pkgng"></a>Использование</h3>
<p>Справки по командам можно получить, например:</p>
<pre># man pkg-upgrade</pre>
<p>или</p>
<pre># pkg help upgrade</pre>
<p>Обновление пакетов:</p>
<pre># pkg upgrade</pre>
<p>Первый апгрейд у меня не получился из-за нарушения зависимостей. Мешал пакет pkg-config. Я решил его удалить, но он был нужен для работы tcpdump, wget и ещё пары пакетов. Я удалил все четыре мешающих пакетов:</p>
<pre># pkg delete libidn libsmi tcpdump wget</pre>
<p>после чего произвёл удачный апгрейд. Далее удалил pkg-config. И, на всякий случай, восстановил все четыре ранее удалённых пакетов, включая tcpdump. Не спрашивайте почему я сразу не удалил pkg-config вместе с четырьмя прочими пакетами. В тот момент мне казалось это неплохой идеей.</p>
<p>Установка пакета:</p>
<pre># pkg install <em>package</em></pre>
<p>Информация по пакету:</p>
<pre># pkg info <em>package</em></pre>
<p>Удалить пакет:</p>
<pre># pkg delete <em>package</em></pre>
<p>Также в pkgng есть своя проверка пакетов на уязвимости, которая заменяет portaudit. Проверить пакеты на уязвимости:</p>
<pre># pkg audit -F</pre>
<p>И ещё много полезного...</p>
<h3><a id="Ссылки" name="Ссылки"></a>Ссылки</h3>
<ol>
<li><a href="http://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/pkgng-intro.html">Using pkgng for Binary Package Management</a></li>
<li><a href="http://www.opennet.ru/opennews/art.shtml?num=34739">04.09.2012 14:04 Вышел pkgng 1.0, новый пакетный менеджер для FreeBSD</a></li>
<li><a id="link3" href="http://www.opennet.ru/opennews/art.shtml?num=36730" name="link3">19.04.2013 09:56  Введён в строй постоянно обновляемый pkgng-репозиторий для PC-BSD и FreeBSD 9.1</a></li>
<li><a id="link4" href="http://wiki.pcbsd.org/index.php/Convert_a_FreeBSD_System_to_PC-BSD%C2%AE#Switching_to_the_PC-BSD.C2.AE_pkgng_Repository" name="link4">Switching to the PC-BSD® pkgng Repository</a></li>
</ol>
