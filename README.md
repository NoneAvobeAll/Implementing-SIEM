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

### Wazuh Agent (For Each Monitored Server)
- **OS:** Linux (Ubuntu, CentOS, RHEL) or Windows Server  
- **CPU:** 1 core  
- **RAM:** 512MB  
- **Disk:** 2GB  
- **Network:** Stable connection to the SIEM server

---

## 3. Deployment Options

- **On-Premise:** Full control on your own hardware (this guide uses an on-premise setup).
- **Cloud:** Deploy on AWS, Azure, or GCP using managed services.
- **Hybrid:** A combination of on-premise and cloud deployments.

---

## 4. Installation Steps

### A. Install Wazuh Server

1. **Update and Install Dependencies**
   ```bash
   sudo apt update && sudo apt upgrade -y
   sudo apt install curl apt-transport-https unzip -y
   ```

2. **Add the Wazuh Repository and Install Wazuh Manager**
   ```bash
   curl -sO https://packages.wazuh.com/key/GPG-KEY-WAZUH
   sudo apt-key add GPG-KEY-WAZUH
   echo "deb https://packages.wazuh.com/4.x/apt/ stable main" | sudo tee /etc/apt/sources.list.d/wazuh.list
   sudo apt update
   sudo apt install wazuh-manager -y
   ```

3. **Start Wazuh Manager**
   ```bash
   sudo systemctl enable wazuh-manager
   sudo systemctl start wazuh-manager
   ```

---

### B. Install OpenSearch (Log Storage)

1. **Install OpenSearch**
   ```bash
   wget -qO - https://artifacts.opensearch.org/packages/2.x/opensearch-keyring.asc | sudo gpg --dearmor -o /usr/share/keyrings/opensearch-keyring.gpg
   echo "deb [signed-by=/usr/share/keyrings/opensearch-keyring.gpg] https://artifacts.opensearch.org/packages/2.x/apt stable main" | sudo tee /etc/apt/sources.list.d/opensearch.list
   sudo apt update && sudo apt install opensearch -y
   ```

2. **Configure OpenSearch**  
   Edit `/etc/opensearch/opensearch.yml` and add/modify:
   ```yaml
   network.host: 0.0.0.0
   cluster.name: wazuh-cluster
   node.name: node-1
   ```

3. **Start OpenSearch**
   ```bash
   sudo systemctl enable opensearch
   sudo systemctl start opensearch
   ```

---

### C. Install OpenSearch Dashboards (Visualization)

1. **Install Dashboards**
   ```bash
   sudo apt install opensearch-dashboards -y
   ```

2. **Configure Dashboards**  
   Edit `/etc/opensearch-dashboards/opensearch_dashboards.yml` and add/modify:
   ```yaml
   server.host: "0.0.0.0"
   elasticsearch.hosts: ["http://localhost:9200"]
   ```

3. **Start Dashboards**
   ```bash
   sudo systemctl enable opensearch-dashboards
   sudo systemctl start opensearch-dashboards
   ```

---

### D. Install Filebeat (Log Forwarding)

1. **Install Filebeat**
   ```bash
   sudo apt install filebeat -y
   ```

2. **Enable the Wazuh Module**
   ```bash
   sudo filebeat modules enable wazuh
   ```

3. **Configure Filebeat**  
   Edit `/etc/filebeat/filebeat.yml` and set:
   ```yaml
   output.elasticsearch:
     hosts: ["http://localhost:9200"]
   setup.kibana:
     host: "http://localhost:5601"
   ```

4. **Start Filebeat**
   ```bash
   sudo systemctl enable filebeat
   sudo systemctl start filebeat
   ```

---

## 5. Deploy and Connect Wazuh Agents

### A. Linux Agent Installation

1. **Download and Install the Agent**
   ```bash
   curl -sO https://packages.wazuh.com/4.x/apt/wazuh-agent.deb
   sudo dpkg -i wazuh-agent.deb
   ```
- [Read official Guide For Linux Agent](https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-package-linux.html)

2. **Configure the Agent**  
   Edit `/var/ossec/etc/ossec.conf` and configure the server address:
   ```xml
   <client>
     <server>
       <address>WAZUH_SERVER_IP</address>
     </server>
   </client>
   ```
   Replace `WAZUH_SERVER_IP` with the IP address of your SIEM server.

3. **Start the Agent**
   ```bash
   sudo systemctl enable wazuh-agent
   sudo systemctl start wazuh-agent
   ```

- [Deployement Variable For Linux](https://documentation.wazuh.com/current/user-manual/agent/agent-enrollment/deployment-variables/deployment-variables-linux.html)

### B. Windows Agent Installation

1. **Download the Wazuh Agent**  
   [Download Wazuh Agent MSI](https://packages.wazuh.com/4.x/windows/wazuh-agent.msi)

2. **Install the Agent via PowerShell**
   ```powershell
   msiexec.exe /i wazuh-agent.msi /quiet WAZUH_MANAGER="WAZUH_SERVER_IP"
   ```
   Replace `WAZUH_SERVER_IP` with the IP address of your SIEM server.

2.1 **Configure with gui**
    ![image](https://github.com/user-attachments/assets/446a0940-1e97-4bf3-b518-1c17a2a1e09d)

4. **Start the Agent Service**
   ```powershell
   net start WazuhSvc
   ```
- [Read Official Guide for Windows Agent](https://documentation.wazuh.com/current/installation-guide/wazuh-agent/wazuh-agent-package-windows.html)
- [Deployement Variables for Windows](https://documentation.wazuh.com/current/user-manual/agent/agent-enrollment/deployment-variables/deployment-variables-windows.html)
---

## 6. Verifying Deployment and Accessing the SIEM Dashboard

1. **Access OpenSearch Dashboards:**  
   Open your browser and navigate to:
   ```
   http://<WAZUH_SERVER_IP>:5601
   ```
   This dashboard lets you view security events, logs, and alerts.

2. **Test the Setup:**  
   Generate a test log entry on the server:
   ```bash
   logger "Test SIEM log entry"
   ```
   Then, verify the log appears in the OpenSearch Dashboards.

3. **Check Agent Connectivity:**  
   On the SIEM server, run:
   ```bash
   /var/ossec/bin/agent_control -l
   ```
   This command lists connected agents to ensure they’re communicating properly with the server.

---

## 7. Conclusion

Your SIEM system is now fully deployed and operational. You can monitor logs, detect threats, and ensure compliance. For further customizations—such as adding custom rules or email alerts—refer to the official documentation:

- [Wazuh Documentation](https://documentation.wazuh.com)
- [OpenSearch Documentation](https://opensearch.org/docs/)

Feel free to adapt this guide based on your environment and additional requirements.
