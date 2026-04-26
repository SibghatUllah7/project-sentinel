# Project Sentinel — Attack Simulation Guide

## Overview

All attacks are performed from the **Kali Linux VM** (192.168.100.4).
Targets are **Kali itself** (127.0.0.1) or **Ubuntu Target** (192.168.100.14).
Watch alerts appear live in the Wazuh dashboard at `https://127.0.0.1:8443`

---

## Prerequisites

```bash
# On Kali — create password wordlist
cat > ~/passwords.txt << 'EOF'
password
123456
admin
root
toor
kali
sentinel
test123
password123
qwerty
letmein
welcome
monkey
dragon
master
EOF

# Start SSH service on Kali
sudo systemctl start ssh
sudo systemctl enable ssh

# Install rsyslog for auth log (if not done)
sudo apt-get install rsyslog -y
sudo bash -c 'echo "auth,authpriv.*    /var/log/auth.log" > /etc/rsyslog.conf'
sudo systemctl restart rsyslog
sudo touch /var/log/auth.log && sudo chmod 640 /var/log/auth.log
```

---

## Attack 1 — SSH Brute Force

**MITRE ATT&CK:** T1110.001 — Brute Force: Password Guessing

```bash
# Attack Kali itself
hydra -l root -P ~/passwords.txt ssh://127.0.0.1 -t 4 -V

# Attack Ubuntu Target
hydra -l target -P ~/passwords.txt ssh://192.168.100.14 -t 4 -V
```

**Expected Alerts:**
- Rule 5716 — SSH authentication failure
- Rule 5760 — SSH brute force attempt
- Rule 25002 — Multiple authentication failures
- Rule 100001 — Sentinel: Multiple SSH failures (custom)
- Rule 100002 — Sentinel: SSH Brute Force detected (custom)

---

## Attack 2 — Port Scan / Reconnaissance

**MITRE ATT&CK:** T1046 — Network Service Discovery

```bash
# Basic SYN scan
sudo nmap -sS -p 1-1000 127.0.0.1

# Service version detection
sudo nmap -sS -sV -p 1-1000 192.168.100.14

# Aggressive scan
sudo nmap -A -O 192.168.100.14
```

**Expected Alerts:**
- Port scan activity logged
- Rule 100003 — Sentinel: Port scan detected (custom)

---

## Attack 3 — SUID File Discovery (Privilege Escalation Recon)

**MITRE ATT&CK:** T1548 — Abuse Elevation Control Mechanism

```bash
# Find SUID binaries
sudo find / -perm -4000 2>/dev/null

# Find world-writable files
find / -writable -type f 2>/dev/null | grep -v proc
```

**Expected Alerts:**
- Rule 510 — Host-based anomaly detection
- Rule 533 — Rootcheck anomaly

---

## Attack 4 — Account Manipulation

**MITRE ATT&CK:** T1136.001 — Create Account: Local Account

```bash
# Create suspicious user account
sudo useradd -m -s /bin/bash hacker123
sudo passwd hacker123

# Add to sudo group (privilege escalation)
sudo usermod -aG sudo hacker123

# Cleanup after demo
sudo userdel -r hacker123
```

**Expected Alerts:**
- New user creation alert
- Rule 100005 — Sentinel: New account created (custom)

---

## Attack 5 — Log Tampering

**MITRE ATT&CK:** T1070.002 — Indicator Removal: Clear Linux Logs

```bash
# Delete auth log (evidence tampering simulation)
sudo rm /var/log/auth.log

# Clear bash history
history -c && cat /dev/null > ~/.bash_history

# Recreate auth.log after demo
sudo touch /var/log/auth.log && sudo chmod 640 /var/log/auth.log
sudo systemctl restart rsyslog
```

**Expected Alerts:**
- File integrity monitoring alert
- Rule 100006 — Sentinel: Log file deletion detected (custom)

---

## Attack 6 — Cron Persistence

**MITRE ATT&CK:** T1053.003 — Scheduled Task: Cron

```bash
# Add malicious cron job (simulation only)
echo "*/5 * * * * root /tmp/backdoor.sh" | sudo tee /etc/cron.d/malicious

# Cleanup
sudo rm /etc/cron.d/malicious
```

**Expected Alerts:**
- Cron modification alert
- File integrity change detected

---

## Watching Alerts in Dashboard

1. Open `https://127.0.0.1:8443`
2. Login: `admin` / `SecretPassword`
3. Go to **Agents** → select agent → **Security Events**
4. Set time range to **Last 15 minutes**
5. Watch alerts appear in real time as attacks run

### Useful Dashboard Views
- **Security Events** — full alert table
- **MITRE ATT&CK** — techniques mapped to framework
- **Policy Monitoring** — compliance score
- **Integrity Monitoring** — file changes

---

## Expected Dashboard Metrics After Full Simulation

| Metric | Expected Value |
|--------|---------------|
| Total Events | 300-500+ |
| Brute Force Alerts | 50-100+ |
| Rootcheck Alerts | 10-30 |
| Policy Alerts | 20-50 |
| Custom Rule Alerts | 10-20 |
| MITRE Techniques | T1110, T1046, T1548, T1136, T1070 |

---

> ⚠️ **Legal Notice:** All attacks are performed in an isolated virtual lab environment for educational purposes only. Never run these tools against systems you don't own.
