# NetExec (NXC) Notes

Part of the **Offensive-Active-Directory** notes series.

NetExec (the successor to CrackMapExec) is a network execution/enumeration
tool used heavily in Active Directory pentesting for credential validation,
domain enumeration, and lateral movement.

---

## Installation

### Option 1: Install from Package Manager

```bash
apt install netexec
```

### Option 2: Install Using Pipx (Recommended)

Using `pipx` ensures NetExec runs in an isolated Python environment.

**Step 1: Install pipx**

```bash
apt install pipx
```

**Step 2: Install NetExec from GitHub**

```bash
pipx install git+https://github.com/Pennyw0rth/NetExec
```

**Step 3: Add Binaries to Global Path (if needed)**

```bash
cp -v /root/.local/bin/netexec /usr/local/bin/
cp -v /root/.local/bin/nxc /usr/local/bin/
cp -v /root/.local/bin/nxcdb /usr/local/bin/
chmod +x /usr/local/bin/nxc
```

---

## Basic SMB Usage

The `nxc smb` module is heavily used for **credential validation and domain
enumeration**.

### Scan a List of IPs

```bash
nxc smb target-ips.txt
```

### Authenticate Against a Single Host

```bash
nxc smb 192.168.1.51 -u administrator -p 'armour123'
```

### Credential Spraying (Username + Password Files)

```bash
nxc smb 192.168.1.51 -u /root/ad/username.txt -p /opt/passwords.txt
```

### Continue After First Success

```bash
nxc smb 192.168.1.51 -u /root/ad/username.txt -p /opt/passwords.txt --continue-on-success
```

### Quick Authentication Test

```bash
nxc smb 192.168.1.51 -u armour -p armour123 --continue-on-success
```

---

## SMB Enumeration Examples

### List Domain Users

```bash
nxc smb 10.10.10.161 --users -u svc-alfresco -p s3rvice
```

---

## Notes / To-Do

- [ ] Add more SMB enumeration modules (shares, sessions, disks, groups, etc.)
- [ ] Add LDAP/WinRM/MSSQL/RDP usage sections
- [ ] Add common flags reference table (`--local-auth`, `-x`, `-M`, etc.)
