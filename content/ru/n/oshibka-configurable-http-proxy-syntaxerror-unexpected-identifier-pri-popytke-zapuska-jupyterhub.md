---
title: "Ошибка configurable-http-proxy: SyntaxError: Unexpected identifier при попытке запуска JupyterHub"
date: 2021-07-07
weight: 10
description: >
  Последняя версия configurable-http-proxy мешает запуску JupyterHub.
tags:
  - JupiterHub
  - Python

categories:
  - Development
---

2021-07-07

После установки JupyterHub наблюдается недачный запуск systemd-юнита 'jupyterhub.service'.

Здесь приведён фрагмент из syslog'а с ошибками после запуска jupyterhub.service:
```
jupyterhub: [I 2021-07-07 15:05:40.359 JupyterHub proxy:703] Starting proxy @ http://0.0.0.0:8000/
jupyterhub: /usr/lib/node_modules/configurable-http-proxy/node_modules/prom-client/lib/registry.js:25
jupyterhub: async getMetricAsPrometheusString(metric) {
jupyterhub: ^^^^^^^^^^^^^^^^^^^^^^^^^^^
jupyterhub: SyntaxError: Unexpected identifier
jupyterhub: at createScript (vm.js:56:10)
jupyterhub: at Object.runInThisContext (vm.js:97:10)
jupyterhub: at Module._compile (module.js:549:28)
jupyterhub: at Object.Module._extensions..js (module.js:586:10)
jupyterhub: at Module.load (module.js:494:32)
jupyterhub: at tryModuleLoad (module.js:453:12)
jupyterhub: at Function.Module._load (module.js:445:3)
jupyterhub: at Module.require (module.js:504:17)
jupyterhub: at require (internal/module.js:20:19)
jupyterhub: at Object.<anonymous> (/usr/lib/node_modules/configurable-http-proxy/node_modules/prom-client/index.js:8:20)
jupyterhub: [C 2021-07-07 15:05:41.436 JupyterHub app:2739] Failed to start proxy
jupyterhub: Traceback (most recent call last):
jupyterhub: File "/opt/jupyterhub/lib64/python3.6/site-packages/jupyterhub/app.py", line 2737, in start
jupyterhub: await self.proxy.start()
jupyterhub: File "/opt/jupyterhub/lib64/python3.6/site-packages/jupyterhub/proxy.py", line 730, in start
jupyterhub: _check_process()
jupyterhub: File "/opt/jupyterhub/lib64/python3.6/site-packages/jupyterhub/proxy.py", line 726, in _check_process
jupyterhub: raise e from None
jupyterhub: RuntimeError: Proxy failed to start with exit code 1
```

Повторяем ошибку ручным запуском `configurable-http-proxy`:
```
[eugene@localhost ~]$ configurable-http-proxy
/usr/lib/node_modules/configurable-http-proxy/node_modules/prom-client/lib/registry.js:25
        async getMetricAsPrometheusString(metric) {
              ^^^^^^^^^^^^^^^^^^^^^^^^^^^

SyntaxError: Unexpected identifier
    at createScript (vm.js:56:10)
    at Object.runInThisContext (vm.js:97:10)
    at Module._compile (module.js:549:28)
    at Object.Module._extensions..js (module.js:586:10)
    at Module.load (module.js:494:32)
    at tryModuleLoad (module.js:453:12)
    at Function.Module._load (module.js:445:3)
    at Module.require (module.js:504:17)
    at require (internal/module.js:20:19)
    at Object.<anonymous> (/usr/lib/node_modules/configurable-http-proxy/node_modules/prom-client/index.js:8:20)
```

Узнал версию установленного пакета `configurable-http-proxy`:
```
[eugene@localhost ~]$ configurable-http-proxy -V
4.4.0
```

В [репозитории домашнего сайта пакета](https://github.com/jupyterhub/configurable-http-proxy/releases) определил предыдущую версию пакета: 4.3.2 и попробовал установить её:
```
[eugene@localhost ~]$ npm install -g configurable-http-proxy@4.3.2
/usr/bin/configurable-http-proxy -> /usr/lib/node_modules/configurable-http-proxy/bin/configurable-http-proxy
- bintrees@1.0.1 node_modules/configurable-http-proxy/node_modules/bintrees
- tdigest@0.1.1 node_modules/configurable-http-proxy/node_modules/tdigest
- prom-client@13.1.0 node_modules/configurable-http-proxy/node_modules/prom-client
/usr/lib
└─┬ configurable-http-proxy@4.3.2
  └─┬ lynx@0.2.0
    ├── mersenne@0.0.4
    └── statsd-parser@0.0.4
```

Для проверки запустил `configurable-http-proxy` и убедился в отсутствии ошибок:
```
15:49:41.655 [ConfigProxy] warn: REST API is not authenticated.
15:49:41.668 [ConfigProxy] info: Proxying http://*:8000 to (no default)
15:49:41.669 [ConfigProxy] info: Proxy API at http://localhost:8001/api/routes
^C15:49:46.740 [ConfigProxy] warn: Interrupted
```

Повторил запуск 'jupyterhub.service', на этот раз успешный:
```
[eugene@localhost ~]$ sudo systemctl start jupyterhub.service
[eugene@localhost ~]$ sudo systemctl status jupyterhub.service
● jupyterhub.service - JupyterHub
   Loaded: loaded (/opt/jupyterhub/etc/systemd/jupyterhub.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2021-07-07 16:08:58 MSK;
...
```
