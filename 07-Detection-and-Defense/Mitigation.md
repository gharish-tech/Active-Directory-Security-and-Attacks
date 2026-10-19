# Mitigation — Active Directory Security Lab

## 1. Objective

The objective of this module is to identify practical security controls that reduce risks related to excessive privileges, SMB permissions, Kerberos authentication, Group Policy changes, and compromised accounts in an Active Directory environment.

Mitigation recommendations are based on the lab configuration and the checks performed. Items not yet verified are identified as recommendations rather than completed changes.

## 2. Lab Environment

| Component | Details |
|---|---|
| Domain | `harish.local` |
| Domain Controller | `DC01` |
| Domain Controller IP | `192.168.182.10` |
| Windows Client | `CLIENT01` |
| Test account | `HARISH\acl.operator` |
| Test SMB share | `NTLM-lab` |
| SMB share path | `C:\NTLM-Lab` |

## 3. Privileged Group Membership Review

The following PowerShell commands were executed on the domain controller:

```powershell
Import-Module ActiveDirectory

Get-ADGroupMember "Domain Admins" |
    Select-Object Name, SamAccountName, ObjectClass

Get-ADGroupMember "Enterprise Admins" |
    Select-Object Name, SamAccountName, ObjectClass
```

### Observed results

Both commands returned the `Administrator` account as a direct group member.

| Group | Observed direct member |
|---|---|
| Domain Admins | `Administrator` |
| Enterprise Admins | `Administrator` |

The output did not list `acl.operator` as a direct member of either group.

### Security significance

Domain Admins and Enterprise Admins are highly privileged groups. Membership should be restricted to accounts that genuinely require those permissions. The observed output describes direct membership only; nested group membership and other privilege paths require separate assessment.

## 4. Mitigation Controls

### 4.1 Apply least privilege

- Keep privileged group membership limited to authorized administrative accounts.
- Review group membership periodically and after staff or role changes.
- Avoid using highly privileged accounts for routine browsing, email, and ordinary workstation activity.
- Use separate standard and administrative accounts where appropriate.
- Verify nested group membership before concluding that an account has no privileged access.

**Verification:** Review privileged group membership using Active Directory PowerShell and document approved members.

### 4.2 Secure SMB shares and NTFS permissions

SMB access is controlled by both share permissions and NTFS permissions. Effective access depends on the combined permissions.

- Grant access only to the users and groups that require it.
- Avoid unnecessarily broad write permissions.
- Separate read-only access from modification access.
- Review both share-level and filesystem-level permissions.
- Remove obsolete permissions only after confirming that they are not required.

Read-only inspection commands:

```powershell
Get-SmbShare -Name "NTLM-lab" |
    Select-Object Name, Path, Description

Get-SmbShareAccess -Name "NTLM-lab" |
    Format-Table Name, AccountName, AccessControlType, AccessRight -AutoSize

Get-Acl "C:\NTLM-Lab" |
    Format-List Owner, AccessToString
```

These commands inspect the current configuration; they do not change permissions.

### 4.3 Protect Kerberos credentials

- Protect domain controllers and endpoints where Kerberos credentials may be cached.
- Limit administrative logons to appropriately secured systems.
- Avoid sharing credential-cache files or leaving them in publicly accessible directories.
- Monitor unusual authentication patterns and investigate suspected credential compromise.
- Use supported incident-response procedures to invalidate compromised credentials or sessions.

A valid Kerberos ticket does not automatically grant administrator access. The principal's permissions and the target service's authorization rules still apply.

### 4.4 Restrict Group Policy administration

- Delegate GPO editing only to approved administrators.
- Separate the ability to edit a GPO from the ability to link it where appropriate.
- Review permissions on GPOs and their linked OUs.
- Monitor changes to GPO settings and links.
- Test changes against a dedicated lab OU before wider deployment.

In this lab, `acl.operator` was previously observed to have delegated `GpoEdit` rights on `GPO-Abuse-Lab-Test`. That permission should be retained only if it is required for the lab objective. No privilege escalation was demonstrated.

### 4.5 Enable useful auditing

- Configure appropriate Windows Security auditing for account logons, Kerberos activity, directory-service changes, policy changes, and file-share access.
- Use Sysmon where additional endpoint process and network telemetry is useful.
- Centralize logs when the environment supports it.
- Ensure system clocks are synchronized so events can be correlated reliably.
- Review events in context instead of treating a single event as proof of malicious activity.

Audit settings and Sysmon deployment have not been confirmed by the privileged-group membership output alone; they must be verified separately.

### 4.6 Protect administrative accounts

- Use strong, unique authentication credentials.
- Restrict interactive and network logons for privileged accounts where operationally appropriate.
- Keep administrative credentials out of scripts and command history.
- Review account activity and disable accounts that are no longer required, following an approved change process.
- Do not disable or remove the built-in Administrator account without a tested recovery plan and a clear operational reason.

## 5. Verification Plan

| Control | Verification method | Status |
|---|---|---|
| Domain Admins membership | `Get-ADGroupMember "Domain Admins"` | Checked |
| Enterprise Admins membership | `Get-ADGroupMember "Enterprise Admins"` | Checked |
| SMB share permissions | `Get-SmbShareAccess` | Requires recorded output |
| NTFS permissions | `Get-Acl` | Requires recorded output |
| GPO delegation | Review GPO security/delegation | Previously assessed |
| Kerberos and SMB auditing | Review audit policy and event logs | Separate verification required |
| Sysmon telemetry | Check installation, service, and configuration | Separate verification required |

## 6. Key Findings

1. The observed direct membership output lists `Administrator` in both Domain Admins and Enterprise Admins.
2. The output does not list `acl.operator` as a direct member of either group.
3. Direct-membership checks do not rule out nested membership or other delegated privilege paths.
4. SMB share and NTFS permissions must be assessed together.
5. Kerberos, GPO, and endpoint logs provide complementary evidence for security investigations.
6. Mitigation should follow least privilege, controlled change management, and evidence-based verification.

## 7. Conclusion

This module documents practical Active Directory mitigation controls and records the observed privileged-group membership results. It establishes a baseline for reviewing administrative access and identifies additional checks needed for SMB permissions, Group Policy delegation, and security telemetry.

Only the checks explicitly recorded above are marked as completed. Recommended controls are not described as implemented until their configuration has been verified.
