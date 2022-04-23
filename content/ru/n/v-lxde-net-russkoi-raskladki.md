---
title: "В LXDE нет русской раскладки"
date: 
weight: 10
description: >
  В LXDE нет русской раскладки
tags:
  - LXDE
---

<p>Устал от тормозов гнома и кде. Поставил lxde. Порадовался скорости, но огорчился при попытке переключиться на русскую раскладку.</p>
<div>Обнаружил <a href="http://smartnik.livejournal.com/189931.html">http://smartnik.livejournal.com/189931.html</a>:</div>
<div><em>Добиться русской раскладки клавиатуры в lxde графическими средствами у меня не получилось, поэтому запускаем</em>
<p><em>setxkbmap -option grp:switch,grp:alt_shift_toggle us,ru</em></p>
<p>После этого клавиатура переключается по Alt+Shift (это поведение можно менять, меняя alt_shift_toggle на то, что вам нравится) между русской и английской (естественно, вместо us,ru можно написать те варианты, которые вам нужны). Чтобы запускать эту команду каждый раз при запуске lxde, нужно добавить в конец файла</p>
<p>/etc/xdg/lxsession/LXDE/autostart</p>
<p>строчку</p>
<p>@setxkbmap -option grp:switch,grp:alt_shift_toggle,grp_led:scroll us,ru</p>
</div>
<p> </p>
