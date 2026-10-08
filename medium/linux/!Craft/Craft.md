
| Property         | Value                    |
| ---------------- | ------------------------ |
| **OS**           | Linux                    |
| **Difficulty**   | Medium                   |
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
sudo nmap -sC -sV craft.htb --open
Starting Nmap 7.95 ( https://nmap.org ) at 2026-10-08 13:33 EDT
Nmap scan report for craft.htb (10.129.162.165)
Host is up (0.037s latency).
Not shown: 998 closed tcp ports (reset)
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
| ssh-hostkey: 
|   2048 bd:e7:6c:22:81:7a:db:3e:c0:f0:73:1d:f3:af:77:65 (RSA)
|   256 82:b5:f9:d1:95:3b:6d:80:0f:35:91:86:2d:b3:d7:66 (ECDSA)
|_  256 28:3b:26:18:ec:df:b3:36:85:9c:27:54:8d:8c:e1:33 (ED25519)
443/tcp open  ssl/http nginx 1.15.8
|_http-title: About
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=craft.htb/organizationName=Craft/stateOrProvinceName=NY/countryName=US
| Not valid before: 2019-02-06T02:25:47
|_Not valid after:  2020-06-20T02:25:47
|_http-server-header: nginx/1.15.8
| tls-alpn: 
|_  http/1.1
| tls-nextprotoneg: 
|_  http/1.1
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 18.33 seconds

```

```
sudo nmap -p- craft.htb --open 
[sudo] password for kali: 
Starting Nmap 7.95 ( https://nmap.org ) at 2026-10-08 13:33 EDT
Nmap scan report for craft.htb (10.129.162.165)
Host is up (0.030s latency).
Not shown: 65414 closed tcp ports (reset), 118 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE
22/tcp   open  ssh
443/tcp  open  https
6022/tcp open  x11

Nmap done: 1 IP address (1 host up) scanned in 16.51 seconds
                                                                  
```

```
nmap -p 6022 craft.htb -sV -sC        
Starting Nmap 7.95 ( https://nmap.org ) at 2026-10-08 13:33 EDT
Nmap scan report for craft.htb (10.129.162.165)
Host is up (0.030s latency).

PORT     STATE SERVICE VERSION
6022/tcp open  ssh     Golang x/crypto/ssh server (protocol 2.0)
| ssh-hostkey: 
|_  2048 5b:cc:bf:f1:a1:8f:72:b0:c0:fb:df:a3:01:dc:a6:fb (RSA)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 39.54 seconds
                                                               
```

### web page Enumeration

clicking on api redirects to:

```
https://api.craft.htb/api/
```

adding to /etc/hosts:

```
echo '10.129.162.165 api.craft.htb' | sudo tee -a /etc/hosts
```

clicking on the icon redirects to:

```
https://gogs.craft.htb/
```

```
echo '10.129.162.165 gogs.craft.htb' | sudo tee -a /etc/hosts
```

```
https://gogs.craft.htb/Craft/craft-api/src/master/craft_api/api/brew/endpoints/brew.py
```

```

from flask import request, jsonify, make_response
from flask_restplus import Resource
from craft_api.api.restplus import api
from craft_api.api.auth.endpoints import auth
from craft_api.api.brew.operations import create_brew, update_brew, delete_brew
from craft_api.api.brew.serializers import beer_entry, page_of_beer_entries
from craft_api.api.brew.parsers import pagination_arguments
from craft_api.database.models import Brew
from functools import wraps
import datetime 

ns = api.namespace('brew/', description='Operations related to beer.')


@ns.route('/')
class BrewCollection(Resource):

    @api.expect(pagination_arguments)
    @api.marshal_with(page_of_beer_entries)
    def get(self):
        """
        Returns list of brews.
        """
        args = pagination_arguments.parse_args(request)
        page = args.get('page', 1)
        per_page = args.get('per_page', 10)

        brews_query = Brew.query
        brews_page = brews_query.paginate(page, per_page, error_out=False)

        return brews_page

    @auth.auth_required
    @api.expect(beer_entry)
    def post(self):
        """
        Creates a new brew entry.
        """

        # make sure the ABV value is sane.
        if eval('%s > 1' % request.json['abv']):
            return "ABV must be a decimal value less than 1.0", 400
        else:
            create_brew(request.json)
            return None, 201

@ns.route('/<int:id>')
@api.response(404, 'Brew not found.')
class BrewItem(Resource):

    @api.marshal_with(beer_entry)
    def get(self, id):
        """
        Returns brew data.
        """
        return Brew.query.filter(Brew.id == id).one()

    @auth.auth_required
    @api.expect(beer_entry)
    @api.response(204, 'Brew successfully updated.')
    def put(self, id):
        """
        Updates a brew.
        """
        data = request.json
        update_brew(id, data)
        return None, 204

    @auth.auth_required
    @api.response(204, 'Brew successfully deleted.')
    def delete(self, id):
        """
        Deletes a brew.
        """
        delete_brew(id)
        return None, 204
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