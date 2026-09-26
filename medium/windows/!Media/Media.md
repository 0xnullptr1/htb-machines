
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