## DDOS-Dashboard

This repository is supposed to give you templates for visualization of F5 Network DDoS events within an ELK stack.
It will also give you the possibility to see all relevant stats for all DoS vectors.

<img src="images/image1.png">

<img src="images/image2.png">

### Templates

	export.ndjson	: this is file includes dashboard, visualizations and other objects
	logstash.conf	: this file contains the logstash configuration to parse syslog messages from bigip
	template_1	: this is the template for fields of the ddos_dashboard
	template-stats	: this is the template for fileds of the ddos_stats dashboard
	node1-pipeline  : this is the request to deploy ingest pipeline

### Commands to configure logging on BIG-IP
	tmsh create ltm pool pool_log_server members add { 1.1.1.1:5558 }
	tmsh create sys log-config destination remote-high-speed-log HSL_LOG_DEST { pool-name pool_log_server protocol udp }
	tmsh create sys log-config destination splunk SPLUNK_LOG_DEST forward-to HSL_LOG_DEST
	tmsh create sys log-config publisher KIBANA_LOG_PUBLISHER destinations add { SPLUNK_LOG_DEST }
	tmsh create security log profile LOG_PROFILE dos-network-publisher KIBANA_LOG_PUBLISHER protocol-dns-dos-publisher KIBANA_LOG_PUBLISHER protocol-sip-dos-publisher KIBANA_LOG_PUBLISHER ip-intelligence { log-translation-fields enabled log-publisher KIBANA_LOG_PUBLISHER } traffic-statistics { syncookies enabled log-publisher KIBANA_LOG_PUBLISHER }
	tmsh modify security log profile global-network dos-network-publisher KIBANA_LOG_PUBLISHER ip-intelligence { log-geo enabled log-rtbh enabled log-scrubber enabled log-shun enabled log-translation-fields enabled log-publisher KIBANA_LOG_PUBLISHER } protocol-dns-dos-publisher KIBANA_LOG_PUBLISHER protocol-sip-dos-publisher KIBANA_LOG_PUBLISHER traffic-statistics { log-publisher KIBANA_LOG_PUBLISHER syncookies enabled }
	tmsh modify security dos device-config dos-device-config log-publisher KIBANA_LOG_PUBLISHER

![Kibana-Logging](https://user-images.githubusercontent.com/58518999/114186432-235da700-9947-11eb-9662-67eede1773d2.png)

### Enable logging for DOS_stats
	modify the crontab on BIG-IP and add: 
	* * * * * nb_of_tmms=$(tmsh show sys tmm-info | grep Sys::TMM | wc -l);tmctl -c dos_stat -s context_name,vector_name,attack_detected,stats_rate,drops_rate,int_drops_rate,ba_stats_rate,ba_drops_rate,bd_stats_rate,bd_drops_rate,detection,mitigation_low,mitigation_high,detection_ba,mitigation_ba_low,mitigation_ba_high,detection_bd,mitigation_bd_low,mitigation_bd_high | grep -v "context_name" | sed '/^$/d' | sed "s/$/,$nb_of_tmms/g" | logger -n 1.1.1.1 --udp --port 5558

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
	sudo systemctl enable elasticsearch
	curl -X GET localhost:9200/?pretty
	
### Kibana (install instructions)
	wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
	sudo apt-get update && sudo apt-get install kibana
	sudo -i service kibana start
	sudo systemctl enable kibana
	you can access kibana from browser http://localhost:5601
	If you want to make it accessable from "external", then change "server.host" in /etc/kibana/kibana.yml to the IP address of the ELK server
	
### Logstash (install instructions)
	sudo apt-get install openjdk-8-jre-headless
	sudo wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
	sudo apt-get update && sudo apt-get install logstash
	sudo systemctl enable logstash
	
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
		“Paste the content of template_1 file”
		“Press play-button”
	For stats dashboard:
		“Paste the content of template-stats file”
		“Press play-button”
	For ingest-node :
		“Paste the content of node1-pipeline file”
		“Press play-button”
		
	Import the dashboards and visualization by importing the export.ndjson file, by going to Stack Management then Saved Objects, and 	  click on import and upload the file
 
	Run Logstash using the following from cli 
	sudo -i service logstash start

### Install syslog-ng

	sudo apt-get install syslog-ng
	
	Add the syslog-ng.conf in /etc/syslog-ng/syslog-ng.conf
	
	sudo systemctl start syslog-ng	
	
### Enabling Minimal Security Settings
	Reference : https://www.elastic.co/guide/en/elasticsearch/reference/7.13/security-minimal-setup.html
	Stop logstash 	   	: service logstash stop
	Stop elasticsearch 	: service elasticsearch stop
	Stop kibana  	   	: service kibana stop	
	Add "xpack.security.enabled: true" in /etc/elasticsearch/elasticsearch.yml (i have added under Node section)
	Start elasticsearch	: service elasticseach start
	Run the password utility to generate the passwords:
		cd /usr/share/elasticsearch/bin
		./elasticsearch-setup-passwords auto 
	This will generate different passwords for system users.
	Save the generated passwords. You’ll need them to add the built-in user to Kibana and Logstash
	
	#### Kibana #### 
	Enable "elasticsearch.username: "kibana_system""  by uncommenting from kibana config file /etc/kibana/kibana.yml
	Change to kibana bin directory : 
	cd /usr/share/kibana/bin
	Create Kibana keystore : 
	./kibana-keystore create
	Add the password for the kibana_system user to the Kibana keystore : 
	./kibana-keystore add elasticsearch.password
	Start Kibana 		: service kibana start
	login to kibana use elastic username and password
	go to Stack Management => security ==> Users and add your own user with the required role 
	
	#### Logstash ####
	In logstash file located in /etc/logstash/conf.d/logstash add username and password as the following in the output:
	
	if "f5ddoskv" in [tags]{
        elasticsearch {
	    user => "elastic"
	    password => "<REPLACE BY THE PASSWORD GENERATED FOR elastic user>"
        }
	}
        if "f5ddos_stats" in [tags] {
        elasticsearch {
            user => "elastic"
            password => "<REPLACE BY THE PASSWORD GENERATED FOR elastic user>"
        }
        }
	
	Start logstash 		: service logstash start
	
	
	
	
	
	
	
