# Lateral Movement (Active Directory)

## Overview

**Lateral movement** is a post-exploitation technique where an attacker moves from one compromised system to other systems within the same network to expand access and escalate privileges.

In **Active Directory (AD)** environments, lateral movement is commonly used to reach:

- File servers
- Application servers
- Domain Controllers
- Backup servers
- Administrator workstations

## Why Lateral Movement Matters

After initial access (phishing, exposed service, weak password, etc.), attackers:

1. Harvest credentials
2. Reuse valid accounts
3. Pivot across the network
4. Escalate privileges
5. Target high-value assets

The ultimate goal is often **Domain Admin** or sensitive data exfiltration.

## Common Lateral Movement Techniques (Windows / AD)

### 1. SMB-Based Movement

Uses Windows file-sharing protocol to authenticate and execute commands remotely.

**Tools often used:**
- CrackMapExec
- PsExec

**Typical method:**
- Authenticate using valid credentials
- Execute commands remotely
- Dump hashes or deploy payloads

### 2. Pass-the-Hash (PtH)

Instead of cracking passwords, attackers reuse NTLM hashes to authenticate.

**Common tools:**
- Mimikatz
- Impacket

### 3. WinRM Remote Execution

Uses Windows Remote Management for command execution.

**Often leveraged through:**
- Evil-WinRM
- CrackMapExec

### 4. RDP Pivoting

Remote Desktop Protocol used with stolen credentials to log into other machines interactively.

### 5. WMI Execution

Windows Management Instrumentation allows remote command execution.

**Used by:**
- Impacket
- PowerShell

### 6. Token Impersonation

Attackers impersonate privileged tokens already present in memory.

**Often combined with:**
- Mimikatz

## Typical Lateral Movement Flow in AD

1. Compromise low-privileged workstation
2. Dump credentials from memory
3. Identify local admin reuse
4. Pivot to another host
5. Escalate privileges
6. Reach Domain Controller
7. Dump NTDS

## Indicators of Lateral Movement (Blue Team View)

- Multiple authentication attempts across many hosts
- Same account logging into many systems quickly
- WinRM usage outside admin baseline
- Remote service creation events
- SMB authentication bursts
- Unusual admin share access (`C$`, `ADMIN$`)

**Windows Event IDs commonly monitored:**

| Event ID | Description |
|----------|-------------|
| 4624 | Logon |
| 4625 | Failed logon |
| 4672 | Special privileges assigned |
| 4688 | Process creation |
| 7045 | Service installed |

## Defensive Controls

- Enforce unique local admin passwords (LAPS)
- Disable NTLM where possible
- Restrict WinRM
- Use tiered admin model
- Enable Credential Guard
- Monitor lateral authentication patterns
- Reduce local admin reuse

## Simple Definition

> Lateral movement is the process of using compromised credentials or access to move from one system to another inside a network.

## Reference: Impacket

**Impacket** is a Python toolkit used in cybersecurity for working with network protocols. It is widely used in penetration testing, red teaming, and Active Directory attacks.

**Protocols it works with:**
- SMB (Server Message Block)
- NTLM authentication
- Kerberos
- LDAP
- MSRPC
- HTTP

**Main uses:**
1. Remote command execution
2. Password hash attacks
3. Active Directory enumeration
4. Lateral movement in networks

---

## Further Topics to Cover

- [ ] Red Team lateral movement workflow
- [ ] Blue Team detection & hunting checklist
- [ ] Lab scenario demonstrating lateral movement in AD
