# Service Principal Names (SPNs)

## Objective

Understand what a Service Principal Name (SPN) is, why Kerberos uses SPNs, how SPNs relate to service accounts, and why SPNs are important in Active Directory security.


## What is an SPN?

SPN stands for **Service Principal Name**.

An SPN is a unique identifier for a service instance in Active Directory. It associates a network service with the AD account under which that service runs.

Simple model:

SPN
 ↓
Identifies a service
 ↓
Maps to an AD account


Example:

MSSQLSvc/sqlserver.harish.local:1433


## Why Does Kerberos Need an SPN?

Kerberos uses tickets to authenticate clients to services.

When a client requests access to a service, Kerberos needs to identify the exact service for which the ticket is required.

Example:

Client
  ↓
"I need SQL Server"
  ↓
SPN
  ↓
MSSQLSvc/sqlserver.harish.local:1433
  ↓
TGS
  ↓
Service Ticket
  ↓
SQL Server

Therefore:

> The SPN allows Kerberos to identify the specific service instance for which a service ticket is requested.

## SPN Structure

A common SPN format is:

service-class/host
Some services also include a port:
service-class/host:port

Examples:
HTTP/webserver.harish.local
CIFS/fileserver.harish.local
MSSQLSvc/sqlserver.harish.local:1433

## Common SPN Service Classes

Some commonly encountered service classes include:

| Service Class | Example                                |
| ------------- | -------------------------------------- |
| HTTP          | `HTTP/webserver.harish.local`          |
| CIFS          | `CIFS/fileserver.harish.local`         |
| MSSQLSvc      | `MSSQLSvc/sqlserver.harish.local:1433` |
| LDAP          | `LDAP/dc01.harish.local`               |
| HOST          | `HOST/dc01.harish.local`               |

The exact SPNs present in an environment depend on the services and computer/service accounts configured there.

## SPN and Service Account

The SPN identifies the service, while the SPN is registered on an AD account representing the service identity.

For example:

Service:
SQL Server

SPN:
MSSQLSvc/sqlserver.harish.local:1433

AD Account:
sqlservice

The account associated with an SPN can be a user account or a computer account, depending on how the service is configured.

## SPN and Kerberos

The relationship can be represented as:

Client
  │
  │ requests service
  ▼
SPN
  │
  │ identifies service
  ▼
KDC / TGS
  │
  │ issues service ticket
  ▼
Service Ticket
  │
  ▼
Target Service

SPNs are therefore an important part of Kerberos service authentication in Active Directory.

## SPN vs DNS

DNS and SPNs have different purposes.

### DNS

DNS helps resolve a hostname to an IP address.

Example:
fileserver.harish.local
        ↓
IP Address

### SPN

SPN identifies a service identity for Kerberos.

Example:
CIFS/fileserver.harish.local

Simple comparison:
DNS → Where is the host?
SPN → Which Kerberos service identity is being requested?
DNS and SPNs often work together, but they are not the same thing.

## SPN vs Port

An SPN is not simply a port number.

Example:
MSSQLSvc/sqlserver.harish.local:1433

Here:

MSSQLSvc
    ↓
Service class

sqlserver.harish.local
    ↓
Host

1433
    ↓
Port

The port can be included in an SPN when required by the service naming convention.
## SPN Uniqueness

An SPN is intended to uniquely identify a service instance.

If the same SPN is incorrectly registered on multiple accounts, Kerberos authentication can fail because the service identity becomes ambiguous.

Duplicate SPNs can therefore cause authentication and service-access problems.

A command that can be used to check for duplicate SPNs is:

setspn -X

Detailed SPN troubleshooting will be performed during the practical lab.

## Viewing SPNs

Windows provides the `setspn` utility for managing and querying SPNs.

Example:
setspn -L <account>
This lists the SPNs registered on the specified account.
Example:
setspn -L DC01

The actual output depends on the SPNs configured in the lab environment.

## SPNs and Active Directory Security

SPNs are especially important in AD security because they connect Kerberos services with AD accounts.

During reconnaissance, security testers can enumerate SPNs to identify service accounts.

This becomes relevant to **Kerberoasting**.

Conceptual relationship:

SPN
 ↓
Service Account
 ↓
Kerberos Service Ticket
 ↓
Kerberoasting
 ↓
Offline password attack

Important:

> The presence of an SPN is normal and does not automatically mean that an account is vulnerable.

The security risk depends on factors such as account configuration, password strength, privileges, and environment design.


## SPN and Kerberoasting

Kerberoasting is covered later in:

05-Credential-Attacks/Kerberoasting.md

At a high level:

Attacker
   ↓
Enumerates SPNs
   ↓
Identifies service accounts
   ↓
Requests Kerberos service tickets
   ↓
Obtains ticket-related material
   ↓
Attempts offline password cracking


This is why understanding SPNs is necessary before learning Kerberoasting.

## SPN and Our Lab

### Domain
harish.local

### Domain Controller

DC01
IP: 192.168.182.10


### Client

CLIENT01


### Example Environment

CLIENT01
   │
   │ Kerberos request
   ▼
DC01 / KDC
   │
   │ SPN lookup
   ▼
Service Account
   │
   ▼
Service Ticket
   │
   ▼
Target Service

The detailed SPN enumeration practical will be performed later during the reconnaissance phase.


## Detection and Defense

Organizations can reduce SPN-related security risk by:

* Monitoring service accounts.
* Using strong, unique passwords for service accounts.
* Using managed service accounts where appropriate.
* Avoiding unnecessary privileges for service accounts.
* Monitoring unusual Kerberos service-ticket requests.
* Investigating unexpected SPNs.
* Detecting duplicate or incorrectly configured SPNs.
* Limiting administrative privileges.

SPNs themselves are a normal part of Kerberos and should not simply be removed.


## Key Takeaways

SPN = Service Principal Name

SPN identifies a Kerberos service instance.

SPN → Service → AD Account

Kerberos uses SPNs when requesting service tickets.

DNS and SPN are different:

DNS → resolves hostnames
SPN → identifies Kerberos service identity

SPNs are important in security because they are relevant to
service-account enumeration and Kerberoasting.

Duplicate SPNs can cause Kerberos authentication problems.

## References

Microsoft documentation:

* Active Directory Domain Services documentation
* Microsoft `setspn` documentation
* Microsoft Kerberos protocol documentation

Official Microsoft documentation:

https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/setspn

https://learn.microsoft.com/en-us/windows-server/security/kerberos/
