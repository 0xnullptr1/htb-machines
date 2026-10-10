
| Property         | Value                    |
| ---------------- | ------------------------ |
| **OS**           | Windows                  |
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
sudo nmap -sV -sC 10.129.229.56 --open
[sudo] password for kali: 
Starting Nmap 7.95 ( https://nmap.org ) at 2026-10-10 09:01 EDT
Nmap scan report for 10.129.229.56
Host is up (0.028s latency).
Not shown: 946 closed tcp ports (reset), 40 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
| http-methods: 
|_  Potentially risky methods: TRACE
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-10-10 17:01:48Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: authority.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-10-10T17:02:37+00:00; +4h00m02s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: othername: UPN:AUTHORITY$@htb.corp, DNS:authority.htb.corp, DNS:htb.corp, DNS:HTB
| Not valid before: 2022-08-09T23:03:21
|_Not valid after:  2024-08-09T23:13:21
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: authority.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-10-10T17:02:38+00:00; +4h00m02s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: othername: UPN:AUTHORITY$@htb.corp, DNS:authority.htb.corp, DNS:htb.corp, DNS:HTB
| Not valid before: 2022-08-09T23:03:21
|_Not valid after:  2024-08-09T23:13:21
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: authority.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-10-10T17:02:37+00:00; +4h00m02s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: othername: UPN:AUTHORITY$@htb.corp, DNS:authority.htb.corp, DNS:htb.corp, DNS:HTB
| Not valid before: 2022-08-09T23:03:21
|_Not valid after:  2024-08-09T23:13:21
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: authority.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: othername: UPN:AUTHORITY$@htb.corp, DNS:authority.htb.corp, DNS:htb.corp, DNS:HTB
| Not valid before: 2022-08-09T23:03:21
|_Not valid after:  2024-08-09T23:13:21
|_ssl-date: 2026-10-10T17:02:38+00:00; +4h00m02s from scanner time.
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
8443/tcp open  ssl/http      Apache Tomcat (language: en)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=172.16.2.118
| Not valid before: 2026-10-08T14:58:38
|_Not valid after:  2028-10-10T02:37:02
|_http-title: Site doesn't have a title (text/html;charset=ISO-8859-1).
Service Info: Host: AUTHORITY; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2026-10-10T17:02:32
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_clock-skew: mean: 4h00m01s, deviation: 0s, median: 4h00m01s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 57.50 seconds
                                                                  
```

port 8443 stands out as it is unusal compared to the standard AD set

```
┌──(kali㉿kali)-[~/machines/authority]
└─$ echo '10.129.229.56 authority.htb' | sudo tee -a /etc/hosts
10.129.229.56 authority.htb
                                                                                                                    
┌──(kali㉿kali)-[~/machines/authority]
└─$ echo '10.129.229.56 authority.htb.corp' | sudo tee -a /etc/hosts
10.129.229.56 authority.htb.corp
                                                        
```

img port 8443
### Service Enumeration

```
 smbclient -N -L \\authority.htb       

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        Department Shares Disk      
        Development     Disk      
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        SYSVOL          Disk      Logon server share 
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to authority.htb failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available

```

```
smbclient -N '//authority.htb/Department Shares'

Try "help" to get a list of possible commands.
smb: \> ls
NT_STATUS_ACCESS_DENIED listing \*
smb: \> 

```

```
 smbclient -N \\\\authority.htb\\Development             
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Fri Mar 17 09:20:38 2023
  ..                                  D        0  Fri Mar 17 09:20:38 2023
  Automation                          D        0  Fri Mar 17 09:20:40 2023

                5888511 blocks of size 4096. 1496891 blocks available
smb: \> cd Automation
smb: \Automation\> ls
  .                                   D        0  Fri Mar 17 09:20:40 2023
  ..                                  D        0  Fri Mar 17 09:20:40 2023
  Ansible                             D        0  Fri Mar 17 09:20:50 2023

                5888511 blocks of size 4096. 1496891 blocks available
smb: \Automation\> cd Ansible
smb: \Automation\Ansible\> ls
  .                                   D        0  Fri Mar 17 09:20:50 2023
  ..                                  D        0  Fri Mar 17 09:20:50 2023
  ADCS                                D        0  Fri Mar 17 09:20:48 2023
  LDAP                                D        0  Fri Mar 17 09:20:48 2023
  PWM                                 D        0  Fri Mar 17 09:20:48 2023
  SHARE                               D        0  Fri Mar 17 09:20:48 2023

                5888511 blocks of size 4096. 1496891 blocks available
smb: \Automation\Ansible\> 

```

```
smb: \Automation\Ansible\> cd PWM
smb: \Automation\Ansible\PWM\> ls
  .                                   D        0  Fri Mar 17 09:20:48 2023
  ..                                  D        0  Fri Mar 17 09:20:48 2023
  ansible.cfg                         A      491  Thu Sep 22 01:36:58 2022
  ansible_inventory                   A      174  Wed Sep 21 18:19:32 2022
  defaults                            D        0  Fri Mar 17 09:20:48 2023
  handlers                            D        0  Fri Mar 17 09:20:48 2023
  meta                                D        0  Fri Mar 17 09:20:48 2023
  README.md                           A     1290  Thu Sep 22 01:35:58 2022
  tasks                               D        0  Fri Mar 17 09:20:48 2023
  templates                           D        0  Fri Mar 17 09:20:48 2023

                5888511 blocks of size 4096. 1496891 blocks available
smb: \Automation\Ansible\PWM\> cd ansible_inventory
cd \Automation\Ansible\PWM\ansible_inventory\: NT_STATUS_NOT_A_DIRECTORY
smb: \Automation\Ansible\PWM\> more ansible_inventory
getting file \Automation\Ansible\PWM\ansible_inventory of size 174 as /tmp/smbmore.yza2On (1.5 KiloBytes/sec) (average 7.1 KiloBytes/sec)
smb: \Automation\Ansible\PWM\> 

```

```
ansible_user: administrator
ansible_password: Welcome1
ansible_port: 5985
ansible_connection: winrm
ansible_winrm_transport: ntlm
ansible_winrm_server_cert_validation: ignore
/tmp/smbmore.yza2On (END)

```

```
smb: \Automation\Ansible\PWM\defaults\> ls
  .                                   D        0  Fri Mar 17 09:20:48 2023
  ..                                  D        0  Fri Mar 17 09:20:48 2023
  main.yml                            A     1591  Sun Apr 23 18:51:38 2023

                5888511 blocks of size 4096. 1496875 blocks available
smb: \Automation\Ansible\PWM\defaults\> more main.yml
getting file \Automation\Ansible\PWM\defaults\main.yml of size 1591 as /tmp/smbmore.ypHcYA (6.6 KiloBytes/sec) (average 7.0 KiloBytes/sec)
smb: \Automation\Ansible\PWM\defaults\> 

```

```
---
pwm_run_dir: "{{ lookup('env', 'PWD') }}"

pwm_hostname: authority.htb.corp
pwm_http_port: "{{ http_port }}"
pwm_https_port: "{{ https_port }}"
pwm_https_enable: true

pwm_require_ssl: false

pwm_admin_login: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          32666534386435366537653136663731633138616264323230383566333966346662313161326239
          6134353663663462373265633832356663356239383039640a346431373431666433343434366139
          35653634376333666234613466396534343030656165396464323564373334616262613439343033
          6334326263326364380a653034313733326639323433626130343834663538326439636232306531
          3438

pwm_admin_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          31356338343963323063373435363261323563393235633365356134616261666433393263373736
          3335616263326464633832376261306131303337653964350a363663623132353136346631396662
          38656432323830393339336231373637303535613636646561653637386634613862316638353530
          3930356637306461350a316466663037303037653761323565343338653934646533663365363035
          6531

ldap_uri: ldap://127.0.0.1/
ldap_base_dn: "DC=authority,DC=htb"
ldap_admin_password: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          63303831303534303266356462373731393561313363313038376166336536666232626461653630
          3437333035366235613437373733316635313530326639330a643034623530623439616136363563
          34646237336164356438383034623462323531316333623135383134656263663266653938333334
          3238343230333633350a646664396565633037333431626163306531336336326665316430613566
          3764
/tmp/smbmore.ypHcYA (END)

```

```
smb: \Automation\Ansible\PWM\meta\> ls
  .                                   D        0  Fri Mar 17 09:20:48 2023
  ..                                  D        0  Fri Mar 17 09:20:48 2023
  main.yml                            A      199  Thu Sep 22 01:31:36 2022

                5888511 blocks of size 4096. 1496874 blocks available
smb: \Automation\Ansible\PWM\meta\> more main.yml
getting file \Automation\Ansible\PWM\meta\main.yml of size 199 as /tmp/smbmore.zUGEb0 (1.7 KiloBytes/sec) (average 5.7 KiloBytes/sec)
smb: \Automation\Ansible\PWM\meta\> 

```

```
galaxy_info:
  author: Authority
  description: PWM web service
  license: GPLv2
  min_ansible_version: 1.5
  platforms:
  - name: Windows
    versions:
    - v1
  categories:
  - web
  - system
```

```
smb: \Automation\Ansible\PWM\templates\> ls
  .                                   D        0  Fri Mar 17 09:20:48 2023
  ..                                  D        0  Fri Mar 17 09:20:48 2023
  context.xml.j2                      A      422  Wed May 18 15:57:54 2022
  tomcat-users.xml.j2                 A      388  Wed Sep 21 18:08:08 2022

                5888511 blocks of size 4096. 1496827 blocks available
smb: \Automation\Ansible\PWM\templates\> more context.xml.j2 
getting file \Automation\Ansible\PWM\templates\context.xml.j2 of size 422 as /tmp/smbmore.9zDe9D (3.6 KiloBytes/sec) (average 6.6 KiloBytes/sec)
smb: \Automation\Ansible\PWM\templates\> more tomcat-users.xml.j2
getting file \Automation\Ansible\PWM\templates\tomcat-users.xml.j2 of size 388 as /tmp/smbmore.QkYz7J (3.3 KiloBytes/sec) (average 6.3 KiloBytes/sec)
smb: \Automation\Ansible\PWM\templates\> 

```

```

<?xml version='1.0' encoding='cp1252'?>

<tomcat-users xmlns="http://tomcat.apache.org/xml" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 xsi:schemaLocation="http://tomcat.apache.org/xml tomcat-users.xsd"
 version="1.0">

<user username="admin" password="T0mc@tAdm1n" roles="manager-gui"/>  
<user username="robot" password="T0mc@tR00t" roles="manager-script"/>

</tomcat-users>
/tmp/smbmore.QkYz7J (END)

```

Cracking hashes:

```
cat > admin_pass.txt << 'EOF'
$ANSIBLE_VAULT;1.1;AES256
31356338343963323063373435363261323563393235633365356134616261666433393263373736
3335616263326464633832376261306131303337653964350a363663623132353136346631396662
38656432323830393339336231373637303535613636646561653637386634613862316638353530
3930356637306461350a316466663037303037653761323565343338653934646533663365363035
6531
EOF
                                                                                                                
┌──(kali㉿kali)-[~/machines/authority]
└─$ cat > ldap.txt << 'EOF'
$ANSIBLE_VAULT;1.1;AES256
63303831303534303266356462373731393561313363313038376166336536666232626461653630
3437333035366235613437373733316635313530326639330a643034623530623439616136363563
34646237336164356438383034623462323531316333623135383134656263663266653938333334
3238343230333633350a646664396565633037333431626163306531336336326665316430613566
3764
EOF

```

```
john admin_pass.hash --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (ansible, Ansible Vault [PBKDF2-SHA256 HMAC-256 256/256 AVX2 8x])
Cost 1 (iteration count) is 10000 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
!@#$%^&*         (admin_pass.txt)     
1g 0:00:00:07 DONE (2026-10-10 11:08) 0.1265g/s 5038p/s 5038c/s 5038C/s 051790..victor2
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
                                                                                                                
┌──(kali㉿kali)-[~/machines/authority]
└─$ john ldap.hash --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (ansible, Ansible Vault [PBKDF2-SHA256 HMAC-256 256/256 AVX2 8x])
Cost 1 (iteration count) is 10000 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
!@#$%^&*         (ldap.txt)     
1g 0:00:00:08 DONE (2026-10-10 11:08) 0.1242g/s 4945p/s 4945c/s 4945C/s 051790..victor2
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
                         
```

```
 john admin_login.hash --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (ansible, Ansible Vault [PBKDF2-SHA256 HMAC-256 256/256 AVX2 8x])
Cost 1 (iteration count) is 10000 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
!@#$%^&*         (admin_login.txt)     
1g 0:00:00:07 DONE (2026-10-10 11:10) 0.1269g/s 5051p/s 5051c/s 5051C/s 051790..victor2
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 

```

```
┌──(kali㉿kali)-[~/machines/authority]
└─$ ansible-vault decrypt admin_login.txt --vault-password-file=<(echo '!@#$%^&*') --output=-
Decryption successful
svc_pwm                                                                                                                
┌──(kali㉿kali)-[~/machines/authority]
└─$ ansible-vault decrypt admin_pass.txt --vault-password-file=<(echo '!@#$%^&*') --output=-
Decryption successful
pWm_@dm!N_!23                                                                                                                
┌──(kali㉿kali)-[~/machines/authority]
└─$ ansible-vault decrypt ldap.txt --vault-password-file=<(echo '!@#$%^&*') --output=-
Decryption successful
DevT3st@123        
```

using the recovered pass is it possible to connect 


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