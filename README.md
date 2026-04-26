# 🛡️ Web Application Firewall Home Lab using SafeLine WAF

> A complete cybersecurity home lab demonstrating how SafeLine WAF detects and blocks real web attacks including SQL Injection, XSS, and Command Injection.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Lab Environment](#lab-environment)
- [Tools & Technologies](#tools--technologies)
- [Setup Steps](#setup-steps)
- [Attack Demonstrations](#attack-demonstrations)
- [Advanced WAF Configurations](#advanced-waf-configurations)
- [Key Takeaways](#key-takeaways)
- [References](#references)

---

## Overview

This lab was built to demonstrate how a **Web Application Firewall (WAF)** works in a real attack-and-defend scenario. Using two virtual machines — Kali Linux as the attacker and Ubuntu Server as the target — SafeLine WAF was deployed as a reverse proxy in front of DVWA (Damn Vulnerable Web Application) to intercept and block malicious traffic.

---

## Lab Environment

| VM | IP Address | Role |
|---|---|---|
| Kali Linux | 10.0.2.5 | Attacker |
| Ubuntu Server 22.04 | 10.0.2.15 | Target / Server |

**Network Type:** VirtualBox NAT Network (both VMs isolated but internet-accessible)

**Traffic Flow:**
```
Kali (Attacker) → SafeLine WAF (port 443) → DVWA on Apache (port 8080)
```

---

## Tools & Technologies

| Tool | Purpose |
|---|---|
| VirtualBox | Hypervisor for running VMs |
| Kali Linux | Attacker machine |
| Ubuntu Server 22.04 LTS | Target server |
| Apache2 + PHP + MySQL | LAMP stack |
| DVWA | Intentionally vulnerable web app |
| SafeLine WAF v9.3.6 | Web Application Firewall (Docker-based) |
| OpenSSL | Self-signed SSL certificate |

---

## Setup Steps

### 1. Network Setup
- Configured both VMs on VirtualBox **NAT Network** (10.0.2.0/24)
- Verified connectivity: `ping 10.0.2.15` from Kali

### 2. Ubuntu Initial Configuration
```bash
sudo apt-get update && sudo apt-get upgrade -y
sudo apt-get install -y net-tools openssl
```

### 3. Install LAMP Stack
```bash
sudo apt-get install -y apache2 php php-mysql mysql-server git
sudo mysql_secure_installation
```

### 4. Install and Configure DVWA
```bash
cd /var/www/html
sudo git clone https://github.com/digininja/DVWA.git
sudo chown -R www-data:www-data DVWA
sudo chmod -R 755 DVWA
```
- Configured DVWA database credentials in `config.inc.php`
- Created MySQL database and user `dvwa_user`
- Added custom test data for SQL injection demos

### 5. Move DVWA to Port 8080
```bash
# /etc/apache2/ports.conf → Listen 8080
# /etc/apache2/sites-available/000-default.conf → <VirtualHost *:8080>
sudo systemctl restart apache2
```

### 6. DNS Setup
Added to `/etc/hosts` on both VMs:
```
10.0.2.15    dvwa.local  www.dvwa.local
```

### 7. Self-Signed SSL Certificate
```bash
sudo mkdir /etc/ssl/dvwa
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /etc/ssl/dvwa/dvwa.key \
  -out /etc/ssl/dvwa/dvwa.crt
```

### 8. Install SafeLine WAF
```bash
sudo bash -c "$(curl -fsSLk https://waf.chaitin.com/release/latest/manager.sh)" -- --en
```
- Admin UI available at: `https://10.0.2.15:9443`
- Imported SSL certificate into SafeLine
- Onboarded DVWA as a protected application (port 443, HTTPS, reverse proxy to port 8080)

---

## Attack Demonstrations

### SQL Injection
**Payload:**
```
1' OR '1'='1
```
**Result:** 🔴 BLOCKED by SafeLine WAF — logged as `SQL Inj` from `10.0.2.5`

---

### Cross-Site Scripting (XSS)
**Payload:**
```html
<script>alert('XSS')</script>
```
**Result:** 🔴 BLOCKED by SafeLine WAF — logged as `XSS` from `10.0.2.5`

---

### Command Injection
**Payload:**
```
; ls -la /etc
```
**Result:** 🔴 BLOCKED by SafeLine WAF — logged as `Cmd Inj` from `10.0.2.5`

---

## Advanced WAF Configurations

### HTTP Flood Defense
Enabled rate limiting rules in SafeLine:
- **Basic Access Limit** — 100 requests in 10 seconds → Anti-Bot challenge for 60 minutes
- **Basic Attack Limit** — 10 attacks in 60 seconds → blocked for 30 minutes
- **Basic Error Limit** — 10 errors in 10 seconds → blocked for 30 minutes

**Test command:**
```bash
for i in {1..200}; do curl -k https://dvwa.local/ ; done
```
**Result:** Kali IP blocked after triggering Basic Access Limit (100 req/10s)

---

### Custom Deny Rules (IP Blacklist)
- Added Kali's IP `10.0.2.5` to SafeLine blacklist
- **Result:** `Access Forbidden` — Security Detection Powered By SafeLine WAF

---

### Authentication Gateway (SSO)
- Configured SafeLine Auth Gateway on port 8443
- Created user: `labuser`
- **Result:** Visiting `https://dvwa.local:8443` prompts SafeLine login before accessing DVWA

---

## Key Takeaways

- A WAF acts as a **reverse proxy**, inspecting and filtering traffic before it reaches the application
- **Defence-in-depth** matters — combining WAF rules, rate limiting, IP blocking, and auth gateway provides multiple layers of protection
- **NAT Network** is safer than Bridged networking for labs with intentionally vulnerable apps
- SafeLine WAF successfully detected and blocked **SQL Injection, XSS, and Command Injection** attacks in real time

---

## References

- [SafeLine WAF Official Documentation](https://docs.waf.chaitin.com/en)
- [DVWA GitHub Repository](https://github.com/digininja/DVWA)
- [Original Lab Guide by Royden Rebello (The Social Dork)](https://youtu.be/N0dEC1nuWCQ)

---

*Lab completed by Halima — April 2026*
