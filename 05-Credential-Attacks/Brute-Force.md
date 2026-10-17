# Brute Force Attack — Controlled Lab Study

## 1. Objective

The objective of this lab is to understand password-based authentication against SMB services, observe successful authentication, and learn how Windows Security logs can help investigate authentication activity.

This exercise was performed in an isolated Active Directory lab using a dedicated test account.

## 2. Lab Environment

| Component | Details |
|---|---|
| Domain | `harish.local` |
| Domain Controller | DC01 |
| Target IP | `192.168.182.10` |
| Test account | `pth.test` |
| Source system | Kali Linux |
| Target service | SMB (TCP port 445) |
| Authentication tool | NetExec |
| Log source | Windows Security Event Log |

## 3. Background

A brute-force attack attempts multiple password candidates against an account or service until a valid password is found.

The number of attempts, speed of attempts, password policy, and account-lockout controls affect the risk and detectability of these attacks.

**Important:** This lab session verified authentication using a known test password. It did not perform a multi-password brute-force attack against Active Directory.

## 4. Authentication Validation

The following command was used to test SMB authentication with the dedicated lab account:

```bash
nxc smb 192.168.182.10 -d HARISH -u pth.test -p '<LAB_TEST_PASSWORD>'
```

The password placeholder is intentional. Do not store real passwords in project documentation, screenshots, or shell commands.

### Observed result

NetExec returned a successful authentication indicator (`[+]`) for `HARISH\pth.test`.

### Interpretation

This confirmed that the supplied test credentials were accepted by the SMB service on DC01.

This result demonstrates successful authentication, not password discovery through brute force.

## 5. Windows Event Log Investigation

After the authentication test, Windows Security logs were inspected on DC01.

**Event ID 4624 — An account was successfully logged on**

This event records a successful logon. To determine whether a particular event corresponds to the test, examine the event details rather than relying on the event ID alone.

Relevant fields include:

- **TargetUserName:** Should identify `pth.test`.
- **TargetDomainName:** Should identify the relevant domain.
- **LogonType:** Type `3` generally indicates a network logon.
- **IpAddress:** Should match the source address when recorded.
- **AuthenticationPackageName:** Shows the authentication package used for that event.
- **TimeCreated:** Helps correlate the event with the test time.

Not every successful logon event necessarily came from this test. Correlation requires matching the account, time, source address, and other available fields.

## 6. Brute-Force Risk and Defensive Controls

The domain's password policy was previously checked in this lab:

| Policy | Observed value |
|---|---|
| Account lockout threshold | 5 failed attempts |
| Account lockout duration | 15 minutes |
| Observation window | 15 minutes |

These controls can limit repeated password guessing, but lockout policies must be balanced against the risk of denial-of-service through deliberate account lockouts.

Additional defensive measures include:

- Use long, unique passwords and block commonly used or compromised passwords.
- Enable multifactor authentication where supported.
- Monitor repeated failed logons, particularly Event ID `4625`.
- Correlate failed attempts by account, source, timing, and target service.
- Alert on unusual authentication patterns and investigate unexpected account lockouts.
- Restrict SMB access to systems and networks that require it.

## 7. Findings

- DC01's SMB service was reachable from Kali Linux.
- The known test credentials were accepted by NetExec.
- Windows Security Event ID `4624` events were present.
- Individual events must be correlated before attributing them to the test.
- A multi-password brute-force attack was not performed in this exercise.

## 8. Conclusion

This exercise demonstrated how to validate SMB authentication with a dedicated lab account and investigate successful-logon records on a Windows Server domain controller.

It also highlighted an important distinction between testing known credentials and conducting a brute-force attack. Further brute-force testing, if required for the project, should be separately scoped and performed only in a controlled environment with safeguards against account lockout.

## 9. Evidence to Capture

Add screenshots to the repository if available:

1. NetExec output showing the successful authentication indicator. Redact any password or sensitive information.
2. Windows Security Event `4624` details matching the test account and time.
3. The relevant event fields, such as logon type and source IP, where present.

Suggested screenshot directory:

`05-Credential-Attacks/Screenshots/`

## 10. Learning Outcome

I learned how SMB authentication can be tested with known credentials, how Windows records successful logons, and why authentication events must be correlated before drawing conclusions about a security test.

