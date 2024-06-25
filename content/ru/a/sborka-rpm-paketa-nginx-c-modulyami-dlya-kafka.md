---
title: "Сборка rpm-пакета nginx c модулями для kafka"
date: 2020-05-26
weight: 10
description: >
  Сборка rpm-пакета nginx c модулями для kafka c последующим размещением в Nexus Repository
tags:
  - Apache Kafka
  - Nginx
  - BigData
---

2020-05-26

Сборку производил в свежеустановленной виртуальной машине CentOS 7 (минимальный дистр), из-под обычного пользователя имеющего беспарольное sudo.

## Сборка пакета librdkafka
Сначала надо собрать свежий `https://github.com/edenhill/librdkafka`, так как в репо CentOS'а librdkafka имеет версию 0.11.5, тогда как на странице разработчиков&nbsp;&mdash;&nbsp;1.5.0.

Сборка этого пакета заняла по времени чуть больше часа, но не из-за сложности. К счастью, разработчики всё продумали и собирать этот пакет проще простого.

Сборка пакета будет производиться в песочнице, с помощью `mock`.
```
mkdir ~/src; cd ~/src
git clone https://github.com/edenhill/librdkafka
sudo yum install mock
sudo usermod -aG mock $USER
exit
```

После повторного входа под своей учётной записью, но уже с членством в группе `mock`, запускаем сборку:
```
$ cd ~/src/librdkafka/packaging/rpm
$ make
```
<details><p>

```
cd ../../ && \
git archive --prefix=librdkafka-1.5.0/ \
        -o packaging/rpm/SOURCES/librdkafka-1.5.0.tar.gz HEAD
mkdir -p pkgs-1.5.0-1-default
rm -f pkgs-1.5.0-1-default/librdkafka*.rpm
/usr/bin/mock \
        -r default \
         \
        --define "__version 1.5.0" \
        --define "__release 1" \
        --enable-network \
        --resultdir=pkgs-1.5.0-1-default \
        --no-clean --no-cleanup-after \
        --install epel-release \
        --buildsrpm \
        --spec=librdkafka.spec \
        --sources=SOURCES || \
(tail -n 100 pkgs-1.5.0*/*log ; false)
INFO: mock.py version 2.2 starting (python version = 3.6.8)...
```
</p></details>

В песочницу будут установлены необходимые пакеты (в моём случае около трёх сотен) и произведена сборка пакета.

После окончания работ, в каталоге  `~/src/librdkafka/packaging/rpm/pkgs-1.5.0-1-default` можно обнаружить полный набор пакетов:
```
-rw-r--r--. 1 u mock  931281 May 20 16:16 librdkafka1-1.5.0-1.el7.x86_64.rpm
-rw-r--r--. 1 u mock 2752290 May 20 16:06 librdkafka-1.5.0-1.el7.src.rpm
-rw-r--r--. 1 u mock 4090554 May 20 16:16 librdkafka-debuginfo-1.5.0-1.el7.x86_64.rpm
-rw-r--r--. 1 u mock  953416 May 20 16:16 librdkafka-devel-1.5.0-1.el7.x86_64.rpm
```

Позже будет необходимо установить два пакета из этих четырёх.

## Сборка пакета nginx-kafka
### Подключение nginx-репозитория
Добавляем в `/etc/yum.repos.d` файл `nginx.repo` со следующим содержимым:
```
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true

[nginx-source]
name=nginx source repo
baseurl=http://nginx.org/packages/centos/7/SRPMS/
gpgcheck=1
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
```

### Установка необходимых для сборки пакетов
```
sudo yum update
sudo yum install yum-utils wget git rpmdevtools
sudo yum install jansson jansson-devel
cd ~/src/librdkafka/packaging/rpm/pkgs-1.5.0-1-default
sudo yum install librdkafka1-1.5.0-1.el7.x86_64.rpm librdkafka-devel-1.5.0-1.el7.x86_64.rpm
```

### Включение себя в группу builder
```
sudo usermod -aG builder $USER
```

### Создание структуры каталогов ~/rpmbuild
```
rpmdev-setuptree
```

### Размещение исходников nginx в ~/rpmbuild
```
cd ~
yumdownloader --source nginx
rpm -Uvh nginx-1.18.0-1.el7.ngx.src.rpm
```

### Установка зависимостей
```
sudo yum-builddep nginx
```

### Скачивание модулей для kafka
```
mkdir ~/src; cd ~/src
git clone https://github.com/brg-liuwei/ngx_kafka_module.git
git clone https://github.com/kaltura/nginx-kafka-log-module
```

### Изменения в ~/rpmbuild
#### ~/rpmbuild/SPECS/nginx.spec
В файле `~/rpmbuild/SPECS/nginx.spec`, в конец строки с передаваемыми параметрами `%define BASE_CONFIGURE_ARGS ... --with-stream_ssl_preread_module)"`, добавляем два модуля, указывая абсолютные пути, и три параметра:
```
--add-module=/home/user/src/ngx_kafka_module --add-module=/home/user/src/nginx-kafka-log-module --with-http_perl_module --with-http_degradation_module --with-pcre
```

Без опции `Conflicts`, наш новый пакет "nginx-kafka" может быть замещен пакетом "nginx". А для использования последней `librdkafka-1.5.0`, а не старой из `epel`, используем опцию `Requires`. Итого, изменяем две существующие строки Summary и Name и добавляем две строки Requires и Conflicts:
```
Summary: High performance web server with kafka modules
Name: nginx-kafka
Requires: librdkafka1 >= 1.5.0
Conflicts: nginx
```

Добавляем пять путей к perl-модулю, и один путь к ману, под существующими строками `%files` и `%defattr(-,root,root)`:
```
%files
%defattr(-,root,root)

%{_libdir}/perl5/perllocal.pod
%{_libdir}/perl5/vendor_perl/auto/nginx/.packlist
%{_libdir}/perl5/vendor_perl/auto/nginx/nginx.bs
%{_libdir}/perl5/vendor_perl/auto/nginx/nginx.so
%{_libdir}/perl5/vendor_perl/nginx.pm
%{_mandir}/man3/nginx.3pm.gz
```

#### ~/rpmbuild/SOURCES/nginx-kafka-1.18.0.tar.gz
Перепаковываем архив с исходниками, чтобы в процессе сборки, при распаковке архива c  исходниками, был создан каталог `nginx-kafka-1.18`, а не просто `nginx-1.18`:
```
mkdir ~/tmp
mv ~/rpmbuild/SOURCES/nginx-1.18.0.tar.gz ~/tmp/
cd ~/tmp
tar xvf nginx-1.18.0.tar.gz
mv nginx-1.18.0 nginx-kafka-1.18.0
tar -zcvf nginx-kafka-1.18.0.tar.gz nginx-kafka-1.18.0
mv nginx-kafka-1.18.0.tar.gz ~/rpmbuild/SOURCES/
```

### Сборка
```
rpmbuild -ba ~/rpmbuild/SPECS/nginx.spec
```

### Результат
В `~/rpmbuild/RPMS` и `~/rpmbuild/SRPMS` наблюдаем только что созданные пакеты.

## Добавление пакетов в Nexus Repository
В нексусе создаём репозиторий "nginx-kafka" и загружаем в него пакеты (пакет `cyrus-sasl`, потребовавшийся в процессе экзерсисов, предварительно скачиваем с помощью `yumdownloader`). Итого должно получиться:
```
cyrus-sasl-2.1.26-23.el7.x86_64.rpm
librdkafka1-1.5.0-1.el7.x86_64.rpm
librdkafka-1.5.0-1.el7.src.rpm
librdkafka-devel-1.5.0-1.el7.x86_64.rpm
nginx-kafka-1.18.0-1.el7.ngx.src.rpm
nginx-kafka-1.18.0-1.el7.ngx.x86_64.rpm
```

На целевой машине добавляем описание репозитория (вариант без авторизации на нексусе):
```
[nginx-kafka]
name = Nginx kafka
baseurl = http://nexus.example.org:8081/repository/nginx-kafka/
gpgcheck = 0
enabled = 1
```

На целевом сервере, после `yum update`, добавляем `nginx-kafka` и видим:
```
Dependencies Resolved

=================================================================================================
 Package               Arch             Version                      Repository             Size
=================================================================================================
Installing:
 nginx-kafka           x86_64           1:1.18.0-1.el7.ngx           nginx-kafka           778 k
Installing for dependencies:
 cyrus-sasl            x86_64           2.1.26-23.el7                nginx-kafka            88 k
 librdkafka1           x86_64           1.5.0-1.el7                  nginx-kafka           909 k

Transaction Summary
=================================================================================================
Install  1 Package (+2 Dependent packages)

Total download size: 1.7 M
Installed size: 5.2 M
```

Модули:
https://github.com/edenhill/librdkafka
https://github.com/brg-liuwei/ngx_kafka_module
https://github.com/kaltura/nginx-kafka-log-module

Использованные материалы:
https://www.fatalerrors.org/a/nginx-integrates-kafka.html
https://nginx.org/ru/linux_packages.html#RHEL-CentOS
