# AS-REP Roasting

## 1. Objective

Understand AS-REP Roasting, identify the Active Directory configuration that can make an account vulnerable, audit the lab environment, and document detection and mitigation techniques.

## 2. What Is AS-REP Roasting?

AS-REP Roasting is an Active Directory attack that targets user accounts configured without Kerberos pre-authentication.

Kerberos normally requires a client to provide pre-authentication data before the Key Distribution Center (KDC) returns an authentication response. If pre-authentication is disabled for an account, an attacker may be able to request an AS-REP response without first proving knowledge of that account's password.

Password-derived material in the response may then be used for offline password guessing.

**Attack classification:** MITRE ATT&CK T1558.004 — AS-REP Roasting.

## 3. Prerequisites

The main configuration prerequisite is an account with Kerberos pre-authentication disabled.

In Active Directory Users and Computers, this corresponds to the account option:

`Do not require Kerberos preauthentication`

- **Unchecked:** Pre-authentication is normally required.
- **Checked:** Pre-authentication is disabled for that account, making it potentially vulnerable to AS-REP Roasting.

A checked option does not automatically mean the account password can be recovered. The outcome depends on factors such as password strength and the feasibility of offline guessing.

## 4. How It Works

### Normal Configuration

1. A client begins Kerberos authentication.
2. The KDC requires pre-authentication data.
3. The client provides the required proof derived from its credentials.
4. The KDC continues the authentication process if the request is valid.

### Potentially Vulnerable Configuration

1. An account has Kerberos pre-authentication disabled.
2. An attacker may request an AS-REP response without first supplying pre-authentication proof.
3. The KDC may return an AS-REP response for that account.
4. Password-derived material from the response may be subjected to offline guessing.
5. If the password is successfully guessed, the account may be compromised.

**Conceptual flow:**

```text
Account without pre-authentication
              |
              v
       AS-REP request
              |
              v
      KDC returns response
              |
              v
   Offline password guessing
              |
       +------+------+
       |             |
   Guess fails   Password guessed
                     |
                     v
             Potential compromise
```

This diagram explains the attack concept; it does not represent a successful attack performed in this lab.

## 5. Lab Environment

| Component | Value |
|---|---|
| Domain | `harish.local` |
| Domain Controller | DC01 |
| DC01 IP | `192.168.182.10` |
| Windows Client | CLIENT01 |
| Test account | `HARISH\harish` |
| Lab type | VMware-based Active Directory lab |

## 6. Lab Configuration Audit

### Method

On DC01, open Active Directory Users and Computers and inspect each test user's **Properties → Account** tab.

Check the `Do not require Kerberos preauthentication` option without changing it.

### Observed Results

- The option was unchecked for `harish`.
- The option was unchecked for all other inspected lab user accounts.
- No AS-REP-Roasting-eligible account was identified during this manual audit.
- No account settings were changed.
- An AS-REP Roasting attack was not performed.

### Conclusion

The inspected accounts currently require Kerberos pre-authentication under the settings reviewed. This is a positive configuration finding for the specific accounts audited.

The manual check documents the accounts inspected; it is not a guarantee that every account in the domain has been audited.

## 7. Security Impact

If a vulnerable account uses a weak or guessable password, offline guessing may lead to account compromise.

Possible consequences include:

- Unauthorized access as the affected user.
- Exposure of resources accessible to that account.
- Further compromise if the account has excessive privileges or access.

Risk depends on the account's privileges, password strength, and the surrounding security controls.

## 8. Detection

Defenders can:

- Audit accounts configured with the `DONT_REQ_PREAUTH` user-account-control flag.
- Review Domain Controller Kerberos authentication logs.
- Investigate unusual AS-REQ/AS-REP activity and authentication patterns.
- Correlate relevant events with account inventory and other security telemetry.

**Windows Event ID 4768** records Kerberos Authentication Ticket (TGT) requests. Review it in context; a single event is not, by itself, proof of an attack.

## 9. Mitigation

- Require Kerberos pre-authentication for ordinary user accounts.
- Review business justifications for any exception.
- Audit account-control flags regularly.
- Enforce long, unique passwords and block commonly used passwords.
- Apply least privilege and monitor suspicious authentication activity.

## 10. AS-REP Roasting vs Kerberoasting

| Feature | AS-REP Roasting | Kerberoasting |
|---|---|---|
| Primary target | Account without Kerberos pre-authentication | Account associated with a Service Principal Name (SPN) |
| Main prerequisite | Pre-authentication disabled | An SPN-associated service account |
| Kerberos response | AS-REP | Service ticket (TGS) |
| Password-guessing approach | Offline guessing using response material | Offline guessing using service-ticket material |
| Key mitigation | Require pre-authentication | Use strong service-account passwords and managed service accounts where appropriate |

## 11. Interview Questions

**Q1. What is AS-REP Roasting?**

AS-REP Roasting targets Active Directory accounts that do not require Kerberos pre-authentication. An attacker may obtain an AS-REP response and attempt offline password guessing.

**Q2. What is Kerberos pre-authentication?**

It is a mechanism that requires the client to provide authentication proof before the KDC issues the initial authentication response.

**Q3. Does a checked checkbox guarantee that an account will be compromised?**

No. It indicates that pre-authentication is disabled. Password recovery still depends on factors such as password strength and guessing feasibility.

**Q4. How is AS-REP Roasting different from Kerberoasting?**

AS-REP Roasting targets accounts without pre-authentication, while Kerberoasting targets service accounts associated with SPNs and their Kerberos service-ticket responses.

**Q5. How can an administrator reduce this risk?**

Require pre-authentication wherever possible, audit account settings, enforce strong passwords, and monitor Kerberos activity.

## 12. Evidence

Store screenshots in `05-Credential-Attacks/Screenshots/`:

- `ASREP-01-Harish-Preauth-Setting.png`
- `ASREP-02-Test-Account-Preauth-Audit.png`

Only include screenshots that were actually captured.

## 13. Status

- [x] AS-REP Roasting concept documented
- [x] Prerequisite explained
- [x] Existing lab accounts manually inspected
- [x] Observed results documented
- [x] Detection and mitigation documented
- [ ] Practical attack demonstration — not performed
- [ ] Additional domain-wide audit — not performed


## 14. Controlled Lab Demonstration — Planned

### Objective

Observe how disabling Kerberos pre-authentication changes an account's exposure to AS-REP Roasting.

### Procedure Overview

1. Create a separate disposable test account in the isolated `harish.local` lab.
2. Record its original account settings.
3. Enable **Do not require Kerberos preauthentication** only on that test account.
4. Verify that the account's configuration reflects the intended change.
5. In an authorized lab, use an appropriate Kerberos testing tool to observe whether the account is eligible and whether an AS-REP response can be obtained.
6. Understand how password-derived response material can be used for offline password guessing.
7. Restore the account to require pre-authentication and verify the final configuration.

### Expected Learning Outcomes

- Understand the relationship between the checkbox and Kerberos pre-authentication.
- Distinguish account eligibility from successful password recovery.
- Understand the difference between obtaining a response and cracking a password.
- Practice documenting configuration changes, observations, and remediation.

### Evidence

Record screenshots and actual command outputs from the lab. Do not report a successful response retrieval or password recovery unless it was actually observed.









### Controlled Test Account Configuration

**Account:** `asrep.test`

**Distinguished Name:** `CN=asrep.test,OU=HR,DC=harish,DC=local`

**Observed LDAP attributes:**

```text
userAccountControl: 4194816
sAMAccountName: ASREP LAB TEST
```

**Verification:** The LDAP base-object query returned exactly one entry with `result: 0 Success`.

**Interpretation:** The `DONT_REQUIRE_PREAUTH` flag (`0x400000`) is set in `userAccountControl`. This confirms that Kerberos pre-authentication is disabled for this test account.

**Scope:** This confirms the account configuration only; it does not establish that a password can be recovered or that the account has been compromised.

# AS-REP Roasting — Active Directory Security Lab

## Lab Environment

| Component | Details |
|---|---|
| Domain | `harish.local` |
| Domain Controller | `DC01` |
| Domain Controller IP | `192.168.182.10` |
| Attack Machine | Kali Linux |
| Test Account | `asrep.test` |
| Enumeration Tool | Impacket `GetNPUsers` |
| Offline Audit Tool | Hashcat |
| Hashcat Mode | `18200` — Kerberos 5, etype 23, AS-REP |

## Background: Kerberos Pre-Authentication

Kerberos is an authentication protocol used by Active Directory.

During normal Kerberos authentication, a client proves knowledge of its password-derived secret before the domain controller issues the initial authentication response.

This process is called **Kerberos pre-authentication**. It helps prevent an unauthenticated party from freely requesting the type of response used in AS-REP Roasting.

If an account has the **Do not require Kerberos preauthentication** setting enabled, an attacker may be able to request an AS-REP response without first completing the normal pre-authentication step.

Depending on the encryption type and account configuration, the returned data may support offline password guessing.

## How AS-REP Roasting Works

1. An account is configured without requiring Kerberos pre-authentication.
2. The tester enumerates domain accounts to identify accounts with this configuration.
3. The tester requests an AS-REP response for an eligible account.
4. The response is saved in a supported format for an offline password audit.
5. A password-auditing tool tests candidate passwords against the captured data.
6. If a candidate matches, the password has been recovered for that account.

**Important:** Capturing an AS-REP response does not automatically mean that the password has been recovered.

## Test Account Preparation

A disposable account named `asrep.test` was created under the HR OU for this exercise.

In Active Directory Users and Computers, the account's **Account** tab was used to enable:

`Do not require Kerberos preauthentication`

The account's logon name was corrected to `asrep.test` to keep the account identity consistent during testing.

### Configuration verification

On DC01, the account's `userAccountControl` value was checked using PowerShell:

```powershell
Get-ADUser -Identity "asrep.test" -Properties UserAccountControl |
    Select-Object SamAccountName, UserAccountControl
```

Observed value:

```text
UserAccountControl: 4194816
```

The `DONT_REQUIRE_PREAUTH` flag is `0x400000`, decimal `4194304`. The observed value includes this flag, confirming that pre-authentication was disabled for the test account.

## LDAP Verification from Kali

The account configuration was verified with a base-object LDAP query:

```bash
ldapsearch -x -H ldap://192.168.182.10 \
  -D 'HARISH\harish' -W \
  -b 'CN=asrep.test,OU=HR,DC=harish,DC=local' \
  -s base '(objectClass=user)' sAMAccountName userAccountControl
```

The query returned one directory entry with:

```text
userAccountControl: 4194816
sAMAccountName: asrep.test

result: 0 Success
numEntries: 1
```

This confirmed that the account existed and that the pre-authentication-disabled flag was present.

## AS-REP Response Capture

The available Impacket command was checked first:

```bash
impacket-GetNPUsers -h
```

The help output confirmed support for domain account enumeration and the `-request` option for requesting AS-REP responses.

The test was then performed from Kali against the lab domain controller:

```bash
impacket-GetNPUsers harish.local/ \
  -dc-ip 192.168.182.10 \
  -usersfile <(printf 'asrep.test\n')
```

### Observed result

The tool returned a response beginning with:

```text
$krb5asrep$23$asrep.test@HARISH.LOCAL:
```

The complete response was saved locally for the controlled offline audit.

The prefix identifies the captured material as Kerberos AS-REP data using encryption type 23 (RC4-HMAC).

**Security note:** The full response is sensitive and is intentionally excluded from this document and the public repository.

## Offline Password Audit

Hashcat was already installed on Kali. Its example-hash information was checked:

```bash
hashcat --example-hashes |
  grep -A 8 -B 3 'Kerberos 5, etype 23'
```

The output identified the relevant mode:

```text
Hash mode #18200
Name: Kerberos 5, etype 23, AS-REP
```

The captured response was stored in a local file named `asrep-hash.txt`. A small, lab-specific wordlist was created in `lab-wordlist.txt`.

The tester reported that Hashcat recovered the password for the disposable test account.

### Result interpretation

This demonstrated the difference between obtaining an AS-REP response and recovering the password through an offline password audit.

The result applies only to the disposable account and lab environment used for this exercise. It does not demonstrate compromise of other domain accounts.

## Security Impact

An account configured without Kerberos pre-authentication can expose response data that allows password guesses to be tested offline.

Potential consequences include:

- Recovery of a weak or predictable account password.
- Unauthorized access using the recovered credentials.
- Further access if the account has excessive privileges or its password is reused.
- Increased risk when vulnerable accounts are not identified and remediated.

The practical impact depends on password strength, encryption type, account privileges, and the wider environment.

## Detection and Investigation

Administrators should review Active Directory accounts that do not require Kerberos pre-authentication.

Example PowerShell audit:

```powershell
Get-ADUser -Filter * -Properties UserAccountControl |
    Where-Object {
        $_.UserAccountControl -band 4194304
    } |
    Select-Object SamAccountName, DistinguishedName, UserAccountControl
```

This identifies user accounts with the `DONT_REQUIRE_PREAUTH` flag. Results should be reviewed with the account owner before making changes.

Security teams can also review domain controller Kerberos authentication logs, particularly Event ID 4768, and investigate unusual patterns of initial-ticket requests in context. Event availability and useful fields depend on the audit configuration and Windows version.

## Mitigation

- Require Kerberos pre-authentication unless a documented business requirement justifies an exception.
- Use long, unique passwords for user and service accounts.
- Review accounts with unusual Kerberos settings regularly.
- Apply least privilege and remove unnecessary permissions.
- Monitor Kerberos authentication activity and investigate suspicious request patterns.
- Follow the organization's change-management process when modifying account settings.

## AS-REP Roasting vs. Kerberoasting

| Feature | AS-REP Roasting | Kerberoasting |
|---|---|---|
| Main condition | Pre-authentication disabled on an account | Account has an applicable service principal name (SPN) |
| Kerberos material | AS-REP response | Service-ticket response (TGS-REP) |
| Typical target | Eligible user accounts | Accounts associated with services |
| Password audit | Offline guessing against captured response data | Offline guessing against captured service-ticket data |
| Primary mitigation | Require pre-authentication | Use strong service-account passwords and managed service accounts where appropriate |

## Interview Questions

**Q1. What is AS-REP Roasting?**

AS-REP Roasting is a Kerberos-related attack in which a tester requests AS-REP response data for an account that does not require Kerberos pre-authentication and then performs offline password guessing against the captured data.

**Q2. What does `DONT_REQUIRE_PREAUTH` mean?**

It is a User Account Control flag indicating that Kerberos pre-authentication is not required for that account.

**Q3. Does obtaining an AS-REP response mean the password is cracked?**

No. It means the response was obtained. Password recovery is a separate offline-auditing result.

**Q4. Why is offline guessing a concern?**

Candidate passwords can be tested against the captured data without sending every guess to the domain controller, reducing the visibility that online guessing would normally generate.

**Q5. How does AS-REP Roasting differ from Kerberoasting?**

AS-REP Roasting targets accounts that do not require pre-authentication. Kerberoasting targets service tickets associated with service accounts that have SPNs.

**Q6. What is the main mitigation?**

Require Kerberos pre-authentication for accounts unless a documented exception is necessary, and use strong, unique passwords.

## Evidence and Screenshots

Store screenshots in the existing AS-REP Roasting screenshots folder.

Suggested evidence:

1. `ASREP-01-Test-Account-Configuration.png` — test account setting.
2. `ASREP-02-UserAccountControl.png` — PowerShell flag verification.
3. `ASREP-03-LDAP-Verification.png` — LDAP query and successful result.
4. `ASREP-04-Impacket-Response.png` — terminal output with the response redacted.
5. `ASREP-05-Hashcat-Mode.png` — Hashcat mode identification.
6. `ASREP-06-Offline-Audit-Result.png` — recovery result with the recovered password redacted.

Do not commit the complete AS-REP response, recovered password, or a wordlist containing sensitive or reused passwords.

## Final Outcome

The lab demonstrated account-configuration verification, LDAP validation, AS-REP response capture using Impacket, identification of the corresponding Hashcat mode, and reported offline password recovery for a disposable test account.

**Status:** Completed in a controlled lab.

The results show why accounts that do not require Kerberos pre-authentication should be reviewed and why password strength remains important even when an attack does not involve repeated online login attempts.
