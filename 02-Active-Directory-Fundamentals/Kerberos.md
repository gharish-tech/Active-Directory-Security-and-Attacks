# Kerberos Authentication

## Objective

Understand how Kerberos authentication works in an Active Directory environment, including:

* Kerberos
* Key Distribution Center (KDC)
* Authentication Service (AS)
* Ticket-Granting Service (TGS)
* Ticket-Granting Ticket (TGT)
* Service Tickets
* AS Exchange
* TGS Exchange
* AP Exchange
* Single Sign-On
* Kerberos vs NTLM
* Kerberos security relevance

# 1. What is Kerberos?

Kerberos is a ticket-based network authentication protocol used to verify the identity of users, computers, and services.

Windows Server implements Kerberos Version 5 (V5) for Active Directory domain authentication.

Kerberos allows authenticated users to access authorized network resources without repeatedly entering their credentials.

### Simple definition

> Kerberos is a ticket-based authentication protocol used by Active Directory to provide secure authentication and Single Sign-On.

# 2. Why is Kerberos Important?

In an enterprise environment, a user may need to access many services:

* File servers
* SQL servers
* Web applications
* Printers
* Internal applications
* Other domain computers

Without an efficient authentication mechanism, the user would repeatedly need to authenticate to different services.

Kerberos solves this using tickets.

### Basic concept

Initial Authentication
        ↓
       TGT
        ↓
Service Ticket Request
        ↓
Service Ticket
        ↓
Target Service

This allows Single Sign-On within the domain/forest where applicable.

# 3. Kerberos in Active Directory

My lab environment:

Domain:       harish.local

Domain Controller:
DC01
192.168.182.10

Client:
CLIENT01
Windows 11

User:
harish

Conceptually:

             CLIENT01
                 │
                 │
                 ▼
          DC01 / KDC
          harish.local
                 │
          ┌──────┴──────┐
          │             │
         AS            TGS

The Kerberos KDC runs on the Domain Controller and uses the Active Directory Domain Services database as its account database.

# 4. KDC — Key Distribution Center

KDC stands for:

> Key Distribution Center

In Active Directory, the KDC is integrated with the Domain Controller.

The KDC provides two major services:

KDC
│
├── AS
│   Authentication Service
│
└── TGS
    Ticket-Granting Service

# 5. Authentication Service (AS)

AS stands for:

> Authentication Service

The AS handles the initial authentication process.

Its important role is to issue a:

> Ticket-Granting Ticket (TGT)

Simplified flow:

CLIENT01
   │
   │ Authentication Request
   ▼
KDC / AS
   │
   │ Verify initial authentication
   ▼
TGT
   │
   ▼
CLIENT01

The client stores the TGT in its Kerberos ticket cache.

# 6. Ticket-Granting Service (TGS)

TGS stands for:

> Ticket-Granting Service

The TGS provides tickets for specific services.

Example:

Harish wants to access:

\\FILESERVER\Share

The client already has a TGT.

It uses the TGT to request a ticket for the required service.

CLIENT01
   │
   │ TGT + Service Request
   ▼
KDC / TGS
   │
   │
   ▼
Service Ticket
   │
   ▼
CLIENT01

# 7. Ticket-Granting Ticket (TGT)

TGT stands for:

> Ticket-Granting Ticket

The TGT is issued by the Authentication Service after the initial authentication process.

The client uses the TGT to request Service Tickets from the TGS.

### Important

The TGT is NOT normally the ticket presented directly to the file server.

Instead:

TGT
 ↓
Request Service Ticket
 ↓
Service Ticket
 ↓
Target Service

### Easy memory

> TGT gets you a Service Ticket.

# 8. Service Ticket

A Service Ticket is a Kerberos ticket issued for a particular service.

Examples may include services such as:

CIFS
HTTP
LDAP
MSSQLSvc

For example, when accessing an SMB file share, a relevant service principal can use the CIFS service class.

Conceptually:

Harish
  ↓
Wants FILESERVER
  ↓
TGT
  ↓
TGS
  ↓
Service Ticket
  ↓
FILESERVER

### Easy memory

> Service Ticket gets you to the service.

# 9. AS Exchange

AS Exchange means:

> Authentication Service Exchange

This is the initial Kerberos exchange.

The client requests authentication from the KDC's Authentication Service.

Simplified:

CLIENT
   │
   │ AS-REQ
   ▼
KDC / AS
   │
   │ AS-REP
   ▼
CLIENT
   │
   ▼
TGT

The result of the AS exchange is the TGT.


# 10. TGS Exchange

TGS Exchange means:

> Ticket-Granting Service Exchange

The client already has a TGT.

Now it wants to access a specific service.

Example:

\\FILESERVER\Share


The client sends a request to the TGS.

CLIENT
   │
   │ TGT + Request for Service
   ▼
KDC / TGS
   │
   │ TGS response
   ▼
Service Ticket
   │
   ▼
CLIENT

# 11. AP Exchange

AP stands for:

> Application Protocol / Client-Server Exchange

After receiving the Service Ticket, the client presents the ticket to the target service.

Simplified:

CLIENT
   │
   │ Service Ticket
   ▼
TARGET SERVICE
   │
   ▼
Authentication
   │
   ▼
Authorization
   │
   ▼
Access

The exact AP exchange details depend on the application protocol.

# 12. Complete Kerberos Flow

                     CLIENT01
                         │
                         │
                         │ 1. AS-REQ
                         ▼
                    DC01 / KDC
                         │
                         │ 2. AS-REP
                         ▼
                        TGT
                         │
                         │
                         │ 3. TGS-REQ
                         ▼
                    DC01 / KDC
                         │
                         │ 4. TGS-REP
                         ▼
                  Service Ticket
                         │
                         │
                         │ 5. AP Exchange
                         ▼
                    FILESERVER
                         │
                         ▼
                  Authentication
                         │
                         ▼
                   Authorization
                         │
                         ▼
                       ACCESS


### Core chain

AS
 ↓
TGT
 ↓
TGS
 ↓
Service Ticket
 ↓
Service

# 13. TGT vs Service Ticket

| Feature          | TGT                           | Service Ticket             |
| ---------------- | ----------------------------- | -------------------------- |
| Full name        | Ticket-Granting Ticket        | Service Ticket             |
| Obtained through | AS Exchange                   | TGS Exchange               |
| Purpose          | Request additional tickets    | Access a specific service  |
| Presented to     | TGS                           | Target service             |
| Example          | Request ticket for FILESERVER | Authenticate to FILESERVER |

### Most important 

> TGT is used to obtain a Service Ticket; the Service Ticket is used to authenticate to the target service.

# 14. Kerberos vs NTLM

## Kerberos

Initial Authentication
        ↓
       TGT
        ↓
Service Ticket
        ↓
Target Service

Kerberos is ticket-based.

## NTLM

NEGOTIATE
    ↓
CHALLENGE
    ↓
AUTHENTICATE
    ↓
Response Validation

NTLM uses a challenge-response model.

### Fundamental difference

> Kerberos is primarily ticket-based, while NTLM uses a challenge-response authentication mechanism.

# 15. Single Sign-On

One major benefit of Kerberos is Single Sign-On.

Example:

Harish logs into CLIENT01
          ↓
      Kerberos
          ↓
          TGT
          ↓
    ┌─────┼─────┐
    ↓     ↓     ↓
 File   SQL   Web
Server Server App

The user does not normally need to repeatedly enter the original password for every domain service.


# 16. Kerberos Security Relevance

Kerberos is extremely important in Active Directory security.

Several attacks and security concepts depend on understanding Kerberos:

Kerberos
   │
   ├── SPNs
   │
   ├── Kerberoasting
   │
   ├── AS-REP Roasting
   │
   ├── Pass-the-Ticket
   │
   ├── Delegation
   │
   └── Golden Ticket

These topics will be studied later in the attack modules.

# 17. Important Kerberos Terms

| Term           | Meaning                              |
| -------------- | ------------------------------------ |
| Kerberos       | Ticket-based authentication protocol |
| KDC            | Key Distribution Center              |
| AS             | Authentication Service               |
| TGS            | Ticket-Granting Service              |
| TGT            | Ticket-Granting Ticket               |
| Service Ticket | Ticket for a specific service        |
| SPN            | Service Principal Name               |
| AS-REQ         | Authentication Service Request       |
| AS-REP         | Authentication Service Reply         |
| TGS-REQ        | Ticket-Granting Service Request      |
| TGS-REP        | Ticket-Granting Service Reply        |
| AP             | Client/Server application exchange   |

# 18. Practical Lab

## Lab Environment

Domain:
harish.local

DC:
DC01
192.168.182.10

Client:
CLIENT01
Windows 11

User:
harish



# 19. Key Takeaways

1. Kerberos is a ticket-based authentication protocol.
2. Active Directory uses Kerberos V5 for domain authentication.
3. The KDC runs on the Domain Controller.
4. KDC contains:

   * Authentication Service (AS)
   * Ticket-Granting Service (TGS)
5. AS issues TGTs.
6. TGTs are used to request Service Tickets.
7. TGS issues Service Tickets.
8. Service Tickets are used to authenticate to specific services.
9. Kerberos supports Single Sign-On.
10. SPNs identify services in Kerberos.
11. Kerberos knowledge is required to understand Kerberoasting, Pass-the-Ticket, Golden Ticket and delegation.


### definition

> Kerberos is a ticket-based authentication protocol used by Active Directory for secure authentication and Single Sign-On.

### KDC

> KDC is the Key Distribution Center running on the Domain Controller and providing AS and TGS services.

### AS

> AS authenticates the client and issues a TGT.

### TGT

> TGT is used to request Service Tickets from the TGS.

### TGS

> TGS issues Service Tickets for specific services.

### Service Ticket

> A Service Ticket is used by the client to authenticate to a specific network service.

### Complete flow

AS
 ↓
TGT
 ↓
TGS
 ↓
Service Ticket
 ↓
Service


### Kerberos vs NTLM

Kerberos → Ticket-Based
NTLM     → Challenge-Response


## References

* Microsoft Learn — Kerberos Authentication Overview:
  https://learn.microsoft.com/en-us/windows-server/security/kerberos/kerberos-authentication-overview

* Microsoft Learn — Key Distribution Center:
  https://learn.microsoft.com/en-us/windows/win32/secauthn/key-distribution-center

* Microsoft Learn — Authentication Service Exchange:
  https://learn.microsoft.com/en-us/windows/win32/secauthn/authentication-service-exchange

* Microsoft Learn — Ticket-Granting Tickets:
  https://learn.microsoft.com/en-us/windows/win32/secauthn/ticket-granting-tickets

* Microsoft Learn — Windows Authentication Overview:
  https://learn.microsoft.com/en-us/windows-server/security/windows-authentication/windows-authentication-overview
