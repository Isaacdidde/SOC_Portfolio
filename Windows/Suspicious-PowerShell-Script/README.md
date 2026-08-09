# LetsDefend SOC153 — Suspicious PowerShell Script Executed

## Incident Response Investigation

![Status](https://img.shields.io/badge/Case-LetsDefend-blue)
![Severity](https://img.shields.io/badge/Severity-High-red)
![Investigation-True%20Positive](https://img.shields.io/badge/Classification-True%20Positive-critical)
![Focus-Windows%20Event%20Logs](https://img.shields.io/badge/Focus-Windows%20Event%20Logs-informational)
![Focus-Sysmon](https://img.shields.io/badge/Focus-Sysmon-informational)
![MITRE%20ATT%26CK](https://img.shields.io/badge/MITRE%20ATT%26CK-mapped-orange)

> A full incident-response investigation of a suspicious PowerShell execution alert involving RDP brute-force access, post-compromise reconnaissance, Cobalt Strike-related PowerShell activity, Netcat, and database exfiltration.

---

## Table of Contents

- [1. Executive Summary](#1-executive-summary)
- [2. Alert Details](#2-alert-details)
- [3. Investigation Objectives](#3-investigation-objectives)
- [4. Investigation Methodology](#4-investigation-methodology)
- [5. Initial Alert Analysis](#5-initial-alert-analysis)
- [6. Initial Access Investigation](#6-initial-access-investigation)
- [7. RDP Authentication Analysis](#7-rdp-authentication-analysis)
- [8. Post-Compromise Activity](#8-post-compromise-activity)
- [9. Process and File Investigation](#9-process-and-file-investigation)
- [10. Internal Reconnaissance](#10-internal-reconnaissance)
- [11. PowerShell and Cobalt Strike](#11-powershell-and-cobalt-strike)
- [12. Network and Exfiltration Analysis](#12-network-and-exfiltration-analysis)
- [13. Attack Timeline](#13-attack-timeline)
- [14. Attack Chain](#14-attack-chain)
- [15. Evidence Summary](#15-evidence-summary)
- [16. MITRE ATT&CK Mapping](#16-mitre-attck-mapping)
- [17. Cyber Kill Chain Mapping](#17-cyber-kill-chain-mapping)
- [18. Detection and Investigation Lessons](#18-detection-and-investigation-lessons)
- [19. Containment](#19-containment)
- [20. Eradication and Recovery](#20-eradication-and-recovery)
- [21. Lessons Learned](#21-lessons-learned)
- [22. Analyst Conclusion](#22-analyst-conclusion)
- [23. Skills Demonstrated](#23-skills-demonstrated)
- [24. Disclaimer](#24-disclaimer)

---

# 1. Executive Summary

This repository documents the investigation of a **LetsDefend SOC153 — Suspicious PowerShell Script Executed** alert.

The investigation started with a suspicious PowerShell script named `endpoint.ps1`. The script contained encoded and compressed PowerShell content. After decoding and decompressing the payload, the activity was identified as malicious and related to **Cobalt Strike**.

The investigation was then expanded beyond the original PowerShell alert to determine:

1. How the attacker initially accessed the endpoint.
2. Which account was compromised.
3. Which source IP was associated with the initial access.
4. What the attacker did after gaining access.
5. Which tools and scripts were executed.
6. Whether internal reconnaissance occurred.
7. Whether command-and-control or external communication occurred.
8. Whether sensitive information was exfiltrated.
9. What containment and eradication actions were appropriate.

The investigation established the following attack sequence:

```text
Internet-facing RDP
        |
        v
Multiple failed Administrator logons
        |
        v
Successful RDP authentication
Source: 3.16.42.241
        |
        v
Administrator session established
        |
        v
Post-compromise reconnaissance
        |
        +--> new1.bat
        +--> ipconfig /all
        +--> netstat -an
        +--> application/process discovery
        +--> sensitive-data searches
        |
        v
Advanced Port Scanner
        |
        v
Internal network reconnaissance
        |
        v
Netcat
        |
        v
endpoint.ps1
        |
        v
Encoded/Compressed PowerShell
        |
        v
Cobalt Strike-related activity
        |
        v
Outbound connection
3.16.42.144:4444
        |
        v
user-db-backup.sql
        |
        v
Confirmed data exfiltration
```

The final assessment is that the host was **compromised through brute-force RDP authentication**, followed by reconnaissance, malicious tool execution, Cobalt Strike-related PowerShell activity, and database exfiltration.

---

# 2. Alert Details

| Field | Value |
|---|---|
| Platform | LetsDefend |
| Event ID | 101 |
| Rule | SOC153 - Suspicious Powershell Script Executed |
| Host | `Matt` |
| User | `Administrator` |
| Suspicious File | `endpoint.ps1` |
| Initial Alert Time | 05 Sep 2021, approximately 12:43 PM |
| Initial Access Source IP | `3.16.42.241` |
| Exfiltration Destination | `3.16.42.144:4444` |
| Primary Protocol for Initial Access | RDP |
| Malware/Framework Observed | Cobalt Strike |
| Transfer Tool | Netcat |
| Exfiltrated File | `user-db-backup.sql` |
| Final Classification | True Positive / Confirmed Compromise |

---

# 3. Investigation Objectives

The investigation was performed with the following objectives:

### Alert validation

- Determine whether the PowerShell alert was malicious.
- Decode and analyze the suspicious PowerShell content.
- Identify the purpose of the script.

### Initial access

- Determine how the attacker obtained access.
- Identify the compromised account.
- Identify the source IP address.
- Determine whether brute-force activity preceded successful access.

### Post-compromise investigation

- Reconstruct the attacker's activity.
- Identify executed processes.
- Identify files created or downloaded.
- Identify reconnaissance activity.
- Identify internal network scanning.
- Identify suspicious network communication.

### Impact assessment

- Determine whether sensitive information was discovered.
- Determine whether data was exfiltrated.
- Identify the exfiltrated file.

### Response

- Determine appropriate containment actions.
- Determine eradication and recovery actions.
- Identify security improvements that would prevent recurrence.

---

# 4. Investigation Methodology

The investigation followed an evidence-driven correlation process rather than treating the PowerShell alert as an isolated event.

```text
Alert
  |
  v
PowerShell analysis
  |
  v
Malicious activity confirmed
  |
  v
Investigate backwards
  |
  +--> RDP logs
  +--> 4625 failed authentication
  +--> 4624 successful authentication
  |
  v
Identify initial access
  |
  v
Investigate forwards
  |
  +--> Process creation
  +--> File creation
  +--> Network connections
  +--> Reconnaissance
  +--> Tool execution
  +--> Exfiltration
  |
  v
Build complete timeline
  |
  v
Containment / Eradication
```

The key principle used throughout the investigation was:

> **Correlate events by timestamp, account, source IP, process, logon session, and network activity rather than interpreting individual events in isolation.**

---

# 5. Initial Alert Analysis

The investigation began with the `SOC153 - Suspicious Powershell Script Executed` alert.

The suspicious command involved:

```text
powershell.exe endpoint.ps1
```

The script contained encoded PowerShell content.

### Analysis process

```text
endpoint.ps1
     |
     v
Base64 encoded content
     |
     v
Base64 decoding
     |
     v
Compressed content
     |
     v
Decompression
     |
     v
Underlying PowerShell commands
```

The use of Base64 encoding and compression was suspicious because it obscured the actual PowerShell commands.

After decoding and decompressing the content, the script was determined to be malicious and related to **Cobalt Strike**.

### L1 → L2 escalation

The alert was therefore treated as a **True Positive** rather than a benign PowerShell execution.

A reasonable SOC workflow at this stage is:

```text
L1 Alert Triage
      |
      v
Suspicious PowerShell
      |
      v
Decode / analyze payload
      |
      v
Cobalt Strike-related content
      |
      v
Potential malware / compromise
      |
      v
Escalate to L2 / Incident Response
```

---

# 6. Initial Access Investigation

After confirming that malicious code had executed, the next question was:

> **How did the attacker get onto the host?**

The investigation moved backward from the PowerShell execution time and examined remote-access activity.

The host had an Internet-accessible RDP service.

The investigation focused on:

- Microsoft-Windows-TerminalServices-LocalSessionManager
- Microsoft-Windows-TerminalServices-RemoteConnectionManager
- Windows Security logs
- Authentication events

---

# 7. RDP Authentication Analysis

## 7.1 Successful RDP session

A successful RDP session was identified through:

```text
Event ID: 21
Log: Microsoft-Windows-TerminalServices-LocalSessionManager/Operational
Account: Administrator
Source IP: 3.16.42.241
Time: 12:12:15 PM
```

This established that a remote interactive session was successfully created from:

```text
3.16.42.241
```

for the `Administrator` account.

---

## 7.2 Failed authentication attempts

Windows Security logs were then searched for:

```text
Event ID: 4625
```

Multiple failed authentication attempts were found involving the same source IP:

```text
3.16.42.241
```

The sequence was consistent with:

```text
3.16.42.241
       |
       +--> Failed Administrator login
       |
       +--> Failed Administrator login
       |
       +--> Failed Administrator login
       |
       +--> Successful authentication
       |
       +--> RDP session
```

This correlation supports the conclusion that the Administrator credentials were obtained through a brute-force attack against the exposed RDP service.

---

## 7.3 Successful logon — Event ID 4624

Successful authentication events were also examined.

A relevant event contained:

```text
Event ID: 4624
Logon Type: 3
Account: Administrator
Source Network Address: 3.16.42.241
Authentication Package: NTLM
NTLM Version: NTLM V2
Logon ID: 0xD2C526
Elevated Token: Yes
```

### Important interpretation

**Logon Type 3 means Network Logon.**

It does **not** independently prove data exfiltration.

The event establishes successful network authentication. The actual resource or service accessed must be determined by correlating additional events.

For RDP, **Logon Type 10** is the relevant RemoteInteractive/RDP logon type.

This distinction is important when analyzing Windows authentication events.

---

# 8. Post-Compromise Activity

After successful RDP access, the attacker began performing activity on the endpoint.

The investigation identified several artifacts:

```text
new1.bat
Advanced_Port_Scanner_2.5.3869.exe
nc111nt.zip
endpoint.ps1
```

These artifacts were correlated with the process and file activity timeline.

---

# 9. Process and File Investigation

## 9.1 `new1.bat`

The attacker created:

```text
new1.bat
```

around:

```text
12:30 PM
```

The file was subsequently executed through `cmd.exe`.

Because the batch file had been deleted, its contents were not directly available.

Instead, the investigation used the **parent/child process relationship** to determine what the batch file executed.

The parent process ID identified in the investigation was:

```text
4360
```

Child processes associated with that parent revealed reconnaissance commands.

---

## 9.2 System reconnaissance

Observed commands included:

```text
ipconfig /all
netstat -an
```

The attacker also enumerated applications, processes, network information, and other system details.

This activity indicates that the attacker was trying to understand the compromised environment after gaining access.

---

# 10. Internal Reconnaissance

The attacker executed:

```text
Advanced_Port_Scanner_2.5.3869.exe
```

around:

```text
12:32 PM
```

Network connection telemetry was then examined to identify scanning activity.

The purpose of the scanning was consistent with identifying:

- Internal hosts
- Open ports
- Available services
- Potential lateral-movement targets

This indicates that the compromise was not limited to the initially compromised host.

---

# 11. PowerShell and Cobalt Strike

The original suspicious PowerShell script was:

```text
endpoint.ps1
```

It was executed around:

```text
12:43 PM
```

The script used encoding and compression to obscure its contents.

After decoding:

```text
Base64
   |
   v
Decode
   |
   v
Decompress
   |
   v
PowerShell payload
   |
   v
Cobalt Strike-related activity
```

This provided evidence that the PowerShell execution was part of a broader malicious intrusion rather than an isolated administrative script.

---

# 12. Network and Exfiltration Analysis

## 12.1 Netcat

The attacker downloaded:

```text
nc111nt.zip
```

and subsequently executed Netcat.

The investigation identified an outbound connection to:

```text
3.16.42.144:4444
```

### Important IP distinction

The two IP addresses have different roles:

```text
INITIAL ACCESS

3.16.42.241
      |
      | RDP
      v
     Matt
```

versus:

```text
EXFILTRATION / OUTBOUND COMMUNICATION

Matt
  |
  | Netcat TCP/4444
  v
3.16.42.144
```

Therefore, the IP used for initial access does not have to be the same IP used for later outbound communication.

---

## 12.2 Exfiltrated data

Network telemetry established communication with:

```text
3.16.42.144:4444
```

The terminal history provided the additional evidence required to determine what was transferred.

The exfiltrated file was:

```text
user-db-backup.sql
```

Location on the compromised endpoint:

```text
C:\Users\Administrator\Documents\Database Backups\
```

The file was approximately:

```text
206 KB
```

This establishes actual data exfiltration rather than merely suspicious outbound communication.

---

# 13. Attack Timeline

| Time | Evidence | Activity | Interpretation |
|---|---|---|---|
| Before 12:12 | 4625 | Failed Administrator authentication attempts | Brute-force activity |
| 12:12:15 | Event 21 | Successful RDP session | Initial access |
| ~12:12 onward | 4624 | Successful authentication events | Post-authentication activity |
| 12:30 | File/process telemetry | `new1.bat` created | Attacker tooling |
| 12:31 | Process telemetry | `new1.bat` executed | Reconnaissance begins |
| 12:31 | Process telemetry | `ipconfig /all`, `netstat -an`, etc. | Host/network discovery |
| 12:32 | Process telemetry | Advanced Port Scanner | Internal reconnaissance |
| 12:39 | File/process telemetry | `nc111nt.zip` / Netcat | Data-transfer capability |
| 12:43 | Event 101 | `endpoint.ps1` executed | Malicious PowerShell |
| Later | Process/network telemetry | Netcat → `3.16.42.144:4444` | Outbound communication |
| Later | Terminal history | `user-db-backup.sql` transferred | Confirmed exfiltration |

---

# 14. Attack Chain

```text
                     INTERNET
                         |
                         v
              Internet-facing RDP
                         |
                         v
             Brute-force Administrator
                         |
                         v
                3.16.42.241
                         |
                         v
              Successful RDP login
                         |
                         v
                Administrator access
                         |
                         v
                 System Discovery
                         |
             +-----------+-----------+
             |                       |
             v                       v
         new1.bat              Findstr searches
             |
             v
     ipconfig / netstat
             |
             v
      Internal Port Scan
             |
             v
     Advanced Port Scanner
             |
             v
          Netcat
             |
             v
     endpoint.ps1
             |
             v
    Encoded/Compressed PowerShell
             |
             v
    Cobalt Strike-related activity
             |
             v
     3.16.42.144:4444
             |
             v
     user-db-backup.sql
             |
             v
         EXFILTRATION
```

---

# 15. Evidence Summary

| Evidence | Significance |
|---|---|
| `SOC153 - Suspicious Powershell Script Executed` | Initial detection |
| `endpoint.ps1` | Malicious PowerShell payload |
| Base64 encoding | Payload obfuscation |
| Compression | Additional payload obfuscation |
| Cobalt Strike-related content | Strong evidence of malicious activity |
| Event ID 21 | Successful RDP session |
| Event ID 4625 | Failed authentication / brute-force evidence |
| Event ID 4624 Type 10 | Remote Interactive/RDP authentication |
| Event ID 4624 Type 3 | Network authentication |
| `3.16.42.241` | Initial-access source IP |
| `new1.bat` | Attacker-created script |
| `Advanced_Port_Scanner_2.5.3869.exe` | Internal reconnaissance |
| `nc111nt.zip` | Netcat archive |
| Sysmon Event ID 1 | Process creation evidence |
| Sysmon Event ID 3 | Network connection evidence |
| Sysmon Event ID 11 | File creation evidence |
| `3.16.42.144:4444` | Later outbound communication destination |
| `user-db-backup.sql` | Confirmed exfiltrated database backup |

---

# 16. MITRE ATT&CK Mapping

| Tactic | Technique / Activity | Evidence |
|---|---|---|
| Initial Access | External Remote Services | RDP exposed to the Internet |
| Initial Access | Valid Accounts | Administrator account successfully used |
| Initial Access | Brute Force | Multiple failed authentication attempts followed by success |
| Execution | Command and Scripting Interpreter: PowerShell | `endpoint.ps1` |
| Execution | Command and Scripting Interpreter: Windows Command Shell | `new1.bat` / `cmd.exe` |
| Execution | Software Deployment / Tool Execution | Advanced Port Scanner / Netcat |
| Discovery | System / Network Discovery | `ipconfig`, `netstat`, process/application enumeration |
| Discovery | Network Service Scanning | Advanced Port Scanner |
| Defense Evasion | Obfuscated/Compressed Files or Information | Encoded and compressed PowerShell |
| Command and Control | Application Layer / Network Communication | Netcat connection to port 4444 |
| Exfiltration | Exfiltration Over Alternative Protocol | Netcat-based transfer |
| Exfiltration | Data from Local System | `user-db-backup.sql` |

> MITRE mappings should be treated as an analytical mapping of the observed behavior. Exact technique/sub-technique selection can depend on the telemetry and ATT&CK version being used.

---

# 17. Cyber Kill Chain Mapping

| Kill Chain Stage | Observed Activity |
|---|---|
| Reconnaissance | Port scanning |
| Weaponization | Malicious PowerShell/Cobalt Strike payload |
| Delivery | RDP exposure |
| Exploitation | RDP brute-force authentication |
| Installation | Malicious tooling / Cobalt Strike-related payload |
| Command & Control | Outbound Netcat communication |
| Actions on Objectives | Database backup exfiltration |

---

# 18. Detection and Investigation Lessons

## 18.1 Do not investigate alerts in isolation

A PowerShell alert may only represent one stage of a much larger attack.

The investigation should ask:

```text
Why was PowerShell executed?
        |
Who executed it?
        |
How did the attacker gain access?
        |
What happened before execution?
        |
What happened after execution?
        |
Was data accessed?
        |
Was data exfiltrated?
```

---

## 18.2 Correlate authentication events

Useful Windows authentication events include:

```text
4624  Successful logon
4625  Failed logon
```

For RDP investigations, also examine:

```text
TerminalServices-LocalSessionManager
TerminalServices-RemoteConnectionManager
```

Do not treat every 4624 as an interactive attacker login. Always inspect:

- Logon Type
- Account
- Source IP
- Timestamp
- Logon ID
- Authentication package
- Surrounding events

---

## 18.3 Understand Logon Type

Important values:

| Logon Type | Meaning |
|---:|---|
| 2 | Interactive |
| 3 | Network |
| 5 | Service |
| 10 | Remote Interactive / RDP |

A **Type 3 logon does not mean data exfiltration**.

Exfiltration requires additional evidence such as:

```text
Process execution
+
Network connection
+
File/data identification
+
Transfer evidence
```

---

## 18.4 Sysmon provides deeper endpoint visibility

The investigation demonstrates the value of Sysmon:

| Sysmon Event | Purpose |
|---:|---|
| 1 | Process creation |
| 3 | Network connection |
| 11 | File creation |

Without Sysmon, Windows native logs can still provide valuable authentication and RDP evidence, but process, file, and network visibility may be significantly weaker depending on the audit configuration.

---

# 19. Containment

Once the compromise was confirmed, the affected host should be isolated from the network.

For the LetsDefend scenario:

```text
Matt
  |
  v
Request Containment
```

The purpose is to:

- Stop ongoing attacker access.
- Prevent additional exfiltration.
- Prevent lateral movement.
- Preserve the endpoint for investigation.

During forensic investigation, the host should not be unnecessarily powered off because volatile evidence may be lost.

---

# 20. Eradication and Recovery

Recommended actions include:

### 1. Reset the compromised Administrator credentials

The credentials were exposed through the successful brute-force attack.

### 2. Review RDP access

If RDP is not required:

- Disable unnecessary Internet exposure.
- Remove unnecessary users from Remote Desktop Users.
- Restrict RDP to trusted IP ranges or VPN access.

### 3. Remove malicious artifacts

Investigate and remove attacker-created files such as:

```text
endpoint.ps1
new1.bat
Advanced_Port_Scanner_2.5.3869.exe
nc111nt.zip
```

### 4. Investigate lateral movement

Because internal network scanning occurred, other systems should be checked for:

- Authentication attempts
- New processes
- Remote access
- File transfers
- Suspicious services
- Similar malicious PowerShell activity

### 5. Continue monitoring

Search for:

```text
3.16.42.241
3.16.42.144
endpoint.ps1
new1.bat
Advanced_Port_Scanner_2.5.3869.exe
nc111nt.zip
user-db-backup.sql
```

across available security telemetry.

---

# 21. Lessons Learned

The primary security weakness was not PowerShell itself.

The intrusion began with exposed remote access and compromised credentials.

### Key defensive improvements

- Avoid exposing RDP directly to the public Internet when possible.
- Use VPN or another controlled remote-access mechanism.
- Restrict RDP source IPs where business requirements allow.
- Use strong, unique passwords for privileged accounts.
- Protect privileged accounts with additional authentication controls where possible.
- Monitor repeated 4625 failures followed by successful 4624/Terminal Services events.
- Alert on suspicious PowerShell encoding and obfuscation.
- Monitor unusual process creation.
- Monitor internal network scanning.
- Monitor unexpected Netcat or similar network-transfer utilities.
- Monitor unusual outbound connections.
- Maintain endpoint telemetry such as Sysmon where appropriate.
- Centralize and retain Windows security logs.

---

# 22. Analyst Conclusion

### Final Verdict: TRUE POSITIVE — COMPROMISED HOST

The investigation confirmed that the suspicious PowerShell alert was part of a larger intrusion.

The evidence supports the following attack narrative:

> An attacker performed repeated authentication attempts against an Internet-accessible RDP service using the Administrator account. The activity originated from `3.16.42.241`. After multiple failed authentication attempts, the attacker successfully established an RDP session and gained access to the host.
>
> Following successful access, the attacker performed system and network reconnaissance, created and executed attacker tooling, and conducted internal network scanning. The attacker later executed `endpoint.ps1`, which contained encoded and compressed PowerShell associated with Cobalt Strike.
>
> Netcat was subsequently used for outbound communication with `3.16.42.144:4444`. Terminal history confirmed that `user-db-backup.sql` was transferred, establishing successful data exfiltration.
>
> The incident therefore represents a confirmed endpoint compromise involving brute-force RDP access, post-compromise reconnaissance, malicious PowerShell/Cobalt Strike activity, network communication, and database exfiltration.

---

# 23. Skills Demonstrated

This investigation demonstrates practical experience with:

- SOC alert triage
- L1 → L2 escalation reasoning
- Windows Event Log analysis
- Event ID 4624 analysis
- Event ID 4625 analysis
- RDP investigation
- Terminal Services logs
- Logon Type analysis
- Source IP correlation
- Authentication timeline reconstruction
- Sysmon Event ID 1 analysis
- Sysmon Event ID 3 analysis
- Sysmon Event ID 11 analysis
- Process-tree investigation
- PowerShell investigation
- Base64 decoding
- Malware/obfuscation analysis
- Cobalt Strike identification
- Network reconnaissance analysis
- Port scanning investigation
- Netcat investigation
- Data exfiltration analysis
- Incident timeline creation
- MITRE ATT&CK mapping
- Cyber Kill Chain analysis
- Incident containment
- Eradication planning
- Root-cause analysis

---

# 24. Disclaimer

This repository documents a cybersecurity training/investigation exercise based on a LetsDefend case.

The analysis is intended for:

- SOC analyst training
- Blue-team practice
- Windows event-log analysis
- Incident-response learning
- Portfolio demonstration

The artifacts and indicators documented here should be treated as **training-case indicators** and should not be assumed to represent an active real-world threat without independent verification.

---

## Investigation Philosophy

> **Don't investigate the alert. Investigate the incident behind the alert.**

A single PowerShell detection can be the visible part of a much larger intrusion.

The objective of a SOC investigation is therefore to connect:

```text
Authentication
      +
Endpoint Activity
      +
Process Activity
      +
Network Activity
      +
File Activity
      +
Threat Intelligence
      =
Incident Narrative
```

