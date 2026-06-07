# Case 01 - Cross-Site Scripting (XSS) Investigation

## Overview

This case study documents the investigation of a web attack alert involving JavaScript code detected within a requested URL. The objective was to determine whether the activity was malicious, identify the attack type, assess its impact, and decide whether escalation was required.

---

## Alert Information

| Field      | Value                                              |
| ---------- | -------------------------------------------------- |
| Event ID   | 116                                                |
| Alert Name | SOC166 - Javascript Code Detected in Requested URL |
| Category   | Web Attack                                         |
| Severity   | Medium                                             |
| Verdict    | True Positive                                      |

---

## Alert Summary

The alert was generated after JavaScript code was identified within an HTTP request URL parameter. Such behavior is commonly associated with Cross-Site Scripting (XSS) attacks, where an attacker attempts to inject executable JavaScript into a web application.

The detection rule specifically identified JavaScript content embedded within the requested URL and generated an alert for further investigation.

---

## Investigation Process

### Rule Analysis

The alert name indicated that JavaScript code was present within a requested URL. This immediately suggested possible XSS activity because legitimate users typically do not include script tags or executable JavaScript within search parameters or URL inputs.

---

### Traffic Analysis

The traffic originated from an external IP address and targeted an internal web server.

**Source IP:** 112.85.42.13

**Destination IP:** 172.16.17.17

**Hostname:** WebServer1002

**Direction of Traffic:** Internet → Internal

This direction indicated that an external entity was attempting to interact with a publicly accessible web application.

---

### Request Examination

Review of the HTTP request revealed JavaScript code embedded within a URL parameter.

The observed payload contained:

* Script tags
* JavaScript alert() function

These elements are commonly used during XSS testing and vulnerability discovery activities.

The payload appeared to be designed to determine whether user-supplied input would be reflected back by the application without proper sanitization.

---

### Source Activity Review

Additional log analysis showed multiple requests originating from the same source IP address.

The requests contained indicators consistent with XSS probing activity targeting the web application.

Observed HTTP response codes included:

* HTTP 200 (OK)
* HTTP 302 (Redirect)

These responses indicated that the web server processed the requests.

---

### Planned Activity Verification

A review was performed to determine whether the activity was associated with:

* Authorized penetration testing
* Attack simulation platforms
* Scheduled security assessments

No evidence of planned testing activity was identified.

The source was determined to be an external system rather than an internal security validation platform.

---

## Findings

### Attack Type

Cross-Site Scripting (XSS)

### Was the Activity Malicious?

Yes.

The request contained JavaScript code commonly associated with XSS testing and exploitation attempts. The activity served no legitimate business purpose and matched known web attack patterns.

### Was the Attack Successful?

During the investigation, the application returned HTTP 200 and HTTP 302 responses, indicating that the requests reached and were processed by the target application. Based on the lab's evaluation criteria, the attack was considered successful.

### Was Tier 2 Escalation Required?

No.

The activity originated from an external source and there was no evidence of internal compromise requiring escalation.

---

## Indicators Identified

### Source IP

112.85.42.13

### Destination IP

172.16.17.17

### Hostname

WebServer1002

### Attack Category

Web Attack

### Attack Technique

Cross-Site Scripting (XSS)

---

## MITRE ATT&CK Mapping

### T1190

Exploit Public-Facing Application

The attacker attempted to exploit a publicly accessible web application through malicious input.

### T1595

Active Scanning

The observed behavior was consistent with reconnaissance and vulnerability discovery activities.

---

## Conclusion

The investigation confirmed a True Positive web attack alert involving a Cross-Site Scripting (XSS) attempt against a public-facing web application.

The attacker submitted JavaScript code through a URL parameter in an effort to identify or exploit potential XSS vulnerabilities. Multiple requests were observed from the same source IP address, and the application processed the requests successfully.

No evidence of authorized testing was identified. The activity was therefore classified as malicious and the alert was closed as a True Positive XSS incident.

---

## Key Takeaways

* XSS attacks often begin with simple JavaScript payloads embedded within URL parameters.
* Reviewing the HTTP request contents is critical for identifying web attacks.
* Traffic direction helps determine whether the activity originates from an internal or external source.
* HTTP response codes provide useful context during web application investigations.
* Verification of planned testing activities should be performed before classifying alerts as malicious.
* Proper alert triage requires combining rule analysis, log review, and threat assessment before reaching a conclusion.
