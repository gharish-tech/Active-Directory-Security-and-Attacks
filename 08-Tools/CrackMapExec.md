# CrackMapExec and NetExec — Active Directory Security Lab

## 1. Objective

Understand CrackMapExec and NetExec, identify their supported protocols, and learn how security professionals use them for authorized network and Active Directory assessments.

## 2. Lab Environment

| Component | Details |
|---|---|
| Domain | `harish.local` |
| NetBIOS domain | `HARISH` |
| Domain Controller | `DC01` |
| Domain Controller IP | `192.168.182.10` |
| Kali Linux IP | `192.168.182.129` |
| SMB test share | `NTLM-lab` |
| Environment | Isolated VMware lab |

## 3. What Is CrackMapExec?

CrackMapExec (CME) is a network assessment tool used to evaluate authentication, services, and security configurations across Windows and other network environments.

It supports several protocols, including SMB, LDAP, WinRM, RDP, and MSSQL. Its functionality can help authorized assessors identify accessible services and investigate account permissions.

CrackMapExec is the original project. **NetExec is its actively maintained successor**, with additional protocol support and ongoing development.

## 4. Installation and Version Verification

The following commands were executed on Kali Linux:

```bash
which crackmapexec
crackmapexec --help
which nxc
which netexec
nxc --help
```

### Observed results

| Command | Observed result |
|---|---|
| `which crackmapexec` | `/usr/bin/crackmapexec` |
| `crackmapexec --help` | Version `5.4.0` |
| `which nxc` | `/usr/bin/nxc` |
| `which netexec` | `/usr/bin/netexec` |
| `nxc --help` | Version `1.5.1` |

The CrackMapExec output also displayed Python `SyntaxWarning` messages from installed source files. These warnings relate to invalid escape sequences in Python string literals and do not, by themselves, establish whether a particular protocol operation will succeed.

## 5. Protocols and Their Uses

| Protocol | Security-assessment purpose |
|---|---|
| SMB | Examine Windows file-sharing services and access controls. |
| LDAP | Assess directory-service exposure and configuration. |
| WinRM | Review remote-management availability and access. |
| RDP | Assess remote-desktop service exposure. |
| WMI | Investigate Windows management interfaces. |
| MSSQL | Assess Microsoft SQL Server exposure and access. |
| SSH | Assess SSH service availability and authorized access. |
| FTP | Review file-transfer service exposure. |
| NFS | Assess network file-system exposure. |
| VNC | Review remote-desktop service exposure. |

Protocol availability differs between the installed versions. For example, the supplied CrackMapExec help lists seven protocols, while the supplied NetExec help lists ten.

## 6. Important Command-Line Options

The installed NetExec help output documented these general options:

| Option | Meaning |
|---|---|
| `-t`, `--threads` | Set the number of concurrent threads. |
| `--timeout` | Set a connection or thread timeout. |
| `--jitter` | Add randomized delays between authentication attempts. |
| `--no-progress` | Disable the progress display. |
| `--log` | Export output to a specified log file. |
| `--verbose` | Enable verbose output. |
| `--debug` | Enable detailed debugging output. |
| `--dns-server` | Specify the DNS server to use. |
| `--dns-tcp` | Use TCP for DNS queries. |
| `-6` | Force IPv6 mode. |

The help output confirms that these options are available; it does not mean they were all used during the lab.

## 7. Practical Work Completed

The following checks were performed:

```bash
crackmapexec --help
nxc --help
```

**Result:** Both tools displayed their help information and version details.

No authentication scan or remote command execution is claimed in this module based on these checks alone.

For future practical exercises, use only the isolated lab systems and dedicated test accounts. Begin with low-impact service and access checks, record the exact output, and distinguish confirmed findings from assumptions.

## 8. Security Significance

These tools can help assess whether network services are exposed and whether accounts have more access than intended.

Potential security concerns include:

- Weak or improperly managed authentication.
- Excessive permissions on network services.
- Unexpected remote-management access.
- Exposed SMB shares or directory services.
- Repeated or unusual authentication attempts.

A service being reachable does not prove that authentication will succeed, and successful authentication does not automatically mean the account has administrative privileges.

## 9. Defensive Detection

Defenders can investigate suspicious activity using Windows Security logs, endpoint telemetry, and network monitoring.

Relevant Windows Security events may include:

- `4624` — successful logon.
- `4625` — failed logon.
- `4768` — Kerberos TGT request.
- `4769` — Kerberos service-ticket request.
- `5140` — network share access.
- `5145` — network-share access check.

Event availability depends on audit policy and system configuration. Correlate timestamps, source IP addresses, account names, target services, and normal administrative activity before classifying an event as malicious.

## 10. Conclusion

The installed CrackMapExec and NetExec versions were verified through their command-line help output. This established which protocol modules and general options are available in the current Kali environment.

The next stage is to conduct a controlled, low-impact assessment against the dedicated lab systems and document only the results that are actually observed.

