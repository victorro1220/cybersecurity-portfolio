# Active Directory Detection Lab

## Objective

Build a small enterprise-style Active Directory environment and progressively add endpoint telemetry, security auditing, Sysmon, and later Wazuh SIEM for detection engineering and SOC investigation practice.

This lab is designed to provide hands-on experience with:

- Windows Server administration
- Active Directory Domain Services
- DNS
- Group Policy
- Security auditing
- Windows Event Logs
- Authentication monitoring
- Account lockout investigation
- PowerShell administration
- Sysmon telemetry
- Process creation analysis
- File creation analysis
- SOC-style event correlation

---

## Environment

### Host System

- Windows 11
- Oracle VirtualBox 7.2.16
- Intel Core i3-1005G1
- 8 GB RAM

### Domain Controller

- Windows Server 2022 Standard Evaluation
- Hostname: `DC01`
- 2 GB RAM
- 1 vCPU
- 50 GB virtual disk

---

## Lab Architecture

The Domain Controller was configured with two virtual network adapters.

### Adapter 1 — NAT

- IPv4: `10.0.2.15`
- Purpose: Internet connectivity

### Adapter 2 — Host-Only

- IPv4: `192.168.56.10/24`
- Purpose: Internal Active Directory lab network

### Host System

- VirtualBox Host-Only IP: `192.168.56.1/24`

The internal network allows communication between the host and the Domain Controller while keeping the lab separated from the physical LAN.

---

## Windows Server Deployment

Windows Server 2022 Standard Evaluation with Desktop Experience was installed manually in VirtualBox.

The virtual machine was configured with:

- VM name: `DC01`
- 2 GB RAM
- 1 vCPU
- 50 GB virtual disk

After installation, the default Windows Server hostname was changed to:

`DC01`

---

## Active Directory Domain Services Deployment

The following roles and tools were installed:

- Active Directory Domain Services
- DNS Server
- Group Policy Management
- Active Directory administrative tools

The server was promoted to a Domain Controller using a new forest.

### Domain Configuration

- Forest: `cyberlab.local`
- NetBIOS domain: `CYBERLAB`
- Domain Controller: `dc01.cyberlab.local`

The Domain Controller was also configured as:

- DNS Server
- Global Catalog

---

## DNS Troubleshooting

During deployment, the Domain Controller initially registered both network interfaces in Active Directory DNS.

### Incorrect DNS Registration

`cyberlab.local → 10.0.2.15`

### Correct DNS Registration

`cyberlab.local → 192.168.56.10`

### Root Cause

The VirtualBox NAT interface was configured to automatically register its IP address in DNS.

Because the Domain Controller was multihomed, both the NAT address and internal lab address were registered.

This could cause domain clients to resolve the Domain Controller using the wrong interface.

### Remediation

DNS registration was disabled on the NAT interface.

The stale DNS record:

`10.0.2.15`

was removed from the `cyberlab.local` DNS zone.

### Validation

DNS resolution was tested using:

    nslookup dc01.cyberlab.local 192.168.56.10

Result:

    Name:    dc01.cyberlab.local
    Address: 192.168.56.10

This confirmed that the Domain Controller was resolving correctly through the internal Active Directory network.

---

## Organizational Unit Structure

A custom Organizational Unit structure was created to separate lab objects from the default Active Directory containers.

Structure:

    cyberlab.local
    └── CyberLab
        ├── Groups
        ├── Servers
        ├── Users
        └── Workstations

This structure allows cleaner administration and makes it easier to apply Group Policies and security controls to specific object types.

---

## Domain Users

Four test domain users were created inside:

`CyberLab → Users`

Accounts:

- `john.smith`
- `anna.finance`
- `mike.it`
- `soc.analyst`

A lab-only password was configured for the accounts.

Passwords are intentionally not published in this repository.

---

## Security Groups

The following Global Security Groups were created:

- `GG_Employees`
- `GG_Finance`
- `GG_IT`
- `GG_SOC`

### Group Membership

- `john.smith`
  - GG_Employees

- `anna.finance`
  - GG_Employees
  - GG_Finance

- `mike.it`
  - GG_Employees
  - GG_IT

- `soc.analyst`
  - GG_Employees
  - GG_SOC

This structure simulates role-based access control inside an organization.

---

## Account Security Group Policy

A custom Group Policy Object was created:

`CyberLab - Account Security Policy`

The GPO was linked at the domain level.

### Account Lockout Policy

The following settings were configured:

- Account lockout threshold: 5 invalid logon attempts
- Account lockout duration: 15 minutes
- Reset account lockout counter after: 15 minutes

The purpose of this configuration is to simulate realistic account security controls and generate authentication events for later SIEM analysis.

---

## Group Policy Troubleshooting

After applying the GPO, the expected lockout policy was not initially visible.

The first validation showed:

    Lockout threshold: Never
    Lockout duration: 30 minutes
    Lockout observation window: 30 minutes

### Investigation

Group Policy link order was reviewed.

Observed configuration:

    Link Order 1 → Default Domain Policy
    Link Order 2 → CyberLab - Account Security Policy

Because a lower Link Order number has higher precedence at the same level, the Default Domain Policy was taking precedence.

### Remediation

The custom GPO was moved to:

    Link Order 1 → CyberLab - Account Security Policy
    Link Order 2 → Default Domain Policy

The policy was refreshed using:

    gpupdate /force

### Validation

The effective account policy was verified using:

    net accounts /domain

Result:

    Lockout threshold:              5
    Lockout duration (minutes):     15
    Lockout observation window:     15

This confirmed that the custom security policy was successfully applied.

---

## Advanced Audit Policy

Advanced auditing was configured through Group Policy to generate security telemetry for future SIEM analysis.

### Logon / Logoff

Configured:

- Audit Logon
  - Success
  - Failure

- Audit Account Lockout
  - Success

### Account Logon

Configured:

- Audit Credential Validation
  - Success
  - Failure

- Audit Kerberos Authentication Service
  - Success
  - Failure

- Audit Kerberos Service Ticket Operations
  - Success
  - Failure

### Account Management

Configured:

- Audit Security Group Management
  - Success
  - Failure

- Audit User Account Management
  - Success
  - Failure

---

## Audit Policy Validation

The effective audit configuration was validated using:

    auditpol /get /category:*

Verified settings included:

- Logon: Success and Failure
- Account Lockout: Success
- Credential Validation: Success and Failure
- Kerberos Authentication Service: Success and Failure
- Kerberos Service Ticket Operations: Success and Failure
- Security Group Management: Success and Failure
- User Account Management: Success and Failure

This confirmed that the advanced audit configuration was successfully applied.

---

## Failed Logon Event Generation

A controlled failed authentication attempt was generated against the test domain account:

`john.smith`

The following command was used:

    runas /user:CYBERLAB\john.smith cmd

An intentionally incorrect password was entered.

Windows generated:

`Event ID 4625 — An account failed to log on`

---

## Security Event Analysis — Event ID 4625

The generated Event ID 4625 was investigated in Windows Event Viewer.

### Event Details

- Event ID: 4625
- Computer: `DC01.cyberlab.local`
- Target Account: `john.smith`
- Target Domain: `CYBERLAB`
- Logon Type: `2`
- Authentication Package: `Negotiate`
- Source Network Address: `::1`

### Failure Information

- Failure Reason: Unknown user name or bad password
- Status: `0xC000006D`
- Sub Status: `0xC000006A`

The substatus:

`0xC000006A`

indicates that the account exists but an incorrect password was supplied.

### Logon Type Analysis

Logon Type `2` represents an interactive logon.

Because the authentication attempt was generated directly from DC01, the source address appeared as:

`::1`

which is the IPv6 loopback address.

### SOC Interpretation

A failed interactive authentication attempt was observed against:

`CYBERLAB\john.smith`

The Windows substatus confirmed that the account existed and that the supplied password was incorrect.

---

## Account Lockout Testing

The account lockout policy was tested using repeated failed authentication attempts.

Before continuing, the account state was inspected using PowerShell:

    Get-ADUser john.smith -Properties badPwdCount,LockedOut | Select-Object SamAccountName,badPwdCount,LockedOut

Initial result:

    badPwdCount: 1
    LockedOut: False

Additional controlled failed authentication attempts were generated.

After five failed attempts:

    badPwdCount: 5
    LockedOut: True

This confirmed that the domain account lockout policy was functioning as expected.

---

## Security Event Analysis — Event ID 4740

When the lockout threshold was reached, Windows generated:

`Event ID 4740 — A user account was locked out`

### Event Details

- Event ID: 4740
- Computer: `DC01.cyberlab.local`
- Locked Account: `CYBERLAB\john.smith`
- Caller Computer: `DC01`
- Subject Account: `DC01$`
- Subject Domain: `CYBERLAB`

### Event Sequence

The security event chain was:

    Failed authentication
            ↓
        Event 4625
            ↓
    Repeated failed attempts
            ↓
       badPwdCount = 5
            ↓
       Account locked
            ↓
        Event 4740

This demonstrates how repeated failed authentication events can be correlated with an account lockout event during a SOC investigation.

---

## Account Remediation

The locked domain account was restored using PowerShell:

    Unlock-ADAccount -Identity john.smith

The account state was then validated using:

    Get-ADUser john.smith -Properties badPwdCount,LockedOut | Select-Object SamAccountName,badPwdCount,LockedOut

Result:

    badPwdCount: 0
    LockedOut: False

This confirmed that the account was successfully unlocked and returned to a normal state.

---

# Sysmon Deployment

Sysmon from Microsoft Sysinternals was installed to add enhanced endpoint telemetry beyond the standard Windows Security logs.

The Sysinternals Suite was extracted to:

`C:\Tools\Sysinternals\SysinternalsSuite`

A separate configuration directory was created:

`C:\Tools\SysmonConfig`

A community Sysmon configuration file was stored as:

`C:\Tools\SysmonConfig\sysmonconfig-export.xml`

Sysmon was installed using:

    Sysmon64.exe -accepteula -i C:\Tools\SysmonConfig\sysmonconfig-export.xml

The installation completed successfully.

---

## Sysmon Service Validation

The Sysmon service was validated using:

    sc query Sysmon64

Result:

    STATE : 4 RUNNING

The Sysmon Operational log was also verified at:

`Applications and Services Logs → Microsoft → Windows → Sysmon → Operational`

Events began appearing immediately after installation.

This confirmed that Sysmon was actively collecting endpoint telemetry.

---

## Sysmon Event ID 1 — Process Creation

Sysmon Event ID 1 records process creation activity.

A normal system event was first observed where:

`ServerManager.exe`

spawned:

`Configure-SMRemoting.exe`

This demonstrated how Sysmon records parent-child process relationships.

Observed fields included:

- Image
- CommandLine
- User
- IntegrityLevel
- ParentImage
- ParentCommandLine
- File hashes

---

## Controlled PowerShell Execution

A controlled PowerShell process was executed to validate Sysmon telemetry.

Command:

    powershell.exe -Command "Write-Host 'CyberLab Sysmon Test'"

Sysmon generated Event ID 1.

### Observed Fields

- Image:
  `C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe`

- CommandLine:
  `powershell.exe -Command "Write-Host 'CyberLab Sysmon Test'"`

- User:
  `CYBERLAB\Administrator`

- Integrity Level:
  `High`

- Parent Process:
  `C:\Windows\System32\cmd.exe`

- Parent User:
  `CYBERLAB\Administrator`

- Current Directory:
  `C:\Tools\Sysinternals\SysinternalsSuite\`

- Hashes captured:
  - MD5
  - SHA256
  - IMPHASH

---

## Sysmon Process Chain Analysis

The process relationship was reconstructed as:

    cmd.exe
        ↓
    powershell.exe
        ↓
    Write-Host "CyberLab Sysmon Test"

This demonstrates how Sysmon can help reconstruct process execution chains during an endpoint investigation.

### SOC Interpretation

PowerShell was launched interactively from `cmd.exe` by:

`CYBERLAB\Administrator`

The command performed a benign test action.

The event itself was not malicious, but it validated that Sysmon was successfully capturing:

- process creation
- command-line arguments
- user context
- integrity level
- parent-child process relationships
- executable hashes

---

## Sysmon Event ID 11 — File Creation

Immediately after the PowerShell execution, Sysmon generated:

`Event ID 11 — File Create`

The event recorded the creation of:

`C:\Users\Administrator\AppData\Local\Temp\__PSScriptPolicyTest_*.ps1`

The process responsible was:

`powershell.exe`

The event also identified:

- Process ID
- Process GUID
- User
- Target filename
- Creation timestamp

### Analysis

The generated `.ps1` file was consistent with normal PowerShell script execution policy testing.

This activity was not considered malicious by itself.

The event demonstrates how Sysmon records secondary filesystem activity generated by a process.

---

## Sysmon Event Correlation

The controlled PowerShell test generated multiple related Sysmon events.

Sequence:

    Event ID 1
    PowerShell process created
            ↓
    Event ID 11
    Temporary PowerShell file created

This demonstrates how multiple endpoint events can be correlated to reconstruct user and process activity.

Later, Wazuh will be used to centralize and analyze this telemetry.

---
# Wazuh SIEM Deployment

## WAZUH01 Server

A dedicated Ubuntu Server virtual machine was deployed to host the Wazuh SIEM environment.

Configuration:

- Hostname: `wazuh01`
- Operating System: Ubuntu Server 24.04 LTS
- RAM: 4 GB
- CPU: 2 vCPU
- Virtual Disk: 60 GB
- Internal IP: `192.168.56.30/24`

The server uses two network interfaces:

- NAT for Internet access
- VirtualBox Host-Only network for communication with the cybersecurity lab

Network architecture:

    Windows Host
    192.168.56.1
          │
          │
    Host-Only Network
          │
     ┌────┴─────┐
     │          │
    DC01      WAZUH01
    .10          .30
     │            │
     │            └── Wazuh SIEM
     │
     └── Active Directory + Sysmon

---

## Linux Remote Administration

OpenSSH Server was installed during the Ubuntu deployment.

The Wazuh server is administered remotely from the Windows host using:

    ssh victor@192.168.56.30

This provides a more practical Linux administration workflow and allows commands to be executed remotely without relying on the VirtualBox console.

---

## Static Network Configuration

The internal Wazuh interface was configured with a static address using Netplan.

Configuration:

- NAT interface: `enp0s3`
- Internal interface: `enp0s8`
- Wazuh internal IP: `192.168.56.30/24`

The internal interface was changed from DHCP to a static address to ensure that Windows agents can consistently connect to the Wazuh manager.

---

## Wazuh Resource Optimization

The Wazuh all-in-one deployment requires significantly more resources than a minimal Ubuntu Server installation.

Because this lab is running on resource-constrained hardware, several adjustments were required.

WAZUH01 was configured with:

- 4 GB RAM
- 2 vCPU
- Approximately 49 GB free disk space
- Additional swap space

A 4 GB swap file was created:

    sudo fallocate -l 4G /swapfile
    sudo chmod 600 /swapfile
    sudo mkswap /swapfile
    sudo swapon /swapfile

This provided additional memory protection during Wazuh installation and service initialization.

---

## Wazuh Installation

Wazuh was deployed using the official all-in-one installation assistant.

The deployment installed:

- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard
- Filebeat

Installation command:

    curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh
    sudo bash ./wazuh-install.sh -a

---

## Wazuh Indexer Startup Troubleshooting

During the initial installation, the Wazuh Indexer failed to finish starting before the systemd startup timeout.

Observed error:

    wazuh-indexer.service: start operation timed out

The installation assistant subsequently removed the incomplete indexer deployment.

### Investigation

The installation log was reviewed at:

    /var/log/wazuh-install.log

System logs showed that the Java/OpenSearch process had begun initializing but did not complete before the service timeout.

The system also experienced temporary CPU contention during the heavy initialization phase.

### Remediation

A systemd override was created to increase the Wazuh Indexer startup timeout:

    sudo mkdir -p /etc/systemd/system/wazuh-indexer.service.d

    printf '[Service]\nTimeoutStartSec=15min\n' | \
    sudo tee /etc/systemd/system/wazuh-indexer.service.d/override.conf

The systemd configuration was reloaded:

    sudo systemctl daemon-reload

The Wazuh installation was then executed again.

### Result

The second installation successfully completed indexer initialization.

The Wazuh Indexer cluster reached:

    GREEN

This allowed the installer to continue with:

- Wazuh Manager
- Filebeat
- Wazuh Dashboard

---

## Wazuh Services Validation

After installation, the following services were validated:

    sudo systemctl status wazuh-indexer
    sudo systemctl status wazuh-manager
    sudo systemctl status filebeat
    sudo systemctl status wazuh-dashboard

All services reached:

    active (running)

The Wazuh Dashboard became available at:

    https://192.168.56.30

The browser displayed a certificate warning because the lab uses a self-signed certificate.

---

## Wazuh API Troubleshooting

During the first dashboard health check, the dashboard displayed:

    Request failed with status code 500
    timeout of 20000ms exceeded

The Wazuh API connection was investigated from WAZUH01.

The API listener was verified on TCP port:

    55000

Connectivity was tested locally using:

    curl -k https://127.0.0.1:55000

The API responded successfully.

Logs showed that some API operations were taking significantly longer during initial service startup because the VM was temporarily under heavy resource load.

The Wazuh Indexer JVM heap was also reviewed:

    -Xms1024m
    -Xmx1024m

No memory change was required.

After allowing the environment to stabilize and refreshing the dashboard, all health checks completed successfully.

This confirmed that the issue was temporary resource contention during the initial SIEM startup rather than an API or authentication failure.

---

## Wazuh Dashboard

The Wazuh Dashboard was successfully accessed from the Windows host using:

    https://192.168.56.30

The following components were verified as operational:

- Wazuh Indexer
- Wazuh Manager
- Wazuh API
- Filebeat
- Wazuh Dashboard

---

## Windows Agent Deployment

The Wazuh Agent was installed on the Active Directory Domain Controller:

`DC01`

Agent configuration:

- Agent name: `DC01`
- Operating System: Windows Server 2022
- Wazuh Manager: `192.168.56.30`
- Group: `Default`

The Windows Wazuh service was validated using:

    Get-Service wazuhsvc

The agent successfully registered with the Wazuh Manager.

The Wazuh Dashboard currently reports:

    1 agent

This confirms successful communication between the Windows Domain Controller and the Wazuh SIEM.

---

## Current SIEM Architecture

The lab currently operates as:

    DC01
    Windows Server 2022
    Active Directory
    Windows Security Logs
    Sysmon
         │
         │ Wazuh Agent
         ▼
    WAZUH01
    Ubuntu Server 24.04
         │
         ├── Wazuh Manager
         ├── Wazuh Indexer
         ├── Filebeat
         └── Wazuh Dashboard

The next phase will configure the Wazuh Agent to explicitly collect Sysmon events from:

    Microsoft-Windows-Sysmon/Operational
## Skills Demonstrated

This lab currently demonstrates hands-on experience with:

- Windows Server 2022
- VirtualBox
- Active Directory Domain Services
- Domain Controller deployment
- DNS configuration
- DNS troubleshooting
- Organizational Units
- Domain users
- Security groups
- Group membership
- Group Policy
- GPO troubleshooting
- Account lockout policies
- Advanced Audit Policy
- Windows Event Viewer
- Windows Security Logs
- PowerShell
- Authentication monitoring
- Event ID 4625 analysis
- Event ID 4740 analysis
- Account lockout investigation
- Account remediation
- Microsoft Sysinternals
- Sysmon deployment
- Sysmon configuration
- Sysmon service validation
- Sysmon Event ID 1 analysis
- Sysmon Event ID 11 analysis
- Process tree analysis
- Parent-child process analysis
- Command-line telemetry
- File hash collection
- Basic endpoint event correlation
- SOC investigation workflow
- Ubuntu Server 24.04
- Linux command-line administration
- SSH remote administration
- Netplan
- Linux networking
- Linux swap management
- systemd
- Linux service troubleshooting
- Wazuh SIEM
- Wazuh Manager
- Wazuh Indexer
- Wazuh Dashboard
- Filebeat
- Wazuh API
- Windows Wazuh Agent
- SIEM infrastructure deployment
- SIEM performance troubleshooting
- Resource-constrained server optimization
---

## Security Events Covered

| Source | Event ID | Description |
|---|---:|---|
| Windows Security | 4625 | Failed logon |
| Windows Security | 4740 | User account locked out |
| Sysmon | 1 | Process creation |
| Sysmon | 11 | File creation |

Additional Windows and Sysmon events will be added as the lab progresses.

---

## Current Status

- [x] Windows Server 2022 deployed
- [x] DC01 hostname configured
- [x] Dual network adapters configured
- [x] Static internal IP configured
- [x] Active Directory Domain Services installed
- [x] `cyberlab.local` forest created
- [x] DNS configured and validated
- [x] Multihomed DNS registration issue remediated
- [x] Organizational Units created
- [x] Domain users created
- [x] Security groups created
- [x] Group memberships configured
- [x] Domain account lockout policy configured
- [x] Group Policy precedence issue investigated and corrected
- [x] Advanced Audit Policy configured
- [x] Audit policy validated
- [x] Failed authentication event generated
- [x] Event ID 4625 investigated
- [x] Account lockout generated
- [x] Event ID 4740 investigated
- [x] Locked account remediated
- [x] Sysmon installed
- [x] Sysmon service validated
- [x] Sysmon Event ID 1 investigated
- [x] Sysmon Event ID 11 investigated
- [x] Basic Sysmon event correlation documented
- [x] Ubuntu WAZUH01 server deployed
- [x] Static Wazuh IP configured
- [x] SSH remote administration configured
- [x] Wazuh resource constraints investigated
- [x] Swap configured
- [x] Wazuh Indexer startup timeout troubleshot
- [x] Wazuh Indexer deployed
- [x] Wazuh Manager deployed
- [x] Filebeat deployed
- [x] Wazuh Dashboard deployed
- [x] Wazuh services validated
- [x] Wazuh API investigated and validated
- [x] Wazuh Dashboard accessed successfully
- [x] DC01 Wazuh Agent installed
- [x] DC01 detected by Wazuh
- [ ] Sysmon channel configured in Wazuh Agent
- [ ] Sysmon events verified inside Wazuh
- [ ] Windows authentication events verified inside Wazuh
- [ ] Custom detection rules created
- [ ] Controlled attack simulation performed
- [ ] SIEM alert investigation documented
- [ ] MITRE ATT&CK mapping completed
- [ ] Final incident report completed

---

## Next Steps

The next phase of the project will introduce:

1. Wazuh SIEM
2. Wazuh agent deployment
3. Windows Security log ingestion
4. Sysmon log ingestion
5. Detection engineering
6. Authentication-based detections
7. Controlled attack simulation
8. Alert correlation
9. MITRE ATT&CK mapping
10. SOC investigation
11. Incident reporting
