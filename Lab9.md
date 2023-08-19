# Lab 9 : week 9

## Preparation

IPsec in tunnel mode is typically set-up between two routers. Your CSA router can be used as a first endpoint. The second endpoint is a new (debian) VM, that you can download. Once installed in VirtualBox, you shall connect this Debian11IPsec VM to the WAN-side of the CSA router: the Host-Only adapter. Verify that the 2nd NIC is connected to this network. Within this network, you now have your CSA router, your Kali Linux, your host and the Debian VM connected to each other.

import ova file

The DebianVM has a fixed IP-address in this network set to 192.168.56.222/24. Ping both your CSA router and the Kali Linux before you continue.



op router: `sudo ip route add 100.66.66.0/24 dev enp0s8 src 192.168.56.254`

op deb vm: `sudo ip route add 172.30.0.0/16 dev enp0s8 src 192.168.56.222`
`ip route add 172.30.0.0/16 via 192.168.56.254`

test by trying this ping on router:

`ping -I 172.30.128.126 100.66.66.254`

On employee1 do the following commands and they should work:

`ping 100.66.66.66`
`curl 100.66.66.66`

Insecure!!!!! So we are going to configure ipsec.

## Attack!

The WAN network has your Kali Linux in the same network as both routers. We will set-up a man-in-the-middle attack, allowing us to inspect the IP packets that are sent between the CSA router and the DebianVM. Arspoof will redirect traffic to the Kali, Wireshark will allow you to inspect traffic (and see if you have IPsec working, or not).

Arspoof can be done using the Ettercap tool:

`sudo ettercap -Tq -i [interface] -M arp:remote /[leftIP]// /[rightIP]//`

```console
where you have to figure out the following information:

[interface]: the interface you use on your kali linux
[leftIP]: the one side of the connection you try to capture, in casu the CSA router
[rightIP]: the othere side of the connection you try to capture, in casu the DebianVM
```
op de kali:
`sudo ettercap -Tq -i eth1 -M arp:remote /192.168.56.254// /192.168.56.222//`

open dan ook wireshark om man in the middle attack te capturen en onderschep pings van router naar debian:

wireshark op eth1.

Ettercap bereikt in dit scenario een man-in-the-middle-positie a.d.h.v. ARP poisoning. Dit werd doorgegeven aan Ettercap a.d.h.v. met de -M arp:remote parameter. ARP staat voor ARP poisoning, remote voor het sniffen van remote bestemmingen (alles achter een gateway; andere subnets). In dit geval is remote eigenlijk niet nodig omdat de Kali machine op hetzelfde subnet zit als beide doelwittoestellen. Indien het 100.66.66.0/24 adres van de Debian machine gebruikt werd, zou het remote argument wel nodig geweest zijn.

In ARP zal altijd de meest recente ARP reply toegepast worden. Bij ARP poisoning zal het programma daarom constant (maar ook niet té veel) kunstmatige ARP replies versturen op het netwerk om een plek in de doelstellen hun ARP-tabel te kunnen garanderen. Dit zien we ook duidelijk in effect wanneer Ettercap actief is.

Het is onmogelijk om de ARP requests van legitieme toestellen te doen ophouden, enkel om de bedoelde ontvanger ervan telkens hun geldige ARP entry te laten overschrijven met een giftige ARP entry.

## IPsec

### Encryption from your router to the debian

run manualIPsec.sh

cp to router and change in to out


### Encryption from the debian to your router!

generate new key: `dd if=/dev/random count=24 bs=1 | xxd -ps`

create new file but one on the other and the other on the one.


Test this by wireshark and capturing traffic on eth1 and ping .
