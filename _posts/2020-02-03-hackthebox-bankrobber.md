---
layout: post
title:  "Hack the Box - Bankrobber"
date:   2020-02-03 00:00:00
categories: [walkthrough, htb, oswe]
comments: false
---
![HTB-Bankrobber](/img/htb/bankrobber.png)

An insane box by Gioo & Cneeliz.  One of my favorites to date.  A really, REALLY cool initial exploitation method on this one and a painful but rewarding shell experience.  The privesc was fun, although it did feel a bit CTF.   That being said this is a great box for OSWE preparation!  

As this box is still active the walkthrough is not available.

{% highlight bash %}
meterpreter > sysinfo
Computer        : BANKROBBER
OS              : Windows 10 (10.0 Build 14393).
Architecture    : x64
System Language : nl_NL
Domain          : WORKGROUP
Logged On Users : 1
Meterpreter     : x86/windows
meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
meterpreter >
{% endhighlight %}
