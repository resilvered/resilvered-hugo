---
title: "installing-ruby-1"
date: 2012-04-10T00:00:00+00:00
draft: false
categories: [blog, Ruby, Solaris 11, libyaml]
tags: [Ruby, Solaris 11, libyaml]
---

Installing Ruby 1.9.3 and libyaml on Solaris 11Recent blog entries would indicate I've upgrade to Solaris 11 proper or Solaris 11/11 as it goes by.  The upgrade was mostly driven by the fact I need to compile some software and figured I may as well have the most recent version rather than on the express edition. I'm looking into Vagrant at the moment as it might fit in with some of the automation bits I'm playing with. I'm not getting into Vagrant  here just yet as there are a couple of steps I need to do to first to prep my Solaris 11 environment.For the "First cloud OS" Solaris 11 is distinctly lacking some items that you could almost say are essentials in todays cloud, Ruby.  And I here the cries from the Oracle faithful we ship Ruby, Hmm Version 1.8 nearly 5 years old and only months away from EOL.  I did attempt to install Vagrant with version 1.8 and gem just failed to meet the prerequisites by 0.0.1 if I recall. None the less I had committed to building a more recent version of Ruby that I'll need for some other stuff in progress.So what do you need to build ruby 1.9.3-p125 on Solaris 11,a compiler obviously and I'm using the gcc 4.5.2 include in the Solaris 11 repo, and libyaml, otherwise we end by with this annoying message.
```
It seems your ruby installation is missing psych (for YAML output). To eliminate this warning, please install libyaml and reinstall your ruby.
```
Installing libyamlWhile vagrant didn't seem to need this I'm not big on annoying messages and who's know when I'll need it. 
```

wget http://pyyaml.org/download/libyaml/yaml-0.1.4.tar.gz
tar xzf yaml-0.1.4.tar.gz
cd yaml-0.1.4
./configure
make
sudo make install

```
Pretty straight forward no issues. Installing Ruby 1.9.3-p125
```

wget http://ftp.ruby-lang.org/pub/ruby/1.9/ruby-1.9.3-p125.tar.gz
tar xzf ruby-1.9.3-p125.tar.gz
cd ruby-1.9.3-p125
./configure --enable-shared --with-opt-dir=/usr/local/lib  
make
sudo make install

```
Worth noting libyaml couldn't be found without specifying ---with-opt-dir
```

ruby -v
ruby 1.9.3p125 (2012-02-16) [i386-solaris2.11]
gem -v 
1.8.11

```
I'm not altogether happy doing the make installs I'd like to serve these packages up via IPS. I think for that I need a proper build server environment, an IPS repo, and probably some tools to help me package that all up, sounds like an ideal use for a zone. For now it will need wait til I have some more time on my hands.
