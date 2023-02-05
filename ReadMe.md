### ELK for BT PMUL (version 8.6.1 as of 2/4/23)
##### https://www.elastic.co/guide/en/elasticsearch/reference/8.6/rpm.html  Instructions below come from this URL with additional steps specific to PMUL
#### Install & Configure Elasticsearch (RHEL/Centos - rpm) Ubuntu - see below
##### 1)Add a new YUM repository (RPM)
```sh
rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
cat - > /etc/yum.repos.d/elasticsearch.repo <<-_END_
[elasticsearch]
name=Elasticsearch repository for 8.x packages
baseurl=https://artifacts.elastic.co/packages/8.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=0
autorefresh=1
type=rpm-md
_END_
```
##### 2) Install Elasticsearch (RPM)
```sh
yum -y install --enablerepo=elasticsearch elasticsearch
yum -y install --enablerepo=elasticsearch kibana
systemctl enable elasticsearch 
systemctl enable kibana
```
##### 3) Copy elastic user password from screen output. This gets generated on install.
##### 4) Linux host firewall ports as needed
```sh
firewall-cmd --permanent --add-port 9200/tcp
firewall-cmd --permanent --add-port 5601/tcp
firewall-cmd --reload
```

#### Install & Configure Elasticsearch (Ubuntu - deb)
##### 1) The following commands will configure the elastic repo and install/start Elasticsearch
```sh
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
sudo apt-get install apt-transport-https
sudo echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
sudo apt-get update && sudo apt-get install elasticsearch kibana
sudo systemctl enable elasticsearch
sudo systemctl enable kibana
sudo systemctl daemon-reload
```
##### 2) UFW ports as needed
```sh
sudo ufw allow 9200/tcp
sudo ufw allow 5601/tcp
sudo ufw reload
```
#### Back on to Common Instructions
##### 1) Configure ElasticSearch and start ElasticSearch. Update the Following items with your own environmental information in /etc/elasticsearch/elasticsearch.yml
```sh
network.host: (update with your fqdn)
```
##### 2) Start Elasticsearch
```sh
systemctl start elasticsearch
```
##### 3) Configure Kibana for remote access. Use vi to edit /etc/kibana/kibana.yml. Uncomment and Update the server.host: key with the host IP address.
```sh
ex: server.host: "192.168.0.10"
vi /etc/kibana/kibana.yml
```
##### 4) Start Kibana
```sh
systemctl start kibana
```
##### 5) Generate a Kibana enrollment token for Kibana instances and save for later
```sh
/usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana
```
##### 6) Browse to server hostname or IP address using HTTP and port 5601 example: http://192.168.100.27:5601 and Paste in enrollment token when prompted
##### 7) Login to Kibana using the elastic user ID and password
##### 8) Note: you may be asked to run /usr/share/kibana/bin/kibana-verification-code from the CLI and enter into the Kibana webUI.
