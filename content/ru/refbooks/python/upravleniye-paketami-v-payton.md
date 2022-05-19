---
title: "Управление пакетами в Python"
date: 2
weight: 10
description: >
  Краткий справочник по работе с пакетами Python.
tags:
  - python
  - pip
  - venv
---

## Способы установки python-пакетов
### С помощью системного менеджера пакетов APT или YUM
```
sudo yum install python3-requests
```

### С помощью python-модуля pip
Если на машине установлена только одна версия python, или установка производится в активированном виртуальном окружении, то можно установить пакет/ы следующим образом:
```
pip install wheel requests
```

Узнать к какой версии Python принадлежит `pip`:
```
[eugene@wks1 ~]$ pip -V
pip 20.3.4 from /usr/lib/python2.7/site-packages/pip (python 2.7)
```

Исключающее ошибку в выборе версии Python, установка пакетов с использованием `pip`:
```
python3.8 -m pip install airflow
```

### Ручная сборка python-пакета
-"- дописать.

### Установка пакетов в виртуальное окружение
Создадим каталог для Python-проекта, где развернём виртуальное окружение, активируем его, после чего установим пакет wheel:
```
[eugene@wks1 ~]$ mkdir -p ~/pythonprojects/airflow
[eugene@wks1 ~]$ cd ~/pythonprojects/airflow
[eugene@wks1 ~]$ python3.6 -m venv .
[eugene@wks1 ~]$ source bin/activate
(airflow) [eugene@wks1 ~]$ pip -V
pip 18.1 from /root/pythonprojects/airflow/lib64/python3.6/site-packages/pip (python 3.6)
(airflow) [eugene@wks1 ~]$ pip install wheel
Collecting wheel
  Using cached wheel-0.37.1-py2.py3-none-any.whl (35 kB)
Installing collected packages: wheel
Successfully installed wheel-0.37.1
```

## Адреса директорий с установленными пакетами
1. Пакеты, добавляемые через `yum`, устанавливаются в зависимости от версии Python в каталог `/usr/lib/python3.x` и доступны для всей системы. Напомню, что пакеты устанавливаемые с помощью `pip install` или `python3 -m pip install` из-под 'root'а , устанавливаются также в `/usr/lib/python3.x`. Чтобы исключить такое поведение, необходимо устанавливать такие пакеты с опцией `--user`. В этом случае, сам пакет установится в `/root/.local/lib/python3.6`. Исполняемые же файлы размещаются в `/root/.local/bin`. Системные пакеты останутся нетронутыми, как и их зависимости.
2. Пакеты, добавляемые через `sudo pip install`, размещаются там же, как в примере выше и переписывают файлы, установленные через `yum`.
2. Пакеты, добавляемые из-под 'root'а через `sudo pip install --user`, устанавливаются в зависимости от версии Python, например в `/root/.local/lib/python3.8`, и доступны только для аккаунта `root`.
3. Пакеты, добавляемые через `pip install`, устанавливаются в зависимости от версии Python, например в `~/.local/lib/python3.8`, то есть в домашний каталог пользователя и доступны только пользователю.
4. Пакеты, устанавливаемые в виртуальном окружении, размещаются в этом же каталоге, где развёрнут 'venv', точнее в подкаталоге `./lib/python3.8`.

## Способы обновления пакетов
### Обновление pip
Запуск из-под обычного пользователя:
```
pip install --upgrade pip
```

Запуск из-под 'root'а:
```
pip install --user --upgrade pip
```

### Принудительное обновление пакета, который установлен через yum
```
ERROR: Cannot uninstall 'systemd-python'. It is a distutils installed project
and thus we cannot accurately determine which files belong to it which would
lead to only a partial uninstall.
```

```
pip install --ignore-installed systemd-python
```

## Способы удаления всех пакетов
### Способ с сохранением списка пакетов
Остаётся возможность восстановления всех пакетов из списка. Сохраняем список, удаляем пакеты, устанавливаем пакеты:
```
pip freeze > requirements.txt
pip uninstall -r requirements.txt -y
pip install -r requirements.txt
```

### Удаление всех пакетов без сохранения списка
```
pip uninstall -y -r <(pip freeze)
```
или
```
pip freeze | xargs pip uninstall -y
```
