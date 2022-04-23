---
title: "IPA Error 4301 Certificate operation cannot be completed: Unable to communicate with CMS (500)"
date: 2021-05-12
weight: 10
description: >
  После обновления Tomcat, во FreeIPA пропала связь с сертификатами. Проблема временно решается добавлением аттрибута 'secretRequired="true"' в AJP Connector.
tags:
  - FreeIPA
  - Apache Tomcat
---

2021-05-12

## Симптомы
Сразу после установки новой FreeIPA на сервер RedOS 7.2, при посещении страницы "Сертификаты", после долгого ожидания в виде вывески "В процессе":
![FreeIPA Certificates Wait](/img/ipa-error-4301-certificate-operation-cannot-be-completed-unable-to-communicate-with-cms-500/freeipa-certificates-wait.png)

Далее появляется баннер с ошибкой:
```
IPA Error 4301: CertificateOperationError
Certificate operation cannot be completed: Unable to communicate with CMS (500)
```
![IPA Error 4301](/img/ipa-error-4301-certificate-operation-cannot-be-completed-unable-to-communicate-with-cms-500/ipa_error_4301.png)

Невозможно выполнение:
```
# ipa cert-show 1
ipa: ERROR: Certificate operation cannot be completed: Unable to communicate with CMS (500)
```

## Диагноз
Строки из 'journalctl -u pki-tomcatd@pki-tomcat' указывают причину такого поведения:
```
SEVERE: Failed to start component [Connector[AJP/1.3-8009]]
Caused by: java.lang.IllegalArgumentException: The AJP Connector is configured with secretRequired="true" but the secret attribute is either null or "". This combination is not valid.
```

Полный листинг:
```
# journalctl -u pki-tomcatd@pki-tomcat
May 11 20:05:28 dev-ipa01.test.lan systemd[1]: Starting PKI Tomcat Server pki-tomcat...
May 11 20:05:30 dev-ipa01.test.lan pki-server[1894]: ----------------------------
May 11 20:05:30 dev-ipa01.test.lan pki-server[1894]: pki-tomcat instance migrated
May 11 20:05:30 dev-ipa01.test.lan pki-server[1894]: ----------------------------
May 11 20:05:32 dev-ipa01.test.lan pkidaemon[1923]: -----------------------
May 11 20:05:32 dev-ipa01.test.lan pkidaemon[1923]: Banner is not installed
May 11 20:05:32 dev-ipa01.test.lan pkidaemon[1923]: -----------------------
May 11 20:05:32 dev-ipa01.test.lan pkidaemon[1923]: ----------------------
May 11 20:05:32 dev-ipa01.test.lan pkidaemon[1923]: Enabled all subsystems
May 11 20:05:32 dev-ipa01.test.lan pkidaemon[1923]: ----------------------
May 11 20:05:32 dev-ipa01.test.lan systemd[1]: Started PKI Tomcat Server pki-tomcat.
May 11 20:05:32 dev-ipa01.test.lan server[2064]: Java virtual machine used: /usr/lib/jvm/jre-1.8.0-openjdk/bin/java
May 11 20:05:32 dev-ipa01.test.lan server[2064]: classpath used: /usr/share/tomcat/bin/bootstrap.jar:/usr/share/tomcat/bin
/tomcat-juli.jar:/usr/lib/java/commons-daemon.jar
May 11 20:05:32 dev-ipa01.test.lan server[2064]: main class used: org.apache.catalina.startup.Bootstrap
May 11 20:05:32 dev-ipa01.test.lan server[2064]: flags used: -DRESTEASY_LIB=/usr/share/java/resteasy
May 11 20:05:32 dev-ipa01.test.lan server[2064]: options used: -Dcatalina.base=/var/lib/pki/pki-tomcat -Dcatalina.home=/us
r/share/tomcat -Djava.endorsed.dirs= -Djava.io.tmpdir=/var/lib/pki/pki-tomcat/temp -Djava.util.logging.config.file=/var/lib/pki
/pki-tomcat/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager     -Djava.security.manag
er     -Djava.security.policy==/var/lib/pki/pki-tomcat/conf/catalina.policy
May 11 20:05:32 dev-ipa01.test.lan server[2064]: arguments used: start
May 11 20:05:40 dev-ipa01.test.lan server[2064]: SEVERE: Failed to start component [Connector[AJP/1.3-8009]]
May 11 20:05:40 dev-ipa01.test.lan server[2064]: org.apache.catalina.LifecycleException: Protocol handler start failed
May 11 20:05:40 dev-ipa01.test.lan server[2064]: at org.apache.catalina.connector.Connector.startInternal(Connector.java:1
075)
May 11 20:05:40 dev-ipa01.test.lan server[2064]: at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:183)
May 11 20:05:40 dev-ipa01.test.lan server[2064]: at org.apache.catalina.core.StandardService.startInternal(StandardService
.java:451)
May 11 20:05:40 dev-ipa01.test.lan server[2064]: at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:183)
May 11 20:05:40 dev-ipa01.test.lan server[2064]: at org.apache.catalina.core.StandardServer.startInternal(StandardServer.j
ava:930)
May 11 20:05:40 dev-ipa01.test.lan server[2064]: at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:183)
May 11 20:05:40 dev-ipa01.test.lan server[2064]: at org.apache.catalina.startup.Catalina.start(Catalina.java:772)
May 11 20:05:40 dev-ipa01.test.lan server[2064]: at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
May 11 20:05:40 dev-ipa01.test.lan server[2064]: at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.jMay 11 20:05:40 dev-ipa01.test.lan server[2064]: at org.apache.catalina.connector.Connector.startInternal(Connector.java:1
075)
May 11 20:05:40 dev-ipa01.test.lan server[2064]: at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:183)
May 11 20:05:40 dev-ipa01.test.lan server[2064]: at org.apache.catalina.core.StandardService.startInternal(StandardService
.java:451)
May 11 20:05:40 dev-ipa01.test.lan server[2064]: at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:183)
May 11 20:05:40 dev-ipa01.test.lan server[2064]: at org.apache.catalina.core.StandardServer.startInternal(StandardServer.j
ava:930)
May 11 20:05:40 dev-ipa01.test.lan server[2064]: at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:183)
May 11 20:05:40 dev-ipa01.test.lan server[2064]: at org.apache.catalina.startup.Catalina.start(Catalina.java:772)
May 11 20:05:40 dev-ipa01.test.lan server[2064]: at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
May 11 20:05:40 dev-ipa01.test.lan server[2064]: at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.j
ava:62)
May 11 20:05:40 dev-ipa01.test.lan server[2064]: at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
May 11 20:05:40 dev-ipa01.test.lan server[2064]: at java.lang.reflect.Method.invoke(Method.java:498)
May 11 20:05:40 dev-ipa01.test.lan server[2064]: at org.apache.catalina.startup.Bootstrap.start(Bootstrap.java:342)
May 11 20:05:40 dev-ipa01.test.lan server[2064]: at org.apache.catalina.startup.Bootstrap.main(Bootstrap.java:473)
May 11 20:05:40 dev-ipa01.test.lan server[2064]: Caused by: java.lang.IllegalArgumentException: The AJP Connector is configured with secretRequired="true" but the secret attribute is either null or "". This combination is not valid.
May 11 20:05:40 dev-ipa01.test.lan server[2064]: at org.apache.coyote.ajp.AbstractAjpProtocol.start(AbstractAjpProtocol.java:270)
May 11 20:05:40 dev-ipa01.test.lan server[2064]: at org.apache.catalina.connector.Connector.startInternal(Connector.java:1072)
May 11 20:05:40 dev-ipa01.test.lan server[2064]: ... 12 more
```

Проверим версию установленных в систему Apache и Tomcat:
```
# rpm -qa httpd
httpd-2.4.29-9.el7.x86_64

# rpm -qa tomcat
tomcat-9.0.44-1.el7.noarch
```

В Tomcat'е, в предыдущей установленной в системе версии 9.0.13, присутствовала потенциальная уязвимость CVE-2020-1938. Для её решения ввели передачу secret'а при использовании протокола AJP, через который в FreeIPA общаются Apache и DogTag. В результате обновления Tomcat обновился до версии 9.0.44, где предлагается использовать secret, но Apache, и его настройки, остался прежним.

Пока не будет обновлён Apache до версии 2.5, то применяем обходной маневр. Так как установленный в системе Apache имеет версию 2.4, модуль `mod_proxy_ajp` которой не умеет передавать параметр 'secret', то необходимо в соответствующем месте Tomcat'а указать аттрибут, отключающий проверку секрета.

## Лечение
Добавить в `/etc/pki/pki-tomcat/server.xml` в строку коннектора аттрибут 'secretRequired="false"':
```
    <!-- Define an AJP 1.3 Connector on port 8009 -->
    <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" address="localhost" secretRequired="false"/>
```

После чего перезапускаем сервис 'pki-tomcatd@pki-tomcat':
```
sudo systemctl restart pki-tomcatd@pki-tomcat
```

## Результат
![FreeIPA Certificates](/img/ipa-error-4301-certificate-operation-cannot-be-completed-unable-to-communicate-with-cms-500/freeipa_authentication_success.png)
```
# ipa cert-show 1
  Issuing CA: ipa
  Certificate: MIIDgjCCAmqgAwIBAgIBATANBgkqhkiG9w0BAQsFADAzMREwDwYDVQQKDAhURVNULkxBTjEeMBwGA1UEAwwVQ2VydGlmaWNhdGUgQXV0aG9yaXR5MB4XDTIxMDUwNDE4MzYwNFoXDTQxMDUwNDE4MzYwNFowMzERMA8GA1UECgwIVEVTVC5MQU4xHjAcBgNVBAMMFUNlcnRpZmljYXRlIEF1dGhvcml0eTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBANwBn6gEBu73KL4OT/yuZNMNaL7esYVhQclx4d3lVVAFS7lXk/7Hh91MY39IL/N0teNvTnZn3I9E7VtkgATBvxXzzaLR+iMvDH1OWLIIcLuh4RfR8mVH5PRs1374aXrRRHlrQXElqm7kRt4yWQXb2ibagdnnQxdf2CvstA0HQCDg5DlNprVXu82RWomiacayyadnaZ/KKJmJAHhV4fY0I8tLn6uyVdOspfgA2ZQUaAXOuRtrbGuowegEbCurRO5egDl7GpSR6fOkxiFcnI/jEufatNe3ZQllF3ywT8rfT+oa+52BPCl9J25XrJYw0g71npocVkRi4gY86h0C2LEHeGMCAwEAAaOBoDCBnTAfBgNVHSMEGDAWgBSSy/ljEEs9wsV19d9s4SzwgaQnqjAPBgNVHRMBAf8EBTADAQH/MA4GA1UdDwEB/wQEAwIBxjAdBgNVHQ4EFgQUksv5YxBLPcLFdfXfbOEs8IGkJ6owOgYIKwYBBQUHAQEELjAsMCoGCCsGAQUFBzABhh5odHRwOi8vaXBhLWNhLnRlc3QubGFuL2NhL29jc3AwDQYJKoZIhvcNAQELBQADggEBAJ5t2ZOsPPl121dmCChndCD2aPAWjDZR5FYlEZZN8pk7ApoZ1yDDqPcX/NS+h3Lsj2SqDR1YjljkU6yLdaZy9RJ6NqfDnqt6EIvX5TM6KEuU8ga78tzP0/gJESeFr4oAh1wy/YwQrl2qhaI23SrRQEQXsxdi1s9fBBpAMlszcu5YYgvMimPAoKTdfAvm/nAfH6tcP6Hfvj5aNwQLPqc+S0xrJanAs97mktp84z1rtY5hjnrYKAYGQIE51h/v+f7grnmShiM55qu+A+qxYBqUozf0pNQ/Mo5zB8WongizCYyqjnhBHhdh/E9ca9kmkz7xEdgwxxTqe958WZ/9K+PgtWs=
  Subject: CN=Certificate Authority,O=TEST.LAN
  Issuer: CN=Certificate Authority,O=TEST.LAN
  Not Before: Tue May 04 18:36:04 2021 UTC
  Not After: Sat May 04 18:36:04 2041 UTC
  Serial number: 1
  Serial number (hex): 0x1
  Revoked: False
```

```
# journalctl -u pki-tomcatd@pki-tomcat
May 11 20:40:20 dev-ipa01.test.lan systemd[1]: Starting PKI Tomcat Server pki-tomcat...
May 11 20:40:20 dev-ipa01.test.lan pki-server[3130]: ----------------------------
May 11 20:40:20 dev-ipa01.test.lan pki-server[3130]: pki-tomcat instance migrated
May 11 20:40:20 dev-ipa01.test.lan pki-server[3130]: ----------------------------
May 11 20:40:20 dev-ipa01.test.lan pkidaemon[3157]: -----------------------
May 11 20:40:20 dev-ipa01.test.lan pkidaemon[3157]: Banner is not installed
May 11 20:40:20 dev-ipa01.test.lan pkidaemon[3157]: -----------------------
May 11 20:40:20 dev-ipa01.test.lan pkidaemon[3157]: ----------------------
May 11 20:40:20 dev-ipa01.test.lan pkidaemon[3157]: Enabled all subsystems
May 11 20:40:20 dev-ipa01.test.lan pkidaemon[3157]: ----------------------
May 11 20:40:20 dev-ipa01.test.lan systemd[1]: Started PKI Tomcat Server pki-tomcat.
May 11 20:40:20 dev-ipa01.test.lan server[3284]: Java virtual machine used: /usr/lib/jvm/jre-1.8.0-openjdk/bin/java
May 11 20:40:20 dev-ipa01.test.lan server[3284]: classpath used: /usr/share/tomcat/bin/bootstrap.jar:/usr/share/tomcat/bin
May 11 20:40:20 dev-ipa01.test.lan server[3284]: main class used: org.apache.catalina.startup.Bootstrap
May 11 20:40:20 dev-ipa01.test.lan server[3284]: flags used: -DRESTEASY_LIB=/usr/share/java/resteasy
May 11 20:40:20 dev-ipa01.test.lan server[3284]: options used: -Dcatalina.base=/var/lib/pki/pki-tomcat -Dcatalina.home=/us
May 11 20:40:20 dev-ipa01.test.lan server[3284]: arguments used: start
```
