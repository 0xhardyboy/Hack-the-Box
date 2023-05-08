# Hack the Box - Backdoor

```CSS
Machine IP: 10.10.11.125 - Linux
Difficulty: Easy
Category: OSCP Preparation
Vulnerabilities:
```

# Reconnaissance
  - Scan for open TCP ports on target machine.
  - Perform service and version detection of open ports.

## Port Scan
```CSS
▶ nmap -Pn -sS -O -p- 10.10.11.125 -T4 --min-rate 1000 -oN ports.nmap

Nmap scan report for 10.10.11.125
Host is up (0.18s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
1337/tcp open  waste
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.93%E=4%D=5/8%OT=22%CT=1%CU=31646%PV=Y%DS=2%DC=I%G=Y%TM=645931ED
OS:%P=x86_64-pc-linux-gnu)SEQ(SP=103%GCD=1%ISR=10A%TI=Z%CI=Z%II=I%TS=A)OPS(
OS:O1=M53CST11NW7%O2=M53CST11NW7%O3=M53CNNT11NW7%O4=M53CST11NW7%O5=M53CST11
OS:NW7%O6=M53CST11)WIN(W1=FE88%W2=FE88%W3=FE88%W4=FE88%W5=FE88%W6=FE88)ECN(
OS:R=Y%DF=Y%T=40%W=FAF0%O=M53CNNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS
OS:%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=
OS:Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=
OS:R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T
OS:=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=
OS:S)

Network Distance: 2 hops

OS detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 81.95 seconds
```

## Service and Version Detection
```CSS
▶ nmap -sC -sV -p 22,80,1337 10.10.11.125 -oN services.nmap

Nmap scan report for 10.10.11.125                                                                                                                                           
Host is up (0.18s latency).                                                           
                                           
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:        
|   3072 b4de43384657db4c213b69f3db3c6288 (RSA)
|   256 aac9fc210f3ef4ec6b3570262253ef66 (ECDSA)
|_  256 d28be4ec0761aacaf8ec1cf88cc1f6e1 (ED25519)
80/tcp   open  http    Apache httpd 2.4.41 ((Ubuntu))                                                                                                                       
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Backdoor &#8211; Real-Life                                              
|_http-generator: WordPress 5.8.1                                                     
1337/tcp open  waste?                                                                 
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel                    
                                                                                      
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 22.51 seconds 
```
![image](https://user-images.githubusercontent.com/83878909/236892792-dee7a77f-2f96-40f7-9375-75b642a52357.png)

---

# Enumeration
## Port 22 - SSH
  - The OpenSSH version that is running is not associated with any critical vulnerabilities, so it’s unlikely to gain initial access through this port, unless some valid credentials are found.

---

## Port 80 HTTP
  - The webpage reveals no interesting information.
![image](https://user-images.githubusercontent.com/83878909/236893295-d3d776d2-ebb0-4c40-899d-0b6f95c26995.png)

  - The `nmap` result shows that the server is running wordpress.
![image](https://user-images.githubusercontent.com/83878909/236892706-e1cebe1f-b77f-47bc-8af8-89369fbf0c86.png)

  - Run `wpscan` on the target.
```CSS
▶ wpscan
```