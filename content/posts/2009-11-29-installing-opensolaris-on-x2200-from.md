---
title: "Installing OpenSolaris on X2200 from USB drive"
date: 2009-11-29T00:00:00+00:00
draft: false
categories: [blog, sun, x2200, opensolaris, usb]
tags: [sun, x2200, opensolaris, usb]
---

I recently aquired an X2200 with no optical drive, and having had a few questions from people about USB media for OpenSolaris , where my standard answer was to download the USB image, I decided it was time I should eat my own dog food as they say and take this USB stuff out for a test drive.

What you will need:
A USB key. I used an el cheap 1GB stick I had lying about.An OpenSolaris USB image I used build 127 from genunix. Of course you can role your own from the iso.
Some way of getting the USB image on to the USB stick. For me this would be my existing OpenSolaris desktop machine.A x86 machine capable of booting from a USB device. I my case this was a SUN a dual CPU quad core X2200Assuming you have the USB stick and downloaded the image I won't bore you with the details on why you can't just copy the image to the stick. Just know you can't.

usbcopy, if you don't already have it installed you will need to install it:

```
pfexec pkg install SUNWdistro-const
```
Next you should disable hal for the time being, see bug id 11654

```
pfexec svcadm disable hal
```
Insert you usb key start the copy , make yourself a coffee.

```
pfexec usbcopy /Images/ISO/osol-dev-127-x86.usb
Found the following USB devices:
0:    /dev/rdsk/c19t0d0p0    3.9 GB    Generic  STORAGE DEVICE   9722
1:    /dev/rdsk/c19t0d1p0     Generic    STORAGE DEVICE   9722
2:    /dev/rdsk/c19t0d2p0     Generic    STORAGE DEVICE   9722
3:    /dev/rdsk/c19t0d3p0     Generic    STORAGE DEVICE   9722
4:    /dev/rdsk/c21t0d0p0    984.0 MB    410      RamBo USB 2.0    PMAP
Enter the number of your choice: 4

WARNING: All data on your USB storage will be lost.
Are you sure you want to install to
410 RamBo USB 2.0 PMAP, 984 MB at /dev/rdsk/c21t0d0p0 ?  (y/n) y
Copying and verifying image to USB device
Finished 936 MB in 1734 seconds (0.5MB/s)
0 block(s) re-written due to verification failure
Installing grub to USB device /dev/rdsk/c21t0d0s0
Completed copy to USB
```
For those of you having the misfortune to still be using Windows try out OpenSolaris Live USB Creator (Windows/.NET) . Linux folk no doubt have a tool for this too and I'll leave it you you to find out.

Ok on to the actual install. I wasted considerable time poking about the net on information about booting the X2200 from USB device. What a I found  was information like, it's too hard , doesn't work, you need to use ELOM KVM-over-IP and mount a remote image. Ergh , quiche eating Linux weenies, waste of time I should have gone with my instinct and previous, although some what limited, x86 SUN experience and just turned the thing on. Basically it just works, no fuss no hassle.

If you do need to go poking around the ELOM and have some familiarity with SUN's other Lom devices then I'd suggest http://brandonhutchinson.com/wiki/ELOM_Notes as a handy reference. If you do have prior experience with SUN's other LOM device's you know full well they like to make things just that little bit different on each machine just to keep you on your toes! If thats not enough information read the ELOM manual.

Getting your console. There are a couple of different ways to get your console on X2200 I choose, in part because I hadn't used it before, to use the ELOM WEB interface method. The consoles redirection in the ELOM was going to give me a virtual graphic view of the console I thought this easiest . So plug the net management port into my network and let it DHCP to find an IP, point the web browser at it, and very familiar looking SUN Microsystems management GUI, login and go to the redirection tab and locate the console redirection button. Very shortly after this I have window showing me the console and my OpenSolaris live USB image is already at a the prompt waiting for me select keyboard and language, that easy. 12 minutes later OpenSolaris installed and running.

Now what to do with dual CPU quad core noise machine?
