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
