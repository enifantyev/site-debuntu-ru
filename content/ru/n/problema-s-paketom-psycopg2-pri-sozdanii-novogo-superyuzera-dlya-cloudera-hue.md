---
title: "Проблема с пакетом psycopg2 при создании нового суперюзера для Cloudera Hue"
date: 2021-06-09
weight: 10
description: >
  При создании нового суперюзера для Cloudera Hue происходит ошибка "psycopg2_version 2.5.4 or newer is required; you have 2.5.1 (dt dec pq3 ext)".
tags:
  - Hue
  - BigData
---

2021-06-09

Манипуляции производятся в Cloudera CDH 6.3.2. На машине с установленной ролью Hue выполняю команду и получаю ошибку:
```
# /usr/lib/hue/build/env/bin/hue createsuperuser --cm-managed
LD_LIBRARY_PATH can't be found, if you are using ORACLE for your Hue database
then it must be set, if not, you can ignore
  export LD_LIBRARY_PATH=/path/to/instantclient
Traceback (most recent call last):
  File "/usr/lib/hue/build/env/bin/hue", line 14, in <module>
    load_entry_point('desktop', 'console_scripts', 'hue')()
  File "/usr/lib/hue/desktop/core/src/desktop/manage_entry.py", line 225, in entry
    raise e
django.core.exceptions.ImproperlyConfigured: psycopg2_version 2.5.4 or newer is required; you have 2.5.1 (dt dec pq3 ext)
```

Проверяю версию установленной библиотеки и убеждаюсь, что версия 2.5.1:
```
# pip2 freeze|grep psycopg
You are using pip version 8.1.2, however version 21.1.2 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.
psycopg2==2.5.1
```

Устанавливаю последнюю версию пакета psycopg в бинарной версии, чтобы не устанавливать пакеты для сборки:
```
# pip2 install psycopg2-binary
Collecting psycopg2-binary
  Downloading http://nexus.example.org:8081/repository/all_pypi.org_proxy/packages/psycopg2-binary/2.8.6/psycopg2_binary-2.8.6-cp27-cp27mu-manylinux1_x86_64.whl (2.9MB)
    100% |████████████████████████████████| 2.9MB 96.1MB/s
Installing collected packages: psycopg2-binary
Successfully installed psycopg2-binary-2.8.6
You are using pip version 8.1.2, however version 21.1.2 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.
```

Вновь пытаюсь создать суперюзера и на этот раз успешно:
```
# /usr/lib/hue/build/env/bin/hue createsuperuser --cm-managed
LD_LIBRARY_PATH can't be found, if you are using ORACLE for your Hue database
then it must be set, if not, you can ignore
  export LD_LIBRARY_PATH=/path/to/instantclient
Username (leave blank to use 'root'): admhue
Email address: admhue@example.org
Password:
Password (again):
Superuser created successfully.
```

Убеждаюсь, что текущая настройка Hue&nbsp;&mdash;&nbsp;'Authentication Backend'&nbsp;&mdash; содержит параметр `desktop.auth.backend.AllowFirstUserDjangoBackend`:
![cloudera hue param authentication backend](/img/problema-s-paketom-psycopg2-pri-sozdanii-novogo-superyuzera-dlya-cloudera-hue/cloudera_hue_param_authentication_backend.png)

После чего я вхожу в Hue под новым суперюзером и выполняю необходимые настройки.
