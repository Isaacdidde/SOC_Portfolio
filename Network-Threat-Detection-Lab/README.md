# Network Threat Detection Lab

## Overview

A hands-on SOC homelab focused on network threat detection, log analysis, and alerting using Suricata IDS and Splunk Enterprise.

The lab uses Kali Linux to generate controlled attack traffic against an Ubuntu Server. Suricata captures network events, while Splunk is used to search, investigate, and alert on suspicious activity.

## Lab Environment

| Component       | Details            |
| --------------- | ------------------ |
| Attacker        | Kali Linux         |
| Attacker IP     | `192.168.100.128`  |
| Target / Sensor | Ubuntu Server      |
| Target IP       | `192.168.100.130`  |
| Network IDS     | Suricata 8.0.6     |
| SIEM            | Splunk Enterprise  |
| Web Server      | Apache             |
| Lab Network     | `192.168.100.0/24` |

## Objectives

* Generate controlled attack traffic in an isolated lab.
* Detect suspicious network activity using Suricata.
* Analyze network and authentication logs in Splunk.
* Create and tune detection searches and alerts.
* Document investigation evidence and detection results.

## Attacks and Detection Labs

### 1. TCP SYN Port Scan

**Activity:** Generated TCP SYN scanning traffic from Kali to Ubuntu.

**Detection:** Custom Suricata rule for repeated SYN packets.

**Evidence:** Suricata alert with SID `1000001`, identifying the possible TCP SYN port scan.

### 2. TCP SYN Flood

**Activity:** Generated a high volume of TCP SYN packets against the Ubuntu server.

**Detection:** Custom Suricata SYN flood rule.

**Evidence:** Suricata detection was observed during the controlled test.

### 3. UDP Port Scan

**Activity:** Performed UDP port scanning from Kali to Ubuntu.

**Detection:** Custom Suricata UDP scan rule.

**Evidence:** Custom UDP scan alert triggered during the Nmap test.

### 4. ICMP Reconnaissance

**Activity:** Generated ICMP Echo Requests from Kali to Ubuntu.

**Detection:** Custom rate-based ICMP Echo Request rule.

**Evidence:** Suricata alert with SID `1000004`.

### 5. SSH Authentication Failures

**Activity:** Performed controlled failed SSH login attempts against Ubuntu.

**Detection:** Splunk searches using Ubuntu authentication logs.

**Evidence:** Failed SSH attempts were identified and a repeated-failures alert was triggered.

### 6. SSH Successful Login

**Activity:** Tested successful SSH authentication after failed login activity.

**Detection:** Splunk alert for SSH brute-force activity followed by a successful login.

**Evidence:** The configured alert appeared in Splunk's Triggered Alerts. Review the underlying events when documenting the exact sequence.

### 7. HTTP Directory Enumeration

**Activity:** Used Gobuster to enumerate possible web paths on the Apache server.

```bash
gobuster dir \
  -u http://192.168.100.130 \
  -w /usr/share/wordlists/dirb/common.txt \
  -t 10
```

**Detection:** Splunk aggregation of Suricata HTTP events by source, destination, and port.

**Observed results:**

| Field        | Result            |
| ------------ | ----------------- |
| Source       | `192.168.100.128` |
| Destination  | `192.168.100.130` |
| Port         | `80`              |
| Requests     | 4,615             |
| Unique paths | 4,615             |
| Status codes | 200, 403, 404     |
| User-Agent   | `gobuster/3.8.2`  |

**Evidence:** Splunk search results and a triggered Medium-severity scheduled alert.

## HTTP Directory Enumeration SPL

```spl
index=* earliest=-15m latest=now
source="/var/log/suricata/eve.json"
sourcetype="suricata:json"
event_type="http"
src_ip="192.168.100.128"
dest_ip="192.168.100.130"
| stats count as requests
        dc(http.url) as unique_paths
        values(http.status) as status_codes
        by src_ip dest_ip dest_port
| where requests > 100 AND unique_paths > 50
```

**Detection thresholds:**

* More than 100 HTTP requests.
* More than 50 unique paths.

These are lab thresholds and may require tuning in a production environment.

## Splunk Alert Configuration

| Setting           | Value                      |
| ----------------- | -------------------------- |
| Alert name        | HTTP Directory Enumeration |
| Type              | Scheduled                  |
| Severity          | Medium                     |
| Cron schedule     | `*/5 * * * *`              |
| Search window     | Last 15 minutes            |
| Trigger condition | Number of results > 0      |

The alert was observed in Splunk's Triggered Alerts interface.

## Evidence

Screenshots are stored in the `screenshots/` directory.

| Screenshot                          | Evidence                      |
| ----------------------------------- | ----------------------------- |
| `01-tcp-syn-scan.png`               | TCP SYN scan detection        |
| `02-syn-flood.png`                  | SYN flood test                |
| `03-udp-port-scan.png`              | UDP scan detection            |
| `04-icmp-recon.png`                 | ICMP detection                |
| `05-ssh-failed-logins.png`          | SSH authentication failures   |
| `06-ssh-successful-login.png`       | SSH successful login alert    |
| `07-http-directory-enumeration.png` | Gobuster scan evidence        |
| `08-splunk-detection-results.png`   | HTTP detection search results |
| `09-triggered-alerts.png`           | Splunk triggered alert        |

## Tools and Technologies

* Suricata IDS
* Splunk Enterprise
* Splunk Search Processing Language (SPL)
* Kali Linux
* Ubuntu Server
* Apache HTTP Server
* Gobuster
* Nmap
* SSH and Linux authentication logs
* MITRE ATT&CK

## Key Skills Demonstrated

* Network traffic analysis
* IDS rule creation and tuning
* SIEM log analysis
* SPL searches and aggregation
* Authentication event investigation
* Detection alert configuration
* Security event documentation

## Limitations

* The lab validates detection and alerting; it does not establish that all tested traffic was blocked.
* The HTTP enumeration alert identifies suspicious path probing, not confirmed exploitation.
* The lab is isolated and intended for controlled security testing.

## Conclusion

This project demonstrates a practical network threat detection workflow: generating controlled traffic, collecting Suricata and host logs, analyzing events in Splunk, configuring alerts, and documenting detection evidence.
