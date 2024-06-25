---
title: "OpenVPN — это просто"
date: 2020-03-22
weight: 10
description: >
  Здесь описана простейшая схема с одним сервером и одним клиентом. Тем не менее используются открытые ключи, работа сервера с правами непривилегированного пользователя и перенос демона openvpn после запуска в песочницу. Копируем код целыми блоками в оболочку bash и выполняем.
tags:
  - OpenVPN
  - VPN
  - Ubuntu
slug: openvpn-eto-prosto
aliases:
  - /a/openvpn-eto-prosto/
---

2020-03-22

Читая сейчас [Reference manual for OpenVPN 2.4](https://openvpn.net/community-resources/reference-manual-for-openvpn-2-4/), я постарался привести свои слегка устаревшие знания по настройке OpenVPN-сети к современным реалиям. В настоящей инструкции, вместо RSA-сертификатов используется криптография на эллиптических кривых. Для шифрования канала выбран протокол с добавочной информацией, позволяющей контролировать целостность пакетов. `tls-auth`+`key-direction` заменены на `tls-crypt`. CBC -> GCM. SHA1 -> SHA512, dh -> ecdh-curve.

## Введение
OpenVPN прекрасный продукт для подключения... (потом допишу).

Для использования OpenVPN сети в минимальной конфигурации необходимо всего три вещи:
1. Центр авторизации.
2. Один OpenVPN-сервер.
3. Один OpenVPN-клиент.

Здесь приведена простая конфигурация для выхода *клиента* в интернет через VPN-сервер. Стороннему наблюдателю будут видны зашифрованные udp-пакеты, которыми обменивается клиент со случайного порта выше 1024-го на порт UDP 9216 удалённого хоста. То есть враг будет знать сам факт использования VPN, но без соответствующих ключей и сертификатов не сможет наблюдать содержимое зашифрованного VPN-туннеля. Маскировка OpenVPN-трафика под HTTPS-протокол реализуется пакетом `obfsproxy`, но о нём я напишу в другой статье.

## 1. Центр Сертификации
**Центр Сертификации** звучит очень сложно. На самом деле никакого дополнительного софта компилировать и собирать не надо. Создадим все необходимые для работы OpenVPN-сети файлы с помощью скриптиков из пакета [Easy-RSA](/a/perevod-easy-rsa-3-quickstart-readme). Будет достаточно разместить EasyRSA в отдельной папке в домашнем каталоге, или, например, на флэшке. В самом крайнем случае, если нет нужды в постоянной или эпизодической работе по созданию новых ключей и сертификатов, то можно всё проделать прямо на OpenVPN-сервере, не забыв, после, надёжно удалить каталог.

**Для создания ключей и сертификатов достаточно прав обычного пользователя.**

### 1.1 Развёртывание "Центра Сертификации"
#### 1.1.1. Клонирование easy-rsa
В выделенном для *CA* каталоге клонируем текущий Easy-RSA и подготавливаем файл с переменными, изменяющими настройки генерации ключей по умолчанию:
```
sudo apt-get install git
git clone https://github.com/OpenVPN/easy-rsa.git
cd easy-rsa/easyrsa3
cp vars.example vars
```

#### 1.1.2. Установка переменных vars
Редактируем файл `vars` для усиления криптографии создаваемых сертификатов (`EASYRSA_ALGO ec`, `EASYRSA_CURVE secp384r1`, `EASYRSA_DIGEST "sha512"`) и увеличения срока действия выданных сертификатов до 1095 дней (`EASYRSA_CERT_EXPIRE 1095`):
```
export DAYSEXP=1095; \
sed -i -e 's/^#set_var EASYRSA_ALGO.*$/set_var EASYRSA_ALGO ec/' vars; \
sed -i -e 's/^#set_var EASYRSA_CURVE.*$/set_var EASYRSA_CURVE secp384r1/' vars; \
sed -i -e 's/^#set_var EASYRSA_DIGEST.*$/set_var EASYRSA_DIGEST "sha512"/' vars; \
sed -i -e "s/^#set_var EASYRSA_CERT_EXPIRE.*$/set_var EASYRSA_CERT_EXPIRE $DAYSEXP/" vars
```

#### 1.1.3. Первичная инициализация PKI
Для создания *Центра Сертификации* выполняем несколько команд:
```
./easyrsa init-pki; \
./easyrsa --batch build-ca nopass
```

#### 1.1.4. Разделяемый секрет ta.key
Будет удобно если здесь же создадим разделяемый секрет для сервера и клиента. Если следовать мануалу OpenVPN, то разделяемый секрет правильно генерировать командой:
```
openvpn --genkey --secret pki/ta.key
```

Если пакет `openvpn` не установлен, то я пользуюсь менее безопасным способом:
```
echo '-----BEGIN OpenVPN Static key V1-----' > pki/ta.key; \
head -c 256 /dev/urandom | hexdump -e '16/1 "%02x" "\n"' >> pki/ta.key; \
echo '-----END OpenVPN Static key V1-----' >> pki/ta.key
```

#### 1.1.5. Что дальше
Переходим к созданию файлов конфигураций, ключей и сертификатов. Имена для новых сертификатов выбираем произвольное. Совпадение с именем хоста не обязательно. Для серверного сертификата выберем имя `srv1`, а для клиентского сертификата `cln1`.

### 1.2. Создание файла конфигурации для OpenVPN-сервера
Для нашего первого OpenVPN-сервера генерируем беспарольный ключ и сертификат командой `easyrsa build-server-full`. Комплект сгенерированных файлов для соответствующего сервера копируем в отдельный каталог `kits/srv1` и создаём **единственный файл конфигурации `srv1.conf` со встроенными ключём, сертификатом и разделяемым секретом**:
```
export DN=srv1; \
export DNEXPDATE=$DN"-"$(date -u +%Y%m%d -d "+$(grep '^set_var EASYRSA_CERT_EXPIRE' vars|awk '{print $3}') days")-$(date -u +%H%M); \
export DIR=kits/$DN; export FILECONF=$DIR/$DN.conf; \
./easyrsa build-server-full $DNEXPDATE nopass; \
mkdir -p kits/$DN; \
cp pki/private/$DNEXPDATE.key kits/$DN; \
cp pki/issued/$DNEXPDATE.crt kits/$DN; \
cp pki/ca.crt kits/$DN; \
cp pki/ta.key kits/$DN; \
echo -e ';local #.#.#.#\n;multihome' > $FILECONF; \
echo -e 'port 9216\nproto udp\ndev tun200' >> $FILECONF; \
echo -e '\nserver 192.168.200.0 255.255.255.0\ntopology subnet' >> $FILECONF; \
echo -e 'push "redirect-gateway def1 bypass-dhcp"' >> $FILECONF; \
echo -e 'push "dhcp-option DNS 1.1.1.1"' >> $FILECONF; \
echo -e '\ncipher AES-256-GCM\necdh-curve secp384r1\ndh none\nkeepalive 10 60' >> $FILECONF; \
echo -e '\nmax-clients 10\nfloat\n;tun-mtu 1300\n;fragment 1300\n;mssfix' >> $FILECONF; \
echo -e '\nchroot jail\nuser nobody\ngroup nogroup\npersist-key\npersist-tun' >> $FILECONF; \
echo -e "\n;ifconfig-pool-persist $DN-ipp.txt\nstatus $DN-status.log" >> $FILECONF; \
echo -e "log $DN.log\n;verb 3\nmute 20\n" >> $FILECONF; \
echo -e "# Expired date $DNEXPDATE UTC" >> $FILECONF; \
echo '<cert>' >> $FILECONF; sed -n '/BEGIN/,$p' $DIR/$DNEXPDATE.crt  >> $FILECONF; echo '</cert>' >> $FILECONF; \
echo '<key>' >> $FILECONF; cat $DIR/$DNEXPDATE.key  >> $FILECONF; echo '</key>' >> $FILECONF; \
echo '<ca>' >> $FILECONF; cat pki/ca.crt >> $FILECONF; echo '</ca>' >> $FILECONF; \
echo '<tls-crypt>' >> $FILECONF; sed -n '/BEGIN/,$p' pki/ta.key >> $FILECONF; echo '</tls-crypt>' >> $FILECONF
```

### 1.3. Создание файла конфигурации для OpenVPN-клиента
Для генерации клиентских ключей предназначена команда `easyrsa build-client-full nopass`. Для первого клиента генерируем беспарольный ключ и сертификат, с копированием набора файлов в каталог `kits/cln1`. Здесь же создаём **файл конфигурации `cln1.ovpn` со встроенными ключём, сертификатом и разделяемым секретом для подключения к серверу**. Будет удобно скопировать только этот файл в OpenVPN клиент.

- Обязательно меняем переменную VPNHOST на актуальный ip-адрес или доменное имя, с портом VPN-сервера, не трогая кавычек.
- Изменив значение переменной REQPASS на отличное от **nopass**, получим запрос ввода пароля для клиентского ключа. Иногда бывает полезно, чтобы *клиент*, при попытке подключиться к серверу, запрашивал пароль от ключа.

```
export DN=cln1; \
export REQPASS=nopass; \
export VPNHOST="203.0.113.1 9216"; \
export DIR=kits/$DN; export FILECONF=$DIR/$DN.ovpn; \
export DNEXPDATE=$DN"-"$(date -u +%Y%m%d -d "+$(grep '^set_var EASYRSA_CERT_EXPIRE' vars|awk '{print $3}') days")-$(date -u +%H%M); \
./easyrsa build-client-full $DNEXPDATE $REQPASS; \
mkdir -p $DIR; \
cp pki/private/$DNEXPDATE.key $DIR; \
cp pki/issued/$DNEXPDATE.crt $DIR; \
cp pki/ca.crt $DIR; \
cp pki/ta.key $DIR; \
echo -e 'client\ndev tun\nproto udp' > $FILECONF; \
echo -e 'auth SHA512' >> $FILECONF; \
echo -e "remote $VPNHOST\nresolv-retry infinite" >> $FILECONF; \
echo -e 'nobind\npersist-key\npersist-tun' >> $FILECONF; \
echo -e 'mute-replay-warnings\nremote-cert-tls server' >> $FILECONF; \
echo -e 'verb 1\nmute 20' >> $FILECONF; \
echo -e "# Expired date $DNEXPDATE UTC" >> $FILECONF; \
echo '<cert>' >> $FILECONF; sed -n '/BEGIN/,$p' $DIR/$DNEXPDATE.crt  >> $FILECONF; echo '</cert>' >> $FILECONF; \
echo '<key>' >> $FILECONF; cat $DIR/$DNEXPDATE.key  >> $FILECONF; echo '</key>' >> $FILECONF; \
echo '<ca>' >> $FILECONF; cat pki/ca.crt >> $FILECONF; echo '</ca>' >> $FILECONF; \
echo '<tls-crypt>' >> $FILECONF; sed -n '/BEGIN/,$p' pki/ta.key >> $FILECONF; echo '</tls-crypt>' >> $FILECONF
```

## 2. Настройка клиентов OpenVPN
### 2.1. Настройка OpenVPN-клиента Windows
После установки [OpenVPN клиента для Windows](https://openvpn.net/community-downloads/), в каталог `C:/Program Files/Openvpn/Config` копируем файл `kits/cln1/cln1.ovpn`. На рабочем столе находим и запускаем GUI OpenVPN. В области уведомлений на панели задач рядом с часами, должен появиться значок OpenVPN. Правой мышкой на нём вызываем меню и выбираем "Подключение". Естественно, делаем это после настройки сервера. Проверил, что работает.

### 2.2. Настройка Android
Устанавливаем [официальный клиент OpenVPN в Google Store](https://play.google.com/store/apps/details?id=net.openvpn.openvpn). С текущими настройками OpenVPN-клиент работает. Проверял на Android 5.1, 7.1.

### 2.3. Настройка "Яблока"
[Официальный клиент для Apple в Apple Store](https://apps.apple.com/us/app/openvpn-connect/id590379981#?platform=iphone). Сейчас под рукой нет эппла. Я помню, что настройка клиента не вызвала затруднений. Подсовываем клиенту файл `cln1.ovpn` и всё работает. При случае проверю и напишу.

### 2.4. Настройка Gnome NetworkManager
Устанавливаем пакет `NetworkManager-openvpn`, например для Ubuntu командой `sudo apt-get install NetworkManager-openvpn`, или в Void Linux команда `sudo xbps-install -Su NetworkManager-openvpn`.

В настройках NetworkManager'а "VPN Connections -> Configure VPN -> + -> Import a saved VPN connection..."", импортируем файл `cln1.ovpn`. При импорте, встроенные в файл конфигурации ключи и сертификаты экспортируются в отдельные файлы `.pem` в каталоге `~/.cert`. Проверил, что работает.

## 3. Создание OpenVPN-сервера
Предполагается, что в [AWS Lightsail](https://aws.amazon.com/ru/lightsail/), [DigitalOcean](https://www.digitalocean.com/), [Vultr](https://www.vultr.com/), [PulseHeberg](https://pulseheberg.com/)... был арендован VPS за несколько долларов, и с Ubuntu 18.04 на борту. Демон SSH слушает 22-ой порт и **имеет запрет на использование паролей**. Файрвол не настроен. Произведен апгрейд системы:
```
$ sudo apt-get -y update && sudo apt-get -y dist-upgrade && sudo apt-get -y --purge autoremove
```

### 3.1. Конфигурирование демона OpenVPN
#### 3.1.1. Установка openvpn на сервер
Переходим в контекст root'а, устанавливаем пакет `openvpn` и создаём поддерево каталогов:
```
sudo su
apt-get install openvpn
cd /etc/openvpnl \
mkdir -p /etc/openvpn/jail /etc/openvpn/jail/tmp
```

#### 3.1.2. Копирование файла конфигурации из CA на сервер
В каталог `/etc/openvpn` копируем тем или иным способом файл `srv1.conf` для сервера, подготовленный нами ранее в *Центре Сертификации*, и размещённый в папке `kits/srv1`. Например, с помощью `rsync` и при наличии на сервере ssh-демона на порте `22` с `PermitRootLogin yes`, делаем так, естественно изменив первые три переменные на актуальные значения:
```
export FILENAME=kits/srv1/srv1.conf; \
export SSHADDRESS=203.0.113.1; \
export SSHPORT=22; \
rsync -avP -e "ssh -p $SSHPORT" $FILENAME root@$SSHADDRESS:/etc/openvpn/
```

#### 3.1.3. Проверка прав на каталог openvpn
На всякий случай устанавливаем необходимые права и атрубуты для скопированного набора файлов:
```
chown root:root /etc/openvpn/*.conf
chmod 600 /etc/openvpn/*.conf
```

#### 3.1.4. Запуск демона
Включаем и запускаем демон:
```
systemctl enable --now openvpn@srv1
```

Замечу что из-за опций `persist-tun` и `persist-key`, вместо команды `systemctl restart openvpn@srv1`, перезапуск демона выполняется двумя командами:
```
systemctl stop openvpn@srv1 && systemctl start openvpn@srv1
```

### 3.2. Настройка файрвола
Сначала временно, то есть до ближайшей перезагрузки сервера, включаем "форвардинг" и добавляем правила файрвола:
```
sysctl -w net.ipv4.ip_forward=1

export GW=$(ip r |awk '/default/{print $5}')
export IP=$(ip a sh dev ens3 |sed -En 's/.*inet (addr:)?(([0-9]*\.){3}[0-9]*).*/\2/p')
export SSHPORT=22
export VPNPORT=9216; export VPNPROTO=udp

iptables -t nat -F
iptables -t mangle -F
iptables -F
iptables -X
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT
iptables -A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
iptables -A INPUT -m conntrack --ctstate INVALID -j DROP
iptables -A INPUT -i lo -j ACCEPT

iptables -A INPUT -p tcp -m tcp --dport $SSHPORT -m conntrack --ctstate NEW -j ACCEPT
iptables -A INPUT -p $VPNPROTO -m $VPNPROTO --dport $VPNPORT -m conntrack --ctstate NEW -j ACCEPT
iptables -A INPUT -i tun+ -p icmp -m icmp --icmp-type echo-request -m conntrack --ctstate NEW -j ACCEPT
iptables -A FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -m conntrack --ctstate INVALID -j DROP
iptables -A FORWARD -i tun200 -o $GW -j ACCEPT
iptables -A FORWARD -j REJECT --reject-with icmp-net-unreachable
iptables -t nat -A POSTROUTING -s 192.168.200.0/24 -o $GW -j SNAT --to-source $IP
```

В случае возникновения проблем на сервере после применения вышеуказанных команд, просто перегружаем сервер через личный кабинет VPS-провайдера.

## 4. Проверка маршрутизации
Проверяем правильность настроек с помощью ранее настроенного OpenVPN-клиента. Подключаемся к серверу и в любом браузере вводим URL `https://www.google.com/search?q=my+ip+address`. Гугл ответит адресом ip. Если адрес совпадает с адресом OpenVPN-сервера, то настройка сервера верна и переходим к закреплению текущей конфигурации сервера на постоянной основе.
