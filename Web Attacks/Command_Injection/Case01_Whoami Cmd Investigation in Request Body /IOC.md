# Indicators of Compromise (IOCs)

## Alert Information

| Field      | Value                                            |
| ---------- | ------------------------------------------------ |
| Event ID   | 118                                              |
| Alert Name | SOC168 - Whoami Command Detected in Request Body |
| Severity   | High                                             |
| Category   | Web Attack                                       |

---

## Source Information

| Indicator  | Value                                                   |
| ---------- | ------------------------------------------------------- |
| Source IP  | 61.177.172.87                                           |
| User-Agent | Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1) |

---

## Destination Information

| Indicator      | Value                       |
| -------------- | --------------------------- |
| Destination IP | 172.16.17.16                |
| Hostname       | WebServer1004               |
| Requested URL  | https://172.16.17.16/video/ |
| HTTP Method    | POST                        |

---

## Command Injection Indicators

### Observed Commands

```bash
whoami
uname
```

### Targeted Resources

```text
/etc/
/etc/passwd
/etc/shadow
```

### Alert Trigger

```text
Request Body Contains whoami string
```

---

## HTTP Indicators

| Indicator     | Value   |
| ------------- | ------- |
| HTTP Response | 200 OK  |
| Device Action | Allowed |

---

## Attack Classification

| Category          | Value               |
| ----------------- | ------------------- |
| Attack Type       | Command Injection   |
| Traffic Direction | Internet → Internal |
| Verdict           | True Positive       |
| Escalation        | Tier 2 Required     |

---

## MITRE ATT&CK Mapping

| Technique ID | Technique                         |
| ------------ | --------------------------------- |
| T1190        | Exploit Public-Facing Application |
| T1059        | Command and Scripting Interpreter |
| T1059.004    | Unix Shell                        |
| T1082        | System Information Discovery      |

```
```
