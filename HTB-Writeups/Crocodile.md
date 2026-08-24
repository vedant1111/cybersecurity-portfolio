# Hack The Box: Crocodile

* **OS:** Linux
* **Difficulty:** Very Easy (Starting Point Tier 1)
* **Target Service:** FTP (Port 21) & HTTP (Port 80)

## 1. Reconnaissance
Ran an Nmap scan with default scripts and version detection to identify running services:

nmap -sC -sV <TARGET_IP>

```

**Result:** Port 21/tcp is open running `vsftpd 3.0.3` and allows Anonymous FTP login. Port 80/tcp is open running `Apache httpd 2.4.41`.

## 2. Enumeration & Access

Connected to the FTP server using anonymous credentials (username: `anonymous`, blank password):


ftp <TARGET_IP>


Listed the directory contents and downloaded two exposed files:

* `allowed.userlist`
* `allowed.userlist.passwd`

Upon inspecting the files locally, mapped the username `admin` to the password `rKXM59ESxesUFHAd`.

## 3. Web Exploitation

Navigated to the web server's administrative login page at `http://<TARGET_IP>/login.php`. Entered the compromised credentials:

* **Username:** `admin`
* **Password:** `rKXM59ESxesUFHAd`

## 4. Flag Retrieval

Successfully authenticated and accessed the Server Management dashboard, where the root flag was displayed directly on the page.

## 5. Remediation

* Disable **Anonymous Login** on the FTP server to prevent unauthorized users from viewing or downloading files.
* Never store plaintext passwords or credential lists in exposed or insecure network directories.
* Avoid password reuse across different services (FTP file context leading to Web App access).

