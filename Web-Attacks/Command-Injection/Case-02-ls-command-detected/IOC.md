# IOC Report – Event ID 117

## Incident Summary

| Field | Value |
|---------|---------|
| Event ID | 117 |
| Incident Type | Web Attack |
| Rule | SOC167 - LS Command Detected in Requested URL |
| Verdict | False Positive |

---

## Network Indicators

### Source IP

```text
172.16.17.46
```

Type:

```text
Internal Client
```

Status:

```text
Benign
```

---

### Destination IP

```text
188.114.96.15
```

Owner:

```text
Cloudflare
```

Status:

```text
Benign
```

---

## URL Indicator

Observed URL:

```text
https://letsdefend.io/blog/?s=skills
```

Detection Trigger:

```text
skills
```

Matched String:

```text
ls
```

Assessment:

```text
False Positive
```

---

## User Agent

```text
Mozilla/5.0 (X11; Ubuntu; Linux i686; rv:24.0)
Gecko/20100101 Firefox/24.0
```

Assessment:

```text
Normal Browser Activity
```

---

## Indicators Found

| IOC Type | Value | Malicious |
|-----------|---------|------------|
| Source IP | 172.16.17.46 | No |
| Destination IP | 188.114.96.15 | No |
| URL | letsdefend.io/blog/?s=skills | No |
| User-Agent | Firefox/24.0 | No |

---

## Indicators Not Found

- Command Injection Payload
- Reverse Shell
- Suspicious Process Creation
- Persistence Activity
- Malware Download
- C2 Communication
- Privilege Escalation
- Lateral Movement

---

## Final Assessment

No malicious indicators of compromise were identified during the investigation.

The alert was triggered because the word:

```text
skills
```

contains the substring:

```text
ls
```

This resulted in a false positive detection.

**Final Verdict: FALSE POSITIVE**