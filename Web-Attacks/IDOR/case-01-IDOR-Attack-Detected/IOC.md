# Indicators of Compromise (IOC) Report

## Incident Information

| Field | Value |
|---------|---------|
| Event ID | 119 |
| Incident Type | Web Attack |
| Detection Rule | SOC169 - Possible IDOR Attack Detected |
| Severity | Medium |
| Verdict | True Positive |

---

# Source Indicators

## Source IP

```text
134.209.118.137
```

Type:

```text
Public IP Address
```

Assessment:

```text
Attack Source
```

---

# Victim Indicators

## Destination IP

```text
172.16.17.15
```

Hostname:

```text
WebServer1005
```

Operating System:

```text
Windows Server 2019
```

Assessment:

```text
Victim Web Server
```

---

# URL Indicators

## Target Endpoint

```text
https://172.16.17.15/get_user_info/
```

---

# Observed Parameters

## Request 1

```text
user_id=2
```

Response:

```text
HTTP Status: 200
Response Size: 253 Bytes
```

---

## Request 2

```text
user_id=5
```

Response:

```text
HTTP Status: 200
Response Size: 267 Bytes
```

---

# Attack Characteristics

Observed Behavior:

- Consecutive requests
- Object ID manipulation
- Parameter tampering
- User enumeration
- Application layer attack

---

# Evidence of Exploitation

| Indicator | Observation |
|------------|-------------|
| Consecutive Requests | Yes |
| Modified Object IDs | Yes |
| HTTP 200 Responses | Yes |
| Different Response Sizes | Yes |
| Endpoint Access Granted | Yes |
| Unauthorized Enumeration | Suspected |

---

# Indicators Not Found

The following were NOT observed:

- Malware execution
- PowerShell activity
- Shell access
- Privilege escalation
- Persistence
- Lateral movement
- Command injection
- Remote code execution

---

# MITRE ATT&CK

| Technique ID | Technique |
|--------------|-----------|
| T1190 | Exploit Public-Facing Application |

---

# Final Assessment

The investigation identified multiple requests manipulating the `user_id` parameter against the `/get_user_info/` endpoint.

The server returned successful HTTP 200 responses with varying response sizes, indicating access to different application objects.

No evidence of host-level compromise was identified; however, the activity strongly suggests successful application-level enumeration consistent with an IDOR vulnerability.

## Final Verdict

```text
TRUE POSITIVE
Likely Successful IDOR Exploitation
```
