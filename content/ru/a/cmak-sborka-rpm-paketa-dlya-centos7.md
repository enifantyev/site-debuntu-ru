---
title: "CMAK. 01. Сборка rpm-пакета для CentOS7"
date: 2021-11-25
weight: 10
description: >
  Сборка CMAK (Cluster Manager for Apache Kafka), инструмента для просмотра тем Kafka и групп потребителей.
tags:
  - Apache Kafka
  - CMAK
  - CentOS
  - CentOS 7
---

2021-11-25

https://github.com/yahoo/CMAK

## 1. Подготовка к сборке
Выбираем по умолчанию интерпретатор python2 и проверяем эту установку:
```
$ sudo update-alternatives --config python

There are 2 programs which provide 'python'.

  Selection    Command
-----------------------------------------------
 + 1           /usr/bin/python2.7
*  2           /usr/bin/python3

Enter to keep the current selection[+], or type selection number: 1

$ python -V

Python 2.7.5
```

В дополнение к Maven'у устанавливаем пакеты 'java-11-openjdk' и 'rpm-build':
```
$ sudo yum -y install java-11-openjdk rpm-build
```

Скачиваем и распаковываем исходники последней версии приложения (на данный момент CMAK 3.0.0.5):
```
FILE="3.0.0.5.tar.gz"

mkdir ~/src && cd ~/src
curl -LO https://github.com/yahoo/CMAK/archive/refs/tags/${FILE}
tar xvf ${FILE}
#
```

## 2. Сборка rpm-пакета
Переходим в каталог с распакованными исходниками и начинаем сборку:
```
cd ~/src/CMAK-3.0.0.5
export JAVA_HOME=/usr/lib/jvm/jre-11-openjdk
export PATH=/usr/lib/jvm/jre-11-openjdk/bin:$PATH
./sbt rpm:packageBin
#
```

stdout:
```
[info] Loading settings for project cmak-3-0-0-5-build from plugins.sbt ...
[info] Loading project definition from /data/src/CMAK-3.0.0.5/project
[info] Loading settings for project root from build.sbt ...
[info] Set current project to cmak (in build file:/data/src/CMAK-3.0.0.5/)
[info] Wrote /data/src/CMAK-3.0.0.5/target/scala-2.12/cmak_2.12-3.0.0.5.pom
[info] Building target platforms: noarch-yahoo-Linux
[info] Building for target noarch-yahoo-Linux
[info] Executing(%install): /bin/sh -e /tmp/sbt_cc7a974b/rpm-tmp.GgFqxU
[info] + umask 022
[info] + cd /data/src/CMAK-3.0.0.5/target/rpm/BUILD
[info] + '[' /data/src/CMAK-3.0.0.5/target/rpm/buildroot '!=' / ']'
[info] + rm -rf /data/src/CMAK-3.0.0.5/target/rpm/buildroot
[info] ++ dirname /data/src/CMAK-3.0.0.5/target/rpm/buildroot
[info] + mkdir -p /data/src/CMAK-3.0.0.5/target/rpm
[info] + mkdir /data/src/CMAK-3.0.0.5/target/rpm/buildroot
[info] + '[' -e /data/src/CMAK-3.0.0.5/target/rpm/buildroot ']'
[info] + mv /data/src/CMAK-3.0.0.5/target/rpm/tmp-buildroot/etc /data/src/CMAK-3.0.0.5/target/rpm/tmp-buildroot/usr /dat
a/src/CMAK-3.0.0.5/target/rpm/tmp-buildroot/var /data/src/CMAK-3.0.0.5/target/rpm/buildroot
[info] + /usr/lib/rpm/check-buildroot
[info] + /usr/lib/rpm/redhat/brp-compress
[info] + /usr/lib/rpm/redhat/brp-strip /usr/bin/strip
[info] + /usr/lib/rpm/redhat/brp-strip-comment-note /usr/bin/strip /usr/bin/objdump
[info] + /usr/lib/rpm/redhat/brp-strip-static-archive /usr/bin/strip
[info] + /usr/lib/rpm/brp-python-bytecompile /usr/bin/python 1
[info] + /usr/lib/rpm/redhat/brp-python-hardlink
[info] Processing files: cmak-3.0.0.5-1.noarch
[info] Provides: cmak = 3.0.0.5-1 config(cmak) = 3.0.0.5-1 osgi(ch.qos.logback.classic) = 1.2.3 osgi(ch.qos.logback.core) = 1.2.3 osgi(com.fasterxml.jackson.core.jackson-annotations) = 2.10.0 osgi(com.fasterxml.jackson.core.jackson-core) = 2.10.0 osgi(com.fasterxml.jackson.core.jackson-databind) = 2.10.0 osgi(com.fasterxml.jackson.dataformat.jackson-dataformat-csv) = 2.10.0 osgi(com.fasterxml.jackson.datatype.jackson-datatype-jdk8) = 2.10.0 osgi(com.fasterxml.jackson.datatype.jackson-datatype-jsr310) = 2.8.11 osgi(com.fasterxml.jackson.module.jackson-module-paranamer) = 2.10.0 osgi(com.fasterxml.jackson.module.jackson.module.scala) = 2.10.0 osgi(com.github.ben-manes.caffeine) = 2.6.2 osgi(com.github.luben.zstd-jni) = 1.4.3 osgi(com.google.guava) = 23.6.1 osgi(com.thoughtworks.paranamer) = 2.8.0 osgi(com.typesafe.akka.actor) = 2.5.19 osgi(com.typesafe.akka.protobuf) = 2.5.19 osgi(com.typesafe.akka.slf4j) = 2.5.19 osgi(com.typesafe.akka.stream) = 2.5.19 osgi(com.typesafe.config) = 1.3.3 osgi(com.typesafe.scala-logging) = 3.9.2 osgi(com.typesafe.sslconfig) = 0.3.6 osgi(com.unboundid.ldap.sdk) = 4.0.9 osgi(curator-client) = 2.12.0 osgi(curator-framework) = 2.12.0 osgi(curator-recipes) = 2.12.0 osgi(io.jsonwebtoken.jjwt) = 0.7.0 osgi(io.netty.buffer) = 4.1.42 osgi(io.netty.codec) = 4.1.42 osgi(io.netty.common) = 4.1.42 osgi(io.netty.handler) = 4.1.42 osgi(io.netty.resolver) = 4.1.42 osgi(io.netty.transport) = 4.1.42 osgi(io.netty.transport-native-epoll) = 4.1.42 osgi(io.netty.transport-native-unix-common) = 4.1.42 osgi(javax.activation-api) = 1.2.0 osgi(jaxb-api) = 2.3.1 osgi(jcl.over.slf4j) = 1.7.25 osgi(joda-time) = 2.9.9 osgi(jul.to.slf4j) = 1.7.25 osgi(log4j.over.slf4j) = 1.7.25 osgi(lz4-java) = 0 osgi(net.sf.jopt-simple.jopt-simple) = 5.0.4 osgi(org.apache.commons.cli) = 1.4.0 osgi(org.apache.commons.codec) = 1.11.0 osgi(org.apache.commons.compress) = 1.9.0 osgi(org.apache.commons.lang3) = 3.6.0 osgi(org.jsr-305) = 3.0.2 osgi(org.reactivestreams.reactive-streams) = 1.0.2 osgi(org.scala-lang.modules.scala-collection-compat) = 2.1.2 osgi(org.scala-lang.modules.scala-parser-combinators) = 1.1.1 osgi(org.scala-lang.modules.scala-xml) = 1.0.6 osgi(org.scala-lang.scala-library) = 2.12.10 osgi(org.scala-lang.scala-reflect) = 2.12.10 osgi(org.scalaz.core) = 7.2.27 osgi(org.xerial.snappy.snappy-java) = 1.1.7 osgi(slf4j.api) = 1.7.28
[info] Requires(interp): /bin/sh /bin/sh /bin/sh /bin/sh
[info] Requires(rpmlib): rpmlib(CompressedFileNames) <= 3.0.4-1 rpmlib(FileDigests) <= 4.6.0-1 rpmlib(PayloadFilesHavePrefix) <= 4.0-1
[info] Requires(pre): /bin/sh
[info] Requires(post): /bin/sh
[info] Requires(preun): /bin/sh
[info] Requires(postun): /bin/sh
[info] Requires: /usr/bin/env
[info] Checking for unpackaged file(s): /usr/lib/rpm/check-files /data/src/CMAK-3.0.0.5/target/rpm/buildroot
[info] Wrote: /data/src/CMAK-3.0.0.5/target/rpm/RPMS/noarch/cmak-3.0.0.5-1.noarch.rpm
[info] Executing(%clean): /bin/sh -e /tmp/sbt_cc7a974b/rpm-tmp.E6uoCQ
[info] + umask 022
[info] + cd /data/src/CMAK-3.0.0.5/target/rpm/BUILD
[info] + /usr/bin/rm -rf /data/src/CMAK-3.0.0.5/target/rpm/buildroot
[info] + exit 0
[success] Total time: 115 s (01:55), completed Nov 25, 2021, 1:59:33 PM
```

## 3. Результат
В каталоге `target/rpm/RPMS/noarch` наблюдаем пакет `cmak-3.0.0.5-1.noarch.rpm`.
