# SOC170 - Passwd Found in Requested URL - Possible LFI Attack

## Overview

This investigation analyzes a suspected Local File Inclusion (LFI) attack detected by the SIEM. The attacker attempted to access the Linux sensitive file `/etc/passwd` through directory traversal sequences within the URL.

The request was successfully delivered to the web server; however, the attack did not achieve its objective. The server responded with **HTTP 500 (Internal Server Error)** and returned **0 bytes**, indicating that no file content was disclosed.

---

## Alert Information

| Field            | Value                                                        |
| ---------------- | ------------------------------------------------------------ |
| Event ID         | 120                                                          |
| Rule             | SOC170 - Passwd Found in Requested URL - Possible LFI Attack |
| Severity         | High                                                         |
| Incident Type    | Web Attack                                                   |
| Request Method   | GET                                                          |
| Destination Host | WebServer1006                                                |
| Destination IP   | 172.16.17.13                                                 |
| Source IP        | 106.55.45.162                                                |
| Device Action    | Permitted                                                    |

---

## Attack Details

### Requested URL

```text
https://172.16.17.13/?file=../../../../etc/passwd
```

### Attack Technique

The attacker attempted to exploit a Local File Inclusion (LFI) vulnerability using directory traversal sequences:

```text
../../../../etc/passwd
```

The objective was to force the application to read and return the contents of the Linux password file.

---

## Investigation Steps

### 1. Validate Alert

The request URL contained a direct reference to:

```text
/etc/passwd
```

This is a well-known target used during LFI exploitation attempts.

### 2. Analyze HTTP Response

| Field         | Value   |
| ------------- | ------- |
| HTTP Status   | 500     |
| Response Size | 0 Bytes |

The server returned an Internal Server Error and no file contents were returned.

### 3. Review Endpoint Information

| Field            | Value               |
| ---------------- | ------------------- |
| Hostname         | WebServer1006       |
| Operating System | Windows Server 2019 |
| Domain           | letsdefend.local    |

Since the target is a Windows-based web server, the Linux file `/etc/passwd` would not normally exist on the host.

### 4. Check Threat Intelligence

VirusTotal analysis showed:

* 0 security vendors flagged the IP as malicious
* Community score: -19
* No strong evidence of active malicious infrastructure

### 5. Review Historical Activity

No previous attacks, related alerts, or suspicious follow-up activity were identified.

---

## MITRE ATT&CK Mapping

| Tactic         | Technique                                 |
| -------------- | ----------------------------------------- |
| Initial Access | T1190 - Exploit Public-Facing Application |
| Discovery      | T1083 - File and Directory Discovery      |

---

## Impact Assessment

| Category                | Result        |
| ----------------------- | ------------- |
| File Disclosure         | No            |
| Successful Exploitation | No            |
| Data Exposure           | No            |
| Persistence             | No            |
| System Impact           | None Observed |

---

## Verdict

### True Positive - Failed LFI Attempt

The alert correctly detected a genuine attempt to access `/etc/passwd` using directory traversal techniques. Although the request reached the server, the attack failed because the application returned an HTTP 500 error and disclosed no data.

No impact to the environment was observed.

---

## Recommendations

1. Validate user-supplied file paths.
2. Implement allowlisting for accessible files.
3. Disable directory traversal patterns.
4. Harden error handling to avoid exposing application behavior.
5. Continue monitoring for repeated LFI attempts from the source IP.
