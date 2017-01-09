# level00

## Task

>This level requires you to find a Set User ID program that will run as the “flag00” account. You could also find this by carefully looking in top level directories in / for suspicious looking directories.


## Solution

`flag00@nebula:~$ find / -xdev \( -perm -4000 \) -type f -print0 | xargs -0 ls -l`

gives the following

```
-rwsr-x--- 1 flag00  level00      7358 2011-11-20 21:22 /bin/.../flag00
-rwsr-xr-x 1 root    root        26260 2011-05-18 03:12 /bin/fusermount
-rwsr-xr-x 1 root    root        88716 2011-08-09 09:15 /bin/mount
-rwsr-xr-x 1 root    root        34740 2011-05-03 03:38 /bin/ping
-rwsr-xr-x 1 root    root        39116 2011-05-03 03:38 /bin/ping6
-rwsr-xr-x 1 root    root        31116 2011-06-24 02:37 /bin/su
-rwsr-xr-x 1 root    root        63592 2011-08-09 09:15 /bin/umount
-rwsr-xr-x 1 root    root        13904 2011-11-13 22:10 /sbin/mount.ecryptfs_private
-rwsr-sr-x 1 daemon  daemon      42736 2011-05-16 03:31 /usr/bin/at
-rwsr-xr-x 1 root    root        40292 2011-06-24 02:36 /usr/bin/chfn
-rwsr-xr-x 1 root    root        31748 2011-06-24 02:36 /usr/bin/chsh
-rwsr-xr-x 1 root    root        57956 2011-06-24 02:36 /usr/bin/gpasswd
-rwsr-xr-x 1 root    root        56208 2011-07-28 13:02 /usr/bin/mtr
-rwsr-xr-x 1 root    root        30896 2011-06-24 02:37 /usr/bin/newgrp
-rwsr-xr-x 1 root    root        41284 2011-06-24 02:36 /usr/bin/passwd
-rwsr-xr-x 2 root    root       156824 2011-09-11 12:06 /usr/bin/sudo
-rwsr-xr-x 2 root    root       156824 2011-09-11 12:06 /usr/bin/sudoedit
-rwsr-xr-x 1 root    root        14012 2011-05-03 03:38 /usr/bin/traceroute6.iputils
-rwsr-xr-- 1 root    messagebus 312728 2011-09-02 02:31 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-xr-x 1 root    root         5564 2011-04-30 08:02 /usr/lib/eject/dmcrypt-get-device
-rwsr-xr-x 1 root    root       243864 2011-07-29 09:02 /usr/lib/openssh/ssh-keysign
-rwsr-xr-x 1 root    root         9732 2011-10-04 15:08 /usr/lib/pt_chown
-r-sr-xr-x 1 root    root         9532 2011-11-20 17:35 /usr/lib/vmware-tools/bin32/vmware-user-suid-wrapper
-r-sr-xr-x 1 root    root        10224 2011-11-20 17:35 /usr/lib/vmware-tools/bin64/vmware-user-suid-wrapper
-rwsr-xr-- 1 root    dip        273272 2011-02-04 00:43 /usr/sbin/pppd
-rwsr-sr-x 1 libuuid libuuid     13860 2011-08-09 09:15 /usr/sbin/uuidd
```

running the only file with flag00 in the name

`flag00@nebula:~$ /bin/.../flag00`

```
Congrats, now run getflag to get your flag!
```

so i run `getflag`

`flag00@nebula:~$ getflag`

```
You have successfully executed getflag on a target account
```
