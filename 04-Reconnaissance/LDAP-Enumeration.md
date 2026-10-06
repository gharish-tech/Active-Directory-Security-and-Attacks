# LDAP Enumeration

## 1. Overview

LDAP (Lightweight Directory Access Protocol) is the protocol used to communicate with directory services such as Microsoft Active Directory.

In an Active Directory environment, LDAP can provide information about:

- Domain structure
- Users
- Groups
- Computers
- Organizational Units (OUs)
- Service Principal Names (SPNs)
- Domain configuration
- Other directory objects

From a security perspective, LDAP enumeration helps an attacker understand the structure of an Active Directory environment and identify information that may be useful during later attack stages.

This module demonstrates authenticated and unauthenticated LDAP enumeration against the lab domain.

---

## 2. Lab Environment

| Component | Configuration |
|---|---|
| Domain Controller | DC01 |
| Operating System | Windows Server 2022 |
| Domain | `harish.local` |
| Domain Controller IP | `192.168.182.10` |
| Attack Machine | Kali Linux |
| Kali IP | `192.168.182.129` |
| LDAP Protocol | LDAP |
| LDAP Port | `389` |
| LDAP User | `HARISH\harish` |

---

## 3. Objectives

The objectives of this practical are to:

1. Understand how LDAP communicates with Active Directory.
2. Query the RootDSE of a domain controller.
3. Identify Active Directory naming contexts.
4. Test anonymous LDAP access.
5. Perform authenticated LDAP enumeration.
6. Enumerate domain users.
7. Enumerate domain groups.
8. Enumerate domain computers.
9. Identify accounts containing SPNs.
10. Enumerate Organizational Units.
11. Understand how LDAP reconnaissance supports later attack techniques.

---

# 4. LDAP Enumeration Fundamentals

## 4.1 LDAP Connection

The lab Domain Controller is accessible through:

```text
ldap://192.168.182.10
```

The default LDAP TCP port is:

```text
389
```

LDAPS normally uses:

```text
636
```

This lab uses standard LDAP on port 389.

---

# 5. Understanding `ldapsearch`

The main tool used in this module is:

```bash
ldapsearch
```

`ldapsearch` is a command-line LDAP client that can query directory services.

A typical authenticated query looks like:

```bash
ldapsearch -x -H ldap://192.168.182.10 \
-D "HARISH\harish" -W \
-b "DC=harish,DC=local" \
"<LDAP_FILTER>" \
<ATTRIBUTES>
```

### Important options

| Option | Meaning |
|---|---|
| `-x` | Use simple authentication instead of SASL |
| `-H` | Specify the LDAP server URI |
| `-D` | Specify the bind identity |
| `-W` | Prompt for the password |
| `-b` | Specify the LDAP search base |
| `-s` | Specify search scope |
| Filter | Select which directory objects to return |
| Attributes | Specify which attributes to display |

---

# 6. Step 1 — Enumerating LDAP Naming Contexts

## Objective

The first practical was to query the Domain Controller's **RootDSE**.

RootDSE is a special LDAP entry that provides information about the directory server and its directory partitions.

### Command

```bash
ldapsearch -x -H ldap://192.168.182.10 \
-s base -b "" namingContexts
```

## Command Breakdown

### `ldapsearch`

Starts the LDAP query.

### `-x`

Uses simple LDAP authentication.

No username/password was supplied, so this query tests anonymous access.

### `-H ldap://192.168.182.10`

Specifies the LDAP server.

```text
ldap://192.168.182.10
```

Therefore the query is sent to DC01.

### `-s base`

Sets the search scope to the base entry only.

This is important because RootDSE is queried as a single directory entry rather than searching the entire domain.

### `-b ""`

An empty search base means the query starts at the LDAP RootDSE.

### `namingContexts`

Requests only the `namingContexts` attribute.

This attribute identifies the directory partitions available on the server.

## Actual Lab Result

The query successfully returned:

```text
namingContexts: DC=harish,DC=local
namingContexts: CN=Configuration,DC=harish,DC=local
namingContexts: CN=Schema,CN=Configuration,DC=harish,DC=local
namingContexts: DC=DomainDnsZones,DC=harish,DC=local
namingContexts: DC=ForestDnsZones,DC=harish,DC=local

result: 0 Success
numEntries: 1
```

## What Was Learned

The Domain Controller exposed five naming contexts:

```text
DC=harish,DC=local
CN=Configuration,DC=harish,DC=local
CN=Schema,CN=Configuration,DC=harish,DC=local
DC=DomainDnsZones,DC=harish,DC=local
DC=ForestDnsZones,DC=harish,DC=local
```

The most important one for normal domain enumeration is:

```text
DC=harish,DC=local
```

This is the domain naming context.

### Security Significance

RootDSE information can help an attacker understand the structure of the Active Directory environment before performing deeper enumeration.

### Screenshot

```text
LDAP-01-Naming-Contexts.png
```

---

# 7. Step 2 — Testing Anonymous LDAP Enumeration

## Objective

The next step was to determine whether an unauthenticated LDAP connection could directly read objects from the domain naming context.

### Command

```bash
ldapsearch -x -H ldap://192.168.182.10 \
-b "DC=harish,DC=local" \
-s base
```

## Command Breakdown

### `-x`

Uses simple LDAP authentication.

Because no `-D` or password was provided, the request is effectively testing anonymous access.

### `-H ldap://192.168.182.10`

Connects to DC01.

### `-b "DC=harish,DC=local"`

Sets the domain naming context as the search base.

### `-s base`

Requests only the base object itself.

This avoids performing a subtree enumeration at this stage.

## Actual Lab Result

The Domain Controller rejected the operation:

```text
result: 1 Operations error

text: 000004DC: LdapErr:
comment: In order to perform this operation a successful bind
must be completed on the connection.
```

## Interpretation

This produced an important security finding.

The server allowed the RootDSE query from Step 1, but it required authentication before allowing this domain-base query.

Therefore:

```text
Anonymous RootDSE access
        ↓
Allowed

Anonymous domain object access
        ↓
Denied
```

This demonstrates that **LDAP anonymous access is not necessarily equivalent to anonymous directory enumeration**.

### Security Significance

Requiring authentication before exposing directory objects reduces the amount of information available to an unauthenticated attacker.

### Screenshot

```text
LDAP-02-Anonymous-Bind-Denied.png
```

---

# 8. Step 3 — Authenticated Domain Enumeration

## Objective

After confirming that anonymous access was restricted, an authenticated LDAP query was performed using the normal domain user `harish`.

### Command

```bash
ldapsearch -x -H ldap://192.168.182.10 \
-D "HARISH\harish" -W \
-b "DC=harish,DC=local" \
-s base
```

## Command Breakdown

### `-D "HARISH\harish"`

Specifies the LDAP bind identity.

The account used was:

```text
HARISH\harish
```

This is the domain user used for the lab.

### `-W`

Tells `ldapsearch` to prompt for the password instead of placing the password directly in the command.

This is preferable to putting a password directly on the command line.

### `-b "DC=harish,DC=local"`

Searches the Active Directory domain naming context.

### `-s base`

Requests the domain object itself.

## Actual Lab Result

The authenticated query successfully returned the domain object:

```text
dn: DC=harish,DC=local
objectClass: top
objectClass: domain
objectClass: domainDNS
distinguishedName: DC=harish,DC=local
name: harish
dc: harish
```

Other useful attributes observed included:

```text
lockoutThreshold: 5
minPwdLength: 8
pwdHistoryLength: 24
ms-DS-MachineAccountQuota: 10
```

The result also contained references to:

- Domain Controllers
- Computers
- Users
- Managed Service Accounts
- GPO links
- FSMO/NTDS information
- DNS-related directory partitions

The query completed successfully:

```text
result: 0 Success
numEntries: 1
```

## Important Findings

### Domain

```text
harish.local
```

### Minimum Password Length

```text
8
```

### Account Lockout Threshold

```text
5
```

### Password History

```text
24
```

### Machine Account Quota

```text
10
```

### Security Significance

An authenticated low-privileged domain account can obtain a considerable amount of directory information through normal LDAP read permissions.

This demonstrates why **having valid domain credentials does not automatically mean the account is privileged**, but those credentials can still provide useful reconnaissance information.

### Screenshot

```text
LDAP-03-Authenticated-Domain-Enumeration.png
```

---

# 9. Step 4 — User Enumeration

## Objective

The next step was to enumerate user accounts in the domain.

### Command

```bash
ldapsearch -x -H ldap://192.168.182.10 \
-D "HARISH\harish" -W \
-b "DC=harish,DC=local" \
"(&(objectCategory=person)(objectClass=user))" \
sAMAccountName
```

## Understanding the LDAP Filter

The filter is:

```text
(&(objectCategory=person)(objectClass=user))
```

LDAP filters use parentheses.

The `&` operator means:

```text
AND
```

Therefore both conditions must be true:

```text
objectCategory = person
AND
objectClass = user
```

This helps select user objects rather than computers or groups.

### `sAMAccountName`

The query requests:

```text
sAMAccountName
```

This is the traditional Windows logon/account name attribute.

## Actual Lab Result

The query returned **9 user accounts**:

```text
Administrator
Guest
krbtgt
harish
priya
sai
kiran
dharma
mallesh
```

The users were located in different containers/OUs, including:

```text
CN=Users
OU=HR
OU=IT
OU=FINANCE
```

The query completed successfully:

```text
result: 0 Success
numEntries: 9
numReferences: 3
```

The three references pointed toward:

```text
ForestDnsZones
DomainDnsZones
Configuration
```

## Security Significance

User enumeration can reveal:

- Valid usernames
- Administrative accounts
- Service-related accounts
- Departmental account structure
- Potential targets for later authentication attacks

The user list itself does not mean that all accounts are vulnerable.

It is reconnaissance information that can be combined with other findings.

### Screenshot

```text
LDAP-04-User-Enumeration.png
```

---

# 10. Step 5 — Group Enumeration

## Objective

Enumerate Active Directory groups.

### Command

```bash
ldapsearch -x -H ldap://192.168.182.10 \
-D "HARISH\harish" -W \
-b "DC=harish,DC=local" \
"(&(objectCategory=group)(objectClass=group))" \
sAMAccountName
```

## LDAP Filter

```text
(&(objectCategory=group)(objectClass=group))
```

This means:

```text
objectCategory = group
AND
objectClass = group
```

The query therefore targets group objects.

## Actual Lab Result

The query returned:

```text
numEntries: 54
```

A total of **54 groups** were enumerated.

The results included built-in groups such as:

```text
Administrators
Users
Guests
Print Operators
Backup Operators
Account Operators
Server Operators
Remote Desktop Users
Remote Management Users
Network Configuration Operators
```

Domain-related groups included:

```text
Domain Admins
Domain Users
Domain Computers
Domain Controllers
Enterprise Admins
Schema Admins
Group Policy Creator Owners
Protected Users
Key Admins
Enterprise Key Admins
DnsAdmins
```

The lab also contained departmental groups:

```text
HR Users
IT Users
FINANCE Users
HR_DL_Modify
IT_DL_Modify
FINANCE_DL_Modify
```

The query completed successfully:

```text
result: 0 Success
numEntries: 54
numReferences: 3
```

## Security Significance

Group enumeration is important because group membership can determine privilege.

For example:

```text
User
  ↓
Group membership
  ↓
Permissions
  ↓
Access
```

An attacker can use group information to identify:

- Privileged groups
- Administrative groups
- Departmental groups
- Security groups
- Potential privilege relationships

However, simply discovering a privileged group does not mean the current user belongs to it.

### Screenshot

```text
LDAP-05-Group-Enumeration.png
```

---

# 11. Step 6 — Computer Enumeration

## Objective

Enumerate computers registered in Active Directory.

### Command

```bash
ldapsearch -x -H ldap://192.168.182.10 \
-D "HARISH\harish" -W \
-b "DC=harish,DC=local" \
"(&(objectCategory=computer)(objectClass=computer))" \
sAMAccountName dNSHostName operatingSystem
```

## LDAP Filter

```text
(&(objectCategory=computer)(objectClass=computer))
```

Both conditions must match:

```text
objectCategory = computer
AND
objectClass = computer
```

### Requested Attributes

The query requests:

```text
sAMAccountName
dNSHostName
operatingSystem
```

These provide:

- Computer account name
- DNS hostname
- Operating system

## Actual Lab Result

Two computer accounts were returned.

### DC01

```text
dn: CN=DC01,OU=Domain Controllers,DC=harish,DC=local
sAMAccountName: DC01$
operatingSystem: Windows Server 2022 Standard Evaluation
dNSHostName: DC01.harish.local
```

### CLIENT01

```text
dn: CN=CLIENT01,OU=WORKSTATIONS,DC=harish,DC=local
sAMAccountName: CLIENT01$
operatingSystem: Windows 11 Pro
dNSHostName: CLIENT01.harish.local
```

The query completed successfully:

```text
result: 0 Success
numEntries: 2
numReferences: 3
```

## Why Does the Computer Account End With `$`?

Active Directory computer accounts conventionally use a trailing `$` in the `sAMAccountName`.

For example:

```text
DC01$
CLIENT01$
```

This distinguishes computer accounts from ordinary user logon names.

## Security Significance

Computer enumeration can reveal:

- Domain Controllers
- Workstations
- Servers
- Operating systems
- DNS hostnames
- Potential attack targets

Knowing which system is the Domain Controller is particularly important during Active Directory reconnaissance.

### Screenshot

```text
LDAP-06-Computer-Enumeration.png
```

---

# 12. Step 7 — SPN Enumeration

## Objective

Service Principal Names (SPNs) associate services with accounts in Active Directory.

Enumerating SPNs is important because service accounts associated with SPNs can become relevant to **Kerberoasting**.

### Command

```bash
ldapsearch -x -H ldap://192.168.182.10 \
-D "HARISH\harish" -W \
-b "DC=harish,DC=local" \
"(&(objectClass=user)(servicePrincipalName=*))" \
sAMAccountName servicePrincipalName
```

## LDAP Filter

```text
(&(objectClass=user)(servicePrincipalName=*))
```

This contains two conditions.

### `objectClass=user`

Selects user-class objects.

### `servicePrincipalName=*`

The `*` means the `servicePrincipalName` attribute exists/is populated.

Together:

```text
User object
AND
has an SPN
```

### Requested Attributes

The query requests:

```text
sAMAccountName
servicePrincipalName
```

This lets us identify the account and its associated SPNs.

## Actual Lab Result

The query returned **3 accounts**:

```text
DC01$
krbtgt
CLIENT01$
```

### DC01$

The Domain Controller had multiple SPNs including services such as:

```text
LDAP
DNS
GC
HOST
RestrictedKrbHost
DFSR
RPC
```

Examples included:

```text
ldap/DC01.harish.local
DNS/DC01.harish.local
GC/DC01.harish.local/harish.local
HOST/DC01.harish.local
```

### krbtgt

The Kerberos ticket-granting account returned:

```text
kadmin/changepw
```

### CLIENT01$

The workstation returned SPNs including:

```text
HOST/CLIENT01
HOST/CLIENT01.harish.local
RestrictedKrbHost/CLIENT01
RestrictedKrbHost/CLIENT01.harish.local
```

The query completed successfully:

```text
result: 0 Success
numEntries: 3
numReferences: 3
```

## Security Significance

SPN enumeration is an important reconnaissance step because service accounts can sometimes have SPNs associated with network services.

A common attack technique related to this is:

```text
LDAP SPN Enumeration
        ↓
Identify service accounts
        ↓
Request Kerberos service tickets
        ↓
Offline password cracking
        ↓
Potential credential compromise
```

This technique is known as **Kerberoasting**.

### Important Lab Observation

The current lab results returned:

```text
DC01$
krbtgt
CLIENT01$
```

These are computer/Kerberos-related accounts rather than a dedicated user service account created specifically for a service.

Therefore, **no Kerberoasting attack was performed at this stage**.

The SPN enumeration result is being documented as reconnaissance and will be used as a foundation for the later Credential Attacks module.

### Screenshot

```text
LDAP-07-SPN-Enumeration.png
```

---

# 13. Step 8 — Organizational Unit Enumeration

## Objective

Enumerate the Organizational Units (OUs) in the domain.

OUs are used to organize Active Directory objects such as users, computers, and groups.

### Command

```bash
ldapsearch -x -H ldap://192.168.182.10 \
-D "HARISH\harish" -W \
-b "DC=harish,DC=local" \
"(&(objectClass=organizationalUnit))" \
distinguishedName
```

## LDAP Filter

```text
(&(objectClass=organizationalUnit))
```

This searches for objects whose class is:

```text
organizationalUnit
```

### `distinguishedName`

The requested attribute is:

```text
distinguishedName
```

A Distinguished Name (DN) uniquely identifies an object in the LDAP directory hierarchy.

For example:

```text
OU=HR,DC=harish,DC=local
```

means:

```text
OU=HR
   ↓
harish.local domain
```

## Actual Lab Result

The query returned **7 OUs**:

```text
OU=Domain Controllers,DC=harish,DC=local
OU=HR,DC=harish,DC=local
OU=IT,DC=harish,DC=local
OU=FINANCE,DC=harish,DC=local
OU=SERVERS,DC=harish,DC=local
OU=WORKSTATIONS,DC=harish,DC=local
OU=SECURITY GROUPS,DC=harish,DC=local
```

The query completed successfully:

```text
result: 0 Success
numEntries: 7
numReferences: 3
```

## Reconstructed Lab Structure

Based on the enumeration:

```text
harish.local
│
├── Domain Controllers
├── HR
├── IT
├── FINANCE
├── SERVERS
├── WORKSTATIONS
└── SECURITY GROUPS
```

## Security Significance

OU enumeration can reveal the organizational structure of an Active Directory environment.

For example:

```text
HR
IT
FINANCE
SERVERS
WORKSTATIONS
```

can tell an attacker how the environment is logically organized.

This information can help correlate:

```text
Users
   ↓
Groups
   ↓
OUs
   ↓
Computers
   ↓
Services
```

with other reconnaissance results.

### Screenshot

```text
LDAP-08-OU-Enumeration.png
```

---

# 14. Complete LDAP Reconnaissance Picture

The enumeration performed in this lab produced the following information:

```text
                    harish.local
                         │
          ┌──────────────┼──────────────┐
          │              │              │
        Users          Groups        Computers
        9 users        54 groups       2 systems
          │              │              │
          └──────────────┼──────────────┘
                         │
                        OUs
                       7 OUs
                         │
                        SPNs
                      3 accounts
```

The RootDSE query additionally revealed the directory naming contexts.

---

# 15. Enumeration Summary

| Enumeration | Result | Status |
|---|---:|---|
| RootDSE naming contexts | 5 | Completed |
| Anonymous domain query | Denied | Completed |
| Authenticated domain query | 1 domain object | Completed |
| Users | 9 | Completed |
| Groups | 54 | Completed |
| Computers | 2 | Completed |
| SPN accounts | 3 | Completed |
| Organizational Units | 7 | Completed |

---

# 16. Security Interpretation

The important lesson from this practical is that Active Directory exposes significant directory information to an authenticated domain user through normal LDAP read operations.

The account used in this lab was:

```text
HARISH\harish
```

The account was able to enumerate:

```text
Domain information
      ↓
Users
      ↓
Groups
      ↓
Computers
      ↓
SPNs
      ↓
Organizational Units
```

This does not mean that the account is privileged.

Instead, it demonstrates how **legitimate directory read permissions can provide useful reconnaissance information**.

An attacker who compromises a normal domain account may therefore be able to perform substantial internal reconnaissance before attempting privilege escalation or credential attacks.

---

# 17. Relationship With Later Attack Modules

LDAP enumeration provides information that can be used by later modules.

### User Enumeration

Can support:

```text
Password Spraying
Brute Force
Account Discovery
```

### Group Enumeration

Can support:

```text
Privilege Analysis
ACL Analysis
Privilege Escalation
```

### Computer Enumeration

Can support:

```text
Target Discovery
SMB Enumeration
Lateral Movement
```

### SPN Enumeration

Can support:

```text
Kerberoasting
```

### OU Enumeration

Can support:

```text
Domain Structure Mapping
Target Identification
GPO/ACL Analysis
```

The important security workflow is:

```text
Reconnaissance
      ↓
Information Collection
      ↓
Identify Interesting Objects
      ↓
Analyze Permissions / Services
      ↓
Identify Attack Surface
      ↓
Controlled Attack Testing
      ↓
Detection
      ↓
Mitigation
```

---

# 18. Defensive Considerations

Organizations should consider:

- Restricting unnecessary anonymous LDAP access.
- Protecting privileged accounts.
- Monitoring unusual LDAP enumeration activity.
- Monitoring authentication anomalies.
- Reviewing service accounts and their SPNs.
- Using strong service-account passwords.
- Applying least privilege.
- Reviewing group memberships regularly.
- Monitoring privileged group changes.
- Monitoring suspicious Kerberos activity.
- Protecting Domain Controllers and administrative accounts.

LDAP itself is a legitimate and essential Active Directory protocol, so defensive monitoring should distinguish normal directory operations from suspicious enumeration patterns.

---

# 19. Key Commands Cheat Sheet

### RootDSE / Naming Contexts

```bash
ldapsearch -x -H ldap://192.168.182.10 \
-s base -b "" namingContexts
```

### Test Anonymous Domain Access

```bash
ldapsearch -x -H ldap://192.168.182.10 \
-b "DC=harish,DC=local" \
-s base
```

### Authenticated Domain Query

```bash
ldapsearch -x -H ldap://192.168.182.10 \
-D "HARISH\harish" -W \
-b "DC=harish,DC=local" \
-s base
```

### Enumerate Users

```bash
ldapsearch -x -H ldap://192.168.182.10 \
-D "HARISH\harish" -W \
-b "DC=harish,DC=local" \
"(&(objectCategory=person)(objectClass=user))" \
sAMAccountName
```

### Enumerate Groups

```bash
ldapsearch -x -H ldap://192.168.182.10 \
-D "HARISH\harish" -W \
-b "DC=harish,DC=local" \
"(&(objectCategory=group)(objectClass=group))" \
sAMAccountName
```

### Enumerate Computers

```bash
ldapsearch -x -H ldap://192.168.182.10 \
-D "HARISH\harish" -W \
-b "DC=harish,DC=local" \
"(&(objectCategory=computer)(objectClass=computer))" \
sAMAccountName dNSHostName operatingSystem
```

### Enumerate SPNs

```bash
ldapsearch -x -H ldap://192.168.182.10 \
-D "HARISH\harish" -W \
-b "DC=harish,DC=local" \
"(&(objectClass=user)(servicePrincipalName=*))" \
sAMAccountName servicePrincipalName
```

### Enumerate OUs

```bash
ldapsearch -x -H ldap://192.168.182.10 \
-D "HARISH\harish" -W \
-b "DC=harish,DC=local" \
"(&(objectClass=organizationalUnit))" \
distinguishedName
```

---

# 20. Key Takeaways

- LDAP is a major interface for Active Directory directory operations.
- RootDSE can provide useful directory metadata without normal domain authentication.
- The lab Domain Controller denied anonymous access to the domain naming context.
- An authenticated normal domain user could enumerate significant directory information.
- LDAP filters determine which objects are returned.
- Users, groups, computers, SPNs, and OUs can all be enumerated through LDAP.
- SPN enumeration is an important precursor to understanding Kerberoasting.
- Enumeration results should be correlated rather than treated as isolated findings.
- Discovery of an object does not automatically mean that the object is vulnerable.
- Reconnaissance should lead to controlled attack testing, detection, and mitigation.

---

## Lab Status

**LDAP Enumeration: COMPLETED**

Actual enumeration performed against:

```text
DC01.harish.local
192.168.182.10
```

Using:

```text
HARISH\harish
```

The practical results documented in this file were obtained directly from the lab environment.