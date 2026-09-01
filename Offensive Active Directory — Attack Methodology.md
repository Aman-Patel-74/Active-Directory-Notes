# Offensive Active Directory — Attack Methodology

Notes on the general attack lifecycle used when assessing/pentesting an Active Directory environment, plus a lab network reference and an example recon finding.

---

## Attack Stages

### 1. 🕵️ Recon (Reconnaissance)
Gathering intelligence about the target network, such as:
- Open ports (using tools like `Nmap`)
- Services (e.g., SMB, LDAP, RDP)
- Domain information (DNS, AD structure)
- Employee details (OSINT via LinkedIn, social media)
- Technology stack (operating systems, software versions)

### 2. 🎯 Domain Enum (Enumeration)
Actively querying the target environment to identify:
- Domain Controllers (e.g., using `nltest`)        
- Active Directory objects (users, groups, permissions)
- Trust relationships (e.g., `BloodHound`, `PowerView`)
- Group Policy Objects (GPO)

### 3. 🚀 Local Privilege Escalation
Exploiting local vulnerabilities to gain higher privileges on a compromised machine.

Common techniques:
- **Exploiting Misconfigurations** (e.g., weak service permissions)
- **Kernel Exploits** (e.g., token stealing, memory corruption)
- **Stealing credentials from memory** (e.g., `mimikatz`)
- **DLL Hijacking**

### 4. 🏆 Admin Recon (Administrative Reconnaissance)
Identifying high-value targets like:
- Domain admins
- Service accounts (especially those with SPNs)
- Group Policy Objects (GPO)

Mapping out:
- Administrative control paths
- Privilege escalation routes
- Trust relationships

### 5. 🔀 Lateral Movement
Expanding control across the network by:
- Exploiting SMB, RDP, and WMI
- Reusing credentials from other compromised systems
- Abusing network shares and administrative tools (`PsExec`, `wmic`)
- Abusing Active Directory Replication for stealth

### 6. 👑 Domain Admin Privs
Obtaining domain administrator-level access using techniques like:
- **Kerberoasting** — Cracking service account hashes
- **Pass-the-Hash** — Reusing NTLM hashes to authenticate
- **DCSync** — Replicating the Active Directory database to extract credentials
- **Golden/Silver Ticket attacks** — Forging Kerberos tickets for persistence

### 7. 🌐 Cross Trust Attacks
Moving between different trusted domains or forests using:
- **SID history abuse** — Exploiting historical SIDs to escalate privileges
- **Trust relationship exploitation** — Leveraging misconfigured domain trusts
- **Kerberos delegation abuse** — Impersonating trusted accounts across domains

### 8. 🛡️ Persist and Exfiltrate
Establishing persistence through:
- Creating backdoors (e.g., scheduled tasks, startup items)
- Registry key manipulation
- Modifying Group Policy

Exfiltrating sensitive data using:
- Encrypted channels (e.g., HTTPS, DNS tunneling)
- Cloud services (e.g., Dropbox, Google Drive)
- Command and Control (C2) channels

---

## 🔥 The Cycle

Once attackers gain administrative control, they tend to repeat the loop of:

> **Recon → Privilege Escalation → Lateral Movement → Persistence**
> — until they've achieved their objective or are detected.

---

## Lab Network Reference — AD Network

Attacker and target layout used for practicing the above methodology.

| Host  | Role | Interface(s) / IP | Subnet Mask | DNS |
|-------|------|--------------------|-------------|-----|
| Kali  | Attacker | 192.168.1.7 | 255.255.255.0 | 8.8.8.8 (GW: 192.168.1.1) |
| WC01  | Workstation | BA: 192.168.1.101 / IN: 192.168.2.101 | 255.255.255.0 | 192.168.2.100 |
| LC01  | Workstation | BA: 192.168.1.103 / IN: 192.168.2.103 | 255.255.255.0 | 192.168.2.100 |
| WC02  | Workstation | IN: 192.168.2.102 | 255.255.255.0 | 192.168.2.100 |
| DC    | Domain Controller | IN: 192.168.2.100 | 255.255.255.0 | 127.0.0.1 |

**Topology:** `Kali → Switch → (WC01, LC01) → Switch → (WC02, DC)`
Kali sits on the 192.168.1.0/24 (BA) segment and pivots through WC01/LC01 into the internal 192.168.2.0/24 (IN) segment where WC02 and the DC live.

---

## Example Recon Finding — `nmap` Scan of WC01

```
rdp-ntlm-info:
  Target_Name:          INFOWARRIOR
  NetBIOS_Domain_Name:  INFOWARRIOR
  NetBIOS_Computer_Name: WC01
  DNS_Domain_Name:      infosecwarrior.local
  DNS_Computer_Name:    WC01.infosecwarrior.local
  Product_Version:      10.0.26100
  System_Time:          2026-02-24T10:17:20+00:00
```

**Open ports of interest:**

| Port | Proto | State | Service | Notes |
|------|-------|-------|---------|-------|
| 5040 | tcp | open | unknown | — |
| 5985 | tcp | open | http | Microsoft HTTPAPI httpd 2.0 — **WinRM** |
| 7680 | tcp | open | pando-pub? | — |
| 47001 | tcp | open | http | Microsoft HTTPAPI httpd 2.0 |
| 49664–49671 | tcp | open | msrpc | Microsoft Windows RPC (dynamic RPC range) |

- Domain identified: `infosecwarrior.local` (NetBIOS: `INFOWARRIOR`)
- Target OS: Windows (build 10.0.26100)
- **Port 5985 open → WinRM is reachable**, worth checking for remote management access if valid credentials are obtained.

---

## Summary

The methodology above follows a standard offensive AD kill-chain: start from external recon, enumerate the domain, escalate locally, identify high-value admin targets, move laterally, obtain domain admin, abuse cross-forest trusts if present, then persist and exfiltrate — repeating the cycle until the objective is met or the operation is detected.
