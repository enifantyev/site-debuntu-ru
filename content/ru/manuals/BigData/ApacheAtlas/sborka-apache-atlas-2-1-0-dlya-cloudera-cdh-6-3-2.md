---
title: "Сборка Apache Atlas 2.1.0 для Cloudera CDH 6.3.2 (устарело)"
date: 2021-08-05
weight: 10
description: >
  Сборка Apache Atlas 2.1.0 для Cloudera CDH 6.3.2 на CentOS 7.
tags:
  - Apache Atlas
  - Cloudera CDH 6.3.2
  - Apache Hadoop
  - CentOS
  - CentOS 7
  - BigData
aliases:
  - /a/sborka-apache-atlas-2-1-0-dlya-cloudera-cdh-6-3-2/
  - /manuals/bigdata/apacheatlas/
---

2021-08-05

## 1. Подготовка хоста
1. Сначала хост добавляем в Cloudera-кластер, чтобы при установке Cloudera CDH были корректно добавлены локальные аккаунты и группы.
2. В процессе добавления в кластер, перед включением TLS-аутентификации для Cloudera Agent, добавляем хост в домен.
3. На хост установлены шлюзы для YARN, HBase, Hive, Kafka, Solr.

## 2. Установка Maven
Сначала устанавливаем maven 3.0.5-17.el7 из стандартных репо для CentOS7:
```
$ sudo yum install maven
...
Dependencies Resolved

=====================================================================================================================
 Package                                   Arch            Version                           Repository        Size
=====================================================================================================================
Installing:
 maven                                     noarch          3.0.5-17.el7                      bases             1.3 M
Installing for dependencies:
 aether-api                                noarch          1.13.1-13.el7                     bases              89 k
 aether-connector-wagon                    noarch          1.13.1-13.el7                     bases              34 k
 aether-impl                               noarch          1.13.1-13.el7                     bases             124 k
 aether-spi                                noarch          1.13.1-13.el7                     bases              19 k
 aether-util                               noarch          1.13.1-13.el7                     bases             119 k
 aopalliance                               noarch          1.0-8.el7                         bases              11 k
 apache-commons-cli                        noarch          1.2-13.el7                        bases              50 k
 apache-commons-codec                      noarch          1.8-7.el7                         bases             223 k
 apache-commons-io                         noarch          1:2.4-12.el7                      bases             189 k
 apache-commons-lang                       noarch          2.6-15.el7                        bases             276 k
 apache-commons-logging                    noarch          1.1.2-7.el7                       bases              78 k
 apache-commons-net                        noarch          3.2-8.el7.centos                  bases             261 k
 atinject                                  noarch          1-13.20100611svn86.el7            bases              13 k
 atk                                       x86_64          2.28.1-2.el7                      bases             263 k
 avalon-framework                          noarch          4.3-10.el7                        bases              88 k
 avalon-logkit                             noarch          2.1-14.el7                        bases              87 k
 bcel                                      noarch          5.2-18.el7                        bases             469 k
 cairo                                     x86_64          1.15.12-4.el7                     bases             741 k
 cal10n                                    noarch          0.7.7-4.el7                       bases              36 k
 cdi-api                                   noarch          1.0-11.SP4.el7                    bases              41 k
 cglib                                     noarch          2.2-18.el7                        bases             255 k
 copy-jdk-configs                          noarch          3.3-10.el7_5                      bases              21 k
 dejavu-fonts-common                       noarch          2.33-6.el7                        bases              64 k
 dejavu-sans-fonts                         noarch          2.33-6.el7                        bases             1.4 M
 easymock2                                 noarch          2.5.2-12.el7                      bases              92 k
 felix-framework                           noarch          4.2.1-5.el7                       bases             481 k
 fontconfig                                x86_64          2.13.0-4.3.el7                    bases             254 k
 fontpackages-filesystem                   noarch          1.44-8.el7                        bases             9.9 k
 fribidi                                   x86_64          1.0.2-1.el7_7.1                   bases              79 k
 gdk-pixbuf2                               x86_64          2.36.12-3.el7                     bases             570 k
 geronimo-annotation                       noarch          1.0-15.el7                        bases              23 k
 geronimo-jms                              noarch          1.1.1-19.el7                      bases              31 k
 giflib                                    x86_64          4.1.6-9.el7                       bases              40 k
 google-guice                              noarch          3.1.3-9.el7                       bases             385 k
 graphite2                                 x86_64          1.3.10-1.el7_3                    bases             115 k
 gtk-update-icon-cache                     x86_64          3.22.30-6.el7                     updates           27 k
 gtk2                                      x86_64          2.24.31-1.el7                     bases             3.4 M
 guava                                     noarch          13.0-6.el7                        bases             1.6 M
 hamcrest                                  noarch          1.3-6.el7                         bases             124 k
 harfbuzz                                  x86_64          1.7.5-2.el7                       bases             267 k
 hicolor-icon-theme                        noarch          0.12-7.el7                        bases              42 k
 httpcomponents-client                     noarch          4.2.5-5.el7_0                     bases             425 k
 httpcomponents-core                       noarch          4.2.4-6.el7                       bases             466 k
 jakarta-commons-httpclient                noarch          1:3.1-16.el7_0                    bases             241 k
 jasper-libs                               x86_64          1.900.1-33.el7                    bases             150 k
 java-1.8.0-openjdk                        x86_64          1:1.8.0.292.b10-1.el7_9           updates          310 k
 java-1.8.0-openjdk-devel                  x86_64          1:1.8.0.292.b10-1.el7_9           updates          9.8 M
 java-1.8.0-openjdk-headless               x86_64          1:1.8.0.292.b10-1.el7_9           updates           33 M
 javamail                                  noarch          1.4.6-8.el7                       bases             758 k
 javapackages-tools                        noarch          3.4.1-11.el7                      bases              73 k
 javassist                                 noarch          3.16.1-10.el7                     bases             627 k
 jbigkit-libs                              x86_64          2.0-11.el7                        bases              46 k
 jboss-ejb-3.1-api                         noarch          1.0.2-10.el7                      bases              54 k
 jboss-el-2.2-api                          noarch          1.0.1-0.7.20120212git2fabd8.el7   bases              44 k
 jboss-interceptors-1.1-api                noarch          1.0.2-0.9.20120319git49a904.el7   bases              27 k
 jboss-jaxrpc-1.1-api                      noarch          1.0.1-7.el7                       bases              44 k
 jboss-servlet-3.0-api                     noarch          1.0.1-9.el7                       bases              82 k
 jboss-transaction-1.1-api                 noarch          1.0.1-8.el7                       bases              40 k
 jline                                     noarch          1.0-8.el7                         bases              69 k
 jsch                                      noarch          0.1.50-5.el7                      bases             239 k
 jsoup                                     noarch          1.6.1-10.el7                      bases             263 k
 junit                                     noarch          4.11-8.el7                        bases             261 k
 jzlib                                     noarch          1.1.1-6.el7                       bases              72 k
 libICE                                    x86_64          1.0.9-9.el7                       bases              66 k
 libSM                                     x86_64          1.2.2-2.el7                       bases              39 k
 libX11                                    x86_64          1.6.7-3.el7_9                     updates          607 k
 libX11-common                             noarch          1.6.7-3.el7_9                     updates          164 k
 libXau                                    x86_64          1.0.8-2.1.el7                     bases              29 k
 libXcomposite                             x86_64          0.4.4-4.1.el7                     bases              22 k
 libXcursor                                x86_64          1.1.15-1.el7                      bases              30 k
 libXdamage                                x86_64          1.1.4-4.1.el7                     bases              20 k
 libXext                                   x86_64          1.3.3-3.el7                       bases              39 k
 libXfixes                                 x86_64          5.0.3-1.el7                       bases              18 k
 libXft                                    x86_64          2.3.2-2.el7                       bases              58 k
 libXi                                     x86_64          1.7.9-1.el7                       bases              40 k
 libXinerama                               x86_64          1.1.3-2.1.el7                     bases              14 k
 libXrandr                                 x86_64          1.5.1-2.el7                       bases              27 k
 libXrender                                x86_64          0.9.10-1.el7                      bases              26 k
 libXtst                                   x86_64          1.2.3-1.el7                       bases              20 k
 libXxf86vm                                x86_64          1.1.4-1.el7                       bases              18 k
 libfontenc                                x86_64          1.1.3-3.el7                       bases              31 k
 libglvnd                                  x86_64          1:1.0.1-0.8.git5baa1e5.el7        bases              89 k
 libglvnd-egl                              x86_64          1:1.0.1-0.8.git5baa1e5.el7        bases              44 k
 libglvnd-glx                              x86_64          1:1.0.1-0.8.git5baa1e5.el7        bases             125 k
 libjpeg-turbo                             x86_64          1.2.90-8.el7                      bases             135 k
 libthai                                   x86_64          0.1.14-9.el7                      bases             187 k
 libtiff                                   x86_64          4.0.3-35.el7                      bases             172 k
 libwayland-client                         x86_64          1.15.0-1.el7                      bases              33 k
 libwayland-server                         x86_64          1.15.0-1.el7                      bases              39 k
 libxcb                                    x86_64          1.13-1.el7                        bases             214 k
 libxshmfence                              x86_64          1.2-1.el7                         bases             7.2 k
 libxslt                                   x86_64          1.1.28-6.el7                      bases             242 k
 lksctp-tools                              x86_64          1.0.17-2.el7                      bases              88 k
 log4j                                     noarch          1.2.17-16.el7_4                   bases             444 k
 maven-wagon                               noarch          2.4-3.el7                         bases             187 k
 mesa-libEGL                               x86_64          18.3.4-12.el7_9                   updates          110 k
 mesa-libGL                                x86_64          18.3.4-12.el7_9                   updates          166 k
 mesa-libgbm                               x86_64          18.3.4-12.el7_9                   updates           39 k
 mesa-libglapi                             x86_64          18.3.4-12.el7_9                   updates           46 k
 nekohtml                                  noarch          1.9.14-13.el7                     bases             152 k
 objectweb-asm                             noarch          3.3.1-9.el7                       bases             197 k
 pango                                     x86_64          1.42.4-4.el7_7                    bases             280 k
 pcsc-lite-libs                            x86_64          1.8.8-8.el7                       bases              34 k
 pixman                                    x86_64          0.34.0-1.el7                      bases             248 k
 plexus-cipher                             noarch          1.7-5.el7                         bases              22 k
 plexus-classworlds                        noarch          2.4.2-8.el7                       bases              54 k
 plexus-component-api                      noarch          1.0-0.16.alpha15.el7              bases              27 k
 plexus-containers-component-annotations   noarch          1.5.5-14.el7                      bases              11 k
 plexus-containers-container-default       noarch          1.5.5-14.el7                      bases             183 k
 plexus-interactivity                      noarch          1.0-0.14.alpha6.el7               bases              17 k
 plexus-interpolation                      noarch          1.15-8.el7                        bases              57 k
 plexus-sec-dispatcher                     noarch          1.4-13.el7                        bases              29 k
 plexus-utils                              noarch          3.0.9-9.el7                       bases             225 k
 python-javapackages                       noarch          3.4.1-11.el7                      bases              31 k
 python-lxml                               x86_64          3.2.1-4.el7                       bases             758 k
 qdox                                      noarch          1.12.1-10.el7                     bases             170 k
 regexp                                    noarch          1.5-13.el7                        bases              47 k
 sisu-inject-bean                          noarch          2.3.0-11.el7                      bases             181 k
 sisu-inject-plexus                        noarch          2.3.0-11.el7                      bases             179 k
 slf4j                                     noarch          1.7.4-4.el7_4                     bases             170 k
 tomcat-servlet-3.0-api                    noarch          7.0.76-16.el7_9                   updates          214 k
 ttmkfdir                                  x86_64          3.0.9-42.el7                      bases              48 k
 tzdata-java                               noarch          2021a-1.el7                       updates          191 k
 xalan-j2                                  noarch          2.7.1-23.el7                      bases             1.9 M
 xbean                                     noarch          3.13-6.el7                        bases             376 k
 xerces-j2                                 noarch          2.11.0-17.el7_0                   bases             1.1 M
 xml-commons-apis                          noarch          1.4.01-16.el7                     bases             227 k
 xml-commons-resolver                      noarch          1.2-15.el7                        bases             108 k
 xorg-x11-font-utils                       x86_64          1:7.5-21.el7                      bases             104 k
 xorg-x11-fonts-Type1                      noarch          7.5-9.el7                         bases             521 k

Transaction Summary
====================================================================================================================
Install  1 Package (+130 Dependent packages)

Total download size: 72 M
Installed size: 214 M
Is this ok [y/d/N]: y
...
Complete!
```

На данный момент последняя версия maven равна 3.8.1, но с этой версией сборка Atlas'а прерывалась ошибками. Поэтому выбираем версию 3.6.3. Скачиваем и устанавливаем переменные:
```
curl -LO https://archive.apache.org/dist/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz  sudo tar -xvf apache-maven-3.8.1-bin.tar.gz -C /opt

sudo ln -s /opt/apache-maven-3.6.3 /opt/maven

cat << EOF | sudo tee /etc/profile.d/maven.sh
# Apache Maven Environmental Variables
# MAVEN_HOME for Maven 1 - M2_HOME for Maven 2
export JAVA_HOME=/usr/lib/jvm/jre-openjdk
export M2_HOME=/opt/maven
export MAVEN_HOME=/opt/maven
export PATH=\${M2_HOME}/bin:\${PATH}
EOF

sudo chmod +x /etc/profile.d/maven.sh

source /etc/profile.d/maven.sh

#
```


Проверяем версию Maven'а:
```
$ mvn --version

Apache Maven 3.6.3 (cecedd343002696d0abb50b32b541b8a6ba2883f)
Maven home: /opt/maven
Java version: 1.8.0_292, vendor: Red Hat, Inc., runtime: /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.292.b10-1.el7_9.x86_64/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "3.10.0-1160.36.2.el7.x86_64", arch: "amd64", family: "unix"
```

## 3. Скачивание исходников

Сайт проекта находится здесь https://atlas.apache.org/. Находим и скачиваем исходники Apache Atlas:
```
mkdir ~/src && cd ~/src
curl -LO https://mirror.linux-ia64.org/apache/atlas/2.1.0/apache-atlas-2.1.0-sources.tar.gz
tar -xvf apache-atlas-2.1.0-sources.tar.gz
cd ~/src/apache-atlas-sources-2.1.0
```

## 4. Подготовка к сборке

В файл 'pom.xml', в раздел 'repository', добавляем клаудеровский репозиторий с maven-артефактами:
```
        <repository>
            <id>cloudera</id>
            <url>https://repository.cloudera.com/artifactory/cloudera-repos</url>
            <releases>
                <enabled>true</enabled>
            </releases>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
```

В этом же файле находим и меняем версии продуктов:
```
<hadoop.version>3.0.0-cdh6.3.2</hadoop.version>
<hbase.version>2.1.0-cdh6.3.2</hbase.version>
<hive.version>2.1.1-cdh6.3.2</hive.version>
<kafka.scala.binary.version>2.11</kafka.scala.binary.version>
<kafka.version>2.2.1-cdh6.3.2</kafka.version>
<lucene-solr.version>7.4.0</lucene-solr.version>
<solr.version>7.4.0-cdh6.3.2</solr.version>
<sqoop.version>1.4.7-cdh6.3.2</sqoop.version>
<zookeeper.version>3.4.5-cdh6.3.2</zookeeper.version>
```

В файле `~/src/apache-atlas-sources-2.1.0/addons/hive-bridge/src/main/java/org/apache/atlas/hive/bridge/HiveMetaStoreBridge.java` переходим на 577 строку, комментируем "String catalogName = hiveDB.getCatalogName() != null ? hiveDB.getCatalogName().toLowerCase() : null;" и добавляем "String catalogName = null;":
```
    public static String getDatabaseName(Database hiveDB) {
        String dbName      = hiveDB.getName().toLowerCase();
        /*String catalogName = hiveDB.getCatalogName() != null ? hiveDB.getCatalogName().toLowerCase() : null;*/
        String catalogName = null;

        if (StringUtils.isNotEmpty(catalogName) && !StringUtils.equals(catalogName, DEFAULT_METASTORE_CATALOG)) {
            dbName = catalogName + SEP + dbName;
        }

        return dbName;
    }
```

В файле `~/src/apache-atlas-sources-2.1.0/addons/hive-bridge/src/main/java/org/apache/atlas/hive/hook/AtlasHiveHookContext.java` переходим на 81 строку "this.metastoreHandler = (listenerEvent != null) ? metastoreEvent.getIHMSHandler() : null;", комментируем её и добавляем "this.metastoreHandler = null;":
```
    public AtlasHiveHookContext(HiveHook hook, HiveOperation hiveOperation, HookContext hiveContext, HiveHookObjectNamesCache knownObjects,
                                HiveMetastoreHook metastoreHook, ListenerEvent listenerEvent) throws Exception {
        this.hook             = hook;
        this.hiveOperation    = hiveOperation;
        this.hiveContext      = hiveContext;
        this.hive             = hiveContext != null ? Hive.get(hiveContext.getConf()) : null;
        this.knownObjects     = knownObjects;
        this.metastoreHook    = metastoreHook;
        this.metastoreEvent   = listenerEvent;
        /*this.metastoreHandler = (listenerEvent != null) ? metastoreEvent.getIHMSHandler() : null;*/
        this.metastoreHandler = null;


        init();
    }
```

## 5. Сборка Apache Atlas
Переходим в каталог с исходниками и собираем пакеты командой (X — debug, T — количество потоков):
```
cd ~/src/apache-atlas-sources-2.1.0
mvn clean -DskipTests package -Pdist -X -T 4
...
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary:
[INFO]
[INFO] Apache Atlas Server Build Tools 1.0 ................ SUCCESS [  0.673 s]
[INFO] apache-atlas 2.1.0 ................................. SUCCESS [  8.714 s]
[INFO] Apache Atlas Test Utility Tools 2.1.0 .............. SUCCESS [ 43.160 s]
[INFO] Apache Atlas Integration 2.1.0 ..................... SUCCESS [ 16.049 s]
[INFO] Apache Atlas Common 2.1.0 .......................... SUCCESS [ 10.399 s]
[INFO] Apache Atlas Client 2.1.0 .......................... SUCCESS [  2.532 s]
[INFO] atlas-client-common 2.1.0 .......................... SUCCESS [  5.335 s]
[INFO] atlas-client-v1 2.1.0 .............................. SUCCESS [  4.889 s]
[INFO] Apache Atlas Server API 2.1.0 ...................... SUCCESS [  2.784 s]
[INFO] Apache Atlas Notification 2.1.0 .................... SUCCESS [ 39.912 s]
[INFO] atlas-client-v2 2.1.0 .............................. SUCCESS [  3.519 s]
[INFO] Apache Atlas Graph Database Projects 2.1.0 ......... SUCCESS [  2.214 s]
[INFO] Apache Atlas Graph Database API 2.1.0 .............. SUCCESS [  4.018 s]
[INFO] Graph Database Common Code 2.1.0 ................... SUCCESS [  5.220 s]
[INFO] Apache Atlas JanusGraph-HBase2 Module 2.1.0 ........ SUCCESS [  9.821 s]
[INFO] Apache Atlas JanusGraph DB Impl 2.1.0 .............. SUCCESS [01:55 min]
[INFO] Apache Atlas Graph DB Dependencies 2.1.0 ........... SUCCESS [  1.697 s]
[INFO] Apache Atlas Authorization 2.1.0 ................... SUCCESS [  8.347 s]
[INFO] Apache Atlas Repository 2.1.0 ...................... SUCCESS [ 57.224 s]
[INFO] Apache Atlas UI 2.1.0 .............................. SUCCESS [01:16 min]
[INFO] Apache Atlas New UI 2.1.0 .......................... SUCCESS [01:16 min]
[INFO] Apache Atlas Web Application 2.1.0 ................. SUCCESS [01:52 min]
[INFO] Apache Atlas Documentation 2.1.0 ................... SUCCESS [  5.600 s]
[INFO] Apache Atlas FileSystem Model 2.1.0 ................ SUCCESS [  3.727 s]
[INFO] Apache Atlas Plugin Classloader 2.1.0 .............. SUCCESS [  4.856 s]
[INFO] Apache Atlas Hive Bridge Shim 2.1.0 ................ SUCCESS [ 13.029 s]
[INFO] Apache Atlas Hive Bridge 2.1.0 ..................... SUCCESS [ 11.428 s]
[INFO] Apache Atlas Falcon Bridge Shim 2.1.0 .............. SUCCESS [  9.584 s]
[INFO] Apache Atlas Falcon Bridge 2.1.0 ................... SUCCESS [  7.693 s]
[INFO] Apache Atlas Sqoop Bridge Shim 2.1.0 ............... SUCCESS [01:32 min]
[INFO] Apache Atlas Sqoop Bridge 2.1.0 .................... SUCCESS [ 12.755 s]
[INFO] Apache Atlas Storm Bridge Shim 2.1.0 ............... SUCCESS [ 29.657 s]
[INFO] Apache Atlas Storm Bridge 2.1.0 .................... SUCCESS [ 10.355 s]
[INFO] Apache Atlas Hbase Bridge Shim 2.1.0 ............... SUCCESS [  7.741 s]
[INFO] Apache Atlas Hbase Bridge 2.1.0 .................... SUCCESS [ 14.665 s]
[INFO] Apache HBase - Testing Util 2.1.0 .................. SUCCESS [ 23.073 s]
[INFO] Apache Atlas Kafka Bridge 2.1.0 .................... SUCCESS [ 22.792 s]
[INFO] Apache Atlas classification updater 2.1.0 .......... SUCCESS [  6.567 s]
[INFO] Apache Atlas Impala Hook API 2.1.0 ................. SUCCESS [  2.240 s]
[INFO] Apache Atlas Impala Bridge Shim 2.1.0 .............. SUCCESS [  1.660 s]
[INFO] Apache Atlas Impala Bridge 2.1.0 ................... SUCCESS [ 13.788 s]
[INFO] Apache Atlas Distribution 2.1.0 .................... SUCCESS [ 57.715 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  07:00 min (Wall Clock)
[INFO] Finished at: 2021-08-07T14:21:40+03:00
[INFO] ------------------------------------------------------------------------
```

Собранные пакеты находятся в каталоге `~/src/apache-atlas-sources-2.1.0/distro/target`.

## 6. Использованные источники
1. [环境篇：Atlas2.1.0兼容CDH6.3.2部署](https://www.cnblogs.com/ttzzyy/p/14143508.html)
