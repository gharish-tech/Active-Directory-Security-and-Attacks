# SMB Enumeration

## 1. Overview

Server Message Block (SMB) is a Windows network file-sharing protocol used to provide access to shared files, folders, printers, administrative resources, and other Windows services.

In an Active Directory environment, SMB enumeration can reveal:

- Available SMB services
- Network shares
- Share names and descriptions
- Accessible departmental resources
- Read/write permissions
- Authorization differences between users and departments
- Potentially sensitive resources exposed through SMB

This module demonstrates SMB reconnaissance against the Windows Server 2022 domain controller in the lab.

---

## 2. Lab Environment

| Component | Configuration |
|---|---|
| Attacker Machine | Kali Linux |
| Target | Windows Server 2022 DC01 |
| Domain | `harish.local` |
| DC01 IP | `192.168.182.10` |
| Kali IP | `192.168.182.129` |
| SMB Ports | TCP 139, TCP 445 |
| Test Account | `HARISH\harish` |

---

## 3. Objectives

The objectives of this practical were:

1. Identify SMB services running on the target.
2. Enumerate available SMB shares.
3. Test authenticated access to departmental shares.
4. Determine whether files can be listed and read.
5. Test write access in the authorized HR share.
6. Compare authorization between HR, IT, and FINANCE.
7. Understand the difference between authentication and authorization.

---

# 4. SMB Port Enumeration

## Command

```bash
nmap -p 139,445 -sV 192.168.182.10
```

## Actual Result

```text
Nmap 7.99
Host is up (0.0065s latency).

PORT    STATE SERVICE       VERSION
139/tcp open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds?
MAC Address: 00:0C:29:7C:FA:E0 (VMware)
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
Nmap done: 1 IP address (1 host up) scanned in 8.41 seconds
```

## Interpretation

Two SMB-related TCP ports were identified:

- **TCP 139** — NetBIOS Session Service, traditionally used for SMB over NetBIOS.
- **TCP 445** — Direct SMB over TCP and the primary SMB port in modern Windows environments.

The scan also identified the target as a Windows system.

### Security Relevance

An exposed SMB service gives an attacker an opportunity to investigate:

- Shares
- Authentication
- File permissions
- Windows host information
- Potentially sensitive resources

### Screenshot

`SMB-01-Port-Scan.png`

---

# 5. SMB Share Enumeration

## Command

```bash
smbclient -L //192.168.182.10 -U 'HARISH\harish'
```

After entering the password, the following shares were returned:

```text
Sharename       Type      Comment
---------       ----      -------
ADMIN$          Disk      Remote Admin
C$              Disk      Default share
FINANCE         Disk
HR              Disk
IPC$            IPC       Remote IPC
IT              Disk
NETLOGON        Disk      Logon server share
NTLM-lab        Disk
SYSVOL          Disk      Logon server share
Wallpapers      Disk      Shared wallpaper resources
```

A total of **10 SMB shares** were enumerated.

The command also displayed:

```text
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 192.168.182.10 failed
Unable to connect with SMB1 -- no workgroup available
```

This occurred during the workgroup-listing fallback and did not prevent the successful share enumeration above.

## Important Shares

### HR

Departmental share used for HR resources.

### IT

Departmental share used for IT resources.

### FINANCE

Departmental share used for Finance resources.

### NETLOGON

Standard Active Directory domain share used for logon-related resources and scripts.

### SYSVOL

Standard Active Directory domain share containing domain-related Group Policy and other domain configuration files.

### ADMIN$ and C$

Administrative Windows shares.

### IPC$

Inter-Process Communication share used for Windows network communication and remote operations.

### Security Relevance

Share enumeration provides an attacker with an initial map of resources available through SMB.

However:

> Seeing a share does not automatically mean the authenticated user can read or write its contents.

Actual authorization must be tested separately.

### Screenshot

`SMB-02-Share-Enumeration.png`

---

# 6. HR Share Enumeration

The HR share was tested using the domain account:

```bash
smbclient //192.168.182.10/HR -U 'HARISH\harish'
```

After successful authentication:

```text
smb: \>
```

The directory was enumerated with:

```text
ls
```

## Actual Result

```text
.                                   D        0
..                                  D        0
New Text Document.txt               A        0
```

The share contained one visible file:

```text
New Text Document.txt
```

The file size was:

```text
0 bytes
```

### Interpretation

The `HARISH\harish` account could:

- Authenticate to the HR share.
- List the contents of the HR directory.
- See the existing file.

### Screenshot

`SMB-03-HR-Share-Enumeration.png`

---

# 7. HR File Read Test

The following command was used:

```text
get "New Text Document.txt"
```

## Actual Result

```text
getting file \New Text Document.txt of size 0 as New Text Document.txt
(0.0 KiloBytes/sec) (average 0.0 KiloBytes/sec)
```

## Interpretation

The file download operation succeeded.

Therefore, `HARISH\harish` had **read access** to the file through the HR SMB share.

The file contained no data because its size was 0 bytes.

### Important Concept

The test established an actual permission chain:

```text
Authentication
      ↓
SMB Share Access
      ↓
Directory Listing
      ↓
File Read Access
```

### Screenshot

`SMB-04-File-Read-Access.png`

---

# 8. IT Share Authorization Test

The IT share was tested with:

```bash
smbclient //192.168.182.10/IT -U 'HARISH\harish'
```

After connecting, directory enumeration was attempted using:

```text
ls
```

## Actual Result

```text
NT_STATUS_ACCESS_DENIED listing \*
```

## Interpretation

The account successfully authenticated to the SMB service, but directory listing was denied.

This demonstrates an important distinction:

```text
Authentication ≠ Authorization
```

The user could authenticate, but did not have sufficient permission to enumerate the contents of the IT share.

### Screenshot

`SMB-05-IT-Access-Denied.png`

---

# 9. FINANCE Share Authorization Test

The FINANCE share was tested with:

```bash
smbclient //192.168.182.10/FINANCE -U 'HARISH\harish'
```

Directory enumeration was attempted using:

```text
ls
```

## Actual Result

```text
NT_STATUS_ACCESS_DENIED listing \*
```

## Interpretation

The account could authenticate to the SMB service, but directory listing was denied for the FINANCE share.

This provides another practical example of departmental authorization.

### Screenshot

`SMB-06-FINANCE-Access-Denied.png`

---

# 10. HR Write Permission Test

After confirming read access to HR, write permission was tested.

A temporary file was created locally on Kali:

```bash
echo "SMB permission test" > smb-write-test.txt
```

The file was then uploaded to the HR share:

```bash
smbclient //192.168.182.10/HR -U 'HARISH\harish' \
-c 'put smb-write-test.txt'
```

## Actual Result

```text
putting file smb-write-test.txt as \smb-write-test.txt
(0.2 kB/s) (average 0.2 kB/s)
```

## Interpretation

The upload succeeded.

Therefore, the `HARISH\harish` account had **write access** to the HR share.

At this point, the practical evidence showed:

- Read access — confirmed
- Write access — confirmed

### Screenshot

`SMB-07-HR-Write-Access.png`

---

# 11. Cleanup

The temporary test file was deleted immediately after the write test.

The deletion was performed with:

```bash
smbclient //192.168.182.10/HR -U 'HARISH\harish' \
-c 'del smb-write-test.txt'
```

The HR directory was then checked again:

```bash
smbclient //192.168.182.10/HR -U 'HARISH\harish' -c 'ls'
```

## Final Result

```text
.                                   D        0
..                                  D        0
New Text Document.txt               A        0
```

The temporary `smb-write-test.txt` file was no longer present.

## Interpretation

The cleanup was successfully verified and the lab was returned to its original state.

---

# 12. Permission Comparison

Based on the actual tests performed:

| Share | Authentication | Directory Listing | File Read | File Write |
|---|---:|---:|---:|---:|
| HR | ✅ | ✅ | ✅ | ✅ |
| IT | ✅ | ❌ | Not tested | Not tested |
| FINANCE | ✅ | ❌ | Not tested | Not tested |

The results demonstrate that SMB authentication alone does not determine what a user can access.

Authorization controls determine which resources the authenticated user can actually use.

---

# 13. Authentication vs Authorization

This practical demonstrated the difference clearly.

### Authentication

Authentication answers:

> "Who are you?"

In this lab:

```text
HARISH\harish
```

successfully authenticated against the SMB service.

### Authorization

Authorization answers:

> "What are you allowed to access?"

The results were different depending on the share:

```text
HR       → access allowed
IT       → directory listing denied
FINANCE  → directory listing denied
```

This distinction is fundamental to Active Directory security.

---

# 14. Security Interpretation

From an attacker's perspective, SMB enumeration can provide useful information about:

- Network-accessible resources
- Department names
- Administrative shares
- Domain controller shares
- File-sharing structure
- User-specific permissions
- Misconfigured permissions
- Potentially sensitive files

For example, if a normal employee account unexpectedly had write access to a sensitive departmental share, that could represent a security weakness.

In this lab, the HR account's access was consistent with the user's departmental role, while IT and FINANCE directory enumeration was denied.

---

# 15. SMB Reconnaissance Workflow

The practical workflow used in this lab was:

```text
             DC01
        192.168.182.10
               │
               │
        TCP 139 / 445
               │
               ▼
        SMB Enumeration
               │
               ▼
       Share Enumeration
               │
       ┌───────┼────────┐
       ▼       ▼        ▼
      HR       IT     FINANCE
       │       │        │
       ▼       ▼        ▼
     Allow    Deny     Deny
       │
       ▼
 Directory Listing
       │
       ▼
    File Read
       │
       ▼
   File Write
```

---

# 16. Commands Used

## Discover SMB ports

```bash
nmap -p 139,445 -sV 192.168.182.10
```

## Enumerate shares

```bash
smbclient -L //192.168.182.10 -U 'HARISH\harish'
```

## Connect to a share

```bash
smbclient //192.168.182.10/HR -U 'HARISH\harish'
```

## List files

```text
ls
```

## Download a file

```text
get "New Text Document.txt"
```

## Upload a file

```text
put smb-write-test.txt
```

## Delete a test file

```text
del smb-write-test.txt
```

## Execute a single SMB command

```bash
smbclient //192.168.182.10/HR -U 'HARISH\harish' -c 'ls'
```

---

# 17. Key Takeaways

1. SMB commonly uses TCP 445 in modern Windows environments.
2. TCP 139 is associated with SMB over NetBIOS.
3. `smbclient -L` can enumerate available shares with valid credentials.
4. Seeing a share does not guarantee access to its contents.
5. Authentication and authorization are different security concepts.
6. Share permissions and NTFS permissions both influence file access.
7. HR access was successfully verified for listing, reading, and writing.
8. IT and FINANCE directory enumeration were denied.
9. A temporary write-test file was created only for permission testing.
10. The test file was deleted and cleanup was verified.
11. SMB enumeration is reconnaissance; it is not exploitation by itself.

---

# 18. Screenshots

Recommended screenshots for this module:

```text
Screenshots/
├── SMB-01-Port-Scan.png
├── SMB-02-Share-Enumeration.png
├── SMB-03-HR-Share-Enumeration.png
├── SMB-04-File-Read-Access.png
├── SMB-05-IT-Access-Denied.png
├── SMB-06-FINANCE-Access-Denied.png
└── SMB-07-HR-Write-Access.png
```

Only screenshots from the actual lab execution should be placed in this directory.

---

# 19. Lab Status

**Status: Completed**

The SMB reconnaissance module successfully demonstrated:

- SMB service discovery
- Share enumeration
- Authenticated SMB access
- Directory enumeration
- File read access
- File write access
- Authorization failures
- Permission comparison
- Safe cleanup and verification

No destructive activity was performed during this reconnaissance module.
