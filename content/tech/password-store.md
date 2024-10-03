---
title: "Using password store"
slug: "password-store"
date: "2024-10-02"
tags:
  - git
  - pass
---

Notes on using password store.

<!--more-->

To generate a new gpg key.

```text
gpg --gen-key
```

Enter real name:
Email address:

Enter a passphrase that you'll remember and is difficult to crack.

Copy the key-id and save it somewhere.

Set the key to never expire.

```text
gpg --edit-key <key-id>

gpg> expireo

    [Follow prompts to set expiration to never]

gpg> save
```

Inititalize a pass repo

```text
pass init <key-id>
```

Turn the password store into a git repo.

```text
pass git init
```

To insert a password.

```text
pass insert <server-name>

ex: pass insert github
    Enter password for github:
    Retype password for github:
```

To generate a password.

```text
pass generate aws

The generated password for aws is:
Hlskjsd%lsdjf1323242kljklj
```

To generate a password under a directory.

```text
pass generate github/personal
```

Find passwords.

```text
pass find github
Search terms: github
|- github
|  |- personal2
|  |- personal
|- github
```

List password names

```text
pass
Password store
|- aws
|  |- personal
|- aws
|- github
|  |- personal
|  |- work
|- github
```

To show a password. Command completion is available for most shells.

```text
pass show aws/personal
```

Delete a password.

```text
pass rm aws/personal
```

Add remote git repo and push passwords to it.

```text
pass git remote add origin git@github.com:davemulford/pwd-store.git
pass git push origin main
```

## Tips

Do not use your account name as the password name. This is used as the filename in the git repo and could be a partial credential leak. Instead, use the terms "github/personal" or "aws/work" and use attributes inside the password file as shown below. The attributes are encrypted because they are inside the file.

```text
pass edit aws/work
Hlskjsd%lsdjf1323242kljklj
email: user@gmail.com
```

Find a password file with a certain email address in it.

```text
pass grep "user@gmail.com"
aws/work
email: user@gmail.com
```

Copy a password to your clipboard.

```text
pass show -c aws/work
```

View git log of your password store.

```text
pass git log
```

Revert deletion of a password.

```text
pass git revert HEAD
```

## Load passwords onto another machine.

git clone git@github.com:davemulford/pwd-store.git $HOME/.password-store

## Export GPG keys from previous machine

```text
mkdir exported-keys
cd exported-keys

gpg --output public.pgp --armor --export user@gmail.com

gpg --output private.pgp --armor --export-secret-key user@gmail.com
Enter password:
```

Copy these files to the new machine using scp or some other file copying method. Import them using gpg.

```text
gpg --import private.pgp
Enter password:

gpg --import public.pgp
```
