###DEPLOYING ELK STACK using ansible playbooks

1. Modify Install_5_AWS.yml file to have needed amount of AWS Debian instances created
2. Run Install_5_AWS.yml, it will write vars to invetory file, so you can use these vars later for faster access and server setup. > ansible-playbook /etc/ansible/install_5_AWS.yml
3. Run Install_ELKStack.yml, it will use already 5 created AWS Debian servers and will configure 3 nodes for elastic search, 1 node for Kibana and 1 node for Logstash one by one, installing all needed software and then pointing to each service > ansible-playbook /etc/ansible/install_ELKstack.yml -i /etc/ansible/inventories/inventory.ini
4. Completed. You have 5 running nodes with all needed services and software.

##Note: Installing elastic search nodes and software is optimized. It always checks for cluster group: elastic and if software is installed for this host, then it will skip installation. But if software is not yet installed, then playbook will download everything.
All hosts information will be stored in /etc/ansible/inventories/inventory.ini file


###Additional task (manually at many steps) to install Grafana, prometheus_node_exporters, add prometheus.yml file to scrape info and then dashboard.json for Grafana dashboard:
1. Create new AWS Debian server and install Grafana and prometheus, configuring completely: ansible-playbook /etc/ansible/install_AWS_and_Grafana.yml 
2. Install prometheus_node_exporters on all hosts in inventory.file ansible-playbook /etc/ansible/Install_Prom_node_exporters.yml -i /etc/ansible/inventories/inventory.ini
3. Move prometheus.yml file to new instance created on step 1 and then also move grafana_dashboard.json to grafana dashboards config folder. Everything is stored under /etc/prometheus
4. Restart services > sudo systemctl restart grafana-server and then > sudo systemctl restart prometheus

###Backup approach:
1. Create step by step instructions how to setup everything, which is running right now (As it's done in this README.md file), show instructions and store all configs for ansible playbooks there.
2. If anything is updated, then it should be updated manually in Git (as an example > dashboard in Grafana, or config for prometheus). But better is to have automatic push from git to Grafana configuration or prometheus yml file, so you need just to update Git repository and it will be checked and pushed automatically from Git to needed place. Can be done by jenkins pipelines.

###To setup TLS for all services:
1. Let's generate a private key: openssl genrsa -out <service-name>.key 2048
2. Generate a Certificate Signing Request: openssl req -new -key <service-name>.key -out <service-name>.csr
3. And then let's use it: openssl x509 -req -days 365 -in <service-name>.csr -signkey <service-name>.key -out <service-name>.crt
4. Use it for service like Elastic search, kibana and so on
5. Copy generated .crt and .key files to servers, where these services are running
6. Configure service to use certificate and key files
7. Restart service
8. Repeat for each service
