# Active Directory Detection & SIEM Lab

## Overview

This project documents the deployment of a small enterprise-style Active Directory environment and its integration with endpoint telemetry and a Wazuh SIEM.

The lab was built progressively to develop hands-on experience with:

- Windows Server administration
- Active Directory Domain Services
- DNS
- Group Policy
- Windows security auditing
- Sysmon
- Wazuh SIEM
- Detection engineering
- Authentication monitoring
- MITRE ATT&CK mapping
- SOC-style investigation
- Security event correlation
- Troubleshooting

Rather than only deploying tools, the project focuses on understanding how security telemetry is generated, collected, analyzed, correlated, and converted into actionable alerts.

---

# Lab Architecture

## Host System

- Windows 11
- Oracle VirtualBox 7.2.16
- Intel Core i3-1005G1
- 8 GB RAM

Because the host system has limited resources, the lab was designed to minimize simultaneous virtual machine usage.

---

## Domain Controller — DC01

Operating system:

- Windows Server 2022 Standard Evaluation
- Desktop Experience

Virtual hardware:

- 2 GB RAM
- 1 vCPU
- 50 GB virtual disk

Hostname:

`DC01`

Domain:

`cyberlab.local`

NetBIOS domain:

`CYBERLAB`

Internal IP:

`192.168.56.10/24`

---

## Wazuh Server — WAZUH01

Operating system:

- Ubuntu Server 24.04 LTS

Virtual hardware:

- 4 GB RAM
- 2 vCPU
- 60 GB virtual disk

Hostname:

`wazuh01`

Internal IP:

`192.168.56.30/24`

Installed components:

- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard
- Filebeat
- OpenSSH Server

---

# Network Architecture

Both virtual machines use two network interfaces.

## NAT Network

Used for Internet access and downloading packages.

Example DC01 NAT address:

`10.0.2.15`

---

## VirtualBox Host-Only Network

Used for communication between lab systems.

Network:

`192.168.56.0/24`

Systems:

    Windows Host
    192.168.56.1

          │

    VirtualBox Host-Only Network

          │
     ┌────┴────┐
     │         │

    DC01     WAZUH01
    .10        .30

DC01:

`192.168.56.10`

WAZUH01:

`192.168.56.30`

Windows host:

`192.168.56.1`

---

# Windows Server Deployment

Windows Server 2022 Standard Evaluation with Desktop Experience was installed manually in VirtualBox.

Initial virtual machine configuration:

- VM name: `DC01`
- RAM: 2 GB
- CPU: 1 vCPU
- Disk: 50 GB

After installation, the default Windows hostname was changed to:

`DC01`

---

# Active Directory Domain Services

The following Windows Server roles and administrative tools were installed:

- Active Directory Domain Services
- DNS Server
- Group Policy Management
- Active Directory administrative tools

DC01 was promoted to the first Domain Controller in a new forest.

Forest:

`cyberlab.local`

NetBIOS:

`CYBERLAB`

Domain Controller FQDN:

`dc01.cyberlab.local`

DC01 also operates as:

- Domain Controller
- DNS Server
- Global Catalog

---

# DNS Configuration

DC01 was configured with two network interfaces:

## NAT Interface

Purpose:

Internet connectivity.

Address:

`10.0.2.15`

---

## Internal Active Directory Interface

Purpose:

Domain and lab communication.

Static address:

`192.168.56.10/24`

DNS:

`192.168.56.10`

No default gateway was configured on this interface.

---

# DNS Troubleshooting

During Active Directory deployment, DC01 registered both network interfaces in DNS.

Incorrect registration:

`cyberlab.local → 10.0.2.15`

Correct registration:

`cyberlab.local → 192.168.56.10`

## Root Cause

The VirtualBox NAT interface was configured to automatically register itself in DNS.

Because DC01 was multihomed, the Domain Controller registered both its NAT and internal addresses.

This could cause domain clients to resolve the Domain Controller using the wrong network interface.

## Remediation

Automatic DNS registration was disabled on the NAT interface.

The stale DNS record:

`10.0.2.15`

was removed from the `cyberlab.local` DNS zone.

DNS was validated using:

    nslookup dc01.cyberlab.local 192.168.56.10

Expected result:

    Name:    dc01.cyberlab.local
    Address: 192.168.56.10

The test confirmed that the Domain Controller correctly resolved using the internal lab interface.

---

# Organizational Unit Structure

A custom OU structure was created to separate lab resources from the default Active Directory containers.

Structure:

    cyberlab.local
    └── CyberLab
        ├── Groups
        ├── Servers
        ├── Users
        └── Workstations

This provides cleaner administration and allows Group Policies and permissions to be applied to specific object types.

---

# Domain Users

The following test accounts were created inside:

`CyberLab → Users`

Accounts:

- `john.smith`
- `anna.finance`
- `mike.it`
- `soc.analyst`

A lab-only password was configured.

Passwords are intentionally not included in this repository.

---

# Active Directory Security Groups

The following Global Security Groups were created:

- `GG_Employees`
- `GG_Finance`
- `GG_IT`
- `GG_SOC`

## Group Membership

### John Smith

- GG_Employees

### Anna Finance

- GG_Employees
- GG_Finance

### Mike IT

- GG_Employees
- GG_IT

### SOC Analyst

- GG_Employees
- GG_SOC

This structure simulates basic role-based access control.

---

# Account Security Group Policy

A custom Group Policy Object was created:

`CyberLab - Account Security Policy`

The GPO was linked at the domain level.

---

## Account Lockout Policy

Configured settings:

- Account lockout threshold: 5 invalid attempts
- Account lockout duration: 15 minutes
- Reset account lockout counter after: 15 minutes

The purpose was to simulate realistic account security controls and generate events for SIEM detection testing.

---

# Group Policy Troubleshooting

After the custom policy was created, the expected lockout configuration was not initially effective.

Initial result:

    Lockout threshold: Never
    Lockout duration: 30
    Lockout observation window: 30

The GPO link order was investigated.

Observed:

    Link Order 1 → Default Domain Policy
    Link Order 2 → CyberLab - Account Security Policy

Because a lower Link Order number has higher precedence at the same level, the Default Domain Policy was taking precedence.

## Remediation

The custom policy was moved to:

    Link Order 1 → CyberLab - Account Security Policy
    Link Order 2 → Default Domain Policy

Policies were refreshed using:

    gpupdate /force

The effective domain policy was verified using:

    net accounts /domain

Result:

    Lockout threshold:              5
    Lockout duration (minutes):     15
    Lockout observation window:     15

---

# Advanced Audit Policy

Advanced auditing was configured through Group Policy.

---

## Logon / Logoff

Configured:

### Audit Logon

- Success
- Failure

### Audit Account Lockout

- Success

---

## Account Logon

Configured:

### Audit Credential Validation

- Success
- Failure

### Audit Kerberos Authentication Service

- Success
- Failure

### Audit Kerberos Service Ticket Operations

- Success
- Failure

---

## Account Management

Configured:

### Audit Security Group Management

- Success
- Failure

### Audit User Account Management

- Success
- Failure

---

# Audit Policy Validation

The effective configuration was validated using:

    auditpol /get /category:*

Confirmed:

- Logon: Success and Failure
- Account Lockout: Success
- Credential Validation: Success and Failure
- Kerberos Authentication Service: Success and Failure
- Kerberos Service Ticket Operations: Success and Failure
- Security Group Management: Success and Failure
- User Account Management: Success and Failure

---

# Windows Security Event Testing

## Event ID 4625 — Failed Logon

A controlled failed authentication attempt was generated using:

    runas /user:CYBERLAB\john.smith cmd

An intentionally incorrect password was supplied.

Windows generated:

`Event ID 4625 — An account failed to log on`

---

## Event 4625 Analysis

Observed fields:

- Target Account: `john.smith`
- Domain: `CYBERLAB`
- Logon Type: `2`
- Status: `0xC000006D`
- SubStatus: `0xC000006A`
- Source: `::1`
- Authentication Package: `Negotiate`

Substatus:

`0xC000006A`

indicates that the account exists but an incorrect password was supplied.

Logon Type:

`2`

represents an interactive logon.

Because the test was generated locally on DC01, the source address appeared as:

`::1`

which is the IPv6 loopback address.

---

# Account Lockout Testing

The `john.smith` account state was checked using:

    Get-ADUser john.smith -Properties badPwdCount,LockedOut |
    Select-Object SamAccountName,badPwdCount,LockedOut

Initial result:

    badPwdCount: 1
    LockedOut: False

Additional failed logon attempts were generated.

After the fifth failed authentication:

    badPwdCount: 5
    LockedOut: True

---

# Event ID 4740 — Account Locked Out

Windows generated:

`Event ID 4740 — A user account was locked out`

Observed:

- Locked account: `CYBERLAB\john.smith`
- Caller computer: `DC01`
- Domain Controller: `DC01.cyberlab.local`

The event chain was:

    Failed authentication
            ↓
        Event 4625
            ↓
    Repeated authentication failures
            ↓
       badPwdCount = 5
            ↓
        Account locked
            ↓
        Event 4740

---

# Account Remediation

The locked account was restored using PowerShell:

    Unlock-ADAccount -Identity john.smith

Validation:

    Get-ADUser john.smith -Properties badPwdCount,LockedOut |
    Select-Object SamAccountName,badPwdCount,LockedOut

Result:

    badPwdCount: 0
    LockedOut: False

---

# Sysmon Deployment

Microsoft Sysinternals Sysmon was installed to provide enhanced endpoint telemetry.

Sysinternals was stored in:

`C:\Tools\Sysinternals\SysinternalsSuite`

Sysmon configuration:

`C:\Tools\SysmonConfig\sysmonconfig-export.xml`

Installation command:

    Sysmon64.exe -accepteula -i C:\Tools\SysmonConfig\sysmonconfig-export.xml

Sysmon installed successfully.

---

# Sysmon Service Validation

The service was verified using:

    sc query Sysmon64

Result:

    STATE : 4 RUNNING

The Sysmon Event Channel was verified at:

`Applications and Services Logs → Microsoft → Windows → Sysmon → Operational`

---

# Sysmon Event ID 1 — Process Creation

Sysmon Event ID 1 records process creation.

A normal process relationship was observed:

    ServerManager.exe
           ↓
    Configure-SMRemoting.exe

Sysmon exposed fields including:

- Image
- CommandLine
- User
- IntegrityLevel
- ParentImage
- ParentCommandLine
- MD5
- SHA256
- IMPHASH

---

# Controlled PowerShell Sysmon Test

A controlled command was executed:

    powershell.exe -Command "Write-Host 'CyberLab Sysmon Test'"

Sysmon generated Event ID 1.

Observed:

- Image: `powershell.exe`
- User: `CYBERLAB\Administrator`
- Integrity Level: High
- Parent Process: `cmd.exe`
- Command line captured
- MD5 captured
- SHA256 captured
- IMPHASH captured

Process tree:

    cmd.exe
       ↓
    powershell.exe
       ↓
    Write-Host "CyberLab Sysmon Test"

---

# Sysmon Event ID 11 — File Creation

The PowerShell execution also generated:

`Sysmon Event ID 11`

A temporary `.ps1` file was created in the Administrator profile.

The event demonstrated how Sysmon records filesystem activity associated with process execution.

Correlation:

    Event ID 1
    PowerShell process creation
            ↓
    Event ID 11
    Temporary PowerShell file creation

---

# Ubuntu Wazuh Server Deployment

A second VM was created:

`WAZUH01`

Operating system:

Ubuntu Server 24.04 LTS.

Configuration:

- RAM: 4 GB
- CPU: 2 vCPU
- Disk: 60 GB
- Internal IP: `192.168.56.30`

---

# Linux Remote Administration

OpenSSH Server was installed.

The Linux server is remotely administered from the Windows host using:

    ssh victor@192.168.56.30

This provided practical experience with SSH-based Linux server administration.

---

# Static Linux Networking

Netplan was used to configure the network.

Interfaces:

`enp0s3`

- NAT
- DHCP

`enp0s8`

- Host-Only
- Static address

Static address:

`192.168.56.30/24`

---

# Wazuh Resource Optimization

The Wazuh all-in-one stack requires significantly more resources than the Ubuntu base system.

Because this environment runs on limited hardware, additional swap space was configured.

A 4 GB swap file was created:

    sudo fallocate -l 4G /swapfile
    sudo chmod 600 /swapfile
    sudo mkswap /swapfile
    sudo swapon /swapfile

WAZUH01 operated with:

- approximately 4 GB physical RAM
- additional swap
- 2 vCPU

---

# Wazuh Installation

The official Wazuh all-in-one installer was used.

Commands:

    curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh

    sudo bash ./wazuh-install.sh -a

Installed components:

- Wazuh Indexer
- Wazuh Manager
- Filebeat
- Wazuh Dashboard

---

# Wazuh Indexer Troubleshooting

During the initial deployment, the Wazuh Indexer did not complete startup before the systemd timeout.

Observed:

    wazuh-indexer.service: start operation timed out

The installation logs were reviewed.

Java/OpenSearch had started initialization but required more time because of limited virtual hardware.

---

## Indexer Remediation

A systemd override was created:

    sudo mkdir -p /etc/systemd/system/wazuh-indexer.service.d

    printf '[Service]\nTimeoutStartSec=15min\n' |
    sudo tee /etc/systemd/system/wazuh-indexer.service.d/override.conf

Systemd configuration was reloaded:

    sudo systemctl daemon-reload

The Wazuh installer was executed again.

The second attempt successfully initialized the indexer.

Cluster status eventually reached:

`GREEN`

---

# Wazuh Service Validation

The following services were validated:

    sudo systemctl status wazuh-indexer

    sudo systemctl status wazuh-manager

    sudo systemctl status filebeat

    sudo systemctl status wazuh-dashboard

All services successfully reached:

`active (running)`

---

# Wazuh Dashboard

The dashboard became available at:

`https://192.168.56.30`

A self-signed certificate warning was expected in the browser.

---

# Wazuh API Troubleshooting

During initial startup, the dashboard reported:

    timeout of 20000ms exceeded

The Wazuh API was investigated.

API listener:

TCP `55000`

Validation:

    curl -k https://127.0.0.1:55000

The API responded.

Logs showed that some requests were taking significantly longer during initial startup because of resource contention.

Indexer JVM heap configuration was reviewed:

    -Xms1024m
    -Xmx1024m

The issue disappeared after the environment stabilized.

---

# Wazuh Manager Startup Troubleshooting

The Wazuh Manager also experienced slow startup under limited resources.

A systemd timeout override was created to allow additional startup time.

The manager was then restarted and successfully reached:

`active (running)`

This provided additional experience troubleshooting Linux services and resource-constrained SIEM infrastructure.

---

# Wazuh Agent Deployment

The Wazuh Windows agent was installed on:

`DC01`

Agent configuration:

- Name: `DC01`
- IP: `192.168.56.10`
- Manager: `192.168.56.30`
- Group: `Default`

The Windows service was validated using:

    Get-Service wazuhsvc

Result:

`Running`

The Wazuh Dashboard detected:

`1 agent`

---

# Sysmon Integration with Wazuh

The Wazuh Windows Agent configuration was updated to ingest the Sysmon event channel.

Added to `ossec.conf`:

    <localfile>
      <location>Microsoft-Windows-Sysmon/Operational</location>
      <log_format>eventchannel</log_format>
    </localfile>

The agent was restarted:

    Restart-Service wazuhsvc

Sysmon events then became visible in Wazuh.

---

# Wazuh Sysmon Validation

Wazuh successfully received Sysmon Event ID 1 events from:

`DC01`

This validated the complete telemetry pipeline:

    Windows
       ↓
    Sysmon
       ↓
    Wazuh Agent
       ↓
    Wazuh Manager
       ↓
    Wazuh Rule Engine
       ↓
    Wazuh Indexer
       ↓
    Threat Hunting

---

# Stock Wazuh Alert Investigation

A Wazuh rule generated a high-severity alert for:

`svchost.exe`

Rule:

`61618`

Description:

`Sysmon - Suspicious Process - svchost.exe`

Severity:

`12`

MITRE:

`T1055 — Process Injection`

The process executed from:

`C:\Windows\System32\svchost.exe`

Command line:

`svchost.exe -k wsappx -p`

Context suggested the activity was likely legitimate Windows behavior.

This provided practical experience distinguishing between:

- detection
- suspicious behavior
- actual malicious activity
- possible false positives

---

# Custom Detection Engineering

Three custom Wazuh rules were created and validated.

---

## Rule 100100 — Controlled PowerShell Detection

Purpose:

Validate that custom Wazuh detection rules could match Sysmon Event ID 1 command-line telemetry.

Controlled command:

    powershell.exe -Command "Write-Output 'CYBERLAB_CUSTOM_DETECTION_TEST'"

Rule:

    <rule id="100100" level="10">
      <if_sid>61603</if_sid>
      <field name="win.eventdata.image" type="pcre2">(?i)powershell\.exe$</field>
      <field name="win.eventdata.commandLine" type="pcre2">CYBERLAB_CUSTOM_DETECTION_TEST</field>
      <description>CyberLab custom detection - controlled PowerShell execution</description>
      <mitre>
        <id>T1059.001</id>
      </mitre>
      <group>cyberlab,powershell,custom_detection,</group>
    </rule>

Generated alert:

- Rule ID: `100100`
- Level: `10`
- Agent: `DC01`
- Technique: PowerShell
- MITRE: `T1059.001`
- Tactic: Execution

Detection pipeline:

    PowerShell
        ↓
    Sysmon Event ID 1
        ↓
    Wazuh base rule 61603
        ↓
    Custom rule 100100
        ↓
    Level 10 alert

---

# Custom Rule Troubleshooting

During development of rule `100100`, several issues were encountered.

These included:

- duplicated rule IDs
- malformed XML
- Group tags not closed correctly
- custom rule not triggering
- Threat Hunting filters hiding results
- manager startup delays after rule changes

Rules were validated using:

    sudo /var/ossec/bin/wazuh-analysisd -t

Duplicate IDs were located using:

    sudo grep -R -n '<rule id="100100"' \
    /var/ossec/etc/rules \
    /var/ossec/ruleset/rules

The local rules file was eventually rebuilt cleanly.

This provided hands-on experience with Wazuh rule troubleshooting and XML configuration.

---

# Rule 100101 — PowerShell ExecutionPolicy Bypass

A more realistic behavioral detection was created.

The rule detects PowerShell when:

`-ExecutionPolicy Bypass`

or:

`-ep Bypass`

appears in the command line.

Rule:

    <rule id="100101" level="12">
      <if_sid>61603</if_sid>
      <field name="win.eventdata.image" type="pcre2">(?i)powershell\.exe$</field>
      <field name="win.eventdata.commandLine" type="pcre2">(?i)(-ExecutionPolicy\s+Bypass|-ep\s+Bypass)</field>
      <description>CyberLab - PowerShell ExecutionPolicy Bypass detected</description>
      <mitre>
        <id>T1059.001</id>
      </mitre>
      <group>cyberlab,powershell,suspicious_execution,</group>
    </rule>

Controlled test:

    powershell.exe -NoProfile -ExecutionPolicy Bypass \
    -Command "Write-Output 'CYBERLAB_BYPASS_TEST'"

Generated alert:

- Rule ID: `100101`
- Severity: `12`
- Agent: `DC01`
- MITRE ATT&CK: `T1059.001`
- Technique: PowerShell
- Tactic: Execution

This detection is behavior-based rather than relying on a unique test string.

---

# Rule 100102 — Multiple Failed Logons

The third custom rule correlates repeated authentication failures against the same account.

Base Windows event:

`4625`

Base Wazuh rule:

`60122`

Target account:

`john.smith`

Correlation:

- 5 failed authentication attempts
- same target account
- within the configured timeframe

Generated custom alert:

- Rule ID: `100102`
- Severity: `12`
- Frequency: `5`
- Description: `CyberLab - Multiple failed logons detected for same account`
- MITRE ATT&CK: `T1110`
- Technique: Brute Force
- Tactic: Credential Access

Detection chain:

    Failed logon
        ↓
    Windows Event ID 4625
        ↓
    Wazuh rule 60122
        ↓
    Repeated failures
        ↓
    Custom correlation rule 100102
        ↓
    Level 12 alert
        ↓
    MITRE T1110

The same sequence eventually caused Active Directory to generate:

`Event ID 4740 — Account Locked Out`

This creates a complete authentication investigation chain:

    Failed logons
        ↓
    Brute force correlation
        ↓
    Account lockout

---

# MITRE ATT&CK Coverage

Custom detections currently cover:

| Rule | Detection | MITRE Technique | Tactic |
|---|---|---|---|
| 100100 | Controlled PowerShell Execution | T1059.001 PowerShell | Execution |
| 100101 | PowerShell ExecutionPolicy Bypass | T1059.001 PowerShell | Execution |
| 100102 | Multiple Failed Logons | T1110 Brute Force | Credential Access |

---

# Security Events Investigated

| Source | Event ID | Description |
|---|---:|---|
| Windows Security | 4625 | Failed logon |
| Windows Security | 4740 | Account locked out |
| Sysmon | 1 | Process creation |
| Sysmon | 11 | File creation |

---

# Detection Engineering Workflow

The custom detection workflow used during the project was:

    Generate controlled activity
            ↓
    Validate event in Event Viewer
            ↓
    Confirm event reaches Wazuh
            ↓
    Identify Wazuh base rule
            ↓
    Write custom detection
            ↓
    Validate XML
            ↓
    Restart Wazuh Manager
            ↓
    Trigger behavior again
            ↓
    Confirm custom alert
            ↓
    Review event JSON
            ↓
    Map to MITRE ATT&CK
            ↓
    Document findings

---

# Troubleshooting Performed

This lab included troubleshooting of:

- VirtualBox resource constraints
- Windows Server networking
- Active Directory DNS
- multihomed Domain Controller DNS registration
- Group Policy precedence
- Windows audit policy
- account lockout behavior
- VirtualBox clipboard limitations
- SSH administration
- Ubuntu static networking
- Linux swap configuration
- Wazuh Indexer startup timeout
- Wazuh Manager startup timeout
- Wazuh API response timeout
- Wazuh Dashboard health checks
- Sysmon integration
- Wazuh Agent configuration
- duplicated custom rule IDs
- malformed Wazuh rule XML
- Wazuh Threat Hunting filters
- custom rule matching
- event correlation

---

# Skills Demonstrated

## Active Directory

- Windows Server 2022
- Active Directory Domain Services
- Domain Controller deployment
- Organizational Units
- Domain users
- Security groups
- Group membership
- Group Policy
- Account lockout policies
- DNS
- DNS troubleshooting

## Windows Security

- Advanced Audit Policy
- Windows Event Viewer
- Event ID 4625
- Event ID 4740
- PowerShell administration
- Authentication investigation

## Sysmon

- Sysmon installation
- Sysmon XML configuration
- Sysmon Event ID 1
- Sysmon Event ID 11
- process tree analysis
- command-line analysis
- file hashes
- parent-child process relationships
- endpoint telemetry

## Linux

- Ubuntu Server 24.04
- SSH
- Linux CLI
- Netplan
- static networking
- swap management
- systemd
- service troubleshooting
- journalctl

## Wazuh

- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard
- Wazuh Agent
- Filebeat
- Wazuh API
- Windows EventChannel ingestion
- Sysmon ingestion
- Threat Hunting
- custom detection rules
- Wazuh rule inheritance
- event correlation
- SIEM troubleshooting

## Detection Engineering

- custom rule development
- PCRE2 matching
- behavioral detection
- event correlation
- severity assignment
- alert validation
- false-positive analysis
- MITRE ATT&CK mapping

## SOC Analysis

- log analysis
- authentication monitoring
- PowerShell monitoring
- account lockout investigation
- process analysis
- endpoint telemetry analysis
- alert triage
- timeline reconstruction
- detection validation

---

# Final Lab Status

- [x] Windows Server 2022 deployed
- [x] DC01 configured
- [x] Active Directory deployed
- [x] DNS configured
- [x] DNS issue investigated and remediated
- [x] Organizational Units created
- [x] Domain users created
- [x] Security groups created
- [x] Group memberships configured
- [x] Account lockout policy configured
- [x] Group Policy precedence troubleshot
- [x] Advanced Audit Policy configured
- [x] Audit policy validated
- [x] Event ID 4625 generated
- [x] Event ID 4625 investigated
- [x] Event ID 4740 generated
- [x] Account lockout investigated
- [x] Account unlocked
- [x] Sysmon installed
- [x] Sysmon Event ID 1 analyzed
- [x] Sysmon Event ID 11 analyzed
- [x] Ubuntu Server deployed
- [x] SSH configured
- [x] Wazuh SIEM deployed
- [x] Wazuh Indexer configured
- [x] Wazuh Manager configured
- [x] Wazuh Dashboard configured
- [x] Wazuh API validated
- [x] Wazuh Agent installed on DC01
- [x] DC01 connected to Wazuh
- [x] Sysmon integrated with Wazuh
- [x] Wazuh Sysmon alert investigated
- [x] Custom rule 100100 created
- [x] Custom rule 100100 validated
- [x] Custom rule 100101 created
- [x] PowerShell Bypass detection validated
- [x] Custom rule 100102 created
- [x] Multiple failed logons correlated
- [x] MITRE ATT&CK mappings implemented
- [x] Detection engineering workflow documented

---

# Conclusion

This lab evolved from a basic Windows Server deployment into a complete small-scale detection engineering environment.

The environment included:

- Active Directory
- DNS
- Group Policy
- Windows Security auditing
- Sysmon endpoint telemetry
- Ubuntu Linux
- Wazuh SIEM
- Windows agents
- custom detection engineering
- event correlation
- MITRE ATT&CK mapping

Several configuration and performance problems occurred during implementation.

Instead of being skipped, those issues were investigated and documented as part of the project.

The final environment successfully demonstrated the complete security monitoring pipeline:

    User / System Activity
            ↓
       Windows Logs
            ↓
          Sysmon
            ↓
       Wazuh Agent
            ↓
       Wazuh Manager
            ↓
       Detection Rules
            ↓
       Wazuh Indexer
            ↓
      Threat Hunting
            ↓
      SOC Investigation

The project provided practical experience in both infrastructure administration and defensive cybersecurity monitoring.

---

# Next Project

The next portfolio lab will move away from local virtualization and focus on cloud Identity and Access Management.

Planned topics include:

- Microsoft Entra ID
- users and groups
- role-based access control
- least privilege
- MFA
- Conditional Access
- identity lifecycle
- enterprise applications
- sign-in logs
- audit logs
- IAM security investigation
- cloud identity risk assessment
