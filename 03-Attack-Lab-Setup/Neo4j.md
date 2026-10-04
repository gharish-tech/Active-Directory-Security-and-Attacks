# Neo4j

## 1. Overview

Neo4j is a graph database used by BloodHound to store and process relationships between objects in an Active Directory environment.

Active Directory contains many interconnected relationships, such as:

* Users
* Groups
* Computers
* Sessions
* Group memberships
* Permissions
* Administrative relationships

These relationships can be represented naturally as a graph.

In this lab, Neo4j is used as the **graph database component of the BloodHound Community Edition (CE) environment**.

---

## 2. Why BloodHound Uses a Graph Database

Traditional relational databases are useful for structured records, but Active Directory security analysis involves many relationships between different objects.

For example:

```text
User
  |
  | MemberOf
  v
Domain Admins
  |
  | HasAdminRights
  v
Domain Controller
```

A graph database represents these relationships using:

* **Nodes** → objects
* **Relationships/Edges** → connections between objects
* **Properties** → information associated with nodes or relationships

This makes graph-based analysis useful for identifying privilege relationships and potential attack paths.

---

# 3. Neo4j's Role in BloodHound CE

The simplified BloodHound architecture used in this lab is:

```text
                 BloodHound CE
                      |
          +-----------+-----------+
          |                       |
          v                       v
     PostgreSQL                 Neo4j
   Application DB             Graph DB
          |                       |
          |                       |
          +-----------+-----------+
                      |
                      v
              BloodHound Analysis
```

### PostgreSQL

PostgreSQL is used by the BloodHound CE application for application-related data.

### Neo4j

Neo4j provides the graph database functionality required to represent and analyze relationships between Active Directory objects.

### BloodHound

BloodHound provides the application interface through which the collected Active Directory data can be explored and analyzed.

---

# 4. Important: Neo4j Was Not Installed Separately

In this project, Neo4j was **not manually installed** using a standalone Neo4j installation.

Instead, Neo4j was deployed automatically as part of the BloodHound CE Docker environment.

This is important because a common misconception is:

```text
Install Neo4j
        ↓
Install BloodHound
        ↓
Connect them manually
```

That was **not** the approach used in this lab.

The actual deployment was:

```text
BloodHound CLI
      |
      v
Docker Compose
      |
      +---- BloodHound application
      |
      +---- PostgreSQL
      |
      +---- Neo4j
```

---

# 5. Docker-Based Neo4j Deployment

The BloodHound environment was started using:

```bash
./bloodhound-cli containers up
```

The BloodHound CLI used the Docker configuration located under:

```text
/root/.config/bloodhound/
```

The Docker environment then started the required containers.

---

# 6. Neo4j Container

The running Neo4j container was verified using:

```bash
docker ps
```

The container appeared as:

```text
bloodhound-graph-db-1
```

The container used the Neo4j image:

```text
neo4j:4.4
```

The container reported a healthy status during the deployment.

---

# 7. Neo4j Ports in the Lab

The Neo4j container exposed the following ports locally:

```text
7474
7687
```

### Port 7474

Neo4j Browser / HTTP interface.

### Port 7687

Bolt protocol used for communication with Neo4j.

The lab showed the following Docker port mappings:

```text
127.0.0.1:7474 -> 7474
127.0.0.1:7687 -> 7687
```

This means the Neo4j services were bound to the Kali host's loopback interface rather than being directly exposed to the external network.

---

# 8. Why Neo4j Is Important for BloodHound

BloodHound's main purpose is to analyze relationships.

For example, an Active Directory environment may contain:

```text
User
 |
 +---- MemberOf ----> Group
 |
 +---- HasSession --> Computer
 |
 +---- AdminTo -----> Computer
 |
 +---- CanRDP ------> Computer
```

Neo4j stores these relationships as a graph.

BloodHound can then query the graph to determine relationships such as:

```text
User
  ↓
Group Membership
  ↓
Privileged Group
  ↓
Computer
  ↓
Domain Controller
```

This allows security analysts to understand how privileges can move through an environment.

---

# 9. Nodes and Relationships

A simplified example:

```text
          +-------------+
          |    User     |
          |   harish    |
          +-------------+
                 |
              MemberOf
                 |
                 v
          +-------------+
          |     IT      |
          |    Group    |
          +-------------+
                 |
              MemberOf
                 |
                 v
          +-------------+
          | Domain Admin|
          +-------------+
```

In this example:

* `User` is a node.
* `IT Group` is a node.
* `Domain Admins` is a node.
* `MemberOf` represents relationships between nodes.

BloodHound uses this graph structure to visualize and analyze privilege relationships.

---

# 10. Neo4j and Active Directory Data

Neo4j does not independently scan the Active Directory environment.

The data collection process is performed by a BloodHound collector, such as SharpHound.

The workflow used in this lab was:

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
      | SCP
      v
Kali Linux
      |
      | Quick Upload
      v
BloodHound CE
      |
      v
Graph Database
      |
      v
Relationship Analysis
```

Therefore:

> **SharpHound collects the data; BloodHound imports and analyzes it; Neo4j provides the graph database layer.**

---

# 11. Verifying the Neo4j Container

The Neo4j container can be checked with:

```bash
docker ps
```

To specifically search for the graph database container:

```bash
docker ps | grep graph-db
```

Expected container name:

```text
bloodhound-graph-db-1
```

The exact status may change depending on whether the BloodHound environment is currently running.

---

# 12. Checking BloodHound Containers

All BloodHound services can be checked using:

```bash
docker ps
```

The lab deployment contained:

```text
bloodhound-bloodhound-1
bloodhound-app-db-1
bloodhound-graph-db-1
```

Their roles are:

| Container                 | Component     | Purpose              |
| ------------------------- | ------------- | -------------------- |
| `bloodhound-bloodhound-1` | BloodHound CE | Web application      |
| `bloodhound-app-db-1`     | PostgreSQL    | Application database |
| `bloodhound-graph-db-1`   | Neo4j         | Graph database       |

---

# 13. Troubleshooting

## Neo4j Was Not Found as a Separate Installation

Initially, it may appear that Neo4j needs to be installed manually.

However, the BloodHound CE Docker deployment already provides the Neo4j graph database container.

Therefore, a separate:

```bash
sudo apt install neo4j
```

installation was **not performed**.

This avoids maintaining a separate Neo4j installation that could conflict with the BloodHound CE environment.

---

## BloodHound Environment Not Running

If the Neo4j container is not running, check:

```bash
docker ps
```

If the BloodHound containers are stopped, start the environment using:

```bash
./bloodhound-cli containers up
```

Then verify again:

```bash
docker ps
```

---

# 14. Security Considerations

Neo4j contains graph data derived from the Active Directory environment.

The BloodHound CE services in this lab were bound to localhost where applicable.

The lab environment is intended only for authorized security testing.

The collected BloodHound ZIP contains information about the lab Active Directory environment and should not be uploaded to public or unauthorized systems.

---

# 15. Practical Result

The following was successfully verified during the lab:

* [x] Docker was available
* [x] Docker Compose was installed
* [x] BloodHound CLI was installed
* [x] BloodHound CE environment was started
* [x] Neo4j container was automatically deployed
* [x] Neo4j container was healthy
* [x] Neo4j graph database was available to the BloodHound environment
* [x] BloodHound CE was accessed through the browser
* [x] SharpHound data was collected
* [x] BloodHound ZIP was transferred to Kali
* [x] Data was uploaded through BloodHound Quick Upload

---

## Key Takeaway

The important architecture to remember is:

```text
SharpHound
    ↓
Collects AD data
    ↓
BloodHound CE
    ↓
Processes / presents the data
    ↓
Neo4j
    ↓
Stores and queries graph relationships
```

**Neo4j is the graph database layer; it is not the Active Directory collector.**

---

## Related Documentation

* [BloodHound Setup](./BloodHound-Setup.md)
* [BloodHound](../04-Reconnaissance/BloodHound.md)
* [SharpHound](../04-Reconnaissance/SharpHound.md)
* [Lab Prerequisites](./Lab-Prerequisites.md)
* [Kali Linux](./Kali-Linux.md)
