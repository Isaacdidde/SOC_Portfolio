# SOC169 - Possible IDOR Attack Detected (Event ID 119)

## Incident Overview

| Field | Value |
|---------|---------|
| Event ID | 119 |
| Rule Name | SOC169 - Possible IDOR Attack Detected |
| Severity | Medium |
| Incident Type | Web Attack |
| Event Time | Feb 28, 2022 10:48 PM |
| Source IP | 134.209.118.137 |
| Destination IP | 172.16.17.15 |
| Hostname | WebServer1005 |
| Device Action | Allowed |
| HTTP Method | POST |

---

## Executive Summary

An alert was generated for a potential Insecure Direct Object Reference (IDOR) attack against the web application hosted on WebServer1005.

Investigation revealed multiple consecutive requests targeting the same endpoint while modifying the `user_id` parameter. The requests were successfully processed by the server and returned HTTP 200 responses with varying response sizes.

The observed behavior is consistent with IDOR enumeration where an attacker attempts to access unauthorized user records by manipulating object identifiers.

---

## What is IDOR?

IDOR (Insecure Direct Object Reference) occurs when an application exposes direct references to objects without proper authorization checks.

Example:

```http
POST /get_user_info/
user_id=1

POST /get_user_info/
user_id=2

POST /get_user_info/
user_id=3
```

An attacker may gain access to records belonging to other users if access controls are not properly enforced.

---

## Investigation Process

### Step 1 – Review Alert Details

The alert indicated:

```text
Alert Reason:
Consecutive requests to the same page
```

Target URL:

```text
https://172.16.17.15/get_user_info/
```

---

### Step 2 – Analyze Source IP

Source IP:

```text
134.209.118.137
```

The source IP was identified as an external public address.

Reputation checks were performed using:

- VirusTotal
- AbuseIPDB
- WHOIS

No malicious reputation was identified.

---

### Step 3 – Analyze HTTP Requests

Observed requests:

```text
POST /get_user_info/
?user_id=2
```

Response:

```text
HTTP Status: 200
Response Size: 253 bytes
```

---

Second request:

```text
POST /get_user_info/
?user_id=5
```

Response:

```text
HTTP Status: 200
Response Size: 267 bytes
```

---

### Findings

Key observations:

- Requests targeted the same endpoint.
- Object identifiers were modified.
- HTTP status returned 200 OK.
- Response sizes differed.

This suggests the application returned different records for different user IDs.

---

### Step 4 – Endpoint Investigation

Victim System:

```text
Hostname: WebServer1005
IP: 172.16.17.15
OS: Windows Server 2019
```

Review of endpoint activity found:

- No command execution
- No PowerShell activity
- No shell access
- No malware execution
- No persistence mechanisms
- No privilege escalation

The attack remained at the application layer.

---

### Step 5 – Determine Success of Attack

Indicators suggesting successful exploitation:

- Sequential user ID enumeration
- Successful HTTP 200 responses
- Different response sizes
- Requests were permitted

These findings strongly indicate that unauthorized data access likely occurred.

---

## MITRE ATT&CK Mapping

| Technique | Description |
|------------|------------|
| T1190 | Exploit Public-Facing Application |
| T1083 | File and Directory Discovery (Potential Enumeration Behavior) |

---

## Containment

Actions performed:

- Endpoint isolated/contained
- Investigation completed
- Escalation recommended to application security team

---

## Root Cause

The web application appears to allow direct access to user objects through the `user_id` parameter without sufficient authorization controls.

---

## Verdict

### TRUE POSITIVE

The activity is consistent with a likely successful IDOR attack involving unauthorized object enumeration.

Although no operating system compromise was identified, evidence suggests unauthorized application-level access to user records.

---

## Recommendations

1. Implement server-side authorization checks.
2. Validate ownership before returning user records.
3. Replace direct object references with indirect identifiers.
4. Enable detailed application logging.
5. Conduct code review of affected endpoint.
6. Monitor for further enumeration attempts.

---

## Lessons Learned

- HTTP 200 responses alone do not confirm compromise.
- Different response sizes can indicate exposure of distinct records.
- IDOR attacks often leave no evidence in terminal history.
- Application logs are critical for validating unauthorized access.

## Visit https://app.letsdefend.io/case-management/casedetail/IsaacD/119 for detailed investigation report on letsdefend.
