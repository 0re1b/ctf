# Level06

## Task

>The flag06 account credentials came from a legacy unix system.

>To do this level, log in as the level06 account with the password level06. Files for this level can be found in /home/flag06.

## Solution

```
level06@nebula:/home/flag06$ cat /etc/passwd | grep flag06
flag06:ueqwOCnSGdsuM:993:993::/home/flag06:/bin/sh
```

putting that into john

```
{17-01-09 10:13}10d1336738e8:/opt/Data root# emacs flag06passwd
{17-01-09 10:13}10d1336738e8:/opt/Data root# john ./flag06passwd
Created directory: /root/.john
Using default input encoding: UTF-8
Loaded 1 password hash (descrypt, traditional crypt(3) [DES 128/128 AVX-16])
Press 'q' or Ctrl-C to abort, almost any other key for status
hello            (flag06)
1g 0:00:00:00 DONE 2/3 (2017-01-09 10:13) 4.761g/s 3571p/s 3571c/s 3571C/s 123456..marley
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

password is `hello`

```
level06@nebula:/home/flag06$ su flag06
Password: hello
sh-4.2$ getflag
You have successfully executed getflag on a target account
```
