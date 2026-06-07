# 🚨 Case 01 – SQL Injection Detection (LetsDefend Lab)

## 📌 Overview

This case study documents the detection and analysis of a SQL Injection (SQLi) attack using web server access logs.
The investigation was performed as part of the LetsDefend SOC Analyst training module.

---

## 🎯 Objective

* Analyze HTTP logs to identify malicious activity
* Determine the attack timeline and phases
* Identify attacker IP
* Classify the SQL Injection type
* Confirm whether the attack was successful

---

## 📂 Data Source

* Apache web server access logs
* Includes timestamps, IP addresses, HTTP requests, and response codes

---

## 🚨 Attacker Details

* **Source IP:** 192.168.31.167
* **Target Application:** DVWA (Damn Vulnerable Web Application)
* **Attack Vector:** URL parameter (`id`)

---

## 🕒 Attack Timeline

| Time     | Activity                 | Stage                 |
| -------- | ------------------------ | --------------------- |
| 08:34:57 | Accessed SQLi page       | Reconnaissance        |
| 08:35:01 | Input `id=1`             | Normal behavior       |
| 08:35:05 | Input `id=2`             | Normal behavior       |
| 08:35:14 | Injected `'`             | Vulnerability Testing |
| 08:37:10 | `' OR 1=1 -- -`          | Exploitation          |
| 08:38:16 | `UNION SELECT version()` | Data Extraction       |
| 08:40:26 | `UNION SELECT user()`    | Advanced Enumeration  |

---

## 🔍 Attack Analysis

### 🔹 Phase 1: Reconnaissance

The attacker accessed the SQLi page and tested normal inputs to understand application behavior.

---

### 🔹 Phase 2: Vulnerability Testing

```http
?id=%27
```

* `%27` represents a single quote `'`
* Used to break SQL query and check for injection vulnerability

---

### 🔹 Phase 3: Exploitation

```http
?id=' OR 1=1 -- -
```

* Injected condition always evaluates to TRUE
* Bypasses authentication / query restrictions

---

### 🔹 Phase 4: Post-Exploitation (Data Extraction)

```http
?id=' OR 1=1 UNION SELECT null, version() -- -
?id=' OR 1=1 UNION SELECT null, user() -- -
```

* Extracts database version and user information
* Confirms full SQL Injection exploitation

---

## ⚠️ Type of SQL Injection

* **Union-Based SQL Injection**

---

## ✅ Was the Attack Successful?

**Yes, the attack was successful.**

### ✔ Evidence:

* Server responded with **HTTP 200 (successful requests)**
* Injection payloads executed without errors
* Database information was retrieved (`version()`, `user()`)
* No filtering or blocking mechanisms observed

---

## 🧠 Key Findings

* Application is vulnerable to SQL Injection due to lack of input validation
* Attacker successfully escalated from testing to data extraction
* Sensitive database information was exposed

---

## 🛡️ Recommendations

* Use parameterized queries / prepared statements
* Implement strict input validation and sanitization
* Deploy Web Application Firewall (WAF)
* Restrict database privileges (least privilege principle)
* Enable centralized logging and monitoring (SIEM)

---

## 📚 Skills Demonstrated

* Log Analysis
* Threat Detection
* SQL Injection Identification
* Attack Lifecycle Analysis
* Incident Investigation

---

## 🏷️ Tags

`SQL Injection` `LetsDefend` `SOC Analyst` `Log Analysis` `Cybersecurity`

---

👉 Refer to `logs.txt` for raw evidence used in this investigation.

