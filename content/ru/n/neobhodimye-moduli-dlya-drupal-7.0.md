---
title: "Необходимые модули для Drupal 7.0"
date: 2011-03-30
weight: 10
description: >
  Необходимые модули для Drupal 7.0
tags:
  - Drupal
  - CMS
---

2011-03-30
[CKEditor](http://drupal.org/project/ckeditor) — wysiwyg редактор для редактирования текста.

[IMCE](http://drupal.org/project/imce) — управление загрузкой, удалением изображения в wysiwyg редакторе. Надо чуток донастроить.

[Pathauto](http://drupal.org/project/pathauto) (для работы нужен [Token](http://drupal.org/project/token)) + [Transliteration](http://drupal.org/project/transliteration)  — Эта связка позволит автоматически назначать статье URL с удобочитаемым видом в англоязычной кодировке. То есть вместо `https://www.example.com/node/45`будет `https://www.example.com/eta-statia-pro-elochku`, что приятно поисковым движкам и людям. Для настройки надо зайти в **Настройки** - **Адреса**. Тут выбрать вкладку **Настройки** и включить:
- "Transliterate prior to creating alias" для автоматического преобразования национальных символов в латинские;
- "Reduce strings to letters and numbers" - для приведения строк к виду, состоящему только из латинских букв и арабских цифр;
- "Create a new alias. Leave the existing alias functioning." - для отключения потери старого URL при смене заголовка статьи. Тут надо подумать и почитать про Global Redirect. Не будет ли дубликатов страниц.

[Global Redirect](http://drupal.org/project/globalredirect) — Эта штука склеивает различные URL одного материала. Например, обратившись к странице этого сайта по drupal'овскому URL'у http://debuntu.ru/node/5  — нас перекинет на хуман-читабельное URL http://debuntu.ru/content/neobhodimye-moduli-dlya-drupal-70
Для дополнительной настройки надо зайти в Настройки - Global Redirect и включить:
"Add Canonical Link" -

[Redirect](http://drupal.org/project/redirect) (бывший Path Redirect) — автоматическое, а при необходимости и ручное, добавление к старым URL  "Redirect 301"  на новые страницы. Я расчитывал, что при настройке Pathauto в режим "Create a new alias. Leave the existing alias functioning" модуль Redirect будет автоматически добавлять к старым URL "Redirect 301" на новые созданные URL. Но, к сожалению, это работает лишь при ручном изменении URL! Надо выяснить - не помогает ли в этом неприятном случае модуль Global Redirect.

[Page Title](http://drupal.org/project/page_title) — Управление заголовками по шаблону и вручную. Поставил dev версию и пропатчил.

[SpamSpan](http://drupal.org/project/spamspan) — для маскировки почтовых адресов в ноде. Необходимо включать в Настройки - Форматы ввода. В каждом шаблоне включить "Фильтр SpamSpan для шифрования электронных адресов". И там же, ниже, на горизонтальной закладке "Фильтр SpamSpan для шифрования электронных адресов" включить <Use a graphical replacement for "@"">.

[XML Sitemap](http://drupal.org/project/xmlsitemap) — ну это понятно.

[Insert](http://drupal.org/project/insert) — добавление к ноде дополнительного поля с типом "Изображение" для вывода картинки к тексту. При удалении ноды, привязанные к ноде изображения, автоматически удаляются.

[Colorbox](http://drupal.org/project/colorbox) — умеет создавать галереи и показывает увеличенные версии изображений в наложенном на страницу полупрозрачном слое.

[Google Analytics](http://drupal.org/project/google_analytics) — ну это понятно.

[Views](http://drupal.org/project/views) (для работы необходим [Chaos tools](http://drupal.org/project/ctools)) — архиважный элемент для построения различных списков с выводом на отдельные страницы или блоки.

[SMTP Authentication Support](http://drupal.org/project/smtp) — для отправки писем с сайта.

[CAPTCHA](http://drupal.org/project/captcha) — модуль защиты от спамботов для форм отправки почты и т.п.
