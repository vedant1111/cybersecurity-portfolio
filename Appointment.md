# Hack The Box: Appointment

* **OS:** Linux
* **Difficulty:** Very Easy (Starting Point Tier 1)
* **Target Service:** HTTP (Port 80)

## 1. Reconnaissance
Ran an Nmap scan to identify running services on the target IP:
```bash
nmap -sV 10.129.249.209

Result: Port 80/tcp is open, running http (Apache httpd).

2. Enumeration & Web Inspection
Navigated to http://10.129.249.209 in the web browser and was presented with an administrative login portal.

3. Exploitation (Authentication Bypass via SQL Injection)
Entered the following payload into the login form:

Username: admin' #

Password: 1234

4. Flag Retrieval
Successfully bypassed authentication and landed on the admin dashboard, where the flag was exposed directly on the page.

5. Remediation
Use Prepared Statements (Parameterized Queries) for all database queries rather than concatenating user input directly into SQL strings.

Implement input validation and sanitization on all web forms.

Apply the Principle of Least Privilege to database user accounts.

