# ⚔️ Kerberoasting Attack — Lab Setup & LDAP Enumeration

> Offensive-Active-Directory / Kerberos-Enumeration / Kerberoasting-Attack

> **Note:** This note currently covers the *lab setup* and *LDAP-based SPN enumeration* portion of the Kerberoasting Attack page. Send over the remaining screenshots (attack execution, hash cracking, detection/mitigation) whenever you have them and this file can be extended.

---

## Lab Setup — Simulate a Vulnerable Service Account

Simulate a vulnerable service account by assigning an SPN.

```powershell
$PASSWORD = ConvertTo-SecureString -AsPlainText -Force -String "Password123"

New-ADUser -Name "Kerbe-roast" `
  -Description "Kerberoasting Lab" `
  -Enabled $true `
  -AccountPassword $PASSWORD

Set-ADUser -Identity Kerbe-roast `
  -ServicePrincipalNames @{Add="HTTP/dc01.infosecwarrior.local"}
```

---

## 1. Enumerate SPNs Using LDAP

Use `ldapsearch` to query Active Directory and extract SPNs.

```bash
ldapsearch -x -H ldap://192.168.2.100 -D '' -w '' -b 'DC=infosecwarrior,DC=local' > ldapsearch-output.txt
```

### Extract `servicePrincipalName`

```bash
grep servicePrincipalName ldapsearch-output.txt
```

```bash
grep servicePrincipalName ldapsearch-output.txt | cut -d " " -f2 > spn-list.txt
```

### Extract `userPrincipalName`

```bash
grep userPrincipalName ldapsearch-output.txt
```

```bash
grep userPrincipalName ldapsearch-output.txt | cut -d " " -f2 > upn-list.txt
```

---

<p align="center"><sub>📚 Notes compiled for personal study & offensive security learning.</sub></p>
