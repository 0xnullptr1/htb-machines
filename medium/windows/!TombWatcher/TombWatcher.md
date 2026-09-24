
| Property         | Value                         |
| ---------------- | ----------------------------- |
| **OS**           | Linux / Windows               |
| **Difficulty**   | Easy / Medium / Hard / Insane |
| **Release Date** | YYYY-MM-DD                    |
| **State**        | YYYY-MM-DD                    |
| **IP**           | 10.10.10.X                    |
| **Techniques**   | technique-1, technique-2      |
| **Tags**         | #web #privesc #linux          |

---
## Summary

Brief 2-3 sentence ogverview of the machine and attack path.

---
## Enumeration

### Nmap Scan

```
nmap -sC -sV tombwatcher.htb --open
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-24 03:23 EDT
Nmap scan report for tombwatcher.htb (10.129.144.23)
Host is up (0.053s latency).
Not shown: 987 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: IIS Windows Server
|_http-server-header: Microsoft-IIS/10.0
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-09-24 11:23:41Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: tombwatcher.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2026-09-24T11:25:03+00:00; +4h00m02s from scanner time.
| ssl-cert: Subject: commonName=DC01.tombwatcher.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.tombwatcher.htb
| Not valid before: 2024-11-16T00:47:59
|_Not valid after:  2025-11-16T00:47:59
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: tombwatcher.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.tombwatcher.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.tombwatcher.htb
| Not valid before: 2024-11-16T00:47:59
|_Not valid after:  2025-11-16T00:47:59
|_ssl-date: 2026-09-24T11:25:02+00:00; +4h00m01s from scanner time.
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: tombwatcher.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.tombwatcher.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.tombwatcher.htb
| Not valid before: 2024-11-16T00:47:59
|_Not valid after:  2025-11-16T00:47:59
|_ssl-date: 2026-09-24T11:25:03+00:00; +4h00m02s from scanner time.
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: tombwatcher.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.tombwatcher.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.tombwatcher.htb
| Not valid before: 2024-11-16T00:47:59
|_Not valid after:  2025-11-16T00:47:59
|_ssl-date: 2026-09-24T11:25:02+00:00; +4h00m01s from scanner time.
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2026-09-24T11:24:23
|_  start_date: N/A
|_clock-skew: mean: 4h00m01s, deviation: 0s, median: 4h00m00s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 93.40 seconds
                                                                
```

```
 nmap -p- tombwatcher.htb --open
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-24 03:23 EDT
Nmap scan report for tombwatcher.htb (10.129.144.23)
Host is up (0.051s latency).
Not shown: 65514 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
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
49666/tcp open  unknown
49695/tcp open  unknown
49696/tcp open  unknown
49698/tcp open  unknown
49716/tcp open  unknown
49727/tcp open  unknown
53575/tcp open  unknown

Nmap done: 1 IP address (1 host up) sc
```

### Service Enumeration

```
nxc smb tombwatcher.htb -u henry -p 'H3nry_987TGV!' --shares

SMB         10.129.144.23   445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:tombwatcher.htb) (signing:True) (SMBv1:False) 
SMB         10.129.144.23   445    DC01             [+] tombwatcher.htb\henry:H3nry_987TGV! 
SMB         10.129.144.23   445    DC01             [*] Enumerated shares
SMB         10.129.144.23   445    DC01             Share           Permissions     Remark
SMB         10.129.144.23   445    DC01             -----           -----------     ------
SMB         10.129.144.23   445    DC01             ADMIN$                          Remote Admin
SMB         10.129.144.23   445    DC01             C$                              Default share
SMB         10.129.144.23   445    DC01             IPC$            READ            Remote IPC
SMB         10.129.144.23   445    DC01             NETLOGON        READ            Logon server share 
SMB         10.129.144.23   445    DC01             SYSVOL          READ            Logon server share 
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/machines/tombwatcher]
└─$ 
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/machines/tombwatcher]
└─$ nxc smb tombwatcher.htb -u henry -p 'H3nry_987TGV!' --users 
SMB         10.129.144.23   445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:tombwatcher.htb) (signing:True) (SMBv1:False) 
SMB         10.129.144.23   445    DC01             [+] tombwatcher.htb\henry:H3nry_987TGV! 
SMB         10.129.144.23   445    DC01             -Username-                    -Last PW Set-       -BadPW- -Description-                                               
SMB         10.129.144.23   445    DC01             Administrator                 2025-04-25 14:56:03 0       Built-in account for administering the computer/domain 
SMB         10.129.144.23   445    DC01             Guest                         <never>             0       Built-in account for guest access to the computer/domain 
SMB         10.129.144.23   445    DC01             krbtgt                        2024-11-16 00:02:28 0       Key Distribution Center Service Account 
SMB         10.129.144.23   445    DC01             Henry                         2025-05-12 15:17:03 0        
SMB         10.129.144.23   445    DC01             Alfred                        2025-05-12 15:17:03 0        
SMB         10.129.144.23   445    DC01             sam                           2025-05-12 15:17:03 0        
SMB         10.129.144.23   445    DC01             john                          2025-05-19 13:25:10 0        
SMB         10.129.144.23   445    DC01             [*] Enumerated 7 local users: TOMBWATCHER

```

---
## Foothold

How you gained initial access to the machine.

### Vulnerability

Description of the vulnerability exploited.

### Exploitation

Step-by-step exploitation with commands.

```shell
# Commands used
```

---
## User Flag

### Lateral Movement (if applicable)

Steps to move from initial foothold to user access.

---
## Privilege Escalation

### Enumeration

What you found that leads to root/admin.

### Exploitation

Step-by-step privilege escalation.

```shell
# Commands used
```

---
## Remediation

- Key takeaway 1
- Key takeaway 2
- Key takeaway 3

---
## References

- [Reference 1](https://github.com/momenbasel/htb-writeups/blob/main/templates/url)
- [Reference 2](https://github.com/momenbasel/htb-writeups/blob/main/templates/url)