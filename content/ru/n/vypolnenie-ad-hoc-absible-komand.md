---
title: "Выполнение ad-hoc absible команд"
date: 2021-03-12
weight: 10
description: >
  Иногда удобно выполнить одну команду на множестве узлов без создания отдельного ansible playbook'а.
tags:
  - Ansible
  - Ansible ad-hoc
  - Automation
---

2021-03-12

- Для предотвращения предупреждения о несоответствии ssh-ключей, можно установить переменную:
`export ANSIBLE_HOST_KEY_CHECKING=False`
или, если не работает, то:
`export ANSIBLE_SSH_ARGS="-o UserKnownHostsFile=/dev/null"`
- Для дебаггинга:
`export ANSIBLE_DEBUG=1`
- Простейший inventory-файл представляет собой текстовый файл со списком из ip-адресов и/или dns-имён хостов.
- Вместо указания инвентори-файла, можно указать ip-адрес.

Далее примеры:
```
# Для предотвращения предупреждения о несоответствии ssh-ключей
export ANSIBLE_HOST_KEY_CHECKING=False
 
# важна запятая после указания одного ip-адреса или dns-имени
ansible all -i 10.1.112.1, -m shell -a 'hostname'
 
# здесь доп запятая не нужна
ansible all -i 10.1.112.1,prod-airf01p -m shell -a 'hostname'
 
# выполняем команды с правами root
ansible all -i 10.1.112.1,prod-airf01p -m shell -a 'cat /etc/shadow' --become
 
# копируем локальный файл в удалённый каталог на все узлы из инвентори-файла
ansible all -i list.hosts -m copy -a 'src=~/soft/nginx.rpm dest=/tmp/'
```

Я не нашёл способа указать подсеть, вместо одного или нескольких ip-адресов. Пришлось использовать inventory-файл. В простейшем случае собираем инвентори-файл:
```
# Вся подсеть 10.1.112.0/25
for i in {1..126}; do echo 10.1.112.$i >> hosts.txt; done
 
# Все узлы с открытым 22-ым портом в подсети 10.1.112.0/25
for i in {1..126}; do nmap -Pn -p22 10.1.112.$i|grep open && echo "10.1.112.$i" >> hosts.txt; done
```
