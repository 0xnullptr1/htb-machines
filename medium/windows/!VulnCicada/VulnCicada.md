
| Property         | Value                    |
| ---------------- | ------------------------ |
| **OS**           | Windows                  |
| **Difficulty**   | Medium                   |
| **Release Date** | YYYY-MM-DD               |
| **State**        | YYYY-MM-DD               |
| **IP**           | 10.10.10.X               |
| **Techniques**   | technique-1, technique-2 |
| **Tags**         | #web #privesc #linux     |

---
## Summary

Brief 2-3 sentence ogverview of the machine and attack path.

---
## Enumeration

### Nmap Scan

```
nmap -sC -sV vulncicada.htb --open
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-29 09:55 EDT
Nmap scan report for vulncicada.htb (10.129.234.48)
Host is up (0.029s latency).
Not shown: 984 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: IIS Windows Server
|_http-server-header: Microsoft-IIS/10.0
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-09-29 13:55:28Z)
111/tcp  open  rpcbind       2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/tcp6  rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  2,3,4        111/udp6  rpcbind
|   100003  2,3         2049/udp   nfs
|   100003  2,3         2049/udp6  nfs
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100005  1,2,3       2049/tcp   mountd
|   100005  1,2,3       2049/tcp6  mountd
|   100005  1,2,3       2049/udp   mountd
|   100005  1,2,3       2049/udp6  mountd
|   100021  1,2,3,4     2049/tcp   nlockmgr
|   100021  1,2,3,4     2049/tcp6  nlockmgr
|   100021  1,2,3,4     2049/udp   nlockmgr
|   100021  1,2,3,4     2049/udp6  nlockmgr
|   100024  1           2049/tcp   status
|   100024  1           2049/tcp6  status
|   100024  1           2049/udp   status
|_  100024  1           2049/udp6  status
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: cicada.vl0., Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC-JPQ225.cicada.vl
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC-JPQ225.cicada.vl
| Not valid before: 2026-09-29T13:35:40
|_Not valid after:  2027-09-29T13:35:40
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: cicada.vl0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC-JPQ225.cicada.vl
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC-JPQ225.cicada.vl
| Not valid before: 2026-09-29T13:35:40
|_Not valid after:  2027-09-29T13:35:40
|_ssl-date: TLS randomness does not represent time
2049/tcp open  nlockmgr      1-4 (RPC #100021)
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: cicada.vl0., Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC-JPQ225.cicada.vl
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC-JPQ225.cicada.vl
| Not valid before: 2026-09-29T13:35:40
|_Not valid after:  2027-09-29T13:35:40
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: cicada.vl0., Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=DC-JPQ225.cicada.vl
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC-JPQ225.cicada.vl
| Not valid before: 2026-09-29T13:35:40
|_Not valid after:  2027-09-29T13:35:40
3389/tcp open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2026-09-29T13:56:50+00:00; +1s from scanner time.
| ssl-cert: Subject: commonName=DC-JPQ225.cicada.vl
| Not valid before: 2026-09-28T13:43:25
|_Not valid after:  2027-03-30T13:43:25
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
Service Info: Host: DC-JPQ225; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2026-09-29T13:56:12
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 113.12 seconds

```

```
sudo nmap --script nfs* -sV -p111,2049 vulncicada.htb
[sudo] password for kali: 
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-29 10:31 EDT
Nmap scan report for vulncicada.htb (10.129.234.48)
Host is up (0.027s latency).

PORT     STATE SERVICE  VERSION
111/tcp  open  rpcbind?
| nfs-statfs: 
|   Filesystem  1K-blocks   Used        Available  Use%  Maxfilesize  Maxlink
|_  /profiles   16105468.0  12721588.0  3383880.0  79%   16.0T        1023
|_rpcinfo: ERROR: Script execution failed (use -d to debug)
| nfs-ls: Volume /profiles
|   access: Read Lookup Modify Extend Delete NoExecute
| PERMISSION  UID         GID         SIZE  TIME                 FILENAME
| rwxrwxrwx   4294967294  4294967294  4096  2025-06-03T10:21:17  .
| ??????????  ?           ?           ?     ?                    ..
| rwxrwxrwx   4294967294  4294967294  64    2024-09-15T13:25:16  Administrator
| rwxrwxrwx   4294967294  4294967294  64    2024-09-13T15:29:28  Daniel.Marshall
| rwxrwxrwx   4294967294  4294967294  64    2024-09-13T15:29:28  Debra.Wright
| rwxrwxrwx   4294967294  4294967294  64    2024-09-13T15:30:51  Jane.Carter
| rwxrwxrwx   4294967294  4294967294  64    2024-09-13T15:29:28  Jordan.Francis
| rwxrwxrwx   4294967294  4294967294  64    2024-09-13T15:29:28  Joyce.Andrews
| rwxrwxrwx   4294967294  4294967294  64    2024-09-13T15:29:28  Katie.Ward
| rwxrwxrwx   4294967294  4294967294  64    2024-09-13T15:29:28  Megan.Simpson
|_
| nfs-showmount: 
|_  /profiles 
2049/tcp open  mountd   1-3 (RPC #100005)
| nfs-showmount: 
|_  /profiles 

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 66.18 seconds
                                                                   
```

### NFS Enumeration

```
 sudo mount -t nfs vulncicada.htb:/ ./NFS/ -o nolock
```

```
┌──(kali㉿kali)-[~/machines/vulncicada/NFS/profiles]
└─$ ls * 
Administrator:
Documents  vacation.png

Daniel.Marshall:

Debra.Wright:

Jane.Carter:

Jordan.Francis:

Joyce.Andrews:

Katie.Ward:

Megan.Simpson:

Richard.Gibbons:

Rosie.Powell:
Documents  marketing.png

Shirley.West:
                                 
```

```
┌──(kali㉿kali)-[~/machines/vulncicada/NFS]
└─$ cd Rosie.Powell
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/machines/vulncicada/NFS/Rosie.Powell]
└─$ ls
Documents  marketing.png
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/machines/vulncicada/NFS/Rosie.Powell]
└─$ sudo chmod 777 marketing.png

```

Rosie.Powell:Cicada123

```
sudo bloodhound-python -u 'Rosie.Powell' -p 'Cicada123' -ns 10.129.150.101 -d cicada.vl -c all --zip
[sudo] password for kali: 
INFO: BloodHound.py for BloodHound LEGACY (BloodHound 4.2 and 4.3)
INFO: Found AD domain: cicada.vl
INFO: Getting TGT for user
INFO: Connecting to LDAP server: dc-jpq225.cicada.vl
INFO: Testing resolved hostname connectivity dead:beef::b7f1:83f4:e73d:b6f3
INFO: Trying LDAP connection to dead:beef::b7f1:83f4:e73d:b6f3
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 1 computers
INFO: Connecting to LDAP server: dc-jpq225.cicada.vl
INFO: Testing resolved hostname connectivity dead:beef::b7f1:83f4:e73d:b6f3
INFO: Trying LDAP connection to dead:beef::b7f1:83f4:e73d:b6f3
INFO: Found 14 users
INFO: Found 54 groups
INFO: Found 2 gpos
INFO: Found 2 ous
INFO: Found 19 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: DC-JPQ225.cicada.vl
INFO: Done in 00M 08S
INFO: Compressing output into 20260929120757_bloodhound.zip

```

```
nxc smb DC-JPQ225.cicada.vl -u Rosie.Powell -p Cicada123 -k --shares
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        [*]  x64 (name:DC-JPQ225) (domain:cicada.vl) (signing:True) (SMBv1:False) (NTLM:False)
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        [+] cicada.vl\Rosie.Powell:Cicada123 
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        [*] Enumerated shares
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        Share           Permissions     Remark
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        -----           -----------     ------
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        ADMIN$                          Remote Admin
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        C$                              Default share
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        CertEnroll      READ            Active Directory Certificate Services share
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        IPC$            READ            Remote IPC
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        NETLOGON        READ            Logon server share 
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        profiles$       READ,WRITE      
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        SYSVOL          READ            Logon server share 
                                                                                                              
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