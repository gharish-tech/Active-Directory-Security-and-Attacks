# BloodHound Setup

## 1. Overview

BloodHound is a security analysis tool used to visualize relationships and privileges in Active Directory environments.

In this lab, BloodHound Community Edition (CE) is deployed on Kali Linux and used to analyze the `harish.local` Active Directory environment.

### Lab Components

| Component            | Details                      |
| -------------------- | ---------------------------- |
| Attack Machine       | Kali Linux                   |
| BloodHound           | BloodHound Community Edition |
| BloodHound CLI       | v0.2.1                       |
| Graph Database       | Neo4j                        |
| Application Database | PostgreSQL                   |
| Domain Controller    | DC01                         |
| Domain               | `harish.local`               |
| Client               | Windows 11 CLIENT01          |
| Kali IP              | `192.168.182.129`            |
| DC01 IP              | `192.168.182.10`             |

---

## 2. Why BloodHound Is Used

Active Directory contains many relationships between:

* Users
* Groups
* Computers
* Sessions
* Permissions
* Group memberships
* Administrative privileges
* Domain objects

Manually identifying all these relationships can be difficult.

BloodHound represents these relationships as a graph, making it easier to identify:

* Privileged accounts
* Group memberships
* Administrative relationships
* Attack paths
* Potential privilege escalation paths

In this project, BloodHound is used only against the controlled `harish.local` lab environment.

---

## 3. BloodHound CE Architecture

The BloodHound CE deployment used in this lab contains multiple services:

```text
                    Kali Linux
                        |
                        |
                BloodHound CE
                        |
          +-------------+-------------+
          |             |             |
          v             v             v
      BloodHound    PostgreSQL      Neo4j
       Web App     Application DB   Graph DB
          |
          |
          v
    Active Directory
       harish.local
```

### Important Note About Neo4j

Neo4j was **not manually installed as a separate application** in this lab.

The BloodHound CE Docker environment automatically deployed the required Neo4j container along with the other BloodHound services.

---

# 4. Prerequisites

Before deploying BloodHound, the following were available on Kali Linux:

* Kali Linux Rolling
* Docker
* Internet connectivity
* Network connectivity to the AD lab
* DNS resolution for `harish.local`

Docker was already installed on the system.

The Docker version verified during the lab was:

```text
Docker version 28.5.2+dfsg4
```

---

# 5. Installing Docker Compose

Initially, the `docker compose` command was not available.

The following package installation was attempted:

```bash
sudo apt install docker-compose-v2
```

The package was not available in the configured Kali repositories.

The available Docker Compose package was then installed:

```bash
sudo apt install docker-compose
```

The installation was verified with:

```bash
docker-compose version
```

Verified version:

```text
Docker Compose version 2.40.3-3
```

Docker Compose is required because the BloodHound CE environment consists of multiple interconnected containers.

---

# 6. Installing BloodHound CLI

BloodHound CLI was downloaded from the official SpecterOps BloodHound CLI release.

After extracting the downloaded archive, the CLI was verified with:

```bash
./bloodhound-cli version
```

The installed version was:

```text
BloodHound CLI v0.2.1 (23 Apr 2026)
```

The CLI also created the required BloodHound configuration directory:

```text
/root/.config/bloodhound
```

This directory contains the configuration used by the BloodHound CLI.

---

# 7. Starting BloodHound CE

The BloodHound environment was started using the BloodHound CLI.

Command:

```bash
./bloodhound-cli containers up
```

The CLI checked Docker and Docker Compose before starting the environment.

The following BloodHound containers were started:

```text
bloodhound-bloodhound-1
bloodhound-app-db-1
bloodhound-graph-db-1
```

The services reached a healthy/running state.

---

# 8. Verifying Docker Containers

The running containers were verified with:

```bash
docker ps
```

The BloodHound environment contained:

```text
bloodhound-bloodhound-1
bloodhound-app-db-1
bloodhound-graph-db-1
```

The containers represent:

| Container                 | Purpose                         |
| ------------------------- | ------------------------------- |
| `bloodhound-bloodhound-1` | BloodHound application          |
| `bloodhound-app-db-1`     | PostgreSQL application database |
| `bloodhound-graph-db-1`   | Neo4j graph database            |

Existing Docker containers used for another lab application were not modified.

---

# 9. Accessing BloodHound CE

BloodHound CE was accessed locally from the Kali browser.

URL:

```text
http://127.0.0.1:8080
```

The BloodHound CE login page was displayed.

After authentication, the BloodHound CE dashboard was accessible.

---

# 10. BloodHound CE Dashboard

The BloodHound CE dashboard provides access to the application's graph-based Active Directory analysis functionality.

The lab used the **Quick Upload** functionality to import the data collected from CLIENT01.

### Screenshot

Add the actual screenshot captured from the lab:

```text
03-Attack-Lab-Setup/Screenshots/BloodHound-CE-Dashboard.png
```

---

# 11. BloodHound Data Collection Workflow

BloodHound itself does not directly collect all Active Directory information from CLIENT01.

A collector called **SharpHound** was used to collect Active Directory data.

The workflow used in this project was:

```text
CLIENT01
   |
   | SharpHound
   | -c All
   v
BloodHound ZIP File
   |
   | SCP transfer
   v
Kali Linux
   |
   | Quick Upload
   v
BloodHound CE
   |
   v
Graph Analysis
```

The SharpHound collection process is documented separately in:

```text
04-Reconnaissance/SharpHound.md
```

---

# 12. Transferring BloodHound Data to Kali

After SharpHound completed collection on CLIENT01, the following ZIP file was generated:

```text
20261004071003_BloodHound.zip
```

The ZIP file was transferred from CLIENT01 to Kali using the built-in Windows `scp` command.

Example:

```powershell
scp .\20261004071003_BloodHound.zip kali@192.168.182.129:/home/kali/
```

The file was then verified on Kali:

```bash
ls -lh ~/20261004071003_BloodHound.zip
```

### Why SCP Was Used

SCP provided a direct transfer between the Windows client and the Kali attack machine over the isolated lab network.

No external file-sharing service was required.

---

# 13. Uploading Data into BloodHound CE

After transferring the ZIP file to Kali, BloodHound CE was opened in the browser.

The **Quick Upload** functionality was used.

The following file was selected:

```text
20261004071003_BloodHound.zip
```

The data was successfully uploaded into BloodHound CE.

### Important

The SharpHound ZIP file was uploaded directly.

It was **not extracted or modified** before uploading.

---

# 14. Initial BloodHound Reconnaissance

After the data was uploaded, the BloodHound **Explore** functionality was opened.

The initial reconnaissance task was to investigate privileged Active Directory relationships.

The first object examined was:

```text
Domain Admins
```

This begins the graph-analysis portion of the project.

The detailed analysis of BloodHound relationships and attack paths will be documented in:

```text
04-Reconnaissance/BloodHound.md
```

---

# 15. Troubleshooting Notes

## `docker compose` command not found

Initial command:

```bash
docker compose version
```

The command was not available.

The required Compose package was installed using:

```bash
sudo apt install docker-compose
```

---

## Invalid BloodHound CLI command

The following command was initially attempted:

```bash
./bloodhound-cli containers status
```

The CLI did not provide a `status` command.

Available container operations included:

```text
build
down
restart
start
stop
up
```

The correct command used to start the environment was:

```bash
./bloodhound-cli containers up
```

---

## Neo4j was not installed separately

No separate Neo4j installation was performed.

Neo4j was deployed automatically as part of the BloodHound CE Docker environment.

The running Neo4j container was:

```text
bloodhound-graph-db-1
```

---

# 16. Security Considerations

This BloodHound environment is intended for a controlled Active Directory security lab.

The environment contains intentionally created test systems and accounts.

All reconnaissance and security testing in this project should remain within systems that are owned or explicitly authorized for testing.

---

# 17. Current Lab Status

At this stage:

* [x] Docker verified
* [x] Docker Compose installed
* [x] BloodHound CLI installed
* [x] BloodHound CE deployed
* [x] PostgreSQL container running
* [x] Neo4j container running
* [x] BloodHound application running
* [x] BloodHound CE accessed through browser
* [x] SharpHound data collected
* [x] BloodHound ZIP transferred to Kali
* [x] BloodHound data uploaded
* [x] BloodHound Explore opened
* [ ] Detailed BloodHound reconnaissance
* [ ] Attack-path analysis
* [ ] Detection and defense mapping

---

## Related Documentation

* [Lab Prerequisites](../03-Attack-Lab-Setup/Lab-Prerequisites.md)
* [Kali Linux](../03-Attack-Lab-Setup/Kali-Linux.md)
* [Neo4j](../03-Attack-Lab-Setup/Neo4j.md)
* [SharpHound](../04-Reconnaissance/SharpHound.md)
* [BloodHound Reconnaissance](../04-Reconnaissance/BloodHound.md)
