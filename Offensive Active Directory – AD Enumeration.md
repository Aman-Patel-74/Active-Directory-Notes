# Offensive Active Directory – AD Enumeration

Part of the **Offensive-Active-Directory** notes series.

---

## 1. LDAP Enumeration (Lightweight Directory Access Protocol)

LDAP is used to query Active Directory for users, groups, computers, and policies.

### Anonymous Bind

```bash
ldapsearch -x -H ldap://192.168.2.100 -D '' -w '' -b "DC=infosecwarrior,DC=local"
```

### Authenticated Bind

```bash
ldapsearch -x -H ldap://192.168.2.100 -D 'jdoe@infosecwarrior.local' -w 'Password123!' -b "DC=infosecwarrior,DC=local"
```

### Key Enumeration Targets

- Users (`objectClass=user`)
- Groups (`objectClass=group`)
- Computers (`objectClass=computer`)
- Domain Controllers
- Password policies

---

## 2. SMB & NetBIOS Enumeration

### Enum4linux

```bash
enum4linux -a 192.168.2.100
```

### Enum4linux-ng

```bash
enum4linux-ng -A 192.168.2.100
```

### Data to Extract

- Usernames
- Shares
- Password policy
- Domain information

---

## 3. BloodHound – AD Attack Path Mapping

**BloodHound** helps visualize relationships in Active Directory.

### Workflow

1. Collect data (SharpHound / BloodHound-python)
2. Import into BloodHound UI
3. Analyze attack paths

### Key Findings

- Shortest path to Domain Admin
- Privilege escalation paths
- ACL abuse opportunities

---

## 4. Access Control Lists (ACLs)

### Types

- **DACL** (Discretionary ACL): Defines permissions on objects
- **SACL** (System ACL): Defines auditing and logging

### Common Abuse Cases

- GenericAll (full control)
- WriteOwner / WriteDACL
- Add user to privileged group

---

## 5. SMB Password & Share Abuse

### Common Issues

- Weak credentials
- Password reuse
- Writable shares for payload placement

---

## 6. Kerberoasting Attack

Uses Service Principal Names (SPNs) to request Kerberos service tickets.

### Tool

**Impacket – GetUserSPNs**

```bash
GetUserSPNs.py infosecwarrior.local/jdoe:Password123! -dc-ip 192.168.2.100 -request
```

### Objective

- Extract TGS hashes for offline cracking

---

## 7. AS-REP Roasting Attack

Targets users without Kerberos pre-authentication enabled.

### Tool

**Impacket – GetNPUsers.py**

```bash
GetNPUsers.py infosecwarrior.local/ -usersfile users.txt -dc-ip 192.168.2.100
```

### Objective

- Retrieve AS-REP hashes for offline cracking

---

## 8. Mimikatz Usage and Execution

**Mimikatz**

### Capabilities

- Dump credentials from memory
- Extract NTLM hashes
- Manipulate Kerberos tickets

### Kerberos Attacks

**Kerberoasting (via memory)**
- Extract service tickets directly

**Golden Ticket Attack**
- Forge Ticket Granting Ticket (TGT) using KRBTGT hash
- Provides persistent domain-level access

**Silver Ticket Attack**
- Forge service-specific tickets (TGS)
- Access services without contacting Domain Controller

---

## 9. LLMNR / NBT-NS Poisoning

### Concept

- Exploits fallback name resolution mechanisms
- Captures NTLM authentication hashes

### Tools

- Responder
- Inveigh

### Outcome

- Captured hashes can be relayed or cracked

---

## 10. SMB Relay Attack

### Concept

- Relays captured NTLM authentication to another system

### Tool

- ntlmrelayx (from Impacket)

### Targets

- SMB
- LDAP
- HTTP

### Requirements

- SMB signing disabled or not enforced

---

## Attack Flow Summary

```
Recon → Enumeration → Credential Access → Lateral Movement → Privilege Escalation
```

### Typical Attack Chain

1. LLMNR/NBT-NS poisoning to capture hashes
2. SMB relay to gain initial access
3. LDAP and SMB enumeration
4. BloodHound to identify attack paths
5. Kerberoasting or AS-REP roasting
6. Credential dumping with Mimikatz
7. Golden Ticket for persistence

---

## Notes

- **Always verify:**
  - SMB signing status
  - LDAP anonymous bind
  - Kerberos configuration
- Combine multiple tools for effective results
- Use BloodHound to map relationships and attack paths
- Misconfigurations are key entry points
