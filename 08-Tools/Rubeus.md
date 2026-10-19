# Rubeus — Active Directory Kerberos Lab

## 1. Objective

Understand Rubeus, its role in Windows Kerberos assessment, and how defenders investigate suspicious Kerberos activity.

## 2. Overview

Rubeus is a C# toolset for interacting with the Windows Kerberos authentication system. It is part of the GhostPack project.

Official sources:

- Source: https://github.com/GhostPack/Rubeus
- Documentation: https://specterops.gitbook.io/ghostpack/rubeus/introduction

## 3. Lab Environment

| Component | Details |
|---|---|
| Domain | `harish.local` |
| Domain Controller | `DC01` |
| Domain Controller IP | `192.168.182.10` |
| Windows client | `CLIENT01` |
| Kali Linux IP | `192.168.182.129` |
| Environment | Isolated VMware lab |

## 4. Why Rubeus Matters

Rubeus helps security practitioners study Kerberos behavior on Windows systems. Its functionality includes requesting and inspecting Kerberos tickets, as well as other advanced ticket operations.

Kerberos tickets are sensitive authentication material. Access to a ticket does not automatically grant administrator privileges; actual access depends on the principal, ticket validity, service, and authorization policy.

## 5. Installation and Build Status

The official GhostPack repository provides source code and build instructions rather than an official precompiled executable.

Record the actual installation progress below:

- [ ] Official source downloaded.
- [ ] Visual Studio and required .NET targeting components checked.
- [ ] Release build completed.
- [ ] Build output and version information recorded.
- [ ] Screenshot of successful build added to the repository.

Do not mark these items complete until the corresponding result has been observed.

## 6. Defensive Investigation

Useful Windows Security events include:

| Event ID | Meaning |
|---|---|
| `4768` | Kerberos TGT request |
| `4769` | Kerberos service-ticket request |
| `4771` | Kerberos pre-authentication failure |
| `4624` | Successful logon |
| `4625` | Failed logon |

Event availability depends on the audit configuration. Investigate events by correlating account, source address, target service, timestamps, and normal activity. A single event does not establish that an attack occurred.

## 7. Lab Scope

This module focuses on source verification, compilation, understanding Kerberos, and defensive monitoring. No ticket theft, ticket forging, credential extraction, or unauthorized impersonation is claimed.

## 8. Conclusion

Rubeus provides a practical way to study Windows Kerberos behavior. Building it from the official source establishes a reproducible starting point for controlled lab work and security monitoring.
