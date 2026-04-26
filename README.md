<div align="center">

<img src="https://img.shields.io/badge/Project-Sentinel-00ACC1?style=for-the-badge&logo=shield&logoColor=white"/>
<img src="https://img.shields.io/badge/Wazuh-4.7.5-1565C0?style=for-the-badge&logo=linux&logoColor=white"/>
<img src="https://img.shields.io/badge/ELK-Stack-00BCD4?style=for-the-badge&logo=elasticsearch&logoColor=white"/>
<img src="https://img.shields.io/badge/Docker-29.x-2496ED?style=for-the-badge&logo=docker&logoColor=white"/>
<img src="https://img.shields.io/badge/Cost-$0-00C853?style=for-the-badge&logo=opensourceinitiative&logoColor=white"/>
<img src="https://img.shields.io/badge/Detection_Rate-100%25-00C853?style=for-the-badge"/>

# 🛡️ Project Sentinel
### Personal Home Lab Security Monitoring System
**A fully functional SOC-in-a-Box built from scratch using 100% free, open-source tools**

*Wazuh SIEM + ELK Stack + Docker + Kali Linux + VirtualBox*

[Features](#-features) • [Architecture](#-architecture) • [Requirements](#-requirements) • [Installation](#-installation) • [Detection Rules](#-custom-detection-rules) • [Results](#-results) • [Roadmap](#-roadmap)

</div>

---

## 📌 Overview

Project Sentinel is a low-cost, personal Security Operations Center (SOC) that brings enterprise-grade cybersecurity monitoring to home networks and lab environments. Built entirely on free tools, it provides:

- **Real-time threat detection** across multiple endpoints
- **Centralized log management** via Wazuh SIEM
- **Visual security dashboard** powered by Kibana (OpenSearch Dashboards)
- **MITRE ATT&CK framework** mapped detection rules
- **Multi-endpoint monitoring** from a single dashboard

> **Motivation:** Cyberattacks on home networks have risen by 60%+ in recent years. Individuals face the same threats as enterprises but lack access to professional SOC tools. Project Sentinel bridges this gap at zero cost.

---

## ✨ Features

| Feature | Description |
|---------|-------------|
| 🔴 Real-Time Detection | Instant alerts on brute force, port scans, privilege escalation |
| 📊 Kibana Dashboard | Custom visualizations — timeline, severity, top alerts, agent activity |
| 🗂️ MITRE ATT&CK | All rules mapped to ATT&CK techniques |
| 🤖 Multi-Endpoint | Monitor multiple VMs/machines simultaneously |
| 🐳 Docker Deployment | Easy setup via Docker Compose — no dependency conflicts |
| 💰 Zero Cost | 100% open-source — Wazuh, ELK, Docker, Kali, Ubuntu |
| 🔧 Custom Rules | Write your own XML detection rules in minutes |

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Windows 11 Host Machine                   │
│                  Browser: https://127.0.0.1:8443            │
└──────────────────────────┬──────────────────────────────────┘
                           │ VirtualBox NatNetwork1
                           │ 192.168.100.0/24
        ┌──────────────────┼──────────────────────┐
        │                  │                      │
┌───────▼────────┐ ┌───────▼────────┐ ┌──────────▼──────────┐
│ Sentinel-Server│ │  Kali Linux    │ │   Ubuntu Target     │
│ 192.168.100.5  │ │ 192.168.100.4  │ │   192.168.100.14    │
│                │ │                │ │                     │
│ ┌────────────┐ │ │ Wazuh Agent    │ │ Wazuh Agent         │
│ │Wazuh Mgr   │ │ │ (Attacker +    │ │ (Victim endpoint)   │
│ │Port 1514   │◄├─┤  Monitored)    │ │                     │
│ ├────────────┤ │ └────────────────┘ └─────────────────────┘
│ │OpenSearch  │ │
│ │Port 9200   │ │
│ ├────────────┤ │
│ │Kibana Dash │ │
│ │Port 443    │ │
│ └────────────┘ │
└────────────────┘
```

### Components

| Component | Technology | Version | IP |
|-----------|-----------|---------|-----|
| SIEM Server | Wazuh Manager | 4.7.5 | 192.168.100.5 |
| Data Indexer | OpenSearch | 2.8.0 | Container |
| Dashboard | OpenSearch Dashboards | 4.7.5 | Port 443 |
| Attacker VM | Kali Linux | 2025.4 | 192.168.100.4 |
| Target VM | Ubuntu Server | 22.04 LTS | 192.168.100.14 |
| Container Runtime | Docker | 29.x | Host |

---

## 💻 Requirements

### Hardware
| Component | Minimum | Recommended |
|-----------|---------|-------------|
| RAM | 8 GB | 16 GB |
| CPU | Quad-core 2.0 GHz | Intel i5 / Ryzen 5 |
| Storage | 80 GB free | 120 GB |
| OS | Windows/Linux/macOS | Windows 11 |

### Software
- [VirtualBox 7.x](https://www.virtualbox.org/) — Hypervisor
- [Ubuntu Server 22.04 LTS](https://ubuntu.com/download/server) — Sentinel-Server OS
- [Kali Linux](https://www.kali.org/) — Attacker/Agent VM
- Docker 29.x — installed on Sentinel-Server
- Wazuh 4.7.5 — installed via Docker Compose

---

## 🚀 Installation

### Step 1 — VirtualBox Setup

Create a NAT Network in VirtualBox:
```
File → Tools → Network Manager → NAT Networks → Create
Name: NatNetwork1
Network: 192.168.100.0/24
Enable DHCP: Yes
```

Add Port Forwarding rules:
```
Wazuh Dashboard:  127.0.0.1:8443  → 192.168.100.5:443
SSH Access:       127.0.0.1:2222  → 192.168.100.5:22
Wazuh API:        127.0.0.1:55000 → 192.168.100.5:55000
Agent Enroll:     127.0.0.1:1515  → 192.168.100.5:1515
Agent Events:     127.0.0.1:1514  → 192.168.100.5:1514
```

### Step 2 — Install Ubuntu Server (Sentinel-Server VM)

```
RAM: 4096 MB | CPU: 2 vCPUs | Disk: 50 GB
Network: NatNetwork1
Username: sentinel | Password: <your-password>
Enable OpenSSH during installation
```

### Step 3 — Install Docker on Sentinel-Server

```bash
sudo apt-get update && sudo apt-get upgrade -y
sudo apt-get install docker.io docker-compose-v2 git -y
sudo usermod -aG docker sentinel
newgrp docker
```

### Step 4 — Deploy Wazuh via Docker

```bash
git clone https://github.com/wazuh/wazuh-docker.git -b v4.7.5
cd wazuh-docker/single-node
docker compose -f generate-indexer-certs.yml run --rm generator
docker compose up -d
```

> ⚠️ **8GB RAM Fix:** Add `OPENSEARCH_JAVA_OPTS="-Xms1g -Xmx1g"` to the indexer environment in `docker-compose.yml`

> ⚠️ **Persistence Fix:** Create `/etc/tmpfiles.d/wazuh-indexer.conf` with:
> `d /var/log/wazuh-indexer 0755 wazuh-indexer wazuh-indexer -`

Verify all containers are running:
```bash
docker compose ps
# Should show 3 containers: wazuh.manager, wazuh.indexer, wazuh.dashboard
```

### Step 5 — Access Dashboard

```
URL:      https://127.0.0.1:8443
Username: admin
Password: SecretPassword
```

### Step 6 — Install Wazuh Agent on Endpoints

```bash
# Add Wazuh repository
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo gpg \
  --no-default-keyring \
  --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import
sudo chmod 644 /usr/share/keyrings/wazuh.gpg

echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] \
  https://packages.wazuh.com/4.x/apt/ stable main" | \
  sudo tee /etc/apt/sources.list.d/wazuh.list

# Install agent (replace MANAGER_IP and AGENT_NAME)
sudo apt-get update
wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.7.5-1_amd64.deb
sudo WAZUH_MANAGER='192.168.100.5' WAZUH_AGENT_NAME='my-agent' \
  dpkg -i ./wazuh-agent_4.7.5-1_amd64.deb

# Start agent
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

### Step 7 — Enable SSH Log Monitoring

```bash
# Install rsyslog
sudo apt-get install rsyslog -y
sudo bash -c 'echo "auth,authpriv.*    /var/log/auth.log" > /etc/rsyslog.conf'
sudo systemctl restart rsyslog
sudo touch /var/log/auth.log && sudo chmod 640 /var/log/auth.log

# Add to /var/ossec/etc/ossec.conf before </ossec_config>
```

```xml
<localfile>
  <log_format>syslog</log_format>
  <location>/var/log/auth.log</location>
</localfile>
```

```bash
sudo systemctl restart wazuh-agent
```

---

## 🔧 Custom Detection Rules

Deploy the custom rules file to the Wazuh Manager container:

```bash
# From Sentinel-Server
sudo docker cp local_rules.xml \
  single-node-wazuh.manager-1:/var/ossec/etc/rules/local_rules.xml

# Restart manager to load rules
sudo docker exec single-node-wazuh.manager-1 \
  /var/ossec/bin/wazuh-control restart
```

### Rules Summary

| Rule ID | Name | Level | MITRE | Trigger |
|---------|------|-------|-------|---------|
| 100001 | Multiple SSH Failures | 5 | T1110 | SSH auth failures from same IP |
| 100002 | SSH Brute Force | 10 | T1110.001 | 8+ failures in 120 seconds |
| 100003 | Port Scan Detected | 8 | T1046 | Nmap/port scan activity |
| 100004 | Privilege Escalation | 12 | T1548.003 | Sudo misuse detected |

See [`local_rules.xml`](./local_rules.xml) for the full rule definitions.

---

## 🎯 Attack Simulation

Run these from the Kali Linux VM to test detection:

```bash
# Attack 1 — SSH Brute Force
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://TARGET_IP -t 4

# Attack 2 — Port Scan
sudo nmap -sS -sV -p 1-1000 TARGET_IP

# Attack 3 — SUID Discovery
sudo find / -perm -4000 2>/dev/null

# Attack 4 — Account Manipulation
sudo useradd -m -s /bin/bash hacker123

# Attack 5 — Log Tampering
sudo rm /var/log/auth.log
```

---

## 📊 Results

| Metric | Value |
|--------|-------|
| Total Security Events | 346+ |
| Attack Detection Rate | 100% |
| Monitored Endpoints | 2 |
| Custom Rules | 4 |
| Attack Types Simulated | 5 |
| Total Cost | $0 |

### Detected Rule IDs
`5716` `5760` `25002` `510` `533` `503` `19005` `19007` `100001` `100002`

---

## 🗺️ Roadmap

- [x] Phase 1 — VirtualBox lab environment
- [x] Phase 2 — Wazuh + ELK Stack via Docker
- [x] Phase 3 — Multi-endpoint agent deployment
- [x] Phase 4 — Custom MITRE ATT&CK detection rules
- [x] Phase 5 — Attack simulation and testing
- [x] Phase 5 — Kibana dashboard
- [ ] Phase 6 — **Cloud deployment on Oracle Cloud** (24/7 monitoring)
- [ ] Phase 7 — **AI/ML anomaly detection** (Isolation Forest + LSTM)
- [ ] Phase 8 — **Mobile push notifications** (Android/iOS)
- [ ] Phase 9 — **Automated threat response** (SOAR)
- [ ] Phase 10 — **Small business extension** (50+ endpoints)

---

## 🧠 Future Enhancement — AI/ML Detection

The next major enhancement integrates machine learning alongside rule-based detection:

```python
# Planned implementation using scikit-learn
from sklearn.ensemble import IsolationForest

# Train on normal behavior captured from Wazuh events
model = IsolationForest(contamination=0.1)
model.fit(normal_events)

# Flag anomalies in real time
predictions = model.predict(new_events)
# -1 = anomaly (potential threat), 1 = normal
```

**Goal:** Detect zero-day threats that signature rules miss by learning normal behavior patterns from existing Wazuh event data.

---

## ☁️ Future Enhancement — Cloud Deployment

```bash
# Deploy on Oracle Cloud Free Tier (24GB RAM — always free)
# Same Docker Compose — just runs on cloud VM instead of local
git clone https://github.com/SibghatUllah7/project-sentinel
cd project-sentinel/docker
docker compose up -d
# Access from anywhere: https://YOUR_ORACLE_IP:443
```

---

## 🛠️ Tech Stack

![Wazuh](https://img.shields.io/badge/Wazuh-SIEM-1565C0?style=flat-square)
![Elasticsearch](https://img.shields.io/badge/OpenSearch-Indexer-005571?style=flat-square)
![Kibana](https://img.shields.io/badge/Kibana-Dashboard-E8478B?style=flat-square)
![Docker](https://img.shields.io/badge/Docker-Container-2496ED?style=flat-square)
![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04-E95420?style=flat-square)
![Kali](https://img.shields.io/badge/Kali-Linux-557C94?style=flat-square)
![VirtualBox](https://img.shields.io/badge/VirtualBox-7.x-183A61?style=flat-square)

---

## 👥 Author

**SibghatUllah** — [@SibghatUllah7](https://github.com/SibghatUllah7)

Built as a personal cybersecurity project. Feel free to fork, use, and improve!

---

## 📄 License

This project is open source under the [MIT License](LICENSE).

---

<div align="center">
<i>If this project helped you, please give it a ⭐ on GitHub!</i>
</div>
