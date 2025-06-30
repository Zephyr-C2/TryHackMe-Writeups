![[Pasted image 20250627180218.png]]

Search for virelia-water

Search records on crt.sh 

![[Pasted image 20250627180636.png]]

Found stage0.virelia-water.it.com

Serch DNS records for TXT record

```bash
dig uplink-fallback.virelia-water.it.com TXT
```

```bash
; <<>> DiG 9.20.10 <<>> uplink-fallback.virelia-water.it.com TXT  
;; global options: +cmd  
;; Got answer:  
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 8461  
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1  
  
;; OPT PSEUDOSECTION:  
; EDNS: version: 0, flags:; udp: 1232  
;; QUESTION SECTION:  
;uplink-fallback.virelia-water.it.com. IN TXT  
  
;; ANSWER SECTION:  
uplink-fallback.virelia-water.it.com. 1799 IN TXT "eyJzZXNzaW9uIjoiVC1DTjE[REDACTED]bm5lbF9jb25maXJtZWR9In0="  
  
;; Query time: 19 msec  
;; SERVER: 10.64.0.1#53(10.64.0.1) (UDP)  
;; WHEN: Fri Jun 27 18:00:47 CDT 2025  
;; MSG SIZE  rcvd: 162
```

Base64 flag 

eyJzZXNzaW9uIjoiVC1DTjEt[REDACTED]bmtfY2hhbm5lbF9jb25maXJtZWR9In0=