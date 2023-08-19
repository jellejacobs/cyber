# Lab 2 : Week 2

## Kali virtual machine aka “red”

De kali laten pingen naar al de andere machines lukt wanneer we deze ip: 192.168.56.2 geven en als def gateway het ip van de router.
Maar wanneer we de nat inschakelen kunnen we niet meer pingen naar de andere vm's, dus nat uitgeschakeld laten. En als dns moeten we het IP van de DC instellen.

## The “insecure” host-only network

In this environment, the host-only network that was created in lecture 1, can be seen as an “insecure” network. Assume hackers can create machines and do whatever they want in this network. The kali virtual machine symbolizes this metaphor. In a real life setting the host-only network could be a “public guest Wi-FI” network for example.

From our kali vm we have to perform red-team attacks:

* `http://www.insecure.cyb/` is reachable

* `http://www.insecure.cyb/cmd` is also reachable, then test out the application: "executing the commands work but the pinging part not"

* Default nmap scan: Bijvoorbeeld voor Emplyee1: `nmap -sS 172.30.128.12` this works also, `nmap -sV 172.30.42.2` `nmap -sV 172.30.42.3` `nmap -sV  172.30.42.4`

* Enumerate the most interesting ports by issuing a service enumeration scan (banner grab scan):
  * What database software is running on the db machine? Sudo nmap -F -T4 –script banner IPADDRESS-DB-MACHINE
  * Nmap script to brute force the data-base: `hydra -l root -P /usr/share/wordlists/rockyou.txt.gz 172.30.42.3 mysql`
    * Login root and passwoord 'password'
  * `Sudo nmap -F -T4 –script banner IPADDRESS-DB-MACHINE`
  * DC scanning doesn't show domain name
  * -sC option means --script=default dat extra informatie geeft over het gevonden services.
* SSh into other machine from Kali works also
* Smdbclient (works perfect): first you have to log on as a vagrant user on the cli by using `sudo su -l vagrant`
  * ```console
    smbclient \\\\172.30.42.4\\C$
    smb: \> dir
    $RECYCLE.BIN                      DHS        0  Fri Aug 12 11:18:44 2022
    $WinREAgent                        DH        0  Fri Aug 12 10:11:21 2022
    chef                                D        0  Fri Aug 12 10:05:01 2022
    Documents and Settings          DHSrn        0  Fri Aug 12 12:03:38 2022
    DumpStack.log.tmp                 AHS    12288  Mon Jan  9 02:46:32 2023
    Financial                           D        0  Tue Sep 27 10:20:30 2022
    opscode                             D        0  Fri Aug 12 10:05:28 2022
    pagefile.sys                      AHS 1073741824  Mon Jan  9 02:46:32 2023
    PerfLogs                            D        0  Sat May  8 04:15:05 2021
    Program Files                      DR        0  Fri Aug 12 10:08:18 2022
    Program Files (x86)                 D        0  Sat May  8 05:34:13 2021
    ProgramData                       DHn        0  Fri Aug 12 10:07:56 2022
    Recovery                         DHSn        0  Tue Sep 27 10:09:51 2022
    System Volume Information         DHS        0  Tue Sep 27 11:21:23 2022
    Users                              DR        0  Tue Sep 27 12:05:50 2022
    vagrant                            Dr        0  Tue Sep 27 10:11:43 2022
    Windows                             D        0  Tue Sep 27 12:06:22 2022
    wip                                 D        0  Tue Sep 27 12:20:17 2022

                33131007 blocks of size 4096. 30642443 blocks available
    smb: \> 
    ```


As you can see, a hacker on this host-only network, has no restrictions to interact with the other machines. This is not a best-practice! A way to resolve this, is using network segmentation. By dividing the network in several segments and properly configuring the access to and from these subnets you can reduce the attack vector a lot.

* Therefor we can see that there is not yet network segmentation done.

* What machines should be in the DMZ? --> all the servers


At least now we have to make sure that the kali doesn't has access anymore to all of the machines. It may only can browse to the 'http://www.insecure.cyb' and all of the derivates to make this possible.

How to fix that?

Any service provided to users on the public internet should be placed in the DMZ network.

## nftables
- We maken een nieuwe file aan met onze volgende regels in. `sudo vi /etc/nftables/rules.nft`
- Daar zetten we de configuratie van hieronder in. Dit laat de dns, http en https services door en blokeerd alle anderen voor forwarding.
- Een de file is aangemaakt gaan we die regels invoeren met `sudo nft -f /etc/nftables/rules.nft`
- Dan zie je bij een scan van kali naar de webserver dat alle poorten die openstonden nu op filtered staan en alleen de poorten van hierboven openstaan {53, 80 & 443}
- Je kan de regels verwijderen met `nft delete chain/table <chain_name/table_name> #if you delete chain you need to specify the table the chain`
`nft delete chain forward`
`nft delete table inet filter`
sh
#!/usr/sbin/nft -f

# Flush the rule set
flush ruleset

table inet filter {
        chain forward {
                type filter hook forward priority 0;

                # laat terugkerende connecties toe
                ct state {established, related} accept;

                # laat http en https plus dns toe
                tcp dport {http, https} accept;
                udp dport {53} accept;

                # alles wordt gedropt buiten de bovenste 2 regels
                policy drop;


        }
}


    



 
