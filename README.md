# Active-Directory-Notes
# Active Directory Notes

A collection of personal notes on offensive Active Directory security — enumeration, lateral movement, credential attacks, Kerberos abuse, and common tooling. Written while learning and practicing in a lab environment.

> ⚠️ **Disclaimer:** These notes are for educational purposes and authorized lab/pentesting use only. Do not use any of this against systems you don't own or have explicit written permission to test.

---

## 📖 Table of Contents

### 🧭 Fundamentals
- [Active Directory Basics](Active-Directory.md)
- [Access Control Model — Active Directory](Access%20Control%20Model%20—%20Active%20Directory.md)
- [Offensive Active Directory — Attack Methodology](Offensive%20Active%20Directory%20—%20Attack%20Methodology.md)
- [Offensive Active Directory — Lab Setup Notes](Offensive%20Active%20Directory%20—%20Lab%20Setup%20Notes.md)

### 🔍 Enumeration
- [Offensive Active Directory — AD Enumeration](Offensive%20Active%20Directory%20–%20AD%20Enumeration.md)
- [Kerberos Enumeration](Kerberos%20Enumeration.md)
- [Enum4linux & Enum4linux-ng](Enum4linux%20&%20Enum4linux-ng.md)
- [PowerView — Active Directory Enumeration](PowerView%20—%20Active%20Directory%20Enumeration.md)
- [BloodHound — Active Directory Enumeration](BloodHound%20—%20Active%20Directory%20Enumeration.md)
- [SMB Password Management & Enumeration](SMB%20Password%20Management%20&%20Enumeration.md)
- [PowerHuntShares — SMB Share Enumeration & Access](PowerHuntShares%20—%20SMB%20Share%20Enumeration%20&%20Access.md)

### 🔑 Credential & Kerberos Attacks
- [Kerberoasting Attack](Kerberoasting%20Attack.md)
- [AS-REP Roasting Attack](AS-REP%20Roasting%20Attack.md)
- [Impacket — GetNPUsers.py](Impacket%20—%20GetNPUsers.py.md)
- [Mimikatz Usage and Execution](Mimikatz%20Usage%20and%20Execution.md)
- [LSASS Dumping & Credential Extraction using pypykatz](LSASS%20Dumping%20&%20Credential%20Extraction%20using%20pypykatz.md)

### 🌐 Network Attacks
- [LLMNR/NBT-NS Poisoning in Windows Domain Environments](LLMNR-NBT-NS%20Poisoning%20in%20Windows%20Domain%20Environments.md)
- [SMB Relay Attack](SMB%20Relay%20Attack.md)

### 🖥️ Lateral Movement & Remote Access
- [Lateral Movement (Active Directory)](Lateral%20Movement%20(Active%20Directory).md)
- [Windows Remote Management (WinRM) & Evil-WinRM Usage](Windows%20Remote%20Management%20(WinRM)%20&%20Evil-WinRM%20Usage.md)

### 🛠️ Tools
- [CrackMapExec (CME)](CrackMapExec%20(CME).md)
- [NetExec (NXC) Notes](NetExec%20(NXC)%20Notes.md)

### 🛡️ Defense Evasion
- [Disable Windows Defender and Windows Update](Disable%20Windows%20Defender%20and%20Windows%20Update.md)

---

## 🗂️ Structure

All notes are currently kept as flat Markdown files at the repo root, one per topic. Each file generally covers:
- **What** the technique/tool is
- **How** it works
- **Commands** to execute it
- **Mitigations / detection** where applicable

## 🤝 Contributing / Feedback

This is a personal learning repo, but corrections, suggestions, and PRs are welcome via Issues.

## ⭐ Support

If you find these notes useful, consider starring the repo — it helps others find it too.
