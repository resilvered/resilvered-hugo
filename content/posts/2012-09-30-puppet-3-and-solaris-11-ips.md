---
title: "Puppet 3 and Solaris 11 IPS"
date: 2012-09-30T00:00:00+00:00
draft: false
categories: [blog, Ruby, puppet, work, IPS, opensolaris, Solaris 11, solaris]
tags: [Ruby, puppet, work, IPS, opensolaris, Solaris 11, solaris]
---

A recent post Solaris Packages for Puppet 3.0.0-rc7, Facter 1.6.0 and Hiera 1.0.0 on a puppet mailing list caught my eye.

While gems has a nifty little package management system of it's own,  it is possible be that your machines do not have direct access to the interent so having a local IPS repo is handy. I'll demonstrate how to add these packages to you own repository. This assumes you have already setup your own local repo.

First up retrive the IPS package file from the Puppet Labs download site.

```
wget http://downloads.puppetlabs.com/solaris/facter@1.6.12,5.11-819.p5p
wget http://downloads.puppetlabs.com/solaris/hiera@1.0.0,5.11-116.p5p
wget http://downloads.puppetlabs.com/solaris/puppet@3.0.0,5.11-9211.p5p

```

Upload them to you local repository:

```
pkgrecv -s ./facter\@1.6.12\,5.11-819.p5p -d /repo/software/ '*'
pkgrecv -s ./hiera\@1.0.0\,5.11-116.p5p -d /repo/software/ '*'
pkgrecv -s ./puppet\@3.0.0\,5.11-9211.p5p -d /repo/software/ '*'

```

And then restart the local repository, note for me I had to do disable and enable, a refresh did not appear to pickup the new publisher.

```
svcadm disable svc:/application/pkg/server:homeips
svcadm enable svc:/application/pkg/server:homeips

pkgrepo list -s http://192.168.0.30:82 -p puppetlabs.com
PUBLISHER      NAME                                          O VERSION
puppetlabs.com application/facter                              1.6.12,5.11-819:20120921T230613Z
puppetlabs.com application/hiera                               1.0.0,5.11-116:20120921T230233Z
puppetlabs.com system/management/puppet                        3.0.0,5.11-9211:20120921T230759Z

```

Once confirmed, add the publisher check again and install

```
pkg set-publisher -p http://192.168.0.30:82/puppetlabs.com puppetlabs.com

pkg list -n facter hiera puppet
NAME (PUBLISHER)                                  VERSION                    IFO
application/facter (puppetlabs.com)               1.6.12-819                 ---
application/hiera (puppetlabs.com)                1.0.0-116                  ---
system/management/puppet (puppetlabs.com)         3.0.0-9211                 ---

pkg install facter hiera puppet

```

This would also install ruby-18 as that is a prerequisite package. So there you have Puppet 3.0 available in your own IPS repository.
