## DDOS-Dashboard

This repository is supposed to give you templates for visualization of F5 Network DDoS events within an ELK stack.

This are the relvant template files:

	export.ndjson	: this is file includes dashboard, visualizations and other objects
	logstash.conf	: this file contains the logstash configuration to parse syslog messages from bigip
	template_1	: this is the template for fields of the ddos_dashboard
	template-stats	: this is the template for fileds of the ddos_stats dashboard

## Commands to configure logging on BIG-IP
	tmsh create ltm pool pool_log_server members add { 1.1.1.1:5557 }
	tmsh create sys log-config destination remote-high-speed-log HSL_LOG_DEST { pool-name pool_log_server protocol udp }
	tmsh create sys log-config destination splunk SPLUNK_LOG_DEST forward-to HSL_LOG_DEST
	tmsh create sys log-config publisher KIBANA_LOG_PUBLISHER destinations add { SPLUNK_LOG_DEST }
	tmsh create security log profile LOG_PROFILE dos-network-publisher KIBANA_LOG_PUBLISHER protocol-dns-dos-publisher KIBANA_LOG_PUBLISHER protocol-sip-dos-publisher KIBANA_LOG_PUBLISHER ip-intelligence { log-translation-fields enabled log-publisher KIBANA_LOG_PUBLISHER } traffic-statistics { syncookies enabled log-publisher KIBANA_LOG_PUBLISHER }
	tmsh modify security log profile global-network dos-network-publisher KIBANA_LOG_PUBLISHER ip-intelligence { log-geo enabled log-rtbh enabled log-scrubber enabled log-shun enabled log-translation-fields enabled log-publisher KIBANA_LOG_PUBLISHER } protocol-dns-dos-publisher KIBANA_LOG_PUBLISHER protocol-sip-dos-publisher KIBANA_LOG_PUBLISHER traffic-statistics { log-publisher KIBANA_LOG_PUBLISHER syncookies enabled }
	tmsh modify security dos device-config dos-device-config log-publisher KIBANA_LOG_PUBLISHER

![Kibana-Logging](https://user-images.githubusercontent.com/58518999/114186432-235da700-9947-11eb-9662-67eede1773d2.png)

## Enable logging for DOS_stats
	modify the crontab on BIG-IP and add: 
	* * * * * tmctl -c dos_stat -s context_name,vector_name,attack_detected,stats_rate,drops_rate,int_drops_rate,ba_stats_rate,ba_drops_rate,bd_stats_rate,bd_drops_rate,detection,mitigation_low,mitigation_high,detection_ba,mitigation_ba_low,mitigation_ba_high,detection_bd,mitigation_bd_low,mitigation_bd_high | grep -v "context_name" | logger -n 1.1.1.1 --udp --port 5556

	Modify IP and port appropiate.
	
	Better approach then using the crontab is to use an external monitor:
	https://support.f5.com/csp/article/K71282813
	
	Anyhow, keep in mind more frequently logging generates more data on you logging device!
	

## Install ELK
### Elasticsearch (install instructions)
	wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
	sudo apt-get install apt-transport-https
	echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
	sudo apt-get update && sudo apt-get install elasticsearch
	sudo -i service elasticsearch start
	curl -X GET localhost:9200/?pretty
### Kibana (install)
	wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
	sudo apt-get update && sudo apt-get install kibana
	sudo -i service kibana start
	you can access kibana from browser http://localhost:5601
	If you want to make it accessable from "external", then change "server.host" in /etc/kibana/kibana.yml to the IP address of the ELK server
### Logstash (install)
	sudo apt-get install openjdk-8-jre-headless
	sudo wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
	sudo apt-get update && sudo apt-get install logstash
### Publishing kibana with NGINX (optional)
	sudo apt-get install nginx
	echo "kibadmin:`openssl passwd -apr1`" | sudo tee -a /etc/nginx/htpasswd.users
	mv /etc/nginx/sites-available/default /etc/nginx/sites-available/original_backup_default
	nano /etc/nginx/sites-available/default
	Put the following into the new NGINX configuration file you just created, putting Kibana’s IP address in the server_name field:
	server {
    		listen 80;
    		server_name <YourKibanaIP>;
   	 	auth_basic "Restricted Access";
    		auth_basic_user_file /etc/nginx/htpasswd.users;
    	location / {
        	proxy_pass http://localhost:5601;
        	proxy_http_version 1.1;
        	proxy_set_header Upgrade $http_upgrade;
        	proxy_set_header Connection 'upgrade';
        	proxy_set_header Host $host;
        	proxy_cache_bypass $http_upgrade;        
    		}
	}
	nginx -t
	systemctl enable nginx
	systemctl start nginx

 
### Running configuration and import objects

	Copy the logstash.conf in /etc/logstash/conf.d/
	
	In the kibana WEBUI (http://localhost:5601) go to devtools, you need to apply index templates for ddos dashboard and stats dashboard:
	For ddos dashboard:
		“Paste the content of template_1.json file”
		“Press play-button”
	For stats dashboard:
		“Paste the content of template-stats.json file”
		“Press play-button”
		
	Import the dashboards and visualization by importing the export.ndjson file, by going to Stack Management then Saved Objects, and click on import and upload the file
 
	Run Logstash using the following from cli 
	sudo systemctl start logstash.service
	
## Important note for stats dashboard
	Please change this line in the logstash to match the number of tmms in your system, in this example the system have 4 tmms
	mutate { add_field => {"tmm" => "4"}} 	

### Install syslog-ng

	sudo apt-get install syslog-ng
	
	Add the following in /etc/syslog-ng/conf.d/syslog-ng.conf

	Make sure the following are in the file:
	############### F5 syslog-ng config ########################################
	source s_f5ddos_stats { udp(ip(0.0.0.0) port(5556) flags(no-hostname)); };
	destination d_f5ddos_stats { file("/var/log/f5ddos_stats.log" owner("root") group("root") perm(0644)); };
	log { source(s_f5ddos_stats); destination(d_f5ddos_stats); };

	source s_f5ddos_kv { udp(ip(0.0.0.0) port(5557) flags(no-hostname)); };
	destination d_f5ddos_kv { file("/var/log/f5ddos_kv.log" owner("root") group("root") perm(0644)); };
	log { source(s_f5ddos_kv); destination(d_f5ddos_kv); };
	###############################################################
	
	sudo systemctl start syslog-ng	
