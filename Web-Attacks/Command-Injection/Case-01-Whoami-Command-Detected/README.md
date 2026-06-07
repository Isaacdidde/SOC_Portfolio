# Case 02 – Command Injection Investigation (SOC168)

## Overview

This case study documents the investigation of a command injection attack detected by the SOC monitoring platform. The alert was triggered after a command execution string (`whoami`) was identified within an HTTP request body targeting a public-facing web application.

The objective of this investigation was to determine whether the activity was malicious, assess the likelihood of successful exploitation, and decide whether escalation was required.

---

## Alert Information

| Field          | Value                                            |
| -------------- | ------------------------------------------------ |
| Event ID       | 118                                              |
| Alert Name     | SOC168 - Whoami Command Detected in Request Body |
| Severity       | High                                             |
| Category       | Web Attack                                       |
| Hostname       | WebServer1004                                    |
| Destination IP | 172.16.17.16                                     |
| Source IP      | 61.177.172.87                                    |
| HTTP Method    | POST                                             |
| Verdict        | True Positive                                    |

---

## Alert Summary

The alert was generated after the string `whoami` was detected within the body of an HTTP POST request.

The `whoami` command is commonly used during command injection attacks to identify the account under which the target application is running. Attackers often use this command as part of the initial reconnaissance phase before attempting privilege escalation or remote command execution.

---

## Investigation Process

### Rule Analysis

The detection rule identified a command execution string within the HTTP request body.

Rule Trigger:

```text
Request Body Contains whoami string
```

This behavior is commonly associated with command injection attempts against web applications that improperly process user-supplied input.

---

### Traffic Analysis

Source IP:

```text
61.177.172.87
```

Destination:

```text
172.16.17.16 (WebServer1004)
```

Direction of Traffic:

```text
Internet → Internal Web Server
```

The attack originated from an external source and targeted a public-facing web application.

---

### Payload Analysis

Review of the request data revealed operating system command execution attempts.

Observed indicators included:

```bash
whoami
uname
```

Additional requests attempted to access sensitive system resources within the Linux `/etc` directory.

These commands are frequently used by attackers to:

* Identify the current user context
* Determine operating system details
* Locate sensitive configuration files
* Prepare for further exploitation

---

### Request Outcome

The web application returned:

```text
HTTP 200 OK
```

A successful HTTP 200 response indicates that the request was processed by the application.

While a 200 response alone does not confirm successful command execution, it significantly increases the likelihood that the payload reached the vulnerable application component.

---

### Planned Activity Verification

A review was conducted to determine whether the activity originated from:

* Authorized penetration testing
* Security validation exercises
* Internal attack simulation tools

No evidence of planned testing activity was identified.

The source IP was determined to be external and unrelated to authorized security operations.

---

## Findings

### Attack Type

Command Injection

### Was the Traffic Malicious?

Yes.

The HTTP requests contained operating system command payloads commonly associated with command injection attacks.

### Was the Attack Successful?

Likely Yes.

The server processed the malicious requests and returned HTTP 200 responses. The presence of command execution payloads combined with successful application responses suggests that exploitation may have been successful.

### Was Tier 2 Escalation Required?

Yes.

Command injection vulnerabilities can lead to:

* Remote Command Execution (RCE)
* Sensitive file disclosure
* Privilege escalation
* Malware deployment
* Full server compromise

Due to the potential impact, escalation to Tier 2 was warranted for further validation and containment activities.

---

## Indicators of Compromise

### Source IP

```text
61.177.172.87
```

### Destination IP

```text
172.16.17.16
```

### Hostname

```text
WebServer1004
```

### HTTP Method

```text
POST
```

### Observed Commands

```bash
whoami
uname
```

### Target Resources

```text
/etc/*
```

---

## MITRE ATT&CK Mapping

| Technique ID | Technique                         |
| ------------ | --------------------------------- |
| T1190        | Exploit Public-Facing Application |
| T1059        | Command and Scripting Interpreter |
| T1059.004    | Unix Shell                        |
| T1082        | System Information Discovery      |

---

## Conclusion

The investigation confirmed a True Positive command injection attack targeting WebServer1004.

Analysis identified operating system command execution attempts within HTTP request bodies, including the use of `whoami` and `uname` commands. The attacker also attempted to access sensitive Linux system resources. The application returned HTTP 200 responses, indicating that the requests were processed successfully.

Given the potential for remote command execution and system compromise, the incident was escalated for deeper investigation and impact assessment.

---

## Key Takeaways

* Command injection attacks are significantly more severe than many other web application attacks due to their ability to interact directly with the operating system.
* Commands such as `whoami`, `uname`, and file access attempts are strong indicators of exploitation attempts.
* HTTP 200 responses may indicate successful payload delivery and should be investigated carefully.
* Command injection incidents frequently require escalation because of the risk of full server compromise.
* Reviewing request bodies, parameters, and server responses is essential during web attack investigations.


## Full Documentation and investingation is available at: https://app.letsdefend.io/case-management/casedetail/IsaacD/118