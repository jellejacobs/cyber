# Week 10: Lab 10

## IPsec vs OpenVPN

sec is a security standard developed to work on the network layer (L3). You create a tunnel in L3, and direct certain traffic for specific L3 network through it. It has to deal with many issues that were not foreseen in it's development, like nodes being behind a NAT or firewalls that are not allowing IPsec traffic. It therefore is a solution that is mainly used between routers, within networks that are completely controlled by one organisation. End users don't use IPsec very often.

OpenVPN is an open-source alternative that implements the same functionality, but in the transport layer. The main idea is to set up an SSL tunnel (like in HTTPs, or in SSH), but then to not only send one application through this tunnel - but an entire network. It becomes a tunnel for specific L3 network traffic through an L4 connection. 

IPsec: Layer3 network layer

== tunnel waar specefiek data doorkomt; Veel problemen zoals firewalls die iPSec niet doorlaten

OpenVPN: Layer4 transport layer

== Ook tunnels maar op transportlaag bv SSH tunnel maar dan niet voor 1 applicatie maar voor gehel network

## OpenVPN - practical installation

router: `https://vitux.com/how-to-install-openvpn-on-almalinux-8-centos-8-or-rocky-linux-8/`

--> deze is de beste om te volgen!!!!!

debian: `https://bobcares.com/blog/install-openvpn-client-debian/`


Om te kijken als het lukt om met je windows eigen pc te verbinden moet je de ovpn file en certificates kopieren naar je pc en uploaden op je openvpn app.