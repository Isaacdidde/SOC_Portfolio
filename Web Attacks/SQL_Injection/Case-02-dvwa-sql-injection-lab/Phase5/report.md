# 🚨 Phase 5: Detection & Monitoring – SQL Injection using Wazuh

---

# 🧾 Executive Summary

This project simulates a **SQL Injection attack** on a vulnerable DVWA web application and demonstrates how it is detected using **Wazuh SIEM**.

The attack was executed using an automated tool (sqlmap), generating malicious HTTP requests. These were successfully detected through log monitoring and custom Wazuh rules.

🔑 **Key Outcome:**

* Attack successfully detected
* High-severity alerts generated
* Mapped to MITRE ATT&CK (T1190)
* Demonstrates real SOC detection workflow

---

# 📌 1. Overview

This phase focuses on identifying and analyzing SQL Injection attacks through:

* Log monitoring
* SIEM alerting
* Rule-based detection

---

# 🎯 2. Objectives

* Simulate SQL Injection attack
* Detect attack using Wazuh
* Analyze logs and alerts
* Build custom detection rules
* Document SOC investigation

---

# 🛠️ 3. Environment Setup

| Component  | Details             |
| ---------- | ------------------- |
| Target     | DVWA (Ubuntu 22.04) |
| SIEM       | Wazuh               |
| Agent      | Installed & Active  |
| Web Server | Apache2             |
| Database   | MySQL               |
| Attacker   | Kali Linux          |
| Tool       | sqlmap              |

---

# ⚔️ 4. Attack Simulation

### Command Used:

```bash
sqlmap -u "http://<target-ip>/DVWA/vulnerabilities/sqli/?id=1&Submit=Submit" --cookie="PHPSESSID=<session>; security=low" --dbs --batch
```

### Attack Behavior:

* Automated SQL Injection
* High request frequency
* Database enumeration

---

# 🕒 5. Attack Timeline (SOC Perspective)

| Time | Event                        |
| ---- | ---------------------------- |
| T0   | Attacker starts sqlmap       |
| T1   | Multiple HTTP requests sent  |
| T2   | Apache logs capture payloads |
| T3   | Wazuh agent forwards logs    |
| T4   | Wazuh manager analyzes logs  |
| T5   | Custom rule triggered        |
| T6   | Alert generated in dashboard |

---

# 🎯 6. Indicators of Compromise (IOCs)

| Indicator Type | Value                         |
| -------------- | ----------------------------- |
| Source IP      | (Redacted)                    |
| Target URL     | `/DVWA/vulnerabilities/sqli/` |
| User-Agent     | sqlmap                        |
| Attack Pattern | UNION SELECT                  |
| Behavior       | High-frequency requests       |
| Payload Type   | SQL Injection                 |

---

# 🔍 7. Detection in Wazuh

### Alert Details:

* **Rule ID:** 100101
* **Level:** 12 (High)
* **Description:** SQLMap automated attack detected

---

# 📊 8. Alert Correlation

### Observed Pattern:

1. Repeated HTTP requests
2. SQL keywords in URL
3. sqlmap user-agent
4. High request rate

### Correlation Logic:

```text
Multiple suspicious requests
+ SQL keywords
+ Known attack tool signature (sqlmap)
= Confirmed SQL Injection attack
```

---

# 📸 9. Evidence

### 🔹 Wazuh Alerts

![Alerts](screenshots/alerts.png)

### 🔹 Logs Evidence

![Logs](screenshots/logs.png)

### 🔹 Dashboard

![Dashboard](screenshots/dashboard.png)

---

# 🧾 10. Log Analysis

### Sample Log:

```
GET /DVWA/vulnerabilities/sqli/?id=1...
User-Agent: sqlmap
```

### Key Indicators:

* Automated scanning behavior
* SQL keywords (SELECT, UNION)
* Repeated requests

---

# 🧠 11. Custom Detection Rules

```xml
<rule id="100101" level="12">
  <if_sid>31100</if_sid>
  <match>sqlmap</match>
  <description>SQLMap automated attack detected</description>
</rule>
```

---

# 🧬 12. MITRE ATT&CK Mapping

| Technique                         | ID    |
| --------------------------------- | ----- |
| Exploit Public-Facing Application | T1190 |

---

# 🚨 13. Findings

* SQL Injection attack successfully executed
* Wazuh detected attack via logs and rules
* High alert volume confirms automated attack
* Detection accuracy improved with custom rules

---

# 🛡️ 14. Mitigation Recommendations

* Input validation & sanitization
* Use prepared statements
* Deploy WAF
* Monitor logs continuously
* Block malicious IPs

---

# 🧠 15. Key Learnings

* SIEM detects patterns, not just attacks
* Logs are critical evidence
* Custom rules improve detection
* Automation leaves clear traces

---

# 🏁 16. Conclusion

This phase demonstrates a complete **attack → detection → analysis pipeline**, replicating real SOC operations. It highlights the importance of SIEM tools, log monitoring, and detection engineering.

---
