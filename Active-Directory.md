# 🛡️ Active Directory — Complete Notes

> Offensive Active Directory — Concepts, Structure & Reference

---

## 📌 Table of Contents

- [What is Active Directory?](#what-is-active-directory)
- [Key Components](#key-components-of-active-directory)
- [Why Active Directory Matters](#why-active-directory-matters)
- [Components / Directory Services](#components-of-active-directory)
- [Fundamental AD Features and Capabilities](#fundamental-ad-features-and-capabilities)
- [Security and Networking Protocols](#security-and-networking-protocols)
- [Active Directory Domain Services (AD DS)](#active-directory-domain-services-ad-ds)
- [AD Data Structures](#ad-data-structures)
- [Active Directory Trust](#active-directory-trust)
- [AD Benefits](#ad-benefits)
- [Ntds.dit](#ntdsdit)
- [Important Network Ports](#important-network-ports)
- [Example Nmap Command](#example-command)
- [Quick AD Structure to Remember](#quick-ad-structure-to-remember)
- [Abbreviations — Quick Revision](#most-important-things-for-revision)

---

## What is Active Directory?

**Active Directory (AD)** is Microsoft's directory and identity management service for Windows domain networks. It was introduced in Windows 2000 and is included with most Windows Server operating systems. AD is used by Microsoft solutions like Exchange Server, SharePoint Server, and third-party applications and services.

AD is a directory service developed by Microsoft to manage Windows-based networks. It provides centralized control over user authentication, authorization, and access to resources within a networked environment.

### 1. A directory service used to manage Windows-based networks

Active Directory is essentially a database that organizes and manages information about network objects (users, computers, and resources). It helps administrators control network security and resource access.

AD enables centralized management of:

- **User accounts** – Create, modify, and delete user accounts across the network.
- **Computer accounts** – Manage device enrollment and permissions.
- **Security policies** – Control user access and system behavior using Group Policy.
- **Network resources** – Manage file shares, printers, and applications.

### 2. A directory service that stores information about network objects

Such as users, computers, and groups — making this information easily accessible to users and administrators for authentication, authorization, and management.

AD stores details about objects in a hierarchical structure:

- **User objects** – Usernames, passwords, groups, email addresses.
- **Computer objects** – Device names, operating system versions, group membership.
- **Group objects** – Collections of users or computers with shared permissions.
- **Organizational Units (OUs)** – Logical containers for organizing objects.
- **Printers and shared folders** – Network resources managed through AD.

Information is stored in a replicated database across all Domain Controllers (DCs), ensuring data consistency and availability. Administrators and users can query AD using **LDAP** (Lightweight Directory Access Protocol) to search for resources.

### 3. A directory service enabling centralized and secure network management

Whether it spans a single building, a city, or multiple locations worldwide.

AD allows a single administrative point of control, even for multi-site networks:

- **Forest** – A collection of one or more domains that share a schema and global catalog.
- **Domain** – A logical boundary for managing users and resources.
- **Trusts** – Allow secure resource sharing between different domains or forests.
- **Sites** – Physical locations represented in AD for efficient replication and authentication.

> ➡️ AD uses **Kerberos authentication** for secure logins and single sign-on (SSO).
>
> ➡️ **Replication** ensures that changes made to objects are synchronized across all domain controllers.

---

## Key Components of Active Directory

| Component | Description |
|---|---|
| **Domain Controller (DC)** | Server that stores and manages the AD database. |
| **Forest** | The top-level AD container that holds multiple domains. |
| **Organizational Unit (OU)** | Logical container used to organize and manage objects within a domain. |
| **Global Catalog** | Stores information about every object in the directory to allow quick searches and authentication across domains. |
| **Group Policy** | A set of rules that control user and computer settings in AD. |
| **Schema** | Defines the structure of AD objects (attributes and types). |
| **LDAP** | Protocol used to access and modify objects in AD. |
| **Kerberos** | Authentication protocol for secure logins. |
| **Query and Index Mechanism** | Enables searching and retrieval of objects and their properties. |
| **Replication Service** | Synchronizes directory data across domain controllers to ensure consistency and availability. |

---

## Why Active Directory Matters

- ✅ Centralized security and access control
- ✅ Simplified user and resource management
- ✅ Scalability for large, complex networks
- ✅ High availability through replication and redundancy

---

## Components of Active Directory

Active Directory is made up of several directory services:

- **Active Directory Domain Services (AD DS)** – The core service used to manage users and resources.
- **Active Directory Lightweight Directory Services (AD LDS)** – A low-overhead version of AD DS for directory-enabled applications.
- **Active Directory Certificate Services (AD CS)** – Issues and manages digital security certificates.
- **Active Directory Federation Services (AD FS)** – Shares identity and access management information across organizations.
- **Active Directory Rights Management Services (AD RMS)** – Controls access permissions to documents, workbooks, and presentations.

---

## Fundamental AD Features and Capabilities

### Schema
Defines the classes of objects and attributes in the directory.

> Example: The Active Directory schema defines objects like `User`, `Group`, `Computer`, and attributes like `name`, `address`, and `telephone number`.

### Global Catalog
Contains detailed information about every object in the directory.

### Query and Index Mechanism
Allows users, administrators, and applications to efficiently find directory information.

### Replication Service
Disseminates directory data across the network.

---

## Security and Networking Protocols

Active Directory integrates with several security and networking protocols:

- **LDAP** – Lightweight Directory Access Protocol
- **DNS** – Domain Name System
- **Kerberos** – Microsoft's version of the Kerberos authentication protocol

---

## Active Directory Domain Services (AD DS)

Active Directory Domain Services is the primary Active Directory service. It is used to authenticate users and control access to network resources. A server running AD DS is called a **domain controller**.

**Key Features:**

- Database of users, groups, services, and resources
- Centralized authentication
- Hierarchical organizational structure
- Single point of access to network resources
- Uses **Kerberos v5** for authentication between server and client
- Non-Windows devices can authenticate using **RADIUS** or **LDAP**

---

## AD Data Structures

Active Directory stores information in a hierarchical structure consisting of:

### Domain
A collection of objects (e.g., users, devices) sharing the same Active Directory database.

```
company.com
```

### Tree
A collection of one or more domains with a contiguous namespace.

```
marketing.company.com
engineering.company.com
```

### Forest
A collection of one or more trees that share a common schema and directory configuration.

> Forest acts as the security boundary.

---

## Active Directory Trust

An Active Directory trust (AD trust) allows users in one domain to authenticate against resources in another.

### Trust Types

| Trust Type | Characteristics | Direction | Authentication Mechanism | Notes |
|---|---|---|---|---|
| **Parent-Child** | Transitive | Two-way | Kerberos V5 or NTLM | Created automatically when a child domain is added. |
| **Tree-Root** | Transitive | Two-way | Kerberos V5 or NTLM | Created automatically when a new tree is added to a forest. |
| **Shortcut** | Transitive | One-way or Two-way | Kerberos V5 or NTLM | Created manually to shorten the trust path for improved authentication times. |
| **Forest** | Transitive | One-way or Two-way | Kerberos V5 or NTLM | Created manually to share resources between AD DS forests. |
| **External** | Non-transitive | One-way | NTLM Only | Created manually to access resources in another forest without a forest trust. |
| **Realm** | Transitive or Non-transitive | One-way or Two-way | Kerberos V5 Only | Created manually to access resources between a non-Windows Kerberos V5 realm and an AD DS domain. |

---

## AD Benefits

Active Directory provides several business and functional benefits:

- **Security** – Improves security by controlling access to network resources.
- **Extensibility** – Organize AD data to align with business structure.
- **Simplicity** – Centralized management of user identities and access.
- **Resiliency** – Supports data replication for high availability.

---

## Ntds.dit

The `Ntds.dit` file is a database that stores Active Directory data, including user objects, groups, and password hashes.

**Location:**

```
%SystemRoot%\ntds\NTDS.DIT
```

**Database engine:**

Extensible Storage Engine (ESE) – Based on the Jet database used by Exchange 5.5 and WINS.

---

## Important Network Ports

Common network ports used in a Windows environment:

| Port | Service | Description |
|---|---|---|
| 53 | DNS | Domain Name System |
| 88 | Kerberos | Authentication |
| 135 | WMI/RPC | Remote Procedure Call |
| 137, 139, 445 | SMB | File sharing and network browsing |
| 389, 636 | LDAP/LDAPS | Directory services |
| 3389 | RDP | Remote Desktop Protocol |
| 5985, 5896 | WinRM | Windows Remote Management |

---

## Example Command

You can use `nmap` to scan for these ports on a network:

```bash
nmap -v -sT -sV -sC -A -p 53,88,135,139,445,389,636,3389,5985,5896 192.168.1.50
```

**Command options:**

- `-v` – Verbose output
- `-sT` – TCP connect scan
- `-sV` – Version detection
- `-sC` – Default scripts
- `-A` – Aggressive scan mode
- `-p` – Specify ports

---

## Quick AD Structure to Remember

```
Forest
│
├── Tree
│   │
│   ├── Domain
│   │   ├── Organizational Units (OUs)
│   │   │   ├── Users
│   │   │   ├── Computers
│   │   │   └── Groups
│   │   │
│   │   └── Resources
│   │
│   └── Child Domain
│
└── Another Tree
```

---

## Most Important Things for Revision

| Abbreviation | Meaning |
|---|---|
| AD | Active Directory |
| AD DS | Active Directory Domain Services |
| DC | Domain Controller |
| OU | Organizational Unit |
| GC | Global Catalog |
| LDAP | Lightweight Directory Access Protocol |
| DNS | Domain Name System |
| AD CS | Active Directory Certificate Services |
| AD FS | Active Directory Federation Services |
| AD LDS | Active Directory Lightweight Directory Services |
| AD RMS | Active Directory Rights Management Services |
| NTDS.dit | Active Directory database |
| Kerberos | Authentication protocol |
| GPO | Group Policy |

---

<p align="center"><sub>📚 Notes compiled for personal study & offensive security learning.</sub></p>
