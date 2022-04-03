---
title: "Отправка почты из контактной формы MODx Evo через gmail"
date: 2013-12-12
weight: 10
description: >
  Отправка почты из контактной формы MODx Evo через gmail
tags:
  - MODx
  - CMS
---

<p>Появился сайт на MODx'е. Отправка почты из контактной формы не работает. Разработчик ушёл в оффлайн. С MODx'ом я ранее дела не имел. Не уверен, что я правильно всё сделал, но пока работает.</p>
<p>В "Инструменты - Конфигурация - Пользователи" заполнено всё правильно:</p>
<table cellpadding="3" border="0" cellspacing="0">
<tbody>
<tr>
<td><strong>Метод отправки писем</strong></td>
<td> MAIL - cообщения отправляются с помощью функции mail().<br /><strong> Через SMTP-сервер</strong></td>
</tr>
<tr>
<td><strong>SMTP авторизация</strong></td>
<td><strong> Да</strong><br /> Нет</td>
</tr>
<tr>
<td><strong>Адрес SMTP-сервера</strong></td>
<td>smtp.gmail.com</td>
</tr>
<tr>
<td><strong>SMTP порт</strong></td>
<td>465</td>
</tr>
<tr>
<td><strong>SMTP почта</strong></td>
<td>ivanmoskvinvogradetule@example.com</td>
</tr>
<tr>
<td><strong>SMTP пароль</strong></td>
</tr>
</tbody>
</table>
<p>Зашёл через ssh на хостинг и дал команду:</p>
<pre><strong>$ grep -ri smtpsecure *</strong>
public_html/manager/includes/controls/phpmailer/class.phpmailer.php:  public $SMTPSecure    = '';
public_html/manager/includes/controls/phpmailer/class.phpmailer.php:        $tls = ($this-&gt;SMTPSecure == 'tls');
public_html/manager/includes/controls/phpmailer/class.phpmailer.php:        $ssl = ($this-&gt;SMTPSecure == 'ssl');</pre>
<p>Почесав репу, изменил в public_html/manager/includes/controls/phpmailer/class.phpmailer.php строку:</p>
<pre>public $SMTPSecure    = 'ssl';</pre>
<p>Отправка заработала.</p>
