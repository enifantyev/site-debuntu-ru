---
title: "Сборка deb-пакета свежего EncFS"
date: 2019-10-13
weight: 10
description: >
  Сборка deb-пакета свежего EncFS
tags:
  - EncFS
  - InfoSec
---

2019-10-13

```
# apt-get install checkinstall
```

Скачал свежий EncFS 1.9.5
```
$ sudo screen
# cd /usr/src
# wget https://github.com/vgough/encfs/releases/download/v1.9.5/encfs-1.9.5.tar.gz
# tar -xvf encfs-1.9.5.tar.gz
# cd encfs-1.9.5
```

Установил галку "Source code repositories" в "Update manager"-"Software Sources" для включения `deb-src` и установил зависимости пакета encfs:

```
# apt-get update
# apt-get build-dep encfs
```

<details><p>

```
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following NEW packages will be installed:
  autoconf automake autopoint autotools-dev cmake cmake-data debhelper dh-autoreconf dh-strip-nondeterminism dwz
  libfile-stripnondeterminism-perl libfuse-dev libjsoncpp1 librhash0 libselinux1-dev libsepol1-dev libtinyxml2-6 libtinyxml2-dev libtool
  libuv1 po-debconf
0 upgraded, 21 newly installed, 0 to remove and 1 not upgraded.
Need to get 8 024 kB/8 052 kB of archives.
After this operation, 35,9 MB of additional disk space will be used.
Do you want to continue? [Y/n]

```

</p></details>

Запустил для сборки пакета:

```
# ./create-dev-pkg.sh
```
<details><p>

```
-- The C compiler identification is GNU 7.4.0
-- The CXX compiler identification is GNU 7.4.0
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Found FUSE: /usr/include/fuse
-- Found OpenSSL: /usr/lib/x86_64-linux-gnu/libcrypto.so (found version "1.1.1")
...

...
======================== Installation successful ==========================

Some of the files created by the installation are inside the build
directory: /usr/src/encfs-1.9.5/build

You probably don't want them to be included in the package,
especially if they are inside your home directory.
Do you want me to list them?  [n]:
Should I exclude them from the package? (Saying yes is a good idea)  [y]:
Copying files to the temporary directory...OK
Stripping ELF binaries and libraries...OK
Compressing man pages...OK
Building file list...OK
Building Debian package...OK
NOTE: The package will not be installed
Erasing temporary files...OK
Writing backup package...OK
OK
Deleting temp dir...OK

**********************************************************************

 Done. The new package has been saved to

 /usr/src/encfs-1.9.5/build/encfs_20191011-1_amd64.deb
 You can install it in your system anytime using:

      dpkg -i encfs_20191011-1_amd64.deb

**********************************************************************

```
</p></details>

В папке build нашёл deb-пакет и установил его:
```
# dpkg -i encfs_20191011-1_amd64.deb
```

