# Lab 5: Week 5

## Mysql

The database server runs mysql. The developers have yet to implement the feature on the webserver that requires mysql.The credentials are root/password

On the webserver first install mysql:

`sudo dnf install mysql`

Then connect to the database server by the following command:

`sudo mysql --host=172.30.42.3 --user=root --password=password mysql`

Now you have access into the db a using mysql.


## AD Windows Client

Now the Employee 2 is added to the ad in server manager by adding the dc in the server maanger. All users from the AD are able to login on the client.

## SSH 

All employees are able to ssh into all servers.

## Suricata

### Step1

What system is best to istall IDS or IPS on? An dwhich traffic can be seen then and what traffic will be missed?

We are going to install suricata on the router. So all traffic will be seen. Because the router stands in the middle of all traffic.

## Step 2: Install tcpdump on the router

`sudo yum install tcpdump`

It was already installed.

## Step3: Verifying that you see packets in tcpdump

Verify that we can capture data from the kali to the database:

`sudo tcpdump -i enp0s9` 

`hydra -l root -P /usr/share/wordlists/rockyou.txt.gz 172.30.42.3 mysql`

We can not see pings between database and webserver.

## Step4: Install and configure suricata

`sudo yum  install epel-release`

`sudo yum install suricata`

`sudo systemctl enable suricata.service`

`sudo chown -R suricata:suricata /var/log/suricata`

`sudo chown -R suricata:suricata /etc/suricata`


We have to focus on one interface, we will take interface enp0s9.

## Step 5: Create your own alert rules.

**Edit the  `/etc/suricata/suricata.yaml` file:**

af-packet: interface must be the one you are going to check; "enp0s9"

community-id set true

address-groups: HOME_NET: ["172.30.42.0/26"]

**Edit the  `/etc/sysconfig/suricata` file:**

```console
# IPS (Layer3)
LISTENMODE=nfqueue

# IDS
# OPTIONS="-i enp0s9 --user suricata "

# IDS + IPS with layer 3 filtering (iptables/nftables)
OPTIONS="-q 0 --user suricata "

# IDS + IPS with layer 2 filtering (af-packet)
# OPTIONS="--af-packet --user suricata "
``` 

Maak een eigen rule file in the directory `/var/lib/suricata/rules`, nl `local.rules` met de volgende inhoud:

```console 
# IDS
alert icmp any any -> any any (msg:"Ping in any direction detected"; sid: 50001; rev:1;)
alert tcp any any -> 172.30.42.3 3306 (msg:"MYSQL port entry at database server"; sid:50002; rev:1;)

# IPS
drop icmp any any -> 172.30.42.0/26 any (msg:"Ping towards servers dropped"; sid:60001; rev:1;)
``` 
Voeg daarna je file toe aan de rule-files in suricata.yaml.

Controleer met `suricata -c /etc/suricata/suricata.yaml -i enp0s9`

**Usefull commands:**

```sudo suricata-update list-sources```

```ls -al /var/lib/suricata/rules/```

```ls -al /var/log/suricata```

```sudo suricata -T -c /etc/suricata/suricata.yaml -v``` to test the configuration file



## Questions:

What is the difference between the fast.log and the eve.json files?

`fast.log`: That's where you can find al the intrusion logs.
`eve.json`: same information but just in json format

Create a rule that alerts as soon as a ping is performed between two machines (for example kali and database)

`alert icmp any any -> any any (msg:"Ping in any direction detected"; sid: 50001; rev:1;)`



Test your out of the box configuration and browse on your kali to www.insecure.cyb/cmd and enter “id” as an evil command. Does it trigger an alert? If not are you able to make it trigger an alert?


Create an alert that checks the mysql tcp port and rerun a hydra attack to check this rule. Can you visually see this bruteforce attack in the fast.log file? Tip: monitor the file live with an option of tail.

`alert tcp any any -> 172.30.42.3 3306 (msg:"MYSQL port entry at database server"; sid:50002; rev:1;)`


Go have a look at the suricata documentation. What is the default configuration of suricata, is it an IPS or IDS?

`Suricata runs in IDS mode by default, which means it will not actively block network traffic. To switch to IPS mode, you’ll need to edit Suricata’s /etc/sysconfig/suricata configuration file.`

What do you have to change to the setup to switch to the other (IPS or IDS)?

`Find the OPTIONS="-i eth0 --user suricata" line and comment it out by adding a # to the beginning of the line. Then add a new line OPTIONS="-q 0 -vvv --user suricata" line that tells Suricata to run in IPS mode.`


You are free to experiment more and go all out with variables (for the networks) and rules. Make sure you can conceptually explain why certain rules would be useful and where (= from which subnet to which subnet) they should be applied?

## Poging 2 installeren suricata

- Install suricata `sudo dnf install suricata`
- config file location `/etc/suricata/suricata.yaml`
sh
## set interface to listen to
 af-packet:
   - interface: enp0s9

# Cross platform libpcap capture support
pcap:
  - interface: enp0s9

 address-groups:
     HOME_NET: "[172.30.42.62/26]"


## In /etc/sysconfig/suricata change the interface
# Add options to be passed to the daemon
OPTIONS="-i enp0s9 --user suricata "
- Voor het maken van onze eigen regel kunnen we het volgende toevoegen in de rules `/var/lib/suricata/rules/suricata.rules`
sh
alert icmp any any -> any any (msg:"ping alert on network";)
# er bestaat al een rule in de default voor check van aanvallen op 
- Mysql login attemts worden al gelogd met een alert
- Door de rules aan te passen van alert naar drop ga je packeten tegenhouden en niet alleen een alert maken, hierdoor ga je van IDS naar IPS

```console
sudo suricata -T
sudo suricata-update
```

## Look if the alert from or own local.rules work


First execute the following command:

`suricata -c /etc/suricata/suricata.yaml -i enp0s9`

Then do some things that should trigger an alert:

`For example pinging from kali to webserver`

Then look at the logfiles in /var/log/suricata if you can see some alert that were triggered.

It works!!!

