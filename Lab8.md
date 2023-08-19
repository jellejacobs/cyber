# Week8: Lab 8

This lab introduces the concept of a honeypot.

As defined by Kaspersky (https://www.kaspersky.com/resource-center/threats/what-is-a-honeypot)  a honeypot in cyber security is typically a system or service that acts like a decoy to lure hackers. It mimics a target for hackers in such a way that evildoers think they found “a way in” only to waste time. The most interesting part of a good honeypot is the logging and monitoring. By observing, a good blue teamer can take actions accordingly (for example ban a specific IP or remove specific commands).

Let’s test this out by creating a SSH honeypot on the router using https://github.com/cowrie/cowrie

You are free to install this tool in any way you see fit. Docker is probably the easiest way to get things up and running but you are free to choose.

 

Steps:

Why is the router, in this environment, an interesting device to configure with a SSH honeypot? What could be a good argument to NOT configure the router with a honeypot service?
Change your current SSH configuration in such a way that the SSH server (daemon) is not listening on port 22 anymore but on 2222.
Install and run the cowrie software on the router and listen on port 22.
Once configured and up and running, verify that you can still SSH to the router normally, using port 2222.
Attack your router and try to SSH normally. What do you notice?
Do you get a shell?
Are your commands logged? Is the IP address of the SSH client logged?
Can an attacker perform malicious things?
Are the actions logged to a file? Which file?
 

If you want to experiment more with honeypots, have a look at the following list for inspiration: https://github.com/paralax/awesome-honeypots

You don’t have to implement them but have a look and answer the following questions:

What type of honeypot is “honeyup”?
What is the idea behind “opencanary”?


1. Edit the config file `/etc/ssh/sshd.d/50-redhat.conf` by adding the following line:

```console
Port 2222
```

2. Let selinux know that ssh is now running on port 2222 by the following command:

`sudo semanage port -a -t ssh_port_t -p tcp 2222`

3. Install docker

`sudo dnf install docker -y`

4. pull the docker image

`docker run --detach --publish 22:2222 --name HONEYPOT cowrie/cowrie:latest`

Dit werkt wel:
`sudo docker run -p 22:2222 cowrie/cowrie:282accc0`

je ziet de logs live

Om de container in de achtergrond te runnen:

`sudo docker run -d -p 22:2222 cowrie/cowrie:282accc0`

To get into the running docker container:
`sudo docker exec -it <NAME> sh`

To check the log from a container running in the background:

`sudo docker logs <NAME>`

Choose the docker.io.

Check: `docker ps -a`

5. Check the cowrie logs:

Try to ssh into your router then you get this:

![pic](/images/foto8.1.png)

Everyting is also logged in the files /var/log/cowrie

## Honeyup & Opencanary

Honeyup is een honeypot die slechte website-security simuleert op een /uploads endpoint om zo een aantrekkelijk doelwit te zijn voor hackers. Daarnaast biedt Opencanary een alerting-systeem aan (e-mail) zodra een Opencanary-honeypot geraakt wordt. Beiden zijn open source.