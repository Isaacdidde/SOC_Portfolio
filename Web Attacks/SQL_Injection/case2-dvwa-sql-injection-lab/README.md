# 🛡️ SQL Injection Detection Homelab (Purple Team Project)

## 📌 Overview

This project demonstrates a complete **end-to-end SQL Injection (SQLi) detection lab** built in a homelab environment. It follows a **Purple Team approach**, combining both offensive (attacker) and defensive (SOC analyst) perspectives.

The lab simulates real-world attack scenarios against a vulnerable web application and analyzes how those attacks are logged, detected, and investigated using security monitoring techniques.

---

## 🎯 Project Objectives

* Build a realistic web attack lab environment
* Perform SQL Injection attacks manually and using automation tools
* Analyze web server logs for attack patterns
* Develop detection mindset similar to a SOC analyst
* Integrate logs into a SIEM for automated detection (Wazuh)
* Document real-world attack → detection → response workflow

---

## 🧠 Purple Team Methodology

This project follows a structured cycle:

1. **Attack Simulation (Red Team)**
2. **Log Analysis (Blue Team)**
3. **Detection Engineering (SOC)**
4. **Improvement & Re-testing (Purple Team Loop)**

---

## 🏗️ Lab Architecture

### 💻 Components

* **Attacker Machine**

  * Kali Linux
  * Tools: Browser, sqlmap

* **Target Server**

  * Ubuntu Server
  * DVWA (Damn Vulnerable Web Application)
  * Apache Web Server
  * MySQL Database

* **Detection Layer (Planned)**

  * Wazuh SIEM
  * Log ingestion & alerting

---

## 🌐 Network Design

* Multi-machine setup using VirtualBox / physical laptops
* Same network (Bridged / LAN)
* Real IP-based communication (not localhost)

---

## 🧰 Tools & Technologies

| Category        | Tool              |
| --------------- | ----------------- |
| Attacker OS     | Kali Linux        |
| Target OS       | Ubuntu Server     |
| Vulnerable App  | DVWA              |
| Web Server      | Apache            |
| Database        | MySQL             |
| Automation Tool | sqlmap            |
| SIEM            | Wazuh             |
| Logs            | Apache access.log |

---

## ⚙️ Project Phases

---

### 🔴 Phase 1: Environment Setup

* Installed Ubuntu Server
* Configured Apache, MySQL, PHP (LAMP stack)
* Deployed DVWA
* Configured database connection
* Verified application accessibility

---

### 🔴 Phase 2: Manual SQL Injection (Attacker Perspective)

* Performed authentication bypass
* Identified number of columns using `ORDER BY`
* Used `UNION SELECT` for data extraction
* Extracted:

  * Database name
  * Database user
  * Database version

---

### 🛡️ Phase 3: Log Analysis (Defender Perspective)

* Monitored Apache logs using:

  ```bash
  tail -f /var/log/apache2/access.log
  ```
* Identified malicious patterns:

  * `' OR '1'='1`
  * `UNION SELECT`
  * `ORDER BY`
* Observed URL encoding in logs (`%20`, `%27`)

---

### ⚔️ Phase 4: Automated Attack

* Use sqlmap to automate SQL Injection
* Generate high-volume attack traffic
* Observe patterns in logs

---

### 🛡️ Phase 5: SIEM Integration 

* Install Wazuh SIEM
* Forward Apache logs to Wazuh
* Visualize attacks in dashboard
* Detect SQL Injection patterns

---

### 🧠 Phase 6: Detection Engineering

* Create custom detection rules
* Identify indicators of compromise (IoCs)
* Reduce false positives

---

### 🔁 Phase 7: Purple Team Loop

* Re-run attacks
* Validate detection accuracy
* Improve rules and monitoring

---

## 🔍 Key SQL Injection Techniques Demonstrated

* Authentication bypass
* Boolean-based injection
* UNION-based injection
* Column enumeration

---

## 🛡️ Detection Indicators

| Indicator        | Description           |
| ---------------- | --------------------- |
| `' OR`           | Authentication bypass |
| `UNION SELECT`   | Data extraction       |
| `ORDER BY`       | Column discovery      |
| Encoded payloads | Obfuscated attacks    |

---

## 📂 Project Structure

```bash
web-detection/
│── case1-letsdefend/
│── case2-dvwa-sql-injection-lab/
│── README.md  <-- (this file)

---

## 🧠 Key Learnings

* SQL Injection is both simple and powerful
* Logs provide critical visibility into attacks
* Manual analysis builds strong detection intuition
* SIEM tools automate but require understanding of patterns
* Purple team approach improves both offense and defense skills

---

## 🚀 Future Enhancements

* Add Web Application Firewall (WAF)
* Simulate real-world attack scenarios
* Expand to XSS and other web attacks
* Integrate alerting and incident response workflows

---

## 🏁 Conclusion

This project provides a comprehensive understanding of SQL Injection from both attacker and defender perspectives. It bridges the gap between penetration testing and SOC operations, making it highly relevant for real-world cybersecurity roles.

---

## 👨‍💻 Author

* SOC Analyst Homelab Project
* Focus: Web Attack Detection & SIEM Monitoring

---
