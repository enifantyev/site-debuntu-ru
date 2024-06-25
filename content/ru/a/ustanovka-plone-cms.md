---
title: "Установка Plone CMS"
date: 2019-10-21
weight: 10
description: >
  Описание установки Plone CMS от скачивания инсталлятора до настройки nginx.
tags:
  - Plone
  - CMS
---

2019-10-21

Пытаясь найти замену Drupal для своих сайтов я набрёл на [Plone](https://plone.org/). После прочтения материалов о данной CMS с воодушевлением принялся за работу.

Для работы Drupal необходимо устанавливать много дополнительного ПО, вследствии чего вся друпаловская программная обвязка поглощает много оперативной памяти. При попытке перейти с седьмого на восьмой Drupal я столкнулся с нехваткой памяти для работы Composer. Доплатил за увеличение RAM до 1GB и вновь неудача. На vps на базе kvm можно было бы создать swap-файл, но на текущем openvz это делать запрещено. Тогда я развернул Drupal с помощью Composer на домашней машине и скопировал эту готовую инсталляцию на vps. Начал добавлять модули и вновь проблемы с зависимостями модуля Markdown... И так как я уже и раньше задумывался о расставании с Drupal, то эти проблемы переполнили мою чашу терпения.

В отличии от Drupal, для работы Plone нужен лишь Python и Nginx, что не требует много оперативной памяти. Устанавливать Plone буду на vps openvz Ubuntu Server с 1GB RAM без swap. Забегая вперёд скажу, что Plone по сравнению с Drupal мне кажется просто ракетой. Markdown здесь присутствует прямо из коробки. Для бэкапа сайта достаточно заархивировать директорию, куда был развёрнут Plone. Язык программирования Python, на котором сделан Plone, мне очень интересен. А так как знакомство моё с Python поверхностное и есть желание знакомство это углубить, то в этом CMS я вижу для себя много приятного и полезного.

Требования для установки Plone описаны здесь:
[Plone Installation Requirements](https://docs.plone.org/manage/installing/requirements.html)
Процесс установки описан здесь:
[Installation](https://docs.plone.org/manage/installing/installation.html)

### Оглавление
- [Подготовка ОС](#PREPARING)
- [Установка Plone](#INSTALLATION)
- [Действия после установки Plone](#AFTER_INSTALLATION)
- [Установка инструментов разработчика, если необходимо](#INSTALLATION_P_D_TOOLS)
- [Запуск Plone](#START_PLONE)
- [Изменение конфигурации](#CHANGE_CONFIGURATION)
- [Модуль для запуска Plone через systemd](#UNIT_FOR_SYSTEMD)
- [Первый вход в web-management Plone](#FIRST_LOGON)
- [Добавление следующего сайта в Plone](#ADDING_NEXT_SITE)
- [Смена пароля администратора Plone](#CHANGE_ADMIN_PASSWORD)
- [Настройка Nginx](#NGINX_SERVER_BLOCK)

<a name="PREPARING"></a>

### Подготовка
Разработчики Plone рекомендуют использовать *унифицированный инсталлер*, который необходимо скачать, распаковать и запустить установку Plone:
```
# apt install python2.7 python2.7-dev python-setuptools python-dev build-essential libssl-dev libxml2-dev libxslt1-dev libbz2-dev libjpeg62-dev zlib1g-dev
# wget https://launchpad.net/plone/5.1/5.1.5/+download/Plone-5.1.5-UnifiedInstaller.tgz
# tar xvf Plone-5.1.5-UnifiedInstaller.tgz
# cd Plone-5.1.5-UnifiedInstaller
```
> *Выпуск обновлений для Python 2.7 прекратится в январе 2020 года, поэтому идёт работа по переводу Plone на третий Пайтон. Plone 5.2, который должен выйти 30 июня, будет поддерживать третий Python. Следующая версия Plone 6.0 пишется исключительно на третьей версии Пайтона и должна выйти через год, то есть весной 2020 года.*

<a name="INSTALLATION"></a>

### Установка
Установка проста до безобразия:

`# ./install.sh standalone --target="/srv/plone"`

<details><p>

```
Testing /usr/bin/python2.7 for Zope/Plone requirements....
/usr/bin/python2.7 looks OK. We will use it.

Root install method chosen. Will install for use by users:
  ZEO & Client Daemons:      plone_daemon
  Code Resources & buildout: plone_buildout

Detailed installation log being written to /srv/Plone-5.1.5-UnifiedInstaller/install.log
Installing Plone 5.1.5 at /srv/plone

Using useradd and groupadd to create users and groups.
Creating Python virtual environment.
New python executable in /srv/plone/zinstance/bin/python2.7
Also creating executable in /srv/plone/zinstance/bin/python
Installing setuptools, pip, wheel...
done.
Installing zc.buildout in virtual environment.
Unpacking buildout cache to /srv/plone/buildout-cache
Copying buildout skeleton
Building Zope/Plone; this takes a while...
Buildout completed

#####################################################################

######################  Installation Complete  ######################

Plone successfully installed at /srv/plone
See /srv/plone/zinstance/README.html
for startup instructions.

Use the account information below to log into the Zope Management Interface
The account has full 'Manager' privileges.

  Username: admin
  Password: HUMKuSRDS882

This account is created when the object database is initialized. If you change
the password later (which you should!), you'll need to use the new password.

Use this account only to create Plone sites and initial users. Do not use it
for routine login or maintenance.- If you need help, ask in IRC channel #plone
 on irc.freenode.net.
- The live support channel also exists at http://plone.org/chat
- You can also ask for help on https://community.plone.org
- Submit feedback and report errors at https://github.com/plone/Products.CMFPlone/issues
(For install problems, https://github.com/plone/Installers-UnifiedInstaller/issues)
```
</p></details>

<a name="AFTER_INSTALLATION"></a>

### После установки
Читаем `lynx /srv/plone/zinstance/README.html` и запоминаем:

- надо заглянуть в `/srv/plone/zinstance/buildout.cfg` и кое-что подправить. Я добавлю строку `ip-address = 127.0.0.1`, чтобы Plone слушал только по localhost-адресу `127.0.0.1:8080`, а nginx проксировал этот порт наружу. Для применения новой конфигурации останавливаем Plone, делаем бэкап папки `/srv/plone`, и из директории `/srv/plone/zinstance` запускаем `sudo -u plone_buildout bin/buildout`. После обновления конфигурации нам возможно понадобится "Check portal_migration in the ZMI after update to perform version migration if necessary. You may also need to visit the product installer to update product versions."
- также запоминаем, что вручную запускаем/останавливаем plone командами:
    - `sudo -u plone_daemon /srv/plone/zinstance/bin/plonectl start`
    - `sudo -u plone_daemon /srv/plone/zinstance/bin/plonectl stop`
- ещё помним о том, что необходимо поменять предоставленный admin пароль (можно найти в `/srv/plone/zinstance/adminPassword.txt`). Смену пароля можно произвести по адресу `http://localhost:8080/acl_users/users/manage_users`. После этого действия информация указанная в `adminPassword.txt` перестанет иметь значение.

Так как установку я производил из-под рута, то исправляю права:
```
# chown plone_buildout:plone_group /srv/plone -R
```

<a name="INSTALLATION_P_D_TOOLS"></a>

### Install Plone Developer Tools
Если вы будете использовать установленный Plone для разработки, тогда добавьте общий набор инструментов разработчика нижеследующей командой. Вам необходимо добавлять "-c develop.cfg" каждый раз при запуске `bin/buildout`, иначе вы лишитесь инструментов разработчика до следующего использования "-c develop.cfg":
```
# cd /srv/plone/zinstance
# sudo -u plone_buildout bin/buildout -c develop.cfg
```

<details><p>

```
...
Generated script '/srv/plone/zinstance/bin/fullrelease'.
Generated script '/srv/plone/zinstance/bin/postrelease'.
Generated script '/srv/plone/zinstance/bin/lasttagdiff'.
Generated script '/srv/plone/zinstance/bin/addchangelogentry'.
Generated script '/srv/plone/zinstance/bin/bumpversion'.
Generated script '/srv/plone/zinstance/bin/prerelease'.
Generated script '/srv/plone/zinstance/bin/release'.
Generated script '/srv/plone/zinstance/bin/longtest'.
Generated script '/srv/plone/zinstance/bin/lasttaglog'.
Generated script '/srv/plone/zinstance/bin/pocompile'.
Installing i18ndude.
Getting distribution for 'i18ndude==4.4.0'.
Got i18ndude 4.4.0.
Getting distribution for 'ordereddict==1.1'.
zip_safe flag not set; analyzing archive contents...
Got ordereddict 1.1.
Generated script '/srv/plone/zinstance/bin/i18ndude'.
Versions had to be automatically picked.
The following part definition lists the versions picked:
[versions]
MarkupSafe = 1.1.1
Products.DocFinderTab = 1.0.5
bobtemplates.plone = 4.0.2
chardet = 3.0.4
collective.checkdocs = 0.2
idna = 2.6
mr.bob = 0.1.2
pkginfo = 1.5.0.1
plone.recipe.command = 1.1
plone.recipe.precompiler = 0.6
regex = 2019.3.12
requests-toolbelt = 0.9.1
zest.pocompile = 1.4

# Required by:
# bobtemplates.plone==4.0.2
case-conversion = 2.1.0

# Required by:
# bobtemplates.plone==4.0.2
# zest.releaser==6.15.0
colorama = 0.4.1
```
</p></details>

<a name="START_PLONE"></a>
### Запуск Plone
Ранее мы уже выяснили из `/srv/plone/zinstance/README.html` способы запуска и остановки Plone. Вот ещё один способ запуска удобный при разработке:
```
# sudo -u plone_daemon /srv/plone/zinstance/bin/plonectl fg
```

<details><p>

```
instance: 2019-04-04 16:59:55 INFO ZServer HTTP server started at Thu Apr  4 16:59:55 2019
        Hostname: 0.0.0.0
        Port: 8080
2019-04-04 16:59:58 INFO DocFinderTab Applied patch version 1.0.5.
2019-04-04 16:59:59 INFO ZODB.blob (857) Blob directory `/srv/plone/zinstance/var/blobstorage` is unused and has no layout marker set. Selected `bushy` layout.
2019-04-04 16:59:59 INFO ZODB.blob (857) Blob temporary directory '/srv/plone/zinstance/var/blobstorage/tmp' does not exist. Created new directory.
/srv/plone/buildout-cache/eggs/plone.app.blob-1.7.4-py2.7.egg/plone/app/blob/content.py:23: DeprecationWarning: MimeTypeException is deprecated. Import from Products.MimetypesRegistry.interfaces instead
  from Products.MimetypesRegistry.common import MimeTypeException
/srv/plone/buildout-cache/eggs/plone.portlet.collection-3.3.1-py2.7.egg/plone/portlet/collection/collection.py:2: DeprecationWarning: isDefaultPage is deprecated. Import from Products.CMFPlone instead
  from plone.app.layout.navigation.defaultpage import isDefaultPage
/srv/plone/buildout-cache/eggs/Products.CMFPlone-5.1.5-py2.7.egg/Products/CMFPlone/browser/syndication/views.py:17: DeprecationWarning: wrap_form is deprecated. Import from plone.z3cform.layout instead.
  from plone.app.z3cform.layout import wrap_form
/srv/plone/buildout-cache/eggs/Zope2-2.13.27-py2.7.egg/OFS/Application.py:102: DeprecationWarning: Expected text
  transaction.get().note("Created Zope Application")
/srv/plone/buildout-cache/eggs/Zope2-2.13.27-py2.7.egg/OFS/Application.py:267: DeprecationWarning: Expected text
  transaction.get().note(note)
/srv/plone/buildout-cache/eggs/Zope2-2.13.27-py2.7.egg/OFS/Application.py:523: DeprecationWarning: Expected text
  transaction.get().note('Prior to product installs')
/srv/plone/buildout-cache/eggs/Zope2-2.13.27-py2.7.egg/OFS/Application.py:777: DeprecationWarning: Expected text
  transaction.get().note('Installed standard objects')
2019-04-04 17:00:10 INFO Zope Ready to handle requests
```
</p></details>
Для остановки нажимаем `Ctrl+c`.

<a name="CHANGE_CONFIGURATION"></a>

### Изменение конфигурации Plone
В файле `buildout.cfg` добавляем строку `ip-address = 127.0.0.1`:
```
[instance]
<= instance_base
recipe = plone.recipe.zope2instance
ip-address = 127.0.0.1
http-address = 8080
```
>  В plone 5.2 я использую одну строку: `http-address = 127.0.0.1:8080`

Запускаем скрипт для обновления конфигурации и перезапускаем Plone. Не забываем добавить `-c develop.cfg`, если в конфигурацию необходимо добавить инструменты разработчика. В данный момент они не нужны, поэтому делаем:
```
# sudo -u plone_buildout bin/buildout
# sudo -u plone_daemon /srv/plone/zinstance/bin/plonectl restart
```

<details><p>

```
Updating instance.
Updating repozo.
Updating backup.
Updating zopepy.
Updating unifiedinstaller.
Updating precompiler.
Compiling Python files.
  File "/srv/plone/buildout-cache/eggs/zodbpickle-0.7.0-py2.7-linux-x86_64.egg/zodbpickle/pickle_3.py", line 178
    def __init__(self, file, protocol=None, *, fix_imports=True):
                                             ^
SyntaxError: invalid syntax

  File "/srv/plone/buildout-cache/eggs/zodbpickle-0.7.0-py2.7-linux-x86_64.egg/zodbpickle/pickletools_3.py", line 2049
    print("%5d:" % pos, end=' ', file=out)
                           ^
SyntaxError: invalid syntax

  File "/srv/plone/buildout-cache/eggs/zodbpickle-0.7.0-py2.7-linux-x86_64.egg/zodbpickle/tests/pickletester_3.py", line 145
    class use_metaclass(object, metaclass=metaclass):
                                         ^
SyntaxError: invalid syntax

Compiling locale files.
Error while compiling /srv/plone/buildout-cache/eggs/python_gettext-3.0-py2.7.egg/pythongettext/tests/test5.po
Error while compiling /srv/plone/buildout-cache/eggs/python_gettext-3.0-py2.7.egg/pythongettext/tests/test_escape.po
Updating setpermissions.
setpermissions: Running # Dummy references to force this to execute after referenced parts
echo /srv/plone/zinstance/var/backups yes > /dev/null
chmod 600 .installed.cfg
# Make sure anything we've created in var is r/w by our group
find /srv/plone/zinstance/var -type d -exec chmod 770 {} \; 2> /dev/null
find /srv/plone/zinstance/var -type f -exec chmod 660 {} \; 2> /dev/null
find /srv/plone/zinstance/var -type d -exec chmod 770 {} \; 2> /dev/null
find /srv/plone/zinstance/var -type f -exec chmod 660 {} \; 2> /dev/null
chmod 754 /srv/plone/zinstance/bin/*
Versions had to be automatically picked.
The following part definition lists the versions picked:
[versions]
plone.recipe.command = 1.1
plone.recipe.precompiler = 0.6
```
</p></details>

Видно, что на содержимое директории `/srv/plone/zinstance/var` применяются особые права без доступа *Другим*. Дело в том, что именно в этой папке будут содержаться загруженные материалы, база данных и логи сайта. Для бэкапа всего проекта достаточно остановить Plone и скопировать директорию `/srv/plone` в другое место.

<a name="UNIT_FOR_SYSTEMD"></a>

### Модуль для запуска Plone через systemd
[Automatic Plone (re)starts](https://docs.plone.org/manage/deploying/production/restarts.html)

Напомню, что ручного запуска Plone используем `sudo -u plone_daemon bin/plonectl start`. (Поправка: при ручном запуске `bin/plonectl start` запускается первый экземпляр `python` из-под рута, который запускает второй экземпляр `python` из-под plone_daemon.) Для автоматического запуска я создам unit для systemd.

`# cat > /etc/systemd/system/plone.service`
```
[Unit]
Description=Plone content management system
After=network.target

[Service]
User=plone_daemon
Type=forking
ExecStart=/srv/plone/zinstance/bin/plonectl start
ExecStop=/srv/plone/zinstance/bin/plonectl stop
ExecReload=/srv/plone/zinstance/bin/plonectl restart

[Install]
WantedBy=multi-user.target
```
`Ctrl+d` для посыла `End Of File`.

Далее стандартные команды:
```
# systemctl daemon-reload
# systemctl enable plone
# systemctl start plone
```
И в результате мы видим:
```
# ps aux|grep plone

root      8604  0.0  1.3 129780 14260 ?        Ssl  13:40   0:00 /srv/plone/zinstance/bin/python2.7 /srv/plone/zinstance/parts/instance/bin/interpreter /srv/plone/buildout-cache/eggs/zdaemon-4.2.0-py2.7.egg/zdaemon/zdrun.py -S /srv/plone/buildout-cache/eggs/Zope2-2.13.27-py2.7.egg/Zope2/Startup/zopeschema.xml -b 10 -d -s /srv/plone/zinstance/var/instance/zopectlsock -x 0,2 -z /srv/plone/zinstance/parts/instance /srv/plone/zinstance/bin/python2.7 /srv/plone/zinstance/parts/instance/bin/interpreter /srv/plone/buildout-cache/eggs/Zope2-2.13.27-py2.7.egg/Zope2/Startup/run.py -C /srv/plone/zinstance/parts/instance/etc/zope.conf

plone_d+  8606  5.5 19.3 548316 202864 ?       Sl   13:40   0:09 /srv/plone/zinstance/bin/python2.7 /srv/plone/zinstance/parts/instance/bin/interpreter /srv/plone/buildout-cache/eggs/Zope2-2.13.27-py2.7.egg/Zope2/Startup/run.py -C /srv/plone/zinstance/parts/instance/etc/zope.conf

# netstat -natup|grep 8080
tcp        0      0 127.0.0.1:8080          0.0.0.0:*               LISTEN      8606/python2.7

# free -h
              total        used        free      shared  buff/cache   available
Mem:           1.0G        248M        167M        7.6M        607M        531M
```

<a name="FIRST_LOGON"></a>

### Первый вход в web-management Plone
Пока у нас нет настроенного Apache или Nginx, то прокидываем удалённый `127.0.0.1:8080` порт на локальную машину:

`$ ssh -L8080:127.0.0.1:8080 root@plone.example.com`

Заходим через любой веб-браузер на адрес `http://127.0.0.1:8080` и видим приглашение к созданию нового Plone-сайта:

![Create new Plone site](/img/ustanovka-plone-cms/plone_install_window_create_new_plone_site_1.png)

Страница после нажатия единственной кнопки и ввода логина/пароля:

![Create first Plone site](/img/ustanovka-plone-cms/plone_install_window_create_new_plone_site_2.png)

В поле "Path identifier", вместо "Plone", я введу придуманный короткий ID моего первого сайта на Plone. Для второго сайта, использую другой ID. Если же новый сайт будет единственным в этом plone-сервере, то можно оставить ID по умолчанию. Этот ID будет светиться в URL сайта, типа `http://127.0.0.1:8080/id1` или `http://127.0.0.1:8080/plone`. В дальнейшем при настройке nginx для проксирования новых сайтов, мы уберём этот ID из URL и обращение к сайтам будет производиться через нормальные URL, типа `https://www.example.com`, `https://dbg.example.com` и т.п.

После заполнения прочих полей и применения мы увидим:

![Create first Plone site](/img/ustanovka-plone-cms/plone_install_window_create_new_plone_site_3.png)

Собственно это уже полноценный сайт. Необходимо настроить, например, nginx и заниматься наполнением сайта.

<a name="ADDING_NEXT_SITE"></a>

### Добавление следующего сайта в Plone
Добавление следующего сайта в Plone производится вновь через `http://127.0.0.1:8080`. Новый сайт будет иметь свой набор материалов, пользователей, своё оформление и будет независим от первого сайта.

<a name="CHANGE_ADMIN_PASSWORD"></a>

### Необходимо вспомнить о смене пароля для admin
Надо поменять не только пароль, но и логин. Это центральная учётная запись к серверу приложений, а значит она имеет доступ ко всем сайтам хостящимся на нём. Смену логина/пароля делаем через этот прямой URL: `http://localhost:8080/acl_users/users/manage_users`. После смены логина и пароля может появиться желание выйти из этого сеанса работы под администраторским аккаунтом. Для этого переходим по адресу `http://localhost:8080/manage` и здесь будет доступна возможность выхода из сеанса.

<a name="NGINX_SERVER_BLOCK"></a>

### Настройка Nginx
[Minimal Nginx front end configuration for Plone on Ubuntu/Debian Linux](https://docs.plone.org/manage/deploying/front-end/nginx.html#minimal-nginx-front-end-configuration-for-plone-on-ubuntu-debian-linux)

Создаём файл `/etc/nginx/sites-available/example.com` со следующим содержимым, указывающим на наш первый сайт `http://127.0.0.1:8080/id1`:
```
# This adds security headers
add_header X-Frame-Options "SAMEORIGIN";
add_header Strict-Transport-Security "max-age=15768000; includeSubDomains";
add_header X-XSS-Protection "1; mode=block";
add_header X-Content-Type-Options "nosniff";
#add_header Content-Security-Policy "default-src 'self'; img-src *; style-src 'self' 'unsafe-inline'; script-src 'self' 'unsafe-inline' 'unsafe-eval'";
add_header Content-Security-Policy-Report-Only "default-src 'self'; img-src *; style-src 'self' 'unsafe-inline'; script-src 'self' 'unsafe-inline' 'unsafe-eval'";

# This specifies which IP and port Plone is running on.
# The default is 127.0.0.1:8080
upstream plone {
   server 127.0.0.1:8080;
}

# Redirect all www-less traffic to the www.site.com domain
# (you could also do the opposite www -> non-www domain)
server {
   listen 80;
   server_name example.com;
   return 301 http://www.example.com$request_uri;
}

server {
   listen 80;
   server_name www.example.com;
   access_log /var/log/nginx/example.com.access.log;
   error_log /var/log/nginx/example.com.error.log;

   # Note that domain name spelling in VirtualHostBase URL matters
   # -> this is what Plone sees as the "real" HTTP request URL.
   # "Plone" in the URL is your site id (case sensitive)
   location / {
         proxy_pass http://plone/VirtualHostBase/http/example.com:80/id1/VirtualHostRoot/;
   }
}
```
