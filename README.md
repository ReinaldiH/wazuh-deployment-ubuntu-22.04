# Installation and Deployment of Wazuh Server and Agent on Ubuntu 22.04

**Deploying Wazuh Server and Adding an Agent**  

**Wazuh Version:** 4.2  
**Wazuh Master:** Ubuntu 20  
**Wazuh Agent:** Windows 10  

This guide provides step-by-step instructions for deploying Wazuh Server version 4.2 on an Ubuntu 20 system to monitor a Windows 10 agent.

---  
### **1. Hardware Requirements**  
Before starting the installation, ensure that the system meets the following hardware requirements:  
- **RAM:** 4GB  
- **CPU:** 2 cores  
These settings can be adjusted in the system configuration while installing Ubuntu on VirtualBox.

---  
### **2. Installing Wazuh Server on Ubuntu 20**  
The Wazuh Server is responsible for collecting and analyzing data received from Wazuh Agents. It operates Wazuh Manager, API, and Filebeat. The first step is to add the Wazuh repository.  

#### **2.1 Adding the Wazuh Repository**  
1. Install required packages:  
   ```bash  
   apt install curl apt-transport-https unzip wget libcap2-bin software-properties-common lsb-release gnupg  
   ```  
2. Add the GPG Key:  
   ```bash  
   curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | apt-key add -  
   ```  
3. Add the repository file:  
   ```bash  
   echo "deb https://packages.wazuh.com/4.x/apt/ stable main" | tee -a /etc/apt/sources.list.d/wazuh.list  
   ```  
4. Update the package list:  
   ```bash  
   apt-get update  
   ```  

#### **2.2 Installing Wazuh Manager**  
1. Install Wazuh Manager:  
   ```bash  
   apt-get install wazuh-manager=4.2.7-1  
   ```  
2. Enable and start the Wazuh Manager service:  
   ```bash  
   systemctl daemon-reload  
   systemctl enable wazuh-manager  
   systemctl start wazuh-manager  
   ```  
3. Verify that the service is running:  
   ```bash  
   systemctl status wazuh-manager  
   ```  

#### **2.3 Installing and Configuring Elasticsearch**  
1. Install Elasticsearch OSS and Open Distro for Elasticsearch:  
   ```bash  
   apt install elasticsearch-oss opendistroforelasticsearch  
   ```  
2. Configure Elasticsearch:  
   ```bash  
   curl -so /etc/elasticsearch/elasticsearch.yml https://packages.wazuh.com/resources/4.2/open-distro/elasticsearch/7.x/elasticsearch_all_in_one.yml  
   ```  
3. Set up Elasticsearch users and roles:  
   ```bash  
   curl -so /usr/share/elasticsearch/plugins/opendistro_security/securityconfig/roles.yml https://packages.wazuh.com/resources/4.2/open-distro/elasticsearch/roles/roles.yml  
   ```  
4. Generate and apply SSL certificates:  
   ```bash  
   rm /etc/elasticsearch/esnode-key.pem /etc/elasticsearch/esnode.pem /etc/elasticsearch/kirk-key.pem /etc/elasticsearch/kirk.pem /etc/elasticsearch/root-ca.pem -f  
   curl -so ~/wazuh-cert-tool.sh https://packages.wazuh.com/resources/4.2/open-distro/tools/certificate-utility/wazuh-cert-tool.sh  
   bash ~/wazuh-cert-tool.sh  
   ```  
5. Enable and start Elasticsearch service:  
   ```bash  
   systemctl daemon-reload  
   systemctl enable elasticsearch  
   systemctl start elasticsearch  
   ```  

#### **2.4 Installing Filebeat**  
Filebeat forwards alerts and archived events securely to Elasticsearch.  
1. Install Filebeat:  
   ```bash  
   apt-get install filebeat  
   ```  
2. Download the pre-configured Filebeat configuration file:  
   ```bash  
   curl -so /etc/filebeat/filebeat.yml https://packages.wazuh.com/resources/4.2/open-distro/filebeat/7.x/filebeat_all_in_one.yml  
   ```  
3. Enable and start Filebeat service:  
   ```bash  
   systemctl daemon-reload  
   systemctl enable filebeat  
   systemctl start filebeat  
   ```  
4. Verify Filebeat installation:  
   ```bash  
   filebeat test output  
   ```  

#### **2.5 Installing Kibana**  
1. Install Kibana:  
   ```bash  
   apt-get install opendistroforelasticsearch-kibana  
   ```  
2. Download and apply the Kibana configuration:  
   ```bash  
   curl -so /etc/kibana/kibana.yml https://packages.wazuh.com/resources/4.2/open-distro/kibana/7.x/kibana_all_in_one.yml  
   ```  
3. Install the Wazuh Kibana plugin:  
   ```bash  
   cd /usr/share/kibana  
   sudo -u kibana /usr/share/kibana/bin/kibana-plugin install https://packages.wazuh.com/4.x/ui/kibana/wazuh_kibana-4.2.7_7.10.2-1.zip  
   ```  
4. Enable and start Kibana service:  
   ```bash  
   systemctl daemon-reload  
   systemctl enable kibana  
   systemctl start kibana  
   ```  
5. Access the web interface:  
   - **URL:** `https://<wazuh_server_ip>`  
   - **User:** `admin`  
   - **Password:** `admin`  

---  
### **3. Installing Wazuh Agent on Windows 10**  
The Wazuh Agent runs on the monitored host and communicates with the Wazuh Manager in near real-time through an encrypted and authenticated channel.

1. Download the Windows installer (`wazuh-agent-4.2.7.1.msi`).  
2. Install the agent and launch the GUI. If the GUI does not appear automatically, use the following command:  
   - **Using CMD:**  
     ```bash  
     wazuh-agent-4.2.7-1.msi /q WAZUH_MANAGER="10.0.0.2" WAZUH_REGISTRATION_SERVER="10.0.0.2"  
     ```  
   - **Using PowerShell:**  
     ```powershell  
     .\wazuh-agent-4.2.7-1.msi /q WAZUH_MANAGER="10.0.0.2" WAZUH_REGISTRATION_SERVER="10.0.0.2"  
     ```  

This completes the setup of the Wazuh Server and its Windows 10 agent.

