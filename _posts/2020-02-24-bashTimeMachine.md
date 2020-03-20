---
layout: post
title:  "Bash Time Machine"
categories: [bash]
tags: [backup]
---

Backup is an important aspect of data security.  Users create backups of physical or cloud based machines and data just in case some unforeseen accident should occur.  Several notable backup solutions are available, to include Apple's *Time Machine*, Windows 10 *File History*, and Ubuntu *Backups*.  

The secret sauce to a good backup solution is to make incremental snapshots of any file that has changed, while not duplicating files that haven't changed.  The easiest way to do this is with hard links to unchanged files.  Hardlinks allow a user to save files once, have incremental snapshots of their entire machine or directory, and then to only fully delete a file when the final hard link is severed.  

In the code below, I illustrate how to do this with a very simple bash script.  This script can be deployed in the terminal on Mac or Ubuntu or in Windows Subsystem for Linux (WSL).  I recommend only using this script as a backup solution if you're fairly knowledgeable about bash scripting and understand hard links.  

  #!/bin/sh
  date=`date "+%Y%m%d%H%M%S"`
  rsync -aPW --link-dest=/backup_drive/Backups/current --exclude=".*/"  --exclude-from='exclude_me.txt'  /directory/to/backup  /backup_drive/Backups/back-$date
  rm -f /backup_drive/Backups/current
  ln -s back-$date /backup_drive/Backups/current
