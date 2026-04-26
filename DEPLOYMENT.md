# Project Sentinel — Docker Deployment Notes

## Quick Start (After Initial Setup)

### Every time you start your PC:

```bash
# 1. Start Sentinel-Server VM in VirtualBox
# 2. SSH into it from Windows PowerShell:
ssh sentinel@127.0.0.1 -p 2222

# 3. Start all Wazuh containers:
cd ~/wazuh-docker/single-node && sudo docker compose up -d

# 4. Wait 2-3 minutes then open in browser:
# https://127.0.0.1:8443
# Login: admin / SecretPassword

# 5. On each agent VM start the agent:
sudo systemctl start wazuh-agent
```

---

## Auto-Start on Boot

Add this to Sentinel-Server crontab so Docker starts automatically:

```bash
sudo crontab -e
```

Add this line:
```
@reboot cd /home/sentinel/wazuh-docker/single-node && docker compose up -d
```

---

## Container Management

```bash
# Check all containers status
sudo docker compose ps

# Start all containers
sudo docker compose up -d

# Stop all containers
sudo docker compose down

# Restart specific container
sudo docker compose restart wazuh.manager

# View logs
sudo docker compose logs --tail=50 wazuh.indexer
sudo docker compose logs --tail=50 wazuh.manager
sudo docker compose logs --tail=50 wazuh.dashboard

# Enter Wazuh Manager container
sudo docker exec -it single-node-wazuh.manager-1 bash
```

---

## Agent Management

```bash
# List all agents (run inside manager container)
sudo docker exec -it single-node-wazuh.manager-1 /var/ossec/bin/manage_agents

# Add new agent: Select A
# Extract key:   Select E
# List agents:   Select L
# Remove agent:  Select R
```

---

## Port Forwarding Reference (VirtualBox NatNetwork1)

| Service | Host | Guest |
|---------|------|-------|
| Dashboard | 127.0.0.1:8443 | 192.168.100.5:443 |
| SSH Sentinel | 127.0.0.1:2222 | 192.168.100.5:22 |
| SSH Target | 127.0.0.1:2223 | 192.168.100.14:22 |
| Wazuh API | 127.0.0.1:55000 | 192.168.100.5:55000 |
| Agent Events | 127.0.0.1:1514 | 192.168.100.5:1514 |
| Agent Enroll | 127.0.0.1:1515 | 192.168.100.5:1515 |

---

## VM IP Reference

| VM | IP | Role |
|----|----|------|
| Sentinel-Server | 192.168.100.5 | Wazuh + ELK (Docker) |
| Kali Linux | 192.168.100.4 | Attacker + Agent |
| Ubuntu Target | 192.168.100.14 | Victim endpoint + Agent |

---

## JVM Heap Fix (8GB RAM Systems)

If wazuh.indexer fails to start, check docker-compose.yml:

```yaml
wazuh.indexer:
  environment:
    - OPENSEARCH_JAVA_OPTS=-Xms1g -Xmx1g
```

Also ensure log directory exists on every boot:
```bash
# Add to /etc/tmpfiles.d/wazuh-indexer.conf:
d /var/log/wazuh-indexer 0755 wazuh-indexer wazuh-indexer -
```

---

## Troubleshooting

### Dashboard not loading
```bash
# Check all containers running
sudo docker compose ps

# Check indexer is responding
curl -k -u admin:SecretPassword https://192.168.100.5:9200

# Restart stack
sudo docker compose down && sudo docker compose up -d
```

### Agent not connecting
```bash
# Check agent status on endpoint
sudo systemctl status wazuh-agent

# Check connection log
sudo strings /var/ossec/logs/ossec.log | grep -i "connected\|error" | tail -10

# Test port connectivity
nc -zv 192.168.100.5 1514
nc -zv 192.168.100.5 1515
```

### Re-enroll agent manually
```bash
# On Sentinel-Server — extract key
sudo docker exec -it single-node-wazuh.manager-1 /var/ossec/bin/manage_agents
# Select E → enter agent ID → copy key

# On agent machine — import key
sudo rm /var/ossec/etc/client.keys
sudo /var/ossec/bin/manage_agents
# Select I → paste key → confirm

sudo systemctl restart wazuh-agent
```
