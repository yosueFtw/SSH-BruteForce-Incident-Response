# Incident Response & Mitigation: SSH Brute Force Attack

## Executive Summary
This project documents the identification, analysis, and containment of an active SSH brute force attack on a production Linux server. It outlines the manual containment strategy using host-based firewalls and provides a concept for proactive automation.

## 1. Detection & Log Analysis
During a routine security review of the system authentication logs located at `/var/log/auth.log`, an anomaly was detected. A single external IP address was generating rapid, unauthorized connection attempts targeting common administrative usernames.

### Raw Log Analysis
```text
Jul 13 18:22:01 srv-prod sshd[4012]: Invalid user admin from 192.168.1.45 port 49210 ssh2
Jul 13 18:22:03 srv-prod sshd[4015]: Invalid user admin from 192.168.1.45 port 49212 ssh2
Jul 13 18:22:05 srv-prod sshd[4018]: Invalid user backup from 192.168.1.45 port 49216 ssh2
Jul 13 18:22:07 srv-prod sshd[4021]: Failed password for invalid user backup from 192.168.1.45 port 49220 ssh2
Jul 13 18:22:10 srv-prod sshd[4025]: Accepted password for root from 192.168.1.45 port 49224 ssh2
```
### Key Findings:
* **Attacker IP:** `192.168.1.45`
* **Target Usernames:** `admin`, `backup`, `root`
* **Incident Severity:** **CRITICAL**. The final log entry confirms a successful compromise (`Accepted password for root`), indicating that the attacker successfully guessed the root password.

---

## 2. Containment & Immediate Response
To stop the threat actor from executing further commands, immediate host-based isolation was performed. 

The malicious IP was dropped at the firewall level using the following `iptables` command:
```bash
sudo iptables -A INPUT -s 192.168.1.45 -j DROP
