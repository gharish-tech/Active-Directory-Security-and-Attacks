# BloodHound Reconnaissance

## 1. Overview

BloodHound Community Edition (CE) is used in this lab to visualize relationships and security permissions inside the Active Directory environment.

Instead of manually checking every user, group, computer, and permission, BloodHound helps identify relationships that may contribute to privilege escalation or lateral movement.

### Lab Environment

| Component | Value |
|---|---|
| Domain | `harish.local` |
| Domain Controller | `DC01.harish.local` |
| Domain Controller IP | `192.168.182.10` |
| Client | `CLIENT01.harish.local` |
| Attack Machine | Kali Linux |
| Kali IP | `192.168.182.129` |
| BloodHound CE | 9.7.1 |
| SharpHound | 2.16.0 |

---

# 2. BloodHound Architecture

The lab uses BloodHound CE with Docker.

```text
                    Active Directory
                         │
                         │ LDAP / AD queries
                         ▼
                  ┌──────────────┐
                  │   CLIENT01   │
                  │ Windows 11   │
                  └──────┬───────┘
                         │
                    SharpHound
                         │
                    Collection
                         │
                         ▼
              20261004071003_BloodHound.zip
                         │
                         │ SCP
                         ▼
                  ┌──────────────┐
                  │     Kali     │
                  │ BloodHound   │
                  │     CE       │
                  └──────┬───────┘
                         │
                         ▼
                 BloodHound Analysis
```

---

# 3. Opening BloodHound

BloodHound CE was deployed using the official BloodHound CLI and Docker Compose.

The BloodHound web interface was opened locally:

```text
http://127.0.0.1:8080
```

The BloodHound CE interface displayed:

- Explore
- Privilege Zones
- Quick Upload
- Profile
- Download Collectors
- Administration
- API Explorer
- Search
- Pathfinding
- Cypher

### Screenshot

Add the actual BloodHound CE interface screenshot here.

```text
03-Attack-Lab-Setup/Screenshots/
```

---

# 4. SharpHound Data Collection

SharpHound was executed from the Windows 11 CLIENT01 machine.

Command used:

```powershell
.\SharpHound.exe -c All
```

The collector resolved the current domain as:

```text
harish.local
```

The enumeration completed successfully.

The generated BloodHound data file was:

```text
20261004071003_BloodHound.zip
```

The ZIP file was transferred from CLIENT01 to Kali using SCP.

```powershell
scp .\20261004071003_BloodHound.zip kali@192.168.182.129:/home/kali/
```

The ZIP was then uploaded into BloodHound CE using **Quick Upload**.

The ZIP was kept intact and was not manually extracted.

---

# 5. Understanding BloodHound Objects

BloodHound represents Active Directory objects as nodes.

Examples observed in this lab:

- Users
- Groups
- Computers
- Domain objects
- GPO-related objects

Relationships between these nodes represent permissions, memberships, administration rights, and other security relationships.

Important relationship concepts observed during reconnaissance:

- GenericAll
- WriteOwner
- AddKeyCredentialLink
- AllExtendedRights
- Local Admin Privileges
- Member Of
- Inbound Object Control
- Outbound Object Control

---

# 6. Domain Admins Reconnaissance

The following object was searched in BloodHound:

```text
Domain Admins@harish.local
```

The object information showed fields such as:

- Node Type
- Object ID
- ACL Inheritance Denied
- Admin Count
- AdminSDHolder Protected
- Created
- Description
- Domain FQDN

## 6.1 Members

BloodHound showed:

```text
Members: 1
```

The member was:

```text
ADMINISTRATOR@HARISH.LOCAL
```

This confirms that the Administrator account is a member of Domain Admins in this lab.

## 6.2 Member Of

Domain Admins was shown as a member of:

```text
Denied RODC Password Replication Group
Administrators
```

## 6.3 Local Admin Privileges

BloodHound showed:

```text
Local Admin Privileges: 2
```

The computers were:

```text
CLIENT01.HARISH.LOCAL
DC01.HARISH.LOCAL
```

This is an important observation because membership in Domain Admins results in administrative privileges on these systems in this lab.

## 6.4 Object Control

BloodHound displayed:

```text
Inbound Object Control: 3
```

and approximately:

```text
Outbound Object Control: 282
```

The outbound objects included examples such as:

- IT
- Print Operators
- Guest
- Administrator
- DC01
- GPO objects
- Mallesh
- Priya
- Harish
- Domain Controllers
- Finance

### Important Note

A large Outbound Object Control number does **not** automatically mean there are 282 direct attack paths.

These are relationships/permissions represented in the BloodHound graph.

---

# 7. Administrators Group Reconnaissance

The following object was searched:

```text
Administrators@HARISH.LOCAL
```

BloodHound showed inbound object-control relationships from:

```text
Enterprise Admins
Domain Admins
Administrator
```

The Administrators group also showed a large number of outbound object-control relationships.

Examples observed included:

- Users
- Computers
- HR Users
- Finance
- CLIENT01
- Network Configuration Operators
- IT_DL_Modify
- Key Admin
- System

This demonstrates how privileged groups can have many relationships throughout an Active Directory environment.

---

# 8. CLIENT01 Reconnaissance

The following computer object was examined:

```text
CLIENT01.HARISH.LOCAL
```

BloodHound showed several inbound relationships.

## 8.1 Domain Admins → CLIENT01

Observed relationship:

```text
DOMAIN ADMINS@HARISH.LOCAL
        │
        │ GenericAll
        ▼
CLIENT01.HARISH.LOCAL
```

`GenericAll` represents broad control over the target object.

---

## 8.2 Enterprise Admins → CLIENT01

Observed relationship:

```text
ENTERPRISE ADMINS@HARISH.LOCAL
        │
        │ GenericAll
        ▼
CLIENT01.HARISH.LOCAL
```

This demonstrates the strong administrative control that Enterprise Admins can have over computer objects.

---

## 8.3 Account Operators → CLIENT01

Observed relationship:

```text
ACCOUNT OPERATORS@HARISH.LOCAL
        │
        │ GenericAll
        ▼
CLIENT01.HARISH.LOCAL
```

BloodHound's abuse information indicated that this level of control can potentially be associated with techniques such as:

- Shadow Credentials
- Resource-Based Constrained Delegation
- ESC14
- Other abuse possibilities depending on the exact environment

### Important Lab Observation

The Account Operators group was inspected and showed:

```text
Members: 0
Member Of: 0
```

Therefore, although the relationship exists in the graph, there was no directly observed Account Operators member available to use for this relationship during the current reconnaissance.

This is an important distinction between:

```text
A relationship exists
```

and:

```text
An immediately usable attack path exists
```

---

## 8.4 Key Admins → CLIENT01

Observed relationship:

```text
KEY ADMINS@HARISH.LOCAL
        │
        │ AddKeyCredentialLink
        ▼
CLIENT01.HARISH.LOCAL
```

BloodHound associated this permission with Shadow Credentials-related abuse.

---

## 8.5 Enterprise Key Admins → CLIENT01

Observed relationship:

```text
ENTERPRISE KEY ADMINS@HARISH.LOCAL
        │
        │ AddKeyCredentialLink
        ▼
CLIENT01.HARISH.LOCAL
```

Again, BloodHound associated this permission with Shadow Credentials-related abuse.

---

## 8.6 Administrators → CLIENT01

Observed relationship:

```text
ADMINISTRATORS@HARISH.LOCAL
        │
        │ WriteOwner
        ▼
CLIENT01.HARISH.LOCAL
```

`WriteOwner` means the source has the ability to modify the ownership of the target object.

---

# 9. Administrator Account Reconnaissance

The following account was examined:

```text
ADMINISTRATOR@HARISH.LOCAL
```

BloodHound showed membership in several groups.

Observed memberships included:

```text
Denied RODC Password Replication Group
Enterprise Admins
Domain Admins
Domain Users
Authenticated Users
Everyone
Administrators
Users
Schema Admins
Group Policy Creator Owners
Pre-Windows 2000 Compatible Access
```

This confirms that the Administrator account is a highly privileged account in the lab domain.

The important security lesson is that compromise of such an account would have significant impact across the Active Directory environment.

---

# 10. HARISH User Reconnaissance

The following user was examined:

```text
HARISH@HARISH.LOCAL
```

## 10.1 Group Membership

BloodHound showed that Harish is a member of:

```text
Authenticated Users
Everyone
Users
HR Users
HR_DL_Modify
Pre-Windows 2000 Compatible Access
Domain Users
```

This represents a normal departmental user baseline in the lab.

Harish was not observed as a member of:

```text
Domain Admins
Enterprise Admins
```

during this reconnaissance.

---

# 11. HARISH Inbound Object Control

BloodHound showed several inbound relationships to the Harish user object.

These relationships are important because they show which objects have permissions over Harish.

---

## 11.1 Account Operators → HARISH

Observed:

```text
ACCOUNT OPERATORS@HARISH.LOCAL
        │
        │ GenericAll
        ▼
HARISH@HARISH.LOCAL
```

BloodHound indicated possible abuse such as:

- ForceChangePassword
- Shadow Credentials
- ESC14
- Targeted Kerberoasting

The relationship was shown as:

```text
Is ACL: TRUE
Is Inherited: FALSE
```

However, Account Operators had:

```text
Members: 0
```

So this relationship was documented as a security relationship, not treated as an immediately exploitable path.

---

## 11.2 Administrators → HARISH

Observed:

```text
ADMINISTRATORS@HARISH.LOCAL
        │
        │ AllExtendedRights
        ▼
HARISH@HARISH.LOCAL
```

The relationship showed:

```text
Is ACL: TRUE
Is Inherited: TRUE
```

BloodHound indicated ForceChangePassword as an associated abuse possibility.

---

## 11.3 Enterprise Admins → HARISH

Observed:

```text
ENTERPRISE ADMINS@HARISH.LOCAL
        │
        │ GenericAll
        ▼
HARISH@HARISH.LOCAL
```

The relationship showed:

```text
Is ACL: TRUE
Is Inherited: TRUE
```

BloodHound associated this relationship with possibilities including:

- ForceChangePassword
- Shadow Credentials
- ESC14
- Targeted Kerberoasting

---

## 11.4 Enterprise Key Admins → HARISH

Observed:

```text
ENTERPRISE KEY ADMINS@HARISH.LOCAL
        │
        │ AddKeyCredentialLink
        ▼
HARISH@HARISH.LOCAL
```

BloodHound provided Shadow Credentials-related abuse information for this permission.

---

## 11.5 Key Admins → HARISH

Observed:

```text
KEY ADMINS@HARISH.LOCAL
        │
        │ AddKeyCredentialLink
        ▼
HARISH@HARISH.LOCAL
```

BloodHound again associated this permission with Shadow Credentials-related abuse.

---

## 11.6 Domain Admins → HARISH

Observed:

```text
DOMAIN ADMINS@HARISH.LOCAL
        │
        │ GenericAll
        ▼
HARISH@HARISH.LOCAL
```

The relationship showed:

```text
Is ACL: TRUE
Is Inherited: FALSE
```

BloodHound indicated that members of Domain Admins have broad control over the Harish user object.

### Important Security Lesson

This relationship does **not** mean that Harish is a Domain Admin.

It means:

```text
Domain Admins
      ↓
has control over
      ↓
Harish
```

This distinction between **object being controlled** and **object having privilege** is important when interpreting BloodHound graphs.

---

# 12. Inbound vs Outbound Object Control

BloodHound's relationship direction must be interpreted carefully.

### Inbound Object Control

Means:

```text
Other Object
      ↓
controls
      ↓
Current Object
```

Example:

```text
Domain Admins
      ↓ GenericAll
Harish
```

Domain Admins controls Harish.

### Outbound Object Control

Means:

```text
Current Object
      ↓
controls
      ↓
Other Object
```

For example, if an object has outbound control over another object, that relationship may be relevant when investigating privilege escalation or lateral movement.

---

# 13. Pathfinding Tests

BloodHound Pathfinding was tested using the Harish account.

## 13.1 HARISH → DOMAIN ADMINS

Test:

```text
HARISH@HARISH.LOCAL
        ↓
DOMAIN ADMINS@HARISH.LOCAL
```

Result:

```text
Path not found
```

This means BloodHound did not identify a graph path from Harish to Domain Admins using the available collected relationships.

---

## 13.2 HARISH → CLIENT01

Test:

```text
HARISH@HARISH.LOCAL
        ↓
CLIENT01.HARISH.LOCAL
```

Result:

```text
Path not found
```

Therefore, BloodHound did not identify a path from Harish to CLIENT01 using the relationships available in the collected dataset.

---

## 13.3 HARISH Outbound Object Control

BloodHound showed:

```text
Outbound Object Control: 0
```

This indicates that no outbound object-control relationships were identified for Harish in the current BloodHound dataset.

---

## 13.4 HARISH Local Admin Privileges

BloodHound showed:

```text
Local Admin Privileges: 0
```

This indicates that Harish was not identified as having local administrator privileges on the systems represented in the current BloodHound data.

---

# 14. Why Pathfinding and Relationships Are Different

An important lesson from this lab was that seeing an individual ACL relationship does not automatically mean BloodHound will show a complete attack path.

For example:

```text
Account Operators
       ↓ GenericAll
     Harish
```

is a valid relationship.

But if:

```text
Account Operators
       ↓
Members = 0
```

then there may not be a currently usable principal through which to exercise that permission.

Therefore, reconnaissance should distinguish between:

1. Existing relationship
2. Effective privilege
3. Available membership
4. Complete attack path
5. Exploitable condition

---

# 15. Cypher Query Experiment

BloodHound's Cypher interface was also tested.

The first query attempted to find GenericAll relationships:

```cypher
MATCH p=(n)-[r:GenericAll]->(m)
RETURN n.name AS Source, type(r) AS Permission, m.name AS Target
LIMIT 50
```

Result:

```text
No results
```

A more generic relationship query was then attempted:

```cypher
MATCH (n)-[r]->(m)
RETURN n.name AS Source, type(r) AS Relationship, m.name AS Target
LIMIT 20
```

This produced a Cypher parser error because `Relationship` was interpreted as a reserved keyword by the query parser.

The alias was changed:

```cypher
MATCH (n)-[r]->(m)
RETURN n.name, type(r), m.name
LIMIT 20
```

Result:

```text
No results
```

### Conclusion

The Cypher experiment did not provide useful relationship results in the current BloodHound CE environment.

However, the normal BloodHound Explore interface successfully displayed AD objects and relationships, so the SharpHound collection and upload were not treated as failed.

No fabricated Cypher results were recorded.

---

# 16. Important BloodHound Concepts Learned

## GenericAll

Represents broad control over an Active Directory object.

Example observed:

```text
Domain Admins → GenericAll → Harish
```

---

## WriteOwner

Allows the source principal to modify ownership of the target object.

Example:

```text
Administrators → WriteOwner → CLIENT01
```

---

## AddKeyCredentialLink

Permission associated with modifying the target's `msDS-KeyCredentialLink` attribute.

This can be relevant to Shadow Credentials techniques.

Observed examples:

```text
Key Admins → AddKeyCredentialLink → CLIENT01
Enterprise Key Admins → AddKeyCredentialLink → CLIENT01
```

---

## AllExtendedRights

Represents extended rights over an AD object.

Observed:

```text
Administrators → AllExtendedRights → Harish
```

BloodHound associated ForceChangePassword as an abuse possibility for this relationship.

---

## Local Admin Privileges

Indicates that a principal has local administrative privileges on a computer.

Example observed:

```text
Domain Admins → CLIENT01
Domain Admins → DC01
```

---

# 17. Security Interpretation

The reconnaissance produced several important observations.

### Observation 1 — Privileged groups have broad control

Domain Admins and Enterprise Admins showed extensive relationships throughout the domain.

### Observation 2 — ACLs can be security-critical

Permissions such as:

```text
GenericAll
WriteOwner
AddKeyCredentialLink
AllExtendedRights
```

can become important during privilege-escalation analysis.

### Observation 3 — Membership matters

A relationship alone does not necessarily mean that an attacker can immediately use it.

The actual membership and privileges of the source principal must also be considered.

### Observation 4 — Harish is a normal-user baseline

Harish had departmental memberships such as:

```text
HR Users
HR_DL_Modify
Domain Users
```

and BloodHound showed:

```text
Outbound Object Control: 0
Local Admin Privileges: 0
```

### Observation 5 — No direct BloodHound path was identified

The tests:

```text
Harish → Domain Admins
Harish → CLIENT01
```

both returned:

```text
Path not found
```

This establishes the current baseline before performing additional reconnaissance techniques.

---

# 18. Reconnaissance Methodology

The practical workflow used in this lab was:

```text
1. Collect AD data
       ↓
2. Upload SharpHound data
       ↓
3. Identify privileged groups
       ↓
4. Examine users
       ↓
5. Examine computers
       ↓
6. Inspect ACL relationships
       ↓
7. Check memberships
       ↓
8. Test meaningful paths
       ↓
9. Interpret security impact
       ↓
10. Document findings
```

This approach is more useful than randomly checking every object in the domain.

---

# 19. Screenshots

Only actual screenshots from this lab should be added.

Suggested screenshots:

```text
04-Reconnaissance/Screenshots/
├── BloodHound-Dashboard.png
├── Domain-Admins-Overview.png
├── Domain-Admins-Members.png
├── Domain-Admins-Local-Admin.png
├── Administrators-Overview.png
├── CLIENT01-Relationships.png
├── CLIENT01-GenericAll-Abuse.png
├── Administrator-Memberships.png
├── Harish-Memberships.png
├── Harish-Inbound-Control.png
├── Harish-Pathfinding-Domain-Admins.png
├── Harish-Pathfinding-CLIENT01.png
└── Harish-Outbound-Control.png
```

Use the actual screenshot filenames when adding them to the repository.

---

# 20. Key Takeaways

- BloodHound represents Active Directory as a graph of objects and relationships.
- SharpHound collects the AD information used by BloodHound.
- Domain Admins and Enterprise Admins are highly privileged groups.
- ACL relationships can be more important than simple group membership.
- `GenericAll` represents broad object control.
- `WriteOwner` represents ownership-related control.
- `AddKeyCredentialLink` can be relevant to Shadow Credentials.
- Inbound control and outbound control must not be confused.
- A relationship does not automatically equal an exploitable attack path.
- Membership of the source principal must be considered.
- Pathfinding can help determine whether relationships form a complete path.
- In this lab, Harish had no identified outbound object control or local admin privileges.
- Harish → Domain Admins and Harish → CLIENT01 returned no path.
- Cypher queries did not return useful results in the current environment.
- The BloodHound Explore interface successfully provided useful reconnaissance data.

---

# 21. Lab Status

**BloodHound Reconnaissance: Completed**

Completed:

- [x] SharpHound data collection
- [x] BloodHound data upload
- [x] BloodHound interface exploration
- [x] Domain Admins analysis
- [x] Administrators analysis
- [x] CLIENT01 analysis
- [x] Administrator analysis
- [x] Harish user analysis
- [x] ACL relationship analysis
- [x] Pathfinding tests
- [x] Cypher query experiment
- [x] Reconnaissance findings documented

Next reconnaissance techniques:

```text
LDAP Enumeration
       ↓
SMB Enumeration
       ↓
DNS Enumeration
```

These will be documented separately and should not be mixed into this BloodHound reconnaissance document.
