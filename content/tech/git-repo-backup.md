---
title: "Backing up git repositories"
slug: "git-repo-backup"
date: "2023-05-01"
tags:
  - git
---

Backing up git repos is a must in the current tech landscape.

<!--more-->

There are times when I've wished I had saved some piece of code I wrote at a previous job or even something going as far back as when I was in university. This quick post should serve as a reminder of how to save git repositories to a USB thumb drive in a way that makes it easy to get back to at a later date.

Ensure you have:

- The repository checked out and the latest changes pulled from the server
- A USB thumb drive plugged into your computer

Use the following commands, substituting the USB thumb drive directory and project name, to create a backup of the repository.

Also be sure you are in the root directory of the repository.

```text
git init --bare /mnt/usb/project-name
git remote add backup /mnt/usb/project-name
git push backup
```

