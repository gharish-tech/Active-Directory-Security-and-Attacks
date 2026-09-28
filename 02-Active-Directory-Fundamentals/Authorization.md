# Authorization

## Objective

Understand authorization in Active Directory, including security principals, SIDs, access tokens, ACLs, ACEs, groups, NTFS permissions, share permissions, inheritance, and the relationship between authorization and privilege escalation.

## What is Authorization?

Authorization is the process of determining what an authenticated identity is allowed to access or perform.

Simple:
Authentication
→ Who are you?

Authorization
→ What are you allowed to access or do?

Example:

Harish
  ↓
Authentication
  ↓
Identity verified
  ↓
Authorization
  ↓
Check permissions
  ↓
Resource access

## Authentication vs Authorization

| Authentication                 | Authorization                             |
| ------------------------------ | ----------------------------------------- |
| Verifies identity              | Determines permissions                    |
| Who are you?                   | What can you access?                      |
| Uses authentication mechanisms | Uses permissions, groups, ACLs and rights |
| Happens before authorization   | Uses the established identity             |
| Example: Kerberos login        | Example: Read access to a folder          |


## Authorization Flow

User
  ↓
Authentication
  ↓
Identity established
  ↓
Logon Session
  ↓
Access Token
  ↓
Authorization Check
  ↓
Resource
  │
  ├── Allowed
  │
  └── Denied

## Access Token

After successful authentication, Windows establishes a logon session and creates an access token associated with that session.

The token contains security information associated with the authenticated identity, including the user's SID and relevant group SIDs.

Conceptually:

Authentication
      ↓
User Identity
      ↓
Access Token
      │
      ├── User SID
      ├── Group SIDs
      └── Other security information
      ↓
Authorization Decision

Windows can use this information when determining whether an operation should be permitted.

## Security Principal

A security principal is an identity that can be assigned permissions or rights.

Examples include:

* User accounts
* Group accounts
* Computer accounts
* Service identities

Example:

harish
IT-Group
CLIENT01$

These identities can appear in access-control configurations.

## SID

SID stands for **Security Identifier**.

Windows uses SIDs to identify security principals.

A user and a group each have their own SID.

To view the current user's SID:
whoami /user

To view group memberships:

whoami /groups

## ACL

ACL stands for **Access Control List**.

An ACL is a collection of access-control entries that specify permissions or access rules.

Example:

Resource
   ↓
ACL
   ├── IT Group → Read
   ├── HR Group → Modify
   └── Administrators → Full Control

## ACE

ACE stands for **Access Control Entry**.

An ACE is an individual entry within an ACL.

Example:

ACL
│
├── ACE → IT Group → Read
├── ACE → HR Group → Modify
└── ACE → Administrators → Full Control

Therefore:
ACL = Collection
ACE = Individual entry

## NTFS Permissions

NTFS permissions control access to files and folders stored on an NTFS file system.

Common permissions include:

* Full Control
* Modify
* Read & Execute
* Read
* Write

Example:

C:\CompanyData\Finance

Finance Group → Modify
IT Group      → Read

## Share Permissions

When a folder is exposed through an SMB network share, share permissions can also control access.

Example:

\\DC01\Finance
       │
       ├── Share Permissions
       │
       └── NTFS Permissions

When accessing a resource through a network share, both share-level and NTFS permissions can affect effective access.

## Effective Access

Effective access depends on the complete authorization configuration.

Relevant factors can include:

* User membership
* Group membership
* Allow entries
* Deny entries
* Explicit permissions
* Inherited permissions
* Share permissions
* NTFS permissions

Therefore, looking at one permission entry alone may not explain the final access result.

## Allow and Deny

Access-control entries can grant or restrict access.

Example:

IT Group
    ↓
Allow Read

Harish
    ↓
Deny Read

Deny entries can override otherwise granted access depending on the specific Windows access-control configuration.

Because explicit deny rules can make permissions harder to understand, they should be used carefully.

## Permission Inheritance

Permissions can be inherited from parent objects or folders.

Example:

Company
  │
  ├── HR
  ├── IT
  └── Finance

If inheritance is enabled:

Parent Permissions
       ↓
Inherited Permissions
       ↓
Child Objects/Folders

Inheritance simplifies administration but can also make troubleshooting permissions more complex.

## Authorization and Groups

Groups are important because permissions can be assigned to groups rather than individual users.

Example:

Harish
   ↓
HR_Global
   ↓
HR_DomainLocal
   ↓
HR Share

This follows the AGDLP model used in the lab.

Accounts
   ↓
Global Groups
   ↓
Domain Local Groups
   ↓
Permissions
   ↓
Resources


Group-based authorization improves scalability and simplifies access management.

## Authorization in Active Directory

Authorization applies to many types of resources:

Authorization
      │
      ├── Files
      ├── Folders
      ├── SMB Shares
      ├── AD Objects
      ├── GPOs
      ├── Applications
      ├── Services
      └── Administrative Rights


Active Directory objects can also have access-control information determining who can perform particular operations on them.


## Authorization and Privilege Escalation

Authorization is directly related to privilege escalation.

A security misconfiguration may give a low-privileged identity excessive rights.

Conceptually:

Low-Privileged User
        ↓
Unexpected Permission
        ↓
Important Object
        ↓
Unauthorized Modification
        ↓
Potential Privilege Escalation

This becomes important later in:

06-Privilege-Escalation/ACL-Abuse.md


## Authorization and AAA

The AAA model is:

Authentication
→ Who are you?

Authorization
→ What can you do?

Accounting / Auditing
→ What did you do?

In an AD security environment:

Authentication
→ Kerberos / NTLM

Authorization
→ Groups / ACLs / Permissions / Rights

Auditing
→ Security Events / Logs / Monitoring
