# 💉 SQL Injection (SQLi) — Web Attack Analysis

![Category](https://img.shields.io/badge/Category-Web%20Attacks-red?style=flat-square)
![Technique](https://img.shields.io/badge/MITRE%20ATT%26CK-T1190-purple?style=flat-square)
![Platform](https://img.shields.io/badge/Platform-LetsDefend%20%7C%20DVWA-blue?style=flat-square)
![Status](https://img.shields.io/badge/Cases-Active-brightgreen?style=flat-square)
![Skill](https://img.shields.io/badge/Role-SOC%20Analyst-informational?style=flat-square)

---

## 📌 Overview

SQL Injection (SQLi) is one of the most persistent and high-impact web application vulnerabilities, consistently ranked in the [OWASP Top 10](https://owasp.org/www-project-top-ten/). It allows an attacker to manipulate backend database queries by injecting malicious SQL through unsanitized user inputs — potentially exposing, modifying, or destroying database contents entirely.

This section of the repository contains structured case studies documenting the **detection, log analysis, and SOC investigation** of SQL Injection attacks. Each case follows a consistent analytical framework mapping findings to MITRE ATT&CK.

---

## 🎯 Objectives

- Understand SQL Injection attack mechanics from an analyst perspective
- Analyze real-world attack patterns from web server logs
- Identify SQLi technique variants and their behavioral signatures
- Extract Indicators of Compromise (IoCs) from log evidence
- Determine attack success, scope, and potential impact
- Practice SOC triage and incident investigation workflows

---

## 🧠 How SQL Injection Works

SQL Injection occurs when user-supplied input is concatenated directly into a SQL query without validation or parameterization. The attacker's input becomes part of the query logic rather than just data.

**Vulnerable query:**
```sql
SELECT * FROM users WHERE id = '$input';
```

**Attacker input:**
```
' OR 1=1 -- -
```

**Resulting query:**
```sql
SELECT * FROM users WHERE id = '' OR 1=1 -- -';
```

> `OR 1=1` is always true → all rows are returned. `-- -` comments out the rest of the original query. Authentication is bypassed.

---

## ⚙️ SQLi Technique Variants

### 1. Union-Based Injection
Appends a `UNION SELECT` statement to retrieve data from other tables. Requires matching column count and data types.

```sql
' UNION SELECT null, username, password FROM users -- -
```
**Analyst signal:** `UNION`, `SELECT`, column-count probing via `ORDER BY n`

---

### 2. Error-Based Injection
Deliberately triggers database error messages that leak schema details, table names, or version info.

```sql
' AND extractvalue(1, concat(0x7e, version())) -- -
```
**Analyst signal:** Repeated `500` responses, database error strings in response body

---

### 3. Boolean-Based Blind Injection
No output is returned — attacker infers data by observing true/false differences in application behavior.

```sql
' AND SUBSTRING(username,1,1)='a' -- -   → page loads normally = TRUE
' AND SUBSTRING(username,1,1)='b' -- -   → page changes = FALSE
```
**Analyst signal:** High-volume near-identical requests with minor payload variations

---

### 4. Time-Based Blind Injection
Injects conditional `SLEEP()` or `WAITFOR DELAY` to infer data when there's no visible response difference.

```sql
' AND IF(1=1, SLEEP(5), 0) -- -
```
**Analyst signal:** Abnormally high response times, repeated delay-correlated requests from single IP

---

## 🔄 Attack Lifecycle

```
[1] Reconnaissance          Identify injectable parameters (forms, URLs, headers)
        ↓
[2] Fingerprinting          Test with ' " -- to observe error behavior
        ↓
[3] Technique Selection     Union-based? Blind? Error-based? Based on response
        ↓
[4] Exploitation            Extract schema, tables, rows using chosen method
        ↓
[5] Post-Exploitation       Dump credentials, bypass auth, or escalate access
```

| Phase               | Attacker Goal                                 | SOC Detection Opportunity                        |
|---------------------|-----------------------------------------------|--------------------------------------------------|
| Reconnaissance      | Find input fields that interact with DB       | Single-quote errors, unusual parameter tampering |
| Fingerprinting      | Identify DB type and error verbosity          | 500 errors on quote injection                    |
| Exploitation        | Extract data using query manipulation         | `UNION`, `SELECT`, `OR 1=1` in access logs       |
| Post-Exploitation   | Pivot, persist, or exfiltrate credentials     | Data volume anomalies, auth bypass patterns      |

---

## 🚨 Indicators of Compromise (IoCs)

### Log Signatures to Hunt For

| Indicator                        | Example                                      | Context                          |
|----------------------------------|----------------------------------------------|----------------------------------|
| SQL metacharacters               | `'`, `"`, `;`, `--`, `/* */`                | Query termination / comment      |
| Tautology payloads               | `OR 1=1`, `OR 'a'='a'`                      | Authentication bypass            |
| UNION injection strings          | `UNION SELECT`, `UNION ALL SELECT`           | Data extraction                  |
| Blind inference patterns         | `AND SLEEP(5)`, `WAITFOR DELAY`             | Time-based blind SQLi            |
| URL-encoded variants             | `%27` (`'`), `%3D` (`=`), `%20` (space)    | WAF/filter evasion               |
| High request volume, single IP   | 200+ requests to same endpoint              | Automated tool (sqlmap, etc.)    |
| Sequential parameter fuzzing     | `id=1`, `id=2`, ..., `id=1'`, `id=1 AND..` | Manual or scripted enumeration   |

### Splunk Detection Query (Baseline)

```spl
index=web_logs
| search uri="*UNION*" OR uri="*SELECT*" OR uri="*OR+1%3D1*" OR uri="*%27*"
| stats count by src_ip, uri, status
| where count > 10
| sort - count
```

---

## 📂 Case Studies

| Case ID | Source | Attack Type | Outcome | Link |
|---|---|---|---|---|
| Case-01 | LetsDefend | Union-Based SQLi | Detected — Confirmed Attack | [View →](./Case-01-LetsDefend-SQLi-Detection/) |

> Additional cases will be added as modules are completed on LetsDefend and DVWA lab exercises are documented.

---

## 🗺️ MITRE ATT&CK Mapping

| Technique ID | Name | Relevance |
|---|---|---|
| [T1190](https://attack.mitre.org/techniques/T1190/) | Exploit Public-Facing Application | Primary SQLi delivery method |
| [T1078](https://attack.mitre.org/techniques/T1078/) | Valid Accounts | Post-exploitation after credential dump |
| [T1005](https://attack.mitre.org/techniques/T1005/) | Data from Local System | Extracting DB contents via injection |
| [T1027](https://attack.mitre.org/techniques/T1027/) | Obfuscated Files or Information | URL-encoding payloads to evade detection |

---

## 🛡️ Detection & Mitigation Reference

| Control | Type | Description |
|---|---|---|
| Parameterized queries | Prevention | Separates SQL logic from user data — eliminates injection |
| Input validation & sanitization | Prevention | Whitelist expected formats; reject metacharacters |
| WAF (Web Application Firewall) | Detection/Prevention | Signature-based blocking of known SQLi patterns |
| Least privilege DB accounts | Containment | Limits damage if injection succeeds |
| Verbose error suppression | Hardening | Prevents error-based information leakage |
| SIEM log monitoring | Detection | Alert on SQLi signatures in access logs |

---

## 🧰 Tools & Platforms

| Tool / Platform | Purpose |
|---|---|
| [LetsDefend](https://letsdefend.io) | Guided SOC analyst case investigations |
| [DVWA](https://github.com/digininja/DVWA) | Hands-on SQLi exploitation in a safe lab |
| Apache2 Access Logs | Primary log source for web attack analysis |
| Splunk | Log ingestion, searching, and alert creation |
| Suricata | Network-level detection of SQLi patterns |

---

## 📁 Folder Structure

```
sql-injection/
├── README.md                              ← This file — technique overview & index
├── Case-01-LetsDefend-SQLi-Detection/
│   ├── README.md                          ← Full case investigation writeup
│   └── screenshots/                       ← Evidence artifacts
└── lab-dvwa/
    ├── README.md                          ← DVWA hands-on lab documentation
    └── screenshots/
```

---

## 📚 References

- [OWASP — SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
- [MITRE ATT&CK — T1190](https://attack.mitre.org/techniques/T1190/)
- [PortSwigger Web Security Academy — SQLi](https://portswigger.net/web-security/sql-injection)
- [NIST NVD — CWE-89](https://cwe.mitre.org/data/definitions/89.html)

---

*Part of an ongoing SOC analyst portfolio. Techniques are documented as they are studied and practiced — depth over breadth, evidence over theory.*
