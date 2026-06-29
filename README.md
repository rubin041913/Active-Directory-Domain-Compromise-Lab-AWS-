# 🚨 Active Directory Domain Compromise Lab (AWS)

## 📌 Overview

This project demonstrates a full Active Directory domain compromise conducted in a controlled AWS-based lab environment.

Starting from a low-privileged domain user, I performed enumeration, credential discovery, lateral movement, and privilege escalation, ultimately achieving Domain Administrator-level access and demonstrating domain compromise capability.

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

## 📏 Scope & Assumptions

* This lab was conducted in an isolated AWS environment
* No production systems were involved
* All actions were performed in an authorized testing environment
* Objectives were limited to Active Directory compromise simulation
* All data and systems were contained within the lab scope

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
9. Full Active Directory compromise demonstrated

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

## 💥 Final Result: Domain Compromise Demonstrated

This attack demonstrated capability for:

* Domain Administrator-level access
* Extraction of all domain credentials
* Compromise of Kerberos authentication trust (krbtgt)
* Pass-the-Hash and Golden Ticket attack potential
* Long-term domain persistence

---

## 🚨 Security Impact

This lab demonstrates how:

* A single misconfigured service account can create domain-wide compromise risk
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

**Key Finding:** sql_svc confirmed as member of Domain Admins - critical privilege escalation point identified.

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

**Impact:** Valid Kerberos authentication established. Domain service access enabled.

---

### Screenshot 3: SPN Enumeration Revealing sql_svc
**Caption: SPN enumeration revealing sql_svc service account in Domain Admins**

```
(kali@kali)~$ impacket-GetUserSPNs corp.local/jsmith:Password123! -dc-ip 172.31.34.227

ServicePrincipalName                 Name       MemberOf
===================================================================
MSSQLSvc/sqlserver.corp.local:1433   sql_svc    CN=Domain Admins,CN=Users,DC=corp,DC=local
```

**Critical Vulnerability:** Service account sql_svc identified with excessive privileges (Domain Admins membership). Creates direct path to domain compromise.

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

**Result:** Remote code execution achieved with Domain Admins privileges via WMI lateral movement.

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

**Security Issue:** Service account membership in Domain Admins confirmed. Overprivilege condition enables domain-wide escalation.

---

### Screenshot 6: Credential Extraction Initiated
**Caption: Secretsdump extracting NTLM hashes from Domain Controller**

```
(kali@kali)~$ impacket-secretsdump 'corp.local/sql_svc:ComplexPassword123!@172.31.34.227'
Impacket v0.14.0.dev0

[*] Target system bootKey: 0x8dc1f3d7d34256a6b8db8d196d8aaaa8
[*] Dumping local SAM hashes
[*] Dumping LSA Secrets
[*] Dumping Domain Credentials
[*] Using the DRSUAPI method to get NTDS.DIT secrets
```

**Key Findings:**
- SAM hashes successfully extracted from local system
- LSA secrets (encryption keys, DPAPI) compromised
- DRSUAPI method enabled NTDS database access
- Domain credential extraction initiated

---

### Screenshot 7: krbtgt Hash Compromise
**Caption: krbtgt hash compromise - Kerberos master key obtained**

```
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:0ac412149d85297951bbaf3c23a453e:::
krbtgt:aes256-cts-hmac-sha1-96:a01e82913d40ec80fbbf0375ee087924da27aa77e0ea7cfe83473e68b20e24a8
krbtgt:aes128-cts-hmac-sha1-96:39655377a50d1140853568aaf7b6b076
```

**CRITICAL FINDING:** krbtgt hash (Kerberos master key) obtained. Enables:
- Golden Ticket creation capability
- Extended ticket validity (10+ years potential)
- Long-term domain persistence capability
- Authentication bypass capability

---

### Screenshot 8: Domain Credentials Extracted
**Caption: Complete domain credential dump - all users and machine accounts extracted**

#### Key Credentials Extracted:
- Administrator (Domain Admin)
- Domain Admin (Jadmin)
- Service account (sql_svc)
- Machine account (EC2AMAZ-J75C1G4$)
- All domain users (jsmith, etc.)

#### Security Impact:
- **Pass-the-Hash attacks** - NTLM hashes enable authentication without passwords
- **Privilege reuse** - Domain Admin credentials enable full domain control
- **Credential persistence** - Hashes can be used indefinitely
- **Service account lateral movement** - sql_svc hash enables movement across domain

**Result:** Comprehensive credential compromise across all domain entities.

---

### Screenshot 9: Complete Domain Compromise Confirmed
**Caption: Complete secretsdump extraction showing full domain compromise capability**

**Key Findings:**
- ✅ All local SAM credentials extracted
- ✅ LSA secrets and DPAPI keys compromised
- ✅ NL$KM (cached credential keys) obtained
- ✅ All domain user credentials extracted
- ✅ Administrator credentials compromised
- ✅ krbtgt (Kerberos master key) obtained
- ✅ All machine account credentials extracted
- ✅ Complete Kerberos encryption keys obtained

**Domain Compromise Capability Confirmed:**
This data extraction demonstrates capability for complete domain takeover through multiple attack vectors (Golden Tickets, Pass-the-Hash, credential reuse, privilege delegation).

---

## 💥 Executive Impact Summary

This single misconfiguration (overprivileged service account) created capability for:

* **Domain Administrator-level access** - Potential to control all domain resources
* **Complete credential compromise** - Access to all domain user password hashes
* **Kerberos authentication system compromise** - Master key (krbtgt) obtained
* **Long-term persistence capability** - Golden Ticket creation potential with extended validity
* **Enterprise-wide security impact** - All domain systems at risk from compromised credentials

**Key Insight:** This demonstrates how a single misconfigured service account, combined with common Active Directory configurations, can create an enterprise-wide security risk. The attack required no zero-days or sophisticated exploits—only an understanding of Active Directory architecture and common misconfigurations found in real organizations.

---

## 🔍 Detection Opportunities (Blue Team Perspective)

This attack could be detected using:

* **Kerberos event monitoring** - Event ID 4769 (TGS request) and 4768 (TGT request) anomalies indicate Kerberoasting attempts
* **Suspicious SPN ticket requests** - Multiple TGS requests for service accounts indicate ticket-granting patterns
* **Unusual service account logins** - sql_svc lateral movement to multiple systems in short timeframe
* **LSASS access detection** - Secretsdump attempts access LSASS for credential extraction (detectable via Sysmon Event ID 10)
* **Admin share usage** - ADMIN$ share writes and remote service creation events indicate psexec/wmiexec execution
* **Abnormal authentication patterns** - Service account authenticating from unexpected source IPs
* **Credential access patterns** - Unusual access to NTDS.DIT or LSA secrets via DRSUAPI

**SIEM Correlation Opportunity:** A SIEM solution monitoring multiple events (Kerberos 4769, unusual logon types, service creation, LSASS access) would likely detect multiple stages of this attack chain, enabling early intervention.

**Key Defensive Insight:** While this attack chain is difficult to prevent entirely, multi-stage detection at the enumeration, lateral movement, and credential access phases would allow defenders to interrupt the attack before full compromise.

---

## 🎯 Key Takeaways

* Active Directory misconfigurations create high-impact attack vectors
* Service accounts are frequently overlooked but represent critical security risks
* Lateral movement is often easier than initial access—focus defenses here
* Privilege separation and monitoring are essential for enterprise security
* Attack detection requires understanding both attacker and defender perspectives

---

## 👤 Author

**Rubin Perez**
Cybersecurity Student | Aspiring Ethical Hacker

---
