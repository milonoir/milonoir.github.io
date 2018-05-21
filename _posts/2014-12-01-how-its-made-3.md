---
title: How it's made? part 3
layout: post
lang: en
ref: how-its-made3
date: 2014-12-01 19:31:04 +0100
categories: archive
---

I’ve just found a solution for a problem that’s been grinding my gears for weeks, thus I’m writing about this now: backing up.

So I have the Raspberry Pi, the website is online. The Django code itself is stored in a version control system, nothing can harm it. But what about the database and static content (e.g. images)? Since it went online at the end of September, I have not had any backup strategy.

Until now! I’ve been always dreaming of syncing my non-version-controlled files into a cloud storage so I could sleep well at night. This afternoon I found the solution [on this page](http://alexbelezjaks.com/automatic-backup-to-dropbox-on-raspberry-pi-tutorial/). However, step 1, the `wget` command, didn’t work for me, so I was just better off cloning the repo [from here](https://github.com/andreafabrizi/Dropbox-Uploader), as it was written in the *Getting started* section.

If I did it right, there’s going to be a new backup file in my Dropbox every morning at 6 am.
