# Windows Remote Management (WinRM) & Evil-WinRM

> Lab notes — configuring WinRM on Windows for remote PowerShell administration, and using Evil-WinRM as a post-exploitation shell.

---

# Part 1: Windows-Remote-Management (WinRM)

## Setting up Windows Remote Management (WinRM) for Remote PowerShell Execution

### Configuring WinRM

- Enables and starts the WinRM service.
- Configures the listener for HTTP (default port 5985).

```powershell
winrm quickconfig
```

### Ensuring the WinRM Service Starts Automatically

- Starts the service immediately.
```powershell
Start-Service WinRM
```

- Check port 5985 (WinRM) is open.
```bash
nmap -v -n -p 5985 -sV -sC -A -T4 192.168.1.61
```

- Ensures WinRM is set to start automatically on boot.
```powershell
Set-Service -Name WinRM -StartupType Automatic
```

### If the PowerShell Method Doesn't Work, Try These Alternatives:

**1️⃣ Force Change via Registry**

Since `Set-Service` might not be enough, modify the registry manually:

1. Open PowerShell as Administrator.
2. Run this command:
```powershell
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\WinRM" -Name "DelayedAutoStart" -Value 0
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\WinRM" -Name "Start" -Value 2
```
   - `DelayedAutoStart = 0` → Disables delayed start.
   - `Start = 2` → Forces automatic start.
3. Restart the computer for the changes to take effect:
```powershell
Restart-Computer -Force
```

**2️⃣ Use SC Config Command**

If the registry method doesn't work, try this:

1. Open Command Prompt as Administrator.
2. Run:
```cmd
sc config WinRM start= auto
```
   - This forces the service to Automatic Start.
3. Restart the system and verify:
```powershell
Get-Service -Name WinRM | Select-Object Name, StartType
```

### Enabling PowerShell Remoting

- Enables remote PowerShell execution on the system.
- Configures firewall exceptions and required permissions.

```powershell
Enable-PSRemoting -Force
```

### Checking WinRM Service Status

- Verifies that WinRM is running.
```powershell
Get-Service -Name WinRM
Get-Service -Name WinRM | Select-Object -Property Name, StartType
```

### Checking Configured Listeners

- Lists the configured listeners (HTTP/HTTPS) and their settings.
```cmd
winrm enumerate winrm/config/listener
```

### Adding Users to the "Remote Management Users" Group

- Adds users (`piyush` and `jdoe`) to the Remote Management Users group.
```cmd
net localgroup "Remote Management Users" mohan /add
net localgroup "Remote Management Users" rohan /add
```

- Lists the current users in the group.
```cmd
net localgroup "Remote Management Users"
```

- Accessing a Windows machine via WinRM with credentials:
```bash
evil-winrm -i 192.168.1.101 -u jdoe -p Password123!
```
```bash
evil-winrm -i 192.168.1.101 -u piyush -p password@12345
```

- `evil-winrm` → Launches the Evil-WinRM tool (used for remote access on Windows via WinRM).
- `-i 192.168.1.101` → Specifies the target IP (192.168.1.101).
- `-u jdoe` → Username: jdoe

### Checking WinRM Service Configuration

- Displays the current WinRM service configuration.
```cmd
winrm get winrm/config/service
```

### Configuring PowerShell Remoting Security

- Opens a GUI to configure security settings for remote PowerShell sessions.
```powershell
Set-PSSessionConfiguration -Name Microsoft.PowerShell -ShowSecurityDescriptorUI
```

### Additional Considerations

**1️⃣ Use HTTPS Instead of HTTP**

For security, consider setting up HTTPS listeners using:
```cmd
winrm create winrm/config/Listener?Address=*+Transport=HTTPS @{CertificateThumbprint="THUMBPRINT"}
```

**2️⃣ Verify WinRM Connectivity**

From another system, test if WinRM is working:
```powershell
Test-WSMan -ComputerName TargetMachine
```

**3️⃣ Enable Trusted Hosts (For Workgroups)**

If the machines are not in the same domain, allow trusted hosts:
```powershell
Set-Item WSMan:\localhost\Client\TrustedHosts -Value "TargetMachine"
```

---

# Part 2: Evil-WinRM (Windows Remote Management)

## Overview

**WinRM (Windows Remote Management)** is a Microsoft protocol that enables remote management and administration of Windows systems. It allows administrators to execute commands, manage configurations, and automate tasks over a network using command-line tools and scripts.

WinRM is based on **WS-Management (WS-Man)**, an industry-standard web services protocol for remote management of hardware and software. It is widely used in enterprise environments for centralized administration and automation.

### Key Features of WinRM

1. **Remote Command Execution** — Execute commands or scripts on remote Windows systems.
2. **PowerShell Remoting** — Enables remote execution of PowerShell commands and scripts.
3. **Multiple Authentication Methods** — Supports:
   - Kerberos (default in domain environments)
   - NTLM
   - Basic Authentication (disabled by default; insecure)
4. **Secure Communication**
   - HTTP (default, encrypted at message level)
   - HTTPS (recommended for external/untrusted networks)
5. **Centralized Management** — Manage multiple machines from a single administrative system.

### How WinRM Works

WinRM communicates over HTTP/HTTPS using SOAP (XML-based) messages.

### Default Ports

| Protocol | Port | Encryption               |
|----------|------|---------------------------|
| HTTP     | 5985 | Message-level encryption  |
| HTTPS    | 5986 | TLS encryption             |

### Common Use Cases

- Remote system administration
- Automated configuration management
- Log collection and monitoring
- Infrastructure automation
- PowerShell Desired State Configuration (DSC)

---

## Enabling WinRM

### 1️⃣ Enable PowerShell Remoting

```powershell
Enable-PSRemoting -Force
```

This:
- Starts the WinRM service
- Sets it to automatic startup
- Creates a listener
- Configures firewall rules

### 2️⃣ Verify WinRM Service Status

```powershell
Get-Service -Name WinRM
```

### 3️⃣ Configure Firewall (If Required)

```powershell
Enable-NetFirewallRule -Name "WINRM-HTTP-In-TCP-PUBLIC"
Enable-NetFirewallRule -Name "WINRM-HTTPS-In-TCP-PUBLIC"
```

---

## Using WinRM with PowerShell

### Establish Interactive Remote Session

```powershell
Enter-PSSession -ComputerName <TargetComputer> -Credential <Credential>
```

### Execute Command Remotely

```powershell
Invoke-Command -ComputerName <TargetComputer> -ScriptBlock { Get-Process }
```

### Exit Session

```powershell
Exit-PSSession
```

---

## Security Considerations

- ✅ Use **HTTPS (5986)** in untrusted networks
- ✅ Prefer **Kerberos authentication** in domain environments
- ❌ Avoid Basic Authentication
- Restrict WinRM access via firewall rules
- Limit access to authorized administrative users

---

## Troubleshooting

- Check WinRM service:
```powershell
Get-Service WinRM
```
- Verify listener configuration:
```cmd
winrm enumerate winrm/config/listener
```
- Test connectivity:
```powershell
Test-WsMan <TargetComputer>
```

---

## Evil-WinRM

**Evil-WinRM** is an open-source penetration testing tool that provides a powerful WinRM shell for post-exploitation and red team operations.

It is commonly used to obtain remote shells on Windows machines when valid credentials are available.

> ⚠️ Use only in authorized environments.

### Features

- Remote PowerShell shell access
- Kerberos and NTLM authentication support
- Pass-the-Hash support
- File upload & download
- Script execution
- Colored interactive interface

### Installation

**Using RubyGems**
```bash
gem install evil-winrm
```

**Manual Installation**
```bash
git clone https://github.com/Hackplayers/evil-winrm.git
cd evil-winrm
gem install bundler
bundle install
```

### Basic Usage

```bash
evil-winrm -i <target_ip> -u <username> -p <password>
```

### Parameters

| Option | Description              |
|--------|---------------------------|
| -i     | Target IP address         |
| -u     | Username                  |
| -p     | Password                  |
| -k     | Kerberos authentication   |
| -P     | Custom port                |
| -s     | Use SSL (HTTPS)           |

---

## ⚠️ Disclaimer

These notes were captured from a personal offensive-security lab (Active Directory / lab-setup environment) for **testing and educational purposes on isolated lab machines only**. WinRM and Evil-WinRM provide powerful remote-access capabilities and should only be used against systems you own or are explicitly authorized to test.
