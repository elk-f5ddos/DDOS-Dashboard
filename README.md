# DDOS-Dashboard
This is the step by step guide to use this repository
the following is description of the files:

	export.ndjson	: this is file includes dashboard, visualizations and other objects
	logstash.conf	: this file contains the logstash configuration to parse syslog messages from bigip
	template_1	: this is the template for fields of the ddos_dashboard
	template-stats	: this is the template for fileds of the ddos_stats dashboard

Install ELK
1.	Elasticsearch (install instructions)
a.	wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
b.	sudo apt-get install apt-transport-https
c.	echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
d.	sudo apt-get update && sudo apt-get install elasticsearch
e.	sudo -i service elasticsearch start
f.	curl -X GET "localhost:9200/?pretty”
2.	Kibana (install)
a.	wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
b.	sudo apt-get update && sudo apt-get install kibana
c.	sudo -i service kibana start
d.	you can access kibana from browser http://localhost:5601
3.	Logstash (install)
a.	sudo apt-get install openjdk-8-jre-headless
b.	sudo wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
c.	sudo apt-get update && sudo apt-get install logstash
 
Running configuration and import objects

a.	Copy the logstash.conf in /etc/logstash/conf.d/
b.	In the kibana WEBUI (http://localhost:5601) go to devtools, you need to apply index templates for ddos dashboard and stats dashboard:
For ddos dashboard:
PUT _index_templates/template_1
“Paste the content of template_1 file”
For stats dashboard:
PUT _index_templates/template-stats
“Paste the content of template-stats file”
c.	Import the dashboards and visualization by importing the export.ndjson file, by going to Stack Management then Saved Objects, and click on import and upload the file
 
d.	Run Logstash using the following from cli 
	cd /usr/share/logstash/bin/
./logstash -f /etc/logstash/conf.d/logstash.conf

 
Testing the configuration

Testing parsing before sending the logs put the following log samples in /var/log/f5ddos.log
Dec 18 12:19:12 10.103.10.215 "Dec 18 2020 13:39:38","172.30.107.11","lon-i5800-1.pme.itc.f5net.com","/Common/PO_10.103.1.50_DNS_L2","","","","","","","","","A query DOS","3061536636","Attack Started","None","0","0","0000000000000000"
Dec 18 12:28:46 10.103.10.215 "Dec 18 2020 13:49:12","172.30.107.11","lon-i5800-1.pme.itc.f5net.com","/Common/PO_10.103.1.50_DNS_L2","/Common/vlan3001","A","aa.f5.com","52.10.252.144","10.103.1.50","21136","53","0","A query DOS","3061536636","Attack Sampled","Allow","11124","0","0000000000000000"
Dec 18 12:28:47 10.103.10.215 "Dec 18 2020 13:49:13","172.30.107.11","lon-i5800-1.pme.itc.f5net.com","/Common/PO_10.103.1.50_DNS_L2","/Common/vlan3001","A","aa.f5.com","52.66.233.170","10.103.1.50","64490","53","0","A query DOS","3061536636","Attack Sampled","Allow","11188","0","0000000000000000"
Dec 18 12:28:48 10.103.10.215 "Dec 18 2020 13:49:14","172.30.107.11","lon-i5800-1.pme.itc.f5net.com","/Common/PO_10.103.1.50_DNS_L2","/Common/vlan3001","A","aa.f5.com","52.168.88.188","10.103.1.50","54558","53","0","A query DOS","3061536636","Attack Sampled","Allow","11257","0","0000000000000000"
Dec 18 12:28:49 10.103.10.215 "Dec 18 2020 13:49:15","172.30.107.11","lon-i5800-1.pme.itc.f5net.com","/Common/PO_10.103.1.50_DNS_L2","/Common/vlan3001","A","aa.f5.com","52.181.100.134","10.103.1.50","30822","53","0","A query DOS","3061536636","Attack Sampled","Allow","11325","0","0000000000000000"
Dec 18 12:28:50 10.103.10.215 "Dec 18 2020 13:49:16","172.30.107.11","lon-i5800-1.pme.itc.f5net.com","/Common/PO_10.103.1.50_DNS_L2","/Common/vlan3001","A","aa.f5.com","52.10.79.31","10.103.1.50","6772","53","0","A query DOS","3061536636","Attack Sampled","Allow","11286","0","0000000000000000"
Dec 18 12:28:51 10.103.10.215 "Dec 18 2020 13:49:17","172.30.107.11","lon-i5800-1.pme.itc.f5net.com","/Common/PO_10.103.1.50_DNS_L2","/Common/vlan3001","A","aa.f5.com","52.208.64.240","10.103.1.50","52342","53","0","A query DOS","3061536636","Attack Sampled","Allow","11366","0","0000000000000000"
Dec 18 12:28:52 10.103.10.215 "Dec 18 2020 13:49:18","172.30.107.11","lon-i5800-1.pme.itc.f5net.com","/Common/PO_10.103.1.50_DNS_L2","/Common/vlan3001","A","aa.f5.com","52.176.138.112","10.103.1.50","20978","53","0","A query DOS","3061536636","Attack Sampled","Allow","11143","0","0000000000000000"
Dec 18 12:28:53 10.103.10.215 "Dec 18 2020 13:49:19","172.30.107.11","lon-i5800-1.pme.itc.f5net.com","/Common/PO_10.103.1.50_DNS_L2","/Common/vlan3001","A","aa.f5.com","52.251.74.208","10.103.1.50","41830","53","0","A query DOS","3061536636","Attack Sampled","Allow","11264","0","0000000000000000"
Dec 18 12:28:54 10.103.10.215 "Dec 18 2020 13:49:20","172.30.107.11","lon-i5800-1.pme.itc.f5net.com","/Common/PO_10.103.1.50_DNS_L2","/Common/vlan3001","A","aa.f5.com","52.84.171.42","10.103.1.50","53350","53","0","A query DOS","3061536636","Attack Sampled","Allow","11120","0","0000000000000000"
Dec 18 12:28:55 10.103.10.215 "Dec 18 2020 13:49:21","172.30.107.11","lon-i5800-1.pme.itc.f5net.com","/Common/PO_10.103.1.50_DNS_L2","/Common/vlan3001","A","aa.f5.com","52.239.170.12","10.103.1.50","1646","53","0","A query DOS","3061536636","Attack Sampled","Allow","11265","0","0000000000000000"
Dec 18 12:28:56 10.103.10.215 "Dec 18 2020 13:49:22","172.30.107.11","lon-i5800-1.pme.itc.f5net.com","/Common/PO_10.103.1.50_DNS_L2","/Common/vlan3001","A","aa.f5.com","52.168.133.66","10.103.1.50","64896","53","0","A query DOS","3061536636","Attack Sampled","Allow","11123","0","0000000000000000"
Dec 18 12:29:16 10.103.10.215 "Dec 18 2020 13:49:42","172.30.107.11","lon-i5800-1.pme.itc.f5net.com","/Common/PO_10.103.1.50_DNS_L2","","","","","","","","","A query DOS","3061536636","Attack Stopped","None","0","0","0000000000000000"

Install syslog-ng

sudo apt-get install syslog-ng

Add the following in /etc/syslog-ng/conf.d/syslog-ng.conf

syslog-ng.conf

Make sure the following are in the file:
############### F5 syslog-ng config ########################################
source s_f5ddos { udp(ip(0.0.0.0) port(5555) flags(no-hostname)); };
destination d_f5ddos { file("/var/log/f5ddos.log" owner("root") group("root") perm(0644)); };
log { source(s_f5ddos); destination(d_f5ddos); };
###############################################################
