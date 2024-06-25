---
title: "Добавление в NXRM единого репозитория для всех версий PostgreSQL"
date: 2020-10-18
weight: 10
description: >
  Добавление в Nexus Repository Manager единого репозитория для всех версий PostgreSQL
tags:
  - Nexus Repository Manager
  - NXRM
  - PostgreSQL
---

2020-10-18

Добавляем отдельный блоб для хранения пакетов PostgreSQL.

Добавляем репозиторий
Remote storage: https://download.postgresql.org/pub/repos/yum/


Содержимое файла pgdg-redhat-all.repo для деплоя на машины:
```
#######################################################
# PGDG Red Hat Enterprise Linux / CentOS repositories #
#######################################################

# PGDG Red Hat Enterprise Linux / CentOS stable common repository for all PostgreSQL versions

[pgdg-common]
name=PostgreSQL common RPMs for RHEL/CentOS $releasever - $basearch
baseurl=http://nxrm.example.org:8081/repository/PostgreSQL/common/redhat/rhel-$releasever-$basearch
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-PGDG-AARCH64

# PGDG Red Hat Enterprise Linux / CentOS stable repositories:

[pgdg13]
name=PostgreSQL 13 for RHEL/CentOS $releasever - $basearch
baseurl=http://nxrm.example.org:8081/repository/PostgreSQL/13/redhat/rhel-$releasever-$basearch
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-PGDG-AARCH64

[pgdg12]
name=PostgreSQL 12 for RHEL/CentOS $releasever - $basearch
baseurl=http://nxrm.example.org:8081/repository/PostgreSQL/12/redhat/rhel-$releasever-$basearch
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-PGDG-AARCH64

[pgdg11]
name=PostgreSQL 11 for RHEL/CentOS $releasever - $basearch
baseurl=http://nxrm.example.org:8081/repository/PostgreSQL/11/redhat/rhel-$releasever-$basearch
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-PGDG-AARCH64

[pgdg10]
name=PostgreSQL 10 for RHEL/CentOS $releasever - $basearch
baseurl=http://nxrm.example.org:8081/repository/PostgreSQL/10/redhat/rhel-$releasever-$basearch
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-PGDG-AARCH64

[pgdg96]
name=PostgreSQL 9.6 for RHEL/CentOS $releasever - $basearch
baseurl=http://nxrm.example.org:8081/repository/PostgreSQL/9.6/redhat/rhel-$releasever-$basearch
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-PGDG-AARCH64

[pgdg95]
name=PostgreSQL 9.5 for RHEL/CentOS $releasever - $basearch
baseurl=http://nxrm.example.org:8081/repository/PostgreSQL/9.5/redhat/rhel-$releasever-$basearch
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-PGDG-AARCH64

[pgdg94]
name=PostgreSQL 9.4 for RHEL/CentOS $releasever - $basearch
baseurl=http://nxrm.example.org:8081/repository/PostgreSQL/9.4/redhat/rhel-$releasever-$basearch
enabled=0
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-PGDG-AARCH64

# PGDG RHEL/CentOS Updates Testing common repositories.

[pgdg-common-testing]
name=PostgreSQL common testing RPMs for RHEL/CentOS $releasever - $basearch
baseurl=http://nxrm.example.org:8081/repository/PostgreSQL/testing/common/redhat/rhel-$releasever-$basearch
enabled=0
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-PGDG-AARCH64

# PGDG RHEL/CentOS Updates Testing repositories. (These packages should not be used in production)
# Available for 9.6 and above.

[pgdg13-updates-testing]
name=PostgreSQL 13 for RHEL/CentOS $releasever - $basearch - Updates testing
baseurl=http://nxrm.example.org:8081/repository/PostgreSQL/testing/13/redhat/rhel-$releasever-$basearch
enabled=0
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-PGDG-AARCH64

[pgdg12-updates-testing]
name=PostgreSQL 12 for RHEL/CentOS $releasever - $basearch - Updates testing
baseurl=http://nxrm.example.org:8081/repository/PostgreSQL/testing/12/redhat/rhel-$releasever-$basearch
enabled=0
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-PGDG-AARCH64

[pgdg11-updates-testing]
name=PostgreSQL 11 for RHEL/CentOS $releasever - $basearch - Updates testing
baseurl=http://nxrm.example.org:8081/repository/PostgreSQL/testing/11/redhat/rhel-$releasever-$basearch
enabled=0
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-PGDG-AARCH64

# PGDG Red Hat Enterprise Linux / CentOS SRPM testing common repository

[pgdg-source-common]
name=PostgreSQL 12 for RHEL/CentOS $releasever - $basearch - Source
baseurl=http://nxrm.example.org:8081/repository/PostgreSQL/srpms/common/redhat/rhel-$releasever-$basearch
enabled=0
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-PGDG-AARCH64

# PGDG RHEL / CentOS testing common SRPM repository for all PostgreSQL versions

[pgdg-common-srpm-testing]
name=PostgreSQL common testing SRPMs for RHEL/CentOS $releasever - $basearch
baseurl=http://nxrm.example.org:8081/repository/PostgreSQL/srpms/testing/common/redhat/rhel-$releasever-$basearch
enabled=0
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-PGDG-AARCH64

# PGDG Source RPMs (SRPM), and their testing repositories:

[pgdg13-source-updates-testing]
name=PostgreSQL 13 for RHEL/CentOS $releasever - $basearch - Source updates testing
failovermethod=priority
baseurl=http://nxrm.example.org:8081/repository/PostgreSQL/srpms/testing/13/redhat/rhel-$releasever-$basearch
enabled=0
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-PGDG-AARCH64

[pgdg12-source]
name=PostgreSQL 12 for RHEL/CentOS $releasever - $basearch - Source
failovermethod=priority
baseurl=http://nxrm.example.org:8081/repository/PostgreSQL/srpms/12/redhat/rhel-$releasever-$basearch
enabled=0
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-PGDG-AARCH64

[pgdg12-source-updates-testing]
name=PostgreSQL 12 for RHEL/CentOS $releasever - $basearch - Source update testing
failovermethod=priority
baseurl=http://nxrm.example.org:8081/repository/PostgreSQL/srpms/testing/12/redhat/rhel-$releasever-$basearch
enabled=0
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-PGDG-AARCH64

[pgdg11-source]
name=PostgreSQL 11 for RHEL/CentOS $releasever - $basearch - Source
failovermethod=priority
baseurl=http://nxrm.example.org:8081/repository/PostgreSQL/srpms/11/redhat/rhel-$releasever-$basearch
enabled=0
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-PGDG-AARCH64

[pgdg11-source-updates-testing]
name=PostgreSQL 11 for RHEL/CentOS $releasever - $basearch - Source update testing
failovermethod=priority
baseurl=http://nxrm.example.org:8081/repository/PostgreSQL/srpms/testing/11/redhat/rhel-$releasever-$basearch
enabled=0
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-PGDG-AARCH64

[pgdg10-source]
name=PostgreSQL 10 for RHEL/CentOS $releasever - $basearch - Source
failovermethod=priority
baseurl=http://nxrm.example.org:8081/repository/PostgreSQL/srpms/10/redhat/rhel-$releasever-$basearch
enabled=0
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-PGDG-AARCH64

[pgdg96-source]
name=PostgreSQL 9.6 for RHEL/CentOS $releasever - $basearch - Source
failovermethod=priority
baseurl=http://nxrm.example.org:8081/repository/PostgreSQL/srpms/9.6/redhat/rhel-$releasever-$basearch
enabled=0
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-PGDG-AARCH64

[pgdg95-source]
name=PostgreSQL 9.5 for RHEL/CentOS $releasever - $basearch - Source
failovermethod=priority
baseurl=http://nxrm.example.org:8081/repository/PostgreSQL/srpms/9.5/redhat/rhel-$releasever-$basearch
enabled=0
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-PGDG-AARCH64

[pgdg94-source]
name=PostgreSQL 9.4 for RHEL/CentOS $releasever - $basearch - Source
failovermethod=priority
baseurl=http://nxrm.example.org:8081/repository/PostgreSQL/srpms/9.4/redhat/rhel-$releasever-$basearch
enabled=0
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-PGDG-AARCH64

# Debuginfo/debugsource packages for stable repos

[pgdg13-updates-debuginfo]
name=PostgreSQL 13 for RHEL/CentOS $releasever - $basearch - Debuginfo
baseurl=http://nxrm.example.org:8081/repository/PostgreSQL/debug/13/redhat/rhel-$releasever-$basearch
enabled=0
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-PGDG-AARCH64

[pgdg12-updates-debuginfo]
name=PostgreSQL 12 for RHEL/CentOS $releasever - $basearch - Debuginfo
baseurl=http://nxrm.example.org:8081/repository/PostgreSQL/debug/12/redhat/rhel-$releasever-$basearch
enabled=0
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-PGDG-AARCH64

[pgdg11-updates-debuginfo]
name=PostgreSQL 11 for RHEL/CentOS $releasever - $basearch - Debuginfo
baseurl=http://nxrm.example.org:8081/repository/PostgreSQL/debug/11/redhat/rhel-$releasever-$basearch
enabled=0
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-PGDG-AARCH64

[pgdg10-updates-debuginfo]
name=PostgreSQL 10 for RHEL/CentOS $releasever - $basearch - Debuginfo
baseurl=http://nxrm.example.org:8081/repository/PostgreSQL/debug/10/redhat/rhel-$releasever-$basearch
enabled=0
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-PGDG-AARCH64

[pgdg96-updates-debuginfo]
name=PostgreSQL 9.6 for RHEL/CentOS $releasever - $basearch - Debuginfo
baseurl=http://nxrm.example.org:8081/repository/PostgreSQL/debug/9.6/redhat/rhel-$releasever-$basearch
enabled=0
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-PGDG-AARCH64

[pgdg95-updates-debuginfo]
name=PostgreSQL 9.5 for RHEL/CentOS $releasever - $basearch - Debuginfo
baseurl=http://nxrm.example.org:8081/repository/PostgreSQL/debug/9.5/redhat/rhel-$releasever-$basearch
enabled=0
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-PGDG-AARCH64

# Debuginfo/debugsource packages for testing repos
# Available for 9.6 and above.

[pgdg13-updates-testing-debuginfo]
name=PostgreSQL 13 for RHEL/CentOS $releasever - $basearch - Debuginfo
baseurl=http://nxrm.example.org:8081/repository/PostgreSQL/testing/debug/13/redhat/rhel-$releasever-$basearch
enabled=0
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-PGDG-AARCH64

[pgdg12-updates-testing-debuginfo]
name=PostgreSQL 12 for RHEL/CentOS $releasever - $basearch - Debuginfo
baseurl=http://nxrm.example.org:8081/repository/PostgreSQL/testing/debug/12/redhat/rhel-$releasever-$basearch
enabled=0
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-PGDG-AARCH64

[pgdg11-updates-testing-debuginfo]
name=PostgreSQL 11 for RHEL/CentOS $releasever - $basearch - Debuginfo
baseurl=http://nxrm.example.org:8081/repository/PostgreSQL/testing/debug/11/redhat/rhel-$releasever-$basearch
enabled=0
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-PGDG-AARCH64
```
