
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
sudo nmap -sC -sV 10.129.234.50 --open                                                              
[sudo] password for kali: 
Starting Nmap 7.95 ( https://nmap.org ) at 2026-10-12 08:40 EDT
Nmap scan report for 10.129.234.50
Host is up (0.047s latency).
Not shown: 984 closed tcp ports (reset)
PORT     STATE SERVICE       VERSION
21/tcp   open  ftp           Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 10-20-24  01:11AM                  434 CyberAudit.txt
| 10-20-24  05:14AM                 2622 Shared.kdbx
|_10-20-24  01:26AM                  580 TrainingAgenda.txt
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: IIS Windows Server
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-10-12 12:40:36Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: redelegate.vl0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
1433/tcp open  ms-sql-s      Microsoft SQL Server 2019 15.00.2000.00; RTM
|_ssl-date: 2026-10-12T12:40:48+00:00; +2s from scanner time.
| ms-sql-info: 
|   10.129.234.50:1433: 
|     Version: 
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
| ms-sql-ntlm-info: 
|   10.129.234.50:1433: 
|     Target_Name: REDELEGATE
|     NetBIOS_Domain_Name: REDELEGATE
|     NetBIOS_Computer_Name: DC
|     DNS_Domain_Name: redelegate.vl
|     DNS_Computer_Name: dc.redelegate.vl
|     DNS_Tree_Name: redelegate.vl
|_    Product_Version: 10.0.20348
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2026-10-12T11:33:41
|_Not valid after:  2056-10-12T11:33:41
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: redelegate.vl0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
3389/tcp open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2026-10-12T12:40:47+00:00; +1s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: REDELEGATE
|   NetBIOS_Domain_Name: REDELEGATE
|   NetBIOS_Computer_Name: DC
|   DNS_Domain_Name: redelegate.vl
|   DNS_Computer_Name: dc.redelegate.vl
|   DNS_Tree_Name: redelegate.vl
|   Product_Version: 10.0.20348
|_  System_Time: 2026-10-12T12:40:39+00:00
| ssl-cert: Subject: commonName=dc.redelegate.vl
| Not valid before: 2026-10-11T11:30:55
|_Not valid after:  2027-04-12T11:30:55
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2026-10-12T12:40:39
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_clock-skew: mean: 1s, deviation: 0s, median: 1s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 22.86 seconds

```

```
 echo '10.129.234.50 redelegate.vl' | sudo tee -a /etc/hosts
[sudo] password for kali: 
10.129.234.50 redelegate.vl
                                                                                                                    
┌──(kali㉿kali)-[~/machines/redelegate]
└─$ echo '10.129.234.50 dc.redelegate.vl' | sudo tee -a /etc/hosts
10.129.234.50 dc.redelegate.vl

```

### FTP Enumeration

```
ftp -a redelegate.vl
```

Downloading all the files:

```
Connected to redelegate.vl.
220 Microsoft FTP Service
331 Anonymous access allowed, send identity (e-mail name) as password.
230 User logged in.
Remote system type is Windows_NT.
ftp> ls
229 Entering Extended Passive Mode (|||62220|)
125 Data connection already open; Transfer starting.
10-20-24  01:11AM                  434 CyberAudit.txt
10-20-24  05:14AM                 2622 Shared.kdbx
10-20-24  01:26AM                  580 TrainingAgenda.txt
226 Transfer complete.
ftp> get *
local: * remote: *
229 Entering Extended Passive Mode (|||62221|)
550 The filename, directory name, or volume label syntax is incorrect. 
ftp> get CyberAudit.txt
local: CyberAudit.txt remote: CyberAudit.txt
229 Entering Extended Passive Mode (|||62223|)
125 Data connection already open; Transfer starting.
100% |***********************************************************************|   434        6.10 KiB/s    00:00 ETA
226 Transfer complete.
434 bytes received in 00:00 (6.06 KiB/s)
ftp> get Shared.kdbx
local: Shared.kdbx remote: Shared.kdbx
229 Entering Extended Passive Mode (|||62224|)
125 Data connection already open; Transfer starting.
100% |***********************************************************************|  2622       52.80 KiB/s    00:00 ETA
226 Transfer complete.
WARNING! 10 bare linefeeds received in ASCII mode.
File may not have transferred correctly.
2622 bytes received in 00:00 (51.19 KiB/s)
ftp> get Shared.kdbx
local: Shared.kdbx remote: Shared.kdbx
229 Entering Extended Passive Mode (|||62225|)
125 Data connection already open; Transfer starting.
100% |***********************************************************************|  2622       62.15 KiB/s    00:00 ETA
226 Transfer complete.
WARNING! 10 bare linefeeds received in ASCII mode.
File may not have transferred correctly.
2622 bytes received in 00:00 (60.76 KiB/s)
ftp> get TrainingAgenda.txt
local: TrainingAgenda.txt remote: TrainingAgenda.txt
229 Entering Extended Passive Mode (|||62226|)
150 Opening ASCII mode data connection.
100% |***********************************************************************|   580        5.40 KiB/s    00:00 ETA
226 Transfer complete.
580 bytes received in 00:00 (5.38 KiB/s)
ftp> 

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