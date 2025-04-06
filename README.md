# Step-by-Step Guide to Installing and Configuring ELK Stack with Wazuh on Ubuntu 22.04 in VMware Fusion

Hi! My name is Reinaldi, and I’m passionate about IT security—especially when it comes to learning how security tools work under the hood.

This guide is my step-by-step journey to install and configure the ELK Stack (Elasticsearch, Filebeat, Kibana) along with Wazuh on Ubuntu 22.04, all running inside VMware Fusion. I’ll be using SSH from my MacBook to access the Ubuntu VM, so it’s all done remotely and cleanly.

Whether you're building a home lab to dive deeper into SIEM, get hands-on with log analysis, or just love tinkering with cybersecurity tools—this guide is for you.

Here’s what we’ll cover:

Installing Elasticsearch to store and index logs
Setting up Filebeat to ship logs from Wazuh to Elasticsearch
Installing Kibana to visualize everything in one clean dashboard
Deploying Wazuh as the core of your SIEM setup
Adding a Windows 10 Wazuh agent to monitor endpoints
And along the way, we’ll also:

Set up SSL certificates to secure communication
Configure Elasticsearch passwords
Install the Wazuh plugin for Kibana to take your dashboards up a notch
This lab setup is perfect if you:

Are learning cybersecurity or want to understand how a SIEM works
Love building your own tools and environments to test stuff
Work in a SOC and need a personal sandbox to explore
Let’s dive in!


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
<img width="743" alt="Image" src="https://github.com/user-attachments/assets/74a3cc12-bedf-46e1-8626-39892defb49f" />

<img width="796" alt="Image" src="https://github.com/user-attachments/assets/293b046b-9dcf-42f7-9568-fa9b17e311c2" />

## Step 2: Add Elasticsearch Repository
```bash
curl -s https://artifacts.elastic.co/GPG-KEY-elasticsearch | gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/elasticsearch.gpg --import && chmod 644 /usr/share/keyrings/elasticsearch.gpg
```
```bash
echo "deb [signed-by=/usr/share/keyrings/elasticsearch.gpg] https://artifacts.elastic.co/packages/7.x/apt stable main" | tee /etc/apt/sources.list.d/elastic-7.x.list
```
<img width="800" alt="Image" src="https://github.com/user-attachments/assets/44577e7a-b0ae-4263-a05b-f965067bf66c" />

<img width="800" alt="Image" src="https://github.com/user-attachments/assets/0e393394-538d-4284-94d7-403f8f03305f" />

## Step 3: Install Elasticsearch
```bash
apt-get update
apt-get install elasticsearch=7.17.12
```
```bash
curl -so /etc/elasticsearch/elasticsearch.yml https://packages.wazuh.com/4.5/tpl/elastic-basic/elasticsearch_all_in_one.yml
```
<img width="671" alt="Image" src="https://github.com/user-attachments/assets/91128090-5888-4542-9b20-0f2f11bbf61d" />

<img width="783" alt="Image" src="https://github.com/user-attachments/assets/5b460a6e-42d5-4da6-a7c6-4a57855e794b" />

<img width="805" alt="Image" src="https://github.com/user-attachments/assets/6d0b5ddb-fb91-496a-9478-715c6b3e5576" />

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
<img width="799" alt="Image" src="https://github.com/user-attachments/assets/f984fc64-7265-440a-b224-0a428719dba1" />

<img width="802" alt="Image" src="https://github.com/user-attachments/assets/f60b4d50-c060-4a4d-97e9-931b51027265" />

<img width="579" alt="Image" src="https://github.com/user-attachments/assets/a09d52de-9f20-44ef-a555-59e63d928ad7" />

<img width="800" alt="Image" src="https://github.com/user-attachments/assets/a4e290a3-29a0-4cdf-840e-a3f52d6fe190" />

## Step 5: Start Elasticsearch Service
```bash
systemctl daemon-reload
systemctl enable elasticsearch
systemctl start elasticsearch
systemctl status elasticsearch
```
<img width="798" alt="Image" src="https://github.com/user-attachments/assets/3a9daa6e-fd2b-4524-ba77-35a7d65ec4ab" />

<img width="800" alt="Image" src="https://github.com/user-attachments/assets/a945251e-1b16-4fb5-a0e9-949017df2a49" />

## Step 6: Set Up Elasticsearch Passwords
```bash
/usr/share/elasticsearch/bin/elasticsearch-setup-passwords auto
```
<img width="804" alt="Image" src="https://github.com/user-attachments/assets/be0951ef-ddb9-4c08-ad44-a6d3964a4e21" />

## Step 7: Verify Elasticsearch Installation
```bash
curl -XGET https://localhost:9200 -u elastic:<elastic_password> -k
```
<img width="793" alt="Image" src="https://github.com/user-attachments/assets/aeb245b4-bda8-4efe-bb51-f1b5bd32b711" />

## Step 8: Add Wazuh Repository
```bash
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import && chmod 644 /usr/share/keyrings/wazuh.gpg
```
```bash
echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | tee -a /etc/apt/sources.list.d/wazuh.list
```
<img width="803" alt="Image" src="https://github.com/user-attachments/assets/919d1359-acec-4e1b-96d4-513ddb0bf1ad" />


<img width="801" alt="Image" src="https://github.com/user-attachments/assets/bb305a3a-4375-4cb7-9720-f075ba353e90" />


## Step 9: Install and Start Wazuh Manager
```bash
apt-get update
apt-get install wazuh-manager
systemctl daemon-reload
systemctl enable wazuh-manager
systemctl start wazuh-manager
systemctl status wazuh-manager
```
<img width="789" alt="Image" src="https://github.com/user-attachments/assets/5109d1dd-82fd-415a-892c-6869c57426d7" />

<img width="729" alt="Image" src="https://github.com/user-attachments/assets/6c199965-106b-4a50-982f-3234c5199afb" />

<img width="798" alt="Image" src="https://github.com/user-attachments/assets/8fd50c81-b043-4ca0-a4b6-af2b7f1f86c5" />

<img width="804" alt="Image" src="https://github.com/user-attachments/assets/ed7c0225-47fb-4842-94a9-26d506dff6a6" />

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
mkdir -p /etc/filebeat/certs && \
cp -r /etc/elasticsearch/certs/ca/ /etc/filebeat/certs/ && \
cp /etc/elasticsearch/certs/elasticsearch.crt /etc/filebeat/certs/filebeat.crt && \
cp /etc/elasticsearch/certs/elasticsearch.key /etc/filebeat/certs/filebeat.key && \
chown -R root:root /etc/filebeat/certs && \
chmod -R 500 /etc/filebeat/certs && \
chmod 400 /etc/filebeat/certs/filebeat.*
ls -l /etc/filebeat/certs/ca/
```
```bash
systemctl daemon-reload
systemctl enable filebeat
systemctl start filebeat
filebeat test output
```

<img width="752" alt="Image" src="https://github.com/user-attachments/assets/6524149c-12a8-4502-8722-c247eaabfb8a" />

<img width="805" alt="Image" src="https://github.com/user-attachments/assets/df554488-623e-4197-a6d7-ddc179d0152a" />

<img width="799" alt="Image" src="https://github.com/user-attachments/assets/e2b3b1cd-3a31-4529-b527-7ded74e5266d" />

<img width="712" alt="Image" src="https://github.com/user-attachments/assets/ea44ec3b-2d40-4fdf-88d4-277a24ba3a71" />

<img width="804" alt="Image" src="https://github.com/user-attachments/assets/0801766f-8065-49a9-937f-c826642991af" />

<img width="807" alt="Image" src="https://github.com/user-attachments/assets/79308511-163d-48e7-ab3f-6c8e29b5533c" />


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
```

<img width="733" alt="Image" src="https://github.com/user-attachments/assets/0f06646b-6429-495c-9c80-31a6fa23498d" />

<img width="797" alt="Image" src="https://github.com/user-attachments/assets/7cf2c1de-8ed9-4934-bc61-e6bbb80cda5a" />

<img width="805" alt="Image" src="https://github.com/user-attachments/assets/49f5b038-5cc8-4e11-9914-535fb43c42e5" />

<img width="799" alt="Image" src="https://github.com/user-attachments/assets/d3ab1637-5052-4a85-b57b-f357fe23ed86" />

<img width="517" alt="Image" src="https://github.com/user-attachments/assets/3d1ec7c6-5031-4a58-aea2-5adac26c9154" />



