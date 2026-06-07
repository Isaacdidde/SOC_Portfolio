# SQL Injection Detection — Purple Team Homelab

**Author:** Didde Isaac | SOC Analyst Portfolio  
**Focus:** Web Attack Detection, Log Analysis & SIEM Monitoring  
**Approach:** Purple Team (Offensive Simulation + Defensive Detection)

---

## Overview

This project documents a complete end-to-end SQL Injection (SQLi) detection lab built in a homelab environment. It follows a **Purple Team methodology** — combining attacker simulation with SOC analyst detection workflows — to demonstrate real-world threat identification, log analysis, and SIEM-based alerting.

---

## Objectives

- Build a realistic web attack lab using DVWA on a LAMP stack
- Perform SQL Injection attacks manually and via automation (sqlmap)
- Analyze Apache access logs to identify attack patterns
- Forward logs to a SIEM (Wazuh) for automated detection
- Engineer custom detection rules and reduce false positives
- Document a complete attack → detection → response workflow

---

## Lab Architecture

| Role | OS | Tools / Services |
|---|---|---|
| Attacker Machine | Kali Linux | Browser, sqlmap |
| Target Server | Ubuntu Server | DVWA, Apache, MySQL |
| Detection Layer | Ubuntu Server | Wazuh SIEM |

**Network:** Multi-machine setup via VirtualBox / physical hardware. Bridged/LAN networking with real IP-based communication.

---

## Tools & Technologies

| Category | Tool |
|---|---|
| Attacker OS | Kali Linux |
| Target OS | Ubuntu Server |
| Vulnerable App | DVWA |
| Web Server | Apache |
| Database | MySQL |
| Automation | sqlmap |
| SIEM | Wazuh |
| Log Source | Apache `access.log` |

---

## Purple Team Methodology

```
Attack Simulation (Red)  →  Log Analysis (Blue)  →  Detection Engineering (SOC)
        ↑                                                        |
        └──────────────── Purple Team Loop ─────────────────────┘
```

Each cycle re-runs attacks, validates detection accuracy, and improves rules.

---

## 📂 Project Phases

---

### 🔴 Phase 1 — Environment Setup

* Installed Ubuntu Server and configured LAMP stack (Apache, MySQL, PHP)
* Deployed DVWA and configured database connectivity
* Verified application accessibility from attacker machine (Kali Linux)

> **MITRE ATT&CK:**
> No mapping — this phase involves infrastructure setup and does not represent adversary behavior.

---

### 🔴 Phase 2 — Manual SQL Injection (Red Team + Initial Blue Visibility)

* Performed authentication bypass using `' OR '1'='1`
* Enumerated database structure using `ORDER BY`
* Extracted database name, user, and version using `UNION SELECT`
* Monitored Apache logs to observe attack traces

```bash
tail -f /var/log/apache2/access.log
```

> **MITRE ATT&CK Mapping**
>
> | Technique                         | ID    | Description                                                |
> | --------------------------------- | ----- | ---------------------------------------------------------- |
> | Exploit Public-Facing Application | T1190 | Manual SQL Injection against DVWA                          |
> | Data from Local System            | T1005 | Extracted database information using UNION-based injection |

---

### 🛡️ Phase 3 — Log Analysis & Detection Thinking (Blue Team)

* Analyzed Apache access logs for SQL Injection indicators
* Identified suspicious patterns:

  * `' OR '1'='1`
  * `UNION SELECT`
  * `ORDER BY`
  * URL-encoded payloads (`%27`, `%20`)
* Correlated attacker actions with log entries

> **MITRE ATT&CK Mapping (Observed Activity)**
>
> | Technique                         | ID    | Description                                            |
> | --------------------------------- | ----- | ------------------------------------------------------ |
> | Exploit Public-Facing Application | T1190 | SQL Injection attempts identified through log analysis |

> 🔍 **Note:**
> This phase focuses on **detecting attacker techniques**, not mapping defender actions.

---

### ⚔️ Phase 4 — Automated Attack (sqlmap)

* Executed automated SQL Injection using sqlmap
* Generated high-frequency attack traffic
* Compared manual vs automated attack patterns in logs
* Observed:

  * Repeated requests
  * Encoded payloads
  * Faster enumeration attempts

> **MITRE ATT&CK Mapping**
>
> | Technique                         | ID    | Description                                |
> | --------------------------------- | ----- | ------------------------------------------ |
> | Exploit Public-Facing Application | T1190 | Automated SQL Injection using sqlmap       |
> | Obfuscated Files or Information   | T1027 | Encoded payloads used in automated attacks |

---

### 🛡️ Phase 5 — SIEM Integration & Detection Engineering *(In Progress)*

* Install Wazuh agent on DVWA server
* Forward Apache logs to Wazuh manager
* Visualize attack activity in SIEM dashboard
* Develop custom detection rules for SQL Injection
* Define Indicators of Compromise (IOCs):

  * `UNION SELECT`
  * `' OR '1'='1`
  * Encoded payloads (`%27`, `%3D`)
* Tune rules to reduce false positives

> **MITRE ATT&CK Mapping (Detection Focus)**
>
> | Technique Detected                | ID    | Rule Purpose                               |
> | --------------------------------- | ----- | ------------------------------------------ |
> | Exploit Public-Facing Application | T1190 | Detect SQL Injection attempts in HTTP logs |
> | Obfuscated Files or Information   | T1027 | Detect encoded payloads in requests        |

---

### 🔁 Phase 6 — Purple Team Validation Loop

* Re-execute attack scenarios after rule deployment
* Validate detection coverage and accuracy
* Identify gaps in detection logic
* Improve and refine rules iteratively

> **MITRE ATT&CK:**
> Reinforces coverage of previously mapped techniques (T1190, T1027)

---

## 🧠 Summary

This phased approach demonstrates a complete **attack → observe → detect → improve** cycle aligned with real-world SOC workflows:

* **Red Team:** Exploitation (SQL Injection)
* **Blue Team:** Log analysis and detection
* **Purple Team:** Continuous improvement and validation

---


## MITRE ATT&CK Mapping

| Technique | ID | Description |
|---|---|---|
| Exploit Public-Facing Application | T1190 | SQL Injection via DVWA |
| OS Command Execution (via SQLi) | T1059 | Potential command execution through DB |
| Data from Local System | T1005 | Database enumeration and extraction |

---

## Detection Indicators (IOCs)

| Indicator | Pattern | Threat |
|---|---|---|
| Authentication Bypass | `' OR '1'='1` | Login bypass |
| Data Extraction | `UNION SELECT` | Column-based data dump |
| Column Enumeration | `ORDER BY [n]--` | Pre-extraction reconnaissance |
| Encoded Payloads | `%27`, `%20`, `%3D` | Obfuscated injection |

---

## Key Learnings

- Manual SQL Injection builds intuition that tools like sqlmap can't replace
- Apache logs expose attack patterns clearly once you know what to look for
- URL encoding is a common obfuscation technique visible in raw log entries
- SIEM rules must account for both raw and encoded payload variants
- Purple Team loops are more effective than single red or blue exercises

---

## Repository Structure

```
SOC_Portfolio/
└── web-detection/
    ├── case1-letsdefend/          # LetsDefend alert investigation write-up
    ├── case2-dvwa-sql-injection-lab/  # This lab (setup, logs, rules)
    └── README.md
```

---

## Future Enhancements

- [ ] Complete Wazuh SIEM integration and share custom rule set
- [ ] Add Web Application Firewall (WAF) — ModSecurity
- [ ] Expand to XSS and Command Injection scenarios
- [ ] Integrate automated alerting and incident response playbook
- [ ] Add pcap captures for network-level evidence

---

## Connect

If you're a recruiter or fellow learner, feel free to connect:

- **LinkedIn:** www.linkedin.com/in/didde-isaac
- **GitHub:** [github.com/Isaacdidde](https://github.com/Isaacdidde)
