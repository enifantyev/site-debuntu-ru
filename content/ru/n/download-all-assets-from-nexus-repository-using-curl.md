---
title: "Download all assets from Nexus's repository using curl"
date: 2022-05-25
weight: 10
description: >
  Скрипт для автоматизации скачивания всех файлов из репозитория Nexus.
tags:
  - curl
  - Nexus Repository Manager
  - bash
slug: download-all-assets-from-nexus-repository-using-curl
---

На скорую руку приготовил скрипт выкачивающий все файлы из какого-нибудь репо.

```
#!/bin/bash -e

NXRM="https://nexus.example.org:8081"
USERPASS="nx-pub-reader:Paqvl8oldO1EOarTcx8FAjuXZ"
REPO="cloudera-cm-6.3.1-hosted"
FILEEXT="rpm"
FILENAME="${REPO}.list"
TOKEN=""

get_token(){
    TOKEN=""
    TMPTOKEN=""
    TMPTOKEN=$(tail ${FILENAME} | grep "continuationToken")
    TOKEN="$(echo $TMPTOKEN | awk '{print $3}' | awk -F\" '{print $2}')"
    if [[ "$TOKEN" = "" ]]; then return; fi
    TOKENFULL="continuationToken=${TOKEN}&"
    echo $TOKEN
}

> ${FILENAME}
while : ; do
    curl -u${USERPASS} -X 'GET' \
        "${NXRM}/service/rest/v1/assets?${TOKENFULL}repository=${REPO}" \
        -H 'accept: application/json' >> ${FILENAME}
    get_token
    if [[ "$TOKEN" = "" ]]; then break; fi
done

grep -n "downloadUrl" ${REPO}.list | grep ${FILEEXT} | awk '{print $4}' | awk -F\" '{print $2}' > ${REPO}.urls
mkdir -p ${REPO}
cd ${REPO}
while read -r URL; do
    curl -u${USERPASS} -LO -C - "${URL}"
done<../${REPO}.urls
```
