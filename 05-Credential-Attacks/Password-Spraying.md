# Password Spraying Lab

## 1. Objective

The objective of this lab is to understand password spraying, learn how it differs from brute-force attacks, and identify authentication failures through Windows Security logs.

## 2. What Is Password Spraying?

Password spraying is a credential attack technique in which an attacker attempts a small number of commonly used passwords across multiple user accounts instead of trying many passwords against one account.

The goal is to avoid triggering account lockout policies while attempting to gain unauthorized access.

## 3. Password Spraying vs. Brute Force

| Feature | Password Spraying | Brute Force |
|---|---|---|
| Targeting | Multiple accounts | Usually one account at a time |
| Password attempts | Few candidate passwords across accounts | Many password candidates against a target |
| Main risk | Unauthorized access across accounts | Account lockouts and password guessing |
| Detection | Failures distributed across accounts | Repeated failures against one account |

## 4. Lab Environment

- **Domain:** `harish.local`
- **Domain controller:** DC01
- **Target test account:** `spray.test`
- **Kali Linux IP:** `192.168.182.129`
- **Authentication protocol observed:** NTLM
- **Monitoring source:** Windows Security event logs

## 5. Lab Activities Performed

### Activity 1: Prepare a dedicated test account

A dedicated account named `spray.test` was created in Active Directory for controlled authentication testing.

The account was initially disabled. A lab-only password was configured, and the account was subsequently enabled.

### Activity 2: Validate a single authentication attempt

A single-account SMB authentication test was performed from Kali Linux against the domain controller using NetExec.

The test returned a successful authentication result for the dedicated account.

**Important:** This was a single-account authentication validation, not a password-spraying attack.

### Activity 3: Inspect Windows Security logs

Windows Security logs were examined on DC01 to understand how failed network logons are recorded.

The observed event contained these details:

- **Event ID:** 4625
- **Account:** `pth.test`
- **Domain:** `HARISH`
- **Logon Type:** 3 — Network
- **Authentication Package:** NTLM
- **Source IP:** `192.168.182.129`
- **Failure reason:** Unknown user name or bad password
- **Substatus:** `0x0000006A`

The substatus is consistent with an incorrect password.

This event is evidence of a failed network authentication attempt. It does not, by itself, establish that password spraying occurred.

## 6. Detection and Investigation

When investigating possible password spraying, a defender should look for:

1. Failed authentication events involving multiple accounts.
2. Repeated attempts originating from the same source IP or host.
3. Attempts distributed over a short or unusual time window.
4. Successful logons following a series of failures.
5. Account lockout events and related authentication logs.

**Relevant Windows Security events:**

- **4625:** Failed logon.
- **4624:** Successful logon.
- **4740:** A user account was locked out.

Event patterns must be correlated with timestamps, source addresses, usernames, and authentication context before classifying activity as password spraying.

## 7. Defensive Measures

- Enforce strong, unique passwords.
- Use multifactor authentication where supported.
- Monitor failed logons across multiple accounts.
- Alert on unusual authentication patterns and suspicious source addresses.
- Apply appropriate account lockout and authentication policies.
- Investigate successful logons that follow suspicious failures.

## 8. Result and Limitations

This lab established a dedicated test account, validated a single SMB authentication attempt, and examined a Windows Event ID 4625 record.

The actual password-spraying technique was **not executed**. Therefore, this module documents preparation, authentication validation, log analysis, and defensive concepts rather than a completed spraying attack.

## 9. Learning Outcome

I learned how password spraying differs from brute force, how NTLM network logons appear in Windows Security logs, and why a failed-logon event must be correlated with additional evidence before identifying an attack.


