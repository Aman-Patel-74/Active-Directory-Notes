# Offensive Active Directory — Lab Setup Notes

Notes on configuring remote access (RDP & WinRM) in an Active Directory lab environment for offensive security practice.

---

## 1. Remote Desktop Access to a Domain User

Remote Desktop Services (RDS) let users connect to a computer remotely, accessing its desktop interface as if physically present — useful for remote administration, troubleshooting, and end-user support.

Group Policy (a Windows Server feature) lets administrators manage and configure OS/app/user settings across an Active Directory environment. It allows enabling Remote Desktop on multiple computers at once, enforcing security settings, and specifying which users/groups have remote access — all centrally managed.

### Prerequisites

- **Active Directory Environment** — network configured with AD, and target computers joined to the domain.
- **Administrative Privileges** — rights to create/modify Group Policy Objects (GPOs).
- **Windows Firewall Configuration** — firewall rules enabled to allow incoming RDP connections.

### Method A: Enable Remote Desktop via Group Policy (GUI)

**Step 1 — Open Group Policy Management Console (GPMC)**
1. Log in to the Domain Controller (or a workstation with GPMC installed).
2. Press `Win + R`, type `gpmc.msc`, press Enter.

**Step 2 — Create a New GPO**
1. In GPMC, navigate to the domain or the specific OU containing the target computers.
2. Right-click the domain/OU → **"Create a GPO in this domain, and Link it here…"**
3. Name the GPO (e.g., `Enable Remote Desktop`) → click **OK**.

**Step 3 — Edit the GPO to Enable Remote Desktop**
- Edit the new GPO and configure the Remote Desktop / Terminal Services settings, then link/apply it to the target OU.

### Method B: Automate with PowerShell

**Step 1 — Add a Domain User to the Remote Desktop Users Group on All Computers**

```powershell
$DomainUser = "infosecwarrior.local\jdoe"   # Change this to the domain user you want to add
$Computers = Get-ADComputer -Filter * | Select-Object -ExpandProperty Name

foreach ($Computer in $Computers) {
    Invoke-Command -ComputerName $Computer -ScriptBlock {
        param($User)
        Add-LocalGroupMember -Group "Remote Desktop Users" -Member $User -ErrorAction SilentlyContinue
    } -ArgumentList $DomainUser -Credential (Get-Credential)
}
```
- Retrieves all domain computers and adds the specified user to the local **Remote Desktop Users** group on each one.
- If running from a non-admin machine, you may need to supply credentials.

**Step 2 — Enable Remote Desktop on All Computers**

```powershell
$Computers = Get-ADComputer -Filter * | Select-Object -ExpandProperty Name

foreach ($Computer in $Computers) {
    Invoke-Command -ComputerName $Computer -ScriptBlock {
        Set-ItemProperty -Path 'HKLM:\System\CurrentControlSet\Control\Terminal Server' -Name "fDenyTSConnections" -Value 0
        Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
    } -Credential (Get-Credential)
}
```
- Enables RDP (`fDenyTSConnections = 0`) and opens the firewall rule group for Remote Desktop on every domain computer.

**Step 3 — Grant Remote Desktop Logon Rights**
- Ensure the target user/group has the "Allow log on through Remote Desktop Services" right (via GPO or local security policy) so they can actually log in remotely.

### Connecting via RDP (from Kali)

```bash
rdesktop 192.168.1.101
```

Example session output/notes:
- You may see a certificate trust warning, e.g.:
  ```
  ATTENTION! The server uses an invalid security certificate which cannot be trusted...
  Issuer: CN=WC01.infosecwarrior.local
  Do you trust this certificate (yes/no)? yes
  ```
- If you see: `Failed to initialize NLA, do you have correct Kerberos TGT initialized?` — this indicates Network Level Authentication (NLA) requires a valid Kerberos ticket; obtain a TGT (e.g., via `kinit`/`getTGT`) before connecting, or disable NLA on the target for testing.
- On success: `Connection established using SSL.`

---

## 2. Windows Remote Management (WinRM)

Setting up WinRM for remote PowerShell execution.

### Configuring WinRM

Enables and starts the WinRM service, and configures the listener for HTTP (default port 5985):

```powershell
winrm quickconfig
```

### Ensuring the WinRM Service Starts Automatically

```powershell
Start-Service WinRM
```

Check that port 5985 (WinRM) is open:

```bash
nmap -v -n -p 5985 -sV -sC -A -T4 192.168.1.61
```

Ensure WinRM starts automatically on boot:

```powershell
Set-Service -Name WinRM -StartupType Automatic
```

### If the PowerShell Method Doesn't Work — Alternatives

**1. Force Change via Registry**

Since `Set-Service` might not be enough, modify the registry manually:

1. Open PowerShell as Administrator.
2. Run:
   ```powershell
   Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\WinRM" -Name "DelayedAutoStart" -Value 0
   Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\WinRM" -Name "Start" -Value 2
   ```
   - `DelayedAutoStart = 0` → disables delayed start.
   - `Start = 2` → forces automatic start.
3. Restart the computer for changes to take effect:
   ```powershell
   Restart-Computer -Force
   ```

**2. Use SC Config Command**

If the registry method doesn't work:

1. Open Command Prompt as Administrator.
2. Run:
   ```cmd
   sc config WinRM start= auto
   ```
   - Forces the service to Automatic start.
3. Restart the system and verify:
   ```powershell
   Get-Service -Name WinRM | Select-Object Name, StartType
   ```

### Enabling PowerShell Remoting

Enables remote PowerShell execution and configures required firewall exceptions/permissions:

```powershell
Enable-PSRemoting -Force
```

### Checking WinRM Service Status

```powershell
Get-Service -Name WinRM
Get-Service -Name WinRM | Select-Object -Property Name, StartType
```

### Checking Configured Listeners

Lists the configured listeners (HTTP/HTTPS) and their settings:

```powershell
winrm enumerate winrm/config/listener
```

### Adding Users to the "Remote Management Users" Group

Adds users to the group so they can connect via WinRM:

```powershell
net localgroup "Remote Management Users" mohan /add
net localgroup "Remote Management Users" rohan /add
```

List current users in the group:

```powershell
net localgroup "Remote Management Users"
```

### Accessing a Windows Machine via WinRM (evil-winrm)

```bash
evil-winrm -i 192.168.1.101 -u mohan -p 'password@12345'
evil-winrm -i 192.168.1.101 -u piyush -p 'password@12345'
```

- `evil-winrm` → launches the Evil-WinRM tool (used for remote access on Windows via WinRM).
- `-i 192.168.1.101` → specifies the target IP.
- `-u <user>` → username.
- `-p <password>` → password for the user.

### Creating a Firewall Rule for WinRM

Opens port 5985 for inbound WinRM connections:

```cmd
netsh advfirewall firewall add rule name="WinRM" dir=in action=allow protocol=TCP localport=5985
```

### Checking WinRM Service Configuration

Displays the current WinRM service configuration:

```powershell
winrm get winrm/config/service
```

### Configuring PowerShell Remoting Security

Opens a GUI to configure security settings for remote PowerShell sessions:

```powershell
Set-PSSessionConfiguration -Name Microsoft.PowerShell -ShowSecurityDescriptorUI
```

### Additional Considerations

**1. Use HTTPS Instead of HTTP**

For security, consider setting up an HTTPS listener:

```powershell
winrm create winrm/config/Listener?Address=*+Transport=HTTPS @{CertificateThumbprint="THUMBPRINT"}
```

**2. Verify WinRM Connectivity**

From another system, test if WinRM is working:

```powershell
Test-WSMan -ComputerName TargetMachine
```

**3. Enable Trusted Hosts (For Workgroups)**

If machines are not in the same domain, allow trusted hosts:

```powershell
Set-Item WSMan:\localhost\Client\TrustedHosts -Value "TargetMachine"
```

---

## Quick Reference — Command Summary

| Purpose | Command |
|---|---|
| Configure WinRM | `winrm quickconfig` |
| Start WinRM service | `Start-Service WinRM` |
| Scan for WinRM port | `nmap -v -n -p 5985 -sV -sC -A -T4 <target>` |
| Set WinRM to auto-start | `Set-Service -Name WinRM -StartupType Automatic` |
| Force auto-start via registry | `Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\WinRM" -Name "Start" -Value 2` |
| Force auto-start via sc | `sc config WinRM start= auto` |
| Enable PS remoting | `Enable-PSRemoting -Force` |
| Check WinRM status | `Get-Service -Name WinRM` |
| List listeners | `winrm enumerate winrm/config/listener` |
| Add user to remote mgmt group | `net localgroup "Remote Management Users" <user> /add` |
| Connect via Evil-WinRM | `evil-winrm -i <ip> -u <user> -p <password>` |
| Open WinRM firewall port | `netsh advfirewall firewall add rule name="WinRM" dir=in action=allow protocol=TCP localport=5985` |
| Connect via RDP (Linux) | `rdesktop <ip>` |

---

*Notes compiled from an internal Offensive Active Directory lab-setup reference (RDP & WinRM configuration).*
