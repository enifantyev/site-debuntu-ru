---
title: "Срипт для добавления PTR-записей по списку из файла в обратную DNS-зону FreeIPA"
date: 2021-01-25
weight: 10
description: >
  Разбирая строки из файла формата hosts, скрипт добавляет PTR-записи в соответствующие обратные зоны.
tags:
  - FreeIPA
  - DNS
  - Bash
---

2021-01-25

Образец файла со списком узлов в формате hosts:
```
# cat ip.hosts
10.1.195.1 is01-airf01p.example.org
10.1.195.3 is01-app01p.example.org
10.1.195.4 is01-app02p.example.org
10.1.112.52 is02-app208p.example.org
10.1.112.53 is02-app209p.example.org
```

Скрипт для добавления PTR-записей в соответствующие обратные зоны:
```
add_dns_record.sh
#!/bin/bash
LIST="ip.hosts"
 
while read line; do
  echo ""
  echo $line     # "10.1.195.1 is01-airf01p.example.org"
  arr1=($line)   # конвертируем переменную в массив из двух элементов
  ip1=${arr1[0]} # "10.1.195.1"
  IFS="."        # разделитель пробел меняем на точку для bash substring
  arr2=($ip1)    # конвертируем ip-адрес в массив "10 15 195 1"
  IFS=" "        # возвращаем разделить
 
  ZONE="${arr2[2]}.${arr2[1]}.${arr2[0]}.in-addr.arpa"
  IPN=${arr2[3]}
  NAME=${arr1[1]}"."
  echo "$IPN $NAME"
  echo "ipa dnsrecord-add $ZONE $IPN --ptr-rec $NAME"
  ipa dnsrecord-add $ZONE $IP --ptr-rec $NAME
done < $LIST
```
