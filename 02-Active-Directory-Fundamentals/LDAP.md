# LDAP — Lightweight Directory Access Protocol

## Objective

Understand the role of LDAP in Active Directory, including:

* What LDAP is
* Why Active Directory uses LDAP
* LDAP and Active Directory relationship
* Directory hierarchy
* LDAP objects
* DN, CN, OU and DC
* LDAP ports
* LDAP security relevance
* LDAP vs Active Directory

# 1. What is LDAP?

LDAP stands for:

> **Lightweight Directory Access Protocol**

LDAP is an application protocol used to access, search, and manage information stored in directory services such as Active Directory.

### Simple definition

> LDAP provides a standard way for clients and applications to communicate with and query directory services.

# 2. LDAP vs Active Directory

LDAP and Active Directory are not the same thing.

Active Directory
      │
      ├── Directory Service
      ├── Users
      ├── Groups
      ├── Computers
      ├── OUs
      ├── Authentication
      ├── Authorization
      ├── Security
      └── Other AD components
              ▲
              │
             LDAP
              │
              └── Protocol used to access/query
                  directory information

### Remember

> **Active Directory is the directory service; LDAP is a protocol used to communicate with the directory.**

# 3. Why does Active Directory use LDAP?

Active Directory stores information about many types of objects:

* Users
* Groups
* Computers
* Organizational Units
* Contacts
* Printers
* Other directory objects

Applications and systems need a way to find and work with this information.

LDAP provides this communication mechanism.

Example:

Application
     │
     │ LDAP request
     ▼
    DC01
     │
     ▼
Active Directory
     │
     ▼
User / Group / Computer information
     │
     │ LDAP response
     ▼
Application

# 4. Example

An application may need to find information about the user `Harish`.

Conceptually:

Application
     │
     │ "Find Harish"
     ▼
    LDAP
     │
     ▼
    DC01
     │
     ▼
Active Directory
     │
     ▼
Harish user object

The application can then receive the relevant directory attributes.

Other possible directory queries include:

Find all users in HR

Find members of a group

Find computers in an OU

Find a particular user

Find directory objects matching specific attributes


# 5. Active Directory Directory Structure

Active Directory uses a hierarchical structure.

Example:

harish.local
│
├── IT
│   ├── Harish
│   └── CLIENT01
│
├── HR
│   └── Ravi
│
└── Finance
    └── Suresh

LDAP provides a way to navigate and query this directory information.


# 6. LDAP Directory Tree

LDAP represents directory information as a hierarchical directory tree.

For the lab domain:

DC=harish,DC=local
        │
        ├── OU=IT
        │     ├── CN=Harish
        │     └── CN=CLIENT01
        │
        ├── OU=HR
        │     └── CN=Ravi
        │
        └── OU=Finance
              └── CN=Suresh

The exact objects and hierarchy in the lab may differ from this example.

# 7. DC — Domain Component

`DC` stands for:

> **Domain Component**

For the domain:

harish.local


the directory naming components can be represented as:

DC=harish,DC=local


# 8. OU — Organizational Unit

`OU` stands for:

> **Organizational Unit**

OUs are containers within Active Directory used to organize objects.

Example:

OU=IT
OU=HR
OU=Finance
OU=Servers

An OU can contain users, computers, groups, and other supported directory objects.

# 9. CN — Common Name

`CN` stands for:

> **Common Name**

It identifies an object by its common name within the directory naming structure.

Example:

CN=Harish

For a computer:

CN=CLIENT01

# 10. Distinguished Name (DN)

A Distinguished Name identifies an object within the directory hierarchy.

Example:

CN=Harish,OU=IT,DC=harish,DC=local

This can be read from the object outward:

Harish
  ↓
IT OU
  ↓
harish.local domain

### Breakdown

CN=Harish
    ↓
Object

OU=IT
    ↓
Organizational Unit

DC=harish
DC=local
    ↓
Domain


# 11. LDAP Ports

Common LDAP-related ports:

| Protocol                    | Port | Purpose               |
| --------------------------- | ---: | --------------------- |
| LDAP                        |  389 | Standard LDAP         |
| LDAPS                       |  636 | LDAP over SSL/TLS     |
| Global Catalog              | 3268 | Global Catalog        |
| Global Catalog over SSL/TLS | 3269 | Secure Global Catalog |

Port numbers should be treated as protocol defaults; actual deployment and configuration can vary.

# 12. LDAP and Authentication

LDAP can be involved in directory authentication and directory access.

However:

> **LDAP itself should not be confused with Kerberos.**

In an Active Directory environment, different protocols/components have different roles.

For example:

Kerberos
   ↓
Authentication

LDAP
   ↓
Directory access / queries

A real application may use multiple protocols depending on what it needs to do.

# 13. LDAP Security Relevance

LDAP is important in Active Directory security because directory information can reveal useful information about the environment.

Examples of directory information include:

Users
Groups
Computers
OUs
Group memberships
Attributes
Service-related information

During authorized security assessments, LDAP queries can therefore be used for directory reconnaissance.

This becomes important in the later:

04-Reconnaissance
       │
       ├── LDAP Enumeration
       ├── BloodHound
       ├── SharpHound
       ├── PowerView
       └── Other enumeration techniques

Detailed LDAP enumeration will be performed in the practical/reconnaissance phase.


# 14. LDAP and Kerberos

LDAP and Kerberos are different technologies with different roles.

              Active Directory
                    │
          ┌─────────┴─────────┐
          │                   │
       Kerberos             LDAP
          │                   │
          ▼                   ▼
    Authentication      Directory Access
          │                   │
          ▼                   ▼
       Tickets          Directory Queries

They can work together in the same Active Directory environment.


# 15. LDAP vs Active Directory vs Kerberos

| Technology       | What it is              | Main role                                            |
| ---------------- | ----------------------- | ---------------------------------------------------- |
| Active Directory | Directory service       | Stores/manages directory identities and objects      |
| LDAP             | Protocol                | Accesses/queries directory information               |
| Kerberos         | Authentication protocol | Authenticates users/computers/services using tickets |

### Easy memory

> **AD = Directory**
> **LDAP = Directory communication**
> **Kerberos = Authentication**


# 16. Practical Lab — Deferred

The detailed practical implementation is intentionally deferred to the later reconnaissance module.

Planned practical topics:

LDAP
 │
 ├── LDAP connection
 ├── LDAP ports
 ├── Directory structure
 ├── DN
 ├── LDAP queries
 ├── ldapsearch
 ├── PowerShell LDAP queries
 ├── User enumeration
 ├── Group enumeration
 ├── Computer enumeration
 └── LDAP traffic analysis

The practical lab will use the authorized `harish.local` environment.

