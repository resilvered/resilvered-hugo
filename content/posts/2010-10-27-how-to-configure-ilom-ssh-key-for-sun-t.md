---
title: "How to configure iLOM ssh key for SUN T-series"
date: 2010-10-27T00:00:00+00:00
draft: false
categories: [blog, sun, work, solaris]
tags: [sun, work, solaris]
---

```
Note if you do not have at least Version 3.0.6.1.e r49743 firmware then you will run into this bug
6823488 Cannot add an SSH key for automatic password authentication.

> ssh root@ilom
Password: 
Waiting for daemons to initialize...

Daemons ready

Sun(TM) Integrated Lights Out Manager

Version 3.0.6.1.e r49743

Copyright 2009 Sun Microsystems, Inc. All rights reserved.
Use is subject to license terms.

-> cd /SP/users/admin/ssh/keys/1
/SP/users/admin/ssh/keys/1

-> set load_uri=http://192.168.0.1/sshkey.pub
Load successful.
```
