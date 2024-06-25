---
title: "Запуск kafka-mirror-maker из systemd"
date: 2020-09-06
weight: 10
description: >
  Создание kafka-mirror-maker.unit для systemd
tags:
  - Apache Kafka
  - kafka-mirror-maker
  - BigData
---

Каталог размещения: /opt/hadoop. Сюда скачиваем и распаковываем kafka_2.13-2.6.0.tgz и создаём ссылку kafka на каталог kafka_2.13-2.6.0, куда был раскапован архив:
```
mkdir /opt/hadoop
cd /opt/hadoop
tar xvf kafka_2.13-2.6.0.tgz
ln -s kafka_2.13-2.6.0 kafka
mkdir kafka/logs
```

Создаём два файла для запуска и остановки kafka-mirror.
`kafka-mirror-maker-start.sh`
```
#!/bin/bash
echo "mirror maker restarted $(date)">>/opt/hadoop/kafka/logs/mirrormaker.log
cd /opt/hadoop/kafka/bin/
./kafka-mirror-maker.sh \
--consumer.config ../config/consumer.properties \
--producer.config ../config/producer.properties \
--whitelist 'adfox_created|adfox_deleted|adfox_updated|mymoscow_created|mymoscow_deleted|mymoscow_updated' \
--num.streams 3 \
--offset.commit.interval.ms 30000 \
--abort.on.send.failure false
```

`kafka-mirror-maker-stop.sh`
```
#!/bin/sh
SIGNAL=${SIGNAL:-TERM}
PIDS=$(ps ax | grep -i 'kafka.tools.Mirrormaker' | grep java | grep -v grep | awk '{print $1}')if [ -z "$PIDS" ]; then
 echo "No mirrormaker service to stop"
 echo "No mirrormaker service to stop" >> /opt/hadoop/kafka/logs/mirrormaker.log
 exit 1
else
 kill -s $SIGNAL $PIDS
 echo "stopped mirrormaker service"
 echo "stopped mirrormaker service $date" >> /opt/hadoop/kafka/logs/mirrormaker.log
fi
```

Создаём системного пользователя kafka, если такового не существует:
```
useradd --system --no-create-home kafka
```

Задаём права и владельцев для созданных файлов:
```
chown -R kafka.kafka kafka_2.13-2.6.0
chown kafka.kafka kafka-mirror-st*.sh
chmod u+x kafka-mirror-st*.sh
```

Создаём unit для kafka-mirror в каталоге /etc/systemd/system/
```
[Unit]
Description=KafkaMirror Daemon
Documentation=https://kafka.apache.org/documentation/
Requires=network.target
After=network.target

[Service]
User=kafka
Group=kafka
ExecStart=/opt/hadoop/kafka-mirror-start.sh
ExecStop=/opt/hadoop/kafka-mirror-stop.sh
TimeoutSec=60
Restart=on-failure

[Install]
WantedBy=default.target
```

Запускаем юнит и смотрим результаты:
```
systemctl daemon-reload
systemctl enable kafka-mirror
systemctl start kafka-mirror
journalctl -f
```
