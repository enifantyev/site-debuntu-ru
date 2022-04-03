---
title: "Добавление reCAPTCHA в Plone"
date: 2019-05-09
weight: 10
description: >
  Добавление reCAPTCHA в Plone
tags:
  - Plone
  - CMS
  - translation
---

2019-05-09

Частичный перевод и компиляция информации из статей:

- [Adding CAPTCHA Support](https://docs.plone.org/working-with-content/managing-content/ploneformgen/captcha.html)
- [PloneFormGen](https://docs.plone.org/develop/plone/forms/ploneformgen.html)
- [Archetypes](https://docs.plone.org/develop/plone/content/archetypes/index.html)
- [Dexterity](https://docs.plone.org/develop/plone/content/dexterity.html)

> PloneFormGen имеет встроенную поддержку Re-Captcha. Здесь рассказано как включить её.

### PloneFormGen и поля Captcha
>- **PloneFormGen** — аддон, добавляющий в Plone генератор форм с полями, виджетами и *валидаторами* из Archetypes. Используется при создании простых однородных вебформ для сохранения вводимых данных или форм для отправки электронной почты.
- **Archetypes** — это подсистема (фреймворк) для создания *типов контента* в Plone версий 2.x, 3.x и 4.x. ***Dexterity*** заменяет её с версии 4.1+ и становится подсистемой по умолчанию в Plone 5. ***Archetypes*** остаётся доступной в пятой версии Plone.
- **Dexterity** — стандратная контент-подсистема для Plone 5 и последующих версий.
- **Validator** — описывает тип поля из *Archetypes*. Возвращает "True", если проверка пройдена, или возвращает строку с описанием ошибки, если проверка провалена.

Когда PFG установлен в Plone через "добавление/удаление продуктов" (*видимо через https://example.com/prefs_install_products_form*), он будет искать наличие аддона *collective.captcha* или *collective.recaptcha*. Если они найдены, то в список доступных полей будет добавлено поле "CAPTCHA".

Если вы используете *collective.recaptcha*, то вам будет необходимо добавить свою *ключевую пару*, полученную в вами созданном бесплатном аккаунте сервиса [reCAPTCHA](https://www.google.com/recaptcha). Добавление ключевой пары происходит в PFG-конфиглете в настройках сайта Plone.

> Если вы добавили аддон *captcha* после установки PFG, то необходимо переустановить PFG (через "добавление/удаление продуктов") для включения поддержки captcha.

### Установка collective.recaptcha
Добавьте нижеследующий код в ваш buildout.cfg для установки двух аддонов *collective.recaptcha* и *Products.PloneformGen*:
```
[buildout]
...
eggs =
    Plone
    ...
    collective.recaptcha
    Products.PloneFormGen
    ...
```

- Стандартно выполните `bin/buildout` и перезапустите ваш экземпляр Plone.
- Откройте *PortalQuickinstaller* или контрольную панель Plone и установите (или переустановите при необходимости) PloneFormGen.
- Откройте PFG-конфиглет в настройках сайта Plone и заполните необходимые поля для ввода полученных из [reCAPTCHA](https://www.google.com/recaptcha) *публичного и приватного ключа*.

Осталось понять как прикрутить проверку CAPTCHA к формам входа на сайт и страницы обратной связи...
