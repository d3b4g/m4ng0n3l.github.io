---
layout: post
title:  "Vulnhub - SecTalks: BNE0x03 - Simple"
date:   2016-06-11 16:07:19
categories: [walkthrough]
comments: false
---
Quick little VM [Simple CTF](https://www.vulnhub.com/entry/sectalks-bne0x03-simple,141/) written by [Robert Winkel](https://www.vulnhub.com/author/robert-winkel,190/), sounded like a good simple one to tackle when brushing up Web App testing.

Short, sweet, and still learned something.
<!--more-->

## Description

Simple CTF is a boot2root that focuses on the basics of web based hacking. Once you load the VM, treat it as a machine you can see on the network, i.e. you don't have physical access to this machine. Therefore, tricks like editing the VM's BIOS or Grub configuration are not allowed. Only remote attacks are permitted. /root/flag.txt is your ultimate goal.

## Host Discovery

First up, lets figure out what IP it is running under as it was configured with Host-Only I chose to enumerate only that network:

{% highlight bash %}
root@kali:~/evidence/simple# netdiscover -r 192.168.92.0/24
4 Captured ARP Req/Rep packets, from 3 hosts.   Total size: 240                                     
_____________________________________________________________________________
  IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
-----------------------------------------------------------------------------                       
192.168.92.141  00:0c:29:f0:40:c9      2     120  VMware, Inc.                                      
root@kali:~/evidence/simple# -
{% endhighlight %}

There we go, `192.168.92.141` time to enumerate.

## Host Enumeration

Starting off, lets use nmap.

{% highlight bash %}
root@kali:~/evidence/simple# nmap -sV -p 0-65535 192.168.92.138

Starting Nmap 7.25BETA2 ( https://nmap.org ) at 2016-09-20 08:17 MDT
Nmap scan report for 192.168.92.141
Host is up (0.00018s latency).
Not shown: 65535 closed ports
PORT  STATE SERVICE VERSION
80/tcp open http    Apache httpd 2.4.7 ((Ubuntu))
MAC Address: 00:0C:29:F0:40:C9 (VMware)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 1320.98 seconds
root@kali:~/evidence/simple# -
{% endhighlight %}

Looks like just 80 is open as expected.

## TCP/80 - HTTP

Surfing to port 80 reveals a simple CMS system called CuteNews v2.0.3.  Cursory recon shows no major vulnerabilities in it.  

![register](/img/simple/register.png)

I attempted some simple SQL injection attacks against the login but they were not injectable.  As there was limited interaction possbilities, I went ahead and registered for an account.  This got me access into the system with little issue.  Exploring the interface I found the `Personal Options` intriguing.  

![personal-options](/img/simple/personal-options.png)

Of course inside `Personal Options` there was a magical upload box.  Would it take a php script instead of an image?

![upload-form](/img/simple/upload-form.png)
![upload-form-with-shell](/img/simple/upload-form-with-shell.png)

Well I had a bit of an issue with my first attempt at the reverse shell.  So i elected to upload a very very simple php backdoor that takes commands from the URI string and executes them while displaying them in the browser.  I went ahead and uploaded it in the same manner and executed the `ls` command via the browser.

![backdoor-php](/img/simple/backdoor.php.png)

Then I attempted a simple cat command to display `/etc/passwd`

![backdoor-php-etc-passwd](/img/simple/backdoor-etc-passwd.png)

At this point I was going to execute a netcat listener `nc -lvp 4444 -e /bin/sh &` however I decided to look at the original reverse-shell backdoor that I had attempted before.  Yep, as I thought I typo'd the LHOST and used the wrong network.  I corrected that, re-uploaded, started my local listener and went to the webpage once again.

![reverse-shell](/img/simple/reverse-shell.png)

And I'm in, now to enumerate possible PrivEsc exploits.  I started off by uploading and running LinEnum.sh.

![linenum.sh](/img/simple/linenum.sh.png)

It grabbed a ton of useful information, but I decided to grab the kernel version and look for simple exploits on kernek-exploits.com.  A quick search for `3.16.0-30-generic` came up with two potential exploits.  Kernel-exploits.com also provided links to exploit-db.com, how convenient.

![kernel-exploit](/img/simple/kernel-exploit.png)

![exploit-db](/img/simple/exploit-db.png)

I proceeded to download and compile the exploit.  Once compiled I executed `./a.out` to gain root privilges.

![fin](/img/simple/fin.png)

Once I was root I changed diretory to `/root` and read `/root/flag.txt`. Game Over.

## End Thoughts

Ok this one was far too easy for me, but I guess I should have expected that from the name.  I suppose I was expecting more to do, but frankly it was a two part hack really.  Overall I think it took longer to write it down than actually exploit it.
