# AWS-SIEM-project
creating VPC  and segmenting newtork design + setting up security groups for restricted IP/port access on AWS
Figure out how to get the logs from router?
If you are using an AT&T router use the following guide to find the logs. 
Log in to your gateway (easily found on ipconfig)

Go to Diagnostics (may be prompt with access code)
Within go Logs and now we know where it is. 

Creating a Syslog client: 
In the diagnostics section, there is also a syslog section that is very GUI-friendly to send a syslog setup. The idea is to send all the logs from local server to SIEM.


Configure VPC on AWS: 
Create VPC
I named it MyVPC and used the VPC CDIR 10.0.0.0/16\
We now create subnets

We now create our subnets corresponding to the architecture divide











Go to route table and create a private- route table in case I need another private subnet.
And rename it Private -RT, link to my VPC. 
Rename the no-name myVPC on the route table to Mainl-RT. 

Edit subnet association of private - RT and link the private subnet 


Now we need to set up an internet gateway so our public subnet resources could connect to the internet. 

Create internet gateway -> I named it (MyIGw)-> create -> attach to VPC 

Now go to route -> checkmark the main-RT->router ->edit route->add 0.0.0.0 and link internet gateway -> save route 

This is for public subnet -> edit subnet settings -> checkmark enable auto assign public IPV4 address
Go to subnet and actions ->

Deploy wazuh SIEM: 
Idea is to create this deployment with EC2 and put it into the private subnet and have it receive the log data from the public subnet. 


Create VPC for SIEM lab

Figure out 

Wazuh indexer storage (Backend):
Reason: 
Store received logs for a period of time 
Fast searching and viewing data 
Ability to provide access to stored logs 
Search timeframes+ search requests 

Master Node: 
In charge of cluster wide settings and changes, deleting or creating indices and fields, adding or removing nodes 
Each cluster has a single master node elected from master eligible nodes using distributed consensus algorithm and is reelected if current node fails 
At least one node in cluster is master role 

Data Node: 
Stores and searches data 
Perform data related operations (indexing,searching, aggregating) on local shards. 
Needs more disk space than any other node type 

Ingest: 
Pre processed data before storing it in the cluster 
Run ingest pipeline transforming data before adding to index 

Set up vlan segment for each section and allow explicit connections ports \

Set up ec2 instance 

Open the ec2 instance

Creating the certs:

Do this after ssh into machine. 


Start changing the set up into its correcting nodes and hostname. (might need to set up DNS name) 

Running the server, dashboard configuration as same as my indexer. 


https://documentation.wazuh.com/current/installation-guide/wazuh-indexer/installation-assistant.html
Refer to this document for using the wazuh assistant. 
If you get wazuh indexer package fail do

 “sudo apt-get update”

Usually fixes to the problem. 



Passwords and user output 
 indexer_username: 'admin'
 indexer_username: 'admin'
  indexer_password: 'xlzW8uIiQua0IbqhlyJL2Fds5q.aMIjy'




Install index, server and dashboard. 


25/11/2022 15:02:40 INFO: --- Summary ---
25/11/2022 15:02:40 INFO: You can access the web interface https://10.0.1.143
    User: admin
    Password: xlzW8uIiQua0IbqhlyJL2Fds5q.aMIjy
25/11/2022 15:02:40 INFO: Installation finished.


wazuh-install-files/wazuh-passwords.txt
# Admin user for the web user interface and Wazuh indexer. Use this user to log in to Wazuh dashboard
  indexer_username: 'admin'
  indexer_password: 'xlzW8uIiQua0IbqhlyJL2Fds5q.aMIjy'

# Wazuh dashboard user for establishing the connection with Wazuh indexer
  indexer_username: 'kibanaserver'
  indexer_password: 'QRNy*zq.04LJq4NCBQ78VEUsz.6D?0Da'

# Regular Dashboard user, only has read permissions to all indices and all permissions on the .kibana index
  indexer_username: 'kibanaro'
  indexer_password: '7wTrdgdf5vNM*kh+G0mUlCrI0Eq?nSS8'

# Filebeat user for CRUD operations on Wazuh indices
  indexer_username: 'logstash'
  indexer_password: '4.6RAVPD?iNee8fo764ViPNOlcpssfQB'

# User with READ access to all indices
  indexer_username: 'readall'
  indexer_password: 'uVeNGzhIuvv+zffvv?dGjLharPXWVIF1'

# User with permissions to perform snapshot and restore operations
  indexer_username: 'snapshotrestore'
  indexer_password: '9cTd4fTV.CyAfbyEeXIS6TmA9tCQCIyt'

# Password for wazuh API user
  api_username: 'wazuh'
  api_password: '.G6c9wv7XLIWI+.dskeEMkIgsPBcCMs+'

# Password for wazuh-wui API user
  api_username: 'wazuh-wui'
  api_password: 'P+M67?90qC4y+1D3PRqYkN09k+F5b2Ea'


Create windows instance in the back end subnet: 







Graylog installation: (log ingestion VPC) 
Collects logs originating on endpoint devices, network devices 
Normalize log field to a universal name so that faster searching and better visualization can be built. Example, source_ip, ipv4 _ip  should be written to a src _ip field 
Cashing of logs in case indexer is offline or busy
Clean the fuck out of these logs 
Supported graylog inputs: 
AWS cloudtrail 
Syslog 
raw/Plaintext 
Etc
Configuring the pre-requisites:
sudo apt update && sudo apt upgrade
sudo apt install apt-transport-https openjdk-11-jre-headless uuid-runtime pwgen dirmngr gnupg wget


MONODB for graylog:
Stores configurations on graylogs
Store meta data 


wget -qO - https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add -
echo "deb http://repo.mongodb.org/apt/debian buster/mongodb-org/4.4 main" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list
sudo apt-get update
sudo apt-get install -y mongodb-org

sudo systemctl daemon-reload
sudo systemctl enable mongod.service
sudo systemctl restart mongod.service
sudo systemctl --type=service --state=active | grep mongod

Make sure to do sudo apt-get update && sudo apt-get install -y gnupg2

Sudo apt-get install -y gnupg




Install  gray log: 


wget https://packages.graylog2.org/repo/packages/graylog-4.3-repository_latest.deb
sudo dpkg -i graylog-4.3-repository_latest.deb
sudo apt-get update && sudo apt-get install graylog-server graylog-integrations-plugins

Extract tar file in wazuh to get the root-ca.pem

Upload it to the graylog instance, I just copy and pasted in nano editor on graylog. 

Run these commands and move root-ca.pem to /etc/graylog/server/certs/ first. 


mkdir /etc/graylog/server/certs

cp -a /usr/lib/jvm/java-11-openjdk-amd64/lib/security/cacerts /etc/graylog/server/certs/cacerts

keytool -importcert -keystore /etc/graylog/server/certs/cacerts -storepass changeit -alias root_ca -file /etc/graylog/server/certs/root-ca.pem

Edit graylog configuration: 


Edit config file 
Needs password secret from that pwgen command 
Needs hash password for root password = look below






