---
title: "Solaris 11 Express SRU4 and VirtualBox 4 bad combo"
date: 2011-04-12T00:00:00+00:00
draft: false
categories: [blog, work, virtualbox, Solaris 11]
tags: [work, virtualbox, Solaris 11]
---

So I've been tracking the Solaris 11 Express Support repo updates for a while. I had issue today where I couldn't make a network connection to my virtual box guests. They would ping but no ssh and no http connection. The later being important as my VB guest is running a local mirror repo. while I could connect to the guest from other machines on the network I could connect to them from the the machine hosting the guests.

To cut the 3 hours I wasted chasing this problem turns out if I reverted my BE to the previous one... no problems everything works as expected. A quick check to the Oracle support rep sees there is now an SRU5 so I did an update nothing to lose. Update completed and my network connectivity to my guest has returned

Solaris 11 Express SRU4 not compatible with VirtualBox 4.0.4.
