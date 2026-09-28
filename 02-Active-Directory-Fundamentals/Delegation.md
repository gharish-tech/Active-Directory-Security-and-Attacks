# Kerberos Delegation

## Objective

Understand Kerberos delegation, why it is used in Active Directory environments, the different delegation models, and the security risks associated with incorrect delegation configuration.

## What is Kerberos Delegation?

Kerberos delegation is a mechanism that allows a service to authenticate to another service **on behalf of a user**.

A common enterprise scenario is:

User
  ↓
Web Server
  ↓
Database Server

The web server may need to access the database while representing the user's identity.

Delegation provides mechanisms for this type of multi-tier authentication.

## Why is Delegation Needed?

Modern enterprise applications commonly use multiple service tiers.

Example:

Employee
   ↓
Web Application
   ↓
Application Server
   ↓
Database Server

The user may authenticate to the web application, while the application server needs to access another backend service.

Delegation allows authentication to flow between services in a controlled manner.

## Normal Kerberos Authentication

In a simple scenario:

User
 ↓
KDC
 ↓
Service Ticket
 ↓
Target Service

The user directly obtains and presents a service ticket for the target service.
## Kerberos Delegation

With delegation:

User
 ↓
Service A
 ↓
Service B

Service A can authenticate to Service B on behalf of the user, when the environment is configured to allow that behavior.

The important concept is:

> Service A is acting on behalf of the user when accessing Service B.

# Types of Kerberos Delegation

The major delegation models are:
Kerberos Delegation
       │
       ├── Unconstrained Delegation
       │
       ├── Constrained Delegation
       │
       └── Resource-Based Constrained Delegation
                    (RBCD)

# 1. Unconstrained Delegation

Unconstrained delegation is an older delegation model that provides broad delegation capability.

Conceptually:

User
 ↓
Server configured for
Unconstrained Delegation
 ↓
Other Services

The delegation is not restricted to a specific list of target services in the way constrained delegation is.

## Security Risk

A system configured for unconstrained delegation becomes security-sensitive.

If an attacker compromises such a system, Kerberos credentials/tickets associated with users authenticating to that system may become valuable attack material.

This is particularly concerning when privileged users authenticate to systems configured for unconstrained delegation.

# 2. Constrained Delegation

Constrained delegation restricts where a service can delegate authentication.

Example:

WEB01
  │
  ├── SQL01 ✓
  │
  ├── File Server ✗
  │
  └── DC01 ✗

The service is configured to delegate only to specified services.

This provides more control than unconstrained delegation.

# 3. Resource-Based Constrained Delegation

RBCD stands for:

> Resource-Based Constrained Delegation

RBCD changes the direction in which the trust relationship is configured.

Traditional constrained delegation can be thought of as:

Source Service
      ↓
defines
      ↓
where it can delegate

RBCD can be thought of as:

Target Resource
      ↓
defines
      ↓
which accounts/services it trusts
to delegate to it

Example:
Traditional:

WEB01
 ↓
"I can delegate to SQL01."


RBCD:

SQL01
 ↓
"I trust WEB01 to delegate to me."

This source-versus-target distinction is one of the most important concepts to remember.

# Delegation Comparison

| Feature                  | Unconstrained                             | Constrained                       | RBCD                                        |
| ------------------------ | ----------------------------------------- | --------------------------------- | ------------------------------------------- |
| Scope                    | Broad                                     | Restricted                        | Restricted                                  |
| Target restriction       | No specific target list in the same sense | Specific services                 | Target resource controls trusted principals |
| Configuration concept    | Source service broadly trusted            | Source specifies allowed services | Target specifies trusted principals         |
| Security concern         | High if compromised                       | Misconfiguration/over-privilege   | Misconfiguration/over-privilege             |
| Important for AD attacks | Yes                                       | Yes                               | Yes                                         |

# Real-World Example

Consider an enterprise application:

Employee
    │
    ▼
Web Server
    │
    ▼
Application Server
    │
    ▼
SQL Server

Suppose Harish logs into the web application.

The application server needs to retrieve Harish's information from SQL Server.

The application may need to authenticate to SQL Server while representing Harish.

Conceptually:
Harish
  ↓
Web/Application Server
  ↓
SQL Server
  ↑
"on behalf of Harish"

Kerberos delegation mechanisms can support this architecture.

# Delegation and Active Directory

Delegation relies on Kerberos and Active Directory configuration.

Important components include:

Active Directory
      ↓
Kerberos
      ↓
Service Accounts / Computer Accounts
      ↓
Delegation Configuration
      ↓
Service-to-Service Authentication

This connects several concepts already studied:

Authentication
      ↓
Kerberos
      ↓
SPN
      ↓
Service Account
      ↓
Delegation

# Delegation and SPNs

SPNs identify services for Kerberos.

Delegation determines whether a service can authenticate to another service on behalf of a user.

Therefore:

SPN
 ↓
Identifies service
 ↓
Kerberos
 ↓
Delegation
 ↓
Access to another service

Understanding SPNs is therefore useful before studying delegation abuse.

# Security Importance

Delegation is a legitimate enterprise feature.

It should not automatically be considered malicious or insecure.

The security problem arises when delegation is:

* Unnecessarily enabled
* Too broadly configured
* Applied to sensitive systems
* Applied to accounts with excessive privileges
* Poorly monitored

Potential attack path:

Compromise
    ↓
Delegation Configuration
    ↓
Kerberos Credential/Ticket Abuse
    ↓
Lateral Movement
    ↓
Potential Privilege Escalation


# Security Risks

Important risks include:

### Unconstrained Delegation

A compromised system configured for unconstrained delegation may expose valuable Kerberos credential/ticket material associated with users authenticating to it.

### Constrained Delegation

Incorrectly configured target services can create unintended access paths.

### RBCD

If an attacker gains the ability to modify the relevant directory attributes or otherwise control delegation configuration, RBCD can become a powerful privilege-escalation/lateral-movement mechanism.

Detailed exploitation will be covered later in the attack modules.


# Detection

Security teams should identify:

* Computers configured for unconstrained delegation
* Accounts configured for delegation
* Constrained delegation settings
* RBCD configuration
* Unexpected changes to delegation-related AD attributes
* Suspicious Kerberos activity
* Authentication involving privileged users and delegation-enabled systems

Useful investigation tools later include:

BloodHound
PowerShell
LDAP queries
Event Logs
Directory auditing

# Defense

Recommended defensive principles include:

* Avoid unnecessary unconstrained delegation.
* Use constrained delegation where appropriate.
* Carefully control RBCD permissions.
* Apply least privilege.
* Prevent privileged accounts from unnecessarily authenticating to delegation-sensitive systems.
* Monitor delegation configuration changes.
* Protect service accounts.
* Monitor suspicious Kerberos authentication activity.
* Regularly review delegation relationships.

