# Lab7: week 7:

## SSH Port Forwarding and Endpoint Security – Log forwarding

A central logging system can be crucial in an IT department. SOC, SIEM and SOAR are one of the many definitions gaining popularity in the security teams. Give these acronyms a quick Google search.

 

You should have a working ELK-stack. Make sure to review the network settings of this virtual machine:

Give this machine 1 NIC and 1 IP address in the server subnet.

`Connect adapter 1 with intet "servers".`

Make sure this machine has internet access and can connect to all other machines in the network

To let this work we have to change our /etc/sysconfig/network-scripts/ifcfg-enp0s3 file.

By adding of changing the following lines:

```console
BOOTPROTO="none"
GATEWAY=172.30.42.62
IPADDR=172.30.42.5
PREFIX=26
DOMAIN="insecure.cyb"
DNS1="172.30.42.4"
PEERDNS="no"
DNS1=172.30.42.4
```

Then the following commands: 

`nmcli connection reload`
`nmcli connection down enp0s3`
`nmcli connection up enp0s3`

check by `ip a`

After that we have to edit al the IP's in the config files of kibana and elasticsearch:

/etc/elasticsearch/elasticsearch.yml: initial masternode and network.host we have to edit, then restart systemctl elasticsearch
/etc/kibana/kibana.yml: also edit the ips

Your host machine will not be able to contact this machine directly. Because we want to browse the kibana website we need to set something up.

## SSH Port Forwarding

Being able to forward ports over SSH is a very important skill to have in cybersecurity both for red, purple and blue team activities. Figure out a way to perform SSH port forwarding to open a port (we suggest the same) on the router that connects back to the ELK virtual machine. In other words when browsing to the host-only network ip of the router (from your host) you should visit the kibana website on the ELK machine. You do not have to make this persistent!

Questions:

Why is this an interesting approach from a security pointof-view?

So it stays intern and you now have access to an GUI

When would you use local port forwarding? 

`local pc to a server`

![pic1](/images/foto7.1.png)

`to access remote sources that i can't access`
`internal remote database = RDP`

When would you use remote port forwarding?

![pic2](/images/foto7.2.png)

`ssh -R 8888:172.30.42.5:5601 pieter@192.168.56.1`

`I want people to access local resources that they don't have access to`

`example: Local Web Server`

Which of the two are more “common” in security?

`Local port forwarding since corona`

Some people call SSH port forwarding also a “poor man’s VPN”. Why?

`SSH is application level and VPN protects all of your internet data by controlling tranport layer`

You can find some visual guides here:

https://unix.stackexchange.com/questions/115897/whats-ssh-port-forwarding-and-whats-the-difference-between-ssh-local-and-remot
https://iximiuz.com/en/posts/ssh-tunnels/
...

We are going to do it by entering the following command on router:

`ssh -f -N -L 192.168.56.254:5601:172.30.42.5:5601 vagrant@172.30.42.5`

Then: `sudo systemctl restart nftables`

Now you have access to the kibana page on your host by searching the following url: `http://192.168.56.254:5601/`

## Endpoint logs

Now that we have a working ELK-stack, we want to push logs from all systems into Elastic. You do not have to configure all systems to do this in this exercise.

Start by configuring the Windows client to log extra useful information. The sysinternals suite has a nice tool called Sysmon to pump a default Windows installation. Have a look and download Sysmon on the windows client: https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon

Sysmon uses a configuration file to tune and tweak the logs. Use the xml found here: https://github.com/SwiftOnSecurity/sysmon-config Have a quick look at the different RuleGroups and includes/excludes written in the xml-file.

 

Finally push the logs to the ELK stack using a service called winlogbeat. You will need to configure the winlogbeat.yml file accordingly. You can find the instructions in your own ELK-stack. Go to "add integrations" and search for "winlogbeat" and select "Windows events". When everything is set up you should be able to open "mspaint" or "notepad.exe" on your client and a log should be visible in windows event viewer. By default (sysmon) this should be stored in "Applications and Services logs > Microsoft > Windows > Sysmon > Operational". These logs should be the ones send to ELK and visible in Kibana. Make sure this works!

 

TIP: Use logstash if needed to push the logs if sending it directly to elasticsearch fails. The following page can help: https://www.elastic.co/guide/en/beats/winlogbeat/master/logstash-output.html

 

 

 

Now try the same exercise on your router! Install filebeat to send the suricata log file to your ELK-stack.


Start with downloading sysmon and also the config file and place them in the same directory: 

then cmd: `sysmon.exe -accepteula -i sysmonconfig-export.xml`


Daarna installeren we winlogbeat via het kibana dashboard staat alles uitgelegd.

`.\winlogbeat.exe setup`

pas ook de winlogbeat.yml aan naar de juiste ip adressen.

en daarna logstash downloaden om een pipeline op te zetten van client naar kibana







 

