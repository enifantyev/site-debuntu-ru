---
title: "Настройка NUT для IPPON BACK POWER PRO 600"
date: 2011-03-30
weight: 10
description: >
  Настройка NUT для IPPON BACK POWER PRO 600
tags:
  - NUT
  - APC
aliases:
  - /note/nastroyka-nut-dlya-ippon-back-power-pro-600
  - /n/nastroika-nut-dlya-ippon-back-power-pro-600/
---

<p><a href="http://blog.shadypixel.com/monitoring-a-ups-with-nut-on-debian-or-ubuntu-linux/">http://blog.shadypixel.com/monitoring-a-ups-with-nut-on-debian-or-ubuntu-linux/</a></p>
<div>man ups.conf</div>
<div>man upsd.conf</div>
<div>man upsd.users</div>
<div>man upsmon.conf</div>
<div>$ aptitude install nut</div>
<p>Надо проверить наличие юзера и группы с именем nut и, если автоматом не были созданы, создать.</p>
<div>Лезем <a href="http://www.networkupstools.org/compat/stable.html">http://www.networkupstools.org/compat/stable.html</a> и видим что для данной модели UPS можно юзать два дрйвера blazer_ser or megatec.<br />Создаём /etc/nut/ups/ups.conf и пишем внутри:
<div># /etc/nut/ups.conf</div>
<div>
<div>[ippon]</div>
<div>driver = megatec</div>
<div>port = /dev/ttyS0 (так как у меня UPS сидит на COM1)</div>
<div> </div>
<div>Создаём /etc/udev/rules.d/99_nut-serialups.rules:</div>
<div># /etc/udev/rules.d/99_nut-serialups.rules</div>
</div>
<div>
<div>KERNEL=="ttyS0", GROUP="nut"</div>
<p>Даём команды, чтобы не перегружать компутер:</p>
<div>$ sudo udevadm control --reload-rules<br />$ sudo udevadm trigger</div>
</div>
<div> </div>
<div>Запускаем:</div>
<div>$ sudo upsdrvctl start<br />и видим ответ:</div>
<div>
<div>Network UPS Tools - UPS driver controller 2.4.1</div>
<div>Network UPS Tools - Megatec protocol driver 1.6 (2.4.1)</div>
<div>Megatec protocol UPS detected.</div>
</div>
<div>Создаём /etc/nut/upsd.conf:</div>
<div># /etc/nut/upsd.conf</div>
</div>
<div>LISTEN 127.0.0.1 3493</div>
<div> </div>
<div>
<div>Создаём  /etc/nut/upsd.users:</div>
<div># /etc/nut/upsd.users</div>
<div>[local_mon]</div>
<div>password = VasyaLiSiCyn83 (здесь придумываем пароль)</div>
<div>    allowfrom = localhost</div>
<div>    upsmon master</div>
<p>Создаём /etc/nut/upsmon.conf:</p>
</div>
<div># /etc/nut/upsmon.conf</div>
<div>MONITOR ippon@localhost:3493 1 local_mon VasyaLiSiCyn83 master</div>
<div>POWERDOWNFLAG /etc/killpower</div>
<div>SHUTDOWNCMD "/sbin/shutdown -h now" </div>
<div>Настраиваем права:</div>
<div>$ sudo chown root:nut /etc/nut/*<br />$ sudo chmod 640 /etc/nut/*</div>
<p>И в финале создаём и редактируем /etc/default/nut</p>
<div>
<pre># /etc/default/nut
START_UPSD=yes
START_UPSMON=yes 
</pre>
<p>Запускаем:</p>
</div>
<div>$ /etc/init.d/nut start</div>
<div>Можно дать команду:</div>
<div>$ upsc ippon и получить данные UPS'а.</div>
<div>
<div>battery.charge: 100.0</div>
<div>battery.voltage: 13.80</div>
<div>battery.voltage.nominal: 12.0</div>
<div>driver.name: megatec</div>
<div>driver.parameter.pollinterval: 2</div>
<div>driver.parameter.port: /dev/ttyS0</div>
<div>driver.version: 2.4.1</div>
<div>driver.version.internal: 1.6</div>
<div>input.frequency: 50.1</div>
<div>input.frequency.nominal: 50.0</div>
<div>input.voltage: 248.0</div>
<div>input.voltage.fault: 248.0</div>
<div>input.voltage.maximum: 250.0</div>
<div>input.voltage.minimum: 234.0</div>
<div>input.voltage.nominal: 230.0</div>
<div>output.voltage: 248.0</div>
<div>ups.beeper.status: disabled</div>
<div>ups.delay.shutdown: 0</div>
<div>ups.delay.start: 2</div>
<div>ups.load: 31.0</div>
<div>ups.mfr: unknown</div>
<div>ups.model: unknown</div>
<div>ups.serial: unknown</div>
<div>ups.status: OL</div>
<div>ups.temperature: 37.8</div>
<div>ups.type: standby</div>
</div>
<div>Дополнительно скачал и установил два пакета из </div>
<div><a href="http://www.lestat.st/informatique/projets/pynut-en">http://www.lestat.st/informatique/projets/pynut-en</a> и <a href="http://www.lestat.st/informatique/projets/nut-monitor-en">http://www.lestat.st/informatique/projets/nut-monitor-en</a>.</div>
<div>В результате в гноме могу глядеть нагрузку на UPS и прочее через GUI.</div>
