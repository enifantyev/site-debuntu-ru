---
title: "08. Первый запуск Apache Atlas и его отладка при необходимости"
date: 2021-11-02
weight: 12
description: >
  Описание процесса установки Apache Atlas 2.2.0 в Cloudera CDH 6.3.2.
tags:
  - BigData
  - Apache Atlas
  - Cloudera CDH 6.3.2
slug: pervyy-zapusk-apache-atlas-i-yego-otladka
---

2021-11-02

## 1. Первый запуск

1.1. Для надёжности выполняем изменение владельца для всего каталога `/opt/atlas`:
```
sudo chown atlas.atlas /opt/atlas/* -R
```

1.2. Перед первым запуском поменяем уровень логгирования с INFO на DEBUG. Так удобней отслеживать процесс первоначального запуска. В файле `/opt/atlas/conf/atlas-log4j.xml` приводим соответствующую строку к следующему виду:
```
    <logger name="org.apache.atlas" additivity="false">
        <level value="debug"/>
        <appender-ref ref="FILE"/>
    </logger>
```

1.3. Первый запуск Apache Atlas производится командой:
```
sudo systemctl enable --now atlas
```

1.4. Наблюдаем процесс запуска в лог-файле /opt/atlas/logs/application.log. Если процесс останавливается на продолжительное время (примерно больше 10 минут) и Web UI не появляется, то просматриваем предыдущие строки в поисках ошибок. При необходимости, останавливаем Atlas и, после увеличения уровня логов до TRACE, вновь запускаем Atlas.

1.5. **Ожидание появления Web UI, при первом запуске Atlas'а на тестовом кластере, занимало от 20 минут до двух часов**. Пока неизвестны причины такого поведения.
Повторные запуски не были такими долгими и по времени занимали, в общей сложности, не более минуты.

## 2. Web UI Apache Atlas
2.1. После долгого ожидания, на порту 21443 Atlas-хоста появится авторизационная форма Atlas'а:
![Apache Atlas Login Windows](/img/pervyy-zapusk-apache-atlas-i-yego-otladka/apacheatlaslogin.png)

2.2. После входа в Atlas, в главном окне можно наблюдать пустоту с надписью, что сущностей HBase нет, или, в другом случае, информацию только о двух Atlas'овских таблицах и столбцах из них, даже если в HBase и Hive уже существуют другие базы и таблицы:
![Apache Atlas Main Windows](/img/pervyy-zapusk-apache-atlas-i-yego-otladka/apacheatlasmainwindow.png)

2.3. Для приведения базы данных Atlas'а в актуальное состояние, необходимо выполнить первоначальный импорт сведений о содержимом в HBase и Hive, как описано в одних их следующих главах этой инструкции.

## 3. Отладка
### 3.1. Установка уровня логгирования для отладки
3.3.1. При необходимости отладки меняем уровень логгирования в файле `/opt/atlas/conf/atlas-log4j.xml`. В большинстве случаев достаточно отрегулировать первый логгер "org.apache.atlas":
```
    <logger name="org.apache.atlas" additivity="false">
        <level value="debug"/>
        <appender-ref ref="FILE"/>
    </logger>

    <logger name="org.janusgraph" additivity="false">
        <level value="warn"/>
        <appender-ref ref="FILE"/>
    </logger>
```

3.3.2. Уровень по умолчанию 'info' или 'warn' увеличиваем до достаточного в большинстве случаев уровня 'debug', или до максимально многословного 'trace'.

3.3.3. Файлы логов находятся в каталоге `/opt/atlas/logs`.
При достижении лог-файла размера 100MB, производится автоматический rotate.

### 3.2. Ручной запуск и остановка Atlas'а
3.2.1 Убедившись, что в оперативной памяти нет активных Atlas-процессов, выполняем ручной запуск:
```
sudo -u atlas /opt/atlas/bin/atlas-start.py
```

3.2.2 Ручная остановка Atlas'а выполняется командой:
```
sudo -u atlas /opt/atlas/bin/atlas-stop.py
```

### 3.3. Проверка принадлежности аккаунта к группам
3.3.1. Просмотр UGI — User Group Information для текущего пользователя и для аккаунта 'atlas':
```
$ kinit

$ hdfs groups
eugene@TEST2.LAN : eugene dud_admins test2_sentry_admins test2_hive_admins test2_solr_admins test2_yarn_admins test2_hue_admins test2_hbase_su test2_hdfs_su test2_airflow_airf1 test2_hue test2_atlas_admins

$ hdfs groups atlas
atlas : atlas hadoop
```

3.3.2. Обновление user and group mappings:
```
$ dfs dfsadmin -refreshUserToGroupsMappings
Refresh user to groups mapping successful for dev-nn110p.test2.lan/10.1.137.5:8020
Refresh user to groups mapping successful for dev-nn111p.test2.lan/10.1.137.4:8020

$yarn rmadmin -refreshUserToGroupsMappings
```
