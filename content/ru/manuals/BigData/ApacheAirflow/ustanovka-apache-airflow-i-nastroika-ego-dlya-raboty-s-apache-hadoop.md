---
title: "Установка Apache Airflow и настройка его для работы с Apache Hadoop (устарело)"
date: 2020-10-23
weight: 10
description: >
  Описывается процесс установки Apache Airflow и настройка его для работы с керберизированным кластером Cloudera CDH на машину, находящуюся в домене FreeIPA. Airflow переключается в режим работы LocalExecutor с хранением метаданных в PostgreSQL.
tags:
  - Apache Airflow
  - Apache Hadoop
  - BigData
  - CentOS
  - CentOS 7
aliases:
  - /a/ustanovka-apache-airflow-i-nastroika-ego-dlya-raboty-s-apache-hadoop/
---

2020-10-23

<span style="color: red;">**Эта версия Airflow устарела. Установка актуальной, второй версии, [описана здесь](/a/ustanovka-apache-airflow-2-freeipa-apache-hadoop-scenarii-udaleniya).**</span>

## Использованные материалы
- [Apache Airflow Documentation](https://airflow.apache.org/docs/stable/)

## Заметки
- Если в DAG'е указать 'run_as_user': 'eugene', то Airflow запускает процесс от имени пользователя командой `sudo -E -H`, в результате которой процесс пользователя наследует переменные родительского процесса, а переменная HOME меняется на пользовательскую директорию в `/home`.
- Надо дать права на рабочую папку airflow для группы пользователей `_airflow-dev`, так как запускаемый дочерний run_as процесс от имени пользователя, должен иметь право на запись в рабочую папку Airflow.
- Установку python-пакетов следует производить под учётной записью пользователя. Лишь пакеты airflow должны быть установлены под системной учётной записью.
- Разграничение DAG'ов между пользователями задаётся внутри DAG'а строкой: 'owner': 'eugene'.
- В `/etc/pip.conf` необходимо добавить python-репо в Nexus'е с пользовательскими библиотеками, чтобы пользователь мог вызвать `pip install` прямо из своего кода, без дополнительных операций.
- Для сборки некоторых python-пакетов (найти где я это делал), pydoop, и ещё один пакет, ...., требуется установить компилятор gcc-c++, полный пакет явы с инструментами разработчика.
- Kerberos-аутентификация, как для входа пользователей в Web UI, так и для подлключения к службам в hadoop-кластере, не работает в Apache Airflow из-за использования древнего python-пакета `python-krbV`, написанного под python2.7 много лет назад. Неизвестно, когда будет решена эта проблема.

## Установка ПО
### Установка PostgreSQL
Для работы Airflow в prod-режиме рекомендуется использовать PostgreSQL, вместо используемого по умолчанию SQLite. Ничего сложного в этом этапе нет.

1. Добавляем в `/etc/yum.repos.d/` [репо-файл для PostgreSQL 13](https://www.postgresql.org/download/linux/redhat/).
1. Устанавливаем актуальную версию PosgreSQL:

```
sudo yum update

# Install PostgreSQL:
sudo yum install -y postgresql13-server libpq5-devel python3 python3-pip

# Optionally initialize the database and enable automatic start:
sudo /usr/pgsql-13/bin/postgresql-13-setup initdb
sudo systemctl enable --now postgresql-13
```

- Для сборки пакета 'psycopg2' понадобилось установить через yum пакет `libpq5-devel`. Обходной способ: установка бинарника `psycopg2-binary`, вместо пакета `psycopg2`.

### Установка дополнительных пакетов
Устанавливаем пакеты для разработки:
```
sudo yum -y groupinstall "Development Tools"
sudo yum -y install python3-devel cyrus-sasl cyrus-sasl-devel libffi-devel krb5-devel
```

Устанавливаем python-пакет для работы с PostgreSQL:
```
sudo pip3 install wheel psycopg2 cryptography
```

- Без пакета 'cryptography' операция `airflow initdb` отработает, но пожалуется на невозможность шифрования значений.

### Установка Airflow-пакетов
Перед установкой Airflow скачиваем свежий ограничительный файл `constraints.txt` и переносим на целевую машину или, в случае доступа машины к интернету, используем URL.

Устанавливаем Airflow с помощью `pip` и актуального файла `constraints.txt`:
```
sudo pip3 install pytest-runner
sudo pip3 install apache-airflow==1.10.12 --constraint https://raw.githubusercontent.com/apache/airflow/constraints-1.10.12/constraints-3.7.txt
```

Устанавливаем дополнительные airflow-пакеты для работы с "Hadoop":
```
sudo pip3 install apache-airflow[async,celery,crypto,druid,jdbc,hdfs,hive,ldap,password,rabbitmq,vertica]
```
Пакет 'apache-airflow[kerberos]' не установится из-за зависимостей на древний пакет 'krbV', версии для python3 которого не существует.

## Подготовка к запуску Airflow
### Создание системного пользователя airflow
Создаём системного пользователя 'airflow' и разрешаем ему беспарольное 'sudo' для имперсонации, то есть запуска скриптов от имени владельца:
```
sudo useradd -r airflow -s /sbin/nologin
sudo echo "airflow ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/airflow
sudo chmod 440 /etc/sudoers.d/airflow
```

### Создание каталогов и конфигурационных файлов для работы Airflow
Создаём каталог для работы Apache Airflow и назначаем права:
```
DIR="/data/airflow"
sudo mkdir $DIR
sudo chmod 750 $DIR
sudo chown airflow.airflow $DIR
sudo setfacl -m d:g:_airflow_workfolder_access:rwx $DIR
```

- В последней команде мы назначаем default-права на каталог 'POSIX'-группе `_airflow_workfolder_access` из FreeIPA. Эти права будут наследоваться с режимом 770 для поддиректорий и 660 для файлов.
- Заметка: 'POSIX ACL' не работает с 'MemberOf'-группами из FreeIPA, но работает с  'POSIX'-группами.

Создаём файл `/etc/sysconfig/airflow` с переменными, использующимися Airflow при запуске из systemd-юнитов:
```
/etc/sysconfig/airflow
FILE="/etc/sysconfig/airflow"
echo "AIRFLOW_HOME=/data/airflow" | sudo tee $FILE
echo "AIRFLOW_CONFIG=/etc/airflow/airflow.cfg" | sudo tee -a $FILE
echo "HADOOP_CONF_DIR=/etc/hadoop/conf" | sudo tee -a $FILE
echo "HIVE_CONF_DIR=/etc/hive/conf" | sudo tee -a $FILE
```

- Без переменных `HADOOP_CONF_DIR` и `HIVE_CONF_DIR`, в Airflow не будет работать 'SparkSubmitOperator', так как он использует файлы `hive_site.xml` и `yarn-site.xml`.
- Каталог `/etc/hadoop/`, в случае его отсутствия из-за, например, недавнего добавления этого узла в кластер, может быть скопирован с любого кластерного узла. Или выполняем перезагрузку кластера и этот каталог, как и прочие кластерные папки, автоматически появятся на своих местах.

Создаём каталог `/etc/airflow` для хранения `airflow.cfg`, сертификатов, keytab'ов и прочих подобных файлов:
```
DIR="/etc/airflow"
sudo mkdir $DIR
sudo chmod 750 $DIR
sudo chown airflow.airflow $DIR
```

### Создание systemd-юнитов
Взято из [https://github.com/apache/airflow/tree/master/scripts/systemd](https://github.com/apache/airflow/tree/master/scripts/systemd). При появлении других зависимостей для запуска сервисов Airflow, добавляем их в строки 'After' и 'Wants'. Примеры можно посмотреть в [https://github.com/apache/airflow/tree/master/scripts/systemd](https://github.com/apache/airflow/tree/master/scripts/systemd).

`/etc/systemd/system/airflow-scheduler.service`:
```
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

[Install]
WantedBy=multi-user.target
```

`/etc/systemd/system/airflow-webserver.service`:
```
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

[Install]
WantedBy=multi-user.target
```

Kerberos в Apache Airflow не работает, но приведу здесь этот 'unit' в надежде на светлое будущее:

`/etc/systemd/system/airflow-kerberos.service`:
```
[Unit]
Description=Airflow kerberos ticket renewer
After=network.target postgresql-13.service
Wants=postgresql-13.service mysql.service

[Service]
EnvironmentFile=/etc/sysconfig/airflow
User=airflow
Group=airflow
Type=simple
ExecStart=/usr/local/bin/airflow kerberos
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

Не забываем выполнить:
```
sudo systemctl daemon-reload
```

Одноразовый запуск Airflow для создания типового файла `/etc/airflow/airflow.cfg`, с последующей зачисткой каталога '/data/airflow', в которой появятся ненужные в дальнейшем файлы:
```
export AIRFLOW_HOME=/data/airflow
export AIRFLOW_CONFIG=/etc/airflow/airflow.cfg
airflow webserver -p 8080
cd /data/airflow && sudo rm -rf *
```
- Первый запуск Apache Airflow должен закончиться ошибкой, а в каталоге `/etc/airflow` должен появиться типовой файл `airflow.cfg`.

### Создание базы данных airflow в PostgreSQL
Далее приведены команды `psql` для создания базы данных:
```
export airflowpassword=$(cat /dev/urandom | tr -dc A-Za-z0-9 | head -c15)
sudo -u postgres psql -c "CREATE DATABASE airflow_metadata;"
sudo -u postgres psql -c "CREATE USER airflow WITH password '$airflowpassword';"
sudo -u postgres psql -c "GRANT ALL PRIVILEGES ON DATABASE airflow_metadata TO airflow;"
echo $airflowpassword | sudo tee /etc/airflow/psql-pass.txt
sudo chmod 400 /etc/airflow/psql-pass.txt
```

## Создание сертификата для работы HTTPS в Web UI Airflow
Так как машина находится в домене FreeIPA, то создаём ключ и сертификаты встроенными средствами FreeIPA.

Подготавливаем ключ и сертификат для Web UI Airflow. Чтобы было удобней, выбираем короткое имя для URL: 'airf1.example.org'.

Выполняем работу из-под root'а.

1. Сначала получаем kerberos-билет.
2. Добавляем 'alias' для существующего принципала, соответствующего хосту.
3. Создаём ключ и запрашиваем сертификат к нему одной командой.
4. Проверяем, что сертификат добавлен и будет обновляться в срок (auto-renew: yes).
5. Добавляем CNAME-запись в DNS-зону.

```
# kinit eugene
# ipa service-add-principal HTTP/srv-app100.example.org HTTP/airf1.example.org
-------------------------------------------------------------------------------------
Added new aliases to the service principal "HTTP/srv-app100.example.org@EXAMPLE.ORG"
-------------------------------------------------------------------------------------
  Principal name: HTTP/srv-app100.example.org@EXAMPLE.ORG
  Principal alias: HTTP/srv-app100.example.org@EXAMPLE.ORG, HTTP/airf1.example.org@EXAMPLE.ORG,
                   HTTP/airf1.example.org@EXAMPLE.ORG

# ipa-getcert request -f /etc/airflow/airf1_srv-app100.crt -k /etc/airflow/airf1_srv-app100.key
New signing request "20201105144139" added.

# getcert list
Request ID '20201105144139':
        status: MONITORING
        stuck: no
        key pair storage: type=FILE,location='/etc/airflow/airf1_srv-app100.key'
        certificate: type=FILE,location='/etc/airflow/airf1_srv-app100.crt'
        CA: IPA
        issuer: CN=Certificate Authority,O=EXAMPLE.ORG
        subject: CN=srv-app100.example.org,O=EXAMPLE.ORG
        expires: 2022-11-06 14:41:39 UTC
        dns: srv-app100.example.org
        principal name: host/srv-app100.example.org@EXAMPLE.ORG
        key usage: digitalSignature,nonRepudiation,keyEncipherment,dataEncipherment
        eku: id-kp-serverAuth,id-kp-clientAuth
        pre-save command:
        post-save command:
        track: yes
        auto-renew: yes

# ipa dnsrecord-add example.org airf1 --cname-rec='srv-app100.example.org.'
  Record name: airf1
  CNAME record: srv-app100.example.org
```

Если понадобится снять сертификат с отслеживания, то выполняем `getcert list`, выбираем сертификат и даём команду снятия с отслеживания:
```
# ipa-getcert stop-tracking -i 20201105144139
```

Если нужно поставить сертификат на автообновление, то выполняем:
```
# ipa-getcert start-tracking -f /etc/airflow/airf1_srv-app100.crt -k /etc/airflow/airf1_srv-app100.key
```

## Настройка Airflow
### Переключение Airflow на использование PostgreSQL
Исправляем/добавляем строки в `/etc/airflow/airflow.cfg`:
```
sed -i 's/^executor.*/executor = LocalExecutor/' /etc/airflow/airflow.cfg
sed -i "s/^sql_alchemy_conn.*/sql_alchemy_conn = postgres+psycopg2:\/\/airflow:$(sudo cat /etc/airflow/psql-pass.txt)@127.0.0.1\/airflow_metadata/" /etc/airflow/airflow.cfg
```

В результате приведённой выше команды, в файле `/etc/airflow/airflow.cfg` будут исправлены две строки. Вторая строка, отвечающая за подключение к PostgreSQL, будет содержать пароль взятый из ранее созданного файла `psql-pass.txt`. Пример результата в файле `/etc/airflow/airflow.cfg`:
```
[core]
executor = LocalExecutor

sql_alchemy_conn = postgres+psycopg2://airflow:airflow-password@127.0.0.1/airflow_metadata
```

Однократная инициализация базы данных Airflow:
```
export AIRFLOW_CONFIG=/etc/airflow/airflow.cfg
airflow initdb
```

### Настройка HTTPS для Web UI
В `/etc/airflow/airflow.cfg` добавляем строки, указывающие использовать https и ранее созданные ключ и сертификат:
```
[webserver]
base_url = https://airf1.example.org:8080
web_server_port = 8080
web_server_ssl_cert = /etc/airflow/airf1_srv-app100.crt
web_server_ssl_key = /etc/airflow/airf1_srv-app100.key
```

Перезапускаем Web UI:
```
sudo systemctl restart airflow-webserver
```

### Настройка LDAP-аутентификации
После создания в FreeIPA отдельного системного пользователя для подключения к LDAP и всех необходимых групп пользователей, заполняем в `/etc/airflow/airflow.cfg` следующий блок:
```
[webserver]
authenticate = True
auth_backend = airflow.contrib.auth.backends.ldap_auth
filter_by_owner = True

# Note that the ldap server needs the "memberOf" overlay to be set up
# in order to user the ldapgroup mode
owner_mode = user

[ldap]
uri = ldaps://ldap.example.org:636

user_filter = memberOf=cn=_airflow-dev,cn=groups,cn=accounts,dc=example,dc=org
user_name_attr = uid
group_member_attr = memberOf
superuser_filter = memberOf=cn=_airflow-dev_super-users,cn=groups,cn=accounts,dc=example,dc=org
data_profiler_filter = memberOf=cn=_airflow-dev_data-profilers,cn=groups,cn=accounts,dc=example,dc=org

bind_user = uid=binddn_airflow-dev,cn=sysaccounts,cn=etc,dc=example,dc=org
bind_password = (bind_password)
basedn = cn=users,cn=accounts,dc=example,dc=org

cacert = /etc/ipa/ca.crt
search_scope = LEVEL
```

## Запуск Airflow
```
systemctl enable --now airflow-webserver
systemctl enable --now airflow-scheduler
```
