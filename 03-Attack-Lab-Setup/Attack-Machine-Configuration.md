# Attack Machine Configuration

## 1. Objective

Configure Kali Linux as the attacker machine for authorized Active Directory security testing in the isolated `harish.local` lab.

## 2. Lab Environment

| Component | Configuration |
|---|---|
| Attacker Machine | Kali Linux |
| Attacker IP | `192.168.182.129/24` |
| Domain Controller | DC01 |
| Domain Controller IP | `192.168.182.10` |
| Windows Client | CLIENT01 |
| Windows Client IP | `192.168.182.20` |
| Active Directory Domain | `harish.local` |
| Network | `192.168.182.0/24` |

## 3. Network Interface Verification

Command:

```bash
ip -br addr
```

**Observed result:** The `eth0` interface was UP with IP address `192.168.182.129/24`.

Docker interfaces were also present and were not modified.

## 4. Domain Controller Connectivity

Command:

```bash
ping -c 4 192.168.182.10
```

**Observed result:** Ping to DC01 was successful.

## 5. DNS Resolution

Command:

```bash
nslookup dc01.harish.local 192.168.182.10
```

**Observed result:** `dc01.harish.local` resolved to `192.168.182.10`.

## 6. Active Directory LDAP Service Discovery

Command:

```bash
nslookup -type=SRV _ldap._tcp.dc._msdcs.harish.local 192.168.182.10
```

**Observed result:**

```text
service = 0 100 389 dc01.harish.local.
```

This confirms that the DNS server returned the LDAP service record for DC01 on port 389.

## 7. Verification Summary

| Test | Status |
|---|---|
| Kali network interface | PASS |
| DC01 network connectivity | PASS |
| DC01 DNS resolution | PASS |
| LDAP SRV record resolution | PASS |

## 8. Security Notes

- Testing is restricted to the authorized lab environment.
- Successful ping confirms network reachability, not that every service is accessible.
- DNS SRV resolution identifies the advertised LDAP service; it does not by itself verify authentication.
- Credential-attack exercises must use lab accounts and controlled testing conditions.

## 9. Screenshots

Save evidence in `03-Attack-Lab-Setup/Screenshots/`:

- `AttackMachine-01-Network-Interface.png`
- `AttackMachine-02-DC01-Ping.png`
- `AttackMachine-03-DNS-Resolution.png`
- `AttackMachine-04-LDAP-SRV-Record.png`

