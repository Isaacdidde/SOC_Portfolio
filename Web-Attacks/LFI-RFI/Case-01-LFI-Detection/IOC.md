# Indicators of Compromise (IOCs)

## Network Indicators

### Source IP

```text
106.55.45.162
```

### Destination IP

```text
172.16.17.13
```

### Hostname

```text
WebServer1006
```

---

## Web Indicators

### Request Method

```text
GET
```

### Requested URL

```text
https://172.16.17.13/?file=../../../../etc/passwd
```

### Target File

```text
/etc/passwd
```

### Directory Traversal Pattern

```text
../../../../
```

---

## User-Agent

```text
Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; .NET CLR 1.1.4322)
```

---

## HTTP Indicators

### Response Status

```text
500
```

### Response Size

```text
0 Bytes
```

### Device Action

```text
Permitted
```

---

## Detection Opportunities

### Sigma Keywords

```text
/etc/passwd
../
../../
file=
```

### WAF Detection Patterns

```text
/etc/passwd
../
..\

directory traversal
local file inclusion
```

---

## MITRE ATT&CK

```text
T1190 - Exploit Public-Facing Application
T1083 - File and Directory Discovery
```

---

## IOC Summary

| IOC Type       | Value                         |
| -------------- | ----------------------------- |
| Source IP      | 106.55.45.162                 |
| Destination IP | 172.16.17.13                  |
| Target File    | /etc/passwd                   |
| Attack Type    | Local File Inclusion (LFI)    |
| HTTP Status    | 500                           |
| Verdict        | True Positive - Failed Attack |
| Impact         | None                          |
