# Hack The Box: Meow

* **OS:** Linux
* **Difficulty:** Very Easy (Starting Point Tier 0)
* **Target Service:** Telnet (Port 23)

## 1. Reconnaissance
Ran an Nmap scan to identify open ports on the target IP:
```bash
nmap -sV <TARGET_IP>

Result: Port 23/tcp is open, running telnet.

2. Exploitation
Connected to the target using the Telnet client:

telnet <TARGET_IP> 23

When prompted for a login username, entered root. Access was granted without a password.

3. Flag Retrieval
Listed the directory contents and read the flag:

ls
cat flag.txt

4. Remediation : 
Disable Telnet and replace it with SSH (sshd), which encrypts network traffic.
Enforce strong password authentication or SSH keys for the root account.
