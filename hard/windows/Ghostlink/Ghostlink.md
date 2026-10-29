
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

### Hosts discovered

| Hostname                          | IP            | Source topic                           |
| --------------------------------- | ------------- | -------------------------------------- |
| `dc01.ghostlink.htb`              | 10.129.189.74 | `systems/node/domain/healthcheck`      |
| `gpz-op26-toolkits.ghostlink.htb` | 172.16.20.20  | `systems/node/repository/healthcheck`  |
| `gpz-op26-secure.ghostlink.htb`   | 172.16.20.10  | `systems/node/secureshare/healthcheck` |
| `transport.ghostlink.htb`         | not disclosed | `network/node/healthcheck` (node-3)    |
| `core-telecom.ghostlink.htb`      | not disclosed | `network/node/keepalive` (node-1)      |

## nslookup

```
 nslookup ghostlink.htb dc01.ghostlink.htb
Server:         dc01.ghostlink.htb
Address:        10.129.188.129#53

Name:   ghostlink.htb
Address: 172.16.20.1
Name:   ghostlink.htb
Address: 10.129.188.129

                                
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