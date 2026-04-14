# 🧪 DVWA SQL Injection Lab — Phase 1: Environment Setup

![Lab Status](https://img.shields.io/badge/Status-Complete-brightgreen?style=flat-square)
![Platform](https://img.shields.io/badge/Platform-VirtualBox-blue?style=flat-square)
![OS](https://img.shields.io/badge/OS-Ubuntu%20Server-orange?style=flat-square)
![Stack](https://img.shields.io/badge/Stack-LAMP-red?style=flat-square)
![App](https://img.shields.io/badge/App-DVWA-critical?style=flat-square)
![ATT&CK](https://img.shields.io/badge/MITRE%20ATT%26CK-T1190-purple?style=flat-square)

---

## 📋 Overview

This phase establishes a controlled, intentionally vulnerable web application environment for simulating and analyzing SQL Injection attacks. The environment is built on a LAMP stack running DVWA (Damn Vulnerable Web Application), deployed on an isolated Ubuntu Server VM accessible from a separate attacker machine.

> ⚠️ **Disclaimer:** This lab is built strictly for educational and authorized security research purposes. All activity is conducted in an isolated, offline virtual environment. Never deploy DVWA on a public or production network.

---

## 🎯 Objectives

- [x] Install and configure Ubuntu Server on VirtualBox
- [x] Deploy a LAMP stack (Apache2, MySQL, PHP)
- [x] Clone and configure the DVWA application
- [x] Establish database connectivity with proper credentials
- [x] Verify network accessibility from the attacker machine

---

## 🏗️ Lab Architecture

```
┌─────────────────────────────────┐       ┌──────────────────────────────┐
│        Attacker Machine         │       │       Ubuntu Server VM        │
│                                 │       │                               │
│  Browser / Terminal             │◄─────►│  Apache2  →  DVWA (PHP)      │
│  IP: 192.168.x.x                │  LAN  │  MySQL    →  dvwa database    │
│                                 │       │  IP: 192.168.x.x              │
└─────────────────────────────────┘       └──────────────────────────────┘
                              VirtualBox Bridged Adapter
```

---

## 🧰 Technology Stack

| Component        | Technology         | Version / Notes              |
|------------------|--------------------|------------------------------|
| Hypervisor       | VirtualBox         | Bridged networking enabled   |
| Operating System | Ubuntu Server      | 2 GB RAM / 2 CPU cores       |
| Web Server       | Apache2            | Serving DVWA from `/dvwa`    |
| Database         | MySQL              | Local socket connection      |
| Backend Language | PHP + php-mysqli   | With `libapache2-mod-php`    |
| Vulnerable App   | DVWA               | Cloned from digininja/DVWA   |

---

## ⚙️ Setup Walkthrough

### 1. 🖥️ Ubuntu Server Installation

- Deployed Ubuntu Server on VirtualBox
- Allocated: **2 GB RAM**, **2 CPU cores**
- Enabled **OpenSSH Server** during setup for remote terminal access

---

### 2. 🌐 Network Configuration

Change the VM's network adapter from **NAT → Bridged Adapter** to allow LAN communication between attacker and target machines.

Verify connectivity:
```bash
ping <ubuntu-server-ip>
```

---

### 3. 🧱 LAMP Stack Installation

```bash
sudo apt update && sudo apt upgrade -y

sudo apt install apache2 mysql-server php php-mysqli php-gd libapache2-mod-php git -y
```

---

### 4. 🔧 Enable and Start Services

```bash
# Apache
sudo systemctl start apache2
sudo systemctl enable apache2

# MySQL
sudo systemctl start mysql
sudo systemctl enable mysql
```

---

### 5. 📦 Clone DVWA

```bash
cd /var/www/html
sudo git clone https://github.com/digininja/DVWA.git dvwa
```

---

### 6. 🔐 Set File Permissions

```bash
sudo chown -R www-data:www-data /var/www/html/dvwa
sudo chmod -R 755 /var/www/html/dvwa
```

---

### 7. 🗄️ Create Database and User

```bash
sudo mysql
```

```sql
CREATE DATABASE dvwa;
CREATE USER 'dvwa'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON dvwa.* TO 'dvwa'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

---

### 8. ⚙️ Configure DVWA Application

```bash
cd /var/www/html/dvwa/config
sudo cp config.inc.php.dist config.inc.php
sudo nano config.inc.php
```

Update the following values:

```php
$_DVWA['db_user']     = 'dvwa';
$_DVWA['db_password'] = 'password';
$_DVWA['db_database'] = 'dvwa';
```

---

### 9. 🔁 Restart Services

```bash
sudo systemctl restart apache2
sudo systemctl restart mysql
```

---

### 10. 🌐 Access DVWA from Attacker Machine

Open a browser and navigate to:
```
http://<ubuntu-server-ip>/dvwa
```

---

### 11. 🧪 Initialize the Database

1. Navigate to: `http://<ubuntu-server-ip>/dvwa/setup.php`
2. Scroll down and click **"Create / Reset Database"**
3. Log in with default credentials:

   | Field    | Value      |
   |----------|------------|
   | Username | `admin`    |
   | Password | `password` |

---

## ✅ Validation Checklist

| Check                                     | Status |
|-------------------------------------------|--------|
| DVWA login page loads from attacker host  | ✅     |
| Database initialized without errors       | ✅     |
| Login succeeds with default credentials   | ✅     |
| Application navigable across security levels | ✅  |

---

## 🐛 Troubleshooting

### ❌ 404 Not Found

| | |
|---|---|
| **Cause** | Incorrect URL path or case mismatch (`DVWA` vs `dvwa`) |
| **Fix** | Use lowercase: `http://<ip>/dvwa` — Linux filesystems are case-sensitive |

---

### ❌ HTTP 500 During Setup

| | |
|---|---|
| **Cause** | MySQL auth failure or credential mismatch between config and database |
| **Fix** | Run: `ALTER USER 'dvwa'@'localhost' IDENTIFIED BY 'password';` then retry |

---

### ❌ UNION Queries Return 500 Error

| | |
|---|---|
| **Cause** | Incorrect number of columns in injected UNION query |
| **Fix** | Use `ORDER BY` incrementally to enumerate column count before UNION |

---

### ❌ Attacker Machine Cannot Reach DVWA

| | |
|---|---|
| **Cause** | VM network adapter set to NAT instead of Bridged |
| **Fix** | Change to **Bridged Adapter** in VirtualBox settings; both machines must be on the same subnet |

---

### ❌ Apache Serves Default Page Instead of DVWA

| | |
|---|---|
| **Cause** | Default `index.html` overrides directory listing |
| **Fix** | `sudo rm /var/www/html/index.html` |

---

### ❌ Database Connection Failure

| | |
|---|---|
| **Cause** | `config.inc.php` missing (only `.dist` template exists) |
| **Fix** | `cp config.inc.php.dist config.inc.php` and update credentials |

---

### ❌ PHP Cannot Connect to MySQL

| | |
|---|---|
| **Cause** | `php-mysqli` extension not installed |
| **Fix** | `sudo apt install php-mysqli` then restart Apache |

---

## 🧠 Key Takeaways

- **Linux is case-sensitive** — URL paths, filenames, and directory names must match exactly
- **HTTP status codes are diagnostic signals**: `404` = path/routing issue; `500` = backend or database error
- **Credential consistency matters** — database credentials in MySQL must exactly match those in `config.inc.php`
- **VM networking requires deliberate configuration** — Bridged Adapter is essential for multi-machine lab setups
- **UNION-based SQL injection requires column count enumeration** — `ORDER BY` is the standard technique for this

---

## 🗺️ MITRE ATT&CK Mapping

| Technique ID | Name                         | Relevance                                           |
|--------------|------------------------------|-----------------------------------------------------|
| [T1190](https://attack.mitre.org/techniques/T1190/) | Exploit Public-Facing Application | DVWA simulates a publicly exposed web app with known vulnerabilities |
| [T1505.003](https://attack.mitre.org/techniques/T1505/003/) | Web Shell | DVWA includes a file upload module for web shell practice |
| [T1059](https://attack.mitre.org/techniques/T1059/) | Command and Scripting Interpreter | SQL injection can be a precursor to OS command execution |

---

## 📁 Repository Structure

```
dvwa-sqli-lab/
├── README.md                  ← Phase 1: Environment Setup (this file)
├── phase-2-attack/
│   └── README.md              ← SQL Injection techniques and payloads
├── phase-3-detection/
│   └── README.md              ← Suricata rules, Splunk queries, detection logic
└── screenshots/
    └── ...                    ← Evidence of each phase
```

---

## 🔭 Next Steps

- **Phase 2:** Perform SQL Injection attacks at varying DVWA security levels (Low → High)
- **Phase 3:** Detect and alert on SQL Injection patterns using Suricata + Splunk
- **Documentation:** Map attack findings to MITRE ATT&CK and publish to GitHub portfolio

---

## 📎 References

- [DVWA GitHub Repository](https://github.com/digininja/DVWA)
- [MITRE ATT&CK — T1190](https://attack.mitre.org/techniques/T1190/)
- [OWASP SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
- [LetsDefend SOC Training Platform](https://letsdefend.io)

---

*Part of an ongoing SOC analyst portfolio project. Built and documented by a security practitioner actively developing detection and threat analysis skills.*
