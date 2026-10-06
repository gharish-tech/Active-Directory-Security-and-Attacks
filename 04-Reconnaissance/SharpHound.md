# SharpHound

## 1. Overview

SharpHound is the official BloodHound data collector for Active Directory environments.

It runs from a Windows system and collects information about Active Directory objects and their relationships.

The collected information can then be imported into BloodHound CE for graph-based security analysis.

In this lab, SharpHound was executed from the Windows 11 `CLIENT01` machine against the controlled `harish.local` Active Directory environment.

---

# 2. Purpose of SharpHound

BloodHound needs Active Directory relationship data before it can build useful graphs.

SharpHound collects this information from the Windows/Active Directory environment.

The simplified workflow is:

```text
Active Directory
       |
       v
    CLIENT01
       |
       | SharpHound
       v
 BloodHound ZIP
       |
       v
    Kali Linux
       |
       v
 BloodHound CE
       |
       v
 Graph Analysis
```

### Key distinction

> **SharpHound = data collector**

> **BloodHound = analysis and visualization platform**

SharpHound does not replace BloodHound. It provides the data that BloodHound analyzes.

---

# 3. Lab Environment

The collection was performed in the controlled lab environment.

| Component         | Details            |
| ----------------- | ------------------ |
| Domain            | `harish.local`     |
| Domain Controller | DC01               |
| DC01 IP           | `192.168.182.10`   |
| Windows Client    | CLIENT01           |
| Attack Machine    | Kali Linux         |
| Kali IP           | `192.168.182.129`  |
| Collector         | SharpHound v2.16.0 |
| Analysis Platform | BloodHound CE      |

---

# 4. Obtaining SharpHound

SharpHound was obtained from the official BloodHound/SpecterOps release.

The BloodHound CE interface provided the collector download options.

The selected collector was:

```text
SharpHound v2.16.0
```

The standard Windows x86 release archive was selected:

```text
sharphound_v2.16.0_windows-x86.zip
```

### Release files encountered

The release contained multiple files, including:

```text
sharphound_v2.16.0+debug_windows_x86.zip
sharphound_v2.16.0_windows-x86.zip
```

The normal release archive was selected rather than the debug build.

---

# 5. Extracting SharpHound

The downloaded ZIP archive was extracted on `CLIENT01`.

The extracted directory contained:

```text
SharpHound.exe
SharpHound.exe.config
SharpHound.pdb
SharpHound.ps1
```

The primary executable used during the lab was:

```text
SharpHound.exe
```

---

# 6. Verifying SharpHound

Before collecting data, the executable was tested using:

```powershell
.\SharpHound.exe --help
```

This displayed the available SharpHound options.

One important option is:

```text
-c
```

or:

```text
--collectionmethods
```

This option determines which types of Active Directory information SharpHound should collect.

---

# 7. Collection Methods

SharpHound provides different collection methods for different types of Active Directory information.

Examples include:

```text
Group
LocalAdmin
Session
Trusts
ACL
ObjectProps
Container
LoggedOn
```

The exact collection behavior depends on the selected methods and the permissions available to the account executing SharpHound.

---

# 8. Running SharpHound

For this lab, the following command was used:

```powershell
.\SharpHound.exe -c All
```

### Meaning

```text
-c
```

specifies the collection methods.

```text
All
```

requests all supported collection methods available to SharpHound in that version.

Therefore:

```powershell
.\SharpHound.exe -c All
```

was used to perform broad collection for the controlled lab environment.

---

# 9. Collection Output

During execution, SharpHound resolved the current Active Directory domain as:

```text
harish.local
```

The collector resolved the requested collection methods and performed Active Directory enumeration.

The output indicated that enumeration completed successfully.

The LDAP communication channel was closed after collection.

SharpHound then finished processing the collected information and saved the resulting data.

---

# 10. Generated BloodHound ZIP

After collection completed, the generated files were checked with:

```powershell
dir *.zip
```

The following BloodHound ZIP file was generated:

```text
20261004071003_BloodHound.zip
```

This file contains the collected BloodHound data.

### Important

The ZIP file was **not extracted or manually modified**.

BloodHound CE can directly ingest the generated ZIP archive.

---

# 11. Why the ZIP Was Not Extracted

The generated ZIP is the expected BloodHound data format.

The workflow is:

```text
SharpHound
    ↓
BloodHound ZIP
    ↓
Transfer
    ↓
BloodHound CE
    ↓
Import
```

There was no need to manually extract the archive before uploading it.

Keeping the original archive also preserves the collector's output as generated.

---

# 12. Transferring the ZIP to Kali

The generated ZIP needed to be transferred from `CLIENT01` to the Kali attack machine.

Kali's lab IP address was:

```text
192.168.182.129
```

The Windows built-in `scp` command was used.

From the SharpHound output directory on `CLIENT01`:

```powershell
scp .\20261004071003_BloodHound.zip kali@192.168.182.129:/home/kali/
```

The command transfers the file to the Kali user's home directory.

---

# 13. Why SCP Was Used

SCP was chosen because it provides a direct transfer between the two lab machines.

```text
CLIENT01
192.168.182.x
     |
     | SCP
     v
Kali
192.168.182.129
```

This avoided:

* Uploading the file to a public website
* Using external file-sharing services
* Configuring VMware Shared Folders
* Installing Python on CLIENT01

The transfer remained inside the controlled lab network.

---

# 14. Verifying the Transfer on Kali

After the transfer, the file was verified on Kali using:

```bash
ls -lh ~/20261004071003_BloodHound.zip
```

The ZIP file was successfully received in the Kali home directory.

---

# 15. Uploading the Data to BloodHound CE

BloodHound CE was already running on Kali.

The BloodHound CE web interface was opened at:

```text
http://127.0.0.1:8080
```

The **Quick Upload** feature was used.

The following file was selected:

```text
20261004071003_BloodHound.zip
```

The upload completed successfully.

The collected Active Directory information was then available for BloodHound analysis.

---

# 16. Complete SharpHound Workflow

The actual workflow performed in this project was:

```text
              Active Directory
                 harish.local
                      |
                      v
                 CLIENT01
                      |
                      |
             SharpHound v2.16.0
                      |
                      |
                -c All
                      |
                      v
       20261004071003_BloodHound.zip
                      |
                      | SCP
                      v
               Kali Linux
                      |
                      v
             BloodHound CE
                      |
                 Quick Upload
                      |
                      v
              Graph Database
                      |
                      v
             Security Analysis
```

---

# 17. What SharpHound Collected

Depending on the collection methods and available permissions, SharpHound can collect information about relationships such as:

* Domain users
* Groups
* Group membership
* Computers
* Local administrator relationships
* Sessions
* Active Directory permissions
* Object properties
* Domain trusts
* Other relationships used by BloodHound

The collected information allows BloodHound to construct a graph representing the Active Directory environment.

---

# 18. Security Relevance

SharpHound is useful to both:

### Attackers / Red Teams

It can help identify:

* Privileged users
* Privileged groups
* Administrative access
* Permission relationships
* Potential attack paths

### Defenders / Blue Teams

The same information can be used to identify:

* Excessive privileges
* Dangerous group memberships
* Unnecessary administrative access
* Weak delegation configurations
* Paths leading toward privileged accounts

Therefore, understanding SharpHound is useful for both offensive and defensive Active Directory security.

---

# 19. Screenshots

Screenshots should contain only actual screenshots captured during the lab.

Recommended screenshots:

```text
04-Reconnaissance/Screenshots/
```

Suggested names:

```text
01-SharpHound-Download.png
02-SharpHound-Files.png
03-SharpHound-Help.png
04-SharpHound-Collection.png
05-BloodHound-ZIP.png
06-SCP-Transfer.png
07-BloodHound-Quick-Upload.png
```

Only add screenshots that were actually captured.

Do not create or claim screenshots that were not taken.

---

# 20. Troubleshooting

## SharpHound executable does not run

Verify that the executable exists:

```powershell
dir SharpHound.exe
```

Then check the available options:

```powershell
.\SharpHound.exe --help
```

---

## No ZIP file appears

Check the current directory:

```powershell
dir *.zip
```

Also review the SharpHound console output for errors or permission-related problems.

---

## SCP command not recognized

Verify that OpenSSH Client is available on Windows:

```powershell
scp
```

If the command is unavailable, Windows OpenSSH Client can be installed through Windows Optional Features.

---

## BloodHound does not show the imported data

First verify that the ZIP exists on Kali:

```bash
ls -lh ~/20261004071003_BloodHound.zip
```

Then verify that the BloodHound containers are running:

```bash
docker ps
```

The BloodHound environment should contain the relevant application, PostgreSQL, and Neo4j containers.

---

# 21. Lab Results

The following steps were successfully completed in this project:

* [x] SharpHound v2.16.0 obtained
* [x] SharpHound extracted on CLIENT01
* [x] `SharpHound.exe --help` verified
* [x] Active Directory domain resolved as `harish.local`
* [x] SharpHound executed with `-c All`
* [x] Active Directory enumeration completed
* [x] BloodHound ZIP generated
* [x] `20261004071003_BloodHound.zip` identified
* [x] ZIP transferred from CLIENT01 to Kali using SCP
* [x] ZIP verified on Kali
* [x] ZIP uploaded through BloodHound CE Quick Upload
* [x] BloodHound data made available for graph analysis

---

# 22. Key Takeaways

### SharpHound

Collects Active Directory information.

### BloodHound

Visualizes and analyzes the collected information.

### Neo4j

Provides the graph database layer used by BloodHound.

The complete relationship is:

```text
SharpHound
   ↓
Collects AD relationships
   ↓
BloodHound ZIP
   ↓
BloodHound CE
   ↓
Neo4j graph database
   ↓
Graph-based security analysis
```

---

## Related Documentation

* [BloodHound Setup](../03-Attack-Lab-Setup/BloodHound-Setup.md)
* [Neo4j](../03-Attack-Lab-Setup/Neo4j.md)
* [BloodHound Reconnaissance](./BloodHound.md)
* [Lab Prerequisites](../03-Attack-Lab-Setup/Lab-Prerequisites.md)
