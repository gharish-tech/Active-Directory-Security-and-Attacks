# Mimikatz — Active Directory Security Lab

## 1. Objective

Understand Mimikatz, its role in Windows authentication security, how to obtain its source from the official project, and how defenders detect credential-access attempts.

## 2. Overview

Mimikatz is a Windows security research tool developed by Benjamin Delpy. It explores Windows authentication mechanisms and can interact with sensitive credential material.

Official resources:

- Source repository: https://github.com/gentilkiwi/mimikatz
- Project documentation: https://github.com/gentilkiwi/mimikatz/wiki
- Build configuration reference: https://deepwiki.com/gentilkiwi/mimikatz/6.1-build-configuration

## 3. Lab Environment

| Component | Details |
|---|---|
| Domain | `harish.local` |
| Domain Controller | `DC01` |
| DC01 IP | `192.168.182.10` |
| Windows client | `CLIENT01` |
| Environment | Isolated VMware lab |
| Intended source directory | `C:\Lab\Mimikatz` |

## 4. Security Concepts

Mimikatz is relevant to the study of:

- Windows authentication and credential handling.
- LSASS, the Local Security Authority Subsystem Service.
- Kerberos authentication and ticket protection.
- Windows security controls and credential isolation.
- Detection of suspicious access to authentication processes.

Credential material can be highly sensitive. A successful authentication or the presence of a Kerberos ticket does not automatically imply administrator privileges.

## 5. Source Acquisition and Build

The source should be obtained from the official `gentilkiwi/mimikatz` repository.

The project uses a Visual Studio solution and requires suitable C++ development components and Windows SDK support.

Progress checklist:

- [ ] Official repository accessed.
- [ ] Source downloaded or cloned.
- [ ] Source extracted to `C:\Lab\Mimikatz`.
- [ ] Visual Studio prerequisites checked.
- [ ] Build result recorded, if compiled.
- [ ] Screenshot of the source or build result added.

Only mark a step complete after observing the corresponding result.

## 6. Defensive Monitoring

Potential sources of evidence include:

- Microsoft Defender alerts and protection history.
- Windows Security logs.
- Sysmon process-creation and process-access events, when configured.
- EDR alerts involving LSASS access or suspicious credential-access behavior.

Useful event references include:

| Event | Purpose |
|---|---|
| Security `4624` | Successful logon |
| Security `4625` | Failed logon |
| Security `4768` | Kerberos TGT request |
| Security `4769` | Kerberos service-ticket request |
| Sysmon `1` | Process creation |
| Sysmon `10` | Process access, if configured |

These events provide investigative context; none alone proves that Mimikatz was executed. Sysmon event availability depends on the installed version and configuration.

## 7. Scope and Limitations

This module documents the tool, source acquisition, build process, and defensive relevance.

It does not claim that Mimikatz has been installed, compiled, or executed until evidence confirms those steps. No credential extraction or unauthorized access is claimed.

## 8. Conclusion

Mimikatz is relevant to Active Directory security because it illustrates the risks associated with credential exposure and weaknesses in endpoint protection. Trusted-source handling, least privilege, credential isolation, and effective monitoring are important defensive measures.
