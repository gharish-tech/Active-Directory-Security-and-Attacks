# Kali Linux

## Objective

This document explains the role of Kali Linux in the Active Directory security lab, its network position, basic system verification, and the tools required for later security-testing activities.

Kali Linux is used as the attack machine in this controlled VMware laboratory.

## 1. Role of Kali Linux

Kali Linux is a Debian-based Linux distribution designed for penetration testing, security research, digital forensics, and related security activities.

In this project, Kali acts as the **security-testing/attack machine**.

The Active Directory environment contains:

DC01
Windows Server 2022
192.168.182.10
        │
        │
        ▼
harish.local
        │
        ▼
CLIENT01
Windows 11

Kali is placed on the same VMware virtual network:

Kali
  │
  │ 192.168.182.0/24
  │
  ├──────────────► DC01
  │                 192.168.182.10
  │
  └──────────────► CLIENT01


## 2. Why Kali Is Used

The attack lab requires a separate machine from which security testing can be performed.

Using Kali provides a controlled environment for:

* Network reconnaissance
* DNS enumeration
* LDAP enumeration
* SMB enumeration
* Active Directory enumeration
* BloodHound data collection
* Credential-security testing
* Kerberos security testing
* NTLM security testing
* Password-spraying demonstrations
* Privilege-escalation research
* Lateral-movement demonstrations
* Detection and defense validation

All testing in this project is performed against the intentionally created laboratory environment.

## 3. Kali and Active Directory

Kali is not a Domain Controller and does not provide the Active Directory domain.

Instead:

Kali
   │
   │ Security Testing
   ▼
Active Directory Environment
   │
   ├── DC01
   │    ├── AD DS
   │    ├── DNS
   │    ├── Kerberos
   │    └── LDAP
   │
   └── CLIENT01

Kali communicates with the existing Windows systems using network protocols and security tools.


## 4. Lab Network Position

Current lab network:

Network: 192.168.182.0/24
Gateway: 192.168.182.2
DC01:    192.168.182.10

Kali uses:

Interface: eth0
Connection: Wired connection 1

The exact Kali IP address may be assigned dynamically by VMware DHCP unless a static address is intentionally configured.

The important requirement is that Kali can communicate with the AD lab.

## 5. DNS Configuration

During Active Directory security testing, Kali uses DC01 as its DNS server:

DNS: 192.168.182.10

This allows Kali to resolve internal AD names such as:

dc01.harish.local
harish.local

and discover Active Directory service records.

Example:

```bash
nslookup dc01.harish.local
```

The LDAP domain-controller SRV record can be tested using:

```bash
nslookup -type=SRV _ldap._tcp.dc._msdcs.harish.local
```

## 6. Basic Kali System Verification

Before installing or configuring security tools, verify the Kali operating system.

### Check Kali version

```bash
cat /etc/os-release
```

### Check kernel

```bash
uname -r
```

### Check hostname

```bash
hostname
```

### Check network interfaces

```bash
ip addr
```

### Check routing table

```bash
ip route
```

### Check active NetworkManager connections

```bash
nmcli connection show --active
```

---

## 7. Verify Internet Connectivity

The attack machine may need Internet connectivity to install or update tools.

Test name resolution:

```bash
ping -c 4 google.com
```

If Internet access is intentionally disabled during an attack exercise, that is also acceptable as long as the required tools are already installed.

The AD lab itself does not require Kali to have Internet access for communication with DC01.


## 8. Verify Connectivity to DC01

Test the Domain Controller:

```bash
ping -c 4 192.168.182.10
```

Expected:

```text
4 packets transmitted
4 received
0% packet loss
```

This confirms basic IP connectivity.

## 9. Verify AD DNS

Test:

```bash
nslookup dc01.harish.local
```

Expected resolution:

```text
dc01.harish.local
        ↓
192.168.182.10
```

This confirms that Kali can resolve the Domain Controller through the AD DNS server.


## 10. Verify Active Directory Service Discovery

Test the LDAP SRV record:

```bash
nslookup -type=SRV _ldap._tcp.dc._msdcs.harish.local
```

This helps verify that the DNS records required for locating Active Directory domain-controller services are available.

## 11. Important Kali Tools

The project will use different tools at different stages.

### Reconnaissance

```text
Nmap
NetExec
ldapsearch
DNS utilities
```

### Active Directory Enumeration

```text
BloodHound
SharpHound
PowerView
LDAP tools
```

### Credential Security Testing

```text
Rubeus
Impacket
Hashcat
John the Ripper
```

### Authentication/Security Research

```text
Mimikatz
Rubeus
Impacket
NetExec
```

Tools will be introduced individually when their corresponding attack or security concept is studied.

## 12. Do Not Install Everything Immediately

The lab should not be treated as:

```text
Install every hacking tool
        ↓
Run random commands
```

Instead, the learning process is:

```text
Understand the AD concept
        ↓
Understand the attack
        ↓
Understand the required tool
        ↓
Install/configure the tool
        ↓
Perform controlled lab test
        ↓
Observe the result
        ↓
Detect the activity
        ↓
Document the attack and defense
```

This prevents the project from becoming a collection of copied commands.


## 13. Snapshots

Before major security experiments, create VMware snapshots.

Recommended clean state:

```text
DC01
 └── Clean AD State

CLIENT01
 └── Clean Windows Client State

Kali
 └── Clean Attack Machine State
```

After completing a major attack demonstration, the environment can be restored if necessary.

## 14. Kali Security-Lab Principle

Kali is the **testing machine**, not the target.

The intended architecture is:

                  ATTACK MACHINE
                       Kali
                        │
                        │
                 Security Testing
                        │
                        ▼
              ┌──────────────────┐
              │   AD LAB         │
              │                  │
              │ DC01             │
              │ 192.168.182.10   │
              │                  │
              │ CLIENT01         │
              │ Windows 11       │
              └──────────────────┘

The lab must remain isolated and controlled.


# Practical Verification

Run these commands on Kali:

```bash
cat /etc/os-release
```

```bash
uname -r
```

```bash
hostname
```

```bash
ip addr
```

```bash
ip route
```

```bash
nmcli connection show --active
```

Then verify:

```bash
ping -c 4 192.168.182.10
```

and:

```bash
nslookup dc01.harish.local
```

Finally:

```bash
nslookup -type=SRV _ldap._tcp.dc._msdcs.harish.local
```

# Screenshots

Store relevant screenshots in:

```text
03-Attack-Lab-Setup/
└── Screenshots/
```

Recommended screenshots:

01-kali-version.png
02-kali-network.png
03-kali-routing.png
04-kali-dc01-connectivity.png
05-kali-ad-dns.png
06-kali-ldap-srv.png

Only screenshots actually captured during the lab should be added.

# Interview Questions

## 1. Why are you using Kali Linux in your AD project?

Kali Linux is used as the security-testing machine from which I perform controlled reconnaissance, enumeration, attack demonstrations, and security validation against my Active Directory lab.

## 2. Is Kali Linux part of Active Directory?

No. Kali is a separate Linux-based security-testing machine. It communicates with the Windows Active Directory environment over the network.

## 3. Why do you need a separate attack machine?

Separating the attack machine from the target environment makes the lab more realistic and allows security testing to be performed without modifying the normal role of the Domain Controller.

## 4. Why is DNS important for Kali in an AD lab?

Active Directory relies heavily on DNS for resolving domain names and locating domain-controller services. Therefore, correct DNS configuration is important for AD enumeration and communication.

## 5. Does Kali need a static IP?

Not necessarily. The important requirement is reliable connectivity to the Active Directory environment. A static IP can be used when a specific lab scenario requires it.

## 6. Why shouldn't you install every security tool immediately?

Because tools should be learned according to the attack or security concept being studied. Installing and running tools without understanding their purpose can lead to command memorization rather than actual understanding.

# Key Takeaways

```text
Kali = Attack/Security Testing Machine

DC01 = Domain Controller

CLIENT01 = Windows Client

Kali communicates with the AD environment over the VMware network.

DC01 provides AD DNS for the lab.

DNS is important for AD service discovery.

Security tools should be introduced according to the attack being studied.

The lab must remain isolated and authorized.
```

# Lab Status

```text
Lesson 2 — Kali Linux

Kali VM                 ✓
eth0                    ✓
VMware network          ✓
DC01 connectivity       ✓
AD DNS                  ✓
LDAP SRV discovery      ✓
Basic system verification  In Progress
```

Next lesson:

**BloodHound Setup**

