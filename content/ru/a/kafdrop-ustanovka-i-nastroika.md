---
title: "Kafdrop. 02. Установка и настройка"
date: 2021-11-25
weight: 10
description: >
  Установка Kafdrop, инструмента для просмотра тем Kafka и групп потребителей.
tags:
  - Apache Kafka
  - Kafdrop
  - CentOS
---

2021-11-25

## 1. Установка
### 1.1. Создание локального аккаунта
Создаём локальный аккаунт 'kafdrop':
```
$ useradd -r -s /sbin/nologin -d /opt/kafdrop -M kafdrop

$ kinit
$ ipa service-add kafdrop/$(hostname)
```

### 1.2. Создание рабочего каталога
Создаём каталог, где будет всё необходимое для работы Kafdrop:
```
mkdir /opt/kafdrop
chown kafdrop.kafdrop /opt/kafdrop
chmod 2770 /opt/kafdrop
#
```

### 1.3. Загрузка ранее скомпилированного jar-файла
Важно! Тем или иным образом копируем ранее собранный пакет `kafdrop-3.27.0.jar` в каталог `/opt/kafdrop`.

После чего делаем link с расчётом на дальнейшие обновления пакета:
```
cd /opt/kafdrop
ln -s kafdrop-3.27.0.jar kafdrop.jar
```

## 2. Подготовка
### 2.1. Создание во FreeIPA сервис-аккаунт 'kafdrop'
Создаём сервис-аккаунт во FreeIPA с таким же именем, не забывая заранее получить kerberos-тикет для УЗ, обладающей правом на добавление сервис-аккаунтов:
```
$ kinit
$ ipa service-add kafdrop/$(hostname)
```

Создаём keytab для сервисной УЗ 'kafdrop':
```
SERVICE="kafdrop"
REALMNAME="$(hostname -d | tr [:lower:] [:upper:])"
IPAADMIN="eugene"
 
## При необходимости получаем kerberos-билет
# kinit ${IPAADMIN}
 
## Если требуются права на создание ключей.
ipa service-allow-create-keytab ${SERVICE}/$(hostname)@${REALMNAME} --users=${IPAADMIN}
ipa-getkeytab -p ${SERVICE}/$(hostname) -k ~/${SERVICE}.keytab
ipa service-disallow-create-keytab ${SERVICE}/$(hostname)@${REALMNAME} --users=${IPAADMIN}
  
sudo mv ~/${SERVICE}.keytab /opt/${SERVICE}/
sudo chown ${SERVICE}.${SERVICE} /opt/${SERVICE}/${SERVICE}.keytab
sudo chmod 600 /opt/${SERVICE}/${SERVICE}.keytab
#
```

### 2.2. Настройка Kafka в Cloudera Managent
Необходимо добавить имя учётной записи 'kafdrop' в параметр 'super.users', в дополнение к уже существующей там записи для аккаунта 'kafka'.

В результате, Apache Kafka будет запускаться со следующей строкой в `kafka.properties':
```
super.users=User:kafka;User:kafdrop
```

### 2.3. Создание systemd-юнита
Добавляем юнит в systemd для автоматического запуска kafdrop:
```
cat << EOF | sudo tee /etc/systemd/system/kafdrop.service
[Unit]
Description=Kafdrop Service
After=network.target
 
[Service]
Type=simple
RuntimeDirectory=kafdrop
RuntimeDirectoryMode=0775
WorkingDirectory=/opt/kafdrop

User=kafdrop
Group=kafdrop

EnvironmentFile=/etc/sysconfig/kafdrop

ExecStart=/usr/bin/java --add-opens=java.base/sun.nio.ch=ALL-UNNAMED \\
  -Djava.security.auth.login.config=/opt/kafdrop/jaas.conf \\
  -jar /opt/kafdrop/kafdrop.jar \\
  --kafka.brokerConnect=\${KAFKABROKERS} \\
  --server.port=\${SRVPORT} --management.server.port=\${MNGSRVPORT}

TimeoutSec=30
 
[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
#
```

Делаем файл с переменными для запуска Kafdrop:
```
cat << EOF | sudo tee /etc/sysconfig/kafdrop
# List of Kafka Brokers
KAFKABROKERS="dev-dn110p.test2.lan:9092,dev-dn111p.test2.lan:9092,dev-dn112p.test2.lan:9092"

# Listen port
SRVPORT="8000"

# Management Server Port
MNGSRVPORT="8001"
EOF
#
```

## Настройка
### Создание jaas.conf
```
SERVICE="kafdrop"
REALMNAME="$(hostname -d | tr [:lower:] [:upper:])"
PRINCIPAL="${SERVICE}/$(hostname)@${REALMNAME}"
 
cat << EOF | sudo tee /opt/${SERVICE}/jaas.conf
KafkaClient {
  com.sun.security.auth.module.Krb5LoginModule required
  useKeyTab=true
  keyTab="/opt/${SERVICE}/${SERVICE}.keytab"
  storeKey=true
  principal="${PRINCIPAL}"
  serviceName=kafka
  debug=false;
};
EOF
 
sudo chown ${SERVICE}.${SERVICE} /opt/${SERVICE}/jaas.conf
#
```

### Создание kafka.properties
Создаём основной файл `kafka.properties`. Предполагается, что параметр 'security.inter.broker.protocol' в настройках Kafka-брокеров установлен в 'SASL_PLAINTEXT':
```
cat << EOF | sudo tee /opt/kafdrop/kafka.properties 
#ssl.keystore.location=/opt/cloudera/security/pki/server.jks
#ssl.keystore.password = changeit

#ssl.truststore.location = /usr/java/jdk1.8.0_181-cloudera/jre/lib/security/jssecacerts
#ssl.truststore.password = changeit

#ssl.enabled.protocols=TLSv1.2,TLSv1.1,TLSv1
#ssl.protocol=TLS
security.protocol=SASL_PLAINTEXT

#log4j.appender.console.threshold=DEBUG
EOF
#
```

### Установка прав на каталог `/opt/kafdrop`
Обновляем права:
```
sudo chown -R kafdrop.kafdrop /opt/kafdrop
```

## Запуск Kafdrop
### Запуск демона
Включаем и запускаем демон 'kafdrop.service':
```
sudo systemctl enable --now kafdrop
```

### Адрес Web UI
Web-морда Kafdrop доступна по адресу 'http://ip-address:8000/'. Необходимо помнить, что никаких встроенных средств для организации аутентификации и TLS-шифрования трафика в Kafdrop не предусмотрено. На сайте проекта https://github.com/obsidiandynamics/kafdrop для этих целей предлагается использовать Nginx.
