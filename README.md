# Security-assessment-report
"Comprehensive security assessment of legacy systems including Apache 2.4.7 vulnerability analysis, SMTP security audit, and investigation of suspicious port 31337. Includes CVSS scoring and remediation recommendations."

# 🔐 Vulnerability Assessment Report – Legacy Linux Server

## 📌 Overview
This report documents the results of a network reconnaissance and vulnerability assessment performed against a target Linux server. The objective was to identify open ports, running services, associated vulnerabilities, and recommend remediation measures.

## 🧰 Tools Used
- **Nmap** – Port scanning and service detection
- **Banner Grabbing** – Service version identification
- **CVE Database** – Vulnerability correlation
- **Manual analysis** – Risk assessment and verification

## 🎯 Target Information

| Field | Value |
|-------|-------|
| **Target Type** | Linux Server |
| **Scan Type** | TCP SYN Scan + Version Detection |
| **Command Used** | `nmap -sS -sV -A target_ip` |
| **Assessment Type** | Vulnerability Assessment |

## 🔎 Open Ports and Services Discovered

| Port | Service | Version | Risk Level |
|------|---------|---------|------------|
| 22 | SSH | OpenSSH 6.6.1p1 | **HIGH** |
| 80 | HTTP | Apache 2.4.7 | **HIGH** |
| 587 | SMTP | tcpwrapped | MEDIUM |
| 9929 | nping-echo | Nping Echo Service | LOW |
| 31337 | Unknown | tcpwrapped | **CRITICAL** |

## 🚨 Vulnerability Analysis

### 1️⃣ Port 22 – SSH (OpenSSH 6.6.1p1)

**Identified Vulnerabilities**

| CVE ID | Description | Impact |
|--------|-------------|--------|
| CVE-2015-5600 | Authentication brute-force bypass | Allows attacker to bypass MaxAuthTries limitation |
| CVE-2015-6563 / CVE-2015-6564 | Privilege escalation vulnerabilities | May allow attacker to gain root access |

**Exploitation Impact**
- Unauthorized access
- Privilege escalation
- System compromise

**Remediation**
```bash
# Update OpenSSH to latest version
sudo apt-get update && sudo apt-get upgrade openssh-server

# Disable password authentication
sudo sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config

# Enable key-based authentication
sudo sed -i 's/#PubkeyAuthentication yes/PubkeyAuthentication yes/' /etc/ssh/sshd_config

# Install Fail2Ban
sudo apt-get install fail2ban -y
sudo systemctl enable fail2ban
sudo systemctl start fail2ban

2️⃣ Port 80 – HTTP (Apache 2.4.7)
Identified Vulnerabilities

CVE ID	Description	Impact
CVE-2017-9798	OptionsBleed - Information disclosure vulnerability	Allows attacker to leak sensitive data
Exploitation Impact

Credential exposure

Information leakage

Memory contents disclosure

Remediation
# Check current Apache version
apache2 -v

# Update Apache to latest version
sudo apt-get update
sudo apt-get upgrade apache2

# For Ubuntu 14.04 (specific fixed version)
sudo apt-get install apache2=2.4.7-1ubuntu4.18

# Disable unnecessary modules
sudo a2dismod autoindex status info

# Restart Apache
sudo systemctl restart apache2

3️⃣ Port 587 – SMTP Service
Risk Assessment

Email spoofing

Open relay attack

Credential interception

Data exfiltration

Remediation

bash
# Edit Postfix configuration (if using Postfix)
sudo nano /etc/postfix/main.cf

# Add/enforce these settings
echo "smtpd_tls_security_level = encrypt
smtpd_sasl_auth_enable = yes
smtpd_relay_restrictions = permit_mynetworks permit_sasl_authenticated defer_unauth_destination" | sudo tee -a /etc/postfix/main.cf

# Restart service
sudo systemctl restart postfix

# Monitor SMTP traffic
sudo tail -f /var/log/mail.log
4️⃣ Port 9929 – Nping Echo Service
Risk Assessment

Network reconnaissance

Denial of Service attacks

Information disclosure

Remediation

bash
# Check if service is running
sudo netstat -tulpn | grep 9929

# Stop and disable the service
sudo systemctl stop nping
sudo systemctl disable nping

# If it's a custom service, kill the process
sudo kill $(sudo lsof -t -i:9929)

# Block port with firewall
sudo ufw deny 9929
5️⃣ Port 31337 – Suspicious Service (CRITICAL)
⚠️ WARNING: Port 31337 is historically associated with:

Back Orifice trojan

Other backdoors

Unauthorized remote access

Malware command & control

Risk Level: CRITICAL

Possible Impact

Full system compromise

Persistent attacker access

Data theft

Botnet participation

Investigation Commands

bash
# Find process using port 31337
sudo netstat -tulpn | grep 31337
sudo lsof -i :31337

# Check running processes
ps aux | grep -i "back\|orifice\|trojan"

# Scan for malware
sudo apt-get install clamav -y
sudo freshclam
sudo clamscan -r /

# Check system integrity
sudo rpm -Va  # For RHEL/CentOS
sudo debsums -c  # For Debian/Ubuntu
Immediate Remediation

bash
# Kill the process immediately
sudo fuser -k 31337/tcp

# Block the port
sudo iptables -A INPUT -p tcp --dport 31337 -j DROP
sudo iptables -A INPUT -p udp --dport 31337 -j DROP

# Save iptables rules
sudo iptables-save > /etc/iptables/rules.v4

# Check for persistence mechanisms
sudo crontab -l
sudo cat /etc/crontab
sudo ls -la /etc/rc*.d/
⚠️ Attack Scenario
An attacker could:

Reconnaissance: Discover open ports using Nmap

Initial Foothold: Identify and exploit vulnerable Apache version (OptionsBleed)

Credential Harvesting: Leak sensitive data from Apache memory

Lateral Movement: Use exposed SMTP for phishing or data exfiltration

Privilege Escalation: Exploit OpenSSH vulnerability to gain root access

Persistence: Maintain access via backdoor on port 31337

🛡️ Overall Risk Rating: HIGH
The system contains multiple outdated services with known vulnerabilities that could lead to full compromise. The presence of a suspicious service on port 31337 suggests possible prior compromise.

✅ Recommended Remediation Summary
Priority	Action	Timeline
🔴 CRITICAL	Investigate and close port 31337	Immediate
🟠 HIGH	Update OpenSSH to latest version	24 hours
🟠 HIGH	Update Apache to patched version	24 hours
🟡 MEDIUM	Secure SMTP with encryption	1 week
🟢 LOW	Disable unnecessary services	1 week
📊 CVSS Score Summary
Port	Service	Vulnerability	CVSS Score	Severity
80	Apache 2.4.7	CVE-2017-9798	7.5	HIGH
22	OpenSSH 6.6.1p1	CVE-2015-5600	6.4	MEDIUM
22	OpenSSH 6.6.1p1	CVE-2015-6563/4	7.0	HIGH
31337	Unknown	Potential malware	N/A	CRITICAL


# Restart SSH service
sudo systemctl restart sshd
