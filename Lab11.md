# Week 11: Lab 11

## Introduction

"System hardening" is defined as a set of best practices, tools and/or techniques to reduce security risk as much as possible. In security terms people talk about "eliminating potential attack vectors and condensing the system's attack surface".

Ansible is a powerfull tool to configure systems in an automated way. Imagine deploying sysmon and winlogbeat to Windows clients or filebeats on Linux machines manually over and over again. This seems like a perfect job for some ansible playbooks.

 ## Ansible

Install ansible on your router. Follow the official installation instructions found over here. We suggest to use pip: https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html

Start of by creating a SSH keypair and copy over the correct (public or private?) key to the remote locations (which file holds this key on the remote machine?). You can use the vagrant user for everything.

Create an inventory file containing the machines in your environment. Below is an example inventory file. Feel free to extend this. The name between brackets is a group. The name below such brackets is the hostname (or IP). When using [name:vars] it is possible to define extra variables (or settings) for a specific group. Do you notice what protocol is used to authenticate with Windows in this case? What protocol is used for the other machines

Inside router cd to `/home/vagrant/.ansible`

Daarin maken we een `inventory.yml` met de volgende inhoud:

```console
[router]
192.168.56.254

[router:vars]
ansible_port=22
ansible_user=vagrant
ansible_password=vagrant

[dc]
172.30.42.4

[dc:vars]
ansible_user=vagrant
ansible_password=vagrant
ansible_connection=winrm
ansible_winrm_server_cert_validation=ignore

[web]
172.30.42.2

[database]
172.30.42.3

[Debian]
192.168.56.222

[Debian:vars]
ansible_user=debian
ansible_password=debian

[elk]
172.30.42.5

[elk:vars]
ansible_user=vagrant
ansible_password=vagrant

[linux_host]
172.30.128.12

[windows_host]
172.30.128.11

[windows_host:vars]
ansible_user=vagrant
ansible_password=vagrant
ansible_connection=winrm
ansible_winrm_server_cert_validation=ignore

[windows]
172.30.42.4
172.30.128.11

[linux]
172.30.42.2
172.30.42.3
172.30.42.6
192.168.56.222
172.30.128.12
192.168.56.254
```

# LAB 8 Hunting and hardening with ansible

- installeren van ansible met python3 -m pip install --upgrade --user ansible
- we maken volgende files aan:
## /home/vagrant/.ansible/inventory.yml
```console
[router]
192.168.56.254

[router:vars]
ansible_port=22
ansible_user=vagrant
ansible_password=vagrant

[dc]
172.30.42.4

[dc:vars]
ansible_user=vagrant
ansible_password=vagrant
ansible_connection=winrm
ansible_winrm_server_cert_validation=ignore

[web]
172.30.42.2

[database]
172.30.42.3

[Debian]
192.168.56.222

[Debian:vars]
ansible_user=debian
ansible_password=debian

[elk]
172.30.42.5

[elk:vars]
ansible_user=vagrant
ansible_password=vagrant

[linux_host]
172.30.128.12

[windows_host]
172.30.128.11

[windows_host:vars]
ansible_user=vagrant
ansible_password=vagrant
ansible_connection=winrm
ansible_winrm_server_cert_validation=ignore

[windows]
172.30.42.4
172.30.128.11

[linux]
172.30.42.2
172.30.42.3
172.30.42.6
192.168.56.222
172.30.128.12
192.168.56.254
```

## /home/vagrant/.ansible/ansible.cfg
[defaults]
private_key_file=/home/vagrant/.ssh/id_rsa



- met ssh-keygen maken we een nieuwe key in .ssh/id_rsa we gaan geen passphrase in geven
- we installeren ook pip pywinrm  op de router en via scp zetten we de key alle VMs
- met scp <location_ssh_pub_key>/id_rsa.pub vagrant@<ip_ansible_target>:/home/vagrant/.ssh/ or for windows C:/Users/Vagrant/.ssh/authorized_keys
- in linux geef dan nog het volgende in cat /home/vagrant/.ssh/id_rsa.pub >> /home/vagrant/.ssh/authorized_keys
- Nu kan je pingen naar alle vms met ansible -m ping linux en win_ping voor de dc

- met ansible linux -m shell -a date checken we de data voor windows gebruiken we win_shell

- voor de passwd file maken we een playbook en voeren we die uit met ansible-playbook <playbook_name>  (passwdpuller.yml)

---
  - hosts: linux
    tasks:
      - name: display multiple file contents
        ansible.builtin.debug: var=item
        with_file:
          - "/etc/passwd"
      - name: Store file into /tmp/fetched/host.example.com/tmp/somefile
        ansible.builtin.fetch:
          src: /etc/passwd
          dest: /home/vagrant


- Voor walt aan te maken gebruiken we onderstaand playbook (waltcreator.yml)

  - hosts: linux
    tasks:
      - name: Create walt user
        ansible.builtin.user:
          name: walt
          state: present
          password: "{{ 'Firday13th!' | password_hash('sha512') }}"
        become: yes
