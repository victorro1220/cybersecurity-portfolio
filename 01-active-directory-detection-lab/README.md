# Active Directory Detection Lab

## Objective

Build a small enterprise-style Active Directory environment that will later be integrated with Sysmon and Wazuh SIEM for detection engineering and SOC investigation practice.

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
- SOC-style event analysis

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

The internal network allows communication between the host and the Domain Controller while keeping the lab isolated from the physical network.

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

The purpose of this configuration is to simulate realistic account security controls and later generate authentication events for SIEM analysis.

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

The domain account policy was verified using:

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
- Basic SOC investigation workflow

---

## Security Events Covered

| Event ID | Description |
|---|---|
| 4625 | Failed logon |
| 4740 | User account locked out |

Future stages of the lab will introduce additional Windows and Sysmon events.

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
- [ ] Sysmon deployed
- [ ] Sysmon configuration applied
- [ ] Wazuh SIEM deployed
- [ ] Wazuh agent connected
- [ ] Windows Security Logs ingested into Wazuh
- [ ] Sysmon logs ingested into Wazuh
- [ ] Detection rules created
- [ ] Controlled attack simulation performed
- [ ] SIEM alert investigation documented
- [ ] Final incident report completed

---

## Next Steps

The next phase of the project will add:

1. Sysmon
2. Enhanced endpoint telemetry
3. Wazuh SIEM
4. Windows event ingestion
5. Sysmon event ingestion
6. Detection engineering
7. Authentication attack simulation
8. Alert correlation
9. SOC investigation
10. Incident reporting
