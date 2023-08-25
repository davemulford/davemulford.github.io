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
