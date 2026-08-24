# Hack The Box (HTB) Writeups & Penetration Testing Portfolio

A collection of structured penetration testing writeups focusing on enumeration, exploitation vectors, and defensive remediation strategies.

## 📋 Target Matrix

| Machine / Target | OS | Difficulty | Primary Attack Vector / Vulnerability | Key Tools Used |
| :--- | :--- | :--- | :--- | :--- |
| **Unified** | Windows | Easy | Log4j / CVE-2021-44228 RCE | Nmap, Burp Suite, Metasploit |
| **Archetype** | Windows | Easy | MSSQL Service Abuse & Credential Reuse | Nmap, Impacket, impacket-mssqlclient |
| **Vaccine** | Linux | Easy | SQL Injection & Local Privilege Escalation | Nmap, sqlmap, Gobuster |
| **Responder** | Windows | Easy | LLMNR/NBT-NS Poisoning & Credential Capture | Responder, CrackMapExec |
| **Three** | Linux | Easy | AWS S3 Bucket Misconfiguration & RCE | Gobuster, AWS CLI |
| **Oopsie** | Linux | Easy | IDOR & Insecure Direct Object References | Burp Suite, FoxyProxy |

---

## 🛡️ Methodology & Approach
Each writeup in this section follows a structured, consultant-grade format:
1. **Executive Summary:** High-level overview of the target's security posture.
2. **Attack Path:** Step-by-step breakdown from initial reconnaissance to root/SYSTEM access.
3. **Remediation:** Actionable security recommendations on how to patch and defend against the exploited vulnerabilities.
