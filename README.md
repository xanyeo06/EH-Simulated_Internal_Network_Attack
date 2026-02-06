# Ethical Hacking Penetration Testing Assignment  
**Simulated Internal Network Attack (Academic Project)**

> ⚠️ **Disclaimer**  
> This project is an **academic exercise conducted in a controlled lab environment** as part of an Ethical Hacking module.  
> All techniques demonstrated were performed on **authorised virtual machines** created for learning purposes only.  
> **No real-world systems, organisations, or individuals were targeted.**

---

## 📘 Project Overview

This repository documents a **simulated internal network penetration test** conducted as part of an Ethical Hacking assignment.  
The engagement models a realistic multi-stage cyberattack starting from an initial phishing compromise and progressing through lateral movement, privilege escalation, Active Directory compromise, and data exfiltration.

The objective of this project was to:
- Understand attacker tactics, techniques, and procedures (TTPs)
- Demonstrate how a single endpoint compromise can escalate into a full domain breach
- Identify security weaknesses and propose effective countermeasures

---

## 🧪 Scenario Summary

The simulated organisation, **Synergy Dynamics**, is a Singapore-based software firm working with public sector and financial institutions.

The engagement assumes:
- A targeted spear-phishing attack
- Initial compromise of an HR executive workstation
- Escalation across internal systems including file servers and Active Directory
- Exfiltration of sensitive corporate data

---

## 🖥️ Penetration Testing Environment

| Role | Operating System | IP Address |
|----|----|----|
| Attacker | Parrot OS | 192.168.8.100 |
| HR Workstation | Windows 10 | 192.168.8.200 |
| CEO Workstation | Windows 10 | 192.168.8.250 |
| File Server | Windows Server 2019 | 192.168.8.20 |
| Domain Controller | Windows Server 2022 | 192.168.8.30 |

**Network**: Internal LAN (VMnet8 – 192.168.8.0/24)

---

## 🔐 Attack Phases Covered

1. **Initial Access**
   - Spear-phishing email delivering a malicious payload
   - Reverse shell establishment

2. **Post-Exploitation**
   - Host and network reconnaissance
   - Credential harvesting
   - Local data access

3. **Lateral Movement**
   - SMB enumeration
   - Password hash extraction and cracking
   - Remote command execution

4. **Privilege Escalation**
   - Local privilege escalation to SYSTEM
   - Abuse of weak credentials and misconfigurations

5. **Active Directory Compromise**
   - Domain enumeration
   - Pass-the-Hash techniques
   - Domain Admin account creation

6. **Persistence and Cleanup**
   - Registry Run Keys and scheduled tasks
   - Removal of malicious artefacts after demonstration

---

## 🛡️ Defensive Countermeasures

This project also documents mitigation strategies, including:
- Phishing awareness and email security controls
- Endpoint Detection and Response (EDR)
- Least privilege enforcement
- Network segmentation and monitoring
- Active Directory hardening
- Persistence detection techniques

---

## 📂 Repository Contents

- Assignment report and documentation  
- Screenshots and technical evidence  
- Attack flow explanations  
- Security recommendations  

---

## 📜 Ethical Notice

This work adheres strictly to ethical hacking principles:
- Conducted only on authorised systems
- Used exclusively for educational purposes
- Intended to demonstrate defensive lessons, not offensive misuse

---

## 📌 Key Learning Outcomes

- Practical understanding of multi-stage cyberattacks  
- Penetration testing methodology and reporting  
- Risk and impact assessment  
- Importance of layered security controls  
