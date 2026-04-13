# Phase 5 — SIEM Integration & Detection Engineering (Wazuh)

> **SOC Home Lab Series** | Red Team → Blue Team → Detection Engineering Pipeline

---

## Table of Contents

1. [Overview](#overview)
2. [Objectives](#objectives)
3. [Lab Architecture](#lab-architecture)
4. [Environment Setup](#environment-setup)
5. [Wazuh Deployment](#wazuh-deployment)
6. [Agent Integration — DVWA Server](#agent-integration--dvwa-server)
7. [Log Collection — Apache](#log-collection--apache)
8. [Attack Simulation — SQL Injection](#attack-simulation--sql-injection)
9. [Log Analysis in Wazuh](#log-analysis-in-wazuh)
10. [Detection Engineering — Custom Rules](#detection-engineering--custom-rules)
11. [MITRE ATT&CK Mapping](#mitre-attck-mapping)
12. [Troubleshooting Log](#troubleshooting-log)
13. [SOC Cheat Sheet — Quick Reference](#soc-cheat-sheet--quick-reference)
14. [Key Learnings](#key-learnings)
15. [Outcome & Next Phase](#outcome--next-phase)

---

## Overview

This phase integrates a **Security Information and Event Management (SIEM)** solution into the home lab using **Wazuh**, completing the full SOC detection pipeline.

The phase bridges two disciplines:

| Team | Activity |
|------|----------|
| 🔴 Red Team | SQL Injection attacks against DVWA |
| 🔵 Blue Team | Apache log ingestion, real-time monitoring, alert generation |
| 🟣 Detection Engineering | Custom Wazuh rules mapped to MITRE ATT&CK |

The end goal is to simulate a realistic SOC analyst workflow — from attack execution to rule-based alerting — including all the real-world deployment challenges encountered along the way.

---

## Objectives

- Deploy Wazuh SIEM in an All-in-One configuration
- Integrate DVWA Apache server logs into Wazuh via agent
- Monitor real-time SQL Injection activity in the Wazuh dashboard
- Author custom detection rules targeting SQLi payloads
- Map all detections to MITRE ATT&CK technique IDs
- Document authentic troubleshooting scenarios with root causes and fixes

---

## Lab Architecture

```
┌─────────────────────┐
│   Kali Linux        │  ← Attacker Machine
│   (SQL Injection)   │
└────────┬────────────┘
         │ HTTP Requests (SQLi Payloads)
         ▼
┌─────────────────────┐
│   Ubuntu — DVWA     │  ← Target (Apache + DVWA)
│   Wazuh Agent       │    Logs: /var/log/apache2/access.log
└────────┬────────────┘
         │ Agent → Manager (port 1514)
         ▼
┌─────────────────────┐
│   Ubuntu — Wazuh    │  ← SIEM Server
│   Manager           │    Port 1514 (agent comms)
│   Indexer           │    Port 9200 (OpenSearch)
│   Dashboard         │    Port 443 / 5601 (UI)
└─────────────────────┘
```

---

## Environment Setup

### Systems Used

| Component | OS | Role |
|-----------|----|------|
| Attacker  | Kali Linux | Executes SQL Injection attacks |
| Target    | Ubuntu 22.04 | Hosts DVWA + Apache web server |
| SIEM      | Ubuntu 22.04 | Runs Wazuh All-in-One stack |

> **Resource Note:** Wazuh (Manager + Indexer + Dashboard) is memory-intensive. Allocate a minimum of **4GB RAM** and **2 vCPUs** to the SIEM VM to prevent JVM crashes and indexer instability.

---

## Wazuh Deployment

### All-in-One Installation

```bash
# Download install script
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh

# Run all-in-one deployment (installs Manager, Indexer, Dashboard)
sudo bash wazuh-install.sh -a
```

> The installer auto-generates TLS certificates and an admin password. **Save the credentials displayed at the end of installation** — they are needed for dashboard access and API authentication.

### Access the Dashboard

```
https://<WAZUH_SERVER_IP>
```

| Field    | Value |
|----------|-------|
| Username | `admin` |
| Password | *Generated during install — printed to terminal* |

### Verify All Services Are Running

```bash
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard
```

All three services should show `active (running)` before proceeding.

---

## Agent Integration — DVWA Server

### Step 1: Install the Wazuh Agent

```bash
# Download the agent package (match version to manager)
curl -sO https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.7.x_amd64.deb

# Install
sudo dpkg -i wazuh-agent_4.7.x_amd64.deb
```

> **Version Rule:** Agent version must be **less than or equal to** Manager version. Mismatches cause registration failures.

### Step 2: Point Agent to Manager

```bash
sudo nano /var/ossec/etc/ossec.conf
```

Find the `<client>` block and set the manager address:

```xml
<client>
  <server>
    <address>WAZUH_SERVER_IP</address>
    <port>1514</port>
    <protocol>tcp</protocol>
  </server>
</client>
```

### Step 3: Register the Agent

**On the Wazuh Manager:**

```bash
sudo /var/ossec/bin/manage_agents
```

Select option `A` to add a new agent. Provide a name and the DVWA server's IP. Then select option `E` to extract the registration key.

**On the DVWA (Agent) Server:**

```bash
sudo /var/ossec/bin/manage_agents
```

Select option `I` to import the key copied from the manager.

### Step 4: Start the Agent

```bash
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```

Verify the agent appears as **Active** in the Wazuh dashboard under `Agents`.

---

## Log Collection — Apache

By default, Wazuh monitors `/var/ossec/logs/`. Apache logs must be explicitly added.

### Configure Apache Log Ingestion

```bash
sudo nano /var/ossec/etc/ossec.conf
```

Add the following inside the `<ossec_config>` block:

```xml
<localfile>
  <log_format>apache</log_format>
  <location>/var/log/apache2/access.log</location>
</localfile>
```

### Apply Configuration

```bash
sudo systemctl restart wazuh-agent
```

### Verify Logs Are Reaching the Manager

```bash
# On DVWA server — watch live Apache traffic
tail -f /var/log/apache2/access.log

# On Wazuh manager — verify Wazuh is processing events
tail -f /var/ossec/logs/ossec.log
```

---

## Attack Simulation — SQL Injection

Attacks are executed from Kali Linux targeting the DVWA application. DVWA security level should be set to **Low** for initial testing.

### Authentication Bypass

Tests whether login can be bypassed using a tautology:

```sql
' OR '1'='1
```

### Column Enumeration

Determines the number of columns in the query result set:

```sql
1 ORDER BY 1--
1 ORDER BY 2--
1 ORDER BY 3--   ← Error here means 2 columns exist
```

### UNION-Based Data Extraction

Extracts database name and other backend data:

```sql
1 UNION SELECT null, database()--
1 UNION SELECT null, user()--
1 UNION SELECT null, version()--
```

### URL-Encoded Variants

Attackers often encode payloads to evade basic filters. These should also appear in logs:

| Encoded | Decoded |
|---------|---------|
| `%27`   | `'` (single quote) |
| `%20`   | ` ` (space) |
| `%3D`   | `=` |
| `%2D%2D` | `--` (comment) |

---

## Log Analysis in Wazuh

### Navigate to Security Events

```
Wazuh Dashboard → Security Events → Search
```

### Key Indicators of SQLi in Apache Logs

Review the raw log entries for these patterns:

| Indicator | Significance |
|-----------|-------------|
| `UNION SELECT` | Data extraction attempt |
| `ORDER BY` | Column enumeration |
| `' OR '1'='1` | Authentication bypass |
| `%27`, `%3D` | URL-encoded SQL characters |
| HTTP 200 on `/dvwa/vulnerabilities/sqli/` | Successful query execution |
| HTTP 500 responses | Syntax errors — active probing |

### Sample Malicious Apache Log Entry

```
192.168.1.50 - - [12/Apr/2025:14:22:08 +0000] "GET /dvwa/vulnerabilities/sqli/?id=1+UNION+SELECT+null%2Cdatabase()--&Submit=Submit HTTP/1.1" 200 1842
```

---

## Detection Engineering — Custom Rules

Wazuh's default ruleset detects many web attacks, but custom rules allow precise targeting of SQLi patterns observed in the lab.

### Create the Custom Rule

```bash
sudo nano /var/ossec/etc/rules/local_rules.xml
```

```xml
<group name="sql_injection,web,attack">

  <!-- SQL Injection Attempt — keyword and encoded payload detection -->
  <rule id="100001" level="10">
    <if_sid>31151</if_sid>
    <match>UNION SELECT|ORDER BY|OR 1=1|%27|%3D|1=1--|drop table|information_schema</match>
    <description>Possible SQL Injection attempt detected in web request</description>
    <mitre>
      <id>T1190</id>
    </mitre>
  </rule>

  <!-- URL-encoded SQLi payload detection -->
  <rule id="100002" level="10">
    <if_sid>31151</if_sid>
    <match>%27|%3D%3D|%20OR%20|%20UNION%20|%20SELECT%20</match>
    <description>URL-encoded SQL Injection payload detected</description>
    <mitre>
      <id>T1027</id>
    </mitre>
  </rule>

</group>
```

> **Rule ID Range:** Custom rules should use IDs between `100000–199999` to avoid conflicting with Wazuh's built-in ruleset.

> **Level 10** maps to "High Severity" in Wazuh and will trigger visible alerts in the dashboard.

### Apply the Rule

```bash
# Validate rule syntax
sudo /var/ossec/bin/wazuh-logtest

# Restart manager to load new rules
sudo systemctl restart wazuh-manager
```

### Validate Detection

Re-run SQL injection attacks from Kali. In the Wazuh dashboard:

- Navigate to **Security Events**
- Filter by Rule ID `100001` or `100002`
- Confirm alerts are generated with payload visible in the log field
- Verify MITRE ATT&CK tags appear on the alert

---

## MITRE ATT&CK Mapping

| Technique | ID | Observed Activity |
|-----------|----|--------------------|
| Exploit Public-Facing Application | T1190 | SQL Injection against DVWA web application |
| Obfuscated Files or Information | T1027 | URL-encoded payloads (`%27`, `%3D`, `%20`) |
| Network Traffic Analysis | T1040 | HTTP request patterns captured in Apache logs |
| Log Enumeration | T1654 | Apache access log review during analysis phase |

---

## Troubleshooting Log

Every issue below was encountered during the actual deployment of this lab environment.

---

### Issue 1 — Dashboard Not Ready

**Symptom:**
```
Wazuh dashboard server is not ready yet
```

**Root Cause:** Wazuh Indexer (OpenSearch) was not running or had failed to initialize, preventing the dashboard from connecting.

**Resolution:**
```bash
# Check indexer status
sudo systemctl status wazuh-indexer

# Check for errors
journalctl -xeu wazuh-indexer.service

# Restart all components in order
sudo systemctl restart wazuh-indexer
sudo systemctl restart wazuh-manager
sudo systemctl restart wazuh-dashboard
```

---

### Issue 2 — Indexer Crash (JVM / GC Log Failure)

**Symptom:**
```
Error opening log file 'gc.log': No such file or directory
```

**Root Cause:** The `/var/log/wazuh-indexer/` directory was missing or had incorrect ownership, preventing the JVM garbage collector from writing its log file.

**Resolution:**
```bash
sudo mkdir -p /var/log/wazuh-indexer
sudo chown -R wazuh-indexer:wazuh-indexer /var/log/wazuh-indexer
sudo chmod -R 755 /var/log/wazuh-indexer
sudo systemctl restart wazuh-indexer
```

---

### Issue 3 — Agent Version Mismatch

**Symptom:** Agent fails to register or shows as disconnected in the manager.

**Root Cause:** Agent version was newer than the Manager version. Wazuh requires `Agent ≤ Manager`.

**Resolution:** Downgrade or reinstall the agent to match the manager's version:

```bash
# Check manager version
sudo /var/ossec/bin/wazuh-control info

# Reinstall agent with matching version
sudo dpkg -i wazuh-agent_<MATCHING_VERSION>_amd64.deb
```

---

### Issue 4 — SSL / TLS Certificate Errors

**Symptom:** Dashboard inaccessible or API returns certificate errors.

**Root Cause:** Manual TLS certificate configuration was incorrect — mismatched CN, wrong file paths, or missing intermediate certs.

**Resolution:** Clean reinstall using the official script to regenerate all certificates automatically:

```bash
sudo bash wazuh-install.sh -a --overwrite
```

---

### Issue 5 — Dashboard Plugin Broken

**Symptom:** Dashboard loads but certain views or panels are blank / throw errors.

**Root Cause:** Manual removal or modification of OpenSearch Dashboard plugins broke the UI state.

**Resolution:** Reinstall the dashboard component:

```bash
sudo bash wazuh-install.sh --wazuh-dashboard
```

---

---

## 🚧 Issue #6 — Authentication Failure (401 Unauthorized / Dashboard Not Ready)

### Symptoms

| Component | Error |
|-----------|-------|
| Wazuh API | `401 Unauthorized` |
| Dashboard | `Wazuh server not ready` |
| Login page | Credentials rejected silently |

### Root Cause

Wazuh uses **centralized authentication through the Indexer (OpenSearch)**. The Dashboard acts as a client to the Indexer — if their credentials fall out of sync, the entire SIEM stack fails. This typically occurs after:

- Running the password reset tool without updating the dashboard config
- Fresh installation where default credentials haven't been retrieved
- Manual password changes applied to only one component

---

### Resolution

#### Method 1 — Auto Reset All Passwords *(Recommended)*

Resets all Wazuh component passwords in one operation:

```bash
sudo /usr/share/wazuh-indexer/plugins/opensearch-security/tools/wazuh-passwords-tool.sh -a
```

> **⚠️ Save the output immediately.** It will not be shown again.

Verify the new credentials work against the Indexer API:

```bash
curl -k -u admin:<NEW_PASSWORD> https://localhost:9200
```

A JSON response confirms success. A `401` response means credentials are still mismatched.

---

#### Method 2 — Set a Custom Password *(Recommended for Lab Environments)*

Useful when you want a consistent, memorable password across rebuilds:

```bash
sudo /usr/share/wazuh-indexer/plugins/opensearch-security/tools/wazuh-passwords-tool.sh \
  -u admin \
  -p "Wazuh@123"
```

After changing the password, **you must update the dashboard config** or the dashboard will immediately break:

```bash
sudo nano /etc/wazuh-dashboard/opensearch_dashboards.yml
```

```yaml
opensearch.username: "admin"
opensearch.password: "Wazuh@123"
```

Then restart both services:

```bash
sudo systemctl restart wazuh-indexer
sudo systemctl restart wazuh-dashboard
```

---

#### Method 3 — Retrieve Default Installation Credentials

For fresh installs where the generated password was never saved:

```bash
sudo tar -xvf wazuh-install-files.tar
sudo cat wazuh-install-files/wazuh-passwords.txt
```

---

### Verification Checklist

```bash
# 1. Test Indexer API authentication
curl -k -u admin:<password> https://localhost:9200

# 2. Check service health
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard

# 3. Confirm dashboard is reachable
# Browser: https://<WAZUH-IP>
```

---

### Common Errors

**`ERROR: The backup could not be created`**  
The Indexer is not running. Start it first:
```bash
sudo systemctl start wazuh-indexer
```

**Still receiving `401` after reset**  
Re-run the auto-reset and verify the dashboard config reflects the new password:
```bash
sudo /usr/share/wazuh-indexer/plugins/opensearch-security/tools/wazuh-passwords-tool.sh -a
sudo nano /etc/wazuh-dashboard/opensearch_dashboards.yml
sudo systemctl restart wazuh-dashboard
```

**Dashboard not loading after password change**  
Almost always a config sync issue. Recheck `/etc/wazuh-dashboard/opensearch_dashboards.yml` and confirm the password matches exactly, then restart the dashboard service.

---

### Key Takeaway

> Wazuh authentication is **Indexer-centric**. The Dashboard does not manage its own credentials — it authenticates *through* the Indexer. Any password change must be reflected in both the Indexer and the dashboard config file, or the stack will fail silently on the next restart.

---

## SOC Cheat Sheet — Quick Reference

### Service Management

```bash
# Status
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-dashboard

# Restart
sudo systemctl restart wazuh-indexer
sudo systemctl restart wazuh-manager
sudo systemctl restart wazuh-dashboard
```

### Log Monitoring

```bash
# Live Apache traffic
tail -f /var/log/apache2/access.log

# Wazuh manager event processing
tail -f /var/ossec/logs/ossec.log

# Detailed service logs
journalctl -xeu wazuh-indexer.service
journalctl -xeu wazuh-dashboard.service
```

### Network & Connectivity

```bash
# Verify ports are listening
ss -tulnp | grep 5601    # Dashboard
ss -tulnp | grep 9200    # Indexer (OpenSearch)
ss -tulnp | grep 1514    # Agent comms

# Test indexer API
curl -k -u admin:<password> https://localhost:9200
```

### Configuration File Reference

| File | Purpose |
|------|---------|
| `/var/ossec/etc/ossec.conf` | Agent config, log sources, manager address |
| `/var/ossec/etc/rules/local_rules.xml` | Custom detection rules |
| `/etc/wazuh-dashboard/opensearch_dashboards.yml` | Dashboard config |
| `/var/ossec/logs/ossec.log` | Manager/agent operational log |

### Agent Management

```bash
# Interactive agent management (add, remove, keys)
sudo /var/ossec/bin/manage_agents

# Version info
sudo /var/ossec/bin/wazuh-control info
```

### Rule Testing

```bash
# Test a rule against a sample log line
sudo /var/ossec/bin/wazuh-logtest
```

---

## Key Learnings

**1. Version alignment is non-negotiable.**
All Wazuh components — Manager, Indexer, Dashboard, and Agent — must be on compatible versions. Always install Agent ≤ Manager version. Mismatches cause silent registration failures.

**2. TLS is the foundation, not an afterthought.**
The entire Wazuh stack communicates over TLS. Manual certificate misconfigurations are difficult to debug. Default auto-generated certs from the installer are reliable and should be used in lab environments.

**3. Logs are the single source of truth.**
Detection quality is only as good as the logs feeding the SIEM. Apache log ingestion must be explicitly configured — it doesn't happen automatically. Verify ingestion before building rules.

**4. Custom rules are what separate analysts from tools.**
Default Wazuh rules catch common patterns, but detection engineering means writing rules tailored to your environment. Understanding rule IDs, severity levels, and `if_sid` chaining is a core analyst skill.

**5. Troubleshooting is a primary SOC skill.**
Every deployment issue in this phase — the JVM crash, version mismatches, certificate failures — reflects real-world SIEM problems. Documenting root causes and fixes is as valuable as the detection work itself.

**6. Resource allocation directly impacts SIEM stability.**
Under-resourced VMs cause Indexer JVM crashes and dashboard timeouts. SIEM stability requires dedicated RAM and CPU, even in lab environments.

---

## Outcome & Next Phase

### Phase 5 Results

| Goal | Status |
|------|--------|
| Deploy Wazuh SIEM (All-in-One) | ✅ Completed |
| Integrate DVWA Apache logs | ✅ Completed |
| Detect SQL Injection attacks live | ✅ Completed |
| Author custom detection rules | ✅ Completed |
| Map detections to MITRE ATT&CK | ✅ Completed |
| Document real troubleshooting scenarios | ✅ Completed |

### Phase 6 — Purple Team Validation *(Upcoming)*

- Re-execute the full SQL Injection attack chain
- Measure detection coverage: which techniques trigger alerts, which are missed
- Tune rules to reduce false positives
- Add detection for additional SQLi variants (blind SQLi, time-based)
- Document gaps and iteratively improve detection logic

---

## Author

**Isaac James**
SOC Analyst | Cybersecurity Enthusiast

*Part of an ongoing SOC Home Lab series covering the full attack-detect-respond pipeline.*

---

*Last updated: April 2025 | Wazuh v4.7 | Ubuntu 22.04*