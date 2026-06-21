---
title: "Installing Jekyll on CentOS 7"
date: 2014-08-26T00:00:00+00:00
draft: false
categories: [blog]
tags: [Jekyll, Linux, CentOS]
---

Having recently move to [Jekyll] as my blogging platform, I thought I would take a quick look at how easy it is to get going on Linux.

Installation on [CentOS 7] is very straight forward. First add the EPEL repository for [CentOS 7] as this is where you will find the [nodejs](http://nodejs.org/) rpm

	sudo yum install http://download.fedoraproject.org/pub/epel/beta/7/x86_64/epel-release-7-0.2.noarch.rpm

Install the required packages, *note npm is not required but handy to have around*.

	sudo yum install nodejs npm ruby ruby-devel rubygems git
	gem install jekyll

Start using [Jekyll].

```sh
jekyll new my-awesome-site
cd my-awesome-site/
jekyll serve
```

[Jekyll]:http://jekyllrb.com/
[CentOS 7]:http://www.centos.org/
