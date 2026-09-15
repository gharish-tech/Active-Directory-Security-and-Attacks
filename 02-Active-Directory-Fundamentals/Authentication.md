# Authentication in Active Directory

## 1. Objective

To understand how Active Directory verifies the identity of users and computers before allowing them to access domain resources.


## 2. What is Authentication?

Authentication is the process of **verifying the identity of a user, computer, or service** before granting access.

In simple terms:

**Authentication answers: "Who are you?"**

For example, when a domain user logs in to CLIENT01 using domain credentials, Windows verifies the user's identity before creating a logon session.

## 3. Authentication in Active Directory

In an Active Directory domain environment, the **Domain Controller (DC)** provides centralized authentication for domain accounts.

Example lab:

Domain       : harish.local
Domain Controller : DC01
Client       : CLIENT01
User         : Harish

When Harish logs in to CLIENT01 using a domain account, the client communicates with the Domain Controller to authenticate the account.


## 4. Authentication Flow

Harish
   │
   │ Domain Credentials
   ▼
CLIENT01
   │
   │ Authentication Request
   ▼
DC01
   │
   ├── Verify Account
   ├── Verify Credentials
   ├── Check Account Status
   ├── Check Account Lockout
   └── Check Password Status
   │
   ▼
Authentication Result
   │
   ├── Success → Logon
   │
   └── Failure → Access Denied

The exact authentication exchange depends on the authentication protocol being used. In Active Directory, Kerberos is the normal domain authentication protocol, while NTLM can be used in certain fallback or compatibility scenarios.

## 5. What Happens After Successful Authentication?

Successful authentication does not automatically mean the user can access every resource.

After authentication, Windows establishes a logon session and creates an **access token** containing security information associated with the user, including the user's SID and group SIDs.

Windows can then use this information during authorization to determine what resources the user is allowed to access.

Authentication
      ↓
Identity Verified
      ↓
Logon Session
      ↓
Access Token
      ↓
Authorization
      ↓
Access Allowed / Denied

## 6. Authentication vs Authorization

Authentication:
-----------------------------------------
Verifies identity
Answers "Who are you?"
Happens before authorization
Example: Validating domain credentials

Autheroization:
------------------------------------------
Determines access                                    
Answers "What can you access?"                       
Uses the authenticated identity/security information 
Example: Allowing access to an HR folder             

## 7. Authentication Protocols

Active Directory environments commonly involve two important authentication protocols:

### Kerberos

Kerberos is the primary authentication protocol used by modern Active Directory domains.

It uses tickets to authenticate users to services without repeatedly sending the user's password across the network.

### NTLM

NTLM is an older Microsoft authentication protocol that can still appear in environments where Kerberos is unavailable or for compatibility reasons.

NTLM uses a challenge-response mechanism.

# 8. Practical Lab

## Lab Environment

Domain       : harish.local
Domain Controller : DC01
Client       : CLIENT01

### Step 1 — Log in using a Domain Account

Log in to CLIENT01 using a domain account.

Example:
harish.local\harish

After successful authentication, the user receives a Windows session.

### Step 2 — Verify the Logged-in Identity

Open Command Prompt and run:

whoami

Example output:

harish\harish

This confirms the security principal associated with the current logon session.

### Step 3 — View the User SID

Run:(IN CMD)

whoami /user

This displays the current user's Security Identifier (SID).

Example:

USER INFORMATION

User Name       SID
harish\harish   S-1-5-21-XXXXXXXXXX

### Step 4 — View Group Membership

Run:(IN CMD)

whoami /groups

This displays the security groups associated with the current user session.

For example:

Domain Users
HR_GG

The exact groups depend on the configuration of the lab.

# 9. What This Practical Demonstrates

The practical confirms that:

1. The domain account can successfully authenticate.
2. Windows identifies the logged-in security principal.
3. The user has a unique SID.
4. The user's group memberships are available to Windows for authorization decisions.

# 10. Security Relevance

Authentication is a major security boundary in Active Directory.

Attackers may attempt to obtain or abuse authentication material such as:

* Passwords
* NTLM hashes
* Kerberos tickets
* Authentication tokens

Examples of attacks related to authentication include:

* Password Spraying
* Kerberoasting
* AS-REP Roasting
* Pass-the-Hash
* Pass-the-Ticket

Understanding how legitimate authentication works is therefore essential before studying these attacks.


# 11. Key Takeaways

* Authentication verifies identity.
* Domain authentication is centrally managed through Active Directory.
* The Domain Controller plays a central role in authenticating domain identities.
* Authentication and authorization are different processes.
* Successful authentication establishes a logon session.
* Windows uses security information such as SIDs and group membership during authorization.
* Kerberos is the primary authentication protocol in modern Active Directory environments.
* NTLM remains relevant for compatibility and fallback scenarios.
* Understanding authentication is essential for understanding many Active Directory attacks.

# 12. Lab Status

**Status:**  Completed

**Environment:** Windows Server 2022 + Windows 11 + Active Directory

**Domain:** `harish.local`
