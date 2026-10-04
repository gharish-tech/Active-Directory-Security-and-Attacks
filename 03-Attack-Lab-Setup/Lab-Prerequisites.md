# Lab Prerequisites

## Objective

This document defines the prerequisites required before starting the Active Directory attack lab.

The goal is to ensure that the Kali Linux attack machine can communicate with the existing Active Directory environment and that the lab network is configured correctly before performing reconnaissance or security testing.


## 1. Existing Active Directory Environment

The attack lab uses the existing Active Directory environment created in the **Enterprise Active Directory Lab**.

### Domain

Domain: harish.local

### Domain Controller

Hostname: DC01
Operating System: Windows Server 2022
IP Address: 192.168.182.10
DNS Server: 192.168.182.10

### Windows Client

Hostname: CLIENT01
Operating System: Windows 11

### Virtualization Platform

VMware Workstation

### Network

Network Type: VMware NAT
Network: 192.168.182.0/24
Gateway: 192.168.182.2

## 2. Attack Machine

The primary attack machine for this project is:

Operating System: Kali Linux
Platform: VMware Workstation
Network Interface: eth0

Kali Linux is used for security testing, reconnaissance, enumeration and controlled attack demonstrations against the isolated Active Directory lab.

## 3. Lab Architecture

                         Internet
                            │
                            │
                     VMware NAT Gateway
                       192.168.182.2
                            │
             ┌──────────────┴──────────────┐
             │                             │
          Kali Linux                    DC01
        Attack Machine             Windows Server 2022
                                      192.168.182.10
                                            │
                                      Active Directory
                                      harish.local
                                            │
                                      ┌─────┴─────┐
                                      │           │
                                   AD DS        DNS
                                      │
                                      │
                                  CLIENT01
                                  Windows 11


All machines used for security testing should remain inside the controlled VMware lab environment.

## 4. Network Requirements

The Kali Linux machine must be connected to the same VMware network as the Active Directory environment.

The important requirement is:


Kali
   │
   │ Same virtual network
   ▼
DC01
192.168.182.10

Kali does not need to use the same IP address as DC01.

Each machine must have its own unique IP address.

## 5. Gateway vs DNS

It is important to understand that the gateway and DNS server have different purposes.

### Gateway

192.168.182.2

The VMware NAT gateway provides the route for traffic leaving the local virtual network.

### AD DNS

192.168.182.10

DC01 also provides DNS for the Active Directory domain.

For AD security testing, Kali should normally use:

DNS → 192.168.182.10

This allows Kali to resolve:

dc01.harish.local
harish.local

and Active Directory service records.

## 6. Kali Network Connection

The active Kali network connection is:

Connection: Wired connection 1
Interface: eth0
Type: Ethernet


The connection can be inspected using:

```bash
nmcli connection show --active
```

Example:

NAME                TYPE      DEVICE
Wired connection 1  ethernet  eth0

## 7. Configure Kali DNS for the AD Lab

For Active Directory lab activities, configure Kali to use DC01 as its DNS server.

```bash
nmcli connection modify "Wired connection 1" ipv4.dns "192.168.182.10"
```

Disable automatically supplied DNS servers:

```bash
nmcli connection modify "Wired connection 1" ipv4.ignore-auto-dns yes
```

Restart the connection:

```bash
nmcli connection down "Wired connection 1"
nmcli connection up "Wired connection 1"
```

Verify:

```bash
nmcli device show eth0 | grep DNS
```

Expected:

IP4.DNS[1]: 192.168.182.10

## 8. Restore Default VMware DNS

The Kali machine can be returned to its normal automatically supplied DNS configuration when AD DNS is not required.

Enable automatic DNS:

```bash
nmcli connection modify "Wired connection 1" ipv4.ignore-auto-dns no
```

Remove the manually configured DNS:

```bash
nmcli connection modify "Wired connection 1" ipv4.dns ""
```

Restart the connection:

```bash
nmcli connection down "Wired connection 1"
nmcli connection up "Wired connection 1"
```

Verify:

```bash
nmcli device show eth0 | grep DNS
```

The DNS server should then be supplied automatically by VMware/network configuration.


# 9. Network Connectivity Verification

Before beginning attack activities, basic connectivity should be verified.

## 9.1 Check IP Configuration
ip addr

The active network interface should be:
eth0

and should have an IPv4 address in the lab network.


## 9.2 Check Routing

ip route

The routing table should contain a route for the local VMware network and normally a default route through:

192.168.182.2


## 9.3 Test DC01 Connectivity

ping -c 4 192.168.182.10

A successful test indicates that Kali can communicate with DC01 at the network layer.

### Verified Lab Result

The Kali machine successfully reached DC01:

4 packets transmitted
4 received
0% packet loss

Therefore:

Kali → DC01 Network Connectivity: PASS

# 10. DNS Verification

After configuring Kali to use DC01 as DNS, test:

nslookup dc01.harish.local

Expected result:

Server:    192.168.182.10
Address:   192.168.182.10#53

Name:      dc01.harish.local
Address:   192.168.182.10

Also test the domain:

nslookup harish.local

# 11. Active Directory Service Record Verification

Active Directory relies on DNS service records to locate domain services.

The LDAP domain-controller service record can be tested with:
nslookup -type=SRV _ldap._tcp.dc._msdcs.harish.local

This verifies that the Active Directory DNS records required for locating domain controllers are available.

# 12. Lab Isolation

All attack activities in this project must be performed against the intentionally created lab environment.

The lab should contain:
Kali Linux
      │
      ▼
Active Directory Lab
      │
 ┌────┴────┐
 ▼         ▼
DC01     CLIENT01

Do not perform attack techniques against:

* Home routers
* Public networks
* College/company networks
* Public websites
* Systems that you do not own or have permission to test

The purpose of this project is controlled cybersecurity learning and attack/defense experimentation.

# 13. VMware Snapshot Strategy

Snapshots should be created before major attack demonstrations.

Recommended snapshots:

DC01
 └── Clean AD State

CLIENT01
 └── Clean Client State

Kali
 └── Clean Attack Machine State

If an experiment changes the environment unexpectedly, the VM can be restored to the appropriate clean state.

# 14. Hardware Considerations

The lab is designed to run on a limited-resource system.

Current host hardware:

CPU: Intel Core i3
RAM: 8 GB
Storage: 512 GB

Because of the limited RAM, additional servers such as:
WEB01
SQL01
FILE01

will not be added by default.

Additional machines can be introduced later only when a particular attack scenario requires them.

# 15. Prerequisite Checklist

| Requirement                    | Status        |
| ------------------------------ | ------------- |
| VMware Workstation             | Ready         |
| Windows Server 2022 DC01       | Ready         |
| Windows 11 CLIENT01            | Ready         |
| Active Directory domain        | Ready         |
| Domain: `harish.local`         | Ready         |
| DC01 IP: `192.168.182.10`      | Ready         |
| Kali Linux VM                  | Ready         |
| Kali `eth0` interface          | Ready         |
| Kali → DC01 ping               | **PASS**      |
| Kali → AD DNS                  | **To verify** |
| `dc01.harish.local` resolution | **To verify** |
| AD LDAP SRV record resolution  | **To verify** |
| VMware routing                 | **To verify** |
| VM snapshots                   | Recommended   |

# 16. Why These Prerequisites Matter

Most Active Directory security tools depend heavily on network communication and DNS.

The general relationship is:

Network Connectivity
        ↓
DNS Resolution
        ↓
Domain Controller Discovery
        ↓
LDAP / Kerberos / SMB Communication
        ↓
AD Enumeration
        ↓
Security Testing

If DNS is incorrect, later tools such as BloodHound and other AD enumeration tools may fail or produce incomplete results.

Therefore, network and DNS verification must be completed before beginning reconnaissance.

# 17. Lab Status

Current status:

Module 03 — Attack Lab Setup
        │
        ▼
Lesson 1 — Lab Prerequisites
        │
        ├── VMware environment       ✓
        ├── Kali Linux               ✓
        ├── DC01 connectivity        ✓
        ├── AD DNS configuration     In Progress
        ├── DNS resolution           Pending
        └── AD service discovery     Pending

The remaining network/DNS verification should be completed before moving to the next document:

Kali-Linux.md

# Interview Questions

## 1. Why does Kali need to communicate with the Domain Controller?

Because the Domain Controller provides important Active Directory services such as DNS, LDAP and Kerberos that are required for AD enumeration and security testing.

## 2. Why should Kali use DC01 as its DNS server during AD testing?

Because Active Directory depends on DNS to resolve domain names and locate services such as domain controllers and LDAP/Kerberos services.

## 3. Is the DNS server the same as the gateway?

No.

The gateway routes traffic between networks, while DNS resolves names to IP addresses and helps locate network services.

## 4. Why can't Kali simply use a public DNS server such as 8.8.8.8 for the AD lab?

A public DNS server does not contain the private DNS records for the `harish.local` Active Directory domain.

## 5. Why are VMware snapshots useful?

They provide a recovery point before security experiments so that the lab can be restored after an attack demonstration or configuration change.

## 6. Why is lab isolation important?

It prevents security-testing activities from affecting systems or networks that the tester does not own or have permission to test.

# Key Takeaways

1. Kali is the attack machine.
2. DC01 is the Domain Controller.
3. CLIENT01 is the Windows client.
4. All machines communicate through the VMware lab network.
5. Gateway = 192.168.182.2
6. DC01/AD DNS = 192.168.182.10
7. Kali should use DC01 DNS during AD testing.
8. Network connectivity must work before enumeration.
9. DNS must work before AD service discovery.
10. Snapshots provide recovery points.

## Next

After the remaining DNS/SRV verification passes, the next lesson is:

**`Kali-Linux.md` — understanding and preparing Kali specifically for the AD attack lab.**
