## DDOS-Dashboard
This is the step by step guide to use this repository
the following is description of the files:

	export.ndjson	: this is file includes dashboard, visualizations and other objects
	logstash.conf	: this file contains the logstash configuration to parse syslog messages from bigip
	template_1	: this is the template for fields of the ddos_dashboard
	template-stats	: this is the template for fileds of the ddos_stats dashboard

## Install ELK
### Elasticsearch (install instructions)
	wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
	sudo apt-get install apt-transport-https
	echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
	sudo apt-get update && sudo apt-get install elasticsearch
	sudo -i service elasticsearch start
	curl -X GET "localhost:9200/?pretty”
### Kibana (install)
	wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
	sudo apt-get update && sudo apt-get install kibana
	sudo -i service kibana start
	you can access kibana from browser http://localhost:5601
### Logstash (install)
	sudo apt-get install openjdk-8-jre-headless
	sudo wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
	sudo apt-get update && sudo apt-get install logstash
 
### Running configuration and import objects

	Copy the logstash.conf in /etc/logstash/conf.d/
	In the kibana WEBUI (http://localhost:5601) go to devtools, you need to apply index templates for ddos dashboard and stats dashboard:
	For ddos dashboard:
		PUT _index_templates/template_1
		“Paste the content of template_1 file”
	For stats dashboard:
		PUT _index_templates/template-stats
		“Paste the content of template-stats file”
	Import the dashboards and visualization by importing the export.ndjson file, by going to Stack Management then Saved Objects, and 	  click on import and upload the file
 
	Run Logstash using the following from cli 
	cd /usr/share/logstash/bin/
	./logstash -f /etc/logstash/conf.d/logstash.conf


### Install syslog-ng

	sudo apt-get install syslog-ng

	Add the following in /etc/syslog-ng/conf.d/syslog-ng.conf

	syslog-ng.conf

	Make sure the following are in the file:
	############### F5 syslog-ng config ########################################
	source s_f5ddos { udp(ip(0.0.0.0) port(5555) flags(no-hostname)); };
	destination d_f5ddos { file("/var/log/f5ddos.log" owner("root") group("root") perm(0644)); };
	log { source(s_f5ddos); destination(d_f5ddos); };
	###############################################################

## Important note for stats dashboard
	Please change this line in the logstash to match the number of tmms in your system, in this example the system have 4 tmms
	mutate { add_field => {"tmm" => "4"}} 
