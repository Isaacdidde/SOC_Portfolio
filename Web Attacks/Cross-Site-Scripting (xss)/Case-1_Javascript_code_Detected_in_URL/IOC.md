# Indicators of Compromise (IOCs)

## Source

| Indicator | Value        |
| --------- | ------------ |
| Source IP | 112.85.42.13 |

## Destination

| Indicator      | Value         |
| -------------- | ------------- |
| Destination IP | 172.16.17.17  |
| Hostname       | WebServer1002 |

## Web Request Indicators

| Indicator      | Value                      |
| -------------- | -------------------------- |
| HTTP Method    | GET                        |
| Attack Type    | Cross-Site Scripting (XSS) |
| Response Codes | 200, 302                   |

## Malicious Payload

```html
<script>alert(1)</script>
```

## Detection Rule

| Indicator | Value                                              |
| --------- | -------------------------------------------------- |
| Event ID  | 116                                                |
| Rule Name | SOC166 - Javascript Code Detected in Requested URL |
| Category  | Web Attack                                         |

## MITRE ATT&CK

| Technique ID | Technique                         |
| ------------ | --------------------------------- |
| T1190        | Exploit Public-Facing Application |
| T1595        | Active Scanning                   |

```
```
