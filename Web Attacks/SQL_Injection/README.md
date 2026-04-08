# 💉 SQL Injection (SQLi) – Web Attack Analysis

## 📌 Overview

SQL Injection (SQLi) is a critical web application vulnerability that allows an attacker to manipulate backend database queries by injecting malicious SQL code through user inputs.

This section of the repository contains multiple case studies demonstrating detection, analysis, and investigation of SQL Injection attacks from a SOC (Security Operations Center) perspective.

---

## 🎯 Objectives

* Understand how SQL Injection attacks work
* Analyze real-world attack patterns from logs
* Identify different SQLi techniques
* Detect indicators of compromise (IoCs)
* Determine attack success and impact

---

## 🧠 What is SQL Injection?

SQL Injection occurs when user input is not properly validated or sanitized, allowing attackers to alter SQL queries.

### 🔍 Example:

```sql
SELECT * FROM users WHERE id = '1'
```

Injected input:

```sql
' OR 1=1 -- -
```

Resulting query:

```sql
SELECT * FROM users WHERE id = '' OR 1=1 -- -
```

👉 This returns all records, bypassing authentication.

---

## ⚠️ Types of SQL Injection

### 🔹 1. Union-Based SQL Injection

* Uses `UNION SELECT` to extract data from the database
* Example:

```sql
' UNION SELECT null, version() -- -
```

---

### 🔹 2. Error-Based SQL Injection

* Exploits database error messages to gather information
* Example:

```sql
'
```

---

### 🔹 3. Boolean-Based Blind SQL Injection

* Uses true/false conditions to infer data
* Example:

```sql
' AND 1=1 -- -
```

---

### 🔹 4. Time-Based Blind SQL Injection

* Uses delays to extract information
* Example:

```sql
' AND SLEEP(5) -- -
```

---

## 🔄 Attack Lifecycle

| Phase                 | Description                          |
| --------------------- | ------------------------------------ |
| Reconnaissance        | Identify input fields and parameters |
| Vulnerability Testing | Inject `'` to test query behavior    |
| Exploitation          | Use payloads like `OR 1=1`           |
| Post-Exploitation     | Extract data using `UNION SELECT`    |

---

## 🚨 Indicators of SQL Injection (Log-Based)

* Presence of `'`, `"`, `--`, `/* */`
* Keywords like `OR`, `UNION`, `SELECT`
* Encoded payloads (`%27`, `%3D`)
* Unusual query patterns in URLs
* Repeated requests with different payloads

---

## 📂 Case Studies

| Case                              | Description                                              |
| --------------------------------- | -------------------------------------------------------- |
| Case-01-LetsDefend-SQLi-Detection | Log-based detection and analysis of SQL Injection attack |

---

## 🛡️ Mitigation & Prevention

* Use parameterized queries / prepared statements
* Implement input validation and sanitization
* Deploy Web Application Firewall (WAF)
* Apply least privilege principle on databases
* Enable logging and real-time monitoring (SIEM)

---

## 🧰 Tools & Platforms Used

* LetsDefend (SOC Analyst Lab)
* DVWA (Damn Vulnerable Web Application)
* Web Server Logs (Apache/Nginx)

---

## 📚 Skills Demonstrated

* Log Analysis
* Threat Detection
* Web Attack Identification
* Incident Investigation
* SOC Workflow Understanding

---

## 🏷️ Tags

`SQL Injection` `Web Security` `SOC Analyst` `Blue Team` `Cybersecurity`

---

👉 Navigate into each case folder to view detailed analysis and investigation.

