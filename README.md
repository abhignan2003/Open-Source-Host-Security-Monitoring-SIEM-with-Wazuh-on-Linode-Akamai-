# 🛡️Open-Source-Host-Security-Monitoring-SIEM-with-Wazuh-on-Linode-Akamai
## 📌 Overview

This project demonstrates the deployment and configuration of a **full-scale SIEM (Security Information and Event Management) system using Wazuh**, including:

* Centralized log collection
* Real-time threat detection
* File integrity monitoring
* Registry monitoring (Windows)
* Automated incident response
* Vulnerability detection
* External alerting via Slack

The setup simulates a **real SOC (Security Operations Center) environment** by integrating multiple agents and detecting real-world attack patterns.

---

## 🧱 Architecture

* **Wazuh Manager** (Linux Server - Cloud/Docker)
* **Agents**:

  * Linux (Kali/Ubuntu)
  * Windows
* **Data Flow**:

  * Agents → Wazuh Manager → Dashboard → Alerts (Slack)

---

## ⚙️ Phase 1: Server Deployment

### Option A: Cloud Deployment (Linode Marketplace)

1. Create a Linode instance from Marketplace using Wazuh image
2. Configure:

   * Email, sudo user, password
   * Ubuntu image
   * Region (nearest location)
3. Select **minimum 4GB RAM plan**
   ⚠️ 2GB will fail for marketplace deployment
4. Launch the instance and connect via SSH:

   ```bash
   ssh user@your_server_ip
   ```
5. Monitor installation:

   ```bash
   htop
   ```
6. Retrieve credentials:

   ```bash
   cat .deployment-secrets.txt
   ```
7. Access dashboard:

   ```
   https://your-server-ip
   ```

---

### Option B: Docker Deployment

1. Setup Ubuntu 22.04 server
2. Install dependencies:

   ```bash
   sudo apt update
   sudo apt install docker.io docker-compose -y
   ```
3. Clone repository:

   ```bash
   git clone https://github.com/wazuh/wazuh-docker.git
   cd wazuh-docker/single-node
   ```
4. Generate certificates (from official docs)
5. Start services:

   ```bash
   docker-compose up -d
   ```
6. Access dashboard:

   ```
   https://your-server-ip
   ```

   Default:

   * Username: admin
   * Password: SecretPassword

---

## 🖥️ Phase 2: Agent Deployment

### Linux Agent

1. Navigate to **Wazuh Dashboard → Add Agent**
2. Select:

   * OS: Linux
   * Architecture: x86_64
3. Provide:

   * Server IP
   * Agent name
4. Copy the generated command and run:

   ```bash
   sudo <generated-command>
   ```
5. Enable service:

   ```bash
   sudo systemctl enable wazuh-agent
   sudo systemctl start wazuh-agent
   ```

---

### Windows Agent

1. Go to **Deploy New Agent**
2. Select Windows
3. Copy PowerShell command
4. Run in the administrator terminal
5. Start service:

   ```powershell
   net start wazuhsvc
   ```

---

## 🔍 Phase 3: Advanced Security Configuration

### 📁 File Integrity Monitoring (Real-Time)

Edit:

```
C:\Program Files (x86)\ossec-agent\ossec.conf
```

Add inside `<syscheck>`:

```xml
<directories realtime="yes" report_changes="yes" check_all="yes">
C:\Users\YourUsername\Desktop
</directories>
```

Restart service:

```powershell
Restart-Service -Name wazuh
```

✅ Detects:

* File creation
* Modification (with exact content changes)
* Deletion

---

### 🧬 Registry Monitoring (Windows)

1. Identify registry key:

```
HKLM\SOFTWARE\Classes\YourKey
```

2. Add:

```xml
<windows_registry>Your_Key_Name</windows_registry>
```

3. Reduce scan interval:

```xml
<frequency>30</frequency>
```

4. Restart service

✅ Detects:

* Key creation
* Modification
* Deletion

---

### 🚫 Active Response (Auto Blocking)

Configure in dashboard:

* Command: `firewall-drop`
* Location: `server`
* Timeout: `180`
* Rule ID: `5710` (failed login attempts)

Restart manager

✅ Automatically blocks malicious IPs (e.g., brute-force attacks)

---

### 🛠️ Vulnerability Detection

Enable in configuration:

```xml
<enabled>yes</enabled>
```

Restart:

* Wazuh Manager
* Agents

✅ Detects:

* CVEs based on installed packages
* Outdated software

---

## 📢 Phase 4: Slack Alert Integration

1. Create Slack channel (e.g., `wazuh-alerts`)
2. Generate Incoming Webhook URL
3. Add integration:

```xml
<integration>
  <name>slack</name>
  <hook_url>YOUR_WEBHOOK_URL</hook_url>
  <rule_id>5710</rule_id>
</integration>
```

Restart manager

✅ Sends real-time alerts to Slack

---

## 🚨 Key Security Use Cases Demonstrated

* Brute-force attack detection (RDP / SSH)
* Unauthorized file modification tracking
* Registry persistence detection
* Automated IP blocking
* Vulnerability exposure monitoring
* Real-time alerting pipeline

---

## 📊 Results

* Centralized logging from multiple endpoints
* Real-time alert generation
* Automated threat response
* Visual dashboards for attack monitoring

---

## 🧠 What This Project Proves

This is not just a setup — it demonstrates:

* SIEM deployment from scratch
* Log ingestion and correlation
* Threat detection and response
* Endpoint monitoring (Windows & Linux)
* SOC-level alerting workflow

---

## ⚠️ Limitations

* Single-node setup (not scalable for enterprise)
* Self-signed certificates (not production-grade)
* Basic rule-based detection (no advanced ML)

---

## 🚀 Future Improvements

* Multi-node cluster deployment
* Integration with threat intelligence feeds
* Custom detection rules
* Advanced dashboards (Kibana/KQL)
* Attack simulation (MITRE ATT&CK mapping)

---
