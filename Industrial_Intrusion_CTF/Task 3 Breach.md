Target machine IP
```
10.10.201.61
```

## Enumeration

### NMAP

Basic Scan

```bash
nmap 10.10.201.61
```

Results
```bash
root@ip-10-10-248-62:~# nmap 10.10.201.61
Starting Nmap 7.80 ( https://nmap.org ) at 2025-06-27 17:57 BST
Nmap scan report for ip-10-10-201-61.eu-west-1.compute.internal (10.10.201.61)
Host is up (0.00024s latency).
Not shown: 997 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
8080/tcp open  http-proxy
MAC Address: 02:13:D1:E6:8D:43 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 0.29 seconds

```

Depth Scan

```bash
nmap -A -p- 10.10.201.61
```

Results
```bash
root@ip-10-10-248-62:~# nmap -A -p- 10.10.201.61
Starting Nmap 7.80 ( https://nmap.org ) at 2025-06-27 17:54 BST
Nmap scan report for ip-10-10-201-61.eu-west-1.compute.internal (10.10.201.61)
Host is up (0.00024s latency).
Not shown: 65528 closed ports
PORT      STATE SERVICE       VERSION
22/tcp    open  ssh           OpenSSH 9.6p1 Ubuntu 3ubuntu13.11 (Ubuntu Linux; protocol 2.0)
80/tcp    open  http          Werkzeug/3.1.3 Python/3.12.3
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Server: Werkzeug/3.1.3 Python/3.12.3
|     Date: Fri, 27 Jun 2025 16:54:22 GMT
|     Content-Disposition: inline; filename=index.html
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 1608
|     Last-Modified: Mon, 23 Jun 2025 00:44:37 GMT
|     Cache-Control: no-cache
|     ETag: "1750639477.176184-1608-3272805344"
|     Date: Fri, 27 Jun 2025 16:54:22 GMT
|     Connection: close
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta charset="UTF-8">
|     <title>Gate Monitor</title>
|     <style>
|     body {
|     font-family: Arial, sans-serif;
|     background: #f4f4f4;
|     text-align: center;
|     padding-top: 50px;
|     color: #007acc;
|     #status {
|     font-size: 1.5em;
|     margin-top: 20px;
|     #flag {
|     margin-top: 10px;
|     font-size: 1.3em;
|     font-weight: bold;
|     color: #c62828;
|     width: 300px;
|     height: auto;
|     margin-top:
|   HTTPOptions: 
|     HTTP/1.1 200 OK
|     Server: Werkzeug/3.1.3 Python/3.12.3
|     Date: Fri, 27 Jun 2025 16:54:22 GMT
|     Content-Type: text/html; charset=utf-8
|     Allow: OPTIONS, GET, HEAD
|     Content-Length: 0
|     Connection: close
|   RTSPRequest: 
|     <!DOCTYPE HTML>
|     <html lang="en">
|     <head>
|     <meta charset="utf-8">
|     <title>Error response</title>
|     </head>
|     <body>
|     <h1>Error response</h1>
|     <p>Error code: 400</p>
|     <p>Message: Bad request version ('RTSP/1.0').</p>
|     <p>Error code explanation: 400 - Bad request syntax or unsupported method.</p>
|     </body>
|_    </html>
|_http-server-header: Werkzeug/3.1.3 Python/3.12.3
|_http-title: Gate Monitor
102/tcp   open  iso-tsap      Siemens S7 PLC
| fingerprint-strings: 
|   TerminalServerCookie: 
|_    Cookie: mstshash=nmap
| s7-info: 
|   Module: 6ES7 315-2EH14-0AB0 
|   Basic Hardware: 6ES7 315-2EH14-0AB0 
|   Version: 3.2.6
|   System Name: SNAP7-SERVER
|   Module Type: CPU 315-2 PN/DP
|   Serial Number: S C-C2UR28922012
|_  Copyright: Original Siemens Equipment
502/tcp   open  mbap?
| fingerprint-strings: 
|   NCP: 
|_    DmdT
1880/tcp  open  vsat-control?
| fingerprint-strings: 
|   DNSVersionBindReqTCP, RPCCheck: 
|     HTTP/1.1 400 Bad Request
|     Connection: close
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Access-Control-Allow-Origin: *
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 1733
|     ETag: W/"6c5-hGVEFL4qpfS9qVbAlfbm9AL7VT0"
|     Date: Fri, 27 Jun 2025 16:54:27 GMT
|     Connection: close
|     <!DOCTYPE html>
|     <html>
|     <head>
|     <meta charset="utf-8">
|     <meta http-equiv="X-UA-Compatible" content="IE=edge">
|     <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=0">
|     <meta name="apple-mobile-web-app-capable" content="yes">
|     <meta name="mobile-web-app-capable" content="yes">
|     <!--
|     Copyright OpenJS Foundation and other contributors, https://openjsf.org/
|     Licensed under the Apache License, Version 2.0 (the "License");
|     this file except in compliance with the License.
|     obtain a copy of the License at
|     http://www.apache.org/licenses/LICENSE-2.0
|     Unless required by applicable law or agreed to in writing, softwa
|   HTTPOptions, RTSPRequest: 
|     HTTP/1.1 204 No Content
|     Access-Control-Allow-Origin: *
|     Access-Control-Allow-Methods: GET,PUT,POST,DELETE
|     Vary: Access-Control-Request-Headers
|     Content-Length: 0
|     Date: Fri, 27 Jun 2025 16:54:27 GMT
|_    Connection: close
8080/tcp  open  http-proxy    Werkzeug/2.3.7 Python/3.12.3
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.1 404 NOT FOUND
|     Server: Werkzeug/2.3.7 Python/3.12.3
|     Date: Fri, 27 Jun 2025 16:54:22 GMT
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 207
|     Vary: Cookie
|     Set-Cookie: session=eyJfcGVybWFuZW50Ijp0cnVlfQ.aF7Mvg.QBiq_MeFUPGs3J2NMsIityT45Go; Expires=Fri, 27 Jun 2025 16:59:22 GMT; HttpOnly; Path=/
|     Connection: close
|     <!doctype html>
|     <html lang=en>
|     <title>404 Not Found</title>
|     <h1>Not Found</h1>
|     <p>The requested URL was not found on the server. If you entered the URL manually please check your spelling and try again.</p>
|   GetRequest: 
|     HTTP/1.1 302 FOUND
|     Server: Werkzeug/2.3.7 Python/3.12.3
|     Date: Fri, 27 Jun 2025 16:54:22 GMT
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 199
|     Location: /login
|     Vary: Cookie
|     Set-Cookie: session=eyJfZnJlc2giOmZhbHNlLCJfcGVybWFuZW50Ijp0cnVlfQ.aF7Mvg.hhANfvas49EjlhUCQ2CczFANejw; Expires=Fri, 27 Jun 2025 16:59:22 GMT; HttpOnly; Path=/
|     Connection: close
|     <!doctype html>
|     <html lang=en>
|     <title>Redirecting...</title>
|     <h1>Redirecting...</h1>
|     <p>You should be redirected automatically to the target URL: <a href="/login">/login</a>. If not, click the link.
|   HTTPOptions: 
|     HTTP/1.1 200 OK
|     Server: Werkzeug/2.3.7 Python/3.12.3
|     Date: Fri, 27 Jun 2025 16:54:22 GMT
|     Content-Type: text/html; charset=utf-8
|     Allow: OPTIONS, HEAD, GET
|     Vary: Cookie
|     Set-Cookie: session=eyJfcGVybWFuZW50Ijp0cnVlfQ.aF7Mvg.QBiq_MeFUPGs3J2NMsIityT45Go; Expires=Fri, 27 Jun 2025 16:59:22 GMT; HttpOnly; Path=/
|     Content-Length: 0
|     Connection: close
|   RTSPRequest: 
|     <!DOCTYPE HTML>
|     <html lang="en">
|     <head>
|     <meta charset="utf-8">
|     <title>Error response</title>
|     </head>
|     <body>
|     <h1>Error response</h1>
|     <p>Error code: 400</p>
|     <p>Message: Bad request version ('RTSP/1.0').</p>
|     <p>Error code explanation: 400 - Bad request syntax or unsupported method.</p>
|     </body>
|_    </html>
|_http-server-header: Werkzeug/2.3.7 Python/3.12.3
| http-title: Site doesn't have a title (text/html; charset=utf-8).
|_Requested resource was /login
44818/tcp open  EtherNetIP-2?
4 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port80-TCP:V=7.80%I=7%D=6/27%Time=685ECCC0%P=x86_64-pc-linux-gnu%r(GetR
SF:equest,7C0,"HTTP/1\.1\x20200\x20OK\r\nServer:\x20Werkzeug/3\.1\.3\x20Py
SF:thon/3\.12\.3\r\nDate:\x20Fri,\x2027\x20Jun\x202025\x2016:54:22\x20GMT\
SF:r\nContent-Disposition:\x20inline;\x20filename=index\.html\r\nContent-T
SF:ype:\x20text/html;\x20charset=utf-8\r\nContent-Length:\x201608\r\nLast-
SF:Modified:\x20Mon,\x2023\x20Jun\x202025\x2000:44:37\x20GMT\r\nCache-Cont
SF:rol:\x20no-cache\r\nETag:\x20\"1750639477\.176184-1608-3272805344\"\r\n
SF:Date:\x20Fri,\x2027\x20Jun\x202025\x2016:54:22\x20GMT\r\nConnection:\x2
SF:0close\r\n\r\n<!DOCTYPE\x20html>\n<html\x20lang=\"en\">\n<head>\n\x20\x
SF:20<meta\x20charset=\"UTF-8\">\n\x20\x20<title>Gate\x20Monitor</title>\n
SF:\x20\x20<style>\n\x20\x20\x20\x20body\x20{\n\x20\x20\x20\x20\x20\x20fon
SF:t-family:\x20Arial,\x20sans-serif;\n\x20\x20\x20\x20\x20\x20background:
SF:\x20#f4f4f4;\n\x20\x20\x20\x20\x20\x20text-align:\x20center;\n\x20\x20\
SF:x20\x20\x20\x20padding-top:\x2050px;\n\x20\x20\x20\x20}\n\x20\x20\x20\x
SF:20h1\x20{\n\x20\x20\x20\x20\x20\x20color:\x20#007acc;\n\x20\x20\x20\x20
SF:}\n\x20\x20\x20\x20#status\x20{\n\x20\x20\x20\x20\x20\x20font-size:\x20
SF:1\.5em;\n\x20\x20\x20\x20\x20\x20margin-top:\x2020px;\n\x20\x20\x20\x20
SF:}\n\x20\x20\x20\x20#flag\x20{\n\x20\x20\x20\x20\x20\x20margin-top:\x201
SF:0px;\n\x20\x20\x20\x20\x20\x20font-size:\x201\.3em;\n\x20\x20\x20\x20\x
SF:20\x20font-weight:\x20bold;\n\x20\x20\x20\x20\x20\x20color:\x20#c62828;
SF:\n\x20\x20\x20\x20}\n\x20\x20\x20\x20img\x20{\n\x20\x20\x20\x20\x20\x20
SF:width:\x20300px;\n\x20\x20\x20\x20\x20\x20height:\x20auto;\n\x20\x20\x2
SF:0\x20\x20\x20margin-top:\x20")%r(HTTPOptions,C7,"HTTP/1\.1\x20200\x20OK
SF:\r\nServer:\x20Werkzeug/3\.1\.3\x20Python/3\.12\.3\r\nDate:\x20Fri,\x20
SF:27\x20Jun\x202025\x2016:54:22\x20GMT\r\nContent-Type:\x20text/html;\x20
SF:charset=utf-8\r\nAllow:\x20OPTIONS,\x20GET,\x20HEAD\r\nContent-Length:\
SF:x200\r\nConnection:\x20close\r\n\r\n")%r(RTSPRequest,16C,"<!DOCTYPE\x20
SF:HTML>\n<html\x20lang=\"en\">\n\x20\x20\x20\x20<head>\n\x20\x20\x20\x20\
SF:x20\x20\x20\x20<meta\x20charset=\"utf-8\">\n\x20\x20\x20\x20\x20\x20\x2
SF:0\x20<title>Error\x20response</title>\n\x20\x20\x20\x20</head>\n\x20\x2
SF:0\x20\x20<body>\n\x20\x20\x20\x20\x20\x20\x20\x20<h1>Error\x20response<
SF:/h1>\n\x20\x20\x20\x20\x20\x20\x20\x20<p>Error\x20code:\x20400</p>\n\x2
SF:0\x20\x20\x20\x20\x20\x20\x20<p>Message:\x20Bad\x20request\x20version\x
SF:20\('RTSP/1\.0'\)\.</p>\n\x20\x20\x20\x20\x20\x20\x20\x20<p>Error\x20co
SF:de\x20explanation:\x20400\x20-\x20Bad\x20request\x20syntax\x20or\x20uns
SF:upported\x20method\.</p>\n\x20\x20\x20\x20</body>\n</html>\n");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port502-TCP:V=7.80%I=7%D=6/27%Time=685ECCE1%P=x86_64-pc-linux-gnu%r(X11
SF:Probe,12,"l\0\x0b\0\0\x03\0\x80\x01\0\0\0\0\0\x03\0\x80\x01")%r(LDAPSea
SF:rchReq,9,"0\x84\0\0\0\x03\x02\x81\x02")%r(NCP,12,"DmdT\0\x03\0\x80\x01\
SF:0\x17\0\0\0\x03\0\x80\x01")%r(ms-sql-s,9,"\x12\x01\x004\0\x03\0\x80\x01
SF:")%r(afp,12,"\0\x03\0\x01\0\x03\x06\x81\x01\0\0\0\0\0\x03\0\x80\x01");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port1880-TCP:V=7.80%I=7%D=6/27%Time=685ECCC5%P=x86_64-pc-linux-gnu%r(Ge
SF:tRequest,799,"HTTP/1\.1\x20200\x20OK\r\nAccess-Control-Allow-Origin:\x2
SF:0\*\r\nContent-Type:\x20text/html;\x20charset=utf-8\r\nContent-Length:\
SF:x201733\r\nETag:\x20W/\"6c5-hGVEFL4qpfS9qVbAlfbm9AL7VT0\"\r\nDate:\x20F
SF:ri,\x2027\x20Jun\x202025\x2016:54:27\x20GMT\r\nConnection:\x20close\r\n
SF:\r\n<!DOCTYPE\x20html>\n<html>\n<head>\n<meta\x20charset=\"utf-8\">\n<m
SF:eta\x20http-equiv=\"X-UA-Compatible\"\x20content=\"IE=edge\">\n<meta\x2
SF:0name=\"viewport\"\x20content=\"width=device-width,\x20initial-scale=1,
SF:\x20maximum-scale=1,\x20user-scalable=0\">\n<meta\x20name=\"apple-mobil
SF:e-web-app-capable\"\x20content=\"yes\">\n<meta\x20name=\"mobile-web-app
SF:-capable\"\x20content=\"yes\">\n<!--\n\x20\x20Copyright\x20OpenJS\x20Fo
SF:undation\x20and\x20other\x20contributors,\x20https://openjsf\.org/\n\n\
SF:x20\x20Licensed\x20under\x20the\x20Apache\x20License,\x20Version\x202\.
SF:0\x20\(the\x20\"License\"\);\n\x20\x20you\x20may\x20not\x20use\x20this\
SF:x20file\x20except\x20in\x20compliance\x20with\x20the\x20License\.\n\x20
SF:\x20You\x20may\x20obtain\x20a\x20copy\x20of\x20the\x20License\x20at\n\n
SF:\x20\x20http://www\.apache\.org/licenses/LICENSE-2\.0\n\n\x20\x20Unless
SF:\x20required\x20by\x20applicable\x20law\x20or\x20agreed\x20to\x20in\x20
SF:writing,\x20softwa")%r(HTTPOptions,DF,"HTTP/1\.1\x20204\x20No\x20Conten
SF:t\r\nAccess-Control-Allow-Origin:\x20\*\r\nAccess-Control-Allow-Methods
SF::\x20GET,PUT,POST,DELETE\r\nVary:\x20Access-Control-Request-Headers\r\n
SF:Content-Length:\x200\r\nDate:\x20Fri,\x2027\x20Jun\x202025\x2016:54:27\
SF:x20GMT\r\nConnection:\x20close\r\n\r\n")%r(RTSPRequest,DF,"HTTP/1\.1\x2
SF:0204\x20No\x20Content\r\nAccess-Control-Allow-Origin:\x20\*\r\nAccess-C
SF:ontrol-Allow-Methods:\x20GET,PUT,POST,DELETE\r\nVary:\x20Access-Control
SF:-Request-Headers\r\nContent-Length:\x200\r\nDate:\x20Fri,\x2027\x20Jun\
SF:x202025\x2016:54:27\x20GMT\r\nConnection:\x20close\r\n\r\n")%r(RPCCheck
SF:,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConnection:\x20close\r\n\r\n
SF:")%r(DNSVersionBindReqTCP,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nCon
SF:nection:\x20close\r\n\r\n");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port8080-TCP:V=7.80%I=7%D=6/27%Time=685ECCC0%P=x86_64-pc-linux-gnu%r(Ge
SF:tRequest,238,"HTTP/1\.1\x20302\x20FOUND\r\nServer:\x20Werkzeug/2\.3\.7\
SF:x20Python/3\.12\.3\r\nDate:\x20Fri,\x2027\x20Jun\x202025\x2016:54:22\x2
SF:0GMT\r\nContent-Type:\x20text/html;\x20charset=utf-8\r\nContent-Length:
SF:\x20199\r\nLocation:\x20/login\r\nVary:\x20Cookie\r\nSet-Cookie:\x20ses
SF:sion=eyJfZnJlc2giOmZhbHNlLCJfcGVybWFuZW50Ijp0cnVlfQ\.aF7Mvg\.hhANfvas49
SF:EjlhUCQ2CczFANejw;\x20Expires=Fri,\x2027\x20Jun\x202025\x2016:59:22\x20
SF:GMT;\x20HttpOnly;\x20Path=/\r\nConnection:\x20close\r\n\r\n<!doctype\x2
SF:0html>\n<html\x20lang=en>\n<title>Redirecting\.\.\.</title>\n<h1>Redire
SF:cting\.\.\.</h1>\n<p>You\x20should\x20be\x20redirected\x20automatically
SF:\x20to\x20the\x20target\x20URL:\x20<a\x20href=\"/login\">/login</a>\.\x
SF:20If\x20not,\x20click\x20the\x20link\.\n")%r(HTTPOptions,161,"HTTP/1\.1
SF:\x20200\x20OK\r\nServer:\x20Werkzeug/2\.3\.7\x20Python/3\.12\.3\r\nDate
SF::\x20Fri,\x2027\x20Jun\x202025\x2016:54:22\x20GMT\r\nContent-Type:\x20t
SF:ext/html;\x20charset=utf-8\r\nAllow:\x20OPTIONS,\x20HEAD,\x20GET\r\nVar
SF:y:\x20Cookie\r\nSet-Cookie:\x20session=eyJfcGVybWFuZW50Ijp0cnVlfQ\.aF7M
SF:vg\.QBiq_MeFUPGs3J2NMsIityT45Go;\x20Expires=Fri,\x2027\x20Jun\x202025\x
SF:2016:59:22\x20GMT;\x20HttpOnly;\x20Path=/\r\nContent-Length:\x200\r\nCo
SF:nnection:\x20close\r\n\r\n")%r(RTSPRequest,16C,"<!DOCTYPE\x20HTML>\n<ht
SF:ml\x20lang=\"en\">\n\x20\x20\x20\x20<head>\n\x20\x20\x20\x20\x20\x20\x2
SF:0\x20<meta\x20charset=\"utf-8\">\n\x20\x20\x20\x20\x20\x20\x20\x20<titl
SF:e>Error\x20response</title>\n\x20\x20\x20\x20</head>\n\x20\x20\x20\x20<
SF:body>\n\x20\x20\x20\x20\x20\x20\x20\x20<h1>Error\x20response</h1>\n\x20
SF:\x20\x20\x20\x20\x20\x20\x20<p>Error\x20code:\x20400</p>\n\x20\x20\x20\
SF:x20\x20\x20\x20\x20<p>Message:\x20Bad\x20request\x20version\x20\('RTSP/
SF:1\.0'\)\.</p>\n\x20\x20\x20\x20\x20\x20\x20\x20<p>Error\x20code\x20expl
SF:anation:\x20400\x20-\x20Bad\x20request\x20syntax\x20or\x20unsupported\x
SF:20method\.</p>\n\x20\x20\x20\x20</body>\n</html>\n")%r(FourOhFourReques
SF:t,21E,"HTTP/1\.1\x20404\x20NOT\x20FOUND\r\nServer:\x20Werkzeug/2\.3\.7\
SF:x20Python/3\.12\.3\r\nDate:\x20Fri,\x2027\x20Jun\x202025\x2016:54:22\x2
SF:0GMT\r\nContent-Type:\x20text/html;\x20charset=utf-8\r\nContent-Length:
SF:\x20207\r\nVary:\x20Cookie\r\nSet-Cookie:\x20session=eyJfcGVybWFuZW50Ij
SF:p0cnVlfQ\.aF7Mvg\.QBiq_MeFUPGs3J2NMsIityT45Go;\x20Expires=Fri,\x2027\x2
SF:0Jun\x202025\x2016:59:22\x20GMT;\x20HttpOnly;\x20Path=/\r\nConnection:\
SF:x20close\r\n\r\n<!doctype\x20html>\n<html\x20lang=en>\n<title>404\x20No
SF:t\x20Found</title>\n<h1>Not\x20Found</h1>\n<p>The\x20requested\x20URL\x
SF:20was\x20not\x20found\x20on\x20the\x20server\.\x20If\x20you\x20entered\
SF:x20the\x20URL\x20manually\x20please\x20check\x20your\x20spelling\x20and
SF:\x20try\x20again\.</p>\n");
MAC Address: 02:13:D1:E6:8D:43 (Unknown)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.80%E=4%D=6/27%OT=22%CT=1%CU=40022%PV=Y%DS=1%DC=D%G=Y%M=0213D1%T
OS:M=685ECD7F%P=x86_64-pc-linux-gnu)SEQ(SP=FC%GCD=1%ISR=110%TI=Z%CI=Z%II=I%
OS:TS=A)SEQ(SP=FC%GCD=1%ISR=110%TI=Z%CI=Z%TS=A)OPS(O1=M2301ST11NW7%O2=M2301
OS:ST11NW7%O3=M2301NNT11NW7%O4=M2301ST11NW7%O5=M2301ST11NW7%O6=M2301ST11)WI
OS:N(W1=F4B3%W2=F4B3%W3=F4B3%W4=F4B3%W5=F4B3%W6=F4B3)ECN(R=Y%DF=Y%T=40%W=F5
OS:07%O=M2301NNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T
OS:3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S
OS:=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R
OS:=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=40%IPL=164%UN=0%
OS:RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=S)

Network Distance: 1 hop
Service Info: OS: Linux; Device: specialized; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE
HOP RTT     ADDRESS
1   0.24 ms ip-10-10-201-61.eu-west-1.compute.internal (10.10.201.61)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 201.73 seconds

```

Analysis

```bash
80/tcp    open  http          Werkzeug/3.1.3 Python/3.12.3
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Server: Werkzeug/3.1.3 Python/3.12.3
|     Date: Fri, 27 Jun 2025 16:54:22 GMT
|     Content-Disposition: inline; filename=index.html
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 1608
|     Last-Modified: Mon, 23 Jun 2025 00:44:37 GMT
|     Cache-Control: no-cache
|     ETag: "1750639477.176184-1608-3272805344"
|     Date: Fri, 27 Jun 2025 16:54:22 GMT
|     Connection: close
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta charset="UTF-8">
|     <title>Gate Monitor</title>
|     <style>
|     body {
|     font-family: Arial, sans-serif;
|     background: #f4f4f4;
|     text-align: center;
|     padding-top: 50px;
|     color: #007acc;
|     #status {
|     font-size: 1.5em;
|     margin-top: 20px;
|     #flag {
|     margin-top: 10px;
|     font-size: 1.3em;
|     font-weight: bold;
|     color: #c62828;
|     width: 300px;
|     height: auto;
|     margin-top:
|   HTTPOptions: 
|     HTTP/1.1 200 OK
|     Server: Werkzeug/3.1.3 Python/3.12.3
|     Date: Fri, 27 Jun 2025 16:54:22 GMT
|     Content-Type: text/html; charset=utf-8
|     Allow: OPTIONS, GET, HEAD
|     Content-Length: 0
|     Connection: close
|   RTSPRequest: 
|     <!DOCTYPE HTML>
|     <html lang="en">
|     <head>
|     <meta charset="utf-8">
|     <title>Error response</title>
|     </head>
|     <body>
|     <h1>Error response</h1>
|     <p>Error code: 400</p>
|     <p>Message: Bad request version ('RTSP/1.0').</p>
|     <p>Error code explanation: 400 - Bad request syntax or unsupported method.</p>
|     </body>
|_    </html>
|_http-server-header: Werkzeug/3.1.3 Python/3.12.3
|_http-title: Gate Monitor
```

`http-title: Gate Monitor` might be of interest as well as `Werkzeug/3.1.3` might be vulnerable version. 

Cursory research shows Werkzeug/3.1.3 is NOT vulnerable.

![Pasted image 20250627120450.png](Images/Pasted%20image%2020250627120450.png)
Web server 

Page Source
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Gate Monitor</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: #f4f4f4;
      text-align: center;
      padding-top: 50px;
    }
    h1 {
      color: #007acc;
    }
    #status {
      font-size: 1.5em;
      margin-top: 20px;
    }
    #flag {
      margin-top: 10px;
      font-size: 1.3em;
      font-weight: bold;
      color: #c62828;
    }
    img {
      width: 300px;
      height: auto;
      margin-top: 30px;
      border: 4px solid #ccc;
      border-radius: 10px;
      box-shadow: 2px 2px 12px rgba(0,0,0,0.2);
    }
  </style>
</head>
<body>

  <h1>\U0001f510 Gate Status Monitor</h1>

  <img id="gate-img" src="static/unknown.png" alt="Gate Status" />
  <p id="status">Loading gate status...</p>
  <p id="flag"></p>

  <script>
    function updateGateStatus() {
      fetch('/api/gate')
        .then(response => response.json())
        .then(data => {
          document.getElementById('gate-img').src = 'static/' + data.image;
          document.getElementById('status').textContent = data.status;
          document.getElementById('flag').textContent = data.flag || '';
        })
        .catch(error => {
          document.getElementById('status').textContent = 'Error retrieving gate status';
          document.getElementById('gate-img').src = 'static/unknown.png';
          document.getElementById('flag').textContent = '';
        });
    }

    updateGateStatus(); // First load
    setInterval(updateGateStatus, 2000); // Repeat every 2 seconds
  </script>

</body>
</html>

```

`fetch('/api/gate')` Gotcha

Caido time, could just use curl but I don't like to 

![Pasted image 20250627122419.png](Images/Pasted%20image%2020250627122419.png)

Curl
```bash
root@ip-10-10-248-62:~# curl http://10.10.201.61/api/gate
{
  "image": "closed-b8ce334bae3faf976fa457702fe7545b.png",
  "status": "Gate CLOSED"
}

```

Gobuster 

```bash
root@ip-10-10-248-62:~# gobuster dir -w /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-medium-words.txt -u http://10.10.201.61
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.201.61
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-medium-words.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/console              (Status: 400) [Size: 167]
Progress: 63087 / 63088 (100.00%)
===============================================================
Finished
===============================================================

```

Gobuster `/api`

```bash
root@ip-10-10-248-62:~# gobuster dir -w /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-medium-words.txt -u http://10.10.201.61/api
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.201.61/api
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-medium-words.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/gate                 (Status: 200) [Size: 88]
Progress: 63087 / 63088 (100.00%)
===============================================================
Finished
===============================================================

```

Gobuster `/api/gate`

```bash
root@ip-10-10-248-62:~# gobuster dir -w /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-medium-words.txt -u http://10.10.201.61/api/gate
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.201.61/api/gate
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-medium-words.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
Progress: 63087 / 63088 (100.00%)
===============================================================
Finished
===============================================================

```

```http
HTTP/1.1 400 BAD REQUEST
Server: Werkzeug/3.1.3 Python/3.12.3
Date: Fri, 27 Jun 2025 17:44:18 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 167
Connection: close

<!doctype html>
<html lang=en>
<title>400 Bad Request</title>
<h1>Bad Request</h1>
<p>The browser (or proxy) sent a request that this server could not understand.</p>

```

Werkzeug debug console exposed

No progress with this. Let's try something else

## Alternate Web Server (8080)
`http://10.10.201.61:8080`

![Pasted image 20250627130701.png](Images/Pasted%20image%2020250627130701.png)

I tried a bunch of default creds. Nothing worked. Checked page source, nothing interesting.

Tried SQL injection, no dice.

### NMAP 

```bash
8080/tcp  open  http-proxy    Werkzeug/2.3.7 Python/3.12.3
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.1 404 NOT FOUND
|     Server: Werkzeug/2.3.7 Python/3.12.3
|     Date: Fri, 27 Jun 2025 16:54:22 GMT
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 207
|     Vary: Cookie
|     Set-Cookie: session=eyJfcGVybWFuZW50Ijp0cnVlfQ.aF7Mvg.QBiq_MeFUPGs3J2NMsIityT45Go; Expires=Fri, 27 Jun 2025 16:59:22 GMT; HttpOnly; Path=/
|     Connection: close
|     <!doctype html>
|     <html lang=en>
|     <title>404 Not Found</title>
|     <h1>Not Found</h1>
|     <p>The requested URL was not found on the server. If you entered the URL manually please check your spelling and try again.</p>
|   GetRequest: 
|     HTTP/1.1 302 FOUND
|     Server: Werkzeug/2.3.7 Python/3.12.3
|     Date: Fri, 27 Jun 2025 16:54:22 GMT
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 199
|     Location: /login
|     Vary: Cookie
|     Set-Cookie: session=eyJfZnJlc2giOmZhbHNlLCJfcGVybWFuZW50Ijp0cnVlfQ.aF7Mvg.hhANfvas49EjlhUCQ2CczFANejw; Expires=Fri, 27 Jun 2025 16:59:22 GMT; HttpOnly; Path=/
|     Connection: close
|     <!doctype html>
|     <html lang=en>
|     <title>Redirecting...</title>
|     <h1>Redirecting...</h1>
|     <p>You should be redirected automatically to the target URL: <a href="/login">/login</a>. If not, click the link.
|   HTTPOptions: 
|     HTTP/1.1 200 OK
|     Server: Werkzeug/2.3.7 Python/3.12.3
|     Date: Fri, 27 Jun 2025 16:54:22 GMT
|     Content-Type: text/html; charset=utf-8
|     Allow: OPTIONS, HEAD, GET
|     Vary: Cookie
|     Set-Cookie: session=eyJfcGVybWFuZW50Ijp0cnVlfQ.aF7Mvg.QBiq_MeFUPGs3J2NMsIityT45Go; Expires=Fri, 27 Jun 2025 16:59:22 GMT; HttpOnly; Path=/
|     Content-Length: 0
|     Connection: close
|   RTSPRequest: 
|     <!DOCTYPE HTML>
|     <html lang="en">
|     <head>
|     <meta charset="utf-8">
|     <title>Error response</title>
|     </head>
|     <body>
|     <h1>Error response</h1>
|     <p>Error code: 400</p>
|     <p>Message: Bad request version ('RTSP/1.0').</p>
|     <p>Error code explanation: 400 - Bad request syntax or unsupported method.</p>
|     </body>
|_    </html>
|_http-server-header: Werkzeug/2.3.7 Python/3.12.3
| http-title: Site doesn't have a title (text/html; charset=utf-8).
|_Requested resource was /login

```

`Werkzeug/2.3.7` Now this might be something for real.

Debugger not enabled.

### Gobuster

```bash
root@ip-10-10-248-62:~# gobuster dir -w /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-medium-words.txt -u http://10.10.201.61:8080
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.201.61:8080
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-medium-words.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/login                (Status: 200) [Size: 4665]
/logout               (Status: 302) [Size: 199] [--> /login]
/users                (Status: 302) [Size: 199] [--> /login]
/settings             (Status: 302) [Size: 199] [--> /login]
/dashboard            (Status: 302) [Size: 199] [--> /login]
/programs             (Status: 302) [Size: 199] [--> /login]
/monitoring           (Status: 302) [Size: 199] [--> /login]
/hardware             (Status: 302) [Size: 199] [--> /login]
Progress: 63087 / 63088 (100.00%)
===============================================================
Finished
===============================================================

```

Let's pivot again.

## Alternate Web Server (1880)

`http://10.10.201.61:1880/`

### NMAP

```bash
1880/tcp  open  vsat-control?
| fingerprint-strings: 
|   DNSVersionBindReqTCP, RPCCheck: 
|     HTTP/1.1 400 Bad Request
|     Connection: close
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Access-Control-Allow-Origin: *
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 1733
|     ETag: W/"6c5-hGVEFL4qpfS9qVbAlfbm9AL7VT0"
|     Date: Fri, 27 Jun 2025 16:54:27 GMT
|     Connection: close
|     <!DOCTYPE html>
|     <html>
|     <head>
|     <meta charset="utf-8">
|     <meta http-equiv="X-UA-Compatible" content="IE=edge">
|     <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=0">
|     <meta name="apple-mobile-web-app-capable" content="yes">
|     <meta name="mobile-web-app-capable" content="yes">
|     <!--
|     Copyright OpenJS Foundation and other contributors, https://openjsf.org/
|     Licensed under the Apache License, Version 2.0 (the "License");
|     this file except in compliance with the License.
|     obtain a copy of the License at
|     http://www.apache.org/licenses/LICENSE-2.0
|     Unless required by applicable law or agreed to in writing, softwa
|   HTTPOptions, RTSPRequest: 
|     HTTP/1.1 204 No Content
|     Access-Control-Allow-Origin: *
|     Access-Control-Allow-Methods: GET,PUT,POST,DELETE
|     Vary: Access-Control-Request-Headers
|     Content-Length: 0
|     Date: Fri, 27 Jun 2025 16:54:27 GMT
|_    Connection: close

```

### Gobuster

```bash
root@ip-10-10-248-62:~# gobuster dir -w /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-medium-words.txt -u http://10.10.201.61:1880
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.201.61:1880
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-medium-words.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/plugins              (Status: 200) [Size: 0]
/.                    (Status: 301) [Size: 151] [--> /./]
/icons                (Status: 200) [Size: 966]
/settings             (Status: 200) [Size: 917]
/theme                (Status: 200) [Size: 313]
/red                  (Status: 301) [Size: 153] [--> /red/]
/ui                   (Status: 301) [Size: 152] [--> /ui/]
/vendor               (Status: 301) [Size: 156] [--> /vendor/]
/diagnostics          (Status: 200) [Size: 1581]
/Settings             (Status: 200) [Size: 917]
/UI                   (Status: 301) [Size: 152] [--> /UI/]
/Icons                (Status: 200) [Size: 966]
/types                (Status: 301) [Size: 155] [--> /types/]
/Plugins              (Status: 200) [Size: 0]
/PlugIns              (Status: 200) [Size: 0]
/Theme                (Status: 200) [Size: 313]
/Diagnostics          (Status: 200) [Size: 1587]
/nodes                (Status: 200) [Size: 1021722]
/ICONS                (Status: 200) [Size: 966]
Progress: 63087 / 63088 (100.00%)
===============================================================
Finished
===============================================================
```

![Pasted image 20250627132522.png](Images/Pasted%20image%2020250627132522.png)

Never seen this before, but it looks like it could be related to gate control.

Can't edit without login BUT looking at function 2 

![Pasted image 20250627134239.png](Images/Pasted%20image%2020250627134239.png)

We see a comment `// Output to badge checker UI`

Now remember the Gobuster output `/ui`

![Pasted image 20250627134332.png](Images/Pasted%20image%2020250627134332.png)

Interesting! I toggle these off. Now let's check the gate again.

![Pasted image 20250627134435.png](Images/Pasted%20image%2020250627134435.png)

Let's go, first challenge complete.