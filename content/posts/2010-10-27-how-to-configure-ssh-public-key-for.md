---
title: "How to configure ssh public key for XSCF"
date: 2010-10-27T00:00:00+00:00
draft: false
categories: [blog, sun, solaris]
tags: [sun, solaris]
---

XSCF> showssh -c pubkey -u admin
XSCF> setssh -c addpubkey -u admin
Please input a public key:
ssh-rsaAAAAB3NzaC1yc2EAAAABIwAAAIEAn5DZLAtDtan37waaAPkhU9b40q0CP5HCtqULaI+V0YU/iDpxeu0/RmFRgYSXdbniAr9wBAWn+vOUmwmyZJoIwgqCN88VJoa+U9g34B2WSMsBqIYleMsvQiBStnnkZgNY/OmDrmq1VU10H8QMw0kCmIQ3Nss7CmR0/E/9tJ6+b6E=

looks like you need a Control D here

XSCF> showssh -c pubkey -u unixadmin
Public key:
     1    ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAIEAn5DZLAtDtan37waaAPkhU9b40q0CP5HCtqULaI+V0YU/iDpxeu0/RmFRgYSXdbniAr9wBAWn+vOUmwmyZJoIwgqCN88VJoa+U9g34B2WSMsBqIYleMsvQiBStnnkZgNY/OmDrmq1VU10H8QMw0kCmIQ3Nss7CmR0/E/9tJ6+b6E=
