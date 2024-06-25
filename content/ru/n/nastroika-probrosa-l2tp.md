---
title: "Настройка проброса L2TP через NAT"
date: 2011-04-01
weight: 10
description: >
  Настройка проброса L2TP через NAT
tags:
  - L2TP
  - NAT
aliases:
  - /note/nastroyka-probrosa-l2tp
  - /n/nastroika-probrosa-l2tp/
---

<p>L2TP сервер слушает на портах udp 1701 и udp 500.</p>
<div>L2TP не обеспечивает шифрования трафика. Для шифрования используется IPsec (протокол #50).</div>
<div>Есть проблема при прохождении IPsec через NAT. Для его решения используется протокол NAT-T (порт udp 4500). В заголовок пакета IPsec вставляется заголовок обычного udp. Таким образом, протокол IPsec инкапсулирован в протокол UDP и обрабатывается в сети, как обычный UDP, что обеспечивает ему прохождение через барьеры.</div>
<div>Есть проблема с клиентами Microsoft. На Windows XP надо проделать процедуру, чтобы IPsec пошёл через NAT.</div>
<div>Итого:</div>
<div>На клиенте - windows xp - надо создать параметр DWORD<br />HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\IPsec\AssumeUDPEncapsulationContextOnSendRule<br />и установить его значение в 2. Перегрузить компутер. Для удобства добавления, я приложил reg-файлик с этим параметром.</div>
<div><br />На шлюзе перед сервером в iptables создаём несколько правил для прохождения пакетов от windows xp к windows 2003 и обратно (eth1 - wan интерфейс):</div>
<div>
<pre>iptables -A FORWARD -d 192.168.0.1/32 -i eth1 -p udp -m udp --dport 500 -j ACCEPT
iptables -A FORWARD -s 192.168.0.1/32 -o eth1 -p udp -m udp --sport 500 -j ACCEPT
iptables -A FORWARD -d 192.168.0.1/32 -i eth1 -p udp -m udp --dport 4500 -j ACCEPT
iptables -A FORWARD -s 192.168.0.1/32 -o eth1 -p udp -m udp --sport 4500 -j ACCEPT
</pre>
</div>
<div>
<pre>iptables -t nat -A PREROUTING -d 88.88.88.88/32 -p udp -m udp --dport 500 -j DNAT --to-destination 192.168.0.1
iptables -t nat -A PREROUTING -d 88.88.88.88/32 -p udp -m udp --dport 4500 -j DNAT --to-destination 192.168.0.1
iptables -t nat -A POSTROUTING -d 192.168.0.1/32 -p udp -m udp --dport 500 -j SNAT --to-source 88.88.88.88
iptables -t nat -A POSTROUTING -d 192.168.0.1/32 -p udp -m udp --dport 4500 -j SNAT --to-source 88.88.88.88</pre>
<div>Если на шлюзе стоит биллинг Traffpro, то заносим эти правила в /etc/traffpro/traffpro_rule.cfg. При запуске биллинга очищается цепочка forward в таблице filter. Добавляются лишь правила из вышеуказанного файла.</div>
<div> </div>
<div>Естественно, не забываем настроить на принимающем windows 2003 server L2TP и шлюз по умолчанию!</div>
<div>Пока вот так работает. Не понимаю почему работает. Надо почитать литературу и сделать виртуальную сетку с чистыми машинами, попробовать ещё раз и посмотреть снифферами чего на узлах происходит.</div>
<h2>Посмотрел TCPDump'ом:</h2>
<div>Создание и разрыв L2TP</div>
<div>192.168.0.110 - внешний интерфейс шлюза</div>
<div>192.168.56.102 - Windows 2003 Standart R2 с поднятым L2TP.</div>
<div>isakmp - порт 500</div>
<div>Порт 1701 не используется на шлюзе.</div>
<div>
<div>16:12:12.065083 IP 192.168.0.110.isakmp &gt; 192.168.56.102.isakmp: isakmp: phase 1 I ident</div>
<div>16:12:12.067619 IP 192.168.56.102.isakmp &gt; 192.168.0.110.isakmp: isakmp: phase 1 R ident</div>
<div>16:12:12.082883 IP 192.168.0.110.isakmp &gt; 192.168.56.102.isakmp: isakmp: phase 1 I ident</div>
<div>16:12:12.105346 IP 192.168.56.102.isakmp &gt; 192.168.0.110.isakmp: isakmp: phase 1 R ident</div>
<div>16:12:12.110944 IP 192.168.0.110.4500 &gt; 192.168.56.102.4500: NONESP-encap: isakmp: phase 1 I ident[E]</div>
<div>16:12:12.112232 IP 192.168.56.102.4500 &gt; 192.168.0.110.4500: NONESP-encap: isakmp: phase 1 R ident[E]</div>
<div>16:12:12.112899 IP 192.168.0.110.4500 &gt; 192.168.56.102.4500: NONESP-encap: isakmp: phase 2/others I oakley-quick[E]</div>
<div>16:12:12.113696 IP 192.168.56.102.4500 &gt; 192.168.0.110.4500: NONESP-encap: isakmp: phase 2/others R oakley-quick[EC]</div>
<div>16:12:12.113958 IP 192.168.0.110.4500 &gt; 192.168.56.102.4500: NONESP-encap: isakmp: phase 2/others I oakley-quick[EC]</div>
<div>16:12:12.114603 IP 192.168.56.102.4500 &gt; 192.168.0.110.4500: NONESP-encap: isakmp: phase 2/others R oakley-quick[EC]</div>
<div>16:12:12.115357 IP 192.168.0.110.4500 &gt; 192.168.56.102.4500: UDP-encap: ESP(spi=0xf8ef9e0a,seq=0x1), length 148</div>
<div>16:12:12.116170 IP 192.168.56.102.4500 &gt; 192.168.0.110.4500: UDP-encap: ESP(spi=0x90dd0da3,seq=0x1), length 148</div>
<div>.........................................................</div>
<div>16:12:12.170299 IP 192.168.0.110.4500 &gt; 192.168.56.102.4500: UDP-encap: ESP(spi=0xf8ef9e0a,seq=0x15), length 188</div>
<div>16:12:16.170445 IP 192.168.0.110.4500 &gt; 192.168.56.102.4500: UDP-encap: ESP(spi=0xf8ef9e0a,seq=0x16), length 188</div>
<div>16:12:17.579110 IP 192.168.56.102.4500 &gt; 192.168.0.110.4500: isakmp-nat-keep-alive</div>
<div>16:12:25.122813 IP 192.168.0.110.4500 &gt; 192.168.56.102.4500: UDP-encap: ESP(spi=0xf8ef9e0a,seq=0x17), length 68</div>
<div>16:12:25.123463 IP 192.168.56.102.4500 &gt; 192.168.0.110.4500: UDP-encap: ESP(spi=0x90dd0da3,seq=0x15), length 68</div>
<div>16:12:25.174438 IP 192.168.0.110.4500 &gt; 192.168.56.102.4500: UDP-encap: ESP(spi=0xf8ef9e0a,seq=0x18), length 76</div>
<div>16:12:25.175869 IP 192.168.56.102.4500 &gt; 192.168.0.110.4500: UDP-encap: ESP(spi=0x90dd0da3,seq=0x16), length 52</div>
<div>16:12:25.176367 IP 192.168.0.110.4500 &gt; 192.168.56.102.4500: UDP-encap: ESP(spi=0xf8ef9e0a,seq=0x19), length 76</div>
<div>16:12:25.177835 IP 192.168.56.102.4500 &gt; 192.168.0.110.4500: UDP-encap: ESP(spi=0x90dd0da3,seq=0x17), length 52</div>
<div>16:12:25.180234 IP 192.168.0.110.4500 &gt; 192.168.56.102.4500: NONESP-encap: isakmp: phase 2/others I inf[E]</div>
<div>16:12:25.181419 IP 192.168.56.102.4500 &gt; 192.168.0.110.4500: NONESP-encap: isakmp: phase 2/others R inf[E]</div>
<div>16:12:25.182303 IP 192.168.0.110.4500 &gt; 192.168.56.102.4500: NONESP-encap: isakmp: phase 2/others I inf[E]</div>
<div>16:12:25.182883 IP 192.168.56.102.4500 &gt; 192.168.0.110.4500: NONESP-encap: isakmp: phase 2/others R inf[E]</div>
</div>
</div>
<div>
<h2><a id="NAT traversal and IPsec" name="NAT traversal and IPsec"></a>NAT traversal and IPsec</h2>
<p><a href="http://en.wikipedia.org/wiki/NAT_traversal">http://en.wikipedia.org/wiki/NAT_traversal</a></p>
<p>In order for <a title="IPsec" href="http://en.wikipedia.org/wiki/IPsec">IPsec</a> to work through a NAT, the following protocols need to be allowed on the firewall:</p>
<ul>
<li>Internet Key Exchange (IKE) - User Datagram Protocol (UDP) port 500</li>
<li>Encapsulating Security Payload (ESP) - IP protocol number 50</li>
</ul>
<p>or, in case of NAT-T:</p>
<ul>
<li>IPsec NAT-T - UDP port 4500</li>
</ul>
<p>Often this is accomplished on home routers by enabling "IPsec Passthrough".</p>
<p>The default behavior of Windows XP SP2 was changed to no longer have NAT-T enabled by default, because of a rare and controversial security issue. This prevents most home users from using IPsec without making adjustments to their computer configuration. To enable NAT-T for systems behind NATs to communicate with other systems behind NATs, the following registry key needs to be added and set to a value of 2:HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\IPsec\AssumeUDPEncapsulationContextOnSendRule</p>
<p>IPsec NAT-T patches are also available for Windows 2000, Windows NT and Windows 98.</p>
<p>One usage of NAT-T and IPsec is to enable <a title="Opportunistic encryption" href="http://en.wikipedia.org/wiki/Opportunistic_encryption">opportunistic encryption</a> between systems. NAT-T allows systems behind NATs to request and establish secure connections on demand.</p>
</div>
<p> </p>
