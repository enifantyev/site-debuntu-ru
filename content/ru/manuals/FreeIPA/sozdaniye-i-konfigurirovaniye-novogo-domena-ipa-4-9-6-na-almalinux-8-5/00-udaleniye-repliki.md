---
title: "00. Удаление реплики"
date: 2021-12-27
weight: 10
description: >
  Описание зачистки машин от неудачно установленного экзмепляра FreeIPA.
tags:
  - FreeIPA
  - FreeIPA 4.9.6
  - AlmaLinux 8.5
slug: udaleniye-repliki
---

2021-12-27

## 1. Ссылки на использованные материалы
[Chapter 20. Uninstalling an IdM replica](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/8/html/installing_identity_management/uninstalling-an-idm-replica_installing-identity-management)

## 2. Перед удалением
Необходимо убедиться, что роли, назначенные на удаляемый сервер, не являются последними в домене. Если удаляемая реплика имеет роль 'CA master' или ей назначена роль генератора 'CRL списков', то необходимо перевести эти роли на другой сервер.

## 3. Неполное удаление для повторного добавления реплики
### 3.1. Операции на удаляемом сервере
3.1.1. На удаляемом сервере запускаем процедуру удаления IPA-сервера:
  ```bash
  $ sudo ipa-server-install --uninstall
  ```
3.1.2. Здесь же зачищаем логи скриптов установки/удаления IPA-сервера:
  ```bash
  sudo rm -rf /var/log/ipa*.log
  ```
3.1.3. Перезагружаем хост:
  ```bash
  sudo reboot
  ```

### 3.2. Операции на остающихся IPA-репликах
3.2.1. В случае невозможности полноценного выполнения пункта 3.1., удаляем реплику из топологии используя любую машину в домене. Для этого, из командной строки любой оставшейся в домене машины выполняем:
  ```bash
  $ kinit admin
  $ export REPLICA="replica.example.org"
  $ ipa server-del ${REPLICA}
  ```
3.2.2. Проверяем через Web UI, что упоминания об удаляемом сервере не остались в следующих местах:
  - Список IPA-серверов (https://dev-ipa04p.test3.lan/ipa/ui/#/e/server/search).
  - Список хостов (https://dev-ipa04p.test3.lan/ipa/ui/#/e/host/search).
  - Записи в DNS-зоне (https://dev-ipa04p.test3.lan/ipa/ui/#/e/dnszone/records/test3.lan.).

3.2.3. Проверяем через Apache Directory Studio, что упоминания об удаляемом сервере зачищены в следующих местах:

  - Убрать диапазон номеров выдаваемых сертификатов cn=20000001,ou=requests,ou=ranges,o=ipaca.
  - Удалить упоминания из ou=people,o=ipaca.
  - Удалить упоминания из ou=certificateRepository,ou=ranges,o=ipaca.
  - Удалить упоминания из cn=CAList,ou=Security Domain,o=ipaca.

3.2.4. Ещё посмотреть упоминания об удалённом сервере в 'o=ipaca'. Удалять выданные для этого сервера сертификаты не требуется.

> После удаления сервера потребуется восстановить содержимое файла `/etc/resolv.conf`.

## 4. Полное удаление

После "неполного" удаления выполняем дополнительные шаги.

4.1. Удаляем пакеты:
  ```bash
  sudo dnf -y module remove idm:DL1/dns
  sudo dnf -y autoremove python3-ipaserver
  ```

4.2. Переключаемся на поток 'idm:client':
  ```bash
  sudo dnf -y module enable idm:client
  sudo dnf -y module reset idm:client
  sudo dnf -y distro-sync
  ```

4.3. Зачищаем логи и прочие каталоги, после чего перезагружаем хост:
  ```bash
  sudo rm -rf /var/log/ipa*.log \
    /var/log/custodia \
    /var/log/dirsrv \
    /var/log/httpd \
    /var/log/ipa \
    /var/log/pki \
    /var/log/opencryptoki

  sudo rm -rf /etc/custodia/* \
    /etc/dirsrv \
    /etc/ipa \

  sudo rm -rf /var/lib/certmonger/* \
    /var/lib/custodia \
    /var/lib/dirsrv \
    /var/lib/httpd \
    /var/lib/ipa \
    /var/lib/opencryptoki

  sudo reboot
  ```

4.4. После удаления сервера потребуется восстановить содержимое файла `/etc/resolv.conf`.
