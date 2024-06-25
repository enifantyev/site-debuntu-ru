---
title: "Использование Composer для установки и управления зависимостями Drupal 8"
date: 2019-03-25
weight: 10
description: >
  Рекомендуемый разработчиками правильный способ разворачивания Drupal 8. Частичный перевод указанных в статье источников и личный опыт.
tags:
  - CMS
  - Drupal
  - Composer
---

2019-03-25
Использованные источники:

1. [Using Composer to Install Drupal and Manage Dependencies](https://www.drupal.org/docs/develop/using-composer/using-composer-to-install-drupal-and-manage-dependencies)
2. [Introduction - Composer](https://getcomposer.org/doc/00-intro.md)

Composer может быть использован для загрузки Drupal, его модулей и тем оформления, а также всех появившихся зависимостей. Эти инструкции могут быть гибко подстроены под ваши методы управления экземпляром Drupal.

## Установка Composer
Вам нужно установить Composer на вашу локальную машину перед выполнением любых composer-команд. Пожалуйста, ознакомтесь с [Getting Started on getcomposer.org](https://getcomposer.org/doc/00-intro.md) для установки Composer. Если вам разрешено устанавливать Composer глобально, так и сделайте, так как нет никакой выгоды от множества установленных на компьютере копий Composer для каждого пользователя или проекта.

Скачиваем последнюю версию Composer, проверяем хэш и устанавливаем глобально:
```
$ php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
$ php -r "if (hash_file('sha384', 'composer-setup.php') === '48e3236262b34d30969dca3c37281b3b4bbe3221bda826ac6a9a62d6444cdb0dcd0615698a5cbe587c3f0fe57a54d8f5') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
Installer verified
$ sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer
```

>*Замечу, что для `composer create-project drupal...` потребовалось 1.5-2.0GB. На маленьком vps с 512MB RAM и 1024MB swapfile, я не смог использовать `composer` из-за появляющихся ошибок про невозможность выделения памяти. Для решения проблемы я создал новый swapfile на 2048MB:*
```
swapoff -a
rm /swapfile
dd if=/dev/zero of=/swapfile bs=1M count=2048 status=progress
chmod 0600 /swapfile
mkswap /swapfile
swapon -a
```
*И ещё заметка. Арендуя vps на OpenVZ с 1GB RAM, у меня не получилось развернуть Drupal через Composer. Всё затыкалось на это моменте:*
```
Reading /root/.composer/cache/repo/https---repo.packagist.org/provider-guzzle$stream.json from cache
Reading /root/.composer/cache/repo/https---repo.packagist.org/provider-guzzlehttp$streams.json from cache
Reading /root/.composer/cache/repo/https---repo.packagist.org/provider-justinrainbow$json-schema.json from cache
Killed
```
*При этом было видно через `top`, что размер свободной памяти приближался к нулю. Создание swap-файла запрещено на OpenVZ, поэтому разворачивал проект на домашней машине и делал `rsync` директорию проекта на этот vps.*

## Загрузка Drupal через Composer
Есть несколько путей установки Drupal через Composer, однако рекомендованным методом (особенно для начинающих пользователей Composer) является использование шаблона из [drupal-composer/drupal-project](https://github.com/drupal-composer/drupal-project).
Это open-source проект, предлагающий пустой стартовый Drupal-сайт, уже обеспеченный composer-конфигурацией по умолчанию, которую иначе пришлось бы устанавливать вручную.

Вот некоторые из вещей, которые этот проект занимается:

- Установка Drupal-сайта в поддиректорию **web/** папки проекта.
- Настройка репозитория Drupal composer.
- Настройка установочных путей, по которым composer будет размещать модули, темы, библиотеки и прочее.
- Установка [Drush](https://www.drush.org/).
- Установка [Drupal Console](https://drupalconsole.com/).

Поддержка и разработка этого composer-проекта размещается по адресу [drupal-composer/drupal-project GitHub
](https://github.com/drupal-composer/drupal-project/issues)

### Использование drupal-composer/drupal-project
### Установка по умолчанию
Запускаем `composer create-project drupal-composer/drupal-project:8.x-dev my_site_name_dir --no-interaction`

Это загрузит 'drupal-composer/drupal-project' в папку с именем 'my_site_name_dir' и автоматически выполнит команду 'composer install', которая загрузит Drupal 8 и все его зависимости.
```
/srv/www/my_site_name_dir$ ls -l
total 352
drwxrwxr-x 1 user      user         318 Mar 25 20:33 ./
drwxrwxr-x 1 www-data  www-data     168 Mar 25 14:05 ../
-rw-rw-r-- 1 user      user         357 Mar 25 20:33 .editorconfig
-rw-rw-r-- 1 user      user         746 Mar 25 14:39 .env.example
-rw-rw-r-- 1 user      user        3858 Mar 25 20:33 .gitattributes
-rw-rw-r-- 1 user      user         466 Mar 25 14:39 .gitignore
-rw-rw-r-- 1 user      user        1875 Mar 25 14:39 .travis.yml
-rw-rw-r-- 1 user      user       18046 Mar 25 14:39 LICENSE
-rw-rw-r-- 1 user      user        6495 Mar 25 14:39 README.md
-rw-rw-r-- 1 user      user        2474 Mar 25 14:39 composer.json
-rw-rw-r-- 1 user      user      297472 Mar 25 20:33 composer.lock
drwxrwxr-x 1 user      user          62 Mar 25 14:39 drush/
-rw-rw-r-- 1 user      user         414 Mar 25 14:39 load.environment.php
-rw-rw-r-- 1 user      user         481 Mar 25 14:39 phpunit.xml.dist
drwxrwxr-x 1 user      user          16 Mar 25 14:39 scripts/
drwxrwxr-x 1 user      user         836 Mar 25 20:33 vendor/
drwxrwxr-x 1 user      user         336 Mar 25 20:33 web/
```
Обратите внимание на директорию **`web/`**. Именно в ней находится корень нового сайта, который мы указываем в настройках http-сервера.

### Установка не по умолчанию
Если вы хотите изменить некоторые настройки в загружаемом файле composer.json перед выполнением 'composer install', используйте флаг `--no-install` в команде 'composer create-project'. Для примера, вероятно вы захотите переименовать поддиректорию **`web/`** как-либо иначе.
Для этого:

1. Запускаем 'composer create-project drupal-composer/drupal-project:8.x-dev my_site_name_dir --stability dev --no-interaction --no-install'
2. Переходим в папку my_site_name_dir и меняем необходимые настройки в файле composer.json
3. Запускаем 'composer install' для загрузки Drupal 8 и всех зависимостей.

Ознакомтесь с README в [drupal-composer/drupal-project](https://github.com/drupal-composer/drupal-project) для более глубокого изучения особенностей проекта и его использования. Если вы используете строительство Drupal-сайта посредством Drush, то обратитесь к [FAQs in Drupal's Composer template documentation](https://www.drupal.org/docs/develop/using-composer/using-composer-template-for-drupal-projects#faq) для изучения различий между composer-вариантом и drush make.

## Загрузка дополнительных модулей, тем оформления и других зависимостей, используя Composer
Распространённое явление, когда дополнительные Drupal-модули имеют зависимости от сторонних библиотек. Некоторые из этих модулей могут быть установлены только с помощью Composer. Если вы загрузили Drupal 8 с помощью Composer, то вы возможно захотите использовать Composer для загрузки всех модулей и тем.

Этот раздел делится на две части:

- Загрузка дополнительных модулей и тем с помощью Composer
- Указание директорий для размещения Drupal-проектов, если такое необходимо

### Загрузка дополнительных модулей и тем с помощью Composer
Для загрузки дополнительных модулей и тем с помощью Composer:

- Выполняем `composer require drupal/<modulename>`
- Для примера: `composer require drupal/token`
- Это необходимо выполнять в папке, являющейся корнем вашего Drupal-проекта (где размещён файл composer.json), a не на том уровне, где находится Drupal-сайт (по умолчанию директория **web/** в папке проекта).

Composer автоматически добавит строчку запроса модуля "drupal/token" в список требуемых модулей в файле composer.json, наподобие следующего:
```
{
    "require": {
        "drupal/core": "^8.6.0",
...
        "drupal/token": "^1.5"
    }
}
```
Composer загрузит модуль и все необходимые для его работы зависимости, но не включит его.

Вы можете включить Drupal-модуль двумя способами:

- Используя стандратный Drupal web-интерфейс.
- Используя командную строку через **Drush** или **Drupal Console** - смотри [Installing Modules from the Command Line](https://www.drupal.org/docs/8/extending-drupal-8/installing-modules-from-the-command-line).

Для запроса модулей, вы можете использовать либо название проекта, либо конкретное название модуля:

- В первом случае Composer загрузит весь проект, содержащий данный модуль.
- Во втором случае, для примера, если вам нужен модуль "fe_block" из проекта "[features_extra](https://drupal.org/project/features_extra)" вы можете загрузить его любым из следующих способов:
  - `composer require drupal/features_extra`
  - `composer require drupal/fe_block`

### Указание версии модуля
Вы можете указать определённую версию скачиваемого модуля или темы:
`composer require 'drupal/<modulename>:<version>'`

Для примера:
```
composer require 'drupal/token:^1.5'
composer require 'drupal/simple_fb_connect:~3.0'
composer require 'drupal/ctools:3.0.0-alpha26'
composer require 'drupal/token:1.x-dev'
```

## Дальнейшая установка через web-интерфейс
Во время первичной инициализации Drupal-сайта через web-интерфейс может произойти ошибка:
```
Обнаружены ошибки
КОНФИГУРАЦИОННЫЙ КАТАЛОГ: SYNC
An automated attempt to create the directory ../config/sync failed, possibly due to a permissions problem.
```

Для её исправления, в папке проекта (где находится файл composer.json) необходимо создать подпапку с вложенной папкой: `mkdir -p config/sync`

## Drush
Drush запускаем из корневой папки проекта (где находится файл composer.json).
```
$ drush sset system.maintenance_mode 1
$ drush drupal:directory                # или "drush dd"
/srv/www/my_site_name_dir/web
```
Вообще рекомендовано, с помощью `composer` добавлять/убирать модули, а с помощью `drush` (или через администраторский web-интерфейс) включать/выключать модули.

## Backup и обновление Drupal-проекта
Из девятой версии drush выбросили возможность бэкапа базы sql и директории сайта с помощью одной команды, поэтому процессы разделяем:
```
$ drush sset system.maintenance_mode 1
$ drush cache:rebuild                   # или "drush cr"
  [success] Cache rebuild complete.
$ drush sql-dump |bzip2  > ../backup/my_site_name_dir_sql_$(date "+%Y%m%d-%H%M%S").bz2
$ tar -cvjf ../backup/my_site_name_dir_files_$(date "+%Y%m%d-%H%M%S").tar.bz2 ../my_site_name_dir
#
```
В вышеуказанных строках произойдёт:

1. Перевод сайта в режим обслуживания.
2. Сброс кэша.
3. Дамп базы данных с добавлением текущих даты/времени к имени файла.
4. Архивирование всей директории проекта с добавлением текущих даты/времени к имени файла.
