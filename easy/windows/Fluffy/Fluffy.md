
| Property         | Value                    |
| ---------------- | ------------------------ |
| **OS**           | Windows                  |
| **Difficulty**   | Easy                     |
| **Release Date** | 24th May, 2025           |
| **State**        | Retired                  |
| **Techniques**   | technique-1, technique-2 |
| **Tags**         | #web #privesc #linux     |

---
## Summary

Brief 2-3 sentence ogverview of the machine and attack path.

---
## Enumeration

initial creds:

`j.fleischman / J0elTHEM4n1990!`

### Nmap Scan

```
nmap -sC -sV fluffy.htb --open                            
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-12 11:52 EDT
Nmap scan report for fluffy.htb (10.129.232.88)
Host is up (0.030s latency).
Not shown: 989 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-09-12 22:52:57Z)
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: fluffy.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.fluffy.htb, DNS:fluffy.htb, DNS:FLUFFY
| Not valid before: 2026-04-30T16:09:59
|_Not valid after:  2106-04-30T16:09:59
|_ssl-date: 2026-09-12T22:54:17+00:00; +7h00m02s from scanner time.
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: fluffy.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.fluffy.htb, DNS:fluffy.htb, DNS:FLUFFY
| Not valid before: 2026-04-30T16:09:59
|_Not valid after:  2106-04-30T16:09:59
|_ssl-date: 2026-09-12T22:54:18+00:00; +7h00m02s from scanner time.
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: fluffy.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.fluffy.htb, DNS:fluffy.htb, DNS:FLUFFY
| Not valid before: 2026-04-30T16:09:59
|_Not valid after:  2106-04-30T16:09:59
|_ssl-date: 2026-09-12T22:54:17+00:00; +7h00m02s from scanner time.
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: fluffy.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.fluffy.htb, DNS:fluffy.htb, DNS:FLUFFY
| Not valid before: 2026-04-30T16:09:59
|_Not valid after:  2106-04-30T16:09:59
|_ssl-date: 2026-09-12T22:54:18+00:00; +7h00m02s from scanner time.
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2026-09-12T22:53:41
|_  start_date: N/A
|_clock-skew: mean: 7h00m01s, deviation: 0s, median: 7h00m01s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 93.10 seconds

```

```
nmap -p- --open fluffy.htb    
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-12 11:55 EDT
Nmap scan report for fluffy.htb (10.129.232.88)
Host is up (0.059s latency).
Not shown: 65516 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE
53/tcp    open  domain
88/tcp    open  kerberos-sec
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
49689/tcp open  unknown
49690/tcp open  unknown
49698/tcp open  unknown
49714/tcp open  unknown
49727/tcp open  unknown
49749/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 204.04 seconds
                                                                                                                
┌──(kali㉿kali)-[~/machines/fluffy]
└─$ nmap -p 9389 -sC -sV --open fluffy.htb 
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-12 11:59 EDT
Nmap scan report for fluffy.htb (10.129.232.88)
Host is up (0.036s latency).

PORT     STATE SERVICE VERSION
9389/tcp open  mc-nmf  .NET Message Framing
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.03 seconds
                                                                     
```

### SMB Enumeration

Shares:

```
nxc smb fluffy.htb -u 'j.fleischman' -p 'J0elTHEM4n1990!' --shares
SMB         10.129.232.88   445    DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:fluffy.htb) (signing:True) (SMBv1:False)
SMB         10.129.232.88   445    DC01             [+] fluffy.htb\j.fleischman:J0elTHEM4n1990! 
SMB         10.129.232.88   445    DC01             [*] Enumerated shares
SMB         10.129.232.88   445    DC01             Share           Permissions     Remark
SMB         10.129.232.88   445    DC01             -----           -----------     ------
SMB         10.129.232.88   445    DC01             ADMIN$                          Remote Admin
SMB         10.129.232.88   445    DC01             C$                              Default share
SMB         10.129.232.88   445    DC01             IPC$            READ            Remote IPC
SMB         10.129.232.88   445    DC01             IT              READ,WRITE      
SMB         10.129.232.88   445    DC01             NETLOGON        READ            Logon server share 
SMB         10.129.232.88   445    DC01             SYSVOL          READ            Logon server share
```

```
smbclient -U j.fleischman //fluffy.htb/IT --password='J0elTHEM4n1990!'
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sat Sep 12 18:57:28 2026
  ..                                  D        0  Sat Sep 12 18:57:28 2026
  Everything-1.4.1.1026.x64           D        0  Fri Apr 18 11:08:44 2025
  Everything-1.4.1.1026.x64.zip       A  1827464  Fri Apr 18 11:04:05 2025
  KeePass-2.58                        D        0  Fri Apr 18 11:08:38 2025
  KeePass-2.58.zip                    A  3225346  Fri Apr 18 11:03:17 2025
  Upgrade_Notice.pdf                  A   169963  Sat May 17 10:31:07 2025

```

Users:

```
nxc smb fluffy.htb -u 'j.fleischman' -p 'J0elTHEM4n1990!' --users 
SMB         10.129.232.88   445    DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:fluffy.htb) (signing:True) (SMBv1:False)
SMB         10.129.232.88   445    DC01             [+] fluffy.htb\j.fleischman:J0elTHEM4n1990! 
SMB         10.129.232.88   445    DC01             -Username-                    -Last PW Set-       -BadPW- -Description-
SMB         10.129.232.88   445    DC01             Administrator                 2025-04-17 15:45:01 0       Built-in account for administering the computer/domain
SMB         10.129.232.88   445    DC01             Guest                         <never>             0       Built-in account for guest access to the computer/domain
SMB         10.129.232.88   445    DC01             krbtgt                        2025-04-17 16:00:02 0       Key Distribution Center Service Account
SMB         10.129.232.88   445    DC01             ca_svc                        2025-04-17 16:07:50 0        
SMB         10.129.232.88   445    DC01             ldap_svc                      2025-04-17 16:17:00 0        
SMB         10.129.232.88   445    DC01             p.agila                       2025-04-18 14:37:08 0        
SMB         10.129.232.88   445    DC01             winrm_svc                     2025-05-18 00:51:16 0        
SMB         10.129.232.88   445    DC01             j.coffey                      2025-04-19 12:09:55 0        
SMB         10.129.232.88   445    DC01             j.fleischman                  2025-05-16 14:46:55 0        
SMB         10.129.232.88   445    DC01             [*] Enumerated 9 local users: FLUFFY
                                                                                         
```

### Bloodhound enumeration

```
sudo bloodhound-python -u 'j.fleischman' -p 'J0elTHEM4n1990!' -ns 10.129.232.88 -d fluffy.htb -c all --zip
INFO: BloodHound.py for BloodHound LEGACY (BloodHound 4.2 and 4.3)
INFO: Found AD domain: fluffy.htb
INFO: Getting TGT for user
WARNING: Failed to get Kerberos TGT. Falling back to NTLM authentication. Error: Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)
INFO: Connecting to LDAP server: dc01.fluffy.htb
INFO: Testing resolved hostname connectivity dead:beef::afbe:3842:e536:2e96
INFO: Trying LDAP connection to dead:beef::afbe:3842:e536:2e96
INFO: Testing resolved hostname connectivity dead:beef::128
INFO: Trying LDAP connection to dead:beef::128
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 1 computers
INFO: Connecting to LDAP server: dc01.fluffy.htb
INFO: Testing resolved hostname connectivity dead:beef::afbe:3842:e536:2e96
INFO: Trying LDAP connection to dead:beef::afbe:3842:e536:2e96
INFO: Testing resolved hostname connectivity dead:beef::128
INFO: Trying LDAP connection to dead:beef::128
INFO: Found 10 users
INFO: Found 54 groups
INFO: Found 3 gpos
INFO: Found 1 ous
INFO: Found 19 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: DC01.fluffy.htb
INFO: Done in 00M 08S
INFO: Compressing output into 20260912120218_bloodhound.zip

```

enumerating the pdf discloses some cve

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