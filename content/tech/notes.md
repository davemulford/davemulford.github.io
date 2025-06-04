---
title: "Notes"
slug: "notes"
date: "2023-08-25"
---

This post covers various notes and tricks I've accumulated over the years.

<!--more-->

## Generating randomized strings

Taken from this [StackOverflow](https://unix.stackexchange.com/questions/230673/how-to-generate-a-random-string) answer but kept here for posterity.

To generate a password-like string with all special characters, letters, and numbers. Replace the "24" in the `head` command with the desired length of the string.

```text
LC_ALL=C tr -dc 'A-Za-z0-9!"#$%&'\''()*+,-./:;<=>?@[\]^_`{|}~' </dev/urandom | head -c 24
```

To generate a string of letters and numbers only.

```text
LC_ALL=C tr -dc A-Za-z0-9 </dev/urandom | head -c 24 ; echo ''
```

## stdout/stderr printing

This is useful for determining which output file is being used by commands.

Add this to the end of a command.

```text
2> >(sed 's/^/2: /') > >(sed 's/^/1: /')
```

Example:

```text
$ ls 2> >(sed 's/^/2: /') > >(sed 's/^/1: /')
1: Desktop
1: Documents
1: Downloads
...

$ ls foo 2> >(sed 's/^/2: /') > >(sed 's/^/1: /')
2: ls: foo: No such file or directory
```

## grep: removing "not permitted" errors

To remove just errors stating "not permitted" use the following.

```text
$ grep -rni "search text" * 2> >(grep -v 'not perm' >&2)
```

## Topic TBD

The command below solves the issue in macOS where the `netstat` command does not print any process information, except the PID. Pay attention to the `awk` command's `system(...)` section on how to run a command from within awk to add contextual information to the output.

This may only work as-is on macOS but can likely be adapted to work on Linux if needed. However, the Linux `ss` command can already output this information, so this may only be helpful to show how to retrieve extra information when using awk.

```text
sudo netstat -anv | grep LISTEN | awk 'NR==1 {print "LOCAL_ADDRESS","PID","COMMAND"} {print $4,$11,system("ps -o command= -p " $11)}' | column -t
```
