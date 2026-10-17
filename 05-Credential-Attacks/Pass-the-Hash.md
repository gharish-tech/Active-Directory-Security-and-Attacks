# Pass-the-Hash (PtH) — Active Directory Security Lab

## 1. Project Overview

**Attack Name:** Pass-the-Hash (PtH)  
**Category:** Credential-based authentication attack  
**Target:** Windows Server 2022 — DC01  
**Domain:** `harish.local`  
**Target IP:** `192.168.182.10`  
**Testing Machine:** Kali Linux  
**Target Service:** SMB (TCP port 445)  
**Test Account:** `HARISH\pth.test`  
**Status:** NTLM hash authentication and SMB share enumeration tested successfully

### Objective

The objective of this lab was to understand how Pass-the-Hash works in a Windows Active Directory environment, observe NTLM hash-based authentication over SMB, and identify the security risks associated with stolen password hashes.

The experiment used a dedicated lab account rather than an administrator account. The objective was to validate authentication behavior and document the account's observed access, not to perform privilege escalation.

---

## 2. What Is Pass-the-Hash?

Pass-the-Hash is an authentication attack in which an attacker uses a user's NTLM password hash to authenticate to a service without knowing the user's plaintext password.

In a normal password-based login, the user supplies a password. Windows authentication mechanisms process the password according to the relevant authentication protocol.

In a Pass-the-Hash attack, an attacker who has obtained a usable NTLM hash may pass that hash to a compatible authentication tool. The target may accept the authentication if the account, protocol, and security policies permit it.

**Important:** An NTLM hash is not the same as an encrypted password. NTLM hashes are password-derived values used in the NTLM authentication ecosystem. Possession of a hash can create a serious security risk, even when the plaintext password is unknown.

### Normal authentication versus Pass-the-Hash

| Normal password authentication | Pass-the-Hash |
|---|---|
| Uses the account password as the supplied credential | Uses an NTLM hash as the supplied credential |
| The client performs password-based authentication | A compatible client performs hash-based NTLM authentication |
| The target verifies the authentication exchange | The target verifies the NTLM authentication exchange |
| Success depends on valid credentials and policy | Success depends on a usable hash, account state, protocol, and policy |

---

## 3. How NTLM Authentication Works

NTLM is a Windows authentication protocol that uses a challenge-response mechanism.

A simplified authentication flow is:

1. The client requests authentication to a service.
2. The server sends a challenge.
3. The client calculates a response using the challenge and its credential material.
4. The server or domain authentication infrastructure validates the response.
5. The service accepts or rejects the authentication.

In a Pass-the-Hash scenario, the attacker does not need to recover the original plaintext password if the available hash can be used by the authentication client.

This does not mean that every Windows service will accept the hash. The authentication protocol, account permissions, domain policies, and target configuration all affect the outcome.

---

## 4. Lab Environment

| Component | Configuration |
|---|---|
| Domain Controller | DC01 |
| Operating system | Windows Server 2022 |
| Domain | `harish.local` |
| NetBIOS domain | `HARISH` |
| DC01 IP address | `192.168.182.10` |
| Client testing platform | Kali Linux |
| Authentication service | SMB |
| SMB port | TCP 445 |
| Dedicated test account | `pth.test` |

The test account was created specifically for this experiment. It was enabled, and a lab-only password was configured.

No claim is made that the account was a domain administrator or had administrative access to DC01.

---

## 5. Lab Preparation

### 5.1 Create the test account

A dedicated domain account named `pth.test` was created in Active Directory.

Its initial state was disabled. A password was configured before the account was enabled.

The account's enabled state and password configuration were verified in Active Directory.

### 5.2 Verify normal authentication

The test account was used to establish a normal Windows authentication session.

The observed identity was:

`HARISH\pth.test`

This established that the account could authenticate normally. It did not, by itself, demonstrate Pass-the-Hash.

### 5.3 Verify network connectivity

The Kali machine could reach DC01 on SMB port 445.

The following test was used:

```bash
nxc smb 192.168.182.10
```

The output identified DC01 as Windows Server 2022 and reported SMB signing enabled.

This confirmed SMB service reachability. Network connectivity alone does not prove successful authentication.

---

## 6. NTLM Hash Authentication Test

### 6.1 Generate a local test hash

The test account's lab password was used locally with Impacket's NTLM utility to calculate its NTLM hash.

The hash was not intended to be committed to the GitHub repository.

**Security note:** The lab password was exposed during troubleshooting. It should be changed after the experiment, and it must not be reused for other accounts.

### 6.2 Perform hash-based SMB authentication

The following command was used with the locally generated test hash:

```bash
nxc smb 192.168.182.10 -d HARISH -u pth.test -H '<REDACTED_NTLM_HASH>'
```

The output included a successful authentication indicator for `HARISH\pth.test`.

This is the central result of the experiment: the tool authenticated to the SMB service using the supplied NTLM hash rather than the plaintext password.

**Observed result:** NTLM hash-based SMB authentication succeeded against DC01.

**Scope limitation:** This proves authentication using the supplied hash in this lab test. It does not prove that the account obtained administrative privileges, accessed protected files, or executed commands on the domain controller.

---

## 7. SMB Share Enumeration

After authentication succeeded, SMB share enumeration was performed using the same dedicated test account.

The command was:

```bash
nxc smb 192.168.182.10 -d HARISH -u pth.test -H '<REDACTED_NTLM_HASH>' --shares
```

### Observed shares

| Share | Observed output | Interpretation |
|---|---|---|
| `ADMIN$` | Permission field blank | Access was not established by this output |
| `C$` | Permission field blank | Access was not established by this output |
| `FINANCE` | Permission field blank | Share was listed; access level remains unverified |
| `HR` | Permission field blank | Share was listed; access level remains unverified |
| `IPC$` | `READ` | Tool reported READ access |
| `IT` | Permission field blank | Share was listed; access level remains unverified |
| `NETLOGON` | `READ` | Tool reported READ access |
| `NTLM-lab` | `READ` | Tool reported READ access |
| `SYSVOL` | `READ` | Tool reported READ access |
| `Wallpapers` | `READ` | Tool reported READ access |

The share enumeration result demonstrated that the authenticated account could query the list of SMB shares.

A listed share is not automatically proof that all files within it are readable. Share permissions and NTFS permissions both affect access.

### Attempted share spidering

A further attempt was made to enumerate content in `NTLM-lab`. The tool reported that it had started spidering the share, but no conclusive file-listing result was obtained.

Therefore, file-level access through this operation is **inconclusive** and is not recorded as a successful file-read test.

---

## 8. Results and Evidence

### Summary of observed results

| Test | Result | Evidence |
|---|---|---|
| Dedicated account created | Success | `pth.test` was present in Active Directory |
| Account enabled | Success | `Enabled` property verified |
| Normal domain authentication | Success | Windows session used `HARISH\pth.test` |
| DC01 SMB reachability | Success | TCP port 445 reachable |
| NTLM hash-based SMB authentication | Success | NetExec reported successful authentication |
| SMB share enumeration | Success | Share list returned |
| File-level spidering of `NTLM-lab` | Inconclusive | Tool remained at `Spidering .` |

### Overall conclusion

The lab demonstrated that a usable NTLM hash can authenticate to an SMB service without supplying the plaintext password to the authentication tool.

The experiment also showed that successful authentication and authorization are separate concepts. Authentication identified the account, while the account's actual permissions determined which resources it could access.

The experiment did not establish administrative access to DC01, successful access to protected shares, or remote command execution.

---

## 9. Security Impact

Pass-the-Hash can allow an attacker to reuse stolen NTLM credential material to authenticate as another account.

The impact depends on the privileges assigned to that account and the services that accept NTLM authentication.

Potential consequences include:

- Unauthorized access to network shares.
- Access to sensitive organizational data.
- Lateral movement between systems where the account has access.
- Abuse of privileged accounts if their credential material is compromised.
- Increased exposure when the same credentials or privileges are reused across multiple systems.

A successful authentication event should not automatically be described as privilege escalation. The attacker's effective access is bounded by the permissions of the authenticated account and any additional weaknesses present in the environment.

---

## 10. Detection and Monitoring

Defenders should monitor Windows authentication activity and investigate unexpected NTLM use.

Useful sources of evidence include:

- **Security Event ID 4624:** Successful logon.
- **Security Event ID 4625:** Failed logon.
- **Security Event ID 4776:** NTLM credential validation.
- SMB access and file-server audit logs.
- Endpoint telemetry showing unusual use of administrative tools or remote management services.

Event availability and detail depend on the audit policy and the Windows component that records the event.

### Detection ideas

1. Identify unusual NTLM authentication between workstations and servers.
2. Monitor accounts authenticating from unexpected source systems.
3. Investigate repeated authentication attempts and unusual access patterns.
4. Correlate successful logons with subsequent SMB share access.
5. Alert on unexpected use of privileged accounts across multiple machines.

A single successful NTLM logon is not sufficient to conclude that an attack occurred. Detection should consider account, source host, target service, timing, and surrounding activity.

---

## 11. Mitigation and Prevention

### 11.1 Reduce NTLM exposure

Audit NTLM use before restricting it. Introduce restrictions through tested Group Policy settings and verify that legitimate applications continue to work.

### 11.2 Protect privileged accounts

- Avoid using domain administrator accounts for routine workstation activity.
- Apply least privilege.
- Use dedicated administrative accounts.
- Restrict where privileged accounts may log on.
- Use separate accounts for standard and administrative tasks.

### 11.3 Reduce credential exposure

- Apply Windows security updates.
- Enable appropriate credential protections where supported.
- Limit local administrator privileges.
- Avoid reusing passwords across accounts and systems.
- Use Windows LAPS where appropriate to manage unique local administrator passwords.

### 11.4 Secure SMB

- Keep SMBv1 disabled.
- Require SMB signing where operationally appropriate.
- Restrict SMB access through host firewalls and network segmentation.
- Apply both share-level and NTFS permissions according to least privilege.

**Important:** SMB signing helps protect SMB message integrity and can mitigate certain relay attacks. It does not, by itself, prevent all Pass-the-Hash authentication.

### 11.5 Respond to suspected compromise

If a real account hash is suspected to be compromised, investigate the affected account and hosts, rotate the relevant credentials, revoke or invalidate active sessions where appropriate, and review authentication logs for related activity.

---

## 12. Limitations

- This was a controlled, authorized lab experiment.
- The test used a dedicated account, not a domain administrator account.
- The experiment confirmed NTLM hash-based SMB authentication.
- The output did not demonstrate remote command execution.
- The output did not establish administrative access to DC01.
- The `NTLM-lab` spidering attempt did not produce conclusive file-level evidence.
- The test does not establish that every Windows service or authentication configuration is vulnerable to the same behavior.

These limitations are important because the purpose of a security report is to document what was actually observed rather than infer additional impact.

---

## 13. Interview Questions and Answers

### Q1. What is Pass-the-Hash?

Pass-the-Hash is an attack in which an attacker uses a stolen NTLM password hash to authenticate to a service without knowing the plaintext password.

### Q2. Does a successful PtH login mean the attacker is an administrator?

No. The authentication identifies the account. The permissions assigned to that account determine its effective access.

### Q3. What is the difference between authentication and authorization?

Authentication verifies an identity. Authorization determines what that identity is allowed to do.

### Q4. What is NTLM?

NTLM is a Windows authentication protocol that uses a challenge-response mechanism.

### Q5. Does SMB signing prevent Pass-the-Hash?

Not by itself. SMB signing provides message integrity and helps mitigate certain relay attacks. It should not be treated as a universal defense against PtH.

### Q6. How can defenders detect suspicious NTLM activity?

They can correlate Windows logon events, NTLM credential-validation events, source and destination systems, account privileges, and SMB access activity.

### Q7. What did this lab prove?

It demonstrated successful NTLM hash-based authentication to DC01's SMB service using the dedicated `pth.test` account, followed by successful SMB share enumeration.

### Q8. What did the lab not prove?

It did not prove domain administrator access, privilege escalation, remote command execution, or successful reading of files in every listed share.

---

## 14. Final Learning Outcome

This experiment improved my understanding of:

- NTLM authentication and password hashes.
- The distinction between plaintext-password authentication and hash-based authentication.
- Using NetExec to test SMB authentication in an authorized lab.
- Enumerating SMB shares and interpreting permission output carefully.
- The difference between authentication success and administrative access.
- Detection opportunities and defensive controls for credential-based attacks.

**Project status:** Pass-the-Hash authentication and SMB share enumeration demonstrated in the lab. File-level access to `NTLM-lab` remains unverified.

