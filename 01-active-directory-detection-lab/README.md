# Active Directory Detection Lab

## Objective

Build a small enterprise-style Active Directory environment that will later be integrated with Sysmon and Wazuh SIEM for detection engineering and SOC investigation practice.

## Environment

### Host System
- Windows 11
- Oracle VirtualBox 7.2.16
- 8 GB RAM
- Intel Core i3-1005G1

### Domain Controller
- Windows Server 2022 Standard Evaluation
- Hostname: DC01
- 2 GB RAM
- 1 vCPU
- 50 GB virtual disk

## Network Architecture

DC01 uses two network interfaces.

### Adapter 1 — NAT
- IPv4: 10.0.2.15
- Purpose: Internet connectivity

### Adapter 2 — Host-Only
- IPv4: 192.168.56.10/24
- Purpose: Internal Active Directory lab network

### Host
- VirtualBox Host-Only IP: 192.168.56.1/24

## Active Directory Deployment

Installed components:

- Active Directory Domain Services
- DNS Server
- Group Policy Management
- Active Directory administrative tools

Domain configuration:

- Forest: `cyberlab.local`
- NetBIOS domain: `CYBERLAB`
- Domain Controller: `dc01.cyberlab.local`

## DNS Troubleshooting

During deployment, the Domain Controller initially registered both network interfaces in Active Directory DNS.

### Incorrect DNS registration

`cyberlab.local → 10.0.2.15`

### Correct DNS registration

`cyberlab.local → 192.168.56.10`

### Root Cause

The VirtualBox NAT interface was configured to automatically register its IP address in DNS.

### Remediation

DNS registration was disabled on the NAT interface and the stale `10.0.2.15` A record was removed from the `cyberlab.local` DNS zone.

### Validation

DNS resolution was tested using:

    nslookup dc01.cyberlab.local 192.168.56.10

Result:

    Name:    dc01.cyberlab.local
    Address: 192.168.56.10

## Current Status

- [x] Windows Server 2022 deployed
- [x] DC01 hostname configured
- [x] Dual network adapters configured
- [x] Static internal IP configured
- [x] Active Directory Domain Services installed
- [x] `cyberlab.local` forest created
- [x] DNS verified
- [x] Multihomed DNS registration issue remediated
- [ ] Organizational Units created
- [ ] Domain users and groups created
- [ ] Group Policies configured
- [ ] Sysmon deployed
- [ ] Wazuh SIEM integrated
- [ ] Detection rules created
- [ ] Authentication attack simulation performed
- [ ] SOC investigation documented
