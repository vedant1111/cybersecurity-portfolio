# Hack The Box: Redeemer

* **OS:** Linux
* **Difficulty:** Very Easy (Starting Point Tier 0)
* **Target Service:** Redis (Port 6379)

## 1. Reconnaissance
Ran an Nmap scan targeting all TCP ports on the host:
```bash
nmap -p- --min-rate 5000 10.129.249.162

Result: Port 6379/tcp is open, running redis (In-Memory Data Store).

## 2. Enumeration & Exploitation
Connected directly to the unauthenticated Redis instance using redis-cli:

redis-cli -h 10.129.249.162

Ran internal Redis commands to query system status and inspect database keys:

10.129.x.x:6379> info
10.129.x.x:6379> keys *

Discovered a key named flag.

## 3. Flag Retrieval
Retrieved the value assigned to the flag key:

10.129.x.x:6379> get flag

## 4.Remediation
* Require authentication by setting a strong password using the requirepass directive in redis.conf.

* Bind the Redis service only to local/internal interfaces (bind 127.0.0.1) to prevent external exposure over the public internet.

* Restrict access to port 6379 via firewall rules (e.g., iptables or ufw).