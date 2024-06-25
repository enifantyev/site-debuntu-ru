---
title: "Установка JupyterHub на CentOS 7 (draft)"
date: 2021-07-07
weight: 10
description: >
  Установка JupyterHub на CentOS 7.
tags:
  - JupiterHub
  - CentOS
  - CentOS 7
---

2021-07-07

## Использованные материалы
https://github.com/jupyterhub/jupyterhub-the-hard-way/blob/HEAD/docs/installation-guide-hard.md

## 1. Установка JupyterHub и JupyterLab

### 1.1. Подготовка виртуального окружения

1.1.1. Cоздаём виртуальное окружение в каталоге `/opt/jupyterhub`:
```bash
sudo python3 -m venv /opt/jupyterhub/
```

<span style="color:red">С этого момента мы должны использовать команду *'/opt/jupyterhub/bin/python3 -m pip install'* каждый раз, когда хотим установить дополнительные пакеты в это виртуальное окружение.</span>

1.1.2. Выполняем в созданном виртуальном окружении следующие команды:
```bash
sudo /opt/jupyterhub/bin/python3 -m pip install --upgrade pip
sudo /opt/jupyterhub/bin/python3 -m pip install wheel
```

### 1.2. Установка необходимых пакетов
1.2.1. Устанавливаем jupyterhub с новым пользовательским интерфейсом jupyterlab:
```bash
sudo /opt/jupyterhub/bin/python3 -m pip install jupyterhub jupyterlab
```
<details><summary>stdout ...</summary>

```bash
...
Successfully built contextvars pandocfilters python-json-logger
Installing collected packages: zipp, typing-extensions, six, ipython-genutils, decorator, traitlets, pyrsistent, importlib-metadata, attrs, wcwidth, tornado, pyzmq, python-dateutil, pyparsing, ptyprocess, parso, nest-asyncio, jupyter-core, jsonschema, immutables, webencodings, urllib3, pygments, pycparser, prompt-toolkit, pickleshare, pexpect, packaging, nbformat, MarkupSafe, jupyter-client, jedi, idna, contextvars, chardet, certifi, backcall, async-generator, testpath, sniffio, requests, pandocfilters, nbclient, mistune, jupyterlab-pygments, jinja2, ipython, entrypoints, defusedxml, dataclasses, cffi, bleach, websocket-client, terminado, Send2Trash, ruamel.yaml.clib, requests-unixsocket, pytz, prometheus-client, nbconvert, ipykernel, greenlet, cryptography, argon2-cffi, anyio, SQLAlchemy, ruamel.yaml, python-json-logger, python-editor, pyopenssl, notebook, Mako, jupyter-server, json5, babel, pamela, oauthlib, nbclassic, jupyterlab-server, jupyter-telemetry, certipy, alembic, jupyterlab, jupyterhub
Successfully installed Mako-1.1.4 MarkupSafe-2.0.1 SQLAlchemy-1.4.20 Send2Trash-1.7.1 alembic-1.6.5 anyio-3.2.1 argon2-cffi-20.1.0 async-generator-1.10 attrs-21.2.0 babel-2.9.1 backcall-0.2.0 bleach-3.3.0 certifi-2021.5.30 certipy-0.1.3 cffi-1.14.5 chardet-4.0.0 contextvars-2.4 cryptography-3.4.7 dataclasses-0.8 decorator-5.0.9 defusedxml-0.7.1 entrypoints-0.3 greenlet-1.1.0 idna-2.10 immutables-0.15 importlib-metadata-4.6.1 ipykernel-5.5.5 ipython-7.16.1 ipython-genutils-0.2.0 jedi-0.18.0 jinja2-3.0.1 json5-0.9.6 jsonschema-3.2.0 jupyter-client-6.2.0 jupyter-core-4.7.1 jupyter-server-1.9.0 jupyter-telemetry-0.1.0 jupyterhub-1.4.1 jupyterlab-3.0.16 jupyterlab-pygments-0.1.2 jupyterlab-server-2.6.0 mistune-0.8.4 nbclassic-0.3.1 nbclient-0.5.3 nbconvert-6.0.7 nbformat-5.1.3 nest-asyncio-1.5.1 notebook-6.4.0 oauthlib-3.1.1 packaging-21.0 pamela-1.0.0 pandocfilters-1.4.3 parso-0.8.2 pexpect-4.8.0 pickleshare-0.7.5 prometheus-client-0.11.0 prompt-toolkit-3.0.19 ptyprocess-0.7.0 pycparser-2.20 pygments-2.9.0 pyopenssl-20.0.1 pyparsing-2.4.7 pyrsistent-0.18.0 python-dateutil-2.8.1 python-editor-1.0.4 python-json-logger-2.0.1 pytz-2021.1 pyzmq-22.1.0 requests-2.25.1 requests-unixsocket-0.2.0 ruamel.yaml-0.17.10 ruamel.yaml.clib-0.2.6 six-1.16.0 sniffio-1.2.0 terminado-0.10.1 testpath-0.5.0 tornado-6.1 traitlets-4.3.3 typing-extensions-3.10.0.0 urllib3-1.26.6 wcwidth-0.2.5 webencodings-0.5.1 websocket-client-1.1.0 zipp-3.5.0
```
</details><br>

1.2.2. Устанавливаем ipywidgets, also known as jupyter-widgets or simply widgets, are interactive HTML widgets for Jupyter notebooks and the IPython kernel:
```bash
$ sudo /opt/jupyterhub/bin/python3 -m pip install ipywidgets
Installing collected packages: widgetsnbextension, jupyterlab-widgets, ipywidgets
Successfully installed ipywidgets-7.6.3 jupyterlab-widgets-1.0.0 widgetsnbextension-3.5.1
```

1.2.3. JupyterHub в настоящее время также по-умолчанию требует 'configurable-http-proxy', для чего нужны nodejs и npm:
```bash
sudo yum -y install nodejs npm
```

<details><summary>stdout ...</summary>

```bash
...
Dependencies Resolved

======================================================================================================
 Package            Arch               Version                                Repository         Size
======================================================================================================
Installing:
 nodejs             x86_64             1:6.17.1-1.el7                         epel7             4.7 M
 npm                x86_64             1:3.10.10-1.6.17.1.1.el7               epel7             2.5 M
Installing for dependencies:
 libicu             x86_64             50.2-4.el7_7                           base              6.9 M
 libuv              x86_64             1:1.41.0-1.el7                         epel7             153 k

Transaction Summary
======================================================================================================
Install  2 Packages (+2 Dependent packages)

Total download size: 14 M
Installed size: 51 M
```
</details><br>

1.2.4. Устанавливаем 'configurable-http-proxy':
```bash
sudo npm install -g configurable-http-proxy
```

<details><summary>stdout ...</summary>

```bash
/usr/bin/configurable-http-proxy -> /usr/lib/node_modules/configurable-http-proxy/bin/configurable-http-proxy
/usr/lib
└─┬ configurable-http-proxy@4.4.0
  ├── commander@7.2.0
  ├─┬ http-proxy@1.18.1
  │ ├── eventemitter3@4.0.7
  │ ├── follow-redirects@1.14.1
  │ └── requires-port@1.0.0
  ├─┬ prom-client@13.1.0
  │ └─┬ tdigest@0.1.1
  │   └── bintrees@1.0.1
  ├── strftime@0.10.0
  └─┬ winston@3.3.3
    ├─┬ @dabh/diagnostics@2.0.2
    │ ├─┬ colorspace@1.1.2
    │ │ ├─┬ color@3.0.0
    │ │ │ ├─┬ color-convert@1.9.3
    │ │ │ │ └── color-name@1.1.3
    │ │ │ └─┬ color-string@1.5.5
    │ │ │   └─┬ simple-swizzle@0.2.2
    │ │ │     └── is-arrayish@0.3.2
    │ │ └── text-hex@1.0.0
    │ ├── enabled@2.0.0
    │ └── kuler@2.0.0
    ├── async@3.2.0
    ├── is-stream@2.0.0
    ├─┬ logform@2.2.0
    │ ├── colors@1.4.0
    │ ├── fast-safe-stringify@2.0.7
    │ ├── fecha@4.2.1
    │ └── ms@2.1.3
    ├─┬ one-time@1.0.0
    │ └── fn.name@1.1.0
    ├─┬ readable-stream@3.6.0
    │ ├── inherits@2.0.4
    │ ├─┬ string_decoder@1.3.0
    │ │ └── safe-buffer@5.2.1
    │ └── util-deprecate@1.0.2
    ├── stack-trace@0.0.10
    ├── triple-beam@1.3.0
    └─┬ winston-transport@4.4.0
      └─┬ readable-stream@2.3.7
        ├── core-util-is@1.0.2
        ├── isarray@1.0.0
        ├── process-nextick-args@2.0.1
        ├── safe-buffer@5.1.2
        └── string_decoder@1.1.1
```
</details><br>

### 1.3. Подготовка файлов для настройки JupyterHub
1.3.1. Для удобства разместим все необходимые файлы в каталоге с ранее созданным виртуальным окружением, точнее в подкаталоге `/opt/jupyterhub/etc/`.
```bash
sudo mkdir -p /opt/jupyterhub/etc/jupyterhub/
cd /opt/jupyterhub/etc/jupyterhub/
```

1.3.2. Создание файла конфигурации по-умолчанию:
```bash
sudo /opt/jupyterhub/bin/jupyterhub --generate-config
```

В результате выполнения этой команды был сгенерирован файл `/opt/jupyterhub/etc/jupyterhub/jupyterhub_config.py`. Все строки в этом файле закомментированы.

1.3.3. Меняем следующие строки. Позже дописать:
```bash

```

### 1.4. Добавление systemd-юнит для запуска JupyterHub
1.4.1. Создаём systemd-юнит в каталоге `/opt/jupyterhub/etc/systemd`:
```bash
sudo mkdir -p /opt/jupyterhub/etc/systemd
cat << EOF | sudo tee /opt/jupyterhub/etc/systemd/jupyterhub.service
[Unit]
Description=JupyterHub
After=syslog.target network.target

[Service]
Type=simple
User=root
Environment="PATH=/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/opt/jupyterhub/bin"
ExecStart=/opt/jupyterhub/bin/jupyterhub --ip 0.0.0.0 --port 8000  -f /opt/jupyterhub/etc/jupyterhub/jupyterhub_config.py

[Install]
WantedBy=multi-user.target
EOF
```

1.4.2. Добавляем новый systemd-юнит в обработку подсистемы systemd:
```bash
sudo ln -s /opt/jupyterhub/etc/systemd/jupyterhub.service /etc/systemd/system/jupyterhub.service

sudo systemctl daemon-reload

sudo systemctl enable jupyterhub.service
```

### 1.5. Запуск JupyterHub
1.5.1. Запускаем новый systemd-юнит:
```bash
sudo systemctl start jupyterhub.service
```

1.5.2. Проверяем статус демона:
```bash
sudo systemctl status jupyterhub.service
```

1.5.3. Возникла проблема с запуском, которую я описал здесь [Ошибка configurable-http-proxy: SyntaxError: Unexpected identifier при попытке запуска JupyterHub](n/oshibka-configurable-http-proxy-syntaxerror-unexpected-identifier-pri-popytke-zapuska-jupyterhub)

## 2. Настройка Conda
### 2.1. Установка Conda
2.1.1 На странице [Anaconda Individual Edition](https://www.anaconda.com/products/individual) находим ссылку на актуальную версию инсталлера, с помощью которой скачиваем файл и запускаем установку, принимаем лицензию, <span style="color:red">указываем каталог установки `/opt/conda`</span>:
```bash
curl -LO https://repo.anaconda.com/archive/Anaconda3-2021.05-Linux-x86_64.sh
sudo /bin/bash Anaconda3-2021.05-Linux-x86_64.sh
...
Do you accept the license terms? [yes|no]
[no] >>> yes

Anaconda3 will now be installed into this location:
/root/anaconda3

  - Press ENTER to confirm the location
  - Press CTRL-C to abort the installation
  - Or specify a different location below

[/root/anaconda3] >>> /opt/conda
PREFIX=/opt/conda
Unpacking payload ...
Collecting package metadata (current_repodata.json): done
Solving environment: done
```

На запрос инициализации Anaconda отвечаем утвердительно:
```bash
Do you wish the installer to initialize Anaconda3
by running conda init? [yes|no]
[no] >>> yes
```

<details><summary>stdout ...</summary>

```bash
no change     /opt/conda/condabin/conda
no change     /opt/conda/bin/conda
no change     /opt/conda/bin/conda-env
no change     /opt/conda/bin/activate
no change     /opt/conda/bin/deactivate
no change     /opt/conda/etc/profile.d/conda.sh
no change     /opt/conda/etc/fish/conf.d/conda.fish
no change     /opt/conda/shell/condabin/Conda.psm1
no change     /opt/conda/shell/condabin/conda-hook.ps1
no change     /opt/conda/lib/python3.8/site-packages/xontrib/conda.xsh
no change     /opt/conda/etc/profile.d/conda.csh
modified      /root/.bashrc

==> For changes to take effect, close and re-open your current shell. <==

If you'd prefer that conda's base environment not be activated on startup,
   set the auto_activate_base parameter to false:

conda config --set auto_activate_base false

Thank you for installing Anaconda3!

===========================================================================

Working with Python and Jupyter notebooks is a breeze with PyCharm Pro,
designed to be used with Anaconda. Download now and have the best data
tools at your fingertips.

PyCharm Pro for Anaconda is available at: https://www.anaconda.com/pycharm
```

</details><br>

Информацию об установленной Anaconda смотрим командой:
```bash
$ /opt/conda/bin/conda info -a
```

2.1.2. Делаем conda доступной для всех пользователей:
```bash
sudo ln -s /opt/conda/etc/profile.d/conda.sh /etc/profile.d/conda.sh
```

## 2.2. Установка окружения conda по умолчанию для всех пользователей
```bash
$ sudo mkdir /opt/conda/envs/
```

```bash
$ sudo /opt/conda/bin/conda create --prefix /opt/conda/envs/jupyter_env python=3.6 ipykernel
...
jupyter_core-4.7.1   | 68 KB     | ########################################################################################## | 100%
Preparing transaction: done
Verifying transaction: done
Executing transaction: done
#
# To activate this environment, use
#
#     $ conda activate /opt/conda/envs/jupyter_env
#
# To deactivate an active environment, use
#
#     $ conda deactivat
```

Посмотреть conda-среды:
```bash
$ conda info --envs

# conda environments:
#
base                  *  /opt/conda
jupyter_env              /opt/conda/envs/jupyter_env
```

```bash
[eugene@localhost /]$ sudo su

(base) [root@localhost ~]#source activate jupyter_env

(jupyter_env) [root@localhost ~]# /opt/conda/envs/jupyter_env/bin/python -m ipykernel install --prefix=/usr/local --name 'python36-spark-conda' --display-name "python36-spark-conda"
Installed kernelspec python36-spark-conda in /usr/local/share/jupyter/kernels/python36-spark-conda
```

```bash
(jupyter_env) [root@localhost ~]# cat /usr/local/share/jupyter/kernels/python36-spark-conda/kernel.json
{
 "argv": [
  "/opt/conda/envs/jupyter_env/bin/python",
  "-m",
  "ipykernel_launcher",
  "-f",
  "{connection_file}"
 ],
 "display_name": "python36-spark-conda",
 "language": "python"
}(jupyter_env)
```

```bash
(jupyter_env) [root@localhost ~]# conda install -c conda-forge jupyter_contrib_nbextensions

[I 12:51:08 InstallContribNbextensionsApp] Up to date: /opt/conda/envs/jupyter_env/share/jupyter/nbextensions/latex_envs/doc/latex_env_doc_files/latex_env_doc_52_0.png
[I 12:51:08 InstallContribNbextensionsApp] - Validating: OK
[I 12:51:08 InstallContribNbextensionsApp] Installing jupyter_contrib_nbextensions items to config in /opt/conda/envs/jupyter_env/etc/jupyter
Enabling: jupyter_nbextensions_configurator
- Writing config: /opt/conda/envs/jupyter_env/etc/jupyter
    - Validating...
      jupyter_nbextensions_configurator 0.4.1 OK
Enabling notebook nbextension nbextensions_configurator/config_menu/main...
Enabling tree nbextension nbextensions_configurator/tree_tab/main...
[I 12:51:08 InstallContribNbextensionsApp] Enabling notebook extension contrib_nbextensions_help_item/main...
[I 12:51:08 InstallContribNbextensionsApp]       - Validating: OK
[I 12:51:08 InstallContribNbextensionsApp] - Editing config: /opt/conda/envs/jupyter_env/etc/jupyter/jupyter_nbconvert_config.json
[I 12:51:08 InstallContribNbextensionsApp] --  Configuring nbconvert template path
[I 12:51:08 InstallContribNbextensionsApp] --  Configuring nbconvert preprocessors
[I 12:51:08 InstallContribNbextensionsApp] - Writing config: /opt/conda/envs/jupyter_env/etc/jupyter/jupyter_nbconvert_config.json
[I 12:51:08 InstallContribNbextensionsApp] --  Writing updated config file /opt/conda/envs/jupyter_env/etc/jupyter/jupyter_nbconvert_config.json
done
```

```bash
(jupyter_env) [root@localhost ~]# conda install -c conda-forge jupyter-resource-usage
...
The following NEW packages will be INSTALLED:

  anyio              conda-forge/linux-64::anyio-3.2.1-py36h5fab9bb_0
  async_generator    conda-forge/noarch::async_generator-1.10-py_0
  brotlipy           conda-forge/linux-64::brotlipy-0.7.0-py36h8f6f2f9_1001
  chardet            conda-forge/linux-64::chardet-4.0.0-py36h5fab9bb_1
  contextvars        conda-forge/noarch::contextvars-2.4-py_0
  cryptography       conda-forge/linux-64::cryptography-3.4.7-py36hb60f036_0
  dataclasses        conda-forge/noarch::dataclasses-0.8-pyh787bdff_0
  idna               conda-forge/noarch::idna-2.10-pyh9f0ad1d_0
  immutables         conda-forge/linux-64::immutables-0.15-py36h8f6f2f9_0
  jupyter-resource-~ conda-forge/noarch::jupyter-resource-usage-0.6.0-pyhd8ed1ab_0
  jupyter_server     conda-forge/noarch::jupyter_server-1.9.0-pyhd8ed1ab_0
  psutil             conda-forge/linux-64::psutil-5.8.0-py36h8f6f2f9_1
  pyopenssl          conda-forge/noarch::pyopenssl-20.0.1-pyhd8ed1ab_0
  pysocks            conda-forge/linux-64::pysocks-1.7.1-py36h5fab9bb_3
  requests           conda-forge/noarch::requests-2.25.1-pyhd3deb0d_0
  requests-unixsock~ conda-forge/noarch::requests-unixsocket-0.2.0-py_0
  sniffio            conda-forge/linux-64::sniffio-1.2.0-py36h5fab9bb_1
  urllib3            conda-forge/noarch::urllib3-1.26.6-pyhd8ed1ab_0
  websocket-client   conda-forge/linux-64::websocket-client-0.57.0-py36h5fab9bb_4
...
Preparing transaction: done
Verifying transaction: done
Executing transaction: done
```

```bash
(jupyter_env) [root@localhost ~]# conda install -c conda-forge jupyterlab
...
The following NEW packages will be INSTALLED:

  babel              conda-forge/noarch::babel-2.9.1-pyh44b312d_0
  json5              conda-forge/noarch::json5-0.9.5-pyh9f0ad1d_0
  jupyterlab         conda-forge/noarch::jupyterlab-3.0.16-pyhd8ed1ab_0
  jupyterlab_server  conda-forge/noarch::jupyterlab_server-2.6.0-pyhd8ed1ab_0
  nbclassic          conda-forge/noarch::nbclassic-0.3.1-pyhd8ed1ab_1
  pytz               conda-forge/noarch::pytz-2021.1-pyhd8ed1ab_0
...
Preparing transaction: done
Verifying transaction: done
Executing transaction: done
```

```bash
(jupyter_env) [root@localhost ~]# conda install -c conda-forge jupyterlab-git
...
The following NEW packages will be INSTALLED:

  colorama           conda-forge/noarch::colorama-0.4.4-pyh9f0ad1d_0
  gitdb              conda-forge/noarch::gitdb-4.0.7-pyhd8ed1ab_0
  gitpython          conda-forge/noarch::gitpython-3.1.18-pyhd8ed1ab_0
  jupyter-server-ma~ conda-forge/noarch::jupyter-server-mathjax-0.2.3-pyhd8ed1ab_0
  jupyterlab-git     conda-forge/noarch::jupyterlab-git-0.30.1-pyhd8ed1ab_0
  nbdime             conda-forge/noarch::nbdime-3.1.0-pyhd8ed1ab_0
  smmap              conda-forge/noarch::smmap-3.0.5-pyh44b312d_0
...
Preparing transaction: done
Verifying transaction: done
Executing transaction: done
```

```bash
(jupyter_env) [root@localhost ~]# conda install -c conda-forge jupyterthemes
...
The following NEW packages will be INSTALLED:

  cycler             conda-forge/noarch::cycler-0.10.0-py_2
  freetype           conda-forge/linux-64::freetype-2.10.4-h0708190_1
  jbig               conda-forge/linux-64::jbig-2.1-h7f98852_2003
  jpeg               conda-forge/linux-64::jpeg-9d-h36c2ea0_0
  jupyterthemes      conda-forge/noarch::jupyterthemes-0.20.0-py_1
  kiwisolver         conda-forge/linux-64::kiwisolver-1.3.1-py36h605e78d_1
  lcms2              conda-forge/linux-64::lcms2-2.12-hddcbb42_0
  lerc               conda-forge/linux-64::lerc-2.2.1-h9c3ff4c_0
  lesscpy            conda-forge/noarch::lesscpy-0.14.0-pyhd8ed1ab_0
  libblas            conda-forge/linux-64::libblas-3.9.0-9_openblas
  libcblas           conda-forge/linux-64::libcblas-3.9.0-9_openblas
  libdeflate         conda-forge/linux-64::libdeflate-1.7-h7f98852_5
  libgfortran-ng     conda-forge/linux-64::libgfortran-ng-9.3.0-hff62375_19
  libgfortran5       conda-forge/linux-64::libgfortran5-9.3.0-hff62375_19
  liblapack          conda-forge/linux-64::liblapack-3.9.0-9_openblas
  libopenblas        conda-forge/linux-64::libopenblas-0.3.15-pthreads_h8fe5266_1
  libpng             conda-forge/linux-64::libpng-1.6.37-h21135ba_2
  libtiff            conda-forge/linux-64::libtiff-4.3.0-hf544144_1
  libwebp-base       conda-forge/linux-64::libwebp-base-1.2.0-h7f98852_2
  lz4-c              conda-forge/linux-64::lz4-c-1.9.3-h9c3ff4c_0
  matplotlib-base    conda-forge/linux-64::matplotlib-base-3.3.4-py36hd391965_0
  numpy              conda-forge/linux-64::numpy-1.19.5-py36h2aa4a07_1
  olefile            conda-forge/noarch::olefile-0.46-pyh9f0ad1d_1
  openjpeg           conda-forge/linux-64::openjpeg-2.4.0-hb52868f_1
  pillow             conda-forge/linux-64::pillow-8.3.1-py36h676a545_0
  ply                conda-forge/noarch::ply-3.11-py_1
  zstd               conda-forge/linux-64::zstd-1.5.0-ha95c52a_0
...
Preparing transaction: done
Verifying transaction: done
Executing transaction: done
```

```bash
(jupyter_env) [root@localhost ~]# conda install -c conda-forge nbresuse
...
The following NEW packages will be INSTALLED:

  nbresuse           conda-forge/noarch::nbresuse-0.4.0-pyhd8ed1ab_0
...
Preparing transaction: done
Verifying transaction: done
Executing transaction: done
```

## 3. Настройка reverse proxy
### 3.1. Установка nginx
```bash
yum -y install nginx
```

```bash
cat << EOF | sudo tee /etc/nginx/conf.d/jupyter_proxy.conf
map \$http_upgrade \$connection_upgrade {
    default upgrade;
    ''      close;
}

server {
    listen 80;
    server_name ${HOSTNAME};
    return 301 https://\$host\$request_uri;
}

server {
    listen 8888 ssl;

    server_name dev-jupyter.example.org;

    # certs sent to the client in SERVER HELLO are concatenated in ssl_certificate
    ssl_certificate /opt/cloudera/security/pki/HTTP_${HOSTNAME}.crt;
    ssl_certificate_key /opt/cloudera/security/pki/HTTP_${HOSTNAME}.key;
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:50m;
    ssl_session_tickets off;



    # intermediate configuration, tweak to your needs
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers 'ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES256-SHA:ECDHE-ECDSA-DES-CBC3-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:DES-CBC3-SHA:!DSS';
    ssl_prefer_server_ciphers on;

    ## verify chain of trust of OCSP response using Root CA and Intermediate certs
    ssl_trusted_certificate /etc/ipa/ca.crt;

    # resolver <IP DNS resolver> valid=300s;
    # resolver_timeout 5s;
    resolver 10.206.226.129 10.206.226.130 valid=30s ipv6=off;
    resolver_timeout 5s;


  location / {    
    # NOTE important to also set base url of jupyterhub to /jupyter in its config
    proxy_pass http://127.0.0.1:8000;

    proxy_set_header X-Real-IP \$remote_addr;
    proxy_set_header Host \$host:\$server_port;
    proxy_set_header X-Scheme \$scheme;
    proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto \$scheme;
    #set \$ref "";
    #access_by_lua_block {
    #  if (ngx.var.http_referer ~= nil and ngx.var.http_referer ~= "") then
    #   ngx.var.ref = string.gsub(ngx.var.http_referer, "https", "http")
    #  end
    # }

    #proxy_set_header Referer \$ref;
    #proxy_set_header  Referer  http://\$host;
    add_header 'Access-Control-Allow-Origin' "\$http_origin";

    # websocket headers
    proxy_http_version 1.1;
    proxy_set_header Upgrade \$http_upgrade;
    proxy_set_header Connection \$connection_upgrade;


    proxy_buffering off;
  }

}
EOF

systemctl enable --now nginx
```
