---
title: "Сборка Apache Atlas 2.2.0 для Cloudera CDH 6.3.2"
date: 2021-09-28
weight: 1
description: >
  Описание процесса сборки Apache Atlas 2.2.0 для Cloudera CDH 6.3.2.
tags:
  - BigData
  - Apache Atlas
  - Cloudera CDH 6.3.2
---

> ⚠ Важное сообщение. Сборка проходит успешно только после комментирования строк с константами MATERIALIZED_VIEW. В исходниках предыдущей версии Apache Atlas 2.1.0 такие константы не применялись. MATERIALIZED_VIEW относится к таблицам Hive 3.x версии, тогда как в CDH 6.3.2 используется Hive 2.1.1.

> ⚠ В этой сборке предусмотривается возможность работы Apache Atlas с Elasticsearch 6.8.20, в расчёте на дальнейшее внедрение Amundsen. В мануале по Amundsen указано, что поддерживается работа только с Elasticsearch 6.x.



## 1. Подготовка хоста
Если запуск Apache Atlas'а будем производить на этой же машине, то:
- Хост добавляем в Cloudera-кластер, чтобы при установке Cloudera CDH были корректно добавлены локальные аккаунты и группы.
- В процессе добавления в кластер, перед включением TLS-аутентификации для Cloudera Agent, добавляем хост в домен.
- На хост устанавливаем шлюзы для YARN, HBase, Hive, Kafka, Solr.
- Лучше смотреть соответствующую инструкцию опубликованную здесь же.

### 1.1. Установка Maven
Сначала устанавливаем maven 3.0.5-17.el7 из стандартных репо для CentOS7:
```bash
sudo yum install maven
```

<details>
<summary>Click here to expand...</summary>

```
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

</details>
<br>

На данный момент последняя версия maven равна 3.8.1, но с этой версией сборка Atlas'а 2.1.0 прерывалась ошибками, тогда как Atlas 2.2.0 собирался без ошибок. Поэтому для сборки Atlas'а 2.1.0 выбираем Maven 3.6.3, а для Atlas 2.2.0 выбираем версию Maven 3.8.1.

Скачиваем и устанавливаем переменные:
```bash
MVNVER="3.8.1"
curl -LO https://archive.apache.org/dist/maven/maven-3/${MVNVER}/binaries/apache-maven-${MVNVER}-bin.tar.gz
sudo tar -xvf apache-maven-${MVNVER}-bin.tar.gz -C /opt

sudo ln -s /opt/apache-maven-${MVNVER} /opt/maven

cat << EOF | sudo tee /etc/profile.d/maven.sh
# Apache Maven Environmental Variables
# MAVEN_HOME for Maven 1 - M2_HOME for Maven 2

#export JAVA_HOME=/usr/lib/jvm/jre-openjdk
export JAVA_HOME=usr/java/jdk1.8.0_181-cloudera

export M2_HOME=/opt/maven
export MAVEN_HOME=/opt/maven
export PATH=\${M2_HOME}/bin:\${PATH}
EOF

sudo chmod +x /etc/profile.d/maven.sh

source /etc/profile.d/maven.sh
```

Проверяем версию Maven'а и используемую версию Java:
```bash
mvn --version

Apache Maven 3.8.1 (05c21c65bdfed0f71a2f2ada8b84da59348c4c5d)
Maven home: /opt/maven
Java version: 1.8.0_181, vendor: Oracle Corporation, runtime: /usr/java/jdk1.8.0_181-cloudera/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "3.10.0-1160.36.2.el7.x86_64", arch: "amd64", family: "unix"
```

### 1.2. Установка NodeJS
Добавляем репо:
```bash
USERNAME=nx-pubrepo-r
PASSWORD=xSAR1zZi7dQzs3QBx2vthdyk2qQrXv

cat << EOF | sudo tee /etc/yum.repos.d/nodejs-el7.repo
[nodejs16]
username=$USERNAME
password=$PASSWORD
name=Node.js Packages for Enterprise Linux 7 - \$basearch
baseurl=http://nexus.example.org:8081/repository/all_nodejs_yum/pub_16.x/el/7/\$basearch
failovermethod=priority
enabled=0
gpgcheck=0
EOF
```

Устанавливаем NodeJS:
```bash
sudo yum install -y --enablerepo=nodejs16 nodejs
```

## 2. Подготовка к сборке
### 2.1. Скачивание и распаковка исходников
Сайт проекта находится здесь https://atlas.apache.org/. Находим и скачиваем исходники Apache Atlas:
```bash
mkdir ~/src && cd ~/src
curl -LO https://mirror.linux-ia64.org/apache/atlas/2.2.0/apache-atlas-2.2.0-sources.tar.gz
tar -xvf apache-atlas-2.2.0-sources.tar.gz
cd ~/src/apache-atlas-sources-2.2.0
```

### 2.2. Изменения в файле `pom.xml`
В файл 'pom.xml', в раздел <repository>, добавляем клаудеровский репозиторий с maven-артефактами:
```java
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

В этом же файле находим и меняем версии продуктов на следующие:
```java
<elasticsearch.version>6.8.20</elasticsearch.version>
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

В этом же файле меняем версию артефакта 'atlas-buildtools' cо значения '1.0' на значение '0.8.1', так как артефакт 'atlas-buildtools-1.0' не может быть найден. Но можно попробовать версию  '1.0.0-alpha':
```java
<artifactId>maven-checkstyle-plugin</artifactId>
    <dependencies>
        <dependency>
            <groupId>org.apache.atlas</groupId>
            <artifactId>atlas-buildtools</artifactId>
            <version>1.0.0-alpha</version>
        </dependency>
    </dependencies>
```

### 2.3. Изменения в других файлах
В файле '~/src/apache-atlas-sources-2.2.0/addons/hive-bridge/src/main/java/org/apache/atlas/hive/bridge/HiveMetaStoreBridge.java' переходим на 577 строку, комментируем "String catalogName = hiveDB.getCatalogName() != null ? hiveDB.getCatalogName().toLowerCase() : null;" и добавляем "String catalogName = null;":
```java
public static String getDatabaseName(Database hiveDB) {
    String dbName      = hiveDB.getName().toLowerCase();
    //String catalogName = hiveDB.getCatalogName() != null ? hiveDB.getCatalogName().toLowerCase() : null;
    String catalogName = null;

    if (StringUtils.isNotEmpty(catalogName) && !StringUtils.equals(catalogName, DEFAULT_METASTORE_CATALOG)) {
        dbName = catalogName + SEP + dbName;
    }

    return dbName;
}
```

В файле '~/src/apache-atlas-sources-2.2.0/addons/hive-bridge/src/main/java/org/apache/atlas/hive/hook/AtlasHiveHookContext.java' переходим на 81 строку "this.metastoreHandler = (listenerEvent != null) ? metastoreEvent.getIHMSHandler() : null;", комментируем её и добавляем "this.metastoreHandler = null;":
```java
public AtlasHiveHookContext(HiveHook hook, HiveOperation hiveOperation, HookContext hiveContext, HiveHookObjectNamesCache knownObjects,
                            HiveMetastoreHook metastoreHook, ListenerEvent listenerEvent) throws Exception {
    this.hook             = hook;
    this.hiveOperation    = hiveOperation;
    this.hiveContext      = hiveContext;
    this.hive             = hiveContext != null ? Hive.get(hiveContext.getConf()) : null;
    this.knownObjects     = knownObjects;
    this.metastoreHook    = metastoreHook;
    this.metastoreEvent   = listenerEvent;
    //this.metastoreHandler = (listenerEvent != null) ? metastoreEvent.getIHMSHandler() : null;
    this.metastoreHandler = null;


    init();
}
```

В файле `~/src/apache-atlas-sources-2.2.0/addons/hive-bridge/src/main/java/org/apache/atlas/hive/hook/events/CreateHiveProcess.java` комментируем строку 263 с упоминанием 'MATERIALIZED_VIEW':
```java
private boolean isDdlOperation(AtlasEntity entity) {
    return entity != null && !context.isMetastoreHook()
        && (context.getHiveOperation().equals(HiveOperation.CREATETABLE_AS_SELECT)
         || context.getHiveOperation().equals(HiveOperation.CREATEVIEW)
         || context.getHiveOperation().equals(HiveOperation.ALTERVIEW_AS));
         //|| context.getHiveOperation().equals(HiveOperation.CREATE_MATERIALIZED_VIEW));
}
```

В файле `~/src/apache-atlas-sources-2.2.0/addons/hive-bridge/src/main/java/org/apache/atlas/hive/hook/HiveHook.java` комментируем строки 212 и 217  с упоминанием 'MATERIALIZED_VIEW':
```java
case DROPTABLE:
case DROPVIEW:
//case DROP_MATERIALIZED_VIEW:
    event = new DropTable(context);
break;

case CREATETABLE_AS_SELECT:
//case CREATE_MATERIALIZED_VIEW:
case CREATEVIEW:
case ALTERVIEW_AS:
case LOAD:
case EXPORT:
case IMPORT:
case QUERY:
    event = new CreateHiveProcess(context);
break;
```

Заготовка bash-скрипта для автоматизации изменений в файлах:
```bash
sed 's|<elasticsearch.version>6.8.15|<elasticsearch.version>6.8.20|'  pom.xml
sed 's|<hadoop.version>3.3.0|<hadoop.version>3.0.0-cdh6.3.2|' pom.xml
sed 's|<hbase.version>2.3.3|<hbase.version>2.1.0-cdh6.3.2|' pom.xml
sed 's|<hive.version>3.1.0|<hive.version>2.1.1-cdh6.3.2|' pom.xml
sed 's|<kafka.scala.binary.version>2.12|<kafka.scala.binary.version>2.11|' pom.xml
sed 's|<kafka.version>2.5.0|<kafka.version>2.2.1-cdh6.3.2|' pom.xml
sed 's|<lucene-solr.version>8.6.3|<lucene-solr.version>7.4.0|' pom.xml
sed '|<solr.version>8.6.3|<solr.version>7.4.0-cdh6.3.2|' pom.xml
sed '|<sqoop.version>1.4.6.2.3.99.0-195|<sqoop.version>1.4.7-cdh6.3.2|' pom.xml
sed '|<zookeeper.version>3.5.7|<zookeeper.version>3.4.5-cdh6.3.2|' pom.xml
sed -i '/<artifactId>atlas-buildtools|{n;s/<version>1.0/<version>1.0.0-alpha/}' pom.xml
```

## 3. Сборка Apache Atlas
Переходим в каталог с исходниками и собираем пакеты с сохранением цветного stderr и stdout в файл `install.log` (X — debug, T — количество потоков):
```bash
cd ~/src/apache-atlas-sources-2.2.0
unbuffer mvn clean -DskipTests package -Pdist -X -T 4 2>&1 | tee install.log

# Посмотреть лог-файл в цвете.
less -r install.log
```

<details>
<summary>Click here to expand...</summary>

```
[DEBUG] Adding file-set in: /data/src/apache-atlas-sources-2.2.0_elastic-6.8.20/distro/../addons/hbase-bridge/target/dependency/hook
to archive location: apache-atlas-hbase-hook-2.2.0/hook/
[DEBUG] Cannot find ArtifactResolver with hint: project-cache-aware
org.codehaus.plexus.component.repository.exception.ComponentLookupException: java.util.NoSuchElementException
      role: org.apache.maven.artifact.resolver.ArtifactResolver
  roleHint: project-cache-aware
    at org.codehaus.plexus.DefaultPlexusContainer.lookup (DefaultPlexusContainer.java:267)
    at org.codehaus.plexus.DefaultPlexusContainer.lookup (DefaultPlexusContainer.java:243)
    at org.apache.maven.shared.repository.DefaultRepositoryAssembler.contextualize (DefaultRepositoryAssembler.java:721)
    at org.eclipse.sisu.plexus.PlexusLifecycleManager.contextualize (PlexusLifecycleManager.java:282)
    at org.eclipse.sisu.plexus.PlexusLifecycleManager.activate (PlexusLifecycleManager.java:203)
    at org.eclipse.sisu.bean.BeanScheduler.schedule (BeanScheduler.java:151)
    at org.eclipse.sisu.plexus.PlexusLifecycleManager.manage (PlexusLifecycleManager.java:147)
    at org.eclipse.sisu.plexus.PlexusBeanBinder.afterInjection (PlexusBeanBinder.java:72)
    at com.google.inject.internal.MembersInjectorImpl.notifyListeners (MembersInjectorImpl.java:131)
    at com.google.inject.internal.ConstructorInjector.provision (ConstructorInjector.java:125)
    at com.google.inject.internal.ConstructorInjector.access$000 (ConstructorInjector.java:32)
    at com.google.inject.internal.ConstructorInjector$1.call (ConstructorInjector.java:98)
    at com.google.inject.internal.ProvisionListenerStackCallback$Provision.provision (ProvisionListenerStackCallback.java:112)
    at com.google.inject.internal.ProvisionListenerStackCallback$Provision.provision (ProvisionListenerStackCallback.java:127)
    at com.google.inject.internal.ProvisionListenerStackCallback.provision (ProvisionListenerStackCallback.java:66)
    at com.google.inject.internal.ConstructorInjector.construct (ConstructorInjector.java:93)
    at com.google.inject.internal.ConstructorBindingImpl$Factory.get (ConstructorBindingImpl.java:306)
    at com.google.inject.internal.InjectorImpl$1.get (InjectorImpl.java:1050)
    at com.google.inject.internal.InjectorImpl.getInstance (InjectorImpl.java:1086)
    at org.eclipse.sisu.space.AbstractDeferredClass.get (AbstractDeferredClass.java:48)
    at com.google.inject.internal.ProviderInternalFactory.provision (ProviderInternalFactory.java:85)
    at com.google.inject.internal.InternalFactoryToInitializableAdapter.provision (InternalFactoryToInitializableAdapter.java:57)
    at com.google.inject.internal.ProviderInternalFactory$1.call (ProviderInternalFactory.java:66)
    at com.google.inject.internal.ProvisionListenerStackCallback$Provision.provision (ProvisionListenerStackCallback.java:112)
    at com.google.inject.internal.ProvisionListenerStackCallback$Provision.provision (ProvisionListenerStackCallback.java:127)
    at com.google.inject.internal.ProvisionListenerStackCallback.provision (ProvisionListenerStackCallback.java:66)
    at com.google.inject.internal.ProviderInternalFactory.circularGet (ProviderInternalFactory.java:61)
    at com.google.inject.internal.InternalFactoryToInitializableAdapter.get (InternalFactoryToInitializableAdapter.java:47)
    at com.google.inject.internal.ProviderToInternalFactoryAdapter.get (ProviderToInternalFactoryAdapter.java:40)
    at com.google.inject.internal.SingletonScope$1.get (SingletonScope.java:168)
    at com.google.inject.internal.InternalFactoryToProviderAdapter.get (InternalFactoryToProviderAdapter.java:39)
    at com.google.inject.internal.InjectorImpl$1.get (InjectorImpl.java:1050)
    at org.eclipse.sisu.inject.LazyBeanEntry.getValue (LazyBeanEntry.java:81)
    at org.eclipse.sisu.plexus.LazyPlexusBean.getValue (LazyPlexusBean.java:51)
    at org.eclipse.sisu.plexus.PlexusRequirements$RequirementProvider.get (PlexusRequirements.java:250)
    at org.eclipse.sisu.plexus.ProvidedPropertyBinding.injectProperty (ProvidedPropertyBinding.java:48)
    at org.eclipse.sisu.bean.BeanInjector.injectMembers (BeanInjector.java:52)
    at com.google.inject.internal.MembersInjectorImpl.injectMembers (MembersInjectorImpl.java:160)
    at com.google.inject.internal.ConstructorInjector.provision (ConstructorInjector.java:124)
    at com.google.inject.internal.ConstructorInjector.access$000 (ConstructorInjector.java:32)
    at com.google.inject.internal.ConstructorInjector$1.call (ConstructorInjector.java:98)
    at com.google.inject.internal.ProvisionListenerStackCallback$Provision.provision (ProvisionListenerStackCallback.java:112)
    at com.google.inject.internal.ProvisionListenerStackCallback$Provision.provision (ProvisionListenerStackCallback.java:127)
    at com.google.inject.internal.ProvisionListenerStackCallback.provision (ProvisionListenerStackCallback.java:66)
    at com.google.inject.internal.ConstructorInjector.construct (ConstructorInjector.java:93)
    at com.google.inject.internal.ConstructorBindingImpl$Factory.get (ConstructorBindingImpl.java:306)
    at com.google.inject.internal.InjectorImpl$1.get (InjectorImpl.java:1050)
    at com.google.inject.internal.InjectorImpl.getInstance (InjectorImpl.java:1086)
    at org.eclipse.sisu.space.AbstractDeferredClass.get (AbstractDeferredClass.java:48)
    at com.google.inject.internal.ProviderInternalFactory.provision (ProviderInternalFactory.java:85)
    at com.google.inject.internal.InternalFactoryToInitializableAdapter.provision (InternalFactoryToInitializableAdapter.java:57)
    at com.google.inject.internal.ProviderInternalFactory$1.call (ProviderInternalFactory.java:66)
    at com.google.inject.internal.ProvisionListenerStackCallback$Provision.provision (ProvisionListenerStackCallback.java:112)
    at org.eclipse.sisu.bean.BeanScheduler$CycleActivator.onProvision (BeanScheduler.java:230)
    at com.google.inject.internal.ProvisionListenerStackCallback$Provision.provision (ProvisionListenerStackCallback.java:120)
    at com.google.inject.internal.ProvisionListenerStackCallback.provision (ProvisionListenerStackCallback.java:66)
    at com.google.inject.internal.ProviderInternalFactory.circularGet (ProviderInternalFactory.java:61)
    at com.google.inject.internal.InternalFactoryToInitializableAdapter.get (InternalFactoryToInitializableAdapter.java:47)
    at com.google.inject.internal.ProviderToInternalFactoryAdapter.get (ProviderToInternalFactoryAdapter.java:40)
    at com.google.inject.internal.SingletonScope$1.get (SingletonScope.java:168)
    at com.google.inject.internal.InternalFactoryToProviderAdapter.get (InternalFactoryToProviderAdapter.java:39)
    at com.google.inject.internal.InjectorImpl$1.get (InjectorImpl.java:1050)
    at org.eclipse.sisu.inject.LazyBeanEntry.getValue (LazyBeanEntry.java:81)
    at org.eclipse.sisu.plexus.LazyPlexusBean.getValue (LazyPlexusBean.java:51)
    at org.eclipse.sisu.wire.EntryListAdapter$ValueIterator.next (EntryListAdapter.java:111)
    at org.apache.maven.plugin.assembly.archive.DefaultAssemblyArchiver.createArchive (DefaultAssemblyArchiver.java:179)
    at org.apache.maven.plugin.assembly.mojos.AbstractAssemblyMojo.execute (AbstractAssemblyMojo.java:429)
    at org.apache.maven.plugin.DefaultBuildPluginManager.executeMojo (DefaultBuildPluginManager.java:137)
    at org.apache.maven.lifecycle.internal.MojoExecutor.execute (MojoExecutor.java:210)
    at org.apache.maven.lifecycle.internal.MojoExecutor.execute (MojoExecutor.java:156)
    at org.apache.maven.lifecycle.internal.MojoExecutor.execute (MojoExecutor.java:148)
    at org.apache.maven.lifecycle.internal.LifecycleModuleBuilder.buildProject (LifecycleModuleBuilder.java:117)
    at org.apache.maven.lifecycle.internal.builder.multithreaded.MultiThreadedBuilder$1.call (MultiThreadedBuilder.java:190)
    at org.apache.maven.lifecycle.internal.builder.multithreaded.MultiThreadedBuilder$1.call (MultiThreadedBuilder.java:186)
    at java.util.concurrent.FutureTask.run (FutureTask.java:266)
    at java.util.concurrent.Executors$RunnableAdapter.call (Executors.java:511)
    at java.util.concurrent.FutureTask.run (FutureTask.java:266)
    at java.util.concurrent.ThreadPoolExecutor.runWorker (ThreadPoolExecutor.java:1149)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run (ThreadPoolExecutor.java:624)
    at java.lang.Thread.run (Thread.java:748)
Caused by: java.util.NoSuchElementException
    at java.util.Collections$EmptyIterator.next (Collections.java:4191)
    at org.codehaus.plexus.DefaultPlexusContainer.lookup (DefaultPlexusContainer.java:263)
    at org.codehaus.plexus.DefaultPlexusContainer.lookup (DefaultPlexusContainer.java:243)
    at org.apache.maven.shared.repository.DefaultRepositoryAssembler.contextualize (DefaultRepositoryAssembler.java:721)
    at org.eclipse.sisu.plexus.PlexusLifecycleManager.contextualize (PlexusLifecycleManager.java:282)
    at org.eclipse.sisu.plexus.PlexusLifecycleManager.activate (PlexusLifecycleManager.java:203)
    at org.eclipse.sisu.bean.BeanScheduler.schedule (BeanScheduler.java:151)
    at org.eclipse.sisu.plexus.PlexusLifecycleManager.manage (PlexusLifecycleManager.java:147)
    at org.eclipse.sisu.plexus.PlexusBeanBinder.afterInjection (PlexusBeanBinder.java:72)
    at com.google.inject.internal.MembersInjectorImpl.notifyListeners (MembersInjectorImpl.java:131)
    at com.google.inject.internal.ConstructorInjector.provision (ConstructorInjector.java:125)
    at com.google.inject.internal.ConstructorInjector.access$000 (ConstructorInjector.java:32)
    at com.google.inject.internal.ConstructorInjector$1.call (ConstructorInjector.java:98)
    at com.google.inject.internal.ProvisionListenerStackCallback$Provision.provision (ProvisionListenerStackCallback.java:112)
    at com.google.inject.internal.ProvisionListenerStackCallback$Provision.provision (ProvisionListenerStackCallback.java:127)
    at com.google.inject.internal.ProvisionListenerStackCallback.provision (ProvisionListenerStackCallback.java:66)
    at com.google.inject.internal.ConstructorInjector.construct (ConstructorInjector.java:93)
    at com.google.inject.internal.ConstructorBindingImpl$Factory.get (ConstructorBindingImpl.java:306)
    at com.google.inject.internal.InjectorImpl$1.get (InjectorImpl.java:1050)
    at com.google.inject.internal.InjectorImpl.getInstance (InjectorImpl.java:1086)
    at org.eclipse.sisu.space.AbstractDeferredClass.get (AbstractDeferredClass.java:48)
    at com.google.inject.internal.ProviderInternalFactory.provision (ProviderInternalFactory.java:85)
    at com.google.inject.internal.InternalFactoryToInitializableAdapter.provision (InternalFactoryToInitializableAdapter.java:57)
    at com.google.inject.internal.ProviderInternalFactory$1.call (ProviderInternalFactory.java:66)
    at com.google.inject.internal.ProvisionListenerStackCallback$Provision.provision (ProvisionListenerStackCallback.java:112)
    at com.google.inject.internal.ProvisionListenerStackCallback$Provision.provision (ProvisionListenerStackCallback.java:127)
    at com.google.inject.internal.ProvisionListenerStackCallback.provision (ProvisionListenerStackCallback.java:66)
    at com.google.inject.internal.ProviderInternalFactory.circularGet (ProviderInternalFactory.java:61)
    at com.google.inject.internal.InternalFactoryToInitializableAdapter.get (InternalFactoryToInitializableAdapter.java:47)
    at com.google.inject.internal.ProviderToInternalFactoryAdapter.get (ProviderToInternalFactoryAdapter.java:40)
    at com.google.inject.internal.SingletonScope$1.get (SingletonScope.java:168)
    at com.google.inject.internal.InternalFactoryToProviderAdapter.get (InternalFactoryToProviderAdapter.java:39)
    at com.google.inject.internal.InjectorImpl$1.get (InjectorImpl.java:1050)
    at org.eclipse.sisu.inject.LazyBeanEntry.getValue (LazyBeanEntry.java:81)
    at org.eclipse.sisu.plexus.LazyPlexusBean.getValue (LazyPlexusBean.java:51)
    at org.eclipse.sisu.plexus.PlexusRequirements$RequirementProvider.get (PlexusRequirements.java:250)
    at org.eclipse.sisu.plexus.ProvidedPropertyBinding.injectProperty (ProvidedPropertyBinding.java:48)
    at org.eclipse.sisu.bean.BeanInjector.injectMembers (BeanInjector.java:52)
    at com.google.inject.internal.MembersInjectorImpl.injectMembers (MembersInjectorImpl.java:160)
    at com.google.inject.internal.ConstructorInjector.provision (ConstructorInjector.java:124)
    at com.google.inject.internal.ConstructorInjector.access$000 (ConstructorInjector.java:32)
    at com.google.inject.internal.ConstructorInjector$1.call (ConstructorInjector.java:98)
    at com.google.inject.internal.ProvisionListenerStackCallback$Provision.provision (ProvisionListenerStackCallback.java:112)
    at com.google.inject.internal.ProvisionListenerStackCallback$Provision.provision (ProvisionListenerStackCallback.java:127)
    at com.google.inject.internal.ProvisionListenerStackCallback.provision (ProvisionListenerStackCallback.java:66)
    at com.google.inject.internal.ConstructorInjector.construct (ConstructorInjector.java:93)
    at com.google.inject.internal.ConstructorBindingImpl$Factory.get (ConstructorBindingImpl.java:306)
    at com.google.inject.internal.InjectorImpl$1.get (InjectorImpl.java:1050)
    at com.google.inject.internal.InjectorImpl.getInstance (InjectorImpl.java:1086)
    at org.eclipse.sisu.space.AbstractDeferredClass.get (AbstractDeferredClass.java:48)
    at com.google.inject.internal.ProviderInternalFactory.provision (ProviderInternalFactory.java:85)
    at com.google.inject.internal.InternalFactoryToInitializableAdapter.provision (InternalFactoryToInitializableAdapter.java:57)
    at com.google.inject.internal.ProviderInternalFactory$1.call (ProviderInternalFactory.java:66)
    at com.google.inject.internal.ProvisionListenerStackCallback$Provision.provision (ProvisionListenerStackCallback.java:112)
    at org.eclipse.sisu.bean.BeanScheduler$CycleActivator.onProvision (BeanScheduler.java:230)
    at com.google.inject.internal.ProvisionListenerStackCallback$Provision.provision (ProvisionListenerStackCallback.java:120)
    at com.google.inject.internal.ProvisionListenerStackCallback.provision (ProvisionListenerStackCallback.java:66)
    at com.google.inject.internal.ProviderInternalFactory.circularGet (ProviderInternalFactory.java:61)
    at com.google.inject.internal.InternalFactoryToInitializableAdapter.get (InternalFactoryToInitializableAdapter.java:47)
    at com.google.inject.internal.ProviderToInternalFactoryAdapter.get (ProviderToInternalFactoryAdapter.java:40)
    at com.google.inject.internal.SingletonScope$1.get (SingletonScope.java:168)
    at com.google.inject.internal.InternalFactoryToProviderAdapter.get (InternalFactoryToProviderAdapter.java:39)
    at com.google.inject.internal.InjectorImpl$1.get (InjectorImpl.java:1050)
    at org.eclipse.sisu.inject.LazyBeanEntry.getValue (LazyBeanEntry.java:81)
    at org.eclipse.sisu.plexus.LazyPlexusBean.getValue (LazyPlexusBean.java:51)
    at org.eclipse.sisu.wire.EntryListAdapter$ValueIterator.next (EntryListAdapter.java:111)
    at org.apache.maven.plugin.assembly.archive.DefaultAssemblyArchiver.createArchive (DefaultAssemblyArchiver.java:179)
    at org.apache.maven.plugin.assembly.mojos.AbstractAssemblyMojo.execute (AbstractAssemblyMojo.java:429)
    at org.apache.maven.plugin.DefaultBuildPluginManager.executeMojo (DefaultBuildPluginManager.java:137)
    at org.apache.maven.lifecycle.internal.MojoExecutor.execute (MojoExecutor.java:210)
    at org.apache.maven.lifecycle.internal.MojoExecutor.execute (MojoExecutor.java:156)
    at org.apache.maven.lifecycle.internal.MojoExecutor.execute (MojoExecutor.java:148)
    at org.apache.maven.lifecycle.internal.LifecycleModuleBuilder.buildProject (LifecycleModuleBuilder.java:117)
    at org.apache.maven.lifecycle.internal.builder.multithreaded.MultiThreadedBuilder$1.call (MultiThreadedBuilder.java:190)
    at org.apache.maven.lifecycle.internal.builder.multithreaded.MultiThreadedBuilder$1.call (MultiThreadedBuilder.java:186)
    at java.util.concurrent.FutureTask.run (FutureTask.java:266)
    at java.util.concurrent.Executors$RunnableAdapter.call (Executors.java:511)
    at java.util.concurrent.FutureTask.run (FutureTask.java:266)
    at java.util.concurrent.ThreadPoolExecutor.runWorker (ThreadPoolExecutor.java:1149)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run (ThreadPoolExecutor.java:624)
    at java.lang.Thread.run (Thread.java:748)
[DEBUG] No dependency sets specified.
[INFO] Building tar: /data/src/apache-atlas-sources-2.2.0_elastic-6.8.20/distro/target/apache-atlas-2.2.0-hbase-hook.tar.gz
...
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary for apache-atlas 2.2.0:
[INFO]
[INFO] Apache Atlas Server Build Tools .................... SUCCESS [  1.431 s]
[INFO] apache-atlas ....................................... SUCCESS [  9.142 s]
[INFO] Apache Atlas Integration ........................... SUCCESS [ 12.408 s]
[INFO] Apache Atlas Test Utility Tools .................... SUCCESS [  5.650 s]
[INFO] Apache Atlas Common ................................ SUCCESS [  4.045 s]
[INFO] Apache Atlas Client ................................ SUCCESS [  0.938 s]
[INFO] atlas-client-common ................................ SUCCESS [  1.985 s]
[INFO] atlas-client-v1 .................................... SUCCESS [  2.526 s]
[INFO] Apache Atlas Server API ............................ SUCCESS [  2.291 s]
[INFO] Apache Atlas Notification .......................... SUCCESS [  5.322 s]
[INFO] atlas-client-v2 .................................... SUCCESS [  2.052 s]
[INFO] Apache Atlas Graph Database Projects ............... SUCCESS [  0.913 s]
[INFO] Apache Atlas Graph Database API .................... SUCCESS [  2.444 s]
[INFO] Graph Database Common Code ......................... SUCCESS [  2.190 s]
[INFO] Apache Atlas JanusGraph-HBase2 Module .............. SUCCESS [  2.468 s]
[INFO] Apache Atlas JanusGraph DB Impl .................... SUCCESS [ 14.401 s]
[INFO] Apache Atlas Graph DB Dependencies ................. SUCCESS [  2.486 s]
[INFO] Apache Atlas Authorization ......................... SUCCESS [  2.550 s]
[INFO] Apache Atlas Repository ............................ SUCCESS [  9.921 s]
[INFO] Apache Atlas UI .................................... SUCCESS [01:02 min]
[INFO] Apache Atlas New UI ................................ SUCCESS [01:01 min]
[INFO] Apache Atlas Web Application ....................... SUCCESS [01:03 min]
[INFO] Apache Atlas Documentation ......................... SUCCESS [  4.470 s]
[INFO] Apache Atlas FileSystem Model ...................... SUCCESS [  3.347 s]
[INFO] Apache Atlas Plugin Classloader .................... SUCCESS [  2.106 s]
[INFO] Apache Atlas Hive Bridge Shim ...................... SUCCESS [  4.490 s]
[INFO] Apache Atlas Hive Bridge ........................... SUCCESS [  8.053 s]
[INFO] Apache Atlas Falcon Bridge Shim .................... SUCCESS [  3.415 s]
[INFO] Apache Atlas Falcon Bridge ......................... SUCCESS [  4.600 s]
[INFO] Apache Atlas Sqoop Bridge Shim ..................... SUCCESS [  8.715 s]
[INFO] Apache Atlas Sqoop Bridge .......................... SUCCESS [  9.089 s]
[INFO] Apache Atlas Storm Bridge Shim ..................... SUCCESS [  6.424 s]
[INFO] Apache Atlas Storm Bridge .......................... SUCCESS [  5.730 s]
[INFO] Apache Atlas Hbase Bridge Shim ..................... SUCCESS [  4.726 s]
[INFO] Apache Atlas Hbase Bridge .......................... SUCCESS [  7.642 s]
[INFO] Apache HBase - Testing Util ........................ SUCCESS [ 22.674 s]
[INFO] Apache Atlas Kafka Bridge .......................... SUCCESS [  3.511 s]
[INFO] Apache Atlas classification updater ................ SUCCESS [  2.244 s]
[INFO] Apache Atlas index repair tool ..................... SUCCESS [  4.034 s]
[INFO] Apache Atlas Impala Hook API ....................... SUCCESS [  0.872 s]
[INFO] Apache Atlas Impala Bridge Shim .................... SUCCESS [  0.498 s]
[INFO] Apache Atlas Impala Bridge ......................... SUCCESS [  7.789 s]
[INFO] Apache Atlas Distribution .......................... SUCCESS [ 52.828 s]
[INFO] atlas-examples ..................................... SUCCESS [  0.456 s]
[INFO] sample-app ......................................... SUCCESS [  2.279 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  03:36 min (Wall Clock)
[INFO] Finished at: 2021-10-18T18:22:49+03:00
[INFO] ------------------------------------------------------------------------
```

</details>

Собранные пакеты находятся в каталоге `~/src/apache-atlas-sources-2.2.0/distro/target`. В файле `apache-atlas-2.2.0-bin.tar.gz` можно найти всё необходимое для дальнейшей работы по развёртыванию Atlas'а.
