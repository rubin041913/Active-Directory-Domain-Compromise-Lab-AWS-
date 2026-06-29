# Active Directory Domain Compromise Lab (AWS)

![Badge](https://img.shields.io/badge/Environment-AWS%20EC2-orange) ![Badge](https://img.shields.io/badge/Difficulty-Advanced-red) ![Badge](https://img.shields.io/badge/Educational-Yes-blue)

## 📌 Overview

This repository documents a comprehensive **Active Directory (AD) domain compromise scenario** in an AWS environment. The lab demonstrates real-world attack techniques, misconfigurations, and exploitation methods used during internal penetration tests and red team engagements.

The attack chain simulates a complete compromise path: from initial enumeration through privilege escalation to full domain credential extraction and persistence.

> **⚠️ Disclaimer:** This project was conducted in a controlled lab environment for educational and authorized testing purposes only. These techniques should only be used in authorized penetration testing engagements and controlled lab environments.

---

## 🎯 Objectives

This lab achieves the following security objectives:

- ✅ Enumerate Active Directory users, services, and infrastructure
- ✅ Identify high-value service accounts and misconfigurations
- ✅ Gain remote code execution (RCE) on domain-joined systems
- ✅ Escalate privileges to SYSTEM level
- ✅ Extract domain credentials from the Domain Controller
- ✅ Demonstrate complete domain compromise capabilities

---

## 🏗️ Lab Environment

| Component | Specification |
|-----------|---------------|
| **Domain Controller** | Windows Server 2019/2022 (AWS EC2) |
| **Attacker Machine** | Kali Linux |
| **Domain Name** | corp.local |
| **Key Services** | SMB, Kerberos, LDAP, WMI, MSSQL |
| **Cloud Provider** | Amazon Web Services (AWS) |

---

## 🧰 Tools & Technologies

### Exploitation Tools
- **Impacket** - GetUserSPNs, psexec, wmiexec, secretsdump
- **Kerberos Tools** - kinit, klist, TicketConverter
- **SMB Utilities** - smbclient, rpcclient

### Operating Systems
- Kali Linux (Attacker)
- Windows Server (Domain Controller)

### Cloud Infrastructure
- AWS EC2 instances
- AWS VPC networking
- AWS Security Groups

---

## 🔍 Attack Chain

### Phase 1: Reconnaissance & Enumeration

**Objective:** Discover and map the Active Directory environment

```bash
# Service Principal Name (SPN) Enumeration
impacket-GetUserSPNs corp.local/username:password

# Results:
# - Identified service account: sql_svc
# - Associated service: MSSQL
# - Critical finding: Member of Domain Admins group
```

**Key Findings:**
- Discovered service account with excessive privileges
- Identified MSSQL service running under privileged account
- Mapped domain structure and user accounts

---

### Phase 2: Credential Access

**Objective:** Obtain valid credentials for the identified service account

**Attack Vectors Attempted:**
1. Kerberoasting (SPN enumeration and TGS requests)
2. AS-REP Roasting
3. Credential harvesting

**Challenges Encountered & Resolution:**
- Initial Kerberos encryption errors
- Time synchronization issues between attacker and domain
- DNS resolution configuration

**Result:** Successfully obtained valid `sql_svc` credentials

---

### Phase 3: Remote Code Execution

**Objective:** Execute commands on the target system

```bash
# Method 1: PSExec (SMB-based execution)
impacket-psexec corp.local/sql_svc:password@target-ip cmd.exe

# Method 2: WMIExec (WMI-based execution)
impacket-wmiexec corp.local/sql_svc:password@target-ip cmd.exe
```

**Achieved:**
- Semi-interactive shell access
- Command execution with service account privileges
- Remote system interaction capabilities

---

### Phase 4: Privilege Escalation

**Objective:** Escalate from service account to SYSTEM level

**Privilege Analysis:**
- Service account (`sql_svc`) membership audit
- Group Policy evaluation
- Token privileges enumeration

**Exploitation:**
- Service account already member of **Domain Admins**
- Execution context allows SYSTEM-level token impersonation
- Token manipulation for privilege escalation

**Result:** SYSTEM-level access achieved

---

### Phase 5: Credential Dumping

**Objective:** Extract all domain credentials and security artifacts

```bash
# Credential extraction from Domain Controller
impacket-secretsdump corp.local/sql_svc:password@dc-ip -outputfile hashes
```

**Extracted Credentials:**
- ✅ Local SAM hashes
- ✅ Domain user NTLM hashes
- ✅ Kerberos session keys
- ✅ **krbtgt account hash** (Critical)
- ✅ Administrator account hash

**Data Retrieved:**
```
Administrator NTLM: [HASH]
krbtgt NTLM: [HASH]
All domain user credentials
DCsync capable
```

---

### Phase 6: Domain Compromise & Persistence

**Objective:** Achieve complete domain compromise and establish persistence

**Capabilities Enabled:**
- **Pass-the-Hash (PtH) attacks** - Authenticate with NTLM hashes
- **Golden Ticket creation** - Forge Kerberos tickets using krbtgt hash
- **Persistent access** - Leverage compromised credentials for ongoing access
- **Lateral movement** - Access all domain-joined systems
- **Credential delegation** - Impersonate any domain user

**Persistence Methods:**
- Kerberos Golden Tickets (valid for 10 years)
- Backdoor service accounts
- Scheduled task persistence
- WMI event subscription persistence

---

## 📊 Key Findings & Vulnerabilities

| Vulnerability | Severity | Impact |
|---------------|----------|--------|
| Service account in Domain Admins | **CRITICAL** | Full domain compromise |
| Weak account segmentation | **CRITICAL** | Unrestricted lateral movement |
| Excessive service privileges | **HIGH** | RCE with admin rights |
| No monitoring on credential access | **HIGH** | Undetected dumping |
| NTLM relay vulnerability | **HIGH** | Alternative compromise path |

---

## 🛡️ Defensive Recommendations

### Immediate Actions (Critical)
1. **Remove service accounts from Domain Admins group**
   - Implement least privilege principle
   - Use service-specific groups instead

2. **Enable Credential Guard**
   - Protect credentials in isolated container
   - Prevent credential dumping attacks

3. **Implement LAPS** (Local Administrator Password Solution)
   - Randomize local admin passwords
   - Prevent credential reuse

### Short-term Mitigations (High Priority)
4. **Enforce strong authentication**
   - Require multi-factor authentication (MFA)
   - Implement passwordless authentication

5. **Monitor Kerberos activity**
   - Alert on unusual TGS requests
   - Monitor for reconnaissance activity

6. **Implement tiered admin model**
   - Dedicated admin workstations
   - Separate credentials per tier

### Long-term Hardening
7. **Enable Windows Defender for Endpoint**
   - Behavioral analysis and threat detection
   - EDR capabilities

8. **Regular security audits**
   - AD security assessments
   - Privilege access reviews

9. **Implement detective controls**
   - Enable advanced audit logging
   - Deploy SIEM for correlation

---

## 📁 Repository Structure

```
Active-Directory-Domain-Compromise-Lab-AWS-/
├── README.md                      # This file
├── screenshots/                   # Evidence and proof-of-concept images
│   ├── 01_spn_enumeration.png
│   ├── 02_rce_shell.png
│   ├── 03_credential_dump.png
│   └── 04_domain_compromise.png
├── scripts/                       # Automation and helper scripts
│   ├── enumeration.sh
│   ├── exploit.sh
│   ├── persistence.sh
│   └── cleanup.sh
├── documentation/                 # Detailed technical write-ups
│   ├── attack_chain.md
│   ├── tools_guide.md
│   ├── forensics.md
│   └── incident_response.md
└── artifacts/                     # Example outputs and logs
    ├── spn_list.txt
    ├── credentials.txt
    └── event_logs/
```

---

## 🚀 Quick Start

### Prerequisites
- AWS account with appropriate permissions
- Kali Linux or penetration testing distribution
- Impacket suite installed
- Network access between lab components

### Setup Instructions

1. **Clone the repository:**
   ```bash
   git clone https://github.com/rubin041913/Active-Directory-Domain-Compromise-Lab-AWS-.git
   cd Active-Directory-Domain-Compromise-Lab-AWS-
   ```

2. **Deploy AWS infrastructure:**
   ```bash
   # See documentation/setup.md for CloudFormation templates
   aws cloudformation create-stack --stack-name ad-lab --template-body file://infrastructure.yaml
   ```

3. **Configure attacker machine:**
   ```bash
   ./scripts/setup_attacker.sh
   ```

4. **Execute attack chain:**
   ```bash
   ./scripts/enumeration.sh
   ./scripts/exploit.sh
   ```

---

## 📚 Learning Outcomes

After completing this lab, you will understand:

- **Active Directory Architecture** - Domain structure, replication, security
- **Kerberos Protocol** - TGT/TGS, SPN enumeration, ticket manipulation
- **Exploitation Techniques** - Kerberoasting, Pass-the-Hash, Golden Tickets
- **Lateral Movement** - Tools and techniques for domain traversal
- **Privilege Escalation** - Service account exploitation, token manipulation
- **Post-Exploitation** - Credential dumping, persistence, covering tracks
- **Defense Mechanisms** - Detection, prevention, and response strategies

---

## 🔐 Security Considerations

### For Lab Users
- Use isolated VPC with no internet access
- Implement network segmentation
- Monitor all activities with logging
- Clean up resources after testing
- Never use production AD credentials

### Ethical Considerations
- Only conduct in authorized environments
- Document all testing activities
- Report findings through proper channels
- Respect system owners' rights
- Follow organization's policies and procedures

---

## 📖 Additional Resources

### Recommended Reading
- [Microsoft AD Security Best Practices](https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/best-practices-for-securing-active-directory)
- [MITRE ATT&CK Framework](https://attack.mitre.org/) - Enterprise AD attacks
- [Impacket Documentation](https://github.com/fortra/impacket)

### Related Projects
- Active Directory Lab (On-Premises)
- Exchange Server Exploitation Lab
- Domain Controller Hardening Guide

---

## 💡 Skills Demonstrated

| Category | Skills |
|----------|--------|
| **Offensive** | Enumeration, exploitation, lateral movement, privilege escalation |
| **Defensive** | Hardening, monitoring, detection, incident response |
| **Cloud** | AWS EC2, VPC, security groups, infrastructure |
| **Tools** | Impacket, Kerberos utilities, SMB tools, PowerShell |
| **Protocols** | Kerberos, LDAP, SMB, WMI, RPC |

---

## 📸 Evidence Gallery

Screenshots demonstrating key attack phases:

- **SPN Enumeration Output** - Service account discovery
- **Remote Shell Access** - Successful RCE via wmiexec
- **Command Execution** - Privilege escalation confirmation
- **Credential Dump** - secretsdump output with domain credentials
- **Golden Ticket** - Kerberos ticket generation and usage

*(See `/screenshots` directory for detailed evidence)*

---

## 🧠 Lessons Learned

1. **Service accounts are high-value targets** - Compromise often leads to domain-wide issues
2. **Privilege segregation is critical** - Service accounts should not be in privileged groups
3. **Kerberos requires proper configuration** - Time sync and DNS are essential
4. **Credential protection is paramount** - Implement LAPS, Credential Guard, and LSA protection
5. **Defense in depth is necessary** - Multiple controls prevent complete compromise
6. **Monitoring enables detection** - Early detection prevents damage
7. **Regular audits are essential** - Proactive security reviews identify risks

---

## 🏁 Conclusion

This lab demonstrates how a single misconfigured service account can lead to **complete Active Directory compromise**. The attack chain presented reflects real-world threat actor methodologies and illustrates why proper Active Directory hardening and least-privilege implementation are critical for enterprise security.

The techniques covered in this project are frequently encountered during:
- Internal penetration tests
- Red team engagements
- Security awareness training
- Incident response investigations

### Key Takeaway
**Compromise is not a matter of if, but when. Defense-in-depth, monitoring, and rapid detection are essential for maintaining a secure Active Directory environment.**

---

## 📝 License & Attribution

This educational project is provided as-is for authorized security testing and learning purposes.

---

## ✉️ Contact & Collaboration

- **Author:** rubin041913
- **GitHub:** [@rubin041913](https://github.com/rubin041913)

For questions, suggestions, or contributions, please open an issue or pull request.

---

**Last Updated:** June 29, 2026

*Remember: With great power comes great responsibility. Use this knowledge ethically and legally.*