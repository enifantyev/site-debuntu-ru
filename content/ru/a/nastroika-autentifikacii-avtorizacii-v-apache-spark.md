---
title: "Настройка аутентификации/авторизации в Apache Spark"
date: 2021-04-30
weight: 10
description: >
  Описание где и что включить, чтобы закрыть несанкционированный доступ к Apache Spark.
tags:
  - Apache Spark
  - BigData
---

## Spark

| Property | Value | Description |
|---|:---:|---|
|**Spark Authentication**<br/>spark.authenticate|☑|Enable whether the Spark communication protocols do authentication using a shared secret.|


## Spark History
Если не настроено, то страница Spark History Web UI доступна анониму, где в логах аноним может увидеть использующиеся логины/пароли пользователей. За доступ к странице отвечают параметры:

| Property | Value | Description |
|---|:---:|---|
**Enable User Authentication**<br>history_server_spnego_enabled| ☑ | Enables user authentication using SPNEGO (requires Kerberos), and enables access control to application history data.|
|**Admin Users**<br>spark.history.ui.admin.acls|eugene,...|Comma-separated list of users who can view all applications when authentication is enabled.|


После включения первого параметра и с пустой строкой во втором параметре, анонимный доступ к Spark History прекращается. Страница доступна только при наличии в веб-браузере действующего Kerberos-билета. В случае отсутствия валидного Kerberos-тикета, страница возвращает сообщение "HTTP ERROR 401. Authentication required".

После заполнения второго параметра, например 'admin' или любое другое случайное значение, остаётся возможность просмотра страницы из-под любого пользователя с валидным Kerberos-тикетом, но нельзя пройти по ссылкам к логам приложений, присутствующих на главной странице.

После заполнения второго параметра действующими логинами IPA-аккаунтов (пользовательские группы не работают), этим пользователям предоставляется доступ к логами приложений.
