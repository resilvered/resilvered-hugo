---
title: "Moving an iscsi zvol to a new pool"
date: 2010-09-20T00:00:00+00:00
draft: false
categories: [blog, zfs, iscsi]
tags: [zfs, iscsi]
---

Been working on fixing my 4K sector disk write performance issue, will post something on that later. So I have create a new zpool and after zfs send | zfs receive on the various zfs volumes and filesystem I was left to figure out how to move my iscsi guid's as follows
