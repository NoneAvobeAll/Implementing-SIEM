# Implementing-SIEM
# **Complete Guide to Implementing SIEM on Your Servers**  

Procedure to set up **SIEM (Security Information and Event Management)** on your servers, including **requirements, installation, configuration, and monitoring**.  

---

## **🔴 1. Understanding SIEM**
A **SIEM** solution collects, analyzes, and correlates security logs from multiple sources, enabling **real-time threat detection, log management, and compliance monitoring**.

### **💡 Key Features:**
✔️ **Log collection & storage**  
✔️ **Real-time threat detection**  
✔️ **Security incident response**  
✔️ **Forensic analysis**  
✔️ **Compliance reporting (PCI-DSS, HIPAA, GDPR, SOC 2, etc.)**  

### **🛠️ Popular SIEM Solutions:**
| Open-Source | Enterprise |
|-------------|-----------|
| Wazuh + ELK Stack | Splunk |
| OSSEC | IBM QRadar |
| Graylog | Microsoft Sentinel |
| SIEMonster | ArcSight |

For this guide, we’ll use **Wazuh + ELK Stack** (free, open-source, and powerful).  

---

## **🟢 2. System Requirements**
**💻 Minimum System Requirements:** (For a small deployment, 5-10 servers)  
| Component | Requirement |
|------------|------------|
| OS | Ubuntu 22.04 (Recommended) / CentOS 8 / RHEL 8 |
| CPU | 4 Cores (8+ for large deployments) |
| RAM | 8GB (16GB+ for large deployments) |
| Disk Space | 100GB SSD+ (500GB+ for long retention) |
| Network | 1 Gbps (low latency recommended) |

For **large-scale deployment (100+ servers)**, consider **distributed architecture** with separate **Wazuh Manager, Elasticsearch, Kibana, and Filebeat servers**.

---

## **🔵 3. Deployment Options**
1️⃣ **On-Premise Deployment** – Installed on your own servers. Best for full control.  
2️⃣ **Cloud Deployment** – Install on **AWS, Azure, GCP** (Use managed Elasticsearch).  
3️⃣ **Hybrid Deployment** – Combination of cloud and on-premise for flexibility.  

For this guide, we’ll use an **on-premise setup** on **Ubuntu 22.04**.

---

## **🔴 4. Installing SIEM (Wazuh + ELK Stack)**
We will install:
✅ **Wazuh Server** – Core security analysis.  
✅ **Elasticsearch** – Stores and indexes logs.  
✅ **Kibana** – Visualizes logs and security alerts.  
✅ **Filebeat** – Collects and forwards logs.

---

### **🔵 A. Install Wazuh Server**
1️⃣ **Update System & Install Dependencies**
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl apt-transport-https unzip -y
```

2️⃣ **Add Wazuh Repository & Install**
```bash
curl -sO https://packages.wazuh.com/key/GPG-KEY-WAZUH
sudo apt-key add GPG-KEY-WAZUH
echo "deb https://packages.wazuh.com/4.x/apt/ stable main" | sudo tee /etc/apt/sources.list.d/wazuh.list
sudo apt update
sudo apt install wazuh-manager -y
```

3️⃣ **Start Wazuh Manager**
```bash
sudo systemctl enable wazuh-manager
sudo systemctl start wazuh-manager
```

---

### **🔵 B. Install Elasticsearch (Log Storage)**
1️⃣ **Install OpenSearch (Elasticsearch Alternative)**
```bash
wget -qO - https://artifacts.opensearch.org/packages/2.x/opensearch-keyring.asc | sudo gpg --dearmor -o /usr/share/keyrings/opensearch-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/opensearch-keyring.gpg] https://artifacts.opensearch.org/packages/2.x/apt stable main" | sudo tee /etc/apt/sources.list.d/opensearch.list
sudo apt update && sudo apt install opensearch -y
```

2️⃣ **Configure OpenSearch**
```bash
sudo nano /etc/opensearch/opensearch.yml
```
Modify:
```yaml
network.host: 0.0.0.0
cluster.name: wazuh-cluster
node.name: node-1
```

3️⃣ **Start and Enable OpenSearch**
```bash
sudo systemctl enable opensearch
sudo systemctl start opensearch
```

---

### **🔵 C. Install Kibana (Dashboard & Visualization)**
1️⃣ **Install Kibana**
```bash
sudo apt install opensearch-dashboards -y
```

2️⃣ **Configure Kibana**
```bash
sudo nano /etc/opensearch-dashboards/opensearch_dashboards.yml
```
Modify:
```yaml
server.host: "0.0.0.0"
elasticsearch.hosts: ["http://localhost:9200"]
```

3️⃣ **Start Kibana**
```bash
sudo systemctl enable opensearch-dashboards
sudo systemctl start opensearch-dashboards
```

---

### **🔵 D. Install Filebeat (Log Forwarding)**
1️⃣ **Install Filebeat**
```bash
sudo apt install filebeat -y
```

2️⃣ **Enable Wazuh Module**
```bash
sudo filebeat modules enable wazuh
```

3️⃣ **Configure Filebeat**
```bash
sudo nano /etc/filebeat/filebeat.yml
```
Modify:
```yaml
output.elasticsearch:
  hosts: ["http://localhost:9200"]
setup.kibana:
  host: "http://localhost:5601"
```

4️⃣ **Start Filebeat**
```bash
sudo systemctl enable filebeat
sudo systemctl start filebeat
```

---

## **🟢 5. Deploy Wazuh Agents**
To monitor servers, install the **Wazuh agent** on each system.

### **A. Install Wazuh Agent on Linux**
```bash
curl -sO https://packages.wazuh.com/4.x/apt/wazuh-agent.deb
sudo dpkg -i wazuh-agent.deb
```
Modify the config:
```bash
sudo nano /var/ossec/etc/ossec.conf
```
Set:
```xml
<client>
  <server>
    <address>WAZUH_SERVER_IP</address>
  </server>
</client>
```
Start Agent:
```bash
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

### **B. Install Wazuh Agent on Windows**
1️⃣ Download **Wazuh Agent**:  
👉 [https://packages.wazuh.com/4.x/windows/wazuh-agent.msi](https://packages.wazuh.com/4.x/windows/wazuh-agent.msi)  

2️⃣ Install via PowerShell:
```powershell
msiexec.exe /i wazuh-agent.msi /quiet WAZUH_MANAGER="WAZUH_SERVER_IP"
```

3️⃣ Start Agent:
```powershell
net start WazuhSvc
```

---

## **🔴 6. Verify & Monitor**
1️⃣ **Access Kibana**  
   - Open: `http://WAZUH_SERVER_IP:5601`  
   - Navigate to **Security Events**  
   - Check logs & alerts.  

2️⃣ **Check Wazuh Agent Status**
```bash
/var/ossec/bin/agent_control -l
```

3️⃣ **Generate Test Log**
```bash
logger "Test SIEM log entry"
```
Check Kibana for logs.

---

## **🟢 7. Configure SIEM Rules & Alerts**
1️⃣ **Enable Security Rules in Wazuh**
```bash
sudo nano /var/ossec/etc/ossec.conf
```
Modify:
```xml
<ruleset>
  <include>rules_config.xml</include>
</ruleset>
```

2️⃣ **Configure Email Alerts**
```bash
sudo nano /var/ossec/etc/ossec.conf
```
Set:
```xml
<global>
  <email_notification>yes</email_notification>
  <email_to>security@yourcompany.com</email_to>
</global>
```

3️⃣ **Restart Wazuh**
```bash
sudo systemctl restart wazuh-manager
```

---

## **✅ END**
🎉 Your **SIEM system is now fully operational!** 🚀 You can now **monitor logs, detect threats, and ensure compliance**.

