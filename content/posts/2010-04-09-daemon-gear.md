---
title: "Daemon gear"
date: 2010-04-09T00:00:00+00:00
draft: false
categories: [blog, SMF, opensolaris, transmission]
tags: [SMF, opensolaris, transmission]
---

Moving right along in the continued migration to  transmission, its in and running in deamon mode on the command line, flexget is in and tested. Just need to add flexget to cron and make sure transmission-deamon starts at  system bootup. Ok SMF script needed, being an inherently lazy unix guy , I'll just google that and  and yes number one with the bullet I find this over on 4amlunch.net thanks to wonko. A  quick mod on the TRANSMISSION_HOME to suit my local setup and and transmission-daemon is running from SMF. flexget cronjob added all should be good to go. I'll wait an hour let the cron job run it's course and see what we get.
