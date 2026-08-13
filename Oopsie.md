# Penetration Test Report: Hack The Box - Oopsie

## Executive Summary

This report details the penetration test of the "Oopsie" machine on the Hack The Box platform. The assessment identified severe Broken Access Control vulnerabilities within the target's web application, specifically Insecure Direct Object References (IDOR). This flaw was leveraged to bypass authentication barriers and upload a malicious PHP payload, resulting in a reverse shell. Subsequent local enumeration revealed a misconfigured SUID binary that was vulnerable to path manipulation, allowing for privilege escalation to the root user.

## Reconnaissance & Enumeration

The engagement began with an `nmap` scan to identify exposed services on the target server.

* **Port 22 (SSH):** Open, but no credentials were known.
* **Port 80 (HTTP):** Hosting the MegaCorp Automotive website.

Manual enumeration of the web application revealed an email address (`admin@megacorp.com`). Further inspection of the application's source code and network requests isolated a hidden administrative portal at `/cdn-cgi/login/`.

## Web Exploitation & IDOR

Connecting to the hidden portal allowed for "Guest" login functionality. Once authenticated as a guest, traffic interception and manipulation (facilitated via Burp Suite and browser proxy tools) were utilized to analyze session mechanics.

The application relied on insecure client-side cookies and URL parameters to determine user identity and access levels.

* **The IDOR Vulnerability:** By modifying the user ID parameter in the HTTP request from the guest ID (2233) to the administrator ID (1), the application granted unauthorized access to the admin dashboard.
* **Cookie Manipulation:** Elevating the session's role cookie to `admin` granted access to a restricted file upload portal designated for internal administrators.

## Initial Access

With administrative access to the upload portal secured, a PHP reverse shell payload was configured to connect back to a local Netcat listener on port 1337.

The payload (`shell.php`) was successfully uploaded to the web server and executed by navigating to its absolute path at `/uploads/shell.php`. This triggered the reverse connection, granting an interactive shell as the `www-data` service account.

**Lateral Movement**
While exploring the web directory structure (`/var/www/html/cdn-cgi/login`), hardcoded database credentials were discovered within the backend PHP files. These credentials were successfully reused to switch users via the `su robert` command, securing standard user privileges and capturing the `user.txt` flag.

## Privilege Escalation

With a stable foothold as `robert`, local enumeration was conducted to identify privilege escalation vectors. A search for binaries with the SUID bit set revealed a custom executable named `bugtracker`, owned by the `root` user.

Analysis of the `bugtracker` binary behavior revealed that it executed the standard system command `cat` to read log files, but failed to specify the absolute path (e.g., `/bin/cat`). This created a severe Path Hijacking vulnerability.

**Path Manipulation Exploit**

1. A malicious script named `cat` was created in the `/tmp` directory. This script contained the command `/bin/sh` to spawn a shell.
2. The script was made executable: `chmod +x /tmp/cat`
3. The system's `PATH` environment variable was modified to prioritize the `/tmp` directory: `export PATH=/tmp:$PATH`

When the `bugtracker` executable was launched, it attempted to run `cat`. Due to the manipulated `PATH`, the system executed the malicious `/tmp/cat` script instead of the legitimate binary. Because `bugtracker` was running with SUID root privileges, the injected `/bin/sh` command spawned a root shell, leading to full system compromise and the retrieval of the `root.txt` flag.