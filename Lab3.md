# Lab 3: Week 3
 
## ELK 

Goal: create a central monitoring environment for logs.

**E**lastic

**L**ogstash

**K**ibana


First we have to create a new VM. I have made an centos-7 Linux vm. 6 GB RAM.

Then we heve to install the ELK-stack using this guideline: `https://www.elastic.co/guide/en/elasticsearch/reference/current/rpm.html`

## Elasticsearch

We have to turn off the firewall.

`sudo systemctl stop firewalld`

Also disable it:

`sudo systemctl disable firewalld` 

After that install java:

`sudo yum install -y java-1.8.0-openjdk`

Import GPG key:

`rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch`

After that make a file in `/etc/yum.repos.d/` with the name `elasticsearch.repo` and enter the following content init:

```console
[elasticsearch]
name=Elasticsearch repository for 8.x packages
baseurl=https://artifacts.elastic.co/packages/8.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=0
autorefresh=1
type=rpm-md
```


Then install elasticsearch by the following command:

`sudo yum install --enablerepo=elasticsearch elasticsearch`

Password for user `elastic` == `fx__64Qhq1rJ*Xp+kscD`
2e poging:
Password for user `elastic` == `9odKJbOCj0Wjs4iEyvD*`

Enable elasticsearch to start at boot:

```console
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable elasticsearch.service
```

## Kibana

Make another yum repo: `/etc/yum.repos.d/kibana.repo` with the following content:

```console
[Kibana-6.x]
name=Kibana repository for 6.x packages
baseurl=https://artifacts.elastic.co/packages/6.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```

After that install kibana:

`yum install -y kibana`

enable kibana:

```console
systemctl daemon-reload
systemctl enable kibana
systemctl start kibana
```


## Logstash

install: 

`sudo yum install logstash`

generate SSL Certficates

`openssl req -subj '/CN=server.example.com/' -x509 -days 3650 -nodes -batch -newkey rsa:2048 -keyout /etc/pki/tls/private/logstash.key -out /etc/pki/tls/certs/logstash.crt`

Create Logstash config file

`vi /etc/logstash/conf.d/01-logstash-simple.conf`

Paste the below content:

```console
input {
  beats {
    port => 5044
    ssl => true
    ssl_certificate => "/etc/pki/tls/certs/logstash.crt"
    ssl_key => "/etc/pki/tls/private/logstash.key"
  }
}

filter {
    if [type] == "syslog" {
        grok {
            match => {
                "message" => "%{SYSLOGTIMESTAMP:syslog_timestamp} %{SYSLOGHOST:syslog_hostname} %{DATA:syslog_program}(?:\[%{POSINT:syslog_pid}\])?: %{GREEDYDATA:syslog_message}"
            }
            add_field => [ "received_at", "%{@timestamp}" ]
            add_field => [ "received_from", "%{host}" ]
        }
        syslog_pri { }
        date {
            match => [ "syslog_timestamp", "MMM  d HH:mm:ss", "MMM dd HH:mm:ss" ]
        }
    }
}

output {
    elasticsearch {
        hosts => "localhost:9200"
        index => "%{[@metadata][beat]}-%{+YYYY.MM.dd}"
    }
}
```

Enable and start logstash service:

```console
systemctl enable logstash
systemctl start logstash
```

## Configure the elasticsearch.yml
`vim /etc/elasticsearch/elasticsearch.yml`

node.name: node-1 --> uit commentaar halen

network.host: 192.168.59.116 --> uit commentaar halen

http.port: 9200 --> uit commentaar halen

xpack security alles op false of disable zetten.

## Check if elasticsearch works

`curl localhost:9200`

or on your host machine

`http://192.168.59.115:9200`

If you get output it works.

## Check if Kibana works

Does not work

Edit the kibana file: `kibana.yml`

Uncomment the port and the elasticsearch host and add the ip.

The issue was in the elasticsearch.yml --> cluster.initial_master_nodes: 192.168.59.115

Now everything works.

## Questions

Try to figure out what the difference is between elasticsearch and logstash. Do you need logstash? It might help to read up a bit on what an ELK-stack is and how it works.

`Logstash is commonly used as an input pipeline for Elasticsearch as it allows for on the fly data transformation. These transformations can be applied through various filter plugins. There are two different versions of Logstash.`


What is difference between Elasticsearch and Kibana?

`Kibana is an open source (Apache Licensed), browser based analytics and search dashboard for Elasticsearch. Kibana is a snap to setup and start using. Kibana strives to be easy to get started with, while also being flexible and powerful, just like Elasticsearch.`

Elasticsearch is an open source, full-text search and analysis engine, based on the Apache Lucene search engine. Logstash is a log aggregator that collects data from various input sources, executes different transformations and enhancements and then ships the data to various supported output destinations. Kibana is a visualization layer that works on top of Elasticsearch, providing users with the ability to analyze and visualize the data. And last but not least — Beats are lightweight agents that are installed on edge hosts to collect different types of data for forwarding into the stack.

(https://logz.io/learn/complete-guide-elk-stack/#:~:text=Elasticsearch%20is%20an,into%20the%20stack.)