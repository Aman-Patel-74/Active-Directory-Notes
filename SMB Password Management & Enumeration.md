# 🔐 SMB Password Management & Enumeration

> **Category:** Offensive Active Directory → Domain Enumeration
> **Tags:** `SMB` `smbclient` `smbpasswd` `Impacket` `Active Directory`

---

## 🎯 Target

**Lab:** Hack The Box — [Fuse](https://www.hackthebox.com/)

---

## 📖 Overview

This section outlines various tools and techniques to interact with SMB shares, reset user passwords remotely, and conduct enumeration. These tools are valuable for both administrators and penetration testers.

---

## 1️⃣ Scan SMB Services with Nmap

```bash
nmap -v -sT -sV -sC -A -p 135,139,445 192.168.1.71
```

Performs a detailed scan of SMB-related services:

| Flag | Meaning |
|---|---|
| `-sT` | TCP Connect scan |
| `-sV` | Service & version detection |
| `-sC` | Default NSE scripts |
| `-A` | OS detection, traceroute, scripts |

**Ports:**

| Port | Service |
|:---:|---|
| `135` | RPC |
| `139` | NetBIOS |
| `445` | SMB |

---

## 2️⃣ Enumerate SMB Shares with `smbclient`

### List Available Shares

```bash
smbclient -L //192.168.1.71 -U u1
```

```bash
smbclient -L //192.168.1.50 -U 'ankit.sharma'
```

```bash
smbclient -L //192.168.10.101 -U jdoe@infosecwarrior.local
```

```bash
smbclient -L //192.168.10.101 -U infosecwarrior.local\jdoe
```

**Notes:**

- `-L` → Lists shares on the target
- Prompts for password
- Useful for:
  - Anonymous access checks
  - Credential validation
  - Share discovery

---

## 3️⃣ `smbpasswd` Usage

### View Help

```bash
smbpasswd -h
```

Displays available options and usage details.

### Change SMB Password (Remote)

```bash
smbpasswd -r 192.168.1.71 -U u1
```

```bash
smbpasswd -r 192.168.1.71 -U Win7
```

```bash
smbpasswd -r 192.168.1.50 -U 'ankit.sharma'
```

**Flags:**

| Flag | Meaning |
|---|---|
| `-r` | Target remote SMB server |
| `-U` | Username |

**Use Cases:**

- Reset compromised credentials
- Test weak password policies
- Validate password change permissions

### Enable Debug Mode

```bash
smbpasswd -D 3 -r 192.168.1.50 -U 'ankit.sharma'
```

- `-D 3` → Debug level 3 (verbose output)
- Helps troubleshoot:
  - Authentication failures
  - Network issues
  - SMB negotiation problems

---

## 4️⃣ Remote Command Execution (Impacket `psexec`)

```bash
impacket-psexec armourinfosec.local/ankit.sharma:'@rmour123'@192.168.1.50
```

Executes commands on the remote system using SMB.

**Requirements:**

- Valid credentials
- Administrative privileges on target

**Common Uses:**
- Remote shell access
- Command execution on compromised hosts
- Post-exploitation lateral movement

---

## 📚 References

- [Impacket — SecureAuth GitHub](https://github.com/fortra/impacket)
- [smbclient man page](https://www.samba.org/samba/docs/current/man-html/smbclient.1.html)
