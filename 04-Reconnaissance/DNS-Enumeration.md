# DNS Enumeration

## 1. Overview

Domain Name System (DNS) translates domain names and hostnames into IP addresses. In Active Directory (AD), DNS also helps clients discover Domain Controllers and important services such as LDAP and Kerberos.

DNS enumeration is the process of querying DNS to identify hosts, service records, and other information about a target domain.

This module documents DNS reconnaissance performed against the Windows Server 2022 Domain Controller in my Active Directory lab.

## 2. Lab Environment

| Component | Configuration |
|---|---|
| Attacker Machine | Kali Linux |
| Target | Windows Server 2022 DC01 |
| Domain | `harish.local` |
| DC01 IP | `192.168.182.10` |
| CLIENT01 IP | `192.168.182.20` |
| Kali IP | `192.168.182.129` |
| DNS Server | `192.168.182.10` |
| DNS Ports Tested | TCP 53, UDP 53 |

## 3. Objectives

- Identify the DNS service on the Domain Controller.
- Resolve the AD domain and computer hostnames.
- Discover LDAP and Kerberos service locations through SRV records.
- Test DNS zone-transfer behavior.
- Test reverse DNS resolution.
- Understand the security relevance of DNS reconnaissance in Active Directory.

## 4. DNS Port Enumeration — TCP 53

### Command

```bash
nmap -p 53 -sV 192.168.182.10
```

### Actual Result

```text
PORT   STATE SERVICE VERSION
53/tcp open  domain  Simple DNS Plus
MAC Address: 00:0C:29:7C:FA:E0 (VMware)
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

### Interpretation

TCP port 53 was open, and Nmap identified the DNS service as `Simple DNS Plus`.

DNS commonly uses UDP for ordinary queries, but TCP is also used for DNS operations, including zone transfers and responses that require TCP.

**Security relevance:** An exposed DNS service provides a starting point for querying domain and service information.

Screenshot: `DNS-01-Port-Scan.png`

## 5. Domain Name Resolution

### Command

```bash
nslookup harish.local 192.168.182.10
```

### Actual Result

```text
Server:         192.168.182.10
Address:        192.168.182.10#53

Name:   harish.local
Address: 192.168.182.10
```

### Interpretation

The DNS server resolved `harish.local` to `192.168.182.10`.

In this lab, the domain name resolves to the Domain Controller's IP address.

Screenshot: `DNS-02-Domain-Resolution.png`

## 6. Domain Controller Hostname Resolution

### Command

```bash
nslookup dc01.harish.local 192.168.182.10
```

### Actual Result

```text
Server:         192.168.182.10
Address:        192.168.182.10#53

Name:   dc01.harish.local
Address: 192.168.182.10
```

### Interpretation

The DNS server correctly resolved `dc01.harish.local` to `192.168.182.10`.

This confirms that the Domain Controller's hostname can be resolved through DNS.

Screenshot: `DNS-03-DC-Resolution.png`

## 7. LDAP SRV Record Enumeration

Service (SRV) records identify hosts providing particular services and include details such as priority, weight, and port.

### Command

```bash
nslookup -type=SRV _ldap._tcp.dc._msdcs.harish.local 192.168.182.10
```

### Actual Result

```text
_ldap._tcp.dc._msdcs.harish.local
    service = 0 100 389 dc01.harish.local.
```

### Interpretation

The SRV record identified `dc01.harish.local` as an LDAP service target on port `389`.

The values are:

- `0` — priority
- `100` — weight
- `389` — service port
- `dc01.harish.local` — target hostname

This connects to the LDAP enumeration module, where the Domain Controller was queried directly.

Screenshot: `DNS-04-LDAP-SRV-Record.png`

## 8. Kerberos SRV Record Enumeration

### Command

```bash
nslookup -type=SRV _kerberos._tcp.harish.local 192.168.182.10
```

### Actual Result

```text
_kerberos._tcp.harish.local
    service = 0 100 88 dc01.harish.local.
```

### Interpretation

The record identified `dc01.harish.local` as a Kerberos service target on TCP port `88`.

Kerberos is a major authentication protocol in Active Directory. Discovering its service location helps explain how domain clients locate authentication services.

Screenshot: `DNS-05-Kerberos-SRV-Record.png`

## 9. Additional LDAP SRV Record

### Command

```bash
nslookup -type=SRV _ldap._tcp.harish.local 192.168.182.10
```

### Actual Result

```text
_ldap._tcp.harish.local
    service = 0 100 389 dc01.harish.local.
```

### Interpretation

This query also returned DC01 as an LDAP service target on port 389.

The query name differs from the earlier `_ldap._tcp.dc._msdcs` record. Both results pointed to the same LDAP service target in this lab.

Screenshot: `DNS-06-LDAP-SRV-Record.png`

## 10. DNS Zone Transfer Test — AXFR

A DNS zone transfer can copy DNS records from a DNS server. If an unauthorized client can transfer a zone, it may obtain a list of internal hostnames and other DNS records.

### Command

```bash
dig @192.168.182.10 harish.local AXFR
```

### Actual Result

```text
; <<>> DiG 9.20.29-1-Debian <<>> @192.168.182.10 harish.local AXFR
; (1 server found)
;; global options: +cmd
; Transfer failed.
```

### Interpretation

The zone-transfer request did not complete successfully.

This means the attempted AXFR did not return the zone contents. It does not, by itself, prove the reason for the failure or establish that every possible zone-transfer request is securely restricted.

Screenshot: `DNS-07-AXFR-Failed.png`

## 11. Reverse DNS Lookup

Reverse DNS attempts to identify a hostname associated with an IP address using a PTR record.

### Command

```bash
nslookup 192.168.182.10 192.168.182.10
```

### Actual Result

```text
server can't find 10.182.168.192.in-addr.arpa: NXDOMAIN
```

### Interpretation

The DNS server returned `NXDOMAIN` for the reverse lookup name.

This indicates that the requested reverse lookup name was not found. A corresponding PTR record or reverse lookup zone may not be configured for this lab address.

This result does not mean forward DNS resolution is broken; the earlier forward lookups succeeded.

Screenshot: `DNS-08-Reverse-Lookup-NXDOMAIN.png`

## 12. DNS Enumeration — UDP 53

### Command

```bash
sudo nmap -sU -p 53 -sV 192.168.182.10
```

### Actual Result

```text
PORT   STATE SERVICE VERSION
53/udp open  domain  Simple DNS Plus
MAC Address: 00:0C:29:7C:FA:E0 (VMware)
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

### Interpretation

UDP port 53 was identified as open, and Nmap identified the DNS service as `Simple DNS Plus`.

DNS uses both UDP and TCP. UDP is common for regular queries, while TCP is used for certain larger responses and operations such as zone transfers.

Screenshot: `DNS-09-UDP-Port-Scan.png`

## 13. Client Hostname Resolution

### Command

```bash
nslookup client01.harish.local 192.168.182.10
```

### Actual Result

```text
Server:         192.168.182.10
Address:        192.168.182.10#53

Name:   client01.harish.local
Address: 192.168.182.20
```

### Interpretation

The DNS server resolved `client01.harish.local` to `192.168.182.20`.

This confirms that the Windows 11 client hostname can be resolved through the lab's DNS server.

Screenshot: `DNS-10-Client-Resolution.png`

## 14. Results Summary

| Test | Result |
|---|---|
| TCP port 53 scan | Open |
| UDP port 53 scan | Open |
| `harish.local` resolution | `192.168.182.10` |
| `dc01.harish.local` resolution | `192.168.182.10` |
| LDAP SRV record | DC01, port 389 |
| Kerberos SRV record | DC01, port 88 |
| Additional LDAP SRV query | DC01, port 389 |
| AXFR zone-transfer request | Failed |
| Reverse DNS lookup | `NXDOMAIN` |
| `client01.harish.local` resolution | `192.168.182.20` |

## 15. Security Relevance

DNS enumeration can help security professionals and authorized testers identify:

- Domain Controllers and domain hostnames
- IP addresses associated with known systems
- LDAP and Kerberos service locations
- Potentially exposed internal DNS information
- Misconfigured zone transfers or missing DNS records

DNS findings should be combined with other evidence, such as LDAP enumeration, SMB enumeration, and network scans, to build a more complete view of the environment.

Enumeration alone does not establish that a host is vulnerable or that an attack is possible.

## 16. Commands Cheat Sheet

### DNS service scan — TCP

```bash
nmap -p 53 -sV 192.168.182.10
```

### DNS service scan — UDP

```bash
sudo nmap -sU -p 53 -sV 192.168.182.10
```

### Resolve a domain

```bash
nslookup harish.local 192.168.182.10
```

### Resolve the Domain Controller

```bash
nslookup dc01.harish.local 192.168.182.10
```

### Resolve the Windows client

```bash
nslookup client01.harish.local 192.168.182.10
```

### Query LDAP SRV records

```bash
nslookup -type=SRV _ldap._tcp.dc._msdcs.harish.local 192.168.182.10
```

### Query Kerberos SRV records

```bash
nslookup -type=SRV _kerberos._tcp.harish.local 192.168.182.10
```

### Test zone transfer

```bash
dig @192.168.182.10 harish.local AXFR
```

### Reverse DNS lookup

```bash
nslookup 192.168.182.10 192.168.182.10
```

## 17. Key Takeaways

1. DNS resolves hostnames to IP addresses.
2. Active Directory uses DNS to help clients locate domain services.
3. SRV records can reveal LDAP and Kerberos service locations.
4. DNS commonly uses both UDP and TCP on port 53.
5. An unsuccessful AXFR request does not reveal the full zone.
6. `NXDOMAIN` indicates that the requested DNS name was not found.
7. Successful DNS enumeration is useful reconnaissance, but it does not prove that an attack is possible.
8. Combining DNS findings with LDAP, SMB, and network enumeration gives a more complete understanding of an AD environment.

## 18. Screenshots

Place only screenshots captured from the actual lab execution in:

```text
04-Reconnaissance/
└── Screenshots/
    ├── DNS-01-Port-Scan.png
    ├── DNS-02-Domain-Resolution.png
    ├── DNS-03-DC-Resolution.png
    ├── DNS-04-LDAP-SRV-Record.png
    ├── DNS-05-Kerberos-SRV-Record.png
    ├── DNS-06-LDAP-SRV-Record.png
    ├── DNS-07-AXFR-Failed.png
    ├── DNS-08-Reverse-Lookup-NXDOMAIN.png
    ├── DNS-09-UDP-Port-Scan.png
    └── DNS-10-Client-Resolution.png
```

## 19. Lab Status

**Status: Practical tests completed; documentation prepared.**

The module covered DNS service discovery, domain and host resolution, SRV records, zone-transfer behavior, and reverse DNS lookup in the `harish.local` lab.

