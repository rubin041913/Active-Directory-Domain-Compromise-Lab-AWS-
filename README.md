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

> (Include screenshots here with captions)

Example captions:

* Enumerating SMB shares and domain users
* Successful remote execution via wmiexec
* SYSTEM-level shell access
* Extracted NTLM hashes using secretsdump
* Domain Administrator compromise

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
