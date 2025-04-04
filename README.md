# Step-by-Step Guide to Installing and Configuring ELK Stack with Wazuh on Ubuntu 22.04 in VMware Fusion

**Deploying Wazuh Server and Adding an Agent**  

**Wazuh Version:** 4.2  
**Wazuh Master:** Ubuntu 20  
**Wazuh Agent:** Windows 10  

This guide provides step-by-step instructions for deploying Wazuh Server version 4.2 on an Ubuntu 20 system to monitor a Windows 10 agent.

---  
## Prerequisites
Ensure you have root privileges before proceeding with the installation.

```bash
sudo su
```

## Step 1: Update and Install Required Packages
```bash
apt-get update
apt-get install apt-transport-https zip unzip lsb-release curl gnupg
```

## Step 2: Add Elasticsearch Repository
```bash
curl -s https://artifacts.elastic.co/GPG-KEY-elasticsearch | gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/elasticsearch.gpg --import && chmod 644 /usr/share/keyrings/elasticsearch.gpg
```
```bash
echo "deb [signed-by=/usr/share/keyrings/elasticsearch.gpg] https://artifacts.elastic.co/packages/7.x/apt stable main" | tee /etc/apt/sources.list.d/elastic-7.x.list
```

## Step 3: Install Elasticsearch
```bash
apt-get update
apt-get install elasticsearch=7.17.12
```
```bash
curl -so /etc/elasticsearch/elasticsearch.yml https://packages.wazuh.com/4.5/tpl/elastic-basic/elasticsearch_all_in_one.yml
```

## Step 4: Generate and Configure Elasticsearch Certificates
```bash
curl -so /usr/share/elasticsearch/instances.yml https://packages.wazuh.com/4.5/tpl/elastic-basic/instances_aio.yml
/usr/share/elasticsearch/bin/elasticsearch-certutil cert ca --pem --in instances.yml --keep-ca-key --out ~/certs.zip
```
```bash
unzip ~/certs.zip -d ~/certs
mkdir -p /etc/elasticsearch/certs/ca
cp -R ~/certs/ca/ ~/certs/elasticsearch/* /etc/elasticsearch/certs/
chown -R elasticsearch: /etc/elasticsearch/certs
chmod -R 500 /etc/elasticsearch/certs
chmod 400 /etc/elasticsearch/certs/ca/ca.* /etc/elasticsearch/certs/elasticsearch.*
rm -rf ~/certs/ ~/certs.zip
```

## Step 5: Start Elasticsearch Service
```bash
systemctl daemon-reload
systemctl enable elasticsearch
systemctl start elasticsearch
systemctl status elasticsearch
```

## Step 6: Set Up Elasticsearch Passwords
```bash
/usr/share/elasticsearch/bin/elasticsearch-setup-passwords auto
```

## Step 7: Verify Elasticsearch Installation
```bash
curl -XGET https://localhost:9200 -u elastic:<elastic_password> -k
```

## Step 8: Add Wazuh Repository
```bash
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import && chmod 644 /usr/share/keyrings/wazuh.gpg
```
```bash
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | tee -a /etc/apt/sources.list.d/wazuh.list
```

## Step 9: Install and Start Wazuh Manager
```bash
apt-get update
apt-get install wazuh-manager
systemctl daemon-reload
systemctl enable wazuh-manager
systemctl start wazuh-manager
systemctl status wazuh-manager
```

## Step 10: Install and Configure Filebeat
```bash
apt-get install filebeat=7.17.12
curl -so /etc/filebeat/filebeat.yml https://packages.wazuh.com/4.5/tpl/elastic-basic/filebeat_all_in_one.yml
```
```bash
curl -so /etc/filebeat/wazuh-template.json https://raw.githubusercontent.com/wazuh/wazuh/4.4/extensions/elasticsearch/7.x/wazuh-template.json
chmod go+r /etc/filebeat/wazuh-template.json
curl -s https://packages.wazuh.com/4.x/filebeat/wazuh-filebeat-0.2.tar.gz | tar -xvz -C /usr/share/filebeat/module
```
```bash
nano /etc/filebeat/filebeat.yml  # Add Elasticsearch password
```
```bash
cp -r /etc/elasticsearch/certs/ca/ /etc/filebeat/certs/
cp /etc/elasticsearch/certs/elasticsearch.crt /etc/filebeat/certs/filebeat.crt
cp /etc/elasticsearch/certs/elasticsearch.key /etc/filebeat/certs/filebeat.key
```
```bash
systemctl daemon-reload
systemctl enable filebeat
systemctl start filebeat
filebeat test output
```

## Step 11: Install and Configure Kibana
```bash
apt-get install kibana=7.17.12
mkdir -p /etc/kibana/certs/ca
cp -R /etc/elasticsearch/certs/ca/ /etc/kibana/certs/
cp /etc/elasticsearch/certs/elasticsearch.key /etc/kibana/certs/kibana.key
cp /etc/elasticsearch/certs/elasticsearch.crt /etc/kibana/certs/kibana.crt
chown -R kibana:kibana /etc/kibana/
chmod -R 500 /etc/kibana/certs
chmod 440 /etc/kibana/certs/ca/ca.* /etc/kibana/certs/kibana.*
```
```bash
curl -so /etc/kibana/kibana.yml https://packages.wazuh.com/4.5/tpl/elastic-basic/kibana_all_in_one.yml
mkdir /usr/share/kibana/data
chown -R kibana:kibana /usr/share/kibana
cd /usr/share/kibana
sudo -u kibana /usr/share/kibana/bin/kibana-plugin install https://packages.wazuh.com/4.x/ui/kibana/wazuh_kibana-4.5.2_7.17.12-1.zip
```
```bash
setcap 'cap_net_bind_service=+ep' /usr/share/kibana/node/bin/node
nano /etc/kibana/kibana.yml  # Add Elasticsearch password
```
```bash
systemctl daemon-reload
systemctl enable kibana
systemctl start kibana
systemctl status kibana
