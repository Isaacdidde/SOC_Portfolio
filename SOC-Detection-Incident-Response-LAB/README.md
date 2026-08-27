HOMELABS --- SOC Attack & Detection Lab

Purpose: A practical isolated SOC lab for learning attack
simulation, Windows/Linux telemetry collection, Splunk detection
engineering, and incident investigation.

Build status: Baseline infrastructure completed and verified on 27
August 2026.

1. Lab Objective

This lab contains:

Kali Linux --- controlled attacker / penetration-testing
workstation

Windows Server --- primary Windows victim / telemetry source

Ubuntu Server --- Linux victim / telemetry source

Splunk Enterprise --- SIEM, indexer, search and detection
platform

Splunk Universal Forwarder (UF) --- telemetry collection from
Windows and Ubuntu

VMware Workstation Pro --- virtualization layer

VMnet10 --- isolated SOC/lab network

VMnet8 NAT --- Internet access for systems that need
updates/downloads

The intended workflow is:

                Internet
                   |
             VMware VMnet8 NAT
                   |
       +-----------+-----------+
       |                       |
    Kali/Ubuntu/etc.       Host machine
       |                       |
       +-----------------------+
                   |
              VMnet10
        192.168.100.0/24
                   |
        +----------+----------+
        |          |          |
      Kali      Windows     Ubuntu
    Attacker     Victim      Linux
        |          |          |
        +----------+----------+
                   |
            Splunk Enterprise
              192.168.100.1
                TCP/9997

Security principle

VMnet10 is the lab network.

Do not bridge the attack network to your physical LAN unless there is a
specific reason and you understand the consequences.

2. Final Architecture

2.1 Network ranges

Network              Purpose                    Example

VMnet10              Isolated SOC/lab network   192.168.100.0/24
VMnet8               VMware NAT / Internet      192.168.22.0/24
Physical Wi-Fi/LAN   Host Internet              Outside the lab

The host's VMnet10 adapter is:

192.168.100.1/24

The host's VMnet8 adapter is:

192.168.22.1/24

The VMnet8 NAT gateway is:

192.168.22.2

2.2 Current VM addressing

The exact address can change if DHCP is used, so verify with ip addr,
ipconfig, or Get-NetIPAddress rather than assuming it.

Current lab values used during this build:

System           Lab adapter                     Lab IP NAT/Internet
adapter

Host / Splunk    VMnet10                192.168.100.1 Physical/NAT as
Enterprise                                              applicable

Kali             VMnet10              192.168.100.129 VMnet8 /
192.168.22.130

Ubuntu           VMnet10              192.168.100.130 VMnet8 /
192.168.22.131

Important

Do not blindly reuse these IPs after rebuilding the VMs. Check the
actual leases.

3. Why Kali Uses Two Network Adapters

Kali has two interfaces for two different purposes:

Kali
 |
 +-- VMnet10
 |     192.168.100.129
 |     Lab/attack network
 |
 +-- VMnet8 NAT
       192.168.22.130
       Internet access

This is intentional.

VMnet10

Used to reach:

Windows Server

Ubuntu Server

Splunk Enterprise

Other lab systems

VMnet8 NAT

Used for:

apt update

downloading tools

Kali updates

Internet access

Why not use VMnet10 for Internet?

Because VMnet10 is intended to be an isolated lab segment. If it is
configured as host-only, there is no route to the Internet.

A common symptom is:

ping google.com
temporary failure in name resolution

Check:

ip route
cat /etc/resolv.conf

The default route should normally use the NAT interface, not the
isolated lab interface.

4. VMware Network Configuration

4.1 VMnet10

Configure a custom VMware network:

VMnet10
Type: Host-only / isolated custom network
Subnet: 192.168.100.0
Mask: 255.255.255.0
Host adapter: 192.168.100.1

DHCP can be enabled or disabled depending on whether you want static
addressing.

For a repeatable SOC lab, static addresses or DHCP reservations are
preferable.

4.2 VMnet8

Leave VMware's NAT network available for Internet access.

Current lab:

VMnet8
Subnet: 192.168.22.0/24
Host adapter: 192.168.22.1
NAT gateway: 192.168.22.2

4.3 VMware adapter selection

Kali

Adapter 1:

Custom: VMnet10

Adapter 2:

NAT: VMnet8

Ubuntu

Adapter 1:

Custom: VMnet10

Adapter 2:

NAT: VMnet8

Windows Server

Use the lab adapter on VMnet10.

A second NAT adapter may be used for Internet updates if needed.

Splunk Enterprise

The host's VMnet10 adapter provides:

192.168.100.1

The Splunk receiving port is:

TCP/9997

5. Software and Official Downloads

Always prefer official sources and verify checksums/signatures where
available.

VMware Workstation Pro

Official Broadcom support/download portal:

https://support.broadcom.com/

Kali's VMware documentation also references VMware's official download
process.

Kali Linux

Official download:

https://www.kali.org/get-kali/

Official Kali VMware images are available from the download page.

Kali documentation:

https://www.kali.org/docs/

Kali specifically recommends obtaining images from official sources and
verifying SHA256/signatures.

Ubuntu Server

Official Ubuntu Server download:

https://ubuntu.com/download/server

Ubuntu documentation:

https://documentation.ubuntu.com/

Windows Server Evaluation

Official Microsoft Evaluation Center:

https://www.microsoft.com/en-in/evalcenter/evaluate-windows-server-2025

The evaluation edition is intended for lab/testing use. Check the
current evaluation terms and expiry information before rebuilding.

Splunk Enterprise

Official download:

https://www.splunk.com/en_us/download/splunk-enterprise.html

Splunk documentation:

https://help.splunk.com/

Splunk Universal Forwarder

Official download:

https://www.splunk.com/en_us/download/universal-forwarder.html

Current download page provides Windows MSI and Linux packages, including
64-bit Debian packages.

Microsoft Sysmon

Official Microsoft Sysinternals page:

https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon

Sysmon documentation:

https://learn.microsoft.com/en-us/sysinternals/

MITRE ATT&CK

https://attack.mitre.org/

Use ATT&CK to map attack activity to tactics and techniques.

Sigma

https://sigmahq.io/sigma/

Useful for studying generic SIEM detection logic and translating
detection ideas into Splunk SPL.

6. Recommended VM Resources

These are practical lab starting points, not hard requirements.

VM                         vCPU        RAM      Disk Network

Splunk Enterprise host       4+   8--16 GB   100+ GB VMnet10
Kali                       2--4    4--8 GB    40+ GB VMnet10 + VMnet8
Windows Server             2--4    4--8 GB    60+ GB VMnet10 + optional NAT
Ubuntu Server                 2    2--4 GB    30+ GB VMnet10 + VMnet8

If the host machine has limited RAM, reduce VM resources and run fewer
machines simultaneously.

7. Splunk Enterprise Setup

7.1 Install Splunk Enterprise

Install Splunk Enterprise on the host machine.

After installation, verify the web interface and log in.

Typical web access:

http://localhost:8000

or:

https://localhost:8000

depending on the installation/configuration.

7.2 Enable receiving on TCP/9997

Splunk Enterprise must listen for UF data.

In Splunk:

Settings
  -> Forwarding and receiving
  -> Configure receiving
  -> New Receiving Port
  -> 9997

Confirm the port is listening.

On Windows host PowerShell:

Get-NetTCPConnection -LocalPort 9997 -State Listen

If required, check the Windows firewall:

Get-NetFirewallRule | Where-Object DisplayName -Match "Splunk"

Do not disable the entire Windows firewall just to troubleshoot. Create
the narrow required rule if necessary.

8. Windows Server Baseline

8.1 Basic configuration

The Windows Server VM was:

Installed

Renamed to WIN-SOC1

Fully updated

Network configured

Standard SOC user created

Verify hostname:

hostname

Verify IPs:

Get-NetIPAddress -AddressFamily IPv4 |
Where-Object {$_.IPAddress -notlike "127.*"} |
Select-Object InterfaceAlias,IPAddress

If the command returns nothing, use:

Get-NetIPConfiguration

or:

ipconfig

9. Windows Security Telemetry

The baseline collects:

Security
System
Application
Setup
PowerShell Operational
Windows Defender Operational
Sysmon Operational

The exact Splunk sourcetypes depend on the UF configuration.

Important Windows events

Useful SOC events include:

    Event ID Meaning

        4624 Successful logon
        4625 Failed logon
        4688 Process creation
        4103 PowerShell module/pipeline activity
        4104 PowerShell Script Block Logging
    Sysmon 1 Process creation
    Sysmon 3 Network connection
   Sysmon 11 File creation
Sysmon 12/13 Registry activity
   Sysmon 22 DNS query

Windows Event 4688 is generated when a new process starts. Microsoft
documents command-line auditing separately; enabling it makes 4688
significantly more useful for SOC analysis.

10. PowerShell Logging

Script Block Logging provides Event ID 4104 in the Windows PowerShell
Operational channel.

Verify:

Get-WinEvent -LogName "Microsoft-Windows-PowerShell/Operational" -MaxEvents 5

Search in Splunk:

index=* host="WIN-SOC1" EventCode=4104 earliest=-30m

PowerShell logging can contain sensitive information. Treat collected
logs as potentially sensitive.

11. Sysmon

Sysmon is installed on WIN-SOC1.

Verify the service:

Get-Service Sysmon*

Verify the event channel:

Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 5 |
Select-Object TimeCreated,Id,ProviderName

The UF input uses:

Microsoft-Windows-Sysmon/Operational

Example UF input:

[WinEventLog://Microsoft-Windows-Sysmon/Operational]
disabled = 0

Verify UF configuration:

cd "C:\Program Files\SplunkUniversalForwarder\bin"

.\splunk.exe btool inputs list "WinEventLog://Microsoft-Windows-Sysmon/Operational" --debug

Search Splunk:

index=* host="WIN-SOC1" source="Sysmon" earliest=-30m

or use the actual sourcetype shown by:

index=* host="WIN-SOC1" earliest=-30m
| stats count by sourcetype

12. Windows UF Verification

Check service:

Get-Service SplunkForwarder

Expected:

Running

Check version:

cd "C:\Program Files\SplunkUniversalForwarder\bin"
.\splunk.exe version

Check forwarding:

.\splunk.exe list forward-server

Expected:

Active forwards:
    192.168.100.1:9997

Check inputs:

.\splunk.exe btool inputs list --debug

Useful filtering:

.\splunk.exe btool inputs list --debug |
Select-String "WinEventLog|Sysmon|PowerShell|Defender"

13. Windows Splunk Verification

First discover what Splunk actually receives:

index=* host="WIN-SOC1" earliest=-30m
| stats count by sourcetype
| sort sourcetype

Do not assume a sourcetype before checking it.

Security:

index=* host="WIN-SOC1" sourcetype="WinEventLog:Security" earliest=-30m

Failed logons:

index=* host="WIN-SOC1" EventCode=4625 earliest=-24h

Process creation:

index=* host="WIN-SOC1" EventCode=4688 earliest=-30m

PowerShell:

index=* host="WIN-SOC1" EventCode=4104 earliest=-30m

Sysmon:

index=* host="WIN-SOC1" source="Sysmon" earliest=-30m

14. Ubuntu Server Baseline

Current Ubuntu hostname:

ubuntu-soc01

Username used during setup:

server-001

Verify:

hostname
whoami

Verify interfaces:

ip -br addr

Expected lab interface:

192.168.100.130

NAT interface:

192.168.22.131

Verify routing:

ip route

15. Ubuntu Splunk UF

UF location:

/opt/splunkforwarder

Verify version:

sudo /opt/splunkforwarder/bin/splunk version

Verify forwarding:

sudo /opt/splunkforwarder/bin/splunk list forward-server

Expected:

Active forwards:
    192.168.100.1:9997

16. Ubuntu Logs Collected

The lab collects:

/var/log/auth.log
/var/log/syslog
/var/log/kern.log
/var/log/apt/history.log
/var/log/audit/audit.log

Corresponding sourcetypes:

linux_secure
linux_syslog
linux_kernel
linux_apt
linux_audit

Index:

linux

17. Ubuntu inputs.conf

The working configuration is conceptually:

[monitor:///var/log/auth.log]
disabled = 0
index = linux
sourcetype = linux_secure
_TCP_ROUTING = *

[monitor:///var/log/syslog]
disabled = 0
index = linux
sourcetype = linux_syslog
_TCP_ROUTING = *

[monitor:///var/log/kern.log]
disabled = 0
index = linux
sourcetype = linux_kernel
_TCP_ROUTING = *

[monitor:///var/log/apt/history.log]
disabled = 0
index = linux
sourcetype = linux_apt
_TCP_ROUTING = *

[monitor:///var/log/audit/audit.log]
disabled = 0
index = linux
sourcetype = linux_audit
_TCP_ROUTING = *

Custom configuration belongs under:

/opt/splunkforwarder/etc/system/local/

Do not edit Splunk's system/default configuration files.

After changes:

sudo /opt/splunkforwarder/bin/splunk restart

18. Ubuntu Auditd

Install:

sudo apt update
sudo apt install auditd audispd-plugins -y

Enable:

sudo systemctl enable --now auditd

Verify:

sudo systemctl status auditd --no-pager

Verify audit events:

sudo tail -20 /var/log/audit/audit.log

19. Ubuntu UF Permissions

This was an important troubleshooting point in the build.

Ubuntu log files commonly look like:

-rw-r----- syslog adm /var/log/auth.log

The UF service account must be able to read them.

Check:

sudo -u splunkfwd head -1 /var/log/auth.log

If you receive:

Permission denied

the forwarder can be connected while still being unable to collect the
log.

Do not solve this by making all logs world-readable. Prefer appropriate
group membership/ACLs or a service-account design that grants only the
required read access.

20. Ubuntu UF Verification

Check configuration:

sudo /opt/splunkforwarder/bin/splunk btool inputs list --debug

Filter:

sudo /opt/splunkforwarder/bin/splunk btool inputs list --debug |
grep -A6 -B2 "/var/log/auth.log"

Check routing:

sudo /opt/splunkforwarder/bin/splunk btool inputs list --debug |
grep "_TCP_ROUTING"

Check metrics:

sudo grep -Ei "group=per_source|auth.log|syslog|audit.log" \
/opt/splunkforwarder/var/log/splunk/metrics.log | tail -50

A healthy UF will show entries such as:

series=/var/log/auth.log
series=/var/log/syslog
series=/var/log/audit/audit.log

and:

group=tcpout_connections
destIp=192.168.100.1
destPort=9997

21. Splunk Linux Index

The Ubuntu UF was configured to send data to:

index=linux

The index must exist on Splunk Enterprise.

Create it:

Settings
 -> Indexes
 -> New Index
 -> Index Name: linux

This was a real issue encountered during the build: the UF was reading
and forwarding events, but Splunk could not return them through
index=linux until the index existed.

22. Ubuntu Splunk Searches

Discover:

index=* earliest=-24h
| stats count by host
| sort - count

Linux index:

index=linux earliest=-30m

Breakdown:

index=linux earliest=-30m
| stats count by host sourcetype source
| sort sourcetype

Authentication:

index=linux sourcetype=linux_secure earliest=-30m

Audit:

index=linux sourcetype=linux_audit earliest=-30m

System:

index=linux sourcetype=linux_syslog earliest=-30m

23. End-to-End Connectivity Tests

From Kali

Lab connectivity:

ping -c 4 192.168.100.1

Windows:

ping -c 4 <WINDOWS_LAB_IP>

Ubuntu:

ping -c 4 192.168.100.130

Internet:

ping -c 4 8.8.8.8

DNS:

ping -c 4 google.com

If IP ping works but DNS fails, inspect:

ip route
cat /etc/resolv.conf

24. SSH to Ubuntu

From the Windows host:

ssh server-001@192.168.100.130

If the username is forgotten:

whoami

or:

ls /home

Ubuntu passwords cannot be retrieved in plaintext. If you still have
sudo access:

sudo passwd server-001

25. Common Problems and Troubleshooting

Problem: VMnet10 has no Internet

This is expected if VMnet10 is isolated/host-only.

Use:

VMnet10 = lab
VMnet8 = NAT/Internet

If a VM needs both, give it two adapters.

Problem: DNS says "temporary failure in name resolution"

Check:

ip route
cat /etc/resolv.conf

You need a valid default route through the NAT interface for Internet
access.

Test:

ping -c 4 8.8.8.8

If that works but:

ping -c 4 google.com

fails, the problem is DNS rather than basic routing.

Problem: UF says "Active" but Splunk has no events

Do not assume "Active" means logs are being indexed.

Check:

sudo /opt/splunkforwarder/bin/splunk list forward-server

Then:

sudo /opt/splunkforwarder/bin/splunk btool inputs list --debug

Then check:

sudo grep -Ei "group=per_source|tcpout_connections" \
/opt/splunkforwarder/var/log/splunk/metrics.log | tail -50

Finally check Splunk Enterprise:

index=* earliest=-30m
| stats count by host sourcetype

Problem: Ubuntu logs exist but are not forwarded

Check permissions:

sudo -u splunkfwd head -1 /var/log/auth.log

If permission is denied, fix access for the UF service account.

Problem: Ubuntu UF reads logs but Splunk does not show them

Check whether the destination index exists.

If the input says:

index = linux

Splunk Enterprise must have:

linux

created as an index.

Problem: Windows data appears but host filter returns nothing

First discover the actual host value:

index=* earliest=-24h
| stats count by host

During this build the Windows host was:

WIN-SOC1

Therefore:

index=* host="WIN-SOC1"

was correct.

Avoid guessing WIN-SOC01.

Problem: Sysmon is configured but no Sysmon events appear

Check locally:

Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 5

Check UF:

.\splunk.exe btool inputs list "WinEventLog://Microsoft-Windows-Sysmon/Operational" --debug

Confirm:

disabled = 0

Restart:

Restart-Service SplunkForwarder

Then generate normal local activity and search:

index=* host="WIN-SOC1" source="Sysmon" earliest=-15m

Problem: PowerShell events are missing

Check:

Get-WinEvent -LogName "Microsoft-Windows-PowerShell/Operational" -MaxEvents 10

For Script Block Logging:

index=* host="WIN-SOC1" EventCode=4104 earliest=-30m

Remember that new PowerShell sessions generate the relevant Script Block
Logging events after logging is enabled.

Problem: btool output is huge

Filter it.

Windows:

.\splunk.exe btool inputs list --debug |
Select-String "Sysmon|PowerShell|Defender|Security|System"

Ubuntu:

sudo /opt/splunkforwarder/bin/splunk btool inputs list --debug |
grep -E "auth.log|syslog|kern.log|audit.log|apt"

Problem: Splunk search returns zero events

Remove filters.

Start:

index=* earliest=-15m

Then:

index=* earliest=-15m
| stats count by host sourcetype

Only after discovering the actual host/sourcetype should you add
filters.

This avoids a common SOC-lab mistake: incorrectly concluding that
collection is broken because the search itself is too restrictive.

26. Snapshot Strategy

Before the attack phase, create clean VMware snapshots.

Recommended snapshots:

01-clean-baseline
02-telemetry-verified

Take the final baseline snapshot after:

Windows updates

Ubuntu updates

UF installation

Sysmon installation

PowerShell logging

Defender logging

Linux audit configuration

Splunk receiving configuration

Successful end-to-end forwarding

Then you can restore victims after destructive exercises.

Do not snapshot an actively compromised system and call that the clean
baseline.

27. Attack-Phase Operating Model

The lab should be used as a controlled environment.

Recommended workflow:

1. Establish baseline
       ↓
2. Perform controlled attack
       ↓
3. Observe endpoint telemetry
       ↓
4. Confirm events in Splunk
       ↓
5. Investigate
       ↓
6. Write SPL detection
       ↓
7. Map to MITRE ATT&CK
       ↓
8. Document findings
       ↓
9. Restore snapshot if required

Do not start by running random malware or destructive payloads.

28. Suggested Attack-Lab Progression

Phase 1 --- Reconnaissance

Goal:

Identify hosts

Identify exposed services

Understand normal network telemetry

Telemetry:

Sysmon network events

Linux audit/network logs

Splunk searches

Phase 2 --- Authentication

Study:

Failed authentication

Successful authentication

SSH activity

Windows logon events

Windows:

4624
4625

Linux:

auth.log
audit.log

Phase 3 --- Execution

Windows:

4688
4104
Sysmon Event ID 1

Linux:

audit.log
auth.log

Phase 4 --- Persistence

Study controlled persistence mechanisms such as:

Scheduled tasks

Services

Cron

Startup mechanisms

Phase 5 --- Privilege Escalation

Study evidence rather than merely executing techniques.

Look for:

New privileged processes

sudo activity

Service changes

Account changes

Token/elevation information

Phase 6 --- Network Activity

Use Sysmon and Linux telemetry to investigate:

Process-to-network relationships

DNS activity

Unexpected destinations

Unusual ports

29. Splunk Detection Development

Do not immediately build hundreds of alerts.

Start with high-value detections:

Windows

Failed logon bursts
Suspicious PowerShell
PowerShell encoded commands
Unexpected process parents
Suspicious child processes
New services
Scheduled task creation
Privileged account activity

Linux

SSH brute force
Successful SSH after failures
Suspicious sudo activity
New users
Privilege changes
Cron persistence
Package installation
Unexpected network activity

Use the actual fields in your collected data rather than assuming a
field exists.

Start with:

index=linux

or:

index=* host="WIN-SOC1"

and inspect the raw events.

30. Useful Splunk Investigation Commands

Discover sourcetypes:

index=* earliest=-24h
| stats count by sourcetype
| sort - count

Discover hosts:

index=* earliest=-24h
| stats count by host
| sort - count

Discover sources:

index=* earliest=-24h
| stats count by source
| sort - count

Inspect fields:

index=* host="WIN-SOC1" earliest=-30m
| head 20

Linux:

index=linux earliest=-30m
| head 20

31. Recommended Documentation Structure

For every attack exercise, document:

Exercise ID
Date/time
Attacker
Victim
Initial condition
Attack objective
Technique
MITRE ATT&CK mapping
Commands/tools used
Expected telemetry
Actual telemetry
Splunk search
Evidence
Detection logic
False positives
Severity
Recommended response
Cleanup
Snapshot restored?

This turns the lab from a collection of VMs into an actual SOC
portfolio.

32. What Is Currently Complete

As of the final baseline:

VMware networking                         COMPLETE
VMnet10 lab network                       COMPLETE
VMnet8 NAT                                COMPLETE

Kali attacker                             COMPLETE

Windows Server                            COMPLETE
Windows updates                           COMPLETE
Windows hostname                          COMPLETE
Windows SOC user                           COMPLETE
Windows UF                                COMPLETE
Security log                              COMPLETE
System log                                COMPLETE
Application log                           COMPLETE
Setup log                                 COMPLETE
PowerShell logging                        COMPLETE
Windows Defender logging                  COMPLETE
Sysmon                                    COMPLETE
Sysmon forwarding                         COMPLETE

Ubuntu Server                             COMPLETE
Ubuntu UF                                 COMPLETE
auth.log                                  COMPLETE
syslog                                    COMPLETE
kern.log                                  COMPLETE
APT history                               COMPLETE
audit.log                                 COMPLETE
linux Splunk index                         COMPLETE

Splunk Enterprise                         COMPLETE
Receiving TCP/9997                        COMPLETE
Windows forwarding                        VERIFIED
Ubuntu forwarding                         VERIFIED

33. Pre-Attack Checklist

Before starting the attack phase:

VMware VMnet10 is isolated

VMnet8 NAT provides Internet where required

Splunk Enterprise is running

TCP/9997 is receiving

Windows UF is active

Ubuntu UF is active

Windows events appear in Search

Ubuntu events appear in Search

Sysmon events appear

PowerShell events appear

Linux audit events appear

Clean VM snapshots exist

Lab-only target IPs are documented

Attack scope is restricted to the lab

Detection searches are ready

34. Golden Rule for Troubleshooting

Troubleshoot the pipeline in this order:

1. Is the source generating events?
             ↓
2. Can the UF read the source?
             ↓
3. Is the UF input enabled?
             ↓
4. Is _TCP_ROUTING correct?
             ↓
5. Is the UF connected to 9997?
             ↓
6. Does the destination index exist?
             ↓
7. Does Splunk index the event?
             ↓
8. Is the search filter correct?
             ↓
9. Only then troubleshoot dashboards/apps

This order prevents wasting time changing Splunk searches when the
actual issue is a file permission, missing index, disabled input, or
network connection.

35. Official Reference Links

VMware Support: https://support.broadcom.com/

Kali Downloads: https://www.kali.org/get-kali/

Kali Documentation: https://www.kali.org/docs/

Ubuntu Server: https://ubuntu.com/download/server

Ubuntu Documentation: https://documentation.ubuntu.com/

Windows Server Evaluation:
https://www.microsoft.com/en-in/evalcenter/evaluate-windows-server-2025

Splunk Enterprise:
https://www.splunk.com/en_us/download/splunk-enterprise.html

Splunk Universal Forwarder:
https://www.splunk.com/en_us/download/universal-forwarder.html

Splunk Documentation: https://help.splunk.com/

Microsoft Sysmon:
https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon

Microsoft PowerShell Logging:
https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_logging

MITRE ATT&CK: https://attack.mitre.org/

Sigma: https://sigmahq.io/sigma/

36. Final Lab Philosophy

The goal is not simply:

Run attack → see alert

The goal is:

Attack
  ↓
Telemetry
  ↓
Evidence
  ↓
Hypothesis
  ↓
SPL query
  ↓
Detection
  ↓
Investigation
  ↓
Incident conclusion
  ↓
Remediation

That workflow is the core of the SOC training environment.

Keep the environment isolated, keep clean snapshots, make one controlled
change at a time, and preserve the telemetry generated by each exercise.
