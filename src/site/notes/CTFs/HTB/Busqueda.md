---
{"dg-publish":true,"permalink":"/ct-fs/htb/busqueda/","dg-note-properties":{}}
---

## by: 0xCapra_Daemon aka William Keller

### Contents:
- [[CTFs/HTB/Busqueda#Phase 1 recon\|#Phase 1 recon]]
- [[CTFs/HTB/Busqueda#Phase 2 Initial Foothold\|#Phase 2 Initial Foothold]]
- [[CTFs/HTB/Busqueda#Phase 3 Privilege Escalation\|#Phase 3 Privilege Escalation]]
- [[CTFs/HTB/Busqueda#Takeaways\|#Takeaways]]



## Phase 1: recon
Just like with any good ctf/penetration test we start with gathering as much info as we can
![Pasted image 20260711213857.png](/img/user/CTFs/HTB/Images/Busqueda%20Images/Pasted%20image%2020260711213857.png)
![Pasted image 20260711213947.png](/img/user/CTFs/HTB/Images/Busqueda%20Images/Pasted%20image%2020260711213947.png)
Scanned target IP through Threader3000 and subsequent Nmap suggested scan.


![Pasted image 20260711214011.png](/img/user/CTFs/HTB/Images/Busqueda%20Images/Pasted%20image%2020260711214011.png)
Preliminary scans show ports 22 and 80 open. 


![Pasted image 20260711214152.png](/img/user/CTFs/HTB/Images/Busqueda%20Images/Pasted%20image%2020260711214152.png)
I immediately add "searcher.htb to my /etc/hosts file"


![Pasted image 20260711214404.png](/img/user/CTFs/HTB/Images/Busqueda%20Images/Pasted%20image%2020260711214404.png)
I always just curl the base domain with the verbose flag to see if anything immediately jumps out in the source code. Nothing crazy this time. It appears to be a "searcher" app, whatever that means.


![Pasted image 20260711214639.png](/img/user/CTFs/HTB/Images/Busqueda%20Images/Pasted%20image%2020260711214639.png)
First looks at it in the browser. Wappalyzer shows that it's running flask in the background. Might be susceptible to Python or SQLi related attacks within this search function. This means we need to monitor those POST requests we send to the server. For this we use BurpSuite.


![Pasted image 20260711215457.png](/img/user/CTFs/HTB/Images/Busqueda%20Images/Pasted%20image%2020260711215457.png)
I also Flask app confirmed and using something called "Searchor 2.4.0". Another possible expliot vector to search in Exploit-DB or online for a relevant CVE. For now we will continue profiling this machine.

![Pasted image 20260711215927.png](/img/user/CTFs/HTB/Images/Busqueda%20Images/Pasted%20image%2020260711215927.png)

I also went to the SearchOr Github repo and confirmed this is a PyPi server running this application.


![Pasted image 20260711215325.png](/img/user/CTFs/HTB/Images/Busqueda%20Images/Pasted%20image%2020260711215325.png)

Also kicked off a subdirectory brute force search against Raft's small words in Seclists --> Discovery --> Web-Content. It reveals nothing so far.

![Pasted image 20260711220101.png](/img/user/CTFs/HTB/Images/Busqueda%20Images/Pasted%20image%2020260711220101.png)

Making sure my proxy is on for Burp I send an innocuous search to Google for the search term "dog". 

![Pasted image 20260711220229.png](/img/user/CTFs/HTB/Images/Busqueda%20Images/Pasted%20image%2020260711220229.png)

Burp intercepts the request and we note that it's sending to the "/search" subdir as a POST request with body parameters for "engine" and "query". These two body parameters might be injectable or otherwise vulnerable.

![Pasted image 20260711220456.png](/img/user/CTFs/HTB/Images/Busqueda%20Images/Pasted%20image%2020260711220456.png)

Forwarding the request from burp we see the response from the server is a prebuilt Google query URL. Source code confirmed this is the only thing loaded in this result.

![Pasted image 20260711220644.png](/img/user/CTFs/HTB/Images/Busqueda%20Images/Pasted%20image%2020260711220644.png)
This time we send the same request through Burp but we have the "Auto redirect" box checked.

![Pasted image 20260711220743.png](/img/user/CTFs/HTB/Images/Busqueda%20Images/Pasted%20image%2020260711220743.png)
This time we note that a new body parameter "auto_redirect" is added to the POST request and it's also interesting that the value for that parameter is blank. This immediately makes me think it might be injectable.

![Pasted image 20260711221003.png](/img/user/CTFs/HTB/Images/Busqueda%20Images/Pasted%20image%2020260711221003.png)
I send the request to Repeater to monitor changes in response from the server as we play with different aspects of this request. I notice the server is running Werkzeug/2.1.2 and Python/3.10.6. More items to search for possible exploits of.

## Phase 2: Initial Foothold
![Pasted image 20260711221804.png](/img/user/CTFs/HTB/Images/Busqueda%20Images/Pasted%20image%2020260711221804.png)
Found arbitrary execution vuln for SearchOr <=2.4.2 (2.4.0) just by googling the version number we noted previously. Essentially there is an unsafe eval() function call in the main.py part of this search script. If we pass valid python script via the "query" parameter to the server, it will execute it server-side. This is looks like our best option for an initial foothold.

![Pasted image 20260711222025.png](/img/user/CTFs/HTB/Images/Busqueda%20Images/Pasted%20image%2020260711222025.png)
In nikn0laty's PoC script we can see that the "query" parameter is vulnerable to a python script injection (seen as "${evil_cmd} in the PoC)

![Pasted image 20260711222834.png](/img/user/CTFs/HTB/Images/Busqueda%20Images/Pasted%20image%2020260711222834.png)
I copy the script over locally and then make it executable with chmod. I give it a dry run and it returns with expected input params.

![Pasted image 20260711225927.png](/img/user/CTFs/HTB/Images/Busqueda%20Images/Pasted%20image%2020260711225927.png)
![Pasted image 20260711230008.png](/img/user/CTFs/HTB/Images/Busqueda%20Images/Pasted%20image%2020260711230008.png)
Reverse shell successfully caught on target machine under 'svc' user.

![Pasted image 20260711230341.png](/img/user/CTFs/HTB/Images/Busqueda%20Images/Pasted%20image%2020260711230341.png)
fully stable shell with proof of compromised user.

## Phase 3: Privilege Escalation

![Pasted image 20260711230451.png](/img/user/CTFs/HTB/Images/Busqueda%20Images/Pasted%20image%2020260711230451.png)
initial investigation into .git directory from landing directory reveals what could be hardcoded creds for a user named "cody". on a new vhost under 'gitea.searcher.htb'

![Pasted image 20260711230850.png](/img/user/CTFs/HTB/Images/Busqueda%20Images/Pasted%20image%2020260711230850.png)
Adding Gitea vhost to our hosts for target IP revealed a gitea server for the app running on target machine. attempting to login under user 'cody' with possible password and/or hash hardcoded into the .git/config file.

![Pasted image 20260711230953.png](/img/user/CTFs/HTB/Images/Busqueda%20Images/Pasted%20image%2020260711230953.png)
Authentication successful to Gitea app. Nothing really of note that we haven't already discovered inside the server itself. Seemingly dead end, but we will keep this in our back pocket in case we need it later.

![Pasted image 20260711231412.png](/img/user/CTFs/HTB/Images/Busqueda%20Images/Pasted%20image%2020260711231412.png)
moving into user's home folder we find our first flag.

![Pasted image 20260711231810.png](/img/user/CTFs/HTB/Images/Busqueda%20Images/Pasted%20image%2020260711231810.png)
initial enumeration shows we don't have sudo permissions as this user. there  are no significantly exploitable SUID binaries. Our next option is to check Cron jobs or hidden processes. Checking /etc/crontab reveals nothing as well as checking crontab for our user. I copy pspy64 into my exploit folder and host that folder with a simple python http server to send that tool over to our victim machine.

![Pasted image 20260711232033.png](/img/user/CTFs/HTB/Images/Busqueda%20Images/Pasted%20image%2020260711232033.png)
Successfully pull pspy64 over to target machine with wget.

![Pasted image 20260711232141.png](/img/user/CTFs/HTB/Images/Busqueda%20Images/Pasted%20image%2020260711232141.png)
Initial pspy results aren't showing any active processes that are run by any other user besides our current one.


![Pasted image 20260711232404.png](/img/user/CTFs/HTB/Images/Busqueda%20Images/Pasted%20image%2020260711232404.png)
Also successfully moved linpeas over to target machine to begin enumerating LPE vectors as current user "svc".

![Pasted image 20260711235325.png](/img/user/CTFs/HTB/Images/Busqueda%20Images/Pasted%20image%2020260711235325.png)
Checked Writeup for hint and it reminded me to try password reuse with the hardcoded "cody" creds we found earlier. Checked sudo privs again for svc user and found this entry.

![Pasted image 20260712000446.png](/img/user/CTFs/HTB/Images/Busqueda%20Images/Pasted%20image%2020260712000446.png)
Output from our allowed sudo commands reveals we are able to list and inspect two different docker containers, One for the Gitea instance we saw and one for a mysql database, and run an option known as "full-checkup" which gives an error when run from user's home directory. 

![Pasted image 20260712000333.png](/img/user/CTFs/HTB/Images/Busqueda%20Images/Pasted%20image%2020260712000333.png)
Inspecting the mysql container revealed hardcoded sql creds. However, mysql is running in a docker container and our current user doesn't have permissions to access these containers directly. We are going to need a different approach.

![Pasted image 20260712002432.png](/img/user/CTFs/HTB/Images/Busqueda%20Images/Pasted%20image%2020260712002432.png)
This also turned out to be the creds for the administrator user on the Gitea instance. once authenticated I found a repo for the source code of the script our current user has sudo perms to. 

![Pasted image 20260712002745.png](/img/user/CTFs/HTB/Images/Busqueda%20Images/Pasted%20image%2020260712002745.png)
Source for said script reveals that it uses a relative path for it's "full-checkup" option. So we will create a malicious "full-checkup.sh" script and abuse this for LPE.

![Pasted image 20260712003049.png](/img/user/CTFs/HTB/Images/Busqueda%20Images/Pasted%20image%2020260712003049.png)
created malicious "full-checkup.sh" in user home folder with bash to sticky bit bash privesc technique.

![Pasted image 20260712003231.png](/img/user/CTFs/HTB/Images/Busqueda%20Images/Pasted%20image%2020260712003231.png)
Script successfully run from svc user's home folder, causing our malicious script to execute due to the relative pathing in "system-checkup.py", and bash was copied into a SUID binary in /tmp directory.

![Pasted image 20260712003612.png](/img/user/CTFs/HTB/Images/Busqueda%20Images/Pasted%20image%2020260712003612.png)
Successfully executed /tmp/bash -p for suid binary root session. Pwned.

## Takeaways
- Always check for password reuse when you find any creds even if the usernames don't always match up to the server/webapp users.
-  Don't be afraid to go back to stuff you've found earlier in the hunt (back to Gitea after finding gitea administrator user creds.)
- enumerate enumerate enumerate services fully. find all users, files, possibilities.
- Read code carefully. Sometimes it's minor details that lead to big Ws (relative path vuln in system-checkup.py source code led to LPE)
[[CTFs/HTB/Busqueda#Contents\|Back to Top]]
