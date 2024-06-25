---
title: "Установка Apache Airflow 2 + FreeIPA + Apache Hadoop + сценарий удаления"
date: 2021-02-23
weight: 10
description: >
  Описание установки на Cloudera Hadoop узел экземпляра Airflow второй версии с назначением ролей из FreeIPA.
tags:
  - Apache Airflow
  - FreeIPA
  - Apache Hadoop
  - CentOS
  - CentOS 7
aliases:
  - /a/ustanovka-apache-airflow-2-freeipa-apache-hadoop-scenarii-udaleniya/
---

## Использованные материалы
- [Apache Airflow Documentation](https://airflow.apache.org/docs/apache-airflow/stable/)
- [Quick Start](https://airflow.apache.org/docs/apache-airflow/stable/start/index.html)
- Предыдущий опыт настройки Airflow прошлых версий.

## Заметки
- В шагах инструкции пока отсутствует. Важно опроовать.
После скачивания свежего файла с зависимостями дл apache-airflow, необходимо изменить в этом файле строку 'Flask-AppBuilder==3.1.1' на 'Flask-AppBuilder==3.2.0', чтобы заработал мэппинг между пользовательскими группами в IPA и ролями в Airflow.
Для уже установленных экземпляров Airflow, можно изменить соответствующую строку в файле '/usr/local/lib/python3.6/site-packages/apache_airflow-2.0.1.dist-info/METADATA'.
- Если пользовательские DAG'и запускаются через 'run_as_user', то необходимо обеспечить чтение файла `/etc/airflow/airflow.cfg` этими пользователями, иначе DAG'и не будут выполнены с ошибкой 'airflow.exceptions.AirflowConfigException: error: cannot use sqlite version < 3.15.0'
- Если в DAG'е указать 'run_as_user': 'eugene', то Airflow запускает процесс от имени пользователя командой `sudo -E -H -u eugene`, в результате которой процесс пользователя наследует переменные родительского процесса, а переменная HOME меняется на пользовательскую директорию в /home.
- Надо дать права на подпапки в рабочей папке airflow для группы пользователей prod_airflow_airf1, так как запускаемый дочерний run_as процесс от имени пользователя, должен иметь право на запись в рабочую папку Airflow.
- Установку python-пакетов производить под учётной записью пользователя. Лишь пакеты airflow должны быть установлены под системной учётной записью.
- Разграничение DAG'ов между пользователями задаётся внутри DAG'а строкой: 'owner': 'eugene'. Это не работает во второй версии Airflow?
- В /etc/pip.conf необходимо добавить python-репо в Nexus'е с пользовательскими библиотеками, чтобы пользователь мог вызвать pip install прямо из своего кода, без дополнительных операций?
- Для сборки некоторых python-пакетов (найти где я это делал), pydoop, и ещё один пакет, ...., требуется установить компилятор gcc-c++, полный пакет явы с инструментами разработчика.

## Создание необходимых групп во FreeIPA для аутентификации в Airflow
Во второй версии Airflow изменился способ аутентификации и авторизации пользователей. Теперь из коробки активирован режим RBAC, то есть аутентификация и авторизация выполняется силами Flask App Builder (FAB).

Для аутентификации создаём одну POSIX-группу 'prod_airflow_airf1', участники которой, будут иметь доступ к Web UI интерфейсу Airflow, а также эта группа будет особым образом назначена на каталог /data/airflow для того, чтобы участники имели доступ к своим DAGам и логам в подпапках этого каталога, но не имели доступа к конфигурационным файлам Airflow.

Управление авторизацией будет происходить через роли в Airflow отображённые во вложенные группы IPA. По умолчанию, новому пользователю назначается роль 'Viewer' в дополнение к роли, полученной из соответствующей группы IPA.

## Установка ПО
Предполагается, что на машине уже установлены пакеты 'python3' и 'python3-pip'.

### Установка PostgreSQL
Для работы Airflow в prod-режиме рекомендуется использовать PostgreSQL, вместо используемого по умолчанию SQLite. Ничего сложного в этом этапе нет.

Добавляем в /etc/yum.repos.d/ подготовленный файл из статьи по Nexus'у pub_postgresql_yum:
```bash
USERNAME="nx-pubrepo-reader"
PASSWORD="xxxxxxxxxxx"

cat << EOF | tee -a /etc/yum.repos.d/pgdg-redhat-all.repo
#######################################################
# PGDG Red Hat Enterprise Linux / CentOS repositories #
#######################################################

# PGDG Red Hat Enterprise Linux / CentOS stable common repository for all PostgreSQL versions

[pgdg-common]
username=$USERNAME
password=$PASSWORD
name=PostgreSQL common RPMs for RHEL/CentOS \$releasever - \$basearch
baseurl=http://nexus.example.org:8081/repository/all_postgresql_yum/common/redhat/rhel-\$releasever-\$basearch
enabled=1
gpgcheck=0
gpgkey=http://nexus.example.org:8081/repository/all_postgresql_yum/RPM-GPG-KEY-PGDG-AARCH64

# PGDG Red Hat Enterprise Linux / CentOS stable repositories:

[pgdg13]
username=$USERNAME
password=$PASSWORD
name=PostgreSQL 13 for RHEL/CentOS \$releasever - \$basearch
baseurl=http://nexus.example.org:8081/repository/all_postgresql_yum/13/redhat/rhel-\$releasever-\$basearch
enabled=1
gpgcheck=0
gpgkey=http://nexus.example.org:8081/repository/all_postgresql_yum/RPM-GPG-KEY-PGDG-AARCH64
EOF
```

Устанавливаем актуальную версию PosgreSQL:
```bash
sudo yum update

# Install PostgreSQL:
sudo yum -y install postgresql13-server

# Optionally initialize the database and enable automatic start:
sudo /usr/pgsql-13/bin/postgresql-13-setup initdb
sudo systemctl enable --now postgresql-13
```

### Установка дополнительных пакетов
Устанавливаем пакеты для разработки:
```bash
sudo yum -y groupinstall "Development Tools"
sudo yum -y install python3-devel cyrus-sasl cyrus-sasl-devel libffi-devel krb5-devel
```

Для работы LDAP-аутентификации в FAB требуется установка пакета python-ldap, который не соберётся, если не установить:
```bash
sudo yum -y install openldap-devel python-devel
```

## Создание системного локального пользователя airflow
Создаём системного пользователя airflow и разрешаем ему беспарольное sudo для имперсонации, то есть запуска скриптов от имени владельца:
```bash
sudo useradd -r airflow -s /sbin/nologin
echo "airflow ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/airflow
sudo chmod 440 /etc/sudoers.d/airflow
```

## Подготовка к запуску Airflow
Необходимо ввести машину в Cloudera-кластер, иначе каталог '/etc/hadoop' и важные переменные будут отсутствовать на машине.

### Создание каталогов и конфигурационных файлов для работы Airflow
Создаём в IPA POSIX-группу для аутентификации в Airflow и, одновременно, для доступа пользователей к каталогам dags,logs,plugins. И сразу создадим вложенные Non-POSIX группы для авторизации в Airflow. Чуть позже настроим их отображение на локальные Airflow-роли. Важно поменять переменные перед использованием этого скрипта:
```bash
# Если требуется kerberos-тикет, то  не забываем:
# kinit

# Номер Jira-задачи
JIRA="SDK-2198"

# Название ИС
IS="prod"
#IS="dev"
#IS="devdmz"

# Название сервиса:
SERVICE="airflow"

# Псевдоним экземпляра Airflow
ALIAS="airf1"

# Группа администраторов в IPA
ADMINSGROUP="admins"

# Составляем название группы:
GROUPNAME="${IS}_${SERVICE}_${ALIAS}"

ipa group-add ${GROUPNAME} --desc="Пользователи имеющие право для аутентификации в ${ALIAS}. Авторизация настраивается через вложенные группы. Это POSIX-группа, потому что она же назначена на каталоги {dags,logs,plugins} для доступа юзеров к их дагам и логам. ${JIRA}."
ipa group-add ${GROUPNAME}_admins --desc="Группа отображается в роль Admin сервиса $SERVICE на ${ALIAS}. ${JIRA}." --nonposix
ipa group-add ${GROUPNAME}_ops --desc="Группа отображается в роль Op сервиса $SERVICE на ${ALIAS}. ${JIRA}." --nonposix
ipa group-add ${GROUPNAME}_users --desc="Группа отображается в роль User сервиса $SERVICE на ${ALIAS}. ${JIRA}." --nonposix
ipa group-add ${GROUPNAME}_viewers --desc="Группа отображается в роль Viewer сервиса $SERVICE на ${ALIAS}. ${JIRA}." --nonposix

ipa group-add-member ${GROUPNAME} --groups=${GROUPNAME}_admins --groups=${GROUPNAME}_ops --groups=${GROUPNAME}_users --groups=${GROUPNAME}_viewers

ipa group-add-member ${GROUPNAME}_admins --groups=${ADMINSGROUP}
```

Сейчас же создаём каталог для работы Apache Airflow и назначаем права:
```bash
# POSIX-группа предназачена для доступа пользователей к нужным папкам.
export POSIXGROUP=${GROUPNAME}

export DIR="/data/airflow"

sudo mkdir -p $DIR/{dags,logs,plugins}
sudo chmod -R 750 $DIR
sudo chmod 2750 $DIR/{dags,plugins}
sudo chown -R airflow.airflow $DIR
sudo setfacl -m d:g:${POSIXGROUP}:rwx $DIR/{dags,logs,plugins}
```
  - В последней команде мы назначаем default-права на каталог общей POSIX-группе prod_airflow_airf1 из FreeIPA, чтобы все наследуемые пользователи из вложенных групп имели доступ к папкам dags, logs и plugins. Эти права будут наследоваться с режимом 770 для поддиректорий и 660 для файлов.
  - Заметка: POSIX ACL не работает с MemberOf-группами из FreeIPA, но работает с  POSIX-группами.

Создаём файл `/etc/sysconfig/airflow` с переменными, использующимися airflow при запуске из systemd-юнитов:
```bash
FILE="/etc/sysconfig/airflow"
echo "AIRFLOW_HOME=/data/airflow" | sudo tee $FILE
echo "AIRFLOW_CONFIG=/etc/airflow/airflow.cfg" | sudo tee -a $FILE
echo "HADOOP_CONF_DIR=/etc/hadoop/conf" | sudo tee -a $FILE
echo "HIVE_CONF_DIR=/etc/hive/conf" | sudo tee -a $FILE
```
  - Без переменных 'HADOOP_CONF_DIR' и 'HIVE_CONF_DIR', в Airflow не будет работать SparkSubmitOperator, так как он использует файлы `hive_site.xml` и `yarn-site.xml`.

Создаём каталог `/etc/airflow` для хранения `airflow.cfg`, сертификатов, keytab'ов и прочих подобных файлов:
```bash
DIR="/etc/airflow"
sudo mkdir $DIR
sudo chmod 2770 $DIR
sudo chown airflow.airflow $DIR
```

### Создание systemd-юнитов
Взято из https://github.com/apache/airflow/tree/master/scripts/systemd. При появлении других зависимостей для запуска сервисов Airflow, добавляем их в строки After и Wants. Примеры можно посмотреть в https://github.com/apache/airflow/tree/master/scripts/systemd.
```bash
cat << EOF | sudo tee /etc/systemd/system/airflow-scheduler.service
[Unit]
Description=Airflow scheduler daemon
After=network.target postgresql-13.service
Wants=postgresql-13.service

[Service]
EnvironmentFile=/etc/sysconfig/airflow
User=airflow
Group=airflow
Type=simple
ExecStart=/usr/local/bin/airflow scheduler
Restart=always
RestartSec=5s
SyslogIdentifier=airf-sched

[Install]
WantedBy=multi-user.target
EOF
```

```bash
cat << EOF | sudo tee /etc/systemd/system/airflow-webserver.service
[Unit]
Description=Airflow webserver daemon
After=network.target postgresql-13.service
Wants=postgresql-13.service

[Service]
EnvironmentFile=/etc/sysconfig/airflow
RuntimeDirectory=airflow
RuntimeDirectoryMode=0775
User=airflow
Group=airflow
Type=simple
ExecStart=/usr/local/bin/airflow webserver -p 8080 --pid /run/airflow/airflow-webserver.pid
Restart=on-failure
RestartSec=5s
PrivateTmp=true
SyslogIdentifier=airf-web

[Install]
WantedBy=multi-user.target
EOF
```

### Настройка логгирования
Настраиваем сразу с заделом на включение Kerberos (airf-krb), каковое включение описано ниже и производится по желанию, потому что пока не понятно работоспособен ли Kerberos с Airflow на Python 3:
```bash
sudo mkdir /var/log/airflow
sudo chmod 750 /var/log/airflow

echo 'if $programname == "airf-sched" then /var/log/airflow/airf-sched.log
& stop' | sudo tee /etc/rsyslog.d/airf-sched.conf
echo 'if $programname == "airf-web" then /var/log/airflow/airf-web.log
& stop' | sudo tee /etc/rsyslog.d/airf-web.conf
echo 'if $programname == "airf-krb" then /var/log/airflow/airf-krb.log
& stop' | sudo tee /etc/rsyslog.d/airf-krb.conf
sudo systemctl restart rsyslog

sudo sed -i '1i /var/log/airflow/airf-sched.log\n/var/log/airflow/airf-web.log\n/var/log/airflow/airf-krb.log' /etc/logrotate.d/syslog
```

### Создание базы данных airflow в PostgreSQL
Далее приведены команды в psql для создания базы данных:
```bash
DBNAME="airf_metadata"
DBUSER="airf_user"
AIRFPASS=$(cat /dev/urandom | tr -dc A-Za-z0-9 | head -c15)

sudo -u postgres psql -c "CREATE DATABASE ${DBNAME};"
sudo -u postgres psql -c "CREATE USER ${DBUSER} WITH password '${AIRFPASS}';"
sudo -u postgres psql -c "GRANT ALL PRIVILEGES ON DATABASE ${DBNAME} TO ${DBUSER};"
echo "${DBUSER}:${AIRFPASS}" | sudo tee /etc/airflow/psql-pass.txt
echo "${DBNAME}" | sudo tee /etc/airflow/psql-db.txt
sudo chmod 400 /etc/airflow/psql-pass.txt /etc/airflow/psql-db.txt
```

## Создание сертификата для работы HTTPS
Подготавливаем ключ и сертификат для Web UI Airflow. Чтобы было удобней, выбираем короткое имя для URL: 'prod-airf1.example.org'.

Если требуется, то сначала получаем kerberos-билет:
```bash
kinit
```

После чего получаем сертификат, который будет действителен для двух dns-имени, prod-airf01p.example.org и airf1.example.org. Важно поменять переменные перед использованием этого скрипта:
```bash
# Меняем на требуемое короткий псевдоним:
export ALIAS="prod-airf1"
#export ALIAS="dev-airf1"
#export ALIAS="devdmz-airf1"

export SHORTHOSTNAME=$(hostname -s)
export DOMAIN=$(hostname -d)
export ALIASHOSTNAME="${ALIAS}.${DOMAIN}"
export DOMAINCAPS=$(echo $(hostname -d)|tr [:lower:] [:upper:])

# Добавляем CNAME-запись для текущей машины в DNS-зону...
ipa dnsrecord-add $DOMAIN $ALIAS --cname-rec="${HOSTNAME}."

# Добавляем сервис-принципал для текущей машины...
ipa service-add HTTP/${HOSTNAME}@${DOMAINCAPS}

# Добавляем alias для созданного ранее сервис-принципала...
ipa service-add-principal HTTP/${HOSTNAME} HTTP/${ALIASHOSTNAME}

# С помощью локального certmonger'а генерируруем приватный ключ и
# сертификат к нему, чтобы airflow web ui смог работать через https.
# Certmonger будет следить за сертификатом и за 28 дней до окончания срока действия,
# автоматически подаст запрос к FreeIPA на новый сертификат.
# Заметим, что сертификат во FreeIPA прикреплен к сервис-принципалу, а не к хосту.
sudo ipa-getcert request -K HTTP/${HOSTNAME} \
-f /etc/airflow/${ALIAS}_${SHORTHOSTNAME}.crt \
-k /etc/airflow/${ALIAS}_${SHORTHOSTNAME}.key \
-D ${ALIASHOSTNAME} \
-C "/bin/bash -c '/usr/bin/systemctl restart airflow-webserver && /bin/chmod 640 /etc/airflow/${ALIAS}_${SHORTHOSTNAME}.{crt,key}'"

# Выставляем права на файлы ключа и сертификата...
# sudo chmod 640 /etc/airflow/${ALIAS}_${SHORTHOSTNAME}.{crt,key}
```
Если понадобится, то отслеживаемые сертификаты выводим командой:
```bash
# Выводим список отслеживаемых сертификатов...
$ sudo getcert list
  Number of certificates and requests being tracked: 1.
  Request ID '20210129140146':
        status: MONITORING
        stuck: no
        key pair storage: type=FILE,location='/etc/airflow/airf1_prod-airf01p.key'
        certificate: type=FILE,location='/etc/airflow/airf1_prod-airf01p.crt'
        CA: IPA
        issuer: CN=Certificate Authority,O=example.org
        subject: CN=prod-airf01p.example.org,O=example.org
        expires: 2023-01-30 14:01:46 UTC
        dns: prod-airf1.example.org,prod-airf01p.example.org
        principal name: HTTP/prod-airf01p.example.org@example.org
        key usage: digitalSignature,nonRepudiation,keyEncipherment,dataEncipherment
        eku: id-kp-serverAuth,id-kp-clientAuth
        pre-save command:
        post-save command:
        track: yes
        auto-renew: yes
```

Если понадобится снять сертификат с отслеживания, то выполняем 'getcert list', выбираем сертификат и даём команду снятия с отслеживания:
```bash
$ sudo ipa-getcert stop-tracking -i 20210129140146
```
Если нужно вновь поставить сертификат на автообновление, то выполняем, например:
```bash
$ sudo ipa-getcert start-tracking -f /etc/airflow/prod-airf1_prod-airf01p.crt -k /etc/airflow/prod-airf1_prod-airf01p.key
```

## Установка Airflow-пакетов
### Как правильно удалить все установленные пакеты
Установку Airflow производим под root для, что сделает airflow доступным под любым аккаунтом, так как исполняемые файлы Airflow будут устанавливаться в `/usr/local/bin`.

На свежей машине установлены только два python-пакета:
```bash
$ sudo pip list
Package    Version
---------- -------
pip        21.0
setuptools 39.2.0
```

Если необходимо удалить все python-пакеты, то сначала создаём файл со списком установленных пакетов:
```bash
$ sudo pip freeze > requirements.txt
```
После чего удаляем все пакеты, используя их список из ранее созданного файла:
```bash
$ sudo pip uninstall -y -r requirements.txt
```

## Установка
Так как на целевой машине используется python 3.6, то перед установкой airflow скачиваем свежий ограничительный файл, это [constraints-3.6.txt для Airflow 2.0.1](https://raw.githubusercontent.com/apache/airflow/constraints-2.0.1/constraints-3.6.txt), и переносим на машину.

Так как версия установленного pip > 20.2.4, то устанавливаем Airflow указывая опцию `--use-deprecated legacy-resolver` и актуальный файл `constraints.txt`:
```bash
# Обновляем pip
$ sudo pip3 install --upgrade pip

# Устанавливаем wheel
$ sudo pip3 install wheel

$ sudo pip3 install apache-airflow==2.0.1 --use-deprecated legacy-resolver --constraint constraints-3.6.txt
Looking in indexes: http://centos-main-reader:****@nexus.example.org:8081/repository/pypi/simple
Collecting apache-airflow==2.0.1
  Downloading http://nexus.example.org:8081/repository/pypi/packages/apache-airflow/2.0.1/apache_airflow-2.0.1-py3-none-any.whl (4.5 MB)
     |████████████████████████████████| 4.5 MB 61.9 MB/s
Collecting cached-property==1.5.2
  Downloading http://nexus.example.org:8081/repository/pypi/packages/cached-property/1.5.2/cached_property-1.5.2-py2.py3-none-any.whl (7.6 kB)
Collecting iso8601==0.1.14
  Downloading http://nexus.example.org:8081/repository/pypi/packages/iso8601/0.1.14/iso8601-0.1.14-py2.py3-none-any.whl (9.5 kB)
Collecting psutil==5.8.0
  Downloading http://nexus.example.org:8081/repository/pypi/packages/psutil/5.8.0/psutil-5.8.0-cp36-cp36m-manylinux2010_x86_64.whl (291 kB)
     |████████████████████████████████| 291 kB 75.9 MB/s
Collecting pendulum==2.1.2
...
Successfully installed Babel-2.9.0 Flask-1.1.2 Flask-AppBuilder-3.1.1 Flask-Babel-1.0.0 Flask-Caching-1.9.0 Flask-JWT-Extended-3.25.1 Flask-Login-0.4.1 Flask-OpenID-1.2.5 Flask-SQLAlchemy-2.4.4 Flask-WTF-0.14.3 Jinja2-2.11.3 Mako-1.1.4 Markdown-3.3.3 MarkupSafe-1.1.1 PyJWT-1.7.1 PyYAML-5.4.1 Pygments-2.8.0 SQLAlchemy-1.3.23 SQLAlchemy-JSONField-1.0.0 SQLAlchemy-Utils-0.36.8 WTForms-2.3.3 Werkzeug-1.0.1 alembic-1.5.4 apache-airflow-2.0.1 apache-airflow-providers-ftp-1.0.1 apache-airflow-providers-http-1.1.0 apache-airflow-providers-imap-1.0.1 apache-airflow-providers-sqlite-1.0.1 apispec-3.3.2 argcomplete-1.12.2 attrs-20.3.0 cached-property-1.5.2 cattrs-1.0.0 certifi-2020.12.5 cffi-1.14.5 chardet-3.0.4 click-7.1.2 clickclick-20.10.2 colorama-0.4.4 colorlog-4.7.2 commonmark-0.9.1 connexion-2.7.0 croniter-0.3.37 cryptography-3.4.5 dataclasses-0.7 defusedxml-0.6.0 dill-0.3.3 dnspython-1.16.0 docutils-0.16 email-validator-1.1.2 graphviz-0.16 gunicorn-19.10.0 idna-2.10 importlib-metadata-1.7.0 importlib-resources-1.5.0 inflection-0.5.1 iso8601-0.1.14 itsdangerous-1.1.0 jsonschema-3.2.0 lazy-object-proxy-1.4.3 lockfile-0.12.2 marshmallow-3.10.0 marshmallow-enum-1.5.1 marshmallow-oneofschema-2.1.0 marshmallow-sqlalchemy-0.23.1 natsort-7.1.1 numpy-1.19.5 openapi-spec-validator-0.2.9 pandas-1.1.5 pendulum-2.1.2 pep562-1.0 prison-0.1.3 psutil-5.8.0 pycparser-2.20 pyrsistent-0.17.3 python-daemon-2.2.4 python-dateutil-2.8.1 python-editor-1.0.4 python-nvd3-0.15.0 python-slugify-4.0.1 python3-openid-3.2.0 pytz-2020.5 pytzdata-2020.1 requests-2.25.1 rich-9.2.0 setproctitle-1.2.2 six-1.15.0 swagger-ui-bundle-0.0.8 tabulate-0.8.7 tenacity-6.2.0 termcolor-1.1.0 text-unidecode-1.3 typing-3.7.4.3 typing-extensions-3.7.4.3 unicodecsv-0.14.1 urllib3-1.25.11 zipp-3.4.0
```

Устанавливаем дополнительные airflow-пакеты для работы с Hadoop:
```bash
$ sudo pip3 install apache-airflow[async,celery,crypto,jdbc,jenkins,hdfs,hive,ldap,password,postgres,rabbitmq,vertica] --use-deprecated legacy-resolver --constraint constraints-3.6.txt
Looking in indexes: http://nx-pubrepo-reader:****@nexus.example.org:8081/repository/pypi/simple
Requirement already satisfied: apache-airflow[async,celery,crypto,hdfs,hive,jdbc,jenkins,ldap,password,pig,p
ostgres,rabbitmq,spark,sqoop,vertica] in /usr/local/lib/python3.6/site-packages (2.0.1)
Requirement already satisfied: requests==2.25.1 in /usr/local/lib/python3.6/site-packages (from -c constrain
ts-3.6.txt (line 409)) (2.25.1)  
Requirement already satisfied: Flask-Caching==1.9.0 in /usr/local/lib/python3.6/site-packages (from -c const
raints-3.6.txt (line 7)) (1.9.0)
Requirement already satisfied: termcolor==1.1.0 in /usr/local/lib/python3.6/site-packages (from -c constrain
ts-3.6.txt (line 458)) (1.1.0)
Requirement already satisfied: apache-airflow-providers-sqlite==1.0.1 in /usr/local/lib/python3.6/site-packa
ges (from -c constraints-3.6.txt (line 98)) (1.0.1)
Requirement already satisfied: Flask==1.1.2 in /usr/local/lib/python3.6/site-packages (from -c constraints-3.6.txt (line 14)) (1.1.2)
Requirement already satisfied: tenacity==6.2.0 in /usr/local/lib/python3.6/site-packages (from -c constraints-3.6.txt (line 457)) (6.2.0)...
Successfully installed Flask-Bcrypt-0.7.1 JPype1-1.2.1 JayDeBeApi-1.2.3 PyHive-0.6.3 amqp-2.6.1 apache-airflow-providers-apache-hdfs-1.0.1 apache-airflow-providers-apache-hive-1.0.1 apache-airflow-providers-apache-spark-1.0.1 apache-airflow-providers-celery-1.0.1 apache-airflow-providers-jdbc-1.0.1 apache-airflow-providers-jenkins-1.0.1 apache-airflow-providers-postgres-1.0.1 apache-airflow-providers-vertica-1.0.1 argparse-1.4.0 bcrypt-3.2.0 billiard-3.6.3.0 celery-4.4.7 eventlet-0.30.1 flower-0.9.7 future-0.18.2 gevent-21.1.2 greenlet-1.0.0 hmsclient-0.1.1 humanize-3.2.0 kombu-4.6.11 ldap3-2.9 multi-key-dict-2.0.3 pbr-5.5.1 prometheus-client-0.8.0 protobuf-3.14.0 psycopg2-binary-2.8.6 py4j-0.10.9 pyasn1-0.4.8 pyasn1-modules-0.2.8 pyspark-3.0.1 python-jenkins-1.7.0 python-ldap-3.3.1 sasl-0.2.1 snakebite-py3-3.0.5 thrift-0.13.0 thrift-sasl-0.4.2 tornado-6.1 vertica-python-1.0.1 vine-1.3.0 zope.event-4.5.0 zope.interface-5.2.0
```

Попробуем установить пакет 'apache-airflow[kerberos]':
```bash
$ sudo pip install apache-airflow[kerberos] --use-deprecated legacy-resolver --constraint constraints-3.6.txt
Looking in indexes: http://centos-main-reader:****@nexus.example.org:8081/repository/pypi/simple
Requirement already satisfied: apache-airflow[kerberos] in /usr/local/lib/python3.6/site-packages (2.0.0)
Requirement already satisfied: croniter==0.3.36 in /usr/local/lib/python3.6/site-packages (from -c constrain
ts-3.6.txt (line 109)) (0.3.36)
Requirement already satisfied: cryptography==3.2.1 in /usr/local/lib64/python3.6/site-packages (from -c cons
traints-3.6.txt (line 110)) (3.2.1)
Requirement already satisfied: iso8601==0.1.13 in /usr/local/lib/python3.6/site-packages (from -c constraints-3.6.txt (line 211)) (0.1.13)
...
Building wheels for collected packages: pykerberos
  Building wheel for pykerberos (setup.py) ... done
  Created wheel for pykerberos: filename=pykerberos-1.2.1-cp36-cp36m-linux_x86_64.whl size=58013 sha256=54044550e3499adf6034c02788de75da7513b0d3f267c4088345e20062a79c57
  Stored in directory: /root/.cache/pip/wheels/3f/67/bb/d23671c8719076ca13dd216b491ed5cea1b0494891aa9e7dba
Successfully built pykerberos
Installing collected packages: pykerberos, requests-kerberos
Successfully installed pykerberos-1.2.1 requests-kerberos-0.12.0
```

Устанавливаем пакет 'python-ldap' для возможности LDAP-аутентификации в FAB:
```bash
$ sudo pip install python-ldap
Installing collected packages: python-ldap
    Running setup.py install for python-ldap ... done
Successfully installed python-ldap-3.3.1
```
  - 2021-02-18 при установке очередного экземпляра Airflow, выяснилось, что 'python-ldap' устанавливается на одном из предыдущих этапах. Видимо разработчики внесли этот пакет в зависимости. Но пока установку этого пакета не убираю.

Для организации мэппинга между группами в IPA и ролями в Airflow устанавливаем свежий модуль 'Flask-AppBuilder 3.2.0', в котором это реализовано. Не обращаем внимания на предупреждения о нарушении зависимостей:
```bash
$ sudo pip install Flask-AppBuilder==3.2.0
...
Installing collected packages: Flask-AppBuilder
  Attempting uninstall: Flask-AppBuilder
    Found existing installation: Flask-AppBuilder 3.1.1
    Uninstalling Flask-AppBuilder-3.1.1:
      Successfully uninstalled Flask-AppBuilder-3.1.1
ERROR: pip's dependency resolver does not currently take into account all the packages that are installed. This behaviour is the source of the following dependency conflicts.
apache-airflow 2.0.1 requires flask-appbuilder~=3.1.1, but you have flask-appbuilder 3.2.0 which is incompatible.
```

## Настройка Airflow
### Создание типового файла `airflow.cfg`
Одноразово запускаем airflow для создания типового файла `/etc/airflow/airflow.cfg`:
```bash
export AIRFLOW_HOME=/data/airflow
export AIRFLOW_CONFIG=/etc/airflow/airflow.cfg
sudo -E -u airflow /usr/local/bin/airflow webserver -p 8080
```

Первый запуск Apache Airflow должен закончиться какой-нибудь ошибкой, например:
```bash
[2021-01-28 18:39:48,191] {cli_action_loggers.py:105} WARNING - Failed to log action with (sqlite3.Operation
alError) no such table: log
[SQL: INSERT INTO log (dttm, dag_id, task_id, event, execution_date, owner, extra) VALUES (?, ?, ?, ?, ?, ?,
 ?)]
[parameters: ('2021-01-28 15:39:48.187080', None, None, 'cli_webserver', None, 'airflow', '{"host_name": "ud
rvs-airf01p.example.org", "full_command": "[\'/usr/local/bin/airflow\', \'webserver\', \'-p\', \'********\']"}
')]
(Background on this error at: http://sqlalche.me/e/13/e3q8)
  ____________       _____________
 ____    |__( )_________  __/__  /________      __
____  /| |_  /__  ___/_  /_ __  /_  __ \_ | /| / /
___  ___ |  / _  /   _  __/ _  / / /_/ /_ |/ |/ /
 _/_/  |_/_/  /_/    /_/    /_/  \____/____/|__/
[2021-01-28 18:39:48,223] {dagbag.py:440} INFO - Filling up the DagBag from /dev/null
[2021-01-28 18:39:48,309] {manager.py:727} WARNING - No user yet created, use flask fab command to do it.
Traceback (most recent call last):
...
sqlalchemy.exc.OperationalError: (sqlite3.OperationalError) no such table: dag
[SQL: SELECT dag.dag_id AS dag_dag_id, dag.root_dag_id AS dag_root_dag_id, dag.is_paused AS dag_is_paused, dag.is_subdag AS dag_is_subdag, dag.is_active AS dag_is_active, dag.last_scheduler_run AS dag_last_scheduler_run, dag.last_pickled AS dag_last_pickled, dag.last_expired AS dag_last_expired, dag.scheduler_lock AS dag_scheduler_lock, dag.pickle_id AS dag_pickle_id, dag.fileloc AS dag_fileloc, dag.owners AS dag_owners, dag.description AS dag_description, dag.default_view AS dag_default_view, dag.schedule_interval AS dag_schedule_interval, dag.concurrency AS dag_concurrency, dag.has_task_concurrency_limits AS dag_has_task_concurrency_limits, dag.next_dagrun AS dag_next_dagrun, dag.next_dagrun_create_after AS dag_next_dagrun_create_after
FROM dag
WHERE dag.is_active = 1 OR dag.is_paused = 1]
(Background on this error at: http://sqlalche.me/e/13/e3q8)
```
Но в каталоге '/etc/airflow' должен появиться типовой файл 'airflow.cfg':
```bash
$ sudo ls -al /etc/airflow
total 112
drwxr-s---    2 airflow airflow   185 Feb 18 13:17 .
drwxr-xr-x. 122 root    root     8192 Feb 18 11:37 ..
-rw-r-----    1 airflow airflow 38914 Feb 18 13:17 airflow.cfg
-rw-r-----    1 root    airflow  1720 Feb 18 12:06 prod-airf1_prod-airf01p.crt
-rw-r-----    1 root    airflow  1708 Feb 18 12:06 prod-airf1_prod-airf01p.key
-r--------    1 root    airflow    15 Jan 28 22:51 psql-db.txt
-r--------    1 root    airflow    27 Jan 28 22:51 psql-pass.txt
```

В каталоге `/data/airflow` должен появиться типовой файл `webserver_config.py`:
```bash
$ sudo ls -al /data/airflow
total 12
drwxr-x---  5 airflow airflow   93 Feb 18 13:06 .
drwxr-xr-x. 3 root    root      21 Feb 18 11:54 ..
drwxr-s---+ 2 airflow airflow    6 Feb 18 11:54 dags
drwxr-x---+ 3 airflow airflow   23 Feb 18 13:09 logs
drwxr-s---+ 2 airflow airflow    6 Feb 18 11:54 plugins
-rw-r--r--  1 airflow airflow 2564 Feb 18 13:03 unittests.cfg
-rw-r--r--  1 airflow airflow 4700 Feb 18 13:03 webserver_config.py
```

### Переключение Airflow на использование PostgreSQL
Делаем бэкап `/etc/airflow/airflow.cfg`, исправляем права на эти файлы, и исправляем/добавляем строки:
```bash
sudo cp /etc/airflow/airflow.cfg /etc/airflow/airflow.cfg.origin
sudo chmod 640 /etc/airflow/airflow.cfg /etc/airflow/airflow.cfg.origin

sudo sed -i 's/^executor.*/executor = LocalExecutor/' /etc/airflow/airflow.cfg
sudo sed -i "s/^sql_alchemy_conn.*/sql_alchemy_conn = postgres+psycopg2:\/\/$(sudo cat /etc/airflow/psql-pass.txt)@127.0.0.1\/$(sudo cat /etc/airflow/psql-db.txt)/" /etc/airflow/airflow.cfg
```

В результате приведённой выше команды, в файле `/etc/airflow/airflow.cfg` будут исправлены две строки. Вторая строка, отвечающая за подключение к PostgreSQL, будет содержать пароль взятый из ранее созданного файла, и не похожий на тот, что можно наблюдать в следующем примере:
```bash
[core]
executor = LocalExecutor

sql_alchemy_conn = postgres+psycopg2://airf-user:xxxxxxxxx@127.0.0.1/airf_metadata
```

Отключение загрузки example DAGs:
```bash
sudo sed -i 's/^load_examples = True/load_examples = False/' /etc/airflow/airflow.cfg
```

Однократная инициализация базы данных Airflow:
```bash
export AIRFLOW_HOME=/data/airflow
export AIRFLOW_CONFIG=/etc/airflow/airflow.cfg
sudo -E -u airflow airflow db init

DB: postgres+psycopg2://airf_user:***@127.0.0.1/airf_metadata
[2021-01-28 19:11:07,021] {db.py:678} INFO - Creating tables
INFO  [alembic.runtime.migration] Context impl PostgresqlImpl.
INFO  [alembic.runtime.migration] Will assume transactional DDL.
INFO  [alembic.runtime.migration] Running upgrade  -> e3a246e0dc1, current schema
INFO  [alembic.runtime.migration] Running upgrade e3a246e0dc1 -> 1507a7289a2f, create is_encrypted
INFO  [alembic.runtime.migration] Running upgrade 1507a7289a2f -> 13eb55f81627, maintain history for compati
bility with earlier migrations
INFO  [alembic.runtime.migration] Running upgrade 13eb55f81627 -> 338e90f54d61, More logging into task_instance
...
[2021-01-28 19:11:09,165] {manager.py:727} WARNING - No user yet created, use flask fab command to do it.
[2021-01-28 19:11:12,349] {migration.py:517} INFO - Running upgrade 2c6edca13270 -> 61ec73d9401f, Add description field to connection
[2021-01-28 19:11:12,351] {migration.py:517} INFO - Running upgrade 61ec73d9401f -> 64a7d6477aae, fix description field in connection to be text
[2021-01-28 19:11:12,353] {migration.py:517} INFO - Running upgrade 64a7d6477aae -> e959f08ac86c, Change field in DagCode to MEDIUMTEXT for MySql
[2021-01-28 19:11:12,524] {dagbag.py:440} INFO - Filling up the DagBag from /data/airflow/dags
[2021-01-28 19:11:12,554] {example_kubernetes_executor_config.py:174} WARNING - Could not import DAGs in example_kubernetes_executor_config.py: No module named 'kubernetes'
[2021-01-28 19:11:12,554] {example_kubernetes_executor_config.py:175} WARNING - Install kubernetes dependencies with: pip install apache-airflow['cncf.kubernetes']
...
[2021-01-28 19:11:12,706] {dag.py:2266} INFO - Setting next_dagrun for example_subdag_operator.section-1 to None
[2021-01-28 19:11:12,707] {dag.py:2266} INFO - Setting next_dagrun for example_subdag_operator.section-2 to None
Initialization done
```

### Настройка HTTPS для Web UI
В `/etc/airflow/airflow.cfg` добавляем строки, указывающие использовать https и ранее созданные ключ и сертификат:
```bash
[webserver]
base_url = https://prod-airf1.example.org:8080
web_server_port = 8080
web_server_ssl_cert = /etc/airflow/prod-airf1_prod-airf01p.crt
web_server_ssl_key = /etc/airflow/prod-airf1_prod-airf01p.key
```
Для этого выполняем следующий скрипт, но **важно помнить, что переменную URL мы берём из переменной ALIASHOSTNAME, которая была определена на этапе добавления сервисов в IPA**:
```bash
URL=${ALIASHOSTNAME}

CRT=$(ls *.crt)
KEY=$(ls *.key)
sudo sed -i "s/^base_url.*/base_url=https:\/\/${URL}\:8080/" /etc/airflow/airflow.cfg
sudo sed -i "s/^web_server_ssl_cert.*/web_server_ssl_cert = \/etc\/airflow\/${CRT}/" /etc/airflow/airflow.cfg
sudo sed -i "s/^web_server_ssl_key.*/web_server_ssl_key = \/etc\/airflow\/${KEY}/" /etc/airflow/airflow.cfg
```

### Настройка LDAP-аутентификации
После создания в Каталоге отдельного системного пользователя для подключения к LDAP и всех необходимых групп пользователей, заполняем в `/data/airflow/webserver_config.py` следующей информацией. **Важно помнить, что некоторые переменные в этом скрипте берутся из ранее проделанных этапов этой инструкции, а некоторые необходимо переопределить**:
```bash
# Сохраняем оригинальный конфиг.
mv /data/airflow/webserver_config.py /data/airflow/webserver_config.py.origin

# Эта переменная берётся из переменной GROUPNAME, которую
# определили на этапе создания групп в IPA:
IPA_AIRFLOW_GROUP=${GROUPNAME}

# Переменные определяющие системную УЗ для подключения к LDAP.
BIND_USER="uid=binddn_airflow,cn=sysaccounts,cn=etc,dc=example,dc=org"
BIND_PASSWORD="xxxxxxxxxxxxxxxx"

# Заполняем новый конфиг данными.
cat << EOF > /data/airflow/webserver_config.py

import os

from flask_appbuilder.security.manager import AUTH_LDAP

basedir = os.path.abspath(os.path.dirname(__file__))

WTF_CSRF_ENABLED = True

AUTH_TYPE = AUTH_LDAP

# Uncomment to setup Full admin role name
AUTH_ROLE_ADMIN = 'Admin'

# Uncomment to setup Public role name, no authentication needed
#AUTH_ROLE_PUBLIC = 'Public'

# Will allow user self registration
AUTH_USER_REGISTRATION = True

# The default user self registration role
AUTH_USER_REGISTRATION_ROLE = "Viewer"

AUTH_LDAP_SERVER = 'ldaps://prod-ipa01.example.org ldaps://prod-ipa02.example.org ldaps://prod-ipa03.example.org'
AUTH_LDAP_SEARCH = 'cn=users,cn=accounts,dc=example,dc=org'
AUTH_LDAP_GROUP_FIELD = 'memberOf'
AUTH_LDAP_SEARCH_FILTER = '(memberOf=cn=${IPA_AIRFLOW_GROUP},cn=groups,cn=accounts,dc=example,dc=org)'
AUTH_LDAP_BIND_USER = '$BIND_USER'
AUTH_LDAP_BIND_PASSWORD = '$BIND_PASSWORD'
AUTH_LDAP_UID_FIELD = 'uid'
#AUTH_LDAP_USE_TLS = True
AUTH_LDAP_ALLOW_SELF_SIGNED = True
AUTH_LDAP_TLS_CACERTFILE = '/etc/ipa/ca.crt'

AUTH_ROLES_SYNC_AT_LOGIN = True
AUTH_ROLES_MAPPING = {
  "cn=${IPA_AIRFLOW_GROUP}_viewers,cn=groups,cn=accounts,dc=example,dc=org": ["Viewer"],
  "cn=${IPA_AIRFLOW_GROUP}_users,cn=groups,cn=accounts,dc=example,dc=org": ["User"],
  "cn=${IPA_AIRFLOW_GROUP}_ops,cn=groups,cn=accounts,dc=example,dc=org": ["Op"],
  "cn=${IPA_AIRFLOW_GROUP}_admin,cn=groups,cn=accounts,dc=example,dc=org": ["Admin"],
}
PERMANENT_SESSION_LIFETIME = 1800
EOF
```

Логгирование работы LDAP во Flask'е можно включить через изменение опции 'fab_logging_level = WARN' в /etc/airflow/airflow.cfg. Включение уровня DEBUG поможет отладке.

Некоторые моменты:
- Попытавшись использовать для одной УЗ роль 'Public' я столкнулся с тем, что пользователь с этой ролью не может войти в Airflow. Web-браузер показывает ошибку перенаправления на самого себя (или как там правильно). Именно поэтому 'AUTH_USER_REGISTRATION_ROLE = "Viewer"', а не "Public", чтобы свежезарегистрировавшийся пользователь смог войти в Airflow.
- С другой стороны, возможно, вообще выключить эту опцию, так как роли берутся из IPA.
- Попробовал отключить эту опцию и новым пользователям, кроме роли из IPA, стала назначаться роль 'Public', что в общем не критично. В общем, нет особой разницы между включением и отключением этой опции при подключённой LDAP-авторизации.

### Попытка настройки Kerberos
Добавляем systemd unit:
```bash
cat << EOF | sudo tee /etc/systemd/system/airflow-kerberos.service
[Unit]
Description=Airflow kerberos ticket renewer
After=network.target airflow-webserver.service
#Wants=postgresql-13.service mysql.service

[Service]
EnvironmentFile=/etc/sysconfig/airflow
User=airflow
Group=airflow
Type=simple
ExecStart=/usr/local/bin/airflow kerberos
Restart=on-failure
RestartSec=5s
SyslogIdentifier=airf-krb

[Install]
WantedBy=multi-user.target
EOF
```

Не забываем:
```bash
sudo systemctl daemon-reload
```

Получаем Kerberos keytab-файл:
```bash
# При необходимости получаем kerbros-билет:
#kinit

ipa service-add airflow/$(hostname)
sudo ipa-getkeytab -s prod-ipa01p.example.org -p airflow/$(hostname) -k /etc/airflow/airflow_$(hostname -s).keytab
sudo chmod 660 /etc/airflow/airflow_$(hostname -s).keytab
```

Забыл проверить, надо ли получать для себя в IPA разрешения для только что созданного сервиса: "Разрешено создавать таблицу ключей" и "Разрешено получать таблицу ключей". Или я, как админ, смог получить keytab без таковых.

В файле `/etc/airflow/airflow.cfg` добавляем/изменяем строки:
```bash
sudo sed -i 's/^security.*/security = kerberos/' /etc/airflow/airflow.cfg
sudo sed -i 's/^ccache.*/ccache = \/tmp\/airflow_krb5_ccache/' /etc/airflow/airflow.cfg
sudo sed -i "s/^principal.*/principal = airflow\/$(hostname)/" /etc/airflow/airflow.cfg
sudo sed -i 's/^reinit_frequency.*/reinit_frequency = 3600/' /etc/airflow/airflow.cfg
sudo sed -i 's/^kinit_path.*/kinit_path = kinit/' /etc/airflow/airflow.cfg
sudo sed -i "s/^keytab.*/keytab = \/etc\/airflow\/airflow_$(hostname -s).keytab/" /etc/airflow/airflow.cfg
```

Перезапускаем и проверяем:
```bash
sudo systemctl start airflow-kerberos
sudo klist -c /tmp/airflow_krb5_ccache
  Ticket cache: FILE:/tmp/airflow_krb5_ccache
  Default principal: airflow/prod-airf01p.example.org@example.org

  Valid starting       Expires              Service principal
  02/16/2021 13:37:08  02/17/2021 13:37:08  krbtgt/example.org@example.org
          renew until 02/19/2021 01:37:06
```

Из-за установленного в `/etc/airflow/airflow.cfg` параметра 'reinit_frequency = 3600', через один час тикет будет обновлён.

Далее надо разобраться с добавлением строк в 'core-site.xml', как описано в https://airflow.apache.org/docs/apache-airflow/stable/security/kerberos.html#hadoop.

## Завершение
Запуск Airflow:
```bash
systemctl daemon-reload # На всякий случай
systemctl enable --now airflow-{webserver,scheduler,kerberos}
```

В течении пары минут должен подняться Web-интерфейс по ранее определённому адресу https://prod-airf1.example.org:8080.

Закидываем один тестовый DAG:
```bash
$ sudo cp /usr/local/lib/python3.6/site-packages/airflow/example_dags/example_bash_operator.py /data/airflow/dags/
```

По умолчанию, сканирование каталога с DAG'ами выполняется каждые пять минут, поэтому придётся немного подождать момента появления нового DAG'а в списке DAG'ов, или переопределить этот параметр 'dag_dir_list_interval'.

После появления тестового DAG'а, его необходимо включить, а потом запустить. Успешное выполнение выглядит так:
![AirfowExampleTask](/img/ustanovka-apache-airflow-2-freeipa-apache-hadoop-scenarii-udaleniya/airflowexampletask.png)

Можно считать, что Airflow установлен.

## Почти полное удаление, если потребуется
Выполняем под `sudo su`:
```bash
systemctl disable --now airflow-{webserver,scheduler,kerberos}
rm -f /etc/systemd/system/airflow-*.service
systemctl daemon-reload

ID=$(getcert list|grep -3 '/etc/airflow' | grep ID | grep -o '[[:digit:]]*')
ipa-getcert stop-tracking -i ${ID}

rm -rf /etc/airflow /data/airflow /var/log/airflow
rm -f /etc/sysconfig/airflow /etc/rsyslog.d/airf-*.conf /etc/sudoers.d/airflow
sed -i '/airflow/d' /etc/logrotate.d/syslog

pip freeze > requirements.txt
pip uninstall -y -r requirements.txt
rm -f requirements.txt
rsync -av /usr/local/lib/python3.6/site-packages/pip* /root/tmp/
rm -rf /usr/local/lib/python3.6/site-packages/*
rsync -av /root/tmp/pip* /usr/local/lib/python3.6/site-packages/
rm -rf /root/tmp/pip*
rm -rf /root/.cache/pip

userdel -r airflow
```

Удаление PostgreSQL базы и пользователя:
```bash
DBNAME="airf_metadata"
DBUSER="airf_user"
sudo -u postgres psql -c "DROP DATABASE ${DBNAME};"
sudo -u postgres psql -c "DROP USER ${DBUSER};"
```

Удаление объектов из FreeIPA из-под обычного пользователя. **Важно помнить, что необходимо поменять некоторые переменные перед использованием этого скрипта**:
```bash
# При необходимости получаем kerberos-тикет:
#kinit

# Меняем значение переменных:
ALIAS="airf1"
IPAGROUP="prod_airflow_airf1"

ALIASHOSTNAME="${ALIAS}.${DOMAIN}"
SHORTHOSTNAME=$(hostname -s)
DOMAIN=$(hostname -d)
DOMAINCAPS=$(echo $(hostname -d)|tr [:lower:] [:upper:])

ipa dnsrecord-del $DOMAIN $ALIAS --cname-rec="${HOSTNAME}."

ipa service-del HTTP/${HOSTNAME}@${DOMAINCAPS}

ipa service-del airflow/${HOSTNAME}@${DOMAINCAPS}

# Удаляем группы из IPA:
ipa group-del ${IPAGROUP}_admins
ipa group-del ${IPAGROUP}_ops
ipa group-del ${IPAGROUP}_users
ipa group-del ${IPAGROUP}_viewers
ipa group-del ${IPAGROUP}

# Вручную отозвать во FreeIPA сертификат из host-объекта.
```

Остались не удалены:
1. TLS-сертификат из host-объекта IPA.
2. Установленный PostgreSQL.
3. Некоторое количество yum-пакетов установленных через:
  - `sudo yum -y groupinstall "Development Tools"`
  - `sudo yum -y install python3-devel cyrus-sasl cyrus-sasl-devel libffi-devel krb5-devel`
