---
layout: post
title:  "Hack the Box - Control"
date:   2020-02-03 00:00:00
categories: [walkthrough, htb]
comments: false
---
![HTB-Control](/img/htb/control.png)

An hard box by TRX.  Excellent privesc on this one!  I had been wanting to do this particular escalation for a while now but had not had the opporunity.  The path to user was interesting... but it did feel a little CTF like to me.  Again, enumeration is key here!

As this box is still active the walkthrough is not available.

{% highlight bash %}
PS C:\Windows\system32> whoami
nt authority\system
PS C:\Windows\system32> hostname
control
{% endhighlight %}
