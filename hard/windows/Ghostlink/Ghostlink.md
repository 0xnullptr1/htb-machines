
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
$ mosquitto_sub -h ghostlink.htb -t "#"
{"timestamp":"2026-29-10-11:12:33","node":"node-1","telemetry":{"ip":"10.1.12.34","loadPercent":44,"hz":54986186.6}}
{"timestamp":"2026-29-10-11:12:33","node":"node-2","telemetry":{"ip":"10.2.45.56","tokenValidation":"delayed","authErrors":37}}
{"timestamp":"2026-29-10-11:12:33","node":"node-2","telemetry":{"healthy":true,"url":"https://energy-grid.ghostlink.htb/canary","lastCheckSecAgo":57,"ip":"10.2.45.56"}}
{"timestamp":"2026-29-10-11:12:33","node":"node-1","telemetry":{"rttMs":187,"status":"error","url":"https://core-telecom.ghostlink.htb/keepalive","ip":"10.1.12.34"}}
{"timestamp":"2026-29-10-11:12:33","node":"node-2","telemetry":{"driftNs":273,"ip":"10.2.45.56","syncState":"locked"}}
{"timestamp":"2026-29-10-11:12:33","node":"node-1","telemetry":{"ip":"10.1.12.34","packetLoss":354823735,"routeStability":"stable"}}
{"timestamp":"2026-29-10-11:12:33","node":"node-1","telemetry":{"throughput":87,"anomaly":"inspection","ip":"10.1.12.34"}}
{"timestamp":"2026-29-10-11:12:46","node":"node-4","telemetry":{"healthy":true,"url":"dc01.ghostlink.htb/healthcheck","latencyMs":209,"ip":"10.129.188.129"}}
{"timestamp":"2026-29-10-11:12:46","node":"node-5","telemetry":{"healthy":true,"url":"gpz-op26-toolkits.ghostlink.htb/healthcheck","lastCheckSecAgo":2,"responseCode":"200","ip":"172.16.20.20"}}
{"timestamp":"2026-29-10-11:12:47","node":"node-6","telemetry":{"healthy":true,"url":"gpz-op26-secure.ghostlink.htb/healthcheck","lastCheckSecAgo":38,"responseCode":"200","ip":"172.16.20.10"}}
{"status":"online", "username":"(null)", "ts":1793309374820,"proto_name":"MQTT","keepalive":60,"return_code":"0","proto_ver":4,"client_id":"mqttui-b69c3d58","clean_start":1, "IPv4":"127.0.0.1"}
{"status":"offline", "username":"(null)","ts":1793309374820,"reason_code":"0","client_id":"mqttui-b69c3d58","IPv4":"127.0.0.1"}
{"status":"online", "username":"(null)", "ts":1793309376890,"proto_name":"MQTT","keepalive":60,"return_code":"0","proto_ver":4,"client_id":"mqttui-16c6a6f7","clean_start":1, "IPv4":"127.0.0.1"}
{"status":"offline", "username":"(null)","ts":1793309376890,"reason_code":"0","client_id":"mqttui-16c6a6f7","IPv4":"127.0.0.1"}
{"status":"online", "username":"(null)", "ts":1793309378976,"proto_name":"MQTT","keepalive":60,"return_code":"0","proto_ver":4,"client_id":"mqttui-fced5b83","clean_start":1, "IPv4":"127.0.0.1"}
{"status":"offline", "username":"(null)","ts":1793309378976,"reason_code":"0","client_id":"mqttui-fced5b83","IPv4":"127.0.0.1"}
{"status":"online", "username":"(null)", "ts":1793309381058,"proto_name":"MQTT","keepalive":60,"return_code":"0","proto_ver":4,"client_id":"mqttui-b7a09f06","clean_start":1, "IPv4":"127.0.0.1"}
{"status":"offline", "username":"(null)","ts":1793309381058,"reason_code":"0","client_id":"mqttui-b7a09f06","IPv4":"127.0.0.1"}
{"status":"online", "username":"(null)", "ts":1793309383145,"proto_name":"MQTT","keepalive":60,"return_code":"0","proto_ver":4,"client_id":"mqttui-d3b5774a","clean_start":1, "IPv4":"127.0.0.1"}
{"status":"offline", "username":"(null)","ts":1793309383147,"reason_code":"0","client_id":"mqttui-d3b5774a","IPv4":"127.0.0.1"}
{"status":"online", "username":"(null)", "ts":1793309385242,"proto_name":"MQTT","keepalive":60,"return_code":"0","proto_ver":4,"client_id":"mqttui-341a5fc9","clean_start":1, "IPv4":"127.0.0.1"}
{"status":"offline", "username":"(null)","ts":1793309385244,"reason_code":"0","client_id":"mqttui-341a5fc9","IPv4":"127.0.0.1"}
{"status":"online", "username":"(null)", "ts":1793309387318,"proto_name":"MQTT","keepalive":60,"return_code":"0","proto_ver":4,"client_id":"mqttui-ed7f324d","clean_start":1, "IPv4":"127.0.0.1"}
{"status":"offline", "username":"(null)","ts":1793309387319,"reason_code":"0","client_id":"mqttui-ed7f324d","IPv4":"127.0.0.1"}
{"status":"online", "username":"(null)", "ts":1793309389400,"proto_name":"MQTT","keepalive":60,"return_code":"0","proto_ver":4,"client_id":"mqttui-db327157","clean_start":1, "IPv4":"127.0.0.1"}
{"status":"offline", "username":"(null)","ts":1793309389400,"reason_code":"0","client_id":"mqttui-db327157","IPv4":"127.0.0.1"}
{"status":"online", "username":"(null)", "ts":1793309391465,"proto_name":"MQTT","keepalive":60,"return_code":"0","proto_ver":4,"client_id":"mqttui-315b10c3","clean_start":1, "IPv4":"127.0.0.1"}
{"status":"offline", "username":"(null)","ts":1793309391465,"reason_code":"0","client_id":"mqttui-315b10c3","IPv4":"127.0.0.1"}
{"status":"online", "username":"(null)", "ts":1793309393559,"proto_name":"MQTT","keepalive":60,"return_code":"0","proto_ver":4,"client_id":"mqttui-63089720","clean_start":1, "IPv4":"127.0.0.1"}
{"status":"offline", "username":"(null)","ts":1793309393559,"reason_code":"0","client_id":"mqttui-63089720","IPv4":"127.0.0.1"}
{"status":"online", "username":"(null)", "ts":1793309395650,"proto_name":"MQTT","keepalive":60,"return_code":"0","proto_ver":4,"client_id":"mqttui-7eb40a01","clean_start":1, "IPv4":"127.0.0.1"}
{"status":"offline", "username":"(null)","ts":1793309395651,"reason_code":"0","client_id":"mqttui-7eb40a01","IPv4":"127.0.0.1"}
{"status":"online", "username":"(null)", "ts":1793309397730,"proto_name":"MQTT","keepalive":60,"return_code":"0","proto_ver":4,"client_id":"mqttui-5fd51f8d","clean_start":1, "IPv4":"127.0.0.1"}
{"status":"offline", "username":"(null)","ts":1793309397730,"reason_code":"0","client_id":"mqttui-5fd51f8d","IPv4":"127.0.0.1"}
{"status":"online", "username":"(null)", "ts":1793309399801,"proto_name":"MQTT","keepalive":60,"return_code":"0","proto_ver":4,"client_id":"mqttui-68ae4a02","clean_start":1, "IPv4":"127.0.0.1"}
{"status":"offline", "username":"(null)","ts":1793309399803,"reason_code":"0","client_id":"mqttui-68ae4a02","IPv4":"127.0.0.1"}
{"status":"online", "username":"(null)", "ts":1793309401898,"proto_name":"MQTT","keepalive":60,"return_code":"0","proto_ver":4,"client_id":"mqttui-21d06855","clean_start":1, "IPv4":"127.0.0.1"}
{"status":"offline", "username":"(null)","ts":1793309401898,"reason_code":"0","client_id":"mqttui-21d06855","IPv4":"127.0.0.1"}
{"status":"online", "username":"(null)", "ts":1793309404007,"proto_name":"MQTT","keepalive":60,"return_code":"0","proto_ver":4,"client_id":"mqttui-28c7cf55","clean_start":1, "IPv4":"127.0.0.1"}
{"status":"offline", "username":"(null)","ts":1793309404009,"reason_code":"0","client_id":"mqttui-28c7cf55","IPv4":"127.0.0.1"}
{"status":"online", "username":"(null)", "ts":1793309406110,"proto_name":"MQTT","keepalive":60,"return_code":"0","proto_ver":4,"client_id":"mqttui-d68bc20f","clean_start":1, "IPv4":"127.0.0.1"}
{"status":"offline", "username":"(null)","ts":1793309406111,"reason_code":"0","client_id":"mqttui-d68bc20f","IPv4":"127.0.0.1"}
{"status":"online", "username":"(null)", "ts":1793309408212,"proto_name":"MQTT","keepalive":60,"return_code":"0","proto_ver":4,"client_id":"mqttui-6a13f224","clean_start":1, "IPv4":"127.0.0.1"}
{"status":"offline", "username":"(null)","ts":1793309408212,"reason_code":"0","client_id":"mqttui-6a13f224","IPv4":"127.0.0.1"}
{"status":"online", "username":"(null)", "ts":1793309410287,"proto_name":"MQTT","keepalive":60,"return_code":"0","proto_ver":4,"client_id":"mqttui-56bcd456","clean_start":1, "IPv4":"127.0.0.1"}
{"status":"offline", "username":"(null)","ts":1793309410287,"reason_code":"0","client_id":"mqttui-56bcd456","IPv4":"127.0.0.1"}
{"status":"online", "username":"(null)", "ts":1793309412374,"proto_name":"MQTT","keepalive":60,"return_code":"0","proto_ver":4,"client_id":"mqttui-5a589f86","clean_start":1, "IPv4":"127.0.0.1"}
{"status":"offline", "username":"(null)","ts":1793309412375,"reason_code":"0","client_id":"mqttui-5a589f86","IPv4":"127.0.0.1"}
{"status":"online", "username":"(null)", "ts":1793309414457,"proto_name":"MQTT","keepalive":60,"return_code":"0","proto_ver":4,"client_id":"mqttui-f537f28e","clean_start":1, "IPv4":"127.0.0.1"}
{"status":"offline", "username":"(null)","ts":1793309414458,"reason_code":"0","client_id":"mqttui-f537f28e","IPv4":"127.0.0.1"}
{"status":"online", "username":"(null)", "ts":1793309416554,"proto_name":"MQTT","keepalive":60,"return_code":"0","proto_ver":4,"client_id":"mqttui-6820a85","clean_start":1, "IPv4":"127.0.0.1"}
{"status":"offline", "username":"(null)","ts":1793309416554,"reason_code":"0","client_id":"mqttui-6820a85","IPv4":"127.0.0.1"}
{"status":"online", "username":"(null)", "ts":1793309418648,"proto_name":"MQTT","keepalive":60,"return_code":"0","proto_ver":4,"client_id":"mqttui-9ed9583a","clean_start":1, "IPv4":"127.0.0.1"}
{"status":"offline", "username":"(null)","ts":1793309418648,"reason_code":"0","client_id":"mqttui-9ed9583a","IPv4":"127.0.0.1"}
{"status":"online", "username":"(null)", "ts":1793309420719,"proto_name":"MQTT","keepalive":60,"return_code":"0","proto_ver":4,"client_id":"mqttui-105c5861","clean_start":1, "IPv4":"127.0.0.1"}
{"status":"offline", "username":"(null)","ts":1793309420719,"reason_code":"0","client_id":"mqttui-105c5861","IPv4":"127.0.0.1"}
{"status":"online", "username":"(null)", "ts":1793309422812,"proto_name":"MQTT","keepalive":60,"return_code":"0","proto_ver":4,"client_id":"mqttui-2770d167","clean_start":1, "IPv4":"127.0.0.1"}
{"status":"offline", "username":"(null)","ts":1793309422815,"reason_code":"0","client_id":"mqttui-2770d167","IPv4":"127.0.0.1"}
{"status":"online", "username":"(null)", "ts":1793309424895,"proto_name":"MQTT","keepalive":60,"return_code":"0","proto_ver":4,"client_id":"mqttui-9584084","clean_start":1, "IPv4":"127.0.0.1"}
{"status":"offline", "username":"(null)","ts":1793309424895,"reason_code":"0","client_id":"mqttui-9584084","IPv4":"127.0.0.1"}
{"status":"online", "username":"(null)", "ts":1793309426986,"proto_name":"MQTT","keepalive":60,"return_code":"0","proto_ver":4,"client_id":"mqttui-ab3d3eb6","clean_start":1, "IPv4":"127.0.0.1"}
{"status":"offline", "username":"(null)","ts":1793309426986,"reason_code":"0","client_id":"mqttui-ab3d3eb6","IPv4":"127.0.0.1"}
{"status":"online", "username":"(null)", "ts":1793309429072,"proto_name":"MQTT","keepalive":60,"return_code":"0","proto_ver":4,"client_id":"mqttui-d44a8192","clean_start":1, "IPv4":"127.0.0.1"}
{"status":"offline", "username":"(null)","ts":1793309429073,"reason_code":"0","client_id":"mqttui-d44a8192","IPv4":"127.0.0.1"}
{"status":"online", "username":"(null)", "ts":1793309431154,"proto_name":"MQTT","keepalive":60,"return_code":"0","proto_ver":4,"client_id":"mqttui-dcda8ea5","clean_start":1, "IPv4":"127.0.0.1"}
{"status":"offline", "username":"(null)","ts":1793309431154,"reason_code":"0","client_id":"mqttui-dcda8ea5","IPv4":"127.0.0.1"}
{"status":"online", "username":"(null)", "ts":1793309433267,"proto_name":"MQTT","keepalive":60,"return_code":"0","proto_ver":4,"client_id":"mqttui-d40c8195","clean_start":1, "IPv4":"127.0.0.1"}
{"status":"offline", "username":"(null)","ts":1793309433269,"reason_code":"0","client_id":"mqttui-d40c8195","IPv4":"127.0.0.1"}
{"status":"online", "username":"(null)", "ts":1793309435363,"proto_name":"MQTT","keepalive":60,"return_code":"0","proto_ver":4,"client_id":"mqttui-f2786ab5","clean_start":1, "IPv4":"127.0.0.1"}
{"status":"offline", "username":"(null)","ts":1793309435366,"reason_code":"0","client_id":"mqttui-f2786ab5","IPv4":"127.0.0.1"}
{"status":"online", "username":"(null)", "ts":1793309437451,"proto_name":"MQTT","keepalive":60,"return_code":"0","proto_ver":4,"client_id":"mqttui-633e0c0b","clean_start":1, "IPv4":"127.0.0.1"}
{"status":"offline", "username":"(null)","ts":1793309437451,"reason_code":"0","client_id":"mqttui-633e0c0b","IPv4":"127.0.0.1"}
{"status":"online", "username":"(null)", "ts":1793309439533,"proto_name":"MQTT","keepalive":60,"return_code":"0","proto_ver":4,"client_id":"mqttui-d63222b7","clean_start":1, "IPv4":"127.0.0.1"}
{"status":"offline", "username":"(null)","ts":1793309439534,"reason_code":"0","client_id":"mqttui-d63222b7","IPv4":"127.0.0.1"}
{"status":"online", "username":"(null)", "ts":1793309441620,"proto_name":"MQTT","keepalive":60,"return_code":"0","proto_ver":4,"client_id":"mqttui-254bb084","clean_start":1, "IPv4":"127.0.0.1"}
{"status":"offline", "username":"(null)","ts":1793309441620,"reason_code":"0","client_id":"mqttui-254bb084","IPv4":"127.0.0.1"}
{"status":"online", "username":"(null)", "ts":1793309443719,"proto_name":"MQTT","keepalive":60,"return_code":"0","proto_ver":4,"client_id":"mqttui-8aa79c8e","clean_start":1, "IPv4":"127.0.0.1"}
{"status":"offline", "username":"(null)","ts":1793309443720,"reason_code":"0","client_id":"mqttui-8aa79c8e","IPv4":"127.0.0.1"}
{"status":"online", "username":"(null)", "ts":1793309445824,"proto_name":"MQTT","keepalive":60,"return_code":"0","proto_ver":4,"client_id":"mqttui-4ab6eb4e","clean_start":1, "IPv4":"127.0.0.1"}
{"status":"offline", "username":"(null)","ts":1793309445826,"reason_code":"0","client_id":"mqttui-4ab6eb4e","IPv4":"127.0.0.1"}
{"status":"online", "username":"(null)", "ts":1793309447921,"proto_name":"MQTT","keepalive":60,"return_code":"0","proto_ver":4,"client_id":"mqttui-3f9a3550","clean_start":1, "IPv4":"127.0.0.1"}
{"status":"offline", "username":"(null)","ts":1793309447922,"reason_code":"0","client_id":"mqttui-3f9a3550","IPv4":"127.0.0.1"}
{"status":"online", "username":"(null)", "ts":1793309450021,"proto_name":"MQTT","keepalive":60,"return_code":"0","proto_ver":4,"client_id":"mqttui-c7b711a6","clean_start":1, "IPv4":"127.0.0.1"}
{"status":"offline", "username":"(null)","ts":1793309450021,"reason_code":"0","client_id":"mqttui-c7b711a6","IPv4":"127.0.0.1"}
{"status":"online", "username":"(null)", "ts":1793309452115,"proto_name":"MQTT","keepalive":60,"return_code":"0","proto_ver":4,"client_id":"mqttui-6b91b575","clean_start":1, "IPv4":"127.0.0.1"}
{"status":"offline", "username":"(null)","ts":1793309452117,"reason_code":"0","client_id":"mqttui-6b91b575","IPv4":"127.0.0.1"}
{"status":"online", "username":"(null)", "ts":1793309454208,"proto_name":"MQTT","keepalive":60,"return_code":"0","proto_ver":4,"client_id":"mqttui-e53e7710","clean_start":1, "IPv4":"127.0.0.1"}
{"status":"offline", "username":"(null)","ts":1793309454209,"reason_code":"0","client_id":"mqttui-e53e7710","IPv4":"127.0.0.1"}
{"status":"online", "username":"(null)", "ts":1793309456305,"proto_name":"MQTT","keepalive":60,"return_code":"0","proto_ver":4,"client_id":"mqttui-6c7e0e6e","clean_start":1, "IPv4":"127.0.0.1"}
{"status":"offline", "username":"(null)","ts":1793309456306,"reason_code":"0","client_id":"mqttui-6c7e0e6e","IPv4":"127.0.0.1"}
{"status":"online", "username":"(null)", "ts":1793309458406,"proto_name":"MQTT","keepalive":60,"return_code":"0","proto_ver":4,"client_id":"mqttui-f278f0a","clean_start":1, "IPv4":"127.0.0.1"}
{"status":"offline", "username":"(null)","ts":1793309458408,"reason_code":"0","client_id":"mqttui-f278f0a","IPv4":"127.0.0.1"}
{"status":"online", "username":"(null)", "ts":1793309460488,"proto_name":"MQTT","keepalive":60,"return_code":"0","proto_ver":4,"client_id":"mqttui-b88105ac","clean_start":1, "IPv4":"127.0.0.1"}
{"status":"offline", "username":"(null)","ts":1793309460488,"reason_code":"0","client_id":"mqttui-b88105ac","IPv4":"127.0.0.1"}
{"status":"online", "username":"(null)", "ts":1793309462578,"proto_name":"MQTT","keepalive":60,"return_code":"0","proto_ver":4,"client_id":"mqttui-1eedf3fc","clean_start":1, "IPv4":"127.0.0.1"}
{"status":"offline", "username":"(null)","ts":1793309462580,"reason_code":"0","client_id":"mqttui-1eedf3fc","IPv4":"127.0.0.1"}
{"status":"online", "username":"(null)", "ts":1793309464672,"proto_name":"MQTT","keepalive":60,"return_code":"0","proto_ver":4,"client_id":"mqttui-42773083","clean_start":1, "IPv4":"127.0.0.1"}
{"status":"offline", "username":"(null)","ts":1793309464674,"reason_code":"0","client_id":"mqttui-42773083","IPv4":"127.0.0.1
```

|Topic|Hostname|IP|
|---|---|---|
|`network/node/healthcheck`|`energy-grid.ghostlink.htb`|10.2.45.56|
|`network/node/keepalive`|`core-telecom.ghostlink.htb`|10.1.12.34|
|`systems/node/domain/healthcheck`|`dc01.ghostlink.htb`|10.129.188.129 (la tua box)|
|`systems/node/repository/healthcheck`|`gpz-op26-toolkits.ghostlink.htb`|172.16.20.20|
|`systems/node/secureshare/healthcheck`|`gpz-op26-secure.ghostlink.htb`|172.16.20.10|

```
mosquitto_sub -h ghostlink.htb -p 1883 -v -t 'GhostProtocolZero/network/node/healthcheck' -W 2
GhostProtocolZero/network/node/healthcheck {"timestamp":"2026-29-10-11:12:33","node":"node-2","telemetry":{"healthy":true,"url":"https://energy-grid.ghostlink.htb/canary","lastCheckSecAgo":57,"ip":"10.2.45.56"}}
Timed out

```

```
mosquitto_pub -h ghostlink.htb -p 1883 -t 'test/check' -m 'ping' -d 
Client null sending CONNECT
Client (null) received CONNACK (0)
Client null sending PUBLISH (d0, q0, r0, m1, 'test/check', ... (4 bytes))
Client null sending DISCONNECT
                                         
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