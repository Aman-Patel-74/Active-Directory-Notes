# Disable Windows Defender and Windows Update

> Lab notes — permanently removing Windows Defender, Windows Update, and Windows Recovery on a Windows host (offline / live-boot method + SYSTEM-level script method).

---

## Folders to Delete (Live Boot Method)

```
C:\ProgramData\Microsoft\Windows Defender
C:\Program Files\Windows Defender
```

**Tool used:** [Defender Control v2.1](https://www.sordum.org/9480/defender-control-v2-1/)

### Steps to Delete Using a Live Boot Environment

**1. Boot into a Live Linux or Windows PE Environment**
- Create a bootable USB with a live Linux distro (Ubuntu, Kali Linux, etc.) or Windows PE.
- Boot your system from the USB.

**2. Access the Windows File System**
- In Linux, open a terminal and mount the Windows partition:
```bash
lsblk
mount /dev/sda1 /mnt
```
*(Replace `/dev/sdX#` with the correct Windows partition.)*
- In Windows PE, open `cmd` or `Explorer` and navigate to the drive.

**3. Delete the Folders**

In Linux:
```bash
sudo rm -rf /mnt/ProgramData/Microsoft/Windows\ Defender
sudo rm -rf /mnt/Program\ Files/Windows\ Defender
```

In Windows PE (Command Prompt):
```cmd
del /s /q "C:\ProgramData\Microsoft\Windows Defender"
del /s /q "C:\Program Files\Windows Defender"
rd /s /q "C:\ProgramData\Microsoft\Windows Defender"
rd /s /q "C:\Program Files\Windows Defender"
```

**4. Unmount and Reboot**

In Linux:
```bash
umount /mnt
```
- Remove the bootable USB and restart your system.

---

## Step 1: Disable Tamper Protection Manually

Tamper Protection blocks registry edits and service modifications, so you must turn it off manually before running the script.

1. Open Windows Security (`Win + S` → Search for Windows Security).
2. Go to **Virus & threat protection**.
3. Click **Manage settings** under "Virus & threat protection settings".
4. Scroll to **Tamper Protection** → Turn it **OFF**.
5. Restart your PC to apply changes.

---

## Step 2: Download PsExec (for SYSTEM-level Execution)

To fully disable Defender and prevent reactivation, the script needs to run as `NT AUTHORITY\SYSTEM`.

1. Download PsExec from Microsoft:
   [https://docs.microsoft.com/en-us/sysinternals/downloads/psexec](https://docs.microsoft.com/en-us/sysinternals/downloads/psexec)
2. Extract `PsExec.exe` to `C:\Windows\System32\` so you can use it anywhere in the command prompt.

---

## Step 3: Save and Run the Removal Script

### 3.1: Save This Script as `Remove_Defender_Update.bat`

```bat
@echo off
title PERMANENT REMOVAL OF WINDOWS DEFENDER, UPDATES and RECOVERY
cls
echo ============================================
echo   PERMANENTLY REMOVING WINDOWS DEFENDER, UPDATES and RECOVERY
echo ============================================
echo.

:: Run as SYSTEM for full permissions
whoami | findstr /i "system" >nul
if %errorlevel% neq 0 (
    echo [!] This script must be run as NT AUTHORITY\SYSTEM for full effect!
    echo [!] Use PsExec or Task Scheduler to run it as SYSTEM.
    echo.
    pause
    exit
)

echo [*] Stopping and deleting Windows Defender services...
sc stop WinDefend
sc delete WinDefend

echo [*] Removing Windows Defender registry keys...
reg delete "HKLM\SOFTWARE\Policies\Microsoft\Windows Defender" /f
reg delete "HKLM\SOFTWARE\Microsoft\Windows Defender" /f
reg delete "HKLM\SYSTEM\CurrentControlSet\Services\WinDefend" /f

echo [*] Taking ownership and deleting Windows Defender files...
takeown /f "%ProgramFiles%\Windows Defender" /a /r /d y
icacls "%ProgramFiles%\Windows Defender" /grant Administrators:F /t /c
rd /s /q "%ProgramFiles%\Windows Defender"

takeown /f "%ProgramData%\Microsoft\Windows Defender" /a /r /d y
icacls "%ProgramData%\Microsoft\Windows Defender" /grant Administrators:F /t /c
rd /s /q "%ProgramData%\Microsoft\Windows Defender"

echo [*] Disabling Windows Update services permanently...
sc stop wuauserv
sc delete wuauserv
sc stop bits
sc delete bits
sc stop UsoSvc
sc delete UsoSvc

echo [*] Removing Windows Update registry keys...
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate" /v "DisableWindowsUpdateAccess" /t REG_DWORD /d 1 /f
reg delete "HKLM\SYSTEM\CurrentControlSet\Services\wuauserv" /f
reg delete "HKLM\SYSTEM\CurrentControlSet\Services\bits" /f
reg delete "HKLM\SYSTEM\CurrentControlSet\Services\UsoSvc" /f

echo [*] Disabling Windows Recovery (to prevent Defender from coming back)...
bcdedit /set {current} recoveryenabled No
bcdedit /set {current} bootstatuspolicy ignoreallfailures

echo [*] Disabling Windows Defender Tamper Protection (requires reboot)...
reg add "HKLM\SOFTWARE\Microsoft\Windows Defender\Features" /v "TamperProtection" /t REG_DWORD /d 0 /f

echo [*] Blocking Microsoft Defender updates via HOSTS file...
echo 0.0.0.0 smartscreen.microsoft.com >> %WINDIR%\System32\drivers\etc\hosts
echo 0.0.0.0 windowsupdate.microsoft.com >> %WINDIR%\System32\drivers\etc\hosts
echo 0.0.0.0 update.microsoft.com >> %WINDIR%\System32\drivers\etc\hosts
echo 0.0.0.0 download.microsoft.com >> %WINDIR%\System32\drivers\etc\hosts

:: Disable Microsoft Defender Security Center (prevents re-enabling Tamper Protection)
sc stop SecurityHealthService
sc config SecurityHealthService start= disabled
reg add "HKLM\SYSTEM\CurrentControlSet\Services\SecurityHealthService" /v "Start" /t REG_DWORD /d 4 /f

echo [*] Microsoft Defender Security Center disabled - Tamper Protection will not return!

echo.
echo ============================================
echo   WINDOWS DEFENDER, WINDOWS UPDATE & RECOVERY REMOVED!
echo   SYSTEM REQUIRES A REBOOT TO FINALIZE CHANGES.
echo ============================================
echo.
pause
exit
```

### 3.2: Run the Script as SYSTEM

> **Note:** Check once again that Real-time Protection and Tamper Protection are disabled.

1. Open Command Prompt as Administrator (`Win + S` → Search `cmd` → Right-click → Run as Administrator).
2. Run the script as SYSTEM:
```cmd
psexec -s -i cmd.exe
```
   This will open a new SYSTEM-level command prompt.
3. Navigate to the script location:
```cmd
cd C:\path\to\your\script
```
4. In the new SYSTEM prompt, run:
```cmd
Remove_Defender_Update.bat
```
5. Wait for the script to finish.

---

## Step 4: Restart the System

After the script completes:

1. Restart your PC (`Win + X` → Shut down or sign out → Restart).
2. After reboot, Windows Defender and Windows Update should be completely removed.

---

## Step 5: Run the Verification Script

To confirm that everything is removed, use this script.

### Save This as `Check_Removal_Status.bat`

```bat
@echo off
title CHECK IF WINDOWS DEFENDER, UPDATES & RECOVERY ARE REMOVED
cls
echo ============================================
echo   VERIFYING REMOVAL OF WINDOWS DEFENDER, UPDATES & RECOVERY
echo ============================================
echo.

:: Check Windows Defender Service
sc query WinDefend | findstr /i "RUNNING"
if %errorlevel% == 0 (
    echo [!] WARNING: Windows Defender is STILL RUNNING!
) else (
    echo [+] Windows Defender is REMOVED.
)

:: Check Tamper Protection
reg query "HKLM\SOFTWARE\Microsoft\Windows Defender\Features" /v "TamperProtection" 2>nul | findstr /i "0x1"
if %errorlevel% == 0 (
    echo [!] WARNING: Tamper Protection is STILL ENABLED!
) else (
    echo [+] Tamper Protection is DISABLED.
)

:: Check Windows Recovery
bcdedit /enum | findstr /i "recoveryenabled No"
if %errorlevel% == 0 (
    echo [+] Windows Recovery is DISABLED.
) else (
    echo [!] WARNING: Windows Recovery is STILL ENABLED!
)

echo.
echo ============================================
echo   VERIFICATION COMPLETE! CHECK FOR ANY WARNINGS ABOVE.
echo ============================================
echo.
pause
exit
```

### Run the Verification Script

1. Open Command Prompt as Administrator.
2. Navigate to the script location.
3. Run the script:
```cmd
Check_Removal_Status.bat
```
4. If all results are `[+] DISABLED`, everything is removed!

---

## Fallback: If Verification Shows Warnings

If you get output like:

```
[!] WARNING: Windows Defender is STILL RUNNING!
```

follow the process below.

### Step 1: Run Commands as SYSTEM (Highest Privilege)

1. Open Command Prompt as Administrator.
2. Run this command to get a SYSTEM-level shell:
```cmd
psexec -s -i cmd.exe
```
3. Querying the Service:
```cmd
sc query WinDefend
sc qc WinDefend
```
4. Deleting the Service:
```cmd
sc delete WinDefend
```
5. Deleting Windows Defender Executable:
```cmd
del C:\Program Files\Windows Defender\MsMpEng.exe
```
6. Taking Ownership and Granting Permissions (before deletion):
```cmd
takeown /f "C:\Program Files\Windows Defender\MsMpEng.exe"
icacls "C:\Program Files\Windows Defender\MsMpEng.exe" /grant Administrators:F /t
del /s /q "C:\Program Files\Windows Defender\MsMpEng.exe"
```

Also try taking ownership of the ProgramData folder if deletion fails:
```cmd
takeown /f "C:\ProgramData\Microsoft\Windows Defender" /r /d y
icacls "C:\ProgramData\Microsoft\Windows Defender" /grant Administrators:F /t
```
If this works, proceed to delete Defender:
```cmd
rd /s /q "C:\ProgramData\Microsoft\Windows Defender"
```
- If this fails, continue to Step 2 below.

### Step 2: Boot Into Safe Mode & Delete Defender Files

If Step 1 fails, Defender is using kernel-level protection. You need to disable it in Safe Mode.

**Boot into Safe Mode with Command Prompt**

1. Press `Win + R`, type `msconfig`, and press Enter.
2. Go to the **Boot** tab, check **Safe boot**, and select **Minimal**.
3. Click **Apply** → **OK**, then restart your PC.
4. Once in Safe Mode, open Command Prompt as Administrator and run:
```cmd
takeown /f "C:\ProgramData\Microsoft\Windows Defender" /r /d y
icacls "C:\ProgramData\Microsoft\Windows Defender" /grant Administrators:F /t
rd /s /q "C:\ProgramData\Microsoft\Windows Defender"
```
5. Reboot into normal mode:
   - Open `msconfig` again and uncheck **Safe boot**.
   - Restart.

### Step 3: Disable Defender via Registry

If files are still protected, block Defender at the system level via registry.

1. Open Command Prompt as Administrator.
2. Run these commands (see registry deletions in Step 3 script above).

### Step 5: Disable Defender Using Group Policy

1. Press `Win + R`, type `gpedit.msc`, and press Enter.
2. Navigate to:
```
Computer Configuration -> Administrative Templates -> Windows Components -> Microsoft Defender Antivirus
```
3. Enable:
   1. "Turn off Microsoft Defender Antivirus"
   2. "Turn off Windows Defender real-time protection"
4. Restart Your PC.

**Then run as Administrator:** `Check_Removal_Status.bat`

---

## ⚠️ Disclaimer

These notes were captured from a personal offensive-security lab (Active Directory / lab-setup environment) for **testing and educational purposes on isolated lab machines only**. Disabling Windows Defender, Windows Update, and Windows Recovery removes core security protections and should **never** be done on a production system or any machine connected to sensitive data/networks.
