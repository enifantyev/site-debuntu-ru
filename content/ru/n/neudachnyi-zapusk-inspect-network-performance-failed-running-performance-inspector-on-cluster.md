---
title: "Неудачный запуск 'Inspect Network Performance': Failed running performance inspector on cluster"
date: 2020-11-08
weight: 10
description: >
  На новом кластере проваливался тест Inspect Network Performance. Причина в установленном по умолчанию python3 на CentOS7.
tags:
  - Cloudera Inspect Network Performance
---

## Проблема

В результате запуска на кластере проверки "Inspect Network Performance" мы можем наблюдать картину:

![Failed running performance inspector on cluster](/img/neudachnyi-zapusk-inspect-network-performance-failed-running-performance-inspector-on-cluster/failed_running_performance_inspector_on_cluster.png)

В логах `/var/log/cloudera-scm-agent/cloudera-scm-agent.log`:
```
Thread-17 supervisor WARNING Failed while getting process info. Retrying. (<Fault
10: 'BAD_NAME: 3050-host-perf-inspector'>)
```

В `/var/run/cloudera-scm-agent/process/3050-host-perf-inspector/logs` присутствует пустой файл `stdout.log`, а в файле `stderr.log` видна ошибка "SyntaxError: invalid syntax":
```
[21/Oct/2020 14:56:56 +0000] 105531 MainThread redactor     INFO     Started launcher: /opt/cloudera/cm-agent/service/perf/host_perf_diag.py input.json logs/result.json
[21/Oct/2020 14:56:56 +0000] 105531 MainThread redactor     ERROR    Redaction rules file doesn't exist, not redacting logs. file: redaction-rules.json, directory: /run/cloudera-scm-agent/process/3050-host-perf-inspector
[21/Oct/2020 14:56:56 +0000] 105531 MainThread redactor     INFO     Re-exec watcher: /opt/cloudera/cm-agent/bin/cm proc_watcher 105541
  File "/opt/cloudera/cm-agent/service/perf/host_perf_diag.py", line 156
    except RuntimeError, rte:
                       ^
SyntaxError: invalid syntax
```

## Решение
На наших машинах с CentOS7 на борту по умолчанию запускается 'python3'. Поэтому для того, чтобы в кластере заработал "network-inspector" необходимо поменять шебанг у файла `/opt/cloudera/cm-agent/service/perf/host_perf_diag.py`. Добавить цифру '2' к '#!/usr/bin/python', чтобы было так:
```
#!/usr/bin/python2
```

После применения решения можем наблюдать успешное выполнения теста:
![Successfully completed running performance inspector on cluster](/img/neudachnyi-zapusk-inspect-network-performance-failed-running-performance-inspector-on-cluster/successfully_completed_running_performance_inspector_on_cluster.png)
