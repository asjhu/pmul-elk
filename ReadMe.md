### ELK for BeyondTust PMUL (ELK ver. 8.6.1 as of 2/4/23)
##### https://www.elastic.co/guide/en/elasticsearch/reference/8.6/rpm.html  Instructions below come from this URL with additional steps specific to PMUL
#### Install & Configure Elasticsearch (RHEL/Centos - rpm) Ubuntu - see "Install & Configure Elasticsearch (Ubuntu - deb)"

##### 1) su - to root
```sh
su -
```
##### 2) Add a new YUM repository (RPM)
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
##### 3) Install Elasticsearch (RPM)
```sh
sudo yum install --enablerepo=elasticsearch elasticsearch --nogpgcheck -y
```
##### 4) Copy elastic user password from screen output. This gets generated on install. Then continue with Kibana
```sh
yum -y install --enablerepo=elasticsearch kibana --nogpgcheck -y
systemctl daemon-reload
systemctl enable elasticsearch 
systemctl enable kibana
```
##### 5) Linux host firewall ports as needed
```sh
firewall-cmd --permanent --add-port 9200/tcp
firewall-cmd --permanent --add-port 5601/tcp
firewall-cmd --reload
```

#### Install & Configure Elasticsearch (Ubuntu - deb)
```sh
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
sudo apt-get install apt-transport-https
sudo echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
sudo apt-get update && sudo apt-get install elasticsearch kibana
sudo systemctl enable elasticsearch
sudo systemctl enable kibana
sudo systemctl daemon-reload
```
##### UFW ports as needed
```sh
sudo ufw allow 9200/tcp
sudo ufw allow 5601/tcp
sudo ufw reload
```
### Back on to Common Instructions
##### 6) Configure ElasticSearch and start ElasticSearch. Update the Following items with your own environment information in /etc/elasticsearch/elasticsearch.yml
```sh
network.host: (IP address of your host)
```
##### 7) Start Elasticsearch
```sh
systemctl start elasticsearch
```
##### 8) Configure Kibana. Use vi to edit /etc/kibana/kibana.yml. Uncomment and update the server.host with your ELK server IP address.  For example, server.host: "192.168.0.80"
```sh
vi /etc/kibana/kibana.yml
server.host: "192.168.0.80"
:wq!
```
##### 9) Start Kibana
```sh
systemctl start kibana
```
##### 10) Generate a Kibana enrollment token for Kibana instances and save for later
```sh
/usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana
```
##### 11) Using web browser go to your ELK server for example, http://192.168.0.80:5601 and paste enrollment token when prompted
##### 12) Login to Kibana using elastic user ID and password saved in step 4
##### 13) Note: you may be asked to run /usr/share/kibana/bin/kibana-verification-code from the CLI and enter into the Kibana webUI.
```sh
/usr/share/kibana/bin/kibana-verification-code
```
##### 14) In ELK server, genenrate a pem format certificate.
```sh
/usr/share/elasticsearch/bin/elasticsearch-certutil cert --self-signed --pem --out /etc/elasticsearch/certs/certs.zip
```
##### 15) Create a directory in PMUL/Policy server to store ELK certs ex. /etc/elkcerts
```sh
mkdir -p /etc/elkcerts
cd /etc/elkcerts
``` 
##### 16) WinSCP certs to PMUL/Policy Server certs.zip and http_ca.crt
##### or use SCP.  SSH to ELK server cd to /etc/elasticsearch/certs/
```sh
cd /etc/elasticsearch/certs/
scp certs.zip http_ca.crt root@YOUR_PMUL_POLICY_SERVER:/etc/elkcerts/
```
##### 17) Back to Policy server, unzip certs.zip, install unzip if you dont have it.
```sh
yum install unzip -y
unzip certs.zip
rm -f certs.zip
```
##### 18) Change file permission to rw
```sh
chmod -R u-x,og-rwx /etc/elkcerts
```
##### 19) Still in policy server, uncomment (remove #) the following in /etc/pb.settings.
```sh
vi /etc/pb.settings
elkinstances    elasticsearch=https://IP_ADDR_OF_ELK_SERVER:9200
elkcredential   elastic_basic  
elkcafile       /etc/elkcerts/http_ca.crt
elkcertfile     /etc/elkcerts/instance/instance.crt
elkkeyfile      /etc/elkcerts/instance/instance.key
elkdatatypes    eventlog iolog
```
##### 20) Configure PMUL logservers to use ELK
```sh
pbadmin --elkcred -s '{"id":"elastic_basic","type":"basic","username":"elastic","password":"PASSWORD IN STEP 4"}'
systemctl restart pblighttpd
```
##### ToDo: BIUL connection setup to ELK and ELK queries, Kibana dashbord

