---
title: "Linux kickstart for Solaris Admin's"
date: 2011-11-07T00:00:00+00:00
draft: false
categories: [blog, kickstart, MSOSUG, Linux, work, virtualbox, solaris, oracle]
tags: [kickstart, MSOSUG, Linux, work, virtualbox, solaris, oracle]
---

Linux kickstart for Solaris Admin'sSo sometimes Solaris Admins need to turn their hand at another OS for various reasons, especial in this day and age of mass production of virtual environments. Solaris Admins will be well versed with jumpstart a tried and truly tested system of automated builds for  over a decade now. While recently this system has been thrown out with the bath water for AI in Solaris 11 and while the jury is still out on that one that's a conversation for another time.

So Kickstart, most of you are think Commodore Amiga right, well there is a less known kickstart for Linux or should I say Redhat Enterprise Linux RHEL and Redhat derived/like systems , CentOS, OEL, and SL, that as the name sugests performs the equivalent task for RHEL as Jumpstart does for Solaris.

I'm not going to get into the deep and dirty of Kickstart and how that work but rather provide a quick and dirty so the Solaris folk can easily find there way. For what ever reason you find yourself in this postion it's at the very least always handy to know what the young kids are up to.

So lets look at what I'm working with:
Kickstart Server built with OEL 6.1 on VirtualBox 4.1.61GB RAM built from DVD as Software Development Workstation optionConfigured with Oracle public yum repo2 NICseth0 bridged to my LAN DHCP configuredeth1 internal VirtualBox network statically allocated 10.0.2.254/data filesystem with copies of DVD install imagesKickstart test client1 NICeth0  internal VirtualBox networkTo anser the obvious question is why OEL, well couple of reason's, readily accessible to download and I'm actual giving a talk on the at MSOSUG and Oracle is providing faciliteis and refreshments so I it only seemed fair.

If you have the setup the same as mine then you should be able to use the scripts I have written to setup your kickstart server and be in business in next to know time. If you just want script without the story then head straight to the end for the download.
                      Prerequisite softwareI have chosen to do my kickstart via http but I could have used NFS or DVD , but again not an lesson in Kickstart just to get you moving quickly.
httpd - already installedxinetd - needed for tftpdtftpd syslinux - for the pxe boot filesdhcpdSome DVD images of a RHEL derivative Linux
```
yum -y install dhcp xinetd tftp-server syslinux
```
                      Setup httpdSo with that out of the way lets configure the web server /etc/httpd/conf.d/welcome.conf comment out everything to allow us to get index of the pages then start it up 

```
sed -i 's/^[< ]/#/' /etc/httpd/conf.d/welcome.conf
chkconfig httpd on
service httpd start
```

I have a /data directory where I have a local copy of the install media. So Create some symlinks in to my web server so I can make the install media available.

```
for distro in `ls /data | grep -v lost`
do 
ln -s /data/$distro /var/www/html/$distro
chcon -R -t httpd_sys_content_t /data/$distro/
done

```
                                            Setup dhcpdNext we need to setup dhcp server. Now for you Solaris folk it's the ISC dhcp server so those of you that struggled with the Solaris dhcp, it never really bothered me, you will have no problems configuring this one. You can grab my pre-caned basic dhcpd.conf to get you going. Turn it on and start it up

```
wget http://dl.dropbox.com/u/2236361/kick/dhcpd.conf -O /etc/dhcp/dhcpd.conf
chkconfig dhcpd on
service dhcpd start
```
                                      Setup tftpdTrival ftp , ok pretty straight forward two minor gotchas. The linux default tftpboot directory is /var/lib/tftptboot now being a Solaris person I though no problems just change it to /tftpboot which I did but to to make that work with selinux , which by default is on on my build and I didn't really want to change, I needed to learn about chcon, which I did. In the end I though well is meant to be a basic exercise in geeting this up and running so perhaps just leave it all where it is, works nicely and less hassle. If you need the sym link /tftpboot

```
chkconfig tftp on
chkconfig xinetd on
service xinetd restart
```

The second "gotcha" iptables, again on by default on my build rather than just trun it off add port 69 and you are in business. The Syntax for iptables has changed between what is supplied with 6.x as to that of 5.x so just be aware if you have and older build. While im here add in the httpd server also.

```
iptables -D INPUT -j REJECT --reject-with icmp-host-prohibited
iptables -A INPUT -p udp -m udp --dport 69 -j ACCEPT
iptables -A INPUT -p tcp -m tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp -m tcp --dport 443 -j ACCEPT
iptables -A INPUT -j REJECT --reject-with icmp-host-prohibited
service iptables save
service iptables restart
```
                          PXE boot filesAh yes I aways seem to get this wrong with Solaris for X86 , maybe is was just messing around with dhcp macros. Anyway a couple of files that are needed, make sure you have installed the syslinux package, you will also need to setup kernel and initial Ram disk pxe files. Now you can't install a 6.x OS from a 5.x pxeboot and vice versa so you need to make sure you have the right files from the right distro. I'm running a couple of different installs from my kickstart server so I have a simple shell loop to take care of setting that up for me.

```
mkdir -p /var/lib/tftpboot/pxelinux.cfg
cp -p /usr/share/syslinux/pxelinux.0 /var/lib/tftpboot
cp -p /usr/share/syslinux/menu.c32 /var/lib/tftpboot

for distro in `ls /data | grep -v lost`
do
mkdir -p /var/lib/tftpboot/$distro
cp -p /var/www/html/$distro/images/pxeboot/initrd.img /var/lib/tftpboot/$distro/
cp -p /var/www/html/$distro/images/pxeboot/vmlinuz /var/lib/tftpboot/$distro/
done
```

For the last part just prepare your pxelinux config file or what is the essentially the boot menu 

```
cat > /var/lib/tftpboot/pxelinux.cfg/default<<EOF
default menu.c32
MENU TITLE Linux Kickstart
label oel57boot
 MENU LABEL Install OEL 5.7 from http://10.0.2.254/oel57.cfg 
 kernel oel5.7/vmlinuz
 append initrd=oel5.7/initrd.img ks=http://10.0.2.254/oel57.cfg ksdevice=eth0
label oel61boot
 MENU LABEL Install OEL 6.1 from http://10.0.2.254/oel61.cfg 
 kernel oel6.1/vmlinuz
 append initrd=oel6.1/initrd.img ks=http://10.0.2.254/oel61.cfg ksdevice=eth0
PROMPT 0
TIMEOUT 0
EOF
```
or just grab my pre-canned one with a few bits setup on it.  

```
wget http://dl.dropbox.com/u/2236361/kick/default -O /var/lib/tftpboot/pxelinux.cfg/default
```
