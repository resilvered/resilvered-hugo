---
title: "How to create Mulitple Solaris Repositories"
date: 2012-06-24T00:00:00+00:00
draft: false
categories: [blog, SMF, IPS, Solaris 11, solaris]
tags: [SMF, IPS, Solaris 11, solaris]
---

Solaris 11 repo setup but you want to have your own repo serving up own home grown packages, separate from your main Solaris 11 image repository. Actually I'd like an extra two repo's I want a dev repo that to test my packaging abilities and a "release" one when I'm ready to publish. Mostly becuase there doesn't seem to be a straight forward way to remove packages from a repo and my packaging skills are work in progress. 

So here goes , a new filesystem first 

```
zfs create lake/repo/homedev
zfs list -r lake/repo
NAME                           USED  AVAIL  REFER  MOUNTPOINT
lake/repo                     18.2G   439G    26K  /repo
lake/repo/Solaris11           7.07G   439G  7.07G  /repo/Solaris11
lake/repo/Solaris_11_Express  10.5G   439G  10.2G  /repo/Solaris_11_Express
lake/repo/homedev             45.9M   439G  45.9M  /repo/homedev

```

Check which port my default repo is on 

```
svccfg -s pkg/server listprop pkg/port
pkg/port count       81
```
Create my new repo 

```
pkgrepo create /repo/homedev
pkgrepo set -s /repo/homedev  publisher/prefix=homedev
```
Create a new SMF service for my new repo 

```
svccfg -s pkg/server add homedev
svccfg -s pkg/server:homedev addpg pkg application 
svccfg -s pkg/server:homedev setprop pkg/port=82
svccfg -s pkg/server:homedev setprop pkg/inst_root=/repo/homedev/ 
svccfg -s pkg/server:homedev setprop pkg/readonly=false
svccfg -s pkg/server:homedev addpg general framework
svccfg -s pkg/server:homedev addpropvalue general/complete astring: homedev
svccfg -s pkg/server:homedev addpropvalue general/enabled boolean: true
svcadm refresh pkg/server:homedev
svcadm enable pkg/server:homedev
```

All ready to go now check out http://localhost:82 and should be running. Note reaonly=flase so this is a writeable IPS repo which is what I'm after to install my packages in.
