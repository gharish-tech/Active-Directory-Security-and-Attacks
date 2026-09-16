# NTLM Authentication

## Objective

Understand how NTLM authentication works in an Active Directory environment, why it is still relevant, how the challenge-response mechanism works, and why NTLM is important from a cybersecurity perspective.

# 1. What is NTLM?

**NTLM (New Technology LAN Manager)** is a family of Windows authentication protocols that uses a **challenge-response mechanism** to authenticate users and computers.

The purpose of NTLM is to prove that a user knows the password associated with an account without sending the user's plaintext password across the network.

NTLM includes different protocol versions, including NTLMv1 and NTLMv2. Modern Windows environments generally use NTLMv2 when NTLM authentication is required.

# 2. Why is NTLM Important in Active Directory?

Active Directory environments primarily use Kerberos when possible, but NTLM can still be encountered in legacy, compatibility, and certain authentication scenarios.

NTLM is important for cybersecurity because attackers may target the credential material and authentication mechanisms associated with NTLM.

Important security topics related to NTLM include:

* NTLM credential theft
* Pass-the-Hash
* NTLM Relay
* Password attacks
* Lateral movement
* NTLM downgrade/legacy protocol risks

Understanding NTLM is therefore important for both attack detection and defense.


# 3. NTLM Authentication Concept

NTLM uses a **challenge-response mechanism**.

The basic idea is:

Client
   │
   │ Authentication request
   ▼
Server
   │
   │ Challenge
   ▼
Client
   │
   │ Cryptographic response
   ▼
Server
   │
   │ Verification
   ▼
Success / Failure

The client does not simply send the plaintext password to the server.

Instead, the client uses password-derived credential information and the challenge to calculate a response.

# 4. NTLM Authentication Flow

A simplified NTLM authentication flow is:

1. NEGOTIATE
      ↓
2. CHALLENGE
      ↓
3. AUTHENTICATE
      ↓
4. Response validation
      ↓
5. Authentication success/failure

The three major NTLM protocol messages are:

### 1. NEGOTIATE_MESSAGE

The client sends an NTLM negotiation message to indicate supported capabilities and authentication options.

### 2. CHALLENGE_MESSAGE

The server sends a challenge to the client.

The NTLM protocol uses a server-generated challenge/nonce.

### 3. AUTHENTICATE_MESSAGE

The client sends authentication information containing the username and a cryptographic response calculated using the challenge and the required credential material.

Microsoft's NTLM protocol specification documents these NEGOTIATE, CHALLENGE, and AUTHENTICATE messages.

# 5. NTLM Challenge-Response

The simplified concept is:

User Password
      │
      ▼
Password-derived credential material
      │
      +
Server Challenge
      │
      ▼
Cryptographic calculation
      │
      ▼
NTLM Response

The response is then sent to the server.

For a domain account, the server can send the username, challenge, and response to a Domain Controller for validation.

The Domain Controller has the required password-related credential information and independently calculates the expected response.

The two responses are then compared.

Client Response
      │
      │
      ▼
    Compare
      ▲
      │
      │
DC Expected Response

       │
   ┌───┴────┐
   │        │
 Match   No Match
   │        │
   ▼        ▼
Success    Failure

Microsoft documents this domain-account validation process, where the resource server sends the authentication information to a Domain Controller for validation.

# 6. Password and NT Hash

For understanding NTLM, it is important to distinguish the password from its derived credential material.

Conceptually:

Password
   │
   ▼
NT Hash

The NT hash is an important password-derived value used by NTLM.

However, NTLMv2 authentication is more than simply "encrypting the challenge with the NT hash." NTLMv2 derives authentication material from the NT hash and uses additional challenge/context information in its cryptographic calculation.


# 7. Domain Authentication Example

### Lab Environment

Domain:
harish.local

Domain Controller:
DC01

IP:
192.168.182.10

Client:
CLIENT01

Domain User:
harish

Suppose Harish accesses a resource on a Windows server.

CLIENT01
    │
    │ Username + authentication request
    ▼
Resource Server
    │
    │ NTLM Challenge
    ▼
CLIENT01
    │
    │ NTLM Response
    ▼
Resource Server
    │
    │ Authentication information
    ▼
DC01
    │
    │ Validate response
    ▼
Authentication Result

If the response is successfully validated:

Authentication
      ↓
Logon Session
      ↓
Access Token
      ↓
Authorization
      ↓
Resource Access

# 8. NTLM and Kerberos

| NTLM                                                         | Kerberos                                               |
| ------------------------------------------------------------ | ------------------------------------------------------ |
| Challenge-response authentication                            | Ticket-based authentication                            |
| Older Windows authentication family                          | Preferred authentication protocol in AD when available |
| Uses NTLM credential material                                | Uses Kerberos tickets                                  |
| Can be used for compatibility/legacy scenarios               | Designed for AD domain authentication                  |
| Associated with attacks such as Pass-the-Hash and NTLM Relay | Uses TGTs and service tickets                          |

Windows' Negotiate security package can select between Kerberos and NTLM. Microsoft documents that Negotiate normally selects Kerberos unless Kerberos cannot be used or sufficient information is unavailable.

# 9. NTLM Security Risks

NTLM is important from an attacker's perspective because authentication-related credential material can be targeted.

Important attacks and techniques include:

### Pass-the-Hash

An attacker who obtains an NT hash may be able to use that credential material in NTLM authentication without knowing the original plaintext password in certain scenarios.

### NTLM Relay

An attacker may attempt to relay NTLM authentication from one system to another instead of directly knowing the user's password.

### Credential Theft

Attackers may attempt to obtain password hashes or other authentication material from compromised systems.

### Lateral Movement

Compromised authentication material can potentially be used to move from one system to another, depending on permissions and configuration.

These attacks will be studied later in the **Credential Attacks** section of this project.

# 10. Practical Lab

## Lab Objective

Generate and identify NTLM authentication inside the isolated Active Directory lab.

### Environment

DC01
Windows Server 2022
192.168.182.10

CLIENT01
Windows 11

Domain
harish.local


### Initial Test

A controlled SMB resource was created on DC01:

\\DC01\NTLM-Lab

The client was then used to access the resource.

A later dedicated lab will deliberately generate NTLM authentication and investigate the corresponding Windows security events.

# 11. Detection

Windows Security auditing can provide evidence of authentication activity.

One important event is:
Event ID: 4624

Event 4624 indicates that an account was successfully logged on.

For NTLM investigation, the important fields include:

Logon Type
Logon Process
Authentication Package
Package Name (NTLM only)

For an NTLM authentication event:

Authentication Package: NTLM

The `Package Name (NTLM only)` field can identify the NTLM-family protocol version, such as NTLM V1 or NTLM V2.

Microsoft documents these fields in Event ID 4624.

The detailed NTLM authentication investigation will be completed later as part of the dedicated attack and detection labs.

# 13. Detection and Defense

Organizations can reduce NTLM-related security risks by:

* Identifying where NTLM is still being used
* Preferencing Kerberos where possible
* Reducing unnecessary NTLM usage
* Monitoring NTLM authentication events
* Using strong password policies
* Protecting privileged accounts
* Limiting unnecessary administrative privileges
* Implementing SMB signing where appropriate
* Monitoring suspicious lateral movement
* Investigating unexpected NTLM authentication

NTLM usage can be monitored through Windows auditing, including Event ID 4624 and related authentication events.

# 14. Key Takeaways


NTLM
 ↓
Windows authentication protocol family
 ↓
Challenge-Response
 ↓
NEGOTIATE
 ↓
CHALLENGE
 ↓
AUTHENTICATE
 ↓
Response validation
 ↓
Authentication


Remember:

1. NTLM uses challenge-response authentication.
2. The plaintext password is not sent across the network as part of the normal NTLM challenge-response exchange.
3. The server generates a challenge.
4. The client calculates a cryptographic response.
5. For a domain account, the resource server can ask the Domain Controller to validate the response.
6. NTLM uses password-derived credential information.
7. NTLMv2 is more sophisticated than simply encrypting a challenge with the NT hash.
8. Kerberos is preferred when available in an Active Directory environment.
9. NTLM is important for understanding Pass-the-Hash and NTLM Relay.
10. NTLM authentication can be investigated through Windows security auditing.

