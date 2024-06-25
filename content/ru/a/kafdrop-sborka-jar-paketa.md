---
title: "Kafdrop. 01. Сборка jar-пакета"
date: 2021-11-25
weight: 10
description: >
  Описание процесса сборки Kafdrop, инструмента для для просмотра тем Kafka и групп потребителей.
tags:
  - Apache Kafka
  - Kafdrop
  - CentOS
---

2021-11-25

## 1. Подготовка к сборке
В дополнение к Maven'у устанавливаем Java11:
```
$ sudo yum -y install java-11-openjdk java-11-openjdk-devel
```

Скачиваем последнюю версию приложения Kafdrop (на данный момент Kafdrop 3.27.0) и распаковываем:
```
FILE="3.27.0.tar.gz"

mkdir ~/src && cd ~/src
curl -LO https://github.com/obsidiandynamics/kafdrop/archive/refs/tags/${FILE}
tar xvf ${FILE}
#
```

## 2. Сборка jar-пакета
Запускаем сборку пакета:
```
export JAVA_HOME=/usr/lib/jvm/jre-11-openjdk

cd ~/src/kafdrop-3.27.0
mvn clean package
#
```

## 3. Результат
В каталоге `~/src/kafdrop-3.27.0/target` наблюдаем пакет `kafdrop-3.27.0.jar`.
