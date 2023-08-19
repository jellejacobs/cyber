# Lab 1: Week 1

## Recap DNS:

**Nslookup** 

```nslookup```

```nslookup <URL-to-question> [DNS-Server IP-address or name]```

**Dig**

```dig +short NS ugent.be```µ

**Reverse Lookup**

```dig +short -x 172.167.12.1```


## Capture traffic using the CLI

First on the router run the following command: ```sudo systemctl restart nftables```

Thereafter install tcpdump on the router: ```sudo dnf install tcpdump```

Check interfaces on tcpdump: `tcpdump -D`

- Which interface on the router will be taken to capture traffic from Empl1 to the internet?

enp0s3

- Which interface on the router will be taken to capture traffic from Empl1 to Empl2?

enp0s10

- How to capture the data in a file?

using the otpion -w on tcpdump and then copie that file to a mounting pount and then shared folder to your own host.

- How to start tcpdump and filter ssh traffic out?

```tcpdump -i enp0s8 port not 22```

- How to filter out http traffic from only the webser

```tcpdump -i any -s 0 'tcp port http' src host 172.30.42.2```

## Homework

### Part 1

Installing a new kali VM. 

The following netw adapters:

* NAT
* Host-Only (the same as used by the other vm's)

### Part 3

- How can you let ping the kali to your web machine?

By setting a static ip to the kali and as def gateway the ip address of the router.

- How to check the users of a vm?

```cat /etc/passwd```

- Who is the dns-server?

the DC.

- How to check wheter the dns server is vulnerable for a dns zone transfer attack?

```console
 dig AXFR DOMAIN-NAME @NAME_SERVER
```

If we check the output of the last command we can see that zone transfer is already disabled. (https://support.cpanel.net/hc/en-us/articles/1500005258382-How-To-Disable-Zone-Transfers-AXFR-On-My-Server-)

To make sure that he will not be vulnerable edit the named.cnf file by adding the next line:

```console
allow-transfer {"none";};
```

This can be done by opening servermanager on the client. Add 'dc' to manage. Then right click on the forward lookup zone of the domain en click properties. Now you can allow of disallow zone transfers.