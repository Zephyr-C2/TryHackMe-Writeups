10.10.181.226

```
Starting Nmap 7.97 ( https://nmap.org ) at 2025-06-27 15:16 -0500  
Nmap scan report for 10.10.181.226  
Host is up (0.18s latency).  
Not shown: 996 closed tcp ports (conn-refused)  
PORT     STATE SERVICE  
22/tcp   open  ssh  
80/tcp   open  http  
5901/tcp open  vnc-1  
8080/tcp open  http-proxy
```

![[Pasted image 20250627151903.png]]

Default credentials worked
username: admin
password: admin

SSH pubkey only.

Did research on ScadaBR, found [this](https://github.com/hev0x/CVE-2021-26828_ScadaBR_RCE) exploit by hev0x. Exploit worked flawlessly, gained RCE and found the flag.